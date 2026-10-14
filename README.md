# Claims Vision iOS Builder

Bu depo **herkese açıktır ve yalnızca bir GitHub Actions iş akışı içerir** (`.github/workflows/ios-simulator-build.yml`).
Uygulamanın kaynak kodu burada **bulunmaz**; özel (private) uygulama deposunda durur.

## Ne işe yarar?

Özel depodaki Claims Vision iOS uygulamasını, GitHub'ın macOS çalıştırıcısında iOS Simulator için derler ve
doğrudan [Appetize.io](https://appetize.io)'ya yükler; böylece uygulama tarayıcıda iPhone simülatöründe denenebilir.

Neden ayrı bir depo? Özel depolarda macOS dakikaları ücretli ve kotalıdır, herkese açık depolarda ücretsizdir.
Kod herkese açılmadan ücretsiz macOS derlemesi almak için derleme işi burada çalışır, kaynak kod yalnızca
derleme sırasında çalıştırıcıya geçici olarak çekilir. Expo/EAS derleme kotası gerekmez.

## Nasıl çalışır?

1. **Actions → iOS Simulator Build (private source) → Run workflow** ile elle başlatılır
   (`ref` alanı derlenecek dal/etiket/commit, varsayılan `main`).
2. Özel uygulama deposu salt okunur bir token ile çekilir.
3. Bağımlılıklar kurulur, `expo prebuild` ve `xcodebuild` (imzasız, simulator) ile `.app` derlenir.
4. `.app` zip'lenir ve Appetize'daki mevcut Claims Vision uygulamasına yüklenir (yeni kopya açılmaz).
   Bağlantı çalışma özetinde (`Summary`) görünür.

Derleme yaklaşık **25–30 dakika** sürer. Uygulama deposundaki her birleştirmeden sonra otomatik değil, **elle** tetiklenir.

## TestFlight (EAS'siz)

`ios-testflight.yml` imzalı bir build alır ve doğrudan App Store Connect / TestFlight'a yükler (Expo build kotası kullanılmaz).
Xcode'un otomatik imzalaması App Store Connect API anahtarıyla çalışır, ayrıca `.p12` / profil dışa aktarmak gerekmez.
Build numarası = `build_number_base` (varsayılan 1000) + çalışma numarası; App Store Connect'teki mevcut build'lerin üstünde kalmalıdır.
Ek secret'lar: `ASC_API_KEY_P8`, `ASC_API_KEY_ID`, `ASC_API_ISSUER_ID`, `APPLE_TEAM_ID`.

## Gerekli secret'lar (Settings → Secrets and variables → Actions)

| Ad | Açıklama |
| --- | --- |
| `SOURCE_REPO_TOKEN` | Yalnızca özel uygulama deposu için, yalnızca **Contents: Read** yetkili fine-grained token. Süresi dolunca yenilenmelidir. |
| `APPETIZE_API_TOKEN` | Appetize hesabı → API Token (Developer rolü yeterli). |
| `APPETIZE_PUBLIC_KEY` | İsteğe bağlı. Boşsa Appetize'da en son güncellenen `com.claimsvision` iOS uygulaması hedeflenir. |

## Güvenlik notları

- İş akışı yalnızca `workflow_dispatch` ile çalışır; `push` / `pull_request` tetikleyicisi yoktur, fork'lar secret'lara ulaşamaz.
  **Bu tetikleyicileri eklemeyin.**
- Derlenen uygulama GitHub'a **artifact olarak yüklenmez** (herkese açık depolarda artifact'ı her GitHub kullanıcısı indirebilir);
  yalnızca Appetize'a gönderilir.
- Bu depoya uygulama kodu, anahtar, token ya da `.env` dosyası **eklemeyin**.
- Herkese açık depoda Actions logları herkese görünür; gizli değerler maskelenir ama derleme çıktısı (dosya adları,
  bağımlılık listesi) görünür.

## Sorun giderme

- **İş 10 saniyede, hiçbir adım çalışmadan düşerse:** çalıştırıcı atanmamıştır; GitHub Actions kotası/faturalandırma
  ayarlarını kontrol edin.
- **"Checkout the private source" adımı hata verirse:** `SOURCE_REPO_TOKEN` süresi dolmuş ya da yetkisi eksik olabilir.
- **Appetize adımı hata verirse:** `APPETIZE_API_TOKEN` / `APPETIZE_PUBLIC_KEY` değerlerini kontrol edin.
