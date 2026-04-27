# Manuel Kurulum Adımları

Bu doküman, Faz 1 + Faz 2 + Faz 3 değişikliklerinin **kod tarafı yapıldıktan sonra senin manuel olarak yapman gereken** adımları sırasıyla listeler.

> Tahmini süre: **10-15 dakika**

---

## Önkoşullar (Daha Önce Yapılmış Olmalı)

Bunlar zaten yapıldıysa atla:

- [ ] n8n lokalde çalışıyor (`http://localhost:5678`)
- [ ] Google Sheets, Gmail, Google Gemini (PaLM) için n8n credentials kurulu
- [ ] Mevcut Google Sheets dosyasında ana sayfa var (Email, Mesaj, Ozet, Duygu, AI Taslak, Yanit Durumu, Gonderilen Yanit kolonlarıyla)

---

## 1. Google Sheets Hazırlığı

### 1.1 Ana sayfaya 3 yeni kolon ekle

Mevcut ana sayfaya (içinde maillerin tutulduğu sayfa), **mevcut kolonların sağına** şu 3 başlığı ekle:

| Kolon Adı | Açıklama |
|-----------|----------|
| `Mesaj ID` | Gmail message ID (dedup için) |
| `Thread ID` | Gmail thread ID |
| `Tarih` | Mail tarihi (ISO format) |

> Tam başlık adlarını **birebir** kullan (büyük/küçük harf, boşluk dahil).

### 1.2 Yeni tab: `Ayarlar`

Sayfanın altındaki **"+"** butonuyla yeni bir sayfa ekle, adını **`Ayarlar`** yap.

1. satıra (başlık satırı) **soldan sağa** şu 7 başlığı yaz:

```
sirket_adi | urun_hizmet | calisma_saatleri | iade_politikasi | kargo_bilgisi | yanit_tonu | ek_bilgi
```

> Hepsi küçük harf, boşluk yok, alt çizgili. Veri satırı (2. satır) boş kalabilir — dashboard'dan dolduracağız.

### 1.3 Yeni tab: `SSS`

Aynı şekilde yeni sayfa ekle, adı **`SSS`**.

1. satıra şu 2 başlığı yaz:

```
Soru | Cevap
```

> İlk harfler büyük. 2. satır ve sonrası boş kalacak — dashboard'dan eklenecek.

---

## 2. n8n Workflow Güncellemesi

### 2.1 Workflow'u yeniden import et

1. n8n'de mevcut "My workflow"u aç
2. Sağ üstteki **"⋮"** menüsünden → **"Import from File..."** seç
3. `n8nworkflow.json` dosyasını seç
4. Soru gelirse **"Replace existing workflow"** seç (üzerine yaz)
5. Workflow yenilendi — yeni node'lar görünmeli (toplam **26 node**)

### 2.2 Sheets tab referanslarını kontrol et

Yeni workflow `Ayarlar` ve `SSS` tab'larına **isimle** erişiyor. n8n her node'da bu referansları doğrular:

Şu node'ları sırayla aç ve **"Sheet name"** dropdown'ını kontrol et:

- `Load Ayarlar` → "Ayarlar" görünmeli ✓
- `Load SSS` → "SSS" görünmeli ✓
- `Read Ayarlar` → "Ayarlar" ✓
- `Read SSS` → "SSS" ✓
- `Clear Ayarlar`, `Save Ayarlar` → "Ayarlar" ✓
- `Clear SSS`, `Save SSS` → "SSS" ✓

Eğer dropdown **kırmızı/uyarı** verirse, dropdown'a tıklayıp listeden tab'ı manuel seç → Save.

> ⚠ Bu adımı atlama: tab adları yanlışsa workflow runtime'da hata verir.

### 2.3 Workflow'u test moduna al

Yeni 2 webhook (`get-settings`, `save-settings`) **test mode**'da çalışmak için her seferinde manuel tetiklenmeli:

1. n8n'de workflow'u aç
2. Sağ üstteki **"Execute Workflow"** butonuna bas (workflow'u dinleme moduna alır)
3. Bu durumda dashboard'dan istek atınca webhook'lar yanıt verir

> 💡 **Prod için:** Workflow'u **"Active"** yaparsan, `webhook-test/` path'leri yerine `webhook/` path'leri otomatik aktif olur ve sürekli dinler. O zaman dashboard'daki URL'leri de `webhook/`'a çevirmen gerekir (bkz. dashboard.html `CONFIG`).

### 2.4 Gmail Trigger'ı aktive et (ana akış için)

Gmail polling'in çalışması için workflow'u **Active** yap:

1. Sağ üstteki **toggle switch**'i (Active/Inactive) aç
2. Gmail Trigger artık dakikada bir mailleri kontrol eder

> **Not:** Active modda webhook'lar `webhook/` path'inde çalışır, `webhook-test/`'te değil. Test sırasında aktif/pasif yapmaya dikkat et.

---

## 3. Dashboard Yapılandırması

### 3.1 Webhook URL'lerini kontrol et

`dashboard.html` dosyasının başında JS'te `CONFIG` objesi var:

```js
const CONFIG = {
    getDataUrl: 'http://localhost:5678/webhook-test/get-crm-data',
    sendReplyUrl: 'http://localhost:5678/webhook-test/send-reply',
    getSettingsUrl: 'http://localhost:5678/webhook-test/get-settings',
    saveSettingsUrl: 'http://localhost:5678/webhook-test/save-settings',
    refreshIntervalMs: 30000
};
```

- **Test modunda kullanıyorsan:** URL'ler bu haliyle kalsın (`webhook-test/`)
- **n8n'i Active yaptıysan:** Tüm URL'lerde `webhook-test/` → `webhook/` olarak değiştir

### 3.2 Dashboard'ı aç

`dashboard.html` dosyasına çift tıkla → tarayıcıda açılır.

> Tarayıcıda CORS hatası alırsan: n8n'in CORS ayarı zaten `*` (her yerden kabul). Hata devam ederse n8n'in çalıştığını ve workflow'un test/active modda olduğunu kontrol et.

---

## 4. End-to-End Test Senaryosu

### 4.1 Settings save/load testi

1. Dashboard'da sağ üstte **⚙️ Ayarlar** tıkla → modal açılır, alanlar boş gelir
2. Doldur:
   - Şirket Adı: `Bakırçay Mağazası`
   - Ürün/Hizmet: `Çevrimiçi giyim mağazası`
   - Çalışma Saatleri: `Hafta içi 09:00-18:00`
   - İade Politikası: `14 gün içinde iade hakkı`
   - Kargo Bilgisi: `Yurtiçi Kargo, 1-3 iş günü`
   - Yanıt Tonu: `Samimi`
3. **+ Yeni SSS Ekle** ile bir SSS gir:
   - Soru: `Kargom ne zaman gelir?`
   - Cevap: `Siparişiniz 1-3 iş günü içinde Yurtiçi Kargo ile gelir.`
4. **💾 Kaydet** → Sağ üstte yeşil toast "Ayarlar kaydedildi" görmeli
5. **Doğrula:** Google Sheets'te `Ayarlar` tab'ı açılınca 2. satırda yazdıkların görünmeli; `SSS` tab'ında 2. satırda Soru-Cevap görünmeli
6. **Reload testi:** Modal'ı kapat, tekrar **⚙️ Ayarlar** aç → alanlar **dolu** gelmeli (load çalışıyor)

### 4.2 AI memory + işletme bağlamı testi

1. Test mailden işletme hesabına şu maili at:
   ```
   Konu: Kargo durumu
   Mesaj: "Merhaba, kargom hala gelmedi, ne zaman bekleyeyim?"
   ```
2. ~1 dakika bekle (Gmail polling)
3. Dashboard'da **🔄 Yenile** → sidebar'da yeni müşteri görünmeli
4. Müşteriye tıkla → konuşma panelinde mail kartı + AI Taslak görmeli
5. **Doğrula:** AI Taslak'ında şu öğeler olmalı:
   - "Bakırçay Mağazası" gibi şirket adı geçmesi
   - "1-3 iş günü içinde Yurtiçi Kargo ile..." gibi SSS cevabını referans alması
   - Ton samimi olması (formal değil)

### 4.3 Memory testi (önceki yazışmalar)

1. Aynı mail adresinden 2. mail at:
   ```
   Mesaj: "Hâlâ bir cevap alamadım"
   ```
2. ~1 dakika bekle, dashboard'ı yenile
3. **Doğrula:** 2. mailin AI Taslak'ında 1. maile referans olmalı (ör: "önceki mailinizdeki kargo sorununa devam ediyoruz...")

### 4.4 Yanıt gönderme testi (Faz 1 bug fix doğrulaması)

1. Bir maile **💬 Yanıtla** → modal açılır, AI taslağı dolu gelir
2. Metni biraz değiştir → **🚀 Yanıtı Gönder**
3. **Doğrula:**
   - Toast "Mail başarıyla gönderildi" (alert değil — Faz 2 toast)
   - Mail kartı satırı yeşil "✓ YANITLANDI" olmalı
   - Test mail adresinin inbox'ında mail gelmeli
   - Google Sheets'te o satır → `Yanit Durumu = Yanıtlandı`, `Email` kolonu temiz, `Gonderilen Yanit` modal'da yazdığın metni içermeli

---

## 5. Sorun Giderme

| Belirti | Olası Sebep | Çözüm |
|---------|-------------|-------|
| Dashboard "Veri çekilemedi" | n8n çalışmıyor / workflow inactive | n8n'i başlat, workflow'u Active yap veya "Execute Workflow" bas |
| ⚙️ Ayarlar modal açılır ama alanlar boş, "Ayarlar yüklenemedi" | `get-settings` webhook test modunda + tekrar Execute Workflow basılmamış | n8n'de "Execute Workflow" bas, modal'ı kapatıp tekrar aç |
| Settings kaydedilmedi, "Kaydetme hatası" | `save-settings` webhook tetiklenmedi | n8n'de "Execute Workflow" bas, tekrar dene |
| n8n console'da "Sheet not found: Ayarlar" | Tab adı tam olarak `Ayarlar` değil (büyük harf, boşluk) | Sheets'te tab adını düzelt veya n8n node'unda dropdown'dan seç |
| Mail geldi ama dashboard'da görünmüyor | `Mesaj ID`/`Thread ID`/`Tarih` kolonları eksik veya başlığı yanlış | Sheets ana sayfada başlıkları kontrol et (1.1 adımı) |
| AI Taslak işletme bilgisini kullanmıyor | Settings boş veya `Load Ayarlar` node çalışmadı | Dashboard'dan ⚙️ Ayarlar'da bilgileri kaydet, yeni mail at |
| Active mode'da webhook'lar 404 | URL'ler hâlâ `webhook-test/` | dashboard.html `CONFIG`'de URL'leri `webhook/`'a çevir |

---

## 6. Sonraki Faza Geçmeden Önce

Tüm checkler ✓ olduktan sonra sistem **Faz 1 + 2 + 3 entegre** çalışıyor demektir. Sonraki potansiyel adımlar (`MEMORY.md`'de değil, plan dosyasında detayları var):

- **Faz 4:** Slack/Telegram bildirim, browser notification, SLA takibi
- **Faz 5:** Auth, config dosyası, prod hazırlığı
- **İleri:** Kategori/aciliyet sınıflandırma, otomatik yanıt eşiği, çok dilli destek

Hangisine geçileceğini birlikte konuşacağız.
