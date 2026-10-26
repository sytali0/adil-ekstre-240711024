# Ürün Spesifikasyonu (SPEC) — AdilEkstre: Ekstre & Sözleşme Denetçisi (`adil-ekstre-240711024`)

## 1. Amaç ve Kapsam
Bireysel tüketicileri hem **sözleşme onaylamadan önce** hem de **ekstre kesildikten sonra** koruyan iki modlu Türkçe yapay zeka uygulaması:

1. **Ekstre & Abonelik Denetimi:** Kredi kartı ve banka ekstresi satırlarındaki **mükerrer (çift) çekimleri**, **habersiz abonelik zamlarını**, **ücretsiz deneme süresi tuzaklarını** ve **iptal sonrası tahsilatları** tespit eder.
2. **Sözleşme Haksız Şart Tarayıcı & Risk Skoru (Onay Öncesi Uyarı):** Kullanıcının onaylamadan önce (veya onayladıktan sonra) yapıştırdığı üyelik/abonelik sözleşmesi metnindeki **6502 sayılı TKHK Madde 5 kapsamında "Haksız Şart"** sayılan yasa dışı maddeleri (*tek taraflı zam hakkı, 14 günlük cayma hakkından feragat, yalnız noter/şube ile zorlaştırılmış iptal şartı, fahiş cezai şart, sorumsuzluk kaydı, Tüketici Hakem Heyeti başvuru kısıtı*) tespit eder; **0–100 arası Tüketici Risk Skoru** hesaplar, kritik maddelerde **"⛔ DİKKAT: Bu Sözleşmeyi Onaylamadan Önce Okuyun / Kesin Hükümsüz Madde"** uyarısı verir.
3. **Mevzuat Atıflı İtiraz & İhtar Dilekçesi Üretimi:** **6502 Sayılı Tüketicinin Korunması Hakkında Kanun**, **Tüketici Sözleşmelerindeki Haksız Şartlar Hakkında Yönetmelik**, **Abonelik Sözleşmeleri Yönetmeliği** ve **BDDK Harcama İtirazı (Chargeback)** rehberine (Hibrit RAG) dayanarak kaynak atıflı (`[1]`, `[2]`) analiz ve **insan onaylı resmi itiraz/ihtar dilekçesi taslağı** üretir.

**Kapsam Dışı:** Yatırım tavsiyesi, borsa/kripto para önerisi, kredi skorlama veya kullanıcı adına otomatik banka/mahkeme/e-Devlet başvurusu yapmak kapsam dışıdır.

## 2. Mimari ve Akış
1. **Girdi & KVKK Maskeleme (`kvkk.py`):** Kullanıcının girdiği ekstre veya sözleşme metnindeki TCKN, IBAN, 16 haneli kart numarası, telefon ve e-posta verileri modele gitmeden önce kurallı olarak maskelenir; prompt injection saldırıları filtrelenir.
2. **Kural Tabanlı Ön Analiz & Risk Skoru (`analiz_araclari.py`):**
   - Ekstrelerde mükerrer işlemler, yüksek oranlı zamlar (≥ %20) ve deneme/iptal sonrası çekimler deterministik Python fonksiyonlarıyla tespit edilir.
   - Sözleşme metinlerinde yasa dışı haksız şart kalıpları taranır ve hem ekstre hem sözleşme için **0–100 arası Tüketici Risk Skoru** ile **Onay Tavsiyesi (`ONAYLANABILIR` / `DIKKATLI_ONAYLA` / `ONAYLAMA_RISKLI`)** hesaplanır.
3. **Hibrit RAG (`rag.py`):** `data/derlem/` altındaki Türkçe tüketici mevzuatı belgeleri üzerinde BM25 + 3-gram vektör RRF hibrit araması yapılarak en ilgili mevzuat parçaları getirilir.
4. **Tipli Çıktı ve Doğrulama (`schemas.py` & `asistan.py`):** Sağlayıcıdan bağımsız LLM istemcisi (`LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL`) ile `EkstreDenetimRaporu` Pydantic şemasına uygun yanıt üretilir (şüpheli ekstre işlemleri + tespit edilen haksız sözleşme şartları + risk skoru + dilekçe taslağı) ve 4 doğrulayıcıdan geçirilir.
5. **İnsan Onayı (`app.py`):** Kullanıcıya kırmızı/sarı/yeşil risk uyarı kutusu gösterilir; üretilen dilekçe taslağı yalnızca kullanıcı inceleyip onay kutusunu işaretlediğinde indirilebilir.

## 3. Kabul Kriterleri
- `pytest -q` ile en az 12 ağsız birim testi (ekstre + sözleşme haksız şart + risk skoru + KVKK dahil) yeşil geçmeli.
- `python eval_calistir.py` ile 30 vakalık eval setinde en az 28/30 başarı sağlanmalı.
- `python guvenlik/kirmizi_takim.py` ile 10 saldırılık red team testinde `v2.txt` promptu 10/10 dayanmalı.
- `portfolyo_kontrol.py` skoru 10/10 olmalı.
