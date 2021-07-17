# Veri Getirme

> Bu doküman Next.js'nin 9.3 versiyonu veya üstü için yazılmıştır. Eğer daha eski versiyon kullanıyorsanız [Önceki Dokümantasyon](https://nextjs.org/docs/tag/v9.2.2/basic-features/data-fetching)umuza bakmalısınız.

[Sayfalar başlığı](pages.md)nda Next.js'nin pre-render için iki biçimi olduğunu açıkladık: **Static Generation** ve **Server-side Rendering**. Bu başlıkta, her durum için veri toplama stratejileri hakkında derinlemesine konuşacağız. Henüz okumadıylsanız [Sayfalar başlığı](pages.md)nı okumanızı öneririz.

Pre-render için veri almakta kullanabileceğiniz üç benzersiz Next.js fonksiyonundan bahsedeceğiz:

* [`getStaticProps`]() \(Static Generation\): Veri build yapılırken getirilir.
* [`getStaticPaths`]() \(Static Generation\): Verilere dayalı olarak sayfaları pre-render etmek üzere özel dinamik yollar \([dynamic routes](https://nextjs.org/docs/routing/dynamic-routes)\) belirtilir.
* [`getServerSideProps`]() \(Server-side Rendering\): Veri **her request'te** yenilenir.

Ek olarak, client-side'de verilerin nasıl getirileceğinden \(fetch edileceğinden\) kısaca bahsedeceğiz.

### `getStaticProps` \(Static Generation\) <a id="basic-features_data-fetching_getstaticprops-static-generation"></a>

> Versiyon Geçmişi
>
> * **v10.0.0:** `locale`, `locales`, `defaultLocale` ve `notFound` seçenekleri eklendi.
> * **v9.5.0:** Kararlı [Incremental Static Regeneration](https://nextjs.org/blog/next-9-5#stable-incremental-static-regeneration)
> * **v9.3.0:** `getStaticProps` tanıtıldı.

Bir sayfadan `getStaticProps` adlı `async` bir fonksiyon `export` ederseniz, Next.js `getStaticProps` tarafından döndürülen öğeleri kullanarak build zamanında bu sayfayı pre-render eder.

```jsx
export async function getStaticProps(context) {
    return {
        props: {}, // sayfa component'ine props olarak geçecek veriler
    }
}
```

Buradaki `context` parametresi şu anahtar değerleri içeren bir objedir:

* `params` dinamik routing için route parametrelerini içerir. Mesela eğer sayfa adı `[id].js` ise `params` değeri `{ id: ... }` şeklinde görünecektir. Daha fazlasını öğrenmek için [Dynamic Routing dokümantasyonu](https://nextjs.org/docs/routing/dynamic-routes)na bakabilirsiniz. Bunu daha sonra anlatacağımız `getStaticPaths` ile birlikte kullanmalısınız.
* `preview` değeri, eğer sayfa preview modundaysa `true` olur. Aksi halde `undefined`'dir.
* `previewData` değeri, `setPreviewData` tarafından ayarlanan önizleme verilerini içerir.
* `locale`, aktif local ayarları içerir \(etkinleştirilmişse\)
* `locales`, desteklenen tüm local ayarları içerir \(etkinleştirilmişse\)
* `defaultLocale`, varsayılan local ayarları gösterir \(etkinleştirilmişse\)

`getStaticProps` şu değerleri içeren bir obje return etmelidir:

* `props` - Sayfa component'i tarafından alınacak değerlere sahip, **isteğe bağlı** bir nesnedir. Bu nesne [serializable](https://en.wikipedia.org/wiki/Serialization) olmalıdır.
* `revalidate` - Sayfanın yeniden oluşturulabilmesi için saniye cinsinden **isteğe bağlı** bir miktar \(varsayılan olarak `false` veya yeniden doğrulama yok\). Daha fazlası için: [Incremental Static Regeneration](veri-getirme.md#basic-features_data-fetching_incremental-static-regeneration)
*  `notFound` - Sayfanın 404 durumu ve sayfası döndürmesine izin vermek için **isteğe bağlı** bir `boolean` değerdir. Aşağıda nasıl çalıştığına dair bir örnek verilmiştir:

  ```text
  export async function getStaticProps(context) {
      const res = await fetch(`https://.../data`)
      const data = await res.json()
    
      if (!data) { return { notFound: true, } }
    
      return { props: { data }, } // sayfa component'ine prop olarak geçilecek veri
  }
  ```

  > **Not:** Bu örnekte `notFound`'un `fallback: false`'ye ihtiyacı yoktur çünkü sadece `getStaticPaths`'ten dönen yollar pre-render edilecektir çünkü diğerlerinin pre-render edilmesinin yolu `if` bloğunda engellenmiştir.

  > **Not:** `notFound: true` ile, sayfa başarıyla oluşturulmuş olsa bile 404 döndürecektir. Bu, kullanıcı tarafından oluşturulan oluşturulan içeriğin yazar tarafından kaldırılması gibi kullanımları desteklemek içindir.

*  `redirect` - Dahili veya harici kaynaklara yönlendirmeye izin vermek için **isteğe bağlı** `{ destination: string, permanent: boolean }` biçiminde bir değerdir. Bazı ender durumlarda, eski HTTP istemcilerinin doğru şekilde yeniden yönlendirme yapması için özel bir durum kodu atamanız gerekebilir. Bu durumlarda `permanent` property'i yerine `statusCode` property'i özelliğini kullanabilirsiniz \(ama ikisini aynı anda kullanamazsınız\). Aşağıda nasıl çalıştığına dair bir örnek verilmiştir:

  ```text
  export async function getStaticProps(context) {
      const res = await fetch(`https://...`)
      const data = await res.json()
    
      if (!data) {
          return {
              redirect: { destination: '/', permanent: false, },
          }
      }
    
      return {
          props: { data }, // sayfa component'ine props olarak gidecek data
      }
  }
  ```

  > **Not:** Şu anda build sırasında redirect'e izin verilmemektedir ve yönlendirmeler build anında biliniyorsa `next.config.js`'ye eklenmelidir.

> **Not:** `getStaticProps` içinde üst seviye `module`'leri `import` edip kullanabilirsiniz.
>
> Yani, sunucu taraflı kodu doğrudan `getStaticProps`'ta kullanabilirsiniz. Mesela dosya sisteminizden veya veritabanından okuma gibi işlemleri burada yapabilirsiniz.
>
> `getStaticProps`'ta kullanılan `import`'lar client-side için bundle edilmeyecektir \(paketlenmeyecektir\).

> **Not:** `getStaticProps` içinde bir API yoluna erişmek için `fetch()` kullanmamalısınız. Bunun yerine API rotanızda kullandığınız mantığı doğrudan `import` edin. Bu yaklaşım için kodunuzu biraz yeniden düzenlemeniz gerekebilir.
>
> Harici bir API'den veri almak sorun değil!

#### Basit Bir Örnek

Burada bir CMS'den blog post'ları listesini alan bir `getStaticProps` örneği yaptık. Bu örnek aynı zamanda [Sayfalar başlığı](pages.md)nda da bulunmaktadır.

```jsx
// post'lar build sırasında getStaticProps() tarafından doldurulacak
function Blog({ posts }) {
    return <ul>{posts.map((post) => <li>{post.title}</li>)}</ul>
}

// Bu fonksiyon build sırasında sunucu tarafında çağırılır.
// İstemci tarafında çağırılmayacak. Böylece siz doğrudan
// veritabanı sorgularınızı bile burada yapabilirsiniz.
// "Technical details" bölümüne bakın.
export async function getStaticProps() {
    // Harici bir API'dan post'lar alınıyor.
    // Herhangi bir veri fetch etme kütüphanesini kullanabilirsiniz.
    const res = await fetch('https://.../posts')
    const posts = await res.json()

    // { props: { posts } } döndürerek, build sırasında 
    // Blog component'ine post'ları gönderiyoruz.
    return { props: { posts, }, }
}

export default Blog
```

#### Ne zaman `getStaticProps` Kullanmalıyız?

Eğer şunları yapacaksanız `getStaticProps` kullanmalısınız:

* Sayfa oluşturmak için gereken veriler, kullanıcının request'inden önce \(build sırasında\) zaten elimizdeyse,
* Veriler bir içerik yönetim sisteminden geliyorsa,
* Veriler herkese açık olarak önbelleğe alınabilirse \(kullanıcıya özel değilse\),
* Sayfa SEO için önceden oluşturulmalıysa ve çok hızlı olmalıysa.

#### Incremental Static Regeneration \(ISR\) \(Artımlı Statik Yenileme\) <a id="basic-features_data-fetching_incremental-static-regeneration"></a>

> Versiyon Geçmişi
>
> * **v9.5.0:** Base Path eklendi.

Next.js, sitenizi oluşturduktan sonra statik sayfalar oluşturmanıza veya güncellemenize olanak tanır. **ISR**, **tüm siteyi yeniden oluşturmaya gerek kalmadan** sayfa başına statik oluşturmayı kullanmanızı sağlar. ISR ile, milyonlarca sayfaya ölçeklendirme yaparken statik sayfaya sahip olmanın avantajlarını da korursunuz.

Önceki `getStaticProps` örneğini düşünün, ama `revalidate` property'i aracılığıyla ISR etkinleştirilmiş olsun:

```jsx
function Blog({ posts }) {
    return <ul>{ posts.map((post) => <li>{post.title}</li>)}</ul>    
}
    
// Bu fonksiyon build sırasında server-side tarafında çağırılacak.
// Ama eğer revalidation etkinleştirildiyse ve yeni bir istek gelirse
// sunucusuz (serverless) bir fonksiyonda tekrar çağırılabilir.
export async function getStaticProps() {
    const res = await fetch('https://.../posts')
    const posts = await res.json()
    
    return {
        props: {
            posts,
        },
        // Next.js şu durumlarda sayfayı yeniden oluşturacak:
        // - Bir istek geldiğinde
        // - Her 10 saniyede bir
        revalidate: 10, // saniye cinsinden
    }
}
    
// Bu fonksiyon, build sırasında server-side tarafında çağırılır.
// Ama eğer path oluşturulmamışsa serverless bir fonksiyonda 
// tekrar çağırılabilir.
export async function getStaticPaths() {
    const res = await fetch('https://.../posts')
    const posts = await res.json()
    
    // Post'lara göre pre-render edilecek yolları alıyoruz.
    const paths = posts.map((post) => ({
        params: { id: post.id },
    }))
    
    // Build sırasında sadece bu yolları pre-render edeceğiz.
    // Eğer path bulunamazsa { fallback: blocking } ile sunucu 
    // ilk request'le birlikte sayfayı oluşturacak.
    return { paths, fallback: 'blocking' }
}
    
export default Blog
```

Build ile pre-render edilmiş bir sayfa istek yapıldığında başlangıçta önbelleğe alınmış sayfayı gösterecektir.

* ilk istekten sonra ve maksimum 10 saniyeden önce sayfaya yapılan tüm istekler de önbelleğe alınır.
* 10 sn'lik döngüden sonra bir sonraki istek halen önbelleğe alınmış \(eski\) sayfayı göndermeye devam edecektir.
* Next.js arka planda sayfanın yenilenmesini tetikler.
* Sayfa başarıyla oluşturulduktan sonra Next.js önbelleği geçersiz kılar ve güncellenmiş ürün sayfasını gösterir. Arka plandaki yenileme başarısız olursa eski sayfa değişmeden kalır.

Oluşturulmamış bir yola istek yapıldığında Next.js, ilk istekte sayfayı sunucu tarafında oluşturur. Sonraki istekler statik dosyayı önbellekten sunacaktır.

Önbelleğin global olarak nasıl kalıcı hale getirileceğini ve geri almaların nasıl yönetileceğini öğrenmek için [ISR](https://vercel.com/docs/next.js/incremental-static-regeneration) hakkında daha fazla bilgi edinin.

#### Reading files: Use `process.cwd()`

Dosyalar doğrudan `getStaticProps` içindeki dosya sisteminden okunabilir.

Bunu yapmak için dosyanın tam yolunu almanız gerekir.

Next.js kodunuzu ayrı bir klasöre derlediğinden `__dirname`'yi kullanamazsınız. Çünkü bunun döneceği yol, `pages` dizininden farklı olacaktır.

Bunun yerine size Next.js'nin çalıştırıldığı dizini veren `process.cwd()`'yi kullanabilirsiniz.

```jsx
import { promises as fs } from 'fs'
import path from 'path'

// posts değeri build sırasında getStaticProps tarafından doldurulacak.
function Blog({ posts }) {
    return (
        <ul>
            {posts.map((post) => (
                <li>
                    <h3>{post.filename}</h3>
                    <p>{post.content}</p>
                </li>
            ))}
        </ul>
    )
}

// Bu fonksiyon build sırasında sunucu tarafında çağırılacak.
// İstemci tarafında çağırılmayacak. Bu yüzden direkt veritabanı
// sorguları yazabilirsiniz. "Technical details" bölümüne bakın.
export async function getStaticProps() {
  const postsDirectory = path.join(process.cwd(), 'posts')
  const filenames = await fs.readdir(postsDirectory)

  const posts = filenames.map(async (filename) => {
    const filePath = path.join(postsDirectory, filename)
    const fileContents = await fs.readFile(filePath, 'utf8')

    // Genellikle içeriği ayrıştırırsınız/dönüştürürsünüz.
    // Mesela markdown'ları html taglarına dönüştürürsünüz.

    return {
      filename,
      content: fileContents,
    }
  })
  // { props: { posts } } return ederek build sırasında
  // Blog component'ine verileri props üzerinden ileteceğiz.
  return {
    props: {
      posts: await Promise.all(posts),
    },
  }
}

export default Blog
```

#### Teknik Detaylar <a id="basic-features_data-fetching_technical-details"></a>

**Sadece build sırasında çalışır**

`getStaticProps` build sırasında çalıştığından, statik HTML oluşturduğu için query parametreleri ya da HTTP header'ları gibi sadece istek sırasında oluşturulabilen verileri almaz.

**Direkt server-side kodu yazın**

`getStaticProps` yalnızca sunucu tarafında çalışır. Asla istemci tarafında çalıştırılmayacaktır. Tarayıcı için JS paketine bile dahil edilmeyecektir. Bu, tarayıcılara gönderilmeden doğrudan veritabanı sorguları gibi kodlar yazabileceğiniz anlamına geliyor. `getStaticProps`'tan bir API yolu almamalısınız. Bunun yerine sunucu tarafı kodunu doğrudan `getStaticProps`'a yazabilirsiniz.

Next.js'nin istemci tarafı paketinden neleri çıkardığını doğrulamak için [bu aracı](https://next-code-elimination.vercel.app/) kullanabilirsiniz.

**Hem HTML'yi hem JSON'u statik olarak üretir**

`getStaticProps` içeren bir sayfa build sırasında pre-render edilerek oluşturulduğunda, sayfanın HTML dosyasına ek olarak Next.js, `getStaticProps` çalıştırmanın sonucunu tutan bir JSON dosyası oluşturur.

Bu JSON dosyası, `next/link` [\(dokümantasyon\)](https://nextjs.org/docs/api-reference/next/link) veya `next/router` [\(dokümantasyon\)](https://nextjs.org/docs/api-reference/next/router) üzerinden istemci taraflı sayfa yönlendirmelerinde kullanılacaktır. `getStaticProps` kullanarak pre-render edilmiş bir sayfaya gittiğinizde Next.js bu JSON dosyasını \(build sırasında önceden hesaplanır\) alır ve bunu sayfa component'i için props olarak kullanılır. Yani yalnızca bu dışa aktarılan JSON dosyası kullanıldığı için `getStaticProps` fonksiyonu isteklere cevap vermek için çağırılmamaktadır.

Incremental Static Generation \(ISR\) kullanırken istemci tarafında navigasyon için gereken JSON'u oluşturmak için `getStaticProps` bant dışında çalıştırılır. Bunu, aynı sayfa için bir sürü istek yapılıyor diye düşünebilirsiniz. Ama bunun son kullanıcın gördüğü performans üzerine etkisi yoktur. \(Çünkü zaten son kullanıcı tüm navigasyonu önbelleğe alınmış içerikler üzerinde gerçekleştirmektedir. \)

**Sadece sayfalarda kullanılır**

`getStaticProps` sadece bir sayfaden export edilebilir. Sayfa olmayan bir dosyadan import edemezsiniz.

Bu kısıtlamanın nedenlerinden biri, React'ın sayfa oluşturulmadan önce gerekli tüm verilere sahip olması gerekliliğidir.

Ayrıca `export async function getStaticProps() {}` kullanmalısınız. Yani özellikle sayfa bileşeninden bağımsız bir fonksiyon olarak export etmelisiniz. Sayfanın bir property'i olarak \(propTypes'ı ekler gibi\) oluşturmaya çalışırsanız çalışmaz.

**Development modundayken her request'te çalışır.**

`next dev` ile çalıştırdığınız development modundayken `getStaticProps` her request'te çalışacaktır.

**Önizleme Modu**

Bazı durumlarda, Staic Generation'u geçici olarak atlamak ve sayfayı build sırasında değil de request sırasında oluşturmak isteyebilirsiniz. Örneğin bir CMS kullanıyor olabilirsiniz ve taslağınızı yayınlamadan önce önizlemek isteyebilirsiniz.

Bu kullanım durumu Next.js tarafından **Preview Mode** adı verilen özellik olarak desteklenir. Önizleme modu hakkında daha fazla bilgi için [Preview Mode dokümantasyonu](https://nextjs.org/docs/advanced-features/preview-mode)nu inceleyin.

### `getStaticPaths` \(Static Generation\) <a id="basic-features_data-fetching_getstaticpaths-static-generation"></a>

> Versiyon Geçmişi
>
> * **v9.5.0:** Kararlı [Incremental Static Regeneration](https://nextjs.org/blog/next-9-5#stable-incremental-static-regeneration)
> * **v9.3.0:** `getStaticPaths` tanıtıldı.

Bir sayfanın dinamik yolları [\(dokümantasyon\)](https://nextjs.org/docs/routing/dynamic-routes) varsa ve `getStaticProps` kullanılıyorsa, oluşturma sırasında HTML'ye dönüştürülmesi gereken yolların bir listesini tanımlaması gerekir.

Dinamik yollar kullanan bir sayfadan `getStaticPaths` adlı bir `async` fonksiyonu `export` ederseniz Next.js `getStaticPaths` tarafından belirtilen tüm yolları statik olarak pre-render edecektir.

```jsx
export async function getStaticPaths() {
    return {
        paths: [
            { params: { ... } } // Aşağıdaki "paths" başlığına bakın
        ],
        fallback: true or false // Aşağıdaki "fallback" başlığına bakın
    };
}
```

#### `paths` key \(zorunlu\) <a id="basic-features_data-fetching_the-paths-key-required"></a>

`paths` hangi yolların pre-render edileceğini belirler. Örneğin `pages/posts/[id].js` adında dinamik yollar kullanan bir sayfanız olduğunu varsayalım. Bu sayfadan `getStaticPaths`'ı `export` ederseniz ve `paths` için aşağıdakileri döndürürseniz:

```javascript
return {
    paths: [
        { params: { id: '1' } },
        { params: { id: '2' } }
    ],
    fallback: ...
}
```

Next.js `paths/1` ve `paths/2` için statik sayfalar üretir.

`params` ile gönderilen her bir key'in sayfa adıyla eşleşmesi gerektiğini unutmayın:

* Eğer sayfa adı `pages/posts/[postId]/[commentId]` ise `params`'ın `postId` ve `commentId` key'lerini içermesi gerekir.
* Eğer sayfa adı örneğin `pages/[...slug]` gibi tüm yolları yakalamaya çalışıyorsa `params` değeri `slug` key'ini dizi olarak içermelidir. Örneğin bu dizi `['foo','bar']` ise Next.js `/foo/bar` yolu için statik sayfa üretecektir.
* Eğer sayfa isteğe bağlı olarak tüm yolları yakalama yapmaya çalışıyorsa en kök yolu sağlamak için `null`, `[]`, `undefined` veya `false` değerini sağlayın. Örneğin `pages/[[...slug]]` için `slug: false` derseniz Next.js `/` yolu için statik sayfa üretecektir.

#### `fallback` key \(zorunlu\) <a id="basic-features_data-fetching_the-fallback-key-required"></a>

`getStaticPaths` fonksiyonu boolean türünden `fallback` key'i içeren bir obje return etmelidir. \(ing:fallback, tr:yedek\)

**fallback: false**

Eğer `fallback: false` ise `getStaticPaths` tarafından return edilmeyen tüm yollar 404'e çıkar. Bunu, pre-render edeceğiniz yolların sayısı azsa yapabilirsiniz. Böylece tüm yollar build sırasında statik olarak oluşturulur. Projeye sık sık yeni sayfalar eklenmediği durumlarda kullanışlıdır. Veri kaynağına daha fazla öğe eklerseniz ve yeni sayfalar oluşturmanız gerekiyorsa yeniden build etmeniz gerekir.

Aşağıda, `pages/posts/[id].js` ile sayfa başına blog post'larını pre-render eden bir örnek verilmiştir. Blog gönderilerinin listesi bir CMS'den alınıp `getStaticPaths` tarafından return edilmektedir. Ardından, her sayfa için post'ları `getStaticProps` kullanarak CMS'den alır. Bu örnek aynı zamanda [Sayfalar başlığı](pages.md)nda da bulunmaktadır.

```jsx
// pages/posts/[id].js

function Post({ post }) {
    // post'u render ediyoruz.
}

// Bu fonksiyon build sırasında çağırılıyor
export async function getStaticPaths() {
    // Harici bir API endpoint'inden veriler alınıyor
    const res = await fetch('https://.../posts')
    const posts = await res.json()

    // Post başına pre-render edilmesini istediğimiz yolları belirliyoruz
    const paths = posts.map((post) => ({
        params: { id: post.id },
    }))

    // Bu yolları sadece build sırasında pre-render edeceğiz.
    // { fallback: false } ile, build sırasında oluşturmadığımız sayfaların 404'e düşsün, diyoruz.
    return { paths, fallback: false }
}

// Bu derleme sırasında çağırılır
export async function getStaticProps({ params }) {
    // params, post id'sini içeriyor.
    // Eğer yol "/posts/1" ise params.id değeri 1 olur.
    const res = await fetch(`https://.../posts/${params.id}`)
    const post = await res.json()

    // Verileri props üzerinden sayfa component'ine iletiyoruz.
    return { props: { post } }
}

export default Post
```

**fallback: true**

Eğer `fallback: true` ise `getStaticPaths`'ın davranışı değişir:

* `getStaticPaths`'tan döndürülen yollar, build sırasında `getStaticProps` tarafından HTML'ye dönüştürülecektir.
* Build sırasında oluşturulmayan yollar 404 sayfasıyla sonuçlanmayacaktır. Bunun yerine Next.js böyle bir yola yönelik ilk istekte sayfanın "fallback" bir sürümünü sunar. Ayrıntılı bilgi için aşağıdaki "Fallback pages"e bakın.
* Arka planda Next.js, istenen HTML ve JSON yolunu statik olarak oluşturur. \(Bu, arka planda `getStaticProps`u çalıştırmayı da içerir\)
* Bu yapıldığında, tarayıcı oluşturulan yol için JSON'u alır. Kullanıcının bakış açısından, fallback \(yedek\) sayfa, yeni oluşturulan sayfa ile değiştirilir.
* Aynı amanda Next.js bu yolu pre-render edilmiş sayfalar listesine ekler. Aynı yola yapılan sonraki istekler, build sırasında pre-render edilen diğer sayfalar gibi olacaktır.

> `fallback: true`, [`next export`](https://nextjs.org/docs/advanced-features/static-html-export) kullanımını desteklemez.

**Fallback Pages**

Bir safyanın "fallback" halinde:

* Sayfanın props değeri boş olacaktır.
* [router](https://nextjs.org/docs/api-reference/next/router)'i kullanarak sayfanın fallback olup olmadığını anlayabilirsiniz. `router.isFallback` değeri `true` olacaktır.

`isFallback` kullanımıyla ilgili bir örnek:

```jsx
// pages/posts/[id].js
import { useRouter } from 'next/router'

function Post({ post }) {
    const router = useRouter()

    // Eğer sayfa henüz oluşturulmadıylsa, getStaticProps()
    // sayfayı oluşturana kadar bu bölüm görünecek.
    if (router.isFallback) {
        return <div>Loading...</div>
    }

    // Post'un render edildiği kodlar burada...
}

// Bu fonksiyon build sırasında çağırılıyor
export async function getStaticPaths() {
    return {
        // Build sırasında sadece `/posts/1` ve `/posts/2` oluşturulsun.
        paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
        // Statik olarak da başka bir sayfa sonradan eklenebilir.
        // Örneğin: `/posts/3`
        fallback: true,
    }
}

// Burası da build sırasında çalışacak. (bir de yeni sayfa oluşturulurken)
export async function getStaticProps({ params }) {
    // params değeri post'un id'sini içeriyor.
    // Mesela eğer yol /posts/1 ise id değeri 1 olacaktır.
    const res = await fetch(`https://.../posts/${params.id}`)
    const post = await res.json()

    // Post verilerini props yoluyla sayfa component'ine iletiyoruz.
    return {
        props: { post },
        // Eğer bir istek gelirse veya en az saniyede 1 kez
        // sayfa yeniden oluşturulup pre-render edilsin.
        revalidate: 1,
    }
}

export default Post
```

**fallback: true ne zaman kullanışlıdır?**

`fallback: true`, uygulamanızın verilere bağlı çok sayıda statik sayfası varsa \(mesela bir e-ticaret sitesi\) kullanışlıdır.

Tüm ürün sayfalarını pre-render etmek istiyorsunuz ama bu sayfaların gelecekte de \(sonsuza kadar\) yeniden pre-render edilmesi de gerekiyordur. Bunun yerine, statik olarak küçük bir alt sayfa kümesi oluşturulabilir ve geri kalanı için `fallback: true` kullanabilirsiniz. Bir kullanıcı henüz oluşturulmamış bir sayfa istediğinde, kullanıcıya loading ekranı gösterilecektir. Kısa bir süre sonra da `getStaticProps` çalışmayı bitirip sayfayı istenen verilere göre oluşturur. Böylece artık isteyen herkes bu sayfaya da erişebilir hale gelecektir.

Bu hem kullanıcıların hız deneyimini kesintiye uğratmamış olur hem de static generation'un faydalarını kullanmış oluruz.

`fallback: true`, zaten oluşturulmuş olan sayfaları güncellemez. Bununla ilgili [Incremental Static Regeneration]() konusunu inceleyin.

**fallback: 'blocking'**

Kısaca, `fallback: true` gibi çalışır. Fark şu ki; henüz oluşturulmamış bir yola istek geldiği zaman bir "fallback" sayfası göstermek yerine `getStaticProps`'un sayfayı oluşturmasını bekler. Yani SSR gibi çalışır. `getStaticProps` işini tamamladığı zaman kullanıcıya bu yeni sayfa gönderilir. Bu arada bu yol için de artık bir statik sayfa oluşturmuştur. Sonraki isteklerde yine `getStaticProps` çalışmaz, sayfalar önbellekten sunulur.

`fallback: 'blocking'`, oluşturulan sayfaları varsayılan olarak güncellemeyecektir. Oluşturulan sayfaları güncellemek için `fallback: 'blocking'` ile birlikte ISR de kullanın.

> `next export` kullanırsanız `fallback: 'blocking'` desteklemez.

#### Teknik detaylar

**getStaticProps ile birlikte kullanın**

Dinamik rota parametreleri olan bir sayfda `getStaticProps` kullandığınızda `getStaticPaths` de kullanmak zorundasınız.

`getStaticPaths`, `getServerSideProps` birlikte kullanılamaz.

**Sadece server-side olarak build sırasında çalışır**

`getStaticPaths` sadece server-side olarak build sırasında çalışır.

**Sadece sayfalarda kullanılır**

`getStaticPaths` sadece sayfadan export edilmelidir. Sayfa olmayan dosyalardan export edemezsiniz.

Ayrıca `export async function getStaticPaths() {}` kullanmalısınız. Yani özellikle sayfa bileşeninden bağımsız bir fonksiyon olarak export etmelisiniz. Sayfanın bir property'i olarak \(propTypes'ı ekler gibi\) oluşturmaya çalışırsanız çalışmaz.

### `getServerSideProps` \(Server-side Rendering\) <a id="basic-features_data-fetching_getserversideprops-server-side-rendering"></a>

> Versiyon Geçmişi
>
> * **v10.0.0:** `locale`, `locales`, `defaultLocale` ve `notFound` seçenekleri eklendi.
> * **v9.3.0:** `getServerSideProps` tanıtıldı.

Bir sayfadan `getStaticProps` adlı `async` bir fonksiyon `export` ederseniz, Next.js `getServerSideProps` tarafından döndürülen öğeleri kullanarak her istekte sayfayı pre-render eder.

```jsx
export async function getServerSideProps(context) {
    return {
        props: {}, // sayfa component'ine props olarak geçecek veriler
    }
}
```

Buradaki `context` parametresi şu anahtar değerleri içeren bir objedir:

* `params` dinamik routing için route parametrelerini içerir. Mesela eğer sayfa adı `[id].js` ise `params` değeri `{ id: ... }` şeklinde görünecektir. Daha fazlasını öğrenmek için [Dynamic Routing dokümantasyonu](https://nextjs.org/docs/routing/dynamic-routes)na bakabilirsiniz. Bunu daha sonra anlatacağımız `getStaticPaths` ile birlikte kullanmalısınız.
* `req` [HTTP IncomingMessage objesi](https://nodejs.org/api/http.html#http_class_http_incomingmessage).
* `res` [HTTP response objesi](https://nodejs.org/api/http.html#http_class_http_serverresponse).
* `query` query string'i temsil eden bir obje.
* `preview` değeri, eğer sayfa preview modundaysa `true` olur. Aksi halde `undefined`'dir.
* `previewData` değeri, `setPreviewData` tarafından ayarlanan önizleme verilerini içerir.
* `resolveUrl` istemci geçişleri için `_next/data` önekini çıkaran ve orijinal query değerlerini içeren istek URL'sinin normalleştirilmiş hali.
* `locale`, aktif local ayarları içerir \(etkinleştirilmişse\)
* `locales`, desteklenen tüm local ayarları içerir \(etkinleştirilmişse\)
* `defaultLocale`, varsayılan local ayarları gösterir \(etkinleştirilmişse\)

`getServerSideProps` şu değerleri içeren bir obje return etmelidir:

* `props` - Sayfa component'i tarafından alınacak değerlere sahip, **isteğe bağlı** bir nesnedir. Bu nesne [serializable](https://en.wikipedia.org/wiki/Serialization) olmalıdır.
* `revalidate` - Sayfanın yeniden oluşturulabilmesi için saniye cinsinden **isteğe bağlı** bir miktar \(varsayılan olarak `false` veya yeniden doğrulama yok\). Daha fazlası için: [Incremental Static Regeneration]()
*  `notFound` - Sayfanın 404 durumu ve sayfası döndürmesine izin vermek için **isteğe bağlı** bir `boolean` değerdir. Aşağıda nasıl çalıştığına dair bir örnek verilmiştir:

  ```text
  export async function getServerSideProps(context) {
      const res = await fetch(`https://.../data`)
      const data = await res.json()
    
      if (!data) { return { notFound: true, } }
    
      return { props: { data }, } // sayfa component'ine prop olarak geçilecek veri
  }
  ```

  > **Not:** Bu örnekte `notFound`'un `fallback: false`'ye ihtiyacı yoktur çünkü sadece `getStaticPaths`'ten dönen yollar pre-render edilecektir çünkü diğerlerinin pre-render edilmesinin yolu `if` bloğunda engellenmiştir.

  > **Not:** `notFound: true` ile, sayfa başarıyla oluşturulmuş olsa bile 404 döndürecektir. Bu, kullanıcı tarafından oluşturulan oluşturulan içeriğin yazar tarafından kaldırılması gibi kullanımları desteklemek içindir.

*  `redirect` - Dahili veya harici kaynaklara yönlendirmeye izin vermek için **isteğe bağlı** `{ destination: string, permanent: boolean }` biçiminde bir değerdir. Bazı ender durumlarda, eski HTTP istemcilerinin doğru şekilde yeniden yönlendirme yapması için özel bir durum kodu atamanız gerekebilir. Bu durumlarda `permanent` property'i yerine `statusCode` property'i özelliğini kullanabilirsiniz \(ama ikisini aynı anda kullanamazsınız\). Aşağıda nasıl çalıştığına dair bir örnek verilmiştir:

  ```text
  export async function getServerSideProps(context) {
      const res = await fetch(`https://...`)
      const data = await res.json()
    
      if (!data) {
          return {
              redirect: { destination: '/', permanent: false, },
          }
      }
    
      return {
          props: { data }, // sayfa component'ine props olarak gidecek data
      }
  }
  ```

  > **Not:** Şu anda build sırasında redirect'e izin verilmemektedir ve yönlendirmeler build anında biliniyorsa `next.config.js`'ye eklenmelidir.

> **Not:** `getStaticProps` içinde üst seviye `module`'leri `import` edip kullanabilirsiniz.
>
> Yani, sunucu taraflı kodu doğrudan `getStaticProps`'ta kullanabilirsiniz. Mesela dosya sisteminizden veya veritabanından okuma gibi işlemleri burada yapabilirsiniz.
>
> `getStaticProps`'ta kullanılan `import`'lar client-side için bundle edilmeyecektir \(paketlenmeyecektir\).

> **Not:** `getStaticProps` içinde bir API yoluna erişmek için `fetch()` kullanmamalısınız. Bunun yerine API rotanızda kullandığınız mantığı doğrudan `import` edin. Bu yaklaşım için kodunuzu biraz yeniden düzenlemeniz gerekebilir.
>
> Harici bir API'den veri almak sorun değil!

#### Basit Bir Örnek

Burada bir CMS'den blog post'ları listesini alan bir `getServersideProps` örneği yaptık. Bu örnek aynı zamanda [Pages dokümantasyonu](pages.md)nda da bulunmaktadır.

```jsx
function Page({ data }) {
    // Render data...
}
    
// Her istekte çalışacak
export async function getServerSideProps() {
    // Harici bir API endpoint'inden veri çekiyoruz
    const res = await fetch(`https://.../data`)
    const data = await res.json()
    
    // Sayfaya props yoluyla veriyi aktarıyoruz.
    return { props: { data } }
}
    
export default Page
```

#### Ne zaman `getServerSideProps` Kullanmalıyız?

`getServerSideProps`'u yalnızca verileri istek yapıldığı zaman alınması gereken bir sayfayı pre-render etmeniz gerekiyorsa kullanmalısınız. İlk byte'ye kadar geçen süre \(TTFB\), sunucunun her istekte sonucu hesaplaması gerektiğinden ve sonuç extra yapılandırma olmadan bir CDN tarafından önbelleğe alınamayacağı için `getStaticProps`'tan daha yavaş olacaktır.

Eğer verileri önceden oluşturmanız gerekmiyorsa, verileri istemci tarafında getirmeyi düşünmelisiniz. Daha fazla bilgi için [buraya tıklayın]().

#### Teknik Detaylar

**Sadece server-side çalışır**

`getServerSideProps` sadece sunucu tarafında çalışır. Asla tarayıcıda çalışmaz. Eğer bir sayfa `getServerSideProps` kullanıyorsa:

* Bu sayfaya bir istek yapıldığında `getServerSideProps` çalışırve sayfayı pre-render ederek props'larıyla döndürür.
* `next/link` [\(dokumantasyon\)](https://nextjs.org/docs/api-reference/next/link) veya `next/router` [\(dokumantasyon\)](https://nextjs.org/docs/api-reference/next/router) aracılığıyla istemci taraflı sayfa geçişleri yaparken bu sayfayı talep ettiğinizde Next.js, `getServerSideProps` çalıştıran sunucuya bir API isteği gönderir. `getServerSideProps` çalışıp sonucu içeren bir JSON döndürür. Bu JSON sayfayı oluşturmak için kullanılır. Tüm bu işler Next.js tarafından otomatik olarak halledilecektir. Yani `getServerSideProps` tanımlı olduğu sürece ekstra bi'şey yapmanıza gerek kalmaz.

Hangi kodun istemci tarafında gönderildiğini test etmek için [bu aracı](https://next-code-elimination.vercel.app/) kullanabilirsiniz.

**Sadece sayfalarda kullanılır**

`getServerSideProps` sadece sayfadan export edilir. Sayfa olmayan bir dosyadan export edemezsiniz.

Sayfa component'inin bir property'i gibi oluşturamazsınız. Sayfa component'inden bağımsız şekilde sayfa dosyasından export edilmelidir \(`export async function getServerSideProps() {}`\).

### Fetching data on the client-side \(İstemci tarafında data almak\)

Sayfanız sık sık güncellenen veriler içeriyorsa ve verileri önceden oluşturmanız gerekmiyorsa, verileri istemci tarafında getirebilirsiniz. Bunun örneği, kullanıcıya özel verilerdir. Şu şekilde çalışır:

* İlk olarak, sayfayı veri olmadan hemen gösterin. Sayfanın bölümleri Static Generation kullanarak pre-render edilebilir. Eksik veriler için de loading state'leri kullanırsınız.
* Ardından, verileri istemci tarafında alın ve hazır olduğunda görüntüleyin.

Bu yaklaşım, örneğin, kullanıcı kontrol paneli sayfaları için iyi çalışır. Çünkü mesela dashboard kullanıcıya özel bir sayfa olduğundan SEO'yla ilgisi yoktur ve sayfanın önceden oluşturulmasına gerek yoktur. Veriler sık sık güncellenir. Bu da her istekte verinin alınmasını gerektirir.

#### SWR <a id="basic-features_data-fetching_swr"></a>

Next.js'nin arkasındaki ekip, veri almak için **SWR** adında bir React Hook oluşturdu. İstemci tarafında veri alıyorsanız bunu kesinlikle öneririz. Önbelleğe alma, yeniden doğrulama, focus izleme, aralıklarla yeniden getirme ve daha fazlasını bu hook ile yönetebilirsiniz. Kullanımı şöyledir:

```jsx
import useSWR from 'swr'

function Profile() {
    const { data, error } = useSWR('/api/user', fetch)

    if (error) return <div>failed to load</div>
    if (!data) return <div>loading...</div>
    return <div>hello {data.name}!</div>
}
```

[SWR dokümantasyonunu inceleyip daha fazlasını öğrenebilirsiniz.](https://swr.vercel.app/)

### Daha Fazlasını Öğrenin

Buradan sonra aşağıdakilerden biriyle devam etmenizi öneririz:

* [**Preview Mode:** Önizleme modu hakkında daha fazlasını öğrenin.](https://nextjs.org/docs/advanced-features/preview-mode)
* [**Routing:** Next.js'te routing ile ilgili daha fazlasını öğrenin.](https://nextjs.org/docs/routing/introduction)
* [**TypeScript:** Sayfalarınıza TypeScript ekleyin.](https://nextjs.org/docs/basic-features/typescript#pages)

