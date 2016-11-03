# 2 - Installare ed usare un pacchetto

In questa sezione imparerai come installare ed utilizzare un pacchetto. Un "pacchetto" è semplicemente del codice che qualcuno ha scritto e messo a disposizione, e che tu puoi sfruttare all'interno delle tue applicazioni. In questo esempio utilizzeremo un pacchetto per la manipolazione dei colori.

- Installa il pacchetto chiamato `color` utilizzando il comando `yarn add color`.

Apri `package.json` per vedere come Yarn ha automaticamente aggiunto `color` nella sezione  `dependencies`, che contiene le "dipendenze" esterne della tua applicazione.

La cartella `node_modules` è stata creata per salvare i pacchetti installati.

- Aggiungi `node_modules/` al tuo file `.gitignore` (ed esegui `git init` per creare un nuovo repository git, se non l'hai ancora fatto).

Ti accorgerai inoltre che Yarn ha creato anche un file `yarn.lock`. Dovresti assicurarti che questo file venga aggiunto al tuo repository git, in questo modo tutti i collaboratori del tuo progetto utilizzeranno la stessa versione del pacchetto. Se stai continuando ad utilizzare NPM invece di Yarn, l'equivalente di questo file è lo *shrinkwrap*.

- Aggiungi `const Color = require('color');` nel file `index.js`
- Puoi utilizzare il pacchetto in questo modo: `const redHexa = Color({r: 255, g: 0, b: 0}).hexString();`
- Aggiungi `console.log(redHexa)`.
- Se esegui `yarn start` dovresti ottenere `#FF0000`.

Complimenti, hai installato ed utilizzato un pacchetto esterno!

Abbiamo utilizzato il pacchetto `color` come esempio di installazione di un semplice pacchetto. Nelle sezioni successive non lo utilizzeremo più, puoi quindi disinstallarlo così:

- Esegui `yarn remove color`

**Nota**: Ci sono due tipi differenti di dipendenze, `"dependencies"` e `"devDependencies"`. `"dependencies"` è più generale di `"devDependencies"`, che contiene pacchetti utilizzati unicamente durante la fase di sviluppo, non in produzione (tipicamente, pacchetti per creare le build, linters, etc). Per le `"devDependencies"`, utilizzeremo il comando `yarn add --dev [package]`.

Prossima sezione: [3 - Configurazione di ES6 con Babel e Gulp](/tutorial/3-es6-babel-gulp)

Torna alla [sezione precedente](/tutorial/1-node-npm-yarn-package-json) o all'[indice](https://github.com/fbertone/js-stack-from-scratch).
