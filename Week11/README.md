# ComputerAsAMedium
### This is a repo for the Computer As A Medium Fall 2026.

### Group Name: [Mac and Patch]

### Group Members: [Amir and Deanna]


## These are the changes to the Player Stats - Dea

using UnityEngine;
using TMPro;
using UnityEngine.SceneManagement;
using System.Collections;

public class PlayerStats : MonoBehaviour
{
    public int maxHealth = 100;
    public int maxMana = 100;
    public TMP_Text healthText;
    public TMP_Text manaText;
    public TMP_Text messageText;

    public float manaRegenDelay = 1f;
    public int manaRegenAmount = 2;

    public int Health { get; private set; }
    public int Mana { get; private set; }
    bool dead;

    void Start()
    {
        Health = maxHealth;
        Mana = maxMana;
        if (messageText) messageText.text = "";
        UpdateUI();
        StartCoroutine(ManaRegen());

    }


    IEnumerator ManaRegen()
    {
        while (true)
        {
            yield return new WaitForSeconds(manaRegenDelay);
            if (!dead && Mana < maxMana)
            {
                Mana = Mathf.Min(maxMana, Mana + manaRegenAmount);
                UpdateUI();
            }
        }
    }

    public void TakeDamage(int amount)
    {
        if (dead) return;
        Health = Mathf.Max(0, Health - amount);
        UpdateUI();
        StartCoroutine(HurtFlash());
        if (Health == 0) StartCoroutine(Die());
    }

    public bool Heal(int amount)
    {
        if (Health >= maxHealth) return false;
        Health = Mathf.Min(maxHealth, Health + amount);
        UpdateUI();
        return true;
    }

    public bool UseMana(int amount)
    {
        if (Mana < amount) return false;
        Mana -= amount;
        UpdateUI();
        return true;
    }


    IEnumerator HurtFlash()
    {
        var sr = GetComponentInChildren<SpriteRenderer>();
        if (!sr) yield break;
        sr.color = Color.red;
        yield return new WaitForSeconds(0.1f);
        sr.color = Color.white;
    }

    IEnumerator Die()
    {
        dead = true;
        if (messageText) messageText.text = "YOU DIED";
        var controller = GetComponent<PlayerController>();
        if (controller) controller.enabled = false;
        yield return new WaitForSeconds(2f);
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }


    void UpdateUI()

    {
        if (healthText) healthText.text = "Health: " + Health;
        if (manaText) manaText.text = "Mana: " + Mana;
    }

}




## Player Controller Script Changes - Dea

using UnityEngine;
using System.Collections;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    [Header("Movement")]
    public float moveSpeed = 5f;

    [Header("Direction Sprites")]
    public Sprite spriteUp;
    public Sprite spriteDown;
    public Sprite spriteLeft;
    public Sprite spriteRight;

    [Header("Weapon")]
    public Transform weapon;
    public Transform firePoint;

    [Header("Shooting")]
    public GameObject bulletPrefab;
    public float fireCooldown = 0.25f;

    [Header("Mana spell (R key)")]
    public int spellManaCost = 10;
    public int spellBullets = 8;
    public Animator castAnimator;

    Rigidbody2D rb;
    SpriteRenderer sr;
    PlayerStats stats;
    Vector2 move;
    Vector2 facing = Vector2.down;
    float nextFire;

     void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        sr = GetComponentInChildren<SpriteRenderer>();
        stats = GetComponent<PlayerStats>();

    }


    void Start() { ApplyFacing(); }

    void Update()
    {
        var kb = Keyboard.current;
        if (kb == null) return;

        move = Vector2.zero;
        if (kb.wKey.isPressed) move.y += 1;
        if (kb.sKey.isPressed) move.y -= 1;
        if (kb.dKey.isPressed) move.x += 1;
        if (kb.aKey.isPressed) move.x -= 1;
        move = move.normalized;

        if (move != Vector2.zero)
        {
            facing = Mathf.Abs(move.x) >= Mathf.Abs(move.y)
                ? new Vector2(Mathf.Sign(move.x), 0)
                : new Vector2(0, Mathf.Sign(move.y));
            ApplyFacing();

        }


        bool shootHeld = kb.spaceKey.isPressed ||
            (Mouse.current != null && Mouse.current.leftButton.isPressed);
        if (shootHeld && Time.time >= nextFire)
        {
            nextFire = Time.time + fireCooldown;
            Shoot();
        }

        if (kb.rKey.wasPressedThisFrame)
            CastSpell();
    }

    void FixedUpdate()
    {
        rb.MovePosition(rb.position + move * moveSpeed * Time.fixedDeltaTime);
    }

    void ApplyFacing()
    {
        if (facing.y > 0) sr.sprite = spriteUp;
        else if (facing.y < 0) sr.sprite = spriteDown;
        else if (facing.x < 0) sr.sprite = spriteLeft;
        else sr.sprite = spriteRight;

        float angle = Mathf.Atan2(facing.y, facing.x) * Mathf.Rad2Deg;
        weapon.rotation = Quaternion.Euler(0, 0, angle);
    }

    void Shoot()
    {
        Instantiate(bulletPrefab, firePoint.position, weapon.rotation);
    }

    void CastSpell()
    {
        if (!stats.UseMana(spellManaCost))
        {
            Debug.Log("Not enough Mana!");
            return;
        }
        if (castAnimator) castAnimator.SetTrigger("Cast");
        StartCoroutine(CastPop());

        for (int i = 0; i < spellBullets; i++)
        {
            float a = i * (360f / spellBullets);
            Instantiate(bulletPrefab, transform.position, Quaternion.Euler(0, 0, a));
        }
    }

    IEnumerator CastPop()
    {
        Vector3 start = sr.transform.localScale;
        sr.color = new Color(0.7f, 0.5f, 1f);
        sr.transform.localScale = start * 1.25f;
        yield return new WaitForSeconds(0.15f);
        sr.transform.localScale = start;
        sr.color = Color.white;
    }    


}

## Amir - Traps

using UnityEngine;

public class Trap : MonoBehaviour
{
    public GameObject[] hiddenObjects;
    public int damage = 10;
    public bool triggeredOnce = true;

    bool sprung;

    void OnTriggerEnter2D(Collider2D other)
    {
        if (!other.CompareTag("Player")) return;
        if (triggeredOnce && sprung) return;

        sprung = true;

        for (int i = 0; i < hiddenObjects.Length; i++)
        {
            hiddenObjects[i].SetActive(true);
        }

        PlayerStats stats = other.GetComponent<PlayerStats>();
        if (stats != null)
        {
            stats.TakeDamage(damage);
        }
    }
}

## Player Health Changes - Dea

using TMPro;
using UnityEngine;

public class PlayerHealth : MonoBehaviour
{
    public int health = 100;
    public TMP_Text healthText;
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        healthText.text = "Health: " + health;
    }

    // Update is called once per frame
    void Update()
    {
        healthText.text = "Health: " + health;
    }
}


