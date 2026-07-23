# Formato Prompt TuaGPT — Schema tecnico e guida redazionale

Documento di riferimento per il formato `.prompt` accettato da TuaGPT (import/export, catalogo esempi, prompt di sistema).

**Destinatari**

| Audience | Sezioni utili |
|----------|----------------|
| Integratori / tool esterni / GitHub | §1–§5 (contratto XML) |
| Autori di prompt (Prompt Wizard) | §6–§8 (redazione e best practice) |
| Chi vuole capire il runtime | §9–§10 (pipeline, DiSistema) |

---

## 1. Panoramica

TuaGPT distingue tre famiglie di prompt:

| Famiglia | Dove | Uso |
|----------|------|-----|
| **Template utente** | DB SQLite + file `.prompt` importati | Chat, tasti FN, agenti background |
| **Prompt di sistema** | `SystemPrompt\*.prompt` (+ DB categoria `DiSistema`) | ChatSystemPrompt, RAG, WebSearch, Workflow, Planner, … |
| **Catalogo esempi** | `PromptCollection\**\*.prompt` | Distribuzione / import manuale |

Il formato di scambio ufficiale è **XML `.prompt` versione `1.0`**. Il testo strutturato legacy (sezioni `OPERAZIONI DA ESEGUIRE:`, …) resta accettato in lettura ma non è il formato consigliato per nuovi prompt.

---

## 2. Contratto XML — struttura

### 2.1 Root

```xml
<?xml version="1.0" encoding="utf-8"?>
<prompt version="1.0">
  <metadata>…</metadata>
  <content>…</content>
</prompt>
```

- Attributo obbligatorio: `version="1.0"`.
- Encoding consigliato: UTF-8.
- Estensione file: `.prompt` (anche `.xml` in import wizard).

### 2.2 `metadata` (non inviato al LLM come blocco dedicato)

| Elemento | Obbl. | Note |
|----------|-------|------|
| `name` | sì* | Nome univoco visualizzato; usato anche come tag `[NomePrompt]` in chat |
| `category` | consigliato | Es. `Scrittura e contenuti`; **non** usare `DiSistema` per template utente |
| `description` | no | Solo uso interno (elenco Gestione Prompt); **non** va al modello |
| `release` | no | Attributi opzionali `version`, `publishedAt` (ISO-8601). Alias legacy: `timestamp`, `date` |

\*Se assente in export, il builder usa `UnnamedPrompt`.

```xml
<metadata>
  <name>RevisoreTesti</name>
  <category>Scrittura e contenuti</category>
  <description>Revisione ortografica e stilistica…</description>
  <release version="1.0.0" publishedAt="2026-04-13T12:00:00Z"/>
</metadata>
```

### 2.3 `content` — sezioni

Ordine consigliato (il converter non dipende dall’ordine, ma catalogo e wizard lo seguono):

| Elemento | Ruolo | Inviato al LLM |
|----------|-------|----------------|
| `introduction/text` | Ruolo e compito principale | Sì (testo piano in testa) |
| `context` | Contesto operativo / dati di scenario | Sì (`CONTESTO: …`) |
| `domain` | Dominio tematico | Sì (`DOMINIO: …`) |
| `outputDesired/format` | Formato risposta | Sì (`OUTPUT DESIDERATO` → Formato) |
| `outputDesired/length` | Lunghezza risposta | Sì (`OUTPUT DESIDERATO` → Lunghezza) |
| `operations/operation` | Passi numerati | Sì (`OPERAZIONI DA ESEGUIRE`) |
| `decisionCriteria` | Quando chiedere / decidere | Sì |
| `recognitionPatterns` | Tipi di richiesta riconosciuti | Sì |
| `behaviorRules/rule` | Regole di comportamento | Sì |
| `examples/example` | Few-shot | Sì |
| `outputConstraints/constraint` | Vincoli duri sull’output | Sì |
| `variables/variable` | Dichiarazione variabili | Indiretto (placeholder `{Nome}`) |

`description` in metadata **non** compare nel testo inviato al modello.

### 2.4 Esempio minimo valido

```xml
<?xml version="1.0" encoding="utf-8"?>
<prompt version="1.0">
  <metadata>
    <name>EsempioMinimo</name>
    <category>Generale</category>
    <description>Prompt minimale per test import</description>
  </metadata>
  <content>
    <introduction>
      <text>Sei un assistente conciso. Rispondi in italiano.</text>
    </introduction>
  </content>
</prompt>
```

### 2.5 Esempio completo con variabili

Vedi anche: `PromptCollection/Professionale e Business/ProductMarketingManagerRelease.prompt`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<prompt version="1.0">
  <metadata>
    <name>PMM TuaGPT — Solo release</name>
    <category>Assistenza</category>
    <description>Comunicazioni di release (titolo, bullet, FAQ)</description>
  </metadata>
  <content>
    <introduction>
      <text>Sei il Product Marketing Manager per TuaGPT in modalità comunicazione di release…</text>
    </introduction>
    <context>Versione {VersioneRelease}. Materiale: {MaterialeRelease}.</context>
    <domain>Release notes, FAQ, email prodotto</domain>
    <outputDesired>
      <format value="5">Markdown con sezioni Titolo, Bullet, FAQ</format>
      <length value="2">Conciso</length>
    </outputDesired>
    <operations>
      <operation order="1">Estrarre novità da {MaterialeRelease}</operation>
      <operation order="2">Scrivere titolo e bullet orientati al beneficio</operation>
    </operations>
    <behaviorRules>
      <rule>Rispondi in italiano salvo richiesta diversa</rule>
    </behaviorRules>
    <examples>
      <example>
        <input>Versione 1.2.6, changelog: fix salvataggio</input>
        <output>Titolo + bullet + FAQ aggiornamento</output>
        <description>Release tecnica</description>
      </example>
    </examples>
    <outputConstraints>
      <constraint>Massimo 6 FAQ salvo richiesta esplicita</constraint>
    </outputConstraints>
    <variables>
      <variable name="VersioneRelease" required="false" description="Numero versione"/>
      <variable name="MaterialeRelease" required="false" description="Changelog / note QA"/>
    </variables>
  </content>
</prompt>
```

---

## 3. Enum e attributi tipizzati

### 3.1 `outputDesired/format/@value` (`OutputFormat`)

| value | Enum | Uso tipico |
|------:|------|------------|
| 1 | `TestoLibero` | Prosa libera |
| 2 | `Elenco` | Bullet / numerato |
| 3 | `Tabella` | Markdown table |
| 4 | `JSON` | Struttura macchina |
| 5 | `Markdown` | Documentazione / sezioni |
| 6 | `HTML` | Markup HTML |

Il **testo interno** del nodo `format` è prioritario rispetto all’enum (istruzioni libere al modello). L’attributo `value` serve al wizard e alla classificazione.

### 3.2 `outputDesired/length/@value` (`OutputLength`)

| value | Enum |
|------:|------|
| 1 | `Breve` |
| 2 | `Media` |
| 3 | `Dettagliata` |

### 3.3 `operation/@order`

Intero 1-based. In conversione XML→testo le operazioni sono ordinate per `order`.

### 3.4 `example`

| Figlio | Obbl. | Note |
|--------|-------|------|
| `input` | sì* | Richiesta utente di esempio |
| `output` | sì* | Risposta attesa |
| `description` | no | Nota per l’autore / contesto |

\*Esempi senza `input`+`output` valorizzati vengono ignorati dal parser.

---

## 4. Variabili

### 4.1 Sintassi nel contenuto

Placeholder: `{NomeVariabile}` (parentesi graffe semplici).

```text
Analizza il documento {TitoloDocumento} per il cliente {NomeCliente}.
```

Regole:

- Il nome nel placeholder deve coincidere con `variable/@name` (sostituzione con `String.Replace` sul nome esatto).
- Usare nomi senza spazi; preferire PascalCase (`VersioneRelease`).
- Non confondere con:
  - `{{item}}` — sintassi del **Workflow DSL** (`SystemPrompt\Workflow.prompt`), non variabili utente
  - `[NomePrompt]` — tag UI che attiva la sostituzione del `ChatSystemPrompt` all’invio
  - placeholder interni wizard (`{TASK_TYPE}`, `{CONTEXT}`, …) — solo template generici interni

### 4.2 Dichiarazione XML

```xml
<variable
  name="Argomento"
  required="true"
  description="Tema della bozza"
  defaultValue=""/>
```

| Attributo | Obbl. | Note |
|-----------|-------|------|
| `name` | sì | Chiave di sostituzione |
| `required` | no | `true` / `false` (default false) |
| `description` | no | Mostrata nel dialogo variabili |
| `defaultValue` | no | Usato se l’utente non valorizza |

**Nota importante:** nel file `.prompt` esportato dal builder **non** sono serializzati tipo UI (`text`, `number`, …), `AllowedValues` né regex. Quei metadati vivono nel database (`PromptVariable`) e si configurano nel Variable Wizard. In import da file, il tipo parte tipicamente da testo e va rifinito in Gestione Prompt se serve.

### 4.3 Tipi in database / UI workflow

Valori usati in `PromptVariable.VariableType` / `ChatWorkflowVariables`:

| Valore | UI | Note |
|--------|-----|------|
| `text` | Testo | Default |
| `number` | Numero | |
| `datetime` / `date` | Data | |
| `bool` / `boolean` | Booleano | |
| `choice` / `choice_exclusive` | Scelta singola | `ChoiceMode` / `AllowedValues` |
| `choice_multi` | Scelta multipla | |
| `email` | Email | |
| `url` | URL | |

Validazione avanzata: `ValidationRule` (regex) nel Variable Wizard.

### 4.4 Algoritmo di sostituzione (`ProcessPrompt`)

Per ogni variabile dichiarata sul template:

1. Valore dal dialogo / dizionario passato →
2. altrimenti `DefaultValue` →
3. altrimenti se `required` → placeholder letterale `[NomeVariabile]` →
4. altrimenti stringa vuota.

Poi: `content.Replace("{Nome}", valore)`.

### 4.5 Variabili workflow per conversazione

- Persistono nella chat corrente (`ChatWorkflowVariables`).
- Si aprono all’inserimento prompt / tasto FN, oppure dal pulsante **Variabili workflow**.
- Valori riutilizzati tra prompt diversi nella stessa conversazione (merge per nome).
- Se il template **non** sostituisce `ChatSystemPrompt`, valori non vuoti possono essere iniettati anche come blocco `=== VARIABILI WORKFLOW CHAT ===` nel system prompt combinato.

### 4.6 Placeholder non riferiti nel testo

Se una variabile è dichiarata ma `{Nome}` non compare in nessuna sezione convertita, `XmlToTextConverter` aggiunge una sezione `DATI DINAMICI:` con descrizione + placeholder, così la sostituzione a runtime resta possibile.

---

## 5. Import / export e compatibilità

| Canale | Supporto |
|--------|----------|
| Prompt Wizard → Esporta / Importa `.prompt` / `.xml` | Completo (XML 1.0) |
| Copia file in `PromptCollection` | Distribuzione statica; import **manuale** dal wizard |
| JSON template intero | Non supportato (JSON solo per metadati variabili nel DB) |
| Testo legacy (sezioni maiuscole) | Lettura / conversione parziale (`PromptParser`) |

**Backward compatibility**

- File senza `<release>`: ok.
- Contenuto che non inizia con `<`: trattato come testo piano (`XmlToTextConverter` lo lascia invariato).
- Categoria `DiSistema`: riservata; non creare template utente con quel nome categoria.

---

## 6. Guida redazionale — come scrivere un buon prompt

Questa sezione è pensata per chi lavora con il **Prompt Wizard** (non è obbligatorio scrivere XML a mano).

### 6.1 Ruolo delle sezioni

| Sezione | Scrivi… | Evita… |
|---------|---------|--------|
| **Introduzione** | Chi è il modello, compito primario, lingua | Lunghi elenchi di regole (mettile in Regole) |
| **Contesto** | Scenario, prodotti, vincoli di dominio, `{variabili}` di scenario | Descrizione marketing del template (usa `description` metadata) |
| **Dominio** | Area tematica corta | Paragrafi lunghi |
| **Formato / lunghezza** | Struttura attesa (Markdown, sezioni, JSON schema verbale) | Contraddire i vincoli di output |
| **Operazioni** | Passi ordinati, azionabili | Astrazioni vaghe (“sii utile”) |
| **Criteri di decisione** | Quando chiedere vs procedere con assunzioni | Criteri che bloccano sempre |
| **Pattern** | Frasi tipiche dell’utente | Elenco infinito |
| **Regole** | Hard constraints di comportamento | Duplicare le operazioni |
| **Esempi** | 1–3 few-shot realistici e corti | Esempi che contraddicono le regole |
| **Vincoli output** | Limiti misurabili (max FAQ, no inventare numeri) | Preferenze soft (meglio in Regole) |

### 6.2 Quando usare variabili

Usa `{Nome}` quando il valore:

- cambia ad ogni utilizzo (versione, cliente, lingua target);
- deve essere chiesto esplicitamente all’utente;
- deve persistere nella chat (workflow).

Preferisci testo fisso quando:

- è parte stabile del ruolo (“Sei un revisore legale…”);
- non serve UI di raccolta dati.

**Obbligatorie vs opzionali**

- `required="true"` solo se senza quel dato il prompt è inutilizzabile.
- Per opzionali, documenta in Contesto: “se vuoto, usa solo il messaggio utente”.

### 6.3 Pattern consigliati

1. **Ruolo stretto** — una missione per template (es. “solo release”, non “tutta la marketing strategy”).
2. **Assunzioni esplicite** — nei criteri: “procedi e marca le assunzioni” quando i dati mancano in parte.
3. **Anti-hallucination** — vincolo: non inventare numeri, certificazioni, fonti.
4. **Output revisionabile** — sezione finale “Review” / checklist per l’umano.
5. **Allineamento tool** — se il prompt guida RAG/web/cartella sicura, rinvia ai nomi tool reali (vedi `DOCUMENTAZIONE_TOOL_LLM_TUAGPT.md`); non inventare API.

### 6.4 Cosa non fare

- Creare / usare template in categoria **DiSistema** per la chat utente.
- Digitare a mano `[NomePrompt]` senza passare dal dialogo variabili se ci sono obbligatorie (restano `[NomeVariabile]` letterali).
- Usare `{{doppia graffa}}` per variabili utente.
- Mettere istruzioni solo in `description` metadata (non arrivano al modello).
- Prompt FN che tentano di “disattivare” WebSearch/RAG di sistema: i template utente si **impilano** nel system prompt combinato; non sostituiscono i blocchi tool di sistema (salvo sostituzione `ChatSystemPrompt` via tag `[Nome]`).

---

## 7. Uso in chat e tasti FN

### 7.1 Flusso tipico

1. **Gestione Prompt** → crea/importa template (non DiSistema).
2. In chat: **Inserisci Prompt** oppure tasto **FN** assegnato.
3. Dialogo variabili → valori salvati in workflow chat.
4. Nel campo messaggio compare il tag `[NomePrompt]` (+ eventuale testo utente).
5. All’invio: il contenuto del template (XML → testo, variabili già sostituite) **sostituisce** il blocco `ChatSystemPrompt` nel system prompt combinato; il messaggio utente resta quello digitato (con o senza testo aggiuntivo).

### 7.2 Tag `[NomePrompt]`

- Attiva il caricamento del template omonimo (case-insensitive sul nome).
- Deve corrispondere a `metadata/name`.
- Prompt DiSistema esclusi da questo percorso.

### 7.3 Tasti funzione

Da Gestione Prompt → **Assegna FN**:

- modalità inserimento nel campo messaggio;
- opzione invio immediato;
- modalità agente in background (compiti lunghi senza occupare la composizione).

Dettagli operativi: `Guida/index.html` § Tasti funzione e § Prompt; approfondimento redazionale per CMS: `Guida/guida-prompt.html`.

---

## 8. Checklist qualità prima della pubblicazione

- [ ] `name` chiaro e stabile (rinominare rompe i riferimenti FN / abitudini utenti)
- [ ] `description` utile in elenco, non duplicata nell’introduzione
- [ ] Introduzione ≤ ~1500 caratteri salvo necessità
- [ ] Ogni `{Variabile}` dichiarata in `<variables>`
- [ ] Ogni variabile dichiarata usata nel testo **oppure** accettabile in `DATI DINAMICI`
- [ ] Almeno un esempio se il formato output è strutturato
- [ ] Vincoli anti-invenzione se dominio legale/medico/fiscale
- [ ] `release/@version` aggiornato se distribuisci in catalogo pubblico
- [ ] Import di prova nel Prompt Wizard su installazione pulita

---

## 9. Pipeline runtime (riferimento tecnico)

```text
File .prompt / DB Content (XML)
        │
        ▼
PromptWizard / GestionePrompt  ──import/export──►  XmlPromptParser / XmlPromptBuilder
        │
        ▼
Inserimento in chat / FN
        │
        ├─► PromptVariablesDialog + ChatWorkflowVariablesManager
        │
        ▼
PromptManager.ProcessPrompt  ({Nome} → valori)
        │
        ▼
Cache _currentTemplateContent  (XML già sostituito)
        │
        ▼
Invio messaggio con [NomePrompt]
        │
        ▼
GetTemplateContentForSend
        │
        ▼
XmlToTextConverter.ConvertToText
        │
        ▼
BuildCombinedSystemPrompt  (sostituisce ChatSystemPrompt; altri blocchi sistema restano)
```

**Prompt di sistema canonici** (`SystemPromptCatalog`):  
`ChatSystemPrompt`, `QueryBoost`, `WebSearch`, `UnifiedSearch`, `Thunderbird`, `ChatTitleGenerator`, `RAG`, `BrowserAutomation`, `DesktopAutomation`, `Planner`, `Workflow`.

---

## 10. Mappa pubblicazione consigliata

| Artefatto | Destinazione | Pubblico |
|-----------|--------------|----------|
| Questo file (`DOCUMENTAZIONE_PROMPT.md`) | Repo prodotto e/o repo pubblico “prompt-spec” | Sviluppatori / partner |
| `Guida/guida-prompt.html` | CMS / sito TuaGPT (frammento HTML minimale) | Utenti finali autori di prompt |
| `PromptCollection/**` + `PromptCollection/README.md` | Repo pubblico esempi (GitHub) | Community / template pack |
| XSD opzionale (futuro) | Accanto alla spec su GitHub | Validazione CI |

Vedi anche la guida operativa a fine di questo flusso nel messaggio di rilascio / README Guida.

---

## 11. Riferimenti correlati

- `DOCUMENTAZIONE_TOOL_LLM_TUAGPT.md` — tool esposti al modello
- `Guida/index.html` — panoramica funzionale Gestione Prompt / FN
- `Guida/guida-prompt.html` — guida redazionale (frammento CMS)
- `PromptCollection/README.md` — indice catalogo esempi
- `IdeeProgetto/DSL_Workflow_Tool_Programmatic.md` — DSL workflow (distinto dai template utente)
