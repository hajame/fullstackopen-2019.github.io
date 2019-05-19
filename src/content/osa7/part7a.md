---
mainImage: ../../images/part-7.svg
part: 7
letter: a
---

<div class="content">

<!-- Kurssin seitsemännen osan tehtävät poikkeavat jossain määrin. Tässä luvussa on normaaliin tapaan [kolme luvun teoriaan liittyvää tehtävää](/osa7/react_router#tehtavia). -->

The exercises in the seventh part of the course differ a bit from the ones before. In this chapter there is, as usual, [three exercises related to the theory in this chapter](/osa7/react_router#tehtavia).

<!-- Tämän luvun tehtävien lisäksi seitsemäs osa sisältää kertaavan ja hieman tämän osan teoriaakin soveltavan [tehtäväsarjan](/osa7/tehtavia_blogilistan_laajennus), jossa laajennetaan osissa 4 ja 5 tehtyä Bloglist-sovellusta. -->

In addition to the exercises in this chapter there is a series of exercises, that revise what we've learned and apply some of the theory in this part, in which we expand the Bloglist-application we worked on in parts 4 and 5.

<!-- ### Sovelluksen navigaatiorakenne -->
### The navigation structure of the application

<!-- Palataan osan 6 jälkeen jälleen Reduxittoman Reactin pariin. -->

Following part 6 we return to React without Redux.

<!-- On erittäin tyypillistä, että web-sovelluksissa on navigaatiopalkki, jonka avulla on mahdollista vaihtaa sovelluksen näkymää. Muistiinpanosovelluksemme voisi sisältää pääsivun: -->

It is very common for web-applications to have a navigation bar, which enables switching the view of application. 

![](../images/7/1b.png)

<!-- ja omat sivunsa muistiinpanojen ja käyttäjien tietojen näyttämiseen: -->
and separate pages for showing the information of notes and users:

![](../images/7/2b.png)

<!-- [Vanhan koulukunnan websovelluksessa](/osa0#perinteinen-web-sovellus) sovelluksen näyttämän sivun vaihto tapahtui siten että selain teki palvelimelle uuden HTTP GET -pyynnön ja renderöi sitten palvelimen palauttaman näkymää vastaavan HTML-koodin. -->

In an [old school web app](/osa0#perinteinen-web-sovellus) changing the page shown by the application would be accomplished by the browser making a HTTP GET request to the server and rendering the HTML representing the view that was returned.

<!-- Single page appeissa taas ollaan todellisuudessa koko ajan samalla sivulla, ja selaimessa suoritettava Javascript-koodi luo illuusion eri "sivuista". Jos näkymää vaihdettaessa tehdään HTTP-kutsuja, niiden avulla haetaan ainoastaan JSON-muotoista dataa, jota uuden näkymän näyttäminen ehkä edellyttää. -->

In single page apps we are, in reality, always on the same page, and the Javascript code run by the browser creates an illusion of different "pages". If HTTP requests are made when switching view it is only for fetching JSON formatted data, which the new view might require for it to be shown.

<!-- Navigaatiopalkki ja useita näkymiä sisältävä sovellus on erittäin helppo toteuttaa Reactilla. -->

The navigation bar and an application containing multiple views is very easy to implement using React.

<!-- Seuraavassa on eräs tapa: -->

Here is one way:

```js
import React, { useState } from 'react'
import ReactDOM from 'react-dom'

const Home = () => (
  <div> <h2>TKTL notes app</h2> </div>
)

const Notes = () => (
  <div> <h2>Notes</h2> </div>
)

const Users = () => (
  <div> <h2>Users</h2> </div>
)

const App = () => {
  const [page, setPage] = useState('home')

 const  toPage = (page) => (event) => {
    event.preventDefault()
    setPage(page)
  }

  const content = () => {
    if (page === 'home') {
      return <Home />
    } else if (page === 'notes') {
      return <Notes />
    } else if (page === 'users') {
      return <Users />
    }
  }

  const padding = {
    padding: 5
  }

  return (
    <div>
      <div>
        <a href="" onClick={toPage('home')} style={padding}>
          home
        </a>
        <a href="" onClick={toPage('notes')} style={padding}>
          notes
        </a>
        <a href="" onClick={toPage('users')} style={padding}>
          users
        </a>
      </div>

      {content()}
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

<!-- Eli jokainen näkymä on toteutettu omana komponenttinaan ja sovelluksen tilassa <i>page</i> pidetään tieto siitä, minkä näkymää vastaava komponentti menupalkin alla näytetään. -->

So each view is implemented as its own component and we store the information for which component representing a view should be shown below the menu bar in the application state called <i>page</i>.

<!-- Menetelmä ei kuitenkaan ole optimaalinen. Kuten kuvista näkyy, sivuston osoite pysyy samana vaikka välillä ollaankin eri näkymässä. Jokaisella näkymällä tulisi kuitenkin olla oma osoitteensa, jotta esim. bookmarkien tekeminen olisi mahdollista. Sovelluksessamme ei myöskään selaimen <i>back</i>-painike toimi loogisesti, eli <i>back</i> ei vie edelliseksi katsottuun sovelluksen näkymään vaan jonnekin ihan muualle. Jos sovellus kasvaisi suuremmaksi ja sinne haluttaisiin esim. jokaiselle käyttäjälle sekä muistiinpanolle oma yksittäinen näkymänsä, itse koodattu <i>reititys</i> eli sivuston navigaationhallinta menisi turhan monimutkaiseksi. -->

However, the method is not very optimal. A we can see from the pictures, the address stays the same even though at times we are in different views. Each view should preferably have its own address, e.g to make bookmarking possible. The <i>back</i>-button doesn't work as expected for our application either, meaning that <i>back</i> doesn't move you to the previously displayed view of the application, but somewhere completely different. If the application were to grow even bigger and we wanted to e.g add their separate views for each user and note, then this self made <i>routing</i>, meaning the navigation management of the application, would get overly complicated.

<!-- Reactissa on onneksi valmis komponentti [React router](https://github.com/ReactTraining/react-router) joka tarjoaa erinomaisen ratkaisun React-sovelluksen navigaation hallintaan. -->

Luckily, in React there exists a component [React router](https://github.com/ReactTraining/react-router), which provides an excellent solution to the navigation management of a React-application.

<!-- Muutetaan ylläoleva sovellus käyttämään React routeria. Asennetaan React router komennolla -->

Lets change the above application to use React router. First we install React router with the command

```js
npm install --save react-router-dom
```

<!-- React routerin tarjoama reititys saadaan käyttöön muuttamalla sovellusta seuraavasti: -->

The routing provided by React router is enabled by changing the application as follows:

```js
import {
  BrowserRouter as Router,
  Route, Link, Redirect, withRouter
} from 'react-router-dom'

const App = () => {

  const padding = { padding: 5 }

  return (
    <div>
      <Router>
        <div>
          <div>
            <Link style={padding} to="/">home</Link>
            <Link style={padding} to="/notes">notes</Link>
            <Link style={padding} to="/users">users</Link>
          </div>
          <Route exact path="/" render={() => <Home />} />
          <Route path="/notes" render={() => <Notes />} />
          <Route path="/users" render={() => <Users />} />
        </div>
      </Router>
    </div>
  )
}
```

<!-- Reititys, eli komponenttien ehdollinen, selaimen <i>urliin perustuva</i> renderöinti otetaan käyttöön sijoittamalla komponentteja <i>Router</i>-komponentin lapsiksi, eli <i>Router</i>-tagien sisälle. -->

Routing, or the conditional rendering of components <i>based on the url</i> in the browser is used by placing components as children of the <i>Router</i> component, meaning inside <i>Router</i>-tags.

<!-- Huomaa, että vaikka komponenttiin viitataan nimellä <i>Router</i> kyseessä on [BrowserRouter](https://reacttraining.com/react-router/web/api/BrowserRouter), sillä importtaus tapahtuu siten, että importattava olio uudelleennimetään: -->

Notice, that even though the component is referred to by the name <i>Router</i> we are in fact talking about [BrowserRouter](https://reacttraining.com/react-router/web/api/BrowserRouter), because here the import happens by renaming the imported object:

```js
import {
  BrowserRouter as Router,
  Route, Link, Redirect, withRouter
} from 'react-router-dom'
```

<!-- Manuaalin mukaan -->

According to the manual

> <i>BrowserRouter</i> is a <i>Router</i> that uses the HTML5 history API (pushState, replaceState and the popState event) to keep your UI in sync with the URL.

<!-- Normaalisti selain lataa uuden sivun osoiterivillä olevan urlin muuttuessa. [HTML5 history API](https://css-tricks.com/using-the-html5-history-api/):n avulla <i>BrowserRouter</i> kuitenkin mahdollistaa sen, että selaimen osoiterivillä olevaa urlia voidaan käyttää React-sovelluksen sisäiseen "reitittämiseen", eli vaikka osoiterivillä oleva url muuttuu, sivun sisältöä manipuloidaan ainoastaan Javascriptillä ja selain ei lataa uutta sisältöä palvelimelta. Selaimen toiminta back- ja forward-toimintojen ja bookmarkien tekemisen suhteen on kuitenkin loogista, eli toimii kuten perinteisillä web-sivuilla. -->

Normally the browser loads a new page when the url in the address bar changes. However, with the help of the [HTML5 history API](https://css-tricks.com/using-the-html5-history-api/) <i>BrowserRouter</i> enables us to use the url in the address bar of the browser for internal "routing" in a React-application. So even if the url in the address bar changes, the content of the page is only manipulated using Javascript, and the browser doesn't load new content form the server. Using the back- and forward-actions, as well as the making bookmarks, is still logical like on a traditional web page.

<!-- Routerin sisälle määritellään selaimen osoiteriviä muokkaavia <i>linkkejä</i> komponentin [Link](https://reacttraining.com/react-router/web/api/Link) avulla. Esim. -->

Inside the router we define <i>links</i> that modify the address bar with the help of the [Link](https://reacttraining.com/react-router/web/api/Link) component. E.g.

```js
<Link to="/notes">notes</Link>
```

<!-- luo sovellukseen linkin, jonka teksti on <i>notes</i> ja jonka klikkaaminen vaihtaa selaimen osoiteriville urliksi <i>/notes</i>. -->

creates a link in the application with the text <i>notes</i>, which when clicked changes the url in the address bar to <i>/notes</i>.

<!-- Selaimen urliin perustuen renderöitävät komponentit määritellään komponentin [Route](https://reacttraining.com/react-router/web/api/Route) avulla. Esim. -->

Components rendered based on the url of the browser are defined with the help of the component [Route](https://reacttraining.com/react-router/web/api/Route). E.g.

```js
<Route path="/notes" render={() => <Notes />} />
```

<!-- määrittelee, että jos selaimen osoiteena on <i>/notes</i>, renderöidään komponentti <i>Notes</i>. -->

defines that if the address in the browser is <i>/notes</i>, then the component <i>Notes</i> is rendered.

<!-- Sovelluksen juuren, eli osoitteen  <code>/</code> määritellään renderöivän komponentti <i>Home</i>: -->

The root of the application, or the address <code>/</code>, is defined to render the component <i>Home</i>:

```js
<Route exact path="/" render={() => <Home />} />
```

<!-- joudumme käyttämään routen <i>path</i> attribuutin edessä määrettä <i>exact</i>, muuten <i>Home</i> renderöityy kaikilla muillakin poluilla, sillä juuri <code>/</code> on kaikkien muiden polkujen <i>alkuosa</i>. -->

we have to use the modifier <i>exact</i> in front of the <i>path</i> attribute of the route. Otherwise <i>Home</i> is also rendered on all other paths, since the root <code>/</code> is included at the <i>start</i> of all other paths.

<!-- ### Parametroitu route -->
### Parameterized route

<!-- Tarkastellaan sitten hieman modifioitua versiota edellisestä esimerkistä. Esimerkin koodi kokonaisuudessaan on [täällä](https://github.com/fullstack-hy2019/misc/blob/master/router-app.js). -->

Lets examine the slightly modified version from the previous example. The code for the example can be found in full over [here](https://github.com/fullstack-hy2019/misc/blob/master/router-app.js).

<!-- Sovellus sisältää nyt viisi eri näkymää, joiden näkyvyyttä kontrolloidaan routerin avulla. Edellisestä esimerkistä tuttujen komponenttien <i>Home</i>, <i>Notes</i> ja <i>Users</i> lisäksi mukana on kirjautumisnäkymää vastaava <i>Login</i> ja yksittäisen muistiinpanon näkymää vastaava <i>Note</i>. -->

The application now contains five different views, the display of of which is controlled by the router. In addition to the components <i>Home</i>, <i>Notes</i> and <i>Users</i> familiar from the previous example we have <i>Login</i> representing the login view and <i>Note</i> representing the view of a single note.

<!-- <i>Home</i> ja <i>Users</i> ovat kuten aiemmassa esimerkissä. <i>Notes</i> on hieman monimutkaisempi, se renderöi propseina saamansa muistiinpanojen listan siten, että jokaisen muistiinpanon nimi on klikattavissa -->

<i>Home</i> and <i>Users</i> are unchanged from the previous exercise.  <i>Notes</i> is a bit more complicated. It renders the list of notes passed to it as props in such a way that the name of each note is clickable.

![](../images/7/3a.png)

<!-- Nimen klikattavuus on toteutettu komponentilla <i>Link</i> ja esim. muistiinpanon, jonka id on 3 nimen klikkaaminen aiheuttaa selaimen osoitteen arvon päivittymisen muotoon <i>notes/3</i>: -->

The ability to click a name is implemented with the component <i>Link</i>, and e.g. clicking the name of a note, whose id is 3 would trigger an event causing the address of the browser to turn into <i>notes/3</i>:

```js
const Notes = (props) => (
  <div>
    <h2>Notes</h2>
    <ul>
      {props.notes.map(note =>
        <li key={note.id}>
          <Link to={`/notes/${note.id}`}>{note.content}</Link>
        </li>
      )}
    </ul>
  </div>
)
```

<!-- Kun selain siirtyy muistiinpanon yksilöivään osoitteeseen, esim. <i>notes/3</i>, renderöidään komponentti <i>Note</i>: -->

When the browser transitions to the address singling out the specific note, e.g. <i>notes/3</i>, the component <i>Note</i> is rendered:

```js
const Note = ({ note }) => (
  <div>
    <h2>{note.content}</h2>
    <div>{note.user}</div>
    <div><strong>{note.important ? 'tärkeä' : ''}</strong></div>
  </div>
)
```

<!-- Tämä tapahtuu laajentamalla komponentissa <i>App</i> olevaa reititystä seuraavasti: -->

This happens by expanding the routing in the <i>App</i> component as follows:

```js
<Router>
  <div>
    <div>
      <Link style={padding} to="/">home</Link>
      <Link style={padding} to="/notes">notes</Link>
      <Link style={padding} to="/users">users</Link>
    </div>

    <Route exact path="/" render={() =>
      <Home />
    } />
    // highlight-start
    <Route exact path="/notes" render={() =>
      <Notes notes={notes} />
    } />
    <Route exact path="/notes/:id" render={({ match }) =>
      <Note note={noteById(match.params.id)} />
    } />
  </div>
  // highlight-end
</Router>
```

<!-- Kaikki muistiinpanon renderöivään routeen on lisätty määre <i>exact path="/notes"</i> sillä muuten se renderöityisi myös <i>/notes/3</i>-muotoisten polkujen yhteydessä. -->

A modifier <i>exact path="/notes"</i> has been added to the route rendering all notes, because otherwise it would also be rendered for a path with the shape <i>/notes/3</i>.

<!-- Yksittäisen muistiinpanon näkymän renderöivä route määritellään "expressin tyyliin" merkkaamalla reitin parametrina oleva osa merkinnällä <i>:id</i> -->

The route rendering a single note is defined "in the style of express" by specifying the parameter of the route with the notation <i>:id</i>

```js
<Route exact path="/notes/:id" />
```

<!-- Renderöityvän komponentin määrittävä <i>render</i>-attribuutti pääsee käsiksi id:hen parametrinsa [match](https://reacttraining.com/react-router/web/api/match) avulla seuraavasti: -->

The render attribute, which defines the component to be rendered, can access the id using its parameter called [match](https://reacttraining.com/react-router/web/api/match) in the following way:

```js
render={({ match }) =>
  <Note note={noteById(match.params.id)} />}
```

<!-- Muuttujassa <i>match.params.id</i> olevaa id:tä vastaava muistiinpano selvitetään apufunktion _noteById_ avulla -->

The note corresponding to the id in <i>match.params.id</i> is resolved using a helper function _noteById_ 

```js
const noteById = (id) =>
  notes.find(note => note.id === Number(id))
```

<!-- renderöityvä <i>Note</i>-komponentti saa siis propsina urlin yksilöivää osaa vastaavan muistiinpanon. -->

and finally the <i>Note</i>-component being rendered gets the note singled out by the unique part of the url as one of its props.

<!-- ### withRouter ja history -->
### withRouter and history

<!-- Sovellukseen on myös toteutettu erittäin yksinkertainen kirjautumistoiminto. Jos sovellukseen ollaan kirjautuneena, talletetaan tieto kirjautuneesta käyttäjästä komponentin <i>App</i> tilaan <i>user</i>. -->

We have also implemented a very simple login feature. If one is logged into the application we store the information about this user in the <i>App</i>'s state <i>user</i>

<!-- Mahdollisuus <i>Login</i>-näkymään navigointiin renderöidään menuun ehdollisesti -->

The option to navigate to the <i>Login</i>-view is rendered conditionally in the menu

```js
<Router>
  <div>
    <div>
      <Link style={padding} to="/">home</Link>
      <Link style={padding} to="/notes">notes</Link>
      <Link style={padding} to="/users">users</Link>
      // highlight-start
      {user
        ? <em>{user} logged in</em>

        : <Link to="/login">login</Link>
      }
      // highlight-end
    </div>
  </div>
</Router>
```

<!-- eli jos käyttäjä on kirjautunut, renderöidäänkin linkin <i>Login</i> sijaan kirjautuneen käyttäjän käyttäjätunnus: -->

so if the user is already logged in, instead of displaying the link <i>Login</i> we show the username of the user:

![](../images/7/4a.png)

<!-- Kirjautumisesta huolehtivan komponentin koodi seuraavassa -->

The code of the component handling the login functionality is as follows 

```js
import {
  // ...
  withRouter // highlight-line
} from 'react-router-dom'

const LoginNoHistory = (props) => {
  const onSubmit = (event) => {
    event.preventDefault()
    props.onLogin('mluukkai')
    props.history.push('/') // highlight-line
  }

  return (
    <div>
      <h2>login</h2>
      <form onSubmit={onSubmit}>
        <div>
          username: <input />
        </div>
        <div>
          password: <input type='password' />
        </div>
        <button type="submit">login</button>
      </form>
    </div>
  )
}

const Login = withRouter(LoginNoHistory) // highlight-line
```

<!-- Lomakkeen toteutukseen liittyy muutama huomionarvoinen seikka. Kirjautumisen yhteydessä funktiossa _onSubmit_ kutsutaan propseina vastaanotetun [history](https://reacttraining.com/react-router/web/api/history)-olion metodia _push_. Käytetty komento _props.history.push('/')_ saa aikaan sen, että selaimen osoiteriville tulee osoitteeksi <code>/</code> ja sovellus renderöi osoitetta vastaavan komponentin <i>Home</i>. -->

There are a few notable things about the implementation of the form. When logging in we call the function _onSubmit_, which calls a method called _push_ of the [history](https://reacttraining.com/react-router/web/api/history)-object received by the component as a prop. The command _props.history.push('/')_ results in the address bar of the browser changing its address to <code>/</code> and thereby makes the application render the respective component, which in this case is <i>Home</i>.

<!-- Komponentti saa käyttöönsä propsin <i>history</i> siten, että se "kääritään" funktiolla [withRouter](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/withRouter.md). Funktio _withRouter_ toimii hyvin samaan tapaan kuin Reduxin yhteydessä käytetty _connect_, se lisää parametrina saamallensa komponentille joukon propseja. -->

The component gets access to the <i>history</i>-prop after it is "wrapped" by the function [withRouter](https://github.com/ReactTraining/react-router/blob/master/packages/react-router/docs/api/withRouter.md).

### redirect

<!-- Näkymän <i>Users</i> routeen liittyy vielä eräs mielenkiintoinen detalji: -->

There is one more interesting detail about the <i>Users</i> route: 

```js
<Route path="/users" render={() =>
  user ? <Users /> : <Redirect to="/login" />
} />
```

<!-- Jos käyttäjä ei ole kirjautuneena, ei renderöidäkään näkymää <i>Users</i> vaan sen sijaan <i>uudelleenohjataan</i> käyttäjä <i>Redirect</i>-komponentin avulla kirjautumisnäkymään -->

If a user isn't logged in the <i>Users</i> is not rendered. Instead the user is <i>redirected</i> using the <i>Redirect</i>-component to the login view

```js
<Redirect to="/login" />
```

<!-- Todellisessa sovelluksessa olisi kenties parempi olla kokonaan näyttämättä navigaatiovalikossa kirjautumista edellyttäviä näkymiä jos käyttäjä ei ole kirjautunut sovellukseen. -->

In reality it would perhaps be better to not even show links in the navigation bar requiring login if the user is not logged into the application.

<!-- Seuraavassa vielä komponentin <i>App</i> koodi kokonaisuudessaan: -->

Here is the <i>App</i> component in its entirety:

```js
const App = () => {
  const [notes, setNotes] = useState([
    {
      id: 1,
      content: 'HTML on helppoa',
      important: true,
      user: 'Matti Luukkainen'
    },
    // ...
  ])

  const [user, setUser] = useState(null)

  const login = (user) => {
    setUser(user)
  }

  const noteById = (id) =>
    notes.find(note => note.id === Number(id))

  const padding = { padding: 5 }

  return (
    <div>
      <Router>
        <div>
          <div>
            <Link style={padding} to="/">home</Link>
            <Link style={padding} to="/notes">notes</Link>
            <Link style={padding} to="/users">users</Link>
            {user
              ? <em>{user} logged in</em>
              : <Link to="/login">login</Link>
            }
          </div>

          <Route exact path="/" render={() =>
            <Home />
          } />
          <Route exact path="/notes" render={() =>
            <Notes notes={notes} />
          } />
          <Route exact path="/notes/:id" render={({ match }) =>
            <Note note={noteById(match.params.id)} />
          } />
          <Route path="/users" render={() =>
            user ? <Users /> : <Redirect to="/login" />
          } />
          <Route path="/login" render={() =>
            <Login onLogin={login} />
          } />
        </div>
      </Router>
      <div>
        <br />
        <em>Note app, Department of Computer Science 2019</em>
      </div>
    </div>
  )
}
```

<!-- Render-metodissa määritellään myös kokonaan <i>Router</i>:in ulkopuolella oleva nykyisille web-sovelluksille tyypillinen <i>footer</i>-elementti, eli sivuston pohjalla oleva osa, joka on näkyvillä riippumatta siitä mikä komponentti sovelluksen reititetyssä osassa näytetään. -->

We define an element common for modern web apps called <i>footer</i>, which defines the part at the bottom of the screen, outside of the <i>Router</i> so that it is shown regardless of the component shown in the routed part of the application.

</div>
<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Jatketaan anekdoottien parissa. Ota seuraaviin tehtäviin pohjaksi repositoriossa <https://github.com/fullstack-hy2019/routed-anecdotes> oleva reduxiton anekdoottisovellus. -->

Lets return to working with anecdotes. Use the redux-free anecdote app found in the repository <https://github.com/fullstack-hy2019/routed-anecdotes> as the starting point for the exercises.

<!-- Jos kloonaat projektin olemassaolevan git-reposition sisälle, <i>poista kloonatun sovelluksen git-konfiguraatio:</i> -->

If you clone the project into an existing git repository remember to <i>delete the git configuration of the cloned application:</i>

```bash
cd routed-anecdotes   // eli mene ensin kloonatun repositorion hakemistoon
rm -rf .git
```

<!-- Sovellus käynnistyy normaaliin tapaan, mutta joudut ensin asentamaan sovelluksen riippuvuudet: -->

The application starts the usual way, but first you need to install the dependencies of the application:

```bash
npm install
npm start
```

#### 7.1: routed anecdotes, step1

<!-- Lisää sovellukseen React Router siten, että <i>Menu</i>-komponentissa olevia linkkejä klikkailemalla saadaan säädeltyä näytettävää näkymää. -->

Add React Router to the application so that by clicking links in the <i>Menu</i>-component the view can be changed.

<!-- Sovelluksen juuressa, eli polulla _/_ näytetään anekdoottien lista: -->

At the root of the application, meaning the path _/_, show the list of anecdotes:

![](../assets/teht/40.png)

<!-- Pohjalla oleva <i>Footer</i>-komponentti tulee näyttää aina. -->

The <i>Footer</i>-component should always be visible at the bottom.

<!-- Uuden anekdootin luominen tapahtuu esim. polulla <i>create</i>: -->

The creation of a new anecdote should happen e.g. in the path <i>create</i>:

![](../assets/teht/41.png)

<!-- Huom: jos saat seuraavan virheilmoituksen -->

NB: if you get the following error message

![](../assets/teht/39.png)

<!-- pääset siitä eroon sisällyttämällä kaiken Router-elementin sisälle tulevaan <i>div</i>-elementtiin: -->

you can get rid of it by enclosing everything inside a <i>div</i>-element placed inside the Router-element:

```bash
<Router>
  <div>
    ...
  </div>
</Router>
```

#### 7.2: routed anecdotes, step2

<!-- Toteuta sovellukseen yksittäisen anekdootin tiedot näyttävä näkymä: -->

Implement a view for showing a single anecdote:

![](../assets/teht/42.png)

<!-- Yksittäisen anekdootin sivulle navigoidaan klikkaamalla anekdootin nimeä -->

Navigating to the page showing the single anecdote is done by clicking the name of that anecdote

![](../assets/teht/43.png)

#### 7.3: routed anecdotes, step3

<!-- Luomislomakkeen oletusarvoinen toiminnallisuus on melko hämmentävä, sillä kun lomakkeen avulla luodaan uusi muistiinpano, mitään ei näytä tapahtuvan. -->

The default functionality of the creation form is quite confusing, because after creating a new anecdote using the form nothing seems to be happening.

<!-- Paranna toiminnallisuutta siten, että luomisen jälkeen siirrytään automaattisesti kaikkien anekdoottien näkymään <i>ja</i> käyttäjälle näytetään 10 sekunnin ajan onnistuneesta lisäyksestä kertova notifikaatio: -->

Improve the functionality such that after creating a new anecdote the application transitions automatically to showing the view for all anecdotes <i>and</i> the user is shown a notification informing them of this successful creation for the next 10 seconds:

![](../assets/teht/44.png)

</i>
