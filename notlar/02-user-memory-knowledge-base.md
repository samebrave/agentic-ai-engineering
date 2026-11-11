# User memory ve knowledge base

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter3/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

## 1. user memory mimarisi ve depolama formatları

### 1.1 temel ikilik: user memory ve knowledge base

kalıcı bilgi sistemleri iki farklı ölçekte ele alınır: 

**user memory**: bireysel ölçek. etkileşimler boyunca tek bir kullanıcının tercihlerini, alışkanlıklarını ve geçmişini öğrenir. agent’ı kişiselleştirilmiş bir asistan yapar.

**knowledge base**: topluluk ölçeği. tüm kullanıcılarca paylaşılan şirket içi prosedürleri veya teknik dokümantasyonunu içerir. agent’ı **alan uzmanı** yapar. her iki yapı da vektör arama ve bilgi sıkıştırma gibi ortak teknolojileri paylaşır ve çelişkili veya güncelliğini yitirmiş bilgi gibi benzer sorunlarla karşılaşır.

---

### 1.2 memory yeteneklerinin evaluation’ı: 3 seviyeli çerçeve

bir memory sisteminin başarısı, **LoCoMo** (long-term conversational memory) gibi benchmark’lar referans alınarak 3 seviyede değerlendirilir.

→ seviye 1: temel hatırlama basic recall: tek bir olguyu veya açıkça söylenmiş bilgiyi (ör: e-posta adresi, pasaport numarası) doğrudan hatırlama

→ seviye 2: çok oturumlu retrieval multi-session retrieval: zaman içerisine ve farklı konuşmalara dağılmış parçalı bilgileri toplama ve değerlendirme. örnek: kullanıcı “arabam için bakım planla” dediğinde, geçmiş oturumlardan kullanıcının hem honda hem de bir tesla sahibi olduğunu çıkarıp tahmin yürütmek yerine hangi aracı kastettiğini sormak

→ seviye 3: proaktif hizmet proactive service: küresel bakış açısı ile anlık detayları birleştirip kullanıcının sormadığı ihtiyacı öngörerek aksiyon alma. örnek: fıstık alerjisi olan kullanıcıya yemek önerirken alerjen uyarısını proaktif yapması veya uçuş rezervasyonu yaparken son kullanma tarihinin süresinin dolacağını önceden bildirmesi

---

### 1.3 belleğin hiyerarşik yapısı (nerede saklanır?)

memory mimarisi, bilginin nerede saklanacağına göre üç katmana ayrılır: 

→ 1.trajectory (yörünge / oturum geçmişi): tek bir konuşmanın eksiksiz, kronolojik ham kaydıdır. salt ekleme yapılır, asla değiştirilmez

→ kullanıcı uzun vadeli belleği: oturumlar arasında damıtılan, birleştirilen, düzenli olarak güncellenen ve budanan kararlı bilgi deposudur.

→ iş durumu (task state): mevcut görevin hangi aşamada olduğunu yöneten geçici çalışma alanıdır

---

### 1.4 dört depolama formatı ve ödünleşimleri (nasıl saklanır?)

kullanıcı bilgisi farklı granülarite ve yapısal karmaşıklık düzeyinde test edilebilir.

simple notes (düşük maliyet, ilişki kaybı) → enhanced notes (anlatısal bütünlük, arama karmaşası) → json cards (3 düzeyli yapı, katı kategoriler) → advanced json cards (ilişkiler, gizem giderme, yüksek üretim maliyeti)

1. simple notes: atomik, bölünemez gerçeklerdir. örneğin kullanıcı epostası: [john@example.com](mailto:john@example.com). O(1) işlem hızı sunar ancak gerçekler arasındaki nedensellik ve ilişki bağlantılarını koptuğu için sentez gerektiren sorgularda zorlanır. 

O(1) → Memory’de 10 tane not da olsa, 10 milyon not da olsa; agent _"Kullanıcının e-postası nedir?"_ sorusuna tüm memory’yi taramadan, doğrudan `e-posta` etiketine bakarak erişebilir. Ortalama O(1) erişim, uygun bir hash tablosu ve anahtarın bilinmesi gibi varsayımlara bağlıdır; depolama ve model işlemleri yine maliyet yaratabilir.

İlişkilerin kopması ve sentez sorgularında zorlanma → bilgiler tekil atomlara bölündüğünde, aralarındaki neden-sonuç veya context bağları kaybolur. 

sorun: agent bu 3 notu yan yana koyduğunda, bu unvanın bu şirkete mi ait olduğunu, yoksa ahmet’in önceki işindeki unvanı mı olduğunu anlamayabilir.

örnek: ahmetin mevcut işindeki sorumlulukları unvanıyla nasıl ilişkileniyor gibi birden fazla bilginin birleştirilmesini yani sentezi gerektiren karmaşık sorular geldiğinde sistem bu kopuk parçaları yeniden bir araya getirmekte ve mantıksal bağ kurmakta zorlanır

1. enhanced notes: paragraf düzeyinde bütünsel bir anlatı saklar. anlamsal olarak eksiksizdir ancak metin uzadıkça embedding modelleri kilit noktaları karıştırır ve güncellemesi zorlaşır. 
2. json cards: kategori → anahtar-değer ilişkisi kurar. (ör: work.position.title). kısmi güncellemeyi destekler ancak katı şema sınırlarına takılır
3. advanced json cards: json yapısına backstory, person, relationship gibi ek context ve ilişki agent’ları ekler. belirsizlikleri giderir ve ifade gücü en yüksektir, ancak üretimi ve bakımı llm açısından pahalıdır. 

**üretim ilkesi**: kritik ve düşük hacimli veri (tercihler, alerjiler) için advanced json cards; yüksek hacimli, kritik olmayan konuşma gerçekleri için simple notes kullanılır.

---

### 1.5 gelişmiş bilgi temsili: çalıştırılabilir kod (user as code)

metin tabanlı biçimler toplama, kısıt kontrolü ve çelişki tespitini llm’in kafadan hesabına bırakır. user as code yaklaşımı ise kullanıcı durumunu tipli nesnelere (dataclass, pydantic) ve kuralları çalıştırılabilir python işlevlerine dönüştürür.

örnek: pasaport son kullanma tarihi ile uluslararası uçuş tarihi arasındaki 180 günlük sınırı kontrol eden koda dayalı bir işlev çalıştırmak, llm’in metin üzerinden tarih hesaplaması yapmasından çok daha güvenilir sonuç verir. 

---

### 1.6: bilişsel bilim temelleri (ne saklanır?)

insan belleğinden esinlenen bu boyut, saklanan bilginin türüne odaklanır:

→ episodic memory: belirli olay diizleri ve deneyimlerdir. örneğin: geçen cuma tokyo uçuşu arandı

→ semantic memory: olaylardan soyutlanmış genel olgulardır (örneğin: pencere kenarı koltuk tercih ediyor)

→ procedural memory: yeniden kullanılabilir davranışsal alışkanlıklar ve prosedürlerdir (örneğin: önce direkt uçuş ara → koltuğu onayla → mil numarasını gir)

→ çalışma belleği: context penceresi içerisindeki aktif görev alanıdır

---

### 1.7: gerçek dünya vaka analizleri: Mem0 ve Memobase

endüstrideki iki belirgin açık kaynak memory yaklaşımı: 

→ Mem0 (yazma anında karardan getirme anında akıl yürütmeye)

- v2 mimarisi: yazma anında uzlaştırma yapar, llm aday olguyu çıkarır, vektör veritabanında arar ve `ADD, UPDATE, DELETE, NOOP` kararı verir. memory derli toplu kalır ancak hatalı bir silme işlemi geçmişi kalıcı olarak yok edebilir
- v3 mimarisi: sadece ekleme yapılan `ADD-only` depodur. geçmiş korunur: çelişkiler getirme (retrieval) anında zaman damgası ve alaka düzeyiyle çözülür. 

→ Memobase (kullanıcı profilleri + olay belleği):

- bilgiyi kararlı kullanıcı profili (yuvalar/slots) ve zaman çizelgesine bağlı olay belleği olarak ikiye ayırır. konuşmaları bir tamponda biriktirip toplu işleyerek llm maliyetini ve yanıt gecikmesini düşürür.

---

### 1.8 memory sıkıştırma mekanizmaları

memory hacmi büyüdükçe sistem üç katmanlı bir sıkıştırma uygular: 

- önem puanlaması: erişim sıklığı, zaman çürümesi, duygusal yoğunluk ve bilgi benzersizliği dikkate alınarak düşük puanlı anılar silinir veya arşivlenir
- kümeleme: benzer konuşmalar gruplandırılarak temsilci bir özet üretilir
- soyutlama: belirli epizodik anılardan anlamsal veya işlemsel kurallar üretilir

---

  

# RAG Temelleri ve Retrieval Pipeline

  

### 2.1 RAG’ın temel mimarisi (retriever + generator)

  

retrieval-augmented generation, llm’in akıl yürütme yeteneğini dışsal bir knowledge base’in genişliği ve güncelliğiyle birleştirir. sistem iki ana bileşenden oluşur: 

  

1. retrieval getirici: kullanıcının sorgusuyla ilgili doküman parçalarını veritabanından bulur
2. generator üretici: getirilen parçaları context olarak alıp soruya yanıt üretir

  

temel akış: sorgu alınır → ilgili parçalar getirilir → contexte eklenir → llm yanıtı sentezler

  

---

  

## 2.2 doküman parçalama chunking ve ödünleşimleri

  

dokümanlar veritabanına yüklenmeden önce çevrimdışı bir ön işlemeden geçerek küçük parçalara bölünür. chunking iki nedenle zorunludur:

  

- embedding modellerinin belirli bir girdi uzunluğu sınırı vardır, devasa bir metin tek bir vektöre sıkıştırılırsa konular birbirine karışır
- büyük parçalar contexte enjekte edildiğinde attention mekanizmasını seyreltir ve context alanını israf eder

  

**chunking stratejileri ve boyut ödünleşimi**

  

- küçük chunk riskleri: parça çok küçük olursa semantik context kopar. örn “şirketin geliri %3 arttı” ifadesinde hangi şirketin veya hangi çeyreğin kastedildiği bilgisi kaybolur
- büyük chunk riskleri: parça büyüdükçe vektörün temsil gücü seyrelir ve arama sırasında çok sayıda ilgisiz içerik sisteme taşınır
- uygulama standardı: genelde chunk başına 256-1024 token ve bitişik parçalar arasında %10-%20 örtüşme overlap kullanılır
- semantik chunking: cümlelerin vektör benzerliklerini hesaplar ve anlamın keskin biçimde değiştiği semantik uçurum noktalarından bölme yapar

---

  

### 2.3 dense embeddingler: anlamsal semantik arama

  

bilgisayarlar metinleri doğrudn anlayamadığı için vektör uzayında birer sayı dizisine dönüştürülür. anlamsal olarak birbirine en yakın metinlerin vektörleri de matematiksel olarak yakın olur. yakınlık genellikle cosinus benzerliği ile ölçülür.

  

1. word2vec (2013) statik vektörler, çok anlamlılık çözülemez
2. bert (2018) context duyarlı, polysemy çözüldü
3. bge-m3 (2024) çok dilli, long-context

  

- word2vec statik vektörler: “kral” - “erkek” + “kadın” ≈ “kraliçe” gibi doğrusal ilişkileri yakalayabilir, ancak çok anlamlılık polysemy sorununu çözemez. “banka” kelimesine nehir kıyısı ile finans kurumu için aynı vektörü atar
- bert & modern modüller context aware: self-attention sayesinde aynı kelime geçtiği cümleye göre farklı bir vektör kazanır
- ann (yaklaşık en yakın komşu) indekslemesi: milyonlarca vektör arasında hızlı arama yapmak için ANNOY (ağaç tabanlı, hızlı inşa, düşük memory, artımlı güncelleme yapamaz) veya HNSW (graf tabanlı, yüksek doğruluk, artımlı güncellemeyi destekler) algoritmaları kullanılır.

---

  

## 2.4 sparse embedding: bm25 ile tam eşleşme

  

dense embeddingler anlamı kavrasa da kod parçaları, ürün modelleri veya özel isimlerde zorlanır. sparse embedding seyrek gömmeler ise geleneksel arama motoru mantığına, yani tam anahtar kelime eşleşmesine dayanır.

  

- ters indeks: bir kitabın arkasındaki indeks gibidir, kelimeden dokümanlara doğru bir haritalama tablosu tutar (“TCP” → sayfa 45, 112)
- BM25 puanlama mantığı
    
    1. tf (term frequency): kelime dokümanda kaç kez geçiyor? doygunluk parametresi k1 sayesinde kelime tekrar ettikçe skora katkısı azalarak dengelenir
    
    2. idf (inverse document frequency): kelime tüm doküman koleksiyonunda ne kadar nadir? nadir geçen kelimeler ör: “damıtma”, çok sık geçen kelimelere ör:”model” göre skora çok daha yüksek katkı sağlar
    
    3. uzunluk normalizasyonu: uzun dokümanların sırf fazla kelime geçirdiği için haksız avantaj elde etmesini engeller

  

---

  

## 2.5 hibrit retrieval ve neural reranking (iki dünyanın en iyisi)

  

dense arama semantiği anlar ama detay kodları kaçırabilir; sparse arama tam eşleşir ama eş anlamlı kelimeleri anlayamaz. “kedicik” aramasında “kedi” kelimesini bulamaz örneğin. hibrit retrieval, iki motoru paralel çalıştırıp sonuçları birleştirir.

  

kullanıcı sorgusu → dense search semantik → kosinüs skoru 0 ile 1 arasında  
  
kullanıcı sorgusu aynısı → sparse search bm25 → bm25 skoru 0 ile 10+ arasında

  

ikisi birleşir RRF

  

1. paralel arama: sorgu her iki motora aynı anda gönderilir.
2. sonuç füzyonu rrf: dense skoru 0 ile 1 arası ile bm25 skoru 0 ile 10+ arası doğrudan toplanamaz. rrf ham puanları çöpe atıp sadece sıralamalara bakar. 
3. neural reranking (cross-encoder)
    
    - bi-encoder (retrieval aşaması): sorgu ve dokümanı ayrı ayrı vektörleştirip karşılaştırır, devasa veride ultra hızlıdır fakat yüzeyseldir (işe alımda özgeçmiş tarayan ik uzmanı gibidir)
    
    - cross-encoder (reranking aşaması): sorgu ile aday dokümanı tek bir metinde birleştirip llm katmanında kelime kelime derin etkileşimle analiz eder. yavaştır ama mükemmel isabet sağlar. (birebir mülakat yapan görüşmeci gibidir)

---

retrieval kalitesini ölçme metrikleri

  

bir rag pipeline’ının başarısı 3 temel metrikle ölçülür: 

  

- recall@k (isabet oranı / hit rate): doğru cevabı içeren dokümanın ilk k sonuç arasında görünme oranı
- mrr (mean reciprocal rank): ilk doğru sonucun kaçıncı sırada çıktığının tersinin ortalaması (1. sırada çıkarsa 1 puan, 10.sırada çıkarsa 0.1 puan)
- nDCG: ilgili dokümanların listedeki sırasını ve önemini indirimli olarak hesaplayan genel sıralama kalitesi

  

---

  

# gelişmiş bilgi organizasyonu ve yapılandırılmış indeksleme

  

## 3.1 raptor: ağaç hiyerarşik endeksi

  

- temel felsefesi: raptor (recursive abstractive processing for tree-organized retrieval), metin parçalarını aşağıdan yukarıya doğru özyinelemeli biçimde soyutlayarak bir bilgi ağacı kurar.
- çalışma adımları:
    
    - önce uzun dokümanlar yaprak düğümler (leaf nodes) olarak küçük metin parçalarına bölünür
    
    - semantik olarak benzer yaprak düğümler kümeleme algoritmalarıyla gruplandırılır
    
    - bir llm, her küme için bir ebeveyn düğüm parent node oluşturacak üst seviye bir özet yazar
    
    - bu süreç özyinelemeli olarak tekrarlanır ve ayrıntılardan genel kavramlara uzanan hiyerarşik bir ağaç oluşur
- getirme (retrieval) dinamiği: katmanlar arası gezinme yapar. sistem önce makro düzeydek ibir özet kavramı bulup ardından ağaç yapısı boyunca derinleşerek ayrıntılı teknik açıklamalara inebilir.
- ideal kullanım senaryosu: makro kavramlardan ayrıntılara kademeli olarak inmeyi gerektiren bir kavramdan detaylara inme sorgularında en yüksek performansı verir

  

---

  

## 3.2 graphrag: varlık-ilişki bilgi grafı 

  

- temel felsefesi: doküman bilgisini varlıklar (düğümler) ve aralarındaki ilişkiler kenarlar üzerinden özne-ilişki-nesne üçlüleri şeklinde bir bilgi ağı olarak modeller
- kilit avantajları: 
    
    1. çok sıçramalı ilişkisel akıl yürütme: multi-hop reasoning “doktorumun çalıştığı hastanenin adresi nedir” gibi sorular doğrudan kullanıcı → doktor → hastane → adres ilişki zinciri boyunca graf kenarları taranarak çözülür. düz memory depolarında bu tür sorgular zincirin kopmasına yol açar
    
    2. varlık belirsizliği giderme (entity disambiguation): aynı adı taşıyan iki farklı bireyi (ör. diş hekimi dr. zhang ile kardiyolog dr. zhang) ayrı düğümler ve ilişki kenarları olarak tutarak karışıklığı ek akıl yürütmeye gerek duymadan çözer
    
    3. topluluk tespiti (community detection): graftaki semantik olarak sıkı varlık kümelerini otomatik keşfedip tematik zihin haritaları ve özetler üretir
- getirme dinamiği: ilişki ağında gezinir. grafta ilgili varlığı bulduktan sonra bağlı olduğu kenarlar boyunca komşu varlıklara ve detaylara ulaşır
- ideal kullanım senaryosu: a ve b arası ilişki nedir ve ya kim kiminle bağlantılı gibi dokümanlar arası sentez ve çoklu varlık ilişkisi gerektiren sorularda en üstün sonucu verir

---

## 3.3 raptor vs graphrag karşılaştırması

  

raptorda gezinme yönü katmanlar arası (makrodan mikroya), graphragda ilişki ağı boyunca (varlık → komşu varlık)

  

raptorda sorgu odak noktası bir kavramdan ayrıntılara adım adım inme, graphrag’da çok sıçramalı ilişkiler, a ve b etkileşimleri

  

raptorda maliyet eşiği yüksek llm özetleme maliyeti, graphrag’da çok yüksek llm varlık çıkarma maliyeti

  

**Mühendislik Kararı:** Basit arama sorguları (_"İade politikası nedir?"_) için standart hibrit retrieval yeterlidir. Ancak dokümanlar arası sentez ve çok düzeyli gezinme gerekiyorsa yapılandırılmış indeksleme kullanılmalıdır. Temel bilgiyi doğal dilde koruyup, dikey ve ilişkisel senaryolar için bilgi graflarını özel bir indeksleme katmanı olarak eklemek en doğru mimari stratejidir.

  

---

  

## 3.4 dosya sistemi paradigması: openviking ve sanal uri’ler

  

akademinin ötesinde bytedance tarafından açık kaynaklanan openviking, dosya sistemi yaklaşımını önerir

  

- sanal uri yapısı (viking://): tüm context, memory ve beceriler sanal bir dosya sistemine eşlenir
    
    - `viking://resources/`: dışsal dokümanlar, kodlar, web sayfaları
    
    - `viking://user/memories/`: kullanıcı tercihleri ve alışkanlıkları
    
    - `viking://agent/skills/` ve memories/: agent’ın kendi becerileri ve çalışma kayıtları
- neden markdown düz metin? özel bir db yerine md seçilmesi, insanların doğrudan okuyup düzenleyebilmesş, git ile sürüm kontrolü yapılabilmesi ve agent’ların otonom olarak write_file ile çalışabilmesi içindir
- yatay bağlantılama (wikipedia modeli): dosyaların izole kalmaması için metinler arasında çapraz referanslar ve indeks sayfaları kurulmalıdır. bu yatay bağlantılar graphrag’ın gezinme gücünün bir kısmını hafif dosyalarla elde etmeyi sağlar

---

  

## 3.5 bilgi güncelleme mimarisi: git pr ve proposer-reviewer akışı

  

canlı sistemde bilginin doğrudan ana db’ye yazılması risklidir. güvenli yaklaşım bilgi değişikliklerini git pr gibi ele almaktır.

  

1. proposer agent (öneren): ham kanıtta yeni bir gerçek bulduğunda çalışma dalında mümkün olan en küçük diff’i önerir
2. reviewer agent (inceleyen): bağımsız bir model olarak çalışır. change öncesi bilgiyi, diff’i ve ham kanıtı denetler; iddianın doğruluğunu ve çelişkilerini kontrol eder
3. yakınsama ve yayın: inceleyen onayına kadar yinelenir. onay sonrası ci testleri çalışır ve değişiklik ana dala birleştirilip indeksler artımlı olarak güncellenir
4. üç katman ayrımı: 
    
    - ham kanıt katmanı: yalnızca ekleme yapılan append-only konuşma ve doc kayıtları
    
    - bilgi katmanı: damıtılmış md ve code files
    
    - servis katmanı: birleştirilmiş sürümden türetilen vektör arama indeksleri
    
      
    

**Heterojen Model İlkesi:** Proposer ve Reviewer'ın farklı model ailelerinden seçilmesi (ör. Claude Proposer + GPT Reviewer) aynı sistemsel kör noktalara ve hatalara düşme riskini azaltır

  

---

  

# agentic rag, contextual retrieval ve çift katmanlı memory sentezi

  

## 4.1 agentic rag: tool tabanlı retrieval paradigması

  

geleneksel rag, sabit ve tek yönlü bir veri akışıdır: kullanıcının sorgusu alınır. tek bir arama yapılıp sonuçlar context’e enjekte edilir ve yanıt üretilir. ancak bu pasif yaklaşım, karmaşık veya çok adımlı problemlerde yetersiz kalır.

  

agentic rag ise retrieval sürecini otomatik bir ön adım olmaktan çıkarıp agent’ın istediği an çağırabileceği bir tool (knowledge_base_search) olarak kapsüller. agent react (düşün → eyle → gözlemle) döngüsünü çalıştırarak süreci otonom yönetir.

  

1. düşün: problemleri alt parçalara böler, eksik bilgileri belirler
2. eyle: arama aracını çağırarak spesifik sorgular çalıştırır
3. gözlemle: gelen bilgiyi değerlendirir; yetersizse arama stratejisini değiştirip ikincil aramalar yapar
4. sentez: yeterli bilgiye ulaştığında nihai yanıtı sentezler

  

**Deney Bulgusu:** Basit sorularda ("İade politikası nedir?") Non-Agentic RAG hızıyla avantaj sağlarken; karmaşık hukuki vakalarda (örneğin birden fazla suç unsurunun birleştiği cezalandırma soruları) Non-Agentic RAG eksik veya yanlış bilgi getirirken, Agentic RAG yinelemeli aramalarla çok sıçramalı (_multi-hop_) doğruluğu belirgin biçimde artırır

  

---

  

## contextual retrieval (context duyarlı parçalama)

  

4.2. Context Duyarlı Parçalama

Geleneksel doküman parçalamanın (_chunking_) temel sorunu, metin bloklarını orijinal context’inden koparmasıdır. Örneğin, “Şirketin ikinci çeyrek geliri %3 arttı” cümlesi tek başına ele alındığında şirket, yıl ve ürün bilgileri belirsizleşir. Bu durum hem BM25 anahtar kelime eşleşmesini hem de vektör embedding kalitesini düşürür.

Anthropic tarafından önerilen [**Contextual Retrieval**](https://www.anthropic.com/research/contextual-retrieval) tekniği, bu sorunu indeksleme aşamasında çözmeyi amaçlar. Metin parçası indekslenmeden önce bir LLM, parçanın dokümandaki konumunu ve konusunu açıklayan kısa bir **contextual prefix** üretir. Bu ön ek, orijinal metinle birleştirilerek indekslenir.

**Örnek ön ek:**

> Bu metin, ACME Corporation'ın 2025 2. Çeyrek Finansal Raporu'ndaki “Temel Performans Göstergeleri” bölümünden alınmıştır.

```text
Orijinal doküman
        ↓
Parçalama
        ↓
LLM ile ön ek üretimi
        ↓
Ön ek + metin
        ↓
Vektör/BM25 indeksi
```

Bu yöntem:

- BM25 araması için daha zengin anahtar kelimeler sağlar.
- Dense retrieval için metin parçasının semantik context’ini güçlendirir.
- Retrieval başarısızlık oranını BM25 ile %49, neural reranker ile birlikte kullanıldığında ise %67 oranında azaltabilir.
- Ek LLM çağrı maliyetini prompt caching ile düşürür.

---

  

### 4.3. Contextual Retrieval'ın Kullanıcı Belleğine Uygulanması

Agentic RAG ve Contextual Retrieval, kullanıcı geçmişine uygulandığında uzun vadeli memory performansını artırır.

- **Context belirsizliğinin giderilmesi:** “Tamam, bunu ayırtalım” gibi izole konuşma parçaları, context ön ekleriyle anlamlandırılabilir. Örneğin:
    
    - `[Patricia Thompson ile ilk banka transferi ayarlanıyor]`
    
    - `[Kullanıcı Şanghay–Seattle uçuşunu tartışıyor]`
    
    Bu sayede agent, geçmiş konuşmalardaki talimatları ve olayların zaman sırasını daha doğru yorumlayabilir.
    
- **Çok oturumlu arama:** Agent, `search_user_memory` aracıyla farklı oturumlarda geçen bilgileri bir araya getirebilir. Örneğin, ayrı konuşmalarda bahsedilen Honda ve Tesla araçları birlikte değerlendirilerek daha tutarlı yanıtlar üretilebilir.

### 4.4. Çift Katmanlı Memory Sentezi

User memory ve RAG knowledge base, **çift katmanlı memory mimarisi** altında birleştirilebilir.

```text
┌──────────────────────────────────────────────────────────────┐
│ 1. Katman: Yerleşik Context                                │
│                                                              │
│ - Kritik ve az sayıdaki kullanıcı bilgisi                   │
│   Örn. alerjiler, pasaport bitiş tarihi                    │
│ - Sürekli olarak sistem prompt'unda veya context'te tutulur │
│ - Genel bakış sağlar                                        │
└──────────────────────────────────────────────────────────────┘
                              ▲
                              │ İlişkilendirme ve tetikleme
                              ▼
┌──────────────────────────────────────────────────────────────┐
│ 2. Katman: Dinamik Retrieval Katmanı                        │
│                                                              │
│ - Geniş konuşma geçmişi veritabanında saklanır              │
│ - Gerektiğinde ayrıntılı bilgiler getirilir                │
│ - Contextual RAG kullanılır                                 │
└──────────────────────────────────────────────────────────────┘
```

Seviye 3: Proaktif Hizmet

1. **Yerleşik bilgilerin incelenmesi:** Agent, kullanıcının pasaportunun 18 Şubat 2025 tarihinde sona ereceğini ve ocak ayında Tokyo seyahati planladığını görür.
2. **İlişkilendirme:** Bu iki bilgi arasındaki yakınlığı fark ederek olası bir risk tespit eder.
3. **Ayrıntı doğrulaması:** Contextual Retrieval ile geçmiş konuşmalardaki uçuş ve pasaport bilgilerini kontrol eder.
4. **Proaktif uyarı:** Kullanıcı sormadan şu uyarıyı sunar:
    
    > Ocak ayındaki Tokyo uçuşunuz için pasaportunuzun geçerlilik süresi yetersiz kalabilir. Acilen yenilemenizi öneririm.
    

> **Kilit ilke:** Yalnızca yerleşik context kullanıldığında ayrıntılar kapasite sınırları nedeniyle kaybolabilir. Yalnızca RAG kullanıldığında ise genel bakış eksikliği nedeniyle bilgiler arasındaki bağlantılar gözden kaçabilir. Çift katmanlı mimari, genel bakış ile ayrıntılı aramayı birleştirerek proaktif hizmeti mümkün kılar.

### 4.5. Çok Modlu Memory

İnsan yüzleri ve ses tonu gibi metinle ifade edilmesi zor veriler için üç temel memory yaklaşımı öne çıkar:

1. **Metin açıklamaları ve dosya referansları:** Görsel veya ses dosyası kaydedilir, metinle açıklanır ve bir dosya referansıyla ilişkilendirilir. Ancak metne aktarılamayan ayrıntılar kaybolabilir.
2. **Context'te çok modlu embedding saklama:** Görsel veya ses verisinin embedding'i oluşturularak context penceresinde ayrılmış bir alanda tutulur.
3. **Model parametrelerine gömme:** Kullanıcı verileri doğrudan model ağırlıklarına yazılmak yerine, önceden eğitilmiş bir Engram modelinin hash tabanlı N-gram yuvalarına yerleştirilir.