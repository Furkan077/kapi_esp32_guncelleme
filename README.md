# SmartDoor APK Güncelleme Sunucusu

Bu repo, [SmartDoor](https://github.com/Furkan077/kapi_Son) Android uygulamasının otomatik güncelleme altyapısını barındırır.

## Nasıl Çalışır?

1. Android uygulaması `version.json` dosyasını kontrol eder
2. `versionCode` mevcut sürümden büyükse kullanıcıya güncelleme sunar
3. Onay verilirse APK bu repo'nun Release'inden indirilir

## Güncelleme URL'si

```
https://raw.githubusercontent.com/Furkan077/kapi_esp32_guncelleme/main/version.json
```

## Yeni Sürüm Yayınlamak

`deploy_apk.sh` scriptini kullanın:

```bash
./deploy_apk.sh <versionCode> <versionName> "Changelog açıklaması"
# Örnek:
./deploy_apk.sh 19 "2.8" "Yeni özellik eklendi"
```
