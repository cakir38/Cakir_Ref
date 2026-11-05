# ÇAKIR REF Uygulaması – Kurulum (Android)

Bu uygulama bir **web uygulamasıdır (PWA)**: Chrome'dan "Ana ekrana ekle" ile telefona uygulama gibi kurulur,
ilk açılıştan sonra **internetsiz** çalışır. Bluetooth'a bağlanabilmesi için sayfanın **HTTPS** adresinden
açılması gerekir (Chrome şartı), bu yüzden dosyaları ücretsiz bir HTTPS adresine koyuyoruz.

## 1) Dosyaları yayınlama (GitHub Pages, ücretsiz)
1. https://github.com adresinde hesap açın / giriş yapın.
2. **New repository** → ad: `cakir-ref` → **Public** → *Create repository*.
3. **Add file → Upload files**: zip'i bilgisayarda açın, içindeki dosyaların **hepsini** sürükleyin
   (`index.html`, `manifest.webmanifest`, `sw.js`, 3 adet `icon-….png`). Klasör değil, dosyalar yüklenecek.
   *Commit changes* deyin.
4. **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main / (root) → Save.**
5. 1–2 dakika sonra adres çıkar: `https://KULLANICI-ADINIZ.github.io/cakir-ref/`

> Uygulamada şifre veya kişisel bilgi yoktur; herkese açık depo sorun değildir.
> Netlify, Cloudflare Pages gibi benzer ücretsiz servisler de aynı şekilde olur.

## 2) Telefona kurma
1. Android'de **Chrome** ile yukarıdaki adresi açın (internet gerekir, sadece bu ilk seferde).
2. Chrome menüsü ⋮ → **Uygulamayı yükle** (veya *Ana ekrana ekle*).
3. Ana ekranda **ÇAKIR REF** simgesi çıkar. Bundan sonra internetsiz de açılır.

## 3) Kullanım
- Telefonda **Bluetooth açık** olmalı. Android 11 ve altında **Konum** da açık olmalı (Android şartı);
  Android 12 ve üstünde "Yakındaki cihazlar" izni istenir, izin verin.
- **Cihazı ara ve bağlan** → listeden `CAKIR_REF…` seçin.
- İlk **ayar kaydında** Android eşleşme penceresi açar: cihaz ekranındaki 6 haneli kodu girin (bir kez).
- Bağlanınca uygulama telefonun saatini cihaza otomatik gönderir (internetsiz ortamda saat sorunu böylece çözülür).
- **Grafik** sekmesindeki **PDF indir** düğmesi özet, grafik ve tüm kayıtları A4 PDF raporu olarak telefona indirir (internet gerekmez).
- **Cihaz olmadan dene (demo)** ile uygulamayı cihaz bağlamadan gezebilirsiniz.

## Bilinen sınırlar
- Web Bluetooth yalnızca **Android Chrome** (ve Chromium tabanlıları) içindir. iPhone'da çalışmaz.
- Bağlantı uygulama **açıkken** sürer; arka planda izleme yoktur. Uzun izleme için Bilgi sekmesinden
  **"Ekran kapanmasın"** seçeneğini açın.
- **Cihazın BLE yazılımı henüz yoktur.** Mevcut 19.0.3 yazılımı Wi-Fi kullanır, Bluetooth yayını yapmaz.
  Uygulama, `PROTOKOL.md` dosyasındaki sözleşmeye göre yazıldı; ESP32 tarafındaki BLE sürümü bu sözleşmeye
  uyacak şekilde ayrıca yazılacaktır. O zamana kadar uygulama **demo modunda** denenebilir.

## Güncelleme
Dosyaları değiştirdiğinizde `sw.js` içindeki `VERSION` değerini (örn. `cakir-ref-v1.1.1`) artırın;
telefon yeni sürümü bir sonraki internetli açılışta alır.
