# Agent evaluation

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter7/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

## 1. evaluation’ın amacı ve temel yaklaşım

bir agent geliştirmek, context, knowledge, tools, memory, skills ve os/as tasarlamakla bitmez. sistemin gerçekten iyileşip iyileşmediğini anlayabilmek için tekrarlanabilir, ölçülebilir ve teşhis edilebilir bir evaluation sistemi gerekir.

evaluation üç temel soruya yanıt verir:

* measurement: agent ne kadar başarılı?
* diagnosis: başarısızlık nerede ve neden oluşuyor?
* improvement: sonuçlar model ve harness tasarımına nasıl yansıtılmalı?

## model or harness?

değerlendirme yapılırken yalnızca modelin kendisi değil, model + harness bileşimi ölçülmelidir. aynı model, farklı prompt'lar, tool tasarımları ve feedback loop'larıyla çok farklı sonuçlar üretebilir.

![Görsel](../gorseller/Pasted%20image%2020261017221716.png)

evaluation ayrıca model güncellemelerinde regression tespit eder. genel benchmark puanı yükselen bir model, gerçek ürün görevlerinde daha kötü sonuç verebilir. bu nedenle model değişikliği, kamuya açık benchmark'larla değil, öncelikle kendi iş kümesinde test edilmelidir.

## 2. evaluation sisteminin dört halkası

![Görsel](../gorseller/Pasted%20image%2020261017221837.png)

Bu halkaların çalışabilmesi için gözlemlenebilirlik, iç evaluation altyapısı, ablation/AB testleri, feature flag'ler ve simülasyon ortamları gerekir.

## 3. metrikler: pass@k ve pass^k

bir agent'ın bir görevi bazen başarabilmesi ile her seferinde güvenilir biçimde başarabilmesi aynı şey değildir.

![Görsel](../gorseller/Pasted%20image%2020261017221954.png)

Önemli ayrım: Pass@k, denemeler arasından en iyisini seçme imkânını temsil eder. Pass^k, istikrarlı çalışma beklentisini ölçer. Production'daki ardışık görevlerin başarı oranı, bağımsız denemelerin başarı olasılığıyla otomatik olarak aynı kabul edilmemelidir. Yan etkili işlemler de kontrolsüz biçimde tekrarlanmamalıdır.

## 4. evaluation ortamının mimarisi

Yinelenebilir bir evaluation ortamı, aynı başlangıç koşullarında Agent'ları karşılaştırılabilir sonuçlar üretecek şekilde çalıştırır. Beş temel bileşeni vardır.

![Görsel](../gorseller/Pasted%20image%2020261017222113.png)

Bir bileşenin eksikliği, ölçümün tekrarlanabilirliğini veya anlamını bozar. Özellikle tool’ların aşırı yüksek seviyeli olması, planlama ve akıl yürütme işini Agent'tan alıp aracın içine gizleyebilir.

### İki tür evaluation ortamı

|Özellik|Tool-calling|Human-computer interaction|
|---|---|---|
|Etkileşim|Agent ve tools|Agent, kullanıcı simülatörü ve tools|
|Durum|Dosya, DB, sandbox vb.|Kullanıcı durumu ve iş sistemleri|
|Doğrulama|Test/assertion|Son durum + iletişim/eylem koşulları|
|Örnek|Verifiers|τ²-bench|

Verifiers ortamları görevin ihtiyaçlarına göre ayrılır:

- SingleTurnEnv: Tek turlu soru-cevap ve matematik.
    
- ToolEnv: Birden fazla tool call ile arama ve bilgi birleştirme.
    
- StatefulToolEnv: Veritabanı gibi kalıcı durumu değiştirme.
    
- SandboxEnv: Yalıtılmış ortamda kod yürütme ve çıktı doğrulama.


Evaluation boyunca observation, action ve reward içeren tam `trajectory` saklanmalıdır. Hatalar yalnızca başarısızlık bayrağı olarak değil, Agent'ın strateji değiştirebilmesini sağlayacak açıklayıcı hata mesajlarıyla raporlanmalıdır.

## 5. τ²-bench: Gerçek görev üzerinden inceleme

τ²-bench'in telecom senaryosu, Agent'ın yalnızca doğru cevabı bilmesini değil, kullanıcıyla etkileşerek sorunu çözmesini de test eder.

Örnek: Fransa'da bulunan bir kullanıcının telefonu internete bağlanamıyor. Başlangıçta uçak modu açık ve data roaming kapalı. Operatör veritabanında ise roaming etkin görünüyor.

### Görev nasıl çalışır?

1. Known information: Kullanıcı adı, telefon numarası ve bulunduğu ülke verilir. Arızanın nedeni doğrudan söylenmez.
    
2. User instructions: Kullanıcı ilk başarısız denemeden sonra hafif memnuniyetsizlik gösterir. Sorunun çözüldüğünü yalnızca hız testi `excellent` döndürdüğünde kabul eder.
    
3. Initial state: Kullanıcı tarafındaki cihaz durumu ile Agent tarafındaki operatör veritabanı ayrı ayrı başlatılır.
    
4. Actions ve assertions: Kullanıcı kendi cihaz ayarlarını değiştirebilir; Agent operatör işlemlerini yapabilir. Son durumda mobil veri çalışmalı ve hız en az 200 Mbps olmalıdır.

Buradaki önemli tasarım ilkeleri:

- Progressive Information Disclosure: Kullanıcı yalnızca bildiği bilgiyi verir; Agent eksik bilgiyi sorarak keşfeder.
    
- Grounding: Cihaz durumuyla ilgili cevaplar gerçek tool sonuçlarına dayanır. Simüle kullanıcı sonuç uyduramaz.
    
- Dual-Control: Hem kullanıcı hem Agent ortamı değiştirebilir.
    
- Layered Verification: Son durum, eylemler, iletilen bilgi ve doğal dil koşulları ayrı ayrı denetlenebilir.


`reward_basis` hangi denetimlerin nihai ödüle katılacağını belirler. Örnekte yalnızca `ENV_ASSERTION` ödüle dahil edildiğinden, Agent süreçte bir politika ihlali yapsa bile son durum doğruysa tam puan alabilir. Dolayısıyla reward = 1.0, her davranışın doğru olduğu anlamına gelmez.

Başarısız örnekte Agent yanlış telefon hattını seçer, mevcut kanıtı doğru değerlendiremez ve gereken data yüklemesini gerçekleştiremez. Sorunun önemli kısmı, kanıtın context'e girmiş olmasına rağmen Agent'ın kararını düzeltmemesidir. Bu tür hatalar yalnızca sonuca bakarak değil, trajectory incelenerek anlaşılır.

## 6. Benchmark tasarımı ve veri kümesinin kalitesi

Benchmark yalnızca görevlerden oluşan bir liste değildir. Görevlerin nasıl üretildiği, hangi ortamda çalıştığı ve nasıl doğrulandığı, benchmark'ın neyi ölçtüğünü belirler.

| Benchmark          | Ölçtüğü temel yetenek                             | Ortam / doğrulama                                        |
| ------------------ | ------------------------------------------------- | -------------------------------------------------------- |
| τ²-bench           | Müşteri hizmetleri, diyalog ve tool kullanımı     | Kullanıcı simülatörü, iş veritabanı, katmanlı assertions |
| SWE-bench Verified | Coding ve gerçek issue çözümü                     | GitHub depoları, test paketi                             |
| AndroidWorld       | Android GUI kullanımı                             | Android emülatörü, son UI durumu                         |
| OSWorld            | Masaüstü GUI kullanımı                            | Sanal makine ve sistem durumu denetimleri                |
| Terminal-Bench     | Terminal ve coding görevleri                      | Docker, dosya sistemi ve gerçek yürütme                  |
| GAIA               | Genel amaçlı bilgi toplama ve çok adımlı görevler | Açık internet, özel ekler, tam dizgi eşleşmesi           |
![Görsel](../gorseller/Pasted%20image%2020261019220335.png)

Örnekler, benchmark kalitesinin yalnızca görev sayısıyla ölçülemeyeceğini gösterir. SWE-bench Verified'da özgün görevlerin önemli bir kısmı insan incelemesi sonrasında elenmiştir. OSWorld'de de yayımlandıktan sonra ortam, görev açıklaması, doğrulama ve başlangıç durumu kaynaklı sorunlar ortaya çıkmıştır.

### Evaluation görevleri nereden gelmeli?

![Görsel](../gorseller/Pasted%20image%2020261019220447.png)

Başlangıçta açık benchmark'lar ve elle yazılmış küçük bir iş kümesi yeterli olabilir. Sistem production'da çalıştıkça gerçek başarısızlık örnekleri, evaluation kümesinin en değerli kaynaklarından biri hâline gelir.

![Görsel](../gorseller/Pasted%20image%2020261019220558.png)

### LLM-as-a-Judge nasıl çalışır?

Hakem modele bir `Candidate Answer`, isteğe bağlı `Reference Solution` ve değerlendirme ölçütlerini tanımlayan bir `Rubric` verilir.

Örnek bir Rubric şu boyutları içerebilir:

- Factual Correctness: Bilgiler doğru mu?
    
- Completeness: Gerekli noktalar eksiksiz mi?
    
- Logical Coherence: Yanıt mantıklı mı?
    
- Hallucination Detection: Desteklenmeyen veya yanlış bilgi var mı?


Sonuçlar ağırlıklı ortalamayla birleştirilebilir. Ancak hallucination gibi kritik bir ihlal için `veto` uygulanarak toplam puan sıfırlanabilir. Birden fazla hakem kullanıldığında medyan alınabilir; hakemler arasındaki büyük anlaşmazlıklar insan incelemesine yönlendirilebilir.

Başlıca riskler:

- Uzun yanıtları gereğinden fazla ödüllendiren length bias.
    
- Hakem modelin kendi önyargıları.
    
- Aynı girdide değişken sonuçlar.
    
- Candidate ve reference'ın aynı kaynaktan gelmesinden doğan ortak yanlılık.
    
- Değerlendirme maliyeti.


Bu nedenle Rubric, gereksiz uzunluğu cezalandırmalı; mümkünse farklı kaynaklardan hakemler kullanılmalı ve düzenli insan değerlendirmesiyle kalibre edilmelidir.

### Çok modlu evaluation

LLM-as-a-Judge yalnızca metin için değildir.

|Alan|Değerlendirme boyutları|
|---|---|
|TTS|Doğruluk, doğallık, prozodi, duygu ve ses tınısı tutarlılığı|
|ASR|Sözcük doğruluğunun yanı sıra hatanın anlamsal ve operasyonel etkisi|
|UI|Metin taşması, renk kontrastı, düğme yerleşimi ve kullanılabilirlik|
|Video editing|Kesme noktaları, efektler ve anahtar kareler|

Örneğin TTS karşılaştırmasında bir sistemin yalnızca dört kaydıyla diğerinden daha iyi olduğu sonucuna varılamaz. Üstelik ses benzerliği için yalnızca bir sağlayıcının referans kaydını kullanmak değerlendirmeyi o sağlayıcı lehine yanlı hâle getirebilir.

Referans yanıt, görsel veya ses seçimi de evaluation tasarımının bir parçasıdır.

## Failure attribution: Başarısızlığı iyileştirmeye dönüştürmek

Bir görevin başarısız olduğunu bilmek, onu düzeltmek için yeterli değildir. Başarısız `trajectory` içinde ilk hatalı karar veya eylem belirlenmelidir.

![Görsel](../gorseller/Pasted%20image%2020261019220724.png)

### Coding Agent için hata sınıfları

|Hata sınıfı|Tipik belirti|
|---|---|
|Gereksinim anlama|İstenen koşulun atlanması veya kapsamın yanlış yorumlanması|
|Süreç/kural eksikliği|Plan yapmadan kod değiştirmek, testleri çalıştırmadan commit atmak|
|Tool call hatası|Yanlış argüman, bozuk schema, başarısız dosya düzenlemesini tekrarlamak|
|Verification ortamını hacklemek|Assertion silmek, test skip etmek veya test edilmemiş işe başarılı demek|
|Eksik değişiklik|Bir API değişikliğinin bazı çağrı noktalarını güncellemeden bırakılması|
|Yanlış kullanıcı bildirimi|İş tamamlanmadığı hâlde tamamlandı demek veya yanlış bilgi vermek|
|İşlevsel regression|Yeni değişikliğin önceden çalışan davranışı bozması|
|Anormal sonlanma|Timeout, yarıda kesilme veya kapanış eylemi olmadan durma|
|Erken vazgeçme|Makul alternatifler denenmeden görevin imkânsız ilan edilmesi|

### Hata kaydı nasıl tutulmalı?

Attribution çıktısı yalnızca “Agent yanlış yaptı” gibi bir açıklama olmamalıdır. Yapılandırılmış bir kayıt en azından şunları içermelidir:

- Hata kategorisi ve alt kategorisi.
    
- İlk hatanın `trajectory` içindeki adım numarası.
    
- İlgili tool call, observation ve somut kanıt.
    
- Kök neden ile sonraki sonuçların ayrımı.
    
- Kurtarılabilirlik ve güven düzeyi.
    
- Görev hedefi, environment state, model ve tool sürümleri, tam trajectory.


Örneğin `edit_file`, `old_string` uyuşmazlığı döndürdükten sonra Agent aynı işlemi üç kez daha başarısız biçimde denerse, üç deneme üç ayrı kök neden değildir. Asıl hata, ilk düzenleme başarısızlığının doğru ele alınamamasıdır.

En verimli yöntem, önce deterministik kontrollerle kolayca saptanabilen hataları elemek, sonra LLM ile daha karmaşık hataların ilk oluştuğu noktayı bulmak ve son olarak insan incelemesiyle ürün kaynaklı sorunları değerlendirmektir.

Bölümün ana fikri: Kapalı iyileştirme döngüsü

![Görsel](../gorseller/Pasted%20image%2020261019220840.png)

Sistemin olgunluğu yalnızca daha fazla görevi geçmesiyle ölçülmez. Başarısızlıkların nedenini açıklayabilmesi, düzeltmelerin etkisini ölçebilmesi ve eski yetenekleri bozmadan ilerleyebilmesi de gerekir.

**Bölüm 7'nin en önemli mühendislik ilkesi: Evaluation, geliştirme sürecinin sonundaki bir test aşaması değil; model ve Harness'in hangi yönde geliştirileceğini belirleyen temel feedback mekanizmasıdır.**

----
---
---

# Agent Evaluation: Hata Atfından Üretim Düzeyinde Evaluation’a

Bu kısmın ana fikri, bir Agent’ın yalnızca başarılı olup olmadığını ölçmekle yetinmemektir. Güvenilir bir değerlendirme sistemi; ilk hatayı bulmalı, kök nedeni doğru katmana atfetmeli, hatayı tekrarlanabilir bir regression task’a dönüştürmeli ve iyileştirmenin gerçekten işe yaradığını istatistiksel olarak doğrulamalıdır.

Bölümün ilerleyişi şu zincirle özetlenebilir:

![Görsel](../gorseller/Pasted%20image%2020261019221054.png)

## 7.5.2 Hata Atfı: Başarısızlığın Gerçek Kaynağını Bulmak

Bir Agent’ın başarısız olması, modelin mutlaka yanlış akıl yürüttüğü anlamına gelmez. Sorun modelde, Harness’te, gözlem kanalında, tool protokolünde veya doğrulayıcıda olabilir. Yanlış katmana atfedilen bir hata, yanlış çözümün uygulanmasına neden olur.

### 7.5.2.1 “Doğru yaptı, yanlış bildirdi” sorunu

Bu hata sınıfında Agent, ortamda işlemleri doğru gerçekleştirse bile sonuç hakkında yanlış bilgi verir. Tersi de mümkündür: Agent işlemleri yanlış yapar ama tamamlandığını söyler.

τ²-bench örneğinde, bilgilendirme gereksinimi olan 704 koşunun 240'ı başarısızdır. Bu başarısızlıkların 162'si bilgilendirme kontrolünden kalmış, 80'inde ise ortam durumu doğru olmasına rağmen Agent yanlış bildirim yapmıştır. Bu son grup, toplam başarısızlıkların yaklaşık üçte biridir.

Buradaki kritik ayrım:

- Environment state: Gerçekte ne oldu?
    
- Agent report: Agent ne olduğunu söyledi?
    
- Verifier: Gerçek durumla bildirimin uyuşup uyuşmadığını nasıl denetledi?


Başarılı bir işlem, doğru raporlamayı garanti etmez. Bu nedenle yalnızca son ortam durumunu denetlemek yeterli değildir; Agent’ın iddiaları da doğrulanmalıdır.

#### Kök nedenin yanlış sınıflandırılması

Örnekte Agent, `expenses.jpg` içindeki harcamaları muhasebe uygulamasına girmeye çalışır. Görseldeki içeriği göremediğini fark etmesine rağmen işlemi durdurmaz; uydurduğu dört harcamayı sisteme girer ve sonunda görevin tamamlandığını bildirir.

İlk kritik hata, Agent’ın görsel içeriğini göremediğini anladığı andır. Ancak bu hata bir tool call sırasında oluşmaz; bir `assistant message` içinde ortaya çıkar.

Dolayısıyla sorun yalnızca OCR yeteneğinin yetersizliği değildir. Agent’ın gözlem uzayında görüntü pikselleri yoksa, modelden görseli okumasını beklemek yanlış teşhistir. Asıl eksiklik, gerekli bilgiyi elde edecek bir gözlem kanalının ve bilgiye erişilemediğinde durmayı ya da açıklama istemeyi sağlayacak meşru bir eylemin bulunmamasıdır.

Mühendislik sonucu: Hata kaydına yalnızca “model görseli okuyamadı” yazmak yerine, gözlem kanalının eksikliğini ve bilgi eksikliği karşısındaki yanlış karar politikasını ayrı ayrı kaydetmek gerekir.

### Sessiz başarısızlıkları teşhis etme

Bir tool call hata döndürmeden de Agent başarısız olabilir. Bu nedenle tool hata kayıtlarını aramak, hata atfı için yeterli değildir.

İki teknik öne çıkar:

1. Fact-anchor comparison: Agent’ın iddialarını tool dönüşleri ve doğrulanmış olgularla karşılaştırarak ilk sapmayı bulmak.
    
2. Trajectory-prefix binary search: Yörüngeyi farklı adımlarda keserek hatanın hangi sınırdan sonra ortaya çıktığını belirlemek.


Bunların amacı aynı değildir: ilki iddia ile kanıt arasındaki uyuşmazlığı bulur; ikincisi hatanın ortaya çıktığı yörünge aralığını daraltır.

### Belge biçimi hataları

“Tırnak biçimi yanlış” gibi bir talep, her tırnağı aynı karaktere dönüştürmek anlamına gelmez. Aynı karakter düzyazıda, alıntılanan kaynakta, inline code'da, code block'ta ve JSON içinde farklı roller üstlenebilir.

Bu yüzden belge, örneğin şu kapsam türlerine ayrılmalıdır:

- `ZH_PROSE`
    
- `EN_PROSE`
    
- `QUOTED_SOURCE`
    
- `INLINE_CODE`
    
- `CODE_BLOCK`
    
- `CODE_COMMENT`
    
- `JSON_OR_SCHEMA`


Her kapsamın izin verilen dönüşümleri ve korunması gereken karakterleri farklı olabilir. Modelden istenen yalnızca hedef karakteri üretmesi değil, hedef dışındaki metni ve sözdizimini bozmadan en küçük geçerli değişikliği yapmasıdır.

Belirsiz bir kapsamda özgün metni korumak ve açıklama istemek, tahmine dayalı bir düzenleme yapmaktan daha doğru bir eylem olabilir.

### `old_string` mismatch ve birebir kopyalama

Bir `old_string` hatası da doğrudan modele atfedilmemelidir. Dize, sistem boyunca farklı katmanlardan geçer:

`original file bytes → tool return → Harness serialization → model context → model token output → decoded string → JSON/tool-call parsing → tool matching`

Hatanın hangi katmanda başladığını belirlemek için şu gösterimler karşılaştırılabilir:

- Byte dizisi
    
- Unicode code point dizisi
    
- Token ID dizisi


Metrikler de yalnızca metinsel eşleşmeyle sınırlı kalmamalıdır: `byte-exact match`, `code-point-exact match`, `token-exact match`, ilk farkın konumu ve gerçek tool başarı oranı birlikte incelenmelidir.

Model doğrudan kopyalama testinde başarılı olup gerçek tool call'da başarısız oluyorsa sorun tokenizer, serialization, Harness veya tool protokolünde olabilir.

## 7.5.3 Regression Tasks: Uçtan Uca ve Trajectory-Prefix Testleri

Hata atfının çıktısı, yeniden çalıştırılabilir bir regression task olmalıdır. İki test katmanı birbirini tamamlar.

|Boyut|End-to-end regression|Trajectory-prefix regression|
|---|---|---|
|Başlangıç|Başlangıç ortamı ve kullanıcı isteği|Sabitlenmiş yörünge öneki|
|Test edilen şey|Bütün iş akışının başarısı|Belirli karar sınırındaki davranış|
|Güçlü yanı|Üretim koşullarına yakınlık|Hatanın kaynağını yalıtma|
|Sınırlılığı|Hatanın hangi adımda çıktığını belirlemek zor|Tüm iş akışını kapsamaz|
|Kullanım|Genel regresyon kontrolü|Politika, tool veya karar hatasını sınama|

End-to-end testler OSWorld, AndroidWorld ve tau-bench gibi benchmark'ların temel yaklaşımıdır. Trajectory-prefix testleri ise Agent’ın belirli bir context’te vermesi gereken sonraki eylemi sınar.

Önek testinin beklenen çıktısı tek bir zorunlu yanıt olmak zorunda değildir. Kabul edilebilir eylemler kümesi tanımlanmalıdır. Örneğin:

- Önce depo kurallarını oku.
    
- Belirsiz gereksinim için kullanıcıya sor.
    
- Tehlikeli işlemi reddet veya onay iste.
    

Buna karşılık yasak eylemler de açıkça belirtilmelidir.

Örneğin bir coding task'ta test assertion'larını değiştirerek testi geçmek kabul edilemez. Benzer şekilde Agent, gerçekten çalıştırılmamış bir komutun sonucunu tamamlanma kanıtı olarak sunmamalıdır.

Hata sınıfları böylece doğrudan test tasarımına bağlanır:

- Tool-call hatası: Yanlış biçim, yanlış tool veya hatalı argüman.
    
- Belirsizlik hatası: Açıklama istemeden varsayım yapma.
    
- Anormal sonlanma: Timeout, kesilme veya tool arızasından kurtulamama.
    
- Tamamlanma hatası: Birden fazla hedefin yalnızca bir kısmını tamamlamak.
    
- Doğrulama sahteciliği: Testleri veya kabul koşullarını manipüle ederek başarı izlenimi oluşturmak.
    
- Bilgi bildirme hatası: Ortam durumuyla bildirimin uyuşmaması.
    

Bu regression veri kümesi, sonraki bölümlerde ele alınan post-training ve Agent self-evolution için de kullanılabilecek bir temel oluşturur.

## 7.5.4 İkili Karşılaştırma ve Model Sıralaması

Mutlak başarı ölçütü tanımlamanın zor olduğu durumlarda iki modelin yanıtlarını doğrudan karşılaştırmak yararlı olabilir.

### Elo ve Bradley–Terry

Elo, modellerin göreli gücünü ikili karşılaşmalardan tahmin eden bir sıralama yaklaşımıdır. İki modelin puan farkı, beklenen kazanma oranını belirler:

![Görsel](../gorseller/Pasted%20image%2020261019221642.png)

Burada \(S_A\), A modelinin karşılaşmadaki puanını temsil eder; \(K\), güncelleme büyüklüğünü belirler.

Örneğin A'nın puanı 1200, B'nin puanı 1000 ise A'nın beklenen kazanma oranı yaklaşık %76'dır. B kazanırsa, beklenmedik sonuç nedeniyle B daha fazla puan kazanır.

Bradley–Terry modeli ikili karşılaşmalardaki göreli kazanma olasılıklarını istatistiksel olarak modeller. Elo ise bu tür bir sıralamayı çevrimiçi ve artımlı biçimde güncellemenin bir yoludur. İkisi ilişkili olsa da aynı algoritma değildir.

### Kör karşılaştırmanın sınırları

Chatbot Arena gibi sistemlerde kullanıcılar model kimliğini bilmeden iki yanıt arasından seçim yapar. Böylece önceden belirlenmiş mutlak bir puanlama Rubric'ine ihtiyaç azalır.

Ancak sıralama, kullanıcıların sorduğu soruların dağılımına bağlıdır. Programlama sorularının ağırlıklı olduğu bir değerlendirmede kodlamada güçlü modeller avantaj kazanabilir. Bu sonuç, her görev türündeki üstünlüğü kanıtlamaz.

LLM-as-a-Judge kullanımında ayrıca Position Bias riski vardır. Yargıç, ilk sırada gösterilen yanıtı sistematik biçimde kayırabilir.

Azaltma yöntemleri:

1. Adayların sırasını değiştirerek değerlendirmeyi iki kez yapmak.
    
2. İki değerlendirme uyuşmadığında sonucu daha temkinli ele almak.
    
3. Gerektiğinde beraberlik veya insan incelemesi kullanmak.
    

## 7.6 Evaluation Güdümlü Model Seçimi

En güçlü görünen modeli seçmek, her iş yükü için en iyi modeli seçmekle aynı değildir. Model seçimi; performans, gecikme, maliyet, güvenilirlik ve görev türü arasında bir denge kurmalıdır.

### Temel metrikler

|Boyut|Ne ölçer?|Neden önemli?|
|---|---|---|
|Input throughput|Girdi işleme hızı|Uzun context'lerin işlenmesi|
|Output throughput|Token üretim hızı|Yanıt ve reasoning süresi|
|TTFT|İlk token'a kadar geçen süre|Algılanan tepki hızı|
|p95 latency|İsteklerin %95'inin aşmadığı gecikme|Kuyruk ve uç gecikmeleri|
|Task success rate|Görev başarısı|İşlevsel kalite|
|Cost per task|Görev başına toplam maliyet|Gerçek ekonomik verim|
|RPM / TPM|İstek ve token limitleri|Kapasite ve eşzamanlılık|
|Robustness|Dağılım dışı ve düşmanca girdilere dayanıklılık|Üretim güvenilirliği|

Prefill, context'in işlenmesiyle ilişkilidir; TTFT üzerinde etkilidir. Decode, çıktının token token üretilmesini kapsar ve üretim hızını etkiler.

Düşünme token'larının artması her zaman daha yüksek başarı anlamına gelmez. Modelin daha uzun reasoning üretmesinin kendi iş yükünüzde gerçekten değer sağlayıp sağlamadığını ölçmek gerekir.

### Performans metriklerini amaca göre seçmek

- Günlük görevlerde: `Pass@1`
    
- Birden çok denemede en az bir başarılı sonuç arandığında: `Pass@k`
    
- Birden çok denemenin tamamında güvenilirlik beklendiğinde: `Pass^k`
    
- Keşif görevlerinde, denemeler arasından en iyi sonucun önemli olduğu durumlarda: `Best@k`
    
- Açık uçlu görevlerde: çok boyutlu Rubric değerlendirmesi
    

Metrikler birbirinin yerine geçmez. Özellikle kritik işlemlerde bir kez başarılı olmakla her denemede güvenilir olmak farklı gereksinimlerdir.

### Bütçe–yetenek eğrileri

Tek bir benchmark puanı, Agent'ın uzun süreli görevlerde ne kadar iyi olduğunu göstermeyebilir. Performans; duvar saati süresi, token sayısı, tool call sayısı ve hesaplama bütçesine göre ölçülmelidir.

Kaynakta verilen RE-Bench örneği bu ayrımı vurgular: sınırlı bir sürede Agent insan uzmanları geçebilirken, daha uzun toplam çalışma süresinde insanlar öne geçebilir. Dolayısıyla kısa bütçedeki üstünlük, uzun süreli çalışma üstünlüğünü otomatik olarak kanıtlamaz.

### Model davranışı: Ne zaman okumayı bırakıp düzenlemeli?

Coding Agent'lar aynı göreve farklı eylem eşikleriyle yaklaşabilir:

- Bazıları daha çok dosya okuyup mimariyi ve testleri anlamadan değişiklik yapmaz.
    
- Bazıları daha erken düzenler ve test geri bildirimini kullanarak anlayışını geliştirir.
    

Bu farkın kaynağı hem modelin davranış politikası hem de Harness'taki sistem prompt'u olabilir.

Aynı Harness altında farklı modellerin davranışını karşılaştırmak model etkisi hakkında kanıt sağlar. Aynı modelin farklı policy'lerle davranışını karşılaştırmak ise Harness etkisini incelemeye yardım eder. Bu iki karşılaştırmayı birbirine karıştırmamak gerekir.

## 7.6.3 Agent Maliyet Analizi

Agent maliyeti yalnızca modelin token fiyatından oluşmaz. Üç ana katman vardır:

1. Model inference: Girdi, çıktı ve reasoning token'ları.
    
2. Tool calls: Harici API ücretleri, kod yürütme kaynakları ve tool sonuçlarının context'e eklenmesinden doğan token maliyeti.
    
3. Infrastructure: Veritabanları, retrieval sistemleri, kuyruklar ve trace depolaması.
    

### Context birikimi

Çok turlu Agent'larda önceki konuşmalar ve tool sonuçları sonraki LLM çağrılarına tekrar dahil edilebilir. Örneğin:

- 1. tur: 1000 girdi token'ı
        
    
- 2. tur: 2000 girdi token'ı
        
    
- 3. tur: 3000 girdi token'ı
        
    

Toplam 3000 değil, 6000 girdi token'ı tüketilir.

Bu nedenle Agent'ın tur sayısı ve her turda taşınan context boyutu maliyeti önemli ölçüde etkileyebilir.

### Optimizasyon kaldıraçları

- KV Cache reuse: Kararlı ön ekleri yeniden kullanmak.
    
- Context compression: Eski trajectory'leri ve uzun tool sonuçlarını sıkıştırmak.
    
- Model routing: Basit işleri daha hafif, zor işleri daha güçlü modellere yönlendirmek.
    
- Asynchronous batch processing: Gerçek zamanlı olmayan görevleri toplu işlemek.
    
- Budget controls: Görev başına maliyet sınırı koymak ve döngüye giren Agent'ları durdurmak.
    

Kaynakta verilen sekiz turluk iş akışında kararlı ön ek ve geçmiş sıkıştırma birlikte kullanıldığında toplam maliyet %30 azalmıştır. Tek başlarına ölçülen tasarruf oranları %28,3 ve %17,5'tir.

Bu oranlar toplanamaz; optimizasyonlar birbirleriyle etkileşir. Sıkıştırma, cache'den yeniden kullanılabilecek ön eki de kısaltabilir. Bu yüzden optimizasyonları tek tek ve birlikte ölçmek gerekir.

## 7.6.4–7.7 Model Seçimi, Sürekli Yineleme ve İstatistiksel Anlamlılık

Model seçimi tek seferlik bir karar değildir. Yeni bir modelin kamuya açık benchmark'larda daha yüksek puan alması, mevcut sistemde mutlaka daha iyi çalışacağı anlamına gelmez.

Sağlam bir karar süreci:

1. Yeni modeli kendi evaluation dataset'inizde çalıştırın.
    
2. Başarı oranını, tool-calling doğruluğunu, gecikmeyi ve maliyeti ölçün.
    
3. Farkın rastlantısal dalgalanmayla açıklanıp açıklanamayacağını değerlendirin.
    
4. Gerekirse tüm sistemi değiştirmek yerine görev bazlı model routing uygulayın.


### İstatistiksel anlamlılık

\(n\) bağımsız test vakasında \(p\) başarı oranı ölçüldüğünde standart hata yaklaşık olarak şöyledir:

![Görsel](../gorseller/Pasted%20image%2020261019221708.png)

100 vakada %70 başarı oranı için yaklaşık %95 güven aralığı, normal yaklaşım altında ±9 yüzde puandır. Bu nedenle %70'ten %73'e çıkış tek başına model değişikliğini haklı çıkarmayabilir.

Aynı görev kümesinde iki modeli karşılaştırırken paired analysis tercih edilmelidir. Her iki yapılandırma aynı görevlerde ve mümkün olduğunda aynı rastgele koşullarda çalıştırılır; ardından farklar incelenir.

Uygun yöntemler arasında şunlar bulunur:

- McNemar testi
    
- Paired bootstrap
    
- Birden fazla seed ile tekrar
    
- Çoklu karşılaştırmalarda anlamlılık kontrolü
    

Pratik karar ölçütü yalnızca puan artışı değildir: fark gürültüyü aşmalı, uygun analizde desteklenmeli ve yeniden üretilebilmelidir.

## 7.8 Observability: Üretimde Gerçekte Ne Olduğunu Görmek

Observability, Agent'ın iç işleyişini doğrudan izleyemediğimiz durumlarda log, metrik ve trace verilerinden ne olduğunu anlamamızı sağlar.

Bir Agent sisteminde observability şu sorulara yanıt vermelidir:

- Hangi adımda hata oluştu?
    
- Hangi tool daha sık başarısız oluyor?
    
- Hangi görevler gereğinden fazla tur çalıştırıyor?
    
- Hangi retrieval sorguları boş sonuç dönüyor?
    
- Maliyet hangi görevlerde anormal derecede yükseliyor?
    
- Yeni sürüm hangi davranışları değiştirdi?
    

### Trace ve span

Bir görev yürütmesi bir `trace` oluşturur. İçindeki her LLM çağrısı, tool call veya retrieval işlemi bir `span` olarak kaydedilebilir.

Span verileri genellikle girdi/çıktı, başlangıç ve bitiş zamanı, token kullanımı ve hata bilgilerini içerir. Span'lerin ebeveyn–çocuk ilişkileri yürütme ağacını oluşturur.

OpenTelemetry genel amaçlı distributed tracing standardı olarak, OpenInference ise LLM uygulamalarına özgü semantik alanlarda kullanılabilir.

### Observability'den evaluation'a

En değerli kapalı döngü şudur:

Üretim trajectory'lerini topla

Başarısız ve şüpheli vakaları seç

Hassas verileri maskele

Yeni evaluation ve regression task'ları oluştur

Değişikliği test et ve yeniden değerlendir

Böylece evaluation dataset'i bir kez hazırlanıp unutulan statik bir varlık olmaktan çıkar. Gerçek sistemde görülen başarısızlık kalıpları yeni testlere dönüşür.

## 7.9 Benchmark Raporlarından Sistem İyileştirmelerine

Benchmark puanı bir sorunun varlığını gösterir; çözümün ne olduğunu tek başına açıklamaz.

Kaynakta verilen AndroidWorld örneğinde Wi-Fi görevlerinde yoğunlaşan başarısızlıklar, Agent'ın gezinme bilgisinden yoksun olması veya UI temsilinin eksikliği gibi farklı hipotezlere yol açmıştır.

İzlenen yaklaşım şöyledir:

1. Başarısızlıkları görev ve yetenek etiketlerine göre grupla.
    
2. İlgili trajectory'leri incele.
    
3. Gözlem, reasoning, eylem ve doğrulama aşamalarından hangisinin sorunlu olduğunu belirle.
    
4. Tek bir değişkeni değiştiren deney tasarla.
    
5. Başarıyı maliyet ve gecikmeyle birlikte değerlendir.
    
6. Sonuç yeterince güçlü ise daha geniş benchmark'a geç.
    

Kaynakta UIAutomator öğe ağacının kullanılması başarıyı artırmış, fakat token maliyetini yükseltmiştir. Ardından gereksiz düğümler sıkıştırılarak başarı korunmuş ve token tüketimi azaltılmıştır.

Buradaki temel ders: Daha fazla bilgi sağlamak başarıyı artırabilir, ancak gereksiz bilgi context'i gürültüye boğabilir. Harness'in görevi yalnızca bilgi eklemek değil, gerekli bilgiyi doğru biçimde sunmaktır.

Dört görevlik bir dilimde elde edilen %100 başarı, tüm benchmark'ta %100 başarı anlamına gelmez. Sonuç, yalnızca test edilen kapsam için kanıttır. Dağıtım kararı daha geniş ve tekrarlı değerlendirme gerektirir.

## 7.10 Üretim Düzeyinde İç Evaluation Altyapısı

Dış evaluation, Agent'ın ne kadar iyi çalıştığını ölçer. İç evaluation altyapısı ise hangi sistem değişikliğinin iyileşmeye neden olduğunu anlamaya çalışır.

Kaynak beş temel mekanizmayı öne çıkarır.

### 1. Ablation study

Bir özelliği devre dışı bırakıp performans değişimini ölçerek o özelliğin gerçek katkısı araştırılır. Düşünme modu, context compression, otomatik memory ve arka plan görevleri buna örnek olabilir.

Her özellik bağımsız biçimde kapatılabilmelidir. Düzenli ablasyonlar, artık katkı sağlamayan özellikleri de ortaya çıkarabilir.

### 2. A/B testing

Eski ve yeni sürümler farklı kullanıcı gruplarında karşılaştırılır. Deneylerde:

- Birden çok varyant kullanılabilir.
    
- Değiştirilen mekanizma ile hedef metrik ayrılmalıdır.
    
- Kullanıcı memnuniyeti ve hata oranı gibi guardrail metrikleri izlenmelidir.
    
- Örneklem büyüklüğü ve dağılım istatistikleri kaydedilmelidir.
    

Örneğin plan dosyasını kısaltmak bir mekanizma metriğidir. Gerçek hedef toplam oturum maliyetini düşürmekse, daha kısa planın ek düzenleme turlarına neden olup olmadığı da ölçülmelidir.

### 3. Feature flags

Özellikleri kodu yeniden dağıtmadan açıp kapatmayı, kademeli yayın yapmayı ve gerektiğinde geri almayı sağlar.

- Compile-time flags: Bazı özellikleri derleme sırasında üründen çıkarabilir.
    
- Runtime flags: Yapılandırmayı çalışma zamanında yönetebilir.
    

Bu altyapı deney ve rollback için temel bir mimari bileşendir.

### 4. Prompt sensitivity evaluation

System prompt değişiklikleri de kod değişiklikleri gibi sürümlenmeli ve regression testlerinden geçmelidir. Aynı yapılandırmanın deterministik biçimde aynı prompt'u üretmesi, değişikliklerin etkisini izlemeyi kolaylaştırır.

### 5. Privacy-aware analytics

Evaluation için veri toplamak, kullanıcı gizliliğini ihlal etmek anlamına gelmemelidir. Hassas verilerin toplanmasını tip sistemi veya başka doğrulama mekanizmalarıyla sınırlandırmak, gizlilik kısıtlarını yalnızca dokümantasyonda bırakmamak açısından önemlidir.

## 7.11 Simülasyon Ortamları: Evaluation'dan Post-Training'e

Evaluation ortamı mevcut yeteneği ölçer. Simulation environment ise Agent'ın tekrar tekrar etkileşime girerek öğrenebileceği bir eğitim ortamı sağlar.

İki önemli gereksinim vardır:

- Reliable reset semantics: Her episode temiz ve tanımlı bir başlangıç durumuna dönmelidir.
    
- High throughput: Eğitim için binlerce değil, çok daha fazla etkileşimin makul sürede yürütülmesi gerekir.
    

Bir Rubric veya deterministik verifier, uygun koşullarda doğrudan reward function olarak kullanılabilir. Böylece evaluation sırasında başarıyı belirleyen ölçütler, RLVR gibi yöntemlerde eğitim sinyaline dönüşebilir.

### Simulation fidelity ve throughput dengesi

Yüksek sadakatli ortamlar gerçek dünyayı daha doğru temsil edebilir, ancak daha pahalıdır. Daha basit ortamlar yüksek throughput sağlayabilir, fakat gerçek dünyaya aktarımda eksiklikler yaratabilir.

Kaynakta verilen örnekler farklı noktaları temsil eder:

- AWorld: Dijital görevler için denetlenebilir ve paralelleştirilebilir MCP sandbox yaklaşımı.
    
- AndroidWorld: Mobil uygulama otomasyon görevleri.
    
- OSWorld: Masaüstü ortamları ve sanal makine tabanlı görevler.
    
- RoboTwin2: Robot manipülasyonu için fizik tabanlı simülasyon.
    

### Domain randomization

Simülasyon ile gerçek dünya arasındaki farkı azaltmak için nesne konumları, yönelimler, görünüm ve sensör koşulları rastgeleleştirilebilir.

Amaç, modelin tek bir sabit yapılandırmayı ezberlemesini önleyerek farklı koşullara genellemesini sağlamaktır. Aşırı rastgeleleştirme ise görevi gereksiz biçimde zorlaştırabilir; bu nedenle sadakat, rastgeleleştirme ve throughput birlikte değerlendirilmelidir.

## 7.12 Bölümün Bütünsel Çıkarımı

Bu bölümün ana tezi, Agent değerlendirmesinin yalnızca benchmark puanı üretmek olmadığıdır. Güvenilir bir sistem şu döngüyü tamamlar:

Başarı ölçütünü tanımla → veri kümesini kur → Agent'ı çalıştır → sonucu doğrula → ilk hatayı ve kök nedeni bul → regression task oluştur → değişikliği test et → istatistiksel olarak doğrula → üretimde izle → yeni vakaları evaluation'a geri ekle.

Üç ayrım özellikle önemlidir:

1. Başarı ile doğru raporlama farklıdır. Ortam durumu doğru olsa bile Agent yanlış bilgi verebilir.
    
2. Model hatası ile sistem hatası farklıdır. Eksik observation channel veya bozuk tool protocol, model değiştirilerek çözülemeyebilir.
    
3. Evaluation ile training farklıdır. Evaluation ölçüm yaparken simulation environment, yeterli reset ve throughput sağlandığında öğrenme sürecinin parçası olabilir.


Sonuç olarak evaluation, model seçimini destekleyen bir tool’dan daha fazlasıdır. Harness değişikliklerini sınar, üretim hatalarını sistematik biçimde toplar, regression testlerini günceller ve post-training için doğrulanabilir sinyaller sağlayabilir. Agent mühendisliğini sezgisel deneme-yanılmadan deneysel ve kanıta dayalı sistem geliştirmeye taşıyan mekanizma budur.