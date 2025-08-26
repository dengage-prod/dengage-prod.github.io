# dEngage Web Push Demo ve Örnek E‑Ticaret Uygulaması

Bu repository, dEngage Web SDK ile web push bildirimleri ve kullanıcı etkileşim takibini sergileyen, statik olarak servis edilebilen bir örnek e‑ticaret tek sayfa uygulamasını (SPA) barındırır. Proje; bir SPA’nın derlenmiş çıktıları, dEngage push servis worker entegrasyonu, tarayıcı gizli mod tespiti ve cihaz/ortam parmak izi üretimi gibi bileşenleri içerir.

## Öne Çıkan Özellikler
- **dEngage Web SDK entegrasyonu**: `index.html` içinde SDK’yı yükler ve `dengage('initialize')` ile başlatır.
- **Web Push Servis Worker**: `dengage-webpush-sw.js` ile dEngage push servis worker’ı dinamik olarak içe aktarılır.
- **Gizli mod tespiti**: `BrowsingModeDetector.js` ile tarayıcının normal/gizli modda olduğu anlaşılabilir.
- **Parmak izi (fingerprinting)**: `fg_deneme.js` (FingerprintJS2) aracılığıyla tarayıcı/cihaz parmak izi üretimi örneklenmiştir.
- **Statik barındırma**: Derlenmiş `static/` varlıklarıyla herhangi bir statik HTTP sunucusunda çalışır.

## Dizin Yapısı
```text
/workspace
├─ index.html                    # Giriş noktası, dEngage SDK yükleme/başlatma
├─ dengage-webpush-sw.js         # dEngage web push servis worker yönlendiricisi
├─ BrowsingModeDetector.js       # Gizli/normal mod tespiti için yardımcı kütüphane
├─ fg_deneme.js                  # FingerprintJS2 (parmak izi) kütüphanesi
├─ static/
│  ├─ css/                       # Derlenmiş CSS çıktıları
│  ├─ js/                        # Derlenmiş JS çıktıları (manifest, vendor, app)
│  └─ img/                       # Görseller
└─ README.md
```

## Hızlı Başlangıç (Lokal Çalıştırma)
Proje statiktir; herhangi bir HTTP sunucusuyla çalışır.

- Python 3 ile:
```bash
python3 -m http.server 8080
# Ardından: http://localhost:8080
```

- Node.js ile (tek komutla):
```bash
npx serve . -l 8080 --single
# Ardından: http://localhost:8080
```

> Not: SPA yönlendirmeleri için 404’leri `index.html`’e düşürmek gerekebilir. `serve` için `--single` bayrağı bu işi üstlenir.

## dEngage Web Push Yapılandırması
Bu repoda servis worker yönlendirmesi `dengage-webpush-sw.js` dosyasıyla yapılır. Dosya, URL sorgu parametrelerinden hesabı okur ve ilgili dEngage SW dosyasını içe aktarır:

```js
// dengage-webpush-sw.js (özet)
const swUrl = new URL(location);
const accountId = swUrl.searchParams.get('account_id') || '';
const appGuid = swUrl.searchParams.get('app_guid') || '';
if (accountId && appGuid) {
  importScripts(`https://pcdn.dengage.com/p/push/${accountId}/${appGuid}/dengage_sw.js`);
}
```

Uygulama tarafında (örnek):
```js
// Tarayıcıda (örnek kayıt)
navigator.serviceWorker.register(
  '/dengage-webpush-sw.js?account_id=HESAP_ID&app_guid=UYGULAMA_GUID'
);
```

- `index.html` içinde SDK aşağıdaki biçimde yüklenir ve başlatılır:
```html
<script>
!function(e,a,n,d,g){
  e.dengage=e.dengage||function(){(e.dengage.q=e.dengage.q||[]).push(arguments)},
  d=a.getElementsByTagName("head")[0],(g=a.createElement("script")).async=1,
  g.src="https://pcdn.dengage.com/p/push/28/b846d121-846e-74c4-6f44-be4ccbc77931/dengage_sdk.js",
  d.appendChild(g)
}(window,document);

dengage('initialize');
</script>
```
> Kendi hesabınız için `pcdn.dengage.com` yolunu, `account_id` ve `app_guid` değerlerini güncelleyin. dEngage kurulum ve anahtar bilgileri için dEngage destek dökümantasyonuna bakın.

## Gizli Mod Tespiti ve Parmak İzi
- `BrowsingModeDetector.js`: Tarayıcı özelliklerini kullanarak gizli (incognito/private) veya normal modda olup olmadığını saptamaya yardımcı olur.
- `fg_deneme.js` (FingerprintJS2): Tarayıcı/cihazdan sinyaller toplayarak deterministik bir parmak izi üretmeye yarayan bir kütüphanenin derlenmiş sürümüdür.

> Uyumluluk uyarısı: Parmak izi üretimi ve gizli mod tespiti, KVKK/GDPR gibi regülasyonlar kapsamında kişisel verilerle ilişkilendirilebilir. Üretim ortamında bu mekanizmaları kullanmadan önce hukuki uygunluk, izin/onay akışları ve aydınlatma metinlerini sağlamanız gerekir.

## Geliştirme Notları
- Bu repo derlenmiş SPA çıktıları içerir. Orijinal kaynak kodlar (ör. Vue/React/Angular) bu repositor’de bulunmaz.
- Stil ve davranış değişiklikleri için doğrudan `static/` içeriği/`index.html` üzerinde düzenleme yapabilir veya orijinal uygulamanın kaynak kodlarında değişiklik yapıp yeniden derleme yoluna gidebilirsiniz.

## Dağıtım (GitHub Pages)
Repository adı bir GitHub Pages kök alanına uygun (`<kullanici>.github.io`) olacak şekilde kurgulanmıştır. Statik içeriği GitHub Pages ile yayınlamak için:
1. İçerikleri `main` (veya `master`) dalına push’layın.
2. GitHub’da repo ayarlarından Pages bölümünde yayın dalını ve kök dizini seçin.
3. Değişiklikten sonra cache’ler nedeniyle dağıtım birkaç dakika sürebilir.

## Sorun Giderme
- Servis worker kayıt olmuyor: Kayıt URL’sinde `account_id` ve `app_guid` parametrelerinin iletildiğini ve sitenin HTTPS üzerinden servis edildiğini doğrulayın.
- Push alamıyorum: Dizin/alan adının dEngage hesabınızda tanımlı olduğundan ve tarayıcı izinlerinin verildiğinden emin olun.
- SPA yönlendirmeleri bozuk: Statik sunucuda 404’lerin `index.html`’e yönlendirildiğinden emin olun (örn. `serve --single`).

## Lisans ve Üçüncü Parti Bileşenler
- FingerprintJS2, MIT lisansı ile yayınlanmıştır (bkz. resmi repo: https://github.com/Valve/fingerprintjs2).
- `BrowsingModeDetector.js` içinde ES6 polyfill ve yardımcı kodlar yer alır; ilgili lisanslar kendi dosyaları içinde bulunmaktadır.
- dEngage SDK, dEngage tarafından sağlanır. Kullanım koşulları için dEngage dokümantasyonuna bakın.