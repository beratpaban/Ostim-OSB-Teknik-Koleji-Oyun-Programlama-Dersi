# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 7

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Oyuncu Arayüzü (HUD - Heads-Up Display), 3 Kalp Can Sistemi (`HeartUI.cs`), Toplanabilir Sağlık İksiri (`HealthPotion.cs`), Kutu & Çömlek İksir Düşürme Entegrasyonu, Altın/Anahtar Sayaçları ve Oyun Sonu (Game Over & Yeniden Başlatma)

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 6'da zindanımıza kilitli demir kapıyı, parıldayan altın anahtarı ve ölümcül zemin dikenlerini ekleyerek klasik Zelda zindan ilerleme döngüsünü kurduk. Ancak şu an oyuncu aldığı hasarı yalnızca konsoldaki `Debug.Log` yazılarından veya ekrandaki kırmızı flaş efektinden anlayabiliyor; kaç canı kaldığını ekranda net bir şekilde göremiyor ve canı azaldığında iyileşebileceği bir sağlık iksiri bulunmuyor.

Bu kılavuzu tamamladığınızda:

1. **Unity UI (uGUI) & Canvas Mimarisi:** Oyun dünyasından bağımsız olarak ekranın sol üst köşesine sabitlenen, farklı ekran çözünürlüklerinde ve pencerelerde bozulmayan profesyonel bir **HUD (Heads-Up Display)** arayüzü inşa edeceksiniz.
2. **Retro 3 Kalp Can Göstergesi (`HeartUI.cs`):** Oyuncunun canını ekranda 3 adet piksel kalp simgesiyle gösterecek; hasar aldığında kalpleri boş kalbe (`heart_empty.png`), iyileştiğinde ise tekrar parıldayan dolu kalbe (`heart_full.png`) dönüştüreceksiniz.
3. **Toplanabilir Sağlık İksiri (`HealthPotion.cs`):** Kenney paketindeki orijinal yuvarlak mantarlı kırmızı iksir şişesini (`tile_0115`) toplanabilir bir eşya haline getirecek, yere zıplayarak düşmesini (`LootBounce.cs`) sağlayacak ve **yalnızca canı eksik olan oyuncu tarafından içilebilmesi** kuralını uygulayacaksınız (can doluyken iksir boşa gitmez!).
4. **Sandık ve Çömlek Ganimet Tablosu Güncellemesi:** Faz 4'te yaptığımız ahşap kasalar (`tile_0075`) ve antik testiler (`tile_0082`) parçalandığında altın sikkenin yanı sıra şansa bağlı (%25) kırmızı sağlık iksiri fırlatmasını sağlayacaksınız.
5. **Envanter Bilgi Sayaçları (Altın & Anahtar HUD):** Sol üst köşede kalplerin hemen altında oyuncunun topladığı altın miktarını (`coin.png x 0`) ve elindeki altın anahtar sayısını (`key.png x 0`) anlık gösteren retro bilgi paneli kuracaksınız.
6. **Oyun Sonu & Yeniden Başlatma (Game Over Screen):** Kahramanımızın 3 kalbi de tükendiğinde ekranda beliren şık bir "OYUN BİTTİ" ekranı ve tek tuşla (`[R]` tuşu) zindanı yeniden deneme döngüsü oluşturacaksınız.

---

## 1. Zihinsel Model: HUD Arayüzü, Can Akışı & İksir Döngüsü

Oyun arayüzleri (UI), oyun içi fizik dünyasından (World Space) ayrı bir katmanda (Screen Space - Overlay) çizilir. Kamera nereye giderse gitsin, can kalpleri ve altın sayaçları oyuncunun ekranında hep aynı köşede sabit kalır:

```text
 ┌─────────────────────────────────────────────────────────────┐
 │ 💖 💖 💖  (HeartUI - 3 Kalp Göstergesi)                      │
 │ 🪙 x 12   (Coin Counter - Altın Sayacı)                      │
 │ 🗝️ x 1    (Key Counter - Anahtar Sayacı)                     │
 │                                                             │
 │                                                             │
 │                 [ 🛡️ ŞÖVALYE (Player) ]                    │
 │                            │                                │
 │       Hasar Alınca         │        İksir İçince            │
 │     (currentHealth - 1)    │     (currentHealth + 1)        │
 │            ▼               │                ▲               │
 │     [ 💖 💖 🖤 ]           │        [ 🧪 SAĞLIK İKSİRİ ]   │
 │   (1 Kalp Boşalır)         │       (tile_0115 Şişe Alınır)  │
 │            │                                │               │
 │       Can = 0 İse                           │               │
 │            ▼                                │               │
 │   ┌─────────────────┐                       │               │
 │   │  ☠️ OYUN BİTTİ  │                       │               │
 │   │ [R] Tekrar Dene │                       │               │
 │   └─────────────────┘                       │               │
 └─────────────────────────────────────────────┴───────────────┘
```

---

## 2. Kullanılacak Görseller ve Katman Ayarları

| Varlık Adı           | Görsel Kaynağı           | Sprite Adı / No                        | Görevi / Açıklama                                    |
| :------------------- | :----------------------- | :------------------------------------- | :--------------------------------------------------- |
| **Dolu Kalp**        | Harici Özel Piksel Asset | **`heart_full.png`** (16x16)           | Canı temsil eden parlak kırmızı dolu kalp            |
| **Boş Kalp**         | Harici Özel Piksel Asset | **`heart_empty.png`** (16x16)          | Hasar alındığında görünen koyu/boş kalp              |
| **Sağlık İksiri**    | Kenney Tiny Dungeon      | **`tile_0115`** (`tilemap_packed_115`) | Orijinal yuvarlak mantar tıpalı kırmızı iksir şişesi |
| **Alternatif İksir** | Kenney Tiny Dungeon      | `tile_0127` (`tilemap_packed_127`)     | İsteğe bağlı kırmızı deney tüpü iksiri               |
| **Altın İkonu**      | Harici Özel Piksel Asset | `coin.png` (16x16)                     | HUD üzerindeki altın sayacı simgesi                  |
| **Anahtar İkonu**    | Harici Özel Piksel Asset | `key.png` (16x16)                      | HUD üzerindeki zindan anahtarı simgesi               |

> 💡 **Asset Hatırlatması:** Kenney Tiny Dungeon paketinde can kalbi görseli bulunmaz. Tıpkı Faz 4'teki altın sikke (`coin.png`) ve Faz 6'daki altın anahtar (`key.png`) gibi, projenizin ana dizinine 16x16 retro piksel tarzıyla kusursuz uyumlu **`heart_full.png`** ve **`heart_empty.png`** hazırlanmıştır!

---

## 3. FAZ 7 Adım Adım Uygulama Rehberi

---

### Adım 7.1: Kalp Görsellerini (`heart_full.png`, `heart_empty.png`) Projeye Alma

1. Projenizin ana klasöründe bulunan **`heart_full.png`** ve **`heart_empty.png`** dosyalarını Unity'de **`Assets/Art`** klasörünün içine sürükleyip bırakın.
2. Unity Project panelinde her iki görseli de seçin (veya tek tek tıklayın) ve sağdaki **Inspector** panelinde piksel grafik ayarlarını yapın:
   - **Texture Type:** `Sprite (2D and UI)`
   - **Sprite Mode:** `Single`
   - **Pixels Per Unit:** `16` _(Karakter ve kutularla aynı çözünürlük oranı)_
   - **Filter Mode:** `Point (no filter)` _(Bulanıklaşmayı önler, pikselleri jilet gibi netleştirir)_
   - **Compression:** `None`
3. Sağ alttaki **Apply** butonuna tıklayarak ayarları kaydedin.

---

### Adım 7.2: Canvas ve HUD Hiyerarşisinin Kurulumu

Oyun arayüzünü ekranın sol üst köşesine yerleştireceğimiz `Canvas` nesnesini yapılandırıyoruz:

1. Hierarchy panelinde boş bir yere sağ tıklayın → **UI > Canvas** deyin.
   - Adını **`HUD_Canvas`** yapın.
   - Sahnenize otomatik olarak bir **EventSystem** nesnesi de gelecektir; bu nesneyi kesinlikle silmeyin.
2. `HUD_Canvas` seçiliyken Inspector panelini inceleyin:
   - **Render Mode:** `Screen Space - Overlay` olsun.
   - **Canvas Scaler (Component):**
     - **UI Scale Mode:** `Scale With Screen Size` seçin.
     - **Reference Resolution:** `X: 1920, Y: 1080` yazın.
     - **Screen Match Mode:** `Match Width Or Height` kalsın.
     - **Match:** `0.5` yapın. _(Bu sayede oyun penceresi büyüse veya küçülse bile kalpler ekranda daima orantılı kalır)._
3. **Kalpler İçin Kapsayıcı Panel Oluşturma:**
   - `HUD_Canvas` nesnesine sağ tıklayın → **Create Empty** deyin (Adını **`Hearts_Panel`** yapın).
   - Inspector'da **Rect Transform** bileşenine gelin:
     - Sol üstteki kare Anchor simgesine tıklayın. **`Shift` ve `Alt` tuşlarına basılı tutarak** **Sol Üst (Top-Left)** kutusunu seçin.
     - **Pos X:** `40`
     - **Pos Y:** `-40`
     - **Width:** `200`
     - **Height:** `50`
4. **3 Adet Kalp Resmi Ekleme:**
   - `Hearts_Panel` nesnesine sağ tıklayın → **UI > Image** deyin (Adını **`Heart_1`** yapın).
     - **Source Image:** `heart_full` görselini sürükleyip bırakın.
     - **Width:** `48` | **Height:** `48` _(16x16 pikselin tam 3 katı, ekranda net ve dolgun görünür)._
     - **Pos X:** `0` | **Pos Y:** `0`
     - **Preserve Aspect:** `İşaretli (True)`
   - `Heart_1` nesnesini seçip klavyeden `Ctrl + D` (Mac'te `Cmd + D`) yaparak iki kere çoğaltın:
     - İkincisinin adını **`Heart_2`** yapın → **Pos X:** `56` yapın.
     - Üçüncüsünün adını **`Heart_3`** yapın → **Pos X:** `112` yapın.
   - _Artık Game ekranının sol üst köşesinde yan yana duran 3 tane parlak kırmızı retro kalp göreceksiniz!_

---

### Adım 7.3: Kalp Takip Kodu (`HeartUI.cs`)

Karakterin can puanını (`Health.cs`) sürekli dinleyen ve can düştükçe kalpleri sırayla boşa çeviren, iyileşince dolduran script:

1. **Project** panelinde `Assets/Scripts` klasörüne gidin.
2. Sağ tıklayın → **Create > C# Script** (Adı tam olarak: **`HeartUI`**).
3. Scripti açın ve şu kodu yapıştırın:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class HeartUI : MonoBehaviour
{
    [Header("Oyuncu Can Referansı")]
    [Tooltip("Canı takip edilecek oyuncunun Health bileşeni")]
    [SerializeField] private Health playerHealth;

    [Header("Kalp Görselleri")]
    [Tooltip("Ekranda sırayla duran 3 adet kalp Image nesnesi")]
    [SerializeField] private Image[] heartImages;

    [Header("Kalp Durum Spriteları")]
    [Tooltip("Can varken gösterilecek kırmızı dolu kalp")]
    [SerializeField] private Sprite fullHeartSprite;

    [Tooltip("Hasar alındığında gösterilecek boş/koyu kalp")]
    [SerializeField] private Sprite emptyHeartSprite;

    private void Start()
    {
        // Eğer Inspector'dan elle bağlanmadıysa sahnedeki Player nesnesinden otomatik bul
        if (playerHealth == null)
        {
            GameObject player = GameObject.FindGameObjectWithTag("Player");
            if (player != null)
            {
                playerHealth = player.GetComponent<Health>();
            }
        }

        UpdateHearts();
    }

    private void Update()
    {
        if (playerHealth != null)
        {
            UpdateHearts();
        }
    }

    /// <summary>
    /// Oyuncunun mevcut canına göre kalplerin görselini yeniler.
    /// </summary>
    public void UpdateHearts()
    {
        if (playerHealth == null || heartImages == null) return;

        int current = playerHealth.GetCurrentHealth();

        for (int i = 0; i < heartImages.Length; i++)
        {
            if (heartImages[i] == null) continue;

            // Eğer döngü indeksi mevcut candan küçükse kalp doludur, değilse boştur
            if (i < current)
            {
                heartImages[i].sprite = fullHeartSprite;
                heartImages[i].color = Color.white;
            }
            else
            {
                heartImages[i].sprite = emptyHeartSprite;
                heartImages[i].color = Color.white;
            }
        }
    }
}
```

4. **Scripti Sahneye Bağlama:**
   - Hierarchy'de `Hearts_Panel` nesnesini seçin.
   - `Assets/Scripts/HeartUI` scriptini `Hearts_Panel` üzerine sürükleyip bırakın.
   - Inspector'da alanları doldurun:
     - **Player Health:** Hierarchy'deki **`Player`** nesnesini sürükleyin (veya boş bırakırsanız kod otomatik bulur).
     - **Heart Images (Size: 3):**
       - Element 0: `Heart_1`
       - Element 1: `Heart_2`
       - Element 2: `Heart_3`
     - **Full Heart Sprite:** `heart_full`
     - **Empty Heart Sprite:** `heart_empty`

---

### Adım 7.4: Toplanabilir Sağlık İksiri Prefabı (`HealthPotion.cs`)

Zindanda kutulardan fırlayan veya odaların gizli köşelerinde duran kırmızı sağlık iksiri:

1. `Assets/Scripts` klasörüne sağ tıklayın → **Create > C# Script** (Adı: **`HealthPotion`**).
2. Scripti açın ve şu kodu yazın:

```csharp
using UnityEngine;

public class HealthPotion : MonoBehaviour
{
    [Header("İyileştirme Değeri")]
    [Tooltip("İksir içildiğinde kazanılacak kalp miktarı")]
    [SerializeField] private int healAmount = 1;

    private bool isCollected = false;

    private void OnTriggerEnter2D(Collider2D other)
    {
        TryConsume(other);
    }

    private void OnTriggerStay2D(Collider2D other)
    {
        TryConsume(other);
    }

    private void TryConsume(Collider2D other)
    {
        if (isCollected) return;

        // Oyuncuyu veya ebeveynindeki Health bileşenini bul
        Health health = other.GetComponentInParent<Health>();
        if (health != null && other.CompareTag("Player"))
        {
            // OYUNCU DOSTU KURAL: Can zaten tam ise iksir boşa gitmesin, yerde beklesin!
            if (health.GetCurrentHealth() < health.GetMaxHealth())
            {
                isCollected = true;
                health.Heal(healAmount);
                Debug.Log($"🧪 Şifa Bulundu! +{healAmount} Kalp yenilendi. Güncel: {health.GetCurrentHealth()}/{health.GetMaxHealth()}");
                Destroy(gameObject);
            }
        }
    }
}
```

3. **İksir Nesnesini Sahnede Hazırlama:**
   - Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`Potion_Red`** yapın). _(Unity alt menüsünde Square, Circle gibi seçenekler çıkar; `Square` seçmeniz yeterlidir. Inspector'da Sprite kısmına iksir görselimizi bağlayınca nesne iksir şeklini alacaktır)._
   - Inspector ayarları:
     - **Sprite:** Kenney paketindeki orijinal yuvarlak kırmızı iksir görselini (**`tile_0115`** / `tilemap_packed_115`) seçin.
     - **Sorting Layer:** **`Interactables`**
     - **Order in Layer:** `1`
     - **Transform Scale:** `X: 0.9, Y: 0.9`
     - **Add Component → Circle Collider 2D:**
       - **Radius:** `0.35`
       - **Is Trigger:** **Mutlaka İŞARETLEYİN (True)!**
     - **Add Component → Loot Bounce:** (Faz 4'te yazdığımız zıplama scripti - kutudan fırlayınca tatlı bir yay çizer).
     - **Add Component → Health Potion:** Az önce yazdığımız script.
4. **Prefab Yapma:** Hierarchy'deki `Potion_Red` nesnesini `Assets/Prefabs` klasörünün içine sürükleyip prefab yapın. Sahnede kalan nesneyi silebilirsiniz.

---

### Adım 7.5: Ahşap Sandık & Testilere İksir Düşürme Şansı Ekleme

Faz 4'te yazdığımız `BreakableCrate.cs` scriptini güncelleyerek, kutular kırıldığında sadece altın değil, bazen can iksiri de düşmesini sağlıyoruz:

1. `Assets/Scripts/BreakableCrate.cs` dosyasını açın.
2. Scripti şu gelişmiş ganimet tablosuyla güncelleyin:

```csharp
using UnityEngine;

public class BreakableCrate : MonoBehaviour, IDamageable
{
    [Header("Görsel Parçalanma")]
    [Tooltip("Kutu kırıldığında patlayacak talaş parçacık prefabı")]
    [SerializeField] private GameObject breakEffectPrefab;

    [Header("Ganimet Prefabları")]
    [Tooltip("Düşebilecek altın sikke prefabı")]
    [SerializeField] private GameObject coinPrefab;

    [Tooltip("Düşebilecek kırmızı iksir prefabı")]
    [SerializeField] private GameObject potionPrefab;

    [Header("Ganimet Olasılıkları (Toplam %100)")]
    [Range(0, 100)] [SerializeField] private int coinChance = 50;   // %50 Altın çıkar
    [Range(0, 100)] [SerializeField] private int potionChance = 25; // %25 İksir çıkar
    // Kalan %25 ihtimalde kutu boş çıkar

    public void TakeDamage(int damageAmount)
    {
        // 1. Talaş patlama efektini kutunun konumunda patlat
        if (breakEffectPrefab != null)
        {
            Instantiate(breakEffectPrefab, transform.position, Quaternion.identity);
        }

        // 2. Şans zarını at (0 ile 100 arası)
        int roll = Random.Range(0, 100);

        if (roll < coinChance)
        {
            // Altın düşür
            if (coinPrefab != null)
            {
                Instantiate(coinPrefab, transform.position, Quaternion.identity);
            }
        }
        else if (roll < (coinChance + potionChance))
        {
            // İksir düşür
            if (potionPrefab != null)
            {
                Instantiate(potionPrefab, transform.position, Quaternion.identity);
            }
        }

        // 3. Kutuyu sahneden yok et
        Destroy(gameObject);
    }
}
```

3. **Prefab Bağlantılarını Tamamlama:**
   - `Assets/Prefabs` klasöründeki **`BreakableCrate.prefab`** ve **`BreakablePot.prefab`** nesnelerine çift tıklayarak Prefab modunu açın.
   - Inspector'da yeni eklenen **Potion Prefab** kutusuna az önce ürettiğimiz `Assets/Prefabs/Potion_Red` prefabını sürükleyip bırakın.
   - Sahneye dönün. _Artık zindandaki kutuları kırdığınızda canınızı kurtaracak iksirlerin fırladığını göreceksiniz!_

---

### Adım 7.6: Altın ve Anahtar Sayaçları (`InventoryUI.cs`)

Oyuncunun topladığı ganimeti ekranda şık bir şekilde görmesi için kalplerin altına altın ve anahtar sayaçlarını ekliyoruz:

1. `Assets/Scripts` klasörüne sağ tıklayın → **Create > C# Script** (Adı: **`InventoryUI`**).
2. Scripti açın ve şu kodu yapıştırın:

```csharp
using UnityEngine;
using UnityEngine.UI;

public class InventoryUI : MonoBehaviour
{
    [Header("Envanter Referansı")]
    [SerializeField] private PlayerInventory playerInventory;

    [Header("Sayaç Metinleri")]
    [Tooltip("Altın sayısını gösteren UI Text")]
    [SerializeField] private Text coinText;

    [Tooltip("Anahtar sayısını gösteren UI Text")]
    [SerializeField] private Text keyText;

    private void Start()
    {
        if (playerInventory == null)
        {
            GameObject player = GameObject.FindGameObjectWithTag("Player");
            if (player != null)
            {
                playerInventory = player.GetComponent<PlayerInventory>();
            }
        }
    }

    private void Update()
    {
        if (playerInventory == null) return;

        if (coinText != null)
        {
            coinText.text = "x " + playerInventory.coinCount.ToString("D2"); // Örnek: x 05
        }

        if (keyText != null)
        {
            keyText.text = "x " + playerInventory.keyCount.ToString(); // Örnek: x 1
        }
    }
}
```

3. **HUD Üzerine Sayaç Nesnelerini Ekleme:**
   - Hierarchy'de `HUD_Canvas` altına sağ tıklayın → **Create Empty** deyin (Adı: **`Counters_Panel`**).
   - Rect Transform: Anchor'ı **Sol Üst (Top-Left)** yapın. **Pos X:** `40`, **Pos Y:** `-95`.
   - **Altın Satırı:**
     - `Counters_Panel` altına sağ tıklayın → **UI > Image** (Adı: `Coin_Icon`). Sprite: `coin.png`, Genişlik/Yükseklik: `28x28`, Pos X: `0`, Pos Y: `0`.
     - `Counters_Panel` altına sağ tıklayın → **UI > Text (veya TextMeshPro)** (Adı: `Coin_Text`). Text: `x 00`, Font Size: `22`, Pos X: `45`, Pos Y: `0`, Renk: Parlak Sarı (`#FFD700`).
   - **Anahtar Satırı:**
     - `Counters_Panel` altına sağ tıklayın → **UI > Image** (Adı: `Key_Icon`). Sprite: `key.png`, Genişlik/Yükseklik: `28x28`, Pos X: `110`, Pos Y: `0`.
     - `Counters_Panel` altına sağ tıklayın → **UI > Text** (Adı: `Key_Text`). Text: `x 0`, Font Size: `22`, Pos X: `150`, Pos Y: `0`, Renk: Beyaz.
   - `Counters_Panel` nesnesine `InventoryUI` scriptini ekleyin ve `Coin_Text` ile `Key_Text` bileşenlerini Inspector'daki yerlerine bağlayın.

---

### Adım 7.7: Oyun Sonu (Game Over) Paneli & [R] Tuşuyla Yeniden Başlatma (`GameOverUI.cs`)

Kahramanın canı 0 olduğunda ekranda karartma ve "OYUN BİTTİ" mesajı çıkartan arcade mekaniği:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`GameOverUI`**).
2. Scriptin içine şu kodu yapıştırın:

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
#if ENABLE_INPUT_SYSTEM
using UnityEngine.InputSystem;
#endif

public class GameOverUI : MonoBehaviour
{
    [Header("Referanslar")]
    [SerializeField] private Health playerHealth;
    [SerializeField] private GameObject gameOverPanel;

    private bool isGameOver = false;

    private void Start()
    {
        if (gameOverPanel != null)
        {
            gameOverPanel.SetActive(false); // Oyun başlarken panel gizli olsun
        }

        if (playerHealth == null)
        {
            GameObject player = GameObject.FindGameObjectWithTag("Player");
            if (player != null) playerHealth = player.GetComponent<Health>();
        }
    }

    private void Update()
    {
        // Can 0'a düştüğünde Game Over panelini aç
        if (!isGameOver && playerHealth != null && playerHealth.GetCurrentHealth() <= 0)
        {
            TriggerGameOver();
        }

        // Oyun bittiyse klavyeden R veya Space tuşuna basarak bölümü yeniden başlat
        if (isGameOver)
        {
            bool restartPressed = false;

#if ENABLE_INPUT_SYSTEM
            if (Keyboard.current != null && (Keyboard.current.rKey.wasPressedThisFrame || Keyboard.current.spaceKey.wasPressedThisFrame))
            {
                restartPressed = true;
            }
#else
            if (Input.GetKeyDown(KeyCode.R) || Input.GetKeyDown(KeyCode.Space))
            {
                restartPressed = true;
            }
#endif

            if (restartPressed)
            {
                RestartGame();
            }
        }
    }

    private void TriggerGameOver()
    {
        isGameOver = true;
        if (gameOverPanel != null)
        {
            gameOverPanel.SetActive(true);
        }
        Debug.Log("☠️ OYUN BİTTİ! Yeniden başlamak için [R] tuşuna basabilirsiniz.");
    }

    public void RestartGame()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }
}
```

3. **Game Over Panelini Canvas'ta Hazırlama:**
   - `HUD_Canvas` altında sağ tıklayın → **UI > Panel** deyin (Adını **`GameOver_Panel`** yapın).
   - Inspector'da **Image** rengini siyah yapın ve Alpha değerini `180` yaparak arkadaki zindanın hafifçe göründüğü yarı-saydam karanlık bir atmosfer verin.
   - `GameOver_Panel` altına sağ tıklayın → **UI > Text**:
     - Text: **`OYUN BİTTİ`**
     - Font Size: `64`
     - Alignment: `Orta / Center`
     - Renk: Kan Kırmızısı (`#FF2020`).
   - Altına bir Text daha ekleyin:
     - Text: **`Zindanı Tekrar Denemek İçin [R] Tuşuna Basın`**
     - Font Size: `28`
     - Renk: Açık Gri / Beyaz.
   - `HUD_Canvas` üzerine `GameOverUI` scriptini ekleyin ve `GameOver_Panel`i scriptteki kutuya sürükleyin.
   - **Önemli:** Hazırladıktan sonra Inspector'dan `GameOver_Panel`in sol üstündeki onay kutusunu (Checkbox) kaldırarak paneli **inaktif (gizli)** hale getirin (kod can 0 olunca açacaktır).

---

## 4. Kendini Test Et & Kontrol Listesi (Self-Check Checklist)

Aşağıdaki adımları sırayla test ederek Faz 7'nin eksiksiz çalıştığını doğrulayın:

- [ ] **Sol Üst Kalp Arayüzü:** Oyunu başlattığınızda sol üst köşede 3 tane parlak kırmızı kalp görünüyor mu?
- [ ] **Hasar ve Kalp Boşalması:** Bir canavara veya zemin dikenine çarptığınızda 3 kalpten biri anında boş/koyu kalbe (`heart_empty`) dönüşüyor mu?
- [ ] **İksir Koruma Kuralı:** Canınız 3/3 iken yerdeki kırmızı iksirin üstünden geçtiğinizde iksir boşa gitmeyip yerde kalıyor mu?
- [ ] **Can İyileşmesi:** Canınız 2 veya 1 iken iksirin üstünden geçtiğinizde iksir yok olup boş kalbiniz anında tekrar kırmızıya dönüyor mu?
- [ ] **Kutu Ganimetleri:** Sandık ve testileri kılıçla kırdığınızda şansa bağlı olarak altın sikke veya kırmızı sağlık iksiri havaya zıplayarak yere düşüyor mu?
- [ ] **Altın ve Anahtar Sayacı:** Yerdeki altınları ve altın anahtarı topladıkça sol üstteki `x 01`, `x 02` sayaçları anlık güncelleniyor mu?
- [ ] **Game Over Ekranı:** Canınız tamamen bitip öldüğünüzde karanlık Game Over paneli ekrana geliyor mu ve `[R]` tuşuna bastığınızda zindan tertemiz baştan başlıyor mu?

---

## 5. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### 1. Canvas veya Kalpler Ekranda Çok Küçük ya da Çok Büyük Görünüyor

- **Sebep:** `Canvas Scaler` bileşeninin `Constant Pixel Size` modunda kalması.
- **Çözüm:** `HUD_Canvas` nesnesini seçin. Inspector'da `Canvas Scaler` bileşenini **`Scale With Screen Size`** yapın ve Reference Resolution değerine `1920 x 1080` yazın.

### 2. Kalp Simgeleri Ekranda Bulanık Görünüyor

- **Sebep:** Sprite import ayarlarında filtrenin açık olması.
- **Çözüm:** `Assets/Art/heart_full` ve `heart_empty` dosyalarını seçin. Inspector'da **Filter Mode** ayarını mutlaka **`Point (no filter)`** ve **Compression** ayarını **`None`** yapıp Apply'a basın.

### 3. Karakter Tam Candayken de İksirleri Yutuyor ve İksirler Kayboluyor

- **Sebep:** `HealthPotion.cs` içindeki can kontrolü koşulunun eksik olması.
- **Çözüm:** Scriptte `if (health.GetCurrentHealth() < health.GetMaxHealth())` kontrolünün yer aldığından emin olun.

### 4. Game Over Ekranı Açılınca [R] Tuşuna Basınca Oyun Tepki Vermiyor

- **Sebep:** Projenizde New Input System aktifken eski `Input.GetKeyDown` metodunun çağrılması.
- **Çözüm:** `GameOverUI.cs` içinde verdiğimiz `#if ENABLE_INPUT_SYSTEM` kod bloğu her iki sistemi de otomatik destekler. Scriptinizin güncel olduğundan emin olun.

---

## 6. Projeyi Git ile Kaydetme & Yedekleme (Adım Adım)

Tebrikler! Zindan oyununuz artık can kalpleri, sağlık iksirleri, HUD envanter sayaçları ve tam bir oyun döngüsü kazandı! Yapılan tüm bu harika işleri Git ile güvenle kaydedelim:

1. Unity'de sahnenizi ve projenizi kaydedin: **`File > Save`** ve **`File > Save Project`** (`Ctrl + S`).
2. Terminal penceresini açın ve projenizin ana dizininde olduğunuzdan emin olun.
3. Sırasıyla şu 3 komutu çalıştırın:

```bash
git add .
git commit -m "Faz 7 tamamlandi: 3 Kalp UI, Saglik Iksirleri, Envanter Sayaclari ve Game Over ekrani eklendi"
git push origin main
```

> ⚠️ **Terminal Yazım İpucu:** Eğer push komutunu yazarken harf hatası yaparsanız (`git psuh` gibi), terminal `git: 'psuh' is not a git command` uyarısı verir. Doğru komut **`git push origin main`** şeklindedir!
