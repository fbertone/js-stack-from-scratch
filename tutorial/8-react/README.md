# 8 - React

Adesso utilizzeremo React per la visualizzazione della nostra app.

Prima di tutto installiamo React e ReactDOM:

- Esegui `yarn add react react-dom`

Questi due pacchetti vanno nella sezione `"dependencies"` e non `"devDependencies"` perche a differenza dei tool utilizzati per lo sviluppo, il client in produzione ne avra bisogno.

Rinominiamo il nostro file `src/client/app.js` in `src/client/app.jsx` e scriviamoci un po' di codice React e JSX:

```javascript
import 'babel-polyfill';

import React, { PropTypes } from 'react';
import ReactDOM from 'react-dom';
import Dog from '../shared/dog';

const dogBark = new Dog('Browser Toby').bark();

const App = props => (
  <div>
    The dog says: {props.message}
  </div>
);

App.propTypes = {
  message: PropTypes.string.isRequired,
};

ReactDOM.render(<App message={dogBark} />, document.querySelector('.app'));
```

**Nota**: Se non sei pratico di React o le PropTypes, studia prima il funzionamento di React e ritorna a seguire questo tutorial dopo. Nei capitoli successivi utilizzeremo parecchio React, assicurati quindi di riuscire a seguire quello che faremo.

Nel Gulpfile, cambia il valore di `clientEntryPoint` modificandone l'estensione a `.jsx`:

```javascript
clientEntryPoint: 'src/client/app.jsx',
```

Siccome utilizzeremo la sintassi JSX, dobbiamo dire a Babel che deve occuparsi anche di questa conversione.
Installa il preset React Babel, che permettera a Babel di convertire la sintassi JSX:
`yarn add --dev babel-preset-react` e cambia la voce `babel` nel `package.json` in questo modo:

```json
"babel": {
  "presets": [
    "latest",
    "react"
  ]
},
```

Adesso, dopo aver eseguito `yarn start`, se apriamo `index.html` dovremmo vedere la scritta "The dog says: Wah wah, I am Browser Toby" inserita da React.

Prossima sezione: [9 - Redux](/tutorial/9-redux)

Torna alla [sezione precedente](/tutorial/7-client-webpack) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
