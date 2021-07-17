# Sayfalar

> Bu belge, Next.js'nin 9.3 ve üstü versiyonları için hazırlanmıştır. Eğer daha eski versiyonları kullanıyorsanız [önceki dokümantasyonumuz](https://nextjs.org/docs/tag/v9.2.2/basic-features/pages)a bakmalısınız.

Next.js'de sayfalar `pages` klasöründeki js, jsx, ts veya tsx dosyalarından export edilen React component'leridir. Her sayfa, dosya adına göre bir rota ile ilişkilendirilir.

**Örnek:** Eğer aşağıdaki gibi bir React component'i export eden `pages/about.js` dosyası oluşturursanız buna `/about` üzerinden erişebileceksiniz.

```jsx
function About() {
    return <div>About</div>
}
    
export default About
```

 **Dinamik Rotalı Sayfalar**

Next.js dinamik rotalı sayfaları destekler. Mesela eğer `pages/posts/[id].js` şeklinde bir dosya oluşturursanız buna `posts/1`, `posts/2` vb şekilde erişebilirsiniz.

> Dinamik routing ile ilgili daha fazla bilgi için [Dynamic Routing dokümantasyonu](https://nextjs.org/docs/routing/dynamic-routes)na bakın.

### Pre-rendering \(Ön işleme\)

Varsayılan olarak Next.js her sayfayı önceden işler. Yani sayfa HTML'ninin istemci tarafında JavaScript ile oluşturulması yerine, istemci tarafında HTML oluşturulup istemciye öyle gönderilir. Bu pre-rendering bize daha iyi bir performans ve SEO sağlar.

Oluşturulan her HTML, o sayfa için gerekli olan minimum JavaScript koduyla ilişkilendirilir. Bir sayfa tarayıcı tarafından yüklendiğinde JavaScript kodu çalışır ve sayfayı tamamen etkileşimli hale getirir. \(Bu işleme hydration/hidrasyon denir\)

#### Pre-rendering'in iki biçimi

Next.js'nin iki ön işleme \(pre-rendering\) biçimi vardır: Statik Oluşturma ve Sunucu Taraflı İşleme. Fark, HTML'nin ne zaman oluşturulduğundadır.

* \*\*\*\*[**Static Generation \(Önerilen\)**](pages.md#static-generation-recommended)**:** HTML, derleme \(build\) sırasında oluşturulur ve her request'te aynı HTML yeniden kullanılır.
* \*\*\*\*[**Server-side Rendering**](pages.md#server-side-rendering)**:** HTML, **her request**'te oluşturulur.

Next.js, her sayfa için kullanmak istediğiniz pre-rendering biçimini seçmenize izin verir. Çoğu sayfa için Statik Oluşturma'yı ve bazı sayfalar için de Sunucu Taraflı İşleme'yi kullanarak hibrit bir Next.js uygulaması oluşturabilirsiniz.

Performans için Server-side Rendering yerine [**Static Generation**](pages.md#static-generation-recommended) kullanmanızı **öneririz**. Statik oluşturulmuş sayfalar, performansı artırmak için ekstra yapılandırma olmadan CDN tarafından önbelleğe alınabilirler. Ama bazı durumlarda Server-side Rendering kullanmaya mecbur kalabilirsiniz.

Static Generation veya Server-side Rendering kullanırken Client-side Rendering de kullanabilirsiniz. Yani sayfanın bazı bölümlerini tamamen client side JavaScript ile oluşturabilirsiniz. Daha fazla bilgi için [Veri Getirme sayfası](veri-getirme.md)nı inceleyin.

### Static Generation \(Önerilen\) <a id="static-generation-recommended"></a>

Eğer bir sayfa Static Generation kullanıyorsa, sayfa HTML'si derleme sırasında, yani `next build` komutu verilince oluşturulur. Bu HTML daha sonra her istekte yeniden kullanılır ve CDN tarafından önbelleğe alınabilir.

Next.js'de oluşturduğunuz statik sayfalar data içerebilir veya içermeyebilir. Bunu biraz inceleyelim...ç

#### Verisiz Static Generation

Varsayılan olarak Next.js veriler olmadan Static Generation kullanarak sayfaları pre-render eder. Mesela:

```jsx
function About() {
    return About
}
    
export default About
```

Bu sayfa pre-render edilecek harici bir veriye ihtiyaç duymuyor. Bu gibi durumlarda Next.js derleme yaparken sayfa başına tek bir HTML dosyası oluşturur.

#### Verili Static Generation

Bazı sayfalar pre-render edilmek için harici verilere ihtiyaç duyarlar. Bununla ilgili iki senaryo var. Biri veya her ikisi birden geçerli olabilir. Her durumda, Next.js'nin sağladığı özel bir işlevi kullanabilirsiniz:

**Senaryo 1: Sayfa içeriğiniz harici verilere bağlıdır.**

**Örnek:** Blog sayfanızın bir CMS'den \(içerik yönetim sisteminden\) içerik alması gerekebilir.

```jsx
// TODO: Bu sayfa pre-render edilmeden önce `posts` değerinin 
//       gönderilmesi gerekiyor (bir API endpoint'i tarafından)
function Blog({ posts }) {
    return <ul>{
        posts.map((post) => ( <li>{post.title}</li> ))
    }</ul>
}

export default Blog
```

Bu verileri pre-render'de getirmek için Next.js, aynı dosyadan `getStaticProps` adlı `async` bir fonksiyonu `export` etmenize olanak verir. Bu fonksiyon build sırasında çağırılır ve alınan verileri pre-rendering sırasında sayfanın `props`'larına iletmenize olanak tanır.

```jsx
function Blog({ posts }) {
    // Post'lar render ediliyor...
}
    
// Bu fonksiyon derleme anında çağırılır
export async function getStaticProps() {
    // posts değerlerini almak için harici bir API endpoint'ini çağırıyoruz.
    const res = await fetch('https://.../posts')
    const posts = await res.json()
        
    // { props: { posts } } şeklinde dönen veri, derleme anında  
    // Blog component'ine `posts` olarak iletiliyor.
    return {
        props: {
            posts,
        },
    }
}
    
export default Blog
```

`getStaticProps`'un nasıl çalıştığı hakkında daha fazla bilgili edinmek için [Veri Getirme Sayfası](veri-getirme.md#basic-features_data-fetching_getstaticprops-static-generation)na bakın.

**Senaryo 2: Sayfanıza dinamik bir rotasyon ile ulaşılıyordur.**

Next.js ile dinamik rotalı sayfalar oluşturabilirsiniz. Örneğin `id` değerine bakarak tek bir blog post'unu göstermesi için `pages/posts/[id].js` yoluna sahip bir dosya oluşturabilirsiniz. Bu şekilde, `posts/1` ile sayfaya eriştiğinizde `id: 1` prop'u ile post'u gösterebilirsiniz.

> Dinamik routing ile ilgili daha fazla bilgi almak için [Dynamic Routing dokümantasyonu](https://nextjs.org/docs/routing/dynamic-routes)nu inceleyin.

Ancak, `id` değeri, harici bir kaynaktan veri almak için kullanılacak olabilir.

**Örnek:** Veritabanına yalnızca bir tane `id:1` olan blog post'u eklediğinizi varsayalım. Bu durumda `posts/1` ile gösterilecek sayfayı derleme anında buna göre düzenleyerek pre-render etmek isteyeceksiniz.

Daha sonra `id:2` değerine sahip ikinci post'u da ekleyebilirsiniz. Bu durumda da `posts/2` için de pre-render oluşturmak istersiniz.

Bu örnekte, pre-render edilmiş sayfanızın yolları harici verilere bağlıdır. Bunu halletmek için Next.js, dinamik bir sayfadan `getStaticPaths` adlı bir `async` fonksiyonu `export` etmenize izin verir \(Bu örnekte `pages/posts/[id].js`\). Bu fonksiyon, derleme sırasında çağırılır ve hangi yolları pre-render etmek istediğinizi belirtmenize olanak tanır.

```jsx
// Bu fonksiyon derleme anında çağırılacak
export async function getStaticPaths() {
    // Bir API endpoint'inden harici bir veri alıyoruz.
    const res = await fetch('https://.../posts')
    const posts = await res.json()

    // Post'lara göre pre-render etmek istediğimiz yolları alıyoruz
    const paths = posts.map((post) => ({
        params: { id: post.id },
    }))

    // Derleme anında yalnızca bu yolları yeniden oluşturacağız.
    // { fallback: false }, diğer rotaların 404'e düşmesi içindir.
    return { paths, fallback: false }
}
```

Ayrıca `page/posts/[id].js`'de bu `id`'ye sahip post hakkındaki verileri alabilmeniz ve sayfayı pre-render sırasında oluşturmanız için `getStaticProps`'u dışa aktarmanız gerekir.

```jsx
    function Post({ post }) {
        // Post'u render ediyoruz...
    }
        
    export async function getStaticPaths() {
        // derleme anında pre-render edilecek yolları belirliyoruz
    }
        
    // Bu fonksiyon da derleme anında çalışacak
    export async function getStaticProps({ params }) {
        // params, id değerini içeriyor.
        // Eğer /posts/1 şeklinde bir yol gelmişse params.id değeri 1 olacaktır.
        const res = await fetch(`https://.../posts/${params.id}`)
        const post = await res.json()
        
        // post değerini props ile sayfaya iletiyoruz.
        return { props: { post } }
    }
        
    export default Post
```

`getStaticPaths`'ın nasıl çalıştığı hakkında daha fazla bilgi edinmek için [Veri Getirme başlığı](veri-getirme.md#basic-features_data-fetching_getstaticprops-static-generation)na bakın.

#### Ne zaman Static Generation Kullanmalıyım?

Sayfanız bir kez build edilir ve CDN tarafından önbelleğe alınabilir. Yani her request'te sayfayı yeniden oluşturmak için sunucuyu yormazsınız ve sayfalara erişim çok daha hızlı olur.

Aşağıdakiler dahil birçok sayfa türü için Static Generation'u kullanabilirsiniz:

* Pazarlama sayfaları
* Blog gönderileri
* E-ticaret ürün listelemeleri
* Yardım ve dokümantasyon sayfaları

Şunu kendinize sormalısınız: "Ben bu sayfayı daha kullanıcı istemeden önce oluşturabilir miyim?" Cevabınız "evet" ise Static Generation \(Statik OLuşturma\) doğru seçimdir.

Öte yandan, kullanıcının isteğinden önce sayfanın oluşturulması mümkün değilse Static Generation iyi bir fikir değildir. Belki sayfanız sık güncellenen veriler gösteriyor ve sayfa içeriği her istekte değişiyor olabilir.

Bu gibi durumlarda aşağıdakilerden birini yapabilirsiniz:

* Static Generation'u **Client-side Rendering** ile kullanın: Bir sayfanın bazı bölümlerini pre-render etmeyi atlayabilir ve ardından bunları doldurmak için client-side JavaScript'i kullanabilirsiniz. Bu yaklaşım hakkında daha fazla bilgi edinmek için yine [Veri Getirme başlığı](veri-getirme.md)nı inceleyin.
* **Server-side Rendering** kullanın: Next.js sayfayı her istekte yeniden pre-render eder. Bu yavaş olacaktır çünkü sayfa CDN tarafından önbelleğe alınmamıştır. Ama pre-render edilen sayfanız her zaman güncel kalacaktır. Aşağıdaki başlıkta bu yaklaşım hakkında konuşacağız.

### Server-side Rendering

> "SSR" veya "Dinamic Rendering" olarak da anılır.

Eğer bir sayfa **sunucu taraflı render** yapıyorsa sayfa HTML'si **her request**te yeniden oluşturulur.

Bir sayfayı sunucu taraflı render etmek için `getServerSideProps` adlı bir `async` fonksiyonu `export` etmeniz gerekir. Bu fonksiyon her istekte sunucu tarafından çağırılır.

Mesela varsayalım ki sayfanız sık sık harici bir API tarafından güncellenerek pre-render edilmeye ihtiyaç duyuyor. Bu güncellenen verileri sayfanıza aktarmak için aşağıdaki gibi `getServerSideProps` yazabilirsiniz:

```jsx
function Page({ data }) {
    // Data render ediliyor...
}
    
// Her istekte çağırılacak fonksiyon
export async function getServerSideProps() {
    // Harici bir API'dan veri çekiyor
    const res = await fetch(`https://.../data`)
    const data = await res.json()
    
    // Veriyi props ile sayfaya iletiyor
    return { props: { data } }
}
    
export default Page
```

Gördüğünüz gibi `getServerSideProps`, `getStaticProps`'a benzer. Aralarındaki fark: `getServerSideProps` derleme anında sayfanın oluşturulması yerine her istekte sayfanın oluşturulmasını sağlar.

`getServerSideProps`'un nasıl çalıştığını öğrenmek için [Veri Getirme başlığı](veri-getirme.md#basic-features_data-fetching_getserversideprops-server-side-rendering)na bakın.

### Özet <a id="basic-features_summary"></a>

Next.js'nin iki pre-render biçimini gördük.

* **Static Generation \(Önerilen\):** HTML, build sırasında oluşturulur ve her istekte yeniden kulanılır. Bir sayfanın Static Generation kullanmasını sağlamak için ya sayfa component'ini `export` edin ya da `getStaticProps` \(veya gerekliyse `getStaticPaths`\) export edin. Bu yöntem, kullanıcının isteğinden önce hazır edilebilecek sayfalar için harikadır. Bu sayfaya ek veriler getirmek isterseniz Client-side Rendering de kullanabilirsiniz.
* **Server-side Rendering:** HTML, **her request**'te yeniden oluşturulur. Sayfayı Server-side Rendering ile oluşturacaksanız `getServerSideProps`'u `export` edersiniz. Server-side Rendering, Static Generation'a göre daha yavaş kalacağı için bunu yalnızca gerekliyse kullanmalısınız.

### Daha Fazlasını Öğrenin

Buradan sonra aşağıdakilerden biriyle devam etmenizi öneririz:

* \*\*\*\*[**Veri Getirme:** Next.js'nin veri işlemesiyle ilgili daha fazlasını öğrenin.](veri-getirme.md)
* [**Preview Mode:** Önizleme modu hakkında daha fazlasını öğrenin.](https://nextjs.org/docs/advanced-features/preview-mode)
* [**Routing:** Next.js'te routing ile ilgili daha fazlasını öğrenin.](https://nextjs.org/docs/routing/introduction)
* [**TypeScript:** Sayfalarınıza TypeScript ekleyin.](https://nextjs.org/docs/basic-features/typescript#pages)

