# Fast Refresh

Fast Refresh \(Hızlı Yenileme\), React component'lerinizde yapılan düzenlemeleri anında görebileceğiniz bir Next.js özelliğidir. Fast Refresh, 9.4 ve üstü sürümlerdeki tüm Next.js uygulamalarında varsayılan olarak aktiftir. Next.js, Fast Refresh aktifken çoğu değişikliklerde component'teki state'yi koruyarak 1 saniye içinde değişikliği yansıtır.

## Nasıl Çalışıyor

* Yalnızca React component'lerini `export` eden bir dosyayı düzenlerseniz, Fast Refresh yalnızca o dosyanın kodunu günceller ve component'inizi yeniden oluşturur. Bu dosyadaki stiller, işleme mantığı, event işleyiciler veya efektler dahil her şeyi düzenleyebilirsiniz.
* React component'leri olmayan `export`'lara sahip bir dosyayı düzenlerseniz Fast Refresh hem o dosyayı hem de onu `import` etmiş diğer dosyaları yeniden çalıştırır. Dolayısıyla hem `Button.js` hem de `Modal.js` component'leri `theme.js`'yi import etmişse, `theme.js`'de yapılan değişiklik sonrası iki component de güncellenir.
* Son olarak, React ağacının dışındaki olaylar tarafından `import` edilen bir dosyayı düzenlerseniz, Fast Refresh tam bir yeniden yükleme yapar. Bir React component'i oluşturan ancak aynı zamanda React olmayan bir component tarafından `import` edilen bir değeri `export` eden bir dosyanız olabilir. Mesela component'iniz bir sabit değeri `export` edebilir ve React olmayan bir yardımcı program onu `import` edebilir. Bu durumda, sabiti ayrı bir dosyaya geçirmeyi ve her iki dosyaya da aktarmayı düşünün. Böylece bu dosyadaki değişiklik, onu import eden component'lerde Fast Refresh'i aktif eder. Diğer durumlar da genelde benzer şekilde çözülebilir.

## Hata Esnekliği

### Syntax Hataları

Geliştirme sırasında bir syntax hatası yaparsanız, düzeltebilir ve dosyayı tekrar kaydedebilirsiniz. Hata otomatik olarak kaybolur ve uygulamayı yeniden yüklemeniz gerekmez. Component'in state'si de korunur.

### Runtime Hataları

Component'inizin içinde runtime hatasına yol açan bir hata yaparsanız, bağlamsal bir overlay \(kaplama\) ile kaşılaşacaksınız. Hatanın düzeltilmesi ,uygulamayı yeniden yüklemeden overlay'ı otomatik olarak kapatacaktır.

Oluşturma \(rendering\) sırasında hata oluşmadıysa component state'si korunacaktır. Hata, oluşturma sırasında meydana gelirse React güncellenmiş kodu kullanarak uygulamanızı yeniden bağlar.

Uygulamanızda hata sınırlarınız \(error boundaries\) varsa \(bu production'daki hassas hatalar için iyi bir fikirdir\) bir rendering hatasından sonra bir sonraki düzenlemede yeniden render etmeyi deneyecektir. Yani bir error boundry'e sahipseniz, uygulamayı komple reset'lemenin önüne geçmiş olursunuz. Ancak error boundry'lerin çok ayrıntılı olmaması gerektiğini unutmayın. React tarafından production'da kullanılırlar ve her zaman kasıtlı olarak tasarlanmalıdırlar.

## Limitler

Fast Refresh, düzenlediğiniz component'te yerel React state'sini korumaya çalışır, eğer bunu yapmak güvenliyse... Bir dosyada yapılan her düzenlemede yerel state reset'leniyorsa bunun nedenlerinden birkaçı şunlardır:

* Class component'leri için yerel state korunmaz \(yalnızca fonksiyonel component'ler ve hook'lar state'yi korur\).
* Düzenlediğiniz dosyanın bir React component'ine ek olarak başka `export`'ları olabilir.
* Bazen bir dosya, `HOC (WrappedComponent)` gibi daha yüksek dereceli component'lerin çağırılmasının sonucunu `export` eder. Döndürülen component bir class ise state sıfırlanır.
* `export default () => <div />;` gibi anonim arrow function'lar, Fast Refresh'in yerel component state'sini korumamasına neden olur. Büyük kod tabanları için [`name-default-component` codemode](https://nextjs.org/docs/advanced-features/codemods#name-default-component)'mizi kullanabilirsiniz:
  * `name-defauld-component`, anonim component'leri Fast Refresh ile çalıştıklarından emin olmak için adlandırılmış \(named\) component'lere dönüştürür. 
  * Örneğin şu kod:

    ```text
    // my-component.js
    export default function () {
      return <div>Hello World</div>
    }
    ```

  * Şu koda dönüştürülür:

    ```text
    // my-component.js
    export default function MyComponent() {
      return <div>Hello World</div>
    }
    ```

  * Component, dosya adına göre bir camel case ada sahip olacak ve ayrıca arrow function'la da çalışacak.
  * **Kullanımı**:
    * Projenize gidin:

      ```text
      cd path-to-your-project/
      ```

    * codemod'u çalıştırın:

      ```text
      npx @next/codemod name-default-component
      ```

Kod tabanınızın çoğu, fonksiyonel component'lere ve hook'lara doğru hareket ettikçe daha fazla durumda state'nin korunmasını bekleyebilirsiniz. \(Class component'ler yerine fonksiyonel component'ler kullanmanızı öneriyoruz.\)

## İpuçları

* Fast Refresh, varsayılan olarak fonksiyonel component'lerinde \(ve hook'larda\) React yerel state'sini korur.
* Bazen state'yi reset'lemeyi ve bir component'in yeniden oluşmasını zorlamak isteyebilirsiniz. Örneğin yalnızca component oluşturulduğunda çalışan bir animasyona ince ayar vermeye çalışıyorsanız bu kullanışlı olabilir. Bunu yapmak için, düzenlemekte olduğunuz dosyanın herhangi bir yerine `// @refresh reset` yazabilirsiniz. Bu yönerge dosya için yereldir ve Fast Refresh'e her düzenlemede o dosyada tanımlanan component'leri yeniden bağlama talimatı verir.
* Geliştirme sırasında düzenlediğiniz component'lere `console.log` veya `debugger;` koyabilirsiniz.

## Fast Refresh ve Hook'lar

Mümkün olduğunda Fast Refresh, düzenlemeler arasında component'inizin state'sini korumaya çalışır. Özellikle `useState` ve `useRef`'i veya Hook çağrılarının sırasını değiştirmediğiniz sürece önceki değerleri korunur.

useEffect, useMemo ve useCallback gibi bağımlılıkları olan kancalar Fast Refresh sırasında her zaman güncellenir. Fast Refresh gerçekleşirken bunların bağımlılık listesi yok sayılır.

Örneğin, `useMemo(() => x * 2, [x])` kodunu `useMemo(() => x * 10, [x])` olarak güncellerseniz, `x` \(bağımlılık\) değişmemiş olsa bile yeniden çalıştırılır. React bunu yapmasaydı, yaptığınız değişiklikler ekrana yansımazdı!

Bazen bu beklenmedik sonuçlara yol açabilir. Örneğin boş bir bağımlılık array'ına sahip bir `useEffect` bile Fast Refresh sırasında bir kez yeniden çalışmaya devam eder.

Ancak, useEffect'in ara sıra yeniden çalıştırılmasına karşı dayanıklı kod yazmak, Fast Refresh olmasa bile iyi bir uygulamadır. Daha sonra ona yeni bağımlılıklar eklemenizi kolaylaştıracak ve etkinleştirmenizi çokça tavsiye ettiğimiz [React Strict Mode](https://nextjs.org/docs/api-reference/next.config.js/react-strict-mode) tarafından zorunlu tutuluyor.

* React Strict Mode
* Uygulamanızı React'ın geleceğine daha iyi hazırlamak için Next.js uygulamanızda Strict Mode \(Katı Mod\) etkinleştirmenizi öneriyoruz.
* Next.js runtime artık Strict Mode uyumludur. Strict Mode'yi aktif etmek için `next.config.js` dosyanızda aşağıdaki seçeneği yapılandırın:

  ```text
  // next.config.js
  module.exports = {
    reactStrictMode: true,
  }
  ```

* Siz veya ekibiniz uygulamanın tamamında Strict Mode'yi kullanmaya hazır değilseniz sorun değil! `<React.StrictMode>` kullanarak sayfa sayfa aşamalı olarak geçiş yapabilirsiniz.
* React'ın Strict Mode'si, bir uygulamadaki olası sorunları vurgulamaya yönelik yalnızca bir development \(geliştirme\) modu özelliğidir. Güvenli olmayan yaşam döngülerini, eski API kullanımını ve bir dizi başka özelliği belirlemeye yardımcı olur.

