# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 4

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Nesneler Arası İletişim (Interface / Sözleşme Mimarisi), Kılıç Vuruş Sistemi (`DamageDealer`), Kırılabilir Ahşap Kasalar (`BreakableCrate`), Talaş Parçacık Efekti (`WoodBreakParticles`), Fırlayan Ganimet Fiziği (`LootBounce`), Envanter & Altın Toplama (`PlayerInventory` & `CoinPickup`) ve Aşama Aşama Açılan Hazine Sandığı & Mimic Sürprizi (`Chest`)

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 3'te şövalyemizi dinamik bir kamerayla takip ettik ve farenin baktığı yöne saliseler içinde kılıç savurabilen anlık kesme mekaniğimizi kurduk. Ancak şu an kılıcımız zindandaki hiçbir nesneyle etkileşime girmiyor ve odalarımız bomboş duruyor.

Bu kılavuzu tamamladığınızda:

1. Profesyonel yazılım mimarisi olan **Arayüz (Interface - `IDamageable`)** mantığını kavrayacak; kılıcın kutuya mı, sandığa mı yoksa bir canavara mı vurduğunu tek satır `if` yazmadan evrensel bir sözleşmeyle yöneteceksiniz.
2. Kılıca vuruş yeteneği (`DamageDealer.cs`) kazandırarak temas ettiği nesnelere hasar vermesini sağlayacaksınız.
3. Kılıçla vurulduğunda etrafa kahverengi tahta kıymıkları saçarak patlayan **Kırılabilir Ahşap Kasa (`BreakableCrate`)** ve Unity **Particle System** ile gerçekçi parçalanma efekti (`WoodBreakParticles`) üreteceksiniz.
4. Nesneler kırıldığında veya sandıklar açıldığında ortaya çıkan altınların pat diye yere düşmesi yerine, gerçekçi bir **parabolik sinüs eğrisiyle zıplayarak saçımasını (`LootBounce.cs`)** kodlayacaksınız.
5. Oyuncunun topladığı altınları ve ileriki odaların kilitlerini açacak anahtarları hafızasında tutan merkezi bir **Oyuncu Envanteri (`PlayerInventory.cs`)** ve toplanabilir altın sikke prefebı (`CoinPickup.cs`) inşa edeceksiniz.
6. Sağ tıklandığında kare kare kapağı aralanan ve içinden ödül altınlar saçan; ancak %20 şansla keskin dişlerini çıkarıp oyuncuyu ısıran **Açılan Hazine Sandığı & Mimic Canavarı (`Chest.cs`)** mekaniğini kuracaksınız.

---

## 1. Zihinsel Model: Neden Interface (`IDamageable`) Kullanıyoruz?

Yeni başlayan geliştiricilerin kılıçla bir nesneye vururken yaptığı en büyük hata spagetti kod yazmaktır:

```csharp
// YANLIŞ VE SÜRDÜRÜLEMEZ YÖNTEM (Spagetti Kod):
void OnTriggerEnter2D(Collider2D other)
{
    if (other.CompareTag("BreakableCrate")) {
        other.GetComponent<BreakableCrate>().CrateHit();
    }
    else if (other.CompareTag("Chest")) {
        other.GetComponent<Chest>().Open();
    }
    else if (other.CompareTag("SkeletonEnemy")) {
        other.GetComponent<SkeletonEnemy>().TakeHit();
    }
    else if (other.CompareTag("BatEnemy")) {
        other.GetComponent<BatEnemy>().Hurt();
    }
    // Yarın oyuna 10 yeni nesne gelirse buraya 10 tane daha 'else if' mi ekleyeceğiz?!
}
```

```text
PROFESYONEL VE TEMİZ YÖNTEM (INTERFACE - SÖZLEŞME):

           [ Kılıç (DamageDealer) ]
                      │
           "Vuruldun! Al sana 1 hasar!"
                      │
                      ▼
             << IDamageable >>  (Ortak Sözleşme: TakeDamage)
             ┌────────┼────────┐
             │        │        │
             ▼        ▼        ▼
       [ Kasa ]   [ Sandık ] [ Düşman ]
   (Talaş saçar)  (Açılır)   (Canı azalır)
```

### 💡 Interface (Arayüz) Nedir?

Bir **Sözleşmedir (Contract)**. `IDamageable` sözleşmesini imzalayan her C# sınıfı şunu taahhüt eder:  
_`"Benim türüm ne olursa olsun, birisi bana TakeDamage(hasarMiktari) dediğinde ben kendi görevimi yapmayı bilirim!"`_

Böylece kılıç kiminle temas ettiğini zerre kadar umursamaz:

```csharp
// TERTEMİZ ENDÜSTRİ STANDARDI:
IDamageable target = other.GetComponent<IDamageable>();
if (target != null)
{
    target.TakeDamage(damage); // Tek satır! Kasa ise parçalanır, sandıksa açılır, canavarsa ölür.
}
```

---

## 2. Kullanılacak Görseller ve Katman Ayarları

Kenney Tiny Dungeon paketindeki şu karoları kullanacağız:

| Nesne Adı                | Kenney Sprite Adı | Sprite Numarası     | Görevi                                      |
| :----------------------- | :---------------- | :------------------ | :------------------------------------------ |
| **Ahşap Kasa (Kutu)**    | `tile_0075`       | `tilemap_packed_75` | Kılıçla kırılabilen ahşap zindan kasası     |
| **Antik Çömlek (Testi)** | `tile_0082`       | `tilemap_packed_82` | Kılıçla kırılabilen seramik zindan testisi  |
| **Kapalı Sandık**        | `tile_0089`       | `tilemap_packed_89` | Başlangıçtaki kapalı ahşap sandık           |
| **Aralanmış Sandık**     | `tile_0090`       | `tilemap_packed_90` | Açılma anındaki 1. ara animasyon karesi     |
| **Tam Açık Sandık**      | `tile_0091`       | `tilemap_packed_91` | İçinden altınlar saçılan açık sandık        |
| **Mimic Canavarı**       | `tile_0092`       | `tilemap_packed_92` | Dişlerini ve dilini çıkaran canavar sandık! |

> 🪙 **Altın Sikke Notu:** Kenney Tiny Dungeon paketinde doğrudan bir altın sikke görseli bulunmaz. Bu yüzden altın sikkeyi harici bir 16x16 piksel sanatı görseli olan **`coin.png`** olarak projemize dahil ediyoruz.

- **Tüm Bu Nesnelerin Sorting Layer'ı:** **`Interactables`**  
  _(Faz 1'de açtığımız katman. Zeminlerin üstünde, karakterle aynı derinlikte olmalıdır)._

---

## 3. FAZ 4 Adım Adım Uygulama Rehberi

---

### Adım 4.1: Hasar Alabilme Sözleşmesi (`IDamageable.cs`)

Projemizdeki tüm vurulabilir varlıkların temelini atıyoruz:

1. **Project** panelinde `Assets/Scripts` klasörüne gidin.
2. Boş bir yere sağ tıklayın → **Create > C# Script** deyin.
3. Adını tam olarak **`IDamageable`** yapın _(Büyük-küçük harfe dikkat edin!)_.
4. Scripti çift tıklayarak açın ve içindeki her şeyi silip şu 4 satırlık tertemiz arayüzü yazın:

```csharp
public interface IDamageable
{
    void TakeDamage(int damageAmount);
}
```

5. Dosyayı kaydedin (`Ctrl + S`) ve Unity'ye dönün.

---

### Adım 4.2: Kılıca Vurma Yeteneği Kazandırma (`DamageDealer.cs`)

Faz 3'te yaptığımız kılıcın temas kutusu açıldığında çarptığı nesneleri algılamasını sağlıyoruz:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`DamageDealer`**).
2. Scriptin içine şu kodu yapıştırın:

```csharp
using UnityEngine;

public class DamageDealer : MonoBehaviour
{
    [Tooltip("Kılıcın tek vuruşta vereceği hasar puanı")]
    [SerializeField] private int damage = 1;

    private void OnTriggerEnter2D(Collider2D other)
    {
        // 1. Kılıç oyuncunun kendisine asla hasar vermesin!
        if (other.CompareTag("Player")) return;

        // 2. Çarptığımız nesnede IDamageable sözleşmesi var mı?
        IDamageable target = other.GetComponent<IDamageable>();
        if (target != null)
        {
            target.TakeDamage(damage);
        }
    }
}
```

3. **Kılıca Ekleme:**
   - Hierarchy panelinde `Player > WeaponPivot > Sword` nesnesini seçin.
   - `Assets/Scripts/DamageDealer` dosyasını tutup `Sword` nesnesinin Inspector paneline sürükleyip bırakın (veya Add Component → Damage Dealer).
   - **Damage:** `1` olarak kalsın.

---

### Adım 4.3: Ahşap Talaş Parçacık Sistemi (`WoodBreakParticles`)

Ahşap kutu parçalandığında etrafa gerçekçi tahta parçaları saçılması için Unity'nin dahili **Particle System** motorunu kullanıyoruz:

1. Hierarchy panelinde boş bir alana sağ tıklayın → **Effects > Particle System** seçin.
2. Adını `F2`ye basıp **`WoodBreakParticles`** yapın.
3. Nesne seçiliyken sağdaki **Inspector** panelinde şu ayarları harfiyen uygulayın:
   - **Ana Modül (En üstteki başlık):**
     - **Duration:** `0.3` _(Efekt saliseler içinde patlasın)_
     - **Looping:** **Kutucuğun işaretini kaldırın (False)!** _(Sürekli tekrar etmesin, tek seferlik patlasın)_
     - **Start Lifetime:** Küçük oka tıklayıp **Random Between Two Constants** seçin: `0.2` ile `0.35`
     - **Start Speed:** Random Between Two Constants seçin: `2` ile `4`
     - **Start Size:** Random Between Two Constants seçin: `0.1` ile `0.2` _(Küçük talaş parçaları)_
     - **Start Color:** Kahverengi ahşap rengi seçin (Renk kutusuna tıklayın, RGB: `145, 90, 45`)
     - **Gravity Modifier:** `1` _(Talaşlar patladıktan sonra yer çekimiyle yere dökülsün)_
     - **Simulation Space:** `World`
     - **Stop Action:** Mutlaka **`Destroy`** seçin!  
       _(HAYATİ AYAR: Patlama bittiğinde bu parçacık nesnesi hafızadan kendini otomatik siler. Oyunun çöplerle dolup kasmasını önler)._

   - **Emission Modülü:**
     - **Rate over Time:** `0` _(Sürekli akış olmasın)_
     - Altındaki **Bursts** tablosunda `+` butonuna basın: **Count:** `10` _(Aynı anda 10 adet talaş fırlasın)_.

   - **Shape Modülü:**
     - **Shape:** `Circle`
     - **Radius:** `0.2` _(Kutunun içinden dairesel olarak saçılsın)_

   - **Renderer Modülü (En altta):**
     - **Sorting Layer:** **`Interactables`**
     - **Order in Layer:** `2`

4. **Prefab Yapma:** Hierarchy'deki `WoodBreakParticles` nesnesini tutun ve `Assets/Prefabs` klasörünün içine sürükleyip bırakın.
5. Prefab oluştuktan sonra Hierarchy'de sahnede kalan `WoodBreakParticles` nesnesini silin (`Delete`).

---

### Adım 4.4: Fırlayan Eşyalar İçin Zıplama Fiziği (`LootBounce.cs`)

Kutulardan ve sandıklardan saçılan altınların doğrudan yerinde belirmek yerine, hafifçe havaya fırlayıp yay çizerek yere düşmesini sağlayan sinüs eğrisi scripti:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`LootBounce`**).
2. Scriptin içine şu kodu yapıştırın:

```csharp
using System.Collections;
using UnityEngine;

public class LootBounce : MonoBehaviour
{
    [Header("Zıplama Parametreleri")]
    [Tooltip("Eşyanın havaya doğru yükseleceği tepe noktası")]
    [SerializeField] private float bounceHeight = 0.6f;

    [Tooltip("Zıplamanın toplam tamamlanma süresi (saniye)")]
    [SerializeField] private float duration = 0.35f;

    private void Start()
    {
        StartCoroutine(BounceRoutine());
    }

    private IEnumerator BounceRoutine()
    {
        Vector3 startPos = transform.position;
        // Rastgele hafif sağa, sola, yukarı veya aşağı saçılma hedefi
        Vector3 targetPos = startPos + new Vector3(Random.Range(-0.4f, 0.4f), Random.Range(-0.2f, 0.2f), 0f);

        float elapsed = 0f;
        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / duration;

            // Parabolik sinüs eğrisi: Başlangıçta 0, ortada 1, sonda tekrar 0 olur (Zıplama yayı)
            float height = Mathf.Sin(t * Mathf.PI) * bounceHeight;
            transform.position = Vector3.Lerp(startPos, targetPos, t) + Vector3.up * height;

            yield return null;
        }

        transform.position = targetPos;
    }
}
```

3. Dosyayı kaydedin ve Unity'ye dönün.

---

### Adım 4.5: Oyuncu Envanteri (`PlayerInventory.cs`)

Oyuncunun topladığı altınları ve anahtarları hafızasında tutan merkezi bileşen:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`PlayerInventory`**).
2. Scriptin içine şu kodu yazın:

```csharp
using UnityEngine;

public class PlayerInventory : MonoBehaviour
{
    [Header("Toplanan Ganimetler")]
    public int coinCount = 0;
    public int keyCount = 0;

    // --- ALTIN FONKSİYONLARI ---
    public void AddCoins(int amount)
    {
        coinCount += amount;
        Debug.Log($"Altın Toplandı! (+{amount}) | Toplam Altın: {coinCount}");
    }

    // --- ANAHTAR FONKSİYONLARI (İleriki Odalar İçin) ---
    public void AddKey()
    {
        keyCount++;
        Debug.Log($"Altın Anahtar Alındı! | Toplam Anahtar: {keyCount}");
    }

    public bool UseKey()
    {
        if (keyCount > 0)
        {
            keyCount--;
            Debug.Log($"Anahtar Kullanıldı! | Kalan Anahtar: {keyCount}");
            return true;
        }

        Debug.Log("Kapıyı açmak için anahtarınız yok!");
        return false;
    }
}
```

3. **Karaktere Ekleme:**
   - Hierarchy'den **`Player`** nesnesini seçin.
   - `Assets/Scripts/PlayerInventory` scriptini Player'ın üzerine sürükleyin (veya Add Component → Player Inventory).

---

### Adım 4.6: Toplanabilir Altın Sikke Prefabı (`CoinPickup.cs`)

Kutulardan ve sandıklardan fırlayan altınların oyuncu tarafından toplanmasını sağlayan yapı:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`CoinPickup`**).
2. Scriptin içine şu kodu yapıştırın:

```csharp
using UnityEngine;

public class CoinPickup : MonoBehaviour
{
    [SerializeField] private int coinValue = 1;
    private bool isCollected = false;

    private void OnTriggerEnter2D(Collider2D other)
    {
        TryCollect(other);
    }

    private void OnTriggerStay2D(Collider2D other)
    {
        // Sandığın dibindeyken altın oyuncunun tam üstüne fırlarsa kaçırmamak için güvenlik kontrolü
        TryCollect(other);
    }

    private void TryCollect(Collider2D other)
    {
        if (isCollected) return;

        // Çarpan nesnede veya ebeveyninde PlayerInventory var mı?
        PlayerInventory inventory = other.GetComponentInParent<PlayerInventory>();
        if (inventory != null)
        {
            isCollected = true;
            inventory.AddCoins(coinValue);
            Destroy(gameObject); // Altını sahneden sil
        }
    }
}
```

3. **Altın Görselini Projeye Ekleme ve Nesneyi Oluşturma:**
   - Projenizin kök dizininde sağlanan **`coin.png`** dosyasını Unity'de `Assets/Art` klasörünün içine sürükleyin.
   - `Assets/Art/coin.png` seçiliyken sağdaki **Inspector** panelinde şu piksel sanatı ayarlarını yapın ve **Apply** butonuna basın:
     - **Pixels Per Unit:** `16`
     - **Filter Mode:** `Point (no filter)`
     - **Compression:** `None`
   - Hierarchy panelinde boş bir yere sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`Coin_Gold`** yapın). _(Unity alt menüsünde Square, Circle gibi seçenekler çıkar; `Square` seçmeniz yeterlidir. Inspector'da Sprite kısmına altın görselimizi bağlayınca nesne altın şeklini alacaktır)._
   - Inspector'da:
     - **Sprite:** Yeni eklediğimiz **`coin`** görselini bağlayın.
     - **Sorting Layer:** **`Interactables`** yapın.
     - **Order in Layer:** `1`
     - **Add Component → Circle Collider 2D:**
       - **Radius:** `0.35`
       - **Is Trigger:** **Mutlaka İŞARETLEYİN (True)!** _(Karakter altına takılıp durmasın, içinden geçebilsin)._
     - **Add Component → Loot Bounce:** Az önce yazdığımız zıplama scriptini ekleyin.
     - **Add Component → Coin Pickup:** Altın toplama scriptini ekleyin (`Coin Value: 1`).
4. **Prefab Yapma:** Hierarchy'deki `Coin_Gold` nesnesini tutup `Assets/Prefabs` içine sürükleyin. Sahnede kalan nesneyi silin.

---

### Adım 4.7: Kırılabilir Ahşap Kasa Prefabı (`BreakableCrate.cs`)

Kılıç darbesiyle kırılan, talaş patlatan ve şansa bağlı altın fırlatan zindan kasası:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`BreakableCrate`**).
2. Scriptin içine şu kodu yazın:

```csharp
using UnityEngine;

public class BreakableCrate : MonoBehaviour, IDamageable
{
    [Header("Görsel Parçalanma")]
    [Tooltip("Kutu kırıldığında patlayacak talaş parçacık prefabı")]
    [SerializeField] private GameObject breakEffectPrefab;

    [Header("Şanslı Ganimet Tablosu (Toplam %100)")]
    [Range(0, 100)] [SerializeField] private int coinDropChance = 60; // %60 ihtimalle Altın çıkar
    // Kalan %40 ihtimalde kutu boş çıkar

    [Header("Düşecek Prefab")]
    [SerializeField] private GameObject coinPrefab;

    public void TakeDamage(int damageAmount)
    {
        // 1. Talaş patlama efektini kutunun konumunda oluştur
        if (breakEffectPrefab != null)
        {
            Instantiate(breakEffectPrefab, transform.position, Quaternion.identity);
        }

        // 2. Şans zarını at (0 ile 100 arası)
        int roll = Random.Range(0, 100);

        if (roll < coinDropChance)
        {
            if (coinPrefab != null)
            {
                Instantiate(coinPrefab, transform.position, Quaternion.identity);
            }
        }

        // 3. Kutuyu sahneden tamamen yok et
        Destroy(gameObject);
    }
}
```

3. **Kırılabilir Nesneleri Sahnede Hazırlama (2 Farklı Çeşit: Kasa ve Testi):**

   Zindana zenginlik ve çeşitlilik katmak için aynı scripti kullanarak iki farklı kırılabilir obje üretiyoruz:
   - **Çeşit A: Ahşap Kasa (`BreakableCrate` - `tile_0075`):**
     1. Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`BreakableCrate`** yapın). _(Alt menüden Square seçin)._
     2. **Sprite:** Ahşap kasa görselini (**`tile_0075`** / `tilemap_packed_75`) seçin.
     3. **Sorting Layer:** **`Interactables`**
     4. **Add Component → Box Collider 2D:** _(Is Trigger işaretlemeyin, katı kalsın)._
     5. **Add Component → Breakable Crate:**
        - **Break Effect Prefab:** `Assets/Prefabs/WoodBreakParticles` prefabını bağlayın.
        - **Coin Drop Chance:** `60`
        - **Coin Prefab:** `Assets/Prefabs/Coin_Gold` prefabını bağlayın.
     6. Bu nesneyi Hierarchy'den tutup `Assets/Prefabs/BreakableCrate.prefab` olarak kaydedin ve sahnedekini silin.

   - **Çeşit B: Antik Çömlek / Testi (`BreakablePot` - `tile_0082`):**
     1. Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`BreakablePot`** yapın). _(Alt menüden Square seçin)._
     2. **Sprite:** Seramik testi görselini (**`tile_0082`** / `tilemap_packed_82`) seçin.
     3. **Sorting Layer:** **`Interactables`**
     4. **Add Component → Circle Collider 2D:** Radius: `0.4` _(Testinin yuvarlak hatlarına tam oturur, Is Trigger işaretlemeyin)._
     5. **Add Component → Breakable Crate:** _(Aynı scripti buraya da ekliyoruz!)_
        - **Break Effect Prefab:** `Assets/Prefabs/WoodBreakParticles` (veya ileride seramik parçacığı)
        - **Coin Drop Chance:** `50`
        - **Coin Prefab:** `Assets/Prefabs/Coin_Gold`
     6. Bu nesneyi de `Assets/Prefabs/BreakablePot.prefab` olarak kaydedin ve sahnedekini silin.

   > 💡 **Tasarım Gücü:** `IDamageable` arayüzü sayesinde, kılıcımız hem kasayı (`tile_0075`) hem de antik testiyi (`tile_0082`) tek bir ek kod yazmadan aynı kolaylıkla parçalayabilir!

---

### Adım 4.8: Açılan Hazine Sandığı & Mimic Canavarı (`Chest.cs`)

Oyun dünyasının en heyecan verici klasiklerinden biri: Yakınına gelip **Sağ Tık** yaptığınızda veya kılıçla vurduğunuzda kapağı kare kare açılan; ancak %20 şansla keskin dişlerini gösteren bir **Mimic**'e dönüşen sandık!

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`Chest`**).
2. Scriptin içine şu kodu yapıştırın:

```csharp
using System.Collections;
using UnityEngine;
#if ENABLE_INPUT_SYSTEM
using UnityEngine.InputSystem;
#endif

public class Chest : MonoBehaviour, IDamageable
{
    [Header("Açılma Animasyon Kareleri (Kenney Tiles)")]
    [SerializeField] private Sprite closedSprite;      // tile_0089: Kapalı Sandık
    [SerializeField] private Sprite openingSprite;     // tile_0090: Kapağı Aralanmış Sandık
    [SerializeField] private Sprite openedSprite;      // tile_0091: Tam Açık Sandık
    [SerializeField] private float frameDuration = 0.12f;

    [Header("Mimic (Canavar Sandık) Şansı")]
    [SerializeField] private bool canBeMimic = true;
    [Range(0, 100)] [SerializeField] private int mimicChance = 20; // %20 şansla Mimic
    [SerializeField] private Sprite mimicSprite;       // tile_0092: Canavar Sandık
    [SerializeField] private int mimicDamage = 1;

    [Header("Ödül Ayarları")]
    [SerializeField] private GameObject rewardPrefab;  // Coin_Gold prefabı
    [SerializeField] private int coinRewardCount = 3;  // Sandıktan saçılacak altın sayısı

    [Header("Etkileşim Ayarları")]
    [SerializeField] private float interactionDistance = 1.3f; // Sağ tık menzili

    private bool isOpened = false;
    private bool isMimicTriggered = false;
    private SpriteRenderer spriteRenderer;
    private Transform playerTransform;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        if (closedSprite != null) spriteRenderer.sprite = closedSprite;

        GameObject player = GameObject.FindGameObjectWithTag("Player");
        if (player != null) playerTransform = player.transform;
    }

    private void Update()
    {
        if (isOpened || isMimicTriggered) return;

        // Oyuncu sandığa yeterince yakın mı?
        if (playerTransform != null && Vector2.Distance(transform.position, playerTransform.position) <= interactionDistance)
        {
            bool rightClicked = false;

            #if ENABLE_INPUT_SYSTEM
            if (Mouse.current != null && Mouse.current.rightButton.wasPressedThisFrame)
                rightClicked = true;
            #endif

            #if ENABLE_LEGACY_INPUT_MANAGER
            if (Input.GetMouseButtonDown(1))
                rightClicked = true;
            #endif

            if (rightClicked)
            {
                Interact();
            }
        }
    }

    // Kılıçla vurulduğunda da açılsın
    public void TakeDamage(int damageAmount)
    {
        if (!isOpened && !isMimicTriggered)
        {
            Interact();
        }
    }

    private void Interact()
    {
        // 1. Şans Zarı: Mimic mi, Hazine Sandığı mı?
        if (canBeMimic && Random.Range(0, 100) < mimicChance)
        {
            TriggerMimic();
        }
        else
        {
            StartCoroutine(OpenAnimationRoutine());
        }
    }

    // --- NORMAL SANDIK: AŞAMA AŞAMA AÇILMA ANİMASYONU ---
    private IEnumerator OpenAnimationRoutine()
    {
        isOpened = true;

        // 1. Aşama: Kapak aralanır (tile_0090)
        if (openingSprite != null) spriteRenderer.sprite = openingSprite;
        yield return new WaitForSeconds(frameDuration);

        // 2. Aşama: Kapak sonuna kadar açılır (tile_0091)
        if (openedSprite != null) spriteRenderer.sprite = openedSprite;
        yield return new WaitForSeconds(frameDuration * 0.5f);

        // 3. Aşama: Altınları fırlat!
        SpawnRewards();
    }

    private void SpawnRewards()
    {
        if (rewardPrefab == null) return;

        Vector3 spawnPos = transform.position + Vector3.up * 0.2f;

        for (int i = 0; i < coinRewardCount; i++)
        {
            Instantiate(rewardPrefab, spawnPos, Quaternion.identity);
        }
    }

    // --- SÜRPRİZ: MIMIC CANAVAR SANDIK ---
    private void TriggerMimic()
    {
        isMimicTriggered = true;

        // Canavar görseline geç (tile_0092)
        if (mimicSprite != null) spriteRenderer.sprite = mimicSprite;

        Debug.LogWarning("DİKKAT! Bu bir Mimic (Canavar Sandık)! Seni ısırdı!");
    }
}
```

3. **Sandık Nesnesini Sahnede Hazırlama:**
   - Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`Chest_Golden`** yapın). _(Alt menüden Square seçin)._
   - Inspector ayarları:
     - **Sprite:** `tile_0089` (Kapalı sandık).
     - **Sorting Layer:** **`Interactables`**
     - **Add Component → Box Collider 2D:** _(Is Trigger İŞARETLEMEYİN, katı kalsın)._
     - **Add Component → Chest:**
       - **Closed Sprite:** `tile_0089` (Kapalı ahşap sandık)
       - **Opening Sprite:** `tile_0090` (Kapağı hafif aralanmış sandık)
       - **Opened Sprite:** `tile_0091` (Tam açık sandık)
       - **Mimic Sprite:** `tile_0092` (Dişli ve dilli canavar sandık)
       - **Reward Prefab:** `Assets/Prefabs/Coin_Gold.prefab`
       - **Coin Reward Count:** `3`
4. **Prefab Yapma:** Hierarchy'deki `Chest_Golden` nesnesini `Assets/Prefabs` içine sürükleyip prefab yapın.

---

### Adım 4.9: Zindanı Canlandırma (Seviyeye Kasa, Testi ve Sandık Dizme)

Artık zindanın odalarını canlı ve ödüllendirici hale getirelim:

1. `Assets/Prefabs/BreakableCrate` ve `Assets/Prefabs/BreakablePot` prefablarını tutun; zindanın odalarındaki koridor köşelerine ve duvar kenarlarına 4-5 adet yerleştirin.
2. `Assets/Prefabs/Chest_Golden` prefabını tutun ve seviye tasarım şemamızdaki (Bkz: `tiny-dungeon-level.md`) 3. ve 5. odalara birer adet sandık yerleştirin.
3. **Play (▶️)** butonuna basın ve büyük testi yapın!

---

## 4. Öğrenci Kontrol Listesi (Self-Check Checklist)

Faz 4'ü tamamladığınızı doğrulamak için şu adımları kontrol edin:

- [ ] `Assets/Scripts/IDamageable.cs` arayüzü hatasız oluşturuldu.
- [ ] `Assets/Scripts/DamageDealer.cs` yazıldı ve `Player > WeaponPivot > Sword` nesnesine eklendi.
- [ ] `WoodBreakParticles` parçacık sistemi oluşturuldu, `Stop Action: Destroy` yapıldı ve prefab olarak kaydedildi.
- [ ] `LootBounce.cs` yazıldı ve sinüs dalgasıyla zıplama mekaniği test edildi.
- [ ] `PlayerInventory.cs` yazıldı ve Hierarchy'deki `Player` nesnesine eklendi.
- [ ] `Coin_Gold` nesnesi oluşturuldu, `Circle Collider 2D (Is Trigger)` ve `CoinPickup` eklendi, prefab yapıldı.
- [ ] `BreakableCrate` (`tile_0075`) ve `BreakablePot` (`tile_0082`) nesneleri katı collider ve `BreakableCrate.cs` ile prefab yapıldı.
- [ ] `Chest_Golden` nesnesi hazırlandı, 4 adet sprite karesi bağlandı, prefab yapıldı.
- [ ] Sahneye kasalar, testiler ve 1 sandık kondu. Oyunu başlatıp sol tıkla kılıç vurulduğunda kutu/testi parçalanıp altın saçıyor mu?
- [ ] Sandığa yaklaşılıp sağ tıklandığında kapağı açılıp 3 adet altın zıplayarak yere düşüyor mu?
- [ ] Karakter altınların üzerinden geçtiğinde Unity konsolunda `Altın Toplandı!` mesajı yazıyor mu?

---

## 5. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Kılıcı kasaya savuruyorum ama kasa hiç tepki vermiyor, parçalanmıyor!

> **Çözüm:** 3 şeyi kontrol edin:
>
> 1. `Sword` nesnesinin üzerindeki `Box Collider 2D`nin **Is Trigger** kutucuğu işaretli mi? (İşaretli olmalıdır).
> 2. `Sword` nesnesinde `DamageDealer` scripti takılı mı?
> 3. `BreakableCrate` nesnesinde `Box Collider 2D` bileşeni var mı? (Collider olmazsa temas tetiklenmez).

### ❓ Soru 2: Altınlar yere saçılıyor ama karakter üstünden geçince toplanmıyor!

> **Çözüm:**
>
> 1. `Coin_Gold` prefabındaki `Circle Collider 2D` bileşeninin **Is Trigger** kutucuğunun işaretli olduğundan emin olun.
> 2. `Player` nesnenizin üzerinde **`PlayerInventory`** scriptinin ekli olduğunu doğrulayın (`CoinPickup` scripti bu bileşeni arar).
> 3. `CoinPickup.cs` içinde hem `OnTriggerEnter2D` hem `OnTriggerStay2D` kodunun bulunduğundan emin olun.

### ❓ Soru 3: Kutu kırılınca oyun bir süre sonra kasmaya başlıyor!

> **Çözüm:** `WoodBreakParticles` prefabını açın. En üst ana modüldeki **Stop Action** ayarının **`Destroy`** olduğundan emin olun. Eğer `None` kalırsa, patlama bitse bile görünmez yüzlerce parçacık sistemi sahnede birikerek performansı düşürür.

### ❓ Soru 4: Sandığa sağ tıklıyorum ama açılmıyor!

> **Çözüm:**
>
> 1. Sandığa yeterince yakın mısınız? `interactionDistance` varsayılan olarak `1.3` birimdir; sandığın dibine kadar yürüyün.
> 2. Hierarchy'deki `Player` nesnenizin Inspector en üstündeki **Tag** ayarının **`Player`** olduğunu doğrulayın (`Chest.cs` oyuncuyu tag'i ile bulur).

### ❓ Soru 5: Sandıktan çıkan altınlar duvara giriyor veya çok uzağa fırlıyor!

> **Çözüm:** `LootBounce.cs` scriptindeki `bounceHeight` (0.6) ve hedef rastgelelik değerlerini (`Random.Range(-0.4f, 0.4f)`) azaltarak eşyaların daha dar bir alana düşmesini sağlayabilirsiniz.

---

🎉 **Tebrikler!** FAZ 4'ü başarıyla tamamladınız. Artık zindanınızda parçalanabilen fiziksel objeler, görsel patlama efektleri, toplanabilir ganimetler, envanter altyapısı ve sürprizlerle dolu hazine sandıkları var!

Bir sonraki fazda (**FAZ 5**) zindanın karanlık odalarında devriye gezen, oyuncuyu görünce peşine takılan iskelet ve yarasa düşman yapay zekasını (`EnemyAI.cs`) ve hem oyuncuyu hem düşmanları kapsayan evrensel can/hasar sistemini (`Health.cs`) inşa edeceğiz!
