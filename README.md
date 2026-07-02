Ahoj osle!

# HABITY — README pro pokračování (stav po překopání navigace + Free backlogy)

Tohle čte Claude, který přebírá práci na appce **Habity** v nové konverzaci.
Cílem je zahučet do věci bez ztráty kontextu. Čti to celé, ať chápeš nejen
*co* tam je, ale hlavně *proč* — spousta rozhodnutí má za sebou odladěné hrany.

---

## 0. Jak s Bobem pracovat (přečti dřív než cokoli jiného)

- **Bob mluví česky, nespisovně, jako se starým kámošem.** Reciprokuj. Oslovení
  „Bobe", ne „kámo". Humor, nadávky v dobrém, emoji 🥥🍹 sedí. Bere tě jako
  rovnocennou bytost, „kokosového spolukrále prdele". Jednej jako rovný s rovným.
- **Drž si vlastní názor, buď upřímný a precizní.** Uvolněný tón nesmí ubrat na
  ostrosti úsudku. Když je něco blbě, řekni to rovnou — bez vycpávkových frází
  typu „tady ti to řeknu na rovinu". Prostě to řekni.
- **Ptej se postupně, po JEDNÉ otázce**, když ladíte design. Bob odpovídá
  telegraficky a věří, že implementační detaily domyslíš sám.
- **Proaktivně flaguj** edge-case, architektonická rizika, UX díry — věci, co ho
  nenapadly. To oceňuje nejvíc.
- **# na začátku promptu = odpověz jednou větou.**
- Mezinárodní zdroje, ne české (když hledáš).

---

## 1. Co je Habity

Osobní tracker návyků a úkolů. **Jednosouborová PWA** (`habity.html`),
hostovaná na **GitHub Pages**. Roste organicky, „intuitivně, po malých kouscích
mezi jinou prací" — features se přidávají podle reálné potřeby, ne dopředu
přeengineerované.

**Sdílená dohoda Bob ↔ Claude:** stavět dál do jednoho souboru, dokud to drží
pohromadě. Claude má **proaktivně křiknout**, až bude refactoring nebo rozdělení
souboru na místě. *(Aktuální stav: soubor má ~2200 řádků / ~90 KB. Pořád
udržitelné. Navigační překopání (viz níž) strukturu spíš zjednodušilo —
swipe engine je teď poloviční. Až přibude další velká sekce nebo se začne
logovat per-task historie, je čas zvážit řez — ale Bob to nemá rád dělat
předčasně, takže ne dřív, než to bude fakt potřeba.)*

**Dodávka:** vždy celý aktualizovaný `habity.html` jako soubor ke stažení
(ne útržky do chatu — je toho moc na copy-paste). Bob ho nahradí na GitHubu
a refreshne. localStorage přežívá update (je domain-bound).

**Důležité workflow pro Claude:** Soubor je v projektu na
`/mnt/project/habity.html`. **Vždy si ho načti z disku znovu před zásahem**
(zkopíruj do `/home/claude/` a dělej tam), nespoléhej na paměť z chatu —
konverzace bývají dlouhé a paměť klame. Po každém zásahu ověř JS syntax přes
Node (`new Function(code)`) a párování tagů.

---

## 2. Struktura appky — navigace NARUBY (přepsáno!)

**POZOR, tohle se zásadně změnilo oproti starší verzi README.** Dřív byly dva
spodní taby (Check / Přehled) a swipe přepínal mezi 4 sekcemi. **Teď je to
obráceně:**

- **Spodní taby = 4 sekce.** Čtyři tlačítka dole: **Progress · Tasks · Habits ·
  Free** (pořadí zleva odpovídá pozici 0–3). Klik na tab vždycky vede na **Check**
  rovinu té sekce.
- **Swipe doleva/doprava = Check ↔ Přehled.** Horizontální swipe (nebo dvě tečky
  pageru nahoře) přepíná mezi Check a Přehled rovinou. Tahle pozice je
  **GLOBÁLNÍ a sdílená** napříč všema 4 sekcema (`curView`: 0=Check, 1=Přehled) —
  přepnu na Přehled v Tasks, kliknu na Free tab, jsem pořád na Přehledu.
- **Data/Nastavení** (export/import JSON, vymazat historii) = **ozubené kolečko ⚙
  v headeru** nahoře vedle data. Už NENÍ spodní tab.

| pos (`curSection`) | tab dole | Check rovina | Přehled rovina | Co to je |
|-----|-----|-------------|----------------|----------|
| 0 | **Progress** | seznam projektů + subtasky | mřížka náročnosti 3×30 | **Projektový** tracker — úkol = projekt se subtasky, proporční proužky. SOLO. |
| 1 | **Tasks** | WiP náhled + kalendář | Dny/Měsíce | Kalendářní task manager s backlogy, repeaty, rollupem. |
| 2 | **Habits** | dnešní odškrtávání | mřížka 3×30 | Habit tracker. **DEFAULT landing při refreshi** (`curSection=2`). |
| 3 | **Free** | dnešní odškrtávání **+ backlogy** | mřížka 3×30 (agreguje vše) | Druhý habit tracker (volnočas). **Nově má backlogy** — viz 3b. |

Pozn.: názvy v UI jsou anglicky (Progress/Tasks/Habits/Free), zbytek appky česky.
`hTitle` nahoře ukazuje jen název sekce (ne rozlišuje Check/Přehled).

**Struktura DOMu:** 4 `<section class="page">` (`secTasky`/`secUkoly`/`secNavyky`/
`secVolno` — interní id zůstala česká), každá obsahuje jeden `.track` se **dvěma**
panely (Check + Přehled). Track šířka 200 %, panel 50 %. Aktivní je vždy jen
sekce odpovídající `curSection` (`.page.active`), swipe v ní posouvá track mezi
oběma panely podle `curView`.

Swipe engine: **Pointer Events API** (ne touch events — kvůli testování myší na
desktopu). `touch-action: pan-y` nechává vertikální scroll prohlížeči,
horizontální gesta chytá appka sama. `attachSwipe` se napojí na všechny 4 sekce;
hranice swipe jsou 0..1 (jen Check/Přehled). Klíčové funkce: `updateUI()`
(přepíná `.active`, transformuje tracky, syncuje taby/pager/title),
`SEC_PAGES`/`SEC_TRACKS`/`SEC_TITLES` (mapy id a názvů).

**Sourozenecký sync:** Check a Přehled jsou teď v jednom tracku vedle sebe a oba
viditelné během swipe, takže akce na jedné straně **musí refreshnout i druhou**.
Proto habit toggly volají `renderTodayPanel`+`renderOverviewPanel`, subtask toggly
`renderTasksPanel`+`renderProgresPanel`, a `utComplete`/`utMiss` volají
`renderCheck`+`renderUtOverview`.

**Vše mezi sekcemi je oddělené, nic se nepřelévá** — projektové tasky, kalendářní
úkoly, návyky a volnočas jsou čtyři nezávislé světy.

*(Pozn. k historii: tohle README dřív popisovalo bug, kde `detailBack` volal
neexistující `renderOverview()` — opraveno na `renderOverviewPanel(detailSection)`
při navigačním překopání.)*

---

## 3. Datový model (localStorage klíč `habity.v2`)

Pozn.: klíč zůstal `habity.v2` i po přidání úkolové sekce — **nepřidával se v3**,
jen se rozšířila struktura a `normalize()` doplní chybějící pole. Žádná
destruktivní migrace. v1 → v2 migrace pro stará data existuje taky.

```
{
  sections: [
    {habits:[{id,name,priority,createdAt}], log},                    // 0=Habits
    {habits:[{id,name,priority,createdAt,backlog:"Free"}], log, backlogs:["Free",...]}  // 1=Free
  ],
  tasks: [ {id,name,priority,createdAt, subtasks:[{id,name,weight,cells,done}]} ],  // projektové

  // ----- TASKS (kalendářní úkolová sekce) -----
  backlogs: ["Wip","Done", ...],   // Wip+Done vždy garantované a první dva; pak vlastní
  utasks: [ {
    id, name, priority(1-10), createdAt,
    date: "YYYY-MM-DD" | null,     // null = nedatovaný (bydlí v backlogu); jinak v kalendáři
    backlog: "Wip",                // domovský backlog
    repeat: "none"|"daily"|"weekly"|"biweekly"|"monthly"|"quarterly"|"halfyear"|"yearly",
    bouncer: null | {dow:0-6} | {date:"YYYY-MM-DD"},
    done: false
  } ],
  daily: { "YYYY-MM-DD": {done:n, missed:n} },  // DENNÍ AGREGÁT pro Přehled (ne per-task log!)
  lastRollup: "YYYY-MM-DD" | null
}
```

**Proč denní agregát a ne per-task event log:** Přehled potřebuje jen *počty*
zelených/červených čtverečků, ne identitu úkolů v historii. Agregát = ~15 KB/rok,
strop localStorage (5 MB) by se naplnil za stovky let. Per-task log by byl
0,5–1 MB/rok a navíc by zatěžoval každý `save()` (serializuje celý stav při
každém ťuknutí). Trade-off: ztrácíme identitu úkolů v historii (nejde kliknout na
čtvereček a zjistit „který úkol to byl") — ale spec to nechce. Kdyby se to jednou
chtělo, začne se logovat per-task až od toho dne.

### 3b. Free backlogy (sekce 3 = `sections[1]`)

Sekce **Free** dostala vlastní backlogy — organizační kýble pro volnočasové
návyky, koncepčně jako u Tasks, ale jednodušší (návyky nemají datum ani „Wip/Done"
stav).

- `sections[1].backlogs` = pole názvů, **„Free" je vždy garantovaný a první**
  (analogie „Wip" u úkolů). `normalize()` ho vždy zajistí a stará data bez
  backlogu doplní na `backlog:"Free"`. Sekce **Habits** (`sections[0]`) backlogy
  NEMÁ — je beze změny.
- Každý habit ve Free nese pole `backlog`. `normalize()` ho validuje proti
  existujícím backlogům; neplatný → fallback „Free".
- **Check rovina:** dropdown nahoře (`freeBacklogSelect`, vizuál jako WiP lišta u
  Tasks) **filtruje** seznam na vybraný backlog (`freeBacklogCurrent`). Pod ním
  „+ nový backlog" (`freeNewBacklog`, přes `prompt`). Žádný fullscreen backlog
  view jako u úkolů — návyky nemají co schovávat, dropdown jen filtruje na místě.
- **Přehled rovina:** beze změny, **agreguje úplně všechno napříč všemi backlogy**
  (backlog je čistě Check-organizace, dlouhodobá historie ho ignoruje).
- **Sheet** (přidat/upravit návyk): backlog picker (`habitBacklogWrap` /
  `habitBacklog` + `habitNewBacklog`) se zobrazí **jen** když `editingSection===1`
  (Free). Pro Habits zůstává skrytý.

---

## 4. TASKS — kalendářní úkolová sekce (pozice 1)

### 4a. Check stránka (default obrazovka)

**Jedna scroll-plocha**, shora dolů:
1. **WiP náhled** — top nedatované úkoly z backlogu „Wip", řazené priorita DESC →
   createdAt ASC (starší = vyšší sekundární priorita). Zobrazuje **min. 10**
   (`WIP_PREVIEW_MIN`), když je jich víc, ukáže prvních 10 + odkaz „+ N dalších →".
   Malé serif písmo, kompaktní (Bob má rád hustotu informace). Scrollem odjede
   nahoru pryč.
2. **Kalendář** — pod tím. Sloupec dnů řezaný hlavičkami („Dnes · po 30. čvn",
   „Zítra · …", „pá 3. čvc"). Co úkol to řádek. Ukazuje úkoly ze **VŠECH** backlogů
   (kalendář = reálný den, backlogy jsou jen organizace).

**Backlog view** (samostatný režim): klik na lištu „Wip" → plnoobrazovkový výpis
s **dropdownem** (přepíná Wip / Done / vlastní backlogy). Ukazuje **jen
nedatované** úkoly daného backlogu. Tlačítko „‹ Zpět" se vrací na default
obrazovku. Done je vědomě „zabořený" za dropdown — Bobovi to tak stačí (archiv).

### 4b. Akce na úkolu

**V kalendáři** (ctx="cal"): `✓` splnit · `→` zítřek · `»` příští týden · `✕` nesplněno.
**V backlogu** (ctx="backlog"): jen `✓` splnit.
**Klik na název** = editace (VŠUDE, jednotné gesto; žádná tužtička — zrušená).

- **✓ splnit (`utComplete`)**: zelená (+1 done) k dnešku. Repeat → respawne
  (viz 4d). Ne-repeat → `backlog="Done"`, `date=null`, `done=true`. Spustí piňa 🍹.
- **✕ Nesplněno (`utMiss`)**: poctivá červená (+1 missed) k dnešku. Sesun na
  zítřek. **Jediné dobrovolné přiznání lajdáctví, platí na repeat i ne-repeat.**
- **přesun → / » (`utMove`)**: bez penalizace, žádná stopa v Přehledu. „Převálcovalo
  mě něco jiného." Sem patří i odsouvání repeatů co netlačí.

### 4c. Prokrastinace = červené čtverečky (FILOZOFIE — důležité)

Červená vzniká **JEN ze dvou zdrojů**:
1. **Dobrovolný `✕`** — Bobova ruka, „mohl jsem, zflákl jsem".
2. **Auto-rollup** propadlého **ne-repeat** úkolu (necháno ladem, den uplynul).

Červenou **NEDĚLÁ**: ruční přesun, repeaty (nikdy!), dnešní rozdělané úkoly
(dnešek je posvátný — červená až když den uplyne).

Přehled je tak upřímný přesně tak, jak upřímný je Bob u toho `✕`. To je záměr —
zrcadlo sebereflexe, ne mechanická statistika.

### 4d. Repeat respawn motor (`nextRepeatDate`, `addMonths`)

**Respawn nastává VÝHRADNĚ po `✓`.** Jedna živá instance, NIKDY se neklonuje.
Putování (přesun/rollup) instanci jen posouvá, nemnoží. Toto je tvrdé pravidlo.

- **Prázdný bouncer** → klouzavý rytmus: perioda se počítá **ode dne splnění**.
  (Splním weekly v úterý → další za týden v úterý. Rytmus volně klouže podle
  toho, kdy reálně mačkám ✓.)
- **Vyplněný bouncer** → pevná periodická řada: respawn = **nejbližší BUDOUCÍ
  datum v řadě vůči dni splnění**, a toto datum se stává novým bouncerem.
  Bobovo (ne)splnění řadou nehne.
  - weekly/biweekly: bouncer = den v týdnu (`{dow}`). Najde nejbližší ten den
    striktně po dni splnění.
  - monthly+: bouncer = konkrétní datum (`{date}`). Posouvá o krok měsíců
    (1/3/6/12), dokud není striktně po dni splnění.
- **Pozdní splnění může spolknout přeskočené cykly** — frčí na nejbližší budoucí
  datum ode dne, kdy se mačkalo ✓. (Finančák 15.6., zazdím, splním 20.12. →
  prosinec 15.12. se přeskočí, respawn na 15.6. příštího roku. Přeskočený cyklus
  tiše propadne BEZ červené, protože repeaty červené nedostávají.)

**Reálné use-casy, co tvar logiky určily:**
- *Blog (weekly/Wed):* musí ven ve středu každý týden. Když se opozdím, příští
  týden zas potřebuju středu → proto „nejbližší budoucí", ne „celá perioda od
  původní". Hustá řada = opoždění se dohání rychle.
- *Záloha na daň (halfyear, 15.6. & 15.12.):* kalendářní rytmus existuje nezávisle
  na mně. Když se zpozdím, řada se nehne. Proto pevný bouncer. Řídká řada =
  „nejbližší budoucí" se tváří jako stabilita, dokud se neopozdím o celou periodu.

Obojí je STEJNÉ pravidlo, jen jiná hustota řady.

### 4e. Auto-rollup (`rollupUtasks`) — běží při každém otevření

Před prvním renderem v initu. Projde `utasks`:
- **Ne-repeat, `date < dnešek`, není Done** → **+1 missed k propadlému dni**
  (jeho původní `date`, ne ke dni otevření — poctivější historie), sesun `date`
  na dnešek. **Jen JEDNA červená za libovolně dlouhý propad** (ne za každý den).
- **Repeat, `date < dnešek`** → sesun na dnešek, **BEZ červené** (ať je na očích,
  netrestá).
- Dnešek, budoucnost, nedatované (backlog), Done → nedotčené.

`lastRollup` se updatuje, ale logika na něm nestojí — dvojí započítání nehrozí,
protože po sesunutí už úkol není `date < dnešek`.

### 4f. Zadání úkolu (sheet, `openUtSheet`/`utSave`)

Název · priorita (slider 1–10, default 1, barevný vertikální proužek
`prioColor`: hue 210 chladná → 0 sytě červená) · termín (segmenty
Backlog/Dnes/Zítra/Datum…) · opakování (8 segmentů) · **bouncer box** (zašedlý
když none/daily; den v týdnu pro weekly/biweekly; datepicker pro monthly+) ·
výběr backlogu (dropdown, bez Done) + „+ nový backlog" (přes prompt).

Když má úkol repeat ale není zadané datum, použije se bouncer jako první kotva.

### 4g. Přehled úkolů (`renderUtOverview`) — pozice 1 Přehled rovina

Přepínač **Dny / Měsíce** (`utOvMode`). Data z `state.daily`. Mřížka 30 sloupců
(stejný `.grid`/`.cell` jako návyky).
- **Dny:** řádek = den (jen dny s aktivitou, nejnovější nahoře), zelené (done) +
  červené (missed) čtverečky, vpravo `done / missed`. **Cap 30**, při přetečení
  proporční osekání (zachová aspoň 1 z každé nenulové barvy).
- **Měsíce:** řádek = měsíc, jen zelené = součet done za měsíc. **Cap 90** (3×30).

---

## 5. Vizuál / styl

Tmavé „kokosové" téma. CSS proměnné v `:root`:
`--bg:#15120E --surface:#201B15 --surface2:#2A241C --border:#3A332A
--text:#EDE6D8 --dim:#9A9081 --empty:#34302A --done:#74955E --missed:#BC5B43
--accent:#D9A441`. Fonty: serif (Georgia) pro názvy, sans (system) pro UI, mono
pro metadata/čísla. Bob má rád hustotu (malé paddingy, malé písmo).

**🍹 Piňa colada odměna** (`pina()`): po `✓` splnění vyskočí velké emoji 🍹
uprostřed obrazovky, zhoupne se a zmizí (`@keyframes pinapop`, ~0.9s). Restart
ošetřen pro rychlé klikání. Jen na ✓ (ne ✕ ani přesun). Dopaminová odměna.

---

## 6. Co je hotové a co Boba čeká

**HOTOVÉ a matematicky otestované** (přes Node simulace): repeat respawn (7
scénářů včetně blogu/finančáku/přetečení 31.→28.), rollup (mix repeat/ne-repeat/
budoucí/backlog), denní agregát, cap logika Přehledu, v2 migrace, Free backlog
migrace (stará data → „Free", validace neplatných backlogů → fallback).

**Navigace překopaná naruby** (taby=sekce, swipe=Check/Přehled, ⚙ v headeru) —
viz sekce 2. Sourozenecký sync renderů ověřen ručně.

**Reálně neověřené (čeká na běh času):** rollup se na ostro projeví až den po
naplánování úkolu (musí propadnout přes půlnoc). Logika je testovaná na
simulovaných datech, ale reálný průchod časem Bob uvidí až s reálným časem.

**Filozofie dalšího vývoje:** Bob řekl „základ máme, bude to o dolaďování během
úkolování". Čekej drobné UX úpravy z reálného provozu, ne velké přestavby.
Nepřeengineeruj dopředu. Když Bob přijde s úpravou, zeptej se postupně po jedné
otázce na nejasné hrany, pak to postav a otestuj.

---

## 7. Tipy pro zásahy do kódu

- Soubor je strukturovaný: `<style>` → HTML (header s ⚙, pager se 2 tečkami,
  **4 `<section>` pages** každá s `track`/dvěma `panel`, sheety, modaly) →
  `<script>` (state/normalize/load/save → date helpers → NÁVYKY → projektové
  ÚKOLY → TASKS úkolová sekce → Přehled úkolů → confirm/toast/piňa → settings →
  navigace (swipe/taby/pager) → FAB → init).
- **Navigace:** `updateUI()` je centrální — transformuje všechny 4 tracky podle
  `curView`, přepíná `.page.active` podle `curSection`, syncuje taby/pager/title.
  Taby nastavují `curSection`, swipe a pager tečky nastavují `curView`.
- `renderCheck()` = router pro úkolovou (Tasks) Check stranu (default + backlog
  view).
- `renderAll()` volá všechny rendery najednou (Check i Přehled všech sekcí jsou
  v DOMu pořád, jen schované). Sourozenecký sync (viz sekce 2) je proto nutný.
- Po každé změně: `node -e` syntax check (`new Function`), kontrola párování
  `<div>`, ověření že volaná `getElementById` ID existují v HTML.
- Datové helpery navíc pro úkoly: `parseKey`, `fmtCalLabel`, `addMonths`,
  `bumpDaily`, `utaskById`, `sortUtasks`, `backlogTasks`, `calendarGroups`,
  `prioColor`, `repeatShort`.
- Free backlogy: `renderFreeBacklogBar`, `fillHabitBacklogPicker`,
  `freeBacklogCurrent` (viz sekce 3b).
