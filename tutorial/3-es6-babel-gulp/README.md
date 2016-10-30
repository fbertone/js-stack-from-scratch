# 3 - Configurazione di ES6 con Babel e Gulp

Adesso utilizzeremo la nuova sintassi di ES6, che rappresenta un notevole miglioramento rispetto alla "vecchia" sintassi ES5. Tutti i browser sono in grado di comprendere la sintassi ES5, ma per il momento non supportano ancora ES6. Utilizzeremo un tool chiamato Babel per trasformere il codice ES6 in codice compatibile con ES5. Per eseguire Babel, utilizzeremo Gulp, un task runner. È simile ai comandi presenti nella sezione `scripts` del `package.json`, ma scrivere i tuoi task in un file JS è più facile e pulito rispetto e scriverli in un file JSON, installeremo quindi Gulp, ed il suo plugin per Babel:

- Esegui `yarn add --dev gulp`
- Esegui `yarn add --dev gulp-babel`
- Esegui `yarn add --dev babel-preset-latest`
- Nel `package.json`, aggiungi un campo `babel` per la configurazione di Babel. Fagli utilizzare l'ultima versione di Babel inserendo:

```json
"babel": {
  "presets": [
    "latest"
  ]
},
```

**Nota**: Un file `.babelrc` nella cartella principale del tuo progetto può essere utilizzato al posto del campo `babel` nel `package.json`. La cartella principale del tuo progetto diventerà sempre più piena con l'avanzare dello sviluppo, cerca quindi di utilizzare `package.json` per mantenere la configurazione di Babel.

- Sposta il file `index.js` in una nuova cartella `src`. Là dentro è dove scriverai tutto il tuo codice ES6. La cartella `lib` conterrà il codice ricompilato in ES5. Gulp e Babel si occuperanno di creare il tutto. Rimuovi tutti il codice precedente relativo a `color`-related nel file `index.js`, e sostituiscilo con:

```javascript
const str = 'ES6';
console.log(`Hello ${str}`);
```

Stiamo utilizzando un *modello* (template) di stringa, si tratta di una nuova funzionalità di ES6 che permette di inserire variabili all'interno di una stringa utilizzando `${}`, senza dover più concatenare i vari pezzetti della stringa.

- Crea un file `gulpfile.js` contenente:

```javascript
const gulp = require('gulp');
const babel = require('gulp-babel');
const del = require('del');
const exec = require('child_process').exec;

const paths = {
  allSrcJs: 'src/**/*.js',
  libDir: 'lib',
};

gulp.task('clean', () => {
  return del(paths.libDir);
});

gulp.task('build', ['clean'], () => {
  return gulp.src(paths.allSrcJs)
    .pipe(babel())
    .pipe(gulp.dest(paths.libDir));
});

gulp.task('main', ['build'], (callback) => {
  exec(`node ${paths.libDir}`, (error, stdout) => {
    console.log(stdout);
    return callback(error);
  });
});

gulp.task('watch', () => {
  gulp.watch(paths.allSrcJs, ['main']);
});

gulp.task('default', ['watch', 'main']);

```

Cerchiamo di capire il contenuto di questo file.

Le API di Gulp sono abbastanza lineari. Definiscono dei `gulp.task`, che possono fare riferimento a dei file sorgente `gulp.src`, applicandogli una catena di modificatori tramite `.pipe()` (ad esempio nel nostro caso `babel()`) e inserisce il file di outputi in `gulp.dest`. Può inoltre utilizzare `gulp.watch` per controllare i cambiamenti nei file. I task i Gulppossono eseguire degli altri task preparatori, utilizzando un array in questo formato (es: `['build']`) come secondo parametro di `gulp.task`. Fai riferimento alla [documentazione](https://github.com/gulpjs/gulp) per una spiegazione più articolata.

Prima definiamo un oggetto `paths` per memorizzare tutti i percorsi dei file e mantenere una configurazione pulita.

Definiamo poi 5 task: `build`, `clean`, `main`, `watch`, e `default`.

- `build` è dove viene chiamato Babel per trasformare tutti i nostri file sorgente nella cartella `src` e scrivere quelli trasformati in `lib`.
- `clean` che cancella tutti i file autogenerati in `lib` prima di eseguire ogni task `build`. Questo è utile per ripulire tutti i file compilati che non sono più necessari, ad esempio perchè gli originali in `src` sono stati cancellati o rinominati , o per assicurarci che la cartella `lib` sia sempre sincronizzata con `src` nel caso in cui la build fallisca. Utilizziamo il pacchetto `del` per cancellare i file in un modo che si integra bene con il flusso di processamento di Gulp (questo è il metodo [raccomandato](https://github.com/gulpjs/gulp/blob/master/docs/recipes/delete-files-folder.md) per cancellare file con Gulp). Esegui `yarn add --dev del` per installare questo pacchetto.
- `main` è l'equivalente di  `node .` che abbiamo utilizzato nel capitolo precedente, con l'eccezione che questa voltavogliamo eseguirlo sul file `lib/index.js`. Siccome `index.js` è il file che Node cerca di default, possiamo semplicemente scrivere `node lib` (utilizziamo la variabile `libDir` per mantenere la configurazione pulita). `require('child_process').exec` e `exec` sono delle funzioni native di Node per l'esecuzione di comandi su shell. Redirigiamo `stdout` su `console.log()` ritorniamo eventuali errori tramite la callback di `gulp.task`. Non preoccuparti se non capisci nei dettagli quello che sta succedendo, ricordati che è semplicemente l'equivalente di eseguire `node lib`.
- `watch` esegue il task `main` quando vengono riscontrate delle modifiche nei file elencati.
- `default` è un task specifico che viene eseguito se esegui semplicemente `gulp` da terminale. Nel nostro caso vogliamo fargli eseguire sia `watch` che `main` (per la prima esecuzione).

**Nota**: Potresti chiederti perchè stiamo utilizzando direttamente del codice ES6 nel file Gulp, senza convertirlo in ES5 con Babel. È perchè stiamo utilizzando una versione di Node che supporta nativamente ES6 (assicurati di utilizzare Node > 6.5.0 con il comando `node -v`).

OK! Vediamo se funziona tutto.

- Nel `package.json`, cambia lo script `start` con: `"start": "gulp"`.
- Esegui `yarn start`. Dovrebbe scrivere "Hello ES6" ed iniziare a controllare le modifiche sui file. Prova ad inserire del codice non funzionante in `src/index.js` per verificare che Gulp ti mostra automaticamente l'errore appena salvi il file.

- Aggiungi `/lib/` al tuo `.gitignore`


Prossima sezione: [4 - Utilizzare la sintassi ES6 scon una classe](/tutorial/4-es6-syntax-class)

Torna alla [sezione precedente](/tutorial/2-packages) o all'[indice](https://github.com/verekia/js-stack-from-scratch).
