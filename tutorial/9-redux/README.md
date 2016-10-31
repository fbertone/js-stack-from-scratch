# 9 - Redux

In questo capitolo (che é il piú difficile finora) aggiungeremo [Redux](http://redux.js.org/) alla nostra applicazione e lo collegheremo a React. Redux controlla lo stato della tua applicazione. É composto da uno **store** che e'un oggetto Javascript che rappresenta lo stato dell'app, **actions**  che sono tipicamente scatenate dagli utenti e **reducers** che possono essere visto come handler delle azioni. I reducer modificano lo stato dell'applicazione (lo *store*) e quando lo stato dell'applicazione viene modificato delle cose succedono nell'app. Una buona rappresentazione grafica di Redux é disponibile [qua](http://slides.com/jenyaterpil/redux-from-twitter-hype-to-production#/9).

Per dimostrare come utilizzare Redux nel modo piú semplice possibile, la lostra app consiste in un messaggio ed un bottone. Il messaggio indica se il cane ha abbaiato o no (inizialmente lo stato é impostato su no) ed il pulsante fa abbaiare il cane, aggiornando di conseguenza il messaggio.

Ci serviremo di due pacchetti: `redux` e `react-redux`.

- Esegui `yarn add redux react-redux`.

Iniziamo creando due cartelle: `src/client/actions` e `src/client/reducers`.

- In `actions`, crea `dog-actions.js`:

```javascript
export const MAKE_BARK = 'MAKE_BARK';

export const makeBark = () => ({
  type: MAKE_BARK,
  payload: true,
});
```
Qua definiremo un tipo di azione, `MAKE_BARK`, e una funzione (anche conosciuta come *action creator*) che scatena un'azione `MAKE_BARK` chiamata `makeBark`. Entrambe sono esportate perché ci serviranno in altri file. Questa azione implementa una modello di [Azione Flux Standard](https://github.com/acdlite/flux-standard-action), ecco perché ha degli attributi `type` e `payload`.

- In `reducers`, crea `dog-reducer.js`:

```javascript
import { MAKE_BARK } from '../actions/dog-actions';

const initialState = {
  hasBarked: false,
};

const dogReducer = (state = initialState, action) => {
  switch (action.type) {
    case MAKE_BARK:
      return { hasBarked: action.payload };
    default:
      return state;
  }
};

export default dogReducer;
```

Qua definiamo lo stato iniziale della nostra app, che é un oggetto contenente la proprietá `hasBarked` impostata a `false`, e `dogReducer`, che é la funzione responsabile di alterare lo stato in base all'azione che é stata eseguita. Lo stato non puó essere modificato in questa funzione, occorre effettuare il return di un nuovo oggetto di stato.

- Modifichiamo `app.jsx` per creare lo *store*. Puoi sostituire il file precedente con questo:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore, combineReducers } from 'redux';
import { Provider } from 'react-redux';
import dogReducer from './reducers/dog-reducer';
import BarkMessage from './containers/bark-message';
import BarkButton from './containers/bark-button';

const store = createStore(combineReducers({
  dog: dogReducer,
}));

ReactDOM.render(
  <Provider store={store}>
    <div>
      <BarkMessage />
      <BarkButton />
    </div>
  </Provider>
  , document.querySelector('.app')
);
```

Il nostro store é creato tramite la funzione Redux `createStore`, é abbastanza esplicito. L'oggetto store é assemblato combinando tutti i reducer (nel nostro caso solo uno) utilizzando la funzione di Redux `combineReducers`. Qua viene assegnato un nome ad ogni reducer, e chiameremo il nostro `dog`.

Questo é tutto per la parte di Redux puro.

Adesso agganceremo Redux con React utilizzando `react-redux`. Per fare in modo che `react-redux` passi lo store alla nostra app React, dobbiamo incapsulare il tutto in un componente `<Provider>`. Questo componente deve avere un unico figlio, quindi abbiamo creato un `<div>`, e questo `<div>` contiene i due elementi principali della nostra app: `BarkMessage` e `BarkButton`.

Come puoi vedere dalla sezione `import`, `BarkMessage` e `BarkButton` sono importati da una cartella `containers`. Adesso é il momento buono per introdurre i concetti di **Components** e **Containers**.

I *Components* sono componenti React *stupidi*, nel senso che non conoscono nulla dello stato di Redux. I *Containers* sono componenti *intelligenti* che sono a conoscenza dello stato e verranno utilizzati per *collegare* i nostri componenti stupidi.

- Crea 2 cartelle, `src/client/components` e `src/client/containers`.

- In `components`, crea questi file:

**button.jsx**

```javascript
import React, { PropTypes } from 'react';

const Button = ({ action, actionLabel }) => <button onClick={action}>{actionLabel}</button>;

Button.propTypes = {
  action: PropTypes.func.isRequired,
  actionLabel: PropTypes.string.isRequired,
};

export default Button;
```

e **message.jsx**:

```javascript
import React, { PropTypes } from 'react';

const Message = ({ message }) => <div>{message}</div>;

Message.propTypes = {
  message: PropTypes.string.isRequired,
};

export default Message;

```

Questi sono esempi di componenti *stupidi*. Non contengono una logica, e mostrano semplicemente quello che gli viene chiesto di mostrare tramite le **props** di React. La differenza principale tra `button.jsx` e `message.jsx` é che `Button` contiene un'**action** fra le sue props. Questa azione é collegata all'evento `onClick`. Nel contesto della nostra applicazione, la label del `Button` non cambierá mai, tuttavia, il componente `Message` rifletterá lo stato dell'app e varierá di conseguenza.

Ripeto: i *components* non sanno nulla delle **actions** di Redux o dello **stato** dell'app, per questo motivo creeremo dei **containers** che forniranno le *azioni* e i *dati* a questi due componenti stupidi.

- In `containers`, crea questi file:

**bark-button.js**

```javascript
import { connect } from 'react-redux';
import Button from '../components/button';
import { makeBark } from '../actions/dog-actions';

const mapDispatchToProps = dispatch => ({
  action: () => { dispatch(makeBark()); },
  actionLabel: 'Bark',
});

export default connect(null, mapDispatchToProps)(Button);
```

e **bark-message.js**:

```javascript
import { connect } from 'react-redux';
import Message from '../components/message';

const mapStateToProps = state => ({
  message: state.dog.hasBarked ? 'The dog barked' : 'The dog did not bark',
});

export default connect(mapStateToProps)(Message);
```

`BarkButton` collegherá il `Button` con l'azione `makeBark` ed il metodo `dispatch` di React, e `BarkMessage` collegherá lo stato dell'applicazione con `Message`. Quando lo stato cambia, `Message` verrá automaticamente ricaricato con la prop `message` corretta. Questi collegamenti sono fatti tramite la funzione `connect` di `react-redux`.

- Puoi eseguire `yarn start` ed aprire `index.html`. Dovresti vedere "The dog did not bark" ed un bottone. Quando clicki sul bottone, il messaggio dovrebbe diventare "The dog barked".

Prossima sezione: [10 - Immutable JS and Redux Improvements](/tutorial/10-immutable-redux-improvements)

Torna alla [sezione precedente](/tutorial/8-react) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
