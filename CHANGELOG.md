# Changelog

Tutte le date fanno riferimento alla build indicata in cima al file HTML.

## v3 — 2026-09-11

Portati da `EntropyPipeline` (stesso autore, stesso repository owner), lo
strumento "manuale" con incolla-bit da cui questo tool eredita il motore
matematico di base: quattro elementi che lì esistevano ma qui mancavano.

- **SHAKE256** (Keccak-f[1600] puro JS, funzione XOF a lunghezza variabile):
  `crypto.subtle` del browser non lo implementa nativamente. Verificato
  contro i vettori di test ufficiali NIST prima dell'integrazione. Aggiunto
  un pulsante opzionale accanto al digest SHA-256 sulla finestra scelta come
  migliore, a parità di lunghezza di output.
- **t-Tuple e Longest Repeated Substring (LRS)** (SP 800-90B §6.3): stimatori
  di min-entropia di ordine superiore, capaci di rilevare strutture
  periodiche/ripetute nel rumore residuo che MCV e Collision (statistiche di
  ordine 0/1) non vedono per costruzione.
- **Repetition Count Test (RCT) e Adaptive Proportion Test (APT)**
  (SP 800-90B §4.4): health test retrospettivi applicati ai bit LSB grezzi di
  ciascuna finestra, prima del debiasing di Peres.

Tre adattamenti espliciti, resi necessari dal contesto diverso (flussi
automatici di centinaia di migliaia di bit per finestra, contro le
centinaia/migliaia di bit incollati a mano della sorgente originale):

1. **t-Tuple/LRS limitati a un campione di 20.000 bit per finestra**
   (`TTUPLE_LRS_SAMPLE_CAP`), per restare entro un tempo di calcolo
   ragionevole; la dimensione del campione è sempre mostrata nello schema
   numerico.
2. **Bug di scalabilità reale trovato e corretto durante il porting**: né
   t-Tuple né LRS, nella sorgente originale, limitano la lunghezza massima di
   pattern esaminata. Su una sequenza periodica il costo può esplodere a
   diversi miliardi di operazioni — riprodotto: il porting iniziale ha
   bloccato la pipeline di test per minuti su un singolo caso periodico non
   patologico in senso stretto (un pattern realistico da artefatto di
   sensore/codifica). Corretto con due limiti (`TTUPLE_MAX_T=64`,
   `LRS_MAX_SEARCH_LEN=256`), verificati non alterare il comportamento su
   rumore reale (dove la più lunga sottostringa ripetuta scala ~O(log2 n),
   ben sotto i limiti) né la rilevazione su sequenze periodiche/strutturate
   (la ripetizione resta visibile già entro il limite).
3. **t-Tuple/LRS non entrano nel calcolo della min-entropia che determina il
   rapporto di estrazione Toeplitz** (a differenza della sorgente originale,
   dove `combinedHmin` è il minimo fra tutti gli stimatori inclusi questi
   due). Scoperto empiricamente durante il testing: su campioni di decine di
   migliaia di bit, LRS in particolare resta strutturalmente intorno a
   0.35-0.43 bit/bit anche su rumore CSPRNG perfettamente equo (natura
   conservativa nota del limite di confidenza di Clopper-Pearson su un
   conteggio quasi sempre minimo, non un segnale reale di scarsa entropia).
   Includerlo nel minimo avrebbe dimezzato il rapporto di estrazione pratico
   su ogni analisi, anche perfettamente sana. Restano invece un **gate
   strutturale dedicato** (soglie calibrate con ampio margine empirico:
   sorgenti sane/moderatamente sbilanciate osservate a LRS≥0.21, t-Tuple≥0.29;
   sorgenti periodiche/strutturate osservate a LRS≤0.015, t-Tuple≤0.05).

RCT/APT sono invece applicati senza campionamento (costo O(n)) come gate
aggiuntivo per finestra, con la stessa priorità "il più pessimista vince"
degli altri gate.

Verificato con una suite di test end-to-end in Node.js prima del rilascio:
unit test sui singoli stimatori, l'intera suite di autotest della pagina, e
simulazioni complete della pipeline con sorgenti sintetiche sane e
patologiche (run costante localizzato, struttura periodica non costante) —
eseguita ripetutamente senza flakiness, nessuna eccezione, nessun blocco.
Durante questo testing sono stati trovati e corretti, oltre al bug di
scalabilità sopra, un refuso di trascrizione nel vettore di test SHAKE256
(un carattere mancante — verificato e corretto contro l'implementazione
nativa OpenSSL/Node) e due soglie di autotest inizialmente mal calibrate.

## v2 — 2026-09-10

- **Rinominato il progetto**: `estrattore_entropia_raw_2photo.html` →
  `entropy-extractor-raw-2photo.html`.
- **Rimosso il linguaggio "finestra vincente/vincitrice".** La finestra con
  il punteggio più alto è ora indicata come "scelta come migliore", con un
  badge neutro al posto della stella (★).
- **Aggiunto il pannello "Confronto fra le finestre e criterio di scelta"**,
  mostrato dopo ogni analisi: elenca per ciascuna finestra lo stato (ammessa
  al confronto / gate fallito / esclusa per correlazione incrociata), i test
  NIST superati, la min-entropia stimata e il punteggio calcolato. Il motivo
  per cui una finestra è stata scelta e le altre no è ora sempre visibile ed
  esplicito, non implicito in un'etichetta.
- Criterio di scelta reso esplicito anche in prosa nel pannello:
  `punteggio = (test NIST superati × 10) + min-entropia finale (bit/bit) + (bit prodotti ÷ 1000)`.

## v1 — 2026-09-10

Prima versione "unificata" del tool RAW a due scatti (dark frame). Sostituisce
il motore di stima naive del tool RAW originale con il motore
matematico rigoroso già auditato del tool "Pipeline di Estrazione Entropica
Multi-Finestra" (audio/PCM):

1. **Motore matematico riusato senza modifiche**: SHA-256/HMAC/HKDF,
   Clopper-Pearson esatto, Peres Extractor completo, Toeplitz strong
   extractor a rapporto dinamico, stime MCV/Collision, suite NIST
   (Monobit/Block Frequency/Runs/Longest Run), controllo di correlazione
   incrociata fra finestre e autotest all'avvio.
2. **Sorgente sostituita**: non più PCM decodificato, ma la differenza
   parola-per-parola (16 bit) fra due dark frame allineati, letta dal blocco
   dati RAW (parser TIFF/IFD o offset manuale).
3. **[Bug reale corretto] Filtro di esclusione tarato sul dominio SIGNED, non
   più UNSIGNED.** Il filtro di saturazione ereditato dal tool PCM escludeva
   anche il valore letto come UNSIGNED 0xFFFF. Per un campione PCM questo è
   un estremo di clipping legittimo; per una differenza con segno in
   complemento a due, 0xFFFF rappresenta invece **diff = −1**, un valore di
   rumore piccolo e comune — escluderlo introduceva un'asimmetria
   sistematica (ogni −1 scartato, ogni +1 mantenuto). Riprodotto e
   verificato (1000 campioni con diff=−1 scartati al 100% con il filtro
   ereditato). Corretto: si esclude solo diff=0 (cancellazione esatta) e i
   due estremi con segno ±32767/−32768 (overflow/clip). Aggiunto un autotest
   dedicato di regressione.
4. **Rimossa la selezione euristica "migliori bit per varianza locale +
   diversità zone"** del tool RAW originale, priva di giustificazione
   statistica formale. Sostituita dall'estrazione Toeplitz a rapporto
   dinamico sulla min-entropia realmente misurata (bound del Leftover Hash
   Lemma).
5. **Rimossi i metodi DIFF/PARITY come canali aggiuntivi concatenati** (la
   versione precedente concatenava LSB + segno-della-differenza-fra-
   campioni-consecutivi + parità dello stesso campione, senza indipendenza
   statistica stabilita fra i tre). Resta un solo canale per campione (LSB
   della differenza dark1−dark2).
6. **Finestre spaziali invece del barcode di posizione**: Inizio / Centro /
   Fine del blocco dati + Blocco intero (quest'ultimo escluso dal confronto
   perché non indipendente per costruzione), con controllo di correlazione
   incrociata multi-lag fra le tre finestre spaziali.
7. **Mantenuti dal tool RAW originale**: dichiarazione obbligatoria di
   unicità/autoproduzione del file, parser TIFF/IFD per il rilevamento
   automatico del blocco dati, campi manuali di offset/lunghezza/ordine
   byte, limite di memoria a 10 MB con lettura a porzione (`File.slice`) per
   file più grandi.
8. **Restyle grafico** sulla palette BIP39/Bootstrap 3 (Ian Coleman),
   identica a quella del tool audio/PCM di riferimento.

## Riferimento pre-v1

Versione originale non unificata: `estrattore_entropia_raw_2photo.html`
(v1.0–v1.5), con parser TIFF/IFD, modalità dark frame opzionale a 2 scatti,
pipeline LSB/DIFF/PARITY + selezione "migliori bit" per varianza locale, e
stima di entropia basata su Shannon entropy × fattore di sicurezza manuale.
Superata dalla v1 di questo repository per i motivi elencati sopra.
