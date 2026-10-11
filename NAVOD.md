# Habity — návod pro uživatele

Osobní tracker návyků, úkolů, pohybu a podpůrných vět. Běží v prohlížeči a
**všechna data jsou jen ve tvém zařízení** (localStorage prohlížeče). Nikam se
neposílají, není tu účet ani přihlášení. Zároveň platí: **když smažeš data
prohlížeče, je to pryč.** Proto občas exportuj zálohu (viz konec).

Tip: v mobilním prohlížeči dej „Přidat na plochu“ a chová se pak jako appka.
Offline režim ale nemá.

---

## Jak se v tom pohybuješ

**Dole je 6 tlačítek = 6 sekcí:** Litánie · Progress · Tasks · Habits · Free ·
Move. Appka se otevírá na **Tasks**.

**Švihnutím doleva/doprava** (nebo klikem na tečky nahoře) přepínáš plochy
uvnitř sekce:
- **Habits a Free:** **Check** (dnešek, odškrtávání) a **Přehled** (mřížka
  90 dnů). Otevírají se rovnou na Přehledu.
- **Move:** pět ploch: Nálada · Spánek · Garmin · Běh · Otužování.
- **Litánie:** dvě plochy: Litánie · Otázky.
- **Progress a Tasks** mají jen jednu plochu, tečky tam nejsou.

**Kulaté „+“ vpravo dole** přidá položku do otevřené sekce. V Litániích jde
nová položka do plochy, na které zrovna jsi. V Move se „+“ neukazuje, tam
se zapisuje klikem na den.

**Ozubené kolečko ⚙ nahoře** otevře zálohu, import, mazání historie úkolů a
znovunačtení dat z Garminu.

---

## Sekce

### Tasks — úkoly s termínem
Hlavní obrazovka appky. Shora dolů:
- **Lišta backlogů:** šipka vlevo otevře výběr všech backlogů (i víc najednou).
  Vedle jsou rychlá tlačítka **CHILL**, **MIX** a zkratky pro skupiny vlastních
  backlogů. Skupinu tvoří backlogy, jejichž název začíná dvěma velkými
  písmeny.
- **Chill náhled:** top 5 úkolů z backlogu Chill (večerní radosti).
- **Kalendář** po dnech (Dnes, Zítra, …). Propadlé úkoly zůstávají pod svým
  dnem („Včera“, „N dní zpět“), dokud je neodškrtneš nebo nepřesuneš.

U každého úkolu v kalendáři jsou čtyři akce:
- **✓** splněno: přiletí 🍹 a appka ti řekne něco hezkého,
- **→** posun o den, **»** o týden. Bez trestu, nic se nezaznamená.
- **✕** nesplněno, dobrovolné přiznání. Úkol se posune na zítřek.

**Klik na název úkolu** = editace. Tam nastavíš:
- prioritu (1–10),
- termín (Dnes / Zítra / datum; bez termínu úkol čeká v backlogu),
- **opakování** (denně až ročně) s volitelnou kotvou,
- **backlog**.

Přes „→ projekt…“ z úkolu uděláš krok projektu v Progressu. Pozor, tohle už
nejde vrátit.

**Systémové backlogy:**
- **Chill:** večerní nápady (sem posílá „+“ z Free). Odškrtnutý úkol z Chill
  prostě zmizí.
- **Wip:** rozdělané věci. Sem jde každý nový úkol, pokud nevybereš jiný
  backlog.
- **Done:** archiv splněných.
- **Mix:** jen pohled, nedá se do něj nic vložit. Sbírá úkoly bez termínu
  s vysokou prioritou ze všech tvých vlastních backlogů (ne z Chill, Wip ani
  Done). Slider **„Priorita od“** určuje práh: 10 = jen desítky, 8 = osm
  a víc. U každého úkolu je šedě vidět, odkud je.

Vlastní backlog přidáš tlačítkem „+ backlog“ v editaci úkolu. Ve výběru
backlogů ho tužtičkou ✎ přejmenuješ a nastavíš mu pořadí v rychlých volbách,
křížkem ✕ ho smažeš.

**Opakující se úkoly** se po odškrtnutí samy naplánují znovu. Buď klouzavě
od dne splnění (bez kotvy), nebo v pevném rytmu podle kotvy (blog vždy ve
středu, záloha na daň vždy 15. 6.).

### Progress — projekty s kroky
Pro věci, co mají víc kroků (např. „Stěhování bytu“). Projekt je placka:
rozklikneš ji a odškrtáváš kroky. Každý krok má **náročnost N** (1–10), která
určuje, jak velký kus proužku postupu ukrojí. Pořadí kroků i projektů měníš
sliderem v editaci. Šedé **+** na placce rovnou přidá další krok. Hotový
projekt dostane 🍹 a spadne dolů.

### Habits — návyky
Check rovina = dnešní ✓ / ✕. Přehled = mřížka **90 dnů** pro každý návyk.
**Klikáním přímo do čtverečků** zapisuješ i zpětně (prázdné → zelená →
červená → prázdné). Klik na název bloku otevře editaci, klik na čísla vpravo
celou historii.

Návyky s hvězdičkou ★ tvoří skupinu **Godzilla**: modrý den = žádný z nich
ten den nebyl červený.

### Free — volnočasové aktivity
To samé jako Habits, ale místo priority mají **Zábavu** (1–10), k tomu
**štítky** a **délku**. Nahoře je **hledání** (snese i překlepy) a **filtr**
podle štítků, zábavy a času. **„+“ u čísla v Přehledu** pošle aktivitu do
Tasks / Chill jako plán na večer.

### Move — tělo a pohyb
Každý den = jedno kolečko, řádek = týden (pondělí vlevo, nejnovější nahoře).
Do minulých dnů zapisuješ klikem na kolečko.
- **Nálada:** velikost = nálada (1–5), prstence = kontakt s lidmi a aktivita,
  pod kolečkem tělo a hlava.
- **Spánek:** velikost a barva = délka spánku (zelená od 7 h), prstence = čas
  usnutí a pocit. Čas usnutí piš jako `2320` nebo `0040`.
- **Garmin:** velikost = kalorie, číslo = minuty, pravý prstenec = vzdálenost,
  levý = tep. Plní se samo z Bobových hodinek. Klik na den zadá čísla ručně,
  den se tím zamkne (kalorie kurzívou) a automatika ho už nepřepíše.
- **Běh:** velikost = forma (tep vůči rychlosti), číslo = tep (má klesat),
  prstence = pravidelnost a „brzda“ proti přetížení. Pod kolečkem ručně
  zapsaná těžkost a achilovky.
- **Otužování:** velikost = výkon (minuty ve vodě vůči teplotě), prstence =
  teplota vody a vzduchu, pod kolečkem vítr a průtok.

### Litánie — podpůrné věty a otázky
Dvě knihovny: **Litánie** (věty, přerámování, mantry; priorita 1–5) a
**Otázky** (poznávací otázky; skóre 1–10). Každá věta může mít ★ oblíbenou.

**Filtr:** tři barevné skupiny štítků (modrá témata, červené části, zelené
skupiny), slider priority a „Bez tagů“. Vedle hledání je rychlá ☆ na oblíbené.
Štítky v jedné skupině se sčítají (NEBO), mezi skupinami se zužují (A ZÁROVEŇ).
Příklad: „terapie“ + „Otázky“ = terapeutické otázky.

Mazání není doopravdy mazání: věta se jen schová a pár vteřin jde vrátit
tlačítkem „Vrátit“.

Obsah je čistě soukromý a nikde se nestahuje. Dostane se sem jen obnovením
zálohy, jinak jen to, co sám přidáš přes „+“.

---

## Zálohování (čti to)

⚙ → **Exportovat zálohu (JSON)** stáhne `habity-zaloha.json` s úplně vším.
Přes **Načíst zálohu ze souboru** ho kdykoli naleješ zpátky. Import po
potvrzení **přepíše všechno**. Zálohuj pravidelně a soubor si někam odlož
(Dropbox, disk), jiná kopie tvých dat neexistuje.

⚙ → **Vymazat historii úkolů** smaže jen interní počty splněných
a nesplněných úkolů, ne samotné úkoly.
