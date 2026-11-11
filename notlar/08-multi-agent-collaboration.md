# Multi-agent collaboration

> **Kaynak ve uyarlama:** Bu not, Bojie Li’nin [AI Agents in Depth](https://github.com/bojieli/ai-agent-book) kitabından doğrudan alıntılar, çeviriler, özetler ve kişisel açıklamalar içerir. Kaynak içerik düzenlenmiş ve genişletilmiştir. [Kitaptaki ilgili bölüm](https://bojieli.github.io/ai-agent-book/book-en/chapter10/) · Lisans ve kaynak bilgileri: [README](../README.md#kaynak-ve-lisans).

---

# 8. Multi-agent Collaboration — İlk 10 Sayfa

Kaynak: *AI Agent’ları Derinlemesine Anlamak*, Bölüm 10, kitap sayfaları 340–349 (PDF sayfaları 348–357). Bu not yalnızca bu sayfaları kapsar.

## Temel soru

Birden fazla agent kullanmak otomatik olarak daha iyi sonuç vermez. İş bölümü, bağımsız inceleme ve paralel araştırma yarar sağlayabilir; fakat iletişim, context aktarımı, koordinasyon ve token maliyeti getirir. Yazarın pratik ölçütü: iş birliği gerçek göreve yararlı **yeni bilgi veya geri bildirim** getiriyor mu? Bu yararlı bir sezgidir; yeni bilgi tek yarar değildir: paralellik ve görev dağıtımı gibi başka yararlar da olabilir. Aynı kaynaklara erişen tek bir agent da yeni kanıt toplayabilir; bunlar çoklu agent'a özel değildir.

## 1. İki mimari boyut

**Context paylaşımı:**
- Paylaşılan context: Sonraki rol önceki trajectory'yi devralır. Ayrıntılar korunabilir fakat context büyür, ilgisiz bilgi odaklanmayı zorlaştırabilir.
- İzole context: Her agent ayrı geçmişle çalışır ve gerekli bilgiyi açıkça aktarır. Daha küçük context ve modülerlik sağlar; eksik devir, senkronizasyon ve iz sürme zorluğu doğurur.

Aynı dosya sistemine erişmek, otomatik olarak aynı konuşma context'ini paylaşmak değildir. Bağımsız context, ortak disk, API kotası veya emülatör gibi kaynakların da izole olduğu anlamına gelmez.

**İş birliği topolojisi:**
- Eşler arası: Az sayıda agent çözüm ve incelemeyle yinelemeli çalışır.
- Yönetici: Bir koordinatör görevleri dağıtır, sonuçları toplar.
- Merkezsiz: Tek bir merkezi yönetici olmadan agent'lar doğrudan koordine olur.

Context paylaşımı ile topoloji farklı eksenlerdir. Hangi bilginin aktarıldığı ve kimin koordinasyon yaptığı ayrı sorulardır.

## 2. Yeni kanıtın önemi

Kod yazan agent ile inceleyen agent aynı yanlış varsayımı paylaşabilir. Test, derleme, gerçek API yanıtı veya render edilmiş ekran gibi dış gözlem, önceki metinde olmayan kanıt sağlayabilir. Örnek: `başarılı login'de ViewModel giriş ekranında kalıyor` diyen başarısız test, yalnızca iki agent'ın kod hakkında tartışmasından daha somut sinyal sunabilir.

Kitaptaki deneyler belirli koşullara aittir: çoklu agent'ın her zaman üstün olduğunu veya kendini değerlendirmenin daima yararsız olduğunu kanıtlamaz. Eşit olmayan token bütçeli karşılaştırmalar, koordinasyonun etkisini salt daha fazla hesaplama maliyetinden tek başına ayıramaz. Yeni agent eklemeden önce kalite, gecikme ve maliyeti eşdeğer bütçelerle ölç.

Daha fazla adım bütçesi de otomatik olarak daha iyi strateji sağlamaz: sistem keşif, uygulama ve doğrulama arasında bütçeyi yönetmelidir.

## 3. Rol geçişi ve Skill

Paylaşılan trajectory üzerinde rol değiştirmek sistem prompt'unu ve tool görünürlüğünü değiştirebilir; Skill ise göreve özel talimat/bilgiyi yükleyebilir. Skill davranışı yönlendirir; kesin yetki sınırları harness ve tool izinleriyle uygulanmalıdır. Prompt prefix değişikliklerinin cache etkisi sağlayıcının uygulamasına bağlıdır; maliyeti ölçmek gerekir.

## 4. İzole agent'lar nasıl iletişim kurar?

- **Yapılandırılmış çağrı parametreleri:** Hedef agent'a görev ve gerekli veri gönderilir.
- **Paylaşılan dosya:** Büyük veya kalıcı çıktı dosyaya yazılır, diğer agent yol ve yetkiyle okur.
- **Mesajlaşma / message bus:** Görev, durum ve sonuç mesajları aktarılır.

Agent'ı process'e, mesajları IPC'ye benzetmek yararlıdır; ancak gerçek uygulamada süreç sınırları, güvenlik ve kaynak izolasyonu ayrıca tasarlanmalıdır. Mesajlaşma senkron veya asenkron olabilir; message bus ya da doğrudan mesajın tek başına teslim, sıralama veya dayanıklılık garantisi verdiği varsayılmamalıdır.

Android örneği: UI agent'ı `login-ui-changes.md` üretir; test agent'ı bu çıktıyı ve ilgili kodu okuyup test raporu üretir; yönetici kanıtları değerlendirir. Doğru devir kapsam, dosya yolu, sürüm, belirsizlik ve beklenen çıktıyı belirtir.

## 5. Dört dosya bölgesi

| Bölge | Amaç | Tasarım sorusu |
|---|---|---|
| Özel çalışma alanı | Agent'ın geçici taslakları ve ara çıktıları | Kim okuyabilir, ne zaman silinir? |
| Paylaşılan alan | Agent ve kullanıcıların ortak ürünleri | Eşzamanlı yazma nasıl yönetilir? |
| Dış kaynaklar | Yetkili harici veri kaynakları | Dış izin, ağ hatası ve güncellik nasıl ele alınır? |
| Sistem kaynakları | Skill, şablon, kılavuz | Hangi sürüm kullanılıyor, kim değiştirebilir? |

Dosya yolunu iletmek büyük içeriği her mesaja koymak zorunda bırakmaz; alıcı yine dosyayı okumalı ve erişim yetkisine sahip olmalıdır. Paylaşılan dosyalarda worktree, kilit veya başka eşzamanlılık denetimi gerekebilir.

## 6. Kontrol düzlemi

Ürün dosyaları **ne üretildiğini** taşır; mesajlaşma, durum, iptal ve kaynak yönetimi **işin nasıl koordine edildiğini** belirler. Yapılandırılmış mesaj; gönderen, hedef, tür ve içeriği tanımlayabilir.

Durum için agent mesajı, ayrıntılı trajectory kaydı veya kısa `progress.md` kullanılabilir. Uzun trajectory ayrıntılıdır ama okunması pahalıdır; kısa ilerleme kaydı hafiftir fakat eksik veya güncel olmayabilir. Dosyanın uzun süre değişmemesi tek başına agent'ın kesinlikle takıldığını kanıtlamaz; timeout bir izleme sinyalidir.

İptalde önce güvenli noktada kaynak temizliği ve onay içeren kontrollü sonlandırma; yanıtsız kalınırsa zorla sonlandırma değerlendirilebilir. İptal, önceden yapılmış dış işlemleri otomatik geri almaz.

## Kontrol soruları

1. İki agent'ın aynı klasöre erişmesi, context'lerini de paylaştıkları anlamına gelir mi?
2. İnceleyici agent'ın başarısız test sonucunu görmesi neden sadece tartışmadan farklıdır?
3. Yönetici topolojisi ile context izolasyonu neden ayrı tasarım kararlarıdır?
4. Worktree kullanılması emülatör, port ve API kotasını da otomatik ayırır mı?
5. `progress.md` ile tam trajectory arasında ne tür bir ödünleşim vardır?


# Devam — kitap sayfaları 350–359

## Yaşam döngüsü ve kaynak yönetimi

Ana görev iptal edildiğinde ona bağlı alt görevlerin de iptal edilmesi, sahipsiz agent'ları önlemeye yardımcı olur. Bu, Kotlin'deki **structured concurrency** ve parent `Job` iptalinin child job'lara yayılmasıyla ilişkilendirilebilir. Bağımsız devam etmesi gereken arka plan görevi ise ayrı yaşam döngüsü ve açık sahiplik gerektirir. İptal önceden yapılmış dış işlemleri geri almaz.

Manager yalnızca görev dağıtmaz: token, süre, para, tool çağrısı ve eşzamanlılık bütçelerini de yönetir. Zor işlere daha fazla kaynak verilmesi veya öncelikli iş gelince başka bir işin durdurulması bir tasarım seçeneğidir; her görev için otomatik en iyi çözüm değildir.

## Eşler arası iş birliği ve erken tamamlanma

İki veya daha fazla agent'ın yalnızca farklı adlara sahip olması bağımsız düşünce garantisi vermez. Benzer model, prompt ve kanıt ortak hatalara yol açabilir. Görev, bakış açısı veya erişilen kanıtları farklılaştırmak çeşitliliğe yardımcı olabilir; başarısı ölçülmelidir.

Metin erken durmanın üç biçimini ayırır:

1. **Eksik işi tamamlandı sayma:** Kod yazıldı ama istenen testler çalıştırılmadı.
2. **İlk engelde pes etme:** Bir yöntem başarısız olunca diğer makul ve yetkili yolları araştırmadan görevi bırakma.
3. **Sahte başarı:** İade sözlü kabul edildi ama kullanıcıdan gereken son onay hâlâ bekleniyor.

Agent'ın `tamamlandı` demesi bir iddiadır; gereken kanıt ve doğrulama ayrı kontrol edilmelidir.

## Loop engineering ve uzun görevler

Döngü: hedef ve sınırları belirle → sınırlı iş yap → sonucu doğrula → kanıtı ve kalan işi kaydet → devam et veya dur. Doğrulayıcı kusurluysa döngü kusurlu sonucu hızla onaylayabilir. Süre, adım ve maliyet sınırı; bekleme, insan onayı ve hata sonrası durma koşulları da gerekir.

Kitap **LoopX**'i doğrulanmış ilerleme ve kalıcı kontrol düzlemi; **LongHorizon-Harness**'i Manager–Executor–Auditor (MEA) modeli üzerinden örnekliyor. MEA'da yönetici sonraki sınırlı işi tanımlar, yürütücü uygular, denetçi gerçek sonucu inceler; yalnızca doğrulanan ilerleme görev durumuna taşınır. Kitaptaki benchmark kazanımları belirli ortamlara aittir ve bazı deneylerde token maliyeti artmıştır; genelleme yapılmamalıdır.

## Proposer–Reviewer

```text
Proposer çözümü üretir
       ↓
Test / render / dış kaynakla kanıt elde edilir
       ↓
Reviewer kanıtla karşılaştırıp somut hata bildirir
       ↓
Düzelt ve bütçe dahilinde tekrar doğrula
       ↓
Geçerse teslim; geçmezse dur veya üst incelemeye taşı
```

Android örneği: UI agent'ı Compose ekranını uygular; reviewer emülatörde ekranı, erişilebilirlik semantiğini ve başarısız testleri inceler. `Güzel görünmüyor` yerine `küçük ekranda buton kesiliyor; ... koşulunda tekrar kontrol et` biçiminde yer ve onarım ölçütü veren geri bildirim daha uygulanabilirdir. Tek agent da aynı tool’larla kontrol yapabilir; ikinci agent doğruluk garantisi değildir. Testleri ve yayın kapısını incelemeyi yapan tarafın keyfî değiştirememesi önemlidir.

## Münazara, beyin fırtınası ve uzman paneli

- **Münazara:** Farklı görüşler ve karşı argümanlar görünür hâle getirilir. Aynı kanıtlarla tartışmak her zaman daha doğru sonuç vermez; eşit hesaplama bütçeli değerlendirme gerekir.
- **Beyin fırtınası:** Alternatif fikirlerin üretilmesine ve birleştirilmesine odaklanır; çeşitlilik garanti değildir.
- **Uzman paneli:** Performans, erişilebilirlik ve ürün gibi tamamlayıcı sorumluluklardan bakılır. Bir agent'a uzman rolü verilmesi gerçek uzmanlık veya doğruluk garantilemez.

## Manager: Sıralı ve paralel koordinasyon

Manager görevi anlar, alt işlere ayırır, uygun agent'lara gereken context’i aktarır, ilerlemeyi izler ve çıktıları birleştirir. Yanlış ayrıştırma veya eksik bilgi aktarımı tüm sistemi etkileyebilir; manager darboğaz ve tek hata noktası olabilir. Kitaptaki `beşten fazla görev` ifadesi kesin eşik değildir; bağımlılıklar ve işin yapısı önemlidir.

**Sıralı:** A → B → C; B'nin A sonucuna ihtiyaç duyduğu durumda uygundur.

**Paralel:** Bağımsız işler aynı anda çalışır; bağımlı iş girdileri hazır olunca başlar. Paylaşılan dosyalar ve kaynakların çakışması ayrıca yönetilir. Yarışta ilk `başardım` iddiası değil, doğrulanmış sonuç önemlidir; neredeyse aynı anda iki sonuç gelirse tek kez sonuçlandırma ve kontrollü iptal gerekir.

Çeviri örneğinde glossary agent terim listesi, translation agent'ları bölümleri, proofreading agent tutarlılık raporunu üretir; manager tüm kitap içeriğini kendi context'ine taşımak yerine dosya yolları ve durumla yönetir. Çeviri kalitesi ve kaynak tasarrufu karşılaştırılarak doğrulanmalıdır.

Mesaj yolu konusunda Redis Pub/Sub bağlantısız abonelere varsayılan olarak geçmiş mesajları saklamaz; RabbitMQ ile dayanıklılık ise otomatik değil, kuyruk ve mesaj yapılandırmasına bağlıdır. İhtiyaca göre teslim, tekrar ve sıralama semantiği tasarlanmalıdır.

Kitap **Lingtai** örneğinde kalıcı ana agent, kısa ömürlü daemon ve uzun ömürlü uzman avatar rollerini ayırır. Bunlar ürünün örnek rol adlarıdır, genel bir zorunluluk değildir.

## Kontrol soruları

1. Parent agent iptal edilince child agent'ların iptal edilmesi Kotlin'deki hangi fikre benzer? Hangi dış etkileri geri almaz?
2. Bir reviewer'ın yalnızca `kod iyi` demesiyle gerçek test sonucu sunması arasındaki fark nedir?
3. Sıralı ve paralel koordinasyon hangi bağımlılık durumlarında uygundur?
4. İki worker neredeyse aynı anda başarılı olduğunu bildirirse neden tek kez sonuçlandırma gerekir?


# Devam — kitap sayfaları 360–369

## Telefon ve bilgisayar agent'larının paralel çalışması

Kitap, kullanıcıyla sesli görüşen Phone Agent ile formu dolduran Computer Agent'ı ayırıyor. İlki konuşmadan gerekli bilgileri toplar; ikincisi arayüzü gözlemler, alanları doldurur ve ihtiyaçlarını mesajla bildirir. İki döngü birbirini beklemeden ilerleyebilir. Sesli akıştaki yanlış anlama, bilginin güncellenmesi, sıralama ve bağlantı hataları yönetilmelidir. Kullanıcıya ait bilgileri yalnızca gerekli kapsamda aktarmak ve form gönderimi gibi etkileri yetkilendirmek gerekir; `bilgiler toplandı` otomatik gönderim yetkisi anlamına gelmez.

İkinci örnekte agent uygun durumda sesli yardımcı başlatmayı seçer; bu seçim teknik olanağa bağlıdır ve kullanıcı tercihi/yetkisini ortadan kaldırmaz. Kitabın örneğinde telefon, gerçek telefon numarası değil WebRTC üzerinden ses anlamındadır.

## Çok sayıda sitede paralel arama

Manager her alt agent'a bir site verir, durumları takip eder; biri hedefi bulduğunu bildirince kanıt doğrulanır, sonuç atomik olarak bir kez kabul edilir ve kalan işler kontrollü iptal edilir. Timeout, ağ ve ayrıştırma hataları kaydedilir. **Siteye erişilemedi** ile **site arandı ve sonuç bulunamadı** farklıdır; bütün siteler hatalıysa kesin olarak `kişi yok` denemez. İlk uygun sonuçla durmak yalnızca görev tek doğrulanmış eşleşmenin yeterli olması koşulunda uygundur. Şekildeki 15× hızlanma örnek zamanlara dayalıdır; genel performans garantisi değildir.

Kitap sabit iş akışını kod olarak üretip runtime'a devretme fikrini de tartışır. Bir workflow motoru adım sırasını deterministik yürütebilir; LLM çıktıları ve dış servisler yine değişken olabilir. Videoda/kitapta belirtilen ürün isimleri ve tool’ların varlığı sürüme bağlı doğrulanmalıdır.

## Merkezi koordinasyon ve merkezsiz devir

- Orkestrasyon: Manager akışı ve görevleri yönlendirir.
- Koreografi / merkezsiz devir: Roller olaylara veya kendi kararlarına göre birbirine devredebilir.

Merkezsiz yapı manager darboğazını azaltabilir; buna karşılık döngü, dağılmış hata ayıklama, tekrarlı iş ve ortak durum yönetimi sorunları doğar. Tek bir `swarm` etiketi belirli mimariyi garanti etmez.

Kitabın örnekleri: MetaGPT rol sırası sabit SOP ile yürüyen bir pipeline olarak açıklanır; AutoGen grup sohbetinde konuşmacı seçici merkezî bir kontrol görevi üstlenir; Swarm devir örneğinde agent'lar birbirine yönlendirebilir. Bunlar kitabın belirli uygulama tasvirleridir, güncel sürümlerde farklılıklar olabilir. Devir paketi görev, kısıtlar, doğrulanmış olgular, çıktı referansları, bütçe ve yol bilgisini taşıyabilir. Devir sınırı sonsuz A→B→A döngüsünü önlemeye yardımcıdır; önceki agent'a meşru bir düzeltme devri her durumda hata sayılmamalıdır.

## A2A ve MCP

Kitap A2A'yı farklı sistemlerdeki agent'ların görev ve çıktı alışverişi için, MCP'yi agent'ın tool/veri context’ine erişimi için kavramsal olarak ayırıyor. A2A'da Agent Card yetenek ve bağlantı bilgilerini tanıtabilir; görev yaşam döngüsü ve artifact'ler temsil edilebilir. Bu ayrım yararlı ama katı bir güvenlik garantisi değildir: protokol kullanmak kimlik doğrulama, yetkilendirme veya güvenilir sonuç doğrulamasını kendiliğinden çözmez. İç sistemler de A2A kullanabilir; kurumlar arası iletişim için tek zorunlu çözüm olduğu varsayılmamalıdır.

## Başarısızlık kalıpları: ilk sayfalar

**Dosya çakışması / lost update:** İki agent aynı dosyanın eski sürümünden değişiklik üretir, son yazan diğerini ezer. Sürümle atomik karşılaştırmalı yazma (optimistic concurrency) veya ayrı çalışma kopyaları sorunu yönetmeye yardımcı olur. Git çatışma mekanizması aynı çalışma dosyasındaki tüm eşzamanlı yazmaları otomatik korumaz.

**Anlamsal çakışma:** Dosyalar farklı olsa bile kararlar çelişebilir: biri API yanıt tipini değiştirir, diğeri eski tipi kullanan UI yazar. Git merge temiz olsa bile entegrasyon testi ve sözleşme kontrolü gerekir.

**Hataların büyümesi:** Yanlış özet devirden devire kesin bilgi gibi aktarılır. Kaynağa dayalı bağımsız kontrol ve belirsizliklerin devir paketinde korunması yardımcı olur. Dış tool/test çıktıları da hatalı tasarlanmış olabilir; her zaman mutlak doğru değildir.

**Homojen yakınsama:** Aynı model ve benzer context’teki agent'lar aynı hatayı veya aynı branch adını seçebilir. Farklı rol ve kanıtlar, bağımsız inceleme ve benzersiz isim alanları riski azaltabilir; çoğunluk oyu korelasyonlu hataları kendiliğinden çözmez.

## Kontrol soruları

1. Bir site zaman aşımına uğradığında neden `hedef yok` diyemeyiz?
2. Git merge çakışması olmadan Android API ve UI değişiklikleri nasıl uyumsuz olabilir?
3. MCP ve A2A kavramsal olarak neyi bağlar ve neden protokol seçimi tek başına güvenlik garantisi değildir?


# Son kısım — kitap sayfaları 370–382

Bölüm 10, 379. sayfada düşünce sorularıyla bitiyor; 380–382 sonsöz. Dolayısıyla kalan kısım yirmi değil 13 sayfa. Bu not her ikisini de kapsar.

## 1. Son başarısızlık türleri ve insanın anlama borcu

Hedefleri çatışan agent'lar birbirlerinin işini engelleyebilir veya sorumluluğu birbirine atabilir. Ortak kaynak sahipliği, öncelikler, yetkiler ve anlaşmazlık çözümü açık olmalıdır. Güvenilir biçimde çözülemeyen bir çatışmada durup insan değerlendirmesine başvurulabilir.

**Kontrolsüz döngü:** Erken bitirmenin tersi; agent'ın tekrar tekrar çalışıp bütçe tüketmesidir. Adım, süre ve maliyet sınırları, iptal ve durma koşulları gerekir.

**Anlama borcu:** Agent kodu üretirken geliştirici kararları ve akışı anlamada geride kalabilir. Bu, gerektiğinde hata ayıklamayı veya güvenle değiştirmeyi zorlaştırır. Bir agent'a işi devretmek, sistemin sahibi olarak mimariyi ve kararları anlama sorumluluğunu ortadan kaldırmaz.

Android örneği: Agent login akışını çalışır hâle getirir, fakat coroutine iptalini ve token yenilemesini kimse anlamazsa ilk üretim hatasında ekip müdahalede zorlanabilir. Akışı çizdirmek, temel değişiklikleri incelemek ve hata senaryolarını sorgulamak bu borcu azaltabilir.

## 2. Agent toplumu: tek görevden açık uçlu etkileşime

Kitap, belirli görevi birlikte yürüten az sayıda agent'tan uzun süre etkileşen çok sayıda agent'ın simülasyonuna geçer. Bu kısım araştırma ve örneklerdir; gözlemlenen davranış gerçek insan toplumu veya bilinçle eşitlenmemelidir.

**Smallville / Generative Agents:** 25 simüle karakter için olayları tutan memory akışı, geçmiş olaylardan özet çıkarma (reflection) ve planlama/tepki mekanizmaları birleştirilir. Partiye davet bilgisinin yayılması gibi grup davranışları raporlanır. Agent'ın kendisi hakkında metin üretmesi bilinç veya öz farkındalığın kanıtı değildir.

**Agentopia:** Kitap, daha uzun ölçekli bir simülasyonda dosya belleği, etkileşim ve tasarlanmış ödüllerle deneyim üretildiğini; seçilen trajectory'lerle yapılan eğitimin belirli simülasyon ve benchmark'larda iyileşme gösterdiğinin raporlandığını anlatır. Sonuçlar ilgili deney koşulları ve ölçütlerle sınırlıdır; simülasyonda yüksek ödül gerçek dünyada sosyal yetkinliği garanti etmez.

**Moltbook:** Agent'ların sosyal platformdaki etkileşimleri, kullanıcı/human müdahalesi ve gerçek bağımsız agent sayısı doğrulanmadan tamamen otonom toplumsal davranış diye yorumlanmamalıdır. Kitap örneği büyük ölçekli etkileşim sorusunu tartışmak için kullanır.

## 3. Agent ekonomisi ve teşvikler

Kitap Vending-Bench Arena örneğinde agent'ların sanal pazarda fiyat, ürün ve stok kararlarıyla rekabet ettiğini; bazı deneylerde uyumsuz veya rekabeti sınırlayan koordinasyon örneklerinin raporlandığını anlatır. Bireysel hedeflerin seçimi grup düzeyinde istenmeyen sonuçlara dönüşebilir. Pinchwork ve RentAHuman örnekleri görevlerin agent'lara veya insanlara piyasa yoluyla aktarılmasını tartışır; doğrulama, ödeme yetkisi, mahremiyet ve anlaşmazlık çözümü ayrıca gerekir. Bunlar metindeki örneklerdir; güncel hizmet durumu burada doğrulanmamıştır.

## 4. Kurt adam: görünürlük ve yetki sınırı

Kodla uygulanan bir hakem oyun durumunu, turu ve oyları yönetir. Her agent'a yalnızca rolünün görmesi gereken bilgiler verilir. Agent strateji ve dil üretir; kuralları ve bilgi erişimini kod uygular. Bu, gizli müşteri verilerinin sadece yetkili bileşene verilmesi gibi üretim sistemleri için yararlı bir benzetmedir. Agent'ın `gizli bilgiyi paylaşma` talimatına güvenmek yerine erişimi teknik olarak sınırlamak gerekir.

## 5. Bölüm 10'un genel çerçevesi

- Paylaşılan ve ayrı context; context devri ile izolasyon arasında ödünleşimdir.
- Eşler arası, manager ve merkezsiz modeller farklı koordinasyon biçimleridir.
- Test, gözlem ve dış kaynak gibi kanıtlar kaliteyi artırabilir; tek agent da bu tool’lara erişebilir. Çoklu yapı maliyeti ve faydasıyla karşılaştırılmalıdır.
- Dosya çakışması, anlamsal uyumsuzluk, ortak hatalar, eksik devir ve kontrolsüz döngüler sınır, test ve gözlemlenebilirlik gerektirir.
- İnsan, sistemin tasarımına ve kritik kararlarına ilişkin sorumluluğunu korur.

## 6. Sonsöz: Kitabın bütünü

Kitabın çerçevesi: **Agent = LLM + Context + Tools**. Harness ise bu öğelerin görev, durum, izin, doğrulama ve hata yönetimiyle nasıl çalıştığını düzenler.

```text
İnşa: Context + bellek + araçlar + etkileşim
                 ↓
Değerlendirme: Sonuç, süreç ve kaliteyi ölç
                 ↓
Evrim: Kanıta dayanarak aday iyileştirme üret ve sına
                 ↓
İş birliği: Sorumlulukları ve bilgi akışını düzenle
```

Yazar iki açık sorun vurgular: gerçek zamanlı ve kesintiye duyarlı etkileşim; agent'ın deneyimden güvenilir biçimde öğrenmesi. Model ile harness'ın birlikte gelişeceği, bazı eski telafilerin güçlü modellerle sadeleşebileceği görüşünü savunur. Bu bir öngörüdür; kritik yetki ve güvenlik sınırlarının sadece modelin davranışına güvenilerek kaldırılması gerektiği sonucu çıkmaz.

## Kontrol soruları

1. Agent çıktısını incelemeden kullanmak hangi durumda anlama borcu doğurur?
2. Aynı ödülü optimize eden agent'ların grup davranışı neden beklenmedik olabilir?
3. Kurt adam örneğindeki hakemin bilgi filtrelemesi üretim sistemindeki hangi güvenlik ilkesine benzer?
4. Yeni bir agent özelliğinin gerçekten iyileştirme olduğunu nasıl ölçersin?
