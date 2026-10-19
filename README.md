Ahoj osle!

# HABITY — README pro pokračování

> **Stav k 11. 10. 2026** (commit po `cbac3fa`). README bylo celé zrevidované
> proti kódu: sekce po sekci, s čísly řádků. Kde se kód a tenhle text rozejdou,
> **pravdu má kód**. Text pak oprav ve stejném commitu, ve kterém měníš kód.

Tohle čte Claude, který přebírá práci na appce **Habity** v nové konverzaci.
Cílem je zahučet do věci bez ztráty kontextu. Čti to celé, ať chápeš nejen
*co* tam je, ale hlavně *proč*. Spousta rozhodnutí má za sebou odladěné hrany.
Stručný rozcestník a pracovní pravidla jsou v `CLAUDE.md`. Uživatelský návod
pro lidi je v `NAVOD.md`.

---

## 0. Jak s Bobem pracovat (přečti dřív než cokoli jiného)

- **Bob mluví česky, nespisovně, jako se starým kámošem.** Mluv s ním stejně.
  Oslovuj ho „Bobe“, ne „kámo“. Humor, nadávky v dobrém a emoji 🥥🍹 sedí.
  Bere tě jako rovnocennou bytost, „kokosového spolukrále prdele“, tak s ním
  jednej jako rovný s rovným.
- **Měj vlastní názor a buď přesný.** Uvolněný tón nesmí ubrat na ostrosti
  úsudku. Když je něco blbě, řekni to rovnou, bez vycpávek typu „tady ti to
  řeknu na rovinu“.
- **Když ladíte design, ptej se postupně, po JEDNÉ otázce.** Bob odpovídá
  telegraficky a věří, že implementační detaily domyslíš sám.
- **Když řekne „jeste nepracuj“, opravdu nepracuj.** Jen prozkoumej a navrhni.
  Kód sáhni, až kejvne.
- **Proaktivně flaguj** edge-casy, architektonická rizika a UX díry, které ho
  nenapadly. To oceňuje nejvíc.
- **Diktuje přes mikrofon.** Zkomoleniny si oprav sám: „VIP“ = Wip, „DAN“ =
  Done, „food/FUT“ = Future (už neexistuje).
- **# na začátku promptu = odpověz jednou větou.**
- Když hledáš na webu, dej přednost mezinárodním zdrojům před českými.

---

## 1. Co je Habity

Osobní tracker návyků, úkolů, pohybu a podpůrných vět. **Jednosouborová PWA**
(`index.html`), hostovaná na **GitHub Pages** (`haryzek.github.io/habity/`).
Roste organicky, „intuitivně, po malých kouscích mezi jinou prací“. Features
se přidávají podle reálné potřeby, ne dopředu přeengineerované.

**Sdílená dohoda Bob ↔ Claude:** stavět dál do jednoho souboru, dokud to drží
pohromadě. Claude má **proaktivně křiknout**, až bude refactoring nebo rozdělení
souboru na místě. *Aktuálně má soubor ~5340 řádků a ~240 KB. Pořád se to dá
udržet, ale je to hranice, kde už se vyplatí hledat přes `grep` a ne číst
celé. Bob řezání nemá rád dělat předčasně.*

**PWA:** iOS meta tagy a manifest i ikona jsou inline jako `data:` URI
(`display:standalone`). **Service worker není**, takže appka jde přidat na
plochu, ale offline cache nemá.

### Prostředí a dodávka

- **Lokálně:** git repo na `E:\_dev\habity` (Claude Code, Windows/PowerShell).
  Edituje se přímo `index.html`, po zásahu `git commit` a `git push` do `main`.
  GitHub Pages pak sám nasadí (pár desítek sekund; ověř `curl` na Pages URL).
- **Cloud (claude.ai/code):** Bob s Claudem pracuje i v cloudových sessions
  (třeba na dovolené, 9–10/2026). Tam se pracuje na přidělené větvi
  `ccr-…`, ta se pushne a **na Bobův pokyn se mergne do `main`**. Cloud
  nevidí složku `local/` ani `gh` CLI. Po cloudové práci si Bob dělá lokálně
  `git pull`.
- **Osobní bordel a tokeny patří do `local/`.** Je globálně gitignorovaná přes
  `core.excludesFile` (viz Bobova paměť). V repu je `.gitignore` jen na
  `.claude/` a `_scratch/`.
- **localStorage přežívá update** (je vázaná na doménu). Je to **jediná ostrá
  kopie dat**. Záloha = export JSON (viz 5d).
- **Automatické commity „Auto: update Garmin data“** vznikají **přímo
  v Bobově lokálním repu** (`E:\_dev\habity`, Task Scheduler, viz 5c). Skript
  je odtud pushuje a předtím si udělá `pull --rebase`. Lokál je jejich zdroj,
  takže **lokálně se kvůli nim pullovat nemusí**. Pull je potřeba jen po práci
  v cloudu. **Cloudová session** má naopak před pushem nebo mergem udělat
  `git fetch`, protože na GitHubu mezitím mohly přibýt Garmin commity.

### Workflow pro Claude

- **Vždycky si `index.html` načti znovu před zásahem.** Nespoléhej na paměť
  z chatu, konverzace bývají dlouhé a paměť klame.
- **Po zásahu ověř JS syntax** přes Node (`new Function(code)` nad obsahem
  `<script>`) a párování tagů.
- **Funkčně testuj v headless prohlížeči.** V cloudu je Playwright a Chromium
  (`$(npm root -g)/playwright`, prohlížeče v `/opt/pw-browsers`). Postup:
  1. Otevři `file://…/index.html`.
  2. Nastav `localStorage["habity.v2"]` na testovací stav.
  3. `reload()`.
  4. Volej funkce nebo klikej a hlídej `pageerror`.
  Takhle byly ověřené Mix, slider i „terapie“.
- **Datové migrace:** klíč zůstává `habity.v2`. Nové pole doplní `normalize()`.
  Na přejmenování systémových backlogů je čítač `blV` (viz 3).

---

## 2. Struktura appky a navigace

### Taby a sekce

**Spodní lišta = 6 tabů = 6 sekcí.** Vizuální pořadí zleva: **Litánie ·
Progress · Tasks · Habits · Free · Move** (ikony `❝ ▤ ✓ ↻ ☆ ◉`). Při šířce
`max-width:430px` se schovají popisky (`.tab .tl`) a zůstanou jen ikony.
Ikony optický centruje `centerTabIcons()` (měří glyf na canvasu, volá se po
`document.fonts.ready`).

⚠️ **Past: pořadí v navu ≠ `curSection` index.** Litánie je v navu první, ale
má **`curSection=5`**. Udělalo se to schválně, aby se nepřečíslovávala Move
(`curSection===4` je na spoustě míst v kódu). Pořadí `.page` sekcí i tabů je
čistě vizuální, logiku řídí index. **Když přidáváš sekci, dej jí nový index na
konec a tab umísti, kam chceš.**

| idx | tab | panely v tracku | default view | co to je |
|---|---|---|---|---|
| 0 | **Progress** | 1 (seznam projektů) | 0 | Projekty s odškrtávacími kroky, vážený progres (3c). |
| 1 | **Tasks** | 1 (Chill náhled + kalendář; v něm i backlog view) | 0 | Kalendářní task manager s backlogy a repeaty (4). **Startovní sekce** (`curSection=1`). |
| 2 | **Habits** | 2: Check · Přehled | **1 (Přehled)** | Habit tracker, mřížka 90 dnů (3a). |
| 3 | **Free** | 2: Check · Přehled | **1 (Přehled)** | Volnočasové aktivity se štítky, filtrem a hledáním (3b). |
| 4 | **Move** | 5: Nálada · Spánek · Garmin · Běh · Otužování | 0 (Nálada) | Kolečka po týdnech (5). |
| 5 | **Litánie** | 2: Litánie · Otázky | 0 | Dvě kolekce vět s tagy. **Soukromá data** (6). |

`SEC_PAGES` / `SEC_TRACKS` / `SEC_TITLES` / `SEC_DEFAULT_VIEW = [0,0,1,1,0,0]`
mají 6 položek. Klik na tab nastaví `curSection` a **resetuje `curView`** na
`SEC_DEFAULT_VIEW[curSection]`.

Názvy sekcí v UI jsou anglicky, zbytek appky česky. `hTitle` v hlavičce ukazuje
název sekce, jen v **Move** a **Litániích** se mění podle plochy:
- Move: Nálada / Spánek / **„Move“** (Garmin plocha nemá vlastní název) / Běh /
  Otužování.
- Litánie: Litánie / Otázky.

### Swipe, pager a FAB

- **Swipe** = Pointer Events API (ne touch events, kvůli testování myší).
  `touch-action: pan-y` nechává vertikální scroll prohlížeči. `attachSwipe` je
  napojený na **všech 6 sekcí** a meze se počítají z počtu panelů
  (`0..children.length-1`, na krajích odpor 0.3). Progress a Tasks mají jen
  jeden panel, takže se jen pružně odrazí. Gesta, která začnou na
  `input[type=range]` nebo na řádku rychlých voleb (`.blog-quick-row`), se
  ignorují.
- **`applyViewToTrack`** je generický:
  `translateX(-min(curView,n-1)*100/n %)`. `curView` je jedna globální
  proměnná. Když swipneš v Move na view 4 a přejdeš jinam, clamp to ošetří
  a klik na tab ho stejně resetuje.
  Neaktivní plochy dostanou `.panel.off` (`height:0`), jinak by track měl
  výšku nejdelší plochy sekce a krátká plocha by scrollovala do prázdna.
  Skládá se až po dojetí animace (260 ms); swipe je přes `unfoldPanels`
  rozloží hned, jak se gesto rozhodne pro vodorovný směr.
- **Pager** (5 teček nahoře): na Progress a Tasks je schovaný přes
  `visibility:hidden` (ať layout neposkakuje). Tečky 2–4 jsou vidět jen v Move.
  Habits, Free a Litánie mají 2 tečky.
- **`updateUI()`** je centrální. Volá `resetScrollOnPageChange()` (při změně
  `sekce:view` odscrolluje nahoru), posune tracky, přepne
  `.page.active` / `.tab.active` / tečky, nastaví `hTitle` a pager a **schová
  FAB jen na Move**.
- **FAB „+“:**
  - Progress: když je rozbalený **právě jeden** projekt, přidá do něj krok
    (`openTaskSheet(id,true)`), jinak založí nový projekt.
  - Tasks: `openUtSheet(null)`.
  - Habits / Free: `openSheet(curSection-2, null)`.
  - Litánie: `openLitSheet(curView===1 ? LIT_COLS.ot : LIT_COLS.lit, null)`.
  - Move: nic (FAB je schovaný).
- **Klávesnice a sheety:** `visualViewport` nastavuje CSS proměnné
  `--kb`/`--vvh`, aby sheet nepřekryla klávesnice.

⚠️ **Past:** `#trackMove .panel` **nesmí dostat `overflow-y:auto`**. Vlastní
scrollovací kontejner na Androidu sebere vodorovné gesto (Chrome zahájí scroll
a pošle `pointercancel`) a swipe jen cukne a odskočí. Move scrolluje oknem jako
všechny ostatní sekce.

⚠️ **Layout past (vyřešená):** `.wrap` je `display:flex` column bez stretch,
takže se smrskával na šířku obsahu a úzká Move se lepila vlevo. Fix je
`.wrap{width:100%; max-width:560px}`.

### Hlavička a ⚙ Data

Hlavička obsahuje `hTitle`, datum (`hDate`) a **ozubené kolečko ⚙**. To
otevře sheet „Data“ s těmito tlačítky:
1. **Exportovat zálohu (JSON)**: stáhne `habity-zaloha.json` s celým `state`.
2. **Načíst zálohu ze souboru**: přijme objekt se `sections`/`habits`/`tasks`,
   po potvrzení **přepíše všechno**, projede `normalize()` a `renderAll()`.
3. **Vymazat historii úkolů**: `state.daily = {}`. Popisek je relikt, Přehled
   úkolů už neexistuje (viz 4g).
4. **Znovu načíst Move data z GitHubu** (viz 5b).
5. Zavřít.

### Init sekvence (konec `<script>`)

1. `var state = load()`: načte `habity.v2`, jinak zmigruje `habity.v1`, jinak
   `blankState()`. Vždycky přes `normalize()`.
2. Napojí Free search inputy a nastaví `hDate`.
3. `renderAll()`: `renderTasksPanel`, `renderCheck`, `renderFreeFilterBar`,
   `renderTodayPanel(0/1)`, `renderOverviewPanel(0/1)`, `renderMove`,
   `renderCold`, `renderVibe`, `renderSleep`, `renderRun`, `refreshLit`.
4. `updateUI()`.
5. `fetchMoveData()` a `fetchRunData()` (async, tiše).
6. `litLoadFromState()`: Litánie se tak vykreslí podruhé. Neškodí to, jen první
   render proběhne před naplněním cache.
7. `centerTabIcons()` po načtení fontů.

**Žádný rollup úkolů se při startu nedělá** (zrušen 2. 7. 2026, viz 4e).

### Sourozenecký sync renderů

Check a Přehled u Habits a Free jsou v jednom tracku vedle sebe a během swipu
jsou vidět oba, takže akce na jedné straně musí refreshnout i druhou:
- Check toggly volají `renderTodayPanel` i `renderOverviewPanel`.
- Klik do buňky Přehledu přepíše svou buňku a statistiku v hlavičce. U Habits
  volá `refreshGodzilla()`. `renderTodayPanel` volá **jen když klikneš na
  dnešek**.
- ⚠️ Klik v detailu (`openDetail`) nesyncuje nic. `detailBack` přerenderuje jen
  Přehled. Když v detailu změníš dnešek, Check panel zůstane zastaralý až do
  dalšího renderu (drobný bug, viz 8).

Progress a Tasks mají jednu plochu: subtask toggly volají `renderTasksPanel`,
akce úkolů `renderCheck`.

---

## 3. Datový model (localStorage klíč `habity.v2`)

> **PAST: klíče ve `state` nesedí na názvy sekcí v UI.**
> - `state.tasks` = sekce **Progress** (projekty s kroky).
> - `state.utasks` = sekce **Tasks** (úkoly s termínem).
> - `state.sections[0]` = **Habits**, `state.sections[1]` = **Free**.
>
> Slovo „task“ tak v kódu znamená různé věci podle místa. I `data-scope="tasks"`
> u backlog pickeru míří na `utasks`. Je to historický relikt a **záměrně se
> nepřejmenovává**, protože rename klíčů bez migrace by smazal data.

Klíč zůstal `habity.v2`. Nové struktury přidává `normalize()` nedestruktivně a
doplní chybějící pole. Migrace v1→v2 existuje: `load()` načte `habity.v1`,
znormalizuje ho a zapíše jako v2. Komentář na začátku `<script>` schéma
neudržuje a odkazuje sem. Platí tohle:

```
// localStorage "habity.v2"; normalize() doplní vše chybějící
{
  // ----- HABITS + FREE -----
  sections: [
    { // [0] = HABITS — habits projdou normalize() RAW (bez validace)
      habits: [ {id:"h…", name, priority:1-10, createdAt:ms, star?:bool} ],  // star = Godzilla
      log: { [habitId]: { "YYYY-MM-DD": "done"|"missed" } }                 // chybí klíč = nic
    },
    { // [1] = FREE — ŽÁDNÉ backlogy (normalize zahodí cokoli navíc, i backlog/star)
      habits: [ {id, name,
                 priority:1-10,        // v UI „Zábava“ (badge Z{n}), filtr funMin
                 createdAt:ms,
                 tags:[…],             // z TAG_GROUPS; neznámé se zahodí, TAG_RENAME migruje
                 durationMin:0-16,     // index do DURATION_TICKS, default 0
                 durationMax:0-16} ],  // default 3 (=30min); normalize prohodí min>max
      log: { …jako [0]… }
    }
  ],
  freeFilter: { tags:{ [groupIdx]:["Tag",…] }, durationMin:0, durationMax:16,
                funMin:1, expanded:false, search:"" },   // sdílený Check i Přehledem Free

  // ----- PROGRESS -----
  tasks: [ {id:"t…", name, createdAt, completedAt:ms|null,
            subtasks:[ {id:"s…", name, weight:>=1 (def 5), cells:>=1|null, done:bool} ]} ],
            // ŽÁDNÉ priority: pořadí = pořadí v poli

  // ----- TASKS -----
  backlogs: ["Chill","Done","Wip","Mix", …vlastní],  // první čtyři garantované, systémové
  blV: 3,                     // verze systémových backlogů: <2 Wip→Chill + Future→Wip; <3 vlastní „Mix“→„Mix 2“
  taskBacklogSel: ["Chill"],  // poslední výběr backlogů (jen existující)
  mixMinPrio: 10,             // práh agregátoru Mix (1-10), „N a víc“
  backlogOrder: { [název]: n>=1 },   // pořadí prefixů v rychlých volbách (není v blankState, doplní normalize)
  utasks: [ {id:"u…", name, priority:1-10 (def 1), createdAt,
             date:"YYYY-MM-DD"|null,  // null = bydlí v backlogu; jinak v kalendáři
             backlog:"Chill",         // domovský backlog; nikdy "Mix" (normalize → Chill), neplatný → Chill
             repeat:"none"|"daily"|"weekly"|"biweekly"|"monthly"|"quarterly"|"halfyear"|"yearly",
             bouncer:null|{dow:0-6}|{date:"YYYY-MM-DD"},
             done:bool} ],
  daily: { "YYYY-MM-DD": {done:n, missed:n} },   // denní agregát; počítá se, nikde se nezobrazuje

  // ----- MOVE (sekce 5) -----
  move:  { days:{ "YYYY-MM-DD":{kcal,durMin,distM,hr|null} },        // ze syncu, merge
           overrides:{ "YYYY-MM-DD":{kcal,durMin,distM,hr|null} },   // ruční zámky (sync nepřepíše)
           caps:{ kcal:600, dist:5000, hrBase:55, hrCap:130 } },     // defaulty; kcal/dist = p90 při importu
  vibe:  { days:{ "YYYY-MM-DD":{mood,soc,act,body,head} } },         // 1-5 (UI chce všech 5)
  sleep: { days:{ "YYYY-MM-DD":{dur, bed:"HHMM"|null, feel:1-10|null, fall:min|null, dream:n|null} } },
  run:   { days:{ "YYYY-MM-DD":{distM,durSec,movSec,hr|null,vo2|null} },
           manual:{ "YYYY-MM-DD":{heavy:0-10|null, ach:0-10|null} } },
  cold:  { days:{ "YYYY-MM-DD":{tw, ta|null, min, wind|null, flow|null} } },

  // ----- KOLEKCE VĚT (LIT_COLS, sekce 6) — SOUKROMÉ, nikdy online. <k> = "lit" | "ot" -----
  litBase:    [ {id:String, text, temata:[], casti:[], skupiny:[], priorita, oblibena, aktivni} ],
  litTemata:  ["…", "terapie"],                            // "terapie" vždy zajištěná (LIT_TEMATA_SEED)
  litCasti:   [ … ],
  litSkupiny: ["Otázky","Litánie","Udělat","Výzvy", …],    // seed jen v prázdném stavu / importu
  litOverlay: { [id]: {text?,temata?,casti?,skupiny?,priorita?,oblibena?,aktivni?} },
  litNew:     [ {id:"ul…", …tvar jako litBase} ],
  litFilter:  { temata:[], casti:[], skupiny:[], prioMin:1, fav:false, noTags:false, expanded:false, search:"" },
  otBase:[…], otTemata:[], otCasti:[], otSkupiny:[], otOverlay:{}, otNew:[{id:"uq…"}], otFilter:{…}
}
```

**Konstanty, které se neukládají:**
- `TAG_GROUPS`: štítky Free.
- `TAG_RENAME`: migrace starých názvů štítků.
- `DURATION_TICKS`: 17 kroků `0,5min,15min,30min,1h,2h,3h,6h,1d,3d,1t,2t,1m,3m,6m,1r,Delší`.
- `GRID = 90`.
- `LIT_COLS` a `LIT_TAG_RENAME`.

**Proč denní agregát a ne per-task event log:** stačily počty zelených a
červených, ne identita úkolů. Agregát má ~15 KB za rok, per-task log by měl
0,5–1 MB za rok a zatěžoval by každý `save()` (serializuje celý stav při každém
ťuknutí). Dnes se `daily` nezobrazuje vůbec (Přehled úkolů je zrušený, viz
4g), ale počítá se dál.

### 3a. Habits (`sections[0]`)

- **Check:** karta na habit, tlačítka ✓/✕ jsou **nezávislé toggly**. ✓
  nastaví nebo zruší `done`, ✕ nastaví nebo zruší `missed`. Řazení
  `sortedHabits`: priorita DESC, pak createdAt ASC (platí i pro Free).
- **Přehled:** blok na habit. Hlavička je `P{priorita}`, název, `nn/90` a šipka.
  Pod ní je mřížka **90 dnů** (`repeat(30,1fr)`, 3×30). Klik do buňky cykluje
  nic → done → missed → nic (`cycle()`), takže jde zapisovat i zpětně.
- **Hlavička bloku je rozdělená:** klik na název (`.name`, `stopPropagation`)
  otevře **editaci** (`openSheet`), klik na čísla a šipku otevře **detail**
  (`openDetail`). Detail je jedna souvislá mřížka od nejstaršího dne
  (createdAt / nejstarší log) po dnešek, bez měsíčních nadpisů, a má tlačítko
  Upravit.
- **Godzilla (hvězdičková skupina):** habit může mít `star:true` (☆/★ v sheetu,
  jen pro Habits). Existující habit uloží hvězdičku hned, bez tlačítka Uložit.
  V Přehledu je pak úplně nahoře **odvozený blok „Godzilla“**
  (`buildGodzillaBlock`, barva `--god`). Má 90 čtverečků a den je modrý,
  když **žádný** ohvězdičkovaný habit nemá ten den `missed`. Nechecknuté dny
  bod nekazí, takže prázdná minulost je modrá (vědomý trade-off). **Nic se
  neukládá**, počítá se za běhu. Buňky nejsou klikací. ★ je vidět v hlavičce
  Přehledu i na Check kartě.
- Smazání habitu (po potvrzení) smaže i jeho celý log.

### 3b. Free (`sections[1]`)

Druhá instance stejné sekce (tentýž kód přes index, **tentýž sheet**
`openSheet(curSection-2,…)`). Navíc má **štítky, délku, filtr a hledání**.
**Backlogy Free NEMÁ.** Kdysi existovaly, ale v kódu nejsou ani v historii
gitu, a `normalize()` pole `backlog` zahodí.

- **Habit:** `priority` se v UI jmenuje **„Zábava“** (badge `Z{n}` místo `P{n}`).
  K tomu `tags` a `durationMin/Max` (indexy do `DURATION_TICKS`). Nový habit má
  default Zábavu 5 a délku 0–30min. Změny štítků a délky existujícího habitu se
  **ukládají hned** (`persistHabitEditIfExisting`).
- **`TAG_GROUPS`:** Plánování[Plan,Spon,We] · Prostředí[Uvnitř,Air] ·
  Společnost[Sám,Kámo,Dav] · Energie[Klid,Pohyb,Int,Sport] ·
  Mysl[Intel,Fok,Výplach] · Smysl[Smysl,Bzz] ·
  Vzdálenost[Dom,Out,Okl,Ph,100,ČR,EU,O]. Každá skupina má vlastní barvu.
  Mrtvé mapování: „Sousedi“ se přejmenuje na „Sou“, které v žádné skupině není,
  takže se zahodí.
- **Search a filtr jsou na DVOU místech:** Check panel (`freeSearch` /
  `freeFilterBar`) i Přehled (`freeSearchOv` / `freeFilterBarOv`). Oba jedou přes
  jeden `state.freeFilter`, takže jsou vždy v sync. `renderFreeFilterBar` staví
  všechny `.ffbar`. Pozor: search input dědí globální
  `input[type=text]{margin-bottom:12px}`, nulováno přes
  `.free-search-row .free-search`.
- **Sémantika filtru (`matchesFreeFilter`):**
  - Štítky: **OR uvnitř skupiny, AND mezi skupinami**.
  - Zábava: `priority >= funMin`.
  - Čas: překryv intervalů (16 = ∞). Habit s délkou max 0 („0“) nesedí nikdy.
  - Hledání: všechna slova musí sedět.
  - Odznak počítá štítky, zúžený čas a funMin>1, hledání ne.
  - Reset vynuluje všechno **kromě hledání**.
  - Klik mimo filtr ho sbalí. Dvojitý slider času umí tap-to-nearest-thumb.
- **Fuzzy hledání** (sdílené s Litániemi): odstraní diakritiku (NFD) a převede
  na malá písmena, prohledává název a štítky. Nejdřív zkusí substring, pak
  Levenshtein na tokeny (max 0 editací pro ≤3 znaky, 1 pro 4–6, 2 pro ≥7).
- **„+“ vlevo od `nn/90` v Přehledu Free** (`.tofut` → `addHabitToFuture`):
  pošle stejnojmenný úkol bez data do Tasks / **Chill** s prioritou 1. Slouží k
  ranním plánům „co budu večer dělat“. Nehotový duplikát v Chill jen toastne.

### 3c. Progress (`state.tasks`) — vážený checklist, pořadí místo priorit

- **Jen jedna plocha:** seznam projektů. Čtverečková plocha je zrušená
  (30. 9. 2026).
- **Pořadí = pořadí v poli.** V editaci projektu je slider **Pořadí** (1..N mezi
  nehotovými, vytlačovací insert `reorderTaskInState`). U hotového projektu je
  skrytý. Nový projekt jde na konec. `sortedTasks()` vrací nehotové v pořadí
  pole, za nimi hotové podle completedAt DESC. Používá ho seznam, FAB i picker
  projektů v Tasks.
- **Krok** má v editoru tři ovládací prvky:
  - **N:** náročnost (weight) 1–10.
  - **#:** pořadí. Slider ukazuje cílovou pozici a řádek přesune až na
    `change`, protože přeskládat DOM pod prstem by na mobilu utrhlo drag.
  - **„Pevná hodnota“:** volitelný počet čtverečků (`cells`) 1–90.
  `renumberSubRows` se volá po každé změně. Uložení převezme pořadí z DOMu.
- **Progres** se počítá jako hotová váha / celková váha (`taskProgress`).
  `cellCounts` rozdělí `GRID=90` metodou největších zbytků: pevné `cells`
  drží, volné kroky dostanou poměr podle váhy (aspoň 1, když je místo). Dnes
  se to používá **jen jako flex poměry mini proužku** na placce. Hotové kroky
  jsou v proužku vlevo (`displayOrder`).
- **Placka projektu:** 14px serif název, 🍹 při 100 % a šedé **„+“**, které
  otevře editaci s novým fokusnutým krokem (`openTaskSheet(id,true)`).
  Rozbalená placka ukazuje kroky (nehotové v pořadí pole, hotové dole, vpravo
  váha) a odkaz „Upravit projekt“. Hotový projekt nemá proužek a název má
  `--dim`. Přidáním kroku se „odhotoví“ (`updateTaskCompletedAt`).
- **`addSubRowFocused(container)`** je obecný helper „přidat krok + kurzor +
  klávesnice“. Používej ho u každého „+ krok“ tlačítka.
- Odškrtnutí kroku **nepřidává** nic do `daily` a **nespouští** piňu.
- **Úkol z Tasks → krok projektu:** viz 4f. Je to jednosměrné, z úkolu zbyde
  jen název.

---

## 4. TASKS — kalendářní úkolová sekce (idx 1)

### 4a. Systémové backlogy: role (stav od 9. 10. 2026)

Úkol s `date` žije v **kalendáři**, úkol s `date=null` žije ve svém domovském
**backlogu**. Kalendářní úkol si domov nese dál (je vidět, až ztratí datum).

| backlog | role (jak s tím Bob pracuje) | technicky |
|---|---|---|
| **Chill** | Večerní radosti: nápady z Free přes „+“. Top 5 je vidět **nad kalendářem**, takže je má Bob pořád na očích. | Náhled na Check stránce. **První rychlá volba.** ✓ na nedatovaném ne-repeat úkolu ho **smaže** (do Done se nearchivuje). Fallback všude (neplatný backlog → Chill). |
| **Wip** | „Work in process“: rozdělané věci. Bob jim dává datum nebo opakování, takže žijou v kalendáři. | **Default backlog nového úkolu** (FAB i kalendář). Schválně **není v rychlých volbách**, jen v dropdownu. |
| **Done** | Archiv splněných. | ✓ ne-repeat úkolu → `backlog="Done"`, `date=null`, `done=true`. Jen v dropdownu, v sheetu se vybrat nedá. |
| **Mix** | **Agregátor priorit:** co hoří napříč vlastními backlogy. | Virtuální, nic v něm nebydlí. Viz 4b. **Druhá rychlá volba.** |

Historie názvů: **1. 10. 2026** se přejmenovaly systémové backlogy (starý
Wip → **Chill**, starý Future → **Wip**, migrace `blV<2`). **9. 10. 2026** přibyl
**Mix** (`blV<3`: případný vlastní backlog „Mix“ se přejmenuje na „Mix 2“).
**Future už neexistuje.**

Systémové backlogy (Chill/Done/Wip/Mix) nejdou smazat ani přejmenovat
(`BACKLOG_SCOPES.tasks.protectedNames`). V pickeru jsou nahoře a tučně v pořadí
`SYS_BL = ["Done","Wip","Chill","Mix"]`.

### 4b. Mix — agregátor (`backlogTasks("Mix")`)

- Ukazuje **nedatované** úkoly s `priority >= state.mixMinPrio` ze všech
  backlogů **kromě** `MIX_EXCLUDE = ["Chill","Done","Wip","Mix"]`. Řazení je
  priorita DESC, pak createdAt ASC.
- **Slider „Priorita od“** (`#mixPrioRow` / `#mixPrio`) je hned pod roletou a
  vidět jen, když je Mix ve výběru. Rozsah 1–10, **default 10** (jen desítky).
  Bob si ho sníží, uloží se do `state.mixMinPrio` a překreslí se při `oninput`.
- **Nejde do něj vkládat:**
  - není v pickeru sheetu (`fillBacklogPicker` vynechá Done a Mix),
  - skupina Mix nemá „+“,
  - když default backlog sheetu vyjde Mix, nahradí se Wip,
  - `normalize()` přesune případný úkol s `backlog:"Mix"` do Chill.
- Řádky v Mixu mají vpravo šedý štítek **domovského backlogu** (`.thome`,
  `makeUtaskRow(t, ctx, showHome)`).
- **Multi-výběr bez duplicit** (`seen` v `renderBlogView`): úkol se ukáže jen
  jednou, v **první skupině podle abecedy**. Vlastní backlog, který je abecedně
  před „Mix“, si proto úkol vezme sám a Mix ho neukáže.
- ✓ v Mixu se chová podle domova úkolu, takže úkol jde do Done.

### 4c. Check stránka (`renderCheckDefault`)

Jedna scroll-plocha, shora dolů:
1. **Lišta backlogů (`.wipbar`):**
   - Vlevo je **mini dropdown**, jen šipka (`.wip-sel--mini`, `data-notext`).
     Otevře picker s multi-výběrem.
   - Vedle je **vodorovně scrollovatelný řádek rychlých voleb** (`#blogQuickRow`,
     `renderBlogQuick()`): **CHILL · MIX ·** a pak prefixy.
   - **Prefix** = první dvě písmena názvu vlastního backlogu, když jsou obě
     velká (`isUpperLetter`). Systémové backlogy se nepočítají.
   - Prefixy se řadí podle **`state.backlogOrder`** (skupina bere nejnižší
     pořadí svých backlogů). Prefixy bez pořadí jdou na konec, shoda pak
     abecedně.
   - Klik na prefix otevře **multi-výběr všech backlogů s tímto prefixem**
     (case-sensitive, `applyTaskBacklogMulti`).
   - Rychlé volby jsou jen na hlavní stránce (v backlog view by se vedle
     „‹ Zpět“ na mobilu nevešly).
2. **Náhled Chill:** nedatované úkoly z Chill, priorita DESC a createdAt ASC,
   **max 5** (`WIP_PREVIEW_MIN = 5`). Odkaz „+ N dalších“ **není**, zbytek je
   vidět v backlog view. Řádky mají jen ✓.
3. **Kalendář:** sloupec dnů s hlavičkami („Dnes · Po 30. čvn“, „Zítra · …“,
   „Včera · …“, „… (N dní zpět)“). Úkol = řádek. Ukazuje úkoly ze **všech**
   backlogů. **Propadlé úkoly zůstávají pod svým původním datem** (žádný
   rollup), proto se tu ukazují i hlavičky „Včera“ a „N dní zpět“.

**Backlog view** (`openBlogView` / `applyTaskBacklogMulti` / `renderBlogView`)
je plnoobrazovkový výpis. Otevře se **vždycky odscrollovaný nahoru**
(`blogScrollTop`, jen při otevření). Ukazuje **jen nedatované** úkoly vybraných
backlogů (Mix podle 4b). Když je vybraných víc, člení se skupinovými
hlavičkami (abecedně), každá s „+“ (kromě Mixu), které přidá úkol do dané
skupiny. Tlačítko „‹ Zpět“ vrací na default obrazovku.

**Picker backlogů (`fselOpen`):**
- Nativní `<select>` zůstává skrytý jako zdroj pravdy, viditelné je tlačítko
  `.fsel-trigger`, které otevře vlastní sheet.
- **Multi-výběr:** checkboxy a dole Reset · Zrušit · Potvrdit. Reset jen odškrtá
  a sheet nechá otevřený. Výběr se nepamatuje (otevře se prázdný).
- **Klik na název backlogu** (ne na checkbox) otevře rovnou jeho backlog view.
  Volba Chill v hlavním dropdownu nedělá nic („jsi doma“).
- U vlastních backlogů jsou na řádku dvě ikony:
  - **✎** otevře editaci backlogu (`#blEditScrim`): název + „Pořadí v rychlých
    volbách“ (celé číslo ≥1, prázdné = na konec). Kontroluje duplicitu.
    Přejmenování (`renameTaskBacklog`) přepíše název na úkolech, ve výběru, v
    `backlogOrder` i v otevřeném sheetu.
  - **✕** smaže backlog (`deleteTaskBacklog`). Smaže jeho **nedatované** úkoly
    a datované přesune do Chill.

### 4d. Akce na úkolu

- **V kalendáři** (ctx `"cal"`): `✓` splnit · `→` zítřek · `»` týden · `✕`
  nesplněno.
- **V backlogu a náhledu** (ctx `"backlog"`): jen `✓`. **V Done není nic**,
  protože Done je log a nic se v něm neodškrtává.
- **Klik na název** = editace, a to všude (jednotné gesto).

Co jednotlivé akce dělají:
- **✓ (`utComplete`):** +1 done k **dnešku** v `daily`, piňa 🍹 a hlas (viz 7).
  - Repeat → respawn (viz 4f).
  - Nedatovaný ne-repeat v **Chill** → úkol se **smaže**.
  - Jinak → Done (`date=null`, `done=true`).
- **✕ (`utMiss`):** +1 missed k dnešku, přesune úkol na zítřek. Platí pro
  repeat i ne-repeat a je to **jediný zdroj červené**.
- **→ (`utMove`):** datum úkolu + 1 den, i když je datum v minulosti (bez clampu).
  Bez penalizace.
- **» (`utMove`):** datum + 7 dní. Propadlý úkol se posune o **týden od
  dneška** (clamp na dnešek). Do 11. 10. 2026 tu byl bug, kvůli kterému `»`
  u propadlého úkolu nedělal nic.

### 4e. Prokrastinace a červená — FILOZOFIE (a zrušený rollup)

**Červená vzniká JEN z dobrovolného `✕`:** Bobova ruka, „mohl jsem, zflákl
jsem“. Ruční přesun ani repeat červenou nedělají.

**Auto-rollup byl ZRUŠEN 2. 7. 2026** (commit `5149489`). Dřív se propadlý
ne-repeat úkol přesouval na dnešek s červenou, i když byl hotový, jen
neodškrtnutý. Teď **propadlé úkoly prostě zůstávají pod svým datem**, dokud je
Bob neodškrtne nebo nepřesune. `rollupUtasks` ani `lastRollup` v kódu
neexistují.

Přehled úkolů (čtverečky z `daily`) je stejně zrušený (4g), takže dnes je
červená jen interní číslo.

### 4f. Repeat respawn motor (`nextRepeatDate`, `addMonths`)

**Respawn nastává VÝHRADNĚ po `✓`.** Existuje jedna živá instance, která se
NIKDY neklonuje. Přesun ji jen posouvá. Tohle je tvrdé pravidlo.

- **Prázdný bouncer** → klouzavý rytmus: perioda se počítá **ode dne splnění**.
- **Vyplněný bouncer** → pevná řada: respawn je **nejbližší BUDOUCÍ datum v
  řadě vůči dni splnění**, a to se stane novým bouncerem.
  - weekly/biweekly: bouncer `{dow}`. Řada se kotví na **datu splňované
    instance** (`t.date`), srovná se na cílový den v týdnu a posouvá se po
    7/14 dnech, dokud není striktně po dni splnění.
    ⚠️ **Na kotvu nesahej.** Dřív se počítalo „nejbližší dow po dni splnění“ a
    biweekly pak vracelo vždycky příští týden. Parita dvoutýdenní řady stojí na
    kotvení na `t.date`.
  - monthly+: bouncer `{date}`, posun o 1/3/6/12 měsíců, dokud není po dni
    splnění. Přetečení 31. na kratší měsíc ošetřuje `addMonths`.
- **Pozdní splnění může spolknout přeskočené cykly**, tiše a bez červené.
- Reálné use-casy, které logiku určily: *Blog (weekly, středa)*, kde opoždění
  se má dohnat hned příští středu. *Záloha na daň (halfyear, 15. 6. a 15. 12.)*,
  kde řada existuje nezávisle na Bobovi. Obojí je stejné pravidlo, jen jiná
  hustota řady.

**Známé chování, které Bob nechal být:** `utComplete` počítá respawn vždycky
od **dneška**. Když zakotvený úkol naplánovaný na zítřek splníš dnes, zůstane
na zítřku a jde splnit znovu. Oprava by byla `max(dnešek, t.date)`, vědomě
odloženo.

### 4g. Sheet úkolu (`openUtSheet` / `utSave`)

**Pole sheetu:**
- **Název.**
- **Priorita:** slider 1–10, default 1. Barevný proužek vlevo dělá `prioColor`
  (hue 210 chladná → 0 sytě červená).
- **Termín:** segmenty Dnes/Zítra a vždy viditelný datepicker.
  - Bez aktivního segmentu a s prázdným datem jde úkol do backlogu.
  - Klik na aktivní segment ho vypne. Vyplnění data zhasne segmenty.
- **Opakování:** 8 segmentů `0/d/t/tt/m/3m/6m/Y`.
- **Kotva (bouncer):** zašedlá u none/daily. U weekly/biweekly se volí den v
  týdnu, druhým klikem na stejný den se kotva sundá. U monthly+ se volí datum.
- **Dva pickery vedle sebe:** backlog (bez Mix; Done jen u úkolu, který v Done
  už je, ať ho editace nevyhodí jinam) a „→ projekt…“.
  Tlačítko **„+ backlog“** (prompt) je v řádku tlačítek.

**Výchozí backlog** nového úkolu: `presetBacklog` (z „+“ u skupiny), jinak v
otevřeném backlog view aktuální `blogCurrent`, jinak **Wip**. Mix i Done se
vždycky nahradí Wip.

**Repeat bez data:**
- daily → dnešek.
- weekly/biweekly s kotvou → nejbližší ten den (včetně dneška).
- monthly+ → datum kotvy.
- weekly bez kotvy → úkol zůstane v backlogu.

**„→ projekt…“** (`fillProjectPicker`, projekty z `sortedTasks`): Uložit
**nezaloží ani neuloží úkol**. Vepíše jeho název jako **nový krok** projektu
(weight 5, bez pevných čtverečků) a úkol smaže z `utasks`. Datum, priorita,
repeat i backlog se zahodí a **cesta zpět není**. Zapisuje se tam i mrtvé pole
`prio:5`, které `normalize()` při načtení zahodí.

### 4h. Přehled úkolů — ZRUŠENO

Čtverečková plocha Tasks (Dny/Měsíce) je odstraněná (30. 9. 2026), Bob ji
nepoužíval. `state.daily` se dál plní, jen se nezobrazuje. ⚙ „Vymazat
historii úkolů“ ho maže.

---

## 5. MOVE — pět kolečkových ploch (idx 4)

Všechny plochy používají **stejnou mřížku**:
- Řádek = týden, Po vlevo a Ne vpravo, **nejnovější týden nahoře**.
- Klik na minulý nebo dnešní den otevře dialog. Budoucnost klikací není.
- SVG kolečka mají `viewBox 0 0 100 100` a `width:100%` buňky, takže se škálují
  s obrazovkou a tloušťka prstence zůstává konstantní.

Ruční plochy (Nálada, Spánek, Běh, Otužování) kreslí mřížku **vždycky, aspoň
8 týdnů zpět**, aby bylo kam kliknout. Garmin kreslí od prvního týdne s daty
a bez dat ukáže jen empty-state.

⚠️ **Past v `moveSemi(cx,cy,r,side,ratio)`:** `side +1` kreslí **LEVOU**
půlku, `side −1` **PRAVOU**. Je to obráceně, než bys čekal. Oblouk jde od
spodního bodu k hornímu a `sweep=1` vede přes levou stranu. Ověřeno přes
`getBBox`, nesahej na to bez změření.

| view | plocha | `hTitle` | data |
|---|---|---|---|
| 0 | **Nálada** | Nálada | ruční `state.vibe` |
| 1 | **Spánek** | Spánek | ruční `state.sleep` |
| 2 | **Garmin** | Move | sync `state.move` + ruční zámky |
| 3 | **Běh** | Běh | sync `state.run.days` + ruční `state.run.manual` |
| 4 | **Otužování** | Otužování | ruční `state.cold` |

### 5a. Garmin plocha (`renderMove` / `moveCell`, view 2)

Účel: „donutit se hýbat“ přes rychlou vizuální gestalt. Jedno mrknutí a víš,
jak plnej byl den.

- **Velikost kruhu = kcal**, poloměr **lineárně** ke stropu (bez odmocniny).
- **Číslo uvnitř** = doba trvání v minutách. **Pod kolečkem** jsou kcal (u
  zamčeného dne kurzívou).
- **PRAVÝ půlprstenec (zelený `--done`) = vzdálenost** ke stropu p90.
- **LEVÝ půlprstenec (červený `--missed`) = tep**, škála
  `(hr−hrBase)/(hrCap−hrBase)` = 55→130, clamp 0..1. Baseline je tam proto, aby
  prázdná půlka znamenala klid, ne „málo dat“.
- Prázdný den = jen tečka.

**Stropy** (`state.move.caps`, default kcal 600, dist 5000, hr 55→130):
`recalcMoveCaps()` počítá p90 (jen hodnoty >0, lineární interpolace) z
**efektivních** dnů. Běží **při importu, uložení zámku a odemčení**, ne při
startu, takže vizualizace roste s tebou. Tep je napevno. kJ se zavrhly
(korelují s tep×čas), jede se na kcal.

**Ruční zámek dne (override odolný syncu):**
- Klik na kolečko (i prázdné) otevře dialog „Upravit dd.mm.yyyy“ se 4 údaji:
  kcal, trvání, vzdálenost, tep (prázdný tep = `null`).
- „Uložit natvrdo“ zapíše hodnoty do `state.move.overrides[datum]`.
- Efektivní hodnota dne je `overrides ?? days` (`moveEffDays()`), import sahá
  jen do `days`, takže editace přežije další sync.
- „Odemknout (vrátit k syncu)“ override smaže.
- Zamčený den poznáš podle **kcal kurzívou**. Zámek na samé nuly ale skryje
  aktivitu jako prázdnou tečku a zámek pak vidět není.

### 5b. Datový most: appka ↔ Garmin

Appka je statická PWA a **s Garminem nikdy nemluví přímo** (CORS/OAuth).
Tok je jednosměrný:

```
Garmin Connect
   │  local/garmin/build_data.py — token login, 90 dní, agregace po dnech
   ▼
move_data.json + run_data.json (KOŘEN repa) → git commit + push (jen při změně)
   ▼
GitHub Pages → appka je při startu tiše fetchne (fetchMoveData + fetchRunData)
```

- `move_data.json` je pole `{date,kcal,durMin,distM,hr,count}` (`count` se
  ignoruje). `run_data.json` je pole `{date,distM,durSec,movSec,hr,vo2}`
  (aktivity s `typeKey` obsahujícím „running“, sečtené po dnech, `hr` vážený
  délkou).
- **Import mergeuje po datech** (`importMoveData` / `importRunData`): datum ze
  souboru přepíše uložený den a dny, které v souboru chybí, **zůstávají
  navždy**. Historie tak přežije 90denní okno. Aktivita smazaná v Garminu ale
  z appky nezmizí (na to je zámek).
- Při startu se importuje jen neprázdné pole. Offline nebo chyba = ticho, jede
  se z localStorage.
- **⚙ „Znovu načíst Move data z GitHubu“** (`#btnRefreshMove`) stáhne
  `move_data.json` s `?t=` a `cache:no-store` a ohlásí výsledek toastem. Po
  úspěchu tiše stáhne i `run_data.json` (bez cache-bustu) a zavře ⚙. **Ruční
  import souboru v UI není.**
- **Agregace dne** (skript): kcal, trvání a vzdálenost se sčítají, tep je
  vážený délkou. Bere **všechny ručně spuštěné aktivity** (chůze, tenis,
  plavání…). Celodenní kroky bez záznamu ne, to je Bobovo vědomé rozhodnutí.

### 5c. Motor syncu — `local/garmin/` (mimo git, v cloudu není)

- **`build_data.py`** zapíše oba json soubory do kořene repa a pak udělá
  commit a push s `pull --rebase --autostash`. Mění jen datové soubory, takže
  s lidským vývojem nekonfliktuje.
- **`.garmintoken/`** drží token (garth). Heslo je potřeba jen poprvé. Token
  vyprší zhruba po roce: ruční běh si pak řekne o heslo, automatický běh jen
  zaloguje a skončí.
- **`sync.log`** je log běhů.
- **Automatika:** Windows Task Scheduler `Habity\GarminSync`, `pythonw.exe
  build_data.py`, **3× denně v 10:00 / 18:00 / 22:00**, `StartWhenAvailable`.
- **Soukromí:** `move_data.json` a `run_data.json` jsou **veřejně čitelné**
  na Pages. Bob to vědomě schválil. **Osobní sekce se takhle NIKDY vystavovat
  nesmí**, Pages nemá autentizaci.

### 5d. Zálohování

⚙ → „Exportovat zálohu (JSON)“ stáhne **`habity-zaloha.json`** s pevným
názvem (přepisuje se a nehromadí v Downloads). Obsahuje **celý `state`**,
tedy i Litánie a Move. Bobův režim: soubor občas houpne do Dropboxu, občas ho
přejmenuje s datem jako milník. **localStorage je jediná ostrá kopie**, proto
zálohovat pravidelně.

**Budoucnost (neřešeno):** sync osobních sekcí desktop↔mobil přes privátní
autentizovanou vrstvu (ne veřejné Pages). Je to velký samostatný projekt.

### 5e. Otužování (`renderCold` / `coldCell`, view 4)

**Čistě ruční zápis** (žádný sync, žádné zámky). Dialog `coldScrim` má pět polí.
`tw` a `min` jsou povinné, zbytek smí zůstat prázdný. Existující zápis jde
smazat. Dialog ukazuje „Výkon X ze stropu Y“.

| Prvek | Veličina | Škála |
|---|---|---|
| levý půlprstenec (`--water` #5B87A8) | teplota vody | `(20 − tw)/20` |
| pravý půlprstenec (`--done`) | teplota vzduchu | `(40 − ta)/60` |
| průměr kruhu (`--accent`) | **výkon** | `sqrt(perf/cap)` |
| číslo uvnitř | minuty (desetinná čárka) | — |
| text pod kruhem | vítr (1 desetinné) / průtok (0 desetinných) | — |

- **Oba prstence jedou REVERZNĚ:** plný prstenec = ledová voda nebo −20 °C.
- **Výkon = `min / max(1, tw)`.** Je to pravidlo „minuta za stupeň“, voda ≤1 °C
  se počítá jako 1 °C. **Průtok do vzorce schválně nevstupuje.** Průtok řeky
  (m³/s, ČHMÚ) není rychlost proudu v místě ponoru a chlad je už zapsaný v `tw`.
- **Strop (`coldCap`) drží na `COLD_PERF_BASE = 7.5`** (Bobův „hodně slušný
  den“ = 15 min ve 2 °C). Povolí se **jen NAHORU**, na p90 zápisů od 4 zápisů
  výš. Čisté klouzavé p90 by zlepšení mezi zimami smazalo.
- **Odmocnina u průměru:** oko čte plochu (`~r²`), ne poloměr.
- `normalize` přijímá `tw` −5..45 a `ta` −50..55.

### 5f. Běh (`renderRun` / `runCell`, view 3)

Bobův návrat k běhu (srpen 2026, Po-St-Pá ~2×800 m okolo plavání). Data jsou
ze syncu (`run_data.json`) a k nim **ruční pocity** (`runScrim`: těžkost a
achilovky 0–10, aspoň jedno pole). Dialog ukazuje Garmin km, tempo a tep.

**Filozofie indexu (KOUČOVSKÁ, nesahej na ni bez rozmyslu):** index nesmí
odměňovat tempo. Bob má achilovky náchylné k zánětu a psychologický odpor, a
honění tempa by znamenalo konec. Odměňuje se docházka a klesající tep na fixní
trase.

| Prvek | Veličina | Jak |
|---|---|---|
| plocha kruhu | **Forma** | EF = `(distM/movSec)/hr`, klouzavý průměr 5 běhů ÷ baseline (průměr prvních 6 EF-běhů, dokud jich je míň, tak všech). Okno `RUN_FORMA_LO=0.85` (prázdno) až `RUN_FORMA_HI=1.25` (plný = +25 %), **lineárně na poloměr**. Strop se povolí nahoru na p90 od ≥4 hodnot. Před prvním EF-dnem je forma 1. |
| číslo uvnitř | průměrný tep dne („–“ bez tepu) | má KLESAT |
| levý půlprstenec (`--done`) | konzistence | běžecké dny za 28 dní ÷ 12 |
| pravý půlprstenec | **Brzda** | acute:chronic z `durSec`: dny 0–6 ÷ (dny 7–27 / 3). Půlka = 1.0, plný = 2.0, od 1.4 červená. Akutní týden se do reference nepočítá, jinak by restart z nuly byl věčně červený. |
| čísla pod kruhem | těžkost / achilovky | ruční (`–/–` když chybí; ukazuje se i u neběžeckého dne) |

**Běžecký den** = `distM>0` nebo `durSec>0`. Den bez tepu nebo vzdálenosti se
počítá do konzistence a Brzdy, ne do EF. `#runStats` nad mřížkou ukazuje:
- „EF baseline: n/6 běhů“, po 6 bězích pak „EF **N** (start=100)“,
- km celkem,
- poslední VO₂max.

### 5g. Nálada (`renderVibe` / `vibeCell`, view 0, default landing Move)

Ruční zápis (`vibeScrim`, v UI všech 5 povinných, škála 1–5).

| Prvek | Veličina | Škála |
|---|---|---|
| průměr kruhu (`--accent`) | nálada `mood` | `(mood−1)/4` |
| číslo uvnitř | `mood` | — |
| levý půlprstenec (`--done`) | kontakt `soc` | `soc/5` (1 samota → 5 hodně lidí) |
| pravý půlprstenec (`--missed`) | aktivita `act` | `act/5` (1 línej → 5 frmol) |
| čísla pod kruhem | tělo / hlava | **reverzně**: tělo 1 forma → 5 na hovno, hlava 1 klid → 5 ruminace turbo |

Prstence se plní po pětinách od `v/5`. Kdyby se plnily od `(v−1)/4`, nula by
splývala s nevyplněným dnem.

### 5h. Spánek (`renderSleep` / `sleepCell`, view 1)

Ruční zápis (`sleepScrim`), povinná je **jen délka**.

| Prvek | Veličina | Škála |
|---|---|---|
| průměr kruhu | délka `dur` (h, po 0.5) | 4 h → 8 h, clamp (číslo ukazuje reálnou hodnotu) |
| barva kruhu | délka | `--done` ≥ 7 h, jinak `--missed` |
| levý půlprstenec (`--accent`) | usnutí `bed` "HHMM" | 23:00 plný → 04:00 prázdný. Hodiny 14–22 = „před 23“ (plný), 5–13 = „po 4“ (prázdný) |
| pravý půlprstenec (`--god`) | pocit `feel` | `feel/10` |
| čísla pod kruhem | usínání (min) / sny (počet) | — |

- Prstence schválně nejsou zeleno-červené, protože barvu nese disk.
- **Rozsahy:** délka 0.5–24 (zaokrouhlí se na 0.5), pocit 1–10, usínání
  0–600 min, sny 0–99. Mimo rozsah vyskočí konkrétní toast.
- **Usnutí** se zadává jako `2320`, `23:20`, `0040` nebo `040` (input má
  `maxlength=5`, nečíslice se zahodí a normalizuje se na "HHMM").

---

## 6. LITÁNIE — kolekce vět s tagy (idx 5, v navu první)

Knihovna podpůrných vět, přerámování, manter a vděčností. Vznikla z Bobova
Evernotu (10 let poznámek, deduplikace a AI tagging, ~1250 vět). Druhá plocha
jsou **Otázky**, poznávací otázky na cestu pro dva (980 z xlsx). Účel je dvojí:
**čistit** (vyhazovat balast) a **tvořit** (přidávat nové).

### 6a. Data — SOUKROMÁ, nikdy online (DŮLEŽITÉ)

> ⚠️ **Litánie a otázky jsou hodně osobní a NESMÍ jít na veřejný GitHub/Pages.**
> Zdrojové soubory (`local/litanie.json`, `local/otazky/…`) se **nikdy
> necommitujou**. Data žijí jen v localStorage a v JSON exportu.
>
> Proč ne Pages fetch: Pages je veřejné bez autentizace. Strategie „pushnout,
> jednou stáhnout a smazat“ je iluze bezpečí (zůstane historie gitu, forky,
> cache, archivy).

**Kolekce (`LIT_COLS`):** jeden motor a dvě kolekce. Každá je objekt
`{key, base, overlay, nw, filter, temata, casti, skupiny, seedSkupiny,
seedTemata?, prioMax, prioDef, idPrefix, dom:{…}, lab:{…}}`. Všechny funkce
motoru berou `c` jako první argument (`litEffective(c)`, `renderLitList(c)`,
`openLitSheet(c,id)`, `litToggleFav(c,id)`, `litSetAktivni(c,id,on)`…).
`refreshLit()` / `litLoadFromState()` bez argumentu zpracují všechny kolekce.

| klíč | plocha | priorita | id nových | tagy |
|---|---|---|---|---|
| `lit` | **Litánie** (view 0) | 1–5, default 2 | `ul…` | temata (modré, vždy včetně **„terapie“**) · casti (červené, IFS figury) · skupiny (zelené, seed Otázky/Litánie/Udělat/Výzvy) |
| `ot` | **Otázky** (view 1) | **1–10**, default 6 (skóre z xlsx) | `uq…` | temata = 11 kategorií z xlsx · casti/skupiny prázdné · bez seedů |

**Klíče ve `state` na kolekci** (`litColBlank` / `litColNormalize`):
- `<k>Base`: základ z importu.
- `<k>Temata/Casti/Skupiny`: definice tagů.
- `<k>Overlay[id]`: přepisy základních vět. Klíč je **string** id. Uložení
  editace zapíše všechna pole sheetu, toggle ★ a smazání jen jedno pole.
- `<k>New`: vlastní věty.
- `<k>Filter`: persistovaný filtr včetně textu hledání (max 100 znaků).

`normalize` ořízne text overlay a new na 4000 znaků. Runtime cache (`c.BASE`,
`c.TEMATA`, `c.CASTI`, `c.SKUPINY`, `c.LOADED`) plní `litLoadFromState(c)`.

**Migrace a seedy tagů:**
- **`LIT_TAG_RENAME`** je jednorázové idempotentní přejmenování přes
  `litRenArr` (např. Postoje → Litánie, Don-Omnipotent → Don).
- **`LIT_SKUPINY_SEED`** se aplikuje jen v prázdném stavu nebo při importu bez
  `skupiny`.
- **`LIT_TEMATA_SEED = ["terapie"]`** + `litEnsureTags` (9. 10. 2026):
  „terapie“ se **doplní na konec modrých témat Litánií**, kdykoli chybí (porovnává
  bez ohledu na velikost písmen). Běží v `litColBlank`, každém
  `litColNormalize` i v `litImport`. Otázky ho nemají.

**Import (`litImport(c,d)`):** formát
`{polozky|litanie:[…], temata:[], casti:[], skupiny:[]}`. Naplní base a
definice tagů. Overlay a new drží přes stabilní id. **Nemá UI volajícího**:
obě kolekce jsou nasáté (Litánie dávno, Otázky 14. 9. 2026) a tlačítka jsou
odstraněná. Při potřebě reimportu ho dočasně zavěs. Komentáře v kódu, které
mluví o importu přes ⚙, jsou zastaralé. Jediná cesta dat dovnitř je dnes
**import celé zálohy v ⚙**. Zdroj Otázek: `local/otazky/Otazky.xlsx` →
`xlsx2json.py` → `otazky.json` (id `q0001…`).

**Slití (`litEffective(c)`):** základ + overlay (přebije) + new (flag `_new`).

### 6b. UI a interakce

Každá plocha má vlastní search, ☆ rychlý filtr, filtr bar, počítadlo a
seznam (dom id v `LIT_COLS[k].dom`). Sheet `#litSheet` je sdílený: při otevření
nastaví placeholder, text tlačítka pro smazání a max slideru podle kolekce.

- **Seznam:** karta `P{priorita} · text · ☆/★`. Klik na text = editace, klik na
  hvězdu = oblíbená. Na seznam je **jeden** delegovaný click handler.
  - Řazení: oblíbené → priorita DESC → id ASC.
  - Strop 5000 karet (`CAP`), nad ním poznámka „…a dalších N“.
  - Počítadlo: „zobrazeno z aktivních · N vyházeno“.
- **Filtr (`renderLitFilterBar`):** **tři skupiny chipů bez popisků**:
  témata **modré** (`#6E8FAE`), části **červené** (`#BC5B43`), skupiny
  **zelené** (`#74955E`). Prázdná skupina se schová. Dál slider priorita-min
  (1..prioMax), toggle „Bez tagů“, Reset a sbalení (i klikem mimo).
  - **Sémantika: OR uvnitř skupiny, AND mezi skupinami**, plus ★, bez tagů,
    priorita a fuzzy hledání.
  - **Use-case:** v Litániích „terapie“ (modrá) + „Otázky“ (zelená) =
    terapeutické otázky. Pozor, tohle „Otázky“ je zelený tag uvnitř Litánií,
    ne plocha Otázky.
  - Odznak počítá tagy, prioMin>1, ★ a bez tagů, hledání ne. Reset hledání
    nemaže.
- **Rychlá ☆** (`#litFavQuick`) sedí vedle searche, ne ve filtr baru, aby měl
  filtr plnou šířku.
- **Sheet:** auto-grow `<textarea>` (bez `maxlength`, výška do 40vh), slider
  priority 1..prioMax, toggle ★ a tři řádky chipů (prázdné se skryjí). Prázdný
  text nejde uložit. Nová věta jde do `state[c.nw]`, úprava základní do
  `state[c.overlay][id]`, úprava `_new` zmutuje objekt.
- **Mazání = `aktivni:false`** (`litSetAktivni`), ne fyzické smazání. Po
  smazání vyskočí **undo lišta** (`#undoBar`) na 6 s s tlačítkem „Vrátit“.
  Když overlay držel jen `aktivni`, smaže se celý. Undo slot je **jeden
  společný**, takže druhé smazání do 6 s přepíše první. **Koš ani obnova není**:
  vyházené jdou vrátit jen ručně v JSONu.
- Flag `videno` (👁 a filtr „Neviděné“) sloužil k jednorázovému pročištění.
  Je **odstraněný** (9/2026).

### 6c. Rotace „piňa dne“ — VĚDOMĚ NEUDĚLÁNO

Denní vážená rotace (věta dne podle priority a ★) byla pro první verzi
odmítnutá: „na špinavý databázi to stejně servíruje blbosti“. Až bude databáze
pročištěná, je to kandidát na druhé kolo. Data (`priorita`, `oblibena`) jsou
připravená.

---

## 7. Vizuál, styl a odměna

Tmavé „kokosové“ téma. CSS proměnné v `:root`:

| proměnná | hodnota | použití |
|---|---|---|
| `--bg` | `#15120E` | |
| `--surface` | `#201B15` | |
| `--surface2` | `#2A241C` | |
| `--border` | `#3A332A` | |
| `--text` | `#EDE6D8` | |
| `--dim` | `#9A9081` | |
| `--empty` | `#34302A` | |
| `--empty2` | `#46403A` | alternace proužků |
| `--done` | `#74955E` | |
| `--done2` | `#8FB179` | alternace proužků |
| `--missed` | `#BC5B43` | |
| `--accent` | `#D9A441` | |
| `--god` | `#5C8DC9` | Godzilla, pocit spánku |
| `--water` | `#5B87A8` | voda v Otužování |

Fonty: `--serif` (Georgia…) na názvy, `--sans` (system) na UI, `--mono` na
metadata a čísla. Modré jsou jen `--god`, `--water` a modré tagy Litánií.
Jinak je paleta zemitá. Bob má rád hustotu (malé paddingy, malé písmo, názvy
14px serif).

**🍹 Piňa odměna** (`pina()`): po ✓ **úkolu v Tasks** vyskočí velké 🍹
uprostřed (`@keyframes pinapop`, **1.9 s**). Restart animace je ošetřený pro
rychlé klikání. Habits ✓ ani kroky Progressu piňu nespouští.

**🗣️ Hlasová odměna** (`speakPina`) jde s každou piňou. Systémový TTS řekne
jednu z `PINA_LINES` („Dej si piňu!“, „Šikovnej osel!“, „Král prdele!“,
„Hustý, vole!“, „To je na piňu!“, „Híhá!“). Hlášky se **střídají v rotaci**.
Hlas je cs-CZ, když je k dispozici, rychlost 1.05, a předchozí hláška se před
novou zruší (`cancel()`).

---

## 8. Co je hotové, známé bugy a co Boba čeká

**HOTOVÉ a otestované:**
- Repeat respawn (Node simulace 7 scénářů).
- Migrace v2, `blV` 2 i 3.
- Celá navigace.
- Move a Garmin pipeline (v provozu, 3× denně).
- Otužování (kalibrace na Bobových číslech; zimní sezóna ukáže, jestli strop
  7.5 sedí).
- Běh, Nálada, Spánek.
- Litánie a Otázky.
- Mix agregátor a slider (headless testy, 9. 10. 2026).
- Seed „terapie“.

**Opraveno po auditu 11. 10. 2026:**
- `»` u propadlého úkolu.
- Editace úkolu v Done ho nevyhodí do Chill.
- V Done není ✓ (žádná falešná zelená a piňa).
- Spánek bere `23:20`.
- Reset filtru Litánií zhasne ☆.
- Detail Free píše „zábava“.
- Text empty-state Garminu a zastaralé komentáře v kódu.

**Známé drobnosti, vědomě nechané být:**
1. Klik v detailu habitu nesyncuje Check panel (2). Srovná se při dalším
   renderu.
2. Garmin: zámek dne na samé nuly není vidět (5a).
3. Litánie: když je `<k>Base` prázdný, `LOADED=false` a vlastní nové věty
   (`<k>New`) se nezobrazí (ukazuje se „Načítám…“). Boba to nepálí, base má.

**Reálně neověřené:** dlouhý výpadek syncu nad 90 dní udělá nedoplnitelnou
díru (dá se zvednout okno `DAYS` ve skriptu). Token Garminu vyprší zhruba po
roce.

**Filozofie dalšího vývoje:** „Základ máme, bude to o dolaďování během
úkolování.“ Čekej drobné UX úpravy z reálného provozu, ne velké přestavby.
Nepřeengineeruj dopředu. Bob slíbil, že po dovolené (10/2026) se backlogy
v Tasks spolu „pročistí“.

---

## 9. Tipy pro zásahy do kódu

**Mapa souboru** (orientačně, hledej `grep -n`):
1. `<style>` (`:root` kolem ř. 15).
2. HTML: header s ⚙, pager s 5 tečkami, 6× `<section class="page">` (Progress 1
   panel, Tasks 1, Habits 2, Free 2, Move 5, Litánie 2), nav (~916), sheety a
   modaly.
3. `<script>`:
   1. Konstanty (`TAG_GROUPS`, `LIT_COLS`…), `blankState`, `normalize`,
      `load`/`save` (~1300–1850).
   2. Habits/Free (render, filtr, fuzzy, Godzilla, sheet).
   3. Progress (`sortedTasks`, `cellCounts`, `renderTasksPanel`,
      `openTaskSheet`).
   4. Tasks (`backlogTasks`, `calendarGroups`, `nextRepeatDate`, `utComplete`,
      `makeUtaskRow`, `renderCheckDefault`, `renderBlogView`, `openUtSheet`,
      backlogy, `fselOpen`, `renderBlogQuick`).
   5. Toast, piňa, hlas, ⚙ export/import.
   6. Navigace (`SEC_*`, `applyViewToTrack`, `updateUI`, `attachSwipe`, FAB,
      taby, klávesnice).
   7. Move (`moveSemi`, `moveCell`, `renderMove`, caps, import/fetch, zámky).
   8. Otužování, Nálada, Spánek, Běh.
   9. Litánie (`litLoadFromState`, `litImport`, `litEffective`, filtr,
      `renderLitList`, sheet, undo).
   10. `renderAll` a init.

**Rady pro zásahy:**
- `renderCheck()` je router Tasks (default obrazovka nebo backlog view).
  `renderAll()` volá všechny rendery najednou (všechny plochy jsou pořád
  v DOMu, jen schované).
- **Datové helpery Tasks:** `parseKey`, `key`, `today`, `addDays`, `addMonths`,
  `fmtCalLabel`, `bumpDaily`, `utaskById`, `sortUtasks`, `backlogTasks`,
  `calendarGroups`, `prioColor`, `repeatShort`.
- **Když měníš tvar `move_data.json`,** musí sedět `build_data.py` (výstup),
  `importMoveData` (vstup) a `moveCell` (render).
- **Po každé změně** udělej syntax check přes Node, zkontroluj, že ID
  z `getElementById` existují v HTML, a v cloudu pusť headless test
  (viz 1, Workflow).
- **Nový systémový backlog nebo jeho rename** potřebuje zvednout `blV`, přidat
  migraci v `normalize` a upravit seznamy na těchto místech: `blankState`,
  `normalize` (`bl`), `deleteTaskBacklog`, `BACKLOG_SCOPES.protectedNames`,
  `SYS_BL` v `fselOpen` a `renderBlogQuick`, `MIX_EXCLUDE`, `fillBacklogPicker`.
