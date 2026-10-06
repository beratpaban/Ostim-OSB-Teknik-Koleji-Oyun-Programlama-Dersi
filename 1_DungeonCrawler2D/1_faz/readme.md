# 🗡️ 2D Zindan Oyunu (Dungeon Crawler) - FAZ 1

**Ders:** Oyun Programlama (OSTİM OSB Teknik Koleji)  
**Konu:** Proje Kurulumu, Kenney 2D Asset Entegrasyonu, Piksel Sanatı Ayarları, Tilemap Mimarisi ve Katı Duvarlar  

---

## 🎯 Bu Fazın Amacı ve Kazanımları

Bu faz, dönem boyunca geliştireceğimiz 2D Zindan Oyunu (Dungeon Crawler) projemizin ilk ve en temel yapı taşıdır. Bu kılavuzu tamamladığınızda:

1. Dünyaca ünlü açık kaynak oyun sanatı sitesi **Kenney**'den **Tiny Dungeon** paketini indirip projenize dahil etmeyi öğreneceksiniz.
2. Piksel sanatı (Pixel Art) oyunlarında görsellerin bulanıklaşmasını önleyen, piksellerin cam gibi keskin kalmasını sağlayan motor ayarlarını (**PPU**, **Point Filter**, **Compression**) kavrayacaksınız.
3. 2D oyunlarda neyin neyin önünde çizileceğini belirleyen **Sorting Layers (Sıralama Katmanları)** mantığını öğreneceksiniz.
4. Unity'nin **Tilemap** sistemiyle performanslı bir zemin ve duvar altyapısı kuracaksınız.
5. Duvarlara **Tilemap Collider 2D** ve **Composite Collider 2D** ekleyerek karakterlerin içinden geçemeyeceği pürüzsüz fiziksel sınırlar çizeceksiniz.

---

## 1. Unity Temel Arayüzü ve Zihinsel Model

Unity motoruna ilk kez adım atıyorsanız, arayüzün mantığını anlamak işinizi çok kolaylaştıracaktır. Unity'de her şey bir **Oyun Nesnesi (GameObject)** ve bu nesnelere takılan **Bileşenlerden (Component)** ibarettir:

* **Hierarchy (Hiyerarşi - Sol Panel):** Sahnenizdeki tüm varlıkların (kamera, ışık, zemin, duvarlar, karakter) isim listesidir. Sahne bir tiyatro sahnesiyse, Hierarchy sahneye çıkan oyuncuların ve dekorların listesidir.
* **Scene (Sahne - Orta Panel):** Oyun dünyasını bir mimar gibi inşa ettiğiniz çalışma alanınızdır. Nesneleri taşır, döndürür ve yerleştirirsiniz.
* **Game (Oyun - Orta Üst Sekme):** Sahnedeki kameranın gördüğü, yani oyuncunun monitöründe göreceği nihai görüntüyü gösteren ekrandır.
* **Project (Proje - Sol Alt Panel):** Bilgisayarınızdaki proje klasörünün aynasıdır. Görseller, sesler, animasyonlar ve C# kodları burada durur. *Unutmayın: Buradaki bir dosyayı Hierarchy veya Scene ekranına taşımadığınız sürece oyunda var olmaz!*
* **Inspector (Denetçi - Sağ Panel):** Hierarchy'de veya Project'te seçtiğiniz herhangi bir nesnenin özelliklerini (konumu, boyutu, rengi, çarpışma kutusu, hızı vb.) inceleyip değiştirdiğiniz ayar panelidir.

---

## 2. Kullanacağımız Grafik Paketi: Kenney - Tiny Dungeon

Projemiz boyunca tüm grafiksel ihtiyaçlarımız için bağımsız oyun geliştiricileri arasında bir standart olan **Kenney** varlıklarını kullanacağız.

### Neden Kenney Tiny Dungeon?
* **Lisans:** CC0 1.0 Universal (Public Domain). Tamamen ücretsizdir, ticari ve eğitim amaçlı serbestçe kullanılabilir.
* **Standart:** 16x16 piksel karolardan (tiles) oluşur. Karakterler, silahlar, zindan duvarları, zemin taşları, kapılar ve canavarlar aynı görsel dille çizilmiştir.

### Asset'i İndirme ve Dosyayı Bulma (Adım Adım):
1. Tarayıcınızdan şu adrese gidin: **[Kenney - Tiny Dungeon](https://kenney.nl/assets/tiny-dungeon)**
2. Sayfadaki büyük **"Download"** butonuna tıklayın *(Site bağış ekranı çıkarırsa "No thanks, just take me to the downloads" seçeneğine tıklayabilirsiniz).*
3. Bilgisayarınıza inen `kenney_tiny-dungeon.zip` arşiv dosyasına sağ tıklayın ve **Klasöre Ayıkla (Extract All)** deyin.
4. Çıkan klasörün içerisine göz attığınızda:
   * `Tiles/` klasöründe yüzlerce tek tek ayrılmış 16x16 PNG resmi bulunur.
   * `Tilemap/` klasöründe ise tüm bu karoların tek bir görselde birleştirildiği **`tilemap_packed.png`** dosyası yer alır.
5. **Bizim kullanacağımız dosya:** `Tilemap/tilemap_packed.png`  
   *(Neden tek dosya? Tüm karoların tek bir görselde toplanması -Spritesheet/Texture Atlas-, hem ekran kartının çizim performansını katbekat artırır hem de Unity Tile Palette içine tek hamlede aktarılmasını sağlar).*

---

## 3. Unity Proje Hazırlığı ve Klasör Düzeni

Temiz bir klasör düzeni, profesyonel oyun geliştirmenin ilk kuralıdır.

1. **Unity Hub'ı Açın:** Yeni bir proje oluştururken şablon olarak mutlaka **2D (veya 2D Core)** seçin. Proje adını örneğin `DungeonCrawler2D` koyun.
2. Unity açıldığında **Project** panelinde `Assets` klasörünün altına sağ tıklayıp **Create > Folder** diyerek şu klasörleri oluşturun:
   * `Assets/Art` *(Görseller ve spritesheet'ler için)*
   * `Assets/Palettes` *(Zindanı çizeceğimiz fırça paletleri için)*
   * `Assets/Scenes` *(Oyun sahneleri için - varsayılan olarak gelir)*
   * `Assets/Scripts` *(İlerleyen fazlarda yazacağımız C# kodları için)*

> [!TIP]
> **Paket Kontrolü:** Unity üst menüsünden **Window > Package Manager** penceresini açın. Sol üstteki filtreden **Packages: In Project** seçin. Listede **2D Sprite** ve **2D Tilemap Editor** paketlerinin yüklü olduğunu doğrulayın *(2D Core şablonuyla açtıysanız otomatik yüklü gelir).*

---

## 4. FAZ 1 Adım Adım Uygulama Rehberi

---

### Adım 1.1: Piksel Görselini Unity'ye Tanıtma (Bulanıklığı Yok Etme)

Bilgisayarınızda ayıkladığınız Kenney klasöründeki `Tilemap/tilemap_packed.png` dosyasını sürükleyip Unity'deki `Assets/Art` klasörünün içine bırakın.

Görsele tıkladığınızda sağdaki **Inspector** panelinde piksel sanatına özel şu kritik ayarları yapmanız gerekir:

1. **Texture Type:** `Sprite (2D and UI)`  
   *(Unity'ye bu görselin 3D kaplama değil, 2D oyun görseli olduğunu söyler).*
2. **Sprite Mode:** `Multiple`  
   *(Varsayılan `Single`dır. Bu görsel tek bir karakter değil, içinde birden fazla karo olan bir tablo olduğu için `Multiple` seçiyoruz).*
3. **Pixels Per Unit (PPU):** `16`  
   *(EN KRİTİK AYAR! Unity dünyasında 1 birim mesafe = 1 metredir. Bizim karolarımız 16x16 piksel olduğu için buraya 16 yazıyoruz. Böylece 1 karo, sahnede tam olarak 1 grid karesine oturur. 100 kalırsa karolarınız küçücük kalır ve boşluklar oluşur).*
4. **Filter Mode:** `Point (no filter)`  
   *(Varsayılan `Bilinear` dır ve 3D dokuları yumuşatmak için kenarları bulanıklaştırır. Piksel sanatında ise keskin pikseller istediğimiz için filtreyi kapatıp `Point` yapıyoruz).*
5. **Compression:** Sayfanın en altındaki bu ayarı `None` yapın.  
   *(Unity varsayılan olarak boyuttan tasarruf etmek için renkleri sıkıştırır; bu da piksel sanatında renk bozulmalarına -artifact- yol açar. `None` diyerek orijinal renkleri koruyoruz).*
6. En alttaki **Apply** butonuna tıklayın.

---

### Adım 1.2: Görseli 16x16 Karolar Halinde Dilimleme (Slice)

Şu an Unity görselin içinde çok sayıda parça olduğunu biliyor (`Multiple`), ancak parçaların sınırlarını henüz bilmiyor. Şimdi görseli 16x16 ızgara çizgileriyle böleceğiz:

1. `tilemap_packed` görseli seçiliyken Inspector panelindeki **Sprite Editor** butonuna tıklayın.
2. Açılan pencerenin sol üstündeki **Slice** açılır menüsüne tıklayın:
   * **Type:** `Grid By Cell Size` *(Karoların piksel boyutuna göre böl).*
   * **Pixel Size:** `X: 16`, `Y: 16`
   * **Pivot:** `Center`
3. Kırmızı **Slice** butonuna basın.
   * *Görselin üzerindeki tüm karoların etrafında ince beyaz sınır çizgileri oluştuğunu göreceksiniz.*
4. Pencerenin sağ üst köşesindeki yeşil **Apply** butonuna tıklayın ve Sprite Editor penceresini kapatın.
5. Project panelinde `tilemap_packed` dosyasının yanındaki küçük oka tıklarsanız, artık yüzlerce bağımsız karonun (`tilemap_packed_0`, `tilemap_packed_1`...) oluştuğunu görebilirsiniz.

---

### Adım 1.3: Çizim Katmanlarını (Sorting Layers) Belirleme

2D oyunlarda Z derinliği olmadığı için ekran kartı nesneleri belirli bir sıraya göre üst üste çizer. Neyin en arkada, neyin en önde duracağını **Sorting Layers** ile yönetiriz:

1. Unity üst menüsünden **Edit > Project Settings** penceresini açın.
2. Sol menüden **Tags and Layers** sekmesine tıklayın.
3. **Sorting Layers** başlığını genişletin.
4. Varsayılan olarak yalnızca `Default` katmanı vardır. `+` butonuna basarak **yukarıdan aşağıya tam olarak şu sırayla** yeni katmanlar ekleyin:
   * `Background` *(En arkada zemin taşları, halılar)*
   * `Decorations` *(Zemindeki çatlaklar, kemikler, kan izleri)*
   * `Walls` *(Duvarlar, sütunlar)*
   * `Interactables` *(Sandıklar, variller, kapılar, meşaleler)*
   * `Characters` *(Karakterimiz, düşmanlar, NPC'ler)*
   * `UI` *(En önde can barları, envanter, yazılar)*

> [!IMPORTANT]
> **Katman Sırası Kuralı:** Listenin en üstündeki katman en arkada çizilir, listenin en altındaki katman ise en önde çizilir. Karakterinizin duvarların veya zeminin arkasında kaybolmaması için bu sıra hayati önem taşır!

5. Ayarlar penceresini kapatın.

---

### Adım 1.4: Sahneye Zemin ve Duvar Tilemap'lerini Ekleme

Şimdi sahneye çizim yapabileceğimiz iki ayrı tuval (Tilemap) ekleyeceğiz: biri zemin için, diğeri ise katı duvarlar için.

1. **Zemin Katmanı (Tilemap_Floor):**
   * Hierarchy panelinde boş bir yere sağ tıklayın → **2D Object > Tilemap > Rectangular** seçin.
   * Sahneye `Grid` adında bir ana nesne ve onun altında `Tilemap` adında bir alt nesne gelir.
   * Bu alt nesneye tıklayın, klavyeden `F2`ye basıp adını **`Tilemap_Floor`** yapın.
   * `Tilemap_Floor` seçiliyken sağdaki Inspector'da **Tilemap Renderer** bileşenini bulun:
     * **Sorting Layer:** `Background` olarak ayarlayın.

2. **Duvar Katmanı (Tilemap_Walls):**
   * `Grid` nesnesine sağ tıklayın → **2D Object > Tilemap > Rectangular** deyin.
   * Oluşan yeni alt nesnenin adını **`Tilemap_Walls`** yapın.
   * `Tilemap_Walls` seçiliyken sağdaki Inspector'da **Tilemap Renderer** bileşenini bulun:
     * **Sorting Layer:** `Walls` olarak ayarlayın.

*(Neden zemin ve duvarları aynı Tilemap'e çizmiyoruz? Çünkü sadece duvarların içinden geçilemez olmasını istiyoruz. İleride duvarlara fizik çarpışması eklediğimizde zemin de katılaşmasın diye iki ayrı Tilemap kullanıyoruz).*

---

### Adım 1.5: Duvarlara Geçilmezlik (Fiziksel Çarpışma) Ekleme

Oyuncumuzun duvarların içinden geçip haritanın dışına çıkmasını engellemek için duvar nesnesine fizik çarpışması ekleyeceğiz:

1. Hierarchy panelinden **`Tilemap_Walls`** nesnesini seçin.
2. Inspector'ın en altındaki **Add Component** butonuna tıklayın → Arama kutusuna yazıp **`Tilemap Collider 2D`** bileşenini ekleyin.
   * *Bu bileşen, duvar olarak boyadığınız her 16x16 karonun etrafına ayrı ayrı yeşil çarpışma kutusu koyar.*
3. Şimdi tekrar **Add Component** butonuna tıklayın → **`Composite Collider 2D`** bileşenini ekleyin.
   * *Unity bu bileşeni eklediğinizde otomatik olarak bir `Rigidbody 2D` bileşeni de ekleyecektir.*

4. **Hayati Önem Taşıyan 2 Ayar (Burayı dikkatle yapın!):**
   * **Ayar 1:** `Tilemap Collider 2D` bileşeni üzerinde bulunan **`Used By Composite`** kutucuğunu işaretleyin!  
     *(Bu ayar, yüzlerce bağımsız karo kutucuğunu tek bir pürüzsüz dış çizgiye bağlar. Karakterinizin karolar arasındaki görünmez dikiş izlerine takılmasını engeller ve performansı artırır).*
   * **Ayar 2:** Otomatik eklenen `Rigidbody 2D` bileşeninde **Body Type** ayarını `Dynamic` yerine mutlaka **`Static`** yapın!  
     *(DİKKAT: `Dynamic` kalırsa Unity'nin yer çekimi devreye girer ve oyunu başlattığınız anda tüm zindan duvarları aşağı doğru düşüp yok olur! `Static` yaparak duvarları dünyaya çivilemiş oluyoruz).*

---

### Adım 1.6: Tile Palette Oluşturma ve Zindan Odanızı Çizme

Artık fırçamızı elimize alıp zindanı çizmeye hazırız:

1. **Palet Penceresini Açma:**
   * Unity üst menüsünden **Window > 2D > Tile Palette** penceresini açın.
   * Bu pencereyi sahne alanının yanına veya altına sürükleyip sabitleyebilirsiniz (dock).
2. **Yeni Palet Oluşturma:**
   * Penceredeki açılır menüden **Create New Palette** seçeneğine tıklayın.
   * **Name:** `DungeonPalette`
   * **Grid:** `Rectangle`
   * **Cell Size:** `Automatic`
   * **Create** butonuna basın. Unity bu paleti nereye kaydedeceğini soracaktır: Daha önce oluşturduğumuz **`Assets/Palettes`** klasörünü seçin.
3. **Karoları Palete Aktarma:**
   * Project panelinden `Assets/Art/tilemap_packed` dosyasını tutun ve Tile Palette penceresinin boş alanına sürükleyip bırakın.
   * Unity karoların veri dosyalarını (`.asset`) nereye kaydedeceğini soracaktır: Yine **`Assets/Palettes`** klasörünü seçin.
   * Birkaç saniye içinde tüm zindan karoları düzenli bir ızgara şeklinde paletinizde belirecektir!
4. **Çizim Araçları:**
   * **Fırça (Brush - Kısayol: B):** Karoyu seçip tek tek boyar.
   * **Kutu Doldurma (Box Fill - Kısayol: U):** Tıklayıp sürükleyerek dikdörtgen zemin alanları çizer.
   * **Silgi (Eraser - Kısayol: D):** Yanlış çizilen karoları siler.
5. **Odayı Çizmeye Başlama (Altın Kural: Active Target):**
   * Tile Palette penceresinin üst kısmında **Active Target** açılır kutusu bulunur. Çizim yapmadan önce buranın doğru Tilemap olduğuna MUTLAKA dikkat edin:
   * **Adım A (Zemin):** Active Target'ı **`Tilemap_Floor`** yapın. Paletten açık gri/kum zemin karosunu seçin ve Scene ekranında farenizle sürükleyerek geniş bir oda zemini boyayın.
   * **Adım B (Duvarlar):** Active Target'ı **`Tilemap_Walls`** yapın. Paletten taş duvar karolarını seçerek zeminin etrafını çerçeveleyin.
   * *Tasarım İpucu:* Zindana 2.5D derinlik hissi katmak için üst duvarları iki sıra çizin: Üst sıraya duvarın üst çatısını, alt sıraya ise gölgeli ön cephe taşını koyun.

---

## 5. Öğrenci Kontrol Listesi (Self-Check Checklist)

Faz 1'i eksiksiz tamamladığınızdan emin olmak için aşağıdaki maddeleri tek tek kontrol edin:

- [ ] Kenney Tiny Dungeon paketi başarıyla indirildi ve `tilemap_packed.png` dosyası `Assets/Art` klasörüne aktarıldı.
- [ ] Görsel ayarlarında **PPU = 16**, **Sprite Mode = Multiple**, **Filter Mode = Point** ve **Compression = None** yapıldı.
- [ ] Sprite Editor ile görsel 16x16 Grid formatında dilimlendi (`Slice`) ve `Apply` dendi.
- [ ] Project Settings altında 6 adet **Sorting Layer** (`Background`, `Decorations`, `Walls`, `Interactables`, `Characters`, `UI`) doğru sırayla açıldı.
- [ ] Hierarchy'de `Grid` altında iki ayrı nesne var: `Tilemap_Floor` (Sorting Layer: `Background`) ve `Tilemap_Walls` (Sorting Layer: `Walls`).
- [ ] `Tilemap_Walls` üzerinde `Tilemap Collider 2D` (**Used By Composite** işaretli) ve `Composite Collider 2D` mevcut.
- [ ] `Rigidbody 2D` bileşeninin **Body Type** ayarı **Static** yapıldı.
- [ ] Sahne ekranında duvarların etrafında yeşil renkli, tek parça birleşik çarpışma çizgileri görünüyor.
- [ ] Unity'de **Play (▶️)** butonuna basıldığında duvarlar aşağı düşmüyor ve ekranda bozulma olmadan sabit duruyor.

---

## 6. Sık Karşılaşılan Sorunlar ve Çözümleri (Troubleshooting)

### ❓ Soru 1: Inspector'da "Sprite Editor" butonu görünmüyor veya tıklayınca açılmıyor!
> **Çözüm:** Unity'nizde 2D Sprite paketi eksik olabilir. Üst menüden **Window > Package Manager**'ı açın. Sol üstten **Unity Registry** seçin, arama çubuğuna `2D Sprite` yazıp paketi **Install** edin.

### ❓ Soru 2: Karoları sahneye çizdiğimde kenarları bulanık görünüyor!
> **Çözüm:** `tilemap_packed` görseline tıklayın. Inspector'daki **Filter Mode** ayarının `Point (no filter)` olduğundan ve en alttaki **Compression** ayarının `None` olduğundan emin olup **Apply** butonuna bastığınızı doğrulayın.

### ❓ Soru 3: Karolar arasında ince boşluklar veya dikiş izleri (seams) kalıyor!
> **Çözüm:** `tilemap_packed` görselinizin **Pixels Per Unit (PPU)** değerinin tam olarak `16` olduğundan emin olun. 16'dan farklı bir sayı grid oturmasını bozar.

### ❓ Soru 4: Duvarları çiziyorum ama zemin taşlarının altında kalıyor, görünmüyor!
> **Çözüm:** Sorting Layer sıralamanızı kontrol edin. `Tilemap_Floor` nesnesinin Sorting Layer'ı `Background`, `Tilemap_Walls` nesnesinin Sorting Layer'ı ise `Walls` olmalıdır. Project Settings'te `Walls` katmanının `Background` katmanının altında yer aldığından emin olun.

### ❓ Soru 5: Play butonuna bastığım an çizdiğim tüm duvarlar aşağı düşüp kayboluyor!
> **Çözüm:** `Tilemap_Walls` üzerindeki `Rigidbody 2D` bileşenini bulun. **Body Type** ayarını `Dynamic` yerine mutlaka **Static** yapın. Dynamic mod yer çekimini etkinleştirir.

---

🎉 **Tebrikler!** FAZ 1'i başarıyla tamamladınız. Artık katı fizik sınırlarına sahip, pikselleri tertemiz görünen bir zindan dünyanız var. Bir sonraki fazda karakterimizi sahneye ekleyecek ve C# ile hareket kodlarını yazacağız!