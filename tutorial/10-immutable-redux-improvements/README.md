# 10 - Immutable JS e Redux Improvements

## Immutable JS

A differenza del capitolo precedente, questo é abbastanza semplice e consiste in piccoli miglioramenti.

Per prima cosa aggiungeremo **Immutable JS** al nostro codice. Immutable é una libreria per manipolare oggetti senza modificarli. Invece di fare:

```javascript
const obj = { a: 1 };
obj.a = 2; // Mutates `obj`
```
Fraremmo:
```javascript
const obj = Immutable.Map({ a: 1 });
obj.set('a', 2); // Returns a new object without mutating `obj`
```

Questo approccio segue il paradigma della **programmazione funzionale**, che funziona molto bene con Redux. Le tue funzioni reducer *devono* essere funzioni pure che non alterano lo stato passato come parametro, ma ritornano invece un nuovo oggetto. Usiamo Immutable per assicurarci che sia cosí.

- Esegui `yarn add immutable`

Utilizzeremo `Map` nel nostro codice, ma ESLint e la configurazione Airbnb si lamenteranno dell'utilizzo di una dell'iniziale maiuscola per quella che non é una classe. Aggiungi questo al `package.json` sotto `eslintConfig`:

```json
"rules": {
  "new-cap": [
    2,
    {
      "capIsNewExceptions": [
        "Map",
        "List"
      ]
    }
  ]
}
```
Questo fa in modo che `Map` e `List` (i due oggetti di Immutable che utilizzeremo in continuazione) siano delle eccezioni alle regole di ESLint. Questa formattazione JSON cosí prolissa é generata automaticamente da Yarn/NPM, non possiamo quindi renderla piú compatta.

Ad ogni modo, torniamo a Immutable:

Modifica `dog-reducer.js` in modo che diventi cosí:

```javascript
import Immutable from 'immutable';
import { MAKE_BARK } from '../actions/dog-actions';

const initialState = Immutable.Map({
  hasBarked: false,
});

const dogReducer = (state = initialState, action) => {
  switch (action.type) {
    case MAKE_BARK:
      return state.set('hasBarked', action.payload);
    default:
      return state;
  }
};

export default dogReducer;
```

Lo stato iniziale viene adesso creato utilizzando una Map di Immutable, e il nuovo stato viene generato utilizzando `set()`, impedendo ogni modifica allo stato precedente.

In `containers/bark-message.js`, aggiorna la funzione `mapStateToProps` in modo da usare `.get('hasBarked')` al posto di `.hasBarked`:

```javascript
const mapStateToProps = state => ({
  message: state.dog.get('hasBarked') ? 'The dog barked' : 'The dog did not bark',
});
```

L'app dovrebbe ancora comportarsi esattamente come prima.

**Nota**: Se Babel si lamenta del fatto che Immutable supera i 100KB, aggiungi `"compact": false` nel `package.json` alla voce `babel`.

Come puoi vedere dal pezzo di codice precedente, il nostro oggetto di stato contiene ancora l'attributo `dog` che non é immutabile. Va bene cosí, ma se vuoi manipolare esclusivamente oggetti immutabili, puoi installare il pacchetto `redux-immutable` per sostituire la funzione `combineReducers` di Redux.

**Opzionale**:
- Esegui `yarn add redux-immutable`
- Sostituisci la funzione `combineReducers` in `app.jsx` per usare quella importata da `redux-immutable`.
- In `bark-message.js` sostituisci `state.dog.get('hasBarked')` con `state.getIn(['dog', 'hasBarked'])`.

## Azioni Redux

Man mano che aggiungi sempre piú azioni alla tua app, ti accorgerai che stai scrivendo molto codice ripetuto. Il pacchetto `redux-actions` ti aiuterá a ridurre le ripetizioni. Con `redux-actions` puoi riscrivere il file `dog-actions.js` in modo piú compatto:

```javascript
import { createAction } from 'redux-actions';

export const MAKE_BARK = 'MAKE_BARK';
export const makeBark = createAction(MAKE_BARK, () => true);
```

`redux-actions` implementa il modello di [Azione Standard Flux](https://github.com/acdlite/flux-standard-action), proprio come l'azione che abbiamo scritto precedentemente, quindi integrare `redux-actions` sará del tutto lineare se segui questo modello.

- Non dimenticarti di eseguire `yarn add redux-actions`.

Prossima sezione: [11 - Testing with Mocha, Chai, and Sinon](/tutorial/11-testing-mocha-chai-sinon)

Torna alla [sezione precedente](/tutorial/9-redux) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
