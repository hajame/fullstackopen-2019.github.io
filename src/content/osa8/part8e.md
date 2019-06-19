---
mainImage: ../../images/part-8.svg
part: 8
letter: e
---

<div class="content">

<!-- Kurssi lähestyy loppuaan. Katsotaan lopuksi vielä muutamaa GraphQL:ään liittyvää asiaa. -->
We are approaching the end of the course. Let's finish by having a look at few more details of GraphQL. 

### fragments

<!-- GraphQL:ssä on suhteellisen yleistä, että eri kyselyt palauttavat samanlaisia vastauksia. Esim. puhelinluettelossa yhden henkilön hakeva kysely -->
It is pretty common in GraphQL that multiple queries return similar results. For example the query for the details of a person

```js
query {
  findPerson(name: "Pekka Mikkola") {
    name
    phone
    address{
      street 
      city
    }
  }
}
```

<!-- ja kaikki henkilöt hakeva kysely -->
and the query for all persons

```js
query {
  allPersons {
    name
    phone
    address{
      street 
      city
    }
  }
}
```

<!-- palauttavat molemmat henkilöitä. Valitessaan palautettavia kenttiä, molemmat kyselyt joutuvat määrittelemään täsmälleen samat kentät.  -->
both return persons. When choosing the fields to return, both queries have to define exactly the same fields. 

<!-- Tälläisiä tilanteita voidaan yksinkertaistaa [fragmenttien](https://graphql.org/learn/queries/#fragments) avulla. Määritellään kaikki henkilön tiedot valitseva fragmentti: -->
These kinds of situations can be simplified with the use of [fragments](https://graphql.org/learn/queries/#fragments). Lets declare a fragment for selecting all fields of a person: 

```js
fragment PersonDetails on Person {
  name
  phone 
  address {
    street 
    city
  }
}
```

<!-- Kyselyt voidan nyt tehdä fragmenttien avulla kompaktimmassa muodossa: -->
With the fragment we can do the queries in a compact form:

```js
query {
  allPersons {
    ...PersonDetails // highlight-line
  }
}

query {
  findPerson(name: "Pekka Mikkola") {
    ...PersonDetails // highlight-line
  }
}
```

<!-- Fragmentteja <i><strong>ei määritellä</strong></i> GraphQL:n skeemassa, vaan kyselyn tekevän clientin puolella. Fragmenttien tulee olla määriteltynä siinä vaiheessa kun client käyttää kyselyssään niitä.  -->
The fragments <i><strong>are not</strong></i> defined in the GraphQL schema, but in the client. The fragments must be declared when the client uses them for queries. 

<!-- Voisimme periaatteessa määritellä fragmentin jokaisen kyselyn yhteydessä seuraavasti: -->
In princible we could declare the fragment with each query like so:

```js
const ALL_PERSONS = gql`
{
  allPersons  {
    ...PersonDetails
  }
}
fragment PersonDetails on Person {
  name
  phone 
  address {
    street 
    city
  }
}
`
```

<!-- Huomattavasti järkevämpää on kuitenkin määritellä fragmentti kertaalleen ja sijoittaa se muuttujaan. -->
however it is much better to declare the fragment once and save it to a variable. 

```js
const PERSON_DETAILS = gql`
fragment PersonDetails on Person {
  id
  name
  phone 
  address {
    street 
    city
  }
}
`
```

<!-- Näin määritelty fragmentti voidaan upottaa kaikkiin sitä tarvitseviin kyselyihin ja mutaatioihin "prosenttiaaltosulku"-operaatiolla: -->
Declared like this the fragment can be placed to any query or mutation with the "precentcurlybrace"-operator:

```js
const ALL_PERSONS = gql`
{
  allPersons  {
    ...PersonDetails
  }
}
${PERSON_DETAILS}  
`
```

### Subscriptions

<!-- GraphQL tarjoaa query- ja mutation-tyyppien lisäksi kolmannenkin operaatiotyypin, [subscriptionin](https://www.apollographql.com/docs/react/advanced/subscriptions.html), jonka avulla clientit voivat <i>tilata</i> palvelimelta tiedotuksia palvelimella tapahtuneista muutoksista. -->
Along with query- and mutation types, GraphQL offers a third operation type, [subscription](https://www.apollographql.com/docs/react/advanced/subscriptions.html). With subscriptions clients can <i>subscribe to</i> updates about changes in the server. 

<!-- Subscriptionit poikkeavatkin radikaalisti kaikesta, mitä kurssilla on tähän mennessä nähty. Toistaiseksi kaikki interaktio on koostunut selaimessa olevan React-sovelluksen palvelimelle tekemistä HTTP-pyynnöistä. Myös GraphQL:n queryt ja mutaatiot on hoidettu näin. Subscriptionien myötä tilanne käänyy päinvastaiseksi. Sen jälkeen kun selaimessa oleva sovellus on tehnyt tilauksen muutostiedoista, alkaa selain kuunnella palvelinta. Muutosten tullessa palvelin lähettää muutostiedon <i>kaikille sitä kuunteleville</i> selaimille. -->
Subscriptions are radically different from anything we have seen in this course so far. Until now all interaction between browser and the server has been React application in the browser making HTTP-requests to the server. GraphQL queries and mutations have also been done this way. 
With subscriptions the situation is the opposite. After an application has made a subscription, it starts to listen to the server. 
When changes occur on the server, it sends a notification to all of its <i>subscribers</i>.


<!-- Teknisesti ottaen HTTP-protokolla ei taivu hyvin palvelimelta selaimeen päin tapahtuvaan kommunikaatioon. Konepellin alla Apollo käyttääkin [WebSocketeja](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) hoitamaan tilauksista aiheutuvan kommunikaation. -->
Technically speaking the HTTP-protocol is not well suited for communication from the server to the browser, so under the hood Apollo uses [WebSocks](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) for server subscriber communication. 

### Subscriptions on the server

<!-- Toteutetaan nyt sovellukseemme subscriptiot, joiden avulla palvelimelta on mahdollista tilata tieto puhelinluetteloon lisätyistä henkilöistä. -->
Let's implement subscriptions for subscribing for notifications about new persons added.

<!-- Palvelimella ei ole tarvetta kovin monille muutoksille. Skeemaan tarvitaan seuraava lisäys: -->
There are not many changes to the server. The schema changes like so:

```js
type Subscription {
  personAdded: Person!
}    
```

<!-- Eli kun uusi henkilö luodaan, palautetaan henkilön tiedot kaikille tilaajille. -->
So when a new person is added, all of it's details are sent to all subscribers. 

<!-- Määritellylle tilaukselle _personAdded_ tarvitaan resolveri. Myös lisäyksen tekevää resolveria _addPerson_ on muutettava siten, että uuden henkilön lisäys aiheuttaa ilmoituksen tilauksen tehneille. -->
The subscription _personAdded_ needs a resolver. The _addPerson_ resolver also has to be modified so, that is sends a notification to subscribers. 

<!-- Muutokset ovat seuraavassa: -->
The required changes are as follows:

```js
const { PubSub } = require('apollo-server') // highlight-line
const pubsub = new PubSub() // highlight-line

  Mutation: {
    addPerson: async (root, args, context) => {
      const person = new Person({ ...args })
      const currentUser = context.currentUser

      if (!currentUser) {
        throw new AuthenticationError("not authenticated")
      }

      try {
        await person.save()
        currentUser.friends = currentUser.friends.concat(person)
        await currentUser.save()
      } catch (error) {
        throw new UserInputError(error.message, {
          invalidArgs: args,
        })
      }

      pubsub.publish('PERSON_ADDED', { personAdded: person })  // highlight-line

      return person
    },  
  },
  // highlight-start
  Subscription: {
    personAdded: {
      subscribe: () => pubsub.asyncIterator(['PERSON_ADDED'])
    },
  },
  // highlight-end
```

<!-- Tilausten yhteydessä kommunikaatio tapahtuu [publish-subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)-periaatteella käyttäen rajapinnan [PubSub](https://www.apollographql.com/docs/graphql-subscriptions/setup.html#setup) toteuttavaa olioa. Uuden henkilön lisäys <i>julkaisee</i> tiedon lisäyksestä kaikille muutokset tilanneille PubSubin metodilla _publish_.  -->
With subscriptions the communication happens using the [publish-subscribe](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)-princible utilizing an object which realizes a [PubSub](https://www.apollographql.com/docs/graphql-subscriptions/setup.html#setup) interface. Adding a new person <i>publishes</i> a notification about the operation to all subscribers with PubSubs method _publish_.

<!-- Subscriptionin _personAdded_ resolveri rekisteröi tiedotteista kiinnostuneet clientit palauttamalla niille sopivan [iteraattoriolion](https://www.apollographql.com/docs/graphql-subscriptions/subscriptions-to-schema.html). -->
_PersonAdded_ subscriptions resolver registers all of the subscribers by returning them a suitable [iterator object](https://www.apollographql.com/docs/graphql-subscriptions/subscriptions-to-schema.html).

<!-- Muutetaan palvelimen käynnistävää koodia seuraavasti -->
Let's do the following changes to the code which starts the server
```js
// ...

server.listen().then(({ url, subscriptionsUrl }) => { // highlight-line
  console.log(`Server ready at ${url}`)
  console.log(`Subscriptions ready at ${subscriptionsUrl}`) // highlight-line
})
```

<!-- Nyt näemme, että palvelin kuuntelee subscriptioita osoitteessa _ws://localhost:4000/graphql_ -->
We see, that the server listens for subscriptions in the address _ws://localhost:4000/graphql_

```js
Server ready at http://localhost:4000/
Subscriptions ready at ws://localhost:4000/graphql
```

<!-- Muita muutoksia palvelimeen ei tarvita.  -->
No other changes to the server are needed.

<!-- Tilauksia on mahdollista testata GraphQL-playgroundin avulla seuraavasti: -->
It's possible to test the subscriptions with the GraphQL playground like this:

![](../images/8/31.png)

<!-- Kun tilauksen "play"-painiketta painetaan, jää playground odottamaan tilaukseen tulevia vastauksia.  -->
When you press "play" on a subscription, the playground waits for notifications to the subscription. 

<!-- Backendin koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-6), branchissa <i>part8-6</i>. -->
The backend code can be found from [github](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-6), branch <i>part8-6</i>.

### Subscriptions on the client

<!-- Jotta saamme tilaukset käyttöön React-sovelluksessa, tarvitaan hieman enemmän muutoksia, erityisesti [konfiguraatioiden osalta](https://www.apollographql.com/docs/react/advanced/subscriptions.html#subscriptions-client). Tiedostossa <i>index.js</i> olevat konfiguraatiot on muokattava seuraavaan muotoon: -->
In order to use subscriptions in our React application, we have to do some changes, especially on its [configurations](https://www.apollographql.com/docs/react/advanced/subscriptions.html#subscriptions-client).
The configurations in <i>index.js</i> have to be modified like so: 

```js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

import { ApolloProvider } from 'react-apollo'
import { ApolloProvider as ApolloHooksProvider } from 'react-apollo-hooks'

import { ApolloClient } from 'apollo-client'
import { createHttpLink } from 'apollo-link-http'
import { InMemoryCache } from 'apollo-cache-inmemory'
import { setContext } from 'apollo-link-context'

import { split } from 'apollo-link'
import { WebSocketLink } from 'apollo-link-ws'
import { getMainDefinition } from 'apollo-utilities'

const wsLink = new WebSocketLink({
  uri: `ws://localhost:4000/graphql`,
  options: { reconnect: true }
})

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
})

const authLink = setContext((_, { headers }) => {
  const token = localStorage.getItem('phonebook-user-token')
  return {
    headers: {
      ...headers,
      authorization: token ? `bearer ${token}` : null,
    }
  }
})

const link = split(
  ({ query }) => {
    const { kind, operation } = getMainDefinition(query)
    return kind === 'OperationDefinition' && operation === 'subscription'
  },
  wsLink,
  authLink.concat(httpLink),
)

const client = new ApolloClient({
  link,
  cache: new InMemoryCache()
})

ReactDOM.render(
  <ApolloProvider client={client}>
    <ApolloHooksProvider client={client}>
      <App />
    </ApolloHooksProvider>
  </ApolloProvider>,
  document.getElementById('root')
)
```

<!-- Jotta kaikki toimisi, on asennettava uusia riippuvuuksia: -->
For this to work, we have to install some dependencies:

```js
npm install --save subscriptions-transport-ws apollo-link-ws
```

<!-- Uudet konfiguraatiot johtuvat kahdesta muutostarpeesta. Simppelimpi näistä on tarve  -->
<!-- [palata malliin](/osa8/react_ja_graph_ql#react-apollo-hooks), jossa voimme käyttää GraphQL:ää sekä hookien että render props -komponenttien kanssa (react-apollo-hooks ei vielä tue subscritpioneita): -->
The new configurations are due to two factors. The first being, that we need to go back to [a model](/osa8/react_ja_graph_ql#react-apollo-hooks), where we can use GraphQL both with hooks and with render props -components (react-apollo-hooks does not support subscriptions as of now).

```js
import { ApolloProvider } from 'react-apollo'
import { ApolloProvider as ApolloHooksProvider } from 'react-apollo-hooks'

// ...

ReactDOM.render(
  <ApolloProvider client={client}>
    <ApolloHooksProvider client={client}>
      <App />
    </ApolloHooksProvider>
  </ApolloProvider>,
  document.getElementById('root')
```

<!-- Toinen konfuguraatiomuutos huomioi sen, että sovelluksella tulee nyt olla HTTP-yhteyden lisäksi websocket-yhteys GraphQL-palvelimelle: -->
The second factor is, that the application must have HTTP-connection as well as websocet- connection to the GraphQL server.

```js
const wsLink = new WebSocketLink({
  uri: `ws://localhost:4000/graphql`,
  options: { reconnect: true }
})

const httpLink = createHttpLink({
  uri: 'http://localhost:4000/graphql',
})
```

<!-- Tilaukset tehdään komponentin  -->
<!-- [Subscription](https://www.apollographql.com/docs/react/advanced/subscriptions.html#subscription-component) avulla. -->
The subscriptions are done using the [Subscription](https://www.apollographql.com/docs/react/advanced/subscriptions.html#subscription-component) component.

<!-- Tehdään koodiin seuraavat muutokset: -->
Let's modify the code like so:

```js
import { Subscription } from 'react-apollo' // highlight-line

// highlight-start
const PERSON_ADDED = gql`
subscription {
  personAdded {
    ...PersonDetails
  }
}
${PERSON_DETAILS}
`
// highlight-end

const App = () => {
  return (
    <div>
      // ...
      // highlight-start
      <Subscription
        subscription={PERSON_ADDED}
        onSubscriptionData={({subscriptionData}) => {
          console.log(subscriptionData)
        }}
      > 
        {() => null}
      </Subscription>
      // highlight-end
    </div>
  )
}
```

<!-- Kun puhelinluetteloon nyt lisätään henkilöitä, tapahtuupa se mistä tahansa, tulostuvat clientin konsoliin lisätyn henkilön tiedot: -->
When a new person is now added to the phonebook, no matter where it's done, the details of the new person are printed to the clients console: 

![](../images/8/32.png)

<!-- Kun luetteloon lisätään uusi henkilö, palvelin lähettää siitä tiedot clientille ja komponentin _Subscription_ attribuuttissa _onSubscriptionData_ määriteltyä callback-funktiota kutsutaan antaen sille parametriksi palvelimelle lisätty henkilö.  -->
When a new person is added, the server sends a notification to the client, and the callback-function defined in the _onSubscriptionData_ attribute of the _Subscription_ component is called and given the details of the new person as parameters. 


<!-- Toinen tapa ottaa palvelimen lähettämää dataa vastaan olisi komponentin _Subscription_ render prop -funktio, joka on nyt ainoastaan
  -->
  Another way to receive data from the server would be the render prop function of the _Subscription_ component, which is currently

```js
() => null
```

<!-- sillä data on käytännöllisempää käsitellä callback-funktiossa. -->
because it is more practical to handle data in a callback-function. 

<!-- Laajennetaan ratkaisua vielä siten, että uuden henkilön tietojen saapuessa henkilö lisätään Apollon välimuistiin, jolloin se renderöityy heti ruudulle. Koodissa on jouduttu huomioimaan se, että sovelluksen itsensä lisäämää henkilöä ei saa lisätä välimuistiin kahteen kertaan: -->
Let's extend our solution so that when the details of a new person are received, the person is added to the Apollo cache, so it is rendered to the screen immediately. 
However we have to keep in mind, that when our application creates a new person, it should not be added to the cache twice: 

```js
import { Subscription } from 'react-apollo'

const App = () => {
  // ...

  const includedIn = (set, object) => 
    set.map(p => p.id).includes(object.id)    

  const addPerson = useMutation(CREATE_PERSON, {
    onError: handleError,
    update: (store, response) => {
      const dataInStore = store.readQuery({ query: ALL_PERSONS })
      // highlight-start
      const addedPerson = response.data.addPerson
      
      if (!includedIn(dataInStore.allPersons, addedPerson)) {
        dataInStore.allPersons.push(addedPerson)
        client.writeQuery({
          query: ALL_PERSONS,
          data: dataInStore
        })
      }
      // highlight-end
    }
  })

  return (
    <div>
      // ...
      <Subscription
        subscription={PERSON_ADDED}
        onSubscriptionData={({subscriptionData}) => {
          const addedPerson = subscriptionData.data.personAdded
          // näytetään muualla tehty lisäys myös notifikaationa
          notify(`${addedPerson.name} added`)

          const dataInStore = client.readQuery({ query: ALL_PERSONS })
          if (!includedIn(dataInStore.allPersons, addedPerson)) {
            dataInStore.allPersons.push(addedPerson)
            client.writeQuery({
              query: ALL_PERSONS,
              data: dataInStore
            })
          }
        }}
      > 
        {() => null}
      </Subscription>
    </div>
  )
}
```

<!-- Clientin lopullinen koodi [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-9), branchissa <i>part8-9</i>. -->
The final code of the client can be found from [github](https://github.com/fullstack-hy2019/graphql-phonebook-frontend/tree/part8-9), branch <i>part8-9</i>.

### n+1-problem

<!-- Laajennetaan vielä backendia hieman. Muutetaan skeemaa siten, että tyypille <i>Person</i> tulee kenttä _friendOf_, joka kertoo kenen kaikkien käyttäjien tuttavalistalla ko henkilö on. -->
Let's add some things to the backend. Let's modify the schema so, that a <i>Person</i> type has a _friendOf_ field, which tells whose firends list the person is on. 

```js
type Person {
  name: String!
  phone: String
  address: Address!
  friendOf: [User!]!
  id: ID!
}
```

<!-- Sovellukseen tulisi siis saada tuki esim. seuraavalle kyselylle: -->
The application should support i.e the following query: 

```js
query {
  findPerson(name: "Leevi Hellas") {
    friendOf{
      username
    }
  }
}
```

<!-- Koska _friendOf_ ei ole tietokannassa olevien <i>Person</i>-olioiden sarake, on sille tehtävä oma resolveri, joka osaa selvittää asian. Tehdään aluksi tyhjän listan palauttava resolveri: -->
Because _friendOf_ is not a field of <i>Person</i>-objects on the database, we have to create a resolver for it, which can solve this issue. Let's first create a resolver that returns an empty list: 

```js
Person: {
  address: (root) => {
    return { 
      street: root.street,
      city: root.city
    }
  },
  // highlight-start
  friendOf: (root) => {
    // return list of users 
    return [
    ]
  }
  // highlight-end
},
```

<!-- Resolverin parametrina _root_ on se henkilöolio jonka tuttavalista on selvityksen alla, eli etsimme olioista _User_ ne, joiden _friends_-listalle sisältyy root._id: -->
The parameter _root_ is the person object whichs friends list is being created, so we search from all _User_ objects the ones which have root._id in their friends list: 

```js
  Person: {
    // ...
    friendOf: async (root) => {
      const friends = await User.find({
        friends: {
          $in: [root._id]
        } 
      })

      return friends
    }
  },
```

<!-- Sovellus toimii nyt.  -->
Now the application works. 

<!-- Voimme samantien tehdä monimutkaisempiakin kyselyitä. On mahdollista selvittää esim. kaikkien henkilöiden tuttavat: -->
We can immediately do even more complicated queries. It is possible for example to find the friends of all users:

```js
query {
  allPersons {
    name
    friendOf {
      username
    }
  }
}
```

<!-- Sovelluksessa on nyt kuitenkin yksi ongelma, tietokantakyselyjä tehdään kohtuuttoman paljon. Jos lisäämme palvelimen jokaiseen tietokantakyselyn tekevään kohtaan konsoliin tehtävän tulostuksen, huomaamme että jos tietokannassa on viisi henkilöä, tehdään seuraavat tietokantakyselyt:  -->
There is however one issue with our solution, it does an unreasonable amount of queries to the database. If we log every query to the database, and we have 5 persons saved, we see the following:

<pre>
Person.find
User.find
User.find
User.find
User.find
User.find
</pre>

<!-- Eli vaikka pääasiallisesti tehdään ainoastaan yksi kysely, haetaan kaikki henkilöt, aiheuttaa jokainen henkilö yhden kyselyn omassa resolverissaan. -->
So even though we primarily do one query for all persons, every person causes one more query in their resolver. 

<!-- Kyseessä on ilmentymä kuuluisasta [n+1-ongelmasta](https://www.google.com/search?q=n%2B1+problem), joka ilmenee aika ajoin eri yhteyksissä, välillä salakavalastikin sovelluskehittäjän huomaamatta aluksi mitään. -->
This is a manifestation of the famous [n+1-problem](https://www.google.com/search?q=n%2B1+problem), which appears every once in a while in different contexts, and sometimes sneaks up on developers without them noticing. 

<!-- Sopiva ratkaisutapa n+1-ongelmaan riippuu tilanteesta. Usein se edellyttää jonkinlaisen liitoskyselyn tekemistä usean yksittäisen kyselyn sijaan. -->
Good solution for n+1 problem debends on the situation. Often it requires using some kind of a join query instead of multiple separate queries. 

<!-- Tilanteessamme helpoimman ratkaisun toisi se, että tallettaisimme _Person_-olioihin viitteet niistä käyttäjistä kenen ystävälistalla henkilö on: -->
In our situation the easiest solution would be to save whose friends list they are on on each _Person_-object:

```js
const schema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
    minlength: 5
  },
  phone: {
    type: String,
    minlength: 5
  },
  street: {
    type: String,
    required: true,
    minlength: 5
  },  
  city: {
    type: String,
    required: true,
    minlength: 5
  },
  // highlight-start
  friendOf: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    }
  ], 
  // highlight-end
})
```

<!-- Tällöin voisimme tehdä "liitoskyselyn", eli hakiessamme _Person_-oliot, voimme populoida niiden _friendOf_-kentät: -->
Then we could do a "join query", or populate the _friendOf_-fields of persons when we fetch the _Person_-objects:

```js
Query: {
  allPersons: (root, args) => {    
    console.log('Person.find')
    if (!args.phone) {
      return Person.find({}).populate('friendOf') // highlight-line
    }

    return Person.find({ phone: { $exists: args.phone === 'YES' } })
      .populate('friendOf') // highlight-line
  },
  // ...
}
```

<!-- Muutoksen jälkeen erilliselle _friendOf_-kentän resolverille ei enää olisi tarvetta. -->
After the change we would not need a separate resolver for the _friendOf_ field. 

<!-- Kaikkien henkilöiden kysely <i>ei aiheuta</i> n+1-ongelmaa, jos kyselyssä pyydetään esim. ainoastaan nimi ja puhelinnumero: -->
The allPersons query <i>does not cause</i> a n+1 problem, if we i.e only  fetch the name and the phonenumber: 

```js
query {
  allPersons {
    name
    phone
  }
}
```

<!-- Jos kyselyä _allPersons_ muokataan tekemään liitoskysely sen varalta, että se aiheuttaa välillä n+1-ongelman, tulee kyselystä hieman raskaampi niissäkin tapaukisssa, joissa henkilöihin liittyviä käyttäjiä ei tarvita. Käyttämällä resolverifunktioiden [neljättä parametria](https://www.apollographql.com/docs/apollo-server/essentials/data.html#type-signature) olisi kyselyn toteutusta mahdollista optimoida vieläkin pidemmälle. Neljännen parametrin avulla on mahdollista tarkastella itse kyselyä, ja näin liitoskysely voitaisiin tehdä ainoastaan niissä tapauksissa, joissa on n+1-ongelman uhka. Tämänkaltaiseen optimointiin ei toki kannata lähteä ennen kun on varmaa, että se todellakin kannattaa.  -->
If we modify _allPersons_ to do a join query because it sometimes causes n+1 problem, it becomes heavier also when we don't need the information on related persons. By using the [fourth parameter](https://www.apollographql.com/docs/apollo-server/essentials/data.html#type-signature) of resolver functions we could optimize the query even further. The fourth parameter can be used to inspect the query itself, so we could do the join query only in cases with predicted threat for n+1 problem. However we should not jump into this level of optimization before we are sure it's worth it. 

[In the words of Donald Knuth](https://en.wikiquote.org/wiki/Donald_Knuth):

> <i>Programmers waste enormous amounts of time thinking about, or worrying about, the speed of noncritical parts of their programs, and these attempts at efficiency actually have a strong negative impact when debugging and maintenance are considered. We should forget about small efficiencies, say about 97% of the time: <strong>premature optimization is the root of all evil.</strong></i>

<!-- Erään varteenotettavan ratkaisun monien muiden seikkojen lisäksi n+1-ongelmaan tarjoaa  -->
<!-- Facebookin kehittämä [dataloader](https://github.com/facebook/dataloader)-kirjasto, dataloaderin käytöstä Apollo serverin kanssa [täällä](https://www.robinwieruch.de/graphql-apollo-server-tutorial/#graphql-server-data-loader-caching-batching) ja [täällä](http://www.petecorey.com/blog/2017/08/14/batching-graphql-queries-with-dataloader/). -->
[Dataloader](https://github.com/facebook/dataloader)-library by Facebook offers a good solution for the n+1 problem among other issues.
More about using dataloader with Apollo server  [here](https://www.robinwieruch.de/graphql-apollo-server-tutorial/#graphql-server-data-loader-caching-batching) and [here](http://www.petecorey.com/blog/2017/08/14/batching-graphql-queries-with-dataloader/).

### Epiloque

<!-- Tässä osassa rakentamamme sovellus ei ole optimaalisella tavalla strukturoitu: skeeman, kyselyiden ja mutaatioiden määrittely olisi ainakin syytä siirtää muualle sovelluskoodin seasta. Esimerkkejä GraphQL-sovellusten parempaan strukturointiin löytyy internetistä, esim. serveriin -->
<!-- [täältä](https://blog.apollographql.com/modularizing-your-graphql-schema-code-d7f71d5ed5f2) ja clientiin [täältä](https://medium.com/@peterpme/thoughts-on-structuring-your-apollo-queries-mutations-939ba4746cd8). -->
The application we created in this part is not optimally structured: the schema, queries and the muatations should at least be moved outside of the application code. Examples for better structuring of GraphQL applications can be found from the internet, for example for the server
[here](https://blog.apollographql.com/modularizing-your-graphql-schema-code-d7f71d5ed5f2) and the client [here](https://medium.com/@peterpme/thoughts-on-structuring-your-apollo-queries-mutations-939ba4746cd8).

<!-- GraphQL on jo melko iäkäs teknologia, se on ollut Facebookin sisäisessä käytössä jo vuodesta 2012 lähtien, teknologian voi siis todeta olevan "battle tested". Facebook julkaisi GraphQL:n vuonna 2015 ja se on pikkuhiljaa saanut enenevissä määrin huomiota ja nousee ehkä lähivuosina uhmaamaan REST:in valta-asemaa. REST:in [kuolemaakin](https://www.stridenyc.com/podcasts/52-is-2018-the-year-graphql-kills-rest) on jo ennusteltu. Vaikka se ei tulekaan ihan heti tapahtumaan, on GraphQL ehdottomasti [tutustumisen arvoinen](https://blog.graphqleditor.com/javascript-predictions-for-2019-by-npm/). -->
GraphQL is already pretty old technology, being used by Facebook since 2012, so we can see it as "battle tested" already. Since Facebook published GraphQL in 2015 it has slowly gotten more and more attention, and might in the near future threaten the dominance of REST. The death of REST has also already been [predicted](https://www.stridenyc.com/podcasts/52-is-2018-the-year-graphql-kills-rest). Even thought that will not happen quite yet, GraphQL is absolutely worth [learning](https://blog.graphqleditor.com/javascript-predictions-for-2019-by-npm/).

</div>

<div class="tasks">

### Exercises

#### 8.23: Subscriptions - server

<!-- Tee palvelimelle toteutus subscriptiolle _bookAdded_, joka palauttaa tilaajilleen lisättyjen kirjojen tiedot. -->
Do an implementation for subscription _bookAdded_, which returns the details of all new books to its subscribers. 

#### 8.24: Subscriptions - client, part 1

<!-- Ota clientillä käyttöön subscriptiot ja tilaa _bookAdded_. Uusien kirjojen tullessa anna ilmoitus käyttäjälle. Mikä tahansa menetelmä käy, voit käyttää esim. funktiota [window.alert](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert). -->
Start using subscriptions in the client, and subscribe to _bookAdded_. When new books are added, notify the user. Any method works, you can use for example the [window.alert](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert) function. 

#### 8.25: Subscriptions - client, part 2

<!-- Pidä sovelluksen käyttöliittymä ajantasaisena, kun palvelin tiedottaa uusista kirjoista. -->
Keep the applications view updated, when the server notifies about new books. 

#### 8.26: n+1

<!-- Ratkaise haluamallasi menetelmällä seuraavaa kyselyä vaivaava n+1-ongelma: -->
Solve the n+1 problem of the following query using any method you like

```js
query {
  allAuthors {
    name 
    bookCount
  }
}
```

</div>
