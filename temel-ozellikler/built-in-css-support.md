# Yerleşik CSS Desteği

Next.js, bir JavaScript dosyasından CSS dosyalarını `import` etmenize olanak tanır. Next.js, `import` kavramını JavaScript'in ötesine taşıdığı için bu mümkün olmaktadır.

### Adding a Global Stylesheet \(Global Bir Stylesheet Eklemek\) <a id="basic-features_built-in-css-support_adding-a-global-stylesheet"></a>

Uygulamanızda bir stil eklemek için CSS dosyasını `pages/_app.js` dosyasına `import` edin.

Örneğin, `styles.css` adında şu içeriğe sahip bir stil dosyanız olsun:

```css
body {
    font-family: 'SF Pro Text', 'SF Pro Icons', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
    padding: 20px 20px 60px;
    max-width: 680px;
    margin: 0 auto;
}
```

Sonra da eğer yoksa `pages/_app.js` dosyası oluşturun ve içine `styles.css` dosyanızı `import` edin.

```jsx
import '../styles.css'

// Bu export işlemi 'pages/_app.js'de zorunludur.
export default function MyApp({ Component, pageProps }) {
    return 
}
```

Bu stiller \(`styles.css`\), uygulamanızdaki tüm sayfalara ve component'lere uygulanacaktır. Stil sayfalarının genel doğası gereği çakışmaları önlemek için bunları yalnızca `pages/_app.js` içinde `import` edebilirsiniz.

Development aşamasında, stil sayfalarını bu şekilde ifade etmek, siz onları düzenlerken stillerinizin anında yüklenmesini sağlar. Yani bu dosyada değişiklik yaptığınızda uygulamanızda o anda bulunan state'ler korunmuş olur.

Production'dayken tüm CSS dosyaları otomatik olarak tek bir minify edilmiş `.css` dosyasında birleştirilir.

#### Import styles from `node_modules` \(`node_modules`'den stil import etmek\)

Next.js, **9.5.4** versiyonundan itibaren `node_modules` içinden uygulamanızın herhangi bir yerine CSS `import` etmenize izin vermektedir.

Bootstrap veya nprogress gibi global stil sayfaları için dosyayı `pages/_app.js` içine aktarmalısınız. Örneğin:

```jsx
// pages/_app.js
import 'bootstrap/dist/css/bootstrap.css'

export default function MyApp({ Component, pageProps }) {
    return 
}
```

Third party bir component'in gerektirdiği CSS'yi `import` etmek istediğinizde bunu component'iniz içinde de yapabilirsiniz. Örneğin:

```jsx
// components/ExampleDialog.js
import { useState } from 'react'
import { Dialog } from '@reach/dialog'
import VisuallyHidden from '@reach/visually-hidden'
import '@reach/dialog/styles.css'

function ExampleDialog(props) {
    const [showDialog, setShowDialog] = useState(false)
    const open = () => setShowDialog(true)
    const close = () => setShowDialog(false)

    return (
        
    )
}
```

### Adding Component-Level CSS \(Component Seviyesinde CSS Eklemek\) <a id="basic-features_built-in-css-support_adding-component-level-css"></a>

Next.js, `[name].module.css` dosya adlandırma kuralını kullanan CSS modüllerini destekler.

CSS modülleri, otomatik olarak benzersiz bir class adı oluşturarak CSS'yi yerel olarak kapsar. Bu, aynı CSS sınıf adını, çakışmalardan endişe etmeden farklı dosyalarda kullanmanıza olanak tanır.

CSS modülü, uygulmanızın herhangi bir yerinde `import` edilebilir.

Örneğin, `components/` klasöründe `Button` adlı bir component olduğunu düşünün:

Öncelikle `components/Button.module.css` dosyası oluşturup içine şunları yazın:

```css
/*
.error class'ının başka bir .css veya .module.css dosyasındaki
aynı isimli class'larla çakışmasından endişe etmenize gerek yok.
*/
.error {
    color: white;
    background-color: red;
}
```

Ardından, yukarıdaki CSS dosyasını içe aktarıp kullanarak `components/Button.js` oluşturun:

```jsx
import styles from './Button.module.css'

export function Button() {
    return (
        <button
            type="button"
            // "error" class'ına styles objesinin bir 
            // property'i gibi erişildiğine dikkat edin.
            className={styles.error}
        >
            Destroy
        </button>
    )
}
```

CSS Modülleri isteğe bağlı bir özelliktir ve sadece `.module.css` uzantısına sahip dosyalar için etkinleştirilir. Yani normal `<link ... />` stil sayfaları ve global CSS dosyaları halen desteklenmektedir.

Production aşamasındaysanız tüm CSS Modulü dosyaları otomatik olarak küçültülmüş ve bir `.css` dosyasında birleştirilmiştir. Böylece uygulamanızı stillendirmek için kullanıcının tarayıcısına minimum miktarda CSS gönderilmiş olur.

### Sass Desteği <a id="basic-features_built-in-css-support_sass-support"></a>

Next.js hem `.scss` hem de `.sass` uzantılarını kullanarak Sass'ı içe aktarmanıza olanak tanır. Component düzeyinde Sass'ı, `.module.scss` veya `.module.sass` uzantılarıyla kullanabilirsiniz.

Next.js'nin yerleşik Sass desteğini kullanmadan önce `sass`'ı kurduğunuzdan emin olun:

```bash
npm install sass
```

Sass desteği, yukarıda ayrıntıları verilen yerleşik CSS desteğiyle aynı avantajlara ve kısıtlamalara sahiptir:

> **Not:** Sass, her biri kendi uzantısına sahip [iki farklı syntax](https://sass-lang.com/documentation/syntax)'ı destekler. `scss` uzantısında [SCSS syntax](https://sass-lang.com/documentation/syntax#scss)'ını kullanmanız gerekirken `sass` uzantısında [Girintili Syntax \("Sass"\)](https://sass-lang.com/documentation/syntax#the-indented-syntax) kullanmanız gerekir.
>
> Hangisini seçeceğinizden emin değilseniz, CSS'nin bir üst kümesi olan ve girintili söz dizimini \("Sass"\) gerektirmeyen `.scss` uzantısıyla başlayın.

#### Customizing Sass Options \(Sass Ayarlarını Özelleştirmek\) <a id="basic-features_built-in-css-support_sass-support_customizing-sass-options"></a>

Sass derleyicisini yapılandırmak istiyorsanız, `next.config.js`'de `sassOptions`'u kullanabilirsiniz.

Örneğin `includePaths` eklemek için:

```jsx
const path = require('path')

module.exports = {
    sassOptions: {
        includePaths: [path.join(__dirname, 'styles')],
    },
}
```

### CSS-in-JS <a id="basic-features_built-in-css-support_css-in-js"></a>

Satır içi stil kullanma örneği:

```jsx
function HiThere() {
    return <p style={{ color: 'red' }}>hi there</p>
}

export default HiThere
```

`styled-jsx` örneği:

```jsx
function HelloWorld() {
    return (
        <div>Hello world<p>scoped!</p>
        <style jsx>{`
            p { color: blue; }
            div { background: red; }
            @media (max-width: 600px) {
                div { background: blue; }
            }
        `}</style>
        <style global jsx>{`
            body { background: black; }
        `}</style>
        </div>
    )
}

export default HelloWorld
```

[`styled-jsx dokümantasyonu`](https://github.com/vercel/styled-jsx)ndan daha fazlasını öğrenebilirsiniz.

#### **JavaScript aktif edilmemişse yine de çalışır mı?**

Evet, `next start` ile başlattığınız production aşamasındaysanız JavaScript disable edilmiş olsa bile CSS yüklenecektir. Ama development aşamasında Fast Refresh için JavaScript'in aktif edilmiş olması gerekir.

### İlişkili Konular

Buradan sonra aşağıdakilerden biriyle devam etmenizi öneririz:

* [**PostCSS Yapılandırmasını Özelleştirmek:** Next.js tarafından eklenen PostCSS yapılandırmasını ve eklentilerini kendinize göre genişletin.](https://nextjs.org/docs/advanced-features/customizing-postcss-config)

