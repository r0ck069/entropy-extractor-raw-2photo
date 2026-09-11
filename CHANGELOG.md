# Changelog

Tutte le date fanno riferimento alla build indicata in cima al file HTML.

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
