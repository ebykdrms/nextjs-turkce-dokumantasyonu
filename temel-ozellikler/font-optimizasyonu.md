# Font Optimizasyonu

Next.js, **10.2** versiyonundan bu yana yerleşik olarak web font optimizasyonuna sahiptir.

Varsayılan olarak Next.js fontları getirmek için fazladan gidip dönüş yapmadan build sırasında otomatik olarak CSS fontunu satır içi olarak kullanır. Bu, [First Contentful Paint \(FCP\)](https://web.dev/fcp/) ve [Largest Contentful Paint'te \(LCP\) ](https://vercel.com/blog/core-web-vitals#largest-contentful-paint)iyileştirme sağlar. Örneğin:

```markup
// Önce
<link
  href="https://fonts.googleapis.com/css2?family=Inter"
  rel="stylesheet"
/>

// Sonra
<style data-href="https://fonts.googleapis.com/css2?family=Inter">
  @font-face{font-family:'Inter';font-style:normal...
</style>
```

## Kullanım

Next.js uygulamanıza bir web font eklemek için `next/head`'ı override edin. Örneğin belirli bir sayfaya font ekleyebilirsiniz:

```jsx
// pages/index.js

import Head from 'next/head'

export default function IndexPage() {
  return (
    <div>
      <Head>
        <link
          href="https://fonts.googleapis.com/css2?family=Inter"
          rel="stylesheet"
        />
      </Head>
      <p>Hello world!</p>
    </div>
  )
}
```

Veya şöyle bir yöntem de izleyebilirsiniz:

```jsx
// pages/_document.js

import Document, { Html, Head, Main, NextScript } from 'next/document'

class MyDocument extends Document {
  render() {
    return (
      <Html>
        <Head>
          <link
            href="https://fonts.googleapis.com/css2?family=Inter"
            rel="stylesheet"
          />
        </Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    )
  }
}

export default MyDocument
```

Otomatik web font optimizasyonu şu anda Google font'larını ve Typekit'i desteklemektedir ve yakında diğer font sağlayıcılarını da destekleyecektir. Arıca [yükleme stratejileri](https://github.com/vercel/next.js/issues/21555) ve `font-display` değerleri üzerinde kontrol eklemeyi planlıyoruz.

## Optimizasyonu Devre Dışı Bırakmak

Next.js'nin font'ları optimize etmesini istemiyorsanız devre dışı bırakabilirsiniz.

```jsx
// next.config.js

module.exports = {
  optimizeFonts: false,
}
```



