# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 2

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Karakteri Canlandırma, Unity New Input System Mimarisi, 2D Fizik (Rigidbody2D & Collider), 8 Yönlü Pürüzsüz Hareket ve C# Kodlaması

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 1'de zindanımızın taş zeminlerini döşedik ve etrafına geçilmez, katı duvarlar ördük. Ancak dünyamız henüz bomboş ve hareketsiz. Bu fazda, zindanın labirentlerinde maceraya atılacak cesur şövalyemizi sahneye çıkarıyoruz!

Bu kılavuzu tamamladığınızda:

1. Unity'nin modern girdi standardı olan **New Input System** paketini kurmayı ve yapılandırmayı öğreneceksiniz.
2. Neden eski `Input.GetAxis()` yöntemi yerine profesyonel oyun stüdyolarının kullandığı **Eylem Haritası (Input Actions)** mimarisine geçtiğimizi kavrayacaksınız.
3. Top-down (kuşbakışı) 2D oyunlarda yer çekimini sıfırlama (`Gravity Scale = 0`) ve açısal kilit koyma (`Freeze Rotation Z`) fiziğini öğreneceksiniz.
4. Karakterin duvarların önünden geçerken kafasının duvarın arkasında kaybolmasını engelleyen, 2.5D derinlik illüzyonu sunan **"Ayak Çarpışma Kutusu" (Footstep Collider)** tekniğini uygulayacaksınız.
5. C# ile **8 yönlü (WASD)** hareket kodunu yazacak; çapraz yürürken karakterin iki kat hızlanmasını önleyen **Vektör Normalizasyonu (`Vector2.normalized`)** matematiğini kavrayacaksınız.
6. Karakterin baktığı yöne göre yüzünü otomatik çevirmesini (`SpriteRenderer.flipX`) sağlayacaksınız.

---

## 1. 2D Top-Down Oyunlarda Fizik ve Girdi Zihinsel Modeli

Koda ve motor ayarlarına geçmeden önce, arkasındaki mantığı bir oyun geliştirici gibi düşünelim:

### A) Platform Oyunları ile Zindan Oyunlarının Fizik Farkı

- **2D Platform Oyunu (Mario):** Karakter yandan görünür. Yer çekimi aşağı çeker, zıplama tuşu yukarı fırlatır.
- **2D Zindan Oyunu (Zelda / Tiny Dungeon):** Dünyaya yukarıdan, kuşbakışı bakarız (Top-Down). Bu yüzden dünyada **aşağıya doğru çeken bir yer çekimi OLAMAZ!** Eğer yer çekimini kapatmazsak şövalyemiz oyun başlar başlamaz sonsuz boşluğa düşer.

### B) Neden Unity New Input System?

Unity'nin eski girdi sistemi (`Input.GetKey(KeyCode.W)`), tuşları doğrudan kodun içine çakar. Yarın bir gün oyuna Gamepad kolu desteği eklemek veya oyuncunun tuşları değiştirmesine (Key Rebinding) izin vermek istediğinizde tüm kod çöker.  
**New Input System** ise bir soyutlama katmanı kurar:

- Oyuncu neye bastı? -> Klavyeden `W` tuşuna veya koldan `Sol Analog Yukarı`ya.
- Kod ne anlar? -> Tek bir şey: **"Move" (Hareket)** aksiyonu tetiklendi!

### C) Render Döngüsü (`Update`) vs Fizik Döngüsü (`FixedUpdate`)

- **`Update()`:** Monitörünüzün yenileme hızına göre (60 FPS, 144 FPS) her karede bir çalışır. Oyuncunun klavye tuşuna basıp basmadığını kaçırmamak için girdi okuması burada yapılır.
- **`FixedUpdate()`:** Monitörden bağımsız olarak saniyede sabit 50 kez (0.02 saniyede bir) çalışır. Unity'nin fizik motoru (`Rigidbody2D`) burada hesaplanır. Fiziksel hareketi burada yapmazsak, yüksek FPS'li bilgisayarlarda karakter duvardan geçer, düşük FPS'li bilgisayarlarda ise takılarak yürür.

---

## 2. Karakter Görseli ve 2.5D Derinlik İllüzyonu (Ayak Kutusu İlkesi)

Kenney Tiny Dungeon paketinde birden fazla kahraman karakteri bulunur:

- **Gümüş Zırhlı Şövalye:** `tile_0096` (`tilemap_packed_96`)
- **Açık Vizörlü Şövalye:** `tile_0097` (`tilemap_packed_97`)

Biz projemizde klasik gümüş zırhlı şövalyeyi (`tile_0096`) ana karakter olarak kullanacağız.

```text
       [ ŞÖVALYE GÖRSELİ ]
     ┌─────────────────────┐
     │      ( ) Baş        │  ──> Çarpışma YOK (Duvarın önünden geçer)
     │     / | \ Gövde     │  ──> Çarpışma YOK
     ├─────────────────────┤
     │     /   \ Ayaklar   │  ──> [ Box Collider 2D ] (Sadece burası katı!)
     └─────────────────────┘
```

> [!IMPORTANT]
> **Ayak Kutusu İlkesi (Footstep Collider):**  
> Karakterin tüm boyuna (başından ayağına) bir çarpışma kutusu koyarsanız, kuzeydeki bir duvarın önüne geldiğinde kafası duvara çarpar ve duvardan 1 metre uzakta durmak zorunda kalır.  
> Oysa çarpışma kutusunu **yalnızca ayak bileklerine** yerleştirirseniz; şövalyenin ayakları duvarın dibinde dururken, başı ve miğferi duvarın alt cephesine hafifçe biner. Bu durum 2D piksel oyunlarında muhteşem bir **3 boyutlu derinlik hissi** oluşturur!

---

## 3. Unity New Input System Paketi Kurulumu

Eğer projenizde henüz New Input System yüklü değilse, ilk işimiz bu paketi motora entegre etmektir:

1. Unity üst menüsünden **Window > Package Manager** penceresini açın.
2. Sol üstteki açılır menüden **Packages: Unity Registry** seçeneğini işaretleyin.
3. Sağ üstteki arama kutusuna `Input System` yazın.
4. Listeden sarı/turuncu ikonlu **Input System** paketini seçin ve sağ alttaki **Install** butonuna tıklayın.
5. Kurulum bittiğinde Unity karşınıza şu uyarıyı çıkaracaktır:  
   _`"This project is using the new input system package but the native platform backends have not been enabled yet. Would you like to enable them? This will restart the editor and disable the old input backends."`_
6. Bu pencereye mutlaka **"Yes"** deyin. Unity kendini kaydedip kapatacak ve yeniden başlatacaktır.

> [!TIP]
> **Editör Yeniden Başlamadıysa veya Manuel Kontrol Etmek İsterseniz:**  
> **Edit > Project Settings > Player** sekmesine gidin. **Other Settings** başlığı altındaki **Active Input Handling** ayarının `Input System Package (New)` veya `Both` olduğundan emin olun.

---

## 4. FAZ 2 Adım Adım Uygulama Rehberi

---

### Adım 2.1: Ayarlar Klasörü ve Girdi Haritası (Input Actions) Oluşturma

Karakterimizin hareket girdilerini yöneteceğimiz ayar dosyasını hazırlıyoruz:

1. **Project** panelinde `Assets` klasörüne sağ tıklayın → **Create > Folder** deyin, adını **`Settings`** koyun.
2. `Settings` klasörünün içine girin. Boş bir yere sağ tıklayın → **Create > Input Actions** seçeneğine tıklayın.
3. Dosyanın adını tam olarak **`PlayerInputActions`** yapın.
4. Oluşturduğunuz `PlayerInputActions` dosyasına çift tıklayarak **Input Actions Editor** penceresini açın.

---

### Adım 2.2: 8 Yönlü WASD Hareket Eylemini (Action) Yapılandırma

Açılan pencerede 3 dikey kolon göreceksiniz: **Action Maps**, **Actions**, ve **Action Properties**.

1. **Action Map Ekleme:**
   - Sol kolondaki **Action Maps** başlığının yanındaki `+` butonuna tıklayın.
   - Adına **`Player`** yazın ve Enter'a basın.

2. **Move Eylemini Tanımlama:**
   - Orta kolonda otomatik oluşan `New action` metnine çift tıklayın (veya sağ tıklayıp Rename deyin) ve adını **`Move`** yapın.
   - `Move` seçiliyken sağdaki **Action Properties** paneline bakın:
     - **Action Type:** `Value` seçin _(Tuş basıldığında yön vektörü değeri taşısın)._
     - **Control Type:** `Vector 2` seçin _(X ve Y eksenlerinde -1 ile +1 arası yön verisi)._

3. **WASD 2D Vektör Tuşlarını Atama (Composite Binding):**
   - Orta kolondaki `Move` satırının sağındaki küçük `+` butonuna tıklayın.
   - Açılan menüden **Add Up\Down\Left\Right Composite** (veya _Add 2D Vector Composite_) seçeneğini seçin.
   - `Move` satırının altına açılan yönlere sırasıyla tıklayıp tuşları bağlayın:
     - **Up:** Tıklayın → Sağ panelde **Path:** açılır kutusundan **Listen** butonuna basın → Klavyeden `W` tuşuna basın ve listeden `W [Keyboard]` seçin.
     - **Down:** Tıklayın → **Path:** Listen → Klavyeden `S` tuşuna basın (`S [Keyboard]`).
     - **Left:** Tıklayın → **Path:** Listen → Klavyeden `A` tuşuna basın (`A [Keyboard]`).
     - **Right:** Tıklayın → **Path:** Listen → Klavyeden `D` tuşuna basın (`D [Keyboard]`).

4. **Kaydetme:**
   - Pencerenin üst barında bulunan **Save Asset** butonuna tıklayın.
   - Pencereyi kapatabilirsiniz.

---

### Adım 2.3: Oyuncu Karakterini Sahneye Koyma ve Görsel Ayarları

1. **Project** panelinde `Assets/Art` klasöründeki `tilemap_packed` görselinin yanındaki küçük oka tıklayarak dilimlenmiş karoları genişletin.
2. Şövalye görselini bulun: **`tilemap_packed_96`** (veya liste modundaysanız 96 numaralı karo).
3. Bu görseli tutun ve doğrudan **Hierarchy** paneline (veya Scene ekranına) sürükleyip bırakın.
4. Hierarchy'de oluşan bu nesneye tıklayın, `F2` tuşuna basıp adını tam olarak **`Player`** yapın.
5. **Inspector Panelindeki Kritik Ayarlar:**
   - **Tag:** Inspector'ın en üstündeki `Tag` açılır kutusundan **`Player`** seçin. _(Eğer listede yoksa 'Add Tag' deyip 'Player' yazın ve tekrar seçin. Bu tag, ileriki fazlarda kılıcın ve düşmanların oyuncuyu tanıması için zorunludur!)_
   - **Transform Position:** `X: 0`, `Y: 0`, `Z: 0` yapın _(Karakteri çizdiğiniz odanın ortasında bir zemin koordinatına da taşıyabilirsiniz)._
   - **Sprite Renderer Bileşeni:**
     - **Sorting Layer:** Faz 1'de oluşturduğumuz **`Characters`** katmanını seçin!
     - **Order in Layer:** `0` olarak kalsın.

---

### Adım 2.4: Karakter Fizik Bileşenleri (Rigidbody 2D Ayarları)

Karakterimizin katı duvarlara çarpıp durabilmesi için bir fizik gövdesine (`Rigidbody 2D`) ihtiyacı vardır. Ancak yanlış bir ayar karakterin uçmasına veya düşmesine yol açabilir:

1. Hierarchy'den **`Player`** nesnesini seçin.
2. Inspector'ın en altındaki **Add Component** butonuna tıklayın → **`Rigidbody 2D`** yazıp ekleyin.
3. Şu 4 ayarı **harfiyen** uygulayın:
   - **Body Type:** `Dynamic` _(Karakter hareketli bir varlık olduğu için dinamik kalmalı)._
   - **Gravity Scale:** Mutlaka **`0`** yapın!  
     _(DİKKAT: Varsayılan değer 1'dir. Sıfır yapmazsanız yer çekimi devreye girer ve şövalyeniz zindanın zemininden aşağı uzaya düşer!)._
   - **Collision Detection:** `Continuous` yapın.  
     _(Varsayılan `Discrete` moddur. Continuous modu, karakter hızlı koşarken veya duvara sert çarptığında duvarın içinden sızmasını -tunnelling- tamamen engeller)._
   - **Constraints (Kısıtlamalar):** `Constraints` başlığını genişletin ve **`Freeze Rotation > Z`** kutucuğunu işaretleyin!  
     _(HAYATİ AYAR: Eğer Z rotasyonunu dondurmazsanız, şövalye duvarın köşesine veya bir sandığa hafifçe sürttüğü anda tekerlek gibi kendi etrafında fırıldak şeklinde dönmeye başlar)._

---

### Adım 2.5: Ayak Çarpışma Kutusunu Boyutlandırma (Box Collider 2D)

Şimdi 2. Bölümde bahsettiğimiz 2.5D "Ayak Kutusu" tekniğini uyguluyoruz:

1. `Player` nesnesi seçiliyken Inspector'dan **Add Component** → **`Box Collider 2D`** ekleyin.
2. `Box Collider 2D` bileşeninin üzerindeki **Edit Collider** (üzerinde küçük noktalar olan kutu ikonu) butonuna tıklayın.
3. Scene ekranında karakterin etrafında beliren yeşil çizgileri fareyle tutarak düzenleyin:
   - Kutunun üst kenarını aşağı çekerek karakterin gövdesini ve başını boşta bırakın.
   - Kutuyu sadece **şövalyenin ayak bilekleri ve tabanını** kaplayacak ince bir dikdörtgen haline getirin.
4. Alternatif olarak doğrudan Inspector'dan şu değerleri girebilirsiniz:
   - **Size:** `X: 0.7`, `Y: 0.35`
   - **Offset:** `X: 0`, `Y: -0.32`

---

### Adım 2.6: Karakter Hareket Kodunu Yazma (PlayerController.cs)

Şimdi karakterimizi klavyeyle yöneteceğimiz profesyonel C# kodunu yazıyoruz:

1. **Project** panelinde `Assets/Scripts` klasörüne girin.
2. Boş bir yere sağ tıklayın → **Create > C# Script** seçin.
3. Dosyanın adını tam olarak **`PlayerController`** yapın _(Büyük-küçük harfe dikkat edin)._
4. Dosyaya çift tıklayıp kod editörünüzde açın ve içerisindeki her şeyi silip şu kodu yapıştırın:

```csharp
using UnityEngine;
using UnityEngine.InputSystem;

public class PlayerController : MonoBehaviour
{
    [Header("Hareket Parametreleri")]
    [Tooltip("Karakterin saniyedeki yürüme hızı (birim/saniye)")]
    [SerializeField] private float moveSpeed = 5f;

    [Header("Girdi Referansı (New Input System)")]
    [Tooltip("PlayerInputActions içindeki Player/Move aksiyonu")]
    [SerializeField] private InputActionReference moveAction;

    // Dahili Bileşen Referansları
    private Rigidbody2D rb;
    private SpriteRenderer spriteRenderer;

    // Klavyeden anlık okunan yön vektörü (X: -1 ile +1, Y: -1 ile +1)
    private Vector2 moveInput;

    private void Awake()
    {
        // Karakterin üzerindeki fizik ve görsel bileşenlerini hafızaya al
        rb = GetComponent<Rigidbody2D>();
        spriteRenderer = GetComponent<SpriteRenderer>();
    }

    private void OnEnable()
    {
        // Script aktif olduğunda klavye dinleyicisini aç
        if (moveAction != null)
        {
            moveAction.action.Enable();
        }
    }

    private void OnDisable()
    {
        // Script veya karakter devre dışı kaldığında dinleyiciyi kapat (Hafıza sızıntısını önler)
        if (moveAction != null)
        {
            moveAction.action.Disable();
        }
    }

    private void Update()
    {
        // 1. Klavyeden gelen WASD verisini her render karesinde oku
        if (moveAction != null)
        {
            moveInput = moveAction.action.ReadValue<Vector2>();
        }

        // 2. Görsel Çevirme: Karakter sola yürüyorsa sprite'ı yatayda ters çevir (flipX)
        if (moveInput.x < -0.01f)
        {
            spriteRenderer.flipX = true;  // Sola bak
        }
        else if (moveInput.x > 0.01f)
        {
            spriteRenderer.flipX = false; // Sağa bak (Orijinal yön)
        }
    }

    private void FixedUpdate()
    {
        // 3. Çapraz Hızlanmayı Önleme (Pisagor Teoremi Düzeltmesi):
        // W (0,1) ve D (1,0) aynı anda basıldığında uzunluk 1.414 birim olur.
        // .normalized kullanarak uzunluğu daima 1 birime sabitleriz.
        Vector2 targetVelocity = moveInput.normalized * moveSpeed;

        // 4. Unity Sürüm Uyumluluğu:
        // Unity 6'da linearVelocity standardı gelmiştir; eski sürümlerde velocity kullanılır.
        #if UNITY_6000_0_OR_NEWER
        rb.linearVelocity = targetVelocity;
        #else
        rb.velocity = targetVelocity;
        #endif
    }
}
```

5. Kodu kaydedin (`Ctrl + S`) ve Unity editörüne geri dönün. Unity kodu derleyecektir (sağ altta küçük döner halka biter).

---

### Adım 2.7: Scripti Karaktere Bağlama ve Test Etme

1. Hierarchy panelinden **`Player`** nesnesini seçin.
2. Project panelindeki `Assets/Scripts/PlayerController` dosyasını tutup Inspector panelinin boş bir alanına sürükleyip bırakın (veya Add Component → Player Controller deyin).
3. Inspector'da scriptin alanlarını bağlayın:
   - **Move Speed:** `5` olarak kalsın (İstediğiniz zaman hızı buradan test ederek değiştirebilirsiniz).
   - **Move Action:** `Assets/Settings/PlayerInputActions` dosyasının solundaki küçük oka tıklayarak genişletin. Altında beliren **`Player/Move`** aksiyonunu tutup scriptin üzerindeki **Move Action** kutusuna sürükleyip bırakın.
4. **Büyük Test Anı:**
   - Üst ortadaki **Play (▶️)** butonuna basın!
   - Klavyeden `W`, `A`, `S`, `D` tuşlarına basarak karakterinizi yürütün:
     - Karakter 8 farklı yöne akıcı şekilde yürüyor mu?
     - `A` tuşuna basınca yüzünü sola, `D` tuşuna basınca sağa çeviriyor mu?
     - Odadaki taş duvarlara çarptığında pürüzsüzce duruyor ve duvarın içine gömülmüyor mu?
     - Ayak kutusu sayesinde duvarın önünden geçerken başı duvarın önünde kalarak derinlik hissi veriyor mu?

---

### Adım 2.8: Önemli Tasarım Kuralı: Neden `transform.Translate` Değil de `rb.linearVelocity`?

Öğrencilerin fizik programlarken sıkça düştüğü bir hatayı ve arkasındaki mühendislik nedenini inceleyelim:

- **Hatalı Yöntem (`transform.position += ...` veya `transform.Translate`):**  
  Bu yöntem karakteri "ışınlar" (teleport eder). Karakter duvara çarptığında fizik motoru onu dışarı itmeye çalışırken kod onu duvarın içine sokmaya çalışır. Sonuç: Karakter duvarda deli gibi titrer (jitter) veya hızlı basıldığında duvarın içinden geçip haritanın dışına fırlar!
- **Doğru Yöntem (`rb.linearVelocity` / `rb.velocity`):**  
  Unity'nin PhysX motoruna doğrudan hız emri veririz. Fizik motoru karakteri duvarla temas ettiği anda milimetrik olarak durdurur; ne titreme olur ne de duvardan sızma yaşanır. Tuşu bıraktığınız anda hız sıfırlandığı için zemin buz pisti gibi kaymaz, anında durur.

---

## 5. Öğrenci Kontrol Listesi (Self-Check Checklist)

Faz 2'yi eksiksiz tamamladığınızdan emin olmak için aşağıdaki maddeleri tek tek doğrulayın:

- [ ] Unity Package Manager üzerinden **Input System** paketi kuruldu ve editör yeniden başlatıldı.
- [ ] `Assets/Settings` altında **`PlayerInputActions`** dosyası oluşturuldu.
- [ ] Action Map olarak **`Player`**, Action olarak Vector 2 tipinde **`Move`** eklendi; `W, S, A, D` tuşları composite olarak bağlandı ve **Save Asset** butonuna basıldı.
- [ ] Sahnedeki oyuncu nesnesinin adı **`Player`** yapıldı ve **Tag**'i `Player` olarak ayarlandı.
- [ ] `Player` nesnesinin Sprite Renderer **Sorting Layer**'ı **`Characters`** olarak seçildi.
- [ ] `Player` nesnesine **`Rigidbody 2D`** eklendi:
  - [ ] **Gravity Scale = 0** yapıldı.
  - [ ] **Collision Detection = Continuous** yapıldı.
  - [ ] **Freeze Rotation Z** kutucuğu işaretlendi.
- [ ] `Player` nesnesine **`Box Collider 2D`** eklendi ve **Edit Collider** ile sadece ayak tabanını kaplayacak şekilde küçültüldü.
- [ ] `Assets/Scripts/PlayerController.cs` scripti oluşturuldu ve `Player` nesnesine eklendi.
- [ ] Inspector'da `PlayerController` üzerindeki **Move Action** alanına `Player/Move` aksiyonu sürüklendi.
- [ ] Oyunu başlattığınızda karakter WASD ile hareket ediyor, sağa/sola dönüyor ve duvarların içinden geçemiyor.

---

## 6. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Karakter duvara veya köşeye çarptığı anda kendi etrafında fırıldak gibi dönüyor!

> **Çözüm:** `Player` üzerindeki `Rigidbody 2D` bileşenini bulun. **Constraints** başlığını açın ve **Freeze Rotation > Z** kutucuğunu işaretleyin. Bu ayar karakterin 2D düzlemde dönmesini engeller.

### ❓ Soru 2: Play butonuna bastığım anda karakter aşağı doğru düşüp ekrandan kayboluyor!

> **Çözüm:** `Player` üzerindeki `Rigidbody 2D` bileşeninde bulunan **Gravity Scale** değerini kontrol edin. Varsayılan `1` kalmış olabilir; mutlaka **`0`** yapmalısınız. Top-down zindanlarda dikey yer çekimi olmamalıdır.

### ❓ Soru 3: Karakter düz (sadece W veya sadece D) giderken normal, ama çapraz (W+D) giderken belirgin şekilde daha hızlı koşuyor!

> **Çözüm:** Kodunuzda `moveInput` vektörünün `.normalized` edilip edilmediğini kontrol edin. Pisagor teoremine göre $(1^2 + 1^2 = \sqrt{2} \approx 1.414)$ yani çapraz hız %41 daha fazladır. `moveInput.normalized * moveSpeed` yazarak bu sorunu gideriyoruz.

### ❓ Soru 4: Klavyede WASD'ye basıyorum ama karakter hiç hareket etmiyor! Konsolda "NullReferenceException" hatası var.

> **Çözüm:** İki ihtimal vardır:
>
> 1. Hierarchy'de `Player` nesnesine tıklayın. `PlayerController` scriptindeki **Move Action** alanının boş (`None`) kalmadığından emin olun. `PlayerInputActions` dosyasını genişletip içindeki `Player/Move` aksiyonunu bu kutuya sürükleyin.
> 2. `PlayerInputActions` dosyasını açıp pencerenin üstündeki **Save Asset** butonuna bastığınızdan emin olun.

### ❓ Soru 5: Unity New Input System'i kurdum ama kodda "The type or namespace name 'InputSystem' does not exist" hatası alıyorum!

> **Çözüm:** Unity üst menüsünden **Edit > Project Settings > Player** sekmesine gidin. **Other Settings** başlığı altındaki **Active Input Handling** ayarını bulun. Eğer `Input Manager (Old)` seçiliyse, bunu **`Input System Package (New)`** (veya `Both`) yapın ve editörün yeniden başlamasını onaylayın.

### ❓ Soru 6: Karakter duvara çarptığında titriyor (jittering) veya takılıyor!

> **Çözüm:** Karakter hareket kodunun `FixedUpdate()` fonksiyonunda çalıştığından emin olun. Hareketi `Update()` içinde yaparsanız fizik motoru ile render motoru senkronize olamaz ve titreme yapar. Ayrıca hareketi `transform.position` ile değil `rb.linearVelocity` (veya `rb.velocity`) ile yaptığınızı doğrulayın.

### ❓ Soru 7: Karakter zemin taşlarının altında kalıyor veya duvarların arkasında kayboluyor!

> **Çözüm:** `Player` nesnesinin `Sprite Renderer` bileşeninde **Sorting Layer** ayarının `Characters` olduğundan emin olun. Eğer `Default` veya `Background` kalırsa zemin taşları karakterin üstüne çizilir.

---

🎉 **Tebrikler!** FAZ 2'yi başarıyla tamamladınız. Artık zindanınızda fizik kurallarına tam uyumlu, pikselleri tertemiz hareket eden bir şövalyeniz var!  
Bir sonraki fazda (**FAZ 3**) Cinemachine ile şövalyemizi yumuşakça takip eden dinamik bir kamera kuracak ve fare sol tıkıyla farenin baktığı yöne şimşek gibi savrulan anlık kılıç mekaniğini inşa edeceğiz!
