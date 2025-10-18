# Prokudin-Gorskii Renkli Görüntü Restorasyonu

Bu proje, Sergey Prokudin-Gorskii’nin erken dönem renkli fotoğrafçılık yönteminden esinlenerek, gri tonlamalı cam plaka fotoğraflardan renkli görüntülerin dijital olarak yeniden oluşturulmasını amaçlamaktadır.  
Proje kapsamında, tek bir gri ölçekli görüntüden elde edilen üç kanal (mavi, yeşil, kırmızı) hizalanmış, iyileştirilmiş ve nihai olarak renkli hale getirilmiştir.

---

## 1. Projenin Amacı

20. yüzyılın başlarında Rus fotoğrafçı Sergey Prokudin-Gorskii, renkli fotoğraf elde etmek için aynı sahneyi üç farklı filtre (mavi, yeşil, kırmızı) ile fotoğraflamıştır.  
Bu görüntüler, orijinalde üst üste pozlanmış üç cam plaka biçimindedir.  
Bu projede amaç, bu cam plaka görüntülerini modern bilgisayarla görü teknikleriyle işleyip, yeniden renkli biçimde bir araya getirmektir.

Projenin hedefleri:
- Üç kanalın doğru biçimde hizalanmasını sağlamak.  
- Farklı benzerlik metriklerinin (SSD ve NCC) karşılaştırmasını yapmak.  
- Gama düzeltmesi ve histogram eşitleme ile kontrastı iyileştirmek.  
- Otomatik kırpma ile kenar artefaktlarını ortadan kaldırmak.  
- İşlem süresi ve görsel kalite arasında dengeli bir sonuç elde etmek.

---

## 2. Kullanılan Yöntemler ve Algoritmalar

### 2.1 Görüntü Bölme (split_image)
Girdi olarak alınan tek bir gri tonlamalı görüntü, dikey olarak üç eşit parçaya bölünür:
- Üst kısım: Mavi kanal (B)
- Orta kısım: Yeşil kanal (G)
- Alt kısım: Kırmızı kanal (R)

Her kanal, 0–1 aralığında normalize edilmiş tek kanallı görüntülerdir.

---

### 2.2 Hizalama (align_channels)
Hizalama işlemi, B kanalının referans alınması ve G ile R kanallarının bu referansa göre kaydırılması prensibine dayanır.  
Her kanal için yatay (x) ve dikey (y) yönde ±15 piksel aralığında arama yapılır.  
İki farklı benzerlik metriği test edilmiştir:

- **SSD (Sum of Squared Differences):**  
  Piksel farklarının karesinin toplamını hesaplar.  
  Parlaklık farklılıklarına karşı duyarlıdır.

- **NCC (Normalized Cross-Correlation):**  
  Normalize edilmiş korelasyon katsayısını hesaplar:  
NCC(A, B) = Σ(A−Ā)(B−B̄) / √[Σ(A−Ā)² Σ(B−B̄)²]

yaml
Kodu kopyala
Bu metrik, pozlama farklarına karşı daha dayanıklıdır.  
Deneysel sonuçlarda NCC daha kararlı ve doğru hizalama sağlamıştır.

**Piramit Tabanlı Hizalama (Pyramid Alignment):**  
Yüksek çözünürlüklü görüntülerde doğrudan tüm pikselleri aramak maliyetlidir.  
Bu nedenle görüntü, düşük çözünürlükten yüksek çözünürlüğe doğru kademeli olarak hizalanır.  
Bu yöntem, işlem süresini yaklaşık %60–70 oranında azaltır.

---

### 2.3 Görüntü İyileştirme (enhance_image)
Hizalanmış görüntü, iki farklı iyileştirme tekniği ile geliştirilmiştir:

1. **Histogram Eşitleme (Y kanalında, YCrCb uzayında):**  
 Görüntü YCrCb uzayına dönüştürülür, yalnızca Y (parlaklık) kanalı eşitlenir.  
 Bu işlem, genel kontrastı artırır ve renk dengesini korur.

2. **Gama Düzeltmesi (Gamma Correction):**  
 Görüntü parlaklık dengesi için gama düzeltmesine tabi tutulur.  
 Kullanılan formül:  
I_out = I_in^(1/γ)

yaml
Kodu kopyala
Bu projede γ = 1.2 olarak seçilmiştir.  
Bu değer, karanlık bölgelerde detayları ön plana çıkarırken genel parlaklığı korur.

---

### 2.4 Otomatik Kenar Kırpma (auto_crop)
Hizalama sonrası görüntüde oluşan siyah veya beyaz kenar bölgeleri otomatik olarak temizlenmiştir.  
Bu işlemde, satır ve sütun bazında parlak piksel oranı analiz edilmiştir.

Kullanılan eşik değerleri:
- `dark_thresh = 0.18`
- `bright_thresh = 0.82`
- `min_content_ratio = 0.8`

Bu oranların üzerindeki kenar kısımları artefakt olarak kabul edilip kırpılmıştır.  
Sonuç olarak, yalnızca anlamlı içeriği barındıran görüntü korunmuştur.

---

## 3. Dosya Yapısı

ProkudinGorskii_Restoration/
├── main.py # Ana yürütülebilir dosya
├── alignment.py # Hizalama işlemleri (SSD / NCC)
├── enhancement.py # Görüntü iyileştirme (histogram eşitleme + gama düzeltme)
├── utils.py # Yardımcı fonksiyonlar (shift, auto_crop vb.)
├── data/ # Girdi cam plaka görüntüleri
├── results/ # Çıktı (unaligned, aligned, enhanced, cropped ve JSON raporlar)
└── README.md # Proje açıklama dosyası

yaml
Kodu kopyala

---

## 4. Kurulum ve Gereksinimler

Bu proje Python 3.8+ sürümleriyle uyumludur.  
Gerekli kütüphaneler:


pip install numpy opencv-python
5. Çalıştırma Adımları
Projeyi çalıştırmak için terminalden şu komutu kullanın:

bash
Kodu kopyala
python main.py
Program varsayılan olarak aşağıdaki parametrelerle çalışır:

Parametre	Açıklama	Varsayılan Değer
DATA_DIR	Girdi klasörü	../data
OUT_DIR	Çıktı klasörü	../results
METRIC	Hizalama metriği	NCC
USE_PYRAMID	Piramit hizalama kullanımı	True
GAMMA	Gama düzeltme değeri	1.2

6. Örnek Sonuç (00056v.jpg)
Projede 00056v görüntüsü üzerinde yapılan işlemler sonucunda aşağıdaki dosyalar üretilmiştir:

Aşama	Çıktı Dosyası	Açıklama
1	00056v_unaligned.jpg	Kanallar hizalanmadan üst üste bindirilmiş görüntü.
2	00056v_aligned.jpg	NCC metriğiyle hizalanmış renkli görüntü.
3	00056v_enhanced.jpg	Histogram eşitleme ve gama düzeltmesi uygulanmış hali.
4	00056v_cropped.jpg	Kenar artefaktları temizlenmiş nihai görüntü.
5	00056v_report.json	Tüm işlem parametrelerini içeren rapor dosyası.

Örnek rapor içeriği:

{
  "image": "../data/00056v.jpg",
  "metric": "NCC",
  "time_sec": 0.279,
  "dx_G": 0,
  "dy_G": 8,
  "score_G": 0.748,
  "dx_R": 1,
  "dy_R": 13,
  "score_R": 0.434,
  "enhancement": {
    "histogram_equalization": "Y channel in YCrCb",
    "gamma": 1.2,
    "formula": "I_out = I_in^(1/gamma)"
  }
}
7. Kullanılan Tekniklerin Özeti

Teknik	Açıklama
SSD	Farkların karesi üzerinden benzerlik ölçümü, aydınlık değişimlerine duyarlıdır.
NCC	Normalize edilmiş korelasyon ölçümü, pozlama farklarına dayanıklıdır.
Pyramid Alignment	Çok katmanlı hizalama, büyük görüntülerde hesaplama süresini azaltır.
Histogram Equalization	Parlaklık dağılımını düzenler, kontrastı artırır.
Gamma Correction	Parlaklık tonlarını dengeler, detayları öne çıkarır.
Auto Crop	Kenar artefaktlarını otomatik tespit edip kırpar.

8. Deneysel Bulgular
NCC metriği, SSD’ye göre daha kararlı sonuçlar vermiştir.

Ortalama hizalama süresi 0.2–3 saniye arasında değişmektedir.

Gama düzeltmesi (γ=1.2), karanlık bölgelerde detay görünürlüğünü artırmıştır.

Histogram eşitleme, kontrastı ve renk dengesini belirgin şekilde iyileştirmiştir.

Otomatik kırpma, hizalama sonrası oluşan kenar hatalarını başarıyla ortadan kaldırmıştır.

9. Yazar Bilgileri
Hazırlayan: Samet Kartal – 220212006
Bölüm: Yapay Zekâ Mühendisliği
Üniversite: OSTİM Teknik Üniversitesi
Ders: Bilgisayarlı Görü
Öğretim Üyesi: Dr. Öğr. Üyesi Ramin ABBASZADİ

10. Lisans Bilgisi
Bu proje yalnızca akademik amaçlarla geliştirilmiştir.
Herhangi bir ticari kullanım veya yeniden dağıtım, yazarın iznine tabidir.

