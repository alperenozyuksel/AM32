# S32 AM32 Targetları


## Eklenen targetlar

| Target | İşlemci | Firmware adı | NTC |
| --- | --- | --- | --- |
| `S32_F051` | STM32F051 | `S32 F051` | Yok |
| `S32_F051_NTC` | STM32F051 | `S32 NTC F051` | PA2 / ADC2 |
| `S32_F421` | AT32F421 | `S32 F421` | Yok |
| `S32_F421_NTC` | AT32F421 | `S32 NTC F421` | PA2 / ADC2 |

## Ortak donanım ayarları

Dört target için kullanılan ortak ayarlar:

- Dead time: `45`
- Akım ölçüm hassasiyeti: `19 mV/A`
- Akım offseti: `0 mV`
- Voltage divider: `110`
- Akım ADC girişi: `PA6 / ADC6`
- Voltaj ADC girişi: `PA3 / ADC3`
- Serial telemetry: Açık

## NTC desteği

`S32_F051_NTC` ve `S32_F421_NTC` targetlarında:

- Harici MOSFET NTC sıcaklığı `PA2 / ADC2` üzerinden okunur.
- AT32F421 ADC DMA dizisi PA2 ölçümü için beş kanala çıkarılmıştır.
- ADC değeri mevcut 191 elemanlı NTC LUT tablosuyla sıcaklığa çevrilir.
- LUT araması binary search ile yapılır ve en yakın tam sıcaklık değeri seçilir.
- Yazılım NTC sıcaklığını `-30°C` ile `127°C` arasında okur.
- Harici NTC sıcaklığı standart seri telemetride ve DShot extended temperature telemetrisinde gönderilir.
- Sıcaklık korumaları harici NTC'ye göre değil, işlemcinin dahili sıcaklık sensörüne göre çalışmaya devam eder.
- Orijinal telemetri kodu bir `else` bloğuna taşınmamıştır. NTC ve standart yollar ayrı `SOE_NTC` / `!SOE_NTC` koşullarıyla seçilir.

## Düşük voltaj davranışı

Motor çalışırken AM32'nin orijinal düşük voltaj koruması korunmuştur:

- Sürüş sırasında kısa süreli voltaj çökmelerinde motor hemen kapatılmaz.
- Orijinal sürekli düşük voltaj sayacı ve yaklaşık 10,5 saniyelik kesme gecikmesi kullanılır.
- Düşük voltaj kesmesi kilitli kalır ve güç döngüsüyle sıfırlanır.

Yalnızca motor kalkışı için ek bir kontrol eklenmiştir:

- Motor durmuşken kalkış komutu geldiğinde voltaj ayarlanan hücre başına veya mutlak kesme eşiğinin altındaysa fazlar enerjilendirilmeden kalkış iptal edilir.
- Bu kontrol hem normal başlangıç hem de sinüzoidal başlangıç için geçerlidir.
- Motor çalıştıktan sonra bu başlangıç kontrolü devre dışı kalır ve orijinal AM32 sürüş koruması kullanılır.

### Hücre başı cutoff tablosu

Firmware hücre başı cutoff değerini `2,50 V` ile `3,50 V` arasında `0,01 V` adımlarla destekler. Aşağıdaki tabloda okunabilirlik için `0,10 V` aralıklar gösterilmiştir. Normal sürüşte voltajın eşik altında kesintisiz yaklaşık 10,5 saniye kalması gerekir; eşik üzerine çıkan bir ölçüm sayacı sıfırlar.

| Hücre başı cutoff | 2S toplam | 3S toplam | 4S toplam | 5S toplam | 6S toplam | Kesme süresi |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2,50 V | 5,00 V | 7,50 V | 10,00 V | 12,50 V | 15,00 V | ≈10,5 sn |
| 2,60 V | 5,20 V | 7,80 V | 10,40 V | 13,00 V | 15,60 V | ≈10,5 sn |
| 2,70 V | 5,40 V | 8,10 V | 10,80 V | 13,50 V | 16,20 V | ≈10,5 sn |
| 2,80 V | 5,60 V | 8,40 V | 11,20 V | 14,00 V | 16,80 V | ≈10,5 sn |
| 2,90 V | 5,80 V | 8,70 V | 11,60 V | 14,50 V | 17,40 V | ≈10,5 sn |
| 3,00 V | 6,00 V | 9,00 V | 12,00 V | 15,00 V | 18,00 V | ≈10,5 sn |
| 3,10 V | 6,20 V | 9,30 V | 12,40 V | 15,50 V | 18,60 V | ≈10,5 sn |
| 3,20 V | 6,40 V | 9,60 V | 12,80 V | 16,00 V | 19,20 V | ≈10,5 sn |
| 3,30 V | 6,60 V | 9,90 V | 13,20 V | 16,50 V | 19,80 V | ≈10,5 sn |
| 3,40 V | 6,80 V | 10,20 V | 13,60 V | 17,00 V | 20,40 V | ≈10,5 sn |
| 3,50 V | 7,00 V | 10,50 V | 14,00 V | 17,50 V | 21,00 V | ≈10,5 sn |

Motor durmuşken başlangıç kontrolü bu süreyi beklemez; ölçülen voltaj eşik altındaysa kalkış hemen iptal edilir.

### Otomatik hücre sayısı algılama tablosu

Önceki hesap ölçülen voltajı `3,70 V` değerine bölüp aşağı yuvarladığı için `22,00 V` değerini 6S yerine 5S algılayabiliyordu. Hücre bazlı cutoff eşiği bu nedenle gereğinden düşük hesaplanıyordu.

Yeni hesap ilk arm sırasında ölçülen batarya voltajını `4,20 V` değerine bölüp yukarı yuvarlar. Hücre sayısı belirlendikten sonra hücre başı cutoff toplam eşik hesabında bu değer kullanılır.

| İlk arm sırasında ölçülen voltaj | Algılanan batarya |
| ---: | ---: |
| 0,01–4,20 V | 1S |
| 4,21–8,40 V | 2S |
| 8,41–12,60 V | 3S |
| 12,61–16,80 V | 4S |
| 16,81–21,00 V | 5S |
| 21,01–25,20 V | 6S |
| 25,21–29,40 V | 7S |
| 29,41–33,60 V | 8S |
| 33,61–37,80 V | 9S |

Örneğin `22,00 V` ve `25,20 V` değerleri 6S, `25,21 V` değeri ise 7S olarak algılanır.

### Hücre sayısının sesli bildirimi

Hücre başı cutoff açıkken ESC ilk arm sırasında otomatik algıladığı hücre sayısını motor üzerinden sesli olarak bildirir. Bu ses düşük voltaj alarmı değildir; algılanan S sayısının onayıdır.

- Motor sargıları hoparlör gibi kullanılarak `playInputTune()` çalıştırılır.
- Ton, algılanan hücre sayısı kadar tekrarlanır: 4S için 4, 5S için 5, 6S için 6 kez.

### Cutoff uyarı tonu

Düşük voltaj cutoff ilk kez oluştuğunda motor üzerinden bir kez `playDefaultTone()` uyarısı çalınır. Cutoff kilitli kaldığı sürece ton tekrar tekrar çalınmaz. Bu uyarı hem başlangıçta anında iptal edilen kalkışta hem de sürüş sırasında gecikme sonunda gerçekleşen kesmede kullanılır.


## Firmware 2.21 çıktıları

Derlenen eklenen dosyalar:

- `AM32_S32_F051_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F051_NTC_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F421_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F421_NTC_2.21.bin` / `.elf` / `.hex`
