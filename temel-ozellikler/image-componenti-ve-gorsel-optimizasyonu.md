# Image Component'i ve Görsel Optimizasyonu

Next.js, **10.0.0** versiyonundan bu yana yerleşik olarak Image Component'ine ve Otomatik Görsel Optimizasyonuna sahiptir.

`next/image`'yi `import` ederek kullanabileceğiniz Image component'i, HTML'nin `<img>` elementinin modern web için geliştirilmiş halidir.

Otomatik görsel optimizasyonu, tarayıcı destekliyorsa, görselleri [WebP](https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Image_types) gibi modern biçimlerde yeniden boyutlandırmaya ve optimize etmeye olanak tanır. Böylece büyük görsellerin daha küçük ekrana sahip cihazlara gönderilmesi önlenir. Ayrıca Next.js'nin gelecekteki görsel biçimlerini de otomatik olarak benimsemesine ve bu yeni biçimleri de destekleyen tarayıcılara sunmasına olanak tanıyacaktır.

Otomatik görsel optimizasyonu, herhangi bir görselle çalışabilir. Görsel, CMS gibi harici bir veri kaynağında barındırılıyor olsa bile optimize edilebilir.

Next.js, görselleri build sırasında optimize etmek yerine, request üzerine kullanıcıların istediği şekilde optimize eder. Böylece build süreniz görsellerin sayısı fazlalaştı diye uzamaz.

Görseller varsayılan olarak lazy load edilir. Yani henüz ekranda görünmemiş olan bir görselin yüklenmesi beklenmez. Böylece sayfa hızınız düşmez. Görseller, ekran kaydırıldıkça \(görsel alanı ekranda belirdiği zaman\) yüklenir.

Görseller her zaman, Google'nin arama sıralamasında kullanacağı [Core Web Vital](https://web.dev/vitals/) olan  [Cumulative Layout Shift](https://web.dev/cls/)'ten kaçınacak şekilde oluşturulur.

## Image Component

Uygulamanıza bir görsel eklemek için `next/image` component'ini `import` edin:

```text
import Image from 'next/image'

function Home() {
  return (
    <>
      <h1>My Homepage</h1>
      <Image
        src="/me.png"
        alt="Picture of the author"
        width={500}
        height={500}
      />
      <p>Welcome to my homepage!</p>
    </>
  )
}

export default Home
```

## Görselleri Import Etmek

Projenizde bulunan görselleri `import` edebilirsiniz. \(`require` desteklenmez. `import` kullanın.\)

Direkt `import` ettiğinizde Image component'ine `width`, `height` ve `blurDataUrl` otomatik olarak sağlanacaktır. Ama `alt` property'ine halen ihtiyacı vardır.

```text
import Image from 'next/image'
import profilePic from '../public/me.png'

function Home() {
  return <>
    <h1>Anasayfam</h1>
    <Image
      src={profilePic}
      alt="Yazılımcının fotoğrafı"
      // width={500} otomatik olarak sağlandı
      // height={500} otomatik olarak sağlandı
      // blurDataURL="data:..." otomatik olarak sağlandı
      // İsteğe bağlı olarak görsel yüklenene kadar sunulacak bir blurred versiyonu kullanabilirsiniz.
      // placeholder="blur"
    />
    <p>Anasayfama hoş geldin!</p>
  </>
}
```

 Ama dinamik veya harici kaynaktan gelen  [`width`](https://nextjs.org/docs/api-reference/next/image#width), [`height`](https://nextjs.org/docs/api-reference/next/image#height) ve[`blurDataURL`](https://nextjs.org/docs/api-reference/next/image#blurdataurl) değerlerini manuel olarak girmelisiniz.

## Propertiy'ler

### Zorunlu Prop'lar

#### src

Zorunludur ve değeri aşağıdakilerden biri olmalıdır:

* Yukarıdaki örnek kodda olduğu gibi statik olarak import edilen bir görsel dosyası.
* Bir path string'i. Bu, [`loader`](image-componenti-ve-gorsel-optimizasyonu.md#loader)'e bağlı olarak harici bir URL veya dahili bir path olabilir.

Harici bir URL kullanırken bunu `next.config.js`'deki [domains](https://nextjs.org/docs/basic-features/image-optimization#domains)'e eklemelisiniz.

#### width

Görselin piksel cinsinden genişliği. Herhangi bir birim adı belirtmeden direkt integer bir sayı verilmelidir. 

Statik olarak `import` edilmediyse veya `layout="fill"` değilse zorunlu bir prop'tur.

#### height

Görselin piksel cinsinden yüksekliği. Herhangi bir birim adı belirtmeden direkt integer bir sayı verilmelidir.

Statik olarak `import` edilmediyse veya `layout="fill"` değilse zorunlu bir prop'tur.

### İsteğe Bağlı Prop'lar

#### layout

Görünen alan değiştikçe görselin yerleşim davranışıdır. Varsayılan olarak **`intrinsic`**'dir \(içsel\).

**`fixed`**: görsel boyutları, native`<img>` elementine benzer şekilde, görünüm alanı değiştikçe değişmez.

**`intrinsic`**: görüntü daha küçük görünüm alanlarında geçince görsel de küçülür. Ancak büyük görünüm alanlarında orijinal boyutlar korunur.

**`responsive`**: görsel, daha küçük görünüm alanlarında küçülür, daha büyük görünüm alanlarında büyür.

**`fill`**: görüntü hem width hem de height olarak kapsayıcısı olan elementin boyutlarına uzatılır. Bu genellikle `objectFit` property'iyle birlikte kullanılır.

Örnnekleri inceleyebilirsiniz:

* [Demo the `fixed` layout](https://image-component.nextjs.gallery/layout-fixed)
* [Demo the `intrinsic` layout](https://image-component.nextjs.gallery/layout-intrinsic)
* [Demo the `responsive` layout](https://image-component.nextjs.gallery/layout-responsive)
* [Demo the `fill` layout](https://image-component.nextjs.gallery/layout-fill)
* [Demo background image](https://image-component.nextjs.gallery/background)

#### loader

URL'leri çözmek için kullanılan özel bir fonksiyon. `next.config.js` dosyasında varsayılan olarak `images` objesidir.

`loader`, aşağıdaki parametreler verildiğinde geriye bir string döndüren bir fonksiyondur:

* [`src`](image-componenti-ve-gorsel-optimizasyonu.md#src)
* [`width`](image-componenti-ve-gorsel-optimizasyonu.md#width)
* [`quality`](image-componenti-ve-gorsel-optimizasyonu.md#height)

```text
import Image from 'next/image'

const myLoader = ({ src, width, quality }) => {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}

const MyImage = (props) => {
  return (
    <Image
      loader={myLoader}
      src="me.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```

#### sizes

Media query'lerini aygın boyutuna eşleyen bir string. Varsayılan olarak `100vw`'dir.

`layout="responsive"` veya `layout="fill"` kullanırken `sizes`'i ayarlamanızı öneririz. Görüntünüz görünüm alanıyla aynı genişlikte olmayacaktır.

[Daha fazlası için...](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-sizes)

#### quality

100'ün en iyi kalite olduğu 1 ile 100 arasında bir integer olan, optimize edilmiş görsel kalitesidir. Varsayılan değeri 75'tir.

#### priority

`true` ise, görsel yüksek öncelikli ve [ön yükleme](https://web.dev/preload-responsive-images/) yapılmış kabul edilecektir. Yalnızca görselin ekranın üst kısmında görünür olduğu durumda kullanılmalıdır. Varsayılan değeri `false`'dir.

> **Preload'a Genel Bakış**
>
> Preload, tarayıcıya, HTML'de keşfedilmeden \(henüz o kod görülmeden\) önce, mümkün olan en kısa sürede yüklemek istediğiniz kritik kaynaklar hakkında bilgi vermenizi sağlar. Bu, özellikle stil sayfalarında bulunan yazı tipleri, arka plan görselleri veya bir komut dosyasından yüklenen kaynaklar gibi kolayca keşfedilemeyen kaynaklar için kullanışlıdır.
>
> `<link rel="preload" as="image" href="important.png" />`
>
> **Responsive görseller + preload = hızlı görsel yüklemeleri**
>
> Responsive görseller ve preload son birkaç yıldır mevcuttu ancak aynı zamanda bi'şeyler eksikti: Responsive görselleri preload etmenin bir yolu yoktu. Chrome 73'ten ile başlayarak, tarayıcı, `img` etiketini henüz keşfetmeden önce, `srcset`'te belirtilen responsive görsellerin doğru varyantını preload edebilir.
>
> Sitenizin yapısına bağlı olarak bu, önemli ölçüde daha hızlı görsel görüntüleme anlamına gelebilir! Responsive görüntüleri lazy load etmek için JavaScript kullanan bir sitede testler yaptık. Preload, görsellerin 1.2 sn daha hızlı yüklenmesini sağladı.
>
> **Daha fazlası için kaynak** \(10.07.2021\) **:** [https://web.dev/preload-responsive-images/](https://web.dev/preload-responsive-images/)

#### placeholder

Görsel yüklenirken kullanılacak yer tutucudur. `blur` veya `empty` olabilir. Varsayılan değeri `empty`'dir.

`blur` olduğunda blurDataUrl prop'u yer tutucu olarak kullanılacaktır. `src` değeri statik olarak import edilmiş bir görselse ve jpg, png veya webp ise `blurDataURL` otomatik olarak doldurulur.

Dinamik görseller için `blurDataURL` prop'unu sağlamalısınız. [Plaiceholder](https://github.com/joe-bell/plaiceholder) gibi çözümler `base64` üretimine yardımcı olabilir.

Bu değer `empty` ise, görsel yüklenirken yer tutucu olmayacak, yalnızca boş alan olacaktır.

## Gelişmiş Prop'lar

Bazı durumlarda daha gelişmiş bir kullanıma ihtiyaç duyabilirsiniz. `<Image />` component'i isteğe bağlı olarak aşağıdaki gelişmiş özellikleri kabul eder:

### objectFit

`layout="fill"` kullanılırsa görsel alana sığdırılır.

[Daha fazlası için...](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit)

### objectPosition

`layout="fill"` kullanılırsa görsel pozisyonu belirlenir.

[Daha falzası için...](https://developer.mozilla.org/en-US/docs/Web/CSS/object-position)

### loading

> Dikkat: Bu prop yalnızca gelişmiş kullanım içindir. Bir görseli `eager` ile yüklemek normalde **performans için kötü**dür. Bunun yerine [`priority`](image-componenti-ve-gorsel-optimizasyonu.md#priority) property'ini kullanmanızı öneririz.

Görselin yüklenme davranışıdır. Varsayılan olarak `lazy`'dir.

`lazy` değeri verilirse, görsel ekranda görünmediği sürece yüklenmez. Ekran kaydırılarak görselin görüneceği yere gelirse görsel o zaman yüklenir.

`eager` değeri verilirse görsel mutlaka yüklenir.

[Daha fazlası için...](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attr-loading)

### blurDataURL

`src` görseli başarıyla yüklenmeden önce görselin yerine kullanılacak bir veri URL'si. Yalnızca `placeholder="blur"` ile birlikte çalışır.

Base64 ile kodlanmış bir görsel olmalıdır. Büyütülecek ve bulanıklaştırılacaktır. Bu nedenle çok küçük bir görüntü \(10px veya daha az\) önerilir. Daha büyük görsellerin `placeholder` olarak dahil edilmesi uygulamanızın performansına zarar verebilir.

Denemek için:

* [Varsayılan blurDataURL prop demosu](https://image-component.nextjs.gallery/placeholder)
* [Shimmer efekt ile blurDataURL prop demosu](https://image-component.nextjs.gallery/shimmer)

Görselinizle eşleşmesi için [düz renkli bir veri URL'si de oluşturabilirsin](https://png-pixel.com/)iz.

### unoptimized

`true` olduğunda; kalite, boyut veya biçimi değiştirmek yerine kaynak görseli olduğu gibi sunulur. Varsayılan olarak `false`'dir.

## Konfigürasyon

`next/image` component'i için kullanılabilen prop'ları kullanmanın yanı sıra `next.config.js` aracılığıyla daha gelişmiş kullanım durumları için de isteğe bağlı olarak görüntü optimizasyonunu yapılandırabilirsiniz.

### Domains

Harici bir web sitesinde barındırılan görsellerde görsel optimizasyonunu etkinleştirmek için görsel `src`'sinde mutlak bir url kullanın ve hangi domainlerin optimize edilmesine izin verildiğini belirtin. Bu, harici URL'lerin kötüye kullanılmamasını sağlamak için gereklidir. `loader` harici bir görüntü hizmetine ayarlandığında bu seçenek yok sayılır.

```text
module.exports = {
  images: {
    domains: ['example.com'],
  },
}
```

### Loader

Görselleri optimize etmek için Next.js'nin yerleşik görüntü optimizasyonunu kullanmak yerine bir bulut sağlayıcısı kullanmak istiyorsanız yükleyici ve yol önekini yapılandırabilirsiniz. Bu, görsel `src`'sindeki url'leri kullanmanıza ve sağlayıcınız için otomatik olarak doğru mutlak url'yi oluşturmanıza olanak tanır.

```text
module.exports = {
  images: {
    loader: 'imgix',
    path: 'https://example.com/myaccount/',
  },
}
```

Aşağıda, görsel optimizasyonu bulut sağlayıcıları bulunmaktadır:

* [Vercel](https://vercel.com/): Vercel'de çalışıyorsanız otomatik olarak çalışır. Yapılandırma gerektirmez. [Daha fazlası...](https://vercel.com/docs/next.js/image-optimization)
* [Imgix](https://www.imgix.com/): `loader: "imgix"`
* [Cloudinary](https://cloudinary.com/): `loader: "cloudinary"`
* [Akamai](https://www.akamai.com/): loader: `"akamai"`
* Varsayılan: `next dev`, `next start` veya özel bir sunucu ile otomatik çalışır.

Farklı bir sağlayıcınız varsa `next/image` ile [loader](image-componenti-ve-gorsel-optimizasyonu.md#loader-1) prop'unu kullanabilirsiniz.

> `next/image` component'inin varsayılan yükleyicisi, `next export` kullanılırken desteklenmez. Bu durumda diğer yükleyici seçeneklerini kullanmalısınız.

## Önbellekleme

Aşağıda, varsayılan [loader](image-componenti-ve-gorsel-optimizasyonu.md#loader-1) için önbelleğe alma algoritması açıklanmaktadır. Diğer tüm yükleyiciler için lütfen bulut sağlayıcınızın belgelerine bakın.

Görseller request geldiğinde dinaimik olarak optimize edilirler ve `<distDir>/cache/images` dizininde saklanırlar. Optimize edilmiş görsel dosyası, son kullanma tarihine ulaşana kadar sonraki request'lerde kullanılır. Önbelleğe alınmış ancak süresi dolmuş bir dosyayla eşleşen bir request yapıldığında, optimize edilmiş yeni bir görüntü oluşturulmadan ve yeni dosya önbelleğe alınmadan önce önbelleğe alınan dosyalar silinir.

Süre sonu \(veya doğrusu Max Age\), upstream sunucusunun `Cache-Control` header'i tarafından tanımlanır.

`Cache-Control`'da `s-maxage` bulunursa kullanılır. `s-maxage` bulunamazsa `max-age` kullanılır. Eğer `max-age` de yoksa 60 saniye kullanılır.

Oluşturulan olası görsellerin toplam sayısını azaltmak için [`deviceSizes`](image-componenti-ve-gorsel-optimizasyonu.md#device-sizes) ve [`imageSizes`](image-componenti-ve-gorsel-optimizasyonu.md#image-sizes) öğelerini yapılandırabilirsiniz.

## Gelişmiş

Aşağıdaki yapılandırma, gelişmiş kullanım durumları içindir ve genellikle gerekli değildir. Aşağıdaki özellikleri yapılandırmayı seçerseniz gelecekteki güncellemelerde Next.js varsayılanlarında yapılan tüm değişiklikleri geçersiz kılacaksınız.

### Device Sizes

Web sitenizin kullanıcılardan beklenen cihaz genişliklerini bildiğiniz bazı durumlarda, `deviceSizes` özelliğini kullanarak bir cihaz genişliği kesme noktaları array'ı belirtebilirsiniz. Bu genişlikler, `next/image` component'i `layout="responsive"` veya `layout="fill"` kullanıldığında kullanılır. Böylece web sitenizi ziyaret eden cihaz için doğru görsel bulunur.

Herhangi bir yapılandırma sağlanmazsa aşağıdaki varsayılan kullanılır:

```text
module.exports = {
  images: {
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
}
```

### Image Sizes

`imageSizes` özelliğini kullanarak bir görsel genişlikleri array'ı ekleyebilirsiniz. Array'lar bir araya getirileceğinden, bu genişlikler `deviceSizes` içinde tanımlanan genişliklerden farklı \(genellikle daha küçük\) olmalıdır. Bu genişlikler, `next/image` component'i `layout="fixed"` veya `layout="intrinsic"` kullanıldığında kullanılır.

Herhangi bir yapılandırma sağlanmazsa aşağıdaki varsayılan kullanılır:

```text
module.exports = {
  images: {
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
}
```

### Statik Import'ları Devre Dışı Bırakmak

Varsayılan olarak, `import icon from "./icon.png"` gibi statik dosyaları import etmenize ve ardından bunu `src` özelliğine aktarabilirsiniz.

Bazı durumlarda import'un farklı davranmasını bekleyen diğer eklentilerle çakışıyorsa bu özelliği devre dışı bırakabilirsiniz.

Aşağıdaki yapılandırma ile statik görsel import'larını devre dışı bırakabilirsiniz:

```text
module.exports = {
  images: {
    disableStaticImages: true,
  },
}
```



