# Context engineering

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter2/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

# context engineering

Agent yalnızca gördüğü bilgilere dayanarak karar verebilir. Context’i tasarlama ve yönetme işine Context Engineering denir. Buna yalnızca konuşma geçmişi değil, geliştiricinin önceden yazdığı davranış kuralları (sistem talimatları), yapay zekanın kullanabileceği dış yeteneklerin açıklamaları (tool açıklamaları) ve başka bilgiler de dahildir.

**agent’ın her karar noktasında hangi bilgiyi, hangi yapıda gördüğünü belirler. iyi tasarlanmış bir context, agent’ın genel reasoning yeteneğini somut bir görevde tam olarak kullanabilmesini sağlayan verimli bir bilgi tedarik sistemidir.**

## 2.1 Context: Agent Yeteneklerinin Üst Sınırını Belirleyen Anahtar 

Ekibinize dahil olan bir deha mühendisi hayal edin. Derin teorik bilgiye ve olağanüstü programlama becerisine sahiptir, ama ürün mimariniz, iş mantığınız, teknik borcunuz veya ekip normlarınız hakkında hiçbir şey bilmez. Daha da kötüsü, kritik mimari kararlar farklı ekip üyelerinin memory’sine dağılmıştır ve kod tabanında dokümantasyon yoktur. Olağanüstü zekaya sahip olsa bile, bu deha gerçek değer sunmakta zorlanır—bu tam olarak günümüz AI Agent’ların karşılaştığı ikilemdir.

Bir Kodlama Agent’ını örnek alalım. Aynı talimat verildiğinde, “Bu hatayı düzeltmeme yardım et,” Agent’ın aldığı context’in kalitesi görevi tamamlayıp tamamlayamayacağını doğrudan belirler:

Gerçek zamanlı kod context’i: Mevcut kod tabanının dizin yapısı, her modülün sorumlulukları, temel veri yapılarının tanımları ve ekibin kodlama standartları. Bunlar olmadan, Agent’ın yazdığı kod sözdizimsel olarak doğru ama projeyle stilistik olarak tutarsız olabilir, hatta mimari çatışmalar yaratabilir.

  
Süreç şartnameleri: Git dallanma stratejisi, kod commit kuralları, kod inceleme süreci, CI/CD pipeline gereksinimleri. Bunlar eksikse, Agent test edilmemiş kodu doğrudan ana dala commit edebilir.

  
Ortam bilgisi: Geliştirme ortamı yapılandırması, test veritabanı bağlantı adresleri, test ortamına dağıtım yöntemleri ve API key yönetim prosedürleri. Bunlar olmadan, yerel olarak çalışan bir düzeltme test ortamında hemen bozulabilir.

---

Bu üç bilgi türü—kod, süreç ve ortam—bir Agent’ın etkili çalışması için gereken asgari bilgiyi oluşturur. Burada context’e giren şey Ortam’ın kendisi değil, Ortam’a ilişkin gözlemler, açıklamalar veya yapılandırmadır; Ortam hâlâ Agent’ın dışarıda etkileşim kurduğu nesnedir. Modelin doğuştan gelen yeteneği yalnızca temeldir; context’in kalitesi Agent yeteneğinin gerçek anahtarıdır. Dikkatle organize edilmiş bir context ile eşleştirilmiş orta düzeyde yetenekli bir model, yetersiz bilgiyle körlemesine ilerleyen üst düzey bir modelden çoğu zaman daha iyi performans gösterebilir.

**Context engineering, bu yüzden mevcut modelleri kullanarak verimli Agent’lar geliştirmenin anahtarıdır. Bu, salt bir prompt’a daha fazla bilgi tıkıştırmaktan ibaret teknik bir mesele değildir; yapay zekanın bir görevi tamamlamak için ihtiyaç duyduğu tüm arka plan bilgisini sistematik olarak tasarlamayı, organize etmeyi ve sağlamayı içerir.**

Üstelik bu yalnızca teknik bir sorun değil, daha çok bir örgütsel sorundur. Çoğu ekipte kilit bilgi örtüktür: mimari kararları yalnızca kıdemli çalışanlar hatırlar, iş kuralları ağızdan ağıza aktarılır, önemli arka plan bilgisi özel sohbet kayıtlarına kilitlenmiştir. Ekibin kendisi bir bilgi kara deliğiyse, en iyi AI Agent bile çaresiz kalır.

Uzaktan çalışmada etkili olan ekipler genellikle AI Agent’lar için de etkili ortamlar sunar. Linux çekirdeği gibi açık kaynak projeler mükemmel örneklerdir: dünya genelinde dağılmış geliştiriciler otuz yılı aşkın süredir projeyi birlikte sürdürüyor. Başarısının sırrı, son derece şeffaf ve dokümantasyon odaklı bir iletişim kültürüdür—tüm tartışmalar herkese açıktır, her karar kaydedilir ve herhangi bir yeni gelen, geçmişi okuyarak kodun evrimini anlayabilir. Bu çalışma tarzı doğal olarak yapay zeka dostu bir ortam yaratır: bilgi herkese açıktır, aranabilir ve yapılandırılmıştır.

OpenAI araştırmacısı Jiayi Weng bunu çarpıcı biçimde ifade etmişti: “Hem insanlar hem de modeller için en önemli şey Context’tir.” Kendi deneyimini örnek gösterdi—“OpenAI’deki işim o kadar zor değil. Başka biri tüm context’ime sahip olsaydı, o da yapabilirdi.” Aynı ilke Agent’lar için de geçerlidir: bir Agent’ın iş dünyasında sunduğu değer çoğu zaman model boyutundan değil, her karar noktasında sağlanan context’in eksiksizliği ve isabetinden kaynaklanır. Weng ayrıca “ekip çalışmasındaki en büyük sorunun da context tutarsızlığı olduğunu” ve “yapay zekanın kısa vadede insanların yerini alamamasının en büyük nedeninin de context olduğunu—çünkü yapay zeka ve insanlar aynı ortamda değildir” belirtti. Bu, context engineering’in çözmeyi amaçladığı temel sorundur: bir Agent’ın ihtiyaç duyduğu yapılandırılmış arka plan bilgisini modele sistematik biçimde iletmek.

## 2.2 Agent’lar Büyük Modelleri Nasıl Çağırır: API’nin Context Yapısını Anlamak

Bu bölüm, bir Agent’ın bir büyük modeli her çağırışında oluşan eksiksiz istek bileşimini ayrıntılı olarak ele almak için OpenAI’nin Chat Completions API’sini örnek alır (Anthropic, Google ve diğer sağlayıcıların API yapıları büyük ölçüde benzerdir). Bu yapıyı anlamak, sonraki tüm context engineering tekniklerinde ustalaşmanın temelidir.

### 2.2.1 mesajların dört rolü

bir büyük model api’sinin özü bir mesaj listesidir (messages). listedeki her mesajın bir role (rol) tanımlayıcısı vardır ve model, her mesajın anlamını ve kaynağını rolüne göre anlar. 

**system**: Sistem talimatı (system prompt). Geliştirici tarafından yazılır, Agent’ın kimliğini, davranış kurallarını ve kısıtlamalarını tanımlar. Model bunu en yüksek öncelikli talimat olarak ele alır. Tüm konuşma boyunca genellikle yalnızca bir system mesajı bulunur ve mesaj listesinin en başına yerleştirilir.

  
**user**: Kullanıcı mesajı. Son kullanıcıdan gelen girdi, Agent’ın yanıtlaması gereken isteği temsil eder.

  
**assistant**: Asistan mesajı. Modelin önceki yanıtları, metin yanıtlarını ve tool çağrısı isteklerini içerir. Çok turlu konuşmalarda, önceki asistan mesajları mesaj listesine geri konur ve bu, modelin ne söylediğini “hatırlamasını” sağlar.

  
**tool**: Tool sonucu. Agent çerçevesi bir aracı çalıştırdıktan sonra, sonuç tool rolüne sahip bir mesaj olarak modele geri gönderilir. Her tool mesajı, tool_call_id aracılığıyla ilgili tool çağrısı isteğiyle ilişkilendirilir.

Bu, Bölüm 1’de “context’in beş bileşeni” olarak tanıtılan aynı API istek yapısının farklı bir açıdan sınıflandırılmasıdır: `system`, `user`, `assistant` ve `tool` olmak üzere dört mesaj rolü sırasıyla system prompt’a, kullanıcı mesajlarına, asistan mesajlarına ve tool sonuçlarına karşılık gelir. Geriye kalan bileşen olan tool tanımları, bir mesaj rolü olarak değil isteğin üst düzey `tools` alanı üzerinden iletilir. Dolayısıyla “dört mesaj rolü + `tools`alanı”, Bölüm 1’deki beş context bileşenini tam olarak kapsar.

### 2.2.2 tek turlu diyalog: en basit api çağrısı

request (agent framework tarafından oluşturulur): system → geliştiricinin yazdığı kurallar, user → “merhaba sen kimsin?”

çağrı yapılır

response (api tarafından döndürülür): assistant → modelin ürettiğ iyanıt, “merhaba! ben bir kodlama asistanıyım…”

**her çağrı stateless’tır, gereken tüm bilgiler request içindeki messages listesinde eksiksiz verilmelidir**

**önce tool çağrısı içermeyen en basit senaryoya bakalım. kullanıcı “merhaba sen kimsin” diye sorar. burada yerel olarak çalıştırılan küçük bir qwen3-0.6b modelini örnek alıyoruz.**

```
// ═══ Agent çerçevesi tarafından oluşturulan istek ═══
{
  "model": "Qwen3-0.6B",
  "messages": [
    {
      "role": "system",                           // ← Geliştirici tarafından yazıldı
      "content": "Sen yardımsever bir kodlama asistanısın. Kullanıcı talimatlarını izle."
    },
    {
      "role": "user",                              // ← Kullanıcı girdisi
      "content": "Merhaba, sen kimsin?"
    }
  ]
}
// ═══ API tarafından döndürülen yanıt ═══
{
  "choices": [{
    "message": {
      "role": "assistant",                         // ← Model tarafından üretildi
      "content": "Merhaba! Ben bir kodlama asistanıyım. Kod yazmanıza, hataları ayıklamanıza ve teknik kavramları açıklamanıza yardımcı olabilirim. Nasıl yardımcı olabilirim?"
    }
  }]
}
```

Bu istek yalnızca iki mesaj içerir: bir system (geliştirici tarafından yazılan kurallar) ve bir user (kullanıcının girdisi). Model, yanıt olarak bir assistant mesajı döndürür. Bu, LLM API’sinin en temel etkileşim kalıbıdır — her çağrı durumsuzdur (stateless); modelin ihtiyaç duyduğu tüm bilgi isteğin mesaj listesinde eksiksiz olarak sağlanmalıdır.

### 2.2.3 tool çağrılarıyla çok turlu etkileşim: bir agent’ın temel döngüsü

gerçek bir agent senaryosu, tek turlu bir soru-cevaptan çok daha karmaşıktır. bir kullanıcı “vancouver’da şu anki saat ve hava durumu nedir” diye sorduğunda, model kendi bilgisinden yanıt veremez. şimdinin ne zaman olduğunu, hele hava durumunu hiç bilmez. tool’lar gerekir.

birinci model api çağrısı

messages: system + user

tools: get_current_time, get_weather 

api →

assistant: tool_calls

get_current_time(timezone=”America/Vancouver”) 

get_weather(city=”Vancouver”, unit=”celsius”)

veri bağımlılığı yok → paralel çalıştırılabilir

agent çerçevesi iki aracı paralel çalıştırır

messages: + tool sonuçları

vancouver saati ve hava durumu

mesaj geçmişine ekle, sonra modeli yeniden çağır

api →

ikinci model api çağrısı, assistant: son yanıt

tool çağrısı yok → döngüyü bitir

“şu an …, hava …”

**stateless api’de tüm mesaj geçmişi her turda yeniden gönderilmelidir**

```
// ═══ Agent çerçevesi tarafından oluşturulan istek (1. çağrı) ═══
{
  "model": "Qwen3-0.6B",
  "messages": [
    {
      "role": "system",                           // ← Geliştirici tarafından yazıldı
      "content": "Sen yardımsever bir asistansın. Gerektiğinde gerçek zamanlı bilgi almak için sağlanan araçları kullan."
    },
    {
      "role": "user",                              // ← Kullanıcı girdisi
      "content": "Vancouver'da şu anki saat ve hava durumu nedir?"
    }
  ],
  "tools": [                                       // ← Geliştirici tarafından tanımlanan araçlar
    {
      "type": "function",
      "function": {
        "name": "get_current_time",
        "description": "Get the current date and time in a specific timezone",
        "parameters": {
          "type": "object",
          "properties": {
            "timezone": { "type": "string", "description": "Timezone name, e.g. America/Vancouver" }
          }
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get the current weather for a specific city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": { "type": "string", "description": "City name" },
            "unit": { "type": "string", "enum": ["celsius", "fahrenheit"] }
          }
        }
      }
```

bu `tools` listesi, geliştiricinin önceden kaydettiği statik tool meta verisidir: tool adları, açıklamalar ve parametre şemaları kodda yazılmıştır ve kullanıcının bu seferki sorusuyla ilgisi yoktur. kullanıcı ister vancouver’ın hava durumunu sorsun ister agent’tan uçak bileti almasını istesin, gönderilen liste aynıdır. örnekte yalnızca ilgili iki aracın yer alması isteği kısa tutmak içindir, gerçek bir Agent ise çoğu zaman onlarca aracı aynı anda tanımlar. Agent, kullanıcı girdisini önce “saati sorgula” ve “hava durumunu sorgula” diye iki alt göreve ayırıp ardından bunlara uygun tool açıklamalarını üretmiş değildir; bu ayrıştırma modelin tarafında gerçekleşir ve tam olarak aşağıdaki yanıttaki tool_calls alanıdır.

Model bir tool çağrısı isteği döndürür (nihai yanıt değil):

```
// ═══ API tarafından döndürülen yanıt (model araç çağırmaya karar veriyor) ═══
{
  "choices": [{
    "message": {
      "role": "assistant",                         // ← Model tarafından üretildi
      "content": null,                             // Metin yanıtı yok
      "tool_calls": [                              // Model iki araç çağrısı istiyor
        {
          "id": "call_abc123",
          "type": "function",
          "function": {
 "content": "Vancouver'da şu anki saat ve hava durumu nedir?"
    }
  ],
  "tools": [                                       // ← Geliştirici tarafından tanımlanan araçlar
    {
      "type": "function",
      "function": {
        "name": "get_current_time",
        "description": "Get the current date and time in a specific timezone",
        "parameters": {
          "type": "object",
          "properties": {
            "timezone": { "type": "string", "description": "Timezone name, e.g. America/Vancouver" }
          }
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get the current weather for a specific city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": { "type": "string", "description": "City name" },
            "unit": { "type": "string", "enum": ["celsius", "fahrenheit"] }
          }
        }
      }
    }
  ]
}
```

Dikkat edin, model kullanıcının sorusunu doğrudan yanıtlamıyor. Bunun yerine iki tool çağrısı isteği döndürüyor—“şu anki saat” ve “hava durumu”nun tools aracılığıyla elde edilmesi gerektiğine karar veriyor ve aralarında bir bağımlılık olmadığından, paralel olarak çağrılabiliyorlar. Model yalnızca çağrı isteklerini verir; tool’ların gerçek yürütülmesi Agent çerçevesi tarafından yönetilir. Bu, Agent mimarisini anlamanın anahtarıdır: model karar almadan sorumludur (hangi aracın çağrılacağı, hangi parametrelerin geçileceği), Agent çerçevesi ise yürütmeyi yönetir (gerçekte API’leri çağırmak, kod çalıştırmak).  
Agent çerçevesi tool’ları çalıştırır ve ardından ikinci bir API çağrısı başlatır:  
Modelin tool çağrısı isteklerini aldıktan sonra, Agent çerçevesi iki aracı gerçekten çalıştırır (örn. saat API’sini ve hava durumu API’sini çağırarak) ve ardından eksiksiz konuşma geçmişini tool yürütme sonuçlarıyla birlikte modele geri gönderir:

```

// ═══ Agent çerçevesi tarafından oluşturulan istek (2. çağrı) ═══
{
  "model": "Qwen3-0.6B",
  "messages": [
    {
      "role": "system",                           // ← 1. çağrıyla aynı
      "content": "Sen yardımsever bir asistansın. Gerektiğinde gerçek zamanlı bilgi almak için sağlanan araçları kullan."
    },
    {
      "role": "user",                              // ← 1. çağrıyla aynı
      "content": "Vancouver'da şu anki saat ve hava durumu nedir?"
    },
    {
      "role": "assistant",                         // ← 1. çağrıdan model çıktısı, aynen dahil edildi
      "content": null,
      "tool_calls": [
        { "id": "call_abc123", "function": { "name": "get_current_time", "arguments": "{\"timezone\": \"America/Vancouver\"}" } },
        { "id": "call_def456", "function": { "name": "get_weather", "arguments": "{\"city\": \"Vancouver\", \"unit\": \"celsius\"}" } }
      ]
    },
    {
      "role": "tool",                              // ← Agent çerçevesi tarafından üretildi (araç yürütme sonucu)
      "tool_call_id": "call_abc123",
      "content": "{\"timezone\": \"America/Vancouver\", \"datetime\": \"2025-09-13T05:18:47\", \"day_of_week\": \"Saturday\"}"
    },
    {
      "role": "tool",                              // ← Agent çerçevesi tarafından üretildi (araç yürütme sonucu)
      "tool_call_id": "call_def456",
      "content": "{\"city\": \"Vancouver\", \"temperature\": 13.2, \"unit\": \"celsius\", \"conditions\": \"clear\", \"humidity\": 93}"
    }
  ],
  "tools": [ ... ]                                 // ← Yukarıdakiyle aynı araç tanımları, kısaltıldı
}
```

Burada üç önemli ayrıntı var:

İkinci istek, ilk istekten gelen tüm konuşma geçmişini içerir — system mesajı, user mesajı, ilk assistant yanıtı (tool çağrılarını içeren) ve yeni eklenen tool sonuçları. Bu, daha önce belirtilen “her çağrı durumsuzdur” ilkesidir. Model önceki konuşmayı “hatırlamaz”; Agent çerçevesi her seferinde eksiksiz geçmişi göndermelidir.  
İlk assistant mesajı mesaj listesine aynen geri konur — bu, modelin daha önce hangi kararları aldığını “görmesini” sağlar.  
Tool mesajları, tool_call_id aracılığıyla ilgili tool çağrılarına bağlanır — model bunu, hangi sonucun hangi çağrıya karşılık geldiğini bilmek için kullanır.

Model, tool sonuçlarına dayanarak nihai yanıtı üretir:

```
// ═══ API tarafından döndürülen yanıt (nihai yanıt) ═══
{
  "choices": [{
    "message": {
      "role": "assistant",                         // ← Model tarafından üretildi
      "content": "Vancouver'da şu anda Cumartesi, 13 Eylül 2025, saat 05:18.\n\nHava durumu: 13,2°C, açık ve %93 nem. Bu sabah oldukça serin - bir ceket almak isteyebilirsiniz."
```

Bu kez model tool_calls döndürmez, doğrudan metin yanıtı verir. Kullanıcının sorusunu yanıtlamak için yeterli bilgiye sahip olduğuna karar verdiğinden Agent yürütmeyi durdurur. Bu “istek → araç çağrısı → yürütme → sonuçları döndürme → yeniden istek” döngüsü, Bölüm 1’de tanıtılan ReAct döngüsünün API düzeyindeki somut uygulamasıdır.

Kullanıcı daha fazla bilgiye ihtiyaç duyarsa, örneğin “Ya Tokyo?” diye sorarsa, Agent çerçevesi takip sorusunu konuşma geçmişinin sonuna ekleyip model API’sini yeniden çağırır. Model tekrar tool_calls üretmeye başlar; çerçeve bunları yürütür, sonuçları geri gönderir ve döngü böyle devam eder.

### 2.2.4 Agent’ın Temel Döngüsünü Kod Olarak Uygulamak

Artık JSON yapısını anladığımıza göre, yukarıda açıklanan etkileşim sürecini bir araya getirmek için Python kodu kullanalım. Aşağıdaki, minimal bir Agent uygulamasıdır — özü basitçe bir while döngüsüdür: Bu bölüm, tam API döngüsünü bilinçli olarak bir protokol referansı olarak korur; diğer bölümler mekanizmaları Python tarzı iskelet kodla anlatır

```
.
from openai import OpenAI

client = OpenAI()

# ── Araç tanımları ──
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_current_time",
            "description": "Get the current date and time in a specific timezone",
            "parameters": {
                "type": "object",
                "properties": {
                    "timezone": {"type": "string", "description": "Timezone name, e.g. America/Vancouver"}
                },
            },
        },
    },
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a specific city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                },
            },
        },
    },
]

# ── Araç yürütme fonksiyonu (sabit sonuçlarla bir taslak; gerçek bir uygulama
#    JSON `arguments`'ı ayrıştırmalı ve gerçek API'leri çağırmalıdır) ──
def execute_tool(name, arguments):
    if name == "get_current_time":
        return '{"datetime": "2025-09-13T05:18:47", "day_of_week": "Saturday"}'
    elif name == "get_weather":
        return '{"temperature": 13.2, "unit": "celsius", "conditions": "clear", "humidity": 93}'

# ── Başlangıç mesaj listesi ──
messages = [
    {"role": "system", "content": "You are a helpful assistant. Use tools to get real-time information when needed."},
    {"role": "user", "content": "What's the current time and weather in Vancouver?"},
]

# ── Agent'ın temel döngüsü ──
# Üretim kodunda burada bir max_iterations üst sınırı gerekir: bu bölümde ilerde tartışıldığı gibi, Agent'lar aynı araç çağrılarını sonsuza kadar tekrarlayarak sıkışabilir
while True:
    response = client.chat.completions.create(
        model="Qwen3-0.6B", messages=messages, tools=tools
    )
    assistant_message = response.choices[0].message

    # Modelin yanıtını mesaj listesine ekle (metin veya araç çağrısı olsun)
    messages.append(assistant_message)

    # Araç çağrısı istenmediyse, model nihai yanıtını üretmiştir
    if not assistant_message.tool_calls:
        print(assistant_message.content)
        break

    # Modelin istediği her aracı çalıştır, sonuçları mesaj listesine ekle
    for tool_call in assistant_message.tool_calls:
        result = execute_tool(tool_call.function.name, tool_call.function.arguments)
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": result,
        })
    # Döngünün başına dön, güncellenmiş mesaj listesiyle modeli yeniden çağır
```

Bu kodun temel mantığı yalnızca tek bir while döngüsü ve tek bir koşuldur: model tool_calls döndürürse, tool’ları çalıştır ve döngüye devam et; döndürmezse, sonucu çıktı ver ve çık. Süreç boyunca messages listesi büyümeye devam eder — her tur, modelin yanıtını ve tool yürütme sonuçlarını ekler.

messages listesindeki değişiklikleri her tur boyunca izleyelim: 

**Başlangıç durumu (1. çağrıdan önce):**

```
messages = [
  { role: "system",  content: "Sen yardımsever bir asistansın..." },     # Geliştirici tarafından yazıldı
  { role: "user",    content: "Vancouver'da şu anki saat ve hava durumu nedir?" },  # Kullanıcı girdisi
]
```

**1.çağrıdan sonra (model tool çağrıları döndürür):** 

```
messages = [
  { role: "system",    content: "..." },
  { role: "user",      content: "Şu anki saat..." },
  { role: "assistant", tool_calls: [get_current_time, get_weather] },  # + Model tarafından üretildi
  { role: "tool",      tool_call_id: "call_abc", content: "{saat...}" },  # + Çerçeve tarafından çalıştırıldı
  { role: "tool",      tool_call_id: "call_def", content: "{hava...}" },  # + Çerçeve tarafından çalıştırıldı
]
```

2.çağrıdan sonra (model nihai yanıtı döndürür, döngü biter):

```
messages = [
  { role: "system",    content: "..." },
  { role: "user",      content: "Şu anki saat..." },
  { role: "assistant", tool_calls: [get_current_time, get_weather] },
  { role: "tool",      tool_call_id: "call_abc", content: "{saat...}" },
  { role: "tool",      tool_call_id: "call_def", content: "{hava...}" },
  { role: "assistant", content: "Vancouver'da şu anda Cumartesi, 13 Eylül 2025..." },  # + Nihai yanıt
]
```

Bu süreçten şu açıkça görülür: *****bir Agent çerçevesinin temel işi bu messages listesini yönetmektir*****—doğru zamanda mesaj eklemek, ardından tüm listeyi modele göndermek. Bu bölümdeki tüm context engineering teknikleri özünde bu listenin içeriğini ve yapısını optimize etmekle ilgilidir.

### 2.2.5 API Perspektifinden Context’in Bileşimi 

Yukarıdaki örnek aracılığıyla, Agent’ın modeli her çağırışında context’in eksiksiz bileşimini net biçimde görebiliyoruz:

statik örnek (turlar boyunca değişmez)

system prompt (sistem istemi), tool definitions (tool tanımları)

konuşma geçmişi / trajectory (etkileşimle büyür)

user → assistant → tool sonucu → user → …

**statik önek + trajectory: kv cache için önek sabit kalır, trajectory sıkıştırılabilir**

Üst kısım (System Prompt + Tool Tanımları) konuşma boyunca değişmeden kalırken, alt kısım (konuşma geçmişi, yani Bölüm 1’de tanımlanan trajectory) her etkileşimle sürekli büyür. Bu, Bölüm 1’deki “context’in beş bileşeni”nin API düzeyinde tam olarak nasıl göründüğüdür: system prompt ve tool tanımları statik bir ön ek (static prefix) oluştururken, kullanıcı mesajları, model yanıtları ve tool yürütme sonuçları dinamik olarak büyüyen bir mesaj geçmişi oluşturur. Bu “static prefix + trajectory” yapısı, KV Cache optimizasyonu, context sıkıştırma ve diğer teknikler üzerine sonraki tartışmaların temelidir—bu yapıyı anlamak, “ön kısım taşınamaz ama arka kısım sıkıştırılabilir” ilkesini açıklar

bu bölmün geri kalanı bu yapının her katmanını inceleyecek: static prefix’in değişmezliğinden çıkarımı hızlandırmak için nasıl yararlanılır (kv cache), iyi bir system prompt nasıl tasarlanır (prompt engineering), dış içeriğin context’i ele geçirmesi nasıl önlenir (prompt injection savunması), özelleşmiş bilgi ihtiyaç halinde nasıl yüklenir (agent skills), konuşmanın sonuna dinamik durum bilgisi nasıl enjekte edilir (agent durum çubuğu) ve konuşma geçmişi çok büyüdüğünde nasıl akıllıca sıkıştırılır (sıkıştırma stratejileri).

Sonraki tekniklerin adı çoktur, ama her istek öncesine indiğinde hepsi tek bir context kurma kararına iner. Aşağıdaki Python tarzı sözde kod, bu kararın en yalın iskeletini korur; yukarıdaki tam API döngüsünü context yerleşimini vurgulayarak tamamlar, mesaj rolleri ve tool_call_id gibi protokol ayrıntılarının yerini almaz.

```

stable_prefix = system_message
stable_tools = core_tool_schemas
trajectory = load_message_history(session)
status_message = make_status_message(derive_current_state(trajectory))

if estimated_tokens(stable_prefix, trajectory, status_message) > budget:
    trajectory = compress_old_evidence(
        trajectory,
        preserve = [decisions, constraints, failures, citations]
    )

request.messages = [stable_prefix] + trajectory + [status_message]
request.tools = stable_tools
response = call_model(request)
```

Sistem promptunu ve çekirdek tool tanımlarını olabildiğince kararlı tutun; eski tool çıktılarını yalnızca bütçeye yaklaşırken toplu olarak sıkıştırın; güncel durumu ise yörüngenin kuyruğuna koyun ki model onu uzun bir geçmişten yeniden türetmek zorunda kalmasın.

---

## 2.3 kv cache dostu context tasarımı

model her token ürettiğinde, kendisinden önceki her token’ın ara hesaplama sonuçlarına geri bakmak zorundadır. bunların tümünü her turda sıfırdan yeniden hesaplamak, context büyüdükçe maliyetin patlamasına yol açar. kv cache bunun yerine ara sonuçları önbelleğe alır, böylece her tur yalnızca yeni eklenen token’ları hesaplar. ön koşul, yeniden kullanılacak context token ön ekinin değişmeden kalmasıdır. token dizisi belirli bir konumdan itibaren farklılaşırsa, ilk farklı token’ın ve sonrasının kv durumları yeniden hesaplanmalıdır; bu konumdan önceki kv durumları değişiklikten etkilenmez.

Terminoloji notu: bu bölüm istekler arasındaki “cache isabetlerinden” bahsederken, API sağlayıcıları buna Prompt Cache der—çıkarım motorunun KV Cache’i üzerine inşa edilmiş istekler arası bir cache; bu iki düzey bu bölümün sonunda tam olarak birbirinden ayrılacak.

Bu sezgi yerleştiğinde, hikaye kendini anlatıyor. Bir ekibin müşteri hizmetleri Agent’ı günde 100.000 konuşmayı işliyordu ve her şey yolunda gidiyordu—ta ki bir gün bir mühendis, Agent’ın şu anki saati “bilmesini” isteyerek system prompt’a Şu anki saat: {{now}} satırını ekleyip zaman damgasını gerçek zamanlı olarak enjekte edene kadar. Ertesi gün izleme uyarıları çaldı: her konuşma için TTFT 0,5 saniyeden 3-5 saniyeye sıçradı ve aylık çıkarım faturası neredeyse ikiye katlandı. Kod tamamen sorunsuz görünüyordu, model değişmemişti—sorun neredeydi?

Cevap: o tek zaman damgası satırı, her istekte token dizisinin zaman damgasının bulunduğu konumdan itibaren farklılaşmasına yol açıyordu; dolayısıyla bu konumdaki ve sonrasındaki KV durumları yeniden kullanılamıyordu. System prompt context’in ön tarafında yer aldığından, model çoğu zaman onu izleyen giriş token’larının büyük bölümüne ait anahtar-değer çiftlerini yeniden hesaplamak zorunda kalıyordu (burada “Key” ve “Value”, attention mekanizmasındaki iki tür vektördür. Bu tür bir “görünmez maliyet” Agent sistemlerinde tekrar tekrar ortaya çıkar—bir geliştiricinin yazdığı zararsız görünen tek bir kod satırı, tüm çıkarım hattını bir büyüklük mertebesi yavaşlatabilir. Bu bölüm bu tuzaklardan nasıl kaçınılacağıyla ilgilidir.

> Teknik Eşik Notu: Bu bölüm, Transformer attention mekanizmasının ve KV Cache’in içsel ilkelerini içerir, bu da onu kitabın teknik açıdan en yoğun kısımlarından biri yapar. Bu temel mekanizmalara aşina değilseniz, ayrıntılı ilkeleri atlayıp yalnızca aşağıdaki üç temel sonucu hatırlayabilirsiniz:
> 
> **System prompt ve araç tanımları netleştikten sonra bunları değiştirmeyin.** Tek bir boşluk eklemek dahil herhangi bir değişiklik token dizisini değiştirebilir ve cache’in ilk farklı token’dan itibaren yeniden kullanılmasını engelleyebilir; değişiklik ne kadar öndeyse gecikme ve maliyet üzerindeki etkisi genellikle o kadar büyük olur (tam büyüklük modele ve yapılandırmaya bağlıdır).  
> 
> **Dinamik bilgiyi her zaman sona ekleyin**—zaman damgaları ve kullanıcı durumu gibi değişen içerikler, mevcut system prompt’u değiştirerek değil, konuşmanın sonuna yeni mesajlar olarak eklenmelidir.
> 
>   
> **Standart API formatını kullanın; mesajları elle birleştirmeyin:** Yapılandırılmış mesajlar, Chat Template tarafından modelin eğitim sırasında gördüğü sabit bir token dizisine çevrilir. Dizeleri elle "USER: ... ASSISTANT: ..." gibi formatlara birleştirmenin temel sorunu, bunun bu eğitim formatından sapması ve modelin çok adımlı reasoning yeteneğini zayıflatmasıdır. Cache’e gelince—o yalnızca token bayt dizisini tanır. Birleştirilmiş ön ek bayt düzeyinde sabit kaldığı sürece, yine de cache’e isabet edebilir. Ancak birleştirme yöntemi kararsızsa (örn. her seferinde ön eke dinamik içerik enjekte etmek), cache de geçersiz olur.
> 
> Bu üç sonucun ardındaki sezgi basittir: büyük bir model context’i işlerken daha önce işlediği içeriği cache’e alır; bir dahaki sefere yalnızca yeni kısmı işlemesi gerekir.  
> Bu üç ilkeyi hatırlayın, aşağıdaki teknik ayrıntıları atlasanız bile bir Agent’ın context yapısını doğru biçimde tasarlayabilirsiniz. Aşağıdaki içerik “neden”in derinine inmek isteyen okuyucular içindir.

### 2.3.1 api mesajlarından model token’larına: chat template

chat template, bu kitap boyunca temel bir kavramdır: yalnızca kv cache ile ilgili değildir, aynı zamanda çok turlu tool calling, düşünce zinciri konumu ve durum çubuğu enjeksiyonu gibi mekanizmaların doğru çalışıp çalışmadığını da belirler. bu yüzden ayrı bir açıklamayı hak eder. token dizileri daha önce görülen api mesajlarının json formatından çok farklıdır. bunun nedeni, api düzeyindeki yapılandırılmış mesajların modelin anlayabileceği doğrusal bir token akışına dönüştürülmesi gerekmesidir - bu dönüşümden sorumlu bileşen chat template’dir.

yapılandırılmış api mesajları

system → yardımcı bir asistansın

user → pekinde bugün hava nasıl

assistant → üretilecek

chat template ile →

modelin gerçekte işlediği doğrusal token akışı

```
<|im_start|>system"Yardımcı bir asistansın."<|im_end|><|im_start|>user"Bugün hava nasıl?"<|im_end|>
<|im_start|>assistant 
```

Chat Template’i bir zarf formatı olarak düşünün: API mesajı mektubun içeriğidir, Chat Template ise zarfa gönderen ve alıcının nasıl yazılacağını belirtir — her mesajın sınırlarını ve rolünü belirtmek için özel token’lar (örn. <|im_start|>system, <|im_end|>) kullanır. Farklı model aileleri (Qwen, Llama, Gemma), tıpkı farklı ülkelerin farklı posta kodu kuralları olması gibi, farklı “zarf formatları” kullanır. API sunucusu (vLLM, Ollama vb.) bu dönüşümü modelin Chat Template’ine göre otomatik olarak gerçekleştirir, bu yüzden geliştiricilerin genellikle bunu elle yönetmesine gerek yoktur. 

Agent geliştiricileri için, Chat Template’i elle yazmanıza veya değiştirmenize gerek yoktur — API sunucusu bunu otomatik olarak yönetir. Ancak varlığını anlamanın Agent geliştirme için iki pratik faydası vardır:

  
Birincisi, standart API biçiminin neden kullanılması gerektiğini açıklar. Geliştirici API’yi atlayıp mesajları kendisi birleştirirse (örneğin tool sonucunu tool türü yerine sıradan bir user mesajı olarak geçirirse), Chat Template tool yanıtını yanlışlıkla yeni bir kullanıcı sorgusu olarak algılar ve modelin düşünce zincirini koruma mekanizmasını bozar.

Qwen3’ün Chat Template’ini ele alalım. Çok turlu tool çağrılarında model, önceki iç düşünme sürecini (<think> etiketleri içindeki içeriği) bir müsveddedeki çözüm adımları gibi koruyarak düşüncenin sürekliliğini sağlar. Ancak Chat Template yeni bir kullanıcı sorgusu algıladığında “kullanıcı konuyu değiştirdi” diye varsayar, önceki düşünceyi temizleyip yeniden başlar. Bir tool sonucu yanlışlıkla kullanıcı mesajı olarak işaretlenirse bu temizlik hatayla tetiklenir; modelin hesaplamanın ortasında müsveddesinin elinden alınması ve baştan başlaması gibi, çok adımlı düşünmenin sürekliliği ciddi biçimde zarar görür.

Farklı model ailelerinin geçmiş düşünce zincirlerini ele alma politikaları büyük ölçüde değişir ve bu politikalar hızla evrilir. DeepSeek R1 döneminde resmi uygulama tüm geçmiş düşünceyi çıkarmaktı: çok turlu konuşmalarda yalnızca content geri gönderilir, reasoning_content gönderilmezdi; çünkü geçmiş CoT, R1 eğitim girdilerinde hiç yer almamıştı ve geri verilmesi dağılım dışı bir girdi olarak çıktıyı bozabilir, ayrıca çok sayıda token tasarrufu sağlardı. Ancak bu strateji Agent senaryolarında kusurludur: ara düşünme, “bu araç neden çağrıldı, hangi varsayımlar elendi” gibi kritik durumu taşır; çıkarıldığında model her turda sıfırdan akıl yürütür, hataları tekrarlamaya ve uzun vadeli planı kaybetmeye yatkın olur. Bu nedenle DeepSeek, V4’te politikayı tamamen tersine çevirdi: istek tools parametresini taşıdığı sürece, iki user mesajı arasındaki her assistant mesajının reasoning_content alanı—o turda gerçekten tool çağırmamış olsa bile—aynen geri gönderilmelidir; aksi halde API doğrudan 400 hatası verir. tools içermeyen düz sohbette ise geçmiş düşünce yine yok sayılır. Agent her zaman tools gönderdiği için bu zorunluluktan kaçış yoktur. Kimi K2, GLM-5 ve diğerleri de aynı protokolü kullanır. Claude da tool çağırma döngüsünde client’ın thinking block’u imza doğrulamasıyla birlikte değiştirmeden API’ye geri göndermesini ister; yeni bir kullanıcı girdisinden sonra server, son gerçek kullanıcı girdisinden önceki thinking block’ları yok sayar. Bu nedenle kullanmadan önce modelin en güncel belgelerine bakılmalıdır. Çok turlu diyalogda bu farklar yalnızca token tasarrufu meselesidir; ama yarım kalmış bir yörüngeyi başka bir sağlayıcının modeline devredip tamamlatmak gerektiği anda gerçek arayüz hatalarına dönüşürler.

İkincisi, KV Cache’in ön eke neden bu kadar duyarlı olduğunu açıklar. Chat Template, system mesajlarını ve tool tanımlarını en başa yerleştirilen sabit bir token dizisine dönüştürür. Bu token’ların anahtar-değer çiftleri istekler arasında önbelleğe alınıp yeniden kullanılabilir. Ancak ön ekteki bir token değişirse — system prompt’a fazladan bir boşluk eklemek dahi olsa — cache ilk farklı token’dan itibaren yeniden kullanılamaz.

### 2.3.2 kv cache’in ilkeleri ve kısıtları

KV Cache’in değerini anlamak için önce onsuz ne olduğunu görelim. Bir Agent’ın konuşmanın 6. turunda olduğunu ve context’in 2000 token biriktirdiğini varsayalım. Önbellekleme olmadan, model her yeni token ürettiğinde bu 2000 token için K ve V vektörlerini yeniden hesaplaması gerekir — özünde tüm ön ek için ileri yönlü hesaplamayı yeniden çalıştırmak. İlk 5 turun içeriği hiç değişmemiş olsa bile, 6. tur yine de 1. tur gibi tüm ön eki sıfırdan hesaplamak zorundadır ve ön ek artık daha uzun olduğundan, maliyet 1. turdan çok daha yüksektir. Önbellekleme olmadan, prefill aşamasındaki (modelin bir yanıt üretmeye resmen başlamadan önce tüm girdi token’larını bir kerede işlediği aşama) attention hesaplaması context uzunluğuyla karesel olarak büyür, bu da konuşma derinleştikçe gecikme ve maliyetin fırlamasına neden olur. Bu, onlarca tool calling gerektiren Agent görevleri için kabul edilemez.

**Basit bir örnekle KV Cache’i anlamak.** Context’in 4 token [A, B, C, D] içerdiğini ve modelin 5. token’ı, E’yi üretmek üzere olduğunu varsayalım. Attention’ın temel işlemi şudur: E’nin Query vektörü, eşleşme skorunu hesaplamak için mevcut tüm token’ların Key vektörleriyle bir nokta çarpımı yapar (nokta çarpımlarının sezgisel bir açıklaması için Deney 2-2’ye bakın), ardından E’nin çıktı temsilini elde etmek için bu skorlara dayanarak tüm Value vektörlerinin ağırlıklı bir toplamı hesaplanır.

KV Cache olmadan, her yeni token üretildiğinde önceki tüm token’ların K ve V vektörleri sıfırdan yeniden hesaplanmalıdır: E’yi üretmek 5 K ve V kümesi hesaplamayı gerektirir, 6. token’ı üretmek 6 küme gerektirir… ve N. token’a gelindiğinde N küme hesaplanmalıdır, toplam hesaplama N² ile orantılıdır.

KV Cache ile, A, B, C ve D’nin K ve V vektörleri bir kez hesaplandıktan sonra önbelleğe alınır. E üretilirken, yalnızca E’nin kendi K ve V’sinin hesaplanması gerekir, ardından attention hesaplaması bunlar ve önbelleğe alınmış 4 küme kullanılarak yapılır. KV Cache’in, geçmiş token’lar için K ve V projeksiyonlarının yeniden hesaplanmasını kurtardığına dikkat edin, bu yüzden her decode adımının tüm ön eki yeniden hesaplamasına gerek yoktur; ancak her yeni token için attention hesaplaması yine de önbelleğe alınmış tüm K ve V değerlerini gezmelidir, hesaplama context uzunluğuyla doğrusal olarak büyür — bu yüzden uzun context’te decode giderek yavaşlar ve KV Cache’in belleği ve bant genişliği çıkarımın darboğazı haline gelir.

**Ön eği değiştirmek neden değişiklik noktasından sonraki cache’i geçersiz kılar?**Büyük dil modelleri, üst üste yığılmış birden fazla Transformer katmanından oluşur (modern büyük modeller tipik olarak onlarca ila yüzlerce katmana sahiptir) ve her katman kendi K ve V cache’ini bağımsız olarak üretir. Bu katmanlar seri halinde bağlıdır: katman 1’in çıktısı katman 2’ye girdi olarak beslenir, katman 2’nin çıktısı katman 3’e beslenir ve bu böyle devam eder, tıpkı bir üretim hattı gibi. Her kelimeyi işlerken, katman 1 o kelimenin ve önceki tüm kelimelerin bilgisini dikkate alır, ardından bir ara sonuç çıktısı verir; katman 2 bu ara sonucu alır ve daha fazla işler. Bu yüzden k’ncı token değişirse (örneğin system prompt’taki bir karakter değiştirilirse), k’dan önceki durumlar etkilenmez; ancak k’dan itibaren temsiller, fark katmanlar boyunca yayıldıkça etkilenir. Pratikte cache yalnızca ilk farklı token’dan önceki noktaya kadar yeniden kullanılabilir ve o konumdan itibaren yeniden hesaplanmalıdır. Maliyet değişikliğin konumuna bağlıdır: değişiklik ne kadar öndeyse genellikle o kadar fazla token’ın yeniden hesaplanıp faturalandırılması gerekir ve gecikme üzerindeki etki o kadar büyük olur (bu bölümün deneyleri birkaç kat artış ölçtü). Bu yüzden kitap tekrar tekrar “system prompt bir kez ayarlandıktan sonra değiştirmeyin” ilkesini vurgular.

### 2.3.3 KV Cache ve Prompt Cache: İki Önbellekleme Düzeyi

kv cache, modelin içindeki bir mekanizmadır: tek bir çıkarım geçişi sırasında önceden hesaplanan token’ların key-value çiftlerini cache’e alarak yinelenen hesabı önler. prompt cache ise çıkarım motorunun bir optimizasyonudur: birden fazla api isteği sırasında aynı ön ekin hesaplama sonuçlarını cache’e alır. ikisi de ön ek değişmezliğinden yararlanır ancak farklı düzeylerde çalışır.

kv cache tek bir istek içinde token üretimini hızlandırır, prompt cache istekler arasındaki yinelenen hesap maliyetini azaltır.

Birden fazla isteğin ön eki aynıysa sağlayıcı önceden hesaplanmış KV Cache’i doğrudan yeniden kullanabilir. Cache’ten okumak ilk hesaplamadan çok daha ucuzdur; örneğin Anthropic, DeepSeek ve GPT-5’te yaklaşık onda bir maliyetlidir. Bununla birlikte etkinleştirme ve faturalandırma ayrıntıları sağlayıcıya göre değişir: bazıları otomatik çalışır, bazıları elle belirtilmelidir. Kullanırken en güncel belgelere bakın.

### 2.3.4 mimari bir kısıt olarak önbellekleme

Üretim ortamındaki Agent’larda cache, sonradan eklenen bir hızlandırma yöntemi değil, mimariyi baştan şekillendiren temel bir kısıttır.

Ana fikirler:

- **Prompt sırası cache’e göre düzenlenir.** Değişmeyen bilgiler cache sınırından önce, kullanıcıya veya oturuma göre değişen bilgiler sonra yerleştirilir.
    
- **Dinamik koşullar cache çeşitlerini artırır.** Örneğin işletim sistemi, çalışma modu ve dil gibi üç ikili koşul, (2^3 = 8) farklı cache anahtarı oluşturur. Bu yüzden değişken bilgileri mümkün olduğunca cache sınırından sonra tutmak gerekir.
    
- **Alt Agent’lar ana Agent’la aynı yapıyı kullanmalıdır.** Prompt, tool tanımları, model ayarları ve mesajların başlangıcı aynı olursa cache isabeti sağlanır; böylece maliyet ve gecikme düşer.
    
- **Tool çıktılarının yer tutucuları sabit kalmalıdır.** Büyük çıktılar özetlense bile kullanılan yer tutucu daha sonra değiştirilmemelidir. Aksi halde sistem eski cache’i kullanamaz.
    
- **Bayt düzeyinde aynılık önemlidir.** Küçük bir metin, sıra veya yapı değişikliği bile cache’in geçersiz olmasına yol açabilir.
    

Kısaca: Agent tasarlanırken önce “Bu yapı cache’i nasıl korur?” sorusu sorulmalıdır. Cache ekonomisi ne kadar erken hesaba katılırsa, sonradan yapılacak düzeltmeler ve maliyet o kadar azalır.

------------------------

## 2.4 prompt engineering: system prompt’u organize etmek

Prompt Engineering’in temel nesnesi System Prompttur—API mesaj listesindeki role: "system" mesajı. Bu, Agent’ın “çalışan el kitabıdır”; Agent’ın kimliğini, davranış kurallarını, kısıtlamalarını ve iş akışını tanımlar. İyi tasarlanmış bir system prompt, modelin genel yeteneklerini belirli görevlerde tam olarak devreye sokmasını sağlar.

System prompt tasarımı için pratik bir turnusol testi vardır: büyük bir dil modeli, son derece yetenekli ama sizin belirli iş akışlarınıza ve iç kurallarınıza tamamen yabancı akıllı bir yeni çalışandır. Akıllı bir yeni çalışan, system prompt’unuzu okuduktan sonra hâlâ ne yapacağını bilmiyorsa, Agent da bilmeyecektir.

### 2.4.1 Ton ve Üslup: System Prompt’un “Kişiliği”

Ton ve üslup, prompt engineering’in en kolay gözden kaçırılan kısmıdır, ama kullanıcı deneyimini derinden şekillendirir. “4 satırdan az, öz biçimde yanıt VERMELİSİN” ifadesini düşünün. Agent bir görevi tamamlayamadığında, prompt “yanıtınızı 1-2 cümleyle sınırlayın” ve “bir şeyi neden yapamadığınızı açıklamayın” talep eder—bu tasarım, Agent’ı uzun kendini savunmalardan uzak tutar. Büyük harfli kelimeler (“ASLA X yapma”), modelin “dikkatini” “Lütfen X yapmaktan kaçının”a göre daha iyi yakalar, ama aşırı kullanım etkiyi seyreltir; bunları yalnızca gerçekten kritik kısıtlamalar için saklayın.

### 2.4.2 Yapılandırılmış Prompt’lar: System Prompt’un “Formatı”

Modern büyük dil modelleri, eğitim verilerindeki büyük miktarda yapılandırılmış içerikten kaynaklanan, yapılandırılmış girdiye karşı belirgin bir duyarlılık gösterir. XML etiketlerinin kullanımı hiyerarşik bir ilkeyi izler, etiket adlarının kendisi semantik bilgi taşır—<working_directory> modele bunun çalışma dizini bilgisi olduğunu anında söylerken, “Mevcut dizin: /Users/project/src” gibi düz metin formatı, modelin iki nokta üst üstenin iki tarafı arasındaki ilişkiyi çözmek için ekstra düşünmesini gerektirir.

  
Markdown, okunabilirliği korurken hafif bir yapı sağlar, bu da onu hiyerarşik talimatları ve bilgiyi organize etmek için özellikle uygun kılar. XML ve Markdown birlikte iki katmanlı bir yapı oluşturur: XML makine tarafından ayrıştırılabilir kesin semantiği yönetirken, Markdown hem insanlar hem de makineler tarafından okunabilir organizasyonel mantığı yönetir.

Örneğin, ikisini aynı anda kullanan bir system prompt:

```
# Araç Kullanım Kuralları

## Dosya İşlemleri
<file_operation&gt>
- Bir dosyayı okumadan önce yolun var olup olmadığı denetlenmeli
- Bir dosyaya yazmadan önce yedek alınmalı
</file_operation>

## Ağ İstekleri
<network_request>
- Zaman aşımı 30 saniye olarak ayarlanmalı
- Başarısızlık durumunda en fazla 3 kez yeniden denenmeli
</network_request>
```

Markdown’ın rolü: # ve ## gibi başlıklar insanın hiyerarşiyi tek bakışta görmesini sağlar, okunabilirliği yüksek tutar.

  
XML’in rolü: <file_operation> ve <network_request> gibi etiketler modele “bu blok dosya işlemleriyle ilgili”, “bu blok ağ istekleriyle ilgili” bilgisini verir; anlam kesindir ve model daha isabetli işler.

İkisi birlikte kullanıldığında prompt insan için açık, model için de anlaşılır olur.

### 2.4.3 Süreç Odaklılık ve Kural Yığma: System Prompt’un “Organizasyonu”

İnsanlar için bilişsel yükü azaltan yöntemler, büyük dil modelleri için de eşit derecede etkilidir—çünkü model eğitim sırasında insan dilini ve düşünme kalıplarını öğrenmiştir. Yeni bir çalışana yüzlerce dağınık kural içeren, akış şeması olmayan, öncelik talimatı olmayan bir el kitabı verdiğinizi hayal edin—en zeki insan bile kafası karışır: birden fazla kural aynı anda geçerli olduğunda hangisi seçilmeli? Ve kurallar tarafından kapsanmayan durumlar ne olacak?

  
Buna karşılık, süreç odaklı bir prompt, mükemmel bir yeni çalışan eğitim el kitabı gibidir, net bir Standart Çalışma Prosedürü (SOP) sağlar:

```
Dosya İşleme Standart Çalışma Prosedürü:

Adım 1: Doğrulama
  Dosyanın var olup olmadığını ve erişilebilir olup olmadığını kontrol et
  - Bulunamazsa → hatayı kaydet ve dur
   ↓
Adım 2: Sınıflandırma
   Uzantıya ve içeriğe göre dosya türünü belirle
   ↓
Adım 3: Ön İşleme
   Yapılandırma dosyaları → yedek oluştur
   Büyük dosyalar (>1MB) → akış işleme
   ↓
Adım 4: Yürütme
   Dosya türüne göre temel işleme mantığını yürüt
   ↓
Adım 5: Doğrulama
   İşlenmiş dosyanın bütünlüğünü sağla
```

Bu süreç tasarımı, modelin her an hangi aşamada olduğunu, mevcut adımın hedefinin ne olduğunu ve tamamlandıktan sonra hangi adıma geçileceğini net biçimde bilmesini sağlar. Bir istisnayla karşılaşıldığında, model tüm kuralları gezip bir eşleşme aramak yerine, mevcut aşamaya dayanarak işleme yöntemine karar verebilir.

### 2.4.4 İş Kuralı İnceltme: System Prompt’un “İçeriği”

Üretim seviyesinde Agent sistemlerinde en kritik ve en çok gözden kaçan konu, iş kurallarının açık ve yürütülebilir biçimde tanımlanmasıdır. Bu yalnızca teknik değil, ürün tasarımı sorunudur; ürün yöneticileri kuralların belirlenmesine doğrudan dahil olmalıdır.

Örnekteki temel dersler:

- Belirsiz ifadeler, Agent’ın aynı görevi farklı zamanlarda farklı biçimde sınıflandırmasına ve tutarsız ücretlendirmesine yol açar.
    
- Komisyon yalnızca mevcut faturanın müzakereyle düşürüldüğü durumlarda kullanılmalıdır.
    
- İadeler ve abonelik iptalleri komisyonla ücretlendirilmemeli; sabit ücret uygulanmalıdır.
    
- Başarı olasılığı adım adım ve standart bir yöntemle tahmin edilmeli, bu tahmin doğrudan ücretlendirme modeline bağlanmalıdır.
    
- Çok düşük başarı ihtimali olan görevler reddedilmeli; belirli eşiklerin üzerindeki görevlerde iade edilebilir veya sabit ücretli modeller kullanılmalıdır.
    
- Ücret hesaplama kuralları açıkça kodlanmalıdır: örneğin telefon görüşmeleri dakika başına ücretlendirilir, toplam tutar yuvarlanır.
    
- “Tasarruf”, yalnızca mevcut faturaya göre hesaplanmalı; gelecekte oluşabilecek zamları önlemek tasarruf sayılmamalıdır.
    

Kısacası, ürün yöneticileri iş mantığını ayrıntılı ve kesin kurallara dönüştürmeli; mühendisler ise bu kuralları prompt’lara doğru ve tutarlı biçimde aktarmalıdır. LLM’ye iş kurallarını kendisinin icat edeceği geniş bir takdir alanı vermek yerine, net prosedürler sağlamak Agent davranışını daha öngörülebilir, tutarlı ve güvenilir kılar.

### 2.4.5 Few-shot Örnekler: Modele Ne Zaman Örnek Gösterilmeli

Örnekler, system prompt’ların kurallar ve süreçler kadar önemli bir parçasıdır; özellikle istenen çıktı biçimi veya üslubu yazılı kurallarla net biçimde tarif edilemiyorsa few-shot örnekler daha etkili olur.

Temel noktalar:

- Belirli bir yazım tarzı, rapor formatı veya müşteri hizmetleri tonu gerektiğinde, uzun açıklamalar yerine 2–3 kaliteli girdi-çıktı örneği verilebilir.
    
- Model, bu örneklerden kalıbı context içinde öğrenir; bu çoğu zaman aynı uzunluktaki soyut kurallardan daha iyi sonuç verir.
    
- Modelin zaten kolayca uyguladığı görevlerde örnek kullanmak gereksiz token tüketimine yol açar.
    
- Örnekler iki yere konabilir: 
    
    - **System prompt’a:** Tüm istekler için geçerli sabit örnekler.
        
    - **İlk user/assistant mesajlarına:** Farklı konuşma türleri için farklı örnek kümeleri gerektiğinde daha esnek yapı.
        
- Örnekler context’in başında yer aldığı için KV Cache açısından sabit tutulmalıdır. Her istek için dinamik olarak “en uygun örnek” seçmek, ön eki değiştirir ve cache’in yeniden oluşturulmasına neden olabilir.
    
- Daha fazla örnek her zaman daha iyi değildir. Sınır durumlarını temsil eden, dikkatle seçilmiş 2–3 örnek; birbirine benzeyen 10 örnekten genellikle daha etkilidir.
    

Özetle: Kuralla ifade edilmesi zor davranışları örneklerle gösterin, kolay ifade edilen davranışları kurallarla tanımlayın ve üretimde örnek kümelerini görev türü başına sabit tutun.

### 2.4.6 Tool Tanımı Tasarımı

Tool tanımları, Agent’ın tool’ları doğru kullanmasını sağlayan bir işletim kılavuzu gibi tasarlanmalıdır. İyi bir tanım:

- Kullanım sınırlarını ve yasakları açıkça belirtir.
    
- Doğru kullanıma dair somut örnekler içerir.
    
- Performans önerileri verir; örneğin çağrıları toplu yapmak.
    
- Tools arası bağımlılıkları açıklar; örneğin düzenlemeden önce dosyayı okumak.
    

Geleneksel yapıda system prompt ve `tools` alanı birlikte statik bir ön ek oluşturur ve önbelleğe alınır. Ancak yeni API’lerde tool tanımları da kademeli olarak yüklenebilir:

- Başlangıçta yalnızca tool adları ve kısa açıklamalar context’e eklenir.
    
- Model ihtiyaç duyduğunda `tool_search` benzeri mekanizmalarla tam şemayı arar.
    
- Tam tool tanımı daha sonra trajectory’nin sonuna eklenir.
    
- Bu şema sonraki turlarda sabit kalır ve yeni mesajlar onun arkasına eklenir.
    

Bu yaklaşım KV Cache’i bozmaz; çünkü causal attention nedeniyle sona eklenen içerik, daha önce önbelleğe alınmış token’ların anahtar-değerlerini değiştirmez. Şema yalnızca ilk keşfedildiğinde hesaplanır, sonraki turlarda cache’ten yararlanılır.

Buradaki temel tasarım ilkesi, tool bilgilerinin de **kademeli açığa çıkarılmasıdır**: statik ön ekte kısa bilgiler, ihtiyaç halinde ise tam şema bulunur. Ancak bunun çalışması için modelin konuşma ortasında yüklenen tool tanımlarını yorumlayacak şekilde eğitilmiş olması gerekir; bu nedenle destek öncelikle daha yeni modellerde bulunur. Self-hosted modellerin ise özel eğitim veya uyarlama gerektirmesi mümkündür.

### 2.4.7 Prompt Injection: Context Güvenliğine Yönelik Temel Tehdit

Prompt injection, Agent’ın işlediği dış içeriğe kötü niyetli talimatlar yerleştirilerek sistem kurallarının ve model davranışının ele geçirilmesidir. Agent’lar tool çağırabildiği için sonuçları sıradan sohbet botlarına göre daha tehlikelidir; saldırı, veri sızdırma, e-posta gönderme veya dosya silme gibi geri döndürülemez işlemlere yol açabilir.

Başlıca savunmalar:

- **Kaynak etiketleme:** Web sayfası, e-posta veya doküman gibi dış içerikleri açıkça işaretleyin. Örneğin içeriği güvenilmeyen veri olarak etiketleyin ve içindeki talimatların uygulanmaması gerektiğini belirtin.
    
- **Yapılandırılmış roller:** `system`, `user`, `assistant` ve `tool` rollerini doğru kullanın. Tool çıktısını kullanıcı mesajıyla birleştirmek, içeriğin kaynağını ve güven seviyesini belirsizleştirir.
    
- **Girdi temizleme:** “Önceki talimatları yok say” gibi şüpheli kalıpları filtreleyin. Ancak bu yalnızca yardımcı bir katmandır; saldırganlar ifadeleri kolayca değiştirebilir.
    
- **Bilinmeyen Skill’leri inceleme:** Skill’ler dış talimatları sisteme yüklediği için kötü amaçlı talimat taşıyabilir. Üçüncü taraf Skill’ler kurulmadan önce kod gibi denetlenmelidir.
    
- **Durum çubuğunu koruma:** Dış kaynaklardan gelen içerikleri doğrudan Agent durumuna aktarmayın; model bu alana yüksek güven duyabilir.
    

Prompt injection’a karşı yalnızca context düzeyindeki önlemlere güvenilmemelidir. İzin kontrolleri, sandbox kullanımı, yüksek riskli işlemler için insan veya bağımsız sistem onayı ve bilgi tabanı zehirlenmesine karşı denetimler gibi yürütme düzeyi savunmaları da gerekir.

Özetle: **Dış içeriği her zaman veri olarak ele alın, talimatlardan ayırın, kaynağını koruyun ve kritik eylemleri modelin tek başına kararına bırakmayın.**

Prompt injection, Agent sistemlerinde sıradan sohbet botlarına göre daha tehlikelidir. Sıradan bir sohbet botu için en kötü senaryo uygunsuz içerik çıktısı vermektir, ama bir Agent’ın tool calling yeteneği vardır—enjekte edilmiş talimatlar, Agent’ın dosyaları silme, e-posta gönderme veya özel verileri sızdırma gibi geri alınamaz eylemler gerçekleştirmesine neden olabilir. Prompt injection için saldırı yüzeyi, Agent’ın yetenekleri büyüdükçe genişler: her algı aracı—web okuma, doküman ayrıştırma, e-posta işleme—potansiyel bir enjeksiyon giriş noktasıdır. Saldırganlar talimatları bir web sayfasının görünmez öğelerine gömebilir, komutları PDF meta verilerinde gizleyebilir, hatta görüntülerin EXIF meta verilerine (görüntü dosyaları içinde gömülü çekim parametresi bilgisi, örn. çekim zamanı, kamera modeli vb.) metin yerleştirebilir.

Context düzeyinde, savunmanın özü modelin “talimatlar” ile “veriyi” ayırt etmesine yardımcı olmaktır—hangi içeriğin ona komut verme yetkisine sahip olduğunu ve hangi içeriğin yalnızca işlenecek malzeme olduğunu bilmesini sağlamak:

Kaynak Etiketleme: Dış içeriği context’e enjekte etmeden önce, onu net işaretlerle sarın ve kaynağı belirtin (örn. <external_content source="webpage">...</external_content>), modele bu içeriğin güvenilmeyen bir dış dünyadan geldiğini ve içindeki herhangi bir “talimatın” yürütülmemesi gerektiğini bildirin.  
Yapılandırılmış Roller: Bilgiyi iletmek için Chat Template’in rol sistemini (system/user/assistant/tool) sıkı biçimde kullanın, bu modelin eğitim sırasında oluşturulan önceliğe dayanarak güvenilir talimatları dış verilerden ayırt etmesini sağlar—bu, bu bölümdeki “mesajları elle birleştirmeyin” ilkesinin bir başka nedenidir: tool sonuçlarını kullanıcı mesajlarına karıştırmak, modelin kaynağı belirleme temelini silmekle eşdeğerdir.  
Girdi Temizleme (Input Sanitization): Dış içerikteki şüpheli kalıpları filtreleyin (örn. “önceki talimatları göz ardı et” gibi yaygın enjeksiyon ifadeleri)

## 2.5 Dinamik Prompt’lar ve Agent Skills

Agent’ın görev alanı büyüdükçe tüm kuralları tek bir system prompt’a eklemek iki temel soruna yol açar:

- **Token israfı:** Mevcut görevle ilgisiz kurallar da her istekte context’e dahil edilir.
    
- **Dikkatin seyrelmesi:** Fazla ve ilgisiz bilgi, modelin önemli talimatlara odaklanmasını zorlaştırır; bu durum zamanla “context çürümesine” yol açabilir.
    

Bu nedenle yaklaşım, her şeyi başlangıçta yükleyen statik prompt’lardan, gereken bilgiyi ihtiyaç anında getiren dinamik prompt’lara evrilir.

**Agent Skills**, bu yaklaşımın pratik bir uygulamasıdır. Her görev alanına ait kurallar ve bilgiler ayrı beceriler halinde tutulur; Agent yalnızca mevcut görev için gerekli Skill’i yükler. Böylece context daha kısa, odaklı ve verimli kalır.

### 2.5.1 Skills: Alan Yeteneğinin Birleştirilebilir Birimleri

Skills, Agent’ın yeteneklerini bağımsız ve gerektiğinde yüklenebilen bilgi paketlerine ayırır. Her Skill, belirli bir görev alanı için hazırlanmış bir işletim kılavuzu gibidir ve **Kademeli Açığa Çıkarma** yaklaşımını kullanır.

Üç katman vardır:

1. **Meta veri katmanı**
    
    - Her Skill, `name` ve `description` içeren YAML ön ekiyle başlar.
        
    - Agent önce yalnızca bu kataloğu görür ve hangi Skill’in ilgili olduğuna karar verir.
        
    - `description`, Skill’in ne yaptığından çok **hangi durumda kullanılacağını ve ne zaman kullanılmayacağını** açıklamalıdır.
        
    - Olumsuz örnekler ve kullanım sınırları, yanlış tetiklenmeleri azaltır.
        
2. **Temel iş akışı katmanı**
    
    - Skill’in tam `SKILL.md` içeriği yalnızca gerektiğinde yüklenir.
        
    - Yükleme iki şekilde gerçekleşebilir: 
        
        - Kullanıcı `/pptx` gibi bir komut verir; istemci bunu doğrudan Skill’e genişletir.
            
        - Agent, meta veriyi değerlendirip özel bir tool çağrısıyla Skill’i etkinleştirir.
            
    - Bazı sistemlerde Skill, dosya okuma aracıyla yüklenir ve içeriği tool result olarak context’e eklenir.
        
3. **Ayrıntı katmanı**
    
    - Ana Skill dosyası, daha ayrıntılı alt dokümanlara ve referanslara bağlantı verir.
        
    - Agent yalnızca ihtiyacı olan alt belgeleri seçerek okur.
        
    - Örneğin bir PPTX Skill’i, temel sunum iş akışını içerirken HTML’den slayt üretme veya dosya formatı ayrıntılarını ayrı belgelerde tutabilir.
        

Özetle, Skills tüm bilgiyi baştan yüklemek yerine önce keşfedilebilir bir katalog, ardından ihtiyaç halinde temel talimatlar ve son olarak seçici ayrıntılar sunar. Bu yapı token tüketimini azaltır, context’i daha odaklı tutar ve Agent yeteneklerini modüler biçimde genişletmeyi sağlar.

### 2.5.2 Kullanılabilir bir Skill nasıl yazılır?

Yararlı bir Skill, yalnızca bilgi depolamaz; modeli uygulanabilir bir iş akışında yönlendirir. Kullanım amacı, işlem sırası, onay gerektiren noktalar ve “tamamlandı” ölçütü açık olmalıdır.

Baoyu’nun yaklaşımına göre Skill içeriği dört bölümle yapılandırılabilir:

1. **Rol ve okuyucu:** Skill’in kime hizmet ettiği, hangi görevleri kapsadığı ve beklenen çıktı standardı.
    
2. **Temel ilkeler:** Üç ila beş ana karar kuralı; her biri olumlu ve olumsuz örneklerle açıklanmalı.
    
3. **Yasaklar:** Yaygın hatalar, kapsam dışı işlemler ve istisnalar. Kurallar yalnızca yasaklı kelime listeleri olarak değil, **kapsam + eylem + istisna + doğrulama** biçiminde yazılmalı.
    
4. **Referanslar:** Sözlükler, şablonlar, örnekler, betikler ve ayrıntılı alt belgeler.
    

Bir yazım Skill’i oluştururken:

- Birkaç özgün metinle başlayın.
    
- Modelden kelime seçimi, cümle yapısı, paragraf düzeni ve tonu çıkarmasını isteyin.
    
- İlk taslağı gerçek bir görevde test edin.
    
- “Daha doğal yaz” gibi belirsiz geri bildirimler yerine, özgün metin ile düzeltme arasındaki farkları inceleyin.
    
- Silinen kelimeleri, bölünen cümleleri ve eklenen bilgileri kurallara ve örneklere dönüştürün.
    
- Tekrarlanan düzeltmeleri Skill’e geri ekleyin.
    

Skills yalnızca prompt yönetimini iyileştirmez; alan bilgisinin birikmesini ve sürdürülebilir biçimde paylaşılmasını sağlar. Her Skill bağımsız olarak geliştirilebilir, test edilebilir, sürümlendirilebilir ve farklı ekiplerce kullanılabilir. Ayrıca çalıştırılabilir tool’ları, şablon dosyalarını ve yardımcı betikleri de içerebilir.

Sonuç olarak Skills, merkezi ve sürekli büyüyen bir system prompt yerine, alan uzmanlığını modüler paketler halinde yöneten dağıtık bir Agent ekosistemi oluşturur. Bu nedenle kullanılan Skill yapısı, model sağlayıcısının modeli eğitirken özellikle desteklediği Agent etkileşim biçimleriyle uyumlu olmalıdır.

### 2.5.3 Skills’in Context’teki Konumu

Skills’in context maliyeti iki ayrı bileşen olarak değerlendirilmelidir:

- **Meta veri kataloğu:** Skill adlarını ve kısa açıklamalarını içerir; Agent’ın hangi yeteneğin ilgili olduğunu keşfetmesini sağlar.
    
- **Tam Skill talimatları:** Yalnızca ilgili Skill seçildiğinde ve ihtiyaç olduğunda yüklenir.
    

Temel ilke, mesaj rollerini değil **yükleme sırasını** tanımlar: katalog önce görünür olmalı, tam Skill gövdesi ise seçimden sonra eklenmelidir. Bunun `system`, `user`, `developer` veya `tool` mesajlarıyla nasıl uygulanacağı Agent Harness’in tasarımına bağlıdır.

Örneğin farklı çalışma zamanları:

- Kataloğu runtime context’e ekleyip tam Skill’i çağrı noktasında yükleyebilir.
    
- Açık bir `/pptx` komutunu istemci tarafında doğrudan Skill içeriğine genişletebilir; bu durumda ayrı bir tool çağrısı görünmeyebilir.
    
- Kataloğu her turda developer context’inde yeniden oluşturup seçilen Skill’i user context’ine ekleyebilir.
    
- Diğer Skill’leri, ihtiyaç halinde dosya veya tool okuma çağrılarıyla yükleyebilir.
    

Bu ayrıntılar değişse de ortak mimari aynıdır: **küçük, keşfedilebilir bir katalog; ihtiyaç halinde yüklenen tam içerik.**

“KV Cache dostu” ifadesi de “maliyetsiz” anlamına gelmez. Katalog ilk kez işlendiğinde hesaplama gerekir; bir Skill’in tam gövdesi ilk yüklendiğinde de ek maliyet oluşur. Ancak sabit context bölümleri değişmeden kaldığında sonraki istekler cache’i yeniden kullanabilir.

Bu yaklaşımın asıl kazancı, tüm Skill gövdelerini başlangıçta yüklememek ve yeni bir Skill etkinleştirildiğinde mevcut context’i baştan yazmak zorunda kalmamaktır.

### 2.5.4 Skills ve Tools Arasındaki İlişki

Skills mekanizması KV Cache ile uyumludur çünkü tüm özel tool tanımlarını başlangıçtaki system prompt’a yüklemek yerine bilgiyi ihtiyaç halinde ekler.

Skill + genel yürütücü modelinde:

- Başlangıçta yalnızca az sayıda temel tool tanımı bulunur.
    
- Özel görev mantığı ve ilgili ayrıntılar Skill içinde tutulur.
    
- Skill içeriği yalnızca gerektiğinde context’e yüklenir.
    
- Daha önce cache’lenmiş sabit ön ek gereksiz yere değiştirilmez.
    
- Böylece token tüketimi ve modelin dikkatinin dağılması azalır.
    

Bu yaklaşım, iki şeyi birbirinden ayırır: Agent’ın genel olarak kullanabildiği temel yürütme tool’ları ve belirli bir göreve ait uzmanlık bilgisi. Sonuç olarak tool seti küçük, Skill’ler ise modüler ve gerektiğinde yüklenebilir kalır.

Skill’lerin kendisi genellikle doğrudan `read`, `write` veya kod çalıştırma yetkisine sahip değildir. Skill, esas olarak Agent’a **ne yapacağını, hangi sırayla yapacağını ve hangi kurallara uyacağını anlatan talimat ve alan bilgisi paketidir**.

İşlem yapma yetkisi ayrı tool’lardan gelir:

- **Skill:** İş akışı, kurallar, örnekler, kontrol listeleri ve referanslar sağlar.
    
- **Tool:** Dosya okur/yazar, kod çalıştırır, web’e erişir, API çağırır veya başka bir eylem gerçekleştirir.
    
- **Agent:** Skill’deki talimatları yorumlar ve uygun tool’ları çağırır.
    

Örneğin bir PPTX Skill’i şunları söyleyebilir:

1. Sunum dosyasını `Read` ile incele.
    
2. Metni belirli bir tool’la çıkar.
    
3. Slaytları verilen şablona göre oluştur.
    
4. Çıktıyı doğrula.
    
5. Dosyayı `Write` ile kaydet.
    

Ancak Skill’in kendisi bu işlemleri yapmaz; bunları gerçekleştirmek için Agent’ın erişebildiği tool’ları kullanır.

Bununla birlikte Skill paketi, çalıştırılabilir betikler, şablonlar veya yardımcı dosyalar içerebilir. Bu dosyalar da ancak Agent’ın uygun bir çalışma veya dosya aracı varsa ve bu araca erişim verilmişse kullanılabilir. Yani Skill **yetki vermez**; mevcut yeteneklerin nasıl kullanılacağını tarif eder.

Kısaca: **Skill = uzmanlık ve prosedür; Tool = eylem ve yetki.**

## 2.6 Agent Durum Çubuğu: Meta Bilgiyle Agent Trajectory Yönetimini Güçlendirmek

Agent Durum Çubuğu, Agent’ın görev sırasında nerede olduğunu ve çevresinde nelerin değiştiğini sürekli görmesini sağlayan yapılandırılmış bir durum özetidir. Bu bilgiler context’e düzenli olarak enjekte edilir.

Amaçları:

- Görev ilerlemesini takip etmek
    
- Ortamdaki değişiklikleri fark etmek
    
- Yapılan tool çağrılarının sayısını izlemek
    
- Sonsuz döngüleri önlemek
    
- Önceki durumu unutmamak
    
- Ana hedeften sapmayı tespit etmek
    
- Agent’ın kendi davranışını düzenlemesini sağlamak
    

Büyük modeller karmaşık talimatları uygulayabilse de mevcut durum ve ilerleme açıkça sunulmazsa tekrarlayan işlemlere, unutmaya veya yanlış yöne ilerlemeye başlayabilir. Durum çubuğu, bu eksikliği yapılandırılmış meta bilgilerle giderir.

Kısaca:

- **Skill:** Agent’a belirli bir görevin nasıl yapılacağını öğretir.
    
- **Tool:** Agent’ın eylem gerçekleştirmesini sağlar.
    
- **Agent Durum Çubuğu:** Agent’a şu anda ne durumda olduğunu gösterir.
    

Bu nedenle durum çubuğu, Agent’ın yalnızca eylem üretmesini değil, görev sürecini izleyip gerektiğinde yönünü düzeltmesini sağlar.

Evet, **kavramsal olarak bir Agent Durum Çubuğu’na benzer**, ancak tamamen aynı şey değildir.

`/recap`, Claude Code oturumunun mevcut durumunu özetleyerek Agent’ın:

- Şimdiye kadar ne yaptığını,
    
- Hangi dosyaları değiştirdiğini,
    
- Hangi kararların alındığını,
    
- Kalan görevleri,
    
- Önemli context’i
    

yeniden görmesini sağlar.

Bu, **durum bilgisini context’e yeniden enjekte etme** işlevi gördüğü için durum farkındalığını artırır ve uzun görevlerde context kaybını azaltır.

Ancak ayrım şudur:

- **Agent Durum Çubuğu:** Genellikle her turda veya düzenli aralıklarla otomatik eklenen, görev ilerlemesi, ortam durumu, tool çağrısı sayısı ve limitler gibi yapılandırılmış canlı meta bilgidir.
    
- `/recap`**:** Kullanıcının veya sistemin tetiklediği, oturumun geçmişini özetleyen bir recontextualization mekanizmasıdır.
    

Dolayısıyla `/recap`, durum çubuğunun dar anlamdaki otomatik canlı göstergesinden çok, **durum özetleme ve context yenileme aracıdır**. Aynı genel amaca—Agent’ın mevcut duruma dair farkındalığını korumaya—hizmet eder.

## 2.6.1 Agent Durum Çubuğunun Teorik Temeli

Agent Durum Çubuğu’nun etkili olmasının nedeni, modelin uzun context’lerdeki bilgiyi **aktif biçimde özetlemekten çok mevcut içerikten geri getirmede (retrieval) başarılı olmasıdır**.

Context içinde:

- Attention, binlerce token arasından ilgili bilgiyi bulabilir.
    
- Ancak içerik kendiliğinden sayılmaz, indekslenmez veya özetlenmez.
    
- Örneğin Agent, geçmişte kaç telefon araması yaptığını her karar anında ham kayıtları tarayarak yeniden hesaplamak zorunda kalabilir.
    
- Context uzadıkça bu hesaplama maliyeti ve hata olasılığı artar.
    

Örneğin satıcı başına en fazla üç arama yapılabiliyorsa, tool çıktısına doğrudan şu tür bir bilgi eklemek daha güvenlidir:

> “Bu satıcıya yapılan 3. arama.”

Böylece Agent’ın geçmiş kayıtları tekrar tekrar tarayıp saymasına gerek kalmaz; sınırın aşıldığını doğrudan görür ve yeni aramayı durdurur.

Durum çubuğu bu nedenle context’e dağılmış örtük durumu, açık ve kullanılabilir meta bilgiye dönüştürür:

- Görev ilerlemesi
    
- Tool çağrısı sayıları
    
- Kullanım limitleri
    
- Ortamın mevcut durumu
    
- Tamamlanan ve kalan adımlar
    

Uzun context’lerde bir başka sorun da attention’ın seyrelmesidir. Erken verilen hedefler ve kısıtlamalar, sonraki tool çıktıları arasında kaybolabilir; model özellikle context’in sonuna daha fazla ağırlık verebilir. Durum özetini context’in sonlarına yakın ve yapılandırılmış biçimde eklemek, kritik bilgiyi modelin bir sonraki üretimine yaklaştırır.

Kısacası, Agent Durum Çubuğu modele yeni bir muhakeme yeteneği kazandırmaz. Modelin zaten context’te bulunan kritik durumu daha hızlı, daha az hesaplamayla ve daha az hatayla kullanmasını sağlar.

Agent Durum Çubuğu, Agent’ın görev sırasında mevcut durumunu kısa ve düzenli bir özet olarak her an görebilmesini sağlayan mekanizmadır.

Bir tür **kontrol paneli** gibi düşünebilirsin. Örneğin şunları gösterebilir:

- Görevde hangi aşamadayım?
    
- Hangi adımlar tamamlandı?
    
- Sırada ne var?
    
- Bu aracı kaç kez kullandım?
    
- Bir işlem limitine ulaştım mı?
    
- Ortamda hangi değişiklikler oldu?
    
- Görev tamamlandı mı, yoksa kullanıcıdan onay mı bekleniyor?
    

Neden gereklidir? Çünkü Agent uzun bir görev sırasında yaptığı işlemleri context’teki eski mesajlardan yeniden bulup saymak zorunda kalabilir. Bu da unutmaya, aynı işlemi tekrarlamaya veya limitleri aşmaya yol açabilir.

Örneğin:

- Kural: Aynı satıcı en fazla 3 kez aranabilir.
    
- Geçmişte 3 arama yapılmış.
    
- Durum çubuğu: **“Satıcı: 3/3 arama kullanıldı — yeni arama yapma.”**
    

Böylece Agent geçmişteki tüm konuşmayı taramak yerine doğrudan güncel özete bakar.

En kısa tanımıyla:

> **Agent Durum Çubuğu = Agent’ın görev ilerlemesini, sınırlarını ve mevcut ortam durumunu takip etmesi için context’e eklenen kısa, yapılandırılmış durum özeti.**

Skill, Agent’a **nasıl çalışacağını** öğretir; tools eylem yapmasını sağlar; durum çubuğu ise Agent’a **şu anda ne durumda olduğunu** söyler.

## 2.6.2 Agent Durum Çubuğunun Bileşimi

Agent Durum Çubuğu üç temel bilgi türü içerir:

1. **Görev planı**
    
    - Karmaşık görevleri adımlara bölen TODO listesi.
        
    - Agent’ın mevcut adımı, tamamlanan işleri ve sıradaki hedefleri görmesini sağlar.
        
    - Böylece Agent yerel bir işe fazla odaklanıp ana amacı unutmaz.
        
2. **Olaylara ait yan bilgiler**
    
    - Bir olayın gerçekleştiği zaman, konum veya önceki yanıttan bu yana geçen süre gibi ek meta veriler.
        
    - Olayın yalnızca ne olduğunu değil, hangi context’te gerçekleştiğini anlamaya yardımcı olur.
        
3. **Mevcut ortam özeti**
    
    - Sistem saati, çalışma dizini ve ortamın diğer dinamik bilgileri.
        
    - Tekrarlanan veya anormal işlemler için uyarılar.
        
    - Context’te dağınık halde bulunan örtük durumların açık ve kısa gözlemlere dönüştürülmesi.
        

Bu bilgilerin güncellenme biçimi farklıdır:

- **Olay yan bilgileri** ilgili olayla birlikte eklenir.
    
- **Görev planı ve ortam özeti** görev ilerledikçe sürekli güncellenir.
    

Kısaca, Agent Durum Çubuğu Agent’a hem **ne yapması gerektiğini**, hem **olayların hangi koşullarda gerçekleştiğini**, hem de **şu anda ortamın ne durumda olduğunu** hatırlatır.

## 2.6.3 Agent Durum Çubuğunun Context’teki Belirli Konumu

Agent Durum Çubuğu, API düzeyinde system mesajını değiştirmek yerine context’in sonuna **otomatik oluşturulmuş bir** `user` **mesajı** olarak eklenir.

Bunun iki nedeni vardır:

- **KV Cache korunur:** System mesajı değiştirilirse tüm sabit ön ekin cache’i geçersiz olabilir. Durum bilgisini sona eklemek, mevcut cache’i bozmadan yeni bilgiyi dahil eder.
    
- **Attention avantajı sağlar:** Durum özeti modelin üreteceği yeni token’lara en yakın konumda olduğundan, modelin bu bilgiye daha fazla dikkat etmesi beklenir.
    

Buradaki `user` rolü gerçek kullanıcı mesajı anlamına gelmez. Bu yalnızca API protokolünde context’in sonuna dinamik bilgi eklemek için kullanılan teknik bir mesaj türüdür. İçerik, Agent Harness tarafından otomatik üretilir ve `&lt;agent_status&gt;` gibi etiketlerle ayrıştırılır.

```js
messages: [
  {
    role: "system",
    content: "Sen bir müşteri hizmetleri asistanısın..."
  },

  {
    role: "user",
    content: "Xfinity planımı iptal etmeme yardım et"
  },

  {
    role: "assistant",
    content: null,
    tool_calls: [
      {
        name: "phone_call",
        arguments: {
          merchant: "Xfinity"
        }
      }
    ]
  },

  {
    role: "tool",
    content: "Arama kaydı..."
  },

  // ... daha fazla assistant/tool turu ...

  {
    role: "user",
    content: "Takip için onları tekrar arayabilir misin?"
  },

  {
    role: "user",
    content: `
<agent_status>
Mevcut Durum:
- phone_call 3 kez çağrıldı
- Xfinity: 3/3 maksimum arama limitine ulaşıldı
- Şu anki saat: 2025-09-14 10:30:45
- TODO: [1] Planı iptal et — devam ediyor
</agent_status>
`
  }
]
```

Özetle:

> **Statik bilgiyi değiştirme, dinamik durumu context’in sonuna ekle.**

Böylece Agent hem güncel görev durumunu doğrudan görür hem de daha önce cache’lenmiş system prompt ve diğer sabit içerikler korunur.

### 2.6.4 Durum Güncellemelerinin İki Uygulaması ve Cache Maliyetleri

Agent Durum Çubuğu güncellendiğinde iki temel yaklaşım vardır:

### 1. Her turda değiştirme

Önceki durum mesajı silinir, güncel durum context’in sonuna eklenir.

**Avantajı:**

- Context’te yalnızca güncel durum bulunur.
    
- Eski durumlar birikmez.
    

**Dezavantajı:**

- Eski mesajın silinmesi, o noktadan sonraki cache’i geçersiz kılar.
    
- Ancak durum mesajı genellikle context’in sonuna yakın olduğu için yalnızca kısa bir son ek yeniden hesaplanır.
    

### 2. Kalıcı ekleme

Eski durum mesajları silinmez; her turda yeni bir durum mesajı eklenir.

**Avantajı:**

- Context yalnızca eklenerek ilerlediği için KV Cache açısından daha verimlidir.
    
- Hiçbir önceki içerik değiştirilmez.
    

**Dezavantajı:**

- Eski durumlar context’te birikir.
    
- Token tüketimi artar.
    
- Modelin eski ve güncel durumları ayırt etmesi gerekir.
    

Claude Code’daki `&lt;system-reminder&gt;` yaklaşımı bu modele örnektir.

Karar genel olarak şu faktörlere bağlıdır:

- Durum mesajının uzunluğu
    
- Güncellemelerin sıklığı
    
- Güncellemeler arasında eklenen mesaj miktarı
    
- Toplam trajectory uzunluğu
    
- Cache’li ve normal token maliyetleri
    

Yaklaşık karşılaştırma:

```text
C_değiştirme ≈ (N - 1)(1 - α)R

C_ekleme ≈ αSN(N - 1) / 2
```

Burada:

- `S`: Her durum mesajının token sayısı
    
- `R`: Güncellemeler arasında eklenen token sayısı
    
- `N`: Toplam güncelleme sayısı
    
- `α`: Cache’li girdinin normal girdiye göre maliyet oranı
    

Yaklaşık seçim kuralı:

```text
αSN / 2 &lt; (1 - α)R
```

Bu koşul sağlanıyorsa kalıcı ekleme, aksi durumda her turda değiştirme daha uygun olabilir. Gerçek seçimde sağlayıcının fiyatlandırması, cache isabet oranı ve context sınırları da ölçülmelidir.

Durum çubuğu kod tarafından oluşturulmalıdır. Sayımlar, limitler ve görev durumu mümkün olduğunca programatik biçimde hesaplanmalıdır. LLM’ye “kaç arama yapıldı?” gibi bilgileri yeniden saydırmak risklidir; model durum çubuğunda `3 arama yapıldı` yazıyorsa bunu çoğu zaman doğrulamadan doğru kabul eder.

Ayrıca ham context silinmemelidir. Durum çubuğu, orijinal geçmişin kayıplı bir özetidir. Sayım ve ilerleme takibi için yeterli olabilir; ancak daha sonra özetlenmemiş başka bir ayrıntıya ihtiyaç duyulursa yalnızca durum çubuğuna güvenmek doğruluk kaybına yol açar.

Özetle: **Durum çubuğu güncel durumu hızlı erişilebilir hale getirir; kodla üretilmeli, cache maliyeti ile token birikimi arasında denge kurulmalı ve orijinal context’in yerine tamamen geçirilmemelidir.**

## 2.7 Context Sıkıştırma Stratejileri

Bu bölüm, çok turlu etkileşimlerde büyüyen context’in neden ve nasıl sıkıştırılması gerektiğini açıklıyor.

### 2.7.1 Sıkıştırma Neden Gereklidir: Sadece Bir Uzunluk Sorunu Değil

Context sıkıştırmanın üç temel nedeni var:

1. **Uzunluk ve maliyeti azaltmak:** Context penceresi sınırlıdır. Tool çıktıları ve uzun konuşmalar pencereyi hızla doldurabilir. Daha fazla token ayrıca API maliyetini ve yanıt gecikmesini artırır.
    
2. **Düşünme kalitesini artırmak:** Context penceresi henüz dolmamış olsa bile ham bilgilerin çok fazla ve dağınık olması modelin önemli noktaları bulmasını zorlaştırır. Bunun yerine bilgiler, “bilinenler, belirsiz kalanlar ve eksik bilgiler” şeklinde yapılandırılmış bir özete dönüştürülürse model daha odaklı ve etkili karar verebilir.
    
3. **Context kaygısını azaltmak:** Model, context penceresinin dolmak üzere olduğunu algıladığında görev tamamlanmadan sonuçlandırmaya çalışabilir. Context’i önceden sıkıştırmak, bu baskıyı azaltarak karar kalitesini yükseltebilir.
    

Kısacası, context sıkıştırma yalnızca yer açmak için değil; maliyeti düşürmek, bilgiyi daha kullanılabilir hâle getirmek ve modelin görev boyunca istikrarlı düşünmesini sağlamak için gereklidir.

### 2.7.2 Context İçi Öğrenmenin İçsel Mekanizması: Retrieval, Reasoning Değil

Attention mekanizması, context içindeki belirli bilgileri bulmada başarılıdır; ancak uzun kayıtlar üzerinden otomatik olarak sayım yapmak, istatistik çıkarmak ve bilgileri birleştirmek daha çok reasoning gerektirir. Bu nedenle modelin, uzun bir kaydı her soruda yeniden tarayarak sonuç üretmesini beklemek verimli değildir.

Context sıkıştırma bu eksikliği giderir. Ham kayıtlar bir kez işlenerek doğrudan kullanılabilir sonuçlara dönüştürülür. Örneğin 100 kafeslik denetim günlüğü yerine context’e “90 siyah kedi, 10 beyaz kedi” bilgisi eklenebilir. Böylece model aynı sayımı her seferinde yeniden yapmak zorunda kalmaz; daha önce reasoning gerektiren bir sonuç, sonraki adımlarda doğrudan erişilebilir bilgiye dönüşür.

Bu yaklaşım, durum çubuğuyla benzer bir işleve sahiptir:

- **Durum çubuğu:** Önceden hesaplanmış bilgiyi context’e ekler.
    
- **Context sıkıştırma:** Şişkin ham içeriği, önceden damıtılmış bir özetle değiştirir.
    

Aralarındaki temel fark, durum çubuğunun çoğunlukla kod tarafından deterministik biçimde güncellenmesi; sıkıştırmanın ise büyük bir metin parçasını özetlemek için genellikle ayrı bir LLM çağrısı kullanmasıdır.

Uzun context’in sorunu yalnızca pencerenin dolması değildir. İçerik hâlâ sığıyor olsa bile model önemli bilgiyi bulmakta zorlanabilir. Bu duruma **context çürümesi** denir. Context taşması “içerik sığmıyor”, context çürümesi ise “içerik sığıyor ama bulunamıyor” anlamına gelir. İlgisiz içerik çoğaldıkça attention daha fazla token’a dağılır; model kritik ayrıntıları gözden kaçırabilir veya daha önce çözülmüş sorunları tekrar ele alabilir.

Bu nedenle context tasarımında amaç, modele mümkün olduğunca çok ham bilgi vermek değil, bilgiyi aktif olarak damıtıp yüksek yoğunluklu temsillere dönüştürmektir. Özetleme için ek LLM çağrıları kullanmak kısa vadede maliyet oluştursa da, sonraki adımlarda daha az token, daha hızlı erişim ve daha tutarlı reasoning sağlar.

Son olarak context içi öğrenme, modelin örnekler ve özetler aracılığıyla mevcut oturum içinde geçici olarak özelleşmesini sağlar. Ancak model parametreleri değişmediği için bu etki kalıcı değildir; oturum sona erdiğinde kaybolur.

### 2.7.3 Sıkıştırma ve KV Cache: Görünüşte Çelişkili, Aslında Tamamlayıcı

Sıkıştırma, KV Cache’in context ön ekinin değişmeden kalması gerekliliğiyle çelişiyor gibi görünür. Ancak bu çelişki, sıkıştırmanın **tek bir API çağrısı sırasında değil, iki çağrı arasındaki mesaj ön işleme aşamasında** yapılmasıyla çözülür.

Bu süreçte:

- **System Prompt ve tool tanımları değiştirilmez.** Bunlar context’in başındaki statik ön ektir ve KV Cache tarafından sürekli yeniden kullanılabilir.
    
- **Sıkıştırılan bölüm konuşma geçmişindeki tool sonuçlarıdır.** Agent çerçevesi ham tool çıktısını özetle değiştirdiğinde, değişiklikten sonraki cache geçersiz olur.
    
- **Değişiklikten önceki cache korunur.** Böylece tüm context’in önbelleği bozulmaz; yalnızca sıkıştırmanın gerçekleştiği noktadan sonraki bölüm yeniden hesaplanır.
    

Bu, bilinçli bir ödünleşmedir. Sıkıştırma yapılmazsa context büyümeye devam eder ve sonunda pencere sınırını aşarak görevin başarısız olmasına yol açabilir. Sıkıştırma yapıldığında ise cache’in bir bölümü kaybedilir; buna karşılık context uzunluğu kontrol altında tutulur ve bilgi daha yoğun bir biçimde temsil edilir.

Bu nedenle sıkıştırmanın zamanlaması önemlidir. Her turda sıkıştırmak cache’i gereksiz yere tekrar tekrar geçersiz kılar. Daha verimli yaklaşım, context belirli bir eşiğe yaklaşana kadar beklemek ve ardından tool sonuçlarını toplu biçimde sıkıştırmaktır.

Örneğin Agent’ın şu mesaj geçmişine sahip olduğunu düşünelim:

```text
1. System Prompt
2. Araç tanımları
3. Kullanıcı: İstanbul için hava durumunu araştır.
4. Agent: Hava durumu aracını çağır.
5. Tool: İstanbul’da bugün 18°C, yağış ihtimali %20.
6. Agent: Ulaşım seçeneklerini de araştır.
7. Tool: Metro ve otobüs bilgileri...
8. Kullanıcı: İkisini karşılaştır.
```

Burada **1 ve 2. mesajlar statik prefix** olduğu için cache’lenir ve sıkıştırmadan etkilenmez.

Agent, tool çıktılarının çok uzadığını fark ettiğinde 5 ve 7. mesajlardaki ham sonuçları şöyle sıkıştırabilir:

```text
1. System Prompt
2. Araç tanımları
3. Kullanıcı: İstanbul için hava durumunu araştır.
4. Agent: Hava durumunu araştırdı.
5. Özet: İstanbul’da hava 18°C, yağış ihtimali %20.
6. Agent: Ulaşım seçeneklerini araştırdı.
7. Özet: Metro daha hızlı, otobüs daha geniş güzergâh sunuyor.
8. Kullanıcı: İkisini karşılaştır.
```

Burada 5. mesajdan itibaren içerik değiştiği için, **5. mesajdan sonraki eski cache geçersiz olur**. Bir sonraki API çağrısında model:

- 1–4. mesajlara ait mevcut cache’i yeniden kullanır.
    
- 5–8. mesajları yeni, sıkıştırılmış hâlleriyle tekrar işler.
    
- Bu yeni bölüm için yeniden cache oluşturur.
    

Yani durum şu şekilde düşünülebilir:

```text
Korunan cache:
[System Prompt + Araç tanımları + sıkıştırma noktasından önceki içerik]

Yeniden oluşturulan cache:
[sıkıştırılmış özetler + sonraki mesajlar]
```

Önemli ayrıntı şu: Sıkıştırma, mesaj geçmişinin ortasındaki içeriği değiştirdiği için yalnızca o noktadan sonraki cache etkilenir. Önceki bölüm değişmediğinden korunabilir.

### 2.7.4 Üretim Düzeyinde Hiyerarşik Sıkıştırma Mekanizması

Üretim düzeyindeki Agent sistemleri genellikle tek bir sıkıştırma yöntemi kullanmaz. Farklı bilgilerin farklı sürelerde değer taşıdığı kabul edilir ve her bilgi türü için uygun sıkıştırma katmanı uygulanır. Bu yaklaşım, sıkıştırmayı basitten karmaşığa doğru ilerleyen **beş katmanlı bir hiyerarşi** olarak ele alır:

1. **Tool sonucu bütçesi:** Büyük tool çıktıları doğrudan context’te tutulmak yerine diske kaydedilir. Model yalnızca kısa bir önizleme veya özet görür. Hangi içeriklerin değiştirileceğine karar verildikten sonra cache tutarlılığını korumak için bu kararlar sabitlenir.
    
2. **Gürültüyü doğrudan silme:** Değeri düşük ve kısa süreli kullanılan içerikler özetlenmeden kaldırılır. Önemsiz bir arama sonucunu özetlemek bile gereksiz token harcamak olacağından, bazı içerikler için en iyi strateji doğrudan silmedir.
    
3. **API düzeyinde mikro sıkıştırma:** API’nin sunduğu context düzenleme özellikleri kullanılarak belirli tool sonuçlarının context’in ön ekinden kaldırılması istenir. Yerel mesaj listesi değişmediği için uygulama tarafında ek işlem gerekmez. Ancak kaldırılan noktadan sonraki cache geçersiz olur. Bu yüzden yöntem, context taşmak üzereyken kullanılmalıdır; sık sık çalıştırmak gereksiz cache yeniden oluşturma maliyeti doğurur.
    
4. **Arşivsel özetleme:** Her turda konuşmanın o ana kadarki mantığı yapılandırılmış bir özet hâline getirilir. Özetler sürekli tek bir kayıtta birleştirilmez; her tur için ayrı kayıtlar tutulur. Bu yaklaşım, konuşmanın zaman içindeki gelişimini ve mantıksal bağlarını korur.
    
5. **Tam sıkıştırma:** Diğer yöntemler yeterli olmadığında, LLM kullanılarak kapsamlı bir sıkıştırma yapılır. Önce oturum belleğinin sıkıştırılması denenir; bu başarısız olursa tam context sıkıştırılır. Tekrarlanan hataların maliyet üretmesini önlemek için bir **circuit breaker** kullanılır. Belirli sayıda ardışık başarısızlıktan sonra sistem yeni sıkıştırma denemelerini durdurur.
    

Özetle, üretim sistemlerinde sıkıştırma hemen en pahalı yöntemle başlamaz. Önce büyük çıktılar sınırlandırılır, gereksiz içerikler silinir, ardından API düzeyinde düzenleme ve arşivsel özetleme uygulanır. Tam LLM sıkıştırması ise son çare olarak kullanılır. Bu hiyerarşi hem bilgi kaybını hem de gereksiz cache ve API maliyetlerini azaltır.

## Sıkıştırma Stratejileri için Tasarım İlkeleri

Etkili sıkıştırma yalnızca token sayısını azaltmayı değil, görev için önemli bilgiyi koruyarak context’i daha kullanışlı hâle getirmeyi amaçlar. Bu nedenle strateji tasarlanırken dört ilke dikkate alınmalıdır:

1. **Bilgi değerinin eşit olmaması:** Kritik kararlar, destekleyici kanıtlardan daha değerlidir; gereksiz arayüz metinleri ve reklamlar ise en düşük değere sahiptir. Sıkıştırma, önce düşük değerli gürültüyü azaltmalı ve kararları destekleyen bilgiyi korumalıdır.
    
2. **Semantik bütünlük:** Özet, önemli ayrıntıları silemez. Örneğin “Sutskever, OpenAI’den Mayıs 2024’te ayrıldı” ifadesi yalnızca “Sutskever ayrıldı” biçiminde özetlenemez; tarih ve kurum adı anlamın temel parçalarıdır.
    
3. **Görev ilgisi:** Aynı içerik, göreve göre farklı biçimde sıkıştırılmalıdır. Kurucular listesini bulmaya çalışan bir Agent ile bir kişinin kariyer geçmişini inceleyen Agent’ın koruması gereken bilgiler aynı değildir.
    
4. **Sıkıştırma anlamaktır:** İyi sıkıştırma, metni mekanik olarak kısaltmak değil, içeriğin özünü anlayıp daha yoğun ve kullanılabilir bir biçimde yeniden ifade etmektir. Açıkça üretilen özetler ayrıca farklı oturumlarda incelenebilir ve yeniden kullanılabilir.
    

Sıkıştırma ek LLM çağrıları gerektirse de, azaltılan token maliyeti ve artan görev başarısı bu maliyeti çoğu zaman karşılar. Özellikle mimari kararların gerekçeleri, kısıtların nedenleri ve başarısız denemeler kolayca kaybolduğu için Agent’ın önemli ilerlemeleri düzenli olarak belgeye yazması gerekir. Bilgiyi yalnızca dağınık konuşma geçmişinde bırakmak yerine, ilerleme belgeleri oluşturulmalı ve güncellenmelidir. Model bunu kendiliğinden yapmıyorsa prompt veya Skill ile bu davranış açıkça teşvik edilmelidir.

## Sıkıştırma Yerine Alt Agent Context İzolasyonu

Sıkıştırma, fazla bilgiyi context’e aldıktan sonra azaltır. **Alt Agent Context İzolasyonu** ise büyük miktardaki ara bilgiyi baştan ana context’e sokmamayı amaçlar.

Örneğin ana Agent’ın kod tabanında ödeme callback’lerini aradığını düşünelim. Ana Agent onlarca dosyayı kendisi incelerse, context’e on binlerce token ham kod eklenebilir. Hedef fonksiyon bulunduktan sonra bu kodların çoğu yalnızca gürültüye dönüşür ve daha sonra sıkıştırılması gerekir.

Bunun yerine görev bir alt Agent’a devredilebilir:

```text
Ana Agent → Alt Agent: Ödeme callback’lerini bul.
Alt Agent → Ana Agent: handle_callback, src/payment/callbacks.py içinde.
```

Ana Agent yalnızca görevi ve kısa sonucu görür. Alt Agent’ın arama sırasında kullandığı dosyalar, ara çıktılar ve diğer ayrıntılar alt Agent’ın context’iyle birlikte dışarıda kalır. Böylece ana context daha temiz kalır ve ana Agent’ın KV Cache prefix’i gereksiz yere bozulmaz.

Bu yaklaşımın temel farkı şöyledir:

- **Sıkıştırma:** Gürültüyü context’e aldıktan sonra azaltır; sonradan yapılan ve kayıplı bir işlemdir.
    
- **İzolasyon:** Gürültünün ana context’e hiç girmemesini sağlar; bu nedenle çoğu durumda daha temiz ve verimlidir.
    

Ancak alt Agent, ana Agent’ın tüm context’ini görmez. Bu yüzden alt Agent’a verilen görev açık, hedefi belirli ve kendi başına anlaşılabilir olmalıdır. Eksik veya belirsiz bir görev tanımı, alt Agent’ın yanlış şeyi aramasına yol açabilir.

Sonuç olarak alt Agent izolasyonu, özellikle geniş arama, kod inceleme ve araştırma gibi çok miktarda ara veri üreten görevlerde sıkıştırmaya güçlü bir alternatiftir. Ana Agent’a yalnızca karar vermek için gerekli sonuç aktarıldığından, context daha küçük, daha odaklı ve daha kararlı kalır.