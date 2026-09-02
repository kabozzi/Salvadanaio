# Il salvadanaio

Paghetta settimanale a gettoni. Pagina statica, nessun build, nessuna dipendenza da installare.

File nella cartella:

```
index.html              tutta l'app (CSS + JS inclusi)
manifest.webmanifest    per l'icona sulla schermata home
icon-192.png            icona
icon-512.png            icona
```

## 1. Provala subito

Apri `index.html` con doppio clic. Funziona già, con i dati salvati nel browser di quel
computer (`localStorage`). Serve per decidere attività e capricci prima di pubblicare.

## 2. Pubblicala su GitHub Pages

1. Nuovo repository su GitHub, ad esempio `salvadanaio`. **Privato va bene**: Pages
   funziona anche da repo privato con un account gratuito, e la pagina pubblicata resta
   raggiungibile da chi ha l'URL.
2. Carica i quattro file nella radice del repo (`Add file` › `Upload files`).
3. `Settings` › `Pages` › Source: `Deploy from a branch`, branch `main`, cartella `/ (root)`.
4. Dopo un minuto l'indirizzo è `https://<tuo-utente>.github.io/salvadanaio/`.

Sul telefono: apri l'indirizzo, poi `Condividi` › `Aggiungi alla schermata Home` (iPhone)
oppure menù `⋮` › `Aggiungi alla schermata Home` (Android). Si apre a tutto schermo con
l'icona del barattolo.

A questo punto l'app è online ma **i dati sono ancora separati per dispositivo**. Per la
sincronizzazione serve il passo 3.

## 3. Sincronizza fra i dispositivi con Firebase

### Sul sito di Firebase

1. Vai su `console.firebase.google.com` con il tuo account Google **personale** e crea un
   progetto. Disattiva Google Analytics, non serve.
2. `Build` › `Firestore Database` › `Crea database`. Scegli la regione `eur3` (Europa) e
   avvia in **modalità produzione**.
3. `Build` › `Authentication` › `Inizia` › scheda `Sign-in method` › abilita **Anonimo**.
4. Nella panoramica del progetto, icona `</>` per registrare un'app web. Ti mostra
   l'oggetto `firebaseConfig`: da lì ti servono quattro valori.
5. Torna in `Firestore Database` › scheda `Regole` e incolla queste, poi `Pubblica`:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /paghette/{id} {
      allow read, write: if request.auth != null;
    }
  }
}
```

### Nel file

Apri `index.html`, cerca in alto nello script il blocco `const FIREBASE` e riempi i quattro
campi con i valori del punto 4:

```js
const FIREBASE = {
  apiKey: "AIza...",
  authDomain: "salvadanaio-xxxx.firebaseapp.com",
  projectId: "salvadanaio-xxxx",
  appId: "1:123456789:web:abc123",
};
```

Ricarica il file su GitHub. Da qui in poi, al primo avvio su ogni dispositivo, l'app chiede
la **parola di famiglia**: la stessa parola apre lo stesso salvadanaio. Va scritta una volta
sola per dispositivo, e non viene mai spedita in chiaro — quello che finisce nel database è
solo il suo SHA-256 usato come nome del documento.

Da quel momento le spunte compaiono in tempo reale su tutti gli schermi aperti.

## Come stanno le cose sulla sicurezza

Le regole qui sopra dicono "chiunque sia autenticato può leggere e scrivere", e
l'autenticazione anonima la può ottenere chiunque. Quello che protegge i dati è il fatto che
il nome del documento è l'hash della vostra parola: senza quella parola non c'è modo di
indovinare dove guardare. Per la lista dei mestieri di casa è abbondantemente sufficiente,
ma è giusto che tu sappia che non è una vera autenticazione. Scegli una parola non banale e
non riusarla altrove.

## Il pallino accanto alla data

- **verde** — collegato, i dati sono condivisi e aggiornati
- **rosso** — connessione assente: puoi continuare a spuntare, resta salvato in locale, ma
  non si sincronizza (le modifiche fatte offline non vengono recuperate automaticamente al
  ritorno della rete: ricontrolla la giornata)
- **grigio** — Firebase non configurato, modalità solo locale

## Echo Show 8

"Alexa, apri Silk browser", poi apri l'indirizzo e salvalo tra i segnalibri, così le volte
successive lo trovi nella pagina iniziale del browser. Il touch funziona e i pulsanti sono
già dimensionati per il dito. Non si può tenere fissa a schermo: dopo un po' di inattività
il dispositivo torna alla sua schermata.

## Manutenzione

Firebase carica il proprio SDK dalla versione indicata in `VERSIONE_SDK` all'inizio dello
script. Se fra un anno vuoi aggiornarla, cambia solo quel numero.
