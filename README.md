# S32 AM32 Targetları

Bu çalışma AM32 firmware **2.21** ve EEPROM sürümü **4** taban alınarak hazırlanmıştır.

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
- Harici NTC sıcaklığı standart seri telemetride ve DShot extended temperature telemetrisinde gönderilir.
- Sıcaklık korumaları harici NTC'ye göre değil, işlemcinin dahili sıcaklık sensörüne göre çalışmaya devam eder.
- Orijinal telemetri kodu bir `else` bloğuna taşınmamıştır. NTC ve standart yollar ayrı `SOE_NTC` / `!SOE_NTC` koşullarıyla seçilir.

## Düşük voltaj davranışı

Motor çalışırken AM32'nin orijinal düşük voltaj koruması korunmuştur:

- Sürüş sırasında kısa süreli voltaj çökmelerinde motor hemen kapatılmaz.
- Orijinal sürekli düşük voltaj sayacı ve yaklaşık 10 saniyelik kesme gecikmesi kullanılır.
- Düşük voltaj kesmesi kilitli kalır ve güç döngüsüyle sıfırlanır.

Yalnızca motor kalkışı için ek bir kontrol eklenmiştir:

- Motor durmuşken kalkış komutu geldiğinde voltaj ayarlanan hücre başına veya mutlak kesme eşiğinin altındaysa fazlar enerjilendirilmeden kalkış iptal edilir.
- Bu kontrol hem normal başlangıç hem de sinüzoidal başlangıç için geçerlidir.
- Motor çalıştıktan sonra bu başlangıç kontrolü devre dışı kalır ve orijinal AM32 sürüş koruması kullanılır.

## Firmware 2.21 çıktıları

Derlenen ve AM32 DroneCAN uygulama imzası eklenen dosyalar:

- `AM32_S32_F051_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F051_NTC_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F421_2.21.bin` / `.elf` / `.hex`
- `AM32_S32_F421_NTC_2.21.bin` / `.elf` / `.hex`
