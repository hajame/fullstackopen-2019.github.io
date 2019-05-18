---
mainImage: ../../images/part-6.svg
part: 6
letter: b
---

<div class="content">

<!-- Jatketaan muistiinpanosovelluksen yksinkertaistetun [redux-version](/osa5#redux-muistiinpanot) laajentamista. -->
Let's continue our work with the simplified [redux version](/osa5#redux-muistiinpanot) of our notes application.

<!-- Sovelluskehitystä helpottaaksemme laajennetaan reduceria siten, että storelle määritellään alkutila, jossa on pari muistiinpanoa: -->
In order to ease our development, let's change our reducer so that the store gets initialized with a state that contains a couple of notes:

```js
const initialState = [
  {
    content: 'reduxin storen toiminnan määrittelee reduceri',
    important: true,
    id: 1,
  },
  {
    content: 'storen tilassa voi olla mielivaltaista dataa',
    important: false,
    id: 2,
  },
]

const noteReducer = (state = initialState, action) => {
  // ...
}

// ...
export default noteReducer
```

<!-- ### Monimutkaisempi tila storessa -->
### Store with complex state

<!-- Toteutetaan sovellukseen näytettävien muistiinpanojen filtteröinti, jonka avulla näytettäviä muistiinpanoja voidaan rajata. Filtterin toteutus tapahtuu [radiobuttoneiden](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio) avulla: -->
Let's implement filtering for the notes that are displayed to the user. The user interface for the filters will be implemented with [radio buttons](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/radio):

![](../assets/6/1.png)

<!-- Aloitetaan todella suoraviivaisella toteutuksella: -->
Let's start with a very simple and straightforward implementation:

```js
import React from 'react'
import NewNote from './components/NewNote'
import Notes from './components/Notes'

const App = (props) => {
  const store = props.store

//highlight-start
  const filterSelected = (value) => () => {
    console.log(value)
  }
//highlight-end

  return (
    <div>
      <NewNote store={store}/>
      <div>
        <div>
        //highlight-start
          kaikki    <input type="radio" name="filter"
            onChange={filterSelected('ALL')} />
          tärkeät   <input type="radio" name="filter"
            onChange={filterSelected('IMPORTANT')} />
          eitärkeät <input type="radio" name="filter"
            onChange={filterSelected('NONIMPORTANT')} />
          //highlight-end
        </div>
      </div>
      
      <Notes store={store} />
    </div>
  )
}
```

<!-- Koska painikkeiden attribuutin <i>name</i> arvo on kaikilla sama, muodostavat ne <i>nappiryhmän</i>, joista ainoastaan yksi voi olla kerrallaan valittuna. -->
Since the <i>name</i> attribute of all the radio buttons is the same, they form a <i>button group</i> where only one option can be selected.

<!-- Napeille on määritelty muutoksenkäsittelijä, joka tällä hetkellä ainoastaan tulostaa painettua nappia vastaavan merkkijonon konsoliin. -->
The buttons have a change handler that currently only prints the string associated with the clicked button to the console.

<!-- Päätämme toteuttaa filtteröinnin siten, että talletamme muistiinpanojen lisäksi sovelluksen storeen myös <i>filtterin arvon</i>. Eli muutoksen jälkeen storessa olevan tilan tulisi näyttää seuraavalta: -->
We decide to implement the filter functionality by storing <i>the value of the filter</i> in the redux store in addition to the notes themselves. The state of the store should look like this after making these changes:

```js
{
  notes: [
    { content: 'reduxin storen toiminnan määrittelee reduceri', important: true, id: 1},
    { content: 'storen tilassa voi olla mielivaltaista dataa', important: false, id: 2}
  ],
  filter: 'IMPORTANT'
}
```

<!-- Tällä hetkellähän tilassa on ainoastaan muistiinpanot sisältävä taulukko. Uudessa ratkaisussa tilalla on siis kaksi avainta, <i>notes</i> jonka arvona muistiinpanot ovat sekä <i>filter</i>, jonka arvona on merkkijono joka kertoo mitkä muistiinpanoista tulisi näyttää ruudulla. -->
Only the array of notes is stored in the state of the current implementation of our application. In the new implementation the state object has two properties, <i>notes</i> that contains the array of notes and <i>filter</i> that contains a string that indicates which notes should be displayed to the user.

<!-- ### Yhdistetyt reducerit -->
### Combined reducers

<!-- Voisimme periaatteessa muokata jo olemassaolevaa reduceria ottamaan huomioon muuttuneen tilanteen. Parempi ratkaisu on kuitenkin määritellä tässä tilanteessa uusi, filtterin arvosta huolehtiva reduceri: -->
We could modify our current reducer to deal with the new shape of the state. However, a better solution in this situation is to define a new separate reducer for the state of the filter:

```js
const filterReducer = (state = 'ALL', action) => {
  switch (action.type) {
    case 'SET_FILTER':
      return action.filter
    default:
      return state
  }
}
```

<!-- Filtterin arvon asettavat actionit ovat siis muotoa -->
The actions for changing the state of the filter look like this:

```js
{
  type: 'SET_FILTER',
  filter: 'IMPORTANT'
}
```

<!-- Määritellään samalla myös sopiva _action creator_ -funktio. Sijoitetaan koodi moduuliin <i>src/reducers/filterReducer.js</i>: -->
Let's also create a new _action creator_ function. We will write the code for the action creator in a new <i>src/reducers/filterReducer.js</i> module:

```js
const filterReducer = (state = 'ALL', action) => {
  // ...
}

export const filterChange = filter => {
  return {
    type: 'SET_FILTER',
    filter,
  }
}

export default filterReducer
```

<!-- Saamme nyt muodostettua varsinaisen reducerin yhdistämällä kaksi olemassaolevaa reduceria funktion [combineReducers](https://redux.js.org/api-reference/combinereducers) avulla. -->
We can create the actual reducer for our application by combining the two existing reducers with the [combineReducers](https://redux.js.org/api-reference/combinereducers) function.

<!-- Määritellään yhdistetty reduceri tiedostossa <i>index.js</i>: -->
Let's define the combined reducer in the <i>index.js</i> file:

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, combineReducers } from 'redux' // highlight-line
import App from './App'
import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'  // highlight-line

 // highlight-start
const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})
 // highlight-end

const store = createStore(reducer)  // highlight-line

console.log(store.getState())

ReactDOM.render(
  <div></div>,  // highlight-line
  document.getElementById('root')
)
```

<!-- Koska sovelluksemme hajoaa tässä vaiheessa täysin, komponentin <i>App</i> sijasta renderöidään tyhjä <i>div</i>-elementti. -->
Since our application breaks completely at this point, we render an empty <i>div</i> element instead of the <i>App</i> component.

<!-- Konsoliin tulostuu storen tila: -->
The state of the store gets printed to the console:

![](../images/6/4.png)

<!-- eli store on juuri siinä muodossa missä haluammekin sen olevan! -->
As we can see from the output, the store has the exact shape we wanted it to!

<!-- Tarkastellaan vielä yhdistetyn reducerin luomista -->
Let's take a closer look at how the combined reducer is created:

```js
const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer,
})
```

<!-- Näin tehdyn reducerin määrittelemän storen tila on olio, jossa on kaksi kenttää, <i>notes</i> ja <i>filter</i>. Tilan kentän <i>notes</i> arvon määrittelee <i>noteReducer</i>, jonka ei tarvitse välittää mitään tilan muista kentistä. Vastaavasti <i>filter</i> kentän käsittely tapahtuu <i>filterReducer</i>:in avulla. -->
The state of the store defined by the reducer above is an object with two properties <i>notes</i> and <i>filter</i>. The value of the <i>notes</i> property is defined by the <i>noteReducer</i>, that does not have to deal with the other properties of the state. Likewise, the <i>filter</i> property is managed by the <i>filterReducer</i>.

<!-- Ennen muun koodin muutoksia, kokeillaan vielä konsolista, miten actionit muuttavat yhdistetyn reducerin muodostamaa staten tilaa. Lisätään seuraavat tiedostoon <i>index.js</i>: -->
Before we make more changes to the code, let's take a look at how different actions change the state of the store defined by the combined reducer. Let's add the following to the <i>index.js</i> file:

```js
import { createNote } from './reducers/noteReducer'
import { filterChange } from './reducers/filterReducer'
//...
store.subscribe(() => console.log(store.getState()))
store.dispatch(filterChange('IMPORTANT'))
store.dispatch(createNote('combineReducers muodostaa yhdistetyn reducerin'))
```

<!-- Kun simuloimme näin filtterin tilan muutosta ja muistiinpanon luomista Konsoliin tulostuu storen tila jokaisen muutoksen jälkeen: -->
By simulating the creation of a note and changing the state of the filter in this fashion, the state of the store gets logged to the console after every change that is made to the store:

![](../images/6/5.png)

<!-- Jo tässä vaiheessa kannattaa laittaa mieleen eräs tärkeä detalji. Jos lisäämme <i>molempien reducerien alkuun</i> konsoliin tulostuksen: -->
At this point it is good to become aware of a tiny but important detail. If we add a console log statement <i>to the beginning of both reducers</i>:

```js
const filterReducer = (state = 'ALL', action) => {
  console.log('ACTION: ', action)
  // ...
};
```

<!-- Näyttää konsolin perusteella siltä, että jokainen action kahdentuu: -->
Based on the console output one might get the impression that every action gets duplicated:

![](../images/6/6.png)

<!-- Onko koodissa bugi? Ei. Yhdistetty reducer toimii siten, että jokainen <i>action</i> käsitellään <i>kaikissa</i> yhdistetyn reducerin osissa. Usein tietystä actionista on kiinnostunut vain yksi reduceri, on kuitenkin tilanteita, joissa useampi reduceri muuttaa hallitsemaansa staten tilaa jonkin actionin seurauksena. -->
Is there a bug in our code? No. The combined reducer works in such a way that every <i>action</i> gets handled in <i>every</i> part of the combined reducer. Typically only one reducer is interested in any given action, but there are situations where multiple reducers change their own piece state based on the same action.

<!-- ### Filtteröinnin viimeistely -->
### Finishing the filters

<!-- Viimeistellään nyt sovellus käyttämään yhdistettyä reduceria, eli palautetaan tiedostossa <i>index.js</i> suoritettava renderöinti muotoon -->
Let's finish the application so that it uses the combined reducer. We start by changing the rendering of the application and hooking up the store to the application in the <i>index.js</i> file:

```js
ReactDOM.render(
  <App store={store} />,
  document.getElementById('root')
)
```

<!-- Korjataan sitten bugi, joka johtuu siitä, että koodi olettaa storen tilan olevan mustiinpanot tallettava taulukko: -->
Next, let's fix a bug that is caused by the code expecting the application store to be an array of notes:

![](../images/6/7.png)

<!-- Korjaus on helppo. Viitteen <i>store.getState()</i> sijaan kaikki muistiinpanot sisältävään taulukkoon viitataan <i>store.getState().notes</i>: -->
There is an easy fix for this. We simply have to change the reference to the array of notes from <i>store.getState()</i> to <i>store.getState()</i>.notes:

```js
const Notes = ({ store }) => {
  return(
    <ul>
      {store.getState().notes.map(note => // highlight-line
        <Note
          key={note.id}
          note={note}
          handleClick={() => store.dispatch(toggleImportanceOf(note.id))}
        />
      )}
    </ul>
  )
}
```

<!-- Eriytetään näkyvyyden säätelyfiltteri omaksi, tiedostoon sijoitettavaksi <i>src/components/VisibilityFilter.js</i> komponentiksi: -->
Let's extract the visibility filter into its own <i>src/components/VisibilityFilter.js</i> component:

```js
import React from 'react'
import { filterChange } from '../reducers/filterReducer'

const VisibilityFilter = (props) => {

  const filterClicked = (value) => {
    props.store.dispatch(filterChange(value))
  }

  return (
    <div>
      kaikki    
      <input 
        type="radio" 
        name="filter" 
        onChange={() => filterClicked('ALL')}
      />
      tärkeät   
      <input
        type="radio"
        name="filter"
        onChange={() => filterClicked('IMPORTANT')}
      />
      eitärkeät 
      <input
        type="radio"
        name="filter"
        onChange={() => filterClicked('NONIMPORTANT')}
      />
    </div>
  )
}

export default VisibilityFilter
```

<!-- Toteutus on suoraviivainen, radiobuttonin klikkaaminen muuttaa storen kentän <i>filter</i> tilaa. -->
The implementation is rather straightforward. Clicking the different radio buttons changes the state of the store's <i>filter</i> property.

<!-- Muutetaan vielä komponentin <i>Notes</i> ottamaan huomioon filtteri -->
Let's change the <i>Notes</i> component to incorporate the filter:

```js
const Notes = ({ store }) => {
  // highlight-start
  const { notes, filter } = store.getState()
  const notesToShow = () => {
    if ( filter === 'ALL' ) {
      return notes
    }

    return filter === 'IMPORTANT'
      ? notes.filter(note => note.important)
      : notes.filter(note => !note.important)
  }
  // highlight-end

  return(
    <ul>
      {notesToShow().map(note => // highlight-line
        <Note
          key={note.id}
          note={note}
          handleClick={() => store.dispatch(toggleImportanceOf(note.id))}
        />
      )}
    </ul>
  )
}
```

<!-- Huomaa miten storen tilan kentät on otettu tuttuun tapaan destrukturoimalla apumuuttujiin -->
Notice how the properties of the store are assigned to helper variables with the destructuring syntax:

```js
const { notes, filter } = store.getState()
```

<!-- siis on sama kuin kirjoittaisimme -->
The syntax above is the same as if we were to write:

```js
const notes = store.getState().notes
const filter = store.getState().filter
```

<!-- Sovelluksen tämänhetkinen koodi on [githubissa](https://github.com/fullstack-hy2019/redux-notes/tree/part6-2) branchissa </i>part6-2</i>. -->
You can find the code for our current application in its entirety in the <i>part6-2</i> branch of [this github repository](https://github.com/fullstack-hy2019/redux-notes/tree/part6-2).

<!-- Sovelluksessa on vielä pieni kauneusvirhe, vaikka oletusarvosesti filtterin arvo on <i>ALL</i>, eli näytetään kaikki muistiinpanot, ei vastaava radiobutton ole valittuna. Ongelma on luonnollisestikin mahdollista korjata, mutta koska kyseessä on ikävä, mutta harmiton feature, jätämme korjauksen myöhemmäksi. -->
There is a slight cosmetic flaw in our application. Even though the filter is set to <i>ALL</i> by default, the associated radio button is not selected. Naturally this issue can be fixed, but since this is an unpleasant but ultimately harmless bug we will save the fix for later. 

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Jatketaan tehtävässä 6.3 aloitetun reduxia käyttävän anekdoottisovelluksen parissa. -->
Let's continue working on the anecdote application using redux that we started in exercise 6.3. 

<!-- #### 6.9 anekdootit, step7 -->
#### 6.9 Better anecdotes, step7

<!-- Sovelluksessa on valmiina komponentin <i>Notification</i> runko: -->
The application has a ready-made body for the <i>Notification</i> component:

```js
import React from 'react';

const Notification = () => {
  const style = {
    border: 'solid',
    padding: 10,
    borderWidth: 1
  }
  return (
    <div style={style}>
      render here notification...
    </div>
  )
}

export default Notification
```

<!-- Laajenna komponenttia siten, että se renderöi redux-storeen talletetun viestin, eli renderöitävä komponentti muuttuu muodoon: -->
Extend the component so that it renders the message stored in the redux store, so that the component gets changed to the form:

```js
return (
  <div style={style}>
    {props.store.getState()...}
  </div>
)
```

<!-- Joudut siis muuttamaan/laajentamaan sovelluksen olemassaolevaa reduceria. Tee toiminnallisuutta varten oma reduceri ja siirry käyttämään sovelluksessa yhdistettyä reduceria tämän osan materiaalin tapaan. -->
You will have to make changes to the application's existing reducer. Create a separate reducer for the new functionality and refactor the application so that it uses a combined reducer as shown in this part of the course material.

<!-- Tässä vaiheessa sovelluksen ei vielä tarvitse osata käyttää <i>Notification</i> komponenttia järkevällä tavalla, riittää että sovellus toimii ja näyttää <i>notificationReducerin</i> alkuarvoksi asettaman viestin. -->
The application does not have to use the <i>Notification</i> component in any intelligent way at this point of the exercise. It is enough for the application display the initial value set for the message in the <i>notificationReducer</i>.

<!-- #### 6.10 paremmat anekdootit, step8 -->
#### 6.10 Better anecdotes, step8

<!-- Laajenna sovellusta siten, että se näyttää <i>Notification</i>-komponentin avulla viiden sekunnin ajan kun sovelluksessa äänestetään tai luodaan uusia anekdootteja: -->
Extend the application so that it uses the <i>Notification</i> component to display a message for the duration of five seconds when the user votes for an anecdote or creates a new anecdote:

![](../images/6/8.png)

<!-- Notifikaation asettamista ja poistamista varten kannattaa toteuttaa [action creatorit](https://redux.js.org/basics/actions#action-creators). -->
It's recommended to create separate [action creatorit](https://redux.js.org/basics/actions#action-creators) for setting and removing notifications.

<!-- #### 6.11* paremmat anekdootit, step9 -->
#### 6.11* Better anecdotes, step9

<!-- Toteuta sovellukseen näytettävien muistiinpanojen filtteröiminen -->
Implement filtering for the anecdotes that are displayed to the user.

![](../images/6/9.png)

<!-- Säilytä filtterin tila redux storessa, eli käytännössä kannattaa jälleen luoda uusi reduceri ja action creatorit. -->
Store the state of the filter in the redux store. It is recommended to create a new reducer and action creators for this purpose.

<!-- Tee filtterin ruudulla näyttämistä varten komponentti <i>Filter</i>. Voit ottaa sen pohjaksi seuraavan -->
Create a new <i>Filter</i> component for displaying the filter. You can use the following code as a template for the component:

```js
import React from 'react'

const Filter = (props) => {
  const handleChange = (event) => {
    // input-kentän arvo muuttujassa event.target.value
  }
  const style = {
    marginBottom: 10
  }

  return (
    <div style={style}>
      filter <input onChange={handleChange} />
    </div>
  )
}

export default Filter
```

</div>

<div class="content">

### Connect

<!-- Reduxin käytön ansiosta sovelluksen rakenne alkaa jo olla mukavan modulaarinen. Pystymme kuitenkin vielä parempaan. -->
The structure of our current application is quite modular thanks to Redux. Naturally there's still room for improvement.

<!-- Eräs tämän hetkisen ratkaisun ikävistä puolista on se, että Redux-store täytyy välittää propseina kaikille sitä tarvitseville komponenteille. <i>App</i> ei itse tarvitse ollenkaan Reduxia, mutta joutuu silti välittämään sen eteenäin lapsikomponenteille:  -->
One unpleasant aspect of our current implementation is that the Redux store has to be passed via props to all of the components that use it. The <i>App</i> component does not actually need Redux for any other purpose than passing it to its children:

```js
const App = (props) => {
  const store = props.store

  return (
    <div>
      <NewNote store={store}/>  
      <VisibilityFilter store={store} />    
      <Notes store={store} />
    </div>
  )
}
```

<!-- Otetaan nyt käyttöön [React Redux](https://github.com/reactjs/react-redux) -kirjaston määrittelemä funktio [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options), joka on tämän hetken defacto-ratkaisu sille, miten Redux-store saadaan välitettyä React-componenteille. -->
To get rid of this unpleasantness we will use the [connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) function provided by the [React Redux](https://github.com/reactjs/react-redux) library. This is currently the de facto solution for passing the Redux store to React components.

<!-- Connect voi olla aluksi haastava sisäistää, mutta hieman vaivaa kannattaa ehdottomasti nähdä. Tutustutaan nyt connectin käyttöön.  -->
It takes some time to wrap your head around how connect works, but your efforts will be rewarded. Next, let's take a look at how connect is used in practice.

```js
npm install --save react-redux
```

<!-- Edellytyksenä kirjaston tarjoaman _connect_-funktion käytölle on se, että sovellus on määritelty React redux kirjaston tarjoaman [Provider](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store)-komponentin lapsena ja että sovelluksen käyttämä store on annettu Provider-komponentin attribuutiksi <i>store</i>.  -->
In order to use the _connect_ function we have to define our application as the child of the [Provider](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) component that is provided by the React Redux library. Additionally, the <i>Provider</i> component must receive the Redux store of the application as its <i>store</i> attribute.

<!-- Eli tiedosto <i>index.js</i> tulee muuttaa seuraavaan muotoon -->
Let's make these changes to the <i>index.js</i> file of our application:

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, combineReducers } from 'redux'
import { Provider } from 'react-redux' // highlight-line
import App from './App'
import noteReducer from './reducers/noteReducer'
import filterReducer from './reducers/filterReducer'

const reducer = combineReducers({
  notes: noteReducer,
  filter: filterReducer
})

const store = createStore(reducer)

ReactDOM.render(
  // highlight-start
  <Provider store={store}>
    <App />
  </Provider>,
  // highlight-end
document.getElementById('root'))
```

<!-- Tutkitaan ensin komponenttia <i>Notes</i>. Funktiota _connect_ käyttämällä "normaaleista" React-komponenteista saadaan muodostettua komponentteja, joiden <i>propseihin</i> on "mäpätty" eli yhdistetty haluttuja osia storen määrittelemästä tilasta. -->
Let's start by taking a closer look at the <i>Notes</i> component. The _connect_ function can be used for transforming "regular" React components so that the state of the Redux store can be "mapped" into the component's props.

<!-- Muodostetaan ensin komponentista <i>Notes</i> connectin avulla <i>yhdistetty komponentti</i>: -->
Let's first use the connect function to transform our <i>Notes</i> component into a <i>connected component</i>:

```js
import React from 'react'
import { connect } from 'react-redux' // highlight-line
import Note from './Note'
import { toggleImportanceOf } from '../reducers/noteReducer'

const Notes = ({ store }) => {
  // ...
}

const ConnectedNotes = connect()(Notes) // highlight-line
export default ConnectedNotes           // highlight-line
```

<!-- Moduuli eksporttaa nyt alkuperäisen komponentin sijaan <i>yhdistetyn komponentin</i>, joka toimii toistaiseksi täsmälleen alkuperäisen komponentin kaltaisesti. -->
The module exports the <i>connected component</i> that works exactly like the previous regular component for now.

<!-- Komponentti tarvitsee storesta sekä muistiinpanojen listan, että filtterin arvon. Funktion _connect_ ensimmäisenä parametrina voidaan määritellä funktio [mapStateToProps](https://github.com/reactjs/react-redux/blob/master/docs/api.md#arguments), joka liittää joitakin storen tilan perusteella määriteltyjä asioita connectilla muodostetun <i>yhdistetyn komponentin</i> propseiksi. -->
The component needs the list of notes and the value of the filter from the Redux store. The _connect_ function accepts a so-called [mapStateToProps](https://github.com/reactjs/react-redux/blob/master/docs/api.md#arguments) function as its first parameter. The function can be used for defining the props of the <i>connected component</i> that are based on the state of the Redux store.

<!-- Jos määritellään: -->
If we define:

```js
const Notes = (props) => {
  // ...
}

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  }
}

const ConnectedNotes = connect(mapStateToProps)(Notes)

export default ConnectedNotes
```

<!-- on komponentin <i>Notes</i> sisällä mahdollista viitata storen tilaan, esim. muistiinpanoihin suoraan propsin kautta <i>props.notes</i> sen sijaan, että käytettäisiin suoraan propseina saatua storea muodossa <i>props.store.getState().notes</i>. Vastaavasti <i>props.filter</i> viittaa storessa olevaan filter-kentän tilaan. -->
The <i>Notes</i> component can access the state of the store directly, e.g. through <i>props.notes</i> that contains the list of notes. Contrast this to the previous <i>props.store.getState().notes</i> implementation that accessed the notes directly from the store. Similarly, <i>props.filter</i> references the value of the filter.

<!-- Komponentti muuttuu seuraavasti -->
The component changes in the following way:

```js
const Notes = (props) => {  // highlight-line
  const notesToShow = () => {
    if ( props.filter === 'ALL' ) { // highlight-line
      return props.notes // highlight-line
    }

    return props.filter === 'IMPORTANT' // highlight-line
      ? props.notes.filter(note => note.important) // highlight-line
      : props.notes.filter(note => !note.important) // highlight-line
  }

  return(
    <ul>
      {notesToShow().map(note =>
        <Note
          key={note.id}
          note={note}
          handleClick={() =>
            props.store.dispatch(toggleImportanceOf(note.id))
          }
        />
      )}
    </ul>
  )
}
```

<!-- Connect-komennolla ja <i>mapStateToProps</i>-määrittelyllä aikaan saatua tilannetta voidaan visualisoida seuraavasti: -->
The situation that results from using <i>connect</i> with the <i>mapStateToProps</i> function we defined can be visualized like this:

![](../images/6/24c.png)

<!-- eli komponentin <i>Notes</i> sisältä on propsien <i>props.notes</i> ja <i>props.filter</i> kautta "suora pääsy" tarkastelemaan Redux storen sisällä olevaa tilaa. -->
The <i>Notes</i> component has "direct access" via <i>props.notes</i> and <i>props.filter</i> for inspecting the state of the Redux store.

<!-- <i>Notes</i> viittaa edelleen propsien avulla saamaansa funktioon _dispatch_, jota se käyttää muuttamaan Reduxin tilaa: -->
The <i>Notes</i> component still uses the _dispatch_ function that it receives through its props to modify the state of the Redux store:

```js
<Note
  key={note.id}
  note={note}
  handleClick={() =>
    props.store.dispatch(toggleImportanceOf(note.id)) // highlight-line
  }
/>
```


<!-- Propsia <i>store</i> ei kuitenkaan ole enää olemassa, joten tilan muutos ei tällä hetkellä toimi. -->
The <i>store</i> prop no longer exists, so altering the state through the function is currently broken.

<!-- Connect-funktion toisena parametrina voidaan määritellä [mapDispatchToProps](https://github.com/reactjs/react-redux/blob/master/docs/api.md#arguments) eli joukko <i>action creator</i> -funktioita, jotka välitetään yhdistetylle komponentille propseina. Laajennetaan connectausta seuraavasti -->
The second parameter of the _connect_ function can be used for defining [mapDispatchToProps](https://github.com/reactjs/react-redux/blob/master/docs/api.md#arguments) which is a group of <i>action creator</i> functions that are passed to the connected component as props. Let's make the following changes to our existing connect operation:

```js
const mapStateToProps = state => {
  return {
    notes: state.notes,
    filter: state.filter,
  }
}


const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  }
}

// highlight-start
const mapDispatchToProps = {
  toggleImportanceOf,
}
// highlight-end

const ConnectedNotes = connect(
  mapStateToProps,
  mapDispatchToProps // highlight-line
)(Notes)
```

<!-- Nyt komponentti voi dispatchata suoraan action creatorin _toggleImportanceOf_ määrittelemän actionin kutsumalla propsien kautta saamaansa funktiota koodissa: -->
Now the component can directly dispatch the action defined by _toggleImportanceOf_ action creator by calling the function through its props:

```js
<Note
  key={note.id}
  note={note}
  handleClick={() => props.toggleImportanceOf(note.id)} // highlight-line
/>
```

<!-- Eli se sijaan että kutsuttaisiin kutsuttaisiin -->
This means that instead of dispatching the action like this:

```js
props.store.dispatch(toggleImportanceOf(note.id))
```

<!-- _connect_-metodia käytettäessä actionin dispatchaamiseen riittää -->
When using _connect_ we can simply do this:

```js
props.toggleImportanceOf(note.id)
```

<!-- Storen _dispatch_-funktiota ei enää tarvitse kutsua, sillä _connect_ on muokannut action creatorin _toggleImportanceOf_ sellaiseen muotoon, joka sisältää dispatchauksen. -->
There is no need to call the _dispatch_ function separately since _connect_ has already modified the _toggleImportanceOf_ action creator into a form that contains the dispatch.

<!-- _mapDispatchToProps_ lienee aluksi hieman haastava ymmärtää, etenkin sen kohta käsiteltävä [vaihtoehtoinen käyttötapa](/osa6/monta_reduseria_connect#map-dispatch-to-propsin-vaihtoehtoinen-kayttotapa). -->
It can take some to time to wrap your head around how _mapDispatchToProps_ works, especially once we take a look at an [alternative way of using it](/osa6/monta_reduseria_connect#map-dispatch-to-propsin-vaihtoehtoinen-kayttotapa).

<!-- Connectin aikaansaamaa tilannetta voidaan havainnollistaa seuraavasti: -->
The resulting situation from using _connect_ can be visualized like this:

![](../images/6/25b.png)

<!-- eli sen lisäksi että <i>Notes</i> pääsee storen tilaan propsien <i>props.notes</i> ja <i>props.filter</i> kautta, se viittaa <i>props.toggleImportanceOf</i>:lla funktioon, jonka avulla storeen saadaan dispatchattua <i>TOGGLE\_IMPORTANCE</i>-tyyppisiä actioneja. -->
In addition to accessing the store's state via <i>props.notes</i> and <i>props.filter</i>, the component also references a function that can be used for dispatching <i>TOGGLE\_IMPORTANCE</i>-type actions via its <i>toggleImportanceOf</i> prop.

<!-- Connectia käyttämään refaktoroitu komponentti <i>Notes</i> on kokonaisuudessaan seuraava: -->
The code for the newly refactored <i>Notes</i> component looks like this:

```js
import React from 'react'
import { connect } from 'react-redux'
import Note from './Note'
import { toggleImportanceOf } from '../reducers/noteReducer'

const Notes = (props) => {
  const notesToShow = () => {
    if ( props.filter === 'ALL' ) {
      return props.notes
    }

    return props.filter === 'IMPORTANT'
      ? props.notes.filter(note => note.important)
      : props.notes.filter(note => !note.important)
  }

  return(
    <ul>
      {notesToShow().map(note =>
        <Note
          key={note.id}
          note={note}
          handleClick={() => props.toggleImportanceOf(note.id)}
        />
      )}
    </ul>
  )
}

const mapStateToProps = (state) => {
  return {
    notes: state.notes,
    filter: state.filter,
  }
}

const mapDispatchToProps = {
  toggleImportanceOf,
}

// eksportoidaan suoraan connectin palauttama komponentti
export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Notes)
```

<!-- Otetaan _connect_ käyttöön myös uuden muistiinpanon luomisessa: -->
Let's also use _connect_ to create new notes:

```js
import React from 'react'
import { connect } from 'react-redux'
import { createNote } from '../reducers/noteReducer'

const NewNote = (props) => {

  const addNote = (event) => {
    event.preventDefault()
    props.createNote(event.target.note.value)
    event.target.note.value = ''
  }
  return (
    <form onSubmit={addNote}>
      <input name="note" />
      <button type="submit">lisää</button>
    </form>
  )
}

export default connect(
  null,
  { createNote }
)(NewNote)
```

<!-- Koska komponentti ei tarvitse storen tilasta mitään, on funktion _connect_ ensimmäinen parametri <i>null</i>. -->
Since the component does not need to access the store's state, we can simply pass <i>null</i> as the first parameter to _connect_. 

<!-- Sovelluksen tämänhetkinen koodi on [githubissa](https://github.com/fullstack-hy2019/redux-notes/tree/part6-3) branchissa <i>part6-3</i>. -->
You can find the code for our current application in its entirety in the <i>part6-3</i> branch of [this github repository](https://github.com/fullstack-hy2019/redux-notes/tree/part6-3).

### Huomio propsina välitettyyn action creatoriin viittaamisesta

Tarkastellaan vielä erästä mielenkiintoista seikkaa komponentista <i>NewNote</i>:

```js
import React from 'react'
import { connect } from 'react-redux'
import { createNote } from '../reducers/noteReducer' // highlight-line

const NewNote = (props) => {

  const addNote = (event) => {
    event.preventDefault()
    props.createNote(event.target.note.value) // highlight-line
    event.target.note.value = ''
  }
  // ...
}

export default connect(
  null,
  { createNote }
)(NewNote)
```

Aloittelevalle connectin käyttäjälle aiheuttaa joskus ihmetystä se, että action creatorista <i>createNote</i> on komponentin sisällä käytettävissä <i>kaksi eri versiota</i>.

Funktioon tulee viitata propsien kautta, eli <i>props.createNote</i>, tällöin kyseessä on _connectin_ muotoilema, <i>dispatchauksen sisältävä</i> versio funktiosta.

Moduulissa olevan import-lauseen

```js
import { createNote } from './../reducers/noteReducer'
```

ansiosta komponentin sisältä on mahdollista viitata funktioon myös suoraan, eli _noteCreation_. Näin ei kuitenkaan tule tehdä, sillä silloin on kyseessä alkuperäinen action creator joka <i>ei sisällä dispatchausta</i>.

Jos tulostamme funktiot koodin sisällä (emme olekaan vielä käyttäneet kurssilla tätä erittäin hyödyllistä debug-kikkaa)

```js
const NewNote = (props) => {
  console.log(createNote)
  console.log(props.createNote)

  const addNote = (event) => {
    event.preventDefault()
    props.createNote(event.target.note.value)
    event.target.note.value = ''
  }

  // ...
}
```

näemme eron:

![](../images/6/10.png)

Ensimmäinen funktioista siis on normaali <i>action creator</i>, toinen taas connectin muotoilema funktio, joka sisältää storen metodin dispatch-kutsun.

Connect on erittäin kätevä työkalu, mutta abstraktiutensa takia se voi aluksi tuntua hankalalta.

### mapDispatchToPropsin vaihtoehtoinen käyttötapa

Määrittelimme siis connectin komponentille <i>NewNote</i> antamat actioneja dispatchaavan funktion seuraavasti:

```js
const NewNote = () => {
  // ...
}

export default connect(
  null,
  { createNote }
)(NewNote)
```

Eli määrittelyn ansiosta komponentti dispatchaa uuden muistiinpanon lisäyksen suorittavan actionin suoraan komennolla <code>props.createNote('uusi muistiinpano')</code>.

Parametrin <i>mapDispatchToProps</i> kenttinä ei voi antaa mitä tahansa funktiota, vaan funktion on oltava <i>action creator</i>, eli Redux-actionin palauttava funktio.

Kannattaa huomata, että parametri <i>mapDispatchToProps</i> on nyt <i>olio</i>, sillä määrittely

```js
{
  createNote
}
```

on lyhempi tapa määritellä olioliteraali

```js
{
  createNote: createNote
}
```

eli olio, jonka ainoan kentän <i>createNote</i> arvona on funktio <i>createNote</i>.

Voimme määritellä saman myös "pitemmän kaavan" kautta, antamalla _connectin_ toisena parametrina seuraavanlaisen <i>funktion</i>:

```js
const NewNote = (props) => {
  // ...
}

// highlight-start
const mapDispatchToProps = dispatch => {
  return {
    createNote: value => {
      dispatch(createNote(value))
    },
  }
}
// highlight-end

export default connect(
  null,
  mapDispatchToProps
)(NewNote)
```

Tässä vaihtoehtoisessa tavassa <i>mapDispatchToProps</i> on funktio, jota _connect_ kutsuu antaen sille parametriksi storen _dispatch_-funktion. Funktion paluuarvona on olio, joka määrittelee joukon funktioita, jotka annetaan connectoitavalle komponentille propsiksi. Esimerkkimme määrittelee propsin <i>createNote</i> olevan funktion

```js
value => {
  dispatch(createNote(value))
}
```

eli action creatorilla luodun actionin dispatchaus.

Komponentti siis viittaa funktioon propsin <i>props.createTodo</i> kautta:

```js
const NewNote = (props) => {

  addNote = (event) => {
    event.preventDefault()
    props.createTodo(event.target.note.value)
    event.target.note.value = ''
  }

  return (
    <form onSubmit={addNote}>
      <input name="note" />
      <button type="submit">lisää</button>
    </form>
  )
}
```

Konsepti on hiukan monimutkainen ja sen selittäminen sanallisesti on haastavaa. Useimmissa tapauksissa onneksi riittää <i>mapDispatchToProps</i>:in yksinkertaisempi muoto. On kuitenkin tilanteita, joissa monimutkaisempi muoto on tarpeen, esim. jos määriteltäessä propseiksi mäpättyjä <i>dispatchattavia actioneja</i> on [viitattava komponentin omiin propseihin](https://github.com/gaearon/redux-devtools/issues/250#issuecomment-186429931).

Egghead.io:sta löytyy Reduxin kehittäjän Dan Abramovin loistava tuoriaali [Getting started with Redux](https://egghead.io/courses/getting-started-with-redux), jonka katsomista voin suositella kaikille. Neljässä viimeisessä videossa käsitellään _connect_-metodia ja nimenomaan sen "hankalampaa" käyttötapaa.

### Presentational/Container revisited

Komponentti <i>Notes</i> käyttää apumetodia <i>notesToShow</i>, joka päättelee filtterin perusteella näytettävien muistiinpanojen listan:

```js
const Notes = (props) => {
  const notesToShow = () => {
    if ( props.filter === 'ALL' ) {
      return props.notes
    }

    return props.filter === 'IMPORTANT'
      ? props.notes.filter(note => note.important)
      : props.notes.filter(note => !note.important)
  }

  // ...
}
```

Komponentin on tarpeetonta sisältää kaikkea tätä logiikkaa. Eriytetään se komponentin ulkopuolelle _connect_-metodin parametrin <i>mapStateToProps</i> yhteyteen:

```js
import React from 'react'
import { connect } from 'react-redux'
import Note from './Note'
import { toggleImportanceOf } from '../reducers/noteReducer'

const Notes = (props) => {
  return(
    <ul>
      {props.visibleNotes.map(note => // highlight-line
        <Note
          key={note.id}
          note={note}
          handleClick={() => props.toggleImportanceOf(note.id)}
        />
      )}
    </ul>
  )
}

const notesToShow = ({ notes, filter }) => { // highlight-line
  if (filter === 'ALL') {
    return notes
  }
  return filter === 'IMPORTANT'
    ? notes.filter(note => note.important)
    : notes.filter(note => !note.important)
}

const mapStateToProps = (state) => {
  return {
    visibleNotes: notesToShow(state), // highlight-line
  }
}


const mapDispatchToProps = {
  toggleImportanceOf,
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Notes)

```

<i>mapStateToProps</i> ei siis tällä kertaa mäppää propsiksi suoraan storessa olevaa asiaa, vaan storen tilasta funktion _notesToShow_ avulla muodostetun sopivasti filtteröidyn datan. Uusi versio funktiosta _notesToShow_ siis saa parametriksi koko tilan ja <i>valitsee</i> siitä sopivan osajoukon välitettäväksi komponentille. Tämänkaltaisia funktioita kutsutaan [selektoreiksi](https://medium.com/@pearlmcphee/selectors-react-redux-reselect-9ab984688dd4).

Uudistettu <i>Notes</i> keskittyy lähes ainoastaan muistiinpanojen renderöimiseen, se on hyvin lähellä sitä minkä sanotaan olevan [presentational](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)-komponentti, joita Dan Abramovin [sanoin](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) kuvaillaan seuraavasti:

- Are concerned with how things look.
- May contain both presentational and container components inside, and usually have some DOM markup and styles of their own.
- Often allow containment via props.children.
- Have no dependencies on the rest of the app, such Redux actions or stores.
- Don’t specify how the data is loaded or mutated.
- Receive data and callbacks exclusively via props.
- Rarely have their own state (when they do, it’s UI state rather than data).
- Are written as functional components unless they need state, lifecycle hooks, or performance optimizations.

Connect-metodin avulla muodostettu _yhdistetty komponentti_

```js
const notesToShow = ({notes, filter}) => {
  // ...
}

const mapStateToProps = (state) => {
  return {
    visibleNotes: notesToShow(state),
  }
}

const mapDispatchToProps = {
  toggleImportanceOf,
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(Notes)
```

taas on selkeästi <i>container</i>-komponentti, joita Dan Abramov [luonnehtii](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) seuraavasti:

- Are concerned with how things work.
- May contain both presentational and container components inside but usually don’t have any DOM markup of their own except for some wrapping divs, and never have any styles.
- Provide the data and behavior to presentational or other container components.
- Call Redux actions and provide these as callbacks to the presentational components.
- Are often stateful, as they tend to serve as data sources.
- Are usually generated using higher order components such as connect from React Redux, rather than written by hand.

Komponenttien presentational vs. container -jaottelu on eräs hyväksi havaittu tapa strukturoida React-sovelluksia. Jako voi olla toimiva tai sitten ei, kaikki riippuu kontekstista.

Abramov mainitsee jaon [eduiksi](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) muunmuassa seuraavat

- Better separation of concerns. You understand your app and your UI better by writing components this way.
- Better reusability. You can use the same presentational component with completely different state sources, and turn those into separate container components that can be further reused.
- Presentational components are essentially your app’s “palette”. You can put them on a single page and let the designer tweak all their variations without touching the app’s logic. You can run screenshot regression tests on that page.

Abramov mainitsee termin [high order component](https://reactjs.org/docs/higher-order-components.html). Esim. <i>Notes</i> on normaali komponentti, React-reduxin _connect_ metodi taas on <i>high order komponentti</i>, eli käytännössä funktio, joka haluaa parametrikseen komponentin muuttuakseen "normaaliksi" komponentiksi.

High order componentit eli HOC:t ovatkin yleinen tapa määritellä geneeristä toiminnallisuutta, joka sitten erikoistetaan esim. renderöitymisen määrittelyn suhteen parametrina annettavan komponentin avulla. Kyseessä on funktionaalisen ohjelmoinnin etäisesti olio-ohjelmoinnin perintää muistuttava käsite.

HOC:it ovat oikeastaan käsitteen [High Order Function](https://en.wikipedia.org/wiki/Higher-order_function) (HOF) yleistys. HOF:eja ovat sellaiset funkiot, jotka joko ottavat parametrikseen funktioita tai palauttavat funkioita. Olemme oikeastaan käyttäneet HOF:eja läpi kurssin, esim. lähes kaikki taulukoiden käsittelyyn tarkoitetut metodit, kuten _map, filter ja find_ ovat HOF:eja.

Sovelluksen tämänhetkinen koodi on [githubissa](https://github.com/fullstack-hy2019/redux-notes/tree/part6-4) branchissa <i>part6-4</i>.

</div>

<div class="tasks">

### Tehtäviä

#### 6.12 paremmat anekdootit, step10

Sovelluksessa välitetään <i>redux store</i> tällä hetkellä kaikille komponenteille propseina.

Ota käyttöön kirjasto [react-redux](https://github.com/reactjs/react-redux) ja muuta komponenttia <i>AnecdoteList</i> niin, että se pääsee käsiksi tilaan _connect_-funktion välityksellä. 

Anekdoottien äänestyksen ja uusien anekdoottien luomisen **ei tarvitse vielä toimia** tämän tehtävän jälkeen.

Tehtävässäsi tarvitsema <i>mapStateToProps</i> on suunilleen seuraavanlainen

```js
const mapStateToProps = (state) => {
  // joskus on hyödyllistä tulostaa mapStateToProps:ista...
  console.log(state)
  return {
    anecdotes: state.anecdotes,
    filter: state.filter
  }
}
```

#### 6.13 paremmat anekdootit, step11

Tee sama komponentille  <i>Filter</i> ja <i>AnecdoteForm</i>.

#### 6.14 paremmat anekdootit, step12

Muuta komponenttia <i>AnecdoteList</i> siten, että anekdoottien äänestys taas onnistuu ja muuta myös <i>Notification</i> käyttämään connectia.

Poista turhaksi staten propseina tapahtuva välittäminen, eli pelkistä <i>App</i> muotoon:

```js
const = () => {
  return (
    <div>
      <h1>Programming anecdotes</h1>
      <Notification />
      <AnecdoteForm />
      <AnecdoteList />
    </div>
  )
}
```

#### 6.15* paremmat anekdootit, step13

Välitä komponentille <i>AnecdoteList</i> connectin avulla ainoastaan yksi stateen liittyvä propsi, filtterin tilan perusteella näytettävät anekdootit samaan tapaan kuin materiaalin luvussa [Presentational/Container revisited](/osa6/monta_reduseria_connect#presentational-container-revisited).

Komponentti <i>AnecdoteList</i> siis typistyy suunnilleen seuraavaan muotoon

```js
const AnecdoteList = (props) => {
  const vote = (id) => {
    // ...
  }

  return (
    <div>
      {props.anecdotesToShow.map(anecdote => // highlight-line
        <div key={anecdote.id}>
          <div>
            {anecdote.content}
          </div>
          <div>
            has {anecdote.votes}
            <button onClick={() => vote(anecdote.id)}>vote</button>
          </div>
        </div>
      )}
    </div>
  )
}
```

</div>
