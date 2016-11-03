# 6 - ESLint

Utilizzeremo un linter per controllare il nostro codice contro potenziali problemi. ESLint è il linter di riferimento per il codice ES6. Invece di configurare da soli li regole per il nostro codice, utilizzeremo la configurazione creata da Airbnb. Questa configurazione utilizza alcuni plugin esterni, dobbiamo quindi installarli perchè tutto funzioni correttamente.

- Esegui `yarn add --dev eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-jsx-a11y eslint-plugin-react`

Come puoi vedere, è possibile installare vari pacchetti contemporaneamente con un unico comando. Tutti verranno aggiunti al file `package.json`, come di consueto.

Nel `package.json`, aggiungi un campo `eslintConfig`:

```json
"eslintConfig": {
  "extends": "airbnb",
  "plugins": [
    "import"
  ]
},
```

La sezione `plugins` serve a comunicare a ESLint che vogliamo utilizzare la sintassi ES6 per gli import.

**Nota**: Un file `.eslintrc.js`, `.eslintrc.json`, oppure `. eslintrc.yaml` inserito nella cartella base del progetto può essere utilizzato in alternativa alla sezione `eslintConfig` nel `package.json`. Come per la configurazione di Babel stiamo cercando di mantenere pulita la cartella del progetto, tuttavia se hai una configurazione di ESLint complessa puoi considerare la creazione di questo file come alternativa.

Creiamo adesso un task Gulp che eseguirà ESLint per noi. Installiamo il plugin ESLint per Gulp:

- Esegui `yarn add --dev gulp-eslint`

Aggiungi questo task nel `gulpfile.babel.js`:

```javascript
import eslint from 'gulp-eslint';

const paths = {
  allSrcJs: 'src/**/*.js',
  gulpFile: 'gulpfile.babel.js',
  libDir: 'lib',
};

// [...]

gulp.task('lint', () => {
  return gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError());
});
```

Stiamo dicendo a Gulp che per questo task vogliamo includere `gulpfile.babel.js` e i file JS contenuti in `src`.

Modifica il task `build` inserendo `lint` come prerequisito, in questo modo:

```javascript
gulp.task('build', ['lint', 'clean'], () => {
  // ...
});
```

- Esegui `yarn start`, e dovresti vedere numerosi errori di linting in questo Gulpfile, e un warning per aver utilizzato `console.log()` in `index.js`.

Un problema che vedrai è `'gulp' should be listed in the project's dependencies, not devDependencies (import/no-extraneous-dependencies)`. In questo caso si tratta di un falso problema. ESLint non è in graado di riconoscere quali file JS fanno parte della nostra build, e quali no, dovremo aiutarlo un po' utilizzando degli appositi commenti all'interno del codice. Nel `gulpfile.babel.js`, all'inizio del file aggiungi:

```javascript
/* eslint-disable import/no-extraneous-dependencies */
```

In questo modo ESLint non applicherà la regola `import/no-extraneous-dependencies` per questo file.

Adesso ci rimane la segnalazione di `Unexpected block statement surrounding arrow body (arrow-body-style)`. Questa è molto utile. ESLint ci sta dicendo che esiste un modo migliore per scrivere:

```javascript
() => {
  return 1;
}
```

Dovrebbe essere sostituito con:

```javascript
() => 1
```

Perchè quando una funzione contiene unicamente un return, in ES6 puoi omettere le parentesi, la parole "return" ed il punto e virgola.

Aggiorniamo quindi il file Gulp:

```javascript
gulp.task('lint', () =>
  gulp.src([
    paths.allSrcJs,
    paths.gulpFile,
  ])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError())
);

gulp.task('clean', () => del(paths.libDir));

gulp.task('build', ['lint', 'clean'], () =>
  gulp.src(paths.allSrcJs)
    .pipe(babel())
    .pipe(gulp.dest(paths.libDir))
);
```

L'ultima segnalazione rimasta si riferisce all'uso di `console.log()`. Diciamo che vogliamo veramente utilizzare la `console.log()` in `index.js` senza ricevere un warning. Come puoi immaginare, inseriremo `/* eslint-disable no-console */` all'inizio del nostro `index.js`.

- Esegui `yarn start` e saremo nuovamente a posto.

**Nota**: Questa sezione ti permette di configurare ESLint nella console. È utile poter eseguire il lint durante la build / prima di eseguire il push del codice, ma probabilmente vorrei integrarlo anche nel tuo IDE. NON utilizzare il linter nativo del tuo IDE quando usi ES6. Configuralo in modo che il linter da eseguire sia quello contenuto all'interno della cartella `node_modules`. In questo modo può utilizzare tutte le configurazioni del tuo progetto, il preset di Airbnb, etc. In caso contrario utilizzesti un linter generico.

Prossima sezione: [7 - App Client con Webpack](/tutorial/7-client-webpack)

Torna alla [sezione precedente](/tutorial/5-es6-modules-syntax) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
