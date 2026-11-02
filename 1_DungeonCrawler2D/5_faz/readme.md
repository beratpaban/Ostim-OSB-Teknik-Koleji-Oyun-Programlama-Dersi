# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 5

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Evrensel Can Sistemi (`Health.cs`), Dokunulmazlık Süresi (I-Frames), Gelişmiş Düşman Devriye Yapay Zekası (`EnemyAI.cs` - Gezin/Bekle/Kovala), Hayalet Süzülme Animasyonu (Floating Sine Wave), Beyaz & Yeşil Hayalet, Fare ve Tek Gözlü Canavar Prefabları

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 4'te zindanımızı kırılabilir ahşap kasalar, antik testiler, açılan hazine sandıkları ve toplanabilir altınlarla doldurduk. Ancak zindanımızda henüz bizi tehdit eden, odalarda canlı gibi kendi etrafında gezinen, bekleyen ve bizi görünce peşimize düşen canavarlarımız yok.

Bu kılavuzu tamamladığınızda:

1. **Evrensel Can Sistemi (`Health.cs`):** Hem şövalyeye hem de tüm canavarlara takılabilen; `IDamageable` sözleşmesini uygulayan, hasar aldığında kırmızı renkte parlayan (`Damage Flash`) profesyonel bir sağlık bileşeni yazacaksınız.
2. **Dokunulmazlık Süresi (I-Frames - Invulnerability Frames):** Karakter bir canavara dokunduğunda saliseler içinde 50 kere hasar alıp anında ölmesini engelleyen, klasik oyunlardaki "hasar sonrası geçici dokunulmazlık" mekaniğini kuracaksınız.
3. **Akıllı Devriye ve Takip Yapay Zekası (`EnemyAI.cs`):**
   - Canavarlar put gibi beklemez: Oyuncu ortalıkta yokken kendi etrafında 1.5 saniye gezinir (**Wandering**), 1.5 saniye durup etrafı dinler (**Idle**), sonra tekrar gezinir.
   - Oyuncu kırmızı radar menziline (`detectionRadius`) girdiği anda hemen kovalama moduna (**Chasing**) geçer ve şövalyenin üstüne koşar!
4. **Hayalet Süzülme Animasyonu (Floating Sine Wave):**
   - Hayaletler yere basarak dümdüz yürümez! Matematiksel sinüs dalgası (`Mathf.Sin`) formülüyle, bize doğru gelirken havada aşağı-yukarı süzülerek gerçek bir hortlak gibi dalgalanır.
5. **Paketteki Gerçek Yaratıklarla Düşman Çeşitliliği:**
   - **Beyaz Zindan Hayaleti (`Enemy_WhiteGhost` - `tile_0121`):** 3 Can, havada süzülen dalgalı hortlak.
   - **Yeşil Balçık Hayaleti (`Enemy_GreenGhost` - `tile_0108`):** 2 Can, zehirli süzülen hayalet.
   - **Hızlı Zindan Faresi (`Enemy_Rat` - `tile_0123`):** 1 Can, yerde fırıl fırıl hızlı koşan kemirgen.
   - **Tek Gözlü Canavar / Tepegöz (`Enemy_Cyclops` - `tile_0109`):** 5 Can, ağır adımlarla yürüyen güçlü mini boss!
6. **Düşmanlardan Ganimet Saçılması:** Canavarlar kılıç darbeleriyle öldüğünde, Faz 4'te hazırladığımız altınların (`Coin_Gold`) zıplayarak yere düşmesini sağlayacaksınız.

---

## 1. Zihinsel Model: Düşman Durum Makinesi (State Machine)

Canavarlarımızın yapay zekası 3 farklı zihinsel durumda çalışır:

```text
                  [ IDLE (Durup Bekleme) ]
                         │          ▲
            1.5 sn sonra │          │ 1.5 sn sonra
                         ▼          │
                 [ WANDERING (Kendi Etrafında Gezinme) ]
                         │
                         │ Oyuncu Radara Girdi! (Menzil <= 5m)
                         ▼
                  [ CHASING (Oyuncuyu Kovala!) ]
                         │
        (Hayalet ise: Y ekseninde sinüs dalgasıyla süzül!)
                         │
                         ▼
        Oyuncuya Temas: 1 Hasar | Kılıç Darbesi: Can Azalır!
```

---

## 2. Kullanılacak Gerçek Görseller ve Özellik Tablosu

Kenney Tiny Dungeon paketinde yer alan gerçek düşman sprite'ları ve yapay zeka parametreleri:

| Düşman Adı            | Kenney Sprite | Sprite No            | Can | Kovalama Hızı | Devriye Hızı | Süzülme (isGhost)  | Rolü                                               |
| :-------------------- | :------------ | :------------------- | :-: | :-----------: | :----------: | :----------------: | :------------------------------------------------- |
| **Beyaz Hayalet**     | `tile_0121`   | `tilemap_packed_121` |  3  |      2.2      |     1.6      |  **AÇIK (True)**   | Havada dalgalanarak süzülen klasik zindan hayaleti |
| **Yeşil Hayalet**     | `tile_0108`   | `tilemap_packed_108` |  2  |      2.4      |     1.8      |  **AÇIK (True)**   | Ağzı açık zehirli yeşil hayalet                    |
| **Zindan Faresi**     | `tile_0123`   | `tilemap_packed_123` |  1  |      3.5      |     2.2      | **KAPALI (False)** | Yerde çok çevik koşan, fırıl fırıl gezen kemirgen  |
| **Tek Gözlü Canavar** | `tile_0109`   | `tilemap_packed_109` |  5  |      1.6      |     1.1      | **KAPALI (False)** | Ağır adımlı, dayanıklı tepegöz / mini boss         |

- **Sorting Layer:** **`Characters`**  
  _(Oyuncu ile aynı derinlik katmanında olmalıdır)._
- **Order in Layer:** `0`

---

## 3. FAZ 5 Adım Adım Uygulama Rehberi

---

### Adım 5.1: Evrensel Can Sistemi (`Health.cs`)

Hem oyuncunun hem de tüm canavarların canını yöneten merkezi sınıf:

1. **Project** panelinde `Assets/Scripts` klasörüne gidin.
2. Sağ tıklayın → **Create > C# Script** (Adı tam olarak: **`Health`**).
3. Scripti açın ve içindekileri silip şu kodu yapıştırın:

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.SceneManagement;

public class Health : MonoBehaviour, IDamageable
{
    [Header("Can Ayarları")]
    [Tooltip("Varlığın sahip olabileceği en yüksek can puanı")]
    [SerializeField] private int maxHealth = 3;
    private int currentHealth;

    [Header("Hasar Tepkisi & I-Frames (Dokunulmazlık)")]
    [Tooltip("Hasar alındığında sprite'ın bürüneceği flaş rengi")]
    [SerializeField] private Color damageFlashColor = Color.red;
    [Tooltip("Hasar aldıktan sonra kaç saniye dokunulmaz kalınacak")]
    [SerializeField] private float invulnerabilityDuration = 0.8f;
    private bool isInvulnerable = false;

    [Header("Ölüm & Ganimet (Yalnızca Düşmanlar İçin)")]
    [Tooltip("Öldüğünde fırlatılacak altın sikke prefabı")]
    [SerializeField] private GameObject lootPrefab;
    [Range(0, 100)] [SerializeField] private int lootDropChance = 60; // %60 ihtimalle altın düşer

    private SpriteRenderer spriteRenderer;
    private Color originalColor;

    private void Awake()
    {
        currentHealth = maxHealth;
        spriteRenderer = GetComponent<SpriteRenderer>();
        if (spriteRenderer != null) originalColor = spriteRenderer.color;
    }

    // IDamageable Sözleşmesi
    public void TakeDamage(int damageAmount)
    {
        // Dokunulmazlık süresindeyse veya zaten ölmüşse hasar alma
        if (isInvulnerable || currentHealth <= 0) return;

        currentHealth -= damageAmount;
        Debug.Log($"{gameObject.name} Hasar Aldı! Kalan Can: {currentHealth}/{maxHealth}");

        // Görsel Hasar Flaşı (Kırmızı yanıp sönme)
        if (spriteRenderer != null)
        {
            StartCoroutine(DamageFeedbackRoutine());
        }

        // Can bitti mi?
        if (currentHealth <= 0)
        {
            Die();
        }
    }

    public void Heal(int healAmount)
    {
        currentHealth = Mathf.Min(currentHealth + healAmount, maxHealth);
        Debug.Log($"{gameObject.name} İyileşti! Güncel Can: {currentHealth}/{maxHealth}");
    }

    private IEnumerator DamageFeedbackRoutine()
    {
        isInvulnerable = true;

        // Anlık kırmızı flaş
        spriteRenderer.color = damageFlashColor;
        yield return new WaitForSeconds(0.12f);
        spriteRenderer.color = originalColor;

        // Dokunulmazlık süresince yarı-saydam yanıp sönme
        float elapsed = 0.12f;
        while (elapsed < invulnerabilityDuration)
        {
            spriteRenderer.color = new Color(originalColor.r, originalColor.g, originalColor.b, 0.4f);
            yield return new WaitForSeconds(0.1f);
            spriteRenderer.color = originalColor;
            yield return new WaitForSeconds(0.1f);
            elapsed += 0.2f;
        }

        spriteRenderer.color = originalColor;
        isInvulnerable = false;
    }

    private void Die()
    {
        Debug.Log($"{gameObject.name} Öldü!");

        // Oyuncu öldüyse bölümü 1 saniye sonra baştan başlat
        if (CompareTag("Player"))
        {
            Debug.LogWarning("OYUN BİTTİ (GAME OVER)! Şövalye düştü.");
            Invoke(nameof(RestartLevel), 1.0f);
            gameObject.SetActive(false);
            return;
        }

        // Düşman öldüyse: Altın fırlat ve sahneden sil
        if (lootPrefab != null && Random.Range(0, 100) < lootDropChance)
        {
            Instantiate(lootPrefab, transform.position, Quaternion.identity);
        }

        Destroy(gameObject);
    }

    private void RestartLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    public int GetCurrentHealth() => currentHealth;
    public int GetMaxHealth() => maxHealth;
}
```

4. **Karaktere (`Player`) Ekleyin:**
   - Hierarchy panelinde **`Player`** nesnesini seçin.
   - `Assets/Scripts/Health` dosyasını Inspector'a sürükleyip bırakın (Add Component → Health).
   - **Max Health:** `3` yapın.
   - **Invulnerability Duration:** `0.8` yapın.

---

### Adım 5.2: Gelişmiş Düşman Devriye & Takip Kodu (`EnemyAI.cs`)

Canavarların kendi etrafında canlı gibi hızlıca gezinmesini, durup etrafı dinlemesini, oyuncuyu görünce koşmasını, **duvarlara veya köşelere takılmadan duvar boyunca kayarak (Wall Sliding) oyuncuya ulaşmasını** ve hayaletlerin havada süzülmesini sağlayan profesyonel yapay zeka kodu:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`EnemyAI`**).
2. Scripti açın ve içindekileri silip şu kodu yapıştırın:

```csharp
using UnityEngine;

public class EnemyAI : MonoBehaviour
{
    public enum EnemyState { Idle, Wandering, Chasing }

    [Header("Durum ve Hız Ayarları")]
    [Tooltip("Oyuncuyu kovalarkenki koşma hızı")]
    [SerializeField] private float chaseSpeed = 2.4f;

    [Tooltip("Kendi etrafında gezinirkenki devriye hızı")]
    [SerializeField] private float wanderSpeed = 1.8f;

    [Tooltip("Oyuncuyu fark etme radar menzili (metre cinsinden)")]
    [SerializeField] private float detectionRadius = 5.5f;

    [Tooltip("Oyuncunun dibine gelince durma mesafesi")]
    [SerializeField] private float stopDistance = 0.4f;

    [Header("Devriye (Yürü - Bekle - Yürü) Ayarları")]
    [Tooltip("Durup bekleme süresi (saniye)")]
    [SerializeField] private float idleDuration = 0.8f;

    [Tooltip("Rastgele yöne yürüme süresi (saniye)")]
    [SerializeField] private float walkDuration = 2.5f;

    [Tooltip("Doğduğu noktadan en fazla kaç metre uzaklaşabilir")]
    [SerializeField] private float wanderRadius = 4.5f;

    [Header("Duvar ve Engel Algılama")]
    [Tooltip("Düşmanın duvara takılmaması için algılayacağı katmanlar (Varsayılan: Default ve Walls)")]
    [SerializeField] private LayerMask obstacleLayer;

    [Tooltip("Düşmanın önündeki duvarı algılama mesafesi")]
    [SerializeField] private float checkDistance = 0.5f;

    [Header("Hayalet Süzülme Animasyonu (Floating)")]
    [Tooltip("Bu yaratık hayalet gibi aşağı-yukarı süzülsün mü?")]
    [SerializeField] private bool isGhost = false;

    [Tooltip("Süzülme dalgalanma hızı")]
    [SerializeField] private float floatFrequency = 4.0f;

    [Tooltip("Yukarı-aşağı süzülme dalga kuvveti")]
    [SerializeField] private float floatAmplitude = 0.8f;

    [Header("Saldırı")]
    [SerializeField] private int contactDamage = 1;

    private Rigidbody2D rb;
    private Collider2D myCollider;
    private SpriteRenderer spriteRenderer;
    private Transform playerTransform;

    private EnemyState currentState = EnemyState.Idle;
    private Vector2 spawnPosition;
    private Vector2 wanderDirection;
    private float stateTimer = 0f;

    private void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        myCollider = GetComponent<Collider2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
        spawnPosition = transform.position;

        GameObject player = GameObject.FindGameObjectWithTag("Player");
        if (player != null) playerTransform = player.transform;

        // Engel katmanı seçilmediyse Default ve Walls katmanlarını baz al
        if (obstacleLayer.value == 0)
        {
            obstacleLayer = LayerMask.GetMask("Default", "Walls");
            if (obstacleLayer.value == 0) obstacleLayer = 1 << 0; // Default layer
        }

        EnterIdleState();
    }

    private void Update()
    {
        if (playerTransform == null) return;

        float distanceToPlayer = Vector2.Distance(transform.position, playerTransform.position);

        // 1. Radar Kontrolü: Oyuncu menzildeyse KOVALA!
        if (distanceToPlayer <= detectionRadius)
        {
            currentState = EnemyState.Chasing;
        }
        else if (currentState == EnemyState.Chasing)
        {
            // Oyuncu menzilden çıktıysa devriyeye dön
            EnterIdleState();
        }

        // 2. Devriye Zamanlayıcı Mantığı (Idle <-> Wandering döngüsü)
        if (currentState != EnemyState.Chasing)
        {
            stateTimer -= Time.deltaTime;
            if (stateTimer <= 0f)
            {
                if (currentState == EnemyState.Idle)
                {
                    EnterWanderState();
                }
                else
                {
                    EnterIdleState();
                }
            }
        }
    }

    private void FixedUpdate()
    {
        Vector2 targetVelocity = Vector2.zero;

        switch (currentState)
        {
            case EnemyState.Chasing:
                if (playerTransform != null)
                {
                    float distance = Vector2.Distance(transform.position, playerTransform.position);
                    if (distance > stopDistance)
                    {
                        Vector2 rawDir = ((Vector2)playerTransform.position - rb.position).normalized;
                        // Akıllı Engel Aşma: Önünde duvar varsa teğet yönü seç
                        Vector2 moveDir = GetSmartMoveDirection(rawDir);

                        targetVelocity = moveDir * chaseSpeed;
                        UpdateFacing(moveDir.x);
                    }
                }
                break;

            case EnemyState.Wandering:
                targetVelocity = wanderDirection * wanderSpeed;
                UpdateFacing(wanderDirection.x);
                break;

            case EnemyState.Idle:
                targetVelocity = Vector2.zero;
                break;
        }

        // Hayalet süzülme dalgası
        if (isGhost)
        {
            float sineOffset = Mathf.Sin(Time.time * floatFrequency) * floatAmplitude;
            targetVelocity.y += sineOffset;
        }

        #if UNITY_6000_0_OR_NEWER
        rb.linearVelocity = targetVelocity;
        #else
        rb.velocity = targetVelocity;
        #endif
    }

    // AKILLI ENGEL DOLANMA (WHISKER SENSOR):
    // Düşmanın kendi collider'ına ASLA çarpmaz; önündeki duvarı görünce sağa/sola teğet açılır.
    private Vector2 GetSmartMoveDirection(Vector2 desiredDir)
    {
        // 1. Düz rotada gerçek bir engel var mı?
        if (!IsBlockedByWall(desiredDir, checkDistance, out RaycastHit2D frontHit))
        {
            return desiredDir; // Önü açıksa dümdüz oyuncuya git
        }

        // 2. Önümüz tıkalı! 50 derece sağa ve 50 derece sola bak
        Vector2 leftWhisker = Rotate(desiredDir, 50f);
        Vector2 rightWhisker = Rotate(desiredDir, -50f);

        bool leftBlocked = IsBlockedByWall(leftWhisker, checkDistance * 0.9f, out _);
        bool rightBlocked = IsBlockedByWall(rightWhisker, checkDistance * 0.9f, out _);

        if (!leftBlocked && rightBlocked)
        {
            return leftWhisker; // Sol açık, oradan ak
        }
        else if (!rightBlocked && leftBlocked)
        {
            return rightWhisker; // Sağ açık, oradan ak
        }
        else if (!leftBlocked && !rightBlocked)
        {
            // İkisi de açıksa duvardan teğet kay (Wall Slide)
            Vector2 normal = frontHit.normal;
            Vector2 slide = desiredDir - Vector2.Dot(desiredDir, normal) * normal;
            return slide.sqrMagnitude > 0.05f ? slide.normalized : leftWhisker;
        }
        else
        {
            // İki bıyık da tıkalıysa (iç köşe): Duvarın normaline dik 90 derece kaç
            Vector2 perp = new Vector2(-frontHit.normal.y, frontHit.normal.x);
            return perp;
        }
    }

    // Kendi vücudunu, oyuncuyu veya diğer düşmanları DUVAR sanmasını %100 engelleyen güvenli kontrol:
    private bool IsBlockedByWall(Vector2 dir, float distance, out RaycastHit2D validHit)
    {
        validHit = default;
        // Kendi collider'ımızın yarıçapından biraz dışarıdan raycast at
        Vector2 startPos = rb.position + dir * 0.2f;
        RaycastHit2D[] hits = Physics2D.RaycastAll(startPos, dir, distance, obstacleLayer);

        foreach (var h in hits)
        {
            if (h.collider == null) continue;
            if (h.collider == myCollider || h.collider.gameObject == gameObject) continue; // Kendini yoksay!
            if (h.collider.isTrigger) continue; // Trigger tetikleyicileri yoksay!
            if (h.collider.CompareTag("Player") || h.collider.CompareTag("Enemy")) continue; // Karakterleri yoksay!

            validHit = h;
            return true; // Gerçek bir duvar bulduk!
        }

        return false;
    }

    private Vector2 Rotate(Vector2 v, float degrees)
    {
        float rad = degrees * Mathf.Deg2Rad;
        float cos = Mathf.Cos(rad);
        float sin = Mathf.Sin(rad);
        return new Vector2(v.x * cos - v.y * sin, v.x * sin + v.y * cos).normalized;
    }

    private void EnterIdleState()
    {
        currentState = EnemyState.Idle;
        stateTimer = Random.Range(idleDuration * 0.7f, idleDuration * 1.3f);
    }

    private void EnterWanderState()
    {
        currentState = EnemyState.Wandering;
        stateTimer = Random.Range(walkDuration * 0.7f, walkDuration * 1.3f);

        // Doğduğu yerden çok uzaklaşmışsa doğduğu noktaya geri yönel
        if (Vector2.Distance(transform.position, spawnPosition) > wanderRadius)
        {
            wanderDirection = (spawnPosition - (Vector2)transform.position).normalized;
        }
        else
        {
            float angle = Random.Range(0f, 360f) * Mathf.Deg2Rad;
            wanderDirection = new Vector2(Mathf.Cos(angle), Mathf.Sin(angle)).normalized;
        }
    }

    private void UpdateFacing(float directionX)
    {
        if (directionX < -0.05f) spriteRenderer.flipX = true;   // Sola bak
        else if (directionX > 0.05f) spriteRenderer.flipX = false; // Sağa bak
    }

    // Devriye gezerken duvara çarparsa duvardan sekip yoluna devam etsin
    private void OnCollisionEnter2D(Collision2D collision)
    {
        if (currentState == EnemyState.Wandering && !collision.gameObject.CompareTag("Player") && !collision.gameObject.CompareTag("Enemy"))
        {
            if (collision.contactCount > 0)
            {
                // Bilardo topu gibi duvardan sekip gezinmeye devam et
                wanderDirection = Vector2.Reflect(wanderDirection, collision.contacts[0].normal).normalized;
            }
            else
            {
                EnterWanderState();
            }
        }
    }

    // Temas Hasarı: Oyuncuya dokunursa hasar ver
    private void OnCollisionStay2D(Collision2D collision)
    {
        if (collision.gameObject.CompareTag("Player"))
        {
            IDamageable playerDamage = collision.gameObject.GetComponent<IDamageable>();
            if (playerDamage != null)
            {
                playerDamage.TakeDamage(contactDamage);
            }
        }
    }

    private void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.red;
        Gizmos.DrawWireSphere(transform.position, detectionRadius);

        Gizmos.color = Color.cyan;
        Gizmos.DrawWireSphere(Application.isPlaying ? (Vector3)spawnPosition : transform.position, wanderRadius);
    }
}
```

3. Dosyayı kaydedin ve Unity'ye dönün.

---

### Adım 5.3: Düşman Prefablarını Hazırlama

Şimdi zindana yerleştireceğimiz 4 farklı karakter prefabını hazırlıyoruz:

---

#### 1. Prefab: Beyaz Zindan Hayaleti (`Enemy_WhiteGhost`)

- **Görsel:** `tile_0121` (`tilemap_packed_121`)
- Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adı: **`Enemy_WhiteGhost`**). _(Unity alt menüsünde Square, Circle gibi seçenekler çıkar; `Square` seçmeniz yeterlidir. Inspector'da Sprite kısmına görseli atayınca nesne o şekli alacaktır)._
- Inspector Ayarları:
  - **Sprite:** `tile_0121`
  - **Sorting Layer:** **`Characters`**, **Order in Layer:** `0`
  - **Tag:** **`Enemy`**
  - **Add Component → Rigidbody 2D:**
    - Body Type: `Dynamic`, Gravity Scale: `0`, Collision Detection: `Continuous`, Interpolate: `Interpolate`, **Freeze Rotation Z: İşaretli**.
  - **Add Component → Box Collider 2D:** Size: `X: 0.7, Y: 0.8` _(Is Trigger KAPALI)._
  - **Add Component → Health:** Max Health: `3`, Loot Prefab: `Coin_Gold`, Loot Drop Chance: `60`.
  - **Add Component → Enemy AI:**
    - Chase Speed: `2.2`, Wander Speed: `1.6`, Detection Radius: `5.5`
    - 👻 **Is Ghost:** **MUTLAKA İŞARETLEYİN (True)!** _(Havada süzülerek dalgalansın)_
    - Float Frequency: `4.0`, Float Amplitude: `0.8`
- **Prefab Yapın:** Hierarchy'den tutup `Assets/Prefabs/Enemy_WhiteGhost.prefab` içine sürükleyin. Sahnede kalan nesneyi silin.

---

#### 2. Prefab: Yeşil Balçık Hayaleti (`Enemy_GreenGhost`)

- **Görsel:** `tile_0108` (`tilemap_packed_108`)
- Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adı: **`Enemy_GreenGhost`**).
- Inspector Ayarları:
  - **Sprite:** `tile_0108`
  - **Sorting Layer:** **`Characters`**, **Tag:** **`Enemy`**
  - **Add Component → Rigidbody 2D:** Gravity Scale: `0`, Freeze Rotation Z: `İşaretli`, Interpolate: `Interpolate`.
  - **Add Component → Circle Collider 2D:** Radius: `0.35`
  - **Add Component → Health:** Max Health: `2`, Loot Prefab: `Coin_Gold`.
  - **Add Component → Enemy AI:**
    - Chase Speed: `2.4`, Wander Speed: `1.8`
    - 👻 **Is Ghost:** **İŞARETLEYİN (True)!**
- **Prefab Yapın:** `Assets/Prefabs/Enemy_GreenGhost.prefab` yapıp sahnedekini silin.

---

#### 3. Prefab: Hızlı Zindan Faresi (`Enemy_Rat`)

- **Görsel:** `tile_0123` (`tilemap_packed_123` - Kahverengi Fare)
- Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adı: **`Enemy_Rat`**).
- Inspector Ayarları:
  - **Sprite:** `tile_0123`
  - **Sorting Layer:** **`Characters`**, **Tag:** **`Enemy`**
  - **Add Component → Rigidbody 2D:** Gravity Scale: `0`, Freeze Rotation Z: `İşaretli`, Interpolate: `Interpolate`.
  - **Add Component → Box Collider 2D:** Size: `X: 0.6, Y: 0.4`
  - **Add Component → Health:** Max Health: `1` _(Tek vuruşta ölsün!)_, Loot Prefab: `Coin_Gold`.
  - **Add Component → Enemy AI:**
    - Chase Speed: `3.5` _(Çok hızlı koşsun!)_, Wander Speed: `2.2`, Detection Radius: `5.0`
    - ❌ **Is Ghost:** **İŞARETLEMEMİŞ (False)** kalsın _(Yerde koşan canlı)._
- **Prefab Yapın:** `Assets/Prefabs/Enemy_Rat.prefab` yapıp sahnedekini silin.

---

#### 4. Prefab: Tek Gözlü Tepegöz Mini Boss (`Enemy_Cyclops`)

- **Görsel:** `tile_0109` (`tilemap_packed_109` - Tek Gözlü Canavar)
- Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adı: **`Enemy_Cyclops`**).
- Inspector Ayarları:
  - **Sprite:** `tile_0109`
  - **Sorting Layer:** **`Characters`**, **Tag:** **`Enemy`**
  - **Add Component → Rigidbody 2D:** Gravity Scale: `0`, Freeze Rotation Z: `İşaretli`, Interpolate: `Interpolate`.
  - **Add Component → Box Collider 2D:** Size: `X: 0.8, Y: 0.8`
  - **Add Component → Health:** Max Health: `5` _(Dayanıklı mini boss!)_, Loot Prefab: `Coin_Gold`, Loot Drop Chance: `100`.
  - **Add Component → Enemy AI:**
    - Chase Speed: `1.6` _(Ağır adımlarla yaklaşsın)_, Wander Speed: `1.1`, Contact Damage: `2` _(Daha güçlü vursun!)_
    - ❌ **Is Ghost:** **KAPALI (False)**.
- **Prefab Yapın:** `Assets/Prefabs/Enemy_Cyclops.prefab` yapıp sahnedekini silin.

---

### Adım 5.4: Düşmanları Zindana Yerleştirme

[tiny-dungeon-level.md](file:///Users/paban/UnityAgent/tiny-dungeon-level.md) seviye tasarım şemamıza göre odalara canavarları dağıtıyoruz:

1. **2. Oda (Tuzak & Keşif Koridoru):**
   - Koridora 1 adet **`Enemy_Rat`** koyun. Karakter odaya girmeden önce farenin kendi etrafında gezinip durduğunu göreceksiniz.
2. **3. Oda (Büyük Savaş & Anahtar Odası):**
   - Odanın sol sütununun arkasına 1 adet **`Enemy_WhiteGhost`**, sağ köşeye ise 1 adet **`Enemy_GreenGhost`** koyun.
   - Her iki hayalet de havada aşağı yukarı süzülerek devriye gezecektir!
3. **5. Oda (Hazine & Çıkış Odası):**
   - Hazine sandığının hemen önüne büyük muhafız olarak 1 adet **`Enemy_Cyclops`** yerleştirin.

---

### Adım 5.5: Büyük Oynanış Testi!

1. Üstteki **Play (▶️)** butonuna basın.
2. Uzaktan izleyin:
   - Canavarlar duruyor, biraz gezinip bekliyor, sonra tekrar gezinmeye devam ediyor mu?
   - Hayaletler süzülürken **aşağı-yukarı gerçek bir hayalet gibi dalgalanıyor** mu?
3. Canavarın kırmızı radar alanına girin:
   - Canavar gezinmeyi anında kesip üzerinize doğru koşmaya/süzülmeye başlıyor mu?
4. Kılıcınızı savurun:
   - Canavara vurduğunuzda kırmızı yanıp sönüyor ve canı bitince patlayıp altın saçıyor mu?
5. Canavar size çarparsa:
   - Şövalyeniz kırmızı parlayıp 0.8 saniye dokunulmaz kalıyor mu?

---

## 4. Öğrenci Kontrol Listesi (Self-Check Checklist)

- [ ] `Health.cs` scripti hazırlandı ve `Player` nesnesine eklendi (`Max Health: 3`).
- [ ] `EnemyAI.cs` scriptine Idle, Wandering ve Chasing durumları eklendi.
- [ ] Hayaletler için `isGhost: true` yapıldı ve aşağı-yukarı süzülme sinüs dalgası doğrulandı.
- [ ] 4 adet prefab (`Enemy_WhiteGhost`, `Enemy_GreenGhost`, `Enemy_Rat`, `Enemy_Cyclops`) hazırlandı.
- [ ] Canavarlar uzaktayken kendi etraflarında gezinip bekliyor; yaklaşıldığında oyuncuyu kovalıyor.
- [ ] Kılıçla vurulan canavarlar hasar alıp ölünce altın fırlatıyor.

---

## 5. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Hayalet aşağı-yukarı süzülmüyor, dümdüz yürüyor!

> **Çözüm:** `Enemy_WhiteGhost` veya `Enemy_GreenGhost` prefabını açın. Inspector'daki `Enemy AI` bileşeninde bulunan **`Is Ghost`** kutucuğunun mutlaka **işaretli (True)** olduğundan emin olun. `Float Frequency: 4.0` ve `Float Amplitude: 0.8` değerlerini kontrol edin.

### ❓ Soru 2: Canavarlar kendi etrafında gezinirken odanın dışına, haritanın sonuna kadar kaçıyor!

> **Çözüm:** `EnemyAI.cs` içindeki `wanderRadius` (gezinme yarıçapı) değerini kontrol edin (Örn: `2.5` metre). Canavarlar bu mesafe kadar doğdukları noktadan uzaklaştıklarında otomatik olarak doğdukları merkeze geri dönerler.

### ❓ Soru 3: Canavar beni gördüğü halde kovalamıyor, devriye gezmeye devam ediyor!

> **Çözüm:** `Player` nesnenizin en üstündeki **Tag** kutusunun **`Player`** olduğunu doğrulayın. Ayrıca `EnemyAI` bileşenindeki `Detection Radius` değerinin en az `5` olduğundan emin olun.

### ❓ Soru 4: Canavar beni kovalarken köşeye veya duvara takılıyor, ben kıpırdamayınca orada kilitli kalıyor!

> **Çözüm:** `EnemyAI.cs` kodumuza eklediğimiz **Akıllı Duvar Kayması (Wall Sliding & Unstuck)** algoritması sayesinde düşman artık duvara kafa atmak yerine duvar boyunca kayarak oyuncuya ulaşır. Bunun tam çalışması için:
>
> 1. Düşman prefabının Inspector'ında **Enemy AI > Obstacle Layer** kısmında zindan duvarlarınızın katmanının (örneğin **`Default`** veya **`Walls`**) seçili olduğundan emin olun.
> 2. **Sıfır Sürtünme İpucu (Friction 0):** Project panelinde `Create > 2D > Physics Material 2D` oluşturup adını `NoFriction` yapın. **Friction** değerini `0` yapıp hem düşmanların hem de Tilemap duvarlarının Collider 2D bileşenindeki **Material** yuvasına sürükleyin. Sürtünme sıfır olduğunda canavarlar köşelerden tereyağından kıl çeker gibi kayarak üzerinize akar!

### ❓ Soru 5: Canavarlar devriye gezerken neden artık daha canlı?

> **Çözüm:** Yeni sistemde `wanderSpeed: 1.8`, `idleDuration: 0.8s`, `walkDuration: 2.5s` ve `wanderRadius: 4.5m` yapıldı. Ayrıca gezinirken duvara toslarlarsa `OnCollisionEnter2D` sayesinde beklemeden saniyesinde yeni bir açıya dönerek odayı zikzaklarla turlarlar.

---

🎉 **Tebrikler!** FAZ 5 artık zindanınızı capcanlı, yaşayan, nefes alan ve havada süzülen gerçek canavarlarla donattı!

Artık bu fazı tamamlayıp doğrudan **[FAZ 6 Kılavuzuna](file:///Users/paban/UnityAgent/fazlar/faz6.md)** geçebilirsiniz!
