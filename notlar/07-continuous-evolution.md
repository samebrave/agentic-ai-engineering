# Continuous evolution of agents

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter9/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

bugünün agent'ları çarpıcı bir yetenek paradoksuyla karşı karşıya. daha önce hiç görmedikleri karmaşık görevleri zero-shot çözebiliyorlar, ama benzer görevleri on bin kez işledikten sonra ertesi gün ilk gün yaptıkları hatayı yapabiliyorlar. model işe başladıktan sonra, tıpkı yeni bir çalışan gibi günlük işten öğrenerek gelişebilir mi? deneyimden otonom biçimde öğrenebilme - bugün sürekli öğrenme denen şey - agent'ın görevi tamamlayabilmeden güvenilir biçimde çalışabilmeye geçişindeki kilit yetenek haline geliyor. bu aynı zamanda yeni nesil modellerin merkezi araştırma konusudur. bugünkü sürekli öğrenme kavramı, "yeni görevi öğrenince eskisini unutma" biçimindeki eski araştırma hattıyla aynı problem değildir. daha zor olanı, günün deneyiminde nelerin doğru nelerin yanlış yapıldığını modele kimsenin söylememesidir. şimdilik modelin kendi sürekli öğrenme yeteneği hala fazlasıyla yetersiz. 

in-context learning, durum yönetimi ve sıkıştırma, agent'ın mevcut görev içinde uyum sağlamasını mümkün kılar, ama context sona erdikten sonra bu değişim bir sonraki göreve kendiliğinden taşınmaz. konuşmaları belleğe kaydetmek de yeni bir davranışı öğrenmiş olmak anlamına gelmez. 

kolayca karşılaştırılan bir ayrım: yaşananları saklamak, yaşananlardan öğrenmekle aynı şey değildir. yüz trajectory'i uzun bir contexte ya da vektör veri tabanına koymak, modelin gerektiğinde bir vakayı geri bulmasına yardımcı olabilir, ama vakalar arası karşılaştırmayı kendiliğinden tamamlamaz. hangi adımlar başarılı trajectory'lerde tekrar tekrar ortaya çıkıyor, hangi uygulamalar yalnızca eski sürüm bir arayüzde işe yarıyor, belirli bir başarı doğru stratejiden mi yoksa ortamın tesadüfünden mi geliyor. öğrenme, sistem *değerlendirme, karşılaştırma, genelleme, doğrulama* işini etkin biçimde tamamladıktan sonra gerçekleşir. logun diske yazıldığı anda değil. 

Peki modelin her görevden sonra doğrudan kendini eğitmesine neden izin vermiyoruz? Çünkü üretim ortamları nadiren temiz öğrenme sinyalleri sağlar. Kullanıcı memnuniyeti kurallara uygunluk anlamına gelmez; yerel parametre güncellemeleri yetenek unutmasına, politika kaymasına veya güvenlikte gerilemeye de yol açabilir. Çalışan bir modelin doğrulanmamış geri bildirime dayanarak kendi parametrelerini doğrudan değiştirmesine izin verilirse, hatalı deneyimler ve prompt injection kalıcı hâle gelip sonraki görevlerde büyümeyi sürdürebilir. Öte yandan temel modellerin periyodik eğitimi genel yetenekleri geliştirebilir, ancak her Agent’ın her gün karşılaştığı özel kuralları, tool değişikliklerini ve yerel deneyimleri zamanında özümseyemez.

Bu nedenle, modelin kendisi henüz güvenilir biçimde sürekli öğrenemiyorken,“öğrenme”nin önce modelin çevresinde otonom bir sistem olarak kurulması gerekir; bu kitap buna, model ağırlıkları düzeyindeki sürekli öğrenmeden ayırmak için sürekli evrim diyor: çalışma kanıtlarını kaydetmek, sonuçları ve süreci doğrulamak, birden çok trajectory’den ortak örüntüleri çıkarmak, ardından bilginin mi, talimatların mı, programların mı yoksa model parametrelerinin mi güncelleneceğine karar vermek. Bütün değişiklikler önce birer aday sürüm hâline gelir; ancak regresyon testlerinden ve güvenlik denetimlerinden geçtikten sonra bir sonraki çalışma turunu değiştirebilir.

![Görsel](../gorseller/Pasted%20image%2020261031213558.png)

## 7.1 çalışma trajectory'lerinden öğrenme sinyali elde etmek

sürekli evrimin başlangıç noktası, bölüm 7'de anlatılan değerlendirmedir. sistem görevin tamamlanıp tamamlanmadığını bilmiyorsa ve hangi adımın başarıya ya da başarısızlığa yol açtığını da bilmiyorsa, dil modelinin ürettiği reflection ancak bir tahmindir

bir trajectory'i değerlendirmek, özünde üç soruyu sırayla yanıtlamaktır: iş yapıldı mı, izin verilen biçimde mi yapıldı ve kullanıcı için rahat bir deneyim miydi? 

![Görsel](../gorseller/Pasted%20image%2020261031213807.png)

en üstteki sonuç doğrulayıcısı iş gerçekten yapıldı mı sorusunu yanıtlar. test sonuçlarını, veritabanı durumunu ve tool dönüşlerini okur: bir kodlama agent'ı testleri, tip kontrollerini ve performans benchmarklarını çalıştırabilir, kullanıcı adına iade işlemi yapan bir agent sipariş durumunu ve gerçekleşen iade tutarını sorgulayabilir. bu tür sinyaller ortamdaki gerçek durumdan gelir ve genellikle modelin kendi davranışına ilişkin anlatımından daha güvenilirdir, bu nedenle üç katman içinde ilk kurulması gereken katmandır. 

ortadaki süreç doğrulayıcısı izin verilen biçimde mi yapıldı sorusunu yanıtlar. sonucun doğru olması sürecin de doğru olduğu anlamına gelmez. başarısız test durumlarını silmek de testleri geçirir, kullanıcıya sözlü olarak iadenizi 7 gün içinde yapacağız, lütfen sabırlı olun demek de geçici bir memnuniyet geri bildirimi getirebilir. bu katman iş kurallarını, yetkileri ve eylem dizisini sabitleyerek sonuca ulaşıldı ile sonuca izin verilen yoldan ulaşıldı arasını ayırır. politika kütüğü, yetki tablosu ve eylem trajectory'si kesin biçimde ifade edilebilir. dolayısıyla bu katman da kodla karara bağlanabilir.

alttaki kalite doğrulayıcısı kullanıcı için rahat bir deneyim miydi sorusunu yanıtlar. örneğin müşteri hizmetlerinin sabırlı olup olmadığı, kurallara uygun sınırlar içinde bir çözüm sunup sunmadığı, bir araştırma raporunun kilit kanıtı yakalayıp yakalamdığı, üretilen metnin doğal ve derli toplu olup olmadığı. bu boyutlar işin yapılıp yapılmadığını belirlemez ancak ux etkiler. burada bölüm 7'de tanıtılan llm-as-a-judge kullanılabilir. değerlendirme ölçeğini önceden tanımlayıp doğrudan her maddeye ayrı puan vermesini ve trajectory'den kanıt göstermesini istemek gerekir.

-Buradaki **LLM-as-a-judge** (“hakem olarak büyük dil modeli”), başka bir yapay zekâ modelinin çıktısını belirli ölçütlere göre değerlendirmek için bir LLM kullanma yöntemidir.

Örneğin, bir müşteri hizmetleri konuşmasını değerlendirirken modele önceden tanımlanmış bir ölçek verirsiniz: “Sabırlı mıydı?”, “Kurallara uygun bir çözüm sundu mu?” Her ölçüt için ayrı puan vermesini ve puanını konuşmadaki belirli ifadelerle gerekçelendirmesini istersiniz. Böylece yalnızca son yanıtı değil, yanıtın oluştuğu konuşma akışını—metindeki ifadeyle *trajectory*’yi—değerlendirir.

Bu yöntem, görev başarıyla tamamlandı mı sorusundan ziyade deneyimin niteliğini ölçer. Ancak modelin değerlendirmesi kusursuz veya tarafsız değildir; bu yüzden ölçütleri açık tanımlamak, gerçek örneklerle kontrol etmek ve önemli değerlendirmelerde insan incelemesiyle desteklemek gerekir.-

müşteri hizmetleri agent'ı örneğinde tablo 9-1, bu üç katmanı madde madde puanlanabilen yedi boyuta açar: görev sonucu sonuç katmanına aittir, kural uyumu, gizlilik sınırı ve söz-eylem tutarlılığı süreç katmanına; ifade kalitesi ve kurallara uygun alternatif kalite katmanına aittir; olgusal güvenilirlik ise iki katmana yayılır --tool dönüşleriyle tek tek karşılaştırılabilen kısmı kodla doğrulanır kalanı rubrice bırakılır

Tablo 9-1 Müşteri Hizmetleri Agent’ı için Trajectory Değerlendirme Boyutları

![Görsel](../gorseller/Pasted%20image%2020261031214513.png)
![Görsel](../gorseller/Pasted%20image%2020261031214527.png)

doğrulayıcının çıktı biçimi, onun bir öğrenme sinyali olup olamayacağını belirler. tek bir toplam puan yalnızca bir çalıştırmanın ne kadar iyi geçtiğini yansıtır, neyin değiştirilmesi gerektiğini göstermez. sonraki aşamaların öğrenebileceği bir değerlendirme en az dört öğe içermelidir: görevin başarılı, kısmen başarılı ya da başarısız olduğu; her boyut için ayrı bir sonuç; her sonucun dayandığı kanıtın konumu (hangi konuşma turu, hangi tool call) ve hata türü etiketi. ayrıca kanıt yetersizken doğrulayıcının puan vermeyi reddedebilmesi gerekir. düşük güvenli bir sonucu olgu olarak sabitlemektense o vakayı öğrenme kümesinin dışında bırakmak yeğdir. ancak bu dört öğe elde olduğunda, sonraki bölümde ele alınan dört güncelleme yöntemi bilgiyi mi, promptu mu, programı mı yoska model parametrelerini mi güncellemek gerektiğine karar verebilir. 

## 7.2 agent'ın sürekli evrimi için dört yöntem

öğrenme sinyali, agent'ın değişmesi gerektiğini söyler, ama değişimin nerede gerçekleşmesi gerektiğini söylemez. güncelleme biçimini seçerken birincil ölçüt, deneyimin ne kadar süredir var olduğu değil, hedeflenen yeteneğin belirli bir taşıyıcıyla doğal biçimde ifade edilip edilemeyeceğidir. olgular ve deneyimler bilgi dokümanına yazılmaya uygundur; dille açıkça anlatılabilen stratejiler prompt'a veya skill'e yazılmaya uygundur; kesin biçimde yürütülebilen süreçler ve kısıtlar programa yazılmaya uygundur, algı, dil üslubu ve örtük stratejiler gibi yüksek boyutlu yetenekler ise model parametrelerine girmek zorundadır. şekil 9-3 bu dört yolu ve aralarındaki ilişkiyi gösteriyor.

![Görsel](../gorseller/Pasted%20image%2020261031220001.png)

tablo 9-2 derli toplu bir karşılaştırma sunuyor. dört yol birbirini dışlamaz: tıbbi görüntüleme agent'ı lezyonları tanımak için parametrelere dayanır, güncel kılavuzları bilgi tabanından alır, risk göstergelerini kodla hesaplar, müşteri hizmetleri modelinin doğal ses tonu post-trainingden gelir, kuruma özgü politikalar bilgi ve skill tarafından sağlanır, kritik uyumluluk kuralları ise sunucu tarafındaki kodla güvence altına alınır.

![Görsel](../gorseller/Pasted%20image%2020261031220139.png)

Aynı yetenek birden çok taşıyıcıya bölünebilir: olgular bilgi tabanına, istisnaları açıklayan ilkeler bir Skill’e, atlanamayacak izinler yine programın kapısına, yüksek boyutlu tanıma yeteneği ise parametrelere gider. Yönlendirmenin sonucu yalnızca bir güncelleme önerisidir; henüz yayına çıkma yetkisi kazanmamıştır.

### 7.2.1 deneyimi bilgi olarak biriktirmek

en hafif evrim biçimi, birden çok çalışmada tekrar tekrar ortaya çıkan deneyimi retrieval yapılabilir bilgi doc'a dönüştürmektir. burada söz edilen deneyim bilgi tabanı depolama, indeksleme ve retrieval teknolojilerini bölüm 3 ile paylaşır, ama bilginin kaynağı ve doğrulama hedefi farklıdır. bölüm 3 ağırlıklı olarak kullanıcı konuşmalarından, dokümanlardan ve veri kümelerinden "kullanıcı ve dünya nasıldır" ı çıkarır. bu bölüm ise agent'ın eylem trajectorylerinden ve sonuçlarından hangi koşullarda ne yapılmalıyı çıkarır. örneğin bu havayolu özel yemeklerin yirmi dört saat önceden ısmarlanmasını şart koşuyor alan bilgisidir, bilet almadan önce özel yemek son başvuru zamanını kontrol et, yoksa ödemeyi yaptıktan sonra talebin karşılanamayacağını fark edersin ise eylem deneyimidir.

ham trajectory resmi bir bilgi birimi olmaya uygun değildir, hem uzun hem gürültülüdür, tool'ların ham çıktısını, tesadüfi sapmaları ve ortam ayrıntılarını içerir. daha sağlam bir sistem üç katman veri saklar: denetim için değişmez ham trajectory'ler, tek bir çalışmanın başarısını, başarısızlığını ve aday derslerini kaydeden çalışma analizleri; ve aynı türden birden çok trajectory'nin karşılaştırılıp kümelenmesi ve genellenmesiyle oluşan, geleceğe dönük md bilgi dokümanları. resmi bir doc genellikle uygulanabilir senaryoyu, önerilen stratejiyi, yasaklanan uygulamaları, istisna koşullarını, kanıt kaynaklarını ve en son doğrulama zamanını açıkça yazar; tek bir görevin tüm sürecini yeniden anlatmaz.

bu tasarım, bölüm 3'teki user-as-code ile aynı iki aşamalı düşünceyi taşır. user-as-code önce konuşmadaki olguları değişmez bir log'a ekler, sonra yapılandırılmış kullanıcı modelini periyodik olarak yeniden kurar; deneyim öğrenmesi de aynı şekilde önce kanıtı saklamalı, sonra offline olarak değiştirilebilir bilgiyi üretmelidir. şekil 9-4 bu süreci gösteriyor. kaydetmeyi düzenlemekten ayırmak, tek bir tesadüfi başarının ya da bir ağ arızasının agent'ı anında değiştirmesini önler, ayrıca sisteme birden çok başarı ve başarısızlığı gördükten sonra ortak yanı belirleme imkanı verir. 

![Görsel](../gorseller/Pasted%20image%2020261031222243.png)

# örnek çalışma:

### 1. Agent farklı görevlerde deneyim kazanıyor

**Birinci çalışma:**

Agent doğrudan yeni endpoint’i yazıyor, testleri çalıştırıyor ve her şey yerelde geçiyor. Ancak PR incelemesinde mevcut projede zaten kullanılan bir `ApiResult` modeli olduğunu fark ediyor. Yazdığı yeni hata modeli gereksiz tekrar yaratmış.

**İkinci çalışma:**

Agent önce mevcut repository ve network katmanını inceliyor. `ApiResult` kullanımını bulup yeni endpoint’i aynı yapıya göre yazıyor. Testler ve inceleme sorunsuz geçiyor.

**Üçüncü çalışma:**

Agent yine önce mevcut yapıyı inceliyor; fakat bu kez endpoint’in döndürdüğü hata biçimi mevcut modelle tam örtüşmüyor. Mevcut yapıyı körü körüne uygulamak yerine farkı tespit edip ekibe açıklıyor.

Burada üç ayrı görev var. Her birinin bütün tool çıktıları ve adımları kendi **ham trajectory** kaydında tutuluyor.

### 2. Tek tek çalışmaları analiz ediyoruz

Her çalışma için kısa bir analiz çıkarılıyor:

|Çalışma|Sonuç|Aday ders|
|---|---|---|
|1|Gereksiz yeni hata modeli oluşturuldu|Mevcut hata modelini önce araştır|
|2|Mevcut yapıyla uyumlu çözüm üretildi|Repository ve network katmanını incelemek işe yaradı|
|3|Mevcut yapıdaki istisna erkenden fark edildi|Mevcut modeli bulmak yeterli değil; yeni gereksinime uyumunu da kontrol et|

Buradaki önemli nokta: Üçüncü çalışma olmasaydı sistem, **“Mevcut `ApiResult` modelini her zaman kullan”** gibi fazla katı bir ders çıkarabilirdi. Çalışmaları karşılaştırınca daha iyi kural ortaya çıkıyor.

### 3. Deneyim, yeniden kullanılabilir bilgiye dönüşüyor

Birden fazla çalışmanın ardından şöyle bir Markdown bilgi dokümanı oluşturulabilir:

Markdown

```
# Yeni API Endpoint’i Eklerken Mevcut Hata Modelini Kontrol Et

## Ne zaman uygulanır?

Mevcut bir Android projesine network endpoint’i veya repository
fonksiyonu eklenirken.

## Önerilen yaklaşım

1. Mevcut network ve repository katmanındaki hata modellerini bul.
2. Yeni endpoint’in hata yanıtlarını mevcut modelin kapsayıp
   kapsamadığını karşılaştır.
3. Model yeterliyse mevcut yaklaşımı kullan.
4. Yetersizse farkı açıkla ve gerekli genişletmeyi değerlendir.

## Kaçınılacak durum

Mevcut uygulamayı incelemeden paralel bir hata modeli oluşturma.

## İstisna

Endpoint’in hata sözleşmesi mevcut modele uymuyorsa sırf
tutarlılık adına bilgiyi kaybettirerek modele zorlama.

## Kanıt

- Çalışma 1: Gereksiz paralel model oluşturuldu.
- Çalışma 2: Mevcut model kullanılarak uyum sağlandı.
- Çalışma 3: Mevcut modelin yeni hata biçimini kapsamadığı görüldü.

## Son doğrulama

[Tarih] — ilgili proje ve API sürümü.
```

### 4. Sonraki görevde ne oluyor?

Agent’a yeni bir endpoint görevi geldiğinde sistem ilgili deneyim dokümanını bulup context’e ekleyebilir. Agent böylece önceki görevlerde öğrenilen yaklaşımı uygulamaya **aday** olarak görür.

Ama doküman bir kez yazıldı diye sonsuza kadar doğru sayılmaz: proje mimarisi veya API değişmiş olabilir. Bu yüzden **kanıt, uygulanacağı koşullar ve son doğrulama tarihi** önemlidir.

Kısaca akış:

```
Ham görev kayıtları
        ↓
Her çalışmanın analizi
        ↓
Birden fazla çalışmayı karşılaştırma
        ↓
Koşulları ve istisnaları belirlenmiş bilgi dokümanı
        ↓
Benzer yeni görevde dokümanı bulup kullanma
```

Burada modelin ağırlıkları değişmiyor. Agent’ın gelişimi, **önceki deneyimlerden çıkarılan bilginin gelecekte doğru anda context’e verilmesiyle** sağlanıyor.


deneyim dokümanı basit bir trajectory özeti değildir. gerçekten aktarım değeri taşıyan içerik karşılaştırmadan doğar. 
aynı türden başarılı trajectory'ler ne yapmış, başarısızlarda ne eksik, belirli bir strateji hangi ortam sürümlerinde işe yaramış, hangi ön koşullar altında çökmüş. bölüm 3 bilgi çıkarımını, kümelemeyi ve retrieval'ı zaten tanıttığı için algoritmalar yinelenmiyor bunun yerine traj. değerlendirmesinin çıkarım koşuluna nasıl dönüştüğüne ve çıkarılan bilginin sonraki görevlerdeki başarımı artırıp artırmadığına odaklanıyor. eksiksiz bir bilgi damıtma boru hattı beş adıma ayrılabilir. önce değişmez trajectory'ler ve ortam sonuçları saklanır; sonra tek bir çalışma için görev türünü, gereken yetenekleri, gözlenen stratejileri, hataları ve istisnaları sıralayan yapılandırılmış bir analiz üretilir; ardından aynı türden çalışmalar görev ailesine bir araya toplanır ve her aday örüntü için hangi traj destekliyor hangileri çürütüyor biçiminde bir kanıt tablosu sunulur; yalnızca destek eşiğine ulaşan adaylar resmi dokümana yazılır, son olarak damıtmaya katılmamış yeni görevler üzerinde aktarım etkisi sınanır. resmi bilginin aday analizlerden ayrı depoda tutulması, sistemin ham kanıtı bozmadan yeniden genelleem yapmasına ve ortam sürümü değiştiğinde belirli bir sonucu tam olarak geri almasına imkan verir.

# örnek2

Şunu söylüyor: **Agent’ın geçmiş görevlerini özetlemek yetmez; birden fazla deneyimi karşılaştırıp hangi yaklaşımın hangi koşullarda işe yaradığını bulmak ve bunu yeni görevlerde sınamak gerekir.**

Android örneğiyle açalım. Diyelim ki agent’tan farklı projelerde API endpoint’i eklemesi istendi.

### Önce görevlerin kayıtları saklanır

Her çalışmanın isteği, tool çağrıları, sonuçları ve hataları birer **trajectory** olarak korunur. Bunlar ham kanıttır.

```text
Görev 1: Endpoint eklendi → testler başarısız.
Görev 2: Endpoint eklendi → testler başarılı.
Görev 3: Endpoint eklendi → testler başarılı.
```

### Sonra her görev ayrı ayrı incelenir

Örneğin:

- **Görev 1:** Agent mevcut test altyapısını incelemeden yeni test yazdı; test ortamıyla uyumsuzluk oluştu.
- **Görev 2:** Önce mevcut testleri inceledi, projenin kullandığı test altyapısıyla ilerledi; test geçti.
- **Görev 3:** Önce testleri inceledi; fakat proje farklı bir modüldeydi ve ek yapılandırma gerekiyordu. Agent bunu fark ederek testi uyarladı.

Bunlar henüz yalnızca **tek tek çalışma analizleri**.

### Ardından çalışmalar karşılaştırılır

Sistem şu soruları sorar:

- Başarılı görevlerde ortak ne vardı?
- Başarısız görevde ne eksikti?
- Başarılı görünen yöntem hangi koşullarda işe yaradı?
- Aynı yöntem hangi durumda yetersiz kaldı?

Böylece aday bir ders çıkar:

> “Test yazmadan önce ilgili modülün mevcut testlerini ve yapılandırmasını incele.”

Ama bu dersin **kanıt tablosu** da tutulur:

| Çalışma | Aday dersi destekliyor mu? | Gözlem |
|---|---|---|
| Görev 1 | Destekliyor | İnceleme yapılmadı; uyumsuz test yazıldı |
| Görev 2 | Destekliyor | İnceleme yapıldı; uygun test yazıldı |
| Görev 3 | Destekliyor, koşullu | İnceleme yararlıydı ama modüle özel yapılandırma da gerekti |

Burada dikkat: Bunlar örnek kayıtlar. **Tek başlarına kesin nedensellik kanıtlamazlar.** Daha fazla çalışma veya kontrollü karşılaştırma, dersin güvenilirliğini artırır.

### Yeterli kanıt varsa resmi bilgiye yazılır

Sistem, belirlediği destek eşiğine ulaşan dersi deneyim dokümanına alır:

```md
## Android projesinde test eklerken

Önce ilgili modüldeki mevcut testleri ve test yapılandırmasını incele.

Proje çok modüllüyse, başka bir modülün kullandığı altyapının
hedef modülde de kurulu olduğunu varsayma.

Kanıt: Görev 1, 2 ve 3.
Geçerlilik: İncelenen proje sürümleri.
```

### Son olarak yeni görevde gerçekten işe yarıyor mu diye bakılır

Bu bilgi, çıkarılırken kullanılmamış **yeni görevlerde** denenir. Örneğin:

- Bazı yeni görevlerde agent’a deneyim dokümanı verilir.
- Benzer diğer görevlerde verilmez.
- Test başarısı, hata sayısı veya görevi tamamlama gibi sonuçlar karşılaştırılır.

Amaç şu soruya cevap vermektir:

> “Bu bilgi kulağa mantıklı geliyor mu?” değil, **“Yeni görevlerde gerçekten daha iyi sonuç almaya yardımcı oluyor mu?”**

Metnin anlattığı beş adım şöyle:

```text
1. Ham trajectory’leri sakla
2. Her çalışmayı ayrı analiz et
3. Benzer çalışmaları karşılaştır; destekleyen ve çürüten kanıtı topla
4. Yeterli kanıt varsa genellenmiş deneyim dokümanı oluştur
5. Bilgiyi yeni görevlerde sınayarak etkisini ölç
```

Son cümlenin anlamı da şu: **Ham kayıtlarla çıkarılan kurallar ayrı tutulur.** Yarın proje veya ortam değişirse eski ham kayıtları bozmadan kuralları yeniden değerlendirebilir, artık geçersiz bir kuralı kaldırabilirsin.

GAIA deneyim öğrenmesi bunun sezgisel bir örneğini veriyor. GAIA1, arama, web sayfası okuma, dosya işleme ve hesaplamayı bir arada gerektiren çok adımlı sorular içerir; AWorld2 ise Agent’ı çalıştıran, bu tool’ları çağıran ve trajectory’leri saklayan yürütme ortamını sağlar. Birincisi sınav kâğıdı gibiyse ikincisi sınav salonu ve deney kayıt sistemidir. Eski usul yaklaşım, bir görev başarıyla tamamlanır tamamlanmaz strateji özeti üretip bunu vektörleştirerek depoya yazmaktı; daha titiz bir uygulama ise önce GAIA cevap doğrulamasını veya başka bir ortam doğrulayıcısını kullanarak çalışmaları başarılı, kısmen başarılı ve başarısız olarak etiketler, ardından aynı görev ailesindeki birden çok yolu karşılaştırır. Başarılı trajectory’ler aday stratejilere katkı verir, başarısız trajectory’ler dışlayıcı bilgiye katkı verir, kısmen başarılı trajectory’ler ise“hangi bölüm işe yaradı, hangi bölümde hâlâ sorun var”ayrımını yapmaya yardımcı olur. Reflexion’ın3 önerdiği doğal dil reflection’ı aday derslerin üretilmesine katılabilir, ama reflection’ın kendisi kanıt değildir; yalnızca ortam sonucuyla örtüşen, trajectory’ler arasında destek bulan ve yeni görevlerde olumlu aktarım gösteren içerik resmî deneyim dokümanına girmelidir.

### 9.2.2 deneyimi talimat olarak yazmak

deneyim bilgi tabanı agent'a başvurabileceği malzemeyi verir, prompt ve skill ise nasıl davranması gerektiğini belirler. deneyimi talimata yükseltmeye ancak birbirine benzer çok sayıda yörünge aynı strateji hatasını üst üste çıkardığında ve o hata sözcüklerle açıkça betimlenebildiğinde değer. önce üç kavramı ayıralım: sistem prompt'u bütün görevlerde geçerlidir, skill yalnızca belirli bir alan ya da tool’la eşleştiğinde talep üzerine yüklenir, program/harness ise yetkileri ve diğer katı kısıtları üstlenir.

andrej karpathy bu yaklaşımı sistem promptu öğrenmesi diye adlandırır: model bir sorunla karşılaştıktan sonra tek bir açık cümleyle gelecekteki kendisini uyarır. dspy bir geliştirme kümesi üzerinde talimat ve örnek arar, opro geçmiş promptlar ve onların puanlarından yeni prompt önerileri üretir, gepa başarısız yörüngeler üzerine doğal dilde yapılan düşünmelerden prompt önerileri üretip eler. bu yöntemler offline toplu optimizasyona uygundur, üretim ortamında ise denetlenebilir en küçük güncelleme önerilerini kullanmak ve hızlı geri alma yolunu açık tutmak daha yerindedir.

sistem promptu öğrenmesi, 2.bölümdeki prompt eng. ile aynı şey değildir. 2.bölüm iyi bir prompt'un nasıl düzenleneceğini tartışıyordu: bu bölüm ise hangi geri bildirimin bir değişikliği tetiklemeye yeteceğini ve güncelleme önerisinin nasıl güvenle yayına alınacağını tartışıyor. değişiklik, kaynağı belirtilmiş en küçük diff olmalı, modele her seferinde promptun tamamını yeniden yazdırmak değil. bu 1.bölümde adı konan en küçük diff + geri alınabilirlik örüntüsünün ta kendisidir. aday sürüm hem hatayı tetikleyen sınır kümesinde hem de halihazırda çalışan koruma kümesinde sınanmalıdır: ilki iyileşmeli, ikincisi gerilememelidir. 

### örnek3

Bu bölüm, agent’ın deneyimlerinden çıkan bir bilginin **ne zaman kalıcı davranış kuralına dönüştürülmesi gerektiğini** anlatıyor.

Önce dört şeyi ayıralım:

| Katman | İşlevi | Android agent örneği |
|---|---|---|
| **Deneyim bilgi tabanı** | Geçmiş görevlerden çıkarılan, gerektiğinde başvurulacak bilgiyi saklar. | “Bu projede API hataları `ApiResult` ile modelleniyor.” |
| **Skill** | Belirli bir görev veya tool için izlenecek yöntemi tarif eder. | “Android endpoint’i eklerken önce mevcut network katmanını incele.” |
| **Sistem prompt’u** | Agent’ın genel davranışına yön veren, görevler boyunca geçerli üst düzey talimatları içerir. | “Değişiklik yapmadan önce ilgili mevcut yapıyı incele.” |
| **Program / harness** | Agent’ın çalışma ortamını, tool erişimini ve kodla uygulanan kısıtlarını yönetir. | Agent’ın hangi klasörlere yazabileceğini teknik olarak sınırlandırır. |

Burada metnin önerdiği ilke şu: **Her öğrenilen şeyi sistem prompt’una ekleme.** Önce deneyim olarak sakla. Aynı hata yeterince farklı ama benzer görevde tekrar ediyorsa ve hatayı önleyecek kural açıkça ifade edilebiliyorsa, onu uygun talimat katmanına taşımayı değerlendir.

## Somut örnek: Agent sürekli yanlış modülde test yazıyor

Bir Android projesinde agent’a farklı görevler verildiğini düşünelim:

```text
Görev 1: :feature:profile modülüne özellik ekle
Sonuç: Testi :app modülüne yazdı.

Görev 2: :feature:checkout modülündeki hatayı düzelt
Sonuç: Testi yine :app modülüne yazdı.

Görev 3: :feature:search modülüne davranış ekle
Sonuç: Testi yine :app modülüne yazdı.
```

Her trajectory incelendiğinde agent’ın ortak hatası görülüyor: **değişikliğin yapıldığı modül ile testin konulacağı modülü eşleştirmiyor.**

Başta bilgi tabanına şu gözlem kaydedilebilir:

> “İncelenen çok modüllü Android görevlerinde testler birkaç kez değişikliğin yapıldığı modül yerine `:app` modülüne yazıldı.”

Fakat hata tekrar ediyorsa, yalnızca kaydetmek yetmeyebilir. Agent’ın Android geliştirme skill’ine küçük bir talimat eklemek aday çözüm olur:

```diff
+ Test eklemeden önce değişikliğin yapıldığı modülü ve
+ o modüldeki mevcut testlerin konumunu kontrol et.
```

Bu kural Android geliştirme görevlerine özgü olduğu için örneğimizde **skill’e** koymak mantıklı. Aynı problem pek çok farklı tür görevde görülseydi daha genel bir talimat düşünülürdü.

Ama dikkat: Üç örnek burada yalnızca fikri anlatıyor; **üç tekrarın kendiliğinden yeterli olduğu evrensel bir eşik yok.** Hatanın etkisi, örneklerin çeşitliliği, karşı örnekler ve test sonuçları da önemli.

## Karpathy’nin “sistem prompt’u öğrenmesi” fikri

Temel fikir şu:

```text
Agent görev yapar
      ↓
Tekrarlanan bir hata gözlemlenir
      ↓
Hatayı önlemeye yönelik açık bir talimat önerilir
      ↓
Talimat denenir
      ↓
İşe yararsa kullanılır; zarar verirse geri alınır
```

Mesela:

> “Çok modüllü bir projede test yazmadan önce hedef modülün mevcut test düzenini incele.”

Bu, gelecekteki agent’a bırakılan kısa bir uyarı gibi düşünülebilir. **Modelin ağırlıkları değişmez; modele verilen talimat değiştirilir.**

DSPy, OPRO ve GEPA gibi yaklaşımlar da daha genel olarak talimat veya örnek adaylarını değerlendirerek iyileştirme arayışına örnektir. Paragrafın ayrımı şu: Bunlar geliştirme ortamında çok sayıda adayı topluca denemek için kullanılabilir; üretimde ise önerilen değişikliği küçük tutmak, kaynağını ve etkisini görmek ve gerekirse geri alabilmek önemlidir.

## Prompt yazmakla prompt’u geri bildirimle geliştirmek arasındaki fark

**Prompt engineering** sorusu:

> “Agent’a görevi ve kuralları anlaşılır biçimde nasıl anlatırım?”

**Sistem prompt’u öğrenmesi** sorusu:

> “Hangi somut hatalar bu kurala ihtiyaç olduğunu gösteriyor ve yeni kuralı eklediğimde sonuçlar gerçekten iyileşiyor mu?”

Yani ilkinde talimatı **tasarlıyorsun**; ikincisinde gerçek çalışmalardan gelen kanıta dayanarak onu **değiştiriyorsun**.

## “En küçük diff” neden önemli?

Diyelim mevcut skill 50 satır. Tekrarlanan bir hata için modelden 50 satırın tamamını yeniden yazmasını istemek yerine, ilgili yere yalnızca gerekli bir iki satırı ekliyorsun.

Böylece:

- **Hangi değişikliğin neden yapıldığı** anlaşılır.
- Önceden işe yarayan talimatlar yanlışlıkla değişmez.
- Sonuç kötüleşirse ilgili ekleme kolayca geri alınır.
- Değişiklik, onu destekleyen trajectory’lere bağlanabilir.

Örneğimizde değişiklik kaydı şöyle olabilir:

```text
Öneri: Testten önce hedef modülün test düzenini incele.
Kaynak: Çok modüllü Android görevleri 1, 2 ve 3.
Amaç: Testlerin yanlış modüle eklenmesini azaltmak.
Değişiklik: Android geliştirme skill'ine iki satır.
```

## İki ayrı test kümesi neden var?

Yeni talimatı eklemek yeterli değil; hem sorunu çözdüğünü hem de başka şeyleri bozmadığını görmelisin.

| Küme | İçeriği | Beklenti |
|---|---|---|
| **Sınır kümesi** | Hatanın ortaya çıktığı veya çıkabileceği görevler | Yeni talimatla yanlış modüle test yazma azalmalı. |
| **Koruma kümesi** | Agent’ın zaten başarılı olduğu farklı görevler | Bu görevlerde başarım anlamlı biçimde gerilememeli. |

Örneğin yeni kural, üç problemli Android görevinde iyi sonuç veriyor olabilir. Ama agent artık **her küçük metin değişikliğinde bile gereksiz test klasörü aramaya** başlıyorsa, talimat fazla geniş yazılmış olabilir.

Bu durumda kuralı daraltabilirsin:

> “Kod değişikliği için test eklerken, hedef modülün mevcut test düzenini incele.”

Sonra iki kümeyi tekrar çalıştırırsın. **Sınır kümesindeki başarı yeni kuralın yararını, koruma kümesindeki sonuçlar ise yan etkilerini gösterir.** Sonuçlar kabul edilebilir değilse öneri düzeltilir veya geri alınır.

## deneyimi modelin parametrelerine yazmak

bir agent deneyim kazandığında bu deneyim ne zaman dışarıdaki bir bilgi veya talimat olarak tutulmalı, ne zaman modelin eğitiminin bir parçası olmalı?

önce dört yöntemi aynı örnek üzerinden karşılaştıralım.

## aynı problemi dört farklı yerde çözmek

bir android kodlama agent'ın olduğunu düşün, farklı projelerde sürekli şu tür görevler alıyor:

"bu api yanıtındaki kullanıcı verisinin uygulamanın domain modeline dönüştür"

agent bazı görevlerde `null`gelebilen alanları dikkatsizce ele alıyor. traj'lar incelendiğinde kimi zaman !! kullandığı, kimi zaman da hata durumunu sessizce varsayılan bir değerle örttüğü görülüyor. bu problemi gidermek için dört farklı katmana müdahale edebilirsin:

### Yöntem A: Knowledge base’i güncellemek

Deneyimlerden koşullu bir bilgi çıkarırsın:

Markdown

```
## API yanıtlarında nullable alanları ele almaKullanıcı adı API sözleşmesine göre eksik gelebilir.Bu durumda mevcut domain modelindeki MissingName durumunu kullan.Kaynak: API sözleşmesi v2 ve ilgili mapping testleri.
```

Yeni bir görev geldiğinde agent bu bilgiyi bulur ve context’ine alır.

**Değişen şey:** Agent’ın görev sırasında ulaşabildiği bilgi.

**Değişmeyen şey:** Modelin ağırlıkları.

Buradaki bilgi belirli bir API sözleşmesine ait olduğu için dışarıda tutulması avantajlıdır: sözleşme değişince belgeyi güncelleyebilirsin.

---

### Yöntem B: Prompt veya skill güncellemek

Agent’a şu tür bir davranış ilkesi verirsin:

TEXT

```
Nullable bir API alanını domain modeline taşırken,önce alanın sözleşmesini ve mevcut hata modelini incele.Eksik veriyi sessizce varsayılan değere dönüştürme.
```

Artık yalnızca belirli bir alanla ilgili bilgi vermiyorsun; **nasıl karar verilmesi gerektiğini** söylüyorsun.

**Değişen şey:** Modele verilen davranış talimatı.

**Değişmeyen şey:** Modelin ağırlıkları.

---

### Yöntem C: Programı veya harness’i güncellemek

Projeye bir statik analiz kuralı veya doğrulama adımı eklersin:

TEXT

```
Değiştirilen production Kotlin dosyalarında `!!` kullanımıtespit edilirse kontrol başarısız olsun.
```

Bu örnekte kapsamı açıkça belirlenmiş bir kod kontrolü var. Agent talimatı unutsa bile kontrol çalıştırılırsa bu kullanımı yakalayabilir.

Ancak bu kontrolün de sınırı var: `!!` olmaması, null durumunun **doğru** ele alındığını tek başına kanıtlamaz. Agent, `?: ""` kullanarak da önemli bir hatayı gizlemiş olabilir. Bu nedenle sözleşme testleri gibi ek kontrollere ihtiyaç vardır.

**Değişen şey:** Agent’ın içinde çalıştığı program veya doğrulama mekanizması.

**Değişmeyen şey:** Modelin ağırlıkları.

---

### Yöntem D: Modelin parametrelerini güncellemek

Çok sayıda değerlendirilmiş örnekten eğitim verisi hazırlarsın. Model, bu örnekler üzerinden yeniden eğitilir veya ince ayarlanır.

Örneğin eğitim örnekleri şunu gösterebilir:

TEXT

```
Girdi:Nullable API alanı içeren bir Kotlin mapping göreviİyi çözüm:API sözleşmesini dikkate alan, eksik değeri uygun domaindurumuna dönüştüren ve ilgili testleri yazan çözüm
```

Eğitim sonrasında model, benzer problemleri çözerken bu tür davranışları **ağırlıklarına yansımış örüntüler sayesinde** daha sık sergileyebilir.

**Değişen şey:** Modelin parametreleri veya eğitim yöntemine bağlı olarak bunlara eklenen eğitilebilir parametreler.

Buradaki “yazmak” ifadesi mecazdır. Eğitim, bir kuralı sabit diskteki bir dosyaya satır ekler gibi belirli bir adrese kaydetmez. Eğitim sırasında modelin parametreleri, çok sayıda örnekte istenen davranışlara göre matematiksel olarak güncellenir.

> Dış bilgi veya prompt değişikliği, modele verilen girdiyi değiştirir.  
> Parametre eğitimi ise modelin bu girdileri işlerken kullandığı öğrenilmiş yapıyı değiştirir.

Bu ayrım önemli; çünkü güncelleme, doğrulama ve geri alma biçimleri de buna göre değişir.

## parametreler tam olarak neyi temsil ediyor?

bir modeli şu şekilde düşünebilirsin:

girdi + model parametreleri -> modelin ürettiği çıktı

örneğin agent'a şu istek geliyor: bu nullable api response'unu domain modeline dönüştür.

modelin üreteceği çözüm, isteğe, contextteki bilgilere ve model parametrelerine bağlıdır.

parametreler, modelin eğitim sırasında öğrendiği çok sayıdaki örüntünün dağılmış temsilidir. bunların içinde kotlin syntax, problem çözme örüntüleri, çeşitli yazım biçimleri veya tool kullanımıyla ilgili davranışlar bulunabilir.

Basitleştirilmiş olarak:

TEXT

```
Eğitim öncesi model
       +
Değerlendirilmiş eğitim örnekleri
       ↓
Eğitim algoritması parametreleri günceller
       ↓
Yeni model sürümü
```

Bu, belirli bir davranışın kesinlikle ortaya çıkacağı anlamına gelmez. Model eğitim sonrasında da hata yapabilir; farklı context’lerde beklenmedik davranabilir veya öğrendiği davranışı başka bir talimatla yanlış biçimde birleştirebilir.

### Android benzetmesi

Bunu Android’deki iki farklı güncelleme gibi düşünebilirsin:

- **Bilgi tabanını güncellemek:** Uygulamanın çalışma sırasında okuduğu yapılandırma veya veriyi değiştirmek.
- **Parametreleri güncellemek:** Uygulamanın karar veren bileşeninin yeni bir sürümünü eğitip dağıtmak.

Benzetmenin sınırı şu: Geleneksel programda bir kural genellikle kaynak kodda açıkça bulunabilir. Modelde ise eğitimden doğan davranışın karşılığı çoğu zaman tek bir okunabilir `if` satırı değildir.

```kotlin
// Açık ve denetlenebilir program kuralı
if (!userConfirmed) {
    rejectPayment()
}
```

Buna karşılık modeli “onaysız ödeme yapmamalısın” örnekleriyle eğitmek, tek başına her durumda aynı kesinliği sağlamaz. Bu yüzden kitap, **katı iş kurallarının yalnızca modelin öğrendiği davranışa bırakılmaması** gerektiğini vurguluyor.

## 3. Trajectory eğitim verisine nasıl dönüşür?

Önceki konuşmalarımızda trajectory’yi bir görev yürütmesinin kaydı olarak ele almıştık.

Örneğin:

TEXT

```
Kullanıcı isteği
    ↓
Agent mevcut kodu inceler
    ↓
API sözleşmesini okur
    ↓
Mapper yazar
    ↓
Test çalıştırır
    ↓
Hata alır
    ↓
Kodu düzeltir
    ↓
Test geçer
    ↓
Sonucu bildirir
```

Bu kayıt, olduğu gibi otomatik olarak kaliteli eğitim verisi değildir.

Çünkü içinde şunlar bulunabilir:

- Yanlış ilk denemeler.
- Eksik veya yanıltıcı tool çıktıları.
- Kullanıcıya ait hassas bilgiler.
- Gereksiz tekrarlar.
- Şans eseri doğru sonuç veren adımlar.
- Testin geçmesine rağmen yanlış olan bir çözüm.
- Görevi başarıyla bitirse bile izlenmemesi gereken bir yöntem.

Dolayısıyla arada bir **veri hazırlama ve değerlendirme süreci** gerekir:

TEXT

```
Ham trajectory'ler
       ↓
Görev sonucunu ve süreci değerlendir
       ↓
Hassas bilgileri temizle
       ↓
Hatalı veya güvenilmez örnekleri ayıkla
       ↓
Eğitim amacına uygun örnekler hazırla
       ↓
Eğitim
       ↓
Bağımsız testler ve regresyon kontrolleri
```

Buradaki kritik nokta şu:

> Başarılı sonuçlanan bir trajectory, içindeki her kararın doğru olduğunu kanıtlamaz.

Örneğin agent testi beşinci denemede geçirmiş olabilir. Test geçene kadar yaptığı dört hatalı değişikliği olduğu gibi “ideal davranış” diye öğretmek istemeyebilirsin. Diğer taraftan, o hataların nasıl teşhis edilip düzeltildiği eğitim hedefi açısından değerli olabilir.

**Hangi adımın tutulacağı, ne öğretmek istediğine bağlıdır.**

---

## 4. Aynı deneyim, farklı eğitim türlerinde nasıl kullanılır?

Kitap burada özellikle **SFT, tercih verisi ve RL** örneklerini veriyor. Bunların eğitim algoritmalarına derinlemesine girmeden, trajectory ile ilişkilerini ayıralım.

### 4.1. SFT: İyi bir gösterimden öğrenmek

**Supervised Fine-Tuning — SFT**, modele örnek girdi ve hedef çıktı göstererek belirli davranışları öğretme yaklaşımıdır.

Temsili bir örnek:

TEXT

```
Görev:
API'den gelen nullable kullanıcı adını domain modeline dönüştür.

İstenen davranış:
Önce API sözleşmesini incele.
Eksik kullanıcı adını uygulamanın mevcut hata modeliyle temsil et.
İlgili başarı ve hata durumları için test yaz.
```

Eğitim verisinde gösterilen hedef, yalnızca son cevap olmak zorunda değildir; eğitim tasarımına göre uygun tool çağrıları veya etkileşim adımları da temsil edilebilir.

Burada öğretmek istediğin şey:

> “Bu tür bir durumda iyi bir çözüm örneği nasıl görünür?”

Ancak gösterimin kaliteli olması gerekir. Agent yanlış bir çözümü kendinden emin şekilde üretmişse ve bu çözüm incelenmeden eğitim verisine girerse, sistem hatalı davranışı da pekiştirebilir.

---

### 4.2. Tercih verisi: İki çözüm arasında hangisi daha iyi?

Bir görev için iki aday çözüm düşün:

**Çözüm A:**

KOTLIN

```
fun UserDto.toDomain(): User =
    User(
        name = name ?: ""
    )
```

**Çözüm B:**

KOTLIN

```
fun UserDto.toDomain(): UserResult =
    when (val validName = name) {
        null -> UserResult.MissingName
        else -> UserResult.Success(User(name = validName))
    }
```

Bu örnekte varsayımımız şu: **API sözleşmesi, isim eksikliğinin hata olarak ele alınmasını gerektiriyor.**

Bu koşul altında bir değerlendirici B’yi tercih edebilir; çünkü A, eksik ismi boş string’e çevirerek sözleşmedeki problemi gizliyor.

Tercih örneği kabaca şöyle temsil edilebilir:

TEXT

```
Görev ve bağlam: ...
Tercih edilen çözüm: B
Tercih edilmeyen çözüm: A
Tercih nedeni: Eksik veri sessizce gizlenmemeli.
```

Burada önemli nüans: B her projede otomatik olarak doğru değildir. Bazı sistemlerde boş veya varsayılan değer, ürün gereksiniminin bilinçli bir parçası olabilir. Değerlendirme **görev context’ine ve sözleşmeye** dayanmalıdır.

Tercih verisi, uygun bir tercih eğitimi yöntemiyle modelin hangi tür çıktılara yönelmesi gerektiği hakkında sinyal sağlayabilir.

---

### 4.3. RL: Ortamdan gelen ödülle öğrenmek

**Reinforcement Learning — RL** tarafında agent, bir ortamda eylemler gerçekleştirir ve bu eylemlere bağlı bir ödül veya puan sinyali alır.

Basit bir kodlama ortamı düşün:

TEXT

```
Agent kod yazar
    ↓
Testler çalıştırılır
    ↓
Test sonuçlarına göre ödül hesaplanır
    ↓
Eğitim algoritması bu sinyali kullanır
```

Fakat “test geçti” sinyaline aşırı güvenmek sorun yaratabilir.

Örneğin agent:

- Gerçek hatayı çözmek yerine testi değiştirebilir.
- Testin kapsamadığı bir hatayı bırakabilir.
- Sadece mevcut örneğe özel bir çözüm yazabilir.
- Kuralları ihlal ederek puanı yükseltebilir.

Bu yüzden ödül tasarımı yalnızca “puan yüksek olsun” sorusuna değil, **puan hangi davranışı teşvik ediyor?** sorusuna da cevap vermelidir.

Örneğin sistem şu sinyalleri birlikte değerlendirebilir:

TEXT

```
+ İlgili testler geçiyor mu?
+ Önceden geçen testler hâlâ geçiyor mu?
+ Değişiklik istenen davranışı gerçekten sağlıyor mu?
+ Agent yasaklı dosyaları değiştirdi mi?
+ Çözüm farklı girdilerde çalışıyor mu?
```

Bu kontroller bile kusursuz bir ödül anlamına gelmez; ama tek bir yüzeysel ölçütten daha açıklayıcıdır.

### Üçünü yan yana koyalım

|Yaklaşım|Temel sinyal|Temsili soru|
|---|---|---|
|**SFT**|İyi davranış gösterimi|“İyi bir çözüm nasıl görünüyor?”|
|**Tercih eğitimi**|Adaylar arasındaki tercih|“Bu iki çözümden hangisi, neden daha iyi?”|
|**RL**|Ortamdan veya değerlendiriciden gelen ödül|“Agent’ın eylemleri nasıl bir sonuç üretti?”|

Bunlar basitleştirilmiş ayrımlardır. Gerçek sistemlerde yöntemlerin veri biçimleri ve eğitim aşamaları daha karmaşık olabilir; ayrıca aynı agent geliştirme sürecinde birden fazla yöntem birlikte kullanılabilir.

## 5. Her deneyimi neden parametrelere yazmıyoruz?

Şöyle düşünelim:

> “Bu projenin API adresi `https://api.example.com/v2`.”

Bunu modele eğitimle öğretmek genellikle iyi bir ilk tercih değildir. Çünkü adres değişebilir. Ayrıca bilgi, tek bir projeye özgüdür.

Bunun yerine agent’ın güncel yapılandırmayı veya proje dokümanını okuması daha uygundur.

Şimdi başka bir bilgi:

> “İki teknik çözümün avantaj ve dezavantajlarını açıkça karşılaştır.”

Bu, belirli bir URL gibi tek bir olgu değildir. Farklı görevlerde karşına çıkabilecek genel bir davranış örüntüsüdür. Talimatla desteklenebilir; yeterli ve uygun eğitim verisiyle modelin bu davranışı üretmesi de iyileştirilmeye çalışılabilir.

Kitabın önerdiği seçim mantığı kabaca şöyle:

|İhtiyaç|Öncelikle düşünülebilecek taşıyıcı|Neden?|
|---|---|---|
|Güncel API adresi veya sürüm bilgisi|Bilgi tabanı / yapılandırma|Değişir; kaynağını güncellemek gerekir.|
|Belirli bir iş akışında izlenecek yöntem|Skill / prompt|Dil ile açıkça ifade edilebilir ve sürümlenebilir.|
|Onay olmadan ödeme yapılmaması|Program / harness|Kritik kuralın teknik olarak uygulanması gerekir.|
|Çok sayıda context’e yayılan, örneklerle öğretilmesi gereken davranış|Parametre eğitimi değerlendirilebilir|Davranış tek bir kısa kurala sığmayabilir; ancak test edilmesi gerekir.|

Burada “genel davranışsa mutlaka eğit” gibi mekanik bir kural yok. Eğitim maliyeti, eldeki veri kalitesi, davranışın risk seviyesi, güncelleme sıklığı ve hangi yöntemin ölçülebilir biçimde daha iyi sonuç verdiği birlikte değerlendirilir.

Bir diğer önemli nokta: **Bu taşıyıcılar birbirini dışlamaz.**

Örneğin bir ödeme agent’ında:

TEXT

```
Model parametreleri:
Kullanıcının isteğini ve doğal dildeki nüansları anlamaya yardım eder.

Bilgi tabanı:
Güncel ürün ve ödeme bilgilerini sağlar.

Skill:
Ödeme öncesi gerekli bilgileri nasıl kontrol edeceğini tarif eder.

Harness:
Gerekli onay olmadan ödeme aracının çalışmasını engeller.
```

Burada her katman farklı bir rol üstlenir.

---

## 6. Eğitimden önce ve sonra neleri kontrol etmeliyiz?

Kitap, üretim trajectory’lerinin doğrudan eğitime alınmaması gerektiğini söylüyor. Özellikle üç hazırlık önemli:

### Hassas bilgileri temizlemek

Gerçek görev kayıtlarında kişisel veriler, erişim bilgileri veya kullanıcıya ait özel içerikler bulunabilir. Eğitim verisine alınmadan önce bu bilgilerin uygun biçimde ayıklanması ve verinin kullanım yetkisinin değerlendirilmesi gerekir.

### Hatalı örnekleri filtrelemek

Bir trajectory’nin sonundaki “başarılı” etiketi yeterli olmayabilir. Çözümün gerçekten doğru olup olmadığı, sürecin kurallara uyup uymadığı ve değerlendiricinin güvenilirliği incelenmelidir.

### Bağımsız evaluation verisi ayırmak

Eğitim sırasında kullanılan örneklerle modelin geliştiğini ölçmek yanıltıcı olabilir. Model o örnekleri öğrenmiş görünür; fakat yeni görevlerde aynı beceriyi gösteremeyebilir.

Bu nedenle eğitimde kullanılmayan görevler ayrılır:

TEXT

```
Eğitim kümesi:
Modelin öğrenmesi için kullanılır.

Bağımsız değerlendirme kümesi:
Modelin görmediği örneklerde ne yaptığını ölçer.
```

Pratikte geliştirme sırasında karar vermek için ayrı bir geliştirme kümesi ve sonunda daha az dokunulan bir test kümesi kullanmak da yararlıdır. Sürekli aynı test kümesine bakıp modeli ona göre değiştirirsen, zamanla bu kümeye de dolaylı biçimde uyum sağlayabilirsin.

### Eğitimden sonra eski yetenekleri de ölçmek

Diyelim yeni model Kotlin mapping görevlerinde gelişti. Bu iyi bir işaret; fakat yeterli değil.

Şunları da sormalıyız:

- Önceden iyi yaptığı diğer Kotlin görevlerinde geriledi mi?
- Java veya genel kodlama görevleri etkilendi mi?
- Güvenlik sınırlarına uyumu değişti mi?
- Doğru durumda kullanıcıdan açıklama istemeyi hâlâ biliyor mu?
- Yeni davranışı yalnızca eğitim örneklerine benzeyen durumlarda mı gösteriyor?

Yeni bir şey öğretirken eski yeteneklerin zarar görmesi, genel olarak **catastrophic forgetting** başlığı altında incelenen bir problemdir. Her fine-tuning çalışmasının mutlaka ağır unutmaya yol açacağı anlamına gelmez; fakat ölçülmesi gereken bir risktir.

---

## 7. Şimdiye kadar konuştuğumuz konularla bağlantısı

Önceki parçalarda deneyimin bilgiye veya talimata dönüşmesini konuşmuştuk. Şimdi aynı hattı parametrelere kadar uzatabiliriz:

TEXT

```
Agent görevleri yürütür        ↓Trajectory ve ortam sonuçları kaydedilir        ↓Çalışmalar değerlendirilir        ↓Tekrarlanan örüntüler ve güvenilir örnekler belirlenir        ↓Sorunun hangi katmanda çözülmesi gerektiği seçilir        ↓Bilgi / talimat / program / eğitim verisi için aday hazırlanır        ↓Aday bağımsız görevlerde ve koruma testlerinde sınanır        ↓Uygunsa yeni sürüm kullanıma alınır
```

Dikkat edersen, **parametre eğitimi bu zincirin otomatik son durağı değil.** Bir deneyimin zamanla mutlaka bilgi tabanından skill’e, oradan programa ve en sonunda modele taşınması gerekmiyor.

Hangi katmanın uygun olduğu, çözmeye çalıştığın şeyin niteliğine bağlı:

- Değişken bir olgu mu?
- Açıkça yazılabilen bir davranış ilkesi mi?
- Kesin uygulanması gereken bir kısıt mı?
- Çok sayıda örnek gerektiren karmaşık bir yetenek mi?

Ve hangi katmanı seçersen seç, son soru aynı:

> **Yeni sürüm, daha önce görülmemiş görevlerde gerçekten daha iyi çalışıyor mu; bunu yaparken mevcut yetenekleri ve güvenlik sınırlarını koruyor mu?**

## Kısa bir düşünme örneği

Bir agent’ın şu üç problem yaşadığını varsayalım:

1. Kullanımdan kaldırılmış bir API sürümünü öneriyor.
2. Kullanıcının tonuna uygun, doğal Türkçe yanıt vermekte zorlanıyor.
3. Kullanıcı onay vermeden ödeme aracını çağırabiliyor.

Bence bunlar için ilk düşünülmesi gereken yerler sırasıyla:

1. **Güncel kaynak veya bilgi tabanı:** API sürümü değişebilen bir olgu.
2. **Prompt ve gerekirse eğitim:** Üslup davranışı örneklerle iyileştirilebilir; hangi yöntemin yararlı olduğu değerlendirilir.
3. **Harness düzeyinde onay kontrolü:** Ödeme yetkisi kesin biçimde sınırlandırılmalı.

Bu üçü üzerinde düşün: **İkinci problemde birkaç örnek içeren bir prompt yeterli sonuç veriyorsa, yine de modeli eğitmek için hangi nedenlerin olması gerekirdi?**

Sonraki parçada buradan devam edip bir seviye yukarı çıkacağız: **Agent’ın bilgisini veya prompt’unu güncellemekle, bu güncellemeleri nasıl üreteceğini ve yöneteceğini geliştirmek arasındaki fark.**

## Güncelleme yöntemini geliştirmek

Bir prompt’u değiştirmek ile prompt’ların nasıl üretildiğini ve yönetildiğini geliştirmek arasındaki fark; ACE, MCE, AFlow ve Meta-Harness.

## 1. İçeriği değiştirmek ile yöntemi değiştirmek

Diyelim ki bir kodlama agent’ı, önceki görevlerden çıkardığı dersleri bir bilgi dosyasında tutuyor.

Bu dosyada şu kural var:

> “Test yazmadan önce projenin test altyapısını incele.”

Agent çok modüllü bir projede hata yapınca bu kurala küçük bir ekleme öneriyorsun:

> “Test yazmadan önce **ilgili modülün** test altyapısını incele.”

Burada **içeriği** güncelledin: Bir dersin kendisi değişti.

Şimdi farklı bir problem düşün. Agent’ın bilgi tabanında doğru ders var, ama agent ilgili göreve geldiğinde onu bulamıyor. Bu kez dersin cümlesini değiştirmek sorunu çözmez. Bunun yerine:

- Hangi bilginin aranacağını,
- Arama sonuçlarının nasıl sıralanacağını,
- Hangilerinin context’e alınacağını

belirleyen yöntemi değiştirmen gerekir.

TEXT

```
İçerik güncellemesi:
“Agent hangi bilgiye sahip?”

Yöntem güncellemesi:
“Agent doğru bilgiyi nasıl buluyor ve kullanıma hazırlıyor?”
```

İkisi de önemli; ama çözmeye çalıştıkları problem farklı. Kitabın bu kısmında anlattığı araştırmalar, bu fikri farklı ölçeklere taşıyor.

### ACE: Context’i sürekli baştan yazmak yerine düzenlemek

Agent’ın her görevden sonra uzun bir bilgi metnini baştan yazdığını düşün. İlk seferde önemli bir istisnayı koruyabilir; ama her yeniden yazmada bir ayrıntı daha kaybolabilir:

TEXT

```
İlk sürüm:
“Mevcut test düzenini incele.
Çok modüllü projede hedef modülün ayarlarını ayrıca kontrol et.”

Birkaç yeniden yazma sonra:
“Testlerden önce projeyi incele.”
```

Son cümle kısa ama kritik ayrıntıyı kaybetmiş.

Kitabın aktardığı **ACE yaklaşımında**, context tek ve sürekli yeniden yazılan bir metin yerine, ayrı ayrı tanımlanabilen girdiler olarak ele alınıyor. Sistem yeni bilgi için yerel güncellemeler öneriyor; bunlar birleştiriliyor ve tekrarlar ayıklanıyor.

Yani fikir şu:

> Bilgiyi her seferinde yeniden anlatma; ilgili parçayı değiştir ve diğer yararlı ayrıntıları koru.

Bu, önceki konuşmamızdaki **en küçük diff** fikrine benziyor.

### MCE: Context’in kendisini değil, yönetim biçimini geliştirmek

ACE örneğinde context’in içindeki bilgiyi düzenliyorduk. Bir adım daha ileri gidince soru değişiyor:

> “Bilgiye ne yazalım?” yerine, “Bilgiyi nasıl arayalım, seçelim ve düzenleyelim?”

Kitabın anlattığı **MCE** yaklaşımı bunu iç ve dış döngü şeklinde düşünür:

- **İç döngü:** Mevcut yöntemle göreve uygun context’i hazırlar.
- **Dış döngü:** Farklı context hazırlama yöntemlerini karşılaştırır ve daha yararlı olan yöntemi seçmeye çalışır.

Örneğin iç döngü, bir Android görevi için bilgi tabanından üç not seçer. Dış döngü ise “anahtar kelimeyle arama mı, anlamsal arama mı, yoksa ikisinin birleşimi mi bu görevlerde daha iyi sonuç veriyor?” sorusunu değerlendirir.

Burada küçük bir ayrım var: Retrieval kuralını elle değiştirmek zaten yöntemi değiştirmektir. Farklı yöntemleri **deney sonuçlarına göre sistematik biçimde karşılaştırıp seçmek** ise yöntemi geliştirme sürecinin kendisidir.

### AFlow ve Meta-Harness: Aynı fikri daha geniş sisteme uygulamak

Aynı mantık yalnızca bilgi getirmeye uygulanmaz.

- **AFlow:** Agent’ın iş akışını —örneğin hangi adımların hangi sırayla çalışacağını— arama ve değerlendirme konusu yapar.
- **Meta-Harness:** Agent’ın çevresindeki daha geniş sistemi; örneğin bilginin nasıl saklanıp getirileceğini belirleyen harness kodunu iyileştirme konusu yapar.

Ölçek giderek büyüyor:

TEXT

```
Tek bir kuralı değiştir
        ↓
Bilginin nasıl seçildiğini değiştir
        ↓
İş akışını değiştir
        ↓
Agent’ı çalıştıran harness’i değiştir
```

Bunlar **bilgi, prompt, program ve parametrelerin yanına eklenen yeni taşıyıcılar değil**. Kitap bunları, sistemin hangi büyüklükte bir parçasını optimize ettiğini gösteren farklı ölçekler olarak anlatıyor.

Ve önemli bir tercih ekliyor: Sorun küçük ve yerel bir değişiklikle çözülebiliyorsa, önce onu dene. Çünkü büyük değişikliklerin etkisini anlamak ve gerektiğinde geri almak daha zor.

## 7.4 uzun süre çalışabilen sürekli evrim döngüsü kurmak

dört güncelleme biçimi ancak otonom döngüye girdiğinde tek seferlik bir optimizasyon olmaktan çıkıp sürekli evrime dönüşür. şekil 9-5, üretim sistemlerinde daha sağlam olan çift döngülü yapıyı gösteriyor. çevrimiçi yürütme döngüsü yalnızca görevi tamamlar ve kanıtı kaydeder, resmi agent'ı doğrudan değiştirmez; çevrimdışı evrim döngüsü ise traj'leri bir araya toplar, kök nedene tanı koyar, aday değişiklikleri üretir ve ancak doğrulama eşiklerini geçtikten sonra yeni sürümü yayımlar. iki döngü, sürümlenmiş deneyim deposu ve değerlendirme kümeleri aracılığıyla birbirine bağlanır.

![Görsel](../gorseller/Pasted%20image%2020261101195547.png)

voyager görece eksiksiz bir sürekli evrim döngüsü sergiliyor. minecraftta mevcut yeteneklerine göre yeni bir hedef seçiyor, ortamdan gelen feedbackle programı yineliyor, doğrulamayı geçen kodu beceri kütüphanesine kaydediyor, sonra eski becerileri birleştirerek daha zor görevleri çözüyor. otomatik müfredat, yürütülebilir skilller ve ortam doğrulaması -- bunların biri bile eksik olamaz: müfredat olmadan yalnızca skill lib varsa, agent bir sonraki adımda ne yapacağını bilemez. ortam doğrulaması olmadan yalnızca kendi kendine reflection varsa, beceri kütüphanesi hata biriktirir; kalıcılık olmadan yalnızca keşif varsa, her görev yine sıfırdan başlar. gerçek dünyadaki agent'ların bilgisi, prompt'u tool’ları ve parametreleri daha karmaşık olsa da temel öğrenme süreci benzerdir.

voyager üç kenetli mekanizmadan oluşur. otomatik müfredat üreticisi, mevcut envanter, ortam ve skillden oluşan uygun zorlukta bir sonraki hedefi önererek keşfi rastgele gezinme olmaktan çıkarır. skill lib, başarılı programları geri çağrılabilir ve birleştirilebilir kod olarak saklar, gelişmiş toplama becerisi temel hareket ve üretim becerilerini çağırabilir. yinelemeli prompting mekanizması, ortam gözlemlerini, yürütme hatalarını ve öz-doğrulama sonuçlarını görev gerçekten geçene dek sonraki kod üretim turuna taşır.

keşif döngüsü: hipotez, deney, değerlendirme, geri bildirim. voyager gibi agent öz-evrim sistemleri, yüzyıllarda olgunlaşan bilimsel yöntem olan bu döngüyü izler. jeff dean ve arkadaşlarının kısa süre önce kurduğu discovery loop, deney önerme, uygulama, değerlendirme, sonucu alma ve sonraki tura besleme sürecini otomatize etmeyi önerir. bu agent öz-evriminin bilime uyarlanmasıdır. 

sürekli agent evriminde sık karıştırılan iki yetenek ayrılmalıdır. harness updating, traj'lerden değerli kalıcı değişiklik üretir; harness benefit, görev agent'ının sonraki çalışmada bu değişikliği bulup etkinleştirme ve doğru kullanma yeteneğidir. bir skill kusursuz yazılmış olsa da zayıf model onu doğru durumda yüklemeyebilir veya uzun süre izleyemeyebilir; sonuç evrim yok gibi görünür. bu yüzden e2e puan tek başına güncelleyiciyi teşhis edemez. lin ve arkadaşlarının model değiştirme deneyleri iki yeteneğin temel model kabiliyetiyle farklı ilişkileri olduğunu gösterir.

![Görsel](../gorseller/Pasted%20image%2020261101200123.png)

değerlendirme, öğrenme bittikten sonra girilen bir sınav değil, kendi kendine evrim sürecinin vazgeçilmez bir parçasıdır. uzun vadeli değerlendirme en az beş tür sonucu aynı anda gözlemelidir:
* gerileme regression, yani yeni deneyimin mevcut diğer deneyimlerle çelişip çelişmediği ve daha önce geçebilen vakalarda gerileme olup olmadığı
* genelleme yeteneği, yani yeni deneyimin test kümesinin henüz kapsamadığı senaryolarda sağladığı iyileşme
* token verimliliği, yani görevi tamamlamak için harcanan token maliyeti
* güvenlik, yani kural, gizlilik ve reddetme sınırlarının evrimle birlikte kayıp kaymadığı
* uzun vadeli mühendislik kalitesi, yani bakım karmaşıklığının, mimari tutarlılığın, sahiplik sınırlarının geriye dönük uyumluluğun ve gelecekteki taşıma ile hata ayıklama yükünün kötüleşip kötüleşmediği

Yalnızca mevcut başarısızlık vakasının sorununu çözüp diğer mevcut vakalarda veya yeni alanlarda gerilemeye yol açmak, başarılı bir sürekli evrim değildir.


## 7.5 Doğrulanabilir döngünün sınırı: Tamamlanma her zaman ilerleme değildir

Kodlama gibi görevlerde test sonucu veya ortam durumu hızlı geri bildirim verebilir. Bilimsel araştırma, strateji ve ürün tasarımında ise doğru cevap tek olmayabilir; anlamlı sonuçlar geç ortaya çıkar ve önemli hedefleri tek bir puana indirgemek zordur. Bu nedenle bir agent bütün adımları tamamlayıp rapor üretse bile asıl hedefe katkısı belirsiz kalabilir.

Metin, otomatik araştırmalarda üç riski ayırıyor:

- **Uygulama kayması:** Zorlaşan özgün hipotezden uzaklaşıp daha tanıdık ama farklı bir probleme çözüm üretmek.
- **Epistemik aşırı iyimserlik:** Gürültülü veya yetersiz sonucu buluş gibi yorumlamak; başarısızlıkları göz ardı etmek.
- **Yargı eksikliği:** Önemli baseline'ı, araştırılacak aykırı sonucu veya vazgeçme noktasını doğru seçememek.

Bu nedenle değerlendirme yalnızca düzgün görünen son rapora dayanmaz:

1. **İddiaları kanıtlara bağla:** Sayıların, yöntemlerin ve sonuçların kaynağı ayrı izlenir. İzlenebilirlik, araştırma sorusunun değerli olduğunu tek başına göstermez.
2. **Olumsuz sonuçları koru:** Başarısız deneyler ve durma nedenleri de erişilebilir olsun; aynı çıkmaz tekrar tekrar denenmesin.
3. **Arama çeşitliliğini koru:** Yalnızca mevcut puanı en yüksek aday değil, farklı mekanizmalara dayanan umut verici alternatifler de saklansın.
4. **İnsanı üst düzey kararlara dahil et:** Problem seçimi, ölçütlerin uygunluğu, sıra dışı bulguların yorumlanması ve durma kararı da insan değerlendirmesi gerektirebilir.

**Android örneği:** Agent, başlangıç süresini iyileştirmek için benchmark'ı geçen bir değişiklik yapıyor. Ancak benchmark gerçek kullanıcıların cihazlarını temsil etmiyor veya uygulamanın sürdürülebilirliğini bozuyor olabilir. `Test geçti` kanıttır; **ürünün gerçekten iyileştiği** sonucunu tek başına garanti etmez.

## 7.6 Sürekli evrimin güvenlik sınırları

Bir agent'ın yaşadıklarından kalıcı bilgi veya tool üretmesi, tek bir hatanın sonraki görevlere yayılmasına neden olabilir. Metindeki üç sınır:

- **Kanıt ve talimatı ayır:** Web sayfası ve tool çıktısı güvenilmeyen veridir. Kaynağı ve içeriği doğrulanmadan kalıcı Skill veya politika hâline getirilmemelidir. LLM ile özetlemek tek başına prompt injection'ı temizlediğini garanti etmez; ayrıca denetim ve güvenilir kontrol gerekir.
- **Aday ve resmî yetenekleri ayır:** Yeni Prompt, Skill, kod veya bağımlılık önce aday olarak değerlendirilir; uygun güvenlik ve regresyon kontrolleri geçilmeden gerçek kullanıma alınmaz.
- **Doğrulama sistemini koru:** Değişikliği üreten agent kendi onay mekanizmasını, saklı testlerini veya yayımlama eşiklerini serbestçe değiştirememelidir. Aksi takdirde testi silerek ya da eşiği düşürerek başarısızlığı başarı gibi gösterebilir.

Özet akış:

```text
Ham deneyim → Aday değişiklik → Bağımsız değerlendirme
            → Onay / ret → Sürümlü yayın veya geri alma
```

Örnek: Kodlama agent'ı Android projesine yeni bir Skill önerirse, Skill'in içeriği, dayandığı kanıt ve etkilediği görevler gözden geçirilir; test sonuçları önceki sürümle karşılaştırılır. Yetki ve onay kontrollerini aynı değişiklikle gevşetmesine izin verilmez.

## 7.7 Uyku öğrenmesi: Bütünleştirme ve budama

**Uyku öğrenmesi**, görev dışındaki çevrimdışı düzenleme için kullanılan bir benzetmedir; işlemin gece yapılması gerekmez. Çevrimiçi agent görevi yürütür ve kanıtları kaydeder. Ayrı bir süreç biriken deneyimleri karşılaştırır, aday iyileştirmeler üretir ve bunları sınar. Böylece tek bir tesadüfi başarı veya zararlı girdi, kalıcı davranışı anında değiştirmez.

Metindeki beş adım:

1. **Tetikleme:** Zaman, kayıt sayısı veya hata eşiğine göre çalışmayı başlat; o sıradaki öncelikli görevleri dikkate al.
2. **Yönelme:** Mevcut bilgi, Skill, sürüm ve değiştirilemez sınırları oku.
3. **Bütünleştirme:** Yeni trajectory'leri karşılaştır, tekrarları birleştir, çelişkileri ve geçerlilik koşullarını belirle; mümkünse küçük aday yamalar üret.
4. **Doğrulama:** Adayları yeni görevler, saklı testler ve güvenlik örnekleri üzerinde sınayarak riskli değişiklikleri uygun onaya sun.
5. **Budama ve indeksleme:** Eski veya çürütülmüş bilgileri uygun şekilde güncelle, arşivle veya kaldır; kaynak ve geri alma bilgisini koru.

Metin Claude Code'un otomatik belleğini sınırlı bir indeks ve ihtiyaç hâlinde okunan ayrıntı dosyaları açısından; Hermes'i ise geçmiş arama, Skill düzenleme ve kürasyon açısından örnek veriyor. Bunların somut özellikleri sürüme bağlıdır; Claude Code'un oturum içi otomatik belleğini doğrudan zamanlanmış bir gece öğrenme işiyle eşitlememek gerekir.

### Neden unutma ve budama da gerekli?

Sürekli ekleme yapmak zamanla yinelenen Skill'lere, birbiriyle çelişen talimatlara ve güncelliğini yitirmiş tool bilgilerine yol açabilir. Bu nedenle kaynak ve sürümleri koruyarak tekrarları birleştirmek, genel talimatları sade tutmak, eski yetenekleri yeniden doğrulamak ve yanlış bilgiyi düzeltmek gerekir.

Metinde LoRA'nın özgün temel modelden yeniden eğitilmesi bir bakım yaklaşımı olarak anılıyor. Bu, **her durumda zorunlu veya tek doğru yöntem değildir**; seçim eğitim hedefi, veri, maliyet ve bağımsız değerlendirme sonuçlarına bağlıdır.

### Kontrol soruları

1. Bir agent'ın araştırma raporunu tamamlaması neden gerçek bilimsel ilerlemeyi kanıtlamaz?
2. Başarısız deneyleri arama ve öğrenme sisteminde tutmak neyi değiştirir?
3. Doğrulayıcıyı güncelleme yapan agent'tan ayırmazsak nasıl bir yanlış başarı ortaya çıkabilir?
4. Uyku öğrenmesinde ham trajectory'leri saklamak ile bunlardan Skill üretmek neden ayrı aşamalardır?

## Bölüm Özeti

Sürekli evrim, Agent’ın en önemli yeteneklerinden biri hâline geliyor; ama bugünün modelleri güvenilir bir sürekli öğrenmeyi kendi başlarına gerçekleştiremiyor. Çıkarım anındaki context uyarlaması kendiliğin- den kalıcılaşmıyor; doğrulanmamış çevrimiçi parametre güncellemesi ise gürültüyü, saldırıları ve yetenek kay- masını büyütüyor. Bu nedenle bu aşamada daha uygulanabilir yol, modelin çevresinde doğrulanabilir bir öğrenme sistemi kurmaktır. 

Kitabın bütünsel yapısı açısından bu bölüm, Bölüm 1’deki keşif döngüsünün deney ve geri bildirim kesitini kurar: öneri zaten vardır, soru artık gerçek gözleme dayanan tek bir deneyin sistemi gerçekten iyileştirip iyileştirmediğini nasıl söyleyeceği ve sonucun bir sonraki tura nasıl taşınacağıdır. 

Agent, öğrenme sinyalini ortamla etkileşiminden ve değerlendirmelerden alır; sonra yeteneğin temsil niteliğine göre bilgiyi, Prompt’u, Skill’i, programı veya model parametrelerini günceller. Sistem, bu artifact’ ları yöneten ve üreten yöntemleri de bir adım öteye taşıyıp optimize edebilir; ama öncelikle nedeni bulunabilir, doğrulanabilir ve geri alınabilir yerel değişiklikler tercih edilmelidir. 

Geçmiş yalnızca durağan deneyime damıtılmakla kalmaz; destek alanı içinde, keşif stratejilerini düşük maliyetle eleyen bir yeniden oynatma ortamına da dönüştürülebilir. Böylece sürekli evrim, yalnızca keşfin ürettiği bilgi, talimat ve programlar üzerinde değil,“keşfin nasıl örgütlendiği”üzerinde de etkili olabilir. 

Sürekli evrim, çevrimiçi yürütmeyi çevrimdışı öğrenmeden ayırmayı gerektirir: çevrimiçi tarafta kanıt kaydedilir, çevrimdışı tarafta aday güncellemeler üretilip doğrulanır, sonra kademeli olarak yayımlanır, düzen- lenir veya geri alınır. Bu döngü, sonucu otomatik olarak doğrulanabilen görevlerde en güvenilir biçimde çalışır; hedefi muğlak, geri bildirimi gecikmeli açık uçlu görevlerde ise problemin tanımlanmasına ve değerlendirme ölçütlerinin belirlenmesine insanın katılması gerekir.