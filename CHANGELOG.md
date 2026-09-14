# Changelog

Tutte le date fanno riferimento alla build indicata in cima al file HTML.

## v4.0.0-beta2 (2026-09-13) — BETA, non ancora pubblicata su GitHub

**Stato: da verificare e auditare prima del rilascio pubblico.** SHA-256 del file
`entropy-extractor-raw-2photo.html` di questa build:
`10f3cbb86ad5269931ab53c6412df226aac32a02df1e8ed54ce3a7b04cb93ffe`.

### Batteria NIST SP 800-22 estesa da 4 a 9 procedure

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
