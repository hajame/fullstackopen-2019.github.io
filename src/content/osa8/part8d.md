---
mainImage: ../../images/part-8.svg
part: 8
letter: d
---

<div class="content">

<!-- Sovelluksen frontend toimii puhelinluettelon näyttämisen osalta päivitetyn palvelimen kanssa. Jotta luetteloon voitaisiin lisätä henkilöitä, tulee backendiin toteuttaa kirjautuminen. -->
The frontend of our application shows the phone directory just fine with the updated server. However if we want to add new persons, we have to add log in functionality to the backend. 

### User log in

<!-- Lisätään sovelluksen tilaan muuttuja _token_, joka tallettaa tokenin siinä vaiheessa kun käyttäjä on kirjautunut. Jos _token_ ei ole määritelty, näytetään kirjautumisesta huolehtiva komponentti <i>LoginForm</i>, joka saa parametriksi mutaation tekevän funktion _login_: -->
Let's add variable _token_ to the applications state. It saves the token when user has logged in. If _token_ is undefined, we show the component responsible for logging in, <i>LoginForm</i>. It is given the function responsible for the mutation, _login_, as a parameter:

```js
const LOGIN = gql`
  mutation login($username: String!, $password: String!) {
    login(username: $username, password: $password)  {
      value
    }
  }
`

const App = () => {
  const [token, setToken] = useState(null)

  // ...

  const login = useMutation(LOGIN)

  const errorNotification = () => errorMessage &&
    <div style={{ color: 'red' }}>
      {errorMessage}
    </div>

  if (!token) {
    return (
      <div>
        {errorNotification()}
        <h2>Login</h2>
        <LoginForm
          login={login}
          setToken={(token) => setToken(token)}
          handleError={handleError}
        />
      </div>
    )
  }

  return (
    // ...
  )
}
```

<!-- Jos kirjautuminen onnistuu, eli funktio _login_ ei heitä poikkeusta, talletetaan funktion palauttama <i>token</i> komponentin <i>App</i> tilaan. Token talletetaan myös <i>local storageen</i>, näin siihen on helpompi päästä käsiksi siinä vaiheessa kun haluamme asettaa tokenin <i>Authorization</i>-headeriin. -->
If login is successfull, so the _login_ function does not throw an error, the <i>token</i> it returns is saved to the state of the <i>App</i> component. The token is also saved to <i>local storage</i>. This way it is easier to access when we want to add it to the <i>Authorization</i>-header of a request.

<!-- Jos operaatio epäonnistuu, kutsutaan propsina saatua funktiota, joka asettaa komponentin <i>App</i> tilaan käyttäjälle näytettävän virheilmoituksen: -->
If the login operation fails, the function received as props is called, and it sets a error message to be shown to the state of the <i>App</i> component. 

```js
import React, { useState } from 'react'

const LoginForm = (props) => {
  const [username, setUsername] = useState('')
  const [password, setPassword] = useState('')

  const submit = async (event) => {
    event.preventDefault()

    try {
      const result = await props.login({
        variables: { username, password }
      })

      const token = result.data.login.value

      props.setToken(token)
      localStorage.setItem('phonenumbers-user-token', token)
    } catch(error){
      props.handleError(error)
    }
  }

  return (
    <div>
      <form onSubmit={submit}>
        <div>
          username <input
            value={username}
            onChange={({ target }) => setUsername(target.value)}
          />
        </div>
        <div>
          password <input
            type='password'
            value={password}
            onChange={({ target }) => setPassword(target.value)}
          />
        </div>
        <button type='submit'>login</button>
      </form>
    </div>
  )
}

export default LoginForm
```

<!-- Lisätään sovellukselle myös nappi, jonka avulla kirjautunut käyttäjä voi kirjautua ulos. Napin klikkauskäsittelijässä asetetaan  _token_ tilaan null, poistetaan token local storagesta ja resetoidaan Apollo clientin välimuisti. Tämä on [tärkeää](https://www.apollographql.com/docs/react/recipes/authentication.html#login-logout), sillä joissain kyselyissä välimuistiin on saatettu hakea dataa, johon vain kirjaantuneella käyttäjällä on oikeus päästä käsiksi. -->
Let's also add a button which enables logged in user to log out. The buttons onClick handler sets the _token_ state to null, removes the token from local storage and resets the cache of the Apollo client. 
The last is [important](https://www.apollographql.com/docs/react/recipes/authentication.html#login-logout), because some queries might have fetced data to cache, which only logged in users should have access to. 


```js
const App = () => {
  const client = useApolloClient()

  // ...

  const logout = () => {
    setToken(null)
    localStorage.clear()
    client.resetStore()
  }

  // ...
}

```

<!-- Sovelluksen tämän vaiheen koodi [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-6), branchissa <i>part8-6</i>. -->
The current code of the application can be found from [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-6), branch <i>part8-6</i>.

### Adding a token to a header

<!-- Backendin muutosten jälkeen uusien henkilöiden lisäys puhelinluetteloon vaatii sen, että käyttäjän token lähetetään pyynnön mukana. Jotta saamme tokenin lähetettyä pyyntöjen mukana, joudumme hieman muuttamaan tapaa, jonka avulla määrittelemme _ApolloClient_-olion tiedostossa <i>index.js</i> -->
After the backend changes creating new persons requires, that a valid user token is sent with the request. In order to send the token, we have to change the way we define the _ApolloClient_-object in <i>index.js</i> a little. 

```js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'
import ApolloClient from 'apollo-boost' // highlight-line
import { ApolloProvider } from 'react-apollo-hooks'

// highlight-start
const client = new ApolloClient({
  uri: "http://localhost:4000/graphql"
})
// highlight-end

ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>,
  document.getElementById('root')
)
```

<!-- Määrittely käyttää apunaan [apollo-boost](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-boost)-kirjastoa, joka dokumentaationsa mukaan -->
The new definition uses [apollo-boost](https://github.com/apollographql/apollo-client/tree/master/packages/apollo-boost)-library. According to its documentation

> <i>Apollo Boost is a zero-config way to start using Apollo Client. It includes some sensible defaults, such as our recommended InMemoryCache and HttpLink, which come configured for you with our recommended settings.</i>

<!-- Eli apollo-boost tarjoaa helpon tavan konfiguroida _ApolloClient_ useisiin tilanteisiin riittävillä oletusasetuksilla.  -->
so apollo-boost offers an easy way to configure _ApolloClient_ with settings suitable for most situations. 

<!-- Vaikka apollo-boostilla olisi myös mahdollista konfiguroida pyyntöihin asetettavat headerit, luovutaan nyt apollo-boostin käytöstä ja tehdään konfiguraatio kokonaan itse. -->
Even though it would be possible to also configure the request headers with apollo-boost, we will now abandon it and do the configuration ourselves. 

<!-- Konfiguraatio on seuraavassa: -->
The configuration is as follows:

```js
import { ApolloClient } from 'apollo-client'
import { createHttpLink } from 'apollo-link-http'
import { InMemoryCache } from 'apollo-cache-inmemory'
import { setContext } from 'apollo-link-context'

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
})

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('phonenumbers-user-token')
  return {
    headers: {
      ...headers,
      authorization: token ? `bearer ${token}` : null,
    }
  }
})

const client = new ApolloClient({
  link: authLink.concat(httpLink),
  cache: new InMemoryCache()
})
```

<!-- Konfiguraatio edellyttää kahden kirjaston asentamista: -->
It requires installing two libraries:

```js
npm install --save apollo-link apollo-link-context
```

<!-- _client_ muodostetaan nyt kirjaston [apollo-link](https://www.apollographql.com/docs/link/index.html) tarjoaman konstruktorifunktion [ApolloClient](https://www.apollographql.com/docs/react/api/apollo-client.html#apollo-client). Parametreja on kaksi, _link_ ja _cache_. Näistä jälkimmäinen määrittelee, että sovelluksen käyttöön tulee keskusmuistissa toimiva välimuisti [InMemoryCache](https://www.apollographql.com/docs/react/advanced/caching.html#smooth-scroll-top).  -->
_client_ is now configured using [ApolloClient](https://www.apollographql.com/docs/react/api/apollo-client.html#apollo-client) constructorfunction of [apollo-link](https://www.apollographql.com/docs/link/index.html). It has two parameters, _link_ and _cache_. The latter defines, that the application now uses a cache operating in the main memory [InMemoryCache](https://www.apollographql.com/docs/react/advanced/caching.html#smooth-scroll-top).

<!-- Ensimmäinen parametri _link_ määrittelee sen, miten client ottaa yhteyttä palvelimeen, jonka pohjalla on [httpLink](https://www.apollographql.com/docs/link/links/http.htm), eli normaali HTTP:n yli tapahtuva yhteys, jota on höystetty siten, että pyyntöjen mukaan [asetetaan headerille](https://www.apollographql.com/docs/react/recipes/authentication.html#Header) <i>authorization</i> arvoksi localStoragessa mahdollisesti oleva token. -->
The first parameter _link_ defines how the clients contacts the server. The communication is based on [httpLink](https://www.apollographql.com/docs/link/links/http.htm), a normal connection over HTTP with the addition that a token from localStorage is set as the value of the <i>authorization</i> [header](https://www.apollographql.com/docs/react/recipes/authentication.html#Header) if it exists. 

<!-- Uusien henkilöiden lisäys ja numeroiden muuttaminen toimii taas. Sovellukseen jää kuitenkin yksi ongelma. Jos yritämme lisätä puhelinnumerotonta henkilöä, se ei onnistu. -->
Creating new persons and changing numbers works again. There is however one remaining problem. If we try to add a person without a phonenumber, it is not possible. 

![](../images/8/25.png)

<!-- Validointi epäonnistuu, sillä frontend lähettää kentän _phone_ arvona tyhjän merkkijonon.  -->
Validation fails, because frontend sends an empty string as the value of _phone_.

<!-- Muutetaan uuden henkilön luovaa funktiota siten, että se asettaa kentälle _phone_  arvon _null_, jos käyttäjä ei ole syöttänyt kenttään mitään: -->
Let's change the function creating new persons so, that it sets _phone_ to null if user has not given a value. 

```js
const PersonForm = (props) => {
  // ...
  const submit = async (e) => {
    e.preventDefault()

    await props.addPerson({ 
      variables: { 
        name, street, city, // highlight-line
        phone: phone.length>0 ? phone : null // highlight-line
      } 
    })

  // ...
  }

  // ...
}
```

<!-- Sovelluksen tämän vaiheen koodi [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-7), branchissa <i>part8-7</i>. -->
Current application code can be found from [github](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-7), branch <i>part8-7</i>.

### Updating cache, revisited

<!-- Uusien henkilöiden lisäyksen yhteydessä on siis  -->
<!-- [päivitettävä](/osa8/react_ja_graph_ql#valimuistin-paivitys) Apollo clientin välimuisti. Päivitys tapahtuu määrittelemällä mutaation yhteydessä option _refetchQueries_ avulla, että kysely <em>ALL\_PERSONS</em> on suoritettava uudelleen: -->
When adding new persons, we must declare that the cache of Apollo client has to be [updated](/osa8/react_ja_graph_ql#valimuistin-paivitys). The cache can be updated by using the option _refetchQueries_ on the mutation to force <em>ALL\_PERSONS</em> query to be rerun. 


```js 
const App = () => {
  // ...

  const addPerson = useMutation(CREATE_PERSON, {
    onError: handleError,
    refetchQueries: [{ query: ALL_PERSONS }]
  })

  // ..
}
```

<!-- Lähestymistapa on kohtuullisen toimiva, ikävänä puolena on toki se, että päivityksen yhteydessä suoritetaan aina myös kysely.  -->
This approach is pretty good, the drawback being that the query is always rerun with any updates. 

<!-- Ratkaisua on mahdollista optimoida hoitamalla välimuistin päivitys itse. Tämä tapahtuu määrittelemällä mutaatiolle sopiva [update](https://www.apollographql.com/docs/react/advanced/caching.html#after-mutations)-callback, jonka Apollo suorittaa mutaation päätteeksi:  -->
It is possible to optimize the solution by handling updating the cache ourselves. This is done by defining a suitable [update](https://www.apollographql.com/docs/react/advanced/caching.html#after-mutations)-callback for the mutation, which Apollo runs after the mutation:


```js 
const App = () => {
  // ...

  const addPerson = useMutation(CREATE_PERSON, {
    onError: handleError,
    // highlight-start
    update: (store, response) => {
      const dataInStore = store.readQuery({ query: ALL_PERSONS })
      dataInStore.allPersons.push(response.data.addPerson)
      store.writeQuery({
        query: ALL_PERSONS,
        data: dataInStore
      })
    }
    // highlight-end
  })
 
  // ..
}  
```

<!-- Callback-funktio saa parametriksi viitteen välimuistiin sekä mutaation mukana palautetun datan, eli esimerkkimme tapauksessa lisätyn käyttäjän. -->
The callback function is given a reference to the cache and the data returned by the mutation as parameters. For example in our case this would be the created person. 

<!-- Koodi lukee funktion [readQuery](https://www.apollographql.com/docs/react/advanced/caching.html#readquery) avulla kyselyn <em>ALL\_PERSONS</em> välimuistiin talletetun tilan ja päivittää välimuistin funktion [writeQuery](https://www.apollographql.com/docs/react/advanced/caching.html#writequery-and-writefragment) avulla lisäten henkilöiden joukkoon mutaation lisäämän henkilön. -->
The code reads the cached state of <em>ALL\_PERSONS</em> query using [readQuery](https://www.apollographql.com/docs/react/advanced/caching.html#readquery) function and updates the cache with [writeQuery](https://www.apollographql.com/docs/react/advanced/caching.html#writequery-and-writefragment) function adding the new person to the cached data. 

<!-- On myös olemassa tilanteita, joissa ainoa järkevä tapa saada välimuisti pidettyä ajantasaisena on _update_-callbackillä tehtävä päivitys.  -->
There are some situations where the only good way to keep the cache up to date is using _update_ -callbacks. 

<!-- Tarvittaessa välimuisti on mahdollista kytkeä pois päältä joko koko sovelluksesta tai yksittäisiltä kyselyiltä määrittelemällä välimuistin käyttöä kontrolloivalle [fetchPolicy](https://www.apollographql.com/docs/react/api/react-apollo.html#query-props):lle arvo <em>no-cache</em>.  -->
When necessary it is possible to disable cache for the whole application or single queries by setting the field managing the use of cache, [fetchPolicy](https://www.apollographql.com/docs/react/api/react-apollo.html#query-props) as <em>no-cache</em>.

<!-- Voisimme määritellä, että yksittäisen henkilön osoitetietoja ei tallenneta välimuistiin: -->
We could declare, that the address details of a single person are not saved to cache:

```js 
const Persons = ({ result }) => {
  // ...
  const show = async (name) => {
    const { data } = await client.query({
      query: FIND_PERSON,
      variables: { nameToSearch: name },
      fetchPolicy: 'no-cache' // highlight-line
    })
    setPerson(data.findPerson)
  }

  // ...
}
``` 

<!-- Jätämme kuitenkin koodin ennalleen.  -->
We will however leave the code as is. 

<!-- Välimuistin kanssa kannattaa olla tarkkana. Välimuistissa oleva epäajantasainen data voi aiheuttaa vaikeasti havaittavia bugeja. Kuten tunnettua, välimuistin ajantasalla pitäminen on erittäin haastavaa. Koodareiden joukossa kulkevan kansanviisauden mukaan  -->
Be diligent with the cache. Old data in cache can cause hard to find bugs. As we know, keeping the cache up to date is very challenging. According to a coder proverb

> <i>There are only two hard things in Computer Science: cache invalidation and naming things.</i> Read more [here](https://www.google.com/search?q=two+hard+things+in+Computer+Science&oq=two+hard+things+in+Computer+Science).


<!-- Sovelluksen tämän vaiheen koodi [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-8), branchissa <i>part8-8</i>. -->
The current code of the application can be found from [github](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-8), branch <i>part8-8</i>.

</div>

<div class="tasks">

### Exercises

#### 8.17 Listing books

<!-- Backendin muutosten jälkeen kirjojen lista ei enää toimi. Korjaa se. -->
After the backend changes the list of books does not work anymore. Fix it. 

#### 8.18 Log in

<!-- Kirjojen lisäys ja kirjailijan syntymävuoden muutos eivät toimi, sillä ne edellyttävät kirjautumista.  -->
Adding new books and changing the birth year of an author do not work, because they require user to be logged in. 

<!-- Toteuta sovellukseesi kirjautuminen ja korjaa mutaatiot. -->
Implement log in functionality and fix the mutations. 

<!-- Sovelluksesi ei ole pakko käsitellä validointivirheitä järkevästi. -->
It is not necessary yet to handle validation errors. 

<!-- Voit päättää itse miltä kirjautuminen näyttää käyttöliittymässä. Eräs mahdollinen ratkaisu on tehdä kirjautumislomakkeesta erillinen näkymä jonne pääsee sovelluksen navigaatiomenusta: -->
You can decide how the log in looks on the interface. One possible solution is to make the log in form into a separate view which can be accessed through a navigation menu: 

![](../images/8/26.png)

<!-- Kirjatumislomake -->
The login form

![](../images/8/27.png)

<!-- Kun käyttäjä on kirjautuneena, muutetaan navigaatio näyttämään ne toiminnot, jotka ovat vain kirjautuneen käytettävissä -->
When a user is logged in, the navigation changes to show the functionalities which can only be done by a logged in user

![](../images/8/28.png)

#### 8.19 Books by genre, part 1

<!-- Laajenna sovellustasi siten, että kirjojen näkymästä voidaan rajata näytettävä kirjalista ainoastaan niihin, jotka kuuluvat valittuun genreen. Toteutuksesi voi näyttää seuraavalta: -->
Complete your application to filter the book list by genre. Your solution might look something like this:

![](../images/8/30.png)

#### 8.20 Books by genre, part 2

<!-- Tee sovellukseen näkymä, joka näyttää kirjautuneelle käyttäjälle käyttäjän lempigenreen kuuluvat kirjat. -->
Implement a view, which shows the user all books in their favourite genre.

![](../images/8/29.png)

#### 8.21 books by genre with GraphQL

<!-- Tietyn genren kirjoihin rajoittamisen voi tehdä kokonaan React-sovelluksen puolella. Voit merkitä tämän tehtävän, jos rajaat näytettävät kirjat tahtävässä 8.5 palvelimelle toteutetun suoran GraphQL-kyselyn avulla.  -->
The filtering can be done using just React. You can mark this exercise as completed if you filter the books using a GraphQL query to the server in exercise 8.5. 

<!-- Tämä **tehtävä on haastava** ja niin kurssin tässä vaiheessa jo kuuluukin olla. Muutama vihje -->
This **exercise is difficult** like it should be this late in the course. Some tips
<!-- - Komponetin <i>Query</i> tai hookin <i>useQuery</i> käytön sijaan saattaa olla parempi tehdä kyselyitä suoraan _client_-oliolla, jonhon päästään käsiksi komponentin [ApolloConsumer](https://www.apollographql.com/docs/react/essentials/queries.html#manual-query) tai hookilla [useApolloClient](https://github.com/trojanowski/react-apollo-hooks#useapolloclient), katso lisää [täältä](/osa8/react_ja_graph_ql#nimetyt-kyselyt-ja-muuttujat). -->
 - Instead of using the <i>Query</i> component or the <i>useQuery</i> hook it might be better to do the queries with the _client_-object, which can be accessed with the [ApolloConsumer](https://www.apollographql.com/docs/react/essentials/queries.html#manual-query) component or [useApolloClient](https://github.com/trojanowski/react-apollo-hooks#useapolloclient) hook. See more [here](/osa8/react_ja_graph_ql#nimetyt-kyselyt-ja-muuttujat).
<!-- - GraphQL-kyselyjen tuloksia kannattaa joskus tallentaan komponentin tilaan. -->
 -  It is sometimes useful to save the results of a GraphQL query to the state of a component. 
<!-- - Huomaa, että voit tehdä GraphQL-kyselyjä <i>useEffect</i>-hookissa. -->
 -  Note, that you can do GraphQL queries in a <i>useEffect</i>-hook.
<!-- - <i>useEffect</i>-hookin [toisesta parametrista](https://reactjs.org/docs/hooks-reference.html#conditionally-firing-an-effect) voi olla tehtävässä apua, se tosin riippuu käyttämästäsi lähestymistavasta. -->
 -  The [second parameter](https://reactjs.org/docs/hooks-reference.html#conditionally-firing-an-effect) of a <i>useEffect</i> - hook can become handy debending on your approach. 

#### 8.22 Up to date cache and book recommendations

<!-- Jos haet kirjasuositukset GraphQL:llä, varmista jollain tavalla se, että kirjojen näkymä säilyy ajantasaisena. Eli kun lisäät uuden kirjan, päivittyy se kirjalistalle **viimeistään** siinä vaiheessa kun painat jotain genrevalintanappia. Ilman uuden genrevalinnan tekemistä, ei näkymän ole pakko päivittyä.  -->
If you fetch the book recommendations with GraphQL, ensure somehow that the books view is kept up to date. So when a new book is added, the books view is updated **at least** when a genre selection button is pressed. When new genre selection is not done, the view does not have to be updated. 

</div>
