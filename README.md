# ⚙️ Kantitatif Karar Destek ve Otonom Portföy Mimarisi (V4.0 ➔ V5.8 PRO)

> **Gizlilik ve Fikri Mülkiyet Notu (Dijital Tapu):** Bu depo, **Abdüsselam Ekiciler (`@Apoxtr`)** tarafından kurgulanan, çoklu yapay zeka ajan (*Multi-Agent*) orkestrasyonu ve **ISO/IEC/IEEE 29148 (SRS) & IEEE 1016 (SDD)** standartlarıyla geliştirilen 6 modüllü çift motorlu (*Multi-Threaded*) karar destek sisteminin mimari evrim ve tasarım kayıtlarını içerir.

---

## 🧬 1. Neden Bu Mimari? (V4.0'dan V5.8 PRO'ya İteratif Mühendislik Evrimi)

Bu projenin ana odak noktası, yüksek frekanslı ve gürültülü veri akışlarında **yanlış pozitifleri (false-positives) sıfırlamak**, **asenkron çift motorlu bir mimariyle arayüz/komuta gecikmesini 0.5 saniyenin altına indirmek** ve **insan/sistem çevrimdışıyken oluşan durum kayıplarını otonom olarak kalibre etmektir.**

Sistem ilk kurulduğunda (`V4.0`) karşılaşılan darboğazlar ve bunları çözmek için geliştirilen mühendislik yamaları (`V5.8 PRO`):

| Mühendislik Problemi | Eski Yapı (`V4.0`) Darboğazı | Geliştirilen Çözüm (`V5.8 PRO` Mimarisi) |
| :--- | :--- | :--- |
| **Metin Madenciliği & Yanlış Pozitifler (False-Positive)** | İngilizce yaygın kelimeler (`near`, `link`, `render`, `apt`) yüzünden sıradan haberlerde yanlış kriz alarmı üretiliyordu. | **Bağlam Duyarlı İstihbarat (`medya.py`):** Tam proje kimlik eşleştirmesi, 80 karakterlik yakınlıkta kriz teyidi ve doğrudan tıklanabilir **Kaynak URL kanıtı** şartı ile yanlış pozitifler **%100** temizlendi. |
| **Gecikme & Thread Güvenliği (Concurrency)** | Arka plan taraması çalışırken kullanıcı komutları bekliyor veya API limitlerine takılıyordu. | **Çift Motorlu Asenkron Mimari (`tg.py`):** 7/24 çalışan *Daemon Thread* nöbetçi motoru ile kullanıcı komutlarına `threading.Lock()` ve `60sn Akıllı Önbellek (Cache)` zırhıyla **0.5 saniyede** yanıt veren *Ana Thread* ayrıştırıldı. |
| **Tek Boyutlu Veri Analizi** | Sadece 15 dakikalık kısa vadeli veriye ve yatay çizgilere bakılıyordu; tarihsel bağlam yoktu. | **6 Bölüm Formasyon & 1W Fraktal İkiz Motoru (`veri_ve_grafik.py`):** Çoklu zaman dilimi (`15m/1S/4S/1W`), 16 mum kalıbı, Harmonik XABCD ve **Pearson korelasyonuyla** geçmiş 52 haftadaki en benzer fiyat hareketini (`≥ %65` korelasyon) bulan tarihsel hafıza motoru kodlandı. |
| **İşlem Maliyeti Körlüğü** | Komisyon maliyetleri (`%0.20`) hesaba katılmadan erken çıkış sinyali üretilebiliyordu. | **Komisyon & Başabaş Kalkanı (`grafik_borsa.py`):** `Giriş × 1.0025` başabaş formülü ve anlamlı net sonuç (`≥ +%1.0`) oluşmadan erken çıkışı engelleyen koruma kalkanı eklendi. |
| **Statik Eşik ve Çevrimdışı Kaçırma Sorunu** | İlk belirlenen koruma noktası sabit kalıyor; kullanıcı uyurken veya sistem dışındayken hedef aşılırsa eski veride takılı kalıyordu. | **4 Kademeli Mandal (Ratchet) & Oto-Kalibrasyon (`grafik_borsa.py`):** İlerleme kaydettikçe koruma noktasını yalnızca yukarı kilitleyen 4 kademeli *Ratchet* algoritması, **Beklenen Değer (EV) Terazisi** ve uyuyakalma/kaçırma sonrası güncel desteklere göre rotayı kendi kendine onaran **Otomatik Yeniden Kalibrasyon** geliştirildi. |

---

## 🏗️ 2. Altı Modüllü Sistem İskeleti ve Katmanlı Mimari

Sistem, tek bir giriş noktasından (`python tg.py`) tetiklenen ve sorumlulukları kesin sınırlarla ayrılmış (**Separation of Concerns**) 6 çekirdek Python modülünden oluşur:

1. **`config.py` (Yapılandırma Katmanı — Bağımsız Kök Modül):**
   - Risk parametreleri, komisyon sabitleri ve 3 katmanlı tarama havuzu tanımlarını barındırır.
2. **`kara_kutu_db.py` (Kalıcılık / Persistence Katmanı — `sqlite3`):**
   - `ekip_kara_kutu.db` veritabanını ve 4 ana tabloyu (`kasa_durumu`, `istihbarat_panosu`, `grafik_haritasi`, `aktif_islemler`) ACID prensiplerine uygun olarak yönetir.
3. **`veri_ve_grafik.py` (Veri İşleme, Fraktal Hafıza ve Görsel Çizim Katmanı — `pandas`, `numpy`, `matplotlib`):**
   - Çoklu zaman dilimi uyumu, 6 bölümlü formasyon taraması, **Pearson korelasyonlu 1W Tarihsel İkiz Motoru** ve 4 bölmeli (`Agg` modunda thread-safe) görsel analiz grafiklerini üretir.
4. **`medya.py` (Bağlam Duyarlı ve URL Kanıtlı İstihbarat Katmanı — `re`, `requests`, `xml`):**
   - Küresel veri akışlarını tarar, kelime çakışmalarını 80 karakterlik pencere analiziyle filtreler ve her uyarıya doğrulama URL'si ekler.
5. **`grafik_borsa.py` (İş Mantığı, Risk Matematiği ve Oto-Kalibrasyon Katmanı):**
   - Güven skoru (`0-100`), Komisyon Kalkanı, 4 Kademeli Mandal (*Ratchet*) kilidi, Beklenen Değer (*Expected Value - EV*) terazisi ve çevrimdışı durumlar için 3 senaryolu otomatik yeniden kalibrasyon motorunu çalıştırır.
6. **`tg.py` (Çift Motorlu Orkestrasyon ve Sunum Katmanı — `threading`):**
   - **Motor 1 (Telegram Kulak Motoru):** Kullanıcı etkileşimlerine `0.5 sn` altında yanıt verir.
   - **Motor 2 (7/24 Otonom Nöbetçi - Daemon Thread):** Açık durumları ve veri akışlarını arka planda kesintisiz denetler.
   - **API Trafik Kilidi:** `threading.Lock()` ve `60sn Cache` ile yarış durumlarını (*race condition*) ve veri darboğazlarını önler.

---

## 🤖 3. Çoklu Yapay Zeka Ajan (Multi-Agent) Orkestrasyonu

Bu projenin mimari tasarım, kod denetimi ve dokümantasyon süreçlerinde; **Abdüsselam Ekiciler** tarafından her biri farklı bir yazılım mühendisliği disiplinine odaklanacak şekilde kurgulanan **Özel AI Ajan 롤leri** kullanılmıştır:
- **Mimari & Şema Ajanları:** Katmanlı mimari ve veri akış şemalarının IEEE 1016 standartlarında modellenmesi.
- **Kod Denetim & Risk Ajanları:** Thread-safety, bellek/önbellek yönetimi, sınır değer analizleri ve matematiksel doğrulama.
- **Veri, Grafik & İstihbarat Ajanları:** Bağlam duyarlı regex filtreleri, Pearson korelasyon pencereleri ve çoklu panel görselleştirme testleri.
