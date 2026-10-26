# AGENTS.md — Ajan Destekli Geliştirme Kuralları

Bu depoda kod yazan veya düzenleyen yapay zeka ajanları aşağıdaki mühendislik kurallarına uymak zorundadır:

1. **Sağlayıcıdan Bağımsızlık:** Hiçbir dosyada sabit API anahtarı veya tek bir sağlayıcıya özel SDK çağrısı bulunmamalıdır. Tüm çağrılar `LLM_BASE_URL`, `LLM_API_KEY` ve `LLM_MODEL` ortam değişkenlerini okuyan OpenAI uyumlu istemci üzerinden yapılmalıdır.
2. **Önce KVKK ve Güvenlik:** Kullanıcı girdisi (`ekstre`, `sozlesme` veya `soru`) LLM'e gönderilmeden önce mutlaka `kvkk.maskele()` fonksiyonundan geçirilmeli; TCKN, IBAN ve kart numaraları maskelenmelidir.
3. **Kural Tabanlı Yeten Yere Model Koyma:** Aritmetik fark, yüzde artış, mükerrer işlem tespiti, sözleşme haksız şart ön taraması ve 0–100 Tüketici Risk Skoru hesabı LLM'e bırakılmamalı; `analiz_araclari.py` içindeki deterministik fonksiyonlarla hesaplanmalıdır.
4. **Tipli Çıktılar ve Kaynak Sadakati:** Tüm rapor çıktıları `schemas.py` içindeki Pydantic modellerinden (`EkstreDenetimRaporu`) geçmeli ve tespit edilen her ekstre/sözleşme ihlali `data/derlem/` içindeki mevzuat parçalarına (`[1]`, `[2]`) atıf yapmalıdır.
5. **Test Zorunluluğu:** Her kod değişikliğinden sonra `pytest -q` ve `python eval_calistir.py --hakem-yok` çalıştırılmalı, kırılan test bırakılmamalıdır.
