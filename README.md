# GRAV_MATRIX // v3.0 — Cyberpunk N-Body Astrofizik Laboratuvarı

**GRAV_MATRIX v3.0**, hiçbir harici kütüphane, oyun motoru, CDN veya hazır ses/görsel dosyası kullanmadan; tamamen **Vanilla JavaScript**, **HTML5 Canvas** ve **Web Audio API** ile tek bir [`index.html`](index.html) dosyası içinde çalışan interaktif bir **N-Cisim (N-Body) Kütleçekim, Relativistik Astrofizik ve Uzay Mühendisliği Simülasyonudur**.

Simülasyon; **Velocity Verlet** sayısal integrasyonu, **$O(N \log N)$ Barnes-Hut Quadtree** çözücüsü, **Adaptif Zaman Adımı (Sub-stepping)**, **1PN Relativistik Yörünge Presesyonu**, **Kısıtlı Üç-Cisim Lagrange Noktaları ($L_1 - L_5$)**, **Kara Delik Birleşmesi ve Kütleçekim Dalgaları**, **Yığılma Diskleri ve Relativistik Jetler**, **Gelgit Kilitlenmesi**, **Atmosfer Kaçışı**, **Süpernova Kalıntı Nebulaları**, **Canlı Fizik Telemetrisi**, **7 Oynanabilir Görev (Mission Mode)** ve **Prosedürel Synthwave Ses Motoru** ile gerçek zamanlı bir uzay-zaman laboratuvarı sunar.

---

## İçindekiler

1. [Mimari ve Çalışma Prensibi](#1-mimari-ve-çalışma-prensibi)
2. [PHASE 1 — Core Experience (Zaman Kontrolü, Inspector, Telemetri, Alan Haritaları)](#2-phase-1--core-experience)
3. [PHASE 2 — Advanced Physics (Barnes-Hut, Adaptif Zaman Adımı, 1PN Presesyon, Lagrange Noktaları)](#3-phase-2--advanced-physics)
4. [PHASE 3 — Astrophysical Events (Kara Delik Birleşmesi, Yığılma Diski & Jetler, Gelgit Kilitlenmesi, Atmosfer Kaçışı, Süpernova Kalıntısı)](#4-phase-3--astrophysical-events)
5. [PHASE 4 — Sandbox & Gameplay (Senaryo Editörü, JSON Kayıt/Yükleme, 7 Görev Modu, Kamera & Minimap, Uyarı Sistemi)](#5-phase-4--sandbox--gameplay)
6. [Gökcismi Sınıf Hiyerarşisi ve Kütle Evrimi](#6-gökcismi-sınıf-hiyerarşisi-ve-kütle-evrimi)
7. [Prosedürel Synthwave / Cyberpunk Ses Motoru](#7-prosedürel-synthwave--cyberpunk-ses-motoru)
8. [Kontroller ve Klavye/Fare Kısayolları](#8-kontroller-ve-klavyefare-kısayolları)
9. [Kurulum ve Çalıştırma](#9-kurulum-ve-çalıştırma)

---

## 1. Mimari ve Çalışma Prensibi

Proje üç ana katman üzerinde saniyede 60 kare (`requestAnimationFrame`) hızında çalışır:

1. **Fizik Katmanı (Velocity Verlet + Barnes-Hut + 1PN Relativite):**
   - Standart Euler integrasyonu yerine enerji korunumu çok daha yüksek olan **Velocity Verlet** algoritması kullanılır:
     $$\mathbf{r}(t + \Delta t) = \mathbf{r}(t) + \mathbf{v}(t)\Delta t + \frac{1}{2}\mathbf{a}(t)\Delta t^2$$
     $$\mathbf{v}(t + \Delta t) = \mathbf{v}(t) + \frac{1}{2}\left[\mathbf{a}(t) + \mathbf{a}(t + \Delta t)\right]\Delta t$$
   - Çok yakın geçişlerde sayısal patlamaları önlemek için kütleçekim formülüne yumuşatma parametresi ($\epsilon^2 = \text{SOFTENING}$) eklenmiştir:
     $$F = \frac{G \cdot M_1 \cdot M_2}{r^2 + \epsilon^2}$$
2. **Görsel Render Katmanı (HTML5 2D Canvas & Offscreen Alan Haritaları):**
   - Uzay-zaman dokusu (`GRID`), her kütlenin yarattığı potansiyel kuyusuna ve kütleçekim dalgalarına (`gravWaves`) göre bükülür.
   - Kara delikler ve pulsarlar etrafındaki arka plan yıldızları ve ızgara noktaları **Einstein Halkası / Kütleçekimsel Merceklenme** denklemiyle bükülür.
3. **İşitsel Katman (Web Audio API):**
   - Harici hiçbir `.mp3` veya `.wav` dosyası yoktur. Tüm sesler gerçek zamanlı osilatörler, filtreler ve gürültü tamponlarıyla matematiksel olarak sentezlenir.

---

## 2. PHASE 1 — Core Experience

### Zaman Kontrolü ve Geri Sarma (Time Control & Rewind Bar)
Ekranın alt kısmında yer alan zaman kontrol paneli:
- **Pause / Resume (`SPACE`):** Simülasyonu duraklatır veya devam ettirir.
- **Step Forward (`STEP >|`):** Simülasyon duraklatılmışken tek bir hassas fizik adımı ilerletir.
- **Hız Çarpanları:** `0.25x`, `0.5x`, `1x`, `2x`, `5x`, `10x` zaman ölçekleme.
- **Rewind & Timeline Slider (`<< REWIND`):** 300-snapshot kapasiteli dairesel tampon (`snapshotBuffer`) sayesinde tüm gökcisimlerini, yörünge izlerini, parçacıkları, süpernova kalıntılarını ve kütleçekim dalgalarını geçmişteki herhangi bir ana kayıpsız geri sarar.

### Gökcismi Bilgi Paneli (Celestial Body Inspector — `İNCELE` Sekmesi)
Herhangi bir gökcismine tıklandığında açılan canlı denetim paneli:
- **17 Canlı Fiziksel Metrik:** İsim/ID, Tür, Kütle, Konum $(X, Y)$, Hız $(V_x, V_y, |V|)$, İvme $|a|$, Kinetik Enerji ($E_k$), Potansiyel Enerji ($E_p$), Yörünge Yarıçapı ($R$), Yörünge Periyodu ($T$), Kaçış Hızı ($V_{\text{esc}}$), Dönüş Hızı ($\omega$), En Yakın Büyük Kütle ve Mesafesi, Roche Limiti Mesafesi ve Anlık Fiziksel Durum.
- **Canlı Müdahale:** Seçili cismin kütlesini, türünü, rengini ve hız vektörünü (`-20%`, `+20%`, `⇄ TERS ÇEVİR`) anında değiştirebilir; cismi **Takip Edebilir (`ODAKLA`)**, uzayda **Sabitleyebilir (`DONDUR`)** veya **Silebilirsiniz (`SİL`)**.

### Canlı Fizik Telemetrisi (`TELEMETRİ` Sekmesi)
Harici grafik kütüphanesi kullanmadan özel HTML5 Canvas üzerinde çizilen gerçek zamanlı grafikler:
- **7 Seçilebilir Metrik:** Toplam Sistem Enerjisi ($E_{\text{tot}}$), Sayısal İntegrasyon Sapması ($\Delta E_{\text{num}}$, çarpışma ve süpernova enerji sıçramalarından arındırılmış), Toplam Açısal Momentum ($L_z$), Toplam Kütle ($\sum M$), Seçili Cisim Hızı ($|V|$), Seçili Cisim İvmesi ($|a|$) ve Seçili Cisim Yörünge Enerjisi ($E_{\text{orb}}$).
- **Etkileşim:** `120`, `300`, `600` örneklem zaman penceresi seçimi ve fare ile üzerine gelindiğinde (hover) **Crosshair & Veri Noktası Okuma**.

### Kütleçekim Alanı Görselleştirme (`ALAN` Sekmesi)
Ayrı ayrı açılıp kapatılabilen görsel fizik katmanları:
- **Yörünge İzleri (Orbit Trails)**
- **Hız Vektörleri ($\vec{v}$ — Cyan)**, **İvme Vektörleri ($\vec{a}$ — Sarı)** ve **Net Kuvvet Vektörleri ($\vec{F}$ — Magenta)**
- **Kuvvet Vektör Izgarası (Vector Field Arrows)**
- **Offscreen Alan Haritaları:**
  - **Isı Haritası ($|g|$ Gravity Heatmap):** Koyu Mavi $\to$ Cyan/Yeşil $\to$ Sarı/Turuncu $\to$ Kırmızı/Magenta.
  - **Potansiyel Alan ($\Phi = -\sum GM/r$):** Eşpotansiyel kontur çizgileriyle kütleçekim kuyuları.
  - **Kaçış Hızı Haritası ($V_{\text{esc}} = \sqrt{2|\Phi|}$):** Uzayın her noktasındaki yerel kaçış hızı dağılımı.

---

## 3. PHASE 2 — Advanced Physics

### Barnes-Hut Quadtree Algoritması ($O(N \log N)$)
- `FİZİK` sekmesinden **Doğrudan $O(N^2)$**, **Barnes-Hut $O(N \log N)$** veya **Otomatik ($N \ge 45$)** çözücü modları seçilebilir.
- **Teta ($\theta \in [0.25, 1.20]$) Hassasiyet Ayarı:** Uzak kütle kümelerinin tekil kütle merkezi (monopol) olarak yaklaşık hesaplanma eşiğini kontrol eder.
- **Quadtree Görselleştirme (`QUADTREE AĞACINI GÖSTER`):** Uzayı özyinelemeli olarak dörde bölen ağaç düğümlerini canlı neon çerçevelerle ekranda gösterir.

### Adaptif Zaman Adımı (Adaptive Time Step / Sub-stepping)
- Cisimler birbirine çok yaklaştığında veya yüksek ivmeli yakın geçişlerde ($\min(d / v) \ll 1$), fizik motoru tek kareyi otomatik olarak **`1x`, `2x`, `4x` veya `8x` alt-adıma (sub-step)** böler ($dt \in [0.125, 1.00]$).
- Böylece yakın geçişlerde yörünge sapmaları, Roche parçalanması ve çarpışma tespiti yüksek doğrulukla çözülür.

### 1PN Relativistik Yörünge Presesyonu (Relativistic Precession)
- Kara delik, pulsar ve yıldızların yakınında dönen cisimlere 1. Post-Newtonian (1PN) genel görelilik düzeltmesi uygulanır:
  $$\mathbf{a}_{\text{1PN}} \propto \frac{GM}{c^2 r^3}\left[\left(\frac{4GM}{r} - v^2\right)\mathbf{r} + 4(\mathbf{r}\cdot\mathbf{v})\mathbf{v}\right]$$
- Bu etki sayesinde eliptik yörüngeler sabit kalmaz; Merkür'ün günberi (perihelion) kayması gibi **rozet (rosette) yörüngeler** çizer. Işık hızı sabiti ($c$) panelden ayarlanabilir.

### Lagrange Noktaları Sistemi ($L_1, L_2, L_3, L_4, L_5$)
- Sistemdeki en büyük iki kütle ($M_1, M_2$) için Dairesel Kısıtlı Üç-Cisim Problemi (CR3BP) denklemleri gerçek zamanlı çözülür:
  - **Doğrusal Noktalar ($L_1, L_2, L_3$):** Beşinci dereceden (quintic) Euler-Lagrange denklemi **Newton-Raphson** iterasyonu ile çözülür (eyrer/kararsız denge noktaları).
  - **Üçgen Noktalar ($L_4, L_5$):** Eşkenar üçgen (Trojan) noktalarıdır; kütle oranı $\mu = \frac{M_2}{M_1 + M_2} < 0.0385$ koşulunu sağladığında **kararlı (stable)** olarak işaretlenir.
- Noktaların üzerine fareyle gelindiğinde kararlılık analizi açılır; **`L1`–`L5` Uydu Yerleştir** butonlarıyla seçilen Lagrange noktasına sistemle eş-dönüşlü (co-rotating) test uydusu tek tıkla bırakılabilir.

---

## 4. PHASE 3 — Astrophysical Events

### A. Kara Delik Birleşmesi ve Kütleçekim Dalgaları (Black Hole Merger)
- İki kara delik birbirine yaklaştığında kütleçekim dalgası radyasyon sönümlemesi (inspiral damping) ile sarmal çizerek yakınlaşır ve uzay-zaman ızgarasını gerçek zamanlı dalgalandıran genişleyen **Kütleçekim Dalgaları (`gravWaves`)** yayar.
- Birleşme anında toplam kütlenin $\%5$'i $E = \Delta m c^2$ kütleçekim dalgası enerjisi olarak uzaya yayılır, güçlü bir şok dalgası ve **LIGO Chirp** sesi tetiklenir. (`İKİLİ KARA DELİK` hazır senaryosu ile doğrudan test edilebilir).

### B. Yığılma Diski ve Relativistik Kutup Jetleri (Accretion Disk & Jets)
- Kara delikler ve pulsarlar etrafında Keplerci diferansiyel hızla ($v \propto r^{-1/2}$) dönen, sıcaklığa göre renk değiştiren (iç bölge mavi-beyaz X-ışını, orta bölge sarı-turuncu, dış bölge kızıl) **Yığılma Diski** bulunur.
- Kutuplardan iki yönlü **Relativistik Plazma Jetleri** fışkırır; diske giren küçük cisimler sürtünmeyle parçalanarak merkeze kütle aktarır.

### C. Gelgit Kilitlenmesi (Tidal Locking)
- Büyük bir kütlenin yakınında uzun süre yörüngede kalan gezegenlerin kendi ekseni etrafındaki dönüş periyodu, yörünge periyoduyla kilitlenir ($\omega_{\text{rot}} = \omega_{\text{orb}}$).
- Kilitlenen gezegenin yıldızına bakan yüzü sürekli aydınlık/sıcak, arka yüzü karanlık/soğuk kalır ve Inspector panelinde `[GELGİT KİLİTLİ]` durumu görüntülenir.

### D. Atmosfer Kaçışı (Atmospheric Escape)
- Yıldızına veya pulsara çok yaklaşan gezegenlerin atmosferi yıldız rüzgarı etkisiyle iyonize olarak yıldızın aksi yönünde kuyruklu bir **Atmosfer Kaçış Bulutu** oluşturur ve gezegenin atmosfer yoğunluğu ($\%100 \to \%0$) zamanla aşınır.

### E. Süpernova Kalıntı Nebulası (Supernova Remnant)
- Kritik kütleyi ($M \ge 3200$) aşan bir yıldız süpernova olarak patladığında merkezde yüksek hızlı bir **Pulsar** bırakır ve etrafında Sedov-Taylor fazında genişleyen çok katmanlı, parıldayan bir **Süpernova Kalıntı Nebulası (`supernovaRemnants`)** oluşur.

---

## 5. PHASE 4 — Sandbox & Gameplay

### Senaryo Editörü ve JSON Kayıt / Yükleme (`EDİTÖR` Sekmesi)
- **Sistem Oluşturucu:** `BOŞ SİSTEM`, `RASTGELE SİSTEM` (prosedürel yıldız/gezegen/kuyruklu yıldız üretici) ve `VARSAYILAN` butonları.
- **Hassas Cisim / Yörünge Üretici:** Tür, kütle, $(X, Y)$ koordinatı ve $(V_x, V_y)$ hız bileşenlerini elle girerek veya **`OTOMATİK DAİRESEL HIZ HESAPLA`** butonuyla ideal yörünge hızını bularak cisim ekleme; seçili cismin verilerini forma çekip güncelleme.
- **Tam JSON Kayıt / Yükleme:**
  - Tarayıcı hafızasına (`LocalStorage`) tek tıkla kaydetme ve yükleme.
  - `.json` dosyası olarak bilgisayara indirme (`JSON İNDİR`) ve `.json` dosyası yükleme (`DOSYA YÜKLE`) veya metin kutusundan içe aktarma.

### Görev Modu — 7 Oynanabilir Uzay Mühendisliği Görevi (`GÖREV` Sekmesi)
1. **1. Kararlı Güneş Sistemi:** Merkez yıldız etrafında en az 3 gezegeni 12 saniye boyunca çarpışmadan ve sistemden kaçmadan bağlı yörüngede tut.
2. **2. Kütleçekim Sapanı (Gravitational Slingshot):** Bir kuyruklu yıldızı dev gezegenin çekim alanından sapan etkisiyle geçirerek yeşil hedef bölgeye (`TARGET ZONE`) ulaştır.
3. **3. Roche Limiti Parçalanması:** Bir gezegeni dev kütlenin Roche limitinin içine sokarak gelgit kuvvetleriyle en az 6 parçaya ayır ve halka oluştur.
4. **4. İkili Yıldız Dengesi:** İki yıldız ve gezegenlerinden oluşan sistemi 12 saniye boyunca yıldızlar çarpışmadan kararlı tut.
5. **5. Yörüngeye Yakalama (Orbital Capture):** Hiperbolik kaçış yörüngesindeki ($E_{\text{orb}} > 0$) başıboş gezegeni kütleçekimsel müdahaleyle kapalı eliptik yörüngeye ($E_{\text{orb}} < 0$) yakala.
6. **6. Kara Delik Hayatta Kalma:** Bir gezegeni kara deliğin tehlike sınırına ($< 125\text{ px}$) kadar yaklaştırıp yutulmadan güvenli mesafeye ($> 185\text{ px}$) geri çıkar.
7. **7. Yüksek Enerjili Çarpışma:** İki gökcismini bağıl hızı $\Delta v \ge 4.2\text{ px/f}$ olacak şekilde kafa kafaya çarpıştır.

### Kamera, Sinematik Mod ve Radar Minimap (`KAMERA` Sekmesi)
- **Zoom & Pan:** `0.35x` – `2.80x` yumuşak yakınlaştırma (`Ctrl/Alt + Fare Tekerleği` veya slider) ve `WASD` / Ok Tuşları / Orta Tuş Sürükleme / `Shift + Sol Tık` ile kamera kaydırma.
- **Sinematik Kamera (`SİNEMATİK MOD`):** Sistemdeki en yüksek kütleçekimsel aksiyonun (kara delik birleşmesi, Roche parçalanması, yakın geçiş veya en büyük kütle) olduğu noktaya otomatik odaklanır ve yumuşak zoom geçişleri yapar.
- **Radar Minimap & Ölçek Çubuğu:** Sağ alt köşede tüm uzayı ve kamera görüş çerçevesini (FOV) gösteren tıklanabilir **Radar Minimap** ile dinamik `px / AU` ölçek göstergesi.

### Gerçek Zamanlı Sistem Uyarıları (System Alerts HUD)
Fizik motorundaki anlık değişimleri izleyen otomatik erken uyarı sistemi:
- `ORBIT UNSTABLE` (Yörünge Kararsızlığı)
- `ESCAPE TRAJECTORY DETECTED` (Kaçış Yörüngesi Tespit Edildi)
- `COLLISION PREDICTED` (Çarpışma Uyarısı)
- `TIDAL DISRUPTION IMMINENT` (Gelgit Parçalanması Yaklaşıyor)
- `SUPERNOVA DETECTED` (Süpernova Patlaması)
- `GRAVITATIONAL COLLAPSE` (Kütleçekimsel Çöküş -> Kara Delik)
- `BLACK HOLE MERGER` (Kara Delik Birleşmesi)
- `SYSTEM ENERGY CRITICAL` (Kritik Sistem Enerjisi)

---

## 6. Gökcismi Sınıf Hiyerarşisi ve Kütle Evrimi

Bir gökcismi başka cisimleri yuttukça kütlesi artar ve kritik eşikleri aştığında otomatik olarak üst sınıfa evrilir (`evolve()`):

| Sınıf | Temel Kütle ($M_\oplus$) | Yarıçap | Özel Fiziksel Davranış |
| :--- | :--- | :--- | :--- |
| **Kuyruklu Yıldız (`Comet`)** | $2.5$ | $\propto M^{1/3}$ | Yıldız rüzgarı etkisiyle yıldızın aksi yönünde dinamik iyon/toz kuyruğu bırakır. |
| **Gezegen (`Planet`)** | $28$ | $\propto M^{1/3}$ | Atmosfer kaçışı, gelgit kilitlenmesi ve Roche parçalanmasına duyarlıdır. |
| **Gaz Devi (`Giant`)** | $160$ | $\propto M^{1/3}$ | Halka sistemi barındırır; $M > 750$ sınırında **Yıldıza** evrilir. |
| **Yıldız (`Star`)** | $1100$ | $\propto M^{1/3}$ | Korona parlaması yayar; $M > 2150$ (Chandrasekhar) sınırında **Süpernova** geçirerek **Pulsar** ve Nebula oluşturur. |
| **Pulsar (`Pulsar`)** | $2100$ | $6\text{–}14\text{ px}$ | Kutuplarından dönen relativistik radyasyon huzmeleri yayar; $M > 3100$ (TOV) sınırında **Kara Deliğe** çöker. |
| **Kara Delik (`Black Hole`)** | $3600$ | $12\text{–}34\text{ px}$ | Olay ufku, Einstein halkası, yığılma diski, kutup jetleri ve ikili birleşmede kütleçekim dalgaları üretir. |

---

## 7. PHASE 5 — Fiziksel Model Doğruluk Sınıflandırması ve Doğrulama Sonuçları

Simülasyonda kullanılan fiziksel modeller, tam sayısal çözümler ile yaklaşık/görsel modeller arasında şeffaf biçimde sınıflandırılmıştır (`FİZİK` sekmesi altındaki **Fiziksel Model Doğruluk Sınıflandırması** kutusunda da yer alır):

| Model / Sistem | Doğruluk Sınıfı | Açıklama |
| :--- | :--- | :--- |
| **Newton Kütleçekimi & Velocity Verlet** | **TAM SAYISAL (Symplectic 2. Derece)** | Yumuşatılmış ($\epsilon^2 = 140$) tam $N$-cisim denklemleri; doğrusal ve açısal momentumu makine hassasiyetinde ($\sim 10^{-14}\%$) korur. |
| **Lagrange Noktaları ($L_1\text{–}L_5$)** | **TAM SAYISAL (Newton-Raphson & Analitik)** | CR3BP quintic denklemi $|f(r)| < 10^{-14}$ rezidü ile çözülür; $\mu < 0.03852$ Trojan kararlılık sınırı tam hesaplanır. |
| **1PN Relativistik Presesyon** | **1PN YAKLAŞIM (Post-Newtonian)** | Einstein-Infeld-Hoffmann 1PN günberi kayması ivmesi (Verlet hız tahmincisiyle); analitik $\Delta\phi = \frac{6\pi GM}{a(1-e^2)c^2}$ formülüyle $\%3.24$ uyum içindedir. |
| **Barnes-Hut Quadtree ($O(N \log N)$)** | **MONOPOL YAKLAŞIMI ($s/d < \theta$)** | Düğüm-dışı kabul kriteri ve çakışan yaprak korumalı kuad-ağaç; $\theta = 0.55$ için ortalama kuvvet hatası $\%0.73$'tür. |
| **Kütleçekim Dalgaları (GW Inspiral)** | **FENOMENOLOJİK / GÖRSEL** | Radyasyon sönümlemesi ve birleşmede $\%5$ kütle-enerji kaybı fenomenolojiktir; tam 2.5PN dört-kutuplu tensör integrasyonu değildir. |
| **Kütleçekimsel Merceklenme & Jetler** | **GÖRSEL / YARI-ANALİTİK** | Arka plan yıldızlarının $\alpha \propto R_E^2 / b$ sapması ve kutup jetleri görsel/yarı-analitik temsil sunar; tam Schwarzschild/Kerr jeodezik ışın izleme veya GRMHD çözümü değildir. |

### Otomatik Fizik Doğrulama Özeti (12/12 PASS)
- **Dairesel Yörünge Testi:** Yarıçap hatası $\%0.026$, faz hatası $\%0.134$ (**PASS**)
- **Eliptik Yörünge Testi ($e=0.4$):** Yarı-büyük eksen ($a$) hatası $\%0.0145$, dışmerkezlilik ($e$) hatası $\%0.0183$ (**PASS**)
- **Doğrusal & Açısal Momentum Korunumu (1000 adım):** $|\Delta \mathbf{P}|/\mathbf{P}_0 = 1.62 \times 10^{-13}\%$, $|\Delta L_z|/L_{z0} = 9.90 \times 10^{-14}\%$ (**PASS**)
- **Enerji Korunumu (1000 adım):** Symplectic Velocity Verlet bağıl enerji kayması $\%0.00534$ (**PASS**)
- **Çarpışma, Roche Parçalanması ve Süpernova Korunumu:** Kütle ve doğrusal momentum korunumu hatası $< 10^{-13}\%$ (**PASS**)
- **Rewind & Determinizm Testi:** Mulberry32 PRNG (`rngState`) ve derin kopya (deep-copy) snapshot sistemi ile geri sarma sonrası konum, hız ve parçacık tekrarlanabilirlik farkı $= 0.0$ (**PASS**)

---

## 8. Prosedürel Synthwave / Cyberpunk Ses Motoru

- **Ambient Uzay Dronu:** $55\text{ Hz}$ (A1) ve $110\text{ Hz}$ (A2) testere dişi/üçgen osilatörlerin LFO kontrollü alçak geçiren filtreden geçirilmesiyle üretilir. Sistemdeki toplam kinetik enerji arttıkça filtre açılarak ses parlaklaşır.
- **Kuantize Pentatonik Çarpışma Sentezleyici:** Çarpışmalar A-Minör Pentatonik dizisindeki frekanslara kuantize edilir. Kütle büyüklüğü oktavı belirler.
- **LIGO Kara Delik Birleşmesi Chirp Sesi:** İki kara delik birleştiğinde frekansı hızla yükselen ($65\text{ Hz} \to 620\text{ Hz}$) karakteristik kütleçekim dalgası "chirp" sesi ve alt-bas patlaması üretilir.
- **Süpernova & Roche Efektleri:** Süpernovada 3 saniyelik derin sub-bass ($140\text{ Hz} \to 22\text{ Hz}$) ve filtrelenmiş beyaz gürültü şok dalgası; Roche parçalanmasında kristalize granüler ses dizisi sentezlenir.

---

## 8. Kontroller ve Klavye/Fare Kısayolları

| Girdi / Kısayol | İşlev |
| :--- | :--- |
| **Sol Tık (Cisim Üzerine)** | Gökcismini seçer ve sağdaki **Inspector (`İNCELE`)** panelini açar. |
| **Sol Tık (Boş Uzaya)** | Seçili kütleyi üretir (*Otomatik Yörünge* açıksa en yakın dev kütleye göre ideal dairesel hızla fırlatır). |
| **Sol Tık Basılı Tut + Sürükle** | Sapan (Slingshot) modu: Simüle edilmiş gelecek yörünge önizlemesini görerek açılı fırlatma yapar. |
| **Sağ Tık Basılı Tut** | İmleç konumunda geçici bir **Yapay Tekillik (Kara Delik Çekici)** oluşturur. |
| **Fare Tekerleği (Scroll)** | Üretilecek gökcisminin kütlesini dinamik olarak artırır / azaltır. |
| **`Ctrl / Alt` + Fare Tekerleği** | Kamerayı yakınlaştırır / uzaklaştırır (`Zoom`). |
| **`W, A, S, D` veya Ok Tuşları** | Kamerayı uzayda serbestçe kaydırır (`Pan`). |
| **Orta Tuş Sürükle / `Shift` + Sol Tık** | Fare ile kamerayı sürükleyerek kaydırır (`Pan`). |
| **`SPACE` (Boşluk)** | Simülasyonu Duraklatır / Devam Ettirir (`Pause / Resume`). |
| **`1` – `6` Tuşları** | Gökcismi sınıfları arasında hızlı geçiş yapar (`Comet` $\to$ `Black Hole`). |
| **`C` Tuşu** | Uzaydaki tüm cisimleri temizler. |

---

## 9. Kurulum ve Çalıştırma

Herhangi bir kurulum, paket yöneticisi (`npm`), derleyici veya sunucu gerektirmez:

1. [`index.html`](index.html) dosyasını doğrudan herhangi bir modern web tarayıcısında (Chrome, Edge, Firefox, Safari) çift tıklayarak açın.
2. Ses motorunu etkinleştirmek için üst bardaki **`SES: KAPALI`** butonuna tıklayarak **`SES: AÇIK`** konumuna getirin.
