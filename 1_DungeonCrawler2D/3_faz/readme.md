# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 3

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Cinemachine ile Dinamik Kamera Takibi, Silah Menteşesi (Weapon Pivot) Mimarisi, Fare Yönelimli Anlık Kılıç Savurma (Melee Attack) ve Vuruş Hissi (Game Juice)

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Faz 2'de şövalyemizi sahneye indirdik, fizik kurallarına bağladık ve WASD tuşlarıyla zindanın odalarında 8 yöne pürüzsüzce yürütmeyi başardık. Ancak şu an iki büyük eksikliğimiz var:

1. Şövalyemiz odanın kenarına doğru yürüdüğünde ekranın dışına çıkıyor; çünkü kamera sabit duruyor ve onu takip etmiyor!
2. Zindanın karanlık dehlizlerindeki kutuları parçalamak veya iskeletlerle savaşmak için henüz bir silahımız yok.

Bu kılavuzu tamamladığınızda:

1. AAA stüdyolarının kullandığı profesyonel kamera sistemi olan **Cinemachine** paketini projenize kuracak ve karakteri yumuşak bir süzülmeyle (**Damping**) takip eden dinamik bir kamera inşa edeceksiniz.
2. Piksel sanatı zindan oyunlarında kamera ölçeğini (**Orthographic Lens Size**) doğru ayarlayarak görüntünün net ve pürüzsüz kalmasını sağlayacaksınız.
3. Hiyerarşide Ebeveyn-Çocuk (**Parent-Child**) mimarisini kullanarak bir **Silah Menteşesi (`WeaponPivot`)** kuracak; kılıcın karakterin gövdesini merkez alarak daire yayı çizmesini sağlayacaksınız.
4. Farenin ekrandaki piksel konumunu dünyadaki zindan koordinatına dönüştürme (**`ScreenToWorldPoint`**) ve trigonometrik açı hesaplama (**`Mathf.Atan2`**) mantığını kavrayacaksınız.
5. Kılıcın karakterin elinde sürekli açık ve kazık gibi durması yerine, sol tıklandığında farenin baktığı yöne saliseler içinde savrulup geri gizlenen **Gizli Silah / Anlık Savurma (`MeleeAttack.cs`)** mekaniğini C# Coroutine ile kodlayacaksınız.
6. Kılıca **Trigger Collider** vererek duvarlara çarpıp takılmasını önleyecek ve ardışık tıklamaları dengeleyen bir vuruş sıklığı (**Attack Cooldown**) mekanizması kuracaksınız.

---

## 1. Zihinsel Model: Kamera ve Silah Mimarisi

Koda ve motor adımlarına geçmeden önce, bir oyun geliştirici gibi bu sistemlerin mantığını kavrayalım:

### A) Neden Kamerayı Karakterin İçine Çocuk (Child) Yapmıyoruz?

Unity'ye yeni başlayanların en sık yaptığı hata: Hierarchy'de `Main Camera`yı tutup `Player` nesnesinin altına sürüklemektir.  
**Neden bu yöntem profesyonel oyunlarda ASLA kullanılmaz?**

- Karakter her adım attığında, duvara hafifçe sürttüğünde veya mikroskobik olarak titrediğinde kamera da zınk diye titrer; bu durum oyuncunun gözünü aşırı yorar ve mide bulantısına yol açar.
- Kameranın arkadan hafif gecikmeli, yumuşak bir süzülmeyle (Damping) gelmesini sağlayamazsınız.
- İlerleyen fazlarda vuruş anında ekran sarsıntısı (Screen Shake) veya zindanın dışını göstermeyen oda sınırları (Camera Confiner) ekleyemezsiniz.  
  **Çözüm:** Kamerayı bağımsız bırakıp onu akıllı bir takip robotu olan **Cinemachine** ile yönetmektir!

---

### B) Cinemachine Nedir ve Nasıl Çalışır?

Cinemachine, sahnedeki gerçek `Main Camera`yı bir kukla gibi yöneten akıllı bir sanal kamera yöneticisidir.

- Sahnedeki `Main Camera` fiziksel gözdür.
- Biz sahneye bir **Cinemachine Camera (Sanal Kamera)** ekleriz ve ona _"Şövalyeyi takip et"_ emri veririz.
- Sanal kamera, şövalye nereye giderse gerçek kamerayı oraya yumuşakça taşır.

---

### C) Silah Menteşesi (Weapon Pivot) İlkesi

Eğer kılıç görselini doğrudan karaktere bağlayıp kılıcı döndürürseniz, kılıç kendi kabzası veya ortası etrafında fırıldak gibi döner. Karakterin etrafında geniş bir yay çizemez.

```text
HATALI YÖNTEM:                   DOĞRU YÖNTEM (MENTEŞE):
(Kendi ekseninde döner)          (Karakterin etrafında yay çizer)

      [ Kılıç ]                        [ Şövalye ] ──(Menteşe: 0,0)──┐
         ↻                                     │                      │ 0.6 birim
   Kılıç kendi sapı                            └───> [ WeaponPivot ]  │
    etrafında döner!                                        ↻         ▼
                                                    Tüm menteşe döner [ Kılıç ]
                                                    kılıç yay çizer!      ) ) )
```

**Menteşe Mantığı:**

1. Karakterin tam göbeğine boş bir menteşe nesnesi koyarız: **`WeaponPivot`** (Konum: `0, 0, 0`).
2. Kılıcı bu menteşenin çocuğu yaparız ve merkezden 0.6 birim sağa öteleriz: **`Sword`** (Konum: `0.6, 0, 0`).
3. Kod ile menteşeyi döndürdüğümüzde, kılıç karakterin etrafında tıpkı bir pergel gibi kusursuz bir kılıç savurma yayı çizer!

---

### D) Ekran Koordinatı vs Dünya Koordinatı (`ScreenToWorldPoint`)

- **Monitör (Screen Point):** Bilgisayar ekranınız piksellerden oluşur (Örn: 1920x1080). Fareniz ekranın ortasındaysa konumu `(960, 540)`tır.
- **Oyun Dünyası (World Point):** Zindanın dünyası metrelerden oluşur. Şövalyenizin durduğu yer `(2.5, -1.2)` koordinatıdır.
- Eğer farenin piksel koordinatını doğrudan koda verirseniz kılıç bambaşka bir galaksiye bakar!  
  `Camera.main.ScreenToWorldPoint()` fonksiyonu, farenin ekrandaki piksel koordinatını zindan dünyasındaki gerçek metre koordinatına tercüme eden sihirli fonksiyondur.

---

## 2. Kullanılacak Görseller ve Katman Ayarları

Kenney Tiny Dungeon paketinde birden fazla silah görseli yer alır:

- **Kısa Kılıç (Hançer / Short Sword):** `tile_0104` (`tilemap_packed_104`)
- **Şövalye Uzun Kılıcı (Broadsword):** `tile_0106` (`tilemap_packed_106`)

Biz projemizde dengeli ve şık bir görünüm sunan **Kısa Kılıç (`tile_0104`)** görselini kullanacağız.

- **Sorting Layer:** `Characters`
- **Order in Layer:** `1`  
  _(Şövalyenin Order in Layer değeri `0`dı. Kılıca `1` vererek kılıcın karakterin gövdesinin hemen önünde, şık bir şekilde görünmesini sağlıyoruz)._

---

## 3. Cinemachine Paketi Kurulumu

Projenizde Cinemachine henüz yüklü değilse, ilk işimiz bu paketi motora dahil etmektir:

1. Unity üst menüsünden **Window > Package Manager** penceresini açın.
2. Sol üstteki filtreden **Packages: Unity Registry** seçin.
3. Sağ üstteki arama kutusuna `Cinemachine` yazın.
4. Listeden **Cinemachine** paketini seçip sağ alttaki **Install** butonuna tıklayın.
5. Kurulum birkaç saniye içinde tamamlanacaktır.

---

## 4. FAZ 3 Adım Adım Uygulama Rehberi

---

### Adım 3.1: Cinemachine ile Takip Eden Dinamik Kamerayı Kurma

Kamerayı şövalyemizi pürüzsüzce takip edecek şekilde yapılandırıyoruz:

1. **Sanal Kamerayı Sahneye Ekleme:**
   - **Unity 6 Kullanıyorsanız:** Hierarchy'de boş bir yere sağ tıklayın → **Cinemachine > Cinemachine Camera** seçin.
   - **Unity 2022 / Eski LTS Kullanıyorsanız:** Hierarchy'de boş bir yere sağ tıklayın → **Cinemachine > 2D Camera** (veya _Virtual Camera_) seçin.
   - Sahneye `CinemachineCamera` (veya `CM vcam1`) adında bir nesne gelecektir.

2. **Karakteri Takip Hedefi Yapma:**
   - Oluşan sanal kamera nesnesine tıklayın, sağdaki **Inspector** panelini açın.
   - **Tracking Target (Unity 2022'de Follow):** Hierarchy panelindeki **`Player`** nesnesini tutun ve bu kutunun içine sürükleyip bırakın!

3. **Piksel Zindan Görünümü (Lens Ayarı):**
   - **Lens > Orthographic Size:** `4` veya `4.5` yapın.  
     _(Bu değer kameranın yakınlaşma derecesidir. Çok büyük olursa zindan minicik kalır, çok küçük olursa karakter ekranı kaplar. 4 değeri 16x16 piksel karolar için mükemmel retro zindan hissiyatı verir)._

4. **Yumuşak Takip (Damping):**
   - **Body (veya Position Control) > Damping:**
     - **X:** `0.5`
     - **Y:** `0.5`
     - **Z:** `0`  
       _(Damping değeri, kameranın oyuncunun arkasından hafif yaylanarak gelmesini sağlar. 0 yaparsanız zınk diye yapışır, 0.5 yaparsanız yağ gibi kayarak takip eder)._

---

### Adım 3.2: Silah Menteşesi (WeaponPivot) ve Kılıç Nesnesini Kurma

Şimdi 1. Bölümde anlattığımız pergel/menteşe sistemini kuruyoruz:

1. **Menteşeyi Oluşturma:**
   - Hierarchy panelinde **`Player`** nesnesine sağ tıklayın → **Create Empty** deyin.
   - Oluşan boş nesneye tıklayın, `F2`ye basıp adını tam olarak **`WeaponPivot`** yapın.
   - Inspector'da **Transform Position** değerlerinin `X: 0, Y: 0, Z: 0` olduğunu doğrulayın (Karakterin tam merkezinde olmalıdır).

2. **Kılıç Sprite'ını Ekleme:**
   - Hierarchy'de yeni oluşturduğunuz **`WeaponPivot`** nesnesine sağ tıklayın → **2D Object > Sprites > Square** (Kare) seçin. _(Unity alt menüsünde Square, Circle gibi seçenekler çıkar; `Square` seçmeniz yeterlidir, Inspector'da kılıç görselimizi bağlayınca nesne kılıç şeklini alacaktır)._
   - Adını **`Sword`** yapın.

3. **Kılıcın Inspector Ayarları:**
   - `Sword` nesnesi seçiliyken sağdaki Inspector'a bakın:
   - **Sprite Renderer > Sprite:** Küçük yuvarlak butona basın ve `tilemap_packed_104` (kısa kılıç) görselini seçin.
   - **Sorting Layer:** **`Characters`** yapın.
   - **Order in Layer:** **`1`** yapın (Karakterin gövdesinin önünde çizilsin).
   - **Transform Position (HAYATİ ÖTELEME AYARI):**
     - **X:** `0.6`
     - **Y:** `0`
     - **Z:** `0`  
       _(Kılıcı merkezden 0.6 birim sağa öteledik. Böylece WeaponPivot döndüğünde kılıç karakterin etrafında yay çizecektir!)._

---

### Adım 3.3: Kılıca Çarpışma Kutusu (Box Collider 2D - Is Trigger) Ekleme

Kılıcımızın ilerleyen fazlarda ahşap kasaları kırması ve iskeletlere hasar vermesi için bir temas alanına ihtiyacı vardır:

1. Hierarchy'den **`Sword`** nesnesini seçin.
2. Inspector'ın en altındaki **Add Component** butonuna tıklayın → **`Box Collider 2D`** ekleyin.
3. **Çok Önemli 2 Ayar:**
   - **`Is Trigger`:** Mutlaka **İŞARETLEYİN (True)!**  
     _(DİKKAT: Eğer Is Trigger işaretlemezseniz kılıç katı bir tuğla gibi davranır; savurduğunuzda duvara çarpıp şövalyenizi geriye fırlatır veya takılır. Is Trigger ile kılıç engellerin ve düşmanların içinden keserek geçer)._
   - **Bileşeni Devre Dışı Bırakma:** `Box Collider 2D` başlığının hemen solunda bulunan küçük **onay kutucuğunu (checkbox) kaldırın (Disabled)**.  
     _(Neden? Karakter elinde kılıç yokken yürürken sandıklara veya canavarlara yanlışlıkla sürtünmesin; kılıç sadece sol tıka basıldığında kod tarafından anlık olarak açılsın!)._

---

### Adım 3.4: PlayerInputActions'a "Attack" (Sol Tık) Eylemini Ekleme

Karakterimize sol tıkla saldırma yeteneği tanımlıyoruz:

1. Project panelinde `Assets/Settings` klasöründeki **`PlayerInputActions`** dosyasına çift tıklayın.
2. Sol kolondaki **Action Maps** listesinden **`Player`** haritasını seçin.
3. Orta kolondaki **Actions** başlığının yanındaki `+` butonuna tıklayın:
   - Adını tam olarak **`Attack`** yapın.
4. `Attack` satırına tıklayın, sağdaki **Action Properties** panelinde:
   - **Action Type:** `Button` olarak seçin.
5. `Attack` satırının altındaki `<No Binding>` satırına tıklayın:
   - Sağdaki **Binding Properties** panelinde **Path:** kutusuna tıklayın.
   - **Listen** butonuna basın ve farenizin **Sol Tıkına** basın.
   - Çıkan listeden **`Left Button [Mouse]`** seçeneğini seçin.
6. Pencerenin üstündeki **Save Asset** butonuna tıklayın ve pencereyi kapatın.

---

### Adım 3.5: Kılıç Savurma Kodunu Yazma (MeleeAttack.cs)

Şimdi kılıcın farenin baktığı yöne şimşek gibi savrulup geri gizlenmesini sağlayan C# kodunu yazıyoruz:

1. Project panelinde `Assets/Scripts` klasörüne girin.
2. Sağ tıklayın → **Create > C# Script** deyin. Adını tam olarak **`MeleeAttack`** yapın.
3. Dosyayı kod editörünüzde açın ve içerisindekileri silip şu kodu yapıştırın:

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.InputSystem;


public class MeleeAttack : MonoBehaviour
{
    [Header("Girdi ve Referanslar")]
    [Tooltip("PlayerInputActions içindeki Player/Attack aksiyonu")]
    [SerializeField] private InputActionReference attackAction;

    [Tooltip("Karakterin merkezindeki silah döndürme menteşesi")]
    [SerializeField] private Transform weaponPivot;

    [Tooltip("Kılıcın görsel çizicisi (SpriteRenderer)")]
    [SerializeField] private SpriteRenderer swordRenderer;

    [Tooltip("Kılıcın temas ve hasar algılama kutusu (Is Trigger)")]
    [SerializeField] private Collider2D attackCollider;

    [Header("Saldırı Ayarları")]
    [Tooltip("Kılıcın savrulma süresi (saniye cinsinden - düşük değer şimşek gibi vurur)")]
    [SerializeField] private float attackDuration = 0.12f;

    [Tooltip("Kılıcın savrulurken tarayacağı açı genişliği (derece)")]
    [SerializeField] private float attackArc = 80f;

    [Tooltip("İki saldırı arasında geçmesi gereken minimum bekleme süresi (cooldown)")]
    [SerializeField] private float attackCooldown = 0.25f;

    private Camera mainCamera;
    private bool isAttacking = false;
    private float nextAttackTime = 0f;

    private void Awake()
    {
        mainCamera = Camera.main;

        // Oyun başlarken kılıcı ve temas kutusunu tamamen gizle
        if (swordRenderer != null) swordRenderer.enabled = false;
        if (attackCollider != null) attackCollider.enabled = false;
    }

    private void OnEnable()
    {
        if (attackAction != null) attackAction.action.Enable();
    }

    private void OnDisable()
    {
        if (attackAction != null) attackAction.action.Disable();
    }

    private void Update()
    {
        // Sol tık basıldı mı ve bekleme süresi doldu mu?
        if (attackAction != null && attackAction.action.WasPressedThisFrame() && Time.time >= nextAttackTime)
        {
            if (!isAttacking)
            {
                StartCoroutine(PerformSlashRoutine());
            }
        }
    }

    /// <summary>
    /// Kılıcı saliseler içinde farenin baktığı yöne savuran Coroutine
    /// </summary>
    private IEnumerator PerformSlashRoutine()
    {
        isAttacking = true;
        nextAttackTime = Time.time + attackCooldown;

        // 1. Farenin ekrandaki piksel konumunu oku ve zindan dünyası metre koordinatına çevir
        Vector2 mouseScreen = Mouse.current.position.ReadValue();
        Vector3 mouseWorld = mainCamera.ScreenToWorldPoint(mouseScreen);
        mouseWorld.z = 0f;

        // 2. Karakterden fareye doğru uzanan yön vektörünü ve taban açısını hesapla
        Vector3 direction = (mouseWorld - weaponPivot.position).normalized;
        float baseAngle = Mathf.Atan2(direction.y, direction.x) * Mathf.Rad2Deg;

        // 3. Kılıcı görünür yap ve temas kutusunu aç
        swordRenderer.enabled = true;
        attackCollider.enabled = true;

        // Kritik Düzeltme: Fare sol taraftayken kılıcın ters (baş aşağı) durmaması için dikey çevir (flipY)
        swordRenderer.flipY = (baseAngle > 90f || baseAngle < -90f);

        // 4. Savurma Açısı: Belirlenen yay genişliğine göre başlangıç ve bitiş açılarını hesapla
        Quaternion startRot = Quaternion.Euler(0, 0, baseAngle + (attackArc / 2));
        Quaternion endRot = Quaternion.Euler(0, 0, baseAngle - (attackArc / 2));

        float elapsed = 0f;
        while (elapsed < attackDuration)
        {
            elapsed += Time.deltaTime;
            float t = elapsed / attackDuration;

            // Küresel enterpolasyon (Slerp) ile menteşeyi başlangıç açısından bitiş açısına pürüzsüzce döndür
            weaponPivot.rotation = Quaternion.Slerp(startRot, endRot, t);
            yield return null;
        }

        // 5. Vuruş bitti: Kılıcı ve temas kutusunu tekrar tamamen gizle
        swordRenderer.enabled = false;
        attackCollider.enabled = false;
        isAttacking = false;
    }
}
```

4. Kodu kaydedin (`Ctrl + S`) ve Unity'ye dönün.

---

### Adım 3.6: Scripti Karaktere Bağlama ve Test Etme

1. Hierarchy panelinden **`Player`** nesnesini seçin.
2. Inspector'ın en altındaki **Add Component** butonuna tıklayın → **`Melee Attack`** yazıp ekleyin (veya scripti sürükleyip bırakın).
3. Scriptin üzerindeki 4 boş referans kutusunu şu şekilde bağlayın:
   - **Attack Action:** `Assets/Settings/PlayerInputActions` dosyasını genişletin → içindeki **`Player/Attack`** aksiyonunu sürükleyin.
   - **Weapon Pivot:** Hierarchy'de `Player`ın altındaki **`WeaponPivot`** nesnesini sürükleyin.
   - **Sword Renderer:** Hierarchy'de `WeaponPivot`ın altındaki **`Sword`** nesnesini sürükleyin.
   - **Attack Collider:** Yine **`Sword`** nesnesini sürükleyin _(Üzerindeki Box Collider 2D'yi otomatik alacaktır)_.

4. **Büyük Test Anı:**
   - **Play (▶️)** butonuna basın!
   - Karakteriniz boş elleriyle zindanda yürümeli; kılıç ekranda görünmemelidir.
   - Karakteri odanın kenarlarına doğru yürütün: Kameranın şövalyeyi yumuşakça takip ettiğini gözlemleyin.
   - Farenizi karakterin etrafında gezdirip **Sol Tık** yapın:
     - Kılıç farenin baktığı yöne doğru anlık bir şimşek gibi savrulup (ark çizip) hemen kayboluyor mu?
     - Fareyi karakterin soluna alıp tıkladığınızda kılıç ters dönmeden düzgünce savruluyor mu?
     - Duvarın dibindeyken kılıç savurduğunuzda karakter duvara takılmadan rahatça vurabiliyor mu?

---

### Adım 3.7: Önemli Tasarım Kuralı: Neden Sürekli Açık Kılıç Değil de Anlık Savurma (Slash)?

Öğrenciler bazen şunu sorar:  
_`"Hocam kılıcı neden gizliyoruz? Karakterin elinde sürekli açık dursa, biz yürüdükçe farenin baktığı yöne dönse daha kolay olmaz mıydı?"`_

**Cevap: Oynanış Hissi (Game Juice) ve Oyun Dengesi Açısından KESİNLİKLE HAYIR.**

1. **Oyun Dengesini Bozma (Exploit):** Eğer kılıç sürekli açık ve katı olsaydı, oyuncunun tek yapması gereken farenin ucunu düşmana doğrultup üzerine doğru yürümek olurdu. Tıklama, zamanlama veya refleks ihtiyacı kalmazdı.
2. **Game Feel (Vuruş Tatmini):** Kılıcın sadece tıkladığınızda saliseler içinde şimşek hızıyla savrulması (`0.12` saniye), oyuncuya bir "kesme" ve "aksiyon" hissi verir.
3. **Görsel Temizlik:** Şövalyenin elinde sürekli 360 derece dönen bir kılıç yerine, gerektiğinde kınından çekilen bir kılıç profesyonel oyun standartıdır.

---

## 5. Öğrenci Kontrol Listesi (Self-Check Checklist)

Faz 3'ü eksiksiz tamamladığınızdan emin olmak için aşağıdaki maddeleri tek tek doğrulayın:

- [ ] Unity Package Manager üzerinden **Cinemachine** paketi kuruldu.
- [ ] Sahneye sanal kamera eklendi, **Tracking Target (veya Follow)** alanına `Player` bağlandı.
- [ ] Kameranın **Orthographic Size** değeri `4` veya `4.5` olarak ayarlandı ve **Damping** `0.5` yapıldı.
- [ ] `Player` nesnesinin altında `WeaponPivot` (Pozisyon: `0, 0, 0`) oluşturuldu.
- [ ] `WeaponPivot` altında `Sword` nesnesi oluşturuldu ve pozisyonu `X: 0.6, Y: 0, Z: 0` olarak ötelendi.
- [ ] `Sword` nesnesinin görseli kısa kılıç (`tile_0104`), Sorting Layer'ı **`Characters`**, Order in Layer'ı **`1`** yapıldı.
- [ ] `Sword` nesnesine **`Box Collider 2D`** eklendi; **`Is Trigger` işaretlendi** ve bileşenin onay kutusu kapatıldı (Disabled).
- [ ] `PlayerInputActions` içine `Attack` buton aksiyonu eklendi, Fare Sol Tıkı bağlandı ve **Save Asset** dendi.
- [ ] `Assets/Scripts/MeleeAttack.cs` scripti oluşturuldu ve `Player` nesnesine eklendi.
- [ ] Inspector'da `MeleeAttack` scriptinin 4 referansı (`Attack Action`, `Weapon Pivot`, `Sword Renderer`, `Attack Collider`) eksiksiz bağlandı.
- [ ] Oyunu başlattığınızda kamera karakteri yumuşakça takip ediyor; sol tık yapıldığında kılıç farenin olduğu yöne savrulup anında gizleniyor.

---

## 6. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Unity 6 kullanıyorum, Hierarchy'de sağ tıklayınca "Cinemachine > Virtual Camera" seçeneği çıkmıyor!

> **Çözüm:** Unity 6 ile birlikte Cinemachine 3.x sürümüne geçilmiştir ve bileşen isimleri sadeleştirilmiştir. Hierarchy'de sağ tıklayıp **Cinemachine > Cinemachine Camera** seçeneğini seçin. Inspector'da `Follow` yerine **`Tracking Target`** kutusunu göreceksiniz; `Player` nesnesini buraya sürükleyin.

### ❓ Soru 2: Sol tıkladığımda kılıç karakterin etrafında yay çizmek yerine kendi sapının etrafında dönüyor!

> **Çözüm:** `Sword` nesnesinin Transform Position değerini kontrol edin. Eğer `X: 0, Y: 0, Z: 0` kaldıysa menteşe çalışmaz. `Sword` nesnesinin pozisyonunun mutlaka **`X: 0.6, Y: 0, Z: 0`** olduğundan ve ebeveyni olan `WeaponPivot`ın pozisyonunun **`0, 0, 0`** olduğundan emin olun.

### ❓ Soru 3: Karakterin sol tarafına doğru tıkladığımda kılıç baş aşağı (ters) görünüyor!

> **Çözüm:** `MeleeAttack.cs` kodunuzdaki şu satırı kontrol edin:  
> `swordRenderer.flipY = (baseAngle > 90f || baseAngle < -90f);`  
> Bu satır fare sol yarı küreye geçtiğinde kılıcı dikeyde ters çevirerek görselin doğru açıda kalmasını sağlar.

### ❓ Soru 4: Sol tıka basıyorum ama kılıç hiç görünmüyor! Konsolda "NullReferenceException" hatası var.

> **Çözüm:** `Player` üzerindeki `MeleeAttack` bileşenini kontrol edin. 4 referans kutusundan biri boş (`None`) kalmış olabilir. Özellikle `Weapon Pivot`, `Sword Renderer` ve `Attack Collider` alanlarına Hierarchy'deki ilgili nesneleri sürüklediğinizden emin olun.

### ❓ Soru 5: Kılıcı savurduğumda karakter duvara çarpıp geriye sekiyor veya yürüyemiyor!

> **Çözüm:** `Sword` nesnesi üzerindeki `Box Collider 2D` bileşenini bulun. **`Is Trigger`** kutucuğunun mutlaka **işaretli (True)** olduğundan emin olun. Eğer işaretli değilse kılıç katı bir engele dönüşür ve duvara çarparak karakteri fırlatır.

### ❓ Soru 6: Kamera karakteri takip ediyor ama görüntü aşırı bulanık veya pikseller titreşiyor!

> **Çözüm:** Faz 1'deki `tilemap_packed` görselinizin **Filter Mode** ayarının `Point (no filter)` olduğundan emin olun. Ayrıca sanal kameranızın **Orthographic Size** değerini tam bir sayı veya buçuklu sayı yapın (`4` veya `4.5`).

### ❓ Soru 7: Kılıç savrulduktan sonra ekranda açık kalıyor, bir daha kaybolmuyor!

> **Çözüm:** `MeleeAttack.cs` dosyasındaki `PerformSlashRoutine()` Coroutine'inin en sonundaki satırları kontrol edin:  
> `swordRenderer.enabled = false;` ve `attackCollider.enabled = false;` satırlarının while döngüsünün DIŞINDA, en sonda yer aldığından emin olun.

---

🎉 **Tebrikler!** FAZ 3'ü başarıyla tamamladınız. Artık zindanınızda şövalyenizi sinematik bir zarafetle takip eden dinamik bir kameranız ve farenizin baktığı yöne şimşek gibi savrulan keskin bir kılıcınız var!

Bir sonraki fazda (**FAZ 4**) kılıcımızla vurup parçalayabileceğimiz ahşap kasaları (`BreakableCrate`), etrafa saçılan talaş parçacık efektlerini (`WoodBreakParticles`), fırlayan altınları toplayabileceğimiz envanter sistemini (`PlayerInventory` & `CoinPickup`) ve gizemli hazine sandıklarını (`Chest` & Mimic sürprizi) inşa edeceğiz!
