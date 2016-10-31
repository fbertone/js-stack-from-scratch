# 4 - Utilizzare la sintassi ES6 con una classe

- Crea un nuovo file, `src/dog.js`, contenente la seguente classe ES6:

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
    return `Wah wah, I am ${this.name}`;
  }
}

module.exports = Dog;
```

Non dovrebbe sorprenderti se hai già fatto della programmazione ad oggetti in un qualsiasi linguaggio. È comunque abbastanza recente nell'ecosistema Javascript. La classe viene esposta al mondo esterno tramite l'assegnazione a `module.exports`.

Tipicamente il codice ES6 utilizza le classi, `const` e `let`, "modelli di stringhe" (utilizzando gli apici inversi) ad esempio `bark()`, e le arrow function (`(param) => { console.log('Hi'); }`), anche se non ne utilizziamo nessuna in questo semplice esempio.

In `src/index.js`, scrivi questo:

```javascript
const Dog = require('./dog');

const toby = new Dog('Toby');

console.log(toby.bark());
```
Come puoi vedere, a differenza del pacchetto `color` che abbiamo utilizzato in precedenza, quando utilizziamo un modulo scritto da noi utilizziamo il prefisso `./` nella funzione `require()`.

- Esegui `yarn start` e dovresti ottenere 'Wah wah, I am Toby'.

- Dai un'occhiata al codice generato in `lib` per vedere com'è il codice generato (`var` al posto di `const` ad esempio).


Prossima sezione: [5 - La sintassi ES6 per i moduli](/tutorial/5-es6-modules-syntax)

Torna alla [sezione precedente](/tutorial/3-es6-babel-gulp) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
