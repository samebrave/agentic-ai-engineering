# Agent nedir?

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter1/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

# AI Agent’ları Derinlemesine Anlamak kitabı

“Bu kitabın çekirdek formülü tek bir cümleden ibaret: Agent = LLM + Context + Tools”, daha sezgisel söylersek beyin + gözler + el ve ayaklar. Beyin LLM düşünme, gözler context agent’ın hangi bilgiyi görebileceği, el ve ayaklar tools agent’ın hangi işleri yapabileceğini belirler.

**context engineering bölümü kitabın en kritik bölümüymüş. context’i sistematik olarak anlatıyor.** 

her bölüm çok sayıda deney ve düşünce sorusu içerir, numaralandırma biçimi deney x-y şeklinde x bölüm no, y bölüm içindeki sıra no. 

deneylerin ve düşünce sorularının başlıklarında zorluk yıldızla belirtilir: ★ giriş seviyesini gösterir ve tüm okurlara uygundur; ★★ orta zorluğu gösterir ve belli bir mühendislik pratiği birikimi gerektirir; ★★★ ise ileri düzey bir meydan okumadır ve genellikle açık uçlu sorular ya da karmaşık sistem tasarımı içerir.

**agent teknolojisi hızla evriliyor ancak iyi mimari tasarım ilkeleri zamanı aşacak güce sahiptir. neden böyle tasarlanıyor sorusunun yanıtını kavradığınızda, teknoloji dalgalarının değişimi içinde berrak bir muhakeme gücünü koruyabilirsiniz.**

## 1. ai agent’lara giriş

“Okuma İpucu: Bu bölüm, kitabın tamamı için kavramsal bir harita niteliğindedir—Agent’ların temel formülüne, çalışma döngüsüne, mühendislik çerçevesine ve tasarım kalıplarına hızlı bir bakış sunarak sonraki bölümlerin üzerine inşa edeceği ortak kelime dağarcığını ve referans noktalarını oluşturur. İlk okumada her kavramı ezberlemeye çalışmayın; genel bir izlenim edinmeyi hedefleyin. Sonraki her bölüm burada tanıtılan bir yönü derinlemesine ele alır ve yönünüzü kontrol etmek için her zaman buraya geri dönebilirsiniz.”

**Agent = LLM (Büyük Dil Modeli) + Context + Tools**

llm beyindir: sadece bir model parametre kümesi değil, agent’ın niyeti anladığı, düşündüğü, plan yaptığı ve karar verdiği bütün karar alma çekirdeğidir. tıpkı insan beyninin salt nöronlar topluluğundan ibaret olmayıp deneyimle şekillenen düşünme biçimlerini de taşıması gibi, bir llm’in yetneeği de iki kaynaktan gelir: pre-training yoluyla biriktirilen dünya bilgisi ve dil yeteneği, ve post-training ile kalıcı hale gelen karar alma stratejileri

context agent’ın gözleridir: sadece modele verilen metin değil, agent’ın her karar noktasında görebildiği her şeydir - ortam, kullanıcı belleği, alan bilgisi, kendi durumu ve görev ilerlemesi. - tıpkı bir kişinin karar verirken durumu değerlendirmesi, ilgili deneyimi hatırlaması ve referanslara başvurması gerektiği gibi, agent’ın context penceresi de o anda görebildiği her şeydir. 

tools, agent’ın el ve ayaklarıdır: bir avuç çağrılabilir api fonksiyonu değil, agent’ın yapabildiği her şeyin tam kümesidir — önceden tanımlanmış tool çağrılarından ihtiyaç halinde yüklenen beceirlere (skills), yeni yetenekler yaratmak için anlık kod üretmekten alt agent’lara (sub-agent) iş devretmeye, kullanıcıya ulaşmaktan dış olaylara yanıt vermeye kadar uzanır.

daha sezgisel ifadeyle **agent = beyin + gözler + el**

agent ve ortam, kapalı döngülü etkileşimin iki tarafıdır, birbirlerinin parçaları değildir. ortam bir gözlem döndürür, agent context’ini kullanarak sonraki eylemi seçer ve bu eylem ortam’ın durumunu değiştirerek sonraki gözlemi üretir.

ortam dosya sistemi, veri tabanı, web sayfaları, users, diğer agents ve fiziksel veya simüle dünyaları içerir, iç düzey agent içindeki model-harness yapısıdır: model politika kararlarını verir, harness, agent sınırları içinde context’i oluşturan, tool arayüzlerini açan, döngü ve durumu yöneten ve izin, doğrulama ve düzeltme uygulayan çalıştırma ve yönetişim katmanıdır. harness bir ortam yaratabilir, yalıtabilir veya proxy olarak sunabilir ancak ortamın durumunu ve geçiş kurallarını içermez.

**mühendislik formülü bu nedenle şöyle açılır: llm model’e karşılık gelir, context + tools asgari harness’i oluşturur, üretim sistemleri aynı sınır içinde kısıtlama, doğrulama ve düzeltme ekler.**

### 1.1.1 observation ve action space: model ile dünya arası arayüz

os ve as birlikte llm ile dış ortam arası arayüzü oluşturur. os ortamdaki bilgiyi modelin işleyebileceği contexte dönüştürür, as ise modelin kararlarını dış dünyadaki işlemlere dönüştürür. os’a girmeyen bilgi yok hükmünde, as’ta bulunmayan işlemse modelin aksiyonlarında yok hükmünde

“Bu nedenle, temel model sabit tutulduğunda Agent performansını artırmanın başlıca sistem mühendisliği kaldıracı, **Observation ve Action Space’i yeniden tanımlamak veya genişletmektir.**” → context’i ve tools’u genişletmek. *****daha akıllı bir model gerektiriyor gibi görünen birçok sorun aslında bir arayüz sorunudur*****

**manus case: ayrı olan alanları birleştirmek, manus’tan önce deep research, coding ve computer use ayrıydı. manus birleştirdi. sanal tarayıcı os’u, dosya sistemi, kod çalıştırma ve komut satırı ise as’ı genişletti. manus’ın olayı daha güçlü bir modele geçmek değil, os ve as’ı birleştirerek sınırı aşmaktı**

**openclaw case: arayüzü kullanıcının dijital yaşamına genişletmek. openclaw her iki agent’ı da bir adım daha dışarı taşıdı. wp, tg, slack, discord gibi zaten kullanılan mesajlaşma uygulamalarından görev alıp sonuç döndürdüğü için neredeyse her yerden erişilebildi. böylece orada zaten varolan dosyalar os’a girdi ve tools işleyebildi.**

*****ürün evrimi çoğu zaman tam olarak os ve as’ın genişlemesidir.*****

**agent ürünü 1: kodlama agent’larının gözleri gereksinim dokümanları, kod tabanı ve terminal ortamı, el ve ayaklar açık uçlu reasoning, kod arama, dosya okuma yazma ve komut çalıştırma, strateji ise artımlı geliştirme: gereksinimi anla → ilgili kodu ara → kodu düzenle → test et ve doğrula → hata ayıkla ve düzelt**

**agent ürünü 2: arama agent’ları: gözler web kaynakları, akademik veritabanları, el ve ayaklar açık uçlu reasoning, arama sorguları, strateji yinelemeli derinleştirme: mevcut bilgiye göre arama yönünü ayarla, tam bir raporu kademeli olarak sentezle.**

**agent ürünü 3: computer control agent’ları: gözler bilgisayar ekranı tarayıcı sayfaları, el ve ayaklar açık uçlu içsel reasoning, tıklama, yazma, ss, strateji ise görsel algı + işlem: ekranı gözlemle → hedef öğeleri belirle → eylemi gerçekleştir → sonuçları doğrula**

**agent ürünü 4: telefon asistan agent’ları: gözler telefon ekranı, apps, el ve ayaklar açık uçlu reasoning, tıklama, kaydırma, strateji ise niyet anlama + app control: kullanıcı ihtiyacını anla, hedef uyg bul, eylemi gerçekleştir.**

**agent ürünü 5: kişisel görev agent’ları: gözler kullanıcı hesap bilgileri, geçmiş faturalar, servis sağlayıcı bilgi tabanı, eller ve ayaklar açık uçlu reasoning, arama yapma, eposta gönderme, strateji çok adımlı görev yürütme: bilgi topla → müzakere stratejisi oluştur → servis sağlayıcıyla iletişime geç → müzakere et → raporla**

******bu sistemlerin üç ortak özelliği: açık uçlu eylem alanı, sabit bir düğme kümesinden seçim yapmak yerine keyfi doğal dil ve kod üretme,**

**içsel düşünme, eylemden önce planlama ve reasoning, ve sürekli iletişim, ortamdan gelen geri bildirime göre stratejiyi ayarlama.** 

### 1.1.2 tools: agent’ın el ve ayakları

agent’ı pasif gözlemciden aktif eylemciye dönüştürür. 

algı tool’ları: agent’ın bilgiye erişmesini sağlar, arama motorları gerçek zamanlı web verisi sunar, dosya sistemleri yerel dokümanları okur, api, db…

yürütme tool’ları: agent’ın dünyayı değiştirmesini sağlar. kod çalıştırma, api işlemleri, dosya, db…

iş birliği tool’ları: agent’ın diğer agent’larla iş bölümü yapması. sub-agent devretmesi, çoklu agent…

olay tetikleyici tools: ilk üç kategoriden temelde farklı şekilde devreye girer, agent bunları çağırmaz. bunlar agent’ı harekete geçiren dış girdiler olarak gelir. yeni bir eposta, zamanlanmış an, bir webhook geri çağrısı… olay agent’ı etkinleştirir ve düşünme ile eylemini başlatır.

kullanıcı iletişim tool’ları: agent’ın kullanıcıya ulaştığı kanal. yürütme tool’ları dış dünyayı değiştirirken iletişim tool’ları bigli taşır.

**arayüzler belirsiz tanımlanırsa model bunları yanlış kullanır, hatalar kötü yönetilirse tek bir başarısız tool agent’ı kilitleyebilir, izin fazla geniş tutulursa tek bir agent hatası telafisi imkansız hale gelebilir. mcp standartının yaygınlaştırılması, tool entegrasyonunu kolaylaştırıyor.**

**tool calling, modern llm agent’larının temel bir yeteneğidir: modelin dış tool’ları yapılandırılmış bir şekilde çağırmasını sağlayarak llm’i salt bir metinden eylem alabilen akıllı sisteme dönüştürür.** 

**tool calling 4 adımdan oluşur: önce context modele hangi tool’ların mevcut olduğunu (ad, amaç ve parameter) bildirir; ardından model bir tool çağırıp çağırmayacağına, hangisini çağıracağına ve hangi argümanlarla çağıracağına kendisi karar verir; sonra tool çalıştıktan sonra sonucu context’e eklenir. son olarak model bu sonuca dayanarak hamlesine karar verir. bu döngü reAct’ın temelidir. geliştirici sadece tool’ları tanımlar ve çağrıları yürütür, modelin kendisi çağırıp çağırmayacağına, hangisini çağıracağına ve hangi arg’ları geçeceğine karar verir.**

******genellik hata riskini ve saldırı yüzeyini de büyütür: kod yalıtılmış bir sandbox’ta çalıştırılmalı, ağ erişimi varsayılan olarak kapalı olmalı, yetkilendirilmiş çalışma dizini dışındaki dosyalar okunmamalı ve yürütme süresi, cpu, memory ile çıktı boyutuna üst sınır konmalıdır.******

**genel amaçlı toollar her zaman özel amaçlı toollardan daha iyi değildir. ödeme yapma, veri silme, e-posta gönderme ve üretim ortamına dağıtım gibi yüksek riskli veya katı iş kısıtlarına tabi işlemler; açık parametreleri, sınırlı yetkili ve baştan sona denetlenebilir özel amaçlı tools olarak sunulmaya devam etmeli, gerektiğinde önizleme ve insan onayı da eklenmelidir. dolayısıyla tool tasarımının temel ilkesi şudur: ****genel amaçlı temel yetenekler birleştirme ve keşif için; özel amaçlı tools ise yüksek riskli işlemleri ve katı iş kurallarını sınırlandırmak için kullanılır.******

### 1.1.3 llm: agent beyni

agent’ın karar alma çekirdeğidir. bir kullanıcı isteği karşısında önce gerçek niyeti çözmesi gerekir. *****kullanıcının söyledikleri çoğu zaman gerçekte istedikleriyle aynı değildir.***** ardından belirsiz veya karmaşık bir görevi yürütülebilir adımlara ayırır. yürütme boyunca sürekli yargılarda bulunur. ne yapılacak, tool çağrılacak mı, hangisi? hangi argümanla? anlama, planlama, yürütme.

### 1.1.3.1 model as agent: modelin kendisi ürün olursa

model as agent (agent olarak model) paradigması, ai agent geliştirmedeki en yeni yöndür. “Model as Agent” yaklaşımında model, pekiştirmeli öğrenme sayesinde tool’ları ne zaman, hangisini ve hangi parametrelerle kullanacağına kendisi karar verir; bu nedenle manuel orkestrasyon azalır.

Ancak güçlü modeller daha öngörülemez olabileceğinden, **Harness** katmanı daha da önem kazanır. Harness; modeli kısıtlayan değil, yeteneklerini güvenilir ve kontrollü görev yürütümüne yönlendiren mühendislik altyapısıdır. Context yönetimi, tool arayüzleri, güvenlik kuralları, doğrulama ve hata düzeltme mekanizmalarını kapsar.

_bir modelin kendi kararını verme alanı ne kadar genişlerse, yanlış bir kararın verebileceği zarar da o kadar büyür_

model tedarikçilerinin (model as agent) gerçek avantajı çerçeveyi inceltmek değil, modeli ve etrafındaki harness’i birlikte optimize edebilmek ve bunu sürekli yinelemektir. 

Modeller zamanla bugün Harness’in yaptığı birçok işi—tool kullanımı, planlama, doğrulama ve hata düzeltme gibi—kendileri öğrenebilir. Ancak bu dönüşüm hızlı gerçekleşmez; modellerin eğitilmesi zaman alır ve tüm gerçek dünya kısıtlarını bir anda içselleştiremez.

Bu yüzden Harness geçici ama gereksiz olmayan bir katmandır:

- Modelin henüz güvenilir biçimde yapamadığı işleri üstlenir.
    
- Model bazı yetenekleri öğrendikçe bu görevleri Harness’ten devralır.
    
- Harness de sürekli olarak modelin yeni yetenek sınırlarını desteklemeye odaklanır.
    

Kısacası: **Harness zamanla küçülebilir, fakat model yeterince güvenilir olana kadar kritik bir mühendislik desteği olarak kalır.**

---

**Harness**, bir AI modelinin güvenli, kontrollü ve güvenilir biçimde görev yapmasını sağlayan mühendislik katmanıdır.

Modelin etrafındaki şu bileşenleri kapsar:

- Context’i ve memory’yi yönetme
    
- Tool’ları tanımlama ve çalıştırma
    
- Güvenlik kuralları uygulama
    
- Model çıktısını doğrulama
    
- Hataları tespit edip düzeltme
    
- Gerekirse modeli yeniden yönlendirme
    

Kısaca: **Model karar veren “motor” ise, Harness bu motorun güvenli ve doğru çalışmasını sağlayan kontrol sistemidir.**

---

### 1.1.3.2 agent öğrenme mekanizmaları: context uyarlamasından kalıcı güncellemelere

yukarıdaki tartışma, bir modelin tool kullanma politikalarını pekiştirmeli öğrenme yoluyla yerleşik yetenekler olarak içselleştirebildiğini keşfetti. yani modelin içine yedirilebildi. ancak bir agent’ın davranışındaki değişiklikler yalnızca eğitim sırasında gerçekleşmez.

güncellemenin nerede yapıldığına ve ne kadar kalıcı olduğuna göre bu değişiklikler, birbirini tamamlayan üç yol olarak anlaşılabilir. görev içi context uyarlaması, görevler arasında harici artefaktların güncellenmesi ve eğitim döngülerindeki parametre güncellemeleri.

**eğitim sonrası:** eğitim zamanı, model ağırlıklarını değiştirir, kalıcı ve genel, yüksek maliyet, yavaş güncellenir

**context içi öğrenme:** çıkarım zamanı, dikkat mekanizmasıyla yumuşak güncelleme, geçici ve anında uyum sağlar, context penceresiyle sınırlı

**dışsallaştırılmış öğrenme:** çalışma zamanı, bilgi tabanı + üretilen tools, kalıcı ve güncellenebilir, güvenilir ve doğrulanabilir örn: bir iş akışını toola dönüştürme

---

context uyarlaması mevcut görev içinde gerçekleşir. örnekler, durum ve retrieval sonuçları contexte girince model davranışını hemen ayarlayabilir ancak bu sonraki oturumun kalıcı durumunu değiştirmez. avantajı hız ve düşük maliyet, sınırlaması ise context penceresi ile bilginin düzenlenme biçimidir.

**değişikliklerin görevler arasında kalıcı olması için sistem harici artefaktları güncelleyebilir: gerçekler ve deneyim bilgi dokümanlarında düzenlenebilir.**

---

**CLAUDE.md** ve **AGENTS.md** gibi dosyalar bu sınıflandırmada **dışsallaştırılmış öğrenme** kapsamına girer.

Bunlar modelin ağırlıklarını değiştirmez ve model tarafından kalıcı olarak “öğrenilmez”. Bunun yerine her görev veya oturum öncesinde context’e eklenerek modelin davranışını yönlendirir.

Daha kesin ifade etmek gerekirse:

- Dosya okunup mevcut göreve context olarak ekleniyorsa: **context uyarlaması**
    
- Dosyanın kendisi agent tarafından güncelleniyor ve bu değişiklikler sonraki görevlerde kullanılıyorsa: **dışsallaştırılmış öğrenme**
    
- Kurallar modelin ağırlıklarına eğitim yoluyla işleniyorsa: **eğitim sonrası kalıcı güncelleme**
    

Dolayısıyla **CLAUDE.md** **ve** **AGENTS.md****’nin kendisi harici bir artefakttır**; içeriğinin modele sunulması context uyarlaması, içeriğin deneyimlerden öğrenilerek görevler arasında güncellenmesi ise dışsallaştırılmış öğrenmedir. Burada önemli nokta dosyanın varlığı değil, **agent davranışındaki değişikliğin dosya üzerinden kalıcılaştırılmasıdır**.

---

### 1.1.4 context: agent’ın gözleri

context, bir agent’ın her karar noktasında görebildiği her şeydir. her bir llm çağrısının context’i beş bölümden oluşur:

- system prompt: kullanıcının sırayla yazdığı promptlardan farklı olarak, **system prompt geliştirici tarafından yazılır ve konuşma boyunca sabit kalır.** agent’ın iş tanımıdır. kimliğini, yetkilerini ve davranış kurallarını tanımlar. system prompt’un özenli prompt engineering’i, agent’ın çalışma biçimini şekillendirme yöntemimizdir. **system prompt ayrıca oturumlar arasında kalıcı olan kullanıcı belleğini de (tercihler, geçmiş davranışlar ve arka plan ayarları gibi kişiselleştirilmiş bilgi) ve dinamik olarak enjekte edilen ortam durumunu da taşır.**
    
- tool definitions: agent’ın kullanabileceği tool’ların adlarını, işlevsel açıklamalarını ve parametre formatlarını bildirir. tool tanımları olmadan agent hiçbir aracı tanıyamaz veya çağıramaz. tool tanımları system prompt ile birlikte, konuşma boyunca değişmeden kalan static prefix’i oluşturur. 
    
- user messages: kullanııdan gelen input. kullanıcı mesajları, rag yoluyla dinamik olarak getirilen dışsal bilgiyi de içerebilir. eğitim verisi kesim tarihinin ötesindeki bilgileri veya özel alan bilgisini kapsar.
    
- assistant messages: modelin daha önce ürettiği yanıtlar, en fazla üç bölümden oluşabilir. `reasoning` tutarlılığı ve karar yorumlanabilirliğini koruyan içsel düşünce zinciri, `content` kullanıcıya verilen yanıt ve `tool_calls` agent’ın eyleme geçme biçimi. belirli bir yanıtta bu üç bölüm aynı anda görünmeyebilir: örneğin agent bir tool çağırmaya karar verdiğinde genellikel yalnızca `reasoning + tool_calls` bulunur; nihai bir yanıt verirken genellikle yalnızca `reasoning + content` bulunur.
    
- tool results: agent çerçevesi bir aracı çalıştırdıktan sonra dönen sonuç. bu sonuçlar agent’ın bir sonraki düşünme adımının doğrudan dayanağıdır, ve hatalarını tekrarlamak yerine sonuçlardan öğrenmesini sağlayan şeydir.
    

ilk iki öğe (system prompt + tool tanımları) static prefix’i oluşturur. son üçü (kullanıcı mesajları + asistan mesajları + tool sonuçları) her etkileşimle büyüyen dinamik mesaj geçmişini oluşturur. bu beş bölüm birlikte, her bir llm çıkarımının context’ini oluşturur. 

---

# Chapter 1-1: The Critical Role of Context 

> Key takeaways and conceptual lessons from **Experiment 1-1 (Context Ablation Study)** in _《深入理解 AI Agent》 (Understanding AI Agents Deeply)_ by Bojie Li.

---

## 1. The Golden Rule of AI Agents

> **An AI Agent is not magic. It can only make decisions based on what is in its immediate context window right now.**  
> If any component of that context is missing or hidden, the Agent's decision-making collapses in predictable ways.

---

## 2. What Makes Up an Agent's Context?

Every time the Agent interacts with the LLM, it sends a context package composed of two parts:

```text
CONTEXT = STATIC PREFIX + DYNAMIC TRAJECTORY
```

1. **Static Prefix** (Fixed — stays the same throughout the task):
    
    - **System Prompt**: Who the AI is and its behavioral boundaries.
        
    - **Tool Definitions**: Descriptions and parameter schemas of all available tools.
        
2. **Dynamic Trajectory** (Grows over time with every turn):
    
    - **User Request**: The problem or question to solve.
        
    - **Agent Decisions & Tool Calls**: What the AI chose to do at each step.
        
    - **Tool Results**: The actual data returned from executing those tools.
        

---

## 3. The 4 Key Lessons from the Experiment

### Lesson 1: Without Tool Definitions -> The Agent is Paralyzed

- **What was tested:** Asked the AI to perform currency conversion or calculation, but omitted the tool definitions from the prompt.
    
- **Observed Behavior:** The AI immediately stated: _"I cannot convert currencies without using my tools. Please enable tool calls..."_
    
- **Takeaway:** LLMs do not inherently have hands or eyes. If you don't explicitly declare tool schemas in the static prefix, the agent has zero ability to act in the environment.
    

---

### Lesson 2: Without Tool Results -> The "Blindfolded" Loop

- **What was tested:** The AI called an external tool, but the harness masked the return value (`[Tool result hidden]`).
    
- **Observed Behavior:** The task failed. The agent had no feedback to verify what happened and could not proceed to the next step.
    
- **Takeaway:** Tool calling is a **closed feedback loop** (Reason -> Act -> Observe). Hiding the result is like giving someone a calculator with the display screen turned off.
    

---

### Lesson 3: Without Message History -> The "Amnesia" Trap

- **What was tested:** Wiped the trajectory after each tool execution so the AI received only the original user question on each step.
    
- **Observed Behavior:** The AI suffered total amnesia. It woke up at step 2, thought it was still at step 1, and called the first tool over and over in an infinite loop.
    
- **Takeaway:** Multi-step problem solving requires state memory. Without accumulating past actions and observations, an agent gets stuck in _Groundhog Day_.
    

---

### Lesson 4: Without Reasoning Steps -> Loss of Coherence

- **What was tested:** Stripped the AI's internal step-by-step thinking process from the history.
    
- **Observed Behavior:** On complex multi-step tasks, subsequent decisions began contradicting earlier choices or omitting necessary intermediate steps.
    
- **Takeaway:** Intermediate reasoning acts as working memory for the plan, ensuring multi-turn decisions remain coherent and goal-directed.
    

---

### 1.1.5 react döngüsü

üç bileşen (llm, context, tools) eldeyken doğal bir soru çıkıyor. bunlar birlikte nasıl çalışır? react döngüsü, llm context ve tools’u tek bir sisteme dizen temel mekanizmadır 

bir agent’ın bir görevi yürütme biçimine temel alınan kalıba ReAct (reasoning + acting) denir. sadece reasoning ve actingden bahsetse de aslında üç aşamadan oluşur. model önce sırada ne yapılacağı hakkında reasoning yapar, ardından eylemde bulunmak için bir aracı çağırır (act), sonra aracın sonucunu gözlemler (observe) ve bir sonraki adım hakkında reasoning yapar. düşün → yap → gör → düşün → yap → gör

Somut bir örnek üzerinden—birden fazla para biriminde geliri toplama—bir Agent’ın trajectory’sini (çalışma geçmişi/yörünge) anlayalım: Agent çalışırken birikmiş mesaj geçmişi; kullanıcı mesajları, asistan mesajları (reasoning ve tool call’larıyla birlikte) ve tool sonuçlarından oluşur. Her bir LLM çağrısında, modelin aldığı eksiksiz context, static prefix (system prompt + tool tanımları) artı trajectory (dinamik mesaj geçmişi) toplamıdır (Şekil 1-4). Bu, temel bir gerçeği ortaya koyar: Agent context’i = static prefix + trajectory. Somut olarak, static prefix yukarıdaki beş bileşenin ilk ikisidir (system prompt + tool tanımları); trajectory ise son üçüdür (kullanıcı mesajları + asistan mesajları + tool sonuçları, her etkileşimle büyür). LLM, bu eksiksiz context’ten bir sonraki yanıtını üretir; bu yanıt da bir sonraki çağrı için trajectory’ye eklenir.

örnek: user toplam yıllık geliri hesapla diyip 3 para biriminde değer veriyor 

tur1

assistant.reasoning → eur ve gbp’yi usd’ye çevirip toplamak gerekiyor

assistant.tool_calls → convert_currency

tool(result) → currency’lerin dönüşmüş hali

tur2

assistant.reasoning → döviz kurları alındı, toplamak için code interpreter çağrılıyor

assistant.tool_calls toplama code interpreter’i

tur3

assistant.content “toplam yıllık gelir X”

**üstteki bir agent trajectory’sidir.**

bu en temel tasarımda, llm’in gördüğü context’e sürekli yeni bilgiler eklenir. her llm çağrısı eksiksiz trajectory’i görür, böylece model görevin hangi aşamasında olduğunu, daha önce nelerin denendiğini ve sonucunun ne olduğunu bilir. insanların bir problemi çözerken sürekli gözden geçirip özetlemesi gibi, agent trajectory’si sayesinde göreve dair küresel bir bakış açısı taşır. ve bu trajectory yapılandırılmış olduğundan -kullanıcı mesajları, asistan mesajları (reasoning + tool_calls) ve tool sonuçları temiz bir biçimde ayrıldığından - sistem son derece yorumlanabilir ve hata ayıklanabilir durumdadır.

Trajectory, bir yürütme kaydından fazlasıdır; Agent’ın yeteneğinin bir aynasıdır. Trajectory’leri büyük ölçekte analiz etmek, davranış kalıplarını, daha iyi karar yollarını ve daha iyi tool tasarımlarını ortaya çıkarır. Trajectory verisi hatta bir bilgi tabanına damıtılabilir, ya da pekiştirmeli öğrenme yoluyla daha güçlü Agent modelleri eğitmek için kullanılabilir—deneyimden öğrenme döngüsünü kapatır.

# 1.2 harness engineering

artık agent’ın özünde nasıl çalıştığını biliyorsun: **bir llm, context tarafından yönlendirilerek ReAct döngüsünü yürütür ve görevi tamamlamak için tools kullanır.**

deneylerden de anlaşılabileceği üzere llm’ler kırılgandır. model halüsinasyon görebilir, yanlış tool’u seçebilir, bir hatadan kurtulamayabilir. **çalışan bir demo ile güvenilir bir ürün arasında devasa bir uçurum vardır** ve bu kırılganlıklar tam olarak **harness engineering**’in var olma nedenidir.

bu bölümün ilk yarısında agent’ın ne olduğunu yanıtladık. ikinci yarısında agent’ın üretimde nasıl güvenilir çalıştığını anlatacağız.

unutmayın ve aklınıza kazıyın: **agent = llm + context + tools.** 

bu formül agent’ın içsel bileşimini tanımlar. harness engineering sisteme ikinci, mühendislik uygulama açısından bir bakış ekler: llm’i tek bir temel bileşen (model) olarak ele alın ve etrafına inşa edilen destekleyici koda harness diyin. bu iki bakış açısı birbirine rakip değildir, aynı sistemi farklı soyutlama düzeyinde tanımlarlar. 

**harness = context + tools**

**agent = llm + context + tools**

harness formülüne artı üç katman güvenlik önlemi eklenir. **constrain (agent’ın neyi yapıp neyi yapamaycağı), verify (bir şeyi doğru yapıp yapmadığı), correct (yapmadığında nasıl kurtarılacağı)**

**harness = context + tools + (constrain+verify+correct)**

llm’e artık daha geniş ifadeyle model diyelim

agent = model + harness 

harness = context yönetimi + tool arayüzleri + constrain + verify + correct

**şunu ayırmak gerekir: harness modelin dışındaki her şey demek değildir, agent sınırları içinde ve model’in dışında bulunan çalıştırma ve yönetişim katmanıdır. model-ortam etkileşimine aracılık eder, ancak ortam’ın kendisini çermez. tool tanımları, çağrı adaptörleri, sandbox izinleri ve sıfırlama mekanizmaları harness’e aittir, sandbox içinde değişen dosyalar ve süreçler, harici veritabanları, web sayfaları, kullanıcılar ve fiziksel dünya ise ortam’a aittir.** 

toparlamak gerekirse: 

işlev, tek cümlelik sorumluluk, pratik örnek şeklinde özetleyelim

context → modele algısal bilgi sağlar, bilgi yeterliliği: agent’ın her karar noktasında yeterli bilgiye dayanarak karar vermesini sağlamak. pratik örnek system promptları, bilgi tabanları, agent durum çubukları

tools → modele eylem tool’ları sağlar, net arayüz: tool adları sezgisel, parametrelerin örnekleri var, sınırlar açıklanmış, pratik örnek mcp tool’ları, code interpreter, arama tool’ları

constrain → davranışsal sınırlar koyar, neyin yapılıp neyin yapılamayacağı: güvenli varsayılanlar (fail-safe defaults): tüm yetenekler, pratik örnek claude code’da her tool, çalıştırılmadan önce varsayılan olarak kullanıcı yetkilendirmesi gerektirir, varsayılan olarak kapalıdır ve açıkça etkinleştirilmelidir

verify → işlem sonuçlarının doğruluğuna otomatik olarak karar verir, girdi izolasyonu: güvenlik kontrolleri yalnızca yapılandırılmış verilere (örn tool’ların döndürdüğü json alanları) bakar, modelin ürettiği serbest formatlı metne bakmaz (çünkü saldırganlar prompt inj. yoluyla model çıktısını manipüle edebilir), pratik örnek linter kontrolleri, tip sistemleri, tool çağrısı sonucu doğrulaması

correct → sorun bulunduğunda otomatik olarak düzeltir veya geri alır; bir arıza kurtarılamaz olduğu doğrulanana kadar ara durumları açığa çıkarmayın (örn kullanıcıya yarım kalmış bir sonuç göstermek yerine başarısız bir tool çağrısını sessizce yeniden deneyin), pratik örnek sessiz yeniden denemeler, devam üretimi, ardışık başarısızlıklarda insan yargısına geri dönüş

**context ve tools, agent’ın işi yapmasını sağlar, görevi anlamasını ve ona göre işlem yapmasını sağlar. constrain, verify ve correct ise işi yanlış yapmamasını sağlar.** **context ve tools’tan ayrı bir şey değil, bunların üretimde güvenilir biçimde çalışmasını sağlayan mühendisliktir.** 

erken dönem agent frameworkleri context ve tools’a odaklandı, modele tools verin, context verin, işi yapmasına izin verin. üretim düzeyindeki sistemler ağırlık merkezlerini constrain, verify ve correct’e kaydırdı: tool çağrılarının güvenli old, context’in yönetildiğinden ve hataların kurtarılabilir olduğundan emin olmak.

claude code’u ele alalım, harness kodunun büyük çoğunluğu context ve tools’u değil, constrain, verify ve correct’i yapar — tool’ların kendisi (dosya okuma yazma, komut çalıştırma arama) yalnızca küçük bir parçadır, bunların etrafında inşa edilen güvenlik önlemleri gerçek çekirdektir.

- süreç durumu yönetimi: agent’ın şu anda hangi adımı yürüttüğünü izler
    
- çok katmanlı context sıkıştırma: bilgi çok fazla olduğunda otomatik budar
    
- izin sınıflandırması: hangi işlemlerin kullanıcı onayı gerektirdiğini kontrol eder
    
- circuit breaker (devre kesici): hatalar art arda oluştuğunda otomatik olarak atar ve yeniden denemeyi durdurur - tıpkı bir ev elektrik sisteminde kısa devre olunca sigorta atması gibi tüm sistemin çökmesini önler
    
- hata kurtarma mekanizmaları: istisnaları yakalar, son kararlı duruma geri döner, yeniden dener veya insana devreder
    

sektör işi yapmaktan işi güvenilir biçimde yapmaya kayıyor ve bu da harness engineering’i agent sistemlerinin temel rekabet avantajı haline getiriyor

## 1.2.1 prompt engineering’den loop engineering’e: mühendislik paradigmalarının evrimi

Prompt Engineering, ilk yenilik dalgasıydı—modele verilen doğal dil talimatlarını iyileştirerek çıktı kalitesini artırmak.

  
Context Engineering, ikinci dalgaydı—yalnızca prompt’u optimize etmenin yetmediğinin fark edilmesi: modelin görebildiği her şeyin (sistem talimatları, tool tanımları, konuşma geçmişi, dışsal bilgi) sistematik olarak yönetilmesi gerekiyordu.

Harness Engineering, üçüncü dalgaydı—bakış açısını “modelin görebildikleri”nden “modelin hangi tür bir sistem içinde çalıştığına” genişletir; modelin dışındaki tüm altyapıyı kapsar: kısıtlama mekanizmaları, doğrulama yöntemleri, geri bildirim döngüleri ve hata kurtarma.

ardından loop engineering ortaya çıktı. bakış açısını tek bir çalıştırmadan, çalıştırmalar boyunca devan eden otonom işleyişe genişletti. bir sonraki işi kimin keşfettiği, ne zaman doğrulama yapılacağı ve görevin ne zaman gerçekten tamamlanmış sayılacağı gibi.

temmuz 2026’da sektör, daha üst düzey bir orkestrasyon perspektifi için **graph engineering** terimini kullanmaya başladı. agent döngülerini, deterministik programları ve insan onaylarını açık bir execution graph içinde düzenlemek, burada node’lar yetenekleri sağlar, edge’ler yönlendirme ile bağımlılıkları tanımlar ve yapılandırılmış state bu edge’ler yönlendirme ile bağımlılıkları tanımlar ve yapılandırılmış state bu edge’ler boyunca taşınıp önemli sınırlarda kalıcılaştırılır

Bu beş aşama birbirinin yerine geçmez, iç içe geçmiş katmanlardır: Prompt Engineering, Context Engineering’in bir alt kümesidir; o da Harness Engineering’in bir alt kümesidir; o da Loop Engineering’in bir alt kümesidir. Her katman, mühendisin ilgi ve etki alanını bir öncekinin ötesine genişletir. Modeller yetenek açısından birbirine yaklaşıp belirleyici bir farklılaştırıcı olmaktan çıktıkça, rekabet avantajı modelin dışındaki mühendisliğe kayar.

Yakın zamandaki mühendislik pratiği bunu doğruluyor. LangChain’in Terminal Bench 2.0 (bir Agent’ın terminal ortamında karmaşık görevleri tamamlama yeteneğini değerlendiren bir benchmark) üzerindeki çalışması çarpıcı bir örnektir: Kodlama Agent’ları %52,8’den %66,5’e yükseldi (lider tablosunda ilk 30’un dışından ilk 5’e sıçradı). Değişen model değil, Harness’ti—Agent’ın kendi yürütme sonuçlarını kontrol etmesi, tekrarlayan bir döngüde sıkışıp kalıp kalmadığını tespit etmesi, düşünme stratejisini inceltmesi.

## 1.2.2 etkili agent’lar inşa etmenin temel ilkeleri

anthropic’in deneyimine dayanarak, başarılı agent sistemleri üç temel ilkeyi izler:

keep it simple, basit tutun: en basit çözümle başlayın ve karmaşıklığı yalnızca gerektiğinde ekleyin. doğrudan api çağrıları karmaşık frameworklerden daha iyidir, net kod akıllıca soyutlamadan daha iyidir, hata ayıklarken her ekstra soyutlama katmanı yeni bir kör noktadır.

şeffaf tutun: agent’ın planlama adımlarını, yürütme günlüklerini ve karar trajectory’sini açıkça gösterin. bu yalnızca bir hata ayıklama kolaylığı değildir, kullanıcı güveninin ön koşuludur. kara kutu içindeki bir hata dışarıdan ne bulunabilir ne de düzeltilebilir

iyi bir tool arayüzü tasarlayın (aci, agent-computer interface): aci, arayüzü geleneksel api’lerde olduğu gibi programcının bakış açısından değil, agent’ın bakış açısından — agent’ın anlaması ve kullanması kolay olacak şekilde — tasarlamak demektir. tool adları ve parametreleri sezgisel olmalı, yanlış kullanım ihtimali varsa, tasarım hatayı baştan imkansız kılmalıdır. sim kartın çentikli köşesi karta tepsiye yalnızca tek yönde girme olanağı verir ve mikrodalga fırın kapağı açıkken ısıtmayı reddeder. imalat sektöründe bu **“hataları tasarımla ortadan kaldırma”** felsefesinin bir adı vardır: toyota üretim sisteminden gelen **poka-yoke.** kötü tasarlanmış bir tool, en güçlü modeli bile sürekli hata yapmaya iter — arayüz, model ile tool arasındaki tek kanaldır ve belirsiz bir arayüz model tarafından sistemik bir hataya dönüştürülerek büyütülür. 

## 1.2.3 - bir model nasıl seçilir

model, agent’ın zeka altyapısıdır ve doğru olanı seçmek çoğu zaman herhangi bir prompt ince ayarından daha etkilidir. modeller belirli sürüm önerilerinin geçerliliğini koruyamayacak kadar hızlı yineleniyor, bu yüzden bu bölüm öneri yerine yönler sunuyor

**kapalı kaynak modeller.** günümüz agent geliştirmesinde en yaygın kullanılan iki kapalı kaynak model sağlayıcısı openai ve anthropictir. kapalı kaynak modeller genelde yetenek bakımından öndedir ancak daha pahalıdır ve sağlayıcının api politikalarıyla sınırlıdır. bir model seçerken yalnızca leaderboard’a bakmayın, kendi görevleriniz üzerinde değerlendirin.

**açık kaynak modeller.** bu kitap yazılırken açık ve kapalı kaynak modeller arasındaki fark altı aydan azdı, buna karşılık açık kaynak modellerin maliyeti belirgin ölçüde daha düşüktü. iş senaryonuz en yüksek model yeteneklerini gerektirmiyorsa açık kaynak model pragmatik bir seçimdir. daha düşük maliyetlidir, özel dağıtımı destekler ve fine-tuning ile özelleştirilebilir. bu da onları maliyete duyarlı veya veri uyumluluğu gerektiren senaryolara uygun kılar. deepseek, kimi ve glm agent yetenekleri güçlü çin modelleridir. modellerin tool calling yetenekleri önemli ölçüde farklılık gösterdiğinden, karar vermeden önce kendi senaryonuzda test edin.

**yeteneğin ötesinde, modelin politika sınırlarını da hesaba katın.** bir modelin bir görevi teknik olarak yapabilmesi, onu barındıran ürünün kullanıcının bu yeteneği kullanmasına izin vereceği anlamına gelmez. Tedarikçiler siber güvenlik, model damıtma, model çıkarımı, özel veriler ve yüksek riskli işlemler için farklı politika sınırları çizer; aynı görev bir chat ürünü, Coding Agent ve API’de farklı sonuçlar da verebilir. Bu nedenle model seçimi yalnızca doğruluk, fiyat ve hızı karşılaştıramaz. Gerçek görevlerinizde modelin ilerlemeye istekli olup olmadığını, arayüzün gerekli yeteneği sunup sunmadığını ve hizmet koşullarının amaçlanan kullanıma izin verip vermediğini test edin. İş açısından kritik görevler için insana devretme veya başka bir uyumlu modeli yedek yol olarak önceden hazırlayın.

**agent’ların büyük çoğunluğu reasoning destekleyen bir model gerektirir.** agent’lar karmaşık kararlar alır—çok adımlı reasoning, tool seçimi—ve reasoning’i olmayan modeller bunları genellikle kötü yapar. İstisnalar azdır: tek bir basit adım, veya sabit bir konuma tıklamaktan ibaret Computer Use GUI işlemleri; bu durumlarda reasoning yapmayan bir model idare edebilir. Çok adımlı reasoning veya dinamik karar alma devreye girdiği anda, bir reasoning modeli şarttır.

**çıktı hızına ve çok modlu yeteneklere dikkat edin. m**aliyetin ötesinde, gözden kaçması kolay iki boyut vardır. Biri çıktı token hızıdır: Agent’lar tipik olarak çok sayıda çıkarım turu çalıştırır ve her tur bir sonraki başlamadan önce bitmelidir, bu yüzden çıktı hızı uçtan uca gecikmeyi doğrudan belirler—her turda 2 saniye daha yavaş çalışan 20 turluk bir Agent görevi, ekstra 40 saniyelik bir bekleme anlamına gelir. Diğeri ise çok modlu (multimodal) destektir: Agent’ınızın görüntüleri, sesi veya videoyu anlaması gerekiyorsa, multimodal yetenek zorunlu bir gereksinimdir ve modeller bu konuda büyük farklılıklar gösterir.

Orkestrasyon kalıpları, bir Harness’in LLM çağrıları, context ve tools arasındaki akışı nasıl yönettiğini belirler. Bu kalıplar basitten karmaşığa doğru **workflow** ve **autonomous Agent** gibi yapılara ayrılır. En iyi yaklaşım, gereksiz yere karmaşık Agent sistemleri kurmamak; önce tek bir LLM çağrısını, ardından sabit alt görevlere ayrılabilen işler için workflow’u kullanmaktır. Yalnızca dinamik kararlar ve esnek yürütme yolları gerektiğinde autonomous Agent tercih edilmelidir. Agent sistemleri daha iyi performans sağlayabilse de genellikle daha yüksek gecikme ve maliyet yaratır.

**Workflow**, LLM’leri ve tool’ları geliştirici tarafından önceden belirlenmiş, deterministik kod yollarıyla çalıştırır. Örneğin uçuş rezervasyonunda kimlik doğrulama, uçuş arama, ödeme ve rezervasyon onayı gibi adımların sırası kodla sabitlenir. Böylece kritik iş kuralları korunur, adımların atlanması önlenir ve prompt injection veya model hatalarının sistemi beklenmeyen dallara yönlendirmesi zorlaşır.

Buna karşılık workflow’ların esnekliği sınırlıdır; önceden öngörülmemiş durumlara yalnızca tanımlanmış istisna yollarıyla veya insan müdahalesiyle yanıt verebilirler. Metinden görüntüye üretimdeki prompt yeniden yazımı buna örnektir: LLM, kullanıcının doğal dildeki isteğini görüntü modelinin anlayacağı biçime dönüştürür, ardından görüntü üretilir. Bu tür ara katmanlar, bir modelin veya aracın yetenek açığını kapatan **uyarlama katmanlarıdır**. Ancak yerel görüntü üretme ve doğal dili anlama yeteneğine sahip çok modlu modeller kullanıldığında bu ara adıma gerek kalmayabilir.  
  

**Sistem tasarımında "basitten karmaşığa" gitme stratejisi nedir ve hangi durumlarda bir Agent sistemine başvurulmalıdır?** Strateji, sorunu çözmek için en basit yöntemden başlayıp ancak ihtiyaç duyuldukça karmaşıklığı artırmaktır. Önce tek bir LLM çağrısı denenmeli, eğer sorun çözülmüyorsa ve görevler sabit alt görevlere ayrılabiliyorsa "workflow" kullanılmalıdır. "Autonomous Agent" sistemine ise yalnızca dinamik kararlara ve esnek bir yürütme yoluna ihtiyaç duyulduğunda başvurulmalıdır.

1. **Workflow (İş Akışı) kalıbını "deterministik" kılan temel özellik nedir?** Yürütme yolunun geliştirici tarafından önceden tasarlanmış ve kodla sabitlenmiş olmasıdır. Her adımın ne yapacağı ve bir sonraki adımın ne olacağı önceden bellidir; LLM'ler sadece düğümlerin içindeki işlemi yönetir, akışın sırasını veya yönünü değiştiremezler.
    
2. **Workflow kalıbının sunduğu güvenlik avantajı, otonom sistemlere kıyasla nasıl bir fark yaratır?** Yol deterministik olduğu için, olası bir prompt injection veya model hatası durumunda sistem kontrolsüz bir şekilde başka dallara sıçrayamaz. Saldırı yüzeyi sadece ilgili düğümle sınırlı kalır, bu da sistemin genel güvenliğini ve öngörülebilirliğini artırır.
    
3. **Bir Workflow sisteminin en temel sınırlaması nedir ve bu sınırlama karşılaşıldığında sistem nasıl tepki verir?**En temel sınırlaması esneklik eksikliğidir. Akışın öngörmediği bir durumla karşılaşıldığında (örneğin kullanıcının işlem sırasında fikrini değiştirmesi), sabit yol buna uyum sağlayamaz. Bu durumda sistem ya önceden tanımlanmış bir istisna dalını izler ya da kontrolü tamamen bir insana devreder.
    
4. **Metindeki "metinden görüntüye üretim" örneğinde, LLM düğümü neden bir "uyarlama katmanı" (adaptation layer) olarak tanımlanmıştır ve bu katman hangi durumda gereksiz hale gelir?** Çünkü görüntü üretim modelleri genellikle doğal insan dilini değil, belirli formatta (virgülle ayrılmış etiketler vb.) prompt'ları kabul eder. LLM burada insan dilini aracın anlayabileceği formata çevirerek modelin "yetenek açığını" yamadığı için bir uyarlama katmanı görevi görür. Eğer kullanılan model yerli (native) görüntü üretimi yeteneğine sahip çok modlu bir modelse (örneğin Nano Banana 2), bu çeviriye gerek kalmaz ve katman gereksizleşir.
    

---

Metin, **Autonomous Agent’ların**, workflow’ların sabit ve önceden tanımlı yollarının yetersiz kaldığı açık uçlu görevlerde kullanılan dinamik sistemler olduğunu açıklamaktadır. Autonomous Agent yürütme adımlarını önceden bilmez; ortamdan aldığı geri bildirimlere göre plan yapar, tool’ları kullanır, hataları fark eder ve gerektiğinde stratejisini değiştirir. Bu yapı, tool’ların döngü içinde kullanıldığı **ReAct yaklaşımına** dayanır.

Otonom sistemlerde görev tamamlanması, maksimum yineleme sayısı veya kurtarılamaz hata gibi açık durdurma koşulları bulunmalıdır; aksi hâlde sonsuz döngü ve gereksiz maliyet oluşabilir. Kodlama Agent’ları, bilgisayar kullanımı ve araştırma görevleri bu yapıya uygundur. Ancak otonomi; maliyet, hata birikimi ve güvenlik risklerini artırdığı için sandbox testleri, guardrail’ler, izleme ve kritik noktalarda insan denetimi gerektirir.

Workflow ve Autonomous Agent’lar birbirinin alternatifi olmak zorunda değildir. Kritik ve uyumluluk gerektiren bölümler workflow ile, esnek karar gerektiren bölümler autonomous Agent ile yürütülebilir. n8n gibi platformlar bu iki yaklaşımı aynı sistemde birleştirebilir.

Metin ayrıca çeşitli Agent çerçevelerini karşılaştırır: Codex Harness ve Claude Agent SDK otonom görevlere, LangChain/LangGraph her iki orkestrasyon biçimine, n8n ve Dify düşük kodlu uygulamalara, CrewAI ise rol tabanlı multi-agent iş birliğine odaklanır.

1. **Autonomous Agent ile workflow arasındaki temel fark nedir?**  
    Workflow’un yürütme yolu önceden tanımlanmış ve sabittir. Autonomous Agent ise ortamdan aldığı geri bildirimlere göre yürütme adımlarını gerçek zamanlı olarak belirler.
    
2. **Autonomous Agent’ın görev sırasında kendi yürütme adımlarını seçebilmesi neden önemlidir?**  
    Çünkü açık uçlu görevlerde tüm olası adımları önceden tahmin etmek mümkün değildir. Agent, karşılaştığı duruma göre plan yapabilir, tool seçebilir, hata durumunda stratejisini değiştirebilir ve görevi ilerletebilir.
    
3. **Otonom sistemlerde sonsuz döngüleri ve aşırı yürütmeyi önlemek için hangi durdurma koşulları tasarlanmalıdır?**  
    Görevin tamamlanması, maksimum yineleme sayısına ulaşılması veya kurtarılamaz bir hatayla karşılaşılması gibi açık durdurma koşulları belirlenmelidir.
    
4. **Autonomous Agent’lar hangi tür açık uçlu problemlerde ve uygulama senaryolarında daha uygundur?**  
    Gereken adım sayısının önceden tahmin edilemediği açık uçlu problemlerde uygundur. Kodlama Agent’ları, bilgisayar arayüzlerini kullanan “Computer Use” Agent’ları ve yinelemeli araştırma-gerektiren analiz görevleri buna örnektir.
    
5. **Workflow ve Autonomous Agent kalıpları aynı sistem içinde nasıl birlikte kullanılabilir?**  
    Sıkı kurallar ve uyumluluk gerektiren kritik süreçler workflow olarak yürütülürken, esnek karar alma gereken bölümler autonomous Agent’a bırakılabilir. Böylece sistem hem güvenilirlik hem de esneklik kazanır.
    

---

codex openai’in coding agent ürünüdür, codex harness ise bu biçimlerin hepsini çalıştıran çalışma zamanı katmanıdır (runtime layer). codex harness üç entegrasyon yolu sunar: `codex exec` betiklerdeki ve ci’daki tek seferlik işler için uygundur; `Codex SDK` , görevleri başlatan, sürdüren ve akış halinde işleyen üçüncü taraf uygulama kodu için uygundur; `app server` ise json-rpc protokolü üzerinden kalıcı oturumlar, olay akışları ve onay geri çağırmaları sağladığından agent’ı doğrudan ürünün içine koymaya uygundur. claude sdk agent ile claude code arasında da benzer ilişki vardır; fark şu ki claude tarafında dışarıya açılan sdk arayüzüdür, harness’ın uygulamasının kendisi açık kaynak değildir.

agent frameworkler hızla gelişir. bu yüzden belirli bir framework’un apisini öğrenmek önemli değil. **seçimde asıl ölçüt framework cmplxty değil, iş mantığına odaklanmanızı sağlayacak kadar ince bir soyutlama katmanı sunup sunmadığıdır.**

orkestrasyon kalıpları, harness içindeki context ve tools’un org’unu çözer. llm çağrılarının, tool’ların ve akışlarının nasıl bağlandığını. ama işi yapmak yeterli değildir, aynı zamanda doğru ve safe yapılması gerekir. bu yüzden şimdi constrain, verify ve correct mekanizmalarının pratikte nasıl hayata geçtiğinin başlıca yolu olan guadrail’lere bakalım.

## 1.2.5 guardrail’ler ve güvenlik

guardrail’ler harness’in constrain, verify ve correct katmanının başlıca uygulama biçimidir. agent davranışını güvenli ve kontrol edilebilir tutan katmanlı bir savunma. iyi tasarlanmış guardrail’ler, veri gizliliği risklerini ve itibar risklerini yönetmeye yardımcı olur. **zaten belirlediğiniz risklere yönelik guardrail’lerle başlayın, yeni zafiyetler ortaya çıktıkça yenilerini ekleyin.**

guardrail’leri derinlemesine savunma olarak düşünün. tek başına hiçbir gr yeterince koruma sağlamaz, ancak uzmanlaşmış birkaçının birleşimi çok daha dayanıklı agent sistemi yaratır.

gr’lerin başka bir hata biçimi daha vardır: yanlış ret. tehlikeli istekleri geçirme olasılığını azaltmaya çalışırken model, yetkili güvenlik testleri, model damıtma araştıramsı gibi meşru fakat hassas görünen işleri de reddedebilir. bu nedenle gr değerlendirmesi sadece yasak isteklerin engellenip engellenmediğini değil, açıkça izin verilen isteklerin hala tammalanıp tamamlanmadığını da sınamalıdır.

### 1.2.5.1 gr türleri

konumlandırıldıkları yerlere göre gr üç katmana ayrılır: context katmanı, yürütme katmanı ve veri katmanı. bu üç katman isteğin işlenme sırasına göre değil, atlatılmalarının ne kadar zor olduğuna göre sıralanmıştır. katman ne kadar aşağıdaysa modelin kendi yargısına o kadar az bağlıdır, dolayısıyla tek bir başarılı saldırıyla delinmesi o kadar güçtür. 

Context katmanı guardrail’leri modelin neyi görebileceğini yönetir ve içeriği context’e girmeden önce keser. Genellikle dört mekanizmadan oluşur. **İlgililik sınıflandırıcısı** konu dışı sorguları işaretler; örneğin bir kodlama asistanına “Empire State binası kaç metre?” diye sorulması. **Güvenlik sınıflandırıcısı** jailbreak’i (Jailbreak, modeli güvenlik sınırlarını aşmaya yöneltmek) ve prompt injection’ı (Prompt Injection, girdiye kötü niyetli talimat gömmek) tespit eder; aradaki temel fark, jailbreak’te kullanıcının kendisinin modelin güvenlik sınırlarını aşmaya çalışması, prompt injection’da ise saldırganın web sayfası veya belge gibi dış veriler üzerinden modelin davranışını dolaylı olarak yönlendirmesidir. **İçerik denetimi** şiddet veya ayrımcılık içeren zararlı ya da uygunsuz girdileri işaretler. **Kural tabanlı koruma** ise kara listeler, girdi uzunluğu sınırları ve düzenli ifade filtreleri gibi belirlenimci önlemlerle SQL enjeksiyonu gibi bilinen tehditleri savuşturur.

yürütme katmanı gr’leri modelin ne yapabileceğini yönetir ve eylem gerçekten etkili olmadan önce doğrular. çekirdeği tool risk derecelendirmesidir: her araca, işlemin geri alınabilirliğine, yetki düzeyine ve mali etkisine göre bir risk derecesi verilir; yüksek riskli işlemler ek inceleme ya da insan onayı gerektirir. kritik nokta, bu incelemenin context’in dışındaki bir mekanizmayla yapılması gerektiğidir. bağımsız bir inceleme süreci, en az ayrıcalıklı kimlik bilgileri, sandbox yalıtımı, döngüdeki insan, uğramış agent’la birlikte düşer. kullanıcıya döndürülen yanıtın kendisi de bir eylemdir. 

veri katmanı guardrail’leri dünyanın nihayetinde neye dönüştürülebileceğini yönetir ve kimin hangi veriye ne yapabileceği kararını istikrarlı, insan denetiminden geçmiş bir mekanizmaya bırakır. veritabanının satır düzeyi güvenlik politikaları, kısıtlar ve doğrulayıcılar, denetimli görünümler ve saklı yordamlar, ayrıca güvenilir çalışma zamanının bağladığı, taklit edilemeyen bir erişim context’i. Bu katmanın değeri tam da üstteki iki katmanın doğruluğuna bağlı olmamasındadır: prompt injection tutsa ve üretilen kod yetki denetimini tümüyle atlasa bile, yetki aşan işlem yine veri katmanında reddedilir. Bölüm 5 bu katmanı dinamik üretilen yazılım örneğiyle açar.

### 1.2.5.2 insan müdahalesi

Human in the loop (sürece insan dahil etme), temel bir koruyucu önlemdir: bir Agent’ın kullanıcı deneyimini bozmadan gerçek dünya performansını iyileştirmesini sağlar. En çok erken dağıtımda önem taşır; başarısızlık modlarını belirlemeye, uç durumları (edge case) ortaya çıkarmaya ve sağlam bir değerlendirme döngüsü kurmaya yardımcı olur.  
Bir human-in-the-loop mekanizmasıyla, bir görevi tamamlayamayan Agent kontrolü zarif bir şekilde devredebilir. Müşteri hizmetlerinde bu, bir insan temsilciye yükseltme anlamına gelir; bir Kodlama Agent’ı için ise kontrolü geliştiriciye geri vermek anlamına gelir.  
İnsan müdahalesini tetikleyen tipik olarak iki ana durum vardır:  
Başarısızlık Eşiklerinin Aşılması Agent’ın yeniden deneme ve işlem sayısına bir üst sınır koyun. Agent bu sınırları aşarsa, bir insana yükseltin.  
Yüksek Riskli İşlemler Hassas, geri alınamaz veya yüksek riskli işlemler—en azından ekip Agent’ın güvenilirliğine yeterli güven inşa edene kadar—insan gözetimini tetiklemelidir. Tipik örnekler: büyük bir iadeyi yetkilendirmek, bir ödemeyi işlemek.

## 1.2.6 harness’ın beş ögesi ve inşa bölümü

önce iki formülün ilişkisini netleştirelim ki kimse iki iskelet ezberlemek zorunda kalmasın.

agent = llm + context + tools

agent = model + harness

---

# 1.4 bölüm özeti

Bu bölüm, pratikten başlayarak AI Agent’ları anlamak ve inşa etmek için temel çerçeveyi kurdu.

  
**Agent = Beyin + Gözler + El ve Ayaklar:** LLM beyindir (karar alma çekirdeği), context gözlerdir (neyi görebildiğini belirler) ve tools el ve ayaklardır (neyi yapabildiğini belirler). Üçünden hiçbiri vazgeçilmez değildir.

**Context ve Tools’u Genişletmek Birincil Yetenek Kaldıracıdır:** Model sabitken Observation ve Action Space’i yeniden tanımlamak veya büyütmek—yani Context’i ve Tools’u genişletmek—çoğu zaman çözülemeyen bir görevi doğrudan çözülebilir hâle getirebilir. Manus’tan OpenClaw’a uzanan evrim, genelliğin büyük ölçüde arayüz sınırını genişletmekten geldiğini gösterir; bu genişleme ihtiyaç anında yapılmalı, izinler ve doğrulamayla eşleştirilmelidir.

  
**Gözler (Context) Belirleyici Faktördür:** Context, bir static prefix’ten (system prompt + tool tanımları) ve dinamik bir trajectory’den (mesaj geçmişi) oluşur. Ablation, herhangi bir bileşenin kaldırılmasının sistemi belirgin biçimde kötüleştirdiğini gösterir. ReAct döngüsünün özü, modelin görevi ilerletmeye devam etmesi için trajectory’ye tekrar tekrar ekleme yapmaktır.

  
**Harness Rekabet Avantajıdır:** Model yeteneği metalaşıyor; gerçek ayırt edici faktör Harness’tir—context ve tools’un etrafında inşa edilmiş, bir Agent’ın “işi güvenilir biçimde yapmasını” sağlayan constrain, verify ve correct mekanizmalarıdır. Üretim düzeyindeki Agent sistemlerinde, Harness kodunun büyük çoğunluğu yalnızca context ve tools’a değil bu güvenlik önlemlerine gider.

**Workflow’dan Autonomous Agent’a:** Önce prompt’lar, sonra workflow’lar, en son autonomous Agent’lar—bu sıralama, sürpriz “riskini düşük tutmanın en pratik yoludur. Her orkestrasyon kalıbının kendi doğal alanı vardır; hiçbiri her yerde en iyisi değildir.

  
**Beş tasarım örüntüsü kitap boyunca tekrarlanır:** Öneren-İnceleyen, aşamalı açıklama, yalnızca ekle, sınır kümesi + koruma kümesi ve en küçük diff + geri alınabilirlik.

  
**Güvenlik mimari bir meseledir:** Lansmandan önce yamalanacak bir şey değil, kodun ilk satırından itibaren düşünülmelidir. Guardrail’ler aşılma zorluğuna göre context, yürütme ve veri katmanlarına ayrılır; sonraki güvenlik tartışmalarının tümü bu iskeleti izler.

  
Sonraki bölüm, Harness’in en merkezi bileşenine—context engineering’e—derinlemesine iner. Agent kavramının pekiştirmeli öğrenmedeki akademik köklerine ve geleneksel RL ile modern LLM Agent’larının daha kapsamlı bir karşılaştırmasına gelince, Bölüm 8 ikisini de sistematik olarak ele alır.