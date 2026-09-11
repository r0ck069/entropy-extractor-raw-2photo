# Entropy Extractor — RAW 2-Photo (Dark Frame)

Estrattore di entropia locale (nessuna dipendenza esterna, nessun invio di dati) che
ricava bit casuali dal rumore elettronico di un sensore fotografico, partendo da
**due dark frame** (due scatti identici a prima vista, fatti con obiettivo
tappato/otturatore chiuso, stesse impostazioni). La differenza pixel-per-pixel fra
i due scatti cancella per sottrazione il rumore a pattern fisso del sensore (pixel
caldi, dark current), lasciando prevalentemente il rumore casuale residuo
(shot/read noise), da cui vengono estratti bit tramite una pipeline crittografica
rigorosa (debiasing di Peres + estrattore forte di Toeplitz), con stima
dell'entropia, gate di sicurezza e test statistici.

Il file è una singola pagina HTML autosufficiente: apri `entropy-extractor-raw-2photo.html`
in un browser moderno (Chrome/Firefox/Edge aggiornati). Nessuna installazione,
nessun server, nessuna libreria esterna: tutto il calcolo avviene nel browser
dell'utente e i file caricati non lasciano mai il dispositivo.

## Uso rapido

1. Apri il file in un browser.
2. Spunta la dichiarazione "i due dark frame sono fotografie uniche e
   autoprodotte" (obbligatoria: sblocca gli input).
3. Carica **Dark frame 1** e **Dark frame 2**: due scatti fatti con la stessa
   fotocamera, stesse impostazioni (esposizione, ISO, temperatura), obiettivo
   tappato o otturatore chiuso, idealmente ravvicinati nel tempo.
4. Al caricamento del Dark frame 1 il tool tenta il rilevamento automatico del
   blocco dati del sensore (funziona per RAW basati su TIFF: DNG e molte
   CR2/NEF/ARW/ORF/PEF). Se il rilevamento fallisce, imposta manualmente
   offset/lunghezza/ordine byte.
5. Imposta un seed (pubblico, non deve essere segreto: viene precompilato a
   caso ad ogni apertura della pagina) e i bit di output desiderati per
   finestra.
6. Premi "AVVIA ANALISI DARK FRAME".
7. Leggi il pannello **"Confronto fra le finestre e criterio di scelta"** per
   capire quale finestra è stata scelta come migliore e perché (vedi sotto).
8. Scarica il buffer binario della/e finestra/e che ti interessano, o calcola
   l'hash SHA-256 finale della finestra scelta come migliore.

## Come funziona, in breve

1. **Sorgente**: la differenza `dark1 − dark2`, parola per parola (16 bit),
   sul blocco dati grezzo del sensore.
2. **Filtro**: vengono esclusi i campioni-differenza uguali a 0 (nessuna
   informazione: i due scatti hanno letto lo stesso valore) e i due estremi
   con segno ±32767/−32768 (probabile overflow/clipping). *Non* vengono
   esclusi valori come diff=±1, che sono rumore normale (vedi CHANGELOG per
   il perché questo è importante).
3. **Debiasing**: estrattore di Peres (1992), efficienza tipica ~95% su
   sorgenti eque, molto superiore al von Neumann classico (~25%).
4. **Stima dell'entropia**: due stimatori indipendenti — Most-Common-Value e
   Collision Estimate — entrambi con limite di confidenza superiore di
   Clopper-Pearson esatto al 99%. Si usa il minimo dei due (più conservativo).
5. **Gate obbligatori** prima di procedere: lunghezza minima post-debiasing,
   min-entropia minima della sorgente, almeno un blocco Toeplitz completo. Se
   un gate fallisce, quella finestra non produce output "creato dal nulla":
   viene segnalata come fallita.
6. **Estrazione forte**: matrice di Toeplitz (Leftover Hash Lemma), con
   rapporto di estrazione dimensionato dinamicamente sull'entropia realmente
   misurata (mai un valore fisso).
7. **Finestre indipendenti**: il blocco dati viene diviso in tre finestre
   spaziali (Inizio / Centro / Fine) più una quarta di controllo (Blocco
   intero, unione delle altre tre — esclusa per costruzione perché non
   indipendente). Le finestre 1/2/3 vengono anche testate fra loro per
   correlazione incrociata (multi-lag): se due risultano correlate, entrambe
   vengono escluse dal confronto (dati comunque mostrati per trasparenza).
8. **Test statistici**: 4 test della suite NIST SP 800-22 (Monobit, Block
   Frequency, Runs, Longest Run of Ones) sull'output finale di ciascuna
   finestra.

### Criterio di scelta della finestra "migliore"

Fra le finestre **indipendenti** (non escluse per correlazione) e **senza
gate falliti**, il tool assegna un punteggio:

```
punteggio = (test NIST superati × 10) + min-entropia finale stimata (bit/bit) + (bit prodotti ÷ 1000)
```

Viene evidenziata quella con il punteggio più alto. In pratica, in ordine di
importanza: prima quanti test statistici supera, poi quanto è alta l'entropia
stimata, infine quanti bit produce. Il pannello "Confronto fra le finestre"
mostra il punteggio e lo stato di ciascuna finestra, così il motivo
dell'esclusione delle altre è sempre visibile — non è un'etichetta
"vincente/perdente" senza spiegazione.

## Limiti e cose da sapere

- **Non sostituisce una suite di test di casualità completa.** I 4 test NIST
  qui inclusi sono un sottoinsieme dei 15 della batteria SP 800-22, e le
  stime di min-entropia (MCV + Collision) sono un sottoinsieme della batteria
  NIST SP 800-90B (mancano, fra gli altri, Markov, compressione, t-Tuple,
  LRS, LZ78Y).
- **Il seed è pubblico per costruzione** (proprietà del Leftover Hash Lemma):
  non deve essere tenuto segreto. Ciò che deve restare segreto/unico è il
  file RAW sorgente stesso.
- **Se i file RAW sono pubblici, noti o riproducibili da terzi, l'entropia
  crittografica reale è vicina a zero**, indipendentemente da qualunque test
  qui presente. Da qui la dichiarazione obbligatoria di unicità/autoproduzione.
- **Formati RAW compressi internamente** (lossless-JPEG/Huffman, comune in
  diverse CR2/NEF/ARW) fanno sì che le statistiche misurate riflettano anche
  l'algoritmo di compressione, non solo il rumore fisico del sensore. Quando
  possibile, usa un RAW non compresso ("Uncompressed RAW"/"Lossless
  Uncompressed" nelle impostazioni della fotocamera).
- **Rilevamento automatico del blocco dati**: funziona solo per contenitori
  basati su TIFF (DNG e molte CR2/NEF/ARW/ORF/PEF). Per CR3, RAF, alcuni RW2
  il rilevamento fallisce: va impostato manualmente offset/lunghezza.
- **File molto grandi (>10 MB)**: non vengono caricati per intero in memoria.
  Viene letta solo una porzione di 10 MB a partire dal 40% del file. In
  questo caso l'header TIFF non è nella porzione letta, quindi l'auto-
  rilevamento viene saltato e l'intera porzione prelevata è trattata
  direttamente come blocco dati (segnalato chiaramente a schermo).
- **I due dark frame devono avere la stessa struttura** (stessa fotocamera,
  stesse impostazioni): offset/lunghezza/ordine byte impostati per il primo
  file vengono applicati anche al secondo.
- **Modalità rigorosa (default, consigliata)**: se una finestra produce meno
  bit del target richiesto, vengono restituiti solo i bit realmente estratti
  — nessuna espansione sintetica silenziosa. Disattivandola, i bit mancanti
  vengono completati con un'espansione HKDF (RFC 5869), chiaramente
  etichettata come "sintetica" nell'output.
- **Uso raccomandato**: solo come una delle sorgenti di entropia in un
  progetto più ampio, con conditioning esterno aggiuntivo se i gate risultano
  borderline. Non usare l'output come unico seed crittografico ad alta
  garanzia senza ulteriori verifiche indipendenti.

## Autotest all'avvio

Ad ogni apertura della pagina viene eseguita in automatico una suite di
autotest di regressione, visibile nel pannello dedicato:

- SHA-256, HMAC-SHA256 (RFC 4231), HKDF (RFC 5869) contro vettori di test
  ufficiali;
- Clopper-Pearson (auto-consistenza + valore di riferimento noto);
- conservazione della massa ed efficienza del Peres Extractor su sorgenti
  deterministiche;
- coerenza del Toeplitz hashing contro una seconda implementazione
  indipendente;
- coerenza della Collision Estimate su dati fortemente equi;
- **filtro diff con segno**: verifica che vengano esclusi solo diff=0 e i
  due estremi ±32767/−32768, e che diff=±1 venga invece mantenuto;
- **cancellazione del pattern fisso**: verifica che la sottrazione
  dark1−dark2 isoli esattamente un rumore sintetico noto, cancellando un
  pattern deterministico sovrapposto.

Se un autotest fallisce, il pannello lo segnala in rosso: in quel caso non
fidarsi dei risultati della pipeline (probabile regressione nel codice).

## Licenza

Vedi [LICENSE](LICENSE).

## Changelog

Vedi [CHANGELOG.md](CHANGELOG.md).
