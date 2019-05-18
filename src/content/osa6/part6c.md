---
mainImage: ../../images/part-6.svg
part: 6
letter: c
---

<div class="content">

<!-- Laajennetaan sovellusta siten, että muistiinpanot talletetaan backendiin. Käytetään osasta 2 tuttua [json-serveriä](/osa2/palvelimella_olevan_datan_hakeminen). -->

We expand the application, such that the notes are stored to the backend. Let's use [json-server](/osa2/palvelimella_olevan_datan_hakeminen) familiar from part 2.

<!-- Tallennetaan projektin juuren tiedostoon <i>db.json</i> tietokannan alkutila: -->

We store the initial state of the database into the file <i>db.json</i> placed in the root of the project:

```json
{
  "notes": [
    {
      "content": "reduxin storen toiminnan määrittelee reduceri",
      "important": true,
      "id": 1
    },
    {
      "content": "storen tilassa voi olla mielivaltaista dataa",
      "important": false,
      "id": 2
    }
  ]
}
```

<!-- Asennetaan projektiin json-server -->

We install json-server for the project

```js
npm install json-server --save
```

<!-- ja lisätään tiedoston <i>package.json</i> osaan <i>scripts</i> rivi -->

and we add the following line to <i>scripts</i> part of the file <i>package.json</i>

```js
"scripts": {
  "server": "json-server -p3001 db.json",
  // ...
}
```

<!-- Käynnistetään json-server komennolla _npm run server_. -->

We launch json-server with the command _npm run server_.

<!-- Tehdään sitten tuttuun tapaan <i>axiosia</i> hyödyntävä backendistä dataa hakeva metodi tiedostoon <i>services/notes.js</i> -->

Next we create a method into the file <i>services/notes.js</i>, which uses <i>axios</i> to fetch data from the backend

```js
import axios from 'axios'

const baseUrl = 'http://localhost:3001/notes'

const getAll = async () => {
  const response = await axios.get(baseUrl)
  return response.data
}

export default { getAll }
```

<!-- Asennetaan myös axios projektiin -->

We add axios to the project

```js
npm install axios --save
```

<!-- Muutetaan <i>nodeReducer</i>:issa tapahtuva muistiinpanojen tilan alustusta, siten että oletusarvoisesti mustiinpanoja ei ole: -->

We change the initialization of the state in <i>nodeReducer</i>, such that by default there are no notes:

```js
const noteReducer = (state = [], action) => {
  // ...
};
```

<!-- Nopea tapa saada storen tila alustettua palvelimella olevan datan perusteella on hakea muistiinpanot tiedostossa <i>index.js</i> ja dispatchata niille yksitellen action <i>NEW\_NOTE</i>: -->

A quick way to initialize the state based on the data on the server is to fetch the notes in the file <i>index.js</i> and dispatch the action <i>NEW\_NOTE</i> for each of them:

```js
// ...
import noteService from './services/notes' // highlight-line

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
});

const store = createStore(reducer);

// highlight-start
noteService.getAll().then(notes =>
  notes.forEach(note => {
    store.dispatch({ type: 'NEW_NOTE', data: note })
  })
)
// highlight-end

// ...
```

<!-- Lisätään reduceriin tuki actionille <i>INIT\_NOTES</i>, jonka avulla alustus voidaan tehdä dispatchaamalla yksittäinen action. Luodaan myös sitä varten oma action creator -funktio _initializeNotes_: -->

Let's add support in the reducer for the action <i>INIT\_NOTES</i>, using which the initialization can be done by dispatching a single action. Let's also create an action creator function _initializeNotes_.

```js
// ...
const noteReducer = (state = [], action) => {
  console.log('ACTION:', action)
  switch (action.type) {
    case 'NEW_NOTE':
      return [...state, action.data]
    case 'INIT_NOTES':   // highlight-line
      return action.data // highlight-line
    // ...
  }
};

export const initializeNotes = (notes) => {
  return {
    type: 'INIT_NOTES',
    data: notes,
  };
};

// ...
```

<!-- <i>index.js</i> yksinkertaistuu: -->
<i>index.js</i> simplifies:

```js
import noteReducer, { initializeNotes } from './reducers/noteReducer'
// ...

noteService.getAll().then(notes =>
  store.dispatch(initializeNotes(notes))
)
```

<!-- > **HUOM:** miksi emme käyttäneet koodissa promisejen ja _then_-metodilla rekisteröidyn tapahtumankäsittelijän sijaan awaitia? -->
> **NB:** why didn't we in place of promises and event handlers registered to _then_-methods use await?
>
<!-- > await toimii ainoastaan <i>async</i>-funktioiden sisällä, ja <i>index.js</i>:ssä oleva koodi ei ole funktiossa, joten päädyimme tilanteen yksinkertaisuuden takia tällä kertaa jättämään <i>async</i>:in käyttämättä. -->
> await only works inside <i>async</i> functions, and the code in <i>index.js</i> is not inside a function, so due to the simple nature of operation we abstain from using <i>async</i> this time.

<!-- Päätetään kuitenkin siirtää muistiinpanojen alustus <i>App</i>-komponentiin, eli kuten yleensä dataa palvelimelta haettaessa, käytetään <i>effect hookia</i>. -->

We do, however, decide to move the initialization of the notes into the <i>App</i> component, and as usual when fetching data from a server, we use the <i>effect hook</i>. 

<!-- Jotta saamme action creatorin <i>initializeNotes</i> käyttöön komponentissa <i>App</i> tarvitsemme jälleen _connect_-metodin apua: -->

To get the action creator <i>initializeNotes</i> into use in the component <i>App</i> we, once again need the the help of the _connect_-method:

```js
import React, { useEffect } from 'react' // highlight-line
import { connect } from 'react-redux'  // highlight-line
import NewNote from './components/NewNote'
import Notes from './components/Notes'
import VisibilityFilter from './components/VisibilityFilter'
import noteService from './services/notes'
import { initializeNotes } from './reducers/noteReducer'

const App = (props) => {
// highlight-start
  useEffect(() => {
    noteService
      .getAll().then(notes => props.initializeNotes(notes))
  },[])
// highlight-end

  return (
    <div>
      <NewNote />
      <VisibilityFilter />
      <Notes />
    </div>
  )
}

export default connect(null, { initializeNotes })(App) // highlight-line
```

<!-- Näin funktio <i>initializeNotes</i> tulee komponentin <i>App</i> propsiksi <i>props.initializeNotes</i> ja sen kutsumiseen ei tarvita _dispatch_-metodia koska _connect_ hoitaa asian puolestamme. -->

This way the function <i>initializeNotes</i> becomes a prop, in the form of <i>props.initializeNotes</i>, for the component <i>App</i>, and when calling it we don't need the _dispatch_-method because _connect_ handles this for us.

<!-- Voimme toimia samoin myös uuden muistiinpanon luomisen suhteen. Laajennetaan palvelimen kanssa kommunikoivaa koodia: -->

We can do the same thing when it comes to creating a new note. We expand the code communicating with the server as follows:

```js
const url = 'http://localhost:3001/notes'

const getAll = async () => {
  const response = await axios.get(url)
  return response.data
}

// highlight-start
const createNew = async content => {
  const object = { content, important: false }
  const response = await axios.post(url, object)
  return response.data
}
// highlight-end

export default {
  getAll,
  createNew,
}
```

<!-- Komponentin <i>NoteForm</i> metodi _addNote_ muuttuu hiukan: -->

The method _addNote_ of the component <i>NoteForm</i> changes slightly:

```js
import React from 'react'
import { connect } from 'react-redux'
import { createNote } from '../reducers/noteReducer'
import noteService from '../services/notes'  // highlight-line

const NewNote = (props) => {
  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value // highlight-line
    const newNote = await noteService.createNew(content) // highlight-line
    props.createNote(newNote.content) // highlight-line
    event.target.note.value = ''
  }

  return (
    // ...
  )
}

export default connect(null, { createNote } )(NewNote)
```

<!-- Koska backend generoi muistiinpanoille id:t, muutetaan action creator _noteCreation_ muotoon -->

Because the backend generates id:s for the notes we change the action creator _noteCreation_

```js
export const createNote = (data) => {
  return {
    type: 'NEW_NOTE',
    data,
  }
}
```

<!-- Muistiinpanojen tärkeyden muuttaminen olisi mahdollista toteuttaa samalla periaatteella, eli tehdä palvelimelle ensin asynkroninen metodikutsu ja sen jälkeen dispatchata sopiva action. -->

Changing the importance of notes could be implemented using the same principle, meaning making an asynchronous method call to the server and then dispatching an appropriate action.

<!-- Sovelluksen tämänhetkinen koodi on [githubissa](https://github.com/fullstack-hy2019/redux-notes/tree/part6-5) branchissa <i>part6-5</i>. -->

The current state of the code for the application can be found on [github](https://github.com/fullstack-hy2019/redux-notes/tree/part6-5) in the branch <i>part6-5</i>.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- #### 6.16 anekdootit ja backend, step1 -->
#### 6.16 Anecdotes and the backend, step1

<!-- Hae sovelluksen käynnistyessä anekdootit json-serverillä toteutetusta backendistä. -->
When the application launches fetch the anecdotes from the backend implemented using json-server.

<!-- Backendin alustavan sisällön saat esim. [täältä](https://github.com/fullstack-hy2019/misc/blob/master/anecdotes.json). -->

The template for the backend can be found, e.g. [here](https://github.com/fullstack-hy2019/misc/blob/master/anecdotes.json).

<!-- #### 6.17 anekdootit ja backend, step2 -->
#### 6.17 Anecdotes and the backend, step2

<!-- Muuta uusien anekdoottien luomista siten, että anekdootit talletetaan backendiin. -->

Modify the creation of new anecdotes, such that the anecdotes are stored in the backend.

</div>

<div class="content">

<!-- ### Asynkroniset actionit ja redux thunk -->
### Asynchronous actions and redux thunk

<!-- Lähestymistapamme on ok, mutta siinä mielessä ikävä, että palvelimen kanssa kommunikointi tapahtuu komponenttien funktioissa. Olisi parempi, jos kommunikointi voitaisiin abstrahoida komponenteilta siten, että niiden ei tarvitsisi kuin kutsua sopivaa <i>action creatoria</i>, esim. <i>App</i> alustaisi sovelluksen tilan seuraavasti: -->

Our approach is ok, but it is not that great that the communication with the server happens inside the functions of the components. It would be better if the communication could be abstracted away from the components, such that they wouldn't have to do anything else but call the appropriate <i>action creator</i>. E.g. <i>App</i> would initialize the state of the application as follows:

```js
const App = (props) => {

  useEffect(() => {
    props.initializeNotes(notes)
  },[])
  // ...
}
```

<!-- ja <i>NoteForm</i> loisi uuden muistiinpanon seuraavasti: -->

and <i>NoteForm</i> would create a new note as follows:

```js
const NewNote = (props) => {
  const addNote = (event) => {
    event.preventDefault()
    const content = event.target.note.value
    props.createNote(content)
    event.target.note.value = ''
  }
```

<!-- Molemmat komponentit käyttäisivät ainoastaan propsina saamaansa funktiota, välittämättä siitä että taustalla tapahtuu todellisuudessa palvelimen kanssa tapahtuvaa kommunikointia. -->

Both components would only use the function provided to them as a prop without caring about the communication with the server that is happening in the background.

<!-- Asennetaan nyt [redux-thunk](https://github.com/gaearon/redux-thunk)-kirjasto, joka mahdollistaa <i>asynkronisten actionien</i> luomisen. Asennus tapahtuu komennolla: -->

Now we install the [redux-thunk](https://github.com/gaearon/redux-thunk)-library, which enables us to create <i>asynchronous actions</i>. Installation is done with the command:

```js
npm install --save redux-thunk
```

<!-- redux-thunk-kirjasto on ns. <i>redux-middleware</i> joka täytyy ottaa käyttöön storen alustuksen yhteydessä. Eriytetään samalla storen määrittely omaan tiedostoon <i>src/store.js</i>: -->

The redux-thunk-library is a so-called <i>redux-middleware</i>, which must be initialized along with the initialization of the store. At the same time we will extract the definition of the store into its own file <i>src/store.js</i>:

```js
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk';

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
})

const store = createStore(reducer, applyMiddleware(thunk))

export default store
```

<!-- Tiedosto <i>src/index.js</i> on muutoksen jälkeen seuraava -->

After the changes the file <i>src/index.js</i> looks like this

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { Provider } from 'react-redux'
import App from './App'
import store from './store'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
document.getElementById('root'))
```

<!-- redux-thunkin ansiosta on mahdollista määritellä <i>action creatoreja</i> siten, että ne palauttavat funktion, jonka parametrina on redux-storen <i>dispatch</i>-metodi. Tämän ansiosta on mahdollista tehdä asynkronisia action creatoreja, jotka ensin odottavat jonkin toimenpiteen valmistumista ja vasta sen jälkeen dispatchaavat varsinaisen actionin. -->

Thanks to redux-thunk it is possible to define <i>action creators</i>, such that they return a function having the <i>dispatch</i>-method of redux-store as its parameter. As a result of this one can make asynchronous action creators, which first wait for some operation to finish, after which they then dispatch the real action.

<!-- Voimme nyt määritellä muistiinpanojen alkutilan palvelimelta hakevan action creatorin <i>initializeNotes</i> seuraavasti: -->

Now we can define the action creator, <i>initializeNotes</i>, that initializes the state of the notes as follows:

```js
export const initializeNotes = () => {
  return async dispatch => {
    const notes = await noteService.getAll()
    dispatch({
      type: 'INIT_NOTES',
      data: notes,
    })
  }
}
```

<!-- Sisemmässä funktiossaan, eli <i>asynkronisessa actionissa</i> operaatio hakee ensin palvelimelta kaikki muistiinpanot ja sen jälkeen <i>dispatchaa</i> muistiinpanot storeen lisäävän actionin. -->

In the inner function, meaning the <i>asynchronous action</i>, the operation first fetches all the notes from the server and then <i>dispatches</i> the notes to the action, which adds them to the store.

<!-- Komponentti <i>App</i> voidaan nyt määritellä seuraavasti: -->

The component <i>App</i> can now the defined as follows:

```js
const App = (props) => {

  useEffect(() => {
    props.initializeNotes()
  },[])

  return (
    <div>
      <NewNote />
      <VisibilityFilter />
      <Notes />
    </div>
  )
}

export default connect(
  null, { initializeNotes }
)(App)
```

<!-- Ratkaisu on elegantti, muistiinpanojen alustuslogiikka on eriytetty kokonaan React-komponenttien ulkopuolelle. -->

The solution is elegant. The initialization logic for the notes has been completely separated to outside the React component.

<!-- Uuden muistiinpanon lisäävä action creator _createNew_ on seuraavassa -->

The action creator _createNew_, which adds a new note looks like this

```js
export const createNew = content => {
  return async dispatch => {
    const newNote = await noteService.createNew(content)
    dispatch({
      type: 'NEW_NOTE',
      data: newNote,
    })
  }
}
```

<!-- Periaate on jälleen sama, ensin suoritetaan asynkroninen operaatio, ja sen valmistuttua <i>dispatchataan</i> storen tilaa muuttava action. -->

The principle here is the same: first an asynchronous operation is executed, after which the action changing the state of the store is <i>dispatched</i>.

<!-- Komponentti <i>NewNote</i> muuttuu seuraavasti: -->

The component <i>NewNote</i> changes as follows:

```js
const NewNote = (props) => {
  const addNote = async (event) => {
    event.preventDefault()
    const content = event.target.note.value
    event.target.note.value = ''
    rops.createNote(content)
  }

  return (
    <form onSubmit={addNote}>
      <input name="note" />
      <button type="submit">lisää</button>
    </form>
  )
}

export default connect(
  null, { createNote }
)(NewNote)
```

<!-- Sovelluksen tämänhetkinen koodi on [githubissa](https://github.com/fullstack-hy2019/redux-notes/tree/part6-6) brachissa <i>part6-6</i>. -->

The current state of the code for the application can be found on [github](https://github.com/fullstack-hy2019/redux-notes/tree/part6-6) in the branch <i>part6-6</i>.

<!-- ### Redux DevTools -->
### Redux DevTools

<!-- Chromeen on asennettavissa [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd), jonka avulla Redux-storen tilaa ja sitä muuttavia actioneja on mahdollisuus seurata selaimen konsolista. -->

There is an extension [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd) that can be installed on Chrome, using which the state of Redux-store and the action that change it can be monitored from the console of the browser.

<!-- Selaimen lisäosan lisäksi debugatessa tarvitaan kirjastoa [redux-devtools-extension](https://www.npmjs.com/package/redux-devtools-extension). Asennetaan se komennolla -->

When debugging, in addition the the browser extension we also the software library [redux-devtools-extension](https://www.npmjs.com/package/redux-devtools-extension). We insatll it using the command:

```js
npm install --save redux-devtools-extension
```

<!-- Storen luomistapaa täytyy hieman muuttaa, että kirjasto saadaan käyttöön: -->

We have to slightly change the definition of the store to get the library up and running:

```js
// ...
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import { composeWithDevTools } from 'redux-devtools-extension' // highlight-line

import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(
  reducer,
  // highlight-start
  composeWithDevTools(
    applyMiddleware(thunk)
  )
  // highlight-end
)

export default store
```

<!-- Kun nyt avaat konsolin, välilehti <i>redux</i> näyttää seuraavalta: -->

![](../images/6/11.png)

<!-- Konsolin avulla on myös mahdollista dispatchata actioneja storeen -->

It's also possible to dispatch actions to the store using the console

![](../images/6/12.png)

<!-- Storen tietyn hetkisen tilan lisäksi on myös mahdollista tarkastella, mikä on kunkin actionin tilalle aiheuttama muutos: -->

In addition the state of the store at a particular time we can inspect the change to the state made by each action: 

![](../images/6/13.png)

<!-- ### Redux ja komponenttien tila -->
### Redux and component state

<!-- Kurssi on ehtinyt pitkälle, ja olemme vihdoin päässeet siihen pisteeseen missä käytämme Reactia "oikein", eli React keskittyy pelkästään näkymien muodostamiseen ja sovelluksen tila sekä sovelluslogiikka on eristetty kokonaan React-komponenttien ulkopuolelle, Reduxiin ja action reducereihin. -->

We have come far along this course and, finally, we have come to the point at which we are using React "the right way", meaning React only focuses on generating the views and the application state is separated completely from the React components to Redux, its actions and reducers.

<!-- Entä _useState_-hookilla saatava komponenttien oma tila, onko sillä roolia jos sovellus käyttää Reduxia tai muuta komponenttien ulkoista tilanhallintaratkaisua? Jos sovelluksessa on monimutkaisempia lomakkeita, saattaa niiden lokaali tila olla edelleen järkevä toteuttaa funktiolla _useState_ saatavan tilan avulla. Lomakkeidenkin tilan voi toki tallettaa myös reduxiin, mutta jos lomakkeen tila on oleellinen ainoastaan lomakkeen täyttövaiheessa (esim. syötteen muodon validoinnin kannalta), voi olla viisaampi jättää tilan hallinta suoraan lomakkeesta huolehtivan komponentin vastuulle. -->

What about the _useState_-hook, which provides components with their own state? Does it have a role if an application is using Redux or some other external state management solution? If the application has more complicated forms it may be beneficial to implement their local state using the state provided by the _useState_ function. One can, of course, have redux manage the state of the forms, however, if the state of the form is only relevant when filling the form (e.g. for validation) it may be wise to leave the management of state to the component responsible for the form.

</div>

<div class="tasks">

<!-- ### tehtäviä -->
### Exercises

<!-- #### 6.18 anekdootit ja backend, step4 -->
#### 6.18 Anecdotes and the backend, step4

<!-- Muuta redux-storen alustus tapahtumaan <i>redux-thunk</i>-kirjaston avulla toteutettuun asynkroniseen actioniin. -->

Modify the initialization of redux-store to happen using asynchronous action creators made possible by the <i>redux-thunk</i>-library.

<!-- #### 6.19 anekdootit ja backend, step5 -->
#### 6.19 Anecdotes and the backend, step5

<!-- Muuta myös uuden anekdootin luominen tapahtumaan <i>redux-thunk</i>-kirjaston avulla toteutettuihin asynkronisiin actioneihin. -->

Also modify the creation of a new anecdote to happen using asynchronous action creators made possible by the <i>redux-thunk</i>-library.

<!-- #### 6.20 anekdootit ja backend, step6 -->
#### 6.20 Anecdotes and the backend, step6

<!-- Äänestäminen ei vielä talleta muutoksia backendiin. Korjaa tilanne <i>redux-thunk</i>-kirjastoa hyödyntäen. -->

Voting does not yet save changes to the backend. Fix the situation with the help of the <i>redux-thunk</i>-library.

<!-- #### 6.21 anekdootit ja backend, step7 -->
#### 6.21 Anecdotes and the backend, step7

<!-- Notifikaatioiden tekeminen on nyt hieman ikävää, sillä se edellyttää kahden actionin tekemistä ja _setTimeout_-funktion käyttöä: -->

The creation of notifications is still a bit tedious, since one has to do two actions and use the _setTimeout_ function:

```js
props.setNotification(`you voted '${anecdote.content}'`)
setTimeout(() => {
  props.clearNotification()
}, 5000)
```

<!-- Tee asynkrooninen action creator, joka mahdollistaa notifikaation antamisen seuraavasti: -->

Make an asynchronous action creator, which enables one to provide the notification as follows:

```js
props.setNotification(`you voted '${anecdote.content}'`, 10)
```

<!-- eli ensimmäisenä parametrina on renderöitävä teksti ja toisena notifikaation näyttöaika sekunneissa. -->

so as the first parameter is the text to be rendered and as the second the amount of time for which to display the notification given in seconds. 

<!-- Ota paranneltu notifikaatiotapa käyttöön sovelluksessasi. -->

Implement the use of this improved notification in your application.

</div>
