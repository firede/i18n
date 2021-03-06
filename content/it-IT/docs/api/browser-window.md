# BrowserWindow

> Crea e controlla le finestre del browser.

Processo: [Main](../glossary.md#main-process)

```javascript
// Nel processo principale(main).
const {BrowserWindow} = require('electron')

// O usa 'remote' dai processi render.
// const {BrowserWindow} = require('electron').remote

let win = new BrowserWindow({width: 800, height: 600})
win.on('closed', () => {
  win = null
})

// Carica un URL remoto
win.loadURL('https://github.com')

// O carica un file HTML
win.loadURL(`file://${__dirname}/app/index.html`)
```

## Finestra senza bordi

Per creare una finestra senza chrome, o una finestra trasparente in forma arbitraria, puoi usare l'API [Frameless Window](frameless-window.md).

## Mostrare la finestra in maniera elegante

Quando si carica una pagina direttamente nella finestra, l'utente potrebbe vedere la pagina caricare in modo incrementale, che non è una esperienza buona per un'app nativa. Per mostrare la finestra senza flash visuale esistono due soluzioni per due differenti situazioni.

### Utilizzare l'evento `ready-to-show`

Durante il caricamento della pagina, l'evento `ready-to-show` verrà emesso quando il Renderer Process ha eseguito il rendering della pagina per la prima volta e se la finestra non è stata ancora visualizzata. Mostrare la finestra dopo questo evento non mostrerà flash visuali:

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow({show: false})
win.once('ready-to-show', () => {
  win.show()
})
```

Questo evento è di solito emesso dopo l'evento `did-finish-load`, ma per le pagine con molte risorse potrebbe essere emesso prima di `did-finish-load`.

### Impostare il colore di sfondo(`backgroundColor`)

Per un'app complessa, l'evento `ready-to-show` potrebbe essere emessa troppo tardi rendendo l'app lenta. In questo caso, è raccomandato mostrare la finestra immediatamente ed usare un colore di sfondo(`backgroundColor`) simile a quello della tua app:

```javascript
const {BrowserWindow} = require('electron')

let win = new BrowserWindow({backgroundColor: '#2e2c29'})
win.loadURL('https://github.com')
```

Nota come anche per le app è usato l'evento `ready-to-show`, è raccomandato impostare il `backgroundColor` per far sembrare le app più native.

## Finestra padre e figlio

Usando l'opzione `parent`, puoi creare finestre figlie, impostando una relazione gerarchica tra le finesre:

```javascript
const {BrowserWindow} = require('electron')

let top = new BrowserWindow()
let child = new BrowserWindow({parent: top})
child.show()
top.show()
```

La finestra `child` sarà sempre mostrata sopra la finestra `top`.

### Finestre modali

Una finestra modale è una finestra figlia che disabilita le finestre padri, per crearne una devi impostare entrambe le opzioni `parent` e `modal`:

```javascript
const {BrowserWindow} = require('electron')

let child = new BrowserWindow({parent: top, modal: true, show: false})
child.loadURL('https://github.com')
child.once('ready-to-show', () => {
  child.show()
})
```

### Visibilità pagina

L' [Api di Visibilità Pagina](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API) lavora come segue:

* Su tutte le piattaforme, lo stato di visibilità traccia se la finestra è nascosta/minimizzata o no.
* In aggiunta, su macOS, lo stato di visibilità traccia anche lo stato di occlusione della finestra. Se la finestra è occlusa (totalmente coperta) da un'altra, lo stato di visibilità sarà nascosta (`nascosta`). Su altre piattaforme, lo stato di visibilità sarà `hidden` solo quando la finestra è minimizzata o nascosta esplicitamente con `win.hide()`.
* Se una nuova `BrowserWindow` è stata creata con `show: false`, lo stato di visibilità iniziale sarà `visibile` nonostante la finestra risulti essere nascosta.
* Se il `backgroundThrottling` è disabilitato, lo stato di visibilità rimarrà `visible` anche se la finestra è minimizzata, occlusa o nascosta.

Si raccomanda di mettere in pausa le operazioni dispendiose quando lo stato di visibilità è `hidden` per minimizzare il consumo energetico.

### Avvisi di piattaforma

* Su macOS le finestre modali saranno mostrate come fogli allegate alla finestra genitore.
* Su macOS le finestre figlie manterranno le proprie posizioni relative alla finestra genitore quando questa si muove, mentre su Windows e Linux queste non si muoveranno.
* Su Windows non è supportato il cambiamento dinamico delle finestre genitori.
* Su Linux il tipo di finestre modali sarà cambiato in `dialog`.
* Su Linux molti ambienti desktop non supportano il nascondere una finestra modale.

## Classe: BrowserWindow

> Crea e controlla le finestre del browser.

Processo: [Main](../glossary.md#main-process)

`BrowserWindow` è un [EventEmitter](https://nodejs.org/api/events.html#events_class_events_eventemitter).

Crea una nuova Finestra `BrowserWindow` con proprietà native come da `options`.

### `new BrowserWindow([options])`

* `options` Object (opzionale) 
  * `width` Intero (opzionale) - La larghezza in pixel della finestra. Di default è di `800`.
  * `height` Intero (opzionale) - L'altezza in pixel della finestra. Di default è di `600`.
  * `x` Intero (opzionale) (**richiesto** se è usato y) - Offset sinistro della finestra dallo schermo. Di default è al centro della finestra.
  * `y` Intero (opzionale) (**richiesto** se è usato x) - L'offset superiore della finestra dallo schermo. Di default è al centro della finestra.
  * `useContentSize` Booleano (opzionale) - La `width` e l'`height` saranno usate come dimensioni della pagina web, il che vuol dire che la dimensione attuale della finestra includerà le dimensioni della cornice della finestra ed è lievemente più grande. Di default è `false`.
  * `center` Boolean (opzionale) - Mostra la finestra al centro dello schermo.
  * `minWidth` Intero (opzionale) - Larghezza minima della finestra. Di default è `0`.
  * `minHeight` Intero (opzionale) - Altezza minima della finestra. Di default è `0`.
  * `maxWidth` Intero (opzionale) - Larghezza massima della finestra. Di default non ha limiti.
  * `maxHeight` Intero (opzionale) - Altezza massima della finestra. Di default è senza limiti.
  * `resizable` Boolean (opzione) - Se la finestra è ridimensionabile. Di default è `true`.
  * `movable` Boolean (opzionale) - Se la finestra è mobile. Non è implementato su Linux. Di default è `true`.
  * `minimizable` Boolean (opzionale) - Se la finestra è minimizzabile. Non implementato su Linux. Di default è `true`.
  * `maximizable` Boolean (opzionale) - Se la finestra è massimizzabile. Non implementato su Linux. Di default è `true`.
  * `closable` Boolean (opzionale) - Se la finestra è chiudibile. Non implementato su Linux. Di default è `true`.
  * `focusable` Boolean (opzionale) - Se la finestra è focalizzabile. Di default è `true`. Su Windows impostando `focusable: false` implica anche l'impostazione `skipTaskbar: true`. Su Linux, impostando `focusable: false` blocca l'interazione della finestra con il wm, così la finestra resterà sempre in primo piano rispetto alle aree di lavoro.
  * `alwaysOnTop` Boolean (opzionale) - Se la finestra dovrebbe sempre rimanere al top delle altre finestre. Di default è `false`.
  * `fullscreen` Boolean (opzionale) - Se la finestra dovrebbe mostrarsi a schermo intero. Quando esplicitamente impostati a `false` il pulsante schermo intero sarà nascosto o disabilitato su macOS. Di default è `false`.
  * `fullscreenable` Boolean (opzionale) - Se la finestra è impostabile in modalità schermo intero. Su macOS, anche se il pulsante massimizza/ingrandisci potrebbe impostare la modalità schermo intero o massimizza finestra. Il valore predefinito è `true`.
  * `simpleFullscreen` Boolean (opzionale) - Modalità a schermo intero su macOS pre-Lion. Default è `false`.
  * `skipTaskbar` Boolean (opzionale) - Se mostrare la finestra nella taskbar. Di default è `false`.
  * `kiosk` Boolean (opzionale) - Modalità kiosk. Di default è `false`.
  * `title` String (opzionale) Titolo di default della finestra. Di default è `"Electron"`.
  * `icon` ([>NativeImage](native-image.md) | Stringa) (opzionale) - L'icona della finestra. Su Windows si raccomanda di usare le icone `ICO` per ottenere migliori effetti visuali, puoi anche lasciarlo non impostato, così sarà usata l'icona dell'eseguibile.
  * `show` Boolean (opzionale) - Se la finestra deve essere visualizzata quando creata. Di default è `true`.
  * `frame` Boolean (opzionale) - Specifica `false` per creare una [Finestra senza bordi](frameless-window.md). Di default è `true`.
  * `parent` BrowserWindow (opzionale) - Specifica la finestra genitore. Di default è `null`.
  * `modal` Boolean (opzionale) - Se si tratta di una finestra modale. Funziona solo se la finestra è figlia. Di default è `false`.
  * `acceptFirstMouse` Boolean (opzionale) - Se la web view accetta un singolo evento mouse-down che simultaneamente attiva la finestra. Di default è `false`.
  * `disableAutoHideCursor` Boolean (opzionale) - Se nascondere il cursore in digitazione. Di default è `false`.
  * `autoHideMenuBar` Boolean (opzionale) - Nascondi automaticamente la barra dei menu senza che il tasto `Alt` sia premuto. Di default è `false`.
  * `enableLargerThanScreen` Boolean (opzionale) - Abilita la finestra ad un ridimensionamento più elevato dello schermo. Di default è `false`.
  * `backgroundColor` String (opzionale) - Colore di sfondo della finestra rappresentato come valore esadecimale, come `#66CD00` o `#FFF` o `#80FFFFFF` (la trasparenza è supportata). Di default è `#FFF` (bianco).
  * `hasShadow` Boolean (opzionale) - Specifica se la finestra debba supportare l'ombreggiamento. Questa impostazione è solo implementata per macOS. Di default è `true`.
  * `opacity` Number (opzionale) - Imposta l'opacità iniziale della finestra, tra 0.0 (completamente trasparente) e 1.0 (completamente opaco). Questo è implementato solo su Windows e macOS.
  * `darkTheme` Boolean (opzionale) - Forza l'utilizzo del tema scuro per la finestra, funziona solo su alcuni ambienti desktop GTK+3. Di default è `false`.
  * `transparent` Boolean (opzionale) - rende la finestra [trasparente](frameless-window.md). Valore predefinito è `false`.
  * `type` String (opzionale): il tipo di finestra, impostazione predefinita è normale finestra. Vedi di più su questo qui sotto.
  * `titleBarStyle` String (opzionale) - lo stile della barra del titolo della finestra. Impostazione predefinita è `default`. I valori possibili sono: 
    * `default` - risultato di colore grigio opaco come barra del titolo del Mac.
    * `hidden` - genera una barra del titolo nascosta e una finestra a tutto schermo per il contenuto e inoltre la barra del titolo contiene nell'angolo in alto a sinistra i tipici controlli per la finestra ("semaforo").
    * `hiddenInset` - Nasconde la barra del titolo, permettendone un aspetto alternativo. I pulsanti a semaforo sono leggermente inseriti verso il bordo della finestra.
    * `customButtonsOnHover` Boolean (opzionale) - Permette di creare bottoni di chiusura, riduci a icona, e schermo intero personalizzati per finestre senza bordo su macOS. Questi pulsanti non verranno visualizzati se non si posiziona il puntatore del mouse in alto a sinistra nella finestra. Questi pulsanti personalizzati prevengono problemi con gli eventi del mouse che si verificano con lo i pulsanti standard della barra degli strumenti di una finestra. **Note:** Questa opzione è attualmente sperimentale.
  * `fullscreenWindowTitle` Boolean (opzionale) - Mostra il titolo nella barra del titolo in modalità a schermo intero su macOS per tutte le opzioni di `titleBarStyle`. Il valore predefinito è `false</ 0>.</li>
<li><code>thickFrame` Boolean (opzionale) - Usa lo stile `WS_THICKFRAME` per finestre senza bordi su Windows, che aggiunge un bordo standard. Impostandolo a `false` le animazioni e le ombre della finestra. Il valore predefinito è `true`.
  * `vibrancy` String (opzionale) - Aggiunge un effetto di trasparenza sulla finestra, solo su macOS. Può essere `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light` or `ultra-dark`. Si prega di notare che utilizzando `frame: false` in combinazione con un valore di trasparenza è necessario utilizzare anche il non predefinito `titleBarStyle</ 0>.</li>
<li><code>zoomToPageWidth` Boolean (optional) - Controls the behavior on macOS when option-clicking the green stoplight button on the toolbar or by clicking the Window > Zoom menu item. If `true`, the window will grow to the preferred width of the web page when zoomed, `false` will cause it to zoom to the width of the screen. This will also affect the behavior when calling `maximize()` directly. Di default è `false`.
  * `tabbingIdentifier` String (opzionale) - Nome del gruppo di schede, permette la finestra come scheda nativa di macOS da 10.12+. Windows con lo stesso identificatore di scheda verrà raggruppato insieme. Questo aggiunge anche un nuovo pulsante nativo per una nuova scheda alla barra delle schede, consentendo l'`app` e la finestra di ricevere l'evento `new-window-for-tab`.
  * `webPreferences` Object (opzionale) - Impostazioni delle funzionalità della pagina web. 
    * `devTools` Boolean (opzionale) - Consente di abilitare gli strumenti di sviluppo. Se impostato su `false`, non sarà possibilite usare `BrowserWindow.webContents.openDevTools()` per aprire gli strumenti di sviluppo. Il valore predefinito è `true`.
    * `nodeIntegration` Boolean (opzionale) - Abilita le integrazioni con Node. Il valore predefinito è `true`.
    * `nodeIntegration` Boolean (opzionale) - Abilita le integrazioni con Node. Il valore predefinito è `true`. Il valore predefinito è `false`. Maggiori informazioni possono essere trovate su [Multithreading](../tutorial/multithreading.md).
    * `preload` String (opzionale) - Specifica uno script che verrà caricato prima che vengano eseguiti gli script della pagina. Questi script avranno sempre accesso alle API di Node, non importa se l'integrazione con Node è attivata o disattivata. Il valore dovrebbe essere il percorso assoluto del file allo script. Quando l'integrazione di Node è disattivata, lo script di `preload` può reintrodurre dei simboli di portata globale. Vedi l'esempio [qua](process.md#event-loaded).
    * `sandbox` Boolean (opzionale) - Se impostato, questo renderà sandbox il renderer associato alla finestra, rendendolo compatibile con Chromium Sandbox a livello di sistema operativo e disabilita il motore di Node.js. Questo non è uguale all'opzione `nodeIntegration` e le API disponibili per lo script di precaricamento sono più limitati. Maggiori informazioni sull'opzione [qui](sandbox-option.md). **Nota:** Questa opzione è attualmente sperimentale e potrebbe cambiare o essere rimossa nelle versioni future di Electron.
    * `session` [Session](session.md#class-session) (opzionale) - Imposta la sessione utilizzata dalla pagina. Invece di passare direttamente l'oggetto Session, puoi anche scegliere di usare l'opzione ` partition `, che accetta una stringa di partizione. Quando sia la `session` sia la `partition` sono fornite, la `session` sarà preferita. Di default è la default session.
    * `partition` String (opzionale) - Imposta la sessione utilizzata dalla pagina in base alla stringa di partizione della sessione. Se `partition` inizia con `persist:`, la pagina userà una sessione persistente disponibile per tutte le pagine dell'app con la stessa `partition`. Se non c'è un prefisso `persist: `, la pagina userà una sessione in memoria. Assegnando la stessa `partition`, è possibile condividere per più pagine la stessa sessione. Di default è la default session.
    * ` affinity ` String (opzionale) - Se specificato, le pagine web con lo stesso ` affinity ` verranno eseguite nello stesso processo di rendering. Si noti che a causa del riutilizzo il processo di rendering, anche alcune opzioni ` webPreferences ` saranno condivise tra le pagine Web anche quando hai specificato valori diversi per loro, inclusi, ma non limitati da ` preload`, ` sandbox ` e ` nodeIntegration `. Pertanto, si consiglia di utilizzare esattamente le stesse ` WebPreferences` per le pagine Web con la stessa `affinity`. *This property is experimental*
    * `zoomFactor` Number (opzionale) - Il fattore di zoom predefinito della pagina, ` 3.0 ` rappresenta il ` 300%`. L'impostazione predefinita è ` 1.0 `.
    * ` javascript ` Boolean (opzionale) - Abilita il supporto JavaScript. L'impostazione predefinita è ` true `.
    * ` webSecurity ` Boolean (opzionale) - Quando è ` false `, esso disabiliterà la politica della stessa origine (di solito utilizzando siti Web di test da parte di persone), e impostata ` allowRunningInsecureContent ` a ` true ` se questa opzione non è stata impostata dall'utente. Il valore predefinito è `true`.
    * ` allowRunningInsecureContent ` Boolean (facoltativo): consente l'esecuzione da una pagina Https: JavaScript, CSS o plugin da URL http. Il valore predefinito è ` falso `.
    * `images` Boolean (opzionale) - Abilita il support alle immagini. Il valore predefinito è `true`.
    * ` textAreasAreResizable ` Boolean (opzionale) - Rende ridimensionabili gli elementi TextArea. Il valore predefinito è ` true`.
    * ` webgl ` Boolean (opzionale) - Abilita il supporto WebGL. L'impostazione predefinita è ` true `.
    * ` webaudio ` Boolean (opzionale) - Abilita il supporto WebAudio. L'impostazione predefinita è ` true `.
    * ` plugins ` Boolean (opzionale) - Se i plug-in devono essere abilitati. Il valore predefinito è ` false`.
    * ` experimentalFeatures ` Boolean (opzionale): abilita le funzionalità sperimentali di Chromium. Il valore predefinito è ` false`.
    * ` experimentalCanvasFeatures ` Boolean (facoltativo): abilita i canvas sperimentali di Chromium. Il valore predefinito è ` false`.
    * ` scrollBounce ` Boolean (opzionale) - Abilita l'effetto bounce di scorrimento (effetto gomma) su macOS. Il valore predefinito è ` false`.
    * ` BlinkFeatures ` String (opzionale) - Un elenco di stringhe di feature separate da `,`, come ` CSSVariables, KeyboardEventKey ` da abilitare. L'elenco completo delle stringe supportate si possono trovare nel file [ RuntimeEnabledFeatures.json5 ](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/platform/runtime_enabled_features.json5?l=70).
    * ` disableBlinkFeatures ` String (opzionale) - Un elenco di stringhe di feature separate da `, `, come ` CSSVariables, KeyboardEventKey ` da disabilitare. L'elenco completo delle stringe supportate si possono trovare nel file [ RuntimeEnabledFeatures.json5 ](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/platform/runtime_enabled_features.json5?l=70).
    * `defaultFontFamily` Object (opzionale) - Imposta il font predefinito per il font-family. 
      * `standard` String (opzionale) - Il valore predefinito è il `Times New Roman`.
      * `serif` String (opzionale) -Il valore predefinito è il `Times New Roman`.
      * `sansSerif` String (opzionale) - Il valore predefinito è l'`Arial`.
      * `monospace` String (opzionale) - Il valore predefinito è il `Courier New`.
      * `cursive` String (opzionale) - Il valore predefinito è l'`Script`.
      * `fantasy` String (opzionale) - Il valore predefinito è l'`Impact`.
    * `defaultFontSize` Integer (opzionalel) - Il valore predefinito è `16`.
    * `defaultMonospaceFontSize` Integer (opzionale) - Il valore predefinito è `13`.
    * `minimumFontSize` Integer (opzionale) - Il valore predefinito è `0`.
    * `defaultEncoding` String (opzionale) - Il valore predefinito è l'`ISO-8859-1`.
    * `backgroundThrottling` Boolean (opzionale) - Limita le animazioni e i timer quando la pagina è in background. Questo influenza anche il [ Page Visibility API](#page-visibility). Il valore predefinito è `true`.
    * `offscreen ` Boolean (opzionale): se abilitato permette il rendering fuori schermo per la finestra del browser. Il valore predefinito è `false`. Vedi il[ Tutorial per il rendering offscreen ](../tutorial/offscreen-rendering.md) per maggiori dettagli.
    * ` contextIsolation ` Boolean (opzionale): esegue le API Electron e lo script ` preload` specificato in un contesto JavaScript separato. Il valore predefinito è `false`. Il contesto in cui viene eseguito lo script ` preload ` continuerà ad avere accesso completo ai ` documenti ` e ` finestre `, ma verrà utilizzato il suo insieme di builtin JavaScript (` Array`, ` Object`, ` JSON `, ecc.) e sarà isolato da eventuali modifiche apportate all'ambiente globale dalla pagina caricata. Le API Electron saranno disponibili solo nello script ` preload` e non nella pagina caricata. Questa opzione dovrebbe essere usata quando il caricamento dei contenuti remoti sono potenzialmente non attendibili per garantire il contenuto caricato non può manomettere lo script ` preload` e tutte le API Electron in uso. Questa opzione utilizza la stessa tecnica utilizzata da [ Chrome Content Scripts](https://developer.chrome.com/extensions/content_scripts#execution-environment). È possibile accedere a questo contesto negli strumenti di sviluppo selezionando La voce "Electron Isolated Context" nella casella combinata nella parte superiore della scheda Console. **Nota:** Questa opzione è attualmente sperimentale e potrebbe cambiare o essere rimossa nelle versioni future di Electron.
    * `nativeWindowOpen` Boolean (opzionale) - Permettere di usare la funzione nativa `window.open()`. Di default è `false`. **Nota:** Questa funzione è sperimentale.
    * `webviewTag` Boolean (opzionale) - Abilita il [`<webview>` tag](webview-tag.md). Il valore di default è come l'opzione di `nodeIntegration`. **Nota:** Lo script di `preload` configurato per la `<webview>` avrà la nodeIntegration abilitata quando viene eseguita, quindi è necessario garantire che il contenuto remoto/non attendibile non sia in grado di creare una `<webview>` con tag di `preload` potenzialmente dannosi. Puoi utilizzare l'evento `will-attach-webview ` su [webContents](web-contents.md) per rimuovere lo script ` preload` e per convalidare o modificare le impostazioni iniziali della `<webview>`.
    * `additionArguments` String[] (opzionale) - Un elenco di stringhe che verranno aggiunte su ` process.argv ` nel renderer process di questa app. Utile per passare un piccolo quantitativo di dati fino al renderer process elaborano dagli script di preload.

When setting minimum or maximum window size with `minWidth`/`maxWidth`/ `minHeight`/`maxHeight`, it only constrains the users. It won't prevent you from passing a size that does not follow size constraints to `setBounds`/`setSize` or to the constructor of `BrowserWindow`.

The possible values and behaviors of the `type` option are platform dependent. Possible values are:

* On Linux, possible types are `desktop`, `dock`, `toolbar`, `splash`, `notification`.
* On macOS, possible types are `desktop`, `textured`. 
  * The `textured` type adds metal gradient appearance (`NSTexturedBackgroundWindowMask`).
  * The `desktop` type places the window at the desktop background window level (`kCGDesktopWindowLevel - 1`). Note that desktop window will not receive focus, keyboard or mouse events, but you can use `globalShortcut` to receive input sparingly.
* On Windows, possible type is `toolbar`.

### Eventi dell'istanza

Objects created with `new BrowserWindow` emit the following events:

**Nota:** Alcuni metodi sono disponibili solo su sistemi operativi specifici e sono etichettati come tali.

#### Event: 'page-title-updated'

Restituisce:

* `event` Event
* `Titolo` Stringa

Emitted when the document changed its title, calling `event.preventDefault()` will prevent the native window's title from changing.

#### Event: 'close'

Restituisce:

* `event` Event

Emitted when the window is going to be closed. It's emitted before the `beforeunload` and `unload` event of the DOM. Calling `event.preventDefault()` will cancel the close.

Usually you would want to use the `beforeunload` handler to decide whether the window should be closed, which will also be called when the window is reloaded. In Electron, returning any value other than `undefined` would cancel the close. Ad esempio:

```javascript
window.onbeforeunload = (e) => {
  console.log('I do not want to be closed')

  // Unlike usual browsers that a message box will be prompted to users, returning
  // a non-void value will silently cancel the close.
  // It is recommended to use the dialog API to let the user confirm closing the
  // application.
  e.returnValue = false // equivalent to `return false` but not recommended
}
```

***Note**: There is a subtle difference between the behaviors of `window.onbeforeunload = handler` and `window.addEventListener('beforeunload', handler)`. It is recommended to always set the `event.returnValue` explicitly, instead of just returning a value, as the former works more consistently within Electron.*

#### Event: 'closed'

Emitted when the window is closed. After you have received this event you should remove the reference to the window and avoid using it any more.

#### Event: 'session-end' *Windows*

Emitted when window session is going to end due to force shutdown or machine restart or session log off.

#### Event: 'unresponsive'

Emitted when the web page becomes unresponsive.

#### Event: 'responsive'

Emitted when the unresponsive web page becomes responsive again.

#### Event: 'blur'

Emitted when the window loses focus.

#### Event: 'focus'

Emitted when the window gains focus.

#### Event: 'show'

Emitted when the window is shown.

#### Event: 'hide'

Emitted when the window is hidden.

#### Event: 'ready-to-show'

Emitted when the web page has been rendered (while not being shown) and window can be displayed without a visual flash.

#### Event: 'maximize'

Emitted when window is maximized.

#### Event: 'unmaximize'

Emitted when the window exits from a maximized state.

#### Event: 'minimize'

Emitted when the window is minimized.

#### Event: 'restore'

Emitted when the window is restored from a minimized state.

#### Event: 'resize'

Emitted when the window is being resized.

#### Event: 'move'

Emitted when the window is being moved to a new position.

**Note**: On macOS this event is just an alias of `moved`.

#### Event: 'moved' *macOS*

Emitted once when the window is moved to a new position.

#### Event: 'enter-full-screen'

Emitted when the window enters a full-screen state.

#### Event: 'leave-full-screen'

Emitted when the window leaves a full-screen state.

#### Event: 'enter-html-full-screen'

Emitted when the window enters a full-screen state triggered by HTML API.

#### Event: 'leave-html-full-screen'

Emitted when the window leaves a full-screen state triggered by HTML API.

#### Event: 'app-command' *Windows*

Restituisce:

* `event` Event
* `command` String

Emitted when an [App Command](https://msdn.microsoft.com/en-us/library/windows/desktop/ms646275(v=vs.85).aspx) is invoked. These are typically related to keyboard media keys or browser commands, as well as the "Back" button built into some mice on Windows.

Commands are lowercased, underscores are replaced with hyphens, and the `APPCOMMAND_` prefix is stripped off. e.g. `APPCOMMAND_BROWSER_BACKWARD` is emitted as `browser-backward`.

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()
win.on('app-command', (e, cmd) => {
  // Navigate the window back when the user hits their mouse back button
  if (cmd === 'browser-backward' && win.webContents.canGoBack()) {
    win.webContents.goBack()
  }
})
```

#### Event: 'scroll-touch-begin' *macOS*

Emitted when scroll wheel event phase has begun.

#### Event: 'scroll-touch-end' *macOS*

Emitted when scroll wheel event phase has ended.

#### Event: 'scroll-touch-edge' *macOS*

Emitted when scroll wheel event phase filed upon reaching the edge of element.

#### Event: 'swipe' *macOS*

Restituisce:

* `event` Event
* `direction` String

Emitted on 3-finger swipe. Possible directions are `up`, `right`, `down`, `left`.

#### Event: 'sheet-begin' *macOS*

Emitted when the window opens a sheet.

#### Event: 'sheet-end' *macOS*

Emitted when the window has closed a sheet.

#### Evento: 'nuova-finestra-per-scheda' *macOS*

Emitted when the native new tab button is clicked.

### Metodi Statici

The `BrowserWindow` class has the following static methods:

#### `BrowserWindow.getAllWindows()`

Returns `BrowserWindow[]` - An array of all opened browser windows.

#### `BrowserWindow.getFocusedWindow()`

Returns `BrowserWindow` - The window that is focused in this application, otherwise returns `null`.

#### `BrowserWindow.fromWebContents(webContents)`

* `ContenutiWeb` [ContenutiWeb](web-contents.md)

Returns `BrowserWindow` - The window that owns the given `webContents`.

#### `BrowserWindow.fromBrowserView(browserView)`

* `browserView` [BrowserView](browser-view.md)

Returns `BrowserWindow | null` - The window that owns the given `browserView`. If the given view is not attached to any window, returns `null`.

#### `BrowserWindow.fromId(id)`

* `id` Numero Intero

Returns `BrowserWindow` - The window with the given `id`.

#### `BrowserWindow.addExtension(path)`

* `path` String

Adds Chrome extension located at `path`, and returns extension's name.

The method will also not return if the extension's manifest is missing or incomplete.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.removeExtension(name)`

* `name` Stringa

Remove a Chrome extension by name.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.getExtensions()`

Returns `Object` - The keys are the extension names and each value is an Object containing `name` and `version` properties.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.addDevToolsExtension(path)`

* `path` String

Adds DevTools extension located at `path`, and returns extension's name.

The extension will be remembered so you only need to call this API once, this API is not for programming use. If you try to add an extension that has already been loaded, this method will not return and instead log a warning to the console.

The method will also not return if the extension's manifest is missing or incomplete.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.removeDevToolsExtension(name)`

* `name` Stringa

Remove a DevTools extension by name.

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

#### `BrowserWindow.getDevToolsExtensions()`

Returns `Object` - The keys are the extension names and each value is an Object containing `name` and `version` properties.

To check if a DevTools extension is installed you can run the following:

```javascript
const {BrowserWindow} = require('electron')

let installed = BrowserWindow.getDevToolsExtensions().hasOwnProperty('devtron')
console.log(installed)
```

**Note:** This API cannot be called before the `ready` event of the `app` module is emitted.

### Proprietà Istanza

Objects created with `new BrowserWindow` have the following properties:

```javascript
const {BrowserWindow} = require('electron')
// In this example `win` is our instance
let win = new BrowserWindow({width: 800, height: 600})
win.loadURL('https://github.com')
```

#### `win.webContents`

A `WebContents` object this window owns. All web page related events and operations will be done via it.

See the [`webContents` documentation](web-contents.md) for its methods and events.

#### `win.id`

A `Integer` representing the unique ID of the window.

### Metodi Istanza

Objects created with `new BrowserWindow` have the following instance methods:

**Nota:** Alcuni metodi sono disponibili solo su sistemi operativi specifici e sono etichettati come tali.

#### `win.destroy()`

Force closing the window, the `unload` and `beforeunload` event won't be emitted for the web page, and `close` event will also not be emitted for this window, but it guarantees the `closed` event will be emitted.

#### `win.chiudi()`

Try to close the window. This has the same effect as a user manually clicking the close button of the window. The web page may cancel the close though. See the [close event](#event-close).

#### `win.focus()`

Focalizza la finestra.

#### `win.blur()`

Rimuove la focalizzazione dalla finestra.

#### `win.isFocused()`

Ritorna `Boolean` - Se la finestra é focalizzata.

#### `win.isDestroyed()`

Ritorna `Boolean` - se la finestra é stata distrutta.

#### `win.show()`

Visualizza la finestra e la focalizza.

#### `win.showInactive()`

Mostra la finestra, senza focalizzarla.

#### `win.hide()`

Nasconde la finestra.

#### `win.isVisible()`

Ritorna `Boolean` - Se la finestra é visibile all'utente.

#### `win.isModal()`

Ritorna `Boolean` - Se la finestra corrente é modale.

#### `win.maximize()`

Ingrandisce la finestra. Nel caso in cui non sia giá visualizzata, la finestra verrá mostrata (ma non focalizzata).

#### `win.unmaximize()`

Rimuove l'ingrandimento della finestra.

#### `win.isMaximized()`

Ritorna `Boolean` - se la finestra é stata ingrandita.

#### `win.minimize()`

Minimizes the window. On some platforms the minimized window will be shown in the Dock.

#### `win.restore()`

Restores the window from minimized state to its previous state.

#### `win.isMinimized()`

Returns `Boolean` - Whether the window is minimized.

#### `win.setFullScreen(flag)`

* `flag` Boolean

Sets whether the window should be in fullscreen mode.

#### `win.isFullScreen()`

Ritorna `Boolean` - Se la finestra é a tutto schermo.

#### `win.setSimpleFullScreen(flag)` *macOS*

* `flag` Boolean

Enters or leaves simple fullscreen mode.

Simple fullscreen mode emulates the native fullscreen behavior found in versions of Mac OS X prior to Lion (10.7).

#### `win.isSimpleFullScreen()` *macOS*

Returns `Boolean` - Whether the window is in simple (pre-Lion) fullscreen mode.

#### `win.setAspectRatio(aspectRatio[, extraSize])` *macOS*

* `aspectRatio` Float - The aspect ratio to maintain for some portion of the content view.
* `extraSize` [Size](structures/size.md) - The extra size not to be included while maintaining the aspect ratio.

This will make a window maintain an aspect ratio. The extra size allows a developer to have space, specified in pixels, not included within the aspect ratio calculations. This API already takes into account the difference between a window's size and its content size.

Consider a normal window with an HD video player and associated controls. Perhaps there are 15 pixels of controls on the left edge, 25 pixels of controls on the right edge and 50 pixels of controls below the player. In order to maintain a 16:9 aspect ratio (standard aspect ratio for HD @1920x1080) within the player itself we would call this function with arguments of 16/9 and [ 40, 50 ]. The second argument doesn't care where the extra width and height are within the content view--only that they exist. Just sum any extra width and height areas you have within the overall content view.

#### `win.previewFile(path[, displayName])` *macOS*

* `path` String - The absolute path to the file to preview with QuickLook. This is important as Quick Look uses the file name and file extension on the path to determine the content type of the file to open.
* `displayName` String (optional) - The name of the file to display on the Quick Look modal view. This is purely visual and does not affect the content type of the file. Defaults to `path`.

Uses [Quick Look](https://en.wikipedia.org/wiki/Quick_Look) to preview a file at a given path.

#### `win.closeFilePreview()` *macOS*

Closes the currently open [Quick Look](https://en.wikipedia.org/wiki/Quick_Look) panel.

#### `win.setBounds(bounds[, animate])`

* `limiti` [Rettangolo](structures/rectangle.md)
* `animate` Boolean (optional) *macOS*

Resizes and moves the window to the supplied bounds

#### `win.getBounds()`

Ritorna [`Rectangle`](structures/rectangle.md)

#### `win.setContentBounds(bounds[, animate])`

* `limiti` [Rettangolo](structures/rectangle.md)
* `animate` Boolean (optional) *macOS*

Resizes and moves the window's client area (e.g. the web page) to the supplied bounds.

#### `win.getContentBounds()`

Ritorna [`Rectangle`](structures/rectangle.md)

#### `win.setEnabled(enable)`

* `enable` Boolean

Disabilita o abilita la finestra.

#### `win.setSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (optional) *macOS*

Resizes the window to `width` and `height`.

#### `win.getSize()`

Returns `Integer[]` - Contains the window's width and height.

#### `win.setContentSize(width, height[, animate])`

* `width` Integer
* `height` Integer
* `animate` Boolean (optional) *macOS*

Resizes the window's client area (e.g. the web page) to `width` and `height`.

#### `win.getContentSize()`

Returns `Integer[]` - Contains the window's client area's width and height.

#### `win.setMinimumSize(width, height)`

* `width` Integer
* `height` Integer

Sets the minimum size of window to `width` and `height`.

#### `win.getMinimumSize()`

Returns `Integer[]` - Contains the window's minimum width and height.

#### `win.setMaximumSize(width, height)`

* `width` Integer
* `height` Integer

Sets the maximum size of window to `width` and `height`.

#### `win.getMaximumSize()`

Returns `Integer[]` - Contains the window's maximum width and height.

#### `win.setResizable(resizable)`

* `resizable` Boolean

Sets whether the window can be manually resized by user.

#### `win.isResizable()`

Returns `Boolean` - Whether the window can be manually resized by user.

#### `win.setMovable(movable)` *macOS* *Windows*

* `movable` Boolean

Sets whether the window can be moved by user. On Linux does nothing.

#### `win.isMovable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be moved by user.

On Linux always returns `true`.

#### `win.setMinimizable(minimizable)` *macOS* *Windows*

* `minimizable` Boolean

Sets whether the window can be manually minimized by user. On Linux does nothing.

#### `win.isMinimizable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually minimized by user

On Linux always returns `true`.

#### `win.setMaximizable(maximizable)` *macOS* *Windows*

* `maximizable` Boolean

Sets whether the window can be manually maximized by user. On Linux does nothing.

#### `win.isMaximizable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually maximized by user.

On Linux always returns `true`.

#### `win.setFullScreenable(fullscreenable)`

* `fullscreenable` Boolean

Sets whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.isFullScreenable()`

Returns `Boolean` - Whether the maximize/zoom window button toggles fullscreen mode or maximizes the window.

#### `win.setClosable(closable)` *macOS* *Windows*

* `closable` Boolean

Sets whether the window can be manually closed by user. On Linux does nothing.

#### `win.isClosable()` *macOS* *Windows*

Returns `Boolean` - Whether the window can be manually closed by user.

On Linux always returns `true`.

#### `win.setAlwaysOnTop(flag[, level][, relativeLevel])`

* `flag` Boolean
* `level` String (optional) *macOS* - Values include `normal`, `floating`, `torn-off-menu`, `modal-panel`, `main-menu`, `status`, `pop-up-menu`, `screen-saver`, and ~~`dock`~~ (Deprecated). The default is `floating`. See the [macOS docs](https://developer.apple.com/reference/appkit/nswindow/1664726-window_levels) for more details.
* `relativeLevel` Integer (optional) *macOS* - The number of layers higher to set this window relative to the given `level`. The default is `0`. Note that Apple discourages setting levels higher than 1 above `screen-saver`.

Sets whether the window should show always on top of other windows. After setting this, the window is still a normal window, not a toolbox window which can not be focused on.

#### `win.isAlwaysOnTop()`

Returns `Boolean` - Whether the window is always on top of other windows.

#### `win.center()`

Moves window to the center of the screen.

#### `win.setPosition(x, y[, animate])`

* `x` Integer
* `y` Integer
* `animate` Boolean (optional) *macOS*

Moves window to `x` and `y`.

#### `win.getPosition()`

Returns `Integer[]` - Contains the window's current position.

#### `win.setTitle(title)`

* `Titolo` Stringa

Changes the title of native window to `title`.

#### `win.getTitle()`

Returns `String` - The title of the native window.

**Note:** The title of web page can be different from the title of the native window.

#### `win.setSheetOffset(offsetY[, offsetX])` *macOS*

* `offsetY` Float
* `offsetX` Float (optional)

Changes the attachment point for sheets on macOS. By default, sheets are attached just below the window frame, but you may want to display them beneath a HTML-rendered toolbar. For example:

```javascript
const {BrowserWindow} = require('electron')
let win = new BrowserWindow()

let toolbarRect = document.getElementById('toolbar').getBoundingClientRect()
win.setSheetOffset(toolbarRect.height)
```

#### `win.flashFrame(flag)`

* `flag` Boolean

Starts or stops flashing the window to attract user's attention.

#### `win.setSkipTaskbar(skip)`

* `skip` Boolean

Makes the window not show in the taskbar.

#### `win.setKiosk(flag)`

* `flag` Boolean

Enters or leaves the kiosk mode.

#### `win.isKiosk()`

Returns `Boolean` - Whether the window is in kiosk mode.

#### `win.getNativeWindowHandle()`

Returns `Buffer` - The platform-specific handle of the window.

The native type of the handle is `HWND` on Windows, `NSView*` on macOS, and `Window` (`unsigned long`) on Linux.

#### `win.hookWindowMessage(message, callback)` *Windows*

* `message` Integer
* `callback` Funzione

Hooks a windows message. The `callback` is called when the message is received in the WndProc.

#### `win.isWindowMessageHooked(message)` *Windows*

* `message` Integer

Returns `Boolean` - `true` or `false` depending on whether the message is hooked.

#### `win.unhookWindowMessage(message)` *Windows*

* `message` Integer

Unhook the window message.

#### `win.unhookAllWindowMessages()` *Windows*

Unhooks all of the window messages.

#### `win.setRepresentedFilename(filename)` *macOS*

* `filename` String

Sets the pathname of the file the window represents, and the icon of the file will show in window's title bar.

#### `win.getRepresentedFilename()` *macOS*

Returns `String` - The pathname of the file the window represents.

#### `win.setDocumentEdited(edited)` *macOS*

* `edited` Boolean

Specifies whether the window’s document has been edited, and the icon in title bar will become gray when set to `true`.

#### `win.isDocumentEdited()` *macOS*

Returns `Boolean` - Whether the window's document has been edited.

#### `win.focusOnWebView()`

#### `win.blurWebView()`

#### `win.capturePage([rect, ]callback)`

* `rect` [Rectangle](structures/rectangle.md) (optional) - The bounds to capture
* `callback` Function 
  * `image` [NativeImage](native-image.md)

Same as `webContents.capturePage([rect, ]callback)`.

#### `win.loadURL(url[, options])`

* `url` Stringa
* `options` Object (opzionale) 
  * `httpReferrer` String (optional) - A HTTP Referrer url.
  * `userAgent` String (optional) - A user agent originating the request.
  * `extraHeaders` String (optional) - Extra headers separated by "\n"
  * `postData` ([UploadRawData[]](structures/upload-raw-data.md) | [UploadFile[]](structures/upload-file.md) | [UploadFileSystem[]](structures/upload-file-system.md) | [UploadBlob[]](structures/upload-blob.md)) (optional)
  * `baseURLForDataURL` String (optional) - Base url (with trailing path separator) for files to be loaded by the data url. This is needed only if the specified `url` is a data url and needs to load other files.

Same as `webContents.loadURL(url[, options])`.

The `url` can be a remote address (e.g. `http://`) or a path to a local HTML file using the `file://` protocol.

To ensure that file URLs are properly formatted, it is recommended to use Node's [`url.format`](https://nodejs.org/api/url.html#url_url_format_urlobject) method:

```javascript
let url = require('url').format({
  protocol: 'file',
  slashes: true,
  pathname: require('path').join(__dirname, 'index.html')
})

win.loadURL(url)
```

You can load a URL using a `POST` request with URL-encoded data by doing the following:

```javascript
win.loadURL('http://localhost:8000/post', {
  postData: [{
    type: 'rawData',
    bytes: Buffer.from('hello=world')
  }],
  extraHeaders: 'Content-Type: application/x-www-form-urlencoded'
})
```

#### `win.loadFile(filePath)`

* `Percorsofile` Stringa

Same as `webContents.loadFile`, `filePath` should be a path to an HTML file relative to the root of your application. See the `webContents` docs for more information.

#### `win.reload()`

Same as `webContents.reload`.

#### `win.setMenu(menu)` *Linux* *Windows*

* `menu` Menu | null

Sets the `menu` as the window's menu bar, setting it to `null` will remove the menu bar.

#### `win.setProgressBar(progress[, options])`

* `progress` Double
* `options` Object (opzionale) 
  * `mode` String *Windows* - Mode for the progress bar. Can be `none`, `normal`, `indeterminate`, `error` or `paused`.

Sets progress value in progress bar. Valid range is [0, 1.0].

Remove progress bar when progress < 0; Change to indeterminate mode when progress > 1.

On Linux platform, only supports Unity desktop environment, you need to specify the `*.desktop` file name to `desktopName` field in `package.json`. By default, it will assume `app.getName().desktop`.

On Windows, a mode can be passed. Accepted values are `none`, `normal`, `indeterminate`, `error`, and `paused`. If you call `setProgressBar` without a mode set (but with a value within the valid range), `normal` will be assumed.

#### `win.setOverlayIcon(overlay, description)` *Windows*

* `overlay` [NativeImage](native-image.md) | null - the icon to display on the bottom right corner of the taskbar icon. If this parameter is `null`, the overlay is cleared
* `description` String - a description that will be provided to Accessibility screen readers

Sets a 16 x 16 pixel overlay onto the current taskbar icon, usually used to convey some sort of application status or to passively notify the user.

#### `win.setHasShadow(hasShadow)` *macOS*

* `hasShadow` Boolean

Sets whether the window should have a shadow. On Windows and Linux does nothing.

#### `win.hasShadow()` *macOS*

Returns `Boolean` - Whether the window has a shadow.

On Windows and Linux always returns `true`.

#### `win.setOpacity(opacity)` *Windows* *macOS*

* `opacity` Number - between 0.0 (fully transparent) and 1.0 (fully opaque)

Sets the opacity of the window. On Linux does nothing.

#### `win.getOpacity()` *Windows* *macOS*

Returns `Number` - between 0.0 (fully transparent) and 1.0 (fully opaque)

#### `win.setThumbarButtons(buttons)` *Windows*

* `buttons` [ThumbarButton[]](structures/thumbar-button.md)

Returns `Boolean` - Whether the buttons were added successfully

Add a thumbnail toolbar with a specified set of buttons to the thumbnail image of a window in a taskbar button layout. Returns a `Boolean` object indicates whether the thumbnail has been added successfully.

The number of buttons in thumbnail toolbar should be no greater than 7 due to the limited room. Once you setup the thumbnail toolbar, the toolbar cannot be removed due to the platform's limitation. But you can call the API with an empty array to clean the buttons.

The `buttons` is an array of `Button` objects:

* `Button` Oggetto 
  * `icon` [NativeImage](native-image.md) - L'icona mostrata nella barra degli strumenti come anteprima.
  * `click` Funzione
  * `tooltip` Stringa (opzionale) - Il testo del tooltip del pulsante.
  * `flags` Stringa[] (opzionale) - Controlla specifici comportamenti e comportamenti del pulsante. Di default è `['enabled']`.

I `flags` sono un insieme che include le seguenti `String`:

* `enabled` - Il pulsante è attivato e disponibile all'utente.
* `disabled` - Il pulsante é disabilitato. È presente ma lo stato visuale che lo indica non risponderà all'azione dell'utente.
* `dismissonclick` - Quando il pulsante è cliccato, la finestra miniaturizzata si chiude immediatamente.
* `nobackground` - Non disegna i bordi del pulsante, usa solo l'immagine.
* `hidden` - Il pulsante non è mostrato all'utente.
* `noninteractive` - Il pulsante è abilitato ma non interattivo; il pulsante è mostrato in uno stato di 'non premuto'. Questo valore è inteso per istanze in cui il pulsante è usato in una notifica.

#### `win.setThumbnailClip(region)` *Windows*

* `region` [Rectangle](structures/rectangle.md) - Region of the window

Sets the region of the window to show as the thumbnail image displayed when hovering over the window in the taskbar. You can reset the thumbnail to be the entire window by specifying an empty region: `{x: 0, y: 0, width: 0, height: 0}`.

#### `win.setThumbnailToolTip(toolTip)` *Windows*

* `toolTip` String

Sets the toolTip that is displayed when hovering over the window thumbnail in the taskbar.

#### `win.setAppDetails(options)` *Windows*

* `options` Oggetto 
  * `appId` String (optional) - Window's [App User Model ID](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391569(v=vs.85).aspx). It has to be set, otherwise the other options will have no effect.
  * `appIconPath` String (optional) - Window's [Relaunch Icon](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391573(v=vs.85).aspx).
  * `appIconIndex` Integer (optional) - Index of the icon in `appIconPath`. Ignored when `appIconPath` is not set. Default is `0`.
  * `relaunchCommand` String (optional) - Window's [Relaunch Command](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391571(v=vs.85).aspx).
  * `relaunchDisplayName` String (optional) - Window's [Relaunch Display Name](https://msdn.microsoft.com/en-us/library/windows/desktop/dd391572(v=vs.85).aspx).

Sets the properties for the window's taskbar button.

**Note:** `relaunchCommand` and `relaunchDisplayName` must always be set together. If one of those properties is not set, then neither will be used.

#### `win.showDefinitionForSelection()` *macOS*

Same as `webContents.showDefinitionForSelection()`.

#### `win.setIcon(icon)` *Windows* *Linux*

* `icona` [ImmagineNativa](native-image.md)

Changes window icon.

#### `win.setAutoHideMenuBar(hide)`

* `hide` Boolean

Sets whether the window menu bar should hide itself automatically. Once set the menu bar will only show when users press the single `Alt` key.

If the menu bar is already visible, calling `setAutoHideMenuBar(true)` won't hide it immediately.

#### `win.isMenuBarAutoHide()`

Returns `Boolean` - Whether menu bar automatically hides itself.

#### `win.setMenuBarVisibility(visible)` *Windows* *Linux*

* `visible` Boolean

Sets whether the menu bar should be visible. If the menu bar is auto-hide, users can still bring up the menu bar by pressing the single `Alt` key.

#### `win.isMenuBarVisible()`

Returns `Boolean` - Whether the menu bar is visible.

#### `win.setVisibleOnAllWorkspaces(visible)`

* `visible` Boolean

Sets whether the window should be visible on all workspaces.

**Note:** This API does nothing on Windows.

#### `win.isVisibleOnAllWorkspaces()`

Returns `Boolean` - Whether the window is visible on all workspaces.

**Note:** This API always returns false on Windows.

#### `win.setIgnoreMouseEvents(ignore[, options])`

* `ignore` Boolean
* `options` Object (opzionale) 
  * `forward` Boolean (optional) *Windows* - If true, forwards mouse move messages to Chromium, enabling mouse related events such as `mouseleave`. Only used when `ignore` is true. If `ignore` is false, forwarding is always disabled regardless of this value.

Makes the window ignore all mouse events.

All mouse events happened in this window will be passed to the window below this window, but if this window has focus, it will still receive keyboard events.

#### `win.setContentProtection(enable)` *macOS* *Windows*

* `enable` Boolean

Prevents the window contents from being captured by other apps.

On macOS it sets the NSWindow's sharingType to NSWindowSharingNone. On Windows it calls SetWindowDisplayAffinity with `WDA_MONITOR`.

#### `win.setFocusable(focusable)` *Windows*

* `focusable` Boolean

Changes whether the window can be focused.

#### `win.setParentWindow(parent)` *Linux* *macOS*

* `parent` BrowserWindow

Sets `parent` as current window's parent window, passing `null` will turn current window into a top-level window.

#### `win.getParentWindow()`

Returns `BrowserWindow` - The parent window.

#### `win.getChildWindows()`

Returns `BrowserWindow[]` - All child windows.

#### `win.setAutoHideCursor(autoHide)` *macOS*

* `autoHide` Boolean

Controls whether to hide cursor when typing.

#### `win.selectPreviousTab()` *macOS*

Selects the previous tab when native tabs are enabled and there are other tabs in the window.

#### `win.selectNextTab()` *macOS*

Selects the next tab when native tabs are enabled and there are other tabs in the window.

#### `win.mergeAllWindows()` *macOS*

Merges all windows into one window with multiple tabs when native tabs are enabled and there is more than one open window.

#### `win.moveTabToNewWindow()` *macOS*

Moves the current tab into a new window if native tabs are enabled and there is more than one tab in the current window.

#### `win.toggleTabBar()` *macOS*

Toggles the visibility of the tab bar if native tabs are enabled and there is only one tab in the current window.

#### `win.addTabbedWindow(browserWindow)` *macOS*

* `browserWindow` BrowserWindow

Adds a window as a tab on this window, after the tab for the window instance.

#### `win.setVibrancy(type)` *macOS*

* `type` String - Can be `appearance-based`, `light`, `dark`, `titlebar`, `selection`, `menu`, `popover`, `sidebar`, `medium-light` or `ultra-dark`. See the [macOS documentation](https://developer.apple.com/documentation/appkit/nsvisualeffectview?preferredLanguage=objc) for more details.

Adds a vibrancy effect to the browser window. Passing `null` or an empty string will remove the vibrancy effect on the window.

#### `win.setTouchBar(touchBar)` *macOS* *Experimental*

* `touchBar` TouchBar

Sets the touchBar layout for the current window. Specifying `null` or `undefined` clears the touch bar. This method only has an effect if the machine has a touch bar and is running on macOS 10.12.1+.

**Note:** The TouchBar API is currently experimental and may change or be removed in future Electron releases.

#### `win.setBrowserView(browserView)` *Sperimentale*

* `browserView` [BrowserView](browser-view.md)

#### `win.getBrowserView()` *Sperimentale*

Returns `BrowserView | null` - an attached BrowserView. Returns `null` if none is attached.

**Nota:** La VistaBrowser API è attualmente sperimentale e potrebbe cambiare o essere rimossa nei rilasci futuri di Electron.