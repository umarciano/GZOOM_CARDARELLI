# Implementazione Sistema Valutato/Valutatore

## Panoramica
Questo documento traccia tutte le modifiche implementate per il sistema di permessi Valutato/Valutatore nei form di ricerca delle performance dei dipendenti.

## Data di Implementazione
- **Inizio**: Settembre 2025
- **Completamento**: Settembre 17, 2025

## Obiettivi del Progetto
1. Implementare auto-popolamento e controllo read-only per campo `evalPartyId` (Valutato)
2. Implementare auto-popolamento e controllo read-only per campo `evalManagerPartyId` (Valutatore)
3. Nascondere campi non necessari per utenti con permessi specifici
4. Implementare sicurezza NO_RESULT per utenti non autorizzati
5. Estendere funzionalità a menu multipli (GP_MENU_00142 e GP_MENU_00139)

## Permessi Implementati

### EMPLVALUTATO_VIEW
- **Ruolo associato**: WEM_EVAL_IN_CHARGE
- **Comportamento**: 
  - Auto-popola campo `evalPartyId` con l'utente corrente
  - Nasconde tutti i campi eccetto "Valutato" e "Stato Attuale"
  - Campo `evalPartyId` diventa read-only
  - Strategia NO_RESULT per utenti non in dropdown

### EMPLVALUTATORE_VIEW
- **Ruolo associato**: WEM_EVAL_MANAGER
- **Comportamento**:
  - Auto-popola campo `evalManagerPartyId` con l'utente corrente
  - Mostra tutti i campi del form
  - Campo `evalManagerPartyId` diventa read-only
  - Strategia NO_RESULT per utenti non in dropdown

## File Modificati

### 1. EmplPerfRootInqyViewForms.xml
**Percorso**: `gzoom-legacy/hot-deploy/emplperf/widget/EmplPerfRootInqyViewForms.xml`

**Modifiche principali**:
- Aggiunto script `checkEmplValutatoPermission.groovy` per gestione permesso Valutato
- Aggiunto script `checkEmplValutatorePermission.groovy` per gestione permesso Valutatore
- Implementato controllo visibilità campi con BSH expressions
- Configurato auto-popolamento e read-only per `evalPartyId` e `evalManagerPartyId`

### 2. EmplPerfRootViewForms.xml
**Percorso**: `gzoom-legacy/hot-deploy/emplperf/widget/EmplPerfRootViewForms.xml`

**Modifiche principali**:
- Aggiunto supporto per permesso Valutatore (GP_MENU_00139)
- Configurato script `checkEmplValutatorePermission.groovy`
- Implementato controllo visibilità e auto-popolamento per `evalManagerPartyId`

### 3. checkEmplValutatoPermission.groovy
**Percorso**: `gzoom-legacy/hot-deploy/emplperf/webapp/emplperf/WEB-INF/actions/checkEmplValutatoPermission.groovy`

**Funzionalità**:
- Verifica permesso EMPLVALUTATO_VIEW
- Auto-popola `evalPartyId` con userLogin.partyId
- Crea lista `evalPartyIdList` per dropdown
- Implementa strategia NO_RESULT per sicurezza
- Nasconde campi non necessari

### 4. checkEmplValutatorePermission.groovy
**Percorso**: `gzoom-legacy/hot-deploy/emplperf/webapp/emplperf/WEB-INF/actions/checkEmplValutatorePermission.groovy`

**Funzionalità**:
- Verifica permesso EMPLVALUTATORE_VIEW
- Auto-popola `evalManagerPartyId` con userLogin.partyId
- Crea lista `evalManagerPartyIdList` per dropdown
- Implementa strategia NO_RESULT per sicurezza
- Mantiene visibilità di tutti i campi

### 5. WorkeffortExtUiLabels.xml
**Percorso**: `gzoom-legacy/hot-deploy/workeffortext/config/WorkeffortExtUiLabels.xml`

**Modifiche**:
- Aggiunta traduzione per `IndividualCard_Referent`
- Correzione riferimenti per "Presa Visione Label"

## Modifiche Aggiuntive

### Validazione Campo weTransValue
- **File**: `WorkEffortMeasureForms.xml`
- **Modifica**: Convertito da input libero a dropdown con valori 1-5
- **Motivazione**: Garantire validazione dati corretta

### Nascondere Campo Riferimenti
- **File**: `WorkEffortMeasureForms.xml`, `GlAccountForms.xml`
- **Modifica**: Nascosti campi "Riferimenti" non necessari
- **Implementazione**: `use-when="false"` attribute

## Menu Interessati

### GP_MENU_00142
- **Form**: EmplPerfRootInqyViewForms.xml
- **Supporta**: Entrambi i permessi Valutato e Valutatore
- **Comportamento**: Campo differenziato basato su permesso utente

### GP_MENU_00139
- **Form**: EmplPerfRootViewForms.xml
- **Supporta**: Permesso Valutatore
- **Comportamento**: Auto-popolamento `evalManagerPartyId`

## Strategia di Sicurezza

### NO_RESULT Strategy
Implementata per entrambi i permessi:
1. Verifica se utente ha permesso
2. Controlla se utente è presente nel dropdown appropriato
3. Se utente non è nel dropdown ma ha permesso → ritorna lista vuota
4. Previene accesso non autorizzato ai dati

### Controlli di Visibilità
- **BSH Expressions**: Utilizzate per nascondere/mostrare campi
- **Read-Only Control**: Implementato tramite `readonly="true"`
- **Conditional Rendering**: Basato su presenza variabili nel context

## Test e Validazione

### Scenari Testati
1. ✅ Utente con EMPLVALUTATO_VIEW - auto-popolamento evalPartyId
2. ✅ Utente con EMPLVALUTATORE_VIEW - auto-popolamento evalManagerPartyId
3. ✅ Utenti senza permessi - accesso standard
4. ✅ Sicurezza NO_RESULT - prevenzione accessi non autorizzati
5. ✅ Visibilità campi - nascosti/visibili secondo permessi
6. ✅ Multi-menu support - GP_MENU_00142 e GP_MENU_00139

### Problemi Risolti
1. **Script Path Error**: Corretto percorso da `workeffortext` a `emplperf`
2. **Unwanted State Population**: Rimossa auto-popolazione stato per Valutatore
3. **Missing Dropdown List**: Aggiunta creazione `evalManagerPartyIdList`

## Configurazione Database

### Permessi Richiesti
```sql
-- Permesso per Valutato
INSERT INTO SecurityPermission VALUES ('EMPLVALUTATO_VIEW', 'View employee evaluation as evaluatee', NULL);

-- Permesso per Valutatore  
INSERT INTO SecurityPermission VALUES ('EMPLVALUTATORE_VIEW', 'View employee evaluation as evaluator', NULL);
```

### Ruoli Associati
- **WEM_EVAL_IN_CHARGE**: Collegato a EMPLVALUTATO_VIEW
- **WEM_EVAL_MANAGER**: Collegato a EMPLVALUTATORE_VIEW

## Note per Manutenzione Futura

### Considerazioni Importanti
1. **Component Boundaries**: Script devono essere nel component corretto (`emplperf`)
2. **Permission Logic**: Differenziare comportamento tra Valutato/Valutatore
3. **Security First**: Sempre implementare strategia NO_RESULT
4. **Path References**: Utilizzare percorsi corretti per script location

### Estensioni Possibili
1. Aggiungere ulteriori livelli di permessi
2. Implementare logging per accessi con permessi speciali
3. Estendere a altri menu del sistema
4. Aggiungere notifiche per auto-popolamenti

## Autori e Contributori
- **Implementazione**: GitHub Copilot
- **Richiesta**: Team GZOOM
- **Data**: Settembre 2025

---
*Questo documento deve essere aggiornato ad ogni modifica del sistema Valutato/Valutatore*

## Nuova Evolutiva: Filtraggio Stampe per Valutato (GP_MENU_00208)

### Data di Implementazione
- **Inizio**: Settembre 17, 2025
- **Completamento**: Settembre 17, 2025

### Obiettivo
Implementare il controllo di visibilità per il menu **GP_MENU_00208** (Stampe) in modo che gli utenti con permesso **EMPLVALUTATO_VIEW** vedano solo la voce "Stampa scheda Obiettivi" e non "Lista Valutazioni Individuali".

### Analisi Tecnica
- **Menu**: GP_MENU_00208 punta a `/emplperf/control/workEffortPrintBirt`
- **Screen**: `EmplPerfPrintBirt` include `WorkEffortPrintBirt` da workeffortext
- **Contesto**: `WE_PRINT_SCHEDA_IND` per emplperf
- **WorkEffortType**: `VD-12` per valutazioni individuali
- **Script principale**: `getWorkEffortPrintBirtList.groovy` popola la lista dei report

### Report Identificati
- **REPORT_SOO**: "SchedaObiettiviOrganizzativi" - "Stampa scheda Obiettivi" ✅ VISIBILE per Valutato
- **REPORT_SLVI**: "SchedaListaValutazioniIndividuali" - "Lista Valutazioni Individuali (ELI4U)" ❌ NASCOSTO per Valutato
- **REPORT_LVI**: "ListaValutazioniIndividuali" - "Lista Valutazioni Individuali" ❌ NASCOSTO per Valutato
- **REPORT_LVI_STA**: "ListaValutazioniIndividuali" - "Statistiche valutazioni" ❌ NASCOSTO per Valutato
- **REPORT_LVI_RIE**: "ListaValutazioniIndividuali" - "Riepilogo valutazioni" ❌ NASCOSTO per Valutato

### File Modificati

#### 1. getPrintBirtWorkEffortTypeList.groovy (CORREZIONE FINALE)
**Percorso**: `hot-deploy/base/webapp/common/WEB-INF/actions/getPrintBirtWorkEffortTypeList.groovy`

**Problema identificato**: Dall'analisi dei log è emerso che il sistema utilizza `getPrintBirtWorkEffortTypeList.groovy` per popolare `context.listReport`, non `getPrintBirtList.groovy`.

**Log di conferma**:
```
******************************* getPrintBirtWorkEffortTypeList.groovy -> context.listReport = [
  [contentId:REPORT_LVI, ...], 
  [contentId:REPORT_SOO, ...]
]
```

**Modifiche**:
- Aggiunto import per `org.ofbiz.security.Security`
- Implementata logica di filtering per permesso `EMPLVALUTATO_VIEW`
- Lista di exclusion per report non autorizzati: `["REPORT_SLVI", "REPORT_LVI", "REPORT_LVI_STA", "REPORT_LVI_RIE"]`
- Logging per debugging e monitoring

**Codice implementato**:
```groovy
// Controllo permessi Valutato: se l'utente ha il permesso EMPLVALUTATO_VIEW, 
// mostra solo il report REPORT_SOO e nasconde gli altri
if (context.listReport && userLogin) {
    if (security && security.hasPermission("EMPLVALUTATO_VIEW", userLogin)) {
        Debug.log("******************************* getPrintBirtWorkEffortTypeList.groovy -> Utente Valutato rilevato, applicando filtri");
        
        // Lista dei report da escludere per gli utenti Valutato
        def excludedReports = ["REPORT_SLVI", "REPORT_LVI", "REPORT_LVI_STA", "REPORT_LVI_RIE"];
        
        // Filtra la lista mantenendo solo i report consentiti
        def filteredList = [];
        context.listReport.each { report ->
            if (!excludedReports.contains(report.contentId)) {
                filteredList.add(report);
                Debug.log("******************************* getPrintBirtWorkEffortTypeList.groovy -> Report consentito: " + report.contentId);
            } else {
                Debug.log("******************************* getPrintBirtWorkEffortTypeList.groovy -> Report nascosto: " + report.contentId);
            }
        }
        
        context.listReport = filteredList;
    }
}
```

#### 2. workeffortPrintBirtBaseParameters.ftl (MIGLIORAMENTO UX)
**Percorso**: `hot-deploy/workeffortext/webapp/workeffortext/birt/ftl/workeffortPrintBirtBaseParameters.ftl`

**Problema identificato**: I filtri non apparivano automaticamente quando la pagina si caricava con un radio button già selezionato, richiedendo un click manuale.

**Modifiche UX**:
- Migliorato il selettore JavaScript per trovare il radio button selezionato
- Aggiunto fallback multipli: `input:checked`, `input[checked="true"]`, `input[type="radio"]`
- Implementato doppio trigger: `document.observe` + `setTimeout` per garantire l'esecuzione
- Aggiunto logging console per debugging

**Codice migliorato**:
```javascript
load: function() {
    // ... existing code ...
    var selectPrintRow = $("select-print-row");
    if (selectPrintRow != null) {
        // Prova diversi selettori per trovare il radio button selezionato
        var list = selectPrintRow.select('input:checked');
        if(list === undefined || list.length === 0) {
            list = selectPrintRow.select('input[checked="true"]');
        }
        if(list === undefined || list.length === 0) {
            // Fallback: prendi il primo radio button se nessuno è marcato come checked
            list = selectPrintRow.select('input[type="radio"]');
        }
        if(list !== undefined && list.length > 0) {    
            console.log("Auto-triggering radioOnChange for: " + list[0].value);
            WorkEffortPrintBirtExtraParameter.radioOnChange(list[0]);
        }
    }
}

// Utilizziamo sia document.observe che setTimeout per assicurarci che funzioni
document.observe("dom:loaded", function() {
    jQuery(WorkEffortPrintBirtExtraParameter.load);
    setTimeout(function() {
        WorkEffortPrintBirtExtraParameter.load();
    }, 100);
});
```
**Percorso**: `hot-deploy/emplperf/webapp/emplperf/WEB-INF/actions/checkEmplValutatoPrintPermission.groovy`

**Funzionalità**:
- Script di supporto per controlli aggiuntivi sui permessi di stampa
- Imposta variabili di contesto per la gestione UI
- Lista dei report esclusi per il Valutato

### Flusso di Rendering Identificato (FINALE)
1. **Menu GP_MENU_00208** → `/emplperf/control/workEffortPrintBirt`
2. **Screen EmplPerfPrintBirt** → include `WorkEffortPrintBirt` 
3. **Action**: `getPrintBirtWorkEffortTypeList.groovy` ✅
4. **Variable**: `context.listReport` popolata con i report
5. **Template**: usa `listReport` per generare i radio button

**Note**: I tentativi precedenti di modifica in `getWorkEffortPrintBirtList.groovy` e `getPrintBirtList.groovy` non erano effettivi perché il sistema utilizza `getPrintBirtWorkEffortTypeList.groovy` come confermato dai log di sistema.

#### 3. checkEmplValutatoPrintPermission.groovy (Script di supporto)
**Percorso**: `hot-deploy/emplperf/webapp/emplperf/WEB-INF/actions/checkEmplValutatoPrintPermission.groovy`

**Funzionalità**:
- Script di supporto per controlli aggiuntivi sui permessi di stampa
- Imposta variabili di contesto per la gestione UI
- Lista dei report esclusi per il Valutato

### Comportamento Implementato

#### Per Utenti con EMPLVALUTATO_VIEW:
- ✅ **Visibili**: Solo "Stampa scheda Obiettivi" (REPORT_SOO)
- ❌ **Nascosti**: Tutti i report di "Lista Valutazioni Individuali" (REPORT_SLVI, REPORT_LVI, REPORT_LVI_STA, REPORT_LVI_RIE)
- ✅ **UX**: Filtri automaticamente visibili al caricamento pagina

#### Per Altri Utenti:
- ✅ **Visibili**: Tutti i report configurati (comportamento standard)
- ✅ **UX**: Filtri automaticamente visibili al caricamento pagina

### Strategia di Sicurezza
- **Filtering a livello di script**: La logica è integrata nel script principale che popola la lista
- **Controllo permessi**: Utilizza `security.hasPermission("EMPLVALUTATO_VIEW", userLogin)`
- **Lista esclusione**: Definita in modo esplicito per controllo granulare
- **Logging**: Implementato per monitoring e debugging

### Estensibilità
- La logica può essere facilmente estesa ad altri permessi
- La lista di exclusion può essere configurata dinamicamente
- Supporta filtering per multiple tipologie di utenti

### Note per Manutenzione
- **Pattern consistente**: Segue lo stesso pattern implementato per i form di ricerca
- **Sicurezza first**: Il filtering è applicato server-side
- **Performance**: Filtering applicato solo quando necessario
- **Logging dettagliato**: Per troubleshooting e auditing

### Test di Validazione
1. ✅ Utente con EMPLVALUTATO_VIEW vede solo "Stampa scheda Obiettivi"
2. ✅ Utente con EMPLVALUTATO_VIEW NON vede "Lista Valutazioni Individuali"
3. ✅ Utenti senza permesso vedono tutti i report (comportamento standard)
4. ✅ Logging funziona correttamente per debugging
5. ✅ Filtri automaticamente visibili al caricamento pagina
6. 🔄 Utente Valutato vede solo campo "Scheda" nei filtri
7. 🔄 Dropdown "Scheda" mostra solo schede dell'utente loggato per Valutato

### Gestione Filtri per Utenti Valutato

#### Obiettivo Filtri
Per gli utenti con permesso **EMPLVALUTATO_VIEW**, i filtri della sezione stampe devono essere limitati a:
- ✅ **Visibile**: Solo campo "Scheda" 
- ❌ **Nascosti**: Tipo obiettivo, Data al, Elemento di valutazione, Modello valutazione, Unità Responsabile, Ruolo, Soggetto, Stato Attuale
- 🔄 **Filtraggio**: Dropdown "Scheda" mostra solo schede assegnate all'utente loggato

#### File Modificati per Filtri

##### 1. checkEmplValutatoFiltersPermission.groovy (NUOVO)
**Percorso**: `hot-deploy/emplperf/webapp/emplperf/WEB-INF/actions/checkEmplValutatoFiltersPermission.groovy`

**Funzionalità**:
- Identifica utenti con permesso `EMPLVALUTATO_VIEW`
- Imposta variabili di contesto per nascondere filtri
- Prepara `userPartyId` per filtering della dropdown Scheda
- Abilita l'uso di `WorkEffortAndWorkEffortPartyAssView` per filtrare

**Variabili di contesto create**:
```groovy
context.isEmplValutato = true/false;
context.hideAllFiltersExceptScheda = true/false;
context.useWorkEffortPartyView = true/false;
context.userPartyId = "partyId_utente";
```

##### 2. EmplPerfScreens.xml (MODIFICATO)
**Percorso**: `hot-deploy/emplperf/widget/screens/EmplPerfScreens.xml`

**Modifica**: Aggiunto script `checkEmplValutatoFiltersPermission.groovy` allo screen `EmplPerfExtraParametersPrintBirt`

##### 3. SchedaIndividuale.ftl (MODIFICATO)
**Percorso**: `hot-deploy/emplperf/webapp/emplperf/ftl/SchedaIndividuale.ftl`

**Modifiche implementate**:

1. **Campi nascosti per Valutato**: Tutti i campi tranne "Scheda" sono nascosti con `<#if !hideAllFiltersExceptScheda?default(false)>`

2. **Filtraggio dropdown Scheda**: 
   - Entità diversa per Valutato: `WorkEffortAndWorkEffortPartyAssView` invece di `WorkEffortView`
   - Constraint specifiche: filtra per `partyId`, `roleTypeId=EMPLOYEE`, `thruDate=null`

**Codice chiave**:
```freemarker
<#-- Entità diversa per utenti Valutato per filtrare le schede -->
<#if useWorkEffortPartyView?default(false)>
    <input class="autocompleter_parameter" type="hidden" name="entityName" value="[WorkEffortAndWorkEffortPartyAssView]"/>
<#else>
    <input class="autocompleter_parameter" type="hidden" name="entityName" value="[WorkEffortView]"/>
</#if>

<#if isEmplValutato?default(false) && userPartyId?has_content>
    <!-- Constraint per utenti Valutato: mostra solo schede dove l'utente è assegnato -->
    <input class="autocompleter_parameter" type="hidden" name="constraintFields" value="[[[isTemplate| equals| N]! [isRoot| equals| Y]! [workEffortSnapshotId| equals| [null-field]]! [partyId| equals| ${userPartyId}]! [roleTypeId| equals| EMPLOYEE]! [thruDate| equals| [null-field]]! ...]]"/>
</#if>
```

#### Entità Utilizzata
**WorkEffortAndWorkEffortPartyAssView**: Vista che combina WorkEffort con WorkEffortPartyAssignment, permettendo di filtrare le schede in base all'assegnazione degli utenti.

#### Comportamento Finale Filtri

##### Per Utenti Valutato (EMPLVALUTATO_VIEW):
- ✅ **Campo visibile**: Solo "Scheda" 
- ✅ **Dropdown filtrata**: Solo schede dove l'utente è assegnato con ruolo EMPLOYEE
- ❌ **Campi nascosti**: Tutti gli altri 8 filtri

##### Per Altri Utenti:
- ✅ **Tutti i campi visibili**: Standard behavior 
- ✅ **Dropdown completa**: Tutte le schede secondo i constraint normali

---

## 🎯 IMPLEMENTAZIONE PORTALE "MIE PERFORMANCE" (Settembre 2025)

### Obiettivo
Implementare un portale dedicato per consentire ai dipendenti di visualizzare le proprie schede di valutazione in modalità **read-only** e solo quando sono nello stato "**Valutazione Conclusa**".

### Funzionalità Implementate

#### 1. **Configurazione Menu e Portale**
- **Voce Menu**: "Mie performance" (sostituita la label tecnica NOPORTAL_MY)
- **Portale ID**: `GP_WE_PORTAL_3`
- **Gruppo Sicurezza**: `NOPORTAL_MY`
- **Accesso**: Solo utenti con gruppo di sicurezza NOPORTAL_MY

#### 2. **Modalità Read-Only Automatica**
Quando si accede dal portale "Mie performance", tutti i campi delle schede diventano automaticamente non modificabili:

**Rilevamento Portale**:
- Analisi dell'URL e referrer HTTP per identificare accesso da `GP_WE_PORTAL_3`
- Gestione della sessione per mantenere lo stato read-only durante la navigazione
- Propagazione del parametro `forceReadOnly=Y` attraverso le tab

**Enforcement Read-Only**:
- Form principali: campi input disabilitati
- Indicatori di performance: righe non modificabili
- Pulsanti azione: nascosti o disabilitati

#### 3. **Filtro Stato Schede**
Nel portale "Mie performance" vengono mostrate **SOLO** le schede nello stato:
- **`WEEVALST_EXECFINAL`** - "Valutazione Conclusa"

Questo garantisce che il dipendente possa vedere le proprie schede solo dopo che il valutatore ha completato la valutazione.

### File Modificati

#### 1. **JavaScript - Iniezione Parametri**
**File**: `WorkEffortMyPerformanceSummary-list-extension.js.ftl`
```javascript
// Rileva click sul portale GP_WE_PORTAL_3 e aggiunge parametri read-only
if (portalPageId === 'GP_WE_PORTAL_3') {
    newUrl += "&forceReadOnly=Y&managementFormType=view";
}
```

#### 2. **Script Groovy - Rilevamento Portale**
**File**: `checkPortalReadOnlyMode.groovy` (NUOVO)
```groovy
// Rileva accesso da portale GP_WE_PORTAL_3
// Analizza URL, queryString e referrer HTTP
// Imposta flag read-only in sessione
```

#### 3. **Script Groovy - Validazione Campi**
**File**: `checkWorkEffortViewFormReadOnly.groovy` (NUOVO)
```groovy
// Imposta isWorkEffortViewFormReadOnly = Y per portale
context.isWorkEffortViewFormReadOnly = "Y"
```

#### 4. **Script Groovy - Controllo Righe**
**File**: `isRowReadOnlyWorkEffortMeasure.groovy` (NUOVO)
```groovy
// Disabilita modifiche alle righe degli indicatori
context.isRowReadOnlyWorkEffortMeasure = true
```

#### 5. **Script Groovy - Filtro Stato**
**File**: `checkPortalMyPerformanceFilter.groovy` (NUOVO)
```groovy
// Filtra schede per stato WEEVALST_EXECFINAL quando accesso da portale
if (isMyPerformancePortal) {
    context.currentStatusId = "WEEVALST_EXECFINAL";
}
```

#### 6. **Schermate XML - Integrazione Script**
**File**: `SubFolderManagementContainerOnlyScreen` e `WorkEffortMeasureIndicatorDetailTransactionPanel`
- Aggiunta inclusione script di controllo read-only
- Copertura completa di tutte le schermate di navigazione

#### 7. **Portale Screen - Filtro Query**
**File**: `WorkeffortExtScreens.xml`
- Modifica schermata `WorkEffortMyPerformanceSummaryListScreen`
- Inclusione script filtro stato prima della query
- Query limitata alle schede con `currentStatusId = WEEVALST_EXECFINAL`

### Configurazione Label
**File**: `it_IT.json` (come risolto dall'utente)
- Aggiunta traduzione italiana "Mie performance" per chiave NOPORTAL_MY

### Comportamento Sistema

#### **Accesso Normale** (da menu standard)
- ✅ **Campi**: Tutti modificabili secondo permessi utente
- ✅ **Schede**: Visibili in tutti gli stati
- ✅ **Funzionalità**: Complete (salvataggio, modifica, ecc.)

#### **Accesso Portale** (da "Mie performance")
- 🔒 **Campi**: Tutti in read-only (non modificabili)
- 🔍 **Schede**: Solo quelle in stato "Valutazione Conclusa"
- 👁️ **Modalità**: Solo visualizzazione
- ✅ **UOC**: Solo schede della propria Unità Operativa Complessa

### Flusso Valutazione
1. **Valutatore** completa la scheda → stato diventa `WEEVALST_EXECFINAL`
2. **Dipendente** accede al portale "Mie performance"
3. **Sistema** mostra la scheda completata in read-only
4. **Dipendente** può consultare la propria valutazione senza modificarla

### Sicurezza
- **Isolamento**: Portale completamente separato dal sistema normale
- **Autorizzazione**: Solo utenti gruppo NOPORTAL_MY
- **Read-Only**: Impossibile modificare dati accidentalmente
- **Filtraggio**: Solo schede proprie e completate

---
