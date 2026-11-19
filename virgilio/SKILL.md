---
name: virgilio
description: Guided, feature-centric walkthrough of all code changes in a branch, PR, or master-to-commit range to help a human approve agent-written code. Use for an interactive code tour or a walkthrough of planned changes, with clear file status, impacts, and complete coverage.
---

# Virgilio - Code Tour

Accompagna l'utente nella revisione di tutto il codice modificato, organizzandolo per comportamento e feature. Rispondi nella lingua dell'utente. Risparmia token con letture mirate e spiegazioni brevi, mai riducendo la copertura delle modifiche. Presenta i file di solo contesto soltanto quando l'utente chiede di approfondire. La decisione di approvare resta umana; il tour non pubblica review, non approva PR e non modifica codice.

## Fissa il perimetro

- Se l'utente chiede di esplorare i file che verranno modificati, usa il piano discusso come perimetro: distingui codice attuale e proposta, segnala i dettagli ancora da verificare e non applicare automaticamente il confronto del branch con master. In questo tour preventivo SHA, commit e hunk futuri non sono disponibili; registra file e interventi previsti, senza inventarli. Non modificare il codice ne eseguire build o test per validare una proposta non implementata. Le istruzioni Git seguenti riguardano i tour di modifiche gia realizzate.
- Leggi le istruzioni applicabili del repository. In workspace con repository separati individua quello interessato; chiedi solo se il target resta ambiguo.
- Target predefinito: branch corrente, base `master`. Risolvi base e target in SHA immutabili con `git rev-parse --verify <ref>^{commit}`. Se `master` manca, usa `origin/master` dichiarandolo; non sostituire silenziosamente con `main`. Per una PR usa i riferimenti effettivi della PR e dichiarali. Nessun fetch o checkout necessario per riferimenti gia disponibili; segnala riferimenti mancanti o potenzialmente obsoleti.
- Calcola `START = git merge-base BASE TARGET`. Revisiona il diff finale `START TARGET` e inventaria tutti i commit in `START..TARGET`, inclusi quelli confluiti tramite merge. Per la richiesta esplicita “dal commit A al commit B”, usa A come START; A e escluso dall'intervallo, B incluso. Spiega eventuali divergenze da master.
- Annota repository, BASE, START, TARGET, stato del working tree. Non includere modifiche locali. Leggi il codice da TARGET tramite Git se il checkout differisce o contiene modifiche locali. Non alterare branch, index o file utente.
- Distingui il risultato finale dalla storia: includi anche modifiche intermedie poi annullate, commit senza effetto finale e risoluzioni dei merge. Non presumere che il solo diff finale copra tutti i commit.

## Inventario completo, lettura progressiva

Inizia con `git diff --name-status -M START TARGET`, `git diff --numstat START TARGET` e `git log --reverse --format="%h %s" START..TARGET`. Espandi gli elenchi a blocchi se grandi, senza perdere righe. I messaggi dei commit suggeriscono intenti, non provano il comportamento.

Crea un registro compatto fuori dal repository (directory temporanea univoca) con SHA, feature, commit, file/hunk e stato: da leggere, letto, spiegato, discusso, bloccato. Mantieni il percorso nel riepilogo per riprendere il tour senza rileggere tutto. Non marcare “discusso” prima del riscontro dell'utente; “avanti” conferma la tappa, non approva la PR. Se non puoi salvare il registro, mantienilo in chat e dichiara il limite.

Raggruppa per feature, non per cartella o commit. Ogni file e ogni hunk devono appartenere ad almeno una tappa, inclusi cancellazioni, test, configurazione, lockfile, localizzazioni e codice generato. Raggruppa modifiche meccaniche ripetitive spiegandone la regola e verificandone l'uniformita; non trattarle come escluse. Per binari o artefatti non ispezionabili registra il limite come bloccante per la copertura completa.

Per la storia, esamina i diff dei commit a blocchi. L'inventario completo e l'unione del diff finale e dei diff storici: include anche file introdotti e poi cancellati. Riusa quanto gia letto nel diff finale; approfondisci gli hunk intermedi non rappresentati, revert e merge. Per merge confronta i parent e verifica le risoluzioni: un combined diff vuoto non dimostra assenza di modifiche. Registra ogni commit come coperto da una feature, intermedio annullato, merge verificato o ancora da analizzare.

Presenta prima la mappa di visita descritta sotto, dichiarando che e provvisoria finche il codice non e letto. Poi inizia la prima tappa; fermati dopo una tappa per permettere domande. Se l'utente richiede solo la mappa, fermati alla mappa. Se richiede esecuzione autonoma, completa tutte le tappe senza attese intermedie.

## Mappa e visita nel proprio editor

Per ogni step proponi una visita guidata con link ai file da aprire nell'editor dell'utente. Non usare il diff integrato, pannelli di review o link `codex://review` per la navigazione. Usa Git per analizzare le modifiche, senza riversare i diff in chat. Lascia all'utente l'apertura dei file; se chiede di aprirli direttamente, usa il connettore del suo editor quando disponibile.

Usa link Markdown con percorso assoluto e riga verificata: `[NomeFile.ts:42](C:/percorso/repo/src/NomeFile.ts:42)`. Per percorsi con spazi racchiudi la destinazione fra parentesi angolari. La riga deve appartenere al file effettivamente aperto, non al diff o a un'altra revisione. Non inventare URI specifici di un editor. Per file assenti, cancellati o diversi da TARGET, estrai con Git una copia temporanea della revisione pertinente e collega quella, etichettandola con lo SHA; per cancellazioni mostra la versione precedente pertinente. Preserva i percorsi relativi nella copia, senza sovrascrivere il checkout. Distingui copie storiche e codice corrente.

Mostra una legenda compatta con le sole etichette usate nella tappa e applicale a ogni file elencato, anche nei richiami successivi. Usa simbolo e testo insieme, senza affidarti solo al colore:

- **✏️ MODIFICATO / DA MODIFICARE**: file esistente con modifiche effettive / previste.
- **➕ NUOVO / DA CREARE**: file aggiunto / proposto; chiarisci che una classe nuova non e gia disponibile nel progetto o nell'SDK.
- **🗑️ ELIMINATO / DA ELIMINARE**: file rimosso / rimozione prevista.
- **↔️ RINOMINATO / DA RINOMINARE**: indica percorso precedente e nuovo; specifica se cambia anche il contenuto.
- **📖 SOLO CONTESTO**: file invariato, da mostrare e collegare soltanto in un approfondimento richiesto dall'utente.
- **❓ DA VALUTARE**: coinvolgimento non ancora deciso; non presentarlo come modifica certa.

Usa le etichette al futuro solo nei tour preventivi. Nei tour Git ricava lo stato dall'inventario; se un file contiene sia parti modificate sia contesto, mantieni l'etichetta del file e distingui i singoli punti. Per modifiche soltanto storiche, poi annullate, indica esplicitamente questo fatto.

Per ogni file da modificare spiega l'intervento concreto e l'impatto: responsabilita o comportamento che cambia, chiamanti coinvolti ed eventuali effetti su API, autenticazione, stato o errori. Solo durante un approfondimento, per ogni file di contesto mostrato spiega perche lo leggiamo e quale contratto esistente riutilizziamo; non lasciare intendere che verra riscritto. Un file invariato puo comunque ricevere chiamate diverse: distingui l'impatto comportamentale dalla modifica al sorgente.

Nel tour ordinario mostra soltanto file modificati, nuovi, eliminati, rinominati o con intervento ancora da valutare. Ometti elenchi, link, conteggi e spiegazioni dei file di solo contesto finche l'utente non seleziona Approfondisci o chiede esplicitamente dettagli. Puoi leggerne internamente i soli punti necessari a verificare impatti e contratti, senza riversarli in chat. Mantieni comunque visibili impatti, rischi e blocchi rilevanti: rinvia la visita del contesto, non le informazioni necessarie alla decisione. Nel registro distingui contesto letto da contesto mostrato; il contesto non richiesto non impedisce la copertura completa delle modifiche. Negli approfondimenti conta separatamente modifiche e contesto, senza ripetere la tappa intera. Per file futuri non creare link a righe inesistenti: indica il percorso proposto come testo e collega soltanto punti di integrazione gia presenti. Se emerge una nuova modifica, aggiorna etichetta, impatto e registro prima di proseguire.

Esempio di itinerario preventivo:

| Stato | File | Intervento o motivo della visita | Impatto |
| --- | --- | --- | --- |
| ✏️ DA MODIFICARE | bootstrap esistente (link verificato) | Collega il nuovo invio | Cambia la scelta del trasporto all'avvio |
| ➕ DA CREARE | `src/TelemetryService.ts` (percorso proposto) | Implementa l'invio autenticato | Gestisce bearer ed errori di esportazione |


La mappa ha due livelli, per evitare un elenco enorme anticipato:

- Panoramica numerata delle feature, con scopo, numero di file interessati dalle modifiche e soli prerequisiti essenziali. Non includere il conteggio del contesto nel tour ordinario. Ordina prima i concetti condivisi necessari, poi le feature dipendenti; non seguire l'ordine alfabetico o quello dei commit.
- All'ingresso di ogni feature, itinerario completo dei suoi file/hunk: `ordine | stato del file | file da aprire (link alla riga) e simbolo | intervento o motivo del contesto | impatto e hint`. Usa ID stabili come F1.1 e conserva ordine, stato e rimandi nel registro. Per file lunghi o modifiche distanti aggiungi link distinti ai punti da visitare. Dividi la visita in step brevi, senza perdere la copertura complessiva.

Scegli l'ordine per rendere comprensibile il comportamento: trigger/entry point, contratto se necessario, logica, stato/API, esito UI, errori e test. Adattalo al codice reale, includendo registrazioni, configurazione e tutte le modifiche residue. Distingui i file modificati dai punti di contesto non modificati. Se un file serve piu feature, rimanda alla prima visita e tratta solo gli hunk ancora da coprire.

Ogni hint e una frase concreta, per esempio: “Segui questo valore fino allo StoreDriver; verifica che l'annullamento lasci lo stato invariato”. Indica la prossima tappa e il motivo del passaggio. Non usare hint generici come “controlla la logica”. Mantieni in vista tappa attuale, gia visitate e rimanenti; aggiorna la mappa quando emergono collegamenti nuovi. Non usare commenti di review o findings per semplici suggerimenti di navigazione.

## Diagrammi ed estratti di codice

Usa diagrammi Mermaid o brevi estratti di codice quando chiariscono la spiegazione, anche su richiesta dell'utente. Scegli il formato in base al concetto; per modifiche semplici puo bastare il testo.

- **Mermaid:** usa blocchi `mermaid` con flowchart per flussi e decisioni, sequence diagram per interazioni e ordine delle chiamate, state diagram per transizioni di stato. Mantieni il diagramma focalizzato sulla tappa, con nomi riconducibili ai simboli letti e percorsi di errore o annullamento quando rilevanti. Rappresenta solo relazioni verificate nel codice e segnala eventuali semplificazioni.
- **Codice:** usa blocchi con il linguaggio corretto e il minimo estratto utile a spiegare il comportamento. Accompagna ogni estratto con il link al file e alla riga verificata, indicando la revisione se storica. Per un confronto mostra brevi estratti prima/dopo etichettati. Segnala omissioni e pseudocodice; non presentarli come codice completo o eseguibile.

Integra questi supporti nella tappa, spiegando cosa osservare. Mantieni i link e l'itinerario dei file come riferimento per la visita e la copertura; evita di duplicare lunghi diff o interi metodi in chat.

## Coordinatore, modelli e subagent

Mantieni il modello della chat come coordinatore e interlocutore del tour. Usa un modello veloce disponibile per compiti circoscritti, come inventari e letture mirate, quando il risultato resta verificabile. Seleziona il modello solo tramite le opzioni realmente supportate dagli strumenti della sessione; la skill non cambia da sola il modello della chat. Se la selezione non e disponibile, usa il modello corrente senza bloccare il tour e senza dichiarare un cambio non effettuato.

Puoi usare fino a due subagent contemporanei quando serve per inventari, letture mirate o feature indipendenti. Valuta il risparmio netto di token e tempo rispetto al costo di avvio, contesto duplicato e coordinamento; non delegare automaticamente per la sola ampiezza del tour. Nei compiti piccoli o con dipendenze strette lavora direttamente. Passa un contesto minimo e mirato, evita letture sovrapposte e richiedi resoconti compatti con riferimenti verificabili. Non chiedere ogni volta il permesso per questa delega gia autorizzata; rispetta i limiti della sessione. Evita ulteriori deleghe da parte dei subagent.

Assegna a ciascun subagent repository, feature, file e obiettivo preciso, con le istruzioni applicabili; aggiungi SHA immutabili, hunk e commit per i tour Git oppure piano e interventi previsti per i tour preventivi. Per un modello diverso passa un contesto nuovo e mirato, se supportato. Richiedi un resoconto con comportamento verificato, riferimenti a file/righe, copertura effettiva, errori o annullamenti rilevanti, dubbi e limiti. I subagent analizzano senza modificare codice e restituiscono risultati al coordinatore; il dialogo con l'utente resta unico.

Il coordinatore riconcilia i risultati con l'inventario completo, verifica riferimenti, contratti tra feature e conclusioni prima di presentarli. Non considera una sintesi del subagent come prova sufficiente di copertura. Approfondisce direttamente le ambiguita, le interazioni complesse e gli aspetti critici; se il modello veloce produce evidenze insufficienti, riprende l'analisi con il modello coordinatore. La velocita non giustifica omissioni o garanzie di qualita non verificate.

## Una tappa alla volta

Leggi tutti gli hunk della feature (`git diff --no-ext-diff --no-textconv --unified=5 START TARGET -- <paths>`), poi il contesto necessario da entrambe le versioni. Segui entry point, chiamanti, dipendenze e test quanto serve a spiegare il flusso completo e verificare le assunzioni. Non caricare interi repository o documenti senza necessita. Su output troncato ripeti per file o intervallo; non contarlo come letto. Usa ricerche mirate ed evita letture duplicate. Per la delega applica la strategia descritta sotto, mantenendo un unico registro di copertura gestito dal coordinatore.

Per ogni tappa mostra, normalmente entro 300 parole (dividi le feature grandi):

1. **Comportamento:** trigger concreto, prima/dopo e risultato osservabile. Se l'intento non e documentato, distingui l'inferenza.
2. **File da aprire, in ordine:** lista numerata o tabella con etichette visive, link alle righe reali e simboli da seguire. Mostra soltanto i file interessati dagli interventi; aggiungi quelli di solo contesto esclusivamente su richiesta di approfondimento, con etichetta distinta. Spiega gli impatti prima dei dettagli di navigazione. Per ogni punto indica cosa leggere, cosa e cambiato, cosa verificare e dove proseguire. Includi percorsi di errore, annullamento e conseguenze su stato/API quando pertinenti. Mostra prima/dopo con una breve spiegazione; per confronti complessi collega entrambe le versioni. Non richiedere il diff integrato.
3. **Verifica:** invarianti, regressioni plausibili, test presenti e scoperture; findings con evidenza concreta e impatto. Distingui lettura statica, test eseguiti e verifiche mancanti. Non inventare problemi per riempire la tappa.
4. **Punto di decisione:** cosa l'utente deve capire/confermare; progresso di file/hunk e commit, elementi ancora aperti. Invita a proseguire o approfondire senza interpretare il silenzio come consenso.

Prima di presentare ogni step verifica esistenza dei file collegati, simboli e numeri di riga. Se l'utente modifica il file durante il tour, ricontrolla le righe prima di riusare il link.

## Conferme guidate dopo ogni tappa

Dopo aver spiegato una tappa, usa lo strumento di domande con opzioni disponibile nella sessione, rispettandone modalita e vincoli. Presenta una domanda autosufficiente con il nome della tappa e queste scelte: **Compreso, avanti**, **Approfondisci**, **Segna un dubbio**. Se servono piu domande distinte sulla stessa tappa, raggruppale e numerale come 1/3, 2/3, 3/3; non aggiungere domande solo per raggiungere un numero. Il contatore grafico dipende dall'interfaccia e non va promesso. Se lo strumento non e disponibile o non e adatto, proponi le stesse scelte in chat.

Attendi la risposta prima di presentare la tappa successiva; una domanda asincrona non equivale a una risposta. Puoi continuare la lettura indipendente gia autorizzata mentre attendi, senza anticipare la visita. Non trattare un'opzione preselezionata, il silenzio o il tempo trascorso come conferma.

- **Compreso, avanti:** segna la tappa come discussa e prosegui; conferma la comprensione, non l'approvazione del codice o della PR.
- **Approfondisci:** resta sulla tappa e mostra soltanto il contesto necessario al punto richiesto, etichettando i file invariati come 📖 SOLO CONTESTO. Usa Mermaid o estratti di codice se aiutano, evita di ripetere l'intera tappa e poi riproponi la conferma.
- **Segna un dubbio:** registra il dubbio con la tappa e i riferimenti pertinenti, chiedendo il dettaglio se manca. Mantienilo aperto nel registro e riproponi la scelta se non e chiaro se l'utente vuole proseguire.

Se l'utente richiede un tour autonomo, ometti le attese intermedie come concordato. Alla fine raccogli separatamente la decisione umana sul codice, con esito esplicito e dubbi residui; non pubblicare approvazioni o review esterne.

## Conclusione verificabile

Non dichiarare il tour completo finche inventario del diff finale, inventario storico e registro non coincidono: tutti i file/hunk e tutti i commit devono essere contabilizzati, spiegati o esplicitamente bloccati. Gli elementi bloccati impediscono di dichiarare copertura completa. Se l'utente interrompe, riporta copertura parziale e punto di ripresa.

Prima del parere conclusivo esegui build e controlli pertinenti quando disponibili e praticabili, rispettando le istruzioni locali (in questo workspace, build Nx prima dei test). Usa una copia isolata di TARGET oppure verifica HEAD=TARGET e assenza di modifiche locali rilevanti prima di usare il checkout corrente. Risultati su altre versioni non validano TARGET. Verifica i comandi e gli effetti prima di lanciarli; non avviare automaticamente deploy, servizi esterni o script distruttivi. Un controllo fallito o non eseguibile resta esplicito. Non aggiungere test o correggere codice durante il tour senza una richiesta.

Chiudi con SHA revisionati, copertura effettiva, verifiche eseguite, blocchi e rischi residui. Formula un parere motivato: pronto per decisione umana, richiede modifiche, oppure evidenza insufficiente. Una build verde o l'assenza di findings non garantiscono assenza di bug. Se TARGET cambia, ricalcola il delta e invalida le conclusioni sulle parti coinvolte prima di riusarle.
