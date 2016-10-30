# 5 - La sintassi ES6 per i moduli

Sostituiamo semplicemente `const Dog = require('./dog')` con `import Dog from './dog'`, che è la nuova sintassi ES6 per i modili (a deifferenza della sintassi "CommonJS").

In `dog.js`, sostituiamo inoltre `module.exports = Dog` con `export default Dog`.

Nota che in `dog.js`, il nome `Dog` è usato unicamente nell'`export`. Sarebbe quindi possibile esportare direttamente una funzione anonima utilizzando la seguente sintassi:

```javascript
export default class {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}
```

Puoi quindi immaginare che il nome 'Dog' usato nell'`import` in `index.js` è a tua discrezione. Funzionerebbe ugualmente:

```javascript
import Cat from './dog';

const toby = new Cat('Toby');
```
Ovviamente la maggiorparte delle volte utilizzerai lo stesso nome utilizzato dalle classe/modulo che stai importando.
Un esempio in cui non lo facciamo è `const babel = require('gulp-babel')` nel nostro file Gulp.

Allora cosa sono questi `require()` nel nostro `gulpfile.js`? Possiamo sostituirli con `import`? L'ultima versione di Node supporta la maggiorparte delle feature di ES6, ma non ancora i moduli. Fortunatamente per noi Gulp può utilizzare Babel per aiutarci. Se rinominiamo `gulpfile.js` in `gulpfile.babel.js`, Babel si occuperà di passare i moduli in `import`a Gulp.

- Rinomina `gulpfile.js` come `gulpfile.babel.js`

- Sostituisci i `require()` con:

```javascript
import gulp from 'gulp';
import babel from 'gulp-babel';
import del from 'del';
import { exec } from 'child_process';
```

Nota il modo in cui abbiamo estatto la funzione `exec` direttamente da `child_process`. Abbastanza elegante!

- `yarn start` dovrebbe continuare a scrivere "Wah wah, I am Toby".

Prossima sezione: [6 - ESLint](/tutorial/6-eslint)

Torna alla [sezione precedente](/tutorial/4-es6-syntax-class) o all'[indice](/js-stack-from-scratch).
