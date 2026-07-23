# TuaGPT Prompt Format

Formato ufficiale **`.prompt`** (XML 1.0) accettato da TuaGPT: specifica, variabili e esempi importabili.

## Documentazione

La specifica completa è in **[DOCUMENTAZIONE_PROMPT.md](DOCUMENTAZIONE_PROMPT.md)** (contratto XML, variabili `{Nome}`, best practice, checklist).

## Uso rapido

1. Scegli un file in [`examples/`](examples/).
2. In TuaGPT apri **Gestione Prompt** → Prompt Wizard → **Importa**.
3. Usa il template con **Inserisci Prompt** o **Assegna FN**.
4. All’uso, i placeholder `{Variabile}` vengono chiesti nel dialogo variabili.

## Esempi inclusi

| File | Uso tipico |
|------|------------|
| [`examples/GeneratoreArticoliBlogSEO.prompt`](examples/GeneratoreArticoliBlogSEO.prompt) | Added a new prompt for generating SEO blog articles, including metadata, content structure, and behavior rules. |
| [`examples/RevisoreTesti.prompt`](examples/RevisoreTesti.prompt) | Added a new prompt for text revision in Italian, including rules and examples. |
| [`examples/RicercatoreTematico.prompt`](examples/RicercatoreTematico.prompt) | Add Ricercatore Tematico prompt for structured research |
| [`examples/ScrittoreEmailProfessionale.prompt`](examples/ScrittoreEmailProfessionale.prompt) | Added a prompt for professional email writing in Italian, including metadata, content structure, and examples. |
| [`examples/ShoppingAdvisor.prompt`](examples/ShoppingAdvisor.prompt) | Added a new prompt for a Shopping Advisor that assists users in finding and comparing products online based on their needs. |
| [`examples/ShoppingAdvisorGuidato.prompt`](examples/ShoppingAdvisorGuidato.prompt) | Added a new prompt for a shopping advisor that utilizes variable data for guided web searches. |

## Struttura del repo

| Percorso | Contenuto |
|----------|-----------|
| `DOCUMENTAZIONE_PROMPT.md` | Specifica e guida redazionale |
| `examples/` | Template `.prompt` di esempio |

## Contribuire

- Un file = un template; non usare la categoria `DiSistema`.
- Ogni `{Variabile}` deve essere dichiarata in `<variables>`.
- Prima della PR: import di prova in TuaGPT + checklist in fondo alla specifica.

## Licenza

MIT

