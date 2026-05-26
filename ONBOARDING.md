# TV4 Livetider — projektöversikt

Projektet hostar en HTML-rapport (TV4:s sändningstider) på GitHub Pages med lösenordsskydd, och pushas automatiskt varje gång generatorscriptet körs.

## Användaren

Carl Montelius (Callegit på GitHub, carl.montelius@tv4.se). Jobbar på TV4/TVM. Tekniknivå: kan följa instruktioner och redigera config, är inte utvecklare på heltid. Föredrar svenska i konversation.

## Vad som finns och var

### På GitHub
- **Repo:** https://github.com/Callegit/tv4-livetider (publikt)
- **Live-sida:** https://callegit.github.io/tv4-livetider/
- **Lösenord (SHA-256-hashat i `docs/index.html`):** `tv4live`

### Lokalt på jobbdatorn (Windows)
- **Repo-mapp:** `C:\Users\CMontelius\OneDrive - TVM\Documents\GitHub\Tidsmail`
- **Generator-mapp:** `C:\Users\CMontelius\OneDrive - TVM\Desktop\Ny mapp (3)`

> ⚠️ Hemmadatorn har förmodligen andra sökvägar — repot kan klonas från GitHub men generator-scripten (.bat + .vbs) ligger inte i repot, bara på jobbdatorn.

## Arkitektur — hur sidan uppdateras

```
[BXF/MPL-loggfiler]
     ↓
Kör.bat
     ├─ Steg 1: Hämtar nya loggar från nätverksmapp (om REMOTE satt i config)
     ├─ Steg 2: Arkiverar gamla loggar (om ARKIVERA=JA)
     ├─ Steg 3: Kör tv4_sandningsplan_html.vbs → genererar Tider\Livetider.html
     ├─ Steg 4: Kopierar Livetider.html till SharePoint-synkmapp (om SHAREPOINT satt)
     ├─ Steg 5: Kopierar Livetider.html till repots docs\ + git commit + git push
     └─ Steg 6: Klart
                ↓
        GitHub Actions workflow (.github/workflows/deploy.yml)
                ↓
        GitHub Pages uppdaterar live-sidan inom ~30-60 sek
```

## Filer i repot

| Fil | Vad den gör |
|---|---|
| `docs/index.html` | Inloggningssida. SHA-256-hash av lösenordet ligger i JS-konstanten `EXPECTED_HASH`. Vid korrekt lösen laddas `Livetider.html` i en iframe. SessionStorage sparar auth för session. |
| `docs/Livetider.html` | Den genererade sändningstabellen (uppdateras automatiskt). UTF-16-kodad output från VBScript. |
| `docs/.nojekyll` | Tom fil. Säger till Pages att skippa Jekyll-bearbetning. |
| `.github/workflows/deploy.yml` | GitHub Actions-workflow som deployar `docs/` till Pages vid varje push till main. |
| `.gitignore` | Ignorerar `.claude/`. |

## Filer utanför repot (på jobbdatorn)

Ligger i `C:\Users\CMontelius\OneDrive - TVM\Desktop\Ny mapp (3)`:

| Fil | Vad den gör |
|---|---|
| `Kör.bat` | Orkestreringsscript. Modifierat — har `[5/6]`-steget som pushar till GitHub. Läser `GITHUB_REPO`-sökvägen från `config.txt`. |
| `tv4_sandningsplan_html.vbs` | Den faktiska HTML-generatorn (~332 KB VBScript). Genererar `Tider\Livetider.html` från BXF/MPL-loggar i `Loggar\`. |
| `config.txt` | Konfiguration. Innehåller `GITHUB_REPO=C:\Users\CMontelius\OneDrive - TVM\Documents\GitHub\Tidsmail` (sökväg till lokala repot). |
| `SetupAutostart.bat` | Sätter upp Windows Task Scheduler för automatisk körning. |
| `Loggar\`, `Arkiv\`, `Tider\` | Arbetsmappar. Tider innehåller den genererade HTML:en. |

## Konfiguration (config.txt)

Relevanta rader:
```
ARKIVERA=NEJ              # Flytta gamla loggar till Arkiv\
SHAREPOINT=AUTO           # Kopiera till SharePoint-synkmapp
AUTOMATISK=NEJ            # JA = skippa pause + hoppa över om inga nya loggar
GITHUB_REPO=C:\Users\CMontelius\OneDrive - TVM\Documents\GitHub\Tidsmail
```

## Vanliga problem och lösningar

### "Det första GitHub Pages-bygget gick fel"
GitHubs legacy `pages-build-deployment`-pipeline är ostabil. Vi använder istället egen workflow (`.github/workflows/deploy.yml`) med `actions/deploy-pages@v4`. Pages-source ska stå på **"GitHub Actions"** i repo-inställningarna (Settings → Pages → Source).

### "Kör.bat frågar efter inloggning vid git push"
Git Credential Manager (bundlad med Git for Windows) poppar upp en webbläsare för OAuth-login mot GitHub första gången. Logga in en gång, sen funkar det tyst. Om GCM saknas: installera Git for Windows från https://git-scm.com/download/win.

### "Livetider.html visas konstigt på sidan"
Den är UTF-16 LE med BOM. Browsers ska hantera det via BOM, men om det glitchar kan VBScript:et behöva ändras för att outputa UTF-8 istället. Inte gjort än — vänta tills problem uppstår.

### "Hur byter jag lösenord?"
Generera ny SHA-256-hash:
```powershell
$pw = "nytt_losen"
[BitConverter]::ToString([Security.Cryptography.SHA256]::Create().ComputeHash([Text.Encoding]::UTF8.GetBytes($pw))).Replace("-","").ToLower()
```
Klistra in i `docs/index.html`, raden `const EXPECTED_HASH = '...'`. Commit + push.

### "Hur lägger jag till en kollega som inte ska kunna ändra?"
De behöver bara länken + lösenord, inget GitHub-konto behövs. Datan är publikt nåbar via direktlänk till `Livetider.html` om någon listar ut URL:en, men kan inte googlas (ingen indexering förrän man hittar dit).

## Förbättringar som diskuterats men inte gjorts

- Riktig auth (email-PIN) via Cloudflare Pages + Access — om TVM vill ha starkare skydd
- Bytt UTF-16 → UTF-8 i VBScript-output för säkrare browser-rendering
- README.md i repot — finns ingen ännu
- Skydd mot direktåtkomst till `Livetider.html` (just nu kan vem som helst gå direkt dit utan lösen) — kräver server-side, ej trivialt på Pages

## Git-historik (de viktiga commits)

```
82e5ae6 Add GitHub Actions workflow for Pages deployment
509276d Trigger Pages build  (tom commit, kan ignoreras)
144208a Add GitHub Pages site with password gate and initial Livetider.html
d2db239 Initial commit
```

## Att fortsätta hemifrån

1. Klona repot: `git clone https://github.com/Callegit/tv4-livetider.git`
2. Generator-scripten (.bat + .vbs) finns inte i repot — om du behöver dem hemifrån, kopiera från OneDrive eller från jobbdatorn
3. För att redigera live-sidan: ändra `docs/index.html`, commit, push → live inom 30-60 sek
