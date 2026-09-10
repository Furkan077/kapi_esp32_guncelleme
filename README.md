# 📱 SmartDoor APK Güncelleme Sunucusu

Bu repo, **SmartDoor** Android uygulamasının otomatik güncelleme altyapısını barındırır.  
Yerel Python HTTPS sunucusunun yerine **GitHub** kullanılarak güncelleme sistemi internete taşındı.

---

## 🗂️ Repo İçeriği

```
kapi_esp32_guncelleme/
├── version.json        ← Android uygulaması bunu okur (hangi sürüm var, APK nerede?)
└── README.md           ← Bu dosya
```

> **APK dosyası bu repoda saklanmaz.**  
> Her sürüm için bir **GitHub Release** oluşturulur; APK o release'e asset olarak eklenir.

---

## ⚙️ Nasıl Çalışır?

```
Android Uygulama (UpdateManager.kt)
        │
        │  1. GET isteği
        ▼
raw.githubusercontent.com/…/version.json
        │
        │  2. versionCode karşılaştırır
        │     (sunucu > mevcut?) → güncelleme var
        ▼
Kullanıcıya dialog gösterir
        │
        │  3. "Hemen Güncelle" tıklanırsa
        ▼
github.com/…/releases/download/vX.Y/SmartDoor.apk
        │
        │  4. APK indirilir, SHA-256 doğrulanır
        ▼
Android sistemi kurulumu başlatır
```

### Güvenlik Katmanları

| Kontrol | Açıklama |
|---------|----------|
| **HTTPS zorunluluğu** | `version.json` ve APK URL'si HTTPS değilse sistem reddeder |
| **SHA-256 doğrulama** | İndirilen APK'nın özeti `version.json`'daki `sha256` ile eşleşmeli |
| **versionCode kontrolü** | Sadece sunucudaki `versionCode` > mevcut kod ise güncelleme önerilir |
| **Atlama hafızası** | Kullanıcı "Bu sürümü atla" derse tekrar hatırlatılmaz (SharedPrefs) |

---

## 📄 version.json Formatı

```json
{
  "versionCode": 18,
  "versionName": "2.7",
  "changelog": "Misafir süresi uzatma ve süresi yaklaşan/dolan kullanıcı bildirimleri eklendi.",
  "sha256": "1b19dbe8c178b099b1d9e085537de397ff25b3d483ba2304413ef8ef6963bac1",
  "apkUrl": "https://github.com/Furkan077/kapi_esp32_guncelleme/releases/download/v2.7/SmartDoor.apk"
}
```

| Alan | Tip | Açıklama |
|------|-----|----------|
| `versionCode` | int | Sayısal sürüm kodu. Her yeni sürümde **mutlaka artmalı**. |
| `versionName` | string | Kullanıcıya gösterilen sürüm adı (örn. `"2.8"`) |
| `changelog` | string | Güncelleme dialogunda gösterilecek yenilikler |
| `sha256` | string | APK dosyasının SHA-256 özeti (küçük harf hex). Bütünlük doğrulaması için. |
| `apkUrl` | string | APK'nın doğrudan indirme linki. Mutlaka **HTTPS** olmalı. |

---

## 🚀 Yeni Sürüm Yayınlama

> **Bu adımları tek komutla otomatik yapan `deploy_apk.sh` scripti projenin kökündedir.**

### Otomatik (Önerilen)

```bash
# esp32kapi proje dizininde çalıştırın:
./deploy_apk.sh <versionCode> "<versionName>" "Changelog açıklaması"

# Örnek:
./deploy_apk.sh 19 "2.8" "Yeni özellikler ve hata düzeltmeleri"
```

Script şu adımları **sırasıyla ve otomatik** yapar:

1. 🔨 `gradlew assembleDebug` ile APK derle
2. 🔒 `sha256sum` ile APK özetini hesapla
3. 📝 `version.json`'u güncelle (`kapi_guncelleme_repo` dizininde)
4. 📤 `git push` ile `version.json`'u GitHub'a gönder
5. 🗑️ Varsa eski aynı tag'li release'i sil
6. 🚀 Yeni GitHub Release oluştur (`vX.Y` tag ile)
7. 📦 APK'yı release asset'i olarak yükle
8. ✅ Tüm URL'leri ekrana yazdır

### Manuel

1. APK'yı derle
2. SHA-256 hesapla: `sha256sum SmartDoor.apk`
3. `version.json`'u düzenle (tüm alanları güncelle)
4. `git add version.json && git commit -m "vX.Y" && git push`
5. GitHub → Releases → "Draft a new release" → Tag: `vX.Y` → APK dosyasını sürükle bırak

---

## 🔗 Önemli URL'ler

| Amaç | URL |
|------|-----|
| Android uygulaması bu URL'yi okur | `https://raw.githubusercontent.com/Furkan077/kapi_esp32_guncelleme/main/version.json` |
| Mevcut release listesi | `https://github.com/Furkan077/kapi_esp32_guncelleme/releases` |
| APK indirme (v2.7) | `https://github.com/Furkan077/kapi_esp32_guncelleme/releases/download/v2.7/SmartDoor.apk` |

---

## 📱 Android Tarafı — UpdateManager.kt

**Dosya:** `bahce/android/app/src/main/java/com/smartdoor/UpdateManager.kt`

```kotlin
// Varsayılan güncelleme sunucu adresi (değiştirilebilir)
const val DEFAULT_UPDATE_URL =
    "https://raw.githubusercontent.com/Furkan077/kapi_esp32_guncelleme/main/version.json"
```

### Güncelleme Kontrolünün Tetiklendiği Yerler

| Tetikleyici | Davranış |
|------------|----------|
| Uygulama başlarken | Sessiz kontrol — güncelleme varsa dialog, yoksa sessiz |
| Ayarlar → "Güncelleme Kontrol Et" butonu | Manuel kontrol — her durumda sonucu Toast ile bildirir |

### SharedPreferences Anahtarları

| Anahtar | Açıklama |
|---------|----------|
| `update_server_url` | Özel sunucu URL'si (boşsa `DEFAULT_UPDATE_URL` kullanılır) |
| `skipped_update_code` | Kullanıcının "Bu sürümü atla" dediği `versionCode` |

---

## 📜 Sürüm Geçmişi

| versionCode | versionName | Tarih | Notlar |
|-------------|-------------|-------|--------|
| 18 | 2.7 | 09.09.2026 | Misafir süresi uzatma; yaklaşan/dolan bildirimler |

---

## 🏛️ Eski Sistem (Arşiv)

Ocak 2026'ya kadar güncelleme dağıtımı yerel ağdaki Python HTTPS sunucusu üzerinden yapılıyordu:

```
Sunucu:    https://192.168.3.80:8443
Script:    serve_updates_https.py
Dizin:     /home/kerim/esp32kapi_server/
Sertifika: Self-signed (tarayıcıda güvensiz uyarısı veriyordu)
```

**Sınırlılıkları:**
- Sadece ev ağındayken (192.168.3.x) çalışıyordu
- Self-signed sertifika → Android güven sorunları yaşanabiliyordu
- Python sunucu sürecinin sürekli çalışması gerekiyordu

**Neden GitHub'a geçildi:**
- ✅ Her yerden erişim (ev dışı, mobil veri)
- ✅ Gerçek TLS sertifikası
- ✅ `git push` = dağıtım tamamlandı
- ✅ Sunucu sürecine gerek yok
- ✅ Ücretsiz

---

## 🔧 İlgili Proje

Ana ESP32 + Android projesi: [Furkan077/kapi_Son](https://github.com/Furkan077/kapi_Son)
