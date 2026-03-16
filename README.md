# Styrande Dokument – Navigator

Interaktiv navigeringssida för dokumentbiblioteket **Styrande Dokument** på Sunds Industrier AB:s SharePoint-site för ledningssystemet.

> 🌐 **GitHub Pages-länk:** `https://felixsunds.github.io/styrande-dokument/`  
> 📁 **SharePoint-site:** `https://sundsindustrier.sharepoint.com/sites/management-system`

---

## Vad är det här?

En statisk HTML-fil (`styrande-dokument-navigator.html`) som bäddas in som en `<iframe>` på SharePoint-sidan. Den ersätter SharePoints inbyggda dokumentbiblioteksvy med en snabb, filtrerbar navigator med stöd för:

- **Hierarkisk filtrering per ISO-standard** (t.ex. ISO 9001 → §8 → 8.5 → 8.5.1)
- **Filtrering per processområde, status och dokumenttyp**
- **Fritextsökning** på dokumentnamn och standardkopplingar
- **Direktlänk** till varje dokument i SharePoint vid klick på ett kort

All data är inbäddad direkt i HTML-filen som en JSON-array (`RAW_DATA`). Det finns ingen backend – sidan är helt statisk.

---

## Hur sidan används

### För slutanvändare (verkstad, kontor)

1. Öppna SharePoint-sidan för ledningssystemet.
2. Navigatorn är inbäddad direkt på sidan via en iframe.
3. Filtrera i **vänsterkolumnen** genom att:
   - Klicka på en standard i trädet (t.ex. `ISO 9001 → §8 → 8.5`) för att visa alla dokument kopplade till den klausulen.
   - Kryssa i ett eller flera **processområden**, **statusar** eller **dokumenttyper**.
4. Använd **sökrutan** högst upp för att söka på ord i dokumentnamnet eller standard-etiketten.
5. Klicka på ett dokumentkort för att öppna dokumentet direkt i SharePoint (öppnas i ny flik).
6. Klicka på **Rensa alla filter** för att återställa vyn.

---

## Hur man uppdaterar navigatorn

Navigatorn visar den data som är hårdkodad i HTML-filen. När dokument läggs till, tas bort eller får ändrade metadata i SharePoint måste filen uppdateras och pushas till GitHub.

### Steg-för-steg

#### 1. Exportera aktuell data från SharePoint

Kör Power Automate-flödet som hämtar alla poster från dokumentbiblioteket **Styrande Dokument** och genererar ett nytt HTML-block med uppdaterat `RAW_DATA`. Flödet hämtar följande fält per dokument:

| Fält i SP | Variabelnamn i JSON |
|---|---|
| Filnamn | `name` |
| Standard (Term Store) | `standard` |
| Processområde (Term Store) | `process` |
| Status | `status` |
| Innehållstyp | `ct` |
| Senast ändrad | `modified` |
| Länk till filen | `url` |

#### 2. Ersätt RAW_DATA i HTML-filen

Öppna `index.html` och hitta raden som börjar med:

```js
const RAW_DATA = [{ ...
```

Ersätt hela arrayen (från `[` till `];`) med den nya datan från Power Automate-exporten.

Uppdatera även tidsstämpeln på raden nedanför:

```js
const GENERATED_AT = "ÅÅÅÅ-MM-DD HH:MM";
```

#### 3. Pusha till GitHub

```bash
git add index.html
git commit -m "Uppdatera dokumentdata ÅÅÅÅ-MM-DD"
git push origin main
```

GitHub Pages publicerar ändringen automatiskt inom ~30 sekunder till 2 minuter.

#### 4. Verifiera

Öppna `https://felixsunds.github.io/styrande-dokument/` i webbläsaren och kontrollera att:
- Tidsstämpeln i övre högra hörnet stämmer.
- Antalet dokument (visas i rubrikraden i höger panel) stämmer.
- Ett par dokument du vet är nya syns i listan.

---

## Inbäddning i SharePoint

Navigatorn bäddas in via en **Bädda in**-webbdel (Embed web part) på SharePoint-sidan med följande iframe-kod:

```html
<iframe
  src="https://felixsunds.github.io/styrande-dokument/"
  width="100%"
  height="1200"
  frameborder="0"
  style="border:none;">
</iframe>
```

SharePoint blockerar externa iframes med CSP (Content Security Policy) om inte sidan är godkänd. GitHub Pages (`github.io`) är godkänt för inbäddning i SharePoint Online via Embed-webbdelen.

---

## Tekniska detaljer

| Egenskap | Värde |
|---|---|
| Ramverk | Vanilla JavaScript (ingen extern runtime) |
| Typsnitt | IBM Plex Sans + IBM Plex Mono (Google Fonts) |
| Datakälla | Inbäddad JSON (`RAW_DATA`) i HTML-filen |
| Standardparser | Stöd för SP Taxonomy-objekt och råsträngar (`9;#Label;#17;#Label2`) |
| Hierarki | Byggs automatiskt från `ISO Standard:§Kapitel:Klausul`-format |
| Hosting | GitHub Pages (gratis, statisk) |

### Standardformat

Standarder lagras i Term Store med formatet:

```
ISO 9001:§8:8.5:8.5.1
ISO 3834-2:§10:10.4
Ej ISO-styrt
```

Kolonen i etiketten (`:`) används som nivåavdelare i trädet. Paragrafsymbolen (`§`) markerar kapitelnivån.

---

## Vanliga frågor

**Filen är uppdaterad på GitHub men navigatorn visar gammal data.**  
Vänta 1–2 minuter för GitHub Pages att publicera. Gör sedan en hård omladdning i webbläsaren (`Ctrl + Shift + R` / `Cmd + Shift + R`) eller töm cachen.

**Ett dokument syns inte i navigatorn trots att det finns i SharePoint.**  
Kontrollera att dokumentet ingick i den senaste dataexporten. Om dokumentet lades till efter senaste export måste ett nytt export och en ny push göras.

**Tidsstämpeln visas inte.**  
Kontrollera att variabeln `GENERATED_AT` är en icke-tom sträng i HTML-filen.

**Länken till ett dokument fungerar inte.**  
Kontrollera att `url`-fältet i `RAW_DATA` för det dokumentet pekar på rätt SharePoint-URL och att användaren har läsbehörighet till filen.

---

## Ansvarig

Underhålls av **Felix** – systemadministratör och controller, Sunds Industrier AB.  
För frågor om innehållet i dokumentbiblioteket, kontakta respektive dokumentägare.
