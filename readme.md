# Scheduler turni camerieri — Requisiti

## 1. Obiettivo

Generare la pianificazione settimanale dei turni dei camerieri di un ristorante,
coprendo gli orari di apertura e rispettando competenze, monte ore e disponibilità,
minimizzando il ricorso a coperture di emergenza e rispettando per quanto possibile
le preferenze.

## 2. Risorse: i camerieri

### 2.1 Tipi di cameriere

- **Con monte ore (a contratto):** hanno un numero di ore settimanali assegnato.
  Le ore vanno coperte (vincolo rigido, vedi 5) e preferibilmente non superate
  (preferenza, vedi 6).
- **A chiamata:** nessun limite minimo né massimo di ore. Lavorano in base alle
  disponibilità che dichiarano (vedi 4.3).

### 2.2 Competenze (attributi di ogni cameriere)

- **Apertura:** sì / no. Solo chi ha questa competenza può coprire l'apertura.
- **Chiusura fiscale:** sì / no. Serve almeno una persona con questa competenza
  a ogni chiusura.

Le due competenze sono indipendenti: un cameriere può avere una, entrambe o nessuna.

### 2.3 Ruoli speciali

- **Caposala:** un solo cameriere. È uno dei camerieri **a monte ore**: valgono per
  lui tutte le relative regole, con l'unica eccezione dei giorni di riposo (vedi
  vincolo 10). È preferibile assegnarlo al servizio di pranzo quando possibile
  (preferenza, non obbligo).
- **Datore di lavoro:** un solo soggetto. Disponibile solo da lunedì a venerdì,
  può coprire esclusivamente il turno giornaliero. Funge da riserva per coprire
  le mancanze: da usare solo quando necessario.

### 2.4 Roster effettivo (da confermare)

Ricavato dagli orari delle settimane di esempio (colonna sala; cucina esclusa).
Le competenze sono ipotesi da confermare.

| Nome      | Tipo                 | Apertura | Chiusura fiscale | Note                                  |
|-----------|----------------------|----------|------------------|---------------------------------------|
| Eugi      | Caposala (monte ore) | sì       | da confermare    | apre il lunedì alle 07; spesso a pranzo |
| Erica     | Monte ore            | sì       | da confermare    | fa sia aperture sia chiusure          |
| Ale       | Datore di lavoro     | sì       | da confermare    | solo lun–ven, diurno (es. 09–12/13)   |
| Gianluca  | A chiamata           | da conf. | da confermare    | prevalentemente sera                  |
| Martin    | A chiamata           | da conf. | da confermare    | pranzo / pomeriggio / sera            |
| Skoke     | A chiamata           | da conf. | da confermare    | pomeriggio / sera                     |
| Regina    | A chiamata           | da conf. | da confermare    | sera (occasionalmente mattina)        |
| Alice     | A chiamata           | da conf. | da confermare    | sera                                  |

Quindi due camerieri a monte ore (Eugi, che è il caposala, ed Erica), il Datore di
lavoro e cinque a chiamata.

## 3. Struttura temporale

Il tempo è modellato a **slot** (fasce di durata fissa, es. 1 ora). Il conteggio
delle ore lavorate, rilevante per il monte ore, si ottiene sommando gli slot
assegnati a ciascun cameriere.

### 3.1 Finestre di apertura da coprire

| Giorno              | Apertura | Chiusura   |
|---------------------|----------|------------|
| Lunedì              | 07:00    | 24:00      |
| Martedì             | 09:00    | 24:00      |
| Mercoledì           | 09:00    | 24:00      |
| Giovedì             | 09:00    | 24:00      |
| Venerdì             | 09:00    | 01:00 (+1) |
| Sabato              | 09:00    | 01:00 (+1) |
| Domenica            | 09:00    | 24:00      |

Note: il lunedì l'apertura è anticipata alle 07:00; venerdì e sabato la chiusura
slitta all'01:00 del giorno successivo.

### 3.2 Fasce di servizio

- **Servizio serale (cena):** dalle 19:00 alla chiusura.
- **Servizio giornaliero (giorno/pranzo):** dall'apertura alle 19:00.

### 3.3 Vincoli sull'orario di inizio (camerieri a chiamata)

Un cameriere a chiamata dichiara la disponibilità per pranzo e/o per cena:

- disponibilità **pranzo** → può essere assegnato, in alternativa, a uno di questi
  due turni:
  - **turno di pranzo:** inizio alle **12:00 o alle 13:00**, fine entro le **17:00**;
  - **turno di mezzo** (pomeriggio): dalle **15:00**, fine preferibilmente alle
    **18:00** e al massimo alle **19:00**, per coprire la fascia centrale quando a
    pranzo non serve;
- disponibilità **cena** → il turno termina sempre alla chiusura e inizia di norma
  dalle **18:00**, ma può essere anticipato fino alle **17:00** se necessario a
  coprire un buco (venerdì/sabato 17:00–01:00 sono esattamente 8 ore, cioè il primo
  inizio possibile). Vale comunque il limite di 8 ore consecutive.

## 4. Vincoli rigidi (hard) — devono valere sempre

Il modello deve rispettarli tutti. I vincoli contrassegnati con **†** possono essere
infranti solo come ultima risorsa e solo per il **caposala** e il **datore di lavoro**,
secondo la politica della sezione 9; per tutti gli altri camerieri restano sempre
inviolabili.

1. **Copertura oraria:** l'intera finestra di apertura di ogni giorno deve essere
   coperta da personale.
2. **Apertura qualificata:** chi copre la **prima ora** della giornata deve avere
   la competenza di apertura (è questo lo slot che definisce l'apertura).
3. **Chiusura fiscale:** chi copre l'**ultima ora** della giornata deve avere la
   competenza di chiusura fiscale (è questo lo slot che definisce la chiusura).
4. **Copertura minima:** almeno 1 persona durante il servizio giornaliero
   (dall'apertura alle 19:00) e almeno 2 persone durante il servizio serale
   (dalle 19:00 alla chiusura). Sono valori di default sovrascrivibili per singolo
   servizio (vedi 8).
5. **Ore consecutive †:** nessun cameriere può lavorare più di 8 ore consecutive.
6. **Monte ore minimo:** per i camerieri a contratto le ore settimanali assegnate
   devono essere coperte.
7. **Inizio turno a chiamata:** rispetto degli orari di inizio/fine in base alla
   disponibilità dichiarata (vedi 3.3).
8. **Disponibilità Datore di lavoro †:** solo lunedì–venerdì e solo turno giornaliero.
9. **Niente turni spezzati:** ogni cameriere lavora un unico blocco continuo al
   giorno (no pranzo + pausa + rientro a cena).
10. **Giorni di riposo settimanali †:**
    - camerieri a monte ore: almeno 2 giorni di riposo;
    - caposala (che è a monte ore): almeno 1 giorno di riposo, in deroga alla regola
      precedente;
    - camerieri a chiamata: nessun vincolo.
11. **Fine del servizio serale †:** chi lavora la sera termina sempre alla chiusura.
12. **Chiusura seguita da apertura:** chi effettua la chiusura può fare l'apertura
    del giorno successivo solo se è a monte ore; per i camerieri a chiamata è vietato.

## 5. Preferenze (soft) — da inserire nell'obiettivo

- **Caposala a pranzo** quando possibile.
- **Non superare il monte ore** dei camerieri a contratto.
- **Minimizzare l'uso del Datore di lavoro** (usarlo solo per coprire mancanze
  altrimenti scoperte).
- **Dare almeno un turno** a ogni cameriere a chiamata che ha dato disponibilità.
- **Evitare chiusura + apertura il giorno dopo** anche per i camerieri a monte ore,
  quando possibile.
- **Limitare la fascia 18:00–19:00** a una sola persona: nell'ora che precede il
  servizio serale è preferibile avere al massimo un cameriere.
- **Turno di mezzo entro le 18:00:** preferibile farlo terminare alle 18:00,
  estendendolo fino alle 19:00 solo se necessario.
- **Cena dalle 18:00:** preferibile far iniziare il turno di cena dalle 18:00,
  anticipandolo alle 17:00 solo se serve a coprire una scopertura.

## 6. Obiettivo di ottimizzazione

L'obiettivo è **gerarchico** (lessicografico, o a penalità di peso molto diverso),
in quest'ordine di priorità:

1. minimizzare le fasce lasciate scoperte (sezione 9);
2. minimizzare le deroghe a carico di caposala e datore di lavoro (sezione 9);
3. ottimizzare le preferenze soft (sezione 5).

Resta da definire l'ordine di priorità tra le singole preferenze soft quando entrano
in conflitto tra loro.

## 7. Punti da chiarire / decisioni aperte

- **Durata dello slot:** 30 o 60 minuti? Tutti gli orari in gioco (07:00, 12:00,
  13:00, 17:00, 18:00, 19:00, 24:00, 01:00) sono multipli dell'ora, quindi lo slot
  da 60 minuti è probabilmente sufficiente.
- **Turno giornaliero del Datore di lavoro:** dagli esempi sembra mattina–primo
  pomeriggio (09–12, 09–13). Confermare l'intervallo massimo consentito.
- **Competenze del roster:** chi ha la competenza di apertura e chi quella di
  chiusura fiscale (vedi tabella 2.4, colonne da confermare).
- **Ore monte ore:** quante ore settimanali per Eugi e per Erica.
- **Orizzonte:** una settimana singola o pianificazione ricorrente?

## 8. Funzionalità future (post-MVP)

- **Minimi di copertura per singolo servizio:** poter sovrascrivere il numero minimo
  di camerieri per uno specifico pranzo o una specifica cena (es. "questo sabato a
  cena almeno 4"), oltre ai default del punto 4. Conviene quindi modellare i minimi
  come un dato per (giorno, servizio), non come costanti globali.

## 9. Gestione della carenza di personale (politica di rilassamento)

Il modello deve sempre rispettare i vincoli rigidi. Quando il personale disponibile
non basta a produrre un orario valido, invece di non restituire nulla il sistema:

1. **segnala la carenza:** indica quali fasce/turni non si riescono a coprire
   rispettando tutte le regole;
2. **propone comunque un orario completo**, infrangendo alcuni vincoli ma solo per il
   **caposala** (Eugi) e il **datore di lavoro** (Ale).

Gerarchia delle soluzioni, dalla migliore alla peggiore (è l'obiettivo della sez. 6):

1. **nessuna deroga:** orario pulito, tutti i vincoli rispettati;
2. **caposala/datore oltre i limiti:** si infrangono i vincoli † della sezione 4 a
   carico solo di Eugi e Ale (più ore consecutive, datore oltre la sua finestra,
   caposala senza il giorno di riposo) per coprire i buchi;
3. **fasce scoperte:** se nemmeno questo basta, alcune fasce restano vuote. È la vera
   carenza di organico da segnalare (serve assumere o chiamare qualcuno).

Vincoli **mai** derogabili, per nessuno: competenze (apertura, chiusura fiscale),
disponibilità dichiarate dagli a chiamata, divieto di turni spezzati. Le deroghe non
si applicano **mai** ai camerieri a chiamata né a Erica (monte ore ma non caposala).

Questo spiega anche gli orari storici: Eugi a 11 ore il lunedì e Ale a tappare i
buchi erano esattamente deroghe di tipo 2, usate informalmente per sopperire alla
mancanza di personale.

Da definire: l'elenco esatto dei vincoli derogabili e di quanto (es. fino a quante ore
consecutive può arrivare il caposala; se il datore può lavorare anche di sera/weekend).