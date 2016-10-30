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

Let's analyze this a bit:

We need this file to `export` stuff for Webpack to read. `output.filename` is the name of the bundle we want to generate. `devtool: 'source-map'` will enable source maps for a better debugging experience in your browser. In `module.loaders`, we have a `test`, which is the JavaScript regex that will be used to test which files should be processed by the `babel-loader`. Since we will use both `.js` files and `.jsx` files (for React) in the next chapters, we have the following regex: `/\.jsx?$/`. The `node_modules` folder is excluded because there is no transpilation to do there. This way, when your code `import`s packages located in `node_modules`, Babel doesn't bother processing those files, which reduces build time. The `resolve` part is to tell Webpack what kind of file we want to be able to `import` in our code using extension-less paths like `import Foo from './foo'` where `foo` could be `foo.js` or `foo.jsx` for instance.

Okay so now we have Webpack set up, but we still need a way to *run* it.

## Integrating Webpack to Gulp

Webpack can do a lot of things. It can actually replace Gulp entirely if your project is mostly client-side. Gulp being a more general tool, it is better suited for things like linting, tests, and back-end tasks though. It is also simpler to understand for newcomers than a complex Webpack config. We have a pretty solid Gulp setup and workflow here, so integrating Webpack to our Gulp build is going to be easy peasy.

Let's create the Gulp task to run Webpack. Open your `gulpfile.babel.js`.

We don't need the `main` task to execute `node lib/` anymore, since we will open `index.html` to run our app.

- Remove `import { exec } from 'child_process'`.

Similarly to Gulp plugins, the `webpack-stream` package lets us integrate Webpack into Gulp very easily.

- Install the package with: `yarn add --dev webpack-stream`

- Add the following `import`s:

```javascript
import webpack from 'webpack-stream';
import webpackConfig from './webpack.config.babel';
```

The second line just grabs our config file.

Like I said earlier, in the next chapter we are going to use `.jsx` files (on the client, and even on the server later on), so let's set that up right now to have a bit of a head start.

- Change the constants to the following:

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

The `.js?(x)` is just a pattern to match `.js` or `.jsx` files.

We now have constants for the different parts of our application, and an entry point file.

- Modify the `main` task like so:

```javascript
gulp.task('main', ['lint', 'clean'], () =>
  gulp.src(paths.clientEntryPoint)
    .pipe(webpack(webpackConfig))
    .pipe(gulp.dest(paths.distDir))
);
```

**Note**: Our `build` task currently transpiles ES6 code to ES5 for every `.js` file located under `src`. Now that we've split our code into `server`, `shared`, and `client` code, we could make this task only compile `server` and `shared` (since Webpack takes care of `client`). However, in the Testing chapter, we are going to need Gulp to also compile the `client` code to test it outside of Webpack. So until you reach that chapter, there is a bit of useless duplicated build being done. I'm sure we can all agree that it's fine for now. We actually aren't even going to be using the `build` task and `lib` folder anymore until that chapter, since all we care about right now is the client bundle.

- Run `yarn start`, you should now see Webpack building your `client-bundle.js` file, and opening `index.html` in your browser should display "Wah wah, I am Browser Toby".

One last thing: unlike our `lib` folder, the `dist/client-bundle.js` and `dist/client-bundle.js.map` files are not being cleaned up by our `clean` task before each build.

- Add `clientBundle: 'dist/client-bundle.js?(.map)'` to our `paths` configuration, and tweak the `clean` task like so:

```javascript
gulp.task('clean', () => del([
  paths.libDir,
  paths.clientBundle,
]));
```

- Add `/dist/client-bundle.js*` to your `.gitignore` file:

Next section: [8 - React](/tutorial/8-react)

Back to the [previous section](/tutorial/6-eslint) or the [table of contents](https://github.com/verekia/js-stack-from-scratch).
