# 11 - Testare con Mocha, Chai, e Sinon

## Mocha e Chai

- Crea una cartella `src/test`. Questa cartella rispecchiera' la struttura delle cartelle della nostra applicazione, crea quindi anche `src/test/client` (se vuoi puoi aggiungere anche `server` e `shared`, ma non scriveremo dei test per quelle).

- In `src/test/client`, crea `state-test.js` che utilizzeremo per testare il ciclo della nostra applicazione Redux.

Useremo [Mocha](http://mochajs.org/) come framework di test principale. Mocha e' semplice da usare, ha molte funzionalita', ed e' attualmente il [framework di test per JavaScript piu' popolare](http://stateofjs.com/2016/testing/). E' molto flessibile e modulare. In particolare, ti permette di utilizzare la libreria di assert che vuoi. [Chai](http://chaijs.com/) e' un'ottima libreria di assert, sono disponibili molti [plugin](http://chaijs.com/plugins/) e ti permette di scegliere diversi stili di assert.

- Installiamo Mocha e Chai eseguendo `yarn add --dev mocha chai`

In `state-test.js`, scrivi:

```javascript
/* eslint-disable import/no-extraneous-dependencies, no-unused-expressions */

import { createStore } from 'redux';
import { combineReducers } from 'redux-immutable';
import { should } from 'chai';
import { describe, it, beforeEach } from 'mocha';
import dogReducer from '../../client/reducers/dog-reducer';
import { makeBark } from '../../client/actions/dog-actions';

should();
let store;

describe('App State', () => {
  describe('Dog', () => {
    beforeEach(() => {
      store = createStore(combineReducers({
        dog: dogReducer,
      }));
    });
    describe('makeBark', () => {
      it('should make hasBarked go from false to true', () => {
        store.getState().getIn(['dog', 'hasBarked']).should.be.false;
        store.dispatch(makeBark());
        store.getState().getIn(['dog', 'hasBarked']).should.be.true;
      });
    });
  });
});
```

OK, analizziamo il tutto.

Innanzitutto, nota come importiamo lo stile di assert `should` da `chai`. Questo ci permette di testare cose come `mynumber.should.equal(3)`, abbastanza chiaro. Per chiamare `should` su un qualsiasi oggetto, dobbiamo eseguire la funzione `should()` prima di qualsiasi altra cosa. Alcune di queste verifiche sono *espressioni*, come `mybook.should.be.true`, che faranno arrabbiare ESLint, abbiamo quindi aggiunto un commento ESLint in cima al file, in modo da disabilitare la regola `no-unused-expressions`.

I test di Mocha funzionano ad albero. Nel nostro caso vogliamo testare la funzione `makeBark`, che dovrebbe modificare l'attributo `dog` dello stato dell'applicazione, ha quindi senso utilizzare la seguente sequenza dii test: `App State > Dog > makeBark`, che dichiariamo utilizzando `describe()`. `it()` e' la funzione di test effettiva, `beforeEach()` e' una funzione che viene richiamata prima di ogni test `it()`. Nel nostro caso vogliamo un nuovo store prima dell'esecuzione di un nuovo test. Dichiariamo una variabile `store` all'inizio del file perche' tornera' utile in ogni test successivo.

Il nostro test per `makeBark` e' molto esplicito, e la descrizione, inserita come stringa in `it()`, lo rende ancora piu' chiaro: verifichiamo che `hasBarked` va da `false` a `true` dopo aver chiamato `makeBark`.

Bene, eseguiamo questo test!

- In `gulpfile.babel.js`, crea il seguente task `test` , che utilizza il plugin `gulp-mocha`:

```javascript
import mocha from 'gulp-mocha';

const paths = {
  // [...]
  allLibTests: 'lib/test/**/*.js',
};

// [...]

gulp.task('test', ['build'], () =>
  gulp.src(paths.allLibTests)
    .pipe(mocha())
);
```

- Esegui `yarn add --dev gulp-mocha`.

Come puoi notare, i test sono eseguiti sul codice convertito presente in `lib`, ecco perche' `build` e' un task che deve essere eseguito come prerequisito di `test`. `build` a sua volta ha un prerequisito, `lint`, ed in fine renderemo `test` un prerequisito di `main`, che ci porta ad avere la seguente cascata di task per `default`: `lint` > `build` > `test` > `main`.

- Modifica il prerequisito di `main` a `test`:

```javascript
gulp.task('main', ['test'], () => /* ... */ );
```

- In `package.json`, sostituisci lo script `"test"` attuale con: `"test": "gulp test"`. In questo modo puoi eseguire `yarn test` per lanciare semplicemente i test. `test` e' inoltre lo script standard che viene lanciato dai tool utilizzati nei servizi di continuous integration, dovresti quindi collegarci sempre i tuoi test. `yarn start` eseguira' inoltre i test prima di fare la build Webpack del client, lo generera' quindi se tutti i test hanno successo.

- Esegui `yarn test` o `yarn start`, e dovrebbe uscire il risultato dei nostri test, si sprea che sia verde.

## Sinon

In alcuni casi avremo la necessita' di *falsificare* cose per eseguire un test. Ad esempio, immaginiamo di avere una funzione, `deleteEverything`, che contiene una chiamata a `deleteDatabases()`. Eseguire realmente `deleteDatabases()` causerebbe molti effetti secondari, cosa che non vogliamo succeda quando eseguiamo dei test.

[Sinon](http://sinonjs.org/) e' una libreria di test che offre la possibilita' di creare **Stubs** (oltre a molte altre cose), che ci permette di neutralizzare `deleteDatabases` semplicemente monitorandola senza andarla ad eseguire realmente. In questo modo possiamo verificare se e' stata chiamata, oppure, ad esempio, con quali parametri e' stata richiamata. Questo tipicamente e' molto utile per simulare chiamate AJAX - che possono avere molti effetti sul back-end.

INel contesto della nostra applicazione, aggiungeremo un metodo `barkInConsole` alla nostra classe `Dog` in `src/shared/dog.js`:

```javascript
class Dog {
  constructor(name) {
    this.name = name;
  }

  bark() {
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

Se eseguiamo `barkInConsole` in un test, `console.log()` scrivera' delle cose nel terminale. Nel contesto dei nostri test, questo e' considerato un effetto indesiderato. Siamo tuttavia interessati a conoscere se `console.log()` *sarebbe stata di norma chiamata*, e vogliamo verificare con quali parametri *sarebbe stata chiamata*.

- Crea un nuovo file `src/test/shared/dog-test.js` e aggiungi quanto segue:

```javascript
/* eslint-disable import/no-extraneous-dependencies, no-console */

import chai from 'chai';
import { stub } from 'sinon';
import sinonChai from 'sinon-chai';
import { describe, it } from 'mocha';
import Dog from '../../shared/dog';

chai.should();
chai.use(sinonChai);

describe('Shared', () => {
  describe('Dog', () => {
    describe('barkInConsole', () => {
      it('should print a bark string with its name', () => {
        stub(console, 'log');
        new Dog('Test Toby').barkInConsole();
        console.log.should.have.been.calledWith('Wah wah, I am Test Toby');
        console.log.restore();
      });
    });
  });
});
```

Qua abbiamo utilizzato delle *stubs* di Sinon, ed un plugin di Chai per poter eseguire delle verifiche di Chai sulle stub di Sinon.

- Esegui `yarn add --dev sinon sinon-chai` per installare queste librerie.

Cosa c'e' di nuovo? Prima di tutto, chiamiamo `chai.use(sinonChai)` per attivare il plugin di Chai. Poi tutta la magia avviene nel costrutto `it()`: `stub(console, 'log')` serve a neutralizzare `console.log` e monitorarla. Quando `new Dog('Test Toby').barkInConsole()` viene eseguita, si suppone che avvenga una `console.log`. Testiamo questa chiamata a `console.log` tramite `console.log.should.have.been.calledWith()`, ed infine facciamo un `restore` della `console.log` neutralizzata, in modo da farla nuovamente funzionare.

**Nota importante**: Fare lo stub di `console.log` non e' raccomandabile, perche' se il test fallisce, `console.log.restore()` non viene mai chiamata, di conseguenza `console.log` rimarra' non funzionante per i successivi comandi che andrai ad eseguire sul terminale! Non riportera' neppure l'errore che ha fatto fallire il test, lasciandoti quindi con poche informazioni su quello che e' successo. Questo puo' confonderti abbastanza. Tuttavia e' un buon esempio per dimostrare l'utilizzo delle stub in questo contesto.

Se tutto e' andato per il meglio in questo capitolo, dovresti avere due test passati con successo.

Prossima sezione: [12 - Type Checking con Flow](/tutorial/12-flow)

Torna alla [sezione precedente](/tutorial/10-immutable-redux-improvements) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
