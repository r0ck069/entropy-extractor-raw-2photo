# Changelog

Tutte le date fanno riferimento alla build indicata in cima al file HTML.
## v4.0.0-beta4 (2026-10-01) — bound LHL per blocco Toeplitz e allineamento della documentazione

**SHA-256 di `entropy-extractor-raw-2photo.html` di questa build:** `4180a9b4dc45584b8c6175fa6750483c83f7a7f98d6152d8dde60d8a5a277a8e` (calcolato con `sha256sum`)

### Correzione — bound del Leftover Hash Lemma calcolato sul totale della finestra invece che per blocco (2026-10-01)

**Bug.** Lo stesso difetto corretto in `entropy-extractor` v4.0.0-beta4. Il margine 2k era sottratto una sola volta dalla min-entropia totale della finestra (h·n) e ne usciva un rapporto globale `ratioFromLHL`. Ma il Toeplitz è applicato a B blocchi da 512 bit (`TOEPLITZ_IN`), ciascuno con la propria matrice (chiavi a finestra scorrevole, sovrapposte), e la garanzia del lemma vale per blocco. Dove il rapporto di estrazione era deciso dal fattore pratico 0,85·h e non dal bound, il margine per blocco era molto inferiore ai circa 100 bit richiesti. Il lemma è una condizione sufficiente: questo non dimostra che l'uscita fosse distinguibile dal casuale, ma la garanzia dichiarata non valeva.

**Correzione.** Nuova funzione `lhlSafeBlockOutBits(h, B, k)`: uscita per blocco al massimo 512·h − 2·(k + log₂B), con B = ⌊bit post-Peres / 512⌋; ε=2⁻ᵏ è l'errore totale sulla finestra e ε/B quello di ogni blocco (argomento ibrido, valido anche con chiavi sovrapposte). Applicata nell'unico punto di calcolo di questo strumento; `ratioFromLHL` è ora per blocco e `lhlSafeOutputBits` = B·uscita sicura per blocco. Fattore pratico 0,85, massimo 0,90 e tetto di 256 bit per blocco restano invariati. Le costanti sono le stesse di `entropy-extractor`: valgono gli stessi valori di effetto (con 1 milione di bit grezzi, Peres al 90%, k=40: uscita per blocco h=0,3: 130 → 52; h=0,5: 217 → 154; h=0,6: 256 → 205; da h=0,7 resta 256). Nella prova in browser con dark frame sintetici: coppia a bassa entropia (h=0,125): 54 bit per blocco → 0 bit; coppia con h=0,81: 256 bit per blocco, invariato (tetto).

**Verifica.** (1) `toeplitz_margin_check.js`, che esegue il codice reale della pagina: valori noti, casi limite e proprietà "errore totale garantito ≤ 2⁻ᵏ" su 20.000 combinazioni casuali di h, B e k: esito tutto OK; sul file della beta3 fallisce perché la funzione non esiste. (2) `lhl_exact_check.py`: distanza statistica esatta su famiglie Toeplitz piccole con la stessa indicizzazione del codice: per un blocco sempre ≤ ½·2^(−(t−m)/2) (21 casi, rapporto massimo 0,747); per due blocchi con chiave a finestra scorrevole sempre ≤ 2 volte l'errore di un blocco (15 casi, rapporto massimo 0,493); non è una prova per le dimensioni reali (512 bit). (3) Autotest della pagina 20 su 20 prima e dopo; `lrs_check.js` e `raw2photo_check.js` invariati. (4) Prova in un browser reale (Chromium senza interfaccia, in un ambiente di sviluppo) con due coppie di dark frame sintetici: nessun errore in console, autotest 20/20, analisi completa. Non provato: file RAW veri (DNG, NEF e simili), browser da telefono, aspetto grafico, pulsanti di download e SHAKE256.

**Ipotesi non verificate dal tool.** Ogni blocco ha min-entropia almeno 512·h anche condizionata ai blocchi precedenti; ai bit in ingresso al Toeplitz si attribuisce la min-entropia per bit h stimata sui bit grezzi (senza accreditare la compressione di Peres, non dimostrato per sorgenti non i.i.d.); il seme è pseudocasuale (SHA-256), garanzia computazionale; la confidenza al 99% dello stimatore aggiunge circa 0,01 non inclusi in ε. Dettaglio in `SECURITY-NOTES.md`, Principio 2.

### Etichette di build, conteggio dei test NIST e rimandi (2026-10-01)

**Modifica.** (a) La riga di intestazione della pagina diceva "Build 4.0.0-beta3 — BETA ... Test: 9/15 NIST SP 800-22": ora "Build 4.0.0-beta4 — BETA" e "Test: 8/15 procedure NIST SP 800-22 (9 test)": i test sono 9 (Monobit, Block Frequency, Runs, Longest Run, Serial, Approximate Entropy, Cumulative Sums diretto e inverso, Binary Matrix Rank) ma le procedure sono 8, perché le due Cumulative Sums sono una sola procedura. (b) La seconda riga di testo sotto l'intestazione, che cominciava con "build 2026-09-11 v3" accanto a "Build 4.0.0-beta3", ora comincia con "Nota storica (v3, build 2026-09-11)", per non lasciare due etichette di build. (c) Tre rimandi errati "CHANGELOG punto 9", "CHANGELOG punto 8" (riga di testo sotto l'intestazione) e "CHANGELOG punto 3" (commento nel codice) puntano ora ad `AUDIT-NOTES.md`, dove si trovano i punti 3, 8 e 9. Nessun'altra riga di testo della pagina è cambiata oltre alla correzione sopra. In questa build `W = u` è alla riga 1156 e `preEntropy = sourceEnt` alla riga 1479 (i numeri di riga delle voci precedenti si riferiscono alla build di quella voce).

### Documentazione allineata (2026-10-01)

- **README:** versione e build a v4.0.0-beta4; conteggio dei test NIST (8 procedure in 9 test); descrizione del margine LHL (ε totale, ε/B per blocco, rapporto per blocco); elenco completo dei file e nuova sezione "Stato di verifica"; una sola frase della sezione "Per la raccolta delle sorgenti" corretta con l'assenso dell'autore: "l'assistente CSV/sensore di questa build" → "l'assistente CSV/sensore di EntropyPipeline (non incluso in questo strumento)", perché in questo strumento l'assistente non è stato portato (vedi "Non portate da EntropyPipeline/entropy-extractor", v4.0.0-beta2) e la pagina non lo contiene; il resto della sezione non è stato modificato (da rivedere a parte).
- **AUDIT-NOTES:** nuovi punti 10, 11 e 12 (le tre correzioni dopo la v3); nota sul punto 9 (verifiche e soglie su LRS anteriori alla correzione); tolta la dicitura "prima di un rilascio pubblico" (il repository è pubblico come BETA dal 14/09/2026).
- **SECURITY-NOTES:** Principio 2 riscritto (i quattro limiti dell'uscita per blocco e le ipotesi del bound); stato di audit aggiornato.
- **CHANGELOG:** note storiche sulle voci beta1 e beta2 ("non ancora pubblicata" riferito alla data della voce), sul conteggio delle procedure della beta2 e sulla descrizione del margine LHL della beta2.

### File aggiunti

`toeplitz_margin_check.js` e `lhl_exact_check.py` (vedi sopra).

## v4.0.0-beta3 (2026-09-28) — correzioni dall'audit indipendente e aggiornamento dell'etichetta di build

**SHA-256 di `entropy-extractor-raw-2photo.html` di questa build:** `a72412130af6c265aef90240b01fbaae1660d088428ac1dd6880997a82ef5a88` (calcolato con `sha256sum` sul Raspberry Pi il 28/09/2026, dopo l'aggiornamento dell'etichetta di build)

### Correzione — stima dell'entropia dopo Peres invece che prima (2026-09-27)

**Bug.** Lo stesso difetto corretto in `entropy-extractor` (`9023e8e`), mai propagato in questo repository separato. Alla riga 1465 `preEntropy` era calcolata su `peresRes.bits` (dopo Peres) invece che sui bit grezzi. Su sorgenti correlate l'entropia post-Peres appare vicina a 1 bit/bit anche quando è molto più bassa, con sovrastima dei bit di output emettibili in sicurezza (violazione della Leftover Hash Lemma).

**Correzione.** `preEntropy = sourceEnt` (bit grezzi, prima di Peres).

**Verifica.** `raw2photo_check.js`, 200.000 bit sintetici:
- correlazione 90% (entropia vera 93.799 bit): circa 120.000 bit accreditati prima del fix (+28% oltre il vero), circa 15.000 dopo;
- correlazione 75% (entropia vera 162.256 bit): circa 170.000 bit prima (+5%), circa 45.000 dopo;
- rumore perfetto: comportamento prudente sia prima sia dopo (invariato).

I valori esatti variano di poche centinaia di bit da un'esecuzione all'altra, perché i dati di prova sono generati a caso: per questo sono riportati arrotondati.

**Commit:** `d142b2f`

### Correzione — stimatore LRS: lunghezza sbagliata, conteggio sempre 1 (2026-09-27)

**Bug.** Come in `entropy-extractor`: in `lrsHmin()` `W = u+1` invece di `W = u`, con conteggio massimo sempre 1 per costruzione (statistica priva di informazione). Già corretto in `EntropyPipeline` (`56aedff`), non propagato qui.

**Correzione.** `W = u` (riga 1142 di `entropy-extractor-raw-2photo.html`).

**Verifica.** `lrs_check.js` (aggiunto al repository): prima del fix conteggio sempre 1; dopo il fix valori calcolati dai dati (circa 0,40-0,47 bit/bit su rumore casuale, a seconda della lunghezza del campione).

**Commit:** `3952f86`

### Aggiornamento dell'etichetta di build (2026-09-28)

**Modifica.** La riga di intestazione della pagina diceva "Build 4.0.0-beta2 — BETA non ancora pubblicata su GitHub". La beta è pubblicata su GitHub dal 14/09/2026, e la pagina include ora le due correzioni sopra: l'etichetta è stata portata a "Build 4.0.0-beta3 — BETA". Nessun'altra riga della pagina è cambiata.

## v4.0.0-beta2 (2026-09-13) — BETA, non ancora pubblicata su GitHub

> *Nota storica (2026-10-01): lo stato "non ancora pubblicata" e il giudizio "da verificare e auditare prima del rilascio pubblico" si riferiscono al 2026-09-13; il repository è pubblico come BETA dal 14/09/2026 (vedi voce v4.0.0-beta3).*

**Stato: da verificare e auditare prima del rilascio pubblico.** SHA-256 del file
`entropy-extractor-raw-2photo.html` di questa build:
`10f3cbb86ad5269931ab53c6412df226aac32a02df1e8ed54ce3a7b04cb93ffe`.

### Batteria NIST SP 800-22 estesa da 4 a 9 procedure

> *Nota (2026-10-01): le procedure sono 8 (4 già presenti più 4 nuove; Cumulative Sums diretto e inverso sono una sola procedura), eseguite in 9 test.*

Stessi 4 test aggiunti in `entropy-extractor` v4.0.0-beta2 (stesso nucleo
condiviso): Serial completo (∇ψ²/∇²ψ², estensione ciclica), Approximate Entropy
completa (estensione ciclica), Cumulative Sums (diretto+inverso), Binary Matrix
Rank (32×32). Stessa verifica rigorosa applicata qui: valori analitici/tabulati
noti (Γ(6)=120, P(χ²₁>3.841)≈0.05, P(χ²₂>9.210)≈0.01), casi avversari
(tutti-zero/tutti-uno tutti correttamente FALLITI), tasso di falsi positivi su
300 sequenze CSPRNG reali indipendenti (50.000 bit ciascuna) risultato entro 3σ
dal teorico per tutti e 4 i test, autotest interno (21 controlli) rieseguito
senza regressioni.

### Margine di sicurezza LHL selezionabile (ε=2⁻ᵏ)

> *Nota (2026-10-01): dalla v4.0.0-beta4 ε è l'errore totale sull'uscita della finestra e il bound `ratioFromLHL` è calcolato per blocco Toeplitz (uscita per blocco ≤ 512·h − 2·(k + log₂B)); la descrizione che segue è quella della beta2.*

Stessa funzionalità di `entropy-extractor`: prima fisso a ε=2⁻⁴⁰, ora
selezionabile (2⁻¹⁶/2⁻²⁰/2⁻⁴⁰/2⁻⁶⁴/2⁻⁸⁰), k=40 default invariato, conferma
richiesta solo scendendo sotto il default. Qui esiste una sola occorrenza del
bound (nessuna funzionalità di fusione in questo tool), quindi l'integrazione
è più semplice che in `entropy-extractor`.

### Demo sbilanciata (test) — stessa lezione di `entropy-extractor`, applicata all'architettura a due file

Questo tool non ha un buffer-in-cache unico da sostituire (legge da due
`<input type="file">` distinti dentro `startAnalysis`, e per motivi di
sicurezza del browser un pulsante non può impostare programmaticamente
`.files` su un vero input file). Introdotto invece un meccanismo di override
esplicito (`demoOverrideBuffers`), usato una sola volta e poi azzerato, che
bypassa la lettura dei due file reali quando presente.

I due dark frame sintetici sono generati così: `dark2` è rumore uniforme reale
a 16 bit (CSPRNG); `dark1 = dark2 + Δ`, dove Δ è una piccola differenza
(±1/±2/±3/±4, mai 0, mai vicina agli estremi) scelta con **parità
deliberatamente sbilanciata** (p≈0.98 verso una parità specifica). Stessa
lezione già imparata in `entropy-extractor` durante questo stesso giro di
sviluppo: forzare il bias su TUTTO il segnale (qui: su ogni bit di ogni parola
di entrambi i frame) farebbe sì che i filtri di esclusione (diff=0, diff agli
estremi) scartino proprio i campioni più sbilanciati, vanificando la demo.
Applicata la correzione preventivamente, non scoperta a posteriori: forzando
la parità solo sulla differenza risultante (non sui frame grezzi) e scegliendo
Δ sempre lontano da 0 e dagli estremi, il filtro non interferisce. Verificato
con test reale: H_min misurato dal vero stimatore crolla a ≈0.025-0.028, il
gate scatta correttamente al primo tentativo.

### Non portate da EntropyPipeline/entropy-extractor, con motivazione esplicita

- **Bit-plane arbitrario**: già scartato in v4.0.0-beta1 (la sorgente è già una
differenza con segno a 16 bit; i bit alti sono polarizzati dal segno) — non
ci sono novità che cambino questa valutazione.
- **Fusione best-2 delle finestre spaziali**: non portata per limiti di tempo,
non per motivi tecnici (invariato da v4.0.0-beta1).
- **Assistente CSV/sensore → bit grezzi** e **parsing tollerante + lista di
byte**: stessa motivazione di `entropy-extractor` — questo tool accetta due
file RAW in caricamento diretto, non ha caselle di testo per bit/hex a cui
applicare queste migliorie.
- **Tag di dominio prima della concatenazione multi-sorgente**: documentato
come principio in `SECURITY-NOTES.md`, non applicabile direttamente (una sola
coppia di file, nessuna combinazione multi-sorgente).

---

## v4.0.0-beta1 (2026-09-13) — BETA, non ancora pubblicata su GitHub

> *Nota storica (2026-10-01): lo stato "non ancora pubblicata" e il giudizio "da verificare e auditare prima del rilascio pubblico" si riferiscono al 2026-09-13; il repository è pubblico come BETA dal 14/09/2026 (vedi voce v4.0.0-beta3).*

**Stato: da verificare e auditare prima del rilascio pubblico.** SHA-256 del file
`entropy-extractor-raw-2photo.html` di questa build:
`cfd3936b530d2461b89df050e2c52eb175b1feb064684e755d5ce16247ee5df0`.

### Aggiunte

1. **Hash di integrità dell'applicazione**: SHA-256 del blocco `<script
id="core-script">`, calcolato e mostrato al caricamento — controllo diagnostico
per verificare di star eseguendo il codice atteso, specialmente offline.
2. **Tetto di emissione conservativo** ⌊TOEPLITZ_IN/2⌋ = 256 bit per blocco,
indipendente dal bound LHL già calcolato — rete di sicurezza a basso costo.
**Nota di comportamento**: abbassa il rapporto di estrazione massimo pratico
nei rari casi di entropia misurata molto alta (prima fino al 90% di 512=460
bit/blocco, ora sempre ≤256 bit/blocco); su rumore da sensore fotografico
tipico l'effetto pratico è minimo, dato che le min-entropie osservate restano
di norma ben sotto questa soglia.

### Riorganizzazione della documentazione (nessun cambiamento funzionale sul resto)

Il pannello "DOCUMENTAZIONE E ASSUNZIONI" con il changelog di adattamento, prima
incorporato nell'HTML, è stato spostato in `AUDIT-NOTES.md`. Aggiunto
`SECURITY-NOTES.md` con i principi di design trasversali (condivisi con
`EntropyPipeline` e `entropy-extractor`, stesso autore). L'HTML riporta ora solo
l'essenziale operativo e i limiti indispensabili all'uso.

### Verifica eseguita prima di questo commit

Il nucleo matematico/crittografico (identico a `entropy-extractor`, incluso il
parser TIFF/IFD e il filtro diff con segno) è **invariato** rispetto alla v3.
Verificato con: (a) l'intera suite di autotest esistente della pagina,
rieseguita senza regressioni; (b) un harness Node.js dedicato che simula
l'intera pipeline end-to-end (due dark frame sintetici pseudo-casuali di
100.000 parole da 16 bit ciascuno, dichiarazione di unicità, click su AVVIA,
rendering di tutte le 4 card) — nessuna eccezione, nessun FAIL negli autotest,
tetto di emissione confermato funzionante (512 bit in → 256 bit out) nell'output
renderizzato. Non ancora sottoposto ad audit indipendente da terzi. **Non
pubblicare su GitHub prima di un audit.**

### Non incluso in questa beta, con motivazione

- **Bit-plane arbitrario**: qui la sorgente è già una differenza con segno a 16
bit (non un campione PCM grezzo); i bit più significativi di una differenza
piccola sono fortemente polarizzati dal segno, rendendo la selezione di
bit-plane diversi dal LSB potenzialmente meno sicura senza un'analisi dedicata
— rimandata a una beta successiva.
- **Fusione best-2 delle finestre spaziali**: la stessa funzionalità introdotta
in `entropy-extractor` non è stata portata qui per limiti di tempo in questa
iterazione, non per motivi tecnici (l'architettura a finestre è identica).
Pianificata per una beta successiva.
- **Selezione bit per varianza locale, campionamento da posizioni prime,
pipeline di confronto diagnostica**: stesse motivazioni già documentate nel
CHANGELOG di `entropy-extractor` (rischio di interazione con gate già
calibrati, tempo).

---

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
- **Repetition Count Test (RCT) e Adaptive Proportion Test (APT)** (SP 800-90B §4.4): health test retrospettivi applicati ai bit LSB grezzi di
ciascuna finestra, prima del debiasing di Peres.

Tre adattamenti espliciti, resi necessari dal contesto diverso (flussi
automatici di centinaia di migliaia di bit per finestra, contro le
centinaia/migliaia di bit incollati a mano della sorgente originale):

1. **t-Tuple/LRS limitati a un campione di 20.000 bit per finestra** (`TTUPLE_LRS_SAMPLE_CAP`), per restare entro un tempo di calcolo
ragionevole; la dimensione del campione è sempre mostrata nello schema
numerico.
2. **Bug di scalabilità reale trovato e corretto durante il porting**: né
t-Tuple né LRS, nella sorgente originale, limitano la lunghezza massima di
pattern esaminata. Su una sequenza periodica il costo può esplodere a
diversi miliardi di operazioni — riprodotto: il porting iniziale ha
bloccato la pipeline di test per minuti su un singolo caso periodico non
patologico in senso stretto. Corretto con due limiti (`TTUPLE_MAX_T=64`,
`LRS_MAX_SEARCH_LEN=256`), verificati non alterare il comportamento su
rumore reale né la rilevazione su sequenze periodiche/strutturate.
3. **t-Tuple/LRS non entrano nel calcolo della min-entropia che determina il
rapporto di estrazione Toeplitz** (a differenza della sorgente originale).
Scoperto empiricamente durante il testing: su campioni di decine di
migliaia di bit, LRS in particolare resta strutturalmente intorno a
0.35-0.43 bit/bit anche su rumore CSPRNG perfettamente equo (natura
conservativa nota, non un segnale reale di scarsa entropia). Restano invece
un **gate strutturale dedicato** (soglie calibrate con ampio margine
empirico: sorgenti sane/moderatamente sbilanciate osservate a LRS≥0.21,
t-Tuple≥0.29; sorgenti periodiche/strutturate osservate a LRS≤0.015,
t-Tuple≤0.05).

RCT/APT sono invece applicati senza campionamento (costo O(n)) come gate
aggiuntivo per finestra, con la stessa priorità "il più pessimista vince"
degli altri gate.

Verificato con una suite di test end-to-end in Node.js prima del rilascio:
unit test sui singoli stimatori, l'intera suite di autotest della pagina, e
simulazioni complete della pipeline con sorgenti sintetiche sane e
patologiche — eseguita ripetutamente senza flakiness, nessuna eccezione,
nessun blocco. Durante questo testing sono stati trovati e corretti, oltre
al bug di scalabilità sopra, un refuso di trascrizione nel vettore di test
SHAKE256 (un carattere mancante — verificato e corretto contro
l'implementazione nativa OpenSSL/Node) e due soglie di autotest inizialmente
mal calibrate.

## v2 — 2026-09-10

- **Rinominato il progetto**: `estrattore_entropia_raw_2photo.html` →
`entropy-extractor-raw-2photo.html`.
- **Rimosso il linguaggio "finestra vincente/vincitrice".** La finestra con
il punteggio più alto è ora indicata come "scelta come migliore", con un
badge neutro al posto della stella (★).
- **Aggiunto il pannello "Confronto fra le finestre e criterio di scelta"**,
mostrato dopo ogni analisi: elenca per ciascuna finestra lo stato (ammessa
al confronto / gate fallito / esclusa per correlazione incrociata), i test
NIST superati, la min-entropia stimata e il punteggio calcolato.
- Criterio di scelta reso esplicito anche in prosa nel pannello:
`punteggio = (test NIST superati × 10) + min-entropia finale (bit/bit) + (bit prodotti ÷ 1000)`.

## v1 — 2026-09-10

Prima versione "unificata" del tool RAW a due scatti (dark frame). Sostituisce
il motore di stima naive del tool RAW originale con il motore matematico
rigoroso già auditato del tool "Pipeline di Estrazione Entropica Multi-Finestra"
(audio/PCM):

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
anche il valore letto come UNSIGNED 0xFFFF. Per una differenza con segno in
complemento a due, 0xFFFF rappresenta invece **diff = −1**, un valore di
rumore piccolo e comune — escluderlo introduceva un'asimmetria sistematica.
Riprodotto e verificato (1000 campioni con diff=−1 scartati al 100% con il
filtro ereditato). Corretto: si esclude solo diff=0 e i due estremi con
segno ±32767/−32768.
4. **Rimossa la selezione euristica "migliori bit per varianza locale +
diversità zone"** del tool RAW originale, priva di giustificazione
statistica formale. Sostituita dall'estrazione Toeplitz a rapporto
dinamico sulla min-entropia realmente misurata.
5. **Rimossi i metodi DIFF/PARITY come canali aggiuntivi concatenati.**
Resta un solo canale per campione (LSB della differenza dark1−dark2).
6. **Finestre spaziali invece del barcode di posizione**: Inizio / Centro /
Fine del blocco dati + Blocco intero (escluso dal confronto), con
controllo di correlazione incrociata multi-lag fra le tre finestre
spaziali.
7. **Mantenuti dal tool RAW originale**: dichiarazione obbligatoria di
unicità/autoproduzione del file, parser TIFF/IFD, campi manuali di
offset/lunghezza/ordine byte, limite di memoria a 10 MB con lettura a
porzione (`File.slice`) per file più grandi.
8. **Restyle grafico** sulla palette BIP39/Bootstrap 3 (Ian Coleman).

## Riferimento pre-v1

Versione originale non unificata: `estrattore_entropia_raw_2photo.html`
(v1.0–v1.5), con parser TIFF/IFD, modalità dark frame opzionale a 2 scatti,
pipeline LSB/DIFF/PARITY + selezione "migliori bit" per varianza locale, e
stima di entropia basata su Shannon entropy × fattore di sicurezza manuale.
Superata dalla v1 di questo repository per i motivi elencati sopra.
