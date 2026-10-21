# Hâre — Güncelleme ve Yenilikler Raporu (v3.0)

Bu belgede, Hâre İnsani Yardım ve Ekip Koordinasyon Platformu'nda yapılan son düzeltmeler ve eklenen özelliklerin detayları yer almaktadır.

### 1. 💬 WhatsApp Geri Tuşu ve Uygulamanın Kapanmasını Önleme

* **Sorun:** WhatsApp bağlantısına tıklandığında arka planda `window.location.href = webUrl` çalıştırıldığı için WebView `api.whatsapp.com` adresine yönleniyor, telefonun donanım/sanal geri tuşuna basıldığında WebView geçmişi kaybolduğu için Hâre kapanıyordu.

* **Çözüm:**

  * Arka plandaki sayfa yönlendirmesi (`setTimeout(..., 450)`) tamamen kaldırıldı.

  * WhatsApp ve telefon arama işlemleri `target="_blank"` özellikli sanal bağlantılar (`<a>`) üzerinden harici Intent olarak tetiklenir; Hâre uygulaması kendi sayfasında kalır.

  * Harici eylem başlatılmadan önce `pushNavState('external')` geçmiş bayrağı oluşturulur. WhatsApp'tan geri dönüldüğünde veya Android donanım geri tuşuna basıldığında `popstate` olayı yakalanarak uygulamanın kapanması engellenir.

### 2. 🔄 Sayfa Yenilemede (Refresh / F5) Mevcut Konumu Koruma

* **Sorun:** Kullanıcı herhangi bir alt sekmedeyken (örneğin Sandık > Sorumlu Kasaları, Bağışçı detayları, veya filtrelenmiş hane listesi) sayfayı yenilediğinde veya üst bardaki "Canlı" eşitlemeye bastığında uygulama sıfırlanıp varsayılan "Haneler" ana sayfasına dönüyordu.

* **Çözüm:**

  * `sessionStorage` destekli **Gezinti Durumu Koruma Motoru** (`saveNavState()` ve `restoreNavState()`) eklendi.

  * Aktif sekme (`activeTab`), Sandık alt sekmesi (`kasaSubTab`), Ekip sekmesi (`adminSubTab`), Sorumlu Kasası (`viewingMemberKasa`), Hayır Defteri (`viewingDonorDetails`) ve Kategori Filtresi (`viewingStatusList`) her geçişte anında hafızaya kaydedilir.

  * Sayfa yenilendiğinde kullanıcı kaldığı sayfadan, seçtiği filtreden ve açık olan detay ekranından çalışmaya devam eder.

### 3. 🤝 Hayırsever (Bağışçı) Bakiyesi ve Kişisel Hayır Defteri

* **Bağışçı Kartlarında Otomatik Bakiye:**

  * `calculateDonorStats(donorName)` motoru ile her hayırseverin Sandık (Kasa) hareketleri otomatik taranır.

  * Her hayırsever kartında:

    * **Toplam Teberru Bakiyesi (₺)** (Mahremiyet modu ile gizlenebilir/açılabilir)

    * **Banka Hesabı ve Elden Nakit** dağılımı

    * Toplam kayıtlı bağış adedi (`X Bağış / Teberru Kaydı`)

* **📖 Hayır Defteri (Ayrıntılı İşlem Geçmişi Ekranı):**

  * Hayırsever kartındaki `[📖 Hayır Defteri (İşlemleri Gör) →]` butonuna basıldığında o hayırsevere özel tam döküm sayfası açılır.

  * Hayırseverin tüm bağışları tarih, ödeme kanalı (🏦 Banka / 💵 Elden), tutar (`+XX.XXX ₺`), açıklama ve emaneti teslim alan kasa sorumlusu ile birlikte ayrı ayrı listelenir.

  * Üstte "Tüm İşlemler", "🏦 Banka", "💵 Elden" filtre butonları yer alır.

  * Sol üstteki `[← Hayırseverlere Dön]` butonu veya cihazın geri tuşu ile kolayca ana listeye dönülür.

### 4. 🗑️ Silinen Kayıtların Kalıcı Olarak Gizlenmesi ve Önbellek Temizleme

* **Sorun:** E-Tablo'dan veya uygulamadan silinen kayıtlar, mobil tarayıcıların veya WebViews'ların GET isteklerini önbelleğe (cache) alması nedeniyle yeniden eşitleme yapıldığında tekrar ekranda belirebiliyordu.

* **Çözüm:**

  * **Önbellek Engelleme (Cache-Busting):** Google Apps Script'e gönderilen her GET isteğine benzersiz zaman damgası (`_ts=...`) ve rastgele anahtar (`_rnd=...`) eklenerek `cache: 'no-store'` ve `Pragma: 'no-cache'` HTTP başlıkları eklendi.

  * **Kalıcı Silinenler Kayıt Defteri (`hare_deleted_ids`):** Silinen tüm hane, kasa, bağışçı, devir ve üye ID'leri cihazın yerel hafızasına işlenir. Sunucudan veya ara bellekten eski bir veri gelse dahi bu ID'ler filtreye takılır ve kesinlikle ekranda gösterilmez.

  * **🧹 Önbelleği Temizle & Canlı Yenile Butonu:** Üst barda Canlı durum butonunun hemen yanına `[🧹 Temizle]` butonu eklendi. Tek dokunuşla cihazın ara belleklerini temizler ve en güncel E-Tablo verilerini sıfırdan çeker.

### 5. 💼 Sorumlu Kasası & Çift Onaylı Emanet Devri (Mevcut Korunan Özellikler)

* Her gönüllünün zimmetindeki bakiye ve hareketleri ayrı sayfada inceleme.

* Gönüllüler arası bakiye aktarımında karşı taraf onaylayana kadar askıda tutma güvenliği.

* Otomatik günlük yedekleme sistemi (Cihazda son 30 günün JSON arşivi ve Google E-Tablo "Yedekler" sayfası entegrasyonu).

### 10. 📍 İl Alanından Önermelerin Kaldırılması ve Serbest Yazım (Varsayılan: Kayseri)

* **Yapılan İyileştirme:**

  * Hane ekleme ve düzenleme formlarındaki "İl" kutusundan `datalist` (otomatik açılır şehir önerileri) tamamen kaldırıldı.

  * İl kutusu varsayılan olarak **Kayseri** yazılı gelir.

  * Gönüllü isterse bu kutuyu serbestçe silip başka bir ili (örneğin Kahramanmaraş, Hatay, Ankara vb.) klavyeden dilediği gibi yazabilir.

### 11. 📱 Ekran Kapanma / Arka Plan Uyku Koruması (Hata ve Kilitlenme Çözümü)

* **Sorun:** Kullanıcı uygulamadayken telefon ekranı kilitlendiğinde veya belli bir süre ekran kapalı kaldığında, telefon işletim sistemi (Android/iOS) ağ bağlantılarını ve arka plan işlemlerini askıya alır. Cihaz uyandığında devam eden istekler kopup `TypeError: Failed to fetch` veya zaman aşımı hatası veriyor; bu hata ekranda kalıcı kırmızı şerit olarak belirip uygulamayı kilitliyor ve kapatıp açmak gerekiyordu.

* **Çözüm:**

  * **Arka Plan İstek Engelleme:** Ekran kapalıyken (`document.hidden`) veya internet yokken her 30 saniyede bir çalışan arka plan eşitleme sayacı durdurulur, pil ve bağlantı korunur.

  * **Zaman Aşımı ve AbortController Koruması:** E-Tablo senkronizasyon (`fetch`) isteklerine 9 ve 12 saniyelik zaman aşımı denetleyicisi (`AbortController`) eklendi; bağlantı kopsa dahi sonsuz askıda kalma engellendi.

  * **Otomatik Uyanma ve Ağ Toparlanması (`visibilitychange` & `online`):** Ekran tekrar açıldığında askıda kalan eşitleme bayrakları sıfırlanır, ağ bağlantısının toparlanması için 600 ms beklenip taze veriler sessizce arka planda eşitlenir.

  * **Zararsız Ağ/Uyku Hatalarını Filtreleme:** Cihaz uyurken oluşan zararsız ağ kopması hataları (`Failed to fetch`, `NetworkError`, `AbortError`) filtrelendi; ekrana kırmızı hata basılması engellendi. Gerçek bir hata durumunda ise 4 saniye sonra otomatik kapanan ve üzerinde `✕` kapatma butonu bulunan esnek uyarı sistemi getirildi.

### 12. 📋 Girişte Daima "Haneler" Bölümünde Açılma Kuralı

* **Yapılan İyileştirme:**

  * Uygulama ilk açıldığında, sayfa yenilendiğinde veya PIN ile giriş yapıldığında; tarayıcı hafızasında hangi sekme kalmış olursa olsun sistem doğrudan **Haneler (Yardımlar)** ana ekranını açar.

  * Açık kalan filtreler, detay modalları veya alt sekmeler temizlenerek gönüllülerin ilk gördüğü ekranın her zaman ihtiyaç sahibi haneler listesi olması garanti altına alındı.

### 13. 🔙 Geri Tuşunda Uygulamadan Çıkmayı Önleme (Gelişmiş WebView Geçmiş Tamponu)

* **Sorun:** Kullanıcı "Hane Ekle (+)", "Sandık", "Hayırsever" veya "Gönüllü" sekmelerindeyken veya yeni hane eklerken Android donanım/hareket geri tuşuna bastığında, WebView geçmişi tükendiği için Android işletim sistemi uygulamayı doğrudan kapatıyordu.

* **Çözüm:**

  * **Kesintisiz Geçmiş Tamponu (`ensureHistoryBuffer`):** Android WebView'in `canGoBack()` denetimini her an canlı tutan döngüsel bir geçmiş tamponu oluşturuldu. Bu sayede Android işletim sistemi hiçbir zaman uygulamayı zorla kapatamaz.

  * **Kademeli Geri Dönüş Hiyerarşisi:**

    1. Açık bir onay veya bilgi penceresi (modal, fotoğraf, PIN değiştirme, düzenleme vb.) varsa önce o pencere kapatılır.

    2. Hayırsever Defteri, Sorumlu Kasası veya Kategori listeleme ekranı açıksa ana sekmeye dönülür.

    3. Hane Ekleme formunda veya başka bir sekmedeyse (`yeni`, `kasa`, `bagiscilar`, `ekip`), tek geri basışta güvenle **Haneler** ana sayfasına dönülür ve ekranda *"← Haneler ana sayfasına dönüldü"* bildirimi gösterilir.

    4. Sadece ve sadece zaten Haneler ana sayfasındayken geri basıldığında kullanıcıya çıkış için sağ üstteki **Çıkış (🚪)** butonunu kullanabileceği nazikçe hatırlatılır; uygulama kazara kapanmaz.

### 14. 🛰️ Mobil GPS Zaman Aşımı Kökten Çözümü (3 Aşamalı Akıllı Konum Motoru)

* **Sorun:** Masaüstü tarayıcılarda konum anında alınırken, cep telefonlarında ve Android WebView (APK) ortamında GPS donanımının uydulara kilitlenmesi 8-15 saniye sürer. Eski sistemdeki 3 ve 5 saniyelik agresif zamanlayıcılar uydular taranırken zaman aşımına uğruyor ve "Alınıyor..." yazısında takılıyordu.

* **Çözüm:**

  * **1. Aşama (0.2s - Anında Hızlı Konum):** Cihazın hafızasındaki veya Google Haritalar'dan kalma son 10 dakikalık hazır konum taranır; varsa 0.2 saniyede koordinatlar forma işlenir.

  * **2. Aşama (16s - Canlı Yüksek Hassasiyetli GPS):** Mobil GPS donanımının gökyüzündeki uyduları bulabilmesi için gerçekçi bir 16 saniyelik süre tanındı. Buton üzerinde saniye sayacıyla *"🛰️ Uydular Taranıyor (X saniye)..."* canlı geri bildirimi verilir.

  * **3. Aşama (Ağ / Baz İstasyonu Fallback):** Kapalı mekan veya bodrum katı gibi GPS sinyalinin ulaşamadığı noktalarda otomatik olarak baz istasyonu/hücresel konum devreye sokulur.

  * **4. Aşama (Akıllı Adres Harita Üretimi):** Cihazda GPS tamamen kapalıysa veya izin verilmediyse formdaki İlçe, Mahalle ve Şehir bilgisi anında taranarak tek dokunuşla kusursuz Google Haritalar linkine dönüştürülür.

### 15. 🛰️ Mobil APK'da Konum İzninin Çıkmaması Sorunu ve Çözüm Rehberi

* **Sorunun Asıl Sebebi (Kullanıcı Tespiti Doğrulandı):**

  * Web sürümünü Chrome üzerinden açtığınızda Chrome zaten Android'in yerel konum yetkisine sahip olduğu için kullanıcıya *"Bu sitenin konumunuza erişmesine izin verilsin mi?"* penceresini çıkarır ve izin verilince GPS'i alır.

  * Ancak WebIntoApp ile üretilen APK'larda durum farklıdır: Android işletim sistemi, bir uygulamanın (APK) donanım GPS'ine erişebilmesi için APK paketinin içindeki `AndroidManifest.xml` dosyasında `<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />` yetkisini şart koşar.

  * WebIntoApp'te APK üretilirken **"App Settings > Permissions"** sekmesindeki **"Location (GPS)"** kutucuğu işaretlenmemişse; Android işletim sistemi APK'nın konuma erişim hakkı olmadığını bildiği için kullanıcıya **izin sorma penceresini dahi açmaz**; isteği işletim sistemi seviyesinde doğrudan ve sessizce engeller. Bu yüzden uygulama ekranda bekler ve zaman aşımına uğrar.

* **APK'da Gerçek GPS İznini Açmak İçin Yapılması Gereken (WebIntoApp Ayarı):**

  1. [WebIntoApp.com](https://www.webIntoApp.com) kontrol panelinize girin ve projenizi açın.

  2. Sol menüden **Settings (Ayarlar) > Permissions (İzinler)** sekmesine gelin.

  3. **"Location (GPS)"** veya **"Geolocation"** seçeneğini **AKTİF (Açık)** yapın.

  4. Yeni APK'yı indirip telefonunuza kurduğunuzda, telefon ilk basışta *"Hâre uygulamasının cihaz konumuna erişmesine izin verilsin mi?"* sistem kutusunu ekrana çıkaracaktır.

* **Uygulama İçinde Yapılan İyileştirmeler (Sahte Beklemeye Son Verildi):**

  * **Hızlı Teşhis (3.5 Saniye):** Konum butonuna basıldığında Android 3.5 saniye içinde izin penceresi açmıyorsa (APK izninin olmadığı kesinleştiğinde), artık sahte sayaçlarla 20 saniye bekletilmez.

  * **Otomatik Adres Linki Üretimi:** Sistem APK'nın izin penceresi açmadığını hemen anlayıp formdaki Mahalle, İlçe ve Şehir bilgisinden anında kusursuz Google Haritalar navigasyon linkini üretir ve kutuya doldurur.

  * **"Adresten Üret" Butonu:** Sahada GPS izniyle veya uydu aramayla hiç vakit kaybetmek istemeyen gönüllüler için **"🗺️ Adresten Üret"** butonu tek dokunuşla hazır durumdadır.
---

### 16. 🚪 Geri Tuşunda 2 Kez Basarak Çıkma (Double-Tap Back to Exit)
- **Sorun:** Kullanıcı ana sayfadayken yanlışlıkla geri tuşuna bastığında veya sekmeler arasında gezinirken uygulamanın aniden kapanması.
- **Çözüm (Mobil Standart 2 Kere Basma Koruması):**
  1. **Alt Sayfalar ve Sekmelerde:** Hane Ekleme formundayken, Sandık, Hayırsever veya Gönüllü sekmelerindeyken veya herhangi bir pencere/modal açıkken geri tuşuna basıldığında **uygulama kesinlikle kapanmaz**; açık pencereyi kapatır veya güvenle **Haneler** ana sayfasına döner (`← Haneler ana sayfasına dönüldü`).
  2. **Ana Sayfada İlk Basış:** Kullanıcı zaten Haneler ana sayfasındayken geri tuşuna basarsa uygulama kapanmaz; ekranda **"🚪 Çıkmak için 2 saniye içinde tekrar basın"** bildirimi çıkar.
  3. **Ana Sayfada İkinci Basış (2 Saniye İçinde):** Kullanıcı gerçekten çıkmak istiyorsa ve 2 saniye içinde tekrar geri tuşuna basarsa, uygulama **"Hâre kapatılıyor..."** diyerek güvenle ve temiz bir şekilde kapanır. 2 saniyeden fazla süre geçerse sayaç sıfırlanır, kazara kapanmalar tamamen önlenir.

---

### 17. 📱 Alt Menü Tasarımı: Beyaz Menü Kartı + Tema Yeşili Sistem Tuşları Barı
- **Kullanıcı Talebi:** Uygulamanın 5 bölümü (Haneler, Sandık, Hane Ekle, Hayırsever, Gönüllü) önceki beyaz, yuvarlak hatlı ve şık tasarımında kalsın; telefonun alt sanal tuşlarının / hareket çubuğunun bulunduğu zemin ise sistemin zümrüt yeşili (`#065f46`) ile kaplansın.
- **Uygulanan Çözüm (Çift Katmanlı Alt Çubuk Mimarisi):**
  1. **Bizim Bölümler (Beyaz Menü Kartı):**
     - Zemin: Arka planı hafif buzlu beyaz (`bg-white/98 backdrop-blur-xl`), üst köşeleri kavisli (`rounded-t-3xl`), üst kenarlık (`border-t border-slate-200/90`) ve yumuşak gölge (`shadow-[0_-8px_30px_rgba(0,0,0,0.12)]`).
     - Butonlar: `188450.jpg` ekranındaki özgün beyaz zeminli tasarım korundu. Aktif sekmede koyu zümrüt yeşili başlık (`text-emerald-800 font-extrabold`) ve altında yeşil parıltılı nokta gösterge yer alır. Pasif sekmeler sade ve net gri/slate tonlarındadır.
     - Merkez "+ Hane Ekle": Menünün üstünden hafifçe taşan, koyu yeşil halkalı (`ring-4 ring-emerald-950`) ve canlı zümrüt yeşili yuvarlak eylem butonu.
  2. **Tuşların Barı (Sistemin Yeşili):**
     - Beyaz menü kartının hemen altına ekranın en tabanına oturan `safe-system-bar` eklendi.
     - Rengi: Hâre'nin ana tema yeşili (`#065f46` / `bg-emerald-900`).
     - Yüksekliği: `max(1.5rem, calc(env(safe-area-inset-bottom, 0px) + 0.6rem))` (asgari 24px - 34px).
     - Sonuç: Android'in 3 sanal gezinti tuşu (üçgen geri, daire ana ekran, kare son uygulamalar) veya yatay hareket çizgisi tamamen bu zümrüt yeşili zemin üzerinde kalır; yukarıdaki beyaz menü butonlarıyla ve yazılarıyla asla çakışmaz.
  3. **İçerik Kaydırma Boşluğu:** Sayfa altlarındaki son kayıtların menü arkasında kaybolmaması için ana gövde alt boşluğu `pb-36` seviyesinde tutuldu.

---

### 18. 🗺️ Harita Butonunda "Tarayıcınızı Güncelleyin" / Play Store Döngüsünün Kökten Çözümü
- **Videodaki Sorun (188454.mp4 Analizi):**
  - Kullanıcı hane detay penceresinde **"Google Haritalar'da Aç / Yol Tarifi"** butonuna bastığında; Android WebView (TrebEdit / APK) `https://www.google.com/maps/...` bağlantısını `target="_blank"` ile kendi gömülü iç tarayıcısında açmaya çalışıyordu.
  - Google Haritalar web sürümü, kısıtlı mobil WebView tarayıcılarını desteklemediği için haritayı yüklemeyi reddedip ekrana **"Google Haritalar'ı kullanmak için tarayıcınızı güncelleyin"** uyarısını çıkarıyor ve kullanıcıyı Play Store indirme sayfasına yönlendiriyordu. Play Store'da Google Haritalar zaten kurulu olduğu için kullanıcı haritaya ulaşamadan çıkmak zorunda kalıyordu.
- **Uygulanan Çözüm:**
  1. **Doğrudan Yerel Uygulama Intent'i (Android Intent & geo URI):**
     - Web URL'si yerine doğrudan cihazda yüklü olan Google Haritalar uygulamasını çalıştıran `intent://maps.google.com/...#Intent;scheme=https;package=com.google.android.apps.maps;...` ve `geo:` protokolü entegre edildi.
     - `target="_blank"` kaldırıldı; WhatsApp ve telefon arama mekanizmasında olduğu gibi Intent doğrudan Android işletim sistemine iletildi.
  2. **İki Ayrı Net Eylem Butonu:**
     - **🗺️ Haritada Gör:** Hanenin konumunu Google Haritalar uygulamasında işaretli pin olarak açar.
     - **🧭 Canlı Navigasyon:** Saha gönüllüsünün yola çıkıp hane kapısına kadar sesli navigasyonla gitmesi için Google Haritalar'ın canlı yol tarifini doğrudan başlatır (`dirflg=d`).
  3. **Çoklu Platform Uyumluluğu:** Android'de yerel Google Haritalar uygulaması, iOS cihazlarda Apple Haritalar (`maps://`), masaüstünde ise web tarayıcı sürümü otomatik olarak devreye girer.
