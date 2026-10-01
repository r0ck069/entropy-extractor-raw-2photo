# Entropy Extractor — RAW 2-Photo (Dark Frame)

> ⚠ **v4.0.0-beta4 — BETA pubblicata su GitHub.** Contiene funzionalità
> nuove non ancora sottoposte ad audit indipendente da terzi (sono però state
> verificate con una suite di test rigorosa e reale — vedi `CHANGELOG.md`). Vedi
> `SECURITY-NOTES.md` per i principi di design.

Estrattore di entropia locale (nessuna dipendenza esterna, nessun invio di dati) che
ricava bit casuali dal rumore elettronico di un sensore fotografico, partendo da
**due dark frame** (due scatti identici a prima vista, fatti con obiettivo
tappato/otturatore chiuso, stesse impostazioni). La differenza pixel-per-pixel fra
i due scatti cancella per sottrazione il rumore a pattern fisso del sensore (pixel
caldi, dark current), lasciando prevalentemente il rumore casuale residuo
(shot/read noise), da cui vengono estratti bit tramite una pipeline crittografica
rigorosa (debiasing di Peres + estrattore forte di Toeplitz), con stima
dell'entropia, gate di sicurezza e test statistici.

Il file è una singola pagina HTML autosufficiente: apri
`entropy-extractor-raw-2photo.html` in un browser moderno. Nessuna
installazione, nessun server, nessuna libreria esterna.

## Uso rapido

1. Apri il file in un browser.
2. Spunta la dichiarazione "i due dark frame sono fotografie uniche e
autoprodotte" (obbligatoria: sblocca gli input) — oppure premi **Demo
sbilanciata (test)** per generare con CSPRNG reale due dark frame sintetici la
cui differenza dovrebbe far scattare i gate di sicurezza (mai dark frame reali).
3. Carica **Dark frame 1** e **Dark frame 2**: due scatti fatti con la stessa
fotocamera, stesse impostazioni, obiettivo tappato o otturatore chiuso.
4. Al caricamento del Dark frame 1 il tool tenta il rilevamento automatico del
blocco dati del sensore. Se fallisce, imposta manualmente offset/lunghezza/ordine
byte.
5. Imposta un seed (pubblico), i bit di output desiderati per finestra, e il
**margine di sicurezza LHL** (ε=2⁻ᵏ, default k=40 — scendere sotto richiede
conferma esplicita; ε è l'errore totale ammesso sull'uscita della finestra, ripartito tra i
blocchi Toeplitz come ε/B, vedi `SECURITY-NOTES.md`, Principio 2).
6. Premi "AVVIA ANALISI DARK FRAME".
7. Leggi il pannello **"Confronto fra le finestre e criterio di scelta"** per
capire quale finestra è stata scelta come migliore e perché.
8. Scarica il buffer binario della/e finestra/e che ti interessano, o calcola
l'hash SHA-256/SHAKE256 finale della finestra scelta come migliore.
9. Consulta il pannello di **integrità dell'applicazione** (hash SHA-256 del
codice core, mostrato al caricamento).

## Come funziona, in breve

1. **Sorgente**: la differenza `dark1 − dark2`, parola per parola (16 bit).
2. **Filtro**: esclusi i campioni-differenza uguali a 0 e i due estremi con segno
±32767/−32768. *Non* esclusi valori come diff=±1 (rumore normale).
3. **Debiasing**: estrattore di Peres (1992), efficienza tipica ~95% su sorgenti eque.
4. **Stima dell'entropia**: MCV + Collision Estimate, entrambi con Clopper-Pearson
esatto al 99%, più un gate strutturale dedicato con t-Tuple/LRS e due health test
retrospettivi (RCT/APT).
5. **Gate obbligatori** prima di procedere, incluso un tetto di
emissione ⌊512/2⌋=256 bit per blocco Toeplitz, indipendente dal bound LHL per blocco
(il cui margine ε è ora selezionabile, vedi sopra).
6. **Estrazione forte**: matrice di Toeplitz (Leftover Hash Lemma), rapporto
dimensionato dinamicamente, per ogni blocco, sull'entropia realmente misurata.
7. **Finestre indipendenti**: Inizio / Centro / Fine del blocco dati + Blocco
intero (escluso per costruzione), con controllo di correlazione incrociata.
8. **Test statistici**: 9 test della suite NIST SP 800-22, cioè 8 procedure
(Frequency, Block Frequency, Runs, Longest Run, Serial completo, Approximate
Entropy completa, Cumulative Sums diretto e inverso — che contano come una sola
procedura —, Binary Matrix Rank).

### Criterio di scelta della finestra "migliore"

```
punteggio = (test NIST superati × 10) + min-entropia finale stimata (bit/bit) + (bit prodotti ÷ 1000)
```

## Limiti e cose da sapere

- **Non sostituisce una suite di test di casualità completa.**
- **Il seed è pubblico per costruzione**: non deve essere tenuto segreto. Ciò che
deve restare segreto/unico è il file RAW sorgente stesso.
- **Se i file RAW sono pubblici, noti o riproducibili da terzi, l'entropia
crittografica reale è vicina a zero.** Da qui la dichiarazione obbligatoria di
unicità/autoproduzione.
- **Formati RAW compressi internamente** fanno sì che le statistiche misurate
riflettano anche l'algoritmo di compressione. Quando possibile, usa un RAW non
compresso.
- **Rilevamento automatico del blocco dati**: funziona solo per contenitori
basati su TIFF. Per CR3, RAF, alcuni RW2 va impostato manualmente.
- **File molto grandi (>10 MB)**: viene letta solo una porzione di 10 MB a
partire dal 40% del file.
- **Modalità rigorosa (default, consigliata)**: nessuna espansione sintetica
silenziosa.
- **Non ancora incluso in questa beta** (vedi CHANGELOG.md): bit-plane
arbitrario, card di fusione best-2, selezione per varianza, campionamento da
posizioni prime, pipeline di confronto diagnostica, assistente CSV/sensore —
tutte già valutate e rimandate con motivazione esplicita.

## Autotest all'avvio

SHA-256, HMAC-SHA256, HKDF contro vettori ufficiali; Clopper-Pearson;
conservazione della massa ed efficienza del Peres Extractor; coerenza del
Toeplitz hashing; coerenza della Collision Estimate; filtro diff con segno;
cancellazione del pattern fisso. Più un pannello dedicato con l'hash di
integrità dell'applicazione (controllo diagnostico, non un autotest pass/fail).

## File del repository

- `entropy-extractor-raw-2photo.html` — l'applicazione completa, build v4.0.0-beta4 (apribile con doppio click, nessuna installazione richiesta).
- `CHANGELOG.md` — storia delle versioni e delle correzioni, con gli SHA-256 delle build.
- `AUDIT-NOTES.md` — note di audit (punti 1-12).
- `SECURITY-NOTES.md` — principi di sicurezza e di design.
- `lrs_check.js`, `raw2photo_check.js`, `toeplitz_margin_check.js` — test di regressione in Node.js delle tre correzioni successive alla v3 (stimatore LRS, stima dell'entropia prima di Peres, bound LHL per blocco); si lanciano dalla cartella del repository con `node <file>`.
- `lhl_exact_check.py` — verifica numerica esatta del lemma dell'hash residuo su famiglie Toeplitz piccole (richiede Python 3 e numpy).
- `LICENSE` — licenza MIT.

## Stato di verifica

Dopo la v3 sono state corrette tre cose, descritte in `CHANGELOG.md` e `AUDIT-NOTES.md` (punti 10-12): la stima dell'entropia calcolata prima di Peres e non dopo (27/09/2026, dall'audit indipendente), la lunghezza di confronto dello stimatore LRS (27/09/2026) e il bound del lemma dell'hash residuo, ora calcolato per ogni blocco Toeplitz con somma sui blocchi (01/10/2026). Le tre correzioni sono verificate con script e non da un audit indipendente; le funzionalità nuove della beta non sono ancora state sottoposte ad audit indipendente. Il bound del lemma vale sotto ipotesi esplicite (vedi `SECURITY-NOTES.md`, Principio 2).

## Licenza

Vedi [LICENSE](LICENSE).

## Changelog

Vedi [CHANGELOG.md](CHANGELOG.md). Note d'audit dettagliate (versioni
precedenti): [AUDIT-NOTES.md](AUDIT-NOTES.md). Principi di design:
[SECURITY-NOTES.md](SECURITY-NOTES.md).

## Per la raccolta delle sorgenti

APP consigliate per la raccolta dei campioni da processare: RØDE Reporter, sensor
logger, registratore Audio di Hardcoded Joy, RecForge II.

Uno o due telefoni cellulari, una macchina fotografica che abbia la possibilità di salvare
le foto in RAW, una o meglio due radio FM economiche a batterie, un dado in buone condizioni,
una moneta possibilmente in buone condizioni e bilanciata (da 2 euro esce certificata dalla
Zecca di Stato), 8 numeri della tombola, un file zip autocreato offline di qualche mega e
poi distrutto.Con il microfono del telefono ed escludendo i filtri in ingresso, raw, mono 
(queste app lo fanno), già due o più minuti di audio in un bar frequentato o in una mensa
genera un audio con tanto materiale difficilmente prevedibile, buono da estrarre. O anche
il campionamento dal sensore del giroscopio o magnetometro o accelerometro in una strada con
buche e dossi, può esserci imprevedibilità nei bit estratti — l'assistente CSV/sensore di
EntropyPipeline (non incluso in questo strumento) è pensato proprio per questo caso d'uso. L'importante è prendere sorgenti
grezze campionate che fra loro non abbiano correlazioni apparenti: il segnale audio di due
radio FM a batterie sintonizzate fuori frequenza, una foto completamente nera fatta in raw
tappando l'obiettivo, un giroscopio e il rumore in un bar o una mensa affollata hanno ben
poco in comune. Basta che una sola tra le sorgenti scelte abbia "qualità entropica", meglio
se presi in autonomia con 2 telefoni diversi ed una reflex.

