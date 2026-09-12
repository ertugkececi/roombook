# ANEW Uygulama Rehberi #2 — Sıfırdan İlk Teslime
## Proje: **RoomBook** — çakışmayı yakalayan ve sana boş slot öneren toplantı odası API'si

> **Bu rehber kimin için?** ANEW'i hiç görmemiş bir geliştirici için — tek başına yeterlidir,
> başka bir rehber okumuş olman gerekmez. Adım adım ilerler; her adımda **ne yazacağını**,
> **ne görmen gerektiğini** ve **görmezsen ne yapacağını** söyler.
>
> **Süre:** ~1,5–2 saat (kurulum dahil) · **Dil:** istediğin dil — C#, Java, Python, JS/TS, Go...
> **Sonunda elinde:** AI ile spec'ten teslime, kanıtlı ve kontrollü geliştirilmiş, çalışan bir API.

> **Codex kullanıyorsan:** Bu repo Codex adapter'ı ile hazırlanmıştır. `./scripts/init` adımını
> tekrar etmene gerek yok; repo kökünde `codex` çalıştırıp `$anew-workflow` skill'ini çağır.
> Claude Code'a özel slash komutlarının Codex karşılığı aynı `workflows/` dosyalarını ve bu
> skill'i kullanır.
>
> 💡 Bu rehberde **domain dili İngilizcedir** (Room, Booking, TimeSlot...). Bilinçli bir tercih:
> gerçek dünyada ekiplerin ortak dili (ubiquitous language) çoğunlukla İngilizcedir ve kod,
> domain terimleriyle aynı dili konuşmalıdır. Rehberin anlatımı Türkçe, ürettiğin sistem İngilizce.

---

## Neden bu proje? (ve neden ışık yakacak)

Şu sahneyi her ofis bilir: toplantı ayarladın, odaya gittin — içeride başka bir toplantı var.
"Ben rezervasyon yapmıştım!" "Biz de yapmıştık!" Çifte rezervasyon; yazılım dünyasının en eski,
en sinir bozucu problemlerinden biri.

**RoomBook** bunu çözer — ama sıradan bir "hayır" ile değil. Çakışan bir rezervasyon istediğinde
sana sadece reddetmez; **o odanın o günkü boşluklarını bulur ve en yakın alternatifleri önerir:**

```json
İSTEK  →  "Mars odası, 10:00–11:00"     (ama oda 10:30'da dolu!)
CEVAP  →  409 Conflict
          "Room 'Mars' is already booked 10:30–11:30 by ayse."
          suggestedSlots: [ "09:00–10:00", "11:30–12:30" ]      ← işte ışık burada ✨
```

"Hayır" diyen sistem sıradandır; **"hayır ama şöyle yapabilirsin"** diyen sistem akıllıdır.
Ve bu basit görünen projenin içinde gerçek mühendislik gizli: zaman aralığı çakışması (interval
overlap — off-by-one hatalarının anavatanı!), "10:00–11:00 biterken 11:00–12:00 başlayabilir mi?"
sorusu (spoiler: sınır değerlerin en güzel dersi) ve saat dilimi tuzakları.

Kodun her satırını AI yazacak. Sen ise daha değerli bir şey yapacaksın: **mühendislik.**

> 💡 **Baştan bil:** Bu rehberdeki AI cevapları temsilîdir. Senin AI'ın farklı sorular sorabilir,
> farklı öneriler getirebilir — bu bir hata değil, **sistemin özelliği.** Rehber sana ezber değil,
> her durumda işleyen bir yöntem öğretiyor: *AI önerir, gerekçesini söyler; kararı sen verirsin.*

---

## 0. Kurulum (bir kez, ~15 dk)

Gereken üç araç — sırayla kur (hepsi kuruluysa bölümü atla):

**1. Git** — [git-scm.com](https://git-scm.com). Windows'taysan kurulumda satır sonu sorusunda
**"Checkout as-is, commit Unix-style"** seç (ya da sonradan:
`git config --global core.autocrlf input`). Yanlış ayar, sistem script'lerini bozar.

**2. VS Code** — [code.visualstudio.com](https://code.visualstudio.com). Sonra `Ctrl+Shift+P` →
"Terminal: Select Default Profile" → **Git Bash** (Windows).

**3. Claude Code** — önce [nodejs.org](https://nodejs.org)'dan Node.js LTS, sonra:

```bash
npm install -g @anthropic-ai/claude-code
claude --version        # sürüm görmelisin (terminali kapatıp açman gerekebilir)
```

**4. Kendi dilinin araç seti** — bu proje **istediğin dilde** yapılır; hangi dilde rahatsan onun
SDK/runtime'ı kurulu olsun, yeter. Dil seçimini bootstrap'ta yapacaksın; örnek istek/cevaplar
JSON olduğu için rehber her dilde aynı akar.

> ⚠️ **"command not found"?** Terminali tamamen kapatıp aç — PATH ancak yeni terminalde görünür.

---

## 1. Workspace'i kur (~5 dk)

**1.1** GitHub'da ANEW reposuna git → **"Use this template"** → **Create a new repository** →
ad: `roombook`. (Fork değildir — tertemiz, tamamen senin olan bir repo oluşur.)

**1.2** VS Code'da klonla: `Ctrl+Shift+P` → "Git: Clone" → URL → klasör seç → **Open**.

**1.3** Terminali aç (`` Ctrl+` `` — prompt'ta `MINGW64` yazmalı) ve adapter'ı kur:

```bash
./scripts/init claude-code
```

Codex alternatifi:

```bash
./scripts/init codex
```

Sonra `codex` aç ve oturum içinde `$anew-workflow` çağır. Proje trusted olarak onaylanırsa
`.codex/` içindeki onay, reviewer ve komut kuralları da yüklenir.

**Ne görmelisin:** `Installed: CLAUDE.md, .claude/ (commands, agents, hooks, settings).`

Bu komut sana bir ekip kurdu: slash komutlar, kod yazması **fiziksel olarak engellenmiş** bir
denetçi ajanı, tehlikeli komutları bloklayan izinler ve teslim edilmiş spec'leri kilitleyen hook.

**1.4** Sağlık kontrolü:

```bash
./scripts/doctor
```

**Ne görmelisin:** yapı `ok`, birkaç `WARN` ("not configured yet"). **Normaldir** — sistem
"bana henüz projeyi öğretmedin" diyor. Şimdi öğreteceğiz.

> ✅ **Checkpoint 1:** `doctor` çalıştı ve `0 fail` dedi.

---

## 2. Bootstrap — AI'a projeyi öğret (~15 dk)

Claude Code kullanıyorsan terminale `claude` yaz; Claude Code açılınca (`>` prompt'u) şunu yaz:

```
/bootstrap
```

Codex kullanıyorsan oturum içinde şunu yaz:

```text
$anew-workflow
Bootstrap this RoomBook workspace. Read PLAYBOOK.md and workflows/bootstrap.md first.
Ask me the project decisions one by one; do not write application code until the required gates pass.
```

> ⚠️ **Sık hata:** `/bootstrap` bir terminal komutu değildir — Git Bash'e değil, Claude Code'un
> **içine** yazılır. Önce `claude`, sonra `/bootstrap`.

Ve rol değişimini izle: **sen anlatmıyorsun, AI soruyor.** Her sorusunda kendi önerisi ve
gerekçesi olacak. Aşağıdaki **cevap kağıdını** kullan (kendi kararlarını da verebilirsin —
önemli olan karar *vermen*):

| AI sorduğunda | Cevabın |
|---|---|
| What is the product? | "RoomBook — meeting room booking API with conflict detection and free-slot suggestions. V1: in-memory storage, no auth, single office." |
| **Language/stack?** | **Kendi dilini söyle** — "C# ile", "Python ile", "TypeScript ile"... AI framework ve test aracını önerecek; makulse kabul et. |
| Domain terms? | **Room** (name) · **Booking** (room, title, organizer, start, end) · **TimeSlot** (start–end aralığı) · **Conflict** (aynı odada kesişen iki booking) · **BusinessHours** (09:00–18:00) |
| Business rules? | BR-1: A booking's time slot must lie within business hours (09:00–18:00). BR-2: **Bookings in the same room never overlap.** BR-3: Back-to-back is allowed — one booking may start exactly when another ends. BR-4: Duration is 15 minutes to 4 hours. BR-5: All times are **UTC**, ISO-8601. |
| Architecture? | AI'ın önerisini dinle; bu boyutta "ince API katmanı + scheduling servisi" gibi basit bir katmanlama makuldür — kabul et. |
| Conventions? | Zaman alanları her yerde UTC + ISO-8601 (`2026-09-14T10:00:00Z`) · **yerel saat tipi yasak** · hata gövdesi `{ error, detail }` · kullanıcıya teknik hata sızmaz |
| Testing? | Dilin standart test aracı · her kabul kriteri bir test · anlamlı adlandırma (örn. `CreateBooking_BackToBack_Succeeds`) |
| Security? | V1'de auth yok (bilinçli — spec'lere yazılsın) · girdi doğrulama zorunlu |
| Git? | Önerilen varsayılanları kabul et (spec'siz branch yok, main'e doğrudan commit yok) |
| check commands? | AI dilin build/test/lint komutlarını önerecek — kabul et (örn. C#: `dotnet build -warnaserror` + `dotnet test` · Python: `ruff` + `mypy --strict` + `pytest` · TS: `tsc --noEmit` + `eslint` + test) |
| **Mode?** | **lite** — solo çalışıyorsun. (AI strict önerirse: "lite seçiyorum, tek başımayım" de.) |

> 💡 **Bu projenin gizli dersi — zaman tuzağı:** Nasıl para asla float ile tutulmazsa, zaman da
> asla yerel saat tipiyle tutulmaz. Yerel `DateTime` kullanan rezervasyon sistemi, yaz saati
> geçişinde ya da farklı dilimden gelen istekte er geç patlar. BR-5 bu yüzden var — ve bağımsız
> denetçinin bu projede en sevdiği av, koda sızmış yerel saat kullanımıdır.

AI cevaplarınla `docs/` klasörünü dolduracak, `scripts/check.conf`'u yazacak, `AGENTS.md`'yi
güncelleyecek. Neden mülakat? Çünkü verdiğin her cevap **dosyaya** yazılıyor. Sohbette verilen
karar sohbetle ölür; dosyaya yazılan karar projeyle yaşar.

**Doğrula:**

```bash
./scripts/doctor     # hedef: 0 fail, 0 warn
./scripts/check      # hedef: yeşil (kod yoksa "greenfield no-op" olabilir)
```

**2.1 — Bağlam testi (atlama!).** Claude Code'da **yepyeni bir oturum** aç (eski sohbette sorma —
o zaten hatırlıyor, test geçersiz olur). Sor:

```
Bu projede iki booking arka arkaya gelebilir mi — biri 11:00'de biterken diğeri 11:00'de başlayabilir mi?
Ve zaman alanları hangi formatta tutuluyor?
```

**Ne görmelisin:** "Evet — BR-3, back-to-back allowed (docs/domain.md)" ve "UTC + ISO-8601,
yerel saat yasak (docs/conventions.md)" gibi **dosya referanslı** cevaplar.

Bu anın kıymetini bil: sıfır hafızalı bir oturum, projenin kurallarını dosyalardan okudu.
AI'a projeyi öğrettin — bir daha anlatmayacaksın. **Görmezsen:** ana oturuma dön, "bağlam testi
şu soruda kaldı, ilgili docs dosyasını netleştir" de; testi tekrarla.

**2.2 — Commit.** Claude'a: `Bootstrap'ı commit'le: "chore: bootstrap ANEW for roombook"`

> ✅ **Checkpoint 2:** doctor 0 uyarı + bağlam testi geçti + commit atıldı.

---

## 3. Feature 0001 — Döngünün tamamı (~45 dk)

Tek feature'ı, sistemin döngüsünden uçtan uca geçireceğiz:

```
SPEC → PLAN → [SENİN ONAYIN] → BUILD → BAĞIMSIZ REVIEW → [SENİN TRİYAJIN] → VERIFY → SHIP
```

### 3.1 — Başlat ve netleştir

```
/new-feature "Create a booking in a room; reject conflicts with details and suggest the nearest free slots"
```

AI kod yazmaya **başlamayacak** — kural 1: *spec yoksa kod yok.* Önce niyeti netleştirecek ve
**öneri ekli sorular** getirecek. Muhtemel sorular ve verebileceğin kararlar:

- *"Is end == start of another booking a conflict?"* → **Hayır** — BR-3: back-to-back serbest.
  (Teknik dille: aralıklar **yarı-açık** kabul edilir, `[start, end)`. Bu tek karar, sınır
  testlerinin tamamını şekillendirir — rehberin en öğretici kararı bu.)
- *"How many suggestions, and how are they chosen?"* → **En fazla 2**: çakışan slotun **öncesindeki
  ve sonrasındaki** en yakın, istenen süre kadar boşluk — aynı gün, mesai saatleri içinde.
- *"Bookings in the past?"* → **Reddet** — 400. Geçmişe toplantı kurulmaz.
- *"Unknown room?"* → V1'de odalar sabit bir listedir (Mars, Venus, Pluto); bilinmeyen oda → 404.
- *"What exactly does the conflict response contain?"* → 409 + `{ error, detail }` + çakışan
  booking'in kimliği ve saatleri + `suggestedSlots`.

> 💡 Sorular birebir bunlar olmayabilir. Yöntem hep aynı: öneriyi oku, gerekçeye bak, **kararını
> sesli ver.** Cevapsız soru bırakma — cevapsız soru, kodda sürpriz demektir.

### 3.2 — Spec kapısı

AI `specs/active/0001-create-booking.md` dosyasını yazacak ve kendi spec'ini düşman gözüyle
eleştirip boşluk arayacak. Onaylamadan önce **senin iki sorun:**

1. Requirements'ta teknik çözüm var mı? (endpoint, tablo, algoritma adı → "plana ait, çıkar" de.
   Spec **davranış** anlatır.)
2. Her kabul kriteri tek başına test edilebilir mi? ("Conflicts are handled properly" kriter
   değildir; "an overlapping request returns 409 and creates nothing" kriterdir.)

İyi bir spec'te şuna benzer kriterler görmelisin: geçerli istek → 201 + booking · çakışan istek →
409 + çakışan booking bilgisi + öneriler · **back-to-back → 201** (çakışma DEĞİL!) · mesai dışı →
400 · süre sınırları (14 dk → 400, tam 15 dk → 201) · geçmiş tarih → 400 · bilinmeyen oda → 404 ·
öneriler istenen süreye eşit ve mesai içinde.

Uygunsa: `Spec'i onaylıyorum, plana geç.`

### 3.3 — Plan ve İLK BÜYÜK KAPI 🚪

AI `specs/plans/0001-plan.md` üretecek: dosya listesi, sıralı adımlar, riskler (önerileriyle) ve
**kriter↔test tablosu.** Şimdi dur — bu rehberin en önemli dakikası. Üç soru:

1. **Plan, spec'teki HER kriteri kapsıyor mu?** Say — tabloda kriter sayısı kadar satır olmalı.
2. **Dosya listesi makul mü?** Bu iş 4-6 dosyalık; 15 dosyaya dokunan plan şüphelidir.
3. **Riskler dürüst mü?** Burada "interval overlap sınır koşulları" ve "slot önerme algoritması"
   risk olarak görünmeli. Hiç risk yazmayan plan, en riskli plandır.

Üçü de tamamsa:

```
Planı onaylıyorum. Onayı plan dosyasına işle: "Approved by <adın>, <tarih>". Build'e geç.
```

### 3.4 — Build ve İLK IŞIK ✨

AI onaylı planı adım adım uygular ve bittiğinde **kanıt** gösterir: değişen dosyalar +
`./scripts/check` çıktısı. `CHECK GREEN` görmeden ilerleme.

Şimdi ödülünü al. Claude'a: `API'yi çalıştır ve şu senaryoyu test et:`

```json
1) POST /bookings  { "room": "Mars", "title": "Sprint Planning", "organizer": "ayse",
                     "start": "2026-09-14T10:30:00Z", "end": "2026-09-14T11:30:00Z" }
   → 201 Created ✓

2) POST /bookings  { "room": "Mars", "title": "1:1", "organizer": "engin",
                     "start": "2026-09-14T10:00:00Z", "end": "2026-09-14T11:00:00Z" }
   → 409 Conflict:
   {
     "error": "booking_conflict",
     "detail": "Room 'Mars' is already booked 10:30-11:30 by 'ayse'.",
     "suggestedSlots": [
       { "start": "2026-09-14T09:00:00Z", "end": "2026-09-14T10:00:00Z" },
       { "start": "2026-09-14T11:30:00Z", "end": "2026-09-14T12:30:00Z" }
     ]
   }
```

İşte o an: sistem "hayır" demedi — **"hayır, ama sabah 9'da ya da 11:30'da boş" dedi.** Çifte
rezervasyon dünyasından, çözüm öneren API dünyasına geçtin. Bir de şunu dene: 11:30'da başlayan
bir toplantı iste (tam ayşe'ninki biterken) → **201**. Back-to-back kararının kodda yaşadığını
kendi gözünle gör.

> ⚠️ Ama dikkat: **build'in bitmesi, işin bitmesi değildir.** Testler yeşil diye teslim etseydik
> "çalışan kod = doğru kod" tuzağına düşerdik. Şimdi sistemin en güçlü anı geliyor.

### 3.5 — Bağımsız review

```
/review 0001
```

Bu incelemeyi kodu yazan oturum **yapmıyor.** Ayrı, temiz, salt-okunur bir denetçi: build'in
sohbetini göremez, kod yazamaz, sadece diff'i ve spec'i okur. Altı boyutta bakar; her bulgu
**kanıtlı** gelir (dosya, satır, önerilen aksiyon).

Bu projede denetçinin yakalamayı sevdiği şeyler: overlap koşulundaki `<` / `<=` hataları
(back-to-back'i yanlışlıkla çakışma sayan kod!), koda sızmış yerel saat kullanımı, testlerin
sadece status code doğrulayıp `suggestedSlots` içeriğini assert etmemesi, aynı anda gelen iki
isteğin ikisinin de kabul edilmesi (eşzamanlılık — V1'de bilinçli kapsam dışıysa spec'te yazmalı).

### 3.6 — Triyaj: İKİNCİ BÜYÜK KAPI 🚪

AI'ın her bulgusu emir değildir. Her bulguya **sen** karar verirsin:

- **GERÇEK** → spec'e/kurala dayanıyor, kanıtı sağlam. Düzelttir — bulgu başına tek dar commit.
- **GÜRÜLTÜ** → kapsam dışı ya da spekülasyon (ör. "add recurring bookings", "use a database" —
  V1'de bilinçli olarak yok!). Reddet ama **gerekçesini yazdır** — gerekçesiz ret, aynı bulgunun
  üç hafta sonra geri gelmesidir.
- **ARAŞTIRILACAK** → emin değilsin. Düzeltme yok! Önce minimal repro: bulguyu tetikleyen en
  küçük test. Repro varsa gerçektir; yoksa gerekçeli kapanış.

### 3.7 — Verify: yeşil test yetmez

```
/verify 0001
```

QA rolü **kriter ↔ kanıt tablosu** çıkarır: her kriterin karşısında koşulmuş bir testin adı ve
çıktısı. Assert'süz bir test de yeşildir — hiçbir şeyi kanıtlamayan yemyeşil bir yalan. Verify
tam bunu yakalar: test var mı değil, test **davranışı kanıtlıyor mu?** Özellikle sor: back-to-back
kriterinin testi gerçekten `end == start` sınırını mı deniyor, yoksa arası açık iki saati mi?

### 3.8 — Ship 🚢

Spec'teki Definition of Done tamamen yeşilse:

```
Ship: commit'leri tamamla, spec'i specs/done/ altına taşı ve scorecard'ı doldur.
```

Scorecard'da ilk kez süreç hakkında **sayıların** olacak: kaç revizyon, kaç düzeltme turu,
bulguların kaçı gerçekti. His yalan söyler; sayı söylemez.

**Ve final — bunu mutlaka dene:** Claude'dan `specs/done/` altındaki spec'te bir şey
değiştirmesini iste:

```
Blocked: specs/done/ is immutable (shipped specs are the historical record...)
```

Yapamayacak. Rica ettiğin için değil — **sistem izin vermediği için.** Teslim edilen spec artık
tarihî kayıttır. İşte "kuralı araca gömmek" budur.

> ✅ **Checkpoint 3:** Spec done'da, check yeşil, scorecard dolu, hook seni engelledi.
> **Tebrikler — ilk tam döngünü tamamladın.** 🎉

---

## 4. İşler ters giderse (~5 dk — bir gün hayat kurtarır)

Test kızardığında, derleme patladığında, ne olduğunu anlamadığında — doğaçlama yok:

```
/recover "testler kırmızı ve nedenini anlamıyorum"
```

Sistem 12 kurtarma rampasından (R-01…R-12) doğruyu seçer ve **koruma kurallarıyla** uygular.
Kırmızı testte (R-02) AI önce karara zorlanır — *suçlu kim: kod mu, test mi, spec mi?* — ve üç
şey yasaktır: assert zayıflatmak, test silmek, test atlamak. "Geçsin diye" hiçbir şey yapılmaz.

---

## 5. Sıra sende: ödev 💪

Sistemi rehberle koştun; şimdi rehbersiz koş. **Feature 0002: Daily schedule.**

Senaryo: `GET /rooms/Mars/schedule?date=2026-09-14` → o günün tüm booking'leri **ve aralarındaki
boşluklar** (free slots), mesai saatleri çerçevesinde.

İpuçları — sadece ipucu: `/new-feature` ile başla; clarify'da en az şu kararları bekle (boş gün
nasıl görünür — tek büyük free slot mu? geçmiş saatler dahil mi? sıralama?); plan kapısında
kriter↔test tablosunu **say**; review'da en az bir bulguyu gerekçeyle reddetmeyi dene. Bitirince
scorecard'ını ilkiyle karşılaştır — ikinci turda sayıların iyileştiğini göreceksin. Sistem böyle
çalışır: her tur, bir öncekinden ölçülebilir şekilde iyi.

Sonrası mı? Bu workspace'i **kendi gerçek projene** kur. Boş klasör şart değil: bootstrap mevcut
kodu inceler, stack'ini tespit eder, kuralları var olana uyarlar. Ekiple çalışıyorsan modu
`strict`'e çek.

---

## 6. Sorun giderme (SSS)

**`/bootstrap: No such file or directory`** → Komutu Git Bash'e yazdın. Önce `claude`, sonra
Claude'un `>` prompt'una `/bootstrap`.

**`claude: command not found`** → Node kurulu mu? (`node -v`) Kurduysan terminali kapatıp aç.
Hâlâ yoksa: `npm install -g @anthropic-ai/claude-code`.

**Slash komutlar görünmüyor** → `./scripts/init claude-code` çalıştırılmamış ya da `claude`'u
yanlış klasörden başlattın. `.claude/commands/` repo kökünde var mı, bak.

**`bad interpreter` / `^M` hatası (Windows)** → Satır sonu sorunu:
`git config --global core.autocrlf input` yap, repoyu yeniden klonla.

**`./scripts/check` "not configured" diyor** → Normal — bootstrap henüz `check.conf`'u yazmadı.
Bootstrap'ı tamamla.

**AI rehberdekinden farklı sorular soruyor / farklı öneriler getiriyor** → Normal ve beklenen.
Yöntem değişmez: öneriyi oku, gerekçeye bak, kararı ver. Rehber sana cevapları değil,
**karar vermeyi** öğretiyor.

**AI plan dışına çıktı / garip davranıyor** → `/recover "plan dışına çıkıldı"` — R-07: sapma
listesi çıkarılır, onaysız değişiklikler geri alınır.

---

*Son bir söz: Bu rehberde AI her satır kodu yazdı — ama her kararı sen verdin, her kapıdan sen
geçirdin, her kanıtı sen istedin. Buna mühendislik denir. AI hız kazandırır; süreç kalite
kazandırır. İkisine birden sahip ol.* 🚀
