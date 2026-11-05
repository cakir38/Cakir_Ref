# ÇAKIR REF – BLE Protokolü (sürüm 1)

Bu belge telefon uygulaması (PWA) ile ESP32-C3 cihaz yazılımı arasındaki sözleşmedir.
Uygulama bu belgeye göre yazıldı; ESP32 tarafındaki BLE sürümü (NimBLE-Arduino) aynen buna uymalıdır.
Tüm çok baytlı sayılar **little-endian** (düşük bayt önce).

## Yayın (advertising)
- Cihaz adı: `CAKIR_REF` (isteğe bağlı sonek: `CAKIR_REF-A1B2`). Uygulama adı `CAKIR_REF` ile başlayanları listeler.
- Reklam paketinde 128-bit servis UUID'si bulunmalı (isim tarama yanıtında olabilir).

## Servis
`a1c0e000-5f2b-4d6e-9c1a-0c4b1e5a7f01`

| Karakteristik | UUID (son 12 hane aynı, ilk grup değişir) | Özellik | Güvenlik | Boyut |
|---|---|---|---|---|
| LIVE | `a1c0e001-…` | read, notify | açık | 10 bayt |
| HIST | `a1c0e002-…` | write, notify | açık | yaz: 1 bayt, bildirim: 12 bayt |
| SETT | `a1c0e003-…` | read, write | **yazma için şifreli/eşleşmiş bağlantı** | 16 bayt |
| TIME | `a1c0e004-…` | write | açık | 6 bayt |
| CMD  | `a1c0e005-…` | write | **şifreli/eşleşmiş bağlantı** | 1 bayt |
| INFO | `a1c0e006-…` | read | açık | ASCII, ≤ 20 bayt |

Tam UUID biçimi: `a1c0e00N-5f2b-4d6e-9c1a-0c4b1e5a7f01` (N = 0…6).

## LIVE (10 bayt) – saniyede bir bildirim
| Ofset | Tür | Anlam |
|---|---|---|
| 0 | int16 | sıcaklık × 100 (°C). `0x8000` = geçersiz |
| 2 | uint16 | nem × 100 (%). `0xFFFF` = geçersiz |
| 4 | uint8 | bayraklar: bit0 sensör bağlı, bit1 stabil, bit2 saat senkronize |
| 5 | uint8 | geçmiş kayıt sayısı (0…30) |
| 6 | uint32 | UTC epoch (sn). Saat yoksa 0 |

## HIST – geçmiş
- Uygulama `0x01` yazar → cihaz kayıtları **eskiden yeniye** doğru, her biri ayrı bildirim olarak yollar (≈20–30 ms arayla).
- Boş geçmişte tek bildirim: `idx=0, total=0`, diğer alanlar 0.

Bildirim (12 bayt):
| Ofset | Tür | Anlam |
|---|---|---|
| 0 | uint8 | kayıt sırası (0'dan) |
| 1 | uint8 | toplam kayıt sayısı |
| 2 | uint32 | zaman damgası: ≥ 1 000 000 000 ise UTC epoch; daha küçükse **açılıştan beri saniye** (saat henüz yokken alınan kayıt) |
| 6 | int16 | sıcaklık × 100 |
| 8 | uint16 | nem × 100 |
| 10 | uint8 | kalite (0 iyi … 3 az örnek) |
| 11 | uint8 | örnek sayısı |

## SETT (16 bayt) – okunur, yazılır
| Ofset | Tür | Aralık (cihaz bu aralığa sıkıştırır) |
|---|---|---|
| 0 | int16 | sıcaklık ofseti × 100 (−500…500) |
| 2 | int16 | nem ofseti × 100 (−1000…1000) |
| 4 | uint16 | sensör okuma aralığı ms (250…60000) |
| 6 | uint16 | ekran yenileme ms (100…5000) |
| 8 | uint32 | geçmiş kayıt aralığı ms (5000…3600000) |
| 12 | uint8 | parlaklık % (10…100) |
| 13 | uint8 | ekran maskesi (bit0 ana, bit1 gösterge, bit2 grafik, bit3 durum; en az 1 bit) |
| 14 | uint16 | ayrılmış, 0 |

Geçersiz yazımda (örn. maske 0) cihaz yazmayı reddeder (GATT hata yanıtı).

## TIME (6 bayt) – yazılır
| Ofset | Tür | Anlam |
|---|---|---|
| 0 | uint32 | telefonun UTC epoch'u (sn) |
| 4 | int16 | telefonun saat dilimi farkı, dakika (Türkiye +180) |

Cihaz bunu aldığında saati ayarlar (SNTP gelmemiş gibi davranır), geçmişteki "açılıştan beri saniye" kayıtlarını gerçek saate çevirir.
Uygulama her bağlantıda bunu otomatik yazar.

## CMD (1 bayt) – yazılır, eşleşme ister
`1` = yeniden başlat, `2` = geçmişi temizle, `3` = fabrika ayarı (ardından yeniden başlar).

## INFO – okunur
`CAKIR_REF|v20.0.0|p1` biçiminde: ad | yazılım sürümü | protokol sürümü (bu belge = `p1`).

## Güvenlik
- Cihaz ekranında 6 haneli **passkey** gösterir (IO yeteneği: DisplayOnly), bağlanma tipi: bonding + MITM + Secure Connections.
- SETT yazma ve CMD yazma karakteristikleri şifreli bağlantı ister. Android, uygulama ilk yazdığında eşleşme penceresini kendiliğinden açar; kullanıcı ekrandaki kodu girer.
- LIVE/HIST/TIME/INFO herkese açıktır (sadece ölçüm okuma ve saat ayarı).
- Passkey sabit değil, her eşleşmede rastgele üretilmelidir.

## Cihaz tarafı notları (ESP32-C3)
- Kütüphane: NimBLE-Arduino (Bluedroid'den çok daha az RAM).
- Wi-Fi ile aynı anda çalışmak RAM'i sıkıştırır; çevrimdışı senaryoda Wi-Fi kapatılıp yalnız BLE kullanılması önerilir.
- Varsayılan MTU 23 olsa da tüm yükler ≤ 16 bayttır, MTU pazarlığı gerekmez.
