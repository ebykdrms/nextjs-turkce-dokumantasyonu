# Statik Dosya Sunumu

Next.js,  kök dizinde bulunan`public` dizininin altında, görseller gibi statik dosyaları sunabilir. `public` altındaki dosyalara kodunuzda base url'den \(`/`\) başlayarak ulaşabilirsiniz.

Örneğin, `public/me.png` görseline aşağıdaki şekilde erişebilirsiniz:

```jsx
import Image from 'next/image'

function Avatar() {
  return <Image src="/me.png" alt="me" width="64" height="64" />
}

export default Avatar
```

> **Not:** next/image kullanmak için Next.js 10 veya üstü gereklidir.

Bu klasör ayrıca `robots.txt`, `favicon.ico`, Google Site Verification ve diğer statik dosyalar \(`.html` dahil\) için de kullanışlıdır.

> **Not:** public klasörüne başka bir isim vermeyin. Bu isim değiştirilemez ve statik dosyalara izin vermek için kullanılan tek klasördür.

> `pages/` klasöründeki bir dosya ile aynı ada sahip statik bir dosya bulunmadığından emin olun. Çünkü bu durum hataya neden olacaktır.
>
> Daha fazlası için: [https://nextjs.org/docs/messages/conflicting-public-file-page](https://nextjs.org/docs/messages/conflicting-public-file-page)

> **Not:** Yalnızca build sırasında `public` klasöründe bulunan varlıklar Next.js tarafından sunulacaktır. Çalışma zamanında eklenen dosyalar kullanılamaz. Kalıcı dosya depolama için [AWS S3](https://aws.amazon.com/s3/) gibi bir üçüncü taraf hizmeti kullanmanızı öneririz.



