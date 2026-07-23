# TuaGPT Prompt Format

Formato ufficiale **`.prompt`** (XML 1.0) accettato da [TuaGPT](https://tuagpt.it): specifica, variabili e esempi importabili.

## Documentazione

La specifica completa è in **[DOCUMENTAZIONE_PROMPT.md](DOCUMENTAZIONE_PROMPT.md)** (contratto XML, variabili `{Nome}`, best practice, checklist).

Guida utente (sito): pagina Prompt / template sul sito TuaGPT.

## Uso rapido

1. Scarica o clona un file `.prompt` da `examples/` (se presente).
2. In TuaGPT apri **Gestione Prompt** → Prompt Wizard → **Importa**.
3. Usa il template con **Inserisci Prompt** o **Assegna FN**.
4. All’uso, i placeholder `{Variabile}` vengono chiesti nel dialogo variabili.

## Struttura del repo

| Percorso | Contenuto |
|----------|-----------|
| `DOCUMENTAZIONE_PROMPT.md` | Specifica e guida redazionale |
| `examples/` | Template `.prompt` di esempio (opzionale) |

## Contribuire

- Un file = un template; non usare la categoria `DiSistema`.
- Ogni `{Variabile}` deve essere dichiarata in `<variables>`.
- Prima della PR: import di prova in TuaGPT + checklist in fondo alla specifica.

## Licenza

[Indica qui la licenza del repo, es. # TuaGPT Prompt Format

Formato ufficiale **`.prompt`** (XML 1.0) accettato da [TuaGPT](https://tuagpt.it): specifica, variabili e esempi importabili.

## Documentazione

La specifica completa è in **[DOCUMENTAZIONE_PROMPT.md](DOCUMENTAZIONE_PROMPT.md)** (contratto XML, variabili `{Nome}`, best practice, checklist).

Guida utente (sito): pagina Prompt / template sul sito TuaGPT.

## Uso rapido

1. Scarica o clona un file `.prompt` da `examples/` (se presente).
2. In TuaGPT apri **Gestione Prompt** → Prompt Wizard → **Importa**.
3. Usa il template con **Inserisci Prompt** o **Assegna FN**.
4. All’uso, i placeholder `{Variabile}` vengono chiesti nel dialogo variabili.

## Struttura del repo

| Percorso | Contenuto |
|----------|-----------|
| `DOCUMENTAZIONE_PROMPT.md` | Specifica e guida redazionale |
| `examples/` | Template `.prompt` di esempio (opzionale) |

## Contribuire

- Un file = un template; non usare la categoria `DiSistema`.
- Ogni `{Variabile}` deve essere dichiarata in `<variables>`.
- Prima della PR: import di prova in TuaGPT + checklist in fondo alla specifica.

## Licenza 
MIT

