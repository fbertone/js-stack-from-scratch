# 11 - Testing con Mocha, Chai, e Sinon

## Mocha e Chai

- Crea una cartella `src/test`. Questa cartella rispecchiera' la struttura delle cartelle della nostra applicazione, crea quindi anche `src/test/client` (se vuoi puoi aggiungere anche `server` e `shared`, ma non scriveremo dei test per quelle).

- In `src/test/client`, crea `state-test.js` che utilizzeremo per testare il ciclo della nostra applicazione Redux.

Useremo [Mocha](http://mochajs.org/) come framework di test principale. Mocha e' semplice da usare, ha molte funzionalita', ed e' attualmente il [framework di test per JavaScript piu' popolare](http://stateofjs.com/2016/testing/). E' molto flessibile e modulare. In particolare, ti permette di utilizzare la libreria di assert che vuoi. [Chai](http://chaijs.com/) e' un'ottima libreria di assert, sono disponibili molti [plugin](http://chaijs.com/plugins/) e ti permette di scegliere diversi stili di assert.

- Installiamo Mocha e Chai eseguendo `yarn --dev mocha chai`

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

- Crea il task `test` seguente, che utilizza il plugin `gulp-mocha`:

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

In some cases, we want to be able to *fake* things in a unit test. For instance, let's say we have a function, `deleteEverything`, which contains a call to `deleteDatabases()`. Running `deleteDatabases()` causes a lot of side-effects, which we absolutely don't want to happen when running our test suite.

[Sinon](http://sinonjs.org/) is a testing library that offers **Stubs** (and a lot of other things), which allow us to neutralize `deleteDatabases` and simply monitor it without actually calling it. This way we can test if it got called, or which parameters it got called with for instance. This is typically very useful to fake or avoid AJAX calls - which can cause side-effects on the back-end.

In the context of our app, we are going to add a `barkInConsole` method to our `Dog` class in `src/shared/dog.js`:

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

If we run `barkInConsole` in a unit test, `console.log()` will print things in the terminal. We are going to consider this to be an undesired side-effect in the context of our unit tests. We are interested in knowing if `console.log()` *would have normally been called* though, and we want to test what parameters it *would have been called with*.

- Create a new `src/test/shared/dog-test.js` file, and add write the following:

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

Here, we are using *stubs* from Sinon, and a Chai plugin to be able to use Chai assertions on Sinon stubs and such.

- Run `yarn add --dev sinon sinon-chai` to install these libraries.

So what is new here? Well first of all, we call `chai.use(sinonChai)` to activate the Chai plugin. Then, all the magic happens in the `it()` statement: `stub(console, 'log')` is going to neutralize `console.log` and monitor it. When `new Dog('Test Toby').barkInConsole()` is executed, a `console.log` is normally supposed to happen. We test this call to `console.log` with `console.log.should.have.been.calledWith()`, and finally, we `restore` the neutralized `console.log` to make it work normally again.

**Important note**: Stubbing `console.log` is not recommended, because if the test fails, `console.log.restore()` is never called, and therefore `console.log` will remain broken for the rest of the command you executed in your terminal! It won't even print the error message that caused the test to fail, so it leaves you with very little information about what happened. That can be quite confusing. It is a good example to illustrate stubs in this simple app though.

If everything went well in this chapter, you should have 2 passing tests.

Next section: [12 - Type Checking with Flow](/tutorial/12-flow)

Back to the [previous section](/tutorial/10-immutable-redux-improvements) or the [table of contents](https://github.com/verekia/js-stack-from-scratch).
