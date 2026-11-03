# AGENTS.md — StudyBuddy

App web (.NET 10 + SQLite + HTML/JS statico) per studiare con flashcard e quiz a risposta multipla.
Progetto didattico **volutamente rotto**: bug, codice morto, problemi di sicurezza, documentazione imprecisa, nessun test. **Non correggere nulla senza richiesta esplicita.**

## Avvio

```bash
cd src/StudyBuddy.Web && dotnet run   # → http://localhost:5200
```
Porta 5200 (non 5000). Il DB SQLite (`database/studybuddy.db`) nasce da `EnsureCreated()` + seed: **non esistono migration**. Le credenziali di prova stanno in `Data/Seed.cs`: non riportarle altrove.

## File principali

| File | Ruolo |
|---|---|
| `Program.cs` | DI, seed, ~10 endpoint (il file più pesante) |
| `Controllers/QuizController.cs` | `/api/quiz/*` |
| `Endpoints/{Stats,Admin}Endpoints.cs` | `/api/stats`, `/api/admin/*` |
| `Services/QuizService.cs` | logica quiz (valutazione risposte) |
| `Services/DeckService.cs` | lista mazzi con conteggi |
| `Data/{StudyDbContext,Seed,DbHelper}.cs` | EF Core, dati demo, SQL raw |
| `Models/`, `Dtos/` | entità e contratti JSON |
| `wwwroot/*.html` + `wwwroot/js/*.js` | frontend, una pagina per funzionalità |

**Modello:** `User` **non** è collegato a `Deck`/`Card` → non esiste la proprietà, tutti gli utenti vedono e modificano tutto. `Questions` e `Results` non hanno chiavi esterne.

## Errori noti (elencati, **non corretti**)

**Sicurezza** — SQL injection in `Data/DbHelper.cs:46` · password in chiaro (`User.cs`, `Program.cs:37`) · nessuna autenticazione/autorizzazione · `/api/admin/users` espone le password (`AdminEndpoints.cs:10`) · `/api/quiz/{deckId}/all` svela `rispostaCorretta` (`QuizController.cs:36`) · nessun controllo di proprietà (`Program.cs:114`) · XSS via `innerHTML` · validazione assente lato server.

**Logica** — i risultati dei quiz finiscono in **un DB diverso** da quello letto da `/api/stats` (`QuizController.cs:52`), quindi le statistiche sono sempre vuote · punteggio con divisione intera (`QuizService.cs:70`) · divisione per zero se le risposte sono vuote · l'ultima carta non viene mai mostrata (`study.js:50`) · off-by-one nella risposta corretta (`deck.html:34-39`) · timer e `NumeroDomande` mai implementati · `Box` Leitner non limitato (`Program.cs:126`) · Leitner non usato in `study.js` · seed non idempotente sulle domande (`Seed.cs:16`) · commenti che descrivono comportamenti inesistenti (`Program.cs:72`, `:85`).

**Frontend** — porta hardcoded in `stats.js:2` · `api.js` ignorato quasi ovunque · `shuffle()` muta l'array (`utils.js:21`) · nessuna gestione degli errori di rete · stili inline mescolati al CSS.

**Doc vs codice** — non esistono Repository pattern, JWT, import CSV, Leitner funzionante né migration. `docs/architettura.md` e i README descrivono cose inesistenti: aggiornali quando correggi il codice.

## Non toccare / ignorare

- **Codice morto**: `Legacy/OldQuizEngine.cs`, `js/legacy.js`, `Tools/StringUtils.cs`, `DeckDto`, `DeckService.{GetDeck,DeleteDeck,OttieniMazzi}`, `Helpers.{CalcolaPercentuale,Giudizio,Capitalize,IsNullOrEmpty}`, `utils.js:mischia()`, pacchetto `Newtonsoft.Json`.
- **Da non tracciare**: `bin/`, `obj/`, `.vs/`, `database/`, e `src/StudyBuddy.Web/studybuddy.db` (artefatto del bug sul DB sbagliato, non è in `.gitignore` → rimuoverlo a mano).

## Regole

1. Verifica tutto: apri il file, builda, avvia, chiama l'API. I commenti e la doc sbagliano.
2. Ogni modifica deve essere spiegabile; se non sai perché è giusta, non è finita.
3. Se correggi un errore elencato sopra, aggiorna anche questa lista e la documentazione.
