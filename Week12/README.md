# ComputerAsAMedium
### This is a repo for the Computer As A Medium Fall 2026.

### Group Name: [Mac and Patch]

### Group Members: [Amir and Deanna]


## Bullet Task - Dea


using UnityEngine;

public class Bullet : MonoBehaviour
{
    public float speed = 10f;
    public float destroyTime = 3f;
    public int damage = 10;

    public Rigidbody2D rb;
    // Start is called once before the first execution of Update after the MonoBehaviour is created
    void Start()
    {
        if (rb == null)
        {
            rb = GetComponent<Rigidbody2D>();
        }

        rb.linearVelocity = transform.up * speed;
        Destroy(gameObject, destroyTime);
    }

    void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Enemy"))
        {
            Enemy enemy = other.GetComponent<Enemy>();
            if (enemy != null)
            {
                enemy.TakeDamage(damage);
            }
            Destroy(gameObject);
        }
        else if (other.CompareTag("Wall"))
        {
            Destroy(gameObject);
        }
    }
}





## Cooroutine Player Health Stats - Dea


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



