# 12 - Type Checking con Flow

[Flow](https://flowtype.org/) é un controllore di tipi di dato statico. Riconosce le inconsistenze nei tipi di dato del tuo codice e puoi aggiungere delle dichiarazioni esplicite dei tipi di dato da utilizzare tramite le sue annotazioni.

- Per fare in modo che Babel comprenda e rimuova le annotazioni di Flow durante il processo di transpiling, installa il preset di Flow per Babel eseguendo `yarn add --dev babel-preset-flow`. Poi, aggiungi `"flow"` alla voce `babel.presets`  nel `package.json`.

- Crea un file vuoto `.flowconfig` nella cartella di base del tuo progetto

- Esegui `yarn add --dev gulp-flowtype` per installare il plugin di Gulp per Flow, e aggiungi `flow()` nel task `lint`:

```javascript
import flow from 'gulp-flowtype';

// [...]

gulp.task('lint', () =>
  gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
    paths.webpackFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError())
    .pipe(flow({ abort: true })) // Add Flow here
);
```

L'opzione `abort` serve ad interrompere il task Gulp quando Flow riscontra un problema.

Bene, dovremmo essere in grado di eseguire Flow.

- Aggiungi le annotazioni di Flow nel file `src/shared/dog.js` in questo modo:

```javascript
// @flow

class Dog {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  bark(): string {
    return `Wah wah, I am ${this.name}`;
  }

  barkInConsole() {
    /* eslint-disable no-console */
    console.log(this.bark());
    /* eslint-enable no-console */
  }

}

export default Dog;
```

Il commento `// @flow` dice a Flow che vogliamo verificare i tipi di dato in questo file. Per il resto, le annotazioni di Flow si fanno con un simbolo di duepunti sopo il nome di un parametro o una variabile. Controlla la documentazione per avere altri dettagli.

Adesso se esegui `yarn start`, Flow funzionerá correttamente, ma ESLint si lamenterá della sintassi non standard che stiamo utilizzando. Siccome il parser di Babel é pronto per macinare le annotazioni di Flow grazie al plugin `babel-preset-flow` che abbiamo installato, sarebbe bello se ESLint potesse fare riferimento al parser di Babel invece di cercare di comprendere le annotazioni di Flow per conto suo. Questo é possibile utilizzando il pacchetto `babel-eslint`. Facciamolo.

- Esegui `yarn add --dev babel-eslint`

- In `package.json`, alla voce `eslintConfig`, aggiungi questa proprietá: `"parser": "babel-eslint"`

`yarn start` adesso dovrebbe eseguire sia il lint che il typecheck del codice.

Adesso che ESLint e Babel condividono lo stesso parser, possiamo fare in modo che ESLint esegua il lint delle nostre annotazioni Flow tramite il plugin `eslint-plugin-flowtype`.

- Esegui `yarn add --dev eslint-plugin-flowtype` e aggiungi `"flowtype"` alla voce `eslintConfig.plugins` in `package.json`, e aggiungi `"plugin:flowtype/recommended"` alla voce `eslintConfig.extends` in un array vicino a `"airbnb"`.

Adesso se scrivi `name:string` come annotazione, ESLint dovrebbe ad esempio lamentarsi del fatto che hai dimenticato uno spazio dopo il duepunti.

**Nota**: La proprietá `"parser": "babel-eslint"` che ti ho fatto scrivere in `package.json` é inclusa nella configurazione `"plugin:flowtype/recommended"`, puoi quindi rimuoverla per ottimizzare il file `package.json`. Se la lasci lí peró rendi il tutto piú esplicito, dipende quindi da te. Siccome questa guida si riferisce alla configurazione minimale, io lo tolgo.

- Adesso puoi aggiungere `// @flow` in ogni file `.js` e `.jsx` nella cartella `src`, eseguire `yarn test` o `yarn start`, aggiungere annotazioni come ti chiede di fare Flow.

Un caso non intuitivo é questo, per `src/client/component/message.jsx`:

```javascript
const Message = ({ message }: { message: string }) => <div>{message}</div>;
```

Come puoi vedere, quando destrutturi i parametri delle funzioni, devi annotare le proprietá estratte utilizzando una sorta di oggetto.

Un altro caso che incontrerai é quello in `src/client/reducers/dog-reducer.js`, Flow si lamenterá che Immutable non ha un export di default. Questo problema é discusso nel [#863 su Immutable](https://github.com/facebook/immutable-js/issues/863), che mostra 2 workaround:

```javascript
import { Map as ImmutableMap } from 'immutable';
// or
import * as Immutable from 'immutable';
```

Fino a quando Immutable non risolve ufficialmente questo problema, scegli semplicemente quello che ti sembra migliore mentre importi componenti Immutable. Io personalmente ho scelto `import * as Immutable from 'immutable'` siccome é piú corto e non richiederá di fare un refactoring del codice quando il problema verrá risolto.

**Nota**: Se Flow riscontra errori nei controlli dei tipi nella cartella `node_modules`, aggiungi una sezione `[ignore]` nel `.flowconfig` per ignorare specificamente i pacchetti che causano gli errori (non ignorare completamente la cartella `node_modules`). Potrebbe assomigliare a qualcosa come:

```flowconfig
[ignore]

.*/node_modules/gulp-flowtype/.*
```

Nel mio caso, il plugin `linter-flow` per Atom riconosceva degli errori nella cartella `node_modules/gulp-flowtype`, che contiene annotazioni per Flow del tipo `// @flow`.

Adesso hai un codice irrobustito da lint, typecheck, e test, ottimo lavoro!

Torna alla [sezione precedente](/tutorial/11-testing-mocha-chai-sinon) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
