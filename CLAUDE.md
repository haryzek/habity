# CLAUDE.md — Habity

**Nejdřív si přečti `README.md`** (celé). Je to podrobná a ověřená mapa appky:
sekce, datový model, proč jsou věci tak, jak jsou, pasti a známé bugy.
`NAVOD.md` je uživatelský návod pro lidi. Tenhle soubor je jen rozcestník a
pracovní pravidla.

## Bob
- Mluv česky, nespisovně, jako se starým kámošem. Oslovuj ho „Bobe“, ne „kámo“.
  🥥🍹 sedí. Jednej s ním jako rovný s rovným, měj vlastní názor, buď přesný
  a bez vycpávkových frází.
- „Ještě nepracuj“ = jen prozkoumej a navrhni. Kód sáhni, až kejvne.
- Když ladíte design, ptej se po jedné otázce. Proaktivně flaguj edge-casy
  a rizika.
- Diktuje přes mikrofon, zkomoleniny si oprav sám („VIP“ = Wip, „DAN“ = Done).
- `#` na začátku promptu = odpověz jednou větou.

## Projekt v kostce
- Jednosouborová PWA `index.html` (~5340 řádků), GitHub Pages
  `haryzek.github.io/habity/`. Bez buildu a bez závislostí.
- **Netlify (od 2026-10-27, kdy GitHub suspendoval účet):** `https://starlit-empanada-417a72.netlify.app`.
  Deploy přes Netlify CLI (Bob přihlášený): zkopírovat **jen** `index.html`, `move_data.json`,
  `run_data.json` do dočasné složky mimo repo (nikdy celé repo, `local/` je soukromé) a
  `netlify deploy --prod --no-build --dir <složka> --site starlit-empanada-417a72`. Jen s Bobovým gou.
- Data jsou jen v localStorage `habity.v2` (+ JSON export). Nové pole přidává
  `normalize()` nedestruktivně. Na systémové backlogy Tasks je verze `blV`
  (teď 3).
- Sekce (`curSection`): 0 Progress · 1 Tasks (start) · 2 Habits · 3 Free ·
  4 Move · 5 Litánie. V navu je Litánie první zleva, ale index má 5.
- Klíče ve `state` nesedí na UI: `tasks` = Progress, `utasks` = Tasks,
  `sections[0/1]` = Habits/Free. Nepřejmenovávat.

## Tvrdá pravidla
- **Litánie a otázky jsou soukromé.** Nikdy necommituj jejich zdroje
  (`local/…`) ani jejich obsah do repa. Pages je veřejné.
- `move_data.json` a `run_data.json` v kořeni zapisuje Bobův Garmin skript
  (commity „Auto: update Garmin data“). Ručně do nich nesahej. Skript běží
  v Bobově lokálním repu, takže lokál kvůli němu pullovat nemusí. Pull je
  potřeba jen po práci v cloudu. Cloudová session má před pushem udělat
  fetch, protože na GitHubu mezitím mohly přibýt Garmin commity.
- Repeat úkol se nikdy neklonuje, respawn je jen po ✓. Červená v Tasks vzniká
  jen z ručního ✕. Auto-rollup je zrušený, nevracet bez Boba.
- Na `moveSemi` (side +1 = LEVÁ) ani na kotvení biweekly na `t.date`
  nesahej bez změření.

## Workflow
1. Před zásahem si `index.html` přečti znovu (cílené `grep -n` a výřezy, soubor
   je velký).
2. Po zásahu udělej syntax check: obsah `<script>` přes Node `new Function(…)`.
3. Funkčně otestuj v headless Chromiu. V cloudu je Playwright
   (`require($(npm root -g)/playwright)`, prohlížeče v `/opt/pw-browsers`):
   1. Otevři `file://…/index.html`.
   2. Nastav `localStorage["habity.v2"]`.
   3. `reload()`.
   4. Volej funkce a hlídej `pageerror`.
4. **README aktualizuj ve stejném commitu jako kód**, když měníš chování,
   datový model nebo UI. Kde se text a kód rozcházejí, pravdu má kód, a text
   pak oprav.
5. Commit zprávy piš česky ve stylu „Sekce: co se změnilo“.
6. **Lokálně** (`E:\_dev\habity`, Windows) se commituje a pushuje do `main`.
   **V cloudu** pracuj na přidělené větvi, pushni ji a do `main` merguj, až
   když to Bob řekne.
