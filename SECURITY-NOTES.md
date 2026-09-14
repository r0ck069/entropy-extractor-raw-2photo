# Principi di sicurezza e di design

Questo file raccoglie i principi che guidano le scelte di design dei tool della
famiglia `EntropyPipeline` / `entropy-extractor` / `entropy-extractor-raw-2photo`
(stesso autore), al di là del changelog build-per-build. Motore matematico
condiviso 1:1 con `entropy-extractor` — vedi anche il `SECURITY-NOTES.md` di
quel repository, che approfondisce i principi 1, 2, 4 e 5 qui riassunti.

## Principio 1 — Mai inventare bit

La **modalità rigorosa** (default) restituisce solo i bit realmente estratti da
Toeplitz. L'espansione HKDF opzionale, quando la modalità rigorosa è disattivata
esplicitamente, resta sempre etichettata come "sintetica" nell'output.

## Principio 2 — Tetto di emissione indipendente dal bound principale

Dalla v4.0.0-beta1, ogni blocco Toeplitz applica un tetto aggiuntivo — **mai più
di ⌊TOEPLITZ_IN/2⌋ = 256 bit per blocco** — indipendente dal bound LHL già
calcolato dinamicamente sulla min-entropia misurata. Rete di sicurezza a basso
costo, non un sostituto del bound principale.

## Principio 3 — Il file RAW sorgente è l'unico segreto che conta

A differenza del seed Toeplitz (pubblico per costruzione, proprietà del
Leftover Hash Lemma), il file RAW dei due dark frame è l'unico elemento che
deve restare unico/non pubblico: se riprodotto o già noto a terzi, l'entropia
crittografica reale tende a zero indipendentemente da qualunque test statistico
qui presente. Per questo la dichiarazione di unicità/autoproduzione resta un
gate bloccante sugli input, non un semplice avviso.

## Principio 4 — Non estendere una funzionalità oltre il contesto per cui è stata calibrata

Il bit-plane arbitrario (introdotto in `entropy-extractor` per sorgenti PCM) non
è stato portato qui nella v4.0.0-beta1: la sorgente di questo tool è già una
differenza con segno a 16 bit, non un campione grezzo — i bit più significativi
di una differenza piccola sono fortemente polarizzati dal segno, e selezionarli
senza un'analisi dedicata rischierebbe di introdurre bias non rilevati dai gate
esistenti (calibrati sul solo LSB della differenza).

## Principio 5 — Trasparenza sui limiti, sempre visibile

Ogni stima statistica dichiara esplicitamente cosa NON è: non è una
certificazione, non sostituisce la batteria completa SP 800-90B/SP 800-22.
Applicato nell'interfaccia e documentato per esteso in `AUDIT-NOTES.md`.

## Principio 6 — Un bias forzato per il testing va isolato al segnale che conta, non sparso ovunque

Stessa lezione di `entropy-extractor` (v4.0.0-beta2, vedi CHANGELOG.md),
applicata qui alla differenza fra due frame invece che a un singolo bit-plane:
forzare il bias su OGNI bit di entrambi i frame sintetici farebbe sì che i
filtri di esclusione (diff=0, diff agli estremi) scartino proprio i campioni
più sbilanciati. La correzione qui è stata applicata preventivamente (bias
solo sulla parità della differenza risultante, mai su diff=0 o vicino agli
estremi), non scoperta a posteriori come nel tool gemello — ma resta la stessa
lezione, ed è per questo che è documentata come principio condiviso.

## Principio 7 — Separazione di dominio quando più sorgenti alimentano una funzione di hash

Non si applica direttamente a questo tool (una sola coppia di dark frame, nessuna
combinazione multi-sorgente basata su hash) — vedi la stessa nota in
`entropy-extractor/SECURITY-NOTES.md`, principio 7, per il dettaglio completo.

## Stato di audit di questi principi

I principi 1, 3 e 5 derivano da bug/lezioni già verificati nella storia del
progetto (vedi CHANGELOG.md). I principi 2, 4, 6 e 7 sono **nuovi/resi espliciti
con la v4.0.0-beta1/beta2** e non sono ancora stati sottoposti allo stesso
livello di verifica indipendente degli altri.
