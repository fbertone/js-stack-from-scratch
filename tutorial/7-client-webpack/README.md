# 7 - App Client con Webpack

## Struttura della nostra app

- Crea una cartella `dist` nella cartella principale del progetto, e aggiungi il seguente `index.html` al suo interno:

```html
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <div class="app"></div>
    <script src="client-bundle.js"></script>
  </body>
</html>
```

Nella cartella `src`, crea le seguenti cartelle: `server`, `shared`, `client`, e sposta il file `index.js` che stai utilizzando all'interno di `server`, e `dog.js` dentro `shared`. Crea `app.js` in `client`.

Per il momento non creeremo nessun backend in Node, ma questa suddivisione ti aiuterà a capire meglio come verranno struttirati i vari moduli. Dovrai sostituire `import Dog from './dog';` in `server/index.js` con `import Dog from '../shared/dog';` altrimenti ESLint si accorgerà che alcuni moduli non sono presenti.

Scrivi questo codice in `client/app.js`:

```javascript
import Dog from '../shared/dog';

const browserToby = new Dog('Browser Toby');

document.querySelector('.app').innerText = browserToby.bark();
```

Aggiungi all'interno di `package.json`, in `eslintConfig`:

```json
"env": {
  "browser": true
}
```
In questo modo potremo utilizzare variabili come `window` o `document`, che sono sempre accessibili nel browser, senza che ESLint si lamenti.

Se vuoi utilizzare alcune delle funzionalità più recenti di ES nel tuo client, come le `Promise`, dovrei includere i [Polyfill di Babel](https://babeljs.io/docs/usage/polyfill/) nel tuo codice.

- Esegui `yarn add babel-polyfill`

E in `app.js`, prima di qualunque altra cosa, aggiungi:

```javascript
import 'babel-polyfill';
```

Includere i polyfill aumenterà di circa 300KB il tuo client finale, quindi non farlo se non devi utilizzare nessuna delle funzionalità che fornisce!

## Webpack

In ambiente Node, puoi liberamente fare `import` di differenti file e Node si occuperà di trovare questi file all'interno del tuo filesystem. In un browser, non c'è filesystem, quindi i tuoi `import` non puntano a nessun file. Per fare in modo che `app.js` riesca ad avere accesso a tutti moduli di cui necessita, andremo ad impacchettare tutte le dipendenze all'interno di un unico file. Webpack è uno strumento che ci permette di fare questo.

Webpack utilizza un file di configurazione, come Gulp, chiamato `webpack.config.js`. È possibile utilizzare import ed export ES6 al suo interno sfruttando Babel, esattamente come abbiamo fatto per Gulp, chiamando quindi il file `webpack.config.babel.js`.

- Crea un file `webpack.config.babel.js` vuoto

- Mentre ci sei, aggiungi `webpack.config.babel.js` al task `lint` di Gulp assieme ad alcune costanti di `paths`:

```javascript
const paths = {
  allSrcJs: 'src/**/*.js',
  gulpFile: 'gulpfile.babel.js',
  webpackFile: 'webpack.config.babel.js',
  libDir: 'lib',
  distDir: 'dist',
};

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
);
```

Dobbiamo spiegare a Webpack come processare file ES6 tramite Babel (come abbiamo fatto per Gulp e `gulp-babel`). In Webpack, quando devi processare dei file che non sono i classici JavaScript standard, dovrai utilizzare dei *loader*. Installiamo il loader di Babel per Webpack:

- Esegui `yarn add --dev babel-loader`

- Aggiungi quanto segue in `webpack.config.babel.js`:

```javascript
export default {
  output: {
    filename: 'client-bundle.js',
  },
  devtool: 'source-map',
  module: {
    loaders: [
      {
        test: /\.jsx?$/,
        loader: 'babel-loader',
        exclude: [/node_modules/],
      },
    ],
  },
  resolve: {
    extensions: ['', '.js', '.jsx'],
  },
};
```

Analizziamo:

Questo file deve fare `export` di cose in modo che Webpack possa accedervi. `output.filename` è il nome del file che vogliamo generare. `devtool: 'source-map'` attiverà le source map per permettere un debug più agevole nel browser. In `module.loaders`, abbiamo un `test`, che è l'espressione regolare (regex) JavaScript regex che verrà utilizzata per definire quali file verranno processati tramite `babel-loader`. Siccome utilizzeremo sia file di tipo `.js` che `.jsx` (per React) nei prossimi capitoli, applichiamo questa regex: `/\.jsx?$/`. La cartella `node_modules` viene esclusa perchè non dovremo fare nessuna manipolazione ai moduli presenti la dentro. In questo modo, quando il tuo codice fa degli `import` di pacchetti presenti in `node_modules`, Babel non si preoccuperà di processare tutti i file, in che riducerà i tempi di build. La sezione `resolve` serve a spiegare a Webpack quale tipi di file vogliamo poter `import`are nel nostro codice utilizzando dei percorsi semplici senza parte finale, ad esempio: `import Foo from './foo'` dove `foo` potrebbe essere `foo.js` oppure `foo.jsx`.

Okay adesso abbiamo configurato Webpack ma ci serve ancora una maniera per *eseguirlo*.

## Integrare Webpack in Gulp

Webpack può svolgere molti compiti. Può anche rimpiazzare completamente Gulp se il tuo è un progetto principalmente client-side. Gulp, essendo un tool più generico, è migliore per svolgere compiti quali linting, test, e task di back-end. È inoltre più intuitivo e semplice da capire per chi sta iniziando rispetto alla configurazione di Webpack. Abbiamo già creato una solida configurazione di Gulp, integrare Webpack sarà quindi un gioco da ragazzi.

Creiamo il task di Gulp per eseguire Webpack. Apri `gulpfile.babel.js`.

Non abbiamo più bisogno che il task `main` esegua `node lib/` perchè apriremo `index.html` per eseguire la nostra app.

- Rimuovi `import { exec } from 'child_process'`.

In modo simile ai plugin di Gulp, il modulo `webpack-stream` ci permette di integrare molto semplicemente Webpack all'interno di Gulp.

- Installa il modulo con: `yarn add --dev webpack-stream`

- Aggiungi i seguenti `import`:

```javascript
import webpack from 'webpack-stream';
import webpackConfig from './webpack.config.babel';
```

La seconda linea prende semplicemente la configurazione di webpack.

Come ho già detto, nei prossimi capitoli utilizzeremo dei file `.jsx`  (lato client, e successivamente anche lato server), iniziamo a configurarli adesso in modo da partire preparati.

- Cambia le costanti in questo modo:

```javascript
const paths = {
  allSrcJs: 'src/**/*.js?(x)',
  serverSrcJs: 'src/server/**/*.js?(x)',
  sharedSrcJs: 'src/shared/**/*.js?(x)',
  clientEntryPoint: 'src/client/app.js',
  gulpFile: 'gulpfile.babel.js',
  webpackFile: 'webpack.config.babel.js',
  libDir: 'lib',
  distDir: 'dist',
};
```

Il `.js?(x)` è semplicemente un pattern per considerare sia i file `.js` che `.jsx`.

Adesso abbiamo delle costanti per le varie sezioni dell'app ed un entrypoint da cui iniziare l'esecuzione.

- Modifica il task `main` in questo modo:

```javascript
gulp.task('main', ['lint', 'clean'], () =>
  gulp.src(paths.clientEntryPoint)
    .pipe(webpack(webpackConfig))
    .pipe(gulp.dest(paths.distDir))
);
```

**Nota**: Il nostro task `build` attualmente transpila il codice ES6 a ES5 per ogni file `.js` contenuti in `src`. Adesso che abbiamo suddiviso il nostro codice in `server`, `shared`, e `client`, possiamo fare in modo che questo task compili unicamente `server` e `shared` (visto che Webpacksi occupa di `client`). Tuttavia, nel capitolo dedicato al Testing, avremo bisogno di far compilare a Gulp anche la parte `client` in modo da testarlo al di fuori di Webpack. Quindi, fino a quando non raggiungerai quel capitolo, alcune fasi di build saranno duplicate. Sonon sicuro che possiamo essere tutti d'accordo che non ci sono problemi. Effettivamente non utilizzeremo più il task `build` e la cartella `lib` fino a reggiungere quel capitolo, perchè per il momento ci occuperemo unicamente di impacchettare il codice client.

- Esegui `yarn start`, dovresti vedere Webpack che costruisce il file `client-bundle.js` e aprendo `index.html`nel browser dovresti vederey "Wah wah, I am Browser Toby".

Un'ultima cosa: a differenza della cartella `lib` i file `dist/client-bundle.js` e `dist/client-bundle.js.map` non vengono cancellati dal nostro task `clean` prima di ogni build.

- Aggiungi `clientBundle: 'dist/client-bundle.js?(.map)'` alla sezione `paths` e modifica il task `clean` in questo modo:

```javascript
gulp.task('clean', () => del([
  paths.libDir,
  paths.clientBundle,
]));
```

- Aggiungi `/dist/client-bundle.js*` al file `.gitignore`:

Prossima sezione: [8 - React](/tutorial/8-react)

Torna alla [sezione precedente](/tutorial/6-eslint) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
