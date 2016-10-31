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

Our store is created by the Redux function `createStore`, pretty explicit. The store object is assembled by combining all our reducers (in our case, only one) using Redux's `combineReducers` function. Each reducer is named here, and we'll name ours `dog`.

That's pretty much it for the pure Redux part.

Now we are going to hook up Redux with React using `react-redux`. In order for `react-redux` to pass the store to our React app, it needs to wrap the entire app in a `<Provider>` component. This component must have a single child, so we created a `<div>`, and this `<div>` contains the 2 main elements of our app, a `BarkMessage` and a `BarkButton`.

As you can tell in the `import` section, `BarkMessage` and `BarkButton` are imported from a `containers` folder. Now is a good time to introduce the concept of **Components** and **Containers**.

*Components* are *dumb* React components, in a sense that they don't know anything about the Redux state. *Containers* are *smart* components that know about the state and that we are going to *connect* to our dumb components.

- Create 2 folders, `src/client/components` and `src/client/containers`.

- In `components`, create the following files:

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

and **message.jsx**:

```javascript
import React, { PropTypes } from 'react';

const Message = ({ message }) => <div>{message}</div>;

Message.propTypes = {
  message: PropTypes.string.isRequired,
};

export default Message;

```

These are examples of *dumb* components. They are logic-less, and just show whatever they are asked to show via React **props**. The main difference between `button.jsx` and `message.jsx` is that `Button` contains an **action** in its props. That action is bound on the `onClick` event. In the context of our app, the `Button` label is never going to change, however, the `Message` component is going to reflect the state of our app, and will vary based on the state.

Again, *components* don't know anything about Redux **actions** or the **state** of our app, which is why we are going to create smart **containers** that will feed the proper *actions* and *data* to these 2 dumb components.

- In `containers`, create the following files:

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

and **bark-message.js**:

```javascript
import { connect } from 'react-redux';
import Message from '../components/message';

const mapStateToProps = state => ({
  message: state.dog.hasBarked ? 'The dog barked' : 'The dog did not bark',
});

export default connect(mapStateToProps)(Message);
```

`BarkButton` will hook up `Button` with the `makeBark` action and Redux's `dispatch` method, and `BarkMessage` will hook up the app state with `Message`. When the state changes, `Message` will now automatically re-render with the proper `message` prop. These connections are done via the `connect` function of `react-redux`.

- You can now run `yarn start` and open `index.html`. You should see "The dog did not bark" and a button. When you click the button, the message should show "The dog barked".

Next section: [10 - Immutable JS and Redux Improvements](/tutorial/10-immutable-redux-improvements)

Back to the [previous section](/tutorial/8-react) or the [table of contents](https://github.com/verekia/js-stack-from-scratch).
