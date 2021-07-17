# Başlarken

Next.js dokümantasyonuna hoşeldiniz!

Next.js'te yeniyseniz [öğretici kurs](https://nextjs.org/learn/basics/create-nextjs-app)la başlamanızı tavsiye ederiz.

Kısa sınavlar içeren etkileşimli kurs, Next.js'yi kullanmak için bilmeniz gereken her şeyde size rehberlik edecek.

Next.js'yle ilgili herhangi bir sorunuz olursa [GitHub Tartışmalar](https://github.com/vercel/next.js/discussions)'da topluluğumuza sorabilirsiniz.

**Sistem Gereklilikleri**

* Node.js 12.0 veya üstü
* MacOs, Windows ve Linux destekleniyor.

### Kurulum

`create-next-app` kullanarak sizin için her şeyi otomatik olarak hazırladığımız kurulumu tercih etmenizi öneririz.

```bash
npx create-next-app
# veya
yarn create next-app
```

Eğer TypeScript projesi olarak başlamak isterseniz de `--typescript` flag'ını kullanabilirsiniz.

```bash
npx create-next-app --typescript
# veya
yarn create next-app --typescript
```

Kurulum tamamlandıktan sonra geliştirici sunucusunda başlatmak için talimatları izleyin. `pages/index.js`'yi düzenleip ve sonucu tarayıcınızda görün.

`create-next-app` kullanımıyla ilgili daha fazla bilgi için ["create-next-app" dokümantasyonu](https://nextjs.org/docs/api-reference/create-next-app)nu inceleyebilirsiniz.

### Manuel Kurulum

Projenize `nextreactreact-dom` kurun:

```bash
npm install next react react-dom
# veya
yarn add next react react-dom
```

`package.json` dosyasını açın ve `scripts` objesine şunları ekleyin:

```javascript
"scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
}
```

Bu komutlar, uygulama geliştirmenin farklı aşamalarını temsil ederler:

* `dev` - `next dev` komutu ile Next.js'yi development modunda çalıştırır.
* `build` - `next build` komutu ile Next.js'yi production moduna hazırlar.
* `start` - `next start` komutu ile Next.js'yi production modunda çalıştırır.
* `lint` - Runs `next lint` which sets up Next.js' built-in ESLint configuration \(çeviremedim\)

Next.js, sayfa kavramı üzerine inşa edilmiştir. Bir sayfa, `pages` dizinindeki bir js, jsx, ts veya tsx dosyasından export edilen bir [React Component](https://reactjs.org/docs/components-and-props.html)'idir.

Sayfalar, dosya adlarına göre bir rota ile ilişkilendirilir. Örneğin `pages/about.js` yolu `/about` url'i ile eşlenir. Dosya adlarını uygun şekilde vererek dinamik rota parametreleri bile verebilirsiniz.

Projeniz içinde `pages` klasörü oluşturun.

`./pages/index.js` dosyasını aşağıdaki içerikle doldurun:

```jsx
function HomePage() {
    return <div>Welcome to Next.js!</div>
}
    
export default HomePage
```

Ve projenizi development modunda çalıştırmak için `npm run dev` veya `yarn dev` komutunu çalıştırın. Projeniz `http://localhost:3000` üzerinde çalışacaktır.

Şimdiye kadar şunları elde ettik:

* Otomatik derleme ve paketleme \(webpack ve babel ile\)
* React Fast Refresh
* `./pages` klasörüyle statik ve server-side sayfa render etme
* `./public` klasörüyle `/` ile eşlenmiş statik dosya serve etme. \(Bunu ne ara elde ettik bilmiyorum\)

Ayrıca, herhangi bir Next.js uygulamasını başlangıçtan production aşamasına kadar hazırlamayı gördük. Daha fazlasını [Deployment \(Dağıtım\) dokümantasyonumuz](https://nextjs.org/docs/deployment)dan okuyun.

Bundan sonra neler yapılacağı hakkında daha fazla bilgi için aşağıdaki bölümleri öneriyoruz:

* [**Sayfalar:** Next.js'de sayfaların ne olduğu hakkında daha fazlasını öğrenin.](temel-ozellikler/pages.md)
* [**CSS Desteği:** Uygulamanıza özel stiller eklemek için yerleşik \(built-in\) CSS desteğini kullanın.](temel-ozellikler/built-in-css-support.md)
* [**CLI:** Next.js'nin CLI'sı hakkında daha fazla bilgi edinin.](https://nextjs.org/docs/api-reference/cli)

