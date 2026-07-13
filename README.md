Ahoj osle!

# HABITY — README pro pokračování (stav po přidání sekce Litánie)

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

Osobní tracker návyků, úkolů a pohybu. **Jednosouborová PWA** (`index.html`),
hostovaná na **GitHub Pages** (`haryzek.github.io/habity/`). Roste organicky,
„intuitivně, po malých kouscích mezi jinou prací" — features se přidávají podle
reálné potřeby, ne dopředu přeengineerované.

**Sdílená dohoda Bob ↔ Claude:** stavět dál do jednoho souboru, dokud to drží
pohromadě. Claude má **proaktivně křiknout**, až bude refactoring nebo rozdělení
souboru na místě. *(Aktuální stav: soubor má ~3590 řádků / ~160 KB. Pořád
udržitelné, ale roste — přibyla sekce Move (viz sekce 5) a Litánie (viz sekce 6).
Až přibude další velká sekce nebo se začne logovat per-task historie, je čas
zvážit řez — ale Bob to nemá rád dělat předčasně, takže ne dřív, než to bude
fakt potřeba.)*

**Prostředí a dodávka:** Projekt je **lokální git repo** na `E:\_dev\habity`
(Claude Code, Windows/PowerShell). Edituje se přímo `index.html`, po zásahu
`git commit` + `git push` do `main`, GitHub Pages sám nasadí (pár desítek sekund;
ověř `curl` na Pages URL, `gh` CLI tu není). localStorage přežívá update
(domain-bound). **Osobní bordel/tokeny patří do `local/`** — je globálně
gitignorovaná (`core.excludesFile`), viz Bobova paměť.

**Důležité workflow pro Claude:** **Vždy si `index.html` načti z disku znovu
před zásahem**, nespoléhej na paměť z chatu — konverzace bývají dlouhé a paměť
klame. Po každém zásahu ověř JS syntax přes Node (`new Function(code)` nad
obsahem `<script>`) a párování tagů.

---

## 2. Struktura appky — navigace NARUBY (přepsáno!)

**POZOR, tohle se zásadně změnilo oproti starší verzi README.** Dřív byly dva
spodní taby (Check / Přehled) a swipe přepínal mezi 4 sekcemi. **Teď je to
obráceně:**

- **Spodní taby = 6 sekcí.** Šest tlačítek dole: **Progress · Tasks · Habits ·
  Free · Litánie · Move**. Klik na tab vždycky vede na **Check** rovinu té sekce
  (Move ani Litánie Check/Přehled nemají — viz níž).
  ⚠️ **Past pořadí vs. index:** vizuální pořadí v navu ≠ `curSection` index!
  Litánie je v navu **mezi Free a Move**, ale logicky má **`curSection=5`**
  (Move zůstal 4). Udělalo se to schválně, aby se nepřečíslovávala Move
  (`curSection===4` je po kódu na spoustě míst). Tab má `data-sec="5"`, ale sedí
  v DOMu mezi Free a Move — pořadí `.page` sekcí je čistě vizuální, logiku řídí
  index. **Když přidáváš další sekci, drž se tohohle: nový index na konec, tab
  umísti kam chceš.**
- **Úzký nav → jen ikony.** Při 6 tabech se popisky na mobilu nevejdou:
  `@media (max-width:430px)` schová `.tab .tl` (label ve spanu) a nechá jen ikony.
- **Swipe doleva/doprava = Check ↔ Přehled.** Horizontální swipe (nebo dvě tečky
  pageru nahoře) přepíná mezi Check a Přehled rovinou. Tahle pozice je
  **GLOBÁLNÍ a sdílená** napříč sekcemi 0–3 (`curView`: 0=Check, 1=Přehled) —
  přepnu na Přehled v Tasks, kliknu na Free tab, jsem pořád na Přehledu.
  **Move (4) a Litánie (5) jsou výjimky** — mají jen jednu plochu, swipe/pager se
  na ně nevztahují. Rozdíl: Move nemá ani FAB (data z importu), **Litánie FAB má**
  (přidání nové věty). `updateUI` proto řeší zvlášť `noPager = 4||5` a
  `fab hidden = jen 4`.
- **Data/Nastavení** (export/import JSON, vymazat historii) = **ozubené kolečko ⚙
  v headeru** nahoře vedle data. Už NENÍ spodní tab.

| pos (`curSection`) | tab dole | Check rovina | Přehled rovina | Co to je |
|-----|-----|-------------|----------------|----------|
| 0 | **Progress** | seznam projektů + kroky | mřížka náročnosti 3×30 | **Projektový** tracker — projekt (např. „Stěhování bytu") se odškrtávacími kroky, proporční proužky. SOLO. |
| 1 | **Tasks** | WiP náhled + kalendář | Dny/Měsíce | Kalendářní task manager s backlogy, repeaty, rollupem. **DEFAULT landing při refreshi** (`curSection=1`). |
| 2 | **Habits** | dnešní odškrtávání | mřížka 3×30 | Habit tracker. |
| 3 | **Free** | dnešní odškrtávání **+ backlogy** | mřížka 3×30 (agreguje vše) | Druhý habit tracker (volnočas). **Nově má backlogy** — viz 3b. |
| 4 | **Move** | — (jedna plocha) | — | Garmin aktivita jako kolečka po týdnech. Data z Garmin Connectu, plní se samy. **Viz sekce 5.** |
| 5 | **Litánie** | — (jedna plocha) | — | Podpůrné věty/přerámování, filtrované tagy. **Soukromá data — import z disku, nikdy online.** V navu mezi Free a Move. **Viz sekce 6.** |

Pozn.: názvy v UI jsou anglicky (Progress/Tasks/Habits/Free/Move), zbytek appky
česky. `hTitle` nahoře ukazuje jen název sekce (ne rozlišuje Check/Přehled).

**Struktura DOMu:** 6 `<section class="page">` (`secProgress`/`secTasks`/`secNavyky`/
`secVolno`/`secMove`/`secLitanie`). Sekce 0–3 mají jeden `.track` se **dvěma** panely
(Check + Přehled), track šířka 200 %, panel 50 %. **`secMove` i `secLitanie` mají
jen jeden panel** (translateX vždy 0). Aktivní je vždy jen sekce odpovídající
`curSection` (`.page.active`). `SEC_PAGES`/`SEC_TRACKS`/`SEC_TITLES` mají teď
**6 položek** (index 5 = Litánie).

Swipe engine: **Pointer Events API** (ne touch events — kvůli testování myší na
desktopu). `touch-action: pan-y` nechává vertikální scroll prohlížeči,
horizontální gesta chytá appka sama. `attachSwipe` se napojí na sekce 0–3
(**`secMove` přeskočena** — nemá co swipovat); hranice swipe jsou 0..1. Klíčové
funkce: `updateUI()` (přepíná `.active`, transformuje tracky, syncuje
taby/pager/title, **skrývá pager a FAB na Move**), `applyViewToTrack` (u
`trackMove` translateX 0, jinak podle `curView`), `SEC_PAGES`/`SEC_TRACKS`/
`SEC_TITLES` (mapy id a názvů, teď 5 položek).

⚠️ **Layout past (vyřešená):** `.wrap` (kontejner sekcí) je `display:flex`
column bez stretch, takže se smrskává na šířku obsahu. Široké sekce (karty) ho
roztáhnou, úzká Move ne → kolečka se lepila vlevo. Fix = `.wrap{width:100%}`
(do `max-width:560px`). Kdyby někdy nová sekce zase kolabovala na šířku, tady
je příčina.

**Sourozenecký sync:** Check a Přehled jsou teď v jednom tracku vedle sebe a oba
viditelné během swipe, takže akce na jedné straně **musí refreshnout i druhou**.
Proto habit toggly volají `renderTodayPanel`+`renderOverviewPanel`, subtask toggly
`renderTasksPanel`+`renderProgresPanel`, a `utComplete`/`utMiss` volají
`renderCheck`+`renderUtOverview`.

**Data se mezi sekcemi nepřelévají, ale nejsou to čtyři nezávislé světy — jsou
tři.** Progress (`state.tasks`) a Tasks (`state.utasks`) jsou vzájemně i vůči
zbytku úplně oddělené; `state.daily` i `state.backlogs` patří výhradně sekci
Tasks. Habits a Free jsou ale **dvě instance jedné sekce**: obě žijí ve
`state.sections[0]` / `[1]`, mají tentýž tvar `{habits, log}`, renderuje je tentýž
kód přes index a edituje **tentýž sheet** (`openSheet(curSection-2, …)`). Free jen
navíc ukládá tagy/délku/fun (větve `editingSection===1`) a sdílí s editorem návyku
globální slovník `TAG_GROUPS`.

*(Pozn. k historii: tohle README dřív popisovalo bug, kde `detailBack` volal
neexistující `renderOverview()` — opraveno na `renderOverviewPanel(detailSection)`
při navigačním překopání.)*

---

## 3. Datový model (localStorage klíč `habity.v2`)

> **PAST: klíče ve `state` nesedí na názvy sekcí v UI.**
> `state.tasks` = sekce **Progress** (projekty s kroky).
> `state.utasks` = sekce **Tasks** (úkoly s termínem).
> `state.sections[0]` = **Habits**, `state.sections[1]` = **Free**.
> Slovo „task" tak v kódu znamená tři různé věci podle místa — i `data-scope="tasks"`
> u backlog dropdownů míří na `utasks`. Je to historický relikt a **záměrně se
> nepřejmenovává**: klíče se serializují do localStorage, takže rename bez migrace
> `v2 → v3` smaže uživateli data. Až se migrace bude dělat z jiného důvodu,
> přibalit i tohle.

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
  lastRollup: "YYYY-MM-DD" | null,

  // ----- MOVE (Garmin aktivita, viz sekce 5) -----
  move: {
    days: { "YYYY-MM-DD": {kcal, durMin, distM, hr} },  // agregát dne (píše skript/import)
    caps: { kcal:679, dist:3729, hrBase:55, hrCap:130 }  // stropy prstenců; kcal/dist p90 z dat, hr napevno
  },

  // ----- LITÁNIE (viz sekce 6) — SOUKROMÁ data, jen v localStorage, nikdy online -----
  litBase: [ {id,text,temata,casti,priorita,oblibena,aktivni} ],             // základ z importu litanie.json
  litTemata: [...], litCasti: [...],                                         // definice tagů (z hlavičky jsonu)
  litOverlay: { "3": {text?,temata?,casti?,priorita?,oblibena?,aktivni?} },  // přepisy základních vět, klíč = string id
  litNew: [ {id:"ul…",text,temata,casti,priorita,oblibena,aktivni} ],        // uživatelem přidané věty
  litFilter: { temata:[], casti:[], prioMin:1, fav:false, noTags:false, expanded:false, search:"" }
}
```

**Pozn.:** `move.days` se **kumuluje** (import merguje, nemaže staré) — historie
přežívá, i když feed nese jen posledních 90 dní. `move` je celé **obnovitelné**
(z GitHubu / Garminu), ale i tak je součást exportu (celý `state`). `normalize()`
ho defenzivně doplní; klíč `habity.v2` beze změny.

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
Backlog/Dnes/Zítra + vždy viditelný datepicker; vyplnění data zhasne
segmenty, klik na segment datum vymaže) · opakování (8 segmentů) · **bouncer box** (zašedlý
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

## 5. MOVE — Garmin aktivita a sync pipeline

Sekce **Move** (pozice 4, tab úplně vpravo) ukazuje denní pohybovou aktivitu
z Garminu jako **kolečka po týdnech**. Cíl: „donutit se hýbat" přes rychlou
vizuální gestalt — jedno mrknutí a víš, jak plnej byl den, aniž bys četl čísla.

### 5a. Vizuál kolečka (jeden den = jedno kolečko)

- **Velikost kruhu = kcal** (souhrnná náročnost dne). Řídí gestalt „jak plnej den".
- **Číslo uvnitř (bold, černé) = doba trvání v minutách.**
- **Levá půlka prstence (zelená `--done`) = vzdálenost**, plní se zdola.
- **Pravá půlka prstence (červená `--missed`) = tep**, plní se zdola.
- **Prázdný den = jen tečka.**

**Stropy** (`state.move.caps`): plná půlka/plný kruh = strop.
- `kcal` a `dist` = **90. percentil z vlastní historie**, přepočítá se při každém
  importu (`recalcMoveCaps`) — vizualizace „roste s tebou".
- `hr` = **napevno baseline→cap `55→130`** (klidový tep → zátěž). Baseline proto,
  aby prázdná půlka znamenala klid, ne „jen málo". Přepočítávat z dat by u řídkého
  nošení hodinek dalo nesmyslný strop.

**Render (`renderMove`/`moveCell`):** řádek = týden (**Po vlevo, Ne vpravo**),
**nejnovější týden nahoře**, nekonečný scroll od prvního záznamu po dnešek.
Kolečka jsou **plně responsivní**: SVG má fixní `viewBox 0 0 100 100` a
`width:100%` buňky, takže se škálují šířkou obrazovky, tloušťka prstence zůstává
konstantní (fix stroke), průměr roste s kcal. `.move-week` je flex se 7 buňkami.

Historická poznámka: kJ jsme zavrhli (silně koreluje s tepem×časem), jedeme kcal.
Data se **kumulují** v localStorage (viz sekce 3), takže scroll roste s historií
(~1 řádek / týden, po 2 letech ~104 řádků — kapacita žádný problém).

### 5b. Datový most: appka ↔ Garmin

Appka je statická PWA — **s Garminem nikdy nemluví přímo** (CORS/OAuth to z
prohlížeče nedovolí). Mezi nima stojí **PC skript**. Tok je **jednosměrný**:

```
Garmin Connect
   │  build_data.py (Python) — token login, stáhne 90 dní, agreguje po dnech
   ▼
move_data.json (KOŘEN repa)  → git commit + push (jen když se změní)
   ▼
GitHub Pages  → appka si ho při startu tiše fetchne (fetchMoveData, cache:no-store)
   ▼
sekce Move   — plná automatika, nulová ruční práce
```

- **Appka strana:** `fetchMoveData()` (volaná v initu) stáhne `move_data.json`
  z Pages a naimportuje `importMoveData(arr, silent=true)` — bez toastu, tiše.
  Offline/chybějící soubor = ignoruje se, jede z localStorage. **Ruční import** (nahrání
  souboru) už v UI není (byl v ⚙, odstraněn — auto-fetch ho nahradil).
- **Ruční refresh** (⚙ Data → „Znovu načíst Move data z GitHubu", `#btnRefreshMove`):
  znovu stáhne `move_data.json` z Pages (`?t=`+timestamp cache-busting + `cache:no-store`)
  a přepíše lokální dny přes `importMoveData(arr, silent=false)` — **s hláškou** (i při
  selhání). Použití: Bob **ručně upraví json na GitHubu** (např. dopíše vzdálenost k
  aktivitě spuštěné jako Cardio místo Walk), počká na Pages deploy, pak v appce refreshne.
- **Agregace dne** (ve skriptu): kcal/trvání/vzdálenost = součet aktivit dne, tep
  = průměr vážený délkou. Bereme **všechny aktivity** (chůze, tenis, plavání…),
  jen ručně **zaznamenané** (Garmin `get_activities`; celodenní kroky bez
  spuštěného záznamu se do feedu nedostanou — Bobovo vědomé rozhodnutí, varianta A).

### 5c. Motor syncu — `local/garmin/` (git-ignorováno)

Celý motor žije v `local/garmin/` (mimo git, osobní/citlivé):
- **`build_data.py`** — skript. Píše `move_data.json` do **kořene repa**
  (`REPO_ROOT`), pak commit+push. Mění jen datový soubor, takže s lidským vývojem
  (jiné soubory) **nikdy nekonfliktuje**; před push dělá `pull --rebase --autostash`.
- **`.garmintoken/`** — přihlašovací token (garth). **Login heslem jen poprvé**,
  pak token (bez hesla, bez `429` rate limitu). Token vyprší ~po roce → skript si
  při ručním běhu řekne o heslo; při automatickém běhu bez konzole to jen
  zaloguje a skončí (nezasekne se na `input()`).
- **`sync.log`** — log běhů.

**Automatika:** Windows **Task Scheduler**, úloha `Habity\GarminSync`, spouští
`pythonw.exe build_data.py` (bez okna) **3× denně v 10:00 / 18:00 / 22:00**,
`StartWhenAvailable` (dožene zmeškané). Git i pythonw čtou systémový PATH.

**Soukromí:** `move_data.json` je v kořeni → **veřejně čitelný na Pages URL**.
Bob to vědomě schválil (Garmin aktivita není citlivá jako osobní texty). **Osobní
sekce (návyky/tasky) se takhle NIKDY veřejně vystavovat nesmí** — GitHub Pages
nemá autentizaci.

### 5d. Zálohování (klasický export)

⚙ → „Exportovat zálohu" stáhne **`habity-zaloha.json`** — **pevný název**
(přepisuje se, nehromadí se v Downloads). Obsahuje **celý `state` včetně Move**.
Bobův režim: přepisovaný soubor houpne občas do Dropboxu, občas přejmenuje s
datem = rolling backup + milníky. localStorage je jediná ostrá kopie, dokud se
nevyexportuje → **zálohovat pravidelně**.

**Budoucnost (fáze 3, neřešeno):** desktop↔mobil **sync osobních sekcí** přes
**privátní autentizovanou vrstvu** (ne veřejné Pages). Obousměrný živý stav =
riziko konfliktů + nutná auth. Velký samostatný projekt, až bude potřeba.

---

## 6. LITÁNIE — podpůrné věty s tagy (pozice 5)

Sekce **Litánie** (v navu mezi Free a Move, `curSection=5`) je knihovna
podpůrných vět / přerámování / manter / vděčností, vytažená z Bobova Evernotu
(10 let, deduplikace + AI tagging). **1254 vět**, tagované tématy (22) a částmi
(13, IFS-style figury). Účel dvojí: **čistit** (Bob prochází a vyhazuje balast)
a **tvořit** (přidávat nové). UI je klon sekce Free — chip filtry, fulltext,
slider priority.

### 6a. Datová architektura — SOUKROMÁ data, nikdy ne online (DŮLEŽITÉ)

> ⚠️ **Litánie jsou hodně osobní a NESMÍ jít na veřejný GitHub/Pages.** `litanie.json`
> se **nikdy necommituje** — bydlí v `local/` (globálně git-ignorováno). Na rozdíl
> od Move (Garmin data = veřejně OK) se litánie do repa nedostanou. Pozor při
> jakémkoli commitu, ať to tam omylem neproklouzne.

Zdroj se **jednorázově naimportoval z disku** do localStorage (import už hotový,
tlačítko odstraněno — viz níž). Odtud data žijí jen v localStorage a v JSON exportu
— **jako zbytek appky** (návyky/tasky). Čtyři kusy ve `state`:
- `litBase[]` — základní věty (z importu). `litTemata[]` / `litCasti[]` = definice
  tagů z hlavičky jsonu. **Toto je jen v localStorage, git to nekryje → zálohovat
  exportem.**
- `litOverlay[id]` — částečné přepisy základních vět (text/temata/casti/priorita/
  oblibena/videno/aktivni). Klíč = **string** id. Ukládá se jen změněné pole.
- `litNew[]` — uživatelem přidané věty (id `"ul"+timestamp`).
- `litFilter` — persistovaný stav filtru.

**Runtime cache:** `LIT_BASE`/`LIT_TEMATA`/`LIT_CASTI` (globály) se při startu
naplní ze `state` přes `litLoadFromState()` (v initu, místo dřívějšího fetche).
`LIT_LOADED = LIT_BASE.length>0`.

**Import:** funkce `importLitanie(d)` v kódu **zůstala** (naparsuje `litanie.json`,
naplní `state.litBase` + definice tagů, `save()`, `litLoadFromState()`, reimport drží
overlay/litNew přes stabilní id), ale **UI tlačítko `#btnImportLit` už bylo odstraněno**
— data jsou dávno nasátá v localStorage/exportu (stejně jako se to udělalo s Garmin
importem). Funkce je tak momentálně bez UI volajícího; kdyby byl potřeba reimport, dá
se dočasně zavěsit zpět.

**Slití (`litEffective`):** vezme `LIT_BASE`, na každou větu napasuje
`litOverlay[id]` (přebije základ), přidá `litNew`. Vrací sjednocený seznam s
flagem `_new`.

**Proč import a ne Pages fetch:** Pages je veřejné bez auth (viz sekce 5d, stejný
důvod proč se osobní sekce nesmí vystavovat). „Push na jedno stažení a pak smazat"
= iluze bezpečí (git historie, forky, cache, archivy). Proto data nikdy neopustí
disk/localStorage. Viz Bobova paměť „litánie jsou soukromé".

### 6b. UI a interakce

- **Seznam:** karty `P{priorita} · text · hvězdička` (priorita vlevo, hvězda u pravého
  kraje). Klik na text = editace (`openLitSheet`), klik na hvězdičku = toggle oblíbené
  (`litToggleFav`). Delegace: **jeden** click handler na `#litList` (ne 1254× per karta).
- **Strop vykreslení 5000 karet** (`CAP` v `renderLitList`) — jen pojistka proti
  extrému (desetitisíce karet by při re-renderu na každý stisk v searchi sekaly).
  Běžná DB (~1250) se zobrazí celá. Přes strop se ukáže poznámka „…a dalších N".
  Řazení: oblíbené nahoře → priorita DESC → id ASC.
- **Filtr** (`renderLitFilterBar`, klon Free): dvě chip skupiny (Témata modré,
  Části červené), slider priorita-min (1–5), mini-toggly „Bez tagů" a **„👁 Neviděné"**
  (reverse — ukáže jen položky s `videno=false`, tj. ještě neprošlé), Reset, sbalení.
  **Rychlá hvězdička** (jen oblíbené, `#litFavQuick`) sedí **vedle keyword searche**
  (ne ve filter baru), aby měl filtr plnou šířku. Fulltext (`litSearch`) sdílí fuzzy
  engine s Free (`normSearchText`/`fuzzyWordMatch`).
- **Editace/přidání** (`litSheet`): `<textarea>` (věty jsou i dlouhé odstavce, max
  1570 znaků — auto-grow do 40vh), slider priority 1–5, řádek se dvěma toggly —
  **oblíbená** (☆/★) a **shlédnuto** (👁 očíčko, `#litSeenToggle` → `videno`), chipy
  témat a částí. Očíčko slouží k **procházení databáze** — po dokončení editace věty
  ho cvakneš a věta zmizí z filtru „Neviděné". Přidání přes FAB (`openLitSheet(null)`).
  Ukládání: nová → `litNew.push`, úprava základní → zápis do `litOverlay[id]`, úprava
  `_new` → mutace objektu.
- **Mazání = `aktivni:false`** (`litSetAktivni`), ne fyzické smazání — Bob může
  vzít zpět a za půl roku uvidí, co vyházel. Po smazání **undo lišta** (`#undoBar`,
  6 s, tlačítko „Vrátit" → `litSetAktivni(id,true)`, přičemž pokud overlay držel
  jen `aktivni`, celý se smaže, ať nezůstává balast).
- **Počítadlo** (`#litCount`): „`<zobrazeno>` z `<aktivních>` · `<N>` vyházeno".

### 6c. Rotace „piňa dne" — VĚDOMĚ NEUDĚLÁNO

Alterego navrhovalo denní váženou rotaci (servírovat větu dne dle priority+oblíbené).
Bob to pro první verzi **odmítl** — „na špinavý databázi to stejně servíruje blbosti".
Až bude databáze pročištěná, je to kandidát na druhé kolo. Priorita a `oblibena` v
datech na to už jsou připravené.

---

## 7. Vizuál / styl

Tmavé „kokosové" téma. CSS proměnné v `:root`:
`--bg:#15120E --surface:#201B15 --surface2:#2A241C --border:#3A332A
--text:#EDE6D8 --dim:#9A9081 --empty:#34302A --done:#74955E --missed:#BC5B43
--accent:#D9A441`. Fonty: serif (Georgia) pro názvy, sans (system) pro UI, mono
pro metadata/čísla. Bob má rád hustotu (malé paddingy, malé písmo).

**🍹 Piňa colada odměna** (`pina()`): po `✓` splnění vyskočí velké emoji 🍹
uprostřed obrazovky, zhoupne se a zmizí (`@keyframes pinapop`, ~0.9s). Restart
ošetřen pro rychlé klikání. Jen na ✓ (ne ✕ ani přesun). Dopaminová odměna.

---

## 8. Co je hotové a co Boba čeká

**HOTOVÉ a matematicky otestované** (přes Node simulace): repeat respawn (7
scénářů včetně blogu/finančáku/přetečení 31.→28.), rollup (mix repeat/ne-repeat/
budoucí/backlog), denní agregát, cap logika Přehledu, v2 migrace, Free backlog
migrace (stará data → „Free", validace neplatných backlogů → fallback).

**Navigace překopaná naruby** (taby=sekce, swipe=Check/Přehled, ⚙ v headeru) —
viz sekce 2. Sourozenecký sync renderů ověřen ručně.

**Move + Garmin pipeline HOTOVÁ a ověřená v provozu** (viz sekce 5): token
cache, automatika 3× denně (Task Scheduler), auto-push `move_data.json` do repa,
auto-fetch v appce, responsivní kolečka. Ověřeno reálnými daty z Bobova Garminu.
Zbývá jen doladit vizuál Move „na závěr všeho" (velikosti/mezery detaily) a
případně „ukázat posledních X týdnů" místo nekonečného scrollu.

**Reálně neověřené (čeká na běh času):** rollup se na ostro projeví až den po
naplánování úkolu (musí propadnout přes půlnoc). Logika je testovaná na
simulovaných datech, ale reálný průchod časem Bob uvidí až s reálným časem.
U Move: **reálný auto-commit+push** se spustí až přijdou nová data (zatím
ověřeno jen „beze změny nic k pushnuti"); dlouhý výpadek syncu >90 dní udělá
nedoplnitelnou díru (skript se tak hluboko nedívá) — dá se zvednout okno `DAYS`.

**Litánie HOTOVÉ a ověřené** (viz sekce 6): lokální import + overlay architektura
otestovaná end-to-end přes DOM/localStorage (import 1254 vět → localStorage,
fav→overlay, delete+undo, edit→overlay, nová věta→litNew, reload bez fetche →
data z localStorage, celý state v exportu), fuzzy search, chip filtry, strop 5000,
undo lišta. Ověřeno, že appka na startu `litanie.json` **nikde nestahuje**
(žádný síťový request). Screenshot nástroj v prostředí timeoutoval, ale reflow
6 ms + 0 chyb → appka svižná. Import **už proběhl** (data v localStorage/exportu),
import tlačítko odstraněno. Přibylo **procházení databáze**: flag `videno` na položce
(toggle 👁 v editaci) + filtr „👁 Neviděné" na odškrtávání, co Bob ještě neprošel.
Čištění databáze je na Bobovi (dlouhodobě); rotace „piňa dne" odložená (6c).

**Filozofie dalšího vývoje:** Bob řekl „základ máme, bude to o dolaďování během
úkolování". Čekej drobné UX úpravy z reálného provozu, ne velké přestavby.
Nepřeengineeruj dopředu. Když Bob přijde s úpravou, zeptej se postupně po jedné
otázce na nejasné hrany, pak to postav a otestuj.

---

## 9. Tipy pro zásahy do kódu

- Soubor je strukturovaný: `<style>` → HTML (header s ⚙, pager se 2 tečkami,
  **5 `<section>` pages** — 0–3 s `track`/dvěma `panel`, `secMove` s jedním —
  sheety, modaly) → `<script>` (state/normalize/load/save → date helpers →
  NÁVYKY → projektové ÚKOLY → TASKS úkolová sekce → Přehled úkolů →
  confirm/toast/piňa → settings → **MOVE (renderMove/moveCell/importMoveData/
  fetchMoveData/recalcMoveCaps)** → **LITÁNIE (litLoadFromState/importLitanie/
  litEffective/matchesLitFilter/renderLitList/renderLitFilterBar/openLitSheet/
  litDelete+undo)** → navigace (swipe/taby/pager) → FAB → init).
- **Navigace:** `updateUI()` je centrální — transformuje tracky podle `curView`
  (Move track vždy 0), přepíná `.page.active` podle `curSection`, syncuje
  taby/pager/title, skrývá pager+FAB na Move. Taby nastavují `curSection`, swipe
  a pager tečky nastavují `curView`.
- `renderCheck()` = router pro úkolovou (Tasks) Check stranu (default + backlog
  view).
- `renderAll()` volá všechny rendery najednou (Check i Přehled všech sekcí jsou
  v DOMu pořád, jen schované), **včetně `renderMove()`**. Sourozenecký sync (viz
  sekce 2) je proto nutný. Auto-fetch Move (`fetchMoveData()`) se volá zvlášť na
  konci initu (async).
- **Move/Garmin:** vizuál a datový model v `index.html` (sekce 5), ale sync motor
  je v `local/garmin/` (mimo git). Když měníš tvar `move_data.json`, musí sedět
  `build_data.py` (výstup) ↔ `importMoveData` (vstup) ↔ `moveCell` (render).
- Po každé změně: `node -e` syntax check (`new Function`), kontrola párování
  `<div>`, ověření že volaná `getElementById` ID existují v HTML.
- Datové helpery navíc pro úkoly: `parseKey`, `fmtCalLabel`, `addMonths`,
  `bumpDaily`, `utaskById`, `sortUtasks`, `backlogTasks`, `calendarGroups`,
  `prioColor`, `repeatShort`.
- Free backlogy: `renderFreeBacklogBar`, `fillHabitBacklogPicker`,
  `freeBacklogCurrent` (viz sekce 3b).
