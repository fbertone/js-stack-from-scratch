# 1 - Node, NPM, Yarn, e package.json

In questa sezione configureremo Node, NPM, Yarn e un file `package.json` di base.

Prima di tutto dobbiamo installare Node, che non è utilizzato solo per il back-end ma è la base di tutti gli strumenti necessari per creare uno stack Javascript moderno lato Front-End.

Vai alla [pagina di download](https://nodejs.org/en/download/current/) per installare su Windows o mac oppure segui le istruzioni sulla [pagina di installazione](https://nodejs.org/en/download/package-manager/) per le distribuzioni Linux.


Ad esempio, su **Ubuntu / Debian**, devi eseguire i seguenti comandi per installare Node:

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```
Ti serve una qualsiasi versione di Node > 6.5.0.

`npm`, il package manager standard di Node, viene installato automaticamente assieme a Node, non devi quindi installarlo tu a mano.

**Nota**: Se Node è già installato, installa `nvm` ([Node Version Manager](https://github.com/creationix/nvm)), utilizza `nvm` per installare in automatico l'ultima versione di Node.

[Yarn](https://yarnpkg.com/) è un nuovo package manager ed è molto più veloce rispetto ad NPM, mantiene una cache dei pacchetti installati, e gestisce le dipendenze in modo più prevedibile. Da quando è [uscito](https://code.facebook.com/posts/1840075619545360) a ottobre 2016, è stato adottato molto rapidamente e sta diventando il package manager di riferimento per la comunità Javascript. In questo tutorial utilizzeremo Yarn. Se tu preferisci rimanere con NPM puoi sostituire tutti i comandi `yarn add` e `yarn add --dev` in questo tutorial rispettivamente con `npm install --save` e `npm install --dev`.

- Installa Yarn seguendo le [istruzioni](https://yarnpkg.com/en/docs/install). Puoi utilizzare `npm install -g yarn` o `sudo npm install -g yarn` (si, stiamo sfruttando NPM per installare Yarn, proprio come utilizzeresti Internet Explorer o Safari per installare Chrome!).

- Crea una nuova cartella di base ed entraci `cd` dentro.
- Esegui `yarn init` e rispondi alle domande (`yarn init -y` per saltare tutte le domande), per generare automaticamente un file `package.json`.
- Crea un file `index.js` contenente `console.log('Hello world')`.
- Esegui `node .` in questa cartella (`index.js` è il file che Node cerca automaticamente all'interno della cartella corrente). Dovrebbe scrivere "Hello world".

Eseguire `node .` per lanciare il nostro programma è un po' troppo di basso livello. Utilizzeremo invece uno script NPM/Yarn per avviare l'esecuzione del nostro programma. Questo ci permetterà di avere un livello di astrazione tale da poter sempre utilizare `yarn start`, anche quando il nostro programma si farà più complesso.

- In `package.json`, aggiungi `"start": "node ."` nell'oggetto script.

`package.json` deve essere un file JSON valido, il che significa che non puoi avere delle virgole "finali" che sono invece utilizzabili nei normali oggetti Javascript. Fai quindi molta attenzione quando modifichi a mano il tuo file `package.json`.

- Esegui `yarn start`. Dovrebbe scrivere `Hello world`.

- Crea un file `.gitignore` ed aggiungici le seguenti righe:

```
npm-debug.log
yarn-error.log
```

**Nota**: Se dai un'occhiata a file `package.json` che fornisco, vedrai uno script `tutorial-test` in ogni capitolor. Questi script mi permettono di verificare che i capitoli sono a posto eseguendo `yarn && yarn start`. Puoi cancellarli tranquillamente nei tuoi progetti.

Prossima sezione: [2 - Installare ed usare un pacchetto](/tutorial/2-packages)

Torna all'[indice](https://github.com/fbertone/js-stack-from-scratch).
