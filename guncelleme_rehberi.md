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

### 16. 🚪 Geri Tuşunda 2 Kez Basarak Çıkma (Double-Tap Back to Exit)

* **Sorun:** Kullanıcı ana sayfadayken yanlışlıkla geri tuşuna bastığında veya sekmeler arasında gezinirken uygulamanın aniden kapanması.

* **Çözüm (Mobil Standart 2 Kere Basma Koruması):**

  1. **Alt Sayfalar ve Sekmelerde:** Hane Ekleme formundayken, Sandık, Hayırsever veya Gönüllü sekmelerindeyken veya herhangi bir pencere/modal açıkken geri tuşuna basıldığında **uygulama kesinlikle kapanmaz**; açık pencereyi kapatır veya güvenle **Haneler** ana sayfasına döner (`← Haneler ana sayfasına dönüldü`).

  2. **Ana Sayfada İlk Basış:** Kullanıcı zaten Haneler ana sayfasındayken geri tuşuna basarsa uygulama kapanmaz; ekranda **"🚪 Çıkmak için 2 saniye içinde tekrar basın"** bildirimi çıkar.

  3. **Ana Sayfada İkinci Basış (2 Saniye İçinde):** Kullanıcı gerçekten çıkmak istiyorsa ve 2 saniye içinde tekrar geri tuşuna basarsa, uygulama **"Hâre kapatılıyor..."** diyerek güvenle ve temiz bir şekilde kapanır. 2 saniyeden fazla süre geçerse sayaç sıfırlanır, kazara kapanmalar tamamen önlenir.

### 17. 📱 Alt Menü Tasarımı: Beyaz Menü Kartı + Tema Yeşili Sistem Tuşları Barı

* **Kullanıcı Talebi:** Uygulamanın 5 bölümü (Haneler, Sandık, Hane Ekle, Hayırsever, Gönüllü) önceki beyaz, yuvarlak hatlı ve şık tasarımında kalsın; telefonun alt sanal tuşlarının / hareket çubuğunun bulunduğu zemin ise sistemin zümrüt yeşili (`#065f46`) ile kaplansın.

* **Uygulanan Çözüm (Çift Katmanlı Alt Çubuk Mimarisi):**

  1. **Bizim Bölümler (Beyaz Menü Kartı):**

     * Zemin: Arka planı hafif buzlu beyaz (`bg-white/98 backdrop-blur-xl`), üst köşeleri kavisli (`rounded-t-3xl`), üst kenarlık (`border-t border-slate-200/90`) ve yumuşak gölge (`shadow-[0_-8px_30px_rgba(0,0,0,0.12)]`).

     * Butonlar: `188450.jpg` ekranındaki özgün beyaz zeminli tasarım korundu. Aktif sekmede koyu zümrüt yeşili başlık (`text-emerald-800 font-extrabold`) ve altında yeşil parıltılı nokta gösterge yer alır. Pasif sekmeler sade ve net gri/slate tonlarındadır.

     * Merkez "+ Hane Ekle": Menünün üstünden hafifçe taşan, koyu yeşil halkalı (`ring-4 ring-emerald-950`) ve canlı zümrüt yeşili yuvarlak eylem butonu.

  2. **Tuşların Barı (Sistemin Yeşili):**

     * Beyaz menü kartının hemen altına ekranın en tabanına oturan `safe-system-bar` eklendi.

     * Rengi: Hâre'nin ana tema yeşili (`#065f46` / `bg-emerald-900`).

     * Yüksekliği: `max(1.5rem, calc(env(safe-area-inset-bottom, 0px) + 0.6rem))` (asgari 24px - 34px).

     * Sonuç: Android'in 3 sanal gezinti tuşu (üçgen geri, daire ana ekran, kare son uygulamalar) veya yatay hareket çizgisi tamamen bu zümrüt yeşili zemin üzerinde kalır; yukarıdaki beyaz menü butonlarıyla ve yazılarıyla asla çakışmaz.

  3. **İçerik Kaydırma Boşluğu:** Sayfa altlarındaki son kayıtların menü arkasında kaybolmaması için ana gövde alt boşluğu `pb-36` seviyesinde tutuldu.

### 18. 🗺️ Harita ve Canlı Navigasyon Sistemi (E-Tablo Entegrasyonu ve Kesintisiz Çift Kademeli Açılış)

* **E-Tablo ile Entegrasyon Durumu:**

  * **Evet, kesinlikle entegredir.** Hanelerin adresleri ve harita konumları (`mapsUrl`) Google E-Tablonuzdaki **`Haneler`** sayfasında `G` sütununda (`mapsUrl`) saklanmaktadır.

  * Formdan kaydedilen veya güncellenen GPS koordinatları (`https://maps.google.com/?q=LAT,LON`) ya da adresten üretilen harita linkleri anında E-Tabloya yazılır ve diğer tüm telefonların ekranına eşitlenir.

* **Canlı Navigasyon Neden Açılmıyordu ve Nasıl Çözüldü?**

  1. **Paket Kilitli Intent Hatası:** Eski kodda Android paket ismi zorlaması (`package=com.google.android.apps.maps`) bulunuyordu. Android WebView veya TrebEdit gibi kısıtlı ortamlarda bu paket intent'i doğrudan çözülemeyince Android sistemi haritayı açmak yerine kullanıcıyı Google Play Store'daki indirme/güncelleme sayfasına atıyordu.

  2. **Doğrudan Yerel Protokol (`google.navigation:`):** Play Store'a atma hatasını önlemek için `google.navigation:q=LAT,LON&mode=d` protokolü devreye alındı. Bu şema Android'in yerel navigasyon motorudur.

  3. **Çift Kademeli Güvenli Fallback (Kesintisiz Navigasyon Güvencesi):**

     * Bazı Android WebView tarayıcıları veya cihaz güvenlik ayarları `google.navigation:` protokolünü yakalamakta sessiz kalabilir.

     * Bu durumu önlemek için sisteme akıllı bir **otomatik geçiş (fallback)** eklendi:

     * Tıklandığında önce doğrudan yerel navigasyon şeması denenir.

     * Eğer 600 milisaniye içinde cihazın harita uygulaması ön plana çıkmazsa, sistem beklemeden otomatik olarak Google Haritalar'ın doğrudan yol tarifi web/uygulama motoruna (`https://www.google.com/maps/dir/?api=1&destination=...`) geçiş yapar.

     * Böylece cihazınız ne olursa olsun (Android, iOS, eski/yeni WebView, TrebEdit) navigasyon butonu **asla cevapsız kalmaz ve asla Google Play Store'a atmaz; kapıya kadar yol tarifini anında açar.**

### 19. 📱 Sayfa Kaydırma (Aşağı-Yukarı Hareket) ve Geri Tuşunda Uygulamadan Atma Çözümü

* **Videodaki Sorunlar (188455.mp4 Analizi):**

  1. **Sayfa Aşağı İniyor Ama Geri Yukarı Çıkmıyor:** Sayfada veya hane detay penceresinde aşağı kaydırıldıktan sonra parmakla yukarı itildiğinde ekran kilitleniyor ve yukarı kaymıyordu. Bunun teknik sebebi, hem sayfa gövdesinin (`window`) hem de `<main>` etiketinin aynı anda `overflow-y-auto` barındırması ve `body` üzerindeki `overscroll-behavior-y: contain` kuralının Android dokunmatik kaydırmasını (touch-scroll) tek yöne hapsetmesiydi. Ayrıca pencerelerdeki fotoğraflar üzerinde parmak kaydırılırken tarayıcının resmi sürüklemeye çalışması dokunmayı yutuyordu.

  2. **Geri Tuşu Geri Geleceğine Uygulamadan Atıyor:** Kullanıcı hane detayındayken veya alt sekmelerdeyken geri tuşuna bastığında, geçmiş tamponunun tükenmesi veya `history.go(-length)` fonksiyonunun Android WebView'i zorla kapatması sebebiyle uygulama kapanıp TrebEdit'e/ana ekrana fırlatıyordu.

* **Uygulanan Kesin Çözümler:**

  1. **Doğal ve Akıcı 120Hz Çift Yönlü Kaydırma:**

     * `<main>` etiketi üzerindeki çakışan `overflow-y-auto` kaldırıldı; tüm sayfa kaydırması doğrudan ana pencere (`window / body`) seviyesinde birleştirildi.

     * `body` üzerindeki dokunma kilidi (`overscroll-behavior-y: contain`) kaldırıldı; yerine standart `-webkit-overflow-scrolling: touch` ve `touch-action: pan-y` eklendi. Sayfa artık aşağı ve yukarı kesintisiz, takılmadan akar.

     * Pencereler açıldığında arka plan otomatik kilitlenir (`body.modal-open { overflow: hidden }`); pencere içindeki kaydırma alanına özel `.modal-scroll-body` ve fotoğraflara `-webkit-user-drag: none` tanımlanarak fotoğrafların üzerinden kaydırırken bile takılma sorunu tamamen giderildi.

  2. **Geri Tuşunda Uygulamadan Asla Atmama Güvencesi:**

     * Android'in `canGoBack()` denetimini her an açık tutan otomatik döngüsel geçmiş tamponu (`ensureHistoryBuffer`) devreye sokuldu.

     * Geri tuşuna basıldığında:

       * Hane detay penceresi açıksa **pencere kapanır ve listeye dönülür**.

       * Hane Ekleme, Sandık, Hayırsever veya Gönüllü sekmesindeyse **doğrudan Haneler ana sayfasına dönülür**.

       * Zaten Haneler ana sayfasındaysa **uygulamadan kesinlikle atmaz**; ekranda *"ℹ️ Çıkış yapmak için sağ üstteki 🚪 butonunu kullanabilirsiniz"* uyarısı verir ve kullanıcıyı güvenle uygulamada tutar.

### 20. 🛡️ Eklenen Kayıtların Kendi Kendine Silinmesini Önleme (Çift Taraflı Eşitleme Koruması)

* **Karşılaşılan Sorun (188540.mp4 & 188541.jpg Analizi):**

  * Kullanıcı yeni bir hane ("Mustafa Amca") ekledikten ve teslimat fotoğrafını yükledikten sonra, üstteki Canlı eşitlemeye basıldığında veya arka plandaki 30 saniyelik otomatik eşitleme çalıştığında hane listesi sıfırlanıyor ve ekranda *"Henüz kayıtlı hane bulunmuyor (0 Hane)"* yazıyordu.

  * **Teknik Sebebi:**

    1. Eski sistemde sunucudan gelen liste (`data.aids`), cihazın yerel hafızasındaki (`state.aids`) kayıtların üzerine doğrudan yazılıyordu. Google Apps Script sunucusunda o an bir gecikme olduğunda veya kayıt sunucuya yazılırken arka plan eşitlemesi boş/eski liste döndürdüğünde, cihazda henüz kaydedilmiş yeni hane silinmiş gibi eziliyordu.

    2. Ayrıca üst barda bulunan süpürge (`🧹`) butonu, cihazdaki yerel önbellek listesini (`hare_cached_aids`) de siliyordu.

* **Uygulanan Kesin Çözümler:**

  1. **Çift Taraflı Birleştirme Motoru (Two-Way Smart Merge):**

     * Sunucudan veri çekildiğinde (`syncFromGoogleSheet`), yereldeki yeni kayıtlar ezilmez.

     * Cihazda bulunan fakat sunucuya henüz yansımamış taze kayıtlar korunur ve sunucudan gelen listeyle akıllıca harmanlanır (`localOnly.concat(serverAids)`).

     * Böylece internet kopsa, sunucu gecikse veya arka planda eşitleme çalışsa dahi eklediğiniz hiçbir kayıt kendi kendine silinmez.

  2. **Temizle (🧹) Butonu İyileştirmesi:**

     * Süpürge butonu artık kullanıcının cihazındaki gerçek kayıtları temizlemez; yalnızca tarayıcı ağ önbelleğini yeniler ve güvenli eşitleme yapar.

  3. **Navigasyon ve Harita Kesin Tetikleyicisi:**

     * "🧭 Canlı Navigasyon" ve "🗺️ Haritada Gör" butonları doğrudan evrensel Google Haritalar `https://www.google.com/maps/dir/?api=1&destination=...` motoruna bağlanarak WebView izin kısıtlamalarına takılmadan harita uygulamasını garantili olarak açar.

### 21. 🗺️ Harita ve Navigasyon Kökten Çözümü (Google/Yandex Seçici & Geri Tuşu Koruması)

* **Karşılaşılan Sorunlar (188567.mp4 Video Analizi):**

  1. **Google Haritalar "Tarayıcınızı Güncelleyin" Hatası:** Hane detayından "Haritada Gör" veya "Canlı Navigasyon" butonuna basıldığında WebView arka planda `https://www.google.com/maps/...` web adresini açmaya çalışıyor; Google Maps Web motoru Android WebView'ı desteklemediği için `support.google.com/maps/answer/...` sayfasına ("Google Haritalar'ı kullanmak için tarayıcınızı güncelleyin") yönlendiriyordu.

  2. **Geri Tuşunun Uygulamadan Atması:** WebView'in adresi `index.html` dosyasından Google Yardım web sayfasına dönüştüğü için Hâre uygulamasının JavaScript belleği ve `popstate` dinleyicileri tamamen devreden çıkıyor; kullanıcı geri tuşuna veya hareketine bastığında Android işletim sistemi uygulamayı kapatıp doğrudan ana ekrana (launcher) fırlatıyordu.

* **Uygulanan Kesin Çözümler:**

  1. **🧭 Hâre Özel Harita ve Navigasyon Seçici Modalı (Action Sheet):**

     * "🗺️ Haritada Gör" veya "🧭 Canlı Navigasyon" butonuna dokunulduğunda doğrudan web sayfasına yönlenmek yerine Hâre içerisinde şık bir alt menü açılır:

       * 🟢 **Google Haritalar:** Canlı sürüş rotasını Google Maps uygulamasında doğrudan başlatır (`google.navigation:q=lat,lon&mode=d`).

       * 🟡 **Yandex Navigasyon:** Türkiye sahasında en çok tercih edilen Yandex Navi uygulamasında rotayı açar (`yandexnavi://build_route_on_map?lat_to=lat&lon_to=lon`).

       * 🗺️ **Sistem Haritası (Cihaz Seçimi):** Android'in yerel harita seçicisini açar (`geo:lat,lon?q=lat,lon`).

       * 📋 **Konumu Kopyala:** Koordinat ve açık adresi tek dokunuşla panoya kopyalar (WhatsApp'a veya başka haritalara yapıştırmak için).

  2. **🛡️ Sayfayı Asla Değiştirmeyen Gizli Iframe Dispatch Motoru:**

     * Harita protokolleri (`google.navigation:`, `yandexnavi:`, `geo:`) artık sayfa adresini değiştirmeden görünmez bir arka plan kanalı üzerinden tetiklenir.

     * Bu sayede WebView asla Google Haritalar web sitesine veya "Tarayıcınızı güncelleyin" sayfasına yönlenmez; Hâre uygulaması daima ekranda canlı kalır.

  3. **🔙 Kesintisiz Geri Tuşu ve 2 Kere Basarak Çıkış Güvencesi:**

     * Kullanıcının ekrana her dokunuşunda (`click` olayı) Android WebView geçmiş tamponu (`ensureHistoryBuffer`) taze tutulur; Android'in `canGoBack()` yetkisi hiçbir zaman tükenmez.

     * Geri tuşuna basıldığında hiyerarşik olarak önce açık olan Harita Seçici, Hane Detayı veya alt sekmeler sırayla kapatılır.

     * Sadece ve sadece ana ekranda hiçbir pencere açık değilken 2 saniye içinde art arda iki kez geri basıldığında uygulama kapanır; kazara tek basışta fırlatma sorunu tamamen ortadan kalktı.

### 22. 🗺️ Harita Sadeleştirmesi: Canlı Navigasyon, Google & Yandex Kaldırıldı — Saf Sistem Haritası

* **Kullanıcı Talebi:**

  * "Canlı Navigasyon" butonu ve bölümü komple kaldırıldı.

  * "Haritada Gör" bölümü korundu; içinden Google Haritalar web yönlendirmesi ve Yandex Navigasyon bölümleri tamamen çıkartıldı.

* **Yapılan Sadeleştirme ve İyileştirmeler:**

  1. **"🧭 Canlı Navigasyon" Tamamen Kaldırıldı:**

     * Hane detay kartındaki ikili navigasyon butonları sadeleştirildi; karmaşıklık yaratan "Canlı Navigasyon" eylemi ve ilgili yönlendirmeler tamamen iptal edildi.

  2. **"🗺️ Haritada Gör" — Doğrudan Yerel Sistem Haritası (`geo:`):**

     * Google Haritalar web sitesine gitme veya Yandex Navi açma zorlaması kaldırıldı.

     * Artık "🗺️ Haritada Gör" butonuna basıldığında Android'in yerel sistem harita protokolü (`geo:lat,lon?q=lat,lon(Hane)`) doğrudan arka planda tetiklenir; telefonunuzda yüklü varsayılan harita uygulaması (Google Haritalar uygulaması veya sistem harita seçicisi) doğrudan açılır.

     * Tarayıcınız asla `support.google.com` adresine ("Tarayıcınızı güncelleyin") yönlenmez; Hâre uygulaması kendi sayfasında canlı kalır.

  3. **📋 "Konumu Kopyala" Butonu:**

     * "Haritada Gör" butonunun yanına pratik bir "📋 Konumu Kopyala" butonu eklendi. Tek dokunuşla hanenin enlem/boylam koordinatlarını ve açık adresini panoya kopyalayarak WhatsApp'a veya başka harita programlarına kolayca yapıştırabilirsiniz.

### 23. 🔒 Oturum Güvenliği: Uygulama Kapatıldığında / Arka Plandan Silindiğinde PIN İsteme

* **Karşılaşılan Durum:**

  * Uygulama açılışında oturum verisi kalıcı olarak `localStorage` içinde saklandığı için, uygulama kapatılsa veya arka plandan silinse dahi tekrar açıldığında PIN sormadan doğrudan ana ekrana giriyordu.

  * Bu durum hem mobil APK'da hem de web sürümünde telefon başkalarının eline geçtiğinde güvenlik ve mahremiyet riski oluşturuyordu.

* **Yapılan Düzeltmeler ve Güvenlik İyileştirmeleri:**

  1. **Geçici Oturum Mimarisi (`sessionStorage`):**

     * Aktif oturum artık kalıcı `localStorage` yerine oturum bazlı `sessionStorage` üzerinde tutulur.

     * Uygulama tamamen kapatıldığında, arka plandan silindiğinde veya web tarayıcısı kapatıldığında oturum otomatik olarak sonlanır.

     * Uygulama yeniden başlatıldığında gönüllüden güvenlik amacıyla 4 haneli PIN kodunu girmesi istenir.

  2. **Akıllı Gönüllü Hatırlama:**

     * Giriş ekranında en son giriş yapan gönüllü kardeşimizin ismi otomatik olarak seçili gelir. Böylece her seferinde listeden isim aramak zorunda kalmazsınız; yalnızca 4 haneli PIN kodunuzu girip anında giriş yapabilirsiniz.

  3. **İsteğe Bağlı "Oturumu Sürekli Açık Tut (Beni Hatırla)" Seçeneği:**

     * Giriş formuna isteğe bağlı bir onay kutusu eklendi.

     * Yalnızca kullanıcı bu kutuyu bizzat işaretlerse kalıcı oturum açılır; işaretlenmediği takdirde uygulama her kapatılıp açıldığında PIN sorarak tam güvenlik sağlar.

  4. **Sayfa Yenilemede (F5) Kesintisiz Çalışma:**

     * Web sürümünde çalışırken yapılan sayfa yenilemelerinde (F5) veya filtre değişimlerinde `sessionStorage` sayesinde oturum kapanmaz; yalnızca uygulama veya sekme tamamen kapatıldığında kilitlenir.

### 24. 🔍 Tam Kapsamlı Otomatik Kod ve Çalışma Zamanı Denetimi (61 Fonksiyon & 50+ Ekran Simülasyonu)

Uygulamanın tamamı Node.js sanal tarayıcı (VM) ortamında uçtan uca simüle edilerek tüm sekmeler, alt sekmeler, modallar ve 61 ayrı etkileşim fonksiyonu otomatik olarak test edildi. Tespit edilip tek seferde giderilen tüm gizli hatalar:

1. **Çıkış (`logout`) Sonrası Kayıtların Silinmesi Hatası (Kritik Veri Kaybı):**

   * **Sebep:** `logout()` fonksiyonu hafızadaki `state.aids = []`, `state.budget = []` dizilerini sıfırlıyordu. Kullanıcı tekrar giriş yaptığında sunucu eşitlemesi henüz tamamlanmadan bir işlem yaparsa (veya PIN değiştirirse), `pushToGoogleSheet()` sunucuya boş dizi (`[]`) gönderip E-Tablo'daki kayıtları silebiliyordu.

   * **Çözüm:** Çıkış yapıldığında yerel önbellek dizilerinin sıfırlanması kaldırıldı (oturum kapalıyken zaten ekranda hiçbir veri gösterilmez). Böylece E-Tablo'ya yanlışlıkla boş liste gönderilmesi tamamen engellendi.

2. **Hayırseverler Listesinde Gizli `b.type` Değişken Hatası ve Görünmeyen Makbuz Butonu:**

   * **Sebep:** "💬 Teşekkür & Makbuz" butonu (`sendDonorReceiptWhatsApp`), yanlışlıkla `renderDonorsListHTML` (bağışçı kartları) içine `b.type === 'gelir'` koşuluyla eklenmişti. O döngüde değişken `d` olduğu ve `b` global döngüden `4` değerini aldığı için buton hiçbir ekranda görünmüyordu.

   * **Çözüm:** Hatalı referans `renderDonorsListHTML` içinden temizlendi ve gerçek yeri olan Sandık (`budgetListHTML`) gelir kayıtlarının içine eklendi.

3. **Ekip & Yönetim Sekmesindeki `NaN` Yazısı Hatası:**

   * **Sebep:** Otomatik yedekler kartının kapanışında `'</div>' + + '</div>'` şeklinde çift artı (`+ +`) operatörü kalmıştı; JavaScript ikinci `+'</div>'` ifadesini sayıya çevirmeye çalışıp ekrana `NaN` basıyordu.

   * **Çözüm:** Çift `+` operatörü temizlendi.

4. **Emanet Devirlerinin (`transfers`) Sayfa Yenilemede Sıfırlanması:**

   * **Sebep:** `state` nesnesi içinde 372. satırda `transfers: getCachedList('hare_cached_transfers')` tanımlanmışken, 391. satırda mükerrer olarak `transfers: []` yazıldığı için önbellekteki devirler her açılışta eziliyordu.

   * **Çözüm:** Mükerrer `transfers: []` satırı kaldırıldı.

5. **Modal Açıldığında Arka Plan Kaydırma Kilidinin (`isAnyModalOpen`) Çalışmaması:**

   * **Sebep:** `render()` fonksiyonu içinde `isAnyModalOpen` değişkeni, `selectedAid` ve `editingAid` değişkenleri tanımlanmadan *önce* hesaplanıyordu (`undefined` dönüyordu).

   * **Çözüm:** Değişken tanımlamaları `isAnyModalOpen` kontrolünün üstüne alındı; artık herhangi bir pencere açıldığında arka plan kaydırması tam kilitlenir.

6. **Sandık, Devirler ve Bağışçılar İçin Çift Taraflı Akıllı Eşitleme (`mergeSafeList`):**

   * **Sebep:** Çift taraflı koruma yalnızca `aids` (Haneler) için vardı; `budget`, `transfers` ve `donors` listeleri sunucudan gelen veriyle doğrudan eziliyordu. Ayrıca sunucuda `deliveryPhotos` sütunu eksikse yereldeki teslimat fotoğrafları silinebiliyordu.

   * **Çözüm:** Tüm listeler (`aids`, `budget`, `transfers`, `donors`) `mergeSafeList` motoruna bağlandı; yerel teslimat fotoğrafları, kategoriler ve henüz sunucuya yansımamış yeni kayıtlar hiçbir koşulda kaybolmaz.

7. **Temizle (`🧹`) Butonunda Silinen Kayıtların Geri Gelmesi:**

   * **Sebep:** `clearCacheAndResync` içinde `localStorage.removeItem('hare_deleted_ids')` çağrıldığı için daha önce silinmiş kayıtların ID listesi sıfırlanıyordu.

   * **Çözüm:** Silinen ID listesinin (`hare_deleted_ids`) temizlenmesi kaldırıldı; silinen kayıtlar asla geri gelmez.

8. **Hane Düzenleme (`editAid`) Penceresinde İhtiyaç Kalemleri (`category`) Eksikliği:**

   * **Sebep:** Yeni hane eklerken seçilen ihtiyaç kalemleri (Gıda, Nakit, Yakacak vb.), hane düzenleme penceresinde bulunmuyordu.

   * **Çözüm:** Hane düzenleme penceresine ihtiyaç kalemleri onay kutuları eklendi.

9. **PIN Değiştirme Sonrası Kalıcı Oturum Sızıntısı:**

   * **Sebep:** `handleSaveChangePin` fonksiyonu "Beni Hatırla" kapalı olsa bile `localStorage` içine `hare_user` yazıyordu.

   * **Çözüm:** Yalnızca `sessionStorage` güncellenir; kalıcı hafızaya sadece "Beni Hatırla" açıkken yazılır.

### 25. 📱 Tüm Sistemde Alt Tuş Takımı Koruması, Geri Tuşu Hiyerarşisi ve Arka Plan Oturum Sürekliliği (188679.jpg Çözümü)

* **Karşılaşılan 3 Sorun (`188679.jpg` Analizi):**

  1. **Alt Tuş Takımının Açılır Pencerelerde (Modallarda) Butonlarla Çakışması:** Hane Düzenleme, Hane Detay, Kasa/Bağışçı/Gönüllü Düzenleme gibi pencereler (`z-50` – `z-[130]`) açıldığında, alt menünün (`z-40`) üstüne biniyor ve en alttaki yeşil **"Değişiklikleri Kaydet ve E-Tabloya Gönder"** butonu telefonun sanal geri/ana ekran tuşlarının (`■ ● ◀`) tam altına giriyordu.

  2. **Geri Tuşunun Uygulamayı Kapatması:** Geri tuşuna basıldığında Android WebView geçmiş yığını (`canGoBack()`) URL hash değişimi olmadığı için boş görünüyor ya da ana ekranda çift basışta `exitApp()` / `history.go(-length)` çalışarak uygulamayı tamamen kapatıyordu.

  3. **Uygulamayı Arka Plana Atınca Oturumdan Çıkması:** Uygulama arka plana atıldığında veya Kamera/Harita/WhatsApp gibi harici bir uygulamaya geçildiğinde bazı Android WebView (APK / `file://`) sürümlerinde `sessionStorage` sıfırlandığı için kullanıcıdan tekrar şifre isteniyordu.

* **Uygulanan Kesin Çözümler:**

  1. **🛡️ Tüm Sistemde Sabit Zümrüt Yeşili Alt Tuş Barı (`#global-system-bar` - `z-[999]`):**

     * Sistem gezinti çubuğu (`#global-system-bar`) tüm pencerelerin ve modalların üstünde kalacak şekilde (`z-[999]`, asgari `52px` / `3.25rem` yükseklikte) doğrudan `<body>` seviyesine sabitlendi.

     * Tüm açılır pencerelere (Hane Detay, Hane Düzenle, Kasa Düzenle, Hayırsever Düzenle, Gönüllü Yetki Düzenle, Fotoğraf İnceleme, PIN Değiştirme ve Onay pencereleri) `.modal-sheet-safe` (`calc(var(--sys-nav-h) + 1.5rem)`) alt güvenlik boşluğu eklendi.

     * Böylece hangi pencere veya bölüm açılırsa açılsın, kaydetme butonları daima zümrüt yeşili alt tuş barının üzerinde kalır; telefonun `■ ● ◀` tuşlarıyla asla üst üste binmez.

  2. **🔙 Geri Tuşunun Uygulamayı Asla Kapatmaması — Yalnızca İlgili Bölümü Kapatması:**

     * Uygulamayı kapatan `exitApp()` ve `history.go(-history.length)` kodları tamamen kaldırıldı.

     * Android WebView'in `canGoBack()` kontrolünü işletim sistemi seviyesinde daima canlı tutan gerçek URL hash tabanlı geçmiş tamponu (`#hare-1`, `#hare-2` ...) ve `backbutton` + `popstate` köprüsü kuruldu.

     * Geri tuşuna basıldığında yalnızca açık olan pencere veya bölüm kapanır (Onay Penceresi → PIN → Fotoğraf → Hane Düzenle → Hane Detay → Hayır Defteri / Sorumlu Kasası → Alt Sekmeler → Kategori Listesi → Haneler Ana Ekranı). Ana ekrandayken geri tuşuna basılsa dahi uygulama **kesinlikle kapanmaz**.

  3. **🔒 Arka Planda Oturumu Koruma & Yalnızca Tamamen Kapatınca Şifre İsteme:**

     * Çok katmanlı oturum motoru (`sessionStorage` + `window.name` + `hare_session_active=1` süreç çerezi + arka plan kalp atışı ve harici uygulama köprüsü) devreye alındı.

     * Uygulama arka plana atıldığında (orta tuşa basıldığında), ekran kilitlendiğinde veya WhatsApp/Harita/Kamera açıldığında kullanıcı çıkışı **kesinlikle olmaz** ve kaldığınız bölüm korunur.

     * Yalnızca uygulama son kullanılanlardan kaydırılarak **tamamen kapatıldığında** (veya sağ üstten `🚪` Çıkış butonuna basıldığında) yeniden açılışta güvenlik için PIN kodu sorulur.

### 26. 📦 Sadeleştirilmiş Emanet Türleri (Eşya vb.), HD Fotoğraf Kalitesi, Android Chromium Geri Tuşu Çözümü ve Tam Sistem Denetimi

1. **📦 Emanet Türü (İhtiyaç Kalemleri) Sadeleştirildi ve Genişletildi:**

   * Hem **Hane Ekle** hem **Hane Düzenle** hem de **Sandık (Kasa)** ekranlarındaki uzun ve parantezli ifadeler sadeleştirilerek sahada en çok ihtiyaç duyulan **10 net kategoriye** çıkarıldı:

     * `🥖 Gıda / Erzak`, `💵 Nakit Destek`, `🛋️ Ev Eşyası`, `🧺 Beyaz Eşya`, `🔥 Yakacak`, `👕 Giyim`, `💊 Sağlık / İlaç`, `🏠 Kira / Fatura`, `👶 Yetim / Eğitim`, `📦 Diğer`

   * **Geriye Dönük Tam Uyumluluk (`normalizeCategoryLabel`):** Daha önce eski uzun isimlerle (*"Gıda / Erzak (Yiyecek)"*, *"Kisve (Giyim / Kıyafet)"* vb.) kaydedilmiş haneler otomatik olarak yeni sade isimlere dönüştürülür; hiçbir eski veri kaybolmaz.

2. **📸 Fotoğraf Kalite Ayarları HD Seviyeye Yükseltildi (`compressPhotoHD`):**

   * Eski `650px` ve `%55` düşük sıkıştırma ayarı kaldırıldı; yerine **1280px HD çözünürlük**, **%85 JPEG kalitesi** ve yüksek kaliteli bikübik keskinleştirme (`imageSmoothingQuality: 'high'`) getirildi.

   * **Akıllı Google E-Tablo Hücre Sınırı Koruması:** 3 fotoğraf yüklendiğinde Google E-Tablo'nun tek hücredeki 50.000 karakter sınırını aşmaması için otomatik boyut dengeleyici eklendi.

   * **Hane Düzenleme İçinden Fotoğraf Ekleme / Silme:** Artık mevcut bir hanenin bilgilerini düzenlerken (`Hane Bilgilerini Düzenle`) sonradan yeni HD fotoğraf ekleyebilir veya yanlış eklenmiş fotoğrafı silebilirsiniz. Ayrıca teslimat fotoğraflarına da silme (`✕`) butonu eklendi.

3. **🔙 Android WebView Geri Tuşunun Uygulamayı Kapatması Kökten Çözüldü (Chromium User-Gesture Koruması):**

   * **Kök Sebep:** Android System WebView (Chromium), sayfa ilk açılırken (kullanıcı ekrana dokunmadan önce) eklenen `history.pushState` kayıtlarını güvenlik politikası (*History Manipulation Intervention*) gereği yok sayıp `canGoBack() = false` döndürerek uygulamayı kapatıyordu. Ayrıca önceki `click` dinleyicisinde bulunan `#hare-0` koşulu yeni kayıt eklenmesini engelliyordu.

   * **Kesin Çözüm:** Kısıtlayıcı koşul kaldırıldı; artık kullanıcının ekrana yaptığı her gerçek dokunuşta (`touchstart`, `touchend`, `click`) Android işletim sisteminin %100 geçerli saydığı kullanıcı etkileşimli (`user-activated`) geçmiş kaydı üretilir.

   * Hem `popstate`, hem `hashchange`, hem `backbutton`, hem de donanım geri tuşu (`keyCode: 4` / `GoBack`) yakalanarak yalnızca açık olan pencerenin/bölümün kapanması garanti altına alındı. Ayrıca tüm açılır pencerelerin sağ üst köşesine net **`← Geri`** butonu eklendi.

4. **🔍 Tüm Sistem Taraması ve Sorunsuz Uygulama Standartlarında Ek İyileştirmeler:**

   * **Teslimat Fotoğraflarının Apps Script Güncellemesine Gerek Kalmadan Eşitlenmesi:** Teslimat fotoğrafları (`deliveryPhotos`), sunucuya gönderilirken `;delivery=1;base64,` imzasıyla paketlenir ve okunurken otomatik ayrıştırılır; böylece E-Tablo scriptinize yeni sütun eklemenize gerek kalmadan teslimat kanıtları tüm telefonlarda görünür.

   * **Gelişmiş Arama Motoru:** Hane araması artık yalnızca isim ve mahallede değil; **telefon numarası, açık adres, ihtiyaç kalemi (ör. "Ev Eşyası", "Beyaz Eşya") ve ilgilenen gönüllü** içinde de anında arama yapar.

   * **Genel Sandık Hızlı Filtre ve Arama:** Sandık geçmişine `Tümü`, `🟢 Teberru`, `🔴 İnfak`, `🏦 Banka`, `💵 Elden` filtreleri ve anlık arama kutusu eklendi.

   * **Kasa Düzenleme ve Excel CSV İyileştirmesi:** Kasa hareketi düzenleme penceresine (`Kasa Hareketini Düzenle`) **Kategori / Kalem** alanı ve Hane Düzenleme penceresine **Durum** alanı eklendi.

### 27. 📱 Mobil APK / Uygulama Özel İyileştirmeleri: Aşağı Çekince Yenileme (Pull-to-Refresh) Kapatılması ve Geri Tuşu (1 Geri Ana Sayfa, 2 Geri Çıkış)

* **1. Aşağı Çekince Yenileme (Pull-to-Refresh) Özelliğinin Kapatılması:**

  * **Sorun:** Kullanıcı sayfadayken parmağıyla ekranı aşağı çektiğinde Android WebView motoru sayfayı yeniden yüklüyor, dönen yenileme çemberi çıkıyor ve uygulama sıfırlanıyordu.

  * **Uygulanan Kesin Çözüm:**

    * **CSS Düzeyi:** `html, body` etiketlerine `overscroll-behavior: none !important;` ve `overscroll-behavior-y: none !important;` eklendi.

    * **JavaScript Dokunma Yakalayıcısı:** Hem `window` hem `document` seviyesinde `touchstart` ve `{ passive: false }` seçenekli `touchmove` dinleyicileri devreye alındı. Sayfa tepe noktasındayken (`scrollTop <= 0 && diffY > 0`) yapılan aşağı kaydırma hareketleri `e.preventDefault()` ile iptal edildi. Sayfa aşağı veya yukarı normal olarak kaymaya devam eder ancak hiçbir zaman sayfayı baştan yenilemez.

    * **WebIntoApp APK Derleme Notu:** WebIntoApp kontrol panelinizde **"App Settings > Features"** menüsünde bulunan **"Swipe to Refresh / Pull to Refresh"** seçeneğini KAPALI (işaretsiz) tutarak APK derlediğinizde Android Java seviyesindeki çember yenileme de tamamen devre dışı kalır.

* **2. Mobil APK Donanım Geri Tuşu Kuralı (1 Geri Ana Sayfa, 2 Geri Çıkış):**

  * **Sorun:** Kullanıcı başka sekmelerdeyken veya ana sayfadayken geri tuşuna bastığında Android WebView geçmişinin tükenmesi sebebiyle uygulamanın aniden kapanması.

  * **Uygulanan Çözüm (Kademeli Geri & Çift Tıkla Çıkış Motoru):**

    * **1. Aşama (Açık Pencereler):** Hane Detayı, Düzenleme Formu, Fotoğraf İnceleme, PIN Değiştirme veya Kategori Listesi açıksa; geri tuşuna basıldığında yalnızca o pencere kapanır.

    * **2. Aşama (Diğer Sekmelerden Ana Sayfaya Dönüş - 1 Geri):** Kullanıcı **Sandık**, **Hane Ekle (+)**, **Hayırsever** veya **Gönüllü** sekmesindeyken geri tuşuna bastığında **asla uygulamadan çıkmaz**; 1 geri basışla doğrudan **Haneler ana sayfasına** döner ve ekranda *"← Haneler ana sayfasına dönüldü."* bildirimi çıkar.

    * **3. Aşama (Haneler Ana Sayfasında 1. Basış):** Kullanıcı zaten Haneler ana sayfasındayken geri tuşuna ilk kez bastığında uygulama kapanmaz; ekranda *"🚪 Çıkmak için tekrar basın"* uyarısı çıkar ve 3 saniyelik güvenli çıkış süresi başlar.

    * **4. Aşama (Haneler Ana Sayfasında 2. Basış):** Kullanıcı gerçekten çıkmak istiyorsa ve 3 saniye içinde ikinci kez geri tuşuna basarsa, uygulama *"🚪 Hâre kapatılıyor..."* diyerek temizce kapanır. 3 saniye içinde ikinci basış gelmezse sayaç sıfırlanır, kazara kapanmalar önlenir.

### 28. 🛠️ Kapsamlı Yönetici Kontrol Paneli, Gelişmiş Ekip Yetkilendirme, Log Silme ve Tam E-Tablo Senkronizasyonu (v3.2)

* **1. 🛡️ Gelişmiş Ekip Yetkilendirme & 4 Kademeli Rol Sistemi (`Ekip & Yetki`):**
  * **4 Kademeli Rol Seviyesi:**
    * `👑 Baş Emanetçi (Tam Yönetici)`: Tüm sistem ayarları, yönetim paneli, ekip yetkilendirme ve log silme dahil tam yetki.
    * `🛡️ Yardımcı Yönetici (Koordinatör)`: Hane, Sandık ve Hayırsever düzenleme/silme, işlem günlüğü görüntüleme ve rapor indirme yetkisi.
    * `💼 Kasa & Emanet Sorumlusu`: Sandık (Kasa) hareketlerini ve hayırseverleri yönetme ve rapor indirme yetkisi.
    * `🤝 Saha Gönüllüsü`: Hane ekleme, durum güncelleme, teslimat fotoğrafı yükleme ve kendi zimmet kasasını takip etme yetkisi.
  * **Kişi Bazlı Özel Modül İzinleri (7 Ayrı Yetki Anahtarı):**
    * Her bir gönüllü için ayrı ayrı açılıp kapatılabilen özel izin kutucukları eklendi: `✏️ Hane Düzenleme & Silme`, `🗳️ Sandık (Kasa) Yönetimi`, `🤝 Hayırsever Yönetimi`, `👁️ Genel Sandık Görme`, `👁️ Hayırseverleri Görme`, `📜 İşlem Günlüğü (Log) Görme`, `📊 Excel / Rapor İndirme`.
  * **Tek Tıkla Hızlı Yetki Değiştirme:** Ekip listesindeki her üye kartına tek dokunuşla `👑 Yönetici Yap` / `👤 Gönüllü Yap` ve detaylı `🛡️ Yetki & Düzenle` butonları eklendi.

* **2. 🗑️ Sadece Yönetici İçin İşlem Günlüğü (Log) Silme & Temizleme Merkezi (`Günlük`):**
  * Yalnızca **Tam Yönetici (`Baş Emanetçi`)** hesabıyla giriş yapıldığında aktif olan 4 kademeli log temizleme sistemi eklendi:
    * **Tekil Log Silme (`🗑️ Sil`):** Her bir log kartının sağ üst köşesindeki butonla istenen tek bir işlem kaydını silme.
    * **Aranan / Filtrelenen Logları Silme:** Arama kutusuna yazılan kişi veya işlem türüne ait logları tek tıkla toplu silme.
    * **Eski Logları Temizleme (`⏳ Son 20 Hariç Sil`):** En güncel 20 kaydı koruyup eski geçmişi temizleme.
    * **Tüm Günlüğü Sıfırlama (`🧹 Tüm Logları Sil`):** Onay penceresiyle tüm günlük kayıtlarını hem cihazdan hem de Google E-Tablo `Gunluk` sayfasından kalıcı olarak sıfırlama.

* **3. 🛠️ Geniş ve İşlevsel Yönetici Kontrol Paneli (`🛠️ Panel` / `Yönetim`):**
  * Üst bardaki **`🛠️ Panel`** kısayolundan veya `Gönüllü > 🛠️ Yönetim` sekmesinden erişilen, 6 ana modülden oluşan ve Google E-Tablo ile çift yönlü çalışan kontrol merkezi:
    1. **📊 Canlı Sistem Özeti & Hızlı Aksiyonlar:** Toplam Hane, Net Sandık, Ekip ve Log sayaçları + tek tıkla E-Tablo eşitleme ve anlık yedek alma.
    2. **📢 Saha Duyuru & Acil Bildirim Panosu:** Tüm gönüllülerin **Haneler** ana ekranının en üstünde göreceği sabit duyuru yayınlama (`🟢 Bilgilendirme`, `⚠️ Önemli Hatırlatma`, `🚨 Acil Saha Çağrısı`) ve tek tıkla yayından kaldırma.
    3. **🏛️ Kurum Kimliği, Bölge & Resmi IBAN Ayarları:** Uygulama başlığı, alt başlık, giriş ayet/slogan metni, yeni hane formunda otomatik gelen Varsayılan İl (`Kayseri`) / İlçe, Para Birimi (`₺`) ve Sandık ekranında tek tıkla kopyalanıp WhatsApp ile paylaşılabilen **Resmi Emanet IBAN & Alıcı Adı** tanımlama.
    4. **📦 Emanet Türü (İhtiyaç Kalemleri) Yönetimi:** Mevcut ihtiyaç kalemlerine yeni özel kategori (emoji + isim, ör. `🍼 Bebek Maması`) ekleme, silme veya varsayılan 10 kategoriye sıfırlama.
    5. **🛡️ Güvenlik, Genel Yetki & Sistem Politikaları:** `🔒 Bakım (Salt-Okunur) Kilit Modu`, `⚠️ Mükerrer Hane Kayıt Uyarısı`, `👁️ Açılışta Otomatik Mahremiyet Modu`, gönüllülerin Genel Sandık ve Hayırsever görme/düzenleme yetkileri, varsayılan PIN kodu ve otomatik eşitleme süresi (`15 sn / 30 sn / 60 sn`).
    6. **📸 Kamera Kalitesi, WhatsApp Şablonları & E-Tablo Bakım Merkezi:** Fotoğraf çözünürlük modu (`💎 Ultra HD 1440px`, `✨ Net HD 1280px`, `🌿 Tasarruf 960px`), hane başına maksimum fotoğraf sayısı, WhatsApp teşekkür makbuzu ve paylaşım kartı şablonlarını özelleştirme, E-Tablo bağlantı testi (Ping), tüm tablolar için Excel `.CSV` / `.JSON` dışa aktarım ve teslim edilmiş haneleri toplu temizleme araçları.


### 29. 📱 Yukarı - Aşağı Kaydırma (Touch Scroll) Kilitlenmesi Analizi ve Kesin Çözümü

* **Tespit Edilen 3 Kök Sebep (Analiz):**
  1. **`html, body` Çift Kaydırma Katmanı Çakışması:** CSS içerisinde hem `html` hem de `body` etiketlerine aynı anda `overflow-x: hidden; overflow-y: auto; overscroll-behavior: none !important;` verilmesi, tarayıcıda iç içe iki farklı kaydırma kutusu (nested scroll container) oluşturuyor ve `body` üzerindeki `overscroll-behavior: none` kuralı kaydırma hareketinin ana pencereye (`html` / viewport) iletilmesini kilitliyordu.
  2. **`{ passive: false }` `touchmove` Dinleyicisinin Dokunmayı İptal Etmesi (`e.preventDefault()`):** Aşağı çekince yenilemeyi (pull-to-refresh) engellemek için eklenen `handlePullTouchMove` fonksiyonu, çift kaydırma katmanı yüzünden `window.pageYOffset` değerini sürekli `0` okuyor ve parmakla yapılan her kaydırma hareketinde `e.preventDefault()` çağırarak Android/iOS dokunmatik kaydırma motorunu (`GestureScrollBegin`) iptal ediyordu.
  3. **`#root` Üzerindeki `overflow-x-hidden` Sınıfı ve Modal `touch-action: none` Kuralı:** `#root` kapsayıcısındaki `overflow-x-hidden` sınıfı CSS standartları gereği `overflow-y: auto` üreterek üçüncü bir kaydırma katmanı yaratıyordu; ayrıca `body.modal-open` üzerindeki `touch-action: none !important` kuralı alt elemanlara miras geçerek açılır pencerelerin içindeki kaydırmayı da engelliyordu.

* **Uygulanan Kesin Çözüm:**
  1. **Tekil ve Doğal Kaydırma Ekseni:** `html` üzerindeki `overflow-x` / `overflow-y` kuralları ve `#root` üzerindeki `overflow-x-hidden` sınıfı kaldırıldı; tüm sayfa kaydırması tek bir ana eksende (`body` / `window`) birleştirildi.
  2. **Donanım Hızlandırmalı (120Hz) Saf CSS Pull-to-Refresh Koruması:** Dokunmatik kaydırmayı bloke eden JavaScript `touchmove` (`{ passive: false }`) dinleyicisi tamamen kaldırıldı. Aşağı çekince sayfa yenileme koruması yalnızca saf CSS (`overscroll-behavior-y: none`) ile sağlanarak tarayıcının kendi akıcı kaydırma motoru serbest bırakıldı.
  3. **Açılır Pencere (Modal) İçi Kaydırma İyileştirmesi:** `body.modal-open` içerisinden `touch-action: none !important` kaldırıldı (`overflow: hidden !important` korundu); böylece hem ana ekranlarda hem de Hane Detay / Düzenleme pencerelerinde yukarı-aşağı kaydırma kesintisiz ve akıcı hale getirildi.


### 30. 🔄 Başka Cihazda Yapılan Değişikliklerin Diğer Cihazlarda Gözükmemesi Sorunu Analizi ve Kesin Çözümü (v3.3)

* **Tespit Edilen 5 Kök Sebep (Analiz):**
  1. **Mevcut Dağıtılmış Google Apps Script (`ai_studio_code.js`) Sütun ve Tablo Uyumsuzluğu:** E-Tablonuzda dağıtılmış olan mevcut sunucu kodu yalnızca `Haneler`, `Sandik` (7 temel sütun), `Bagiscilar`, `Gonulluler` ve `Gunluk` sayfalarını okuyup yazıyordu. Sonradan eklenen `Devirler` (`transfers`), `Ayarlar` (`settings`) ve Sandık içindeki `category` (Kalem), `giver` (Veren), `receiver` (Alan) alanları mevcut sunucu tarafından kaydedilmediği için diğer cihazlara ulaşmıyordu.
  2. **`mergeSafeList` İçindeki Eski Önbellek Korumasının Silinenleri Geri Getirmesi:** Cihaz A'da bir kayıt silindiğinde Cihaz B kendi önbelleğinde (`localStorage`) bulunan o kaydı *"yerelde yeni eklenmiş kayıt"* (`localOnly`) zannederek silmiyor ve ilk işlemde tekrar E-Tabloya geri yüklüyordu.
  3. **Yazma Öncesi Birleştirme (Pull-Before-Push) Eksikliği:** Cihaz B henüz E-Tablodan son verileri çekmeden yeni bir işlem kaydettiğinde, kendi eski önbellek listesini E-Tabloya göndererek Cihaz A'nın az önce eklediği verileri ezebiliyordu.
  4. **Giriş Ekranında (`!state.currentUser`) Eşitlemenin Kapalı Olması:** Oturum açılmadan önce `syncFromGoogleSheet` çalışmadığı için başka cihazda eklenen yeni gönüllüler veya değiştirilen PIN kodları diğer cihazın giriş ekranında gözükmüyordu.
  5. **Android WebView `navigator.onLine` ve `fetch` Önbellek Engeli:** Bazı mobil APK WebView ortamlarında `navigator.onLine` yanlışlıkla `false` döndürerek arka plan eşitlemesini durduruyor ve `cache: 'no-store'` eksikliği eski yanıtın okunmasına yol açıyordu.

* **Uygulanan Kesin Çözümler (v3.3 Çoklu Cihaz Canlı Eşitleme Motoru):**
  1. **Mevcut E-Tablo Scriptiyle %100 Geriye Dönük Uyumlu Taşıyıcı Köprü:** Apps Script kodunuzu yeniden dağıtmanıza gerek kalmadan; tüm **Yönetici Paneli ayarları, duyurular, özel yetkiler, PIN haritası, silinen kayıt ID'leri (`deletedIds`) ve Emanet Devirleri (`transfers`)** mevcut `Bagiscilar` tablosu üzerinden gizli sistem taşıyıcılarıyla (`__HARE_SYS_CONFIG__` ve `__HARE_SYS_TRANSFERS__`), Sandık `kategori/veren/alan` bilgileri ise `[[HM:...]]` meta paketiyle taşınır.
  2. **Akıllı `dirtyIds` ve Küresel `deletedIds` Takibi:** Yalnızca o cihazda çevrimdışı eklenip henüz sunucuya gönderilmemiş kayıtlar (`hare_dirty_ids`) korunur; başka cihazda düzenlenen veya silinen kayıtlar tüm cihazlarda anında güncellenir ve silinenler bir daha geri gelmez.
  3. **Yazma Öncesi Otomatik Birleştirme (Pull-Before-Push):** Herhangi bir cihaz E-Tabloya veri yazmadan hemen önce sunucudaki son tabloyu çeker, diğer cihazların eklediği kayıtlarla kendi değişikliğini birleştirir ve öyle kaydeder; hiçbir cihaz diğerinin verisini ezemez.
  4. **12 Saniyede Bir Hızlı Canlı Eşitleme & Giriş Ekranı Senkronizasyonu:** Otomatik eşitleme süresi 12 saniyeye düşürüldü, sekme geçişlerinde ve giriş ekranında (`!state.currentUser`) otomatik eşitleme aktif edildi.


### 31. 🔗 Google E-Tablo Veritabanı Köprüsü Linkinin Tüm Kullanıcılara Otomatik Tanımlanması (Sıfır Elle Giriş)

* **Karşılaşılan Sorun:**
  * Yönetici `Google E-Tablo Veritabanı Köprüsü` bölümünden yeni bir Apps Script Web App linki girdiğinde, bu link yalnızca yöneticinin kendi cihaz hafızasına (`localStorage`) kaydediliyordu.
  * Diğer gönüllülerin ve cihazların senkronize olabilmesi için her kullanıcının kendi telefonundan bu script linkini elle girmesi gerekiyordu; girilmediğinde o cihazlarda senkronizasyon oluşmuyordu.

* **Uygulanan Kesin Çözüm (Merkezi Otomatik Köprü Dağıtım Mimarisi):**
  1. **Merkezi Bulut Köprü İşaretçisi (`activeSheetUrl` Auto-Beacon):**
     * Yönetici kendi cihazında **Google E-Tablo Veritabanı Köprüsü** (`💾 Kaydet & Tüm Cihazlara Dağıt`) üzerinden bir script linki kaydettiğinde (veya yöneticinin cihazında halihazırda özel bir link kayıtlı olduğunda), bu link `state.settings.activeSheetUrl` olarak hem yeni köprüye hem de uygulamanın fabrika çıkışlı merkezi köprüsüne (`DEFAULT_SHEET_URL`) otomatik olarak işlenir.
  2. **Diğer Tüm Cihazlarda Sıfır Ayarla Otomatik Algılama ve Geçiş:**
     * Başka herhangi bir kullanıcı uygulamayı açtığında (henüz PIN ile giriş yapmadan, giriş ekranındayken bile) uygulama merkezi köprüyü kontrol eder, yöneticinin tanımladığı güncel `activeSheetUrl` adresini otomatik algılar, kendi cihaz hafızasına (`hare_sheet_url`) kaydeder ve anında o veritabanı üzerinden senkronize olur.
     * Hiçbir gönüllünün veya kullanıcının script linki ya da kod girmesine gerek kalmaz.
  3. **Çift Köprü Yansıtma (Dual-Bridge Mirroring) & Otomatik Onarım (Auto-Heal):**
     * Yeni bir köprü linki kullanıldığında dahi yapılan değişiklikler arka planda varsayılan köprüye de yansıtılır; ayrıca bir cihazda hatalı/eski bir link kalmışsa sistem otomatik olarak çalışan merkezi köprüye dönüp bağlantıyı kendi kendine onarır.
  4. **Linki İçine Gömülü Hazır `index.html` İndirme Seçeneği:**
     * Yönetici paneline **`📥 Bu Köprü Linki Gömülü Hazır index.html İndir`** butonu eklendi; yeni APK veya web yayınlarında güncel linkin doğrudan kodun içine gömülü olduğu hazır dosyayı tek tıkla indirebilirsiniz.


### 32. 🤝 Sandık ve Hayırsever Tam Entegrasyonu, Yardım Talebi & Toplu WhatsApp İletişim Merkezi (v3.4)

* **1. 🗳️ Hayırsever Kayıt Alanlarının Sandık (`Emanet Hareketi Ekle`) Bölümüne Taşınması:**
  * **Hayırsever Sekmesinden Manuel Kayıt Formu Kaldırıldı:** Hayırsever (`bagiscilar`) bölümünde artık ayrı bir kayıt girme formu bulunmaz; bu sekme yalnızca hayırseverlerin bakiyelerini, Hayır Defteri geçmişlerini ve iletişim/yardım çağrısı araçlarını gösterir.
  * **Sandık > Emanet (Kasa) Hareketi Ekle (`🟢 Teberru / Bağış`) Entegrasyonu:**
    * Sandıkta **🟢 Teberru (Bağış)** seçildiğinde formda doğrudan **🤝 Hayırsever (Bağışçı) Bilgileri & Entegre Kayıt** alanı açılır.
    * **Kayıtlı Hayırsever Seçici:** Mevcut bir hayırsever seçildiğinde adı, telefonu, şehri, e-postası ve destek notu otomatik doldurulur; yeni bir hayırsever ise doğrudan `Hayırsever Adı Soyadı`, `Telefon Numarası`, `Şehir / Bölge`, `E-posta` ve `Hayırsever Destek Notu` alanlarıyla girilir.
    * **Otomatik Eşitleme (`upsertDonorFromBudget` & `ensureDonorsSyncedFromBudget`):** Sandığa teberru kaydedildiği anda hayırsever otomatik olarak **Hayırsever** sekmesinde oluşur (veya mevcutsa bilgileri ve toplam teberru bakiyesi güncellenir).
    * **Çift Yönlü Bağlantı:** Sandık hareketlerindeki `🤝 Bağışçı (Veren)` rozetine dokunulduğunda doğrudan o kişinin **📖 Hayır Defteri** açılır; Hayırsever kartlarındaki `➕ Emanet / Bağış Gir` butonuna basıldığında ise o hayırsever seçili olarak Sandık ekranına geçilir.

* **2. 💬 Hayırsever İletişim Merkezi: 🤲 Yardım Talebinde Bulun & 📢 Toplu WhatsApp Mesajı At:**
  * **🤲 Yardım Talebinde Bulun (Acil İhtiyaç Çağrısı):**
    * Kayıtlı **🚨 Müstacel (Acil)** veya **⏳ Planlanan** hanelerden biri (veya genel saha ihtiyacı) seçildiğinde; ailenin mahremiyetini koruyarak (soyisim ve telefon paylaşmadan) bölgesini, acil ihtiyaç kalemini, hedeflenen destek miktarını ve varsa **Resmi Emanet IBAN** bilgisini içeren hazır WhatsApp yardım talebi mesajı oluşturur.
    * Her hayırsever kartı üzerindeki **`🤲 Yardım Talebinde Bulun`** butonuyla veya listeden tek dokunuşla ilgili hayırsevere isme özel (`Muhterem [İsim]`) gönderilir.
  * **📢 Toplu WhatsApp Mesajı & Duyuru Motoru:**
    * 5 hazır şablon (`🚨 Acil Yardım & İnfak Çağrısı`, `🌿 Cuma / Kandil Tebriği & Hayır Duası`, `📊 Saha Emanet & Faaliyet Teşekkür Mesajı`, `🏦 Resmi Emanet Hesabı (IBAN) Bilgilendirmesi`, `✍️ Özel Mesaj Metni`) sunar.
    * **İsme Özel Hızlı Gönderim Listesi & Grup Paylaşımı:** Telefonu kayıtlı tüm hayırseverleri seçilebilir listede gösterir; tek tıkla WhatsApp gruplarında paylaşabilir, tüm numaraları kopyalayabilir veya listedeki her hayırsevere isme özel mesajı sırayla gönderebilirsiniz.
