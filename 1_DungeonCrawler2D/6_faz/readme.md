# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 6

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Zindan İlerleme Döngüsü (Zelda Mantığı), Altın Anahtar Sistemi (`KeyPickup.cs`), Kilitli Demir Geçit Kapısı (`LockedDoor.cs`), Zamanlamalı Zemin Dikenleri / Mazgal Tuzağı (`SpikeTrap.cs`) ve Koridor Engelleri

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 5'te canavarlarımızı (Hayalet ve Yarasa) zindana indirdik, evrensel can sistemini kurduk ve kılıcımızla düello yapabileceğimiz aksiyon döngüsünü inşa ettik. Ancak zindan oyunlarının en büyüleyici parçası sadece canavar kesmek değil; **keşif, kapalı kapılar, gizli anahtarlar ve ölümcül tuzaklardır (Zelda-Lite döngüsü)**.

Bu kılavuzu tamamladığınızda:

1. **Klasik Zindan İlerleme Döngüsünü (Lock & Key Loop):** Oyuncunun önce kilitli bir kapıyla karşılaşıp hedef belirlemesini, ardından zindanın tehlikeli odalarını aşarak anahtarı ele geçirmesini sağlayan seviye tasarım felsefesini kavrayacaksınız.
2. **Toplanabilir Altın Anahtar (`KeyPickup.cs`):** Yerde parıldayan, alındığında oyuncunun envanterine eklenen (`PlayerInventory.AddKey()`) ve konsolda geri bildirim veren anahtar mekaniğini kuracaksınız.
3. **Kilitli Demir Kapı (`LockedDoor.cs`):** Anahtarsız geçit vermeyen katı bir engel oluşturan; oyuncu anahtarla dokunduğunda kilidi açılıp içinden geçilebilir açık bir geçide dönüşen kapı sistemini kodlayacaksınız.
4. **Zamanlamalı Zemin Dikenleri (`SpikeTrap.cs`):** C# Coroutine kullanarak 2 saniyede bir zeminden fırlayan ve açıkken basan oyuncuya `IDamageable` üzerinden hasar veren dinamik bir çevre tuzağı üreteceksiniz.
5. **Seviye Entegrasyonu:** [tiny-dungeon-level.md](file:///Users/paban/UnityAgent/tiny-dungeon-level.md) seviye şemamıza sadık kalarak 4. koridora diken tuzaklarını, 3. odaya altın anahtarı ve 5. oda girişine kilitli demir kapıyı yerleştireceksiniz.

---

## 1. Zihinsel Model: Zelda Tipi "Kilit & Anahtar" Döngüsü

Oyun tasarımında oyuncuya rastgele yürümek yerine **net bir amaç ve keşif motivasyonu** vermek gerekir:

```text
                           [ 1. MERKEZ ODA ]
                                  │
                                  ▼
                     [ KİLİTLİ DEMİR KAPI ] ◄──────┐
                   ("Kapı kilitli! Anahtar lazım") │
                                  │                │
            ┌─────────────────────┴──────┐         │
            │                            │         │
            ▼                            ▼         │
    [ 2. TUZAK ODASI ]           [ 3. SAVAŞ ODASI ]│
 (Zamanlamalı Dikenler)       (Hayalet & Yarasa Dövüşü)
            │                            │         │
            │                            ▼         │
            │                  [ 🗝️ ALTIN ANAHTAR ]│
            │                   (Envantere eklenir) │
            │                            └─────────┘
            ▼
    [ 5. HAZİNE & ÇIKIŞ ODASI ] (Kapı açıldı! Zafer!)
```

---

## 2. Kullanılacak Görseller ve Katman Ayarları

| Varlık Adı                 | Görsel Kaynağı      | Sprite Adı / No                   | Görevi                                               |
| :------------------------- | :------------------ | :-------------------------------- | :--------------------------------------------------- |
| **Altın Anahtar**          | Harici Özel Asset   | **`key.png`** (16x16)             | Kilitli kapıyı açan toplanabilir anahtar             |
| **Kilitli Demir Kapı**     | Kenney Tiny Dungeon | `tile_0045` (`tilemap_packed_45`) | Kapalıyken geçit vermeyen demir parmaklık            |
| **Açık Kapı Kemeri**       | Kenney Tiny Dungeon | `tile_0021` (`tilemap_packed_21`) | Kilit açıldığında dönüşeceği açık geçit              |
| **Kapalı Zemin Tuzağı**    | Kenney Tiny Dungeon | `tile_0049` (`tilemap_packed_49`) | Dikenler gizliyken görünen düz zemin taşı            |
| **Açık / Tehlikeli Diken** | Kenney Tiny Dungeon | `tile_0041` (`tilemap_packed_41`) | Dikenler fırladığında hasar veren sivri zemin tuzağı |

- **Sorting Layer:**
  - **Demir Kapı:** **`Walls`** (Order in Layer: `1`)
  - **Altın Anahtar:** **`Interactables`** (Order in Layer: `1`)
  - **Zemin Tuzağı:** **`Floor`** (Order in Layer: `1` - Zeminin hemen üstünde çizilsin)

---

## 3. FAZ 6 Adım Adım Uygulama Rehberi

---

### Adım 6.1: Altın Anahtar Görselini Projeye Alma

Kenney paketinde sikke olmadığı gibi metal anahtar sprite'ı da bulunmaz. Bu yüzden Faz 4'teki altın sikke gibi, retro 16x16 piksel tarzına tam uyumlu **`key.png`** görseli projenizin ana dizinine hazırlandı:

1. Proje ana dizinindeki **`key.png`** dosyasını Unity'de **`Assets/Art`** klasörünün içine sürükleyin.
2. `Assets/Art/key.png` seçiliyken sağdaki **Inspector** panelinde şu 3 ayarı yapıp **Apply** butonuna basın:
   - **Pixels Per Unit:** `16`
   - **Filter Mode:** `Point (no filter)`
   - **Compression:** `None`

---

### Adım 6.2: Toplanabilir Altın Anahtar Prefabı (`KeyPickup.cs`)

Oyuncu anahtarın üstünden geçtiğinde anahtarı envantere kaydedip sahneden silen script:

1. **Project** panelinde `Assets/Scripts` klasörüne gidin.
2. Sağ tıklayın → **Create > C# Script** (Adı tam olarak: **`KeyPickup`**).
3. Scripti açın ve şu kodu yapıştırın:

```csharp
using UnityEngine;

public class KeyPickup : MonoBehaviour
{
    private bool isCollected = false;

    private void OnTriggerEnter2D(Collider2D other)
    {
        TryCollect(other);
    }

    private void OnTriggerStay2D(Collider2D other)
    {
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
            inventory.AddKey();
            Debug.Log("🗝️ TEBRİKLER! Altın Zindan Anahtarını buldun! Artık kilitli kapıyı açabilirsin.");
            Destroy(gameObject);
        }
    }
}
```

4. **Anahtar Nesnesini Sahnede Hazırlama:**
   - Hierarchy panelinde boş bir yere sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`Key_Gold`** yapın). _(Unity alt menüsünde Square, Circle gibi seçenekler çıkar; `Square` seçmeniz yeterlidir. Inspector'da Sprite kısmına anahtar görselimizi bağlayınca nesne anahtar şeklini alacaktır)._
   - Inspector ayarları:
     - **Sprite:** `Assets/Art/key` görselini seçin.
     - **Sorting Layer:** **`Interactables`**
     - **Order in Layer:** `1`
     - **Add Component → Circle Collider 2D:**
       - **Radius:** `0.35`
       - **Is Trigger:** **Mutlaka İŞARETLEYİN (True)!**
     - **Add Component → Loot Bounce:** (Yere fırlayınca zıplama efekti için).
     - **Add Component → Key Pickup:** Az önce yazdığımız scripti ekleyin.
5. **Prefab Yapma:** Hierarchy'deki `Key_Gold` nesnesini tutup **`Assets/Prefabs`** içine sürükleyin. Sahnede kalan nesneyi silin.

---

### Adım 6.3: Kilitli Demir Geçit Kapısı (`LockedDoor.cs`)

Karakter anahtarsızken geçemeyen; ancak anahtarla dokunduğunda kilidi açılıp açık kapıya dönüşen kapı mimarisi:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`LockedDoor`**).
2. Scripti açın ve şu kodu yapıştırın:

```csharp
using UnityEngine;

public class LockedDoor : MonoBehaviour
{
    [Header("Görseller")]
    [Tooltip("Kilit açıldığında dönüşeceği açık kapı görseli (tile_0021)")]
    [SerializeField] private Sprite openedDoorSprite;

    private BoxCollider2D doorCollider;
    private SpriteRenderer spriteRenderer;
    private bool isUnlocked = false;

    private void Awake()
    {
        doorCollider = GetComponent<BoxCollider2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void OnCollisionEnter2D(Collision2D collision)
    {
        if (isUnlocked) return;

        // Kapıya çarpan kişi Oyuncu mu?
        if (collision.gameObject.CompareTag("Player"))
        {
            PlayerInventory inventory = collision.gameObject.GetComponent<PlayerInventory>();
            if (inventory != null && inventory.UseKey())
            {
                Unlock();
            }
            else
            {
                Debug.Log("🔒 Bu kapı kilitli! Açmak için zindandaki Altın Anahtarı bulmalısın.");
            }
        }
    }

    private void Unlock()
    {
        isUnlocked = true;

        // 1. Fiziksel engeli kaldır (Artık içinden geçilebilir)
        if (doorCollider != null)
        {
            doorCollider.enabled = false;
        }

        // 2. Görseli açık kapıya çevir (veya nesneyi sil)
        if (openedDoorSprite != null)
        {
            spriteRenderer.sprite = openedDoorSprite;
        }
        else
        {
            Destroy(gameObject);
        }

        Debug.Log("🔓 KLİK! Kilit açıldı, demir parmaklıklar aralandı!");
    }
}
```

3. **Kapı Nesnesini Sahnede Hazırlama:**
   - Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adını **`LockedDoor`** yapın). _(Alt menüden Square seçin)._
   - Inspector ayarları:
     - **Sprite:** Kenney demir parmaklık görselini (**`tile_0045`** / `tilemap_packed_45`) seçin.
     - **Sorting Layer:** **`Walls`**
     - **Order in Layer:** `1`
     - **Add Component → Box Collider 2D:**  
       _(DİKKAT: Is Trigger kutucuğunu İŞARETLEMEYİN! Kapalıyken katı bir duvar gibi durmalıdır)._
     - **Add Component → Locked Door:**
       - **Opened Door Sprite:** Açık kapı kemeri görselini (**`tile_0021`** / `tilemap_packed_21`) sürükleyip bağlayın.
4. **Prefab Yapma:** Bu `LockedDoor` nesnesini tutup **`Assets/Prefabs`** içine sürükleyin.

---

### Adım 6.4: Zamanlamalı Zemin Dikenleri / Mazgal Tuzağı (`SpikeTrap.cs`)

Koridor zemininde düzenli aralıklarla açılıp kapanan ölümcül çevre tuzağı:

1. `Assets/Scripts` içine sağ tıklayın → **Create > C# Script** (Adı: **`SpikeTrap`**).
2. Scripti açın ve şu kodu yapıştırın:

```csharp
using System.Collections;
using UnityEngine;

public class SpikeTrap : MonoBehaviour
{
    [Header("Tuzak Görselleri (Kenney Tiles)")]
    [Tooltip("Dikenler gizliyken görünen düz taş (tile_0049)")]
    [SerializeField] private Sprite spikesRetracted;

    [Tooltip("Dikenler fırladığında görünen tehlikeli sivri diken (tile_0041)")]
    [SerializeField] private Sprite spikesExtended;

    [Header("Zamanlama & Hasar")]
    [Tooltip("Dikenlerin kaç saniyede bir açılıp kapanacağı")]
    [SerializeField] private float toggleInterval = 2.0f;

    [Tooltip("Açık dikenlere basıldığında verilecek hasar puanı")]
    [SerializeField] private int trapDamage = 1;

    private SpriteRenderer spriteRenderer;
    private bool isSpikeActive = false;

    private void Awake()
    {
        spriteRenderer = GetComponent<SpriteRenderer>();
        if (spikesRetracted != null) spriteRenderer.sprite = spikesRetracted;

        StartCoroutine(TrapCycleRoutine());
    }

    private IEnumerator TrapCycleRoutine()
    {
        while (true)
        {
            yield return new WaitForSeconds(toggleInterval);
            isSpikeActive = !isSpikeActive;

            if (isSpikeActive)
            {
                // Dikenler çıktı (Tehlike!)
                if (spikesExtended != null) spriteRenderer.sprite = spikesExtended;
            }
            else
            {
                // Dikenler içeri çekildi (Güvenli)
                if (spikesRetracted != null) spriteRenderer.sprite = spikesRetracted;
            }
        }
    }

    private void OnTriggerStay2D(Collider2D other)
    {
        // SADECE dikenler dışarıdayken ve basan kişi Oyuncu ise hasar ver
        if (isSpikeActive && other.CompareTag("Player"))
        {
            IDamageable damageable = other.GetComponent<IDamageable>();
            if (damageable != null)
            {
                damageable.TakeDamage(trapDamage);
            }
        }
    }
}
```

3. **Tuzak Nesnesini Sahnede Hazırlama:**
   - Hierarchy'de sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin (Adı: **`SpikeTrap`**). _(Alt menüden Square seçin)._
   - Inspector ayarları:
     - **Sprite:** Düz taş görselini (**`tile_0049`** / `tilemap_packed_49`) seçin.
     - **Sorting Layer:** **`Floor`**
     - **Order in Layer:** `1` _(Zeminin üstünde görünsün)._
     - **Add Component → Box Collider 2D:**
       - **Is Trigger:** **Mutlaka İŞARETLEYİN (True)!** _(Karakter tuzağa basıp üzerinden geçebilmelidir)._
     - **Add Component → Spike Trap:**
       - **Spikes Retracted:** `tile_0049` (Düz taş)
       - **Spikes Extended:** `tile_0041` (Dikenli zemin tuzağı)
       - **Toggle Interval:** `2.0`
       - **Trap Damage:** `1`
4. **Prefab Yapma:** `SpikeTrap` nesnesini **`Assets/Prefabs`** içine sürükleyip prefab yapın.

---

### Adım 6.5: Seviye Şemasına Sadık Yerleşim (Zindana Dizme)

[tiny-dungeon-level.md](file:///Users/paban/UnityAgent/tiny-dungeon-level.md) seviye haritasına göre nesneleri yerleştiriyoruz:

1. **3. Oda (Savaş & Anahtar Odası):**
   - Faz 5'te yerleştirdiğimiz Hayalet ve Yarasa canavarlarının arkasındaki sunak/köşeye 1 adet **`Key_Gold`** prefabı koyun. _(Oyuncu düşmanları temizleyip bu anahtarı almak zorundadır)._
2. **4. Koridor (Tuzak Koridoru & Kestirme):**
   - Koridorun zeminine yan yana 2 veya 3 adet **`SpikeTrap`** prefabı yerleştirin.
3. **5. Oda Girişi (Hazine Odası Önü):**
   - Hazine odasına geçişi sağlayan 1 karo genişliğindeki kapı boşluğuna **`LockedDoor`** prefabını yerleştirin.

---

### Adım 6.6: Büyük Oynanış Testi!

1. **Play (▶️)** butonuna basın.
2. Doğrudan kilitli demir kapıya doğru yürüyün:
   - Karakter kapıya çarpıp duracak ve konsolda _"Bu kapı kilitli!"_ uyarısı çıkacak mı?
3. 3. Odaya gidin, canavarları savuşturup **Altın Anahtarın** üzerinden geçin:
   - Konsolda anahtarın toplandığı yazacak ve anahtar sahneden kaybolacak mı?
4. Kilitli kapıya geri dönün ve kapıya dokunun:
   - Kapının parmaklıkları açılacak ve oyuncu artık engelsizce Hazine Odası'na geçebilecek mi?
5. Koridordaki **Zemin Dikenlerinin** üzerinden geçin:
   - Dikenler kapalıyken hasar almayıp, tam fırladığı anda basarsanız şövalyenizin canı azalıp kırmızı parlayacak mı?

---

## 4. Öğrenci Kontrol Listesi (Self-Check Checklist)

Faz 6'yı tamamladığınızı doğrulamak için şu adımları kontrol edin:

- [ ] `Assets/Art/key.png` dosyası PPU: 16 ve Point (no filter) olarak import edildi.
- [ ] `Assets/Scripts/KeyPickup.cs` scripti oluşturuldu ve `Key_Gold` prefabına eklendi.
- [ ] `Assets/Scripts/LockedDoor.cs` scripti oluşturuldu ve katı `Box Collider 2D`li `LockedDoor` prefabına eklendi.
- [ ] `Assets/Scripts/SpikeTrap.cs` scripti oluşturuldu ve `Is Trigger` işaretli `SpikeTrap` prefabına eklendi.
- [ ] Anahtar olmadan kilitli kapıdan geçilemiyor.
- [ ] Anahtar alındıktan sonra kapıya dokunulduğunda kapı açılıyor ve geçiş sağlanıyor.
- [ ] Zemin tuzakları 2 saniyede bir düzenli olarak açılıp kapanıyor ve açıkken basılınca hasar veriyor.

---

## 5. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Anahtarı aldım ama kapıya dokununca hala "Bu kapı kilitli" diyor!

> **Çözüm:**
>
> 1. Hierarchy'de `Player` nesnenizin Inspector'ında **`PlayerInventory`** bileşeninin takılı olduğunu kontrol edin.
> 2. Anahtarı aldığınızda konsolda _"Altın Anahtar Alındı!"_ yazısını gördünüz mü? Eğer görmediyseniz `Key_Gold` prefabındaki `Circle Collider 2D > Is Trigger` ayarının açık olduğunu kontrol edin.

### ❓ Soru 2: Demir kapı açıldığında içinden hala geçemiyorum, görünmez bir duvara çarpıyorum!

> **Çözüm:** `LockedDoor.cs` scriptinde kilit açıldığında `doorCollider.enabled = false;` satırının çalıştığından emin olun. Ayrıca kapının arkasında Faz 1'deki Tilemap duvar collider'ı kalmış olabilir; kapının durduğu koordinatta `Tilemap_Walls` karosu olmamalıdır.

### ❓ Soru 3: Diken tuzağının üstünde duruyorum ama hiç hasar almıyorum!

> **Çözüm:**
>
> 1. `SpikeTrap` üzerindeki `Box Collider 2D`nin **`Is Trigger`** kutucuğunun işaretli olduğunu doğrulayın.
> 2. `SpikeTrap.cs` içinde `OnTriggerEnter2D` yerine **`OnTriggerStay2D`** kullanıldığından emin olun (çünkü oyuncu tuzağın üstünde beklerken dikenler sonradan açılabilir).
> 3. `Player` nesnenizde Faz 5'te yazdığımız **`Health.cs`** bileşeninin ekli olduğunu teyit edin.

---

🎉 **Tebrikler!** FAZ 6'yı başarıyla tamamladınız. Artık zindanınızda bulmaca ve keşif hissi veren bir anahtar-kilit sistemi ve zamanlama gerektiren ölümcül zemin tuzakları var!

Bir sonraki ve son aşamalardan biri olan **FAZ 7**'de ekranda oyuncunun canını gösteren grafiksel **Kalp Arayüzü (HUD Canvas)** ve yaralandığımızda can yenileyen **Sağlık İksirleri (`HealthPotion.cs`)** mekaniğini inşa edeceğiz!
