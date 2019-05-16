---
mainImage: ../../images/part-8.svg
part: 8
letter: a
---

<div class="content">

<!-- Tälläkin kurssilla moneen kertaan käytetty REST on ollut pitkään vallitseva tapa toteuttaa palvelimen selaimelle tarjoama rajapinta ja yleensäkin verkossa toimivien sovellusten välinen integraatio. -->
REST, familiar to us from the previous parts of the course, has long been the most prevalent way to implement the interfaces servers offer for browsers, and in general the integration between different applications in the web. 

<!-- RESTin rinnalle selaimessa ja mobiililaitteessa toimivan logiikan ja palvelimien väliseen kommunikointiin on viime vuosina noussut alunperin Facebookin kehittämä [GraphQL](http://graphql.org/). -->
In the recent years [GraphQL](http://graphql.org/), developed by Facebook, has become popular for communication between web applications and servers. 

<!-- GraphQL on filosofialtaan todella erilainen RESTiin verrattuna. REST on <i>resurssipohjainen</i>. Jokaisella resurssilla, esim. <i>käyttäjällä</i> on oma sen identifioiva osoite, esim. <i>/users/10</i> ja kaikki resursseille tehtävät operaatiot toteutetaan tekemällä URL:ille kohdistuvia pyyntöjä, joiden toiminta määrittyy käytetyn HTTP-metodin avulla. -->
The GraphQL philosophy is very different from REST. REST is <i>resource based</i>. Every resource, for example a <i>user</i> has its own address which identifies it, for example <i>/users/10</i>. All operations done to the resource are done with HTTP requests to its URL. The action depends on the used HTTP-method. 

<!-- RESTin resurssiperustaisuus toimii hyvin useissa tapauksissa, joissain tapauksissa se voi kuitenkin olla hieman kankea. -->
The resource basedness of REST works well in most situations. However, it can be bit awkward sometimes. 

<!-- Oletetaan että blogilistasovelluksemme sisältäisi somemaista toiminnallisuutta ja haluaisimme esim. näyttää sovelluksessa listan, joka sisältää kaikkien seuraamiemme (follow) käyttäjien blogeja kommentoineiden käyttäjien lisäämien blogien nimet. -->
Lets assume our bloglist application contains social media like functionality, and we would i.e want to show a list of all the blogs the users who have commented on the blogs we follow have added. 

<!-- Jos palvelin toteuttaisi REST API:n, joutuisimme todennäköisesti tekemään monia HTTP-pyyntöjä selaimen koodista, ennen kuin saisimme muodostettua halutun datan. Pyyntöjen vastauksena tulisi myös paljon ylimääräistä dataa ja halutun datan keräävä selaimen koodi olisi todennäköisesti kohtuullisen monimutkainen. -->
If the server implemented a REST API, we would propably have to do multiple HTTP-requests from the browser before we had all the data we wanted. The requests would also return a lot of unnecessary data, and the code on the browser would propably be quite complicated. 

<!-- Jos kyseessä olisi usein käytetty toiminnallisuus, voitaisiin sitä varten toteuttaa oma REST-endpoint. Jos vastaavia skeaarioita olisi paljon, esim. kymmeniä, tulisi erittäin työlääksi toteuttaa kaikille toiminnallisuuksille oma REST-endpoint. -->
If this was an often used functionality, there could be a REST-endpoint for it. If there were a lot of these kind of scenarios however, it would become very laborious to implement REST-endpoints for all of them. 

<!-- GraphQL:n avulla toteutettava palvelin sopii tämänkaltaisiin tilanteisiin hyvin. -->
A GraphQL server is well suited for these kinds of situations. 

<!-- GraphQL:ssä periaatteena on, että selaimen koodi muodostaa <i>kyselyn</i>, joka kuvailee halutun datan ja lähettää sen API:lle HTTP POST -pyynnöllä. Toisin kuin REST:issä, GraphQL:ssä kaikki kyselyt kohdistetaan samaan osoitteeseen ja ovat POST-tyyppisiä. -->
The main princible of GraphQl is, that the code on the browser forms a <i>query</i> describing the data wanted, and sends it to the API with a HTTP POST request. Unlike REST, all GraphQl queries are sent to the same address, and their type is POST. 

<!-- Edellä kuvatun skenaarion data saataisiin haettua (suurinpiirtein) seuraavan kaltaisella kyselyllä: -->
The data described in the above scenario could be fetched with ( roughly ) the following query: 

```bash
query FetchBlogsQuery {
  user(username: "mluukkai") {
    followedUsers {
      blogs {
        comments {
          user {
            blogs {
              title
            }
          }
        }
      }
    }
  }
}
```


<!-- Palvelimen vastaus pyyntöön olisi suunnilleen seuraavanlainen JSON-olio: -->
The servers response would be about the following JSON-object: 

```bash
{
  "data": {
    "followedUsers": [
      {
        "blogs": [
          {
            "comments": [
              {
                "user": {
                  "blogs": [
                    {
                      "title": "Goto considered harmful"
                    },
                    {
                      "title": "End to End Testing with Puppeteer and Jest"
                    },
                    {
                      "title": "Navigating your transition to GraphQL"
                    },
                    {
                      "title": "From REST to GraphQL"
                    }
                  ]
                }
              }
            ]
          }
        ]
      }
    ]
  }
}
```

<!-- Sovelluslogiikka säilyy yksinkertaisena ja selaimen koodi saa täsmälleen haluamansa datan yksittäisellä kyselyllä. -->
The application logic stays simple, and the code on the browser gets exactly the data it needs with a single query. 

### Schemas and queries

<!-- Tutustutaan GraphQL:n peruskäsitteistöön toteuttamalla GraphQL-version osien 2 ja 3 puhelinluettelosovelluksesta. -->
We will get to know the basics of GraphQL by implementing a GraphQL version of the phonebook application from parts 2 and 3. 

<!-- Jokaisen GraphQL-sovelluksen ytimessä on [skeema](https://graphql.org/learn/schema/), joka määrittelee minkä muotoista dataa sovelluksessa vaihdetaan clientin ja palvelimen välillä. Puhelinluettelon alustava skeema on seuraavassa: -->
In the heart of all GraphQL applications is a [schema](https://graphql.org/learn/schema/), which describes the data sent between client and the server. The initial schema for our phonebook is as follows: 

```js
type Person {
  name: String!
  phone: String
  street: String!
  city: String!
  id: ID! 
}

type Query {
  personCount: Int!
  allPersons: [Person!]!
  findPerson(name: String!): Person
}
```

<!-- Skeema määrittelee kaksi [tyyppiä](https://graphql.org/learn/schema/#type-system). Tyypeistä ensimmäinen <i>Person</i> määrittelee, että henkilöillä on viisi kenttää. Kentistä neljä on tyyppiä <i>String</i>, joka on yksi GraphQL:n määrittelemistä [valmiista tyypeistä](https://graphql.org/learn/schema/#scalar-types). String-arvoisista kentistä muilla paitsi puhelinnumerolla (<i>phone</i>) on oltava arvo, tämä on merkitty skeemaan huutomerkillä. Kentän <i>id</i> tyyppi on <i>ID</i>. Arvoltaan <i>ID</i>-tyyppiset kentät ovat merkkijonoja, mutta GraphQL takaa, että ne ovat uniikkeja. -->
The schema describes two [types](https://graphql.org/learn/schema/#type-system). The first type, <i>Person</i>, determines that persons have five fields. Four of the fields are type  <i>String</i>, which is one of the [scalar types](https://graphql.org/learn/schema/#scalar-types) of GraphQL. 
All of the String fields, except <i>phone</i>, must be given a value. This is marked by the exclamation mark on the schema. The type of the field <i>id</i> is <i>ID</i>. <i>ID</i> fields are strings, but GraphQL ensures they are unique.  


<!-- Toinen skeeman määrittelemistä tyypeistä on [Query](https://graphql.org/learn/schema/#the-query-and-mutation-types). Käytännössä jokaisessa GraphQL-skeemassa määritellään tyyppi Query, joka kertoo mitä kyselyjä API:iin voidaan tehdä.  -->
The second type is a [Query](https://graphql.org/learn/schema/#the-query-and-mutation-types). Practically every GraphQL schema describes a Query, which tells what kind of queries can be made to the API. 

<!-- Puhelinluettelo määrittelee kolme erilaista kyselyä ja _personCount_ palauttaa kokonaisluvun. _allPersons_ palauttaa listan <i>Person</i>-tyyppisiä olioita. <i>findPerson</i> saa merkkijonomuotoisen parametrin ja palauttaa <i>Person</i>-olion.  -->
The phonebook describes three different queries. _PersonCount_ returns an integer, _allPersons_ returns a list of <i>Person</i> objects and <i>findPerson</i> is given a string parameter and it returns a <i>Person</i> object. 

<!-- Queryjen paluuarvon ja parametrin määrittelyssä on jälleen käytetty välillä huutomerkkiä merkkaamaan <i>pakollisuutta</i>, eli _personCount_ palauttaa varmasti kokonaisluvun. Kyselylle _findPerson_ on pakko antaa parametriksi merkkijono. Kysely palauttaa <i>Person</i>-olion tai arvon <i>null</i>. _allPersons_ palauttaa listan <i>Person</i>-olioita, listalla ei ole <i>null</i>-arvoja.  -->
Again exclamation marks are used to mark which return values and parameters are <i>Non-Null</i>. _PersonCount_ will, for sure, return a string. The query _findPerson_ must be given a string as a parameter. The query returns a <i>Person</i>-object or <i>null</i>. _AllPersons_ returns a list of <i>Person</i> objects, and the list does not contain any <i>null</i>-values. 

<!-- Skeema siis määrittelee mitä kyselyjä client pystyy palvelimelta tekemään, minkälaisia parametreja kyselyillä voi olla sekä sen, minkä muotoista kyselyjen palauttama data on.  -->
So the schema describes what queries the client can send to the server, what kind of parameters the queries can have, and what kind of data the queries return. 

<!-- Kyselyistä yksinkertaisin _personCount_ näyttää seuraavalta -->
The simplest of the queries, _personCount_, looks as follows: 

```js
query {
  personCount
}
```

<!-- Olettaen että sovellukseen olisi talletettu kolmen henkilön tiedot, vastaus kyselyyn näyttäisi seuraavalta: -->
Assuming our applications has saved the information of three people, the response would look like this: 

```js
{
  "data": {
    "personCount": 3
  }
}
```

<!-- Kaikkien henkilöiden tiedot hakeva _allPersons_ on hieman monimutkaisempi. Koska kysely palauttaa listan <i>Person</i>-olioita, on kyselyn yhteydessä määriteltävä <i>mitkä kentät</i> kyselyn [halutaan palauttavan](https://graphql.org/learn/queries/#fields):  -->
The query fetching the information of all of the people, _allPersons_, is a bit more complicated. Because the query returns a list of <i>Person</i>-objects, the query must describe 
<i>which fields</i> of the objects the query [returns](https://graphql.org/learn/queries/#fields):
```js
query {
  allPersons {
    name
    phone
  }
}
```

<!-- Vastaus voisi näyttää seuraavalta: -->
The response could look like this: 


```js
{
  "data": {
    "allPersons": [
      {
        "name": "Arto Hellas",
        "phone": "040-123543"
      },
      {
        "name": "Matti Luukkainen",
        "phone": "040-432342"
      },
      {
        "name": "Venla Ruuska",
        "phone": null
      }
    ]
  }
}
```

<!-- Kysely voi määritellä palautettavaksi mitkä tahansa skeemassa mainitut kentät, esim. seuraava olisi myös mahdollista: -->
A query can be made to return any field described in the schema. For example the following would also be possible: 

```js
query {
  allPersons{
    name
    city
    street
  }
}
```

<!-- Vielä esimerkki parametria edellyttävästä kyselystä, joka hakee yksittäisen henkilön tiedot palauttavasta kyselystä. -->
The last example shows a query which requires a parameter, and returns the details of one person. 

```js
query {
  findPerson(name: "Arto Hellas") {
    phone 
    city 
    street
    id
  }
}
```

<!-- Kyselyn parametri siis annetaan suluissa, ja sen jälkeen määritellään aaltosuluissa paluuarvona tulevan olion halutut kentät.  -->
So first the parameter is described in round brackets, and then the fields of the return value object are listed in curly brackets. 

<!-- Vastaus on muotoa: -->
The response is like this: 

```js
{
  "data": {
    "findPerson": {
      "phone": "040-123543",
      "city": "Espoo",
      "street": "Tapiolankatu 5 A"
      "id": "3d594650-3436-11e9-bc57-8b80ba54c431"
    }
  }
}
```

<!-- Kyselyn paluuarvoa ei oltu merkitty pakolliseksi, eli jos etsitään tuntematonta henkilöä -->
The return value was not marked as non-Null, so if we search for the details of an unknown

```js
query {
  findPerson(name: "Donald Trump") {
    phone 
  }
}
```

<!-- vastaus on <i>null</i> -->
the return value is <i>null</i>.

```js
{
  "data": {
    "findPerson": null
  }
}
```

<!-- Kuten huomaamme, GraphQL kyselyn ja siihen vastauksena tulevan JSON:in muodoilla on vahva yhteys, voidaan ajatella että kysely kuvailee sen minkälaista dataa vastauksena halutaan. Ero REST:issä tehtäviin pyyntöihin on suuri, REST:iä käytettäessä pyynnon url ja sen tyyppi (GET, POST, PUT, DELETE) ei kerro mitään palautettavan datan muodosta.  -->
As you can see, there is a direct link between a GraphQR query and  the returned JSON object. One can think that the query describes what kind of data it wants as a response. 
The difference to REST queries is stark. With REST, the URL and the type of the request have nothing to do with the form of the return data. 

<!-- GraphQL:n skeema kuvaa ainoastaan palvelimen ja sitä käyttävien clientien välillä liikkuvan tiedon muodon. Tieto voi olla organisoituna ja talletettuna palvelimeen ihan missä muodossa tahansa. -->
GraphQL query describes only the data moving between a server and the client. On the server the data can be organized and saved any way we like. 

<!-- Nimestään huolimatta GraphQL:llä ei itseasiassa ole mitään tekemistä tietokantojen kanssa, se ei ota mitään kantaa siihen miten data on tallennettu. GraphQL-periaattella toimivan API:n käyttämä data voi siis olla talletettu relaatiotietokantaan, dokumenttitietokantaan tai muille palvelimille, joita GraphQL-palvelin käyttää vaikkapa REST:in välityksellä.  -->
Despite of its name, GraphQL does not actually have anything to do with databases. It does not care how the data is saved. 
The data a GraphQL API uses can be saved into a relational database, document database, or to other servers which GraphQL-server can access with for example REST. 

### Apollo server

<!-- Toteuteaan nyt GraphQL-palvelin tämän hetken johtavaa kirjastoa [Apollo -serveriä](https://www.apollographql.com/docs/apollo-server/) käyttäen.  -->
Lets implement a GraphQL-server with today's leading library [Apollo -server](https://www.apollographql.com/docs/apollo-server/).

<!-- Luodaan uusi npm-projekti komennolla _npm init_ ja asennetaan tarvittavat riippuvuuet -->
Create a new npm-project with _npm init_ and install the required dependencies.

```js
npm install --save apollo-server graphql
```

<!-- Alustava toteutus on seuraavassa -->
The initial code is as follows: 

```js
const { ApolloServer, gql } = require('apollo-server')

let persons = [
  {
    name: "Arto Hellas",
    phone: "040-123543",
    street: "Tapiolankatu 5 A",
    city: "Espoo",
    id: "3d594650-3436-11e9-bc57-8b80ba54c431"
  },
  {
    name: "Matti Luukkainen",
    phone: "040-432342",
    street: "Malminkaari 10 A",
    city: "Helsinki",
    id: '3d599470-3436-11e9-bc57-8b80ba54c431'
  },
  {
    name: "Venla Ruuska",
    street: "Nallemäentie 22 C",
    city: "Helsinki",
    id: '3d599471-3436-11e9-bc57-8b80ba54c431'
  },
]

const typeDefs = gql`
  type Person {
    name: String!
    phone: String
    street: String!
    city: String! 
    id: ID!
  }

  type Query {
    personCount: Int!
    allPersons: [Person!]!
    findPerson(name: String!): Person
  }
`

const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) =>
      persons.find(p => p.name === args.name)
  }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
})

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`)
})
```

<!-- Toteutuksen ytimessä on _ApolloServer_, joka saa kaksi parametria  -->
The heart of the code is an _ApolloServer_, which is given two parameters

```js
const server = new ApolloServer({
  typeDefs,
  resolvers,
})
```

<!-- parametreista ensimmäinen _typeDefs_ sisältää sovelluksen käyttämän GraphQL-skeeman.  -->
The first parameter, _typeDefs_, contains the GraphQL schema. 

<!-- Toinen parametri on olio, joka sisältää palvelimen [resolverit](https://www.apollographql.com/docs/apollo-server/essentials/data.html#resolver-map), eli käytännössä koodin, joka määrittelee <i>miten</i> GraphQL-kyselyihin vastataan. -->
The second parameter is an object, which contains the [resolvers](https://www.apollographql.com/docs/apollo-server/essentials/data.html#resolver-map) of the server. These are the code, which defines <i>how</i> GraphQL queries are responded to. 

<!-- Resolverien koodi on seuraavassa: -->
The code of the resolvers is the following: 

```js
const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) =>
      persons.find(p => p.name === args.name)
  }
}
```

<!-- kuten huomataan, vastaavat resolverit rakenteeltaan skeemassa määriteltyjä kyseilyitä: -->
As you can see, the resolvers correspond to the queries described in the schema. 

```js
type Query {
  personCount: Int!
  allPersons: [Person!]!
  findPerson(name: String!): Person
}
```

<!-- eli jokaista skeemassa määriteltyä kyselyä kohti on määritelty oma kentän <i>Query</i> alle tuleva kenttänsä. -->
So there is a field under <i>Query</i> for every query described in the schema. 

<!-- Kyselyn  -->
The query 

```js
query {
  personCount
}
```

<!-- resolveri on funktio -->
Has the resolver

```js
() => persons.length
```

<!-- eli kyselyyn palautetaan vastauksena henkilöt tallentavan taulukon _persons_ pituus.  -->
So the response to the query is the length of the array _persons_.

<!-- Kaikki luettelossa olevat henkilöt hakevan kyselyn  -->
The query which fethches all persons

```js
query {
  allPersons {
    name
  }
}
```

<!-- resolveri on funktio, joka palauttaa <i>kaikki</i> taulukon _persons_ oliot -->
has a resolver which returns <i>all</i> objects from the _persons_ array. 

```js
() => persons
```

### GraphQL-playground

<!-- Kun Apollo -serveriä suoritetaan sovelluskehitysmoodissa, käynnistää se osoitteeseen [http://localhost:4000/graphql](http://localhost:4000/graphql) sovelluskehittäjälle erittäin hyödyllisen [GraphQL-playground](https://www.apollographql.com/docs/apollo-server/features/graphql-playground.html) näkymän, joka avulla on mahdollista tehdä kyselyjä palvelimelle. -->
When Apollo-server is run on development mode, it starts a [GraphQL-playground](https://www.apollographql.com/docs/apollo-server/features/graphql-playground.html) to address [http://localhost:4000/graphql](http://localhost:4000/graphql). This is very useful for a developer, and can be used to make queries to the server. 

<!-- Kokeillaan  -->
Lets try it out

![](../images/8/1.png)

<!-- Playgroundin kanssa pitää olla välillä tarkkana. Jos kysely on syntaktisesti virheellinen, on virheilmoitus aika huomaamaton ja kyselyn suoritusnappia painamalla ei tapahdu mitään: -->
Sometimes the Playground requires you to be quite pedantic. If the syntax of a query is wrong, the error message is quite unnoticeable and nothing happens when you press go. 

![](../images/8/2.png)

<!-- Edellisen kyselyn tulos näkyy edelleen playgroundin oikeassa osassa kyselyn virheellisyydestä huolimatta.  -->
The result from the previous query stays visible on the right side of the playground even when the current query is faulty. 

<!-- Osoittamalla oikeaa kohtaa virheelliseltä riviltä saa virheilmoituksen näkyville -->
By pointing at the right place on the line with the errors, you can see the error message

![](../images/8/3.png)

<!-- Jos Playground vaikuttaa olevan jumissa, niin sivun reloadaaminen yleensä auttaa. -->
If the playground seems to be stuck, refreshing the page usually helps. 

<!-- Klikkaamalla oikean reunan tekstiä <i>schema</i> näyttää Playground palvelimen GraphQL-skeeman. -->
By clicking the text  <i>schema</i> on the right, the playground shows the GraphQL schema of the server. 

![](../images/8/4.png)

### Parameters of a resolver

<!-- Yksittäisen henkilön hakevan kyselyn -->
The query fetching a single person

```js
query {
  findPerson(name: "Arto Hellas") {
    phone 
    city 
    street
  }
}
```

<!-- resolveri on funktio, joka poikkeaa kahdesta aiemmasta resolverista siinä että se saa <i>kaksi parametria</i>: -->
has a resolver which differs from the previous ones because it is given <i>two parameters</i>:

```js
(root, args) => persons.find(p => p.name === args.name)
```

<!-- Parametreista toinen _args_ sisältää kyselyn parametrit. Resolveri siis palauttaa taulukosta -->
The second parameter, _args_, contains the parameters of the query. 
The resolver then returns from the array _persons_ the person whose name is the same as the value of <i>args.name</i>. 
The resolver does not need the first parameter _root_.
 <!-- _persons_ henkilön, jonka nimi on sama kuin <i>args.name</i> arvo. Ensimmäisenä olevaa parametria _root_ resolveri ei tarvitse. -->

 <!-- Itseasiassa kaikki resolverifunktiot saavat [neljä parametria](https://www.apollographql.com/docs/graphql-tools/resolvers.html#Resolver-function-signature). Javascriptissa parametrit voidaan kuitenkin jättää määrittelemättä, jos niitä ei tarvita. Tulemme käyttämään resolverien ensimmäistä ja kolmatta parametria vielä myöhemmin tässä osassa. -->
 In fact all resolver functions are given [four parameters](https://www.apollographql.com/docs/graphql-tools/resolvers.html#Resolver-function-signature). With JavaScript the parameters don't have to be defined, if they are not needed. We will be using the first and the third parameter of a resolver later in this part. 

### The default resolver

<!-- Kun teemme kyselyn, esim -->
When we do a query, for example

```js
query {
  findPerson(name: "Arto Hellas") {
    phone 
    city 
    street
  }
}
```

<!-- osaa palvelin liittää vastaukseen täsmälleen ne kentät, joita kysely pyytää. Miten tämä tapahtuu? -->
the server knows to send back exactly the fields required by the query. How does that happen?

<!-- GraphQL-palvelimen tulee määritellä resolverit <i>jokaiselle</i> skeemassa määritellyn tyypin kentälle. Olemme nyt määritelleet resolverit ainoastaan tyypin <i>Query</i> kentille, eli kaikille sovelluksen tarjoamille kyselyille.  -->
A GraphQL-server must define resolvers for <i>each</i> field of each  type in the schema. 
We have so far only defined resolvers for fields of the type <i>Query</i>, so for each query of the application. 

<!-- Koska skeemassa olevan tyypin <i>Person</i> kentille ei ole määritelty resolvereita, Apollo on määritellyt niille [oletusarvoisen resolverin](https://www.apollographql.com/docs/graphql-tools/resolvers.html#Default-resolver), joka toimii samaan tapaan kuin seuraavassa itse määritelty resolveri: -->
Because we did not define resolvers for the fields of the type <i>Person</i>, Apollo has defined [default resolvers](https://www.apollographql.com/docs/graphql-tools/resolvers.html#Default-resolver) for them. 
They work like the one shown below: 


```js
const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) => persons.find(p => p.name === args.name)
  },
  // highlight-start
  Person: {
    name: (root) => root.name,
    phone: (root) => root.phone,
    street: (root) => root.street,
    city: (root) => root.city,
    id: (root) => root.id
  }
  // highlight-end
}
```

<!-- Oletusarvoinen resolveri siis palauttaa olion vastaavan kentän arvon. Itse olioon se pääsee käsiksi resolverin ensimmäisen parametrin _root_ kautta.  -->
The default resolver returns the value of the corresponding field of the object. The object itself can be accessed through the first parameter of the resolver, _root_.

<!-- Jos oletusarvoisen resolverin toiminnallisuus riittää, ei omaa resolveria tarvitse määritellä. On myös mahdollista määritellä ainoastaan joillekin tyypin yksittäiselle kentille oma resolverinsa ja antaa oletusarvoisen resolverin hoitaa muut kentät. -->
If the functionality of the default resolver is enough, you don't need to define your own. It is also possible to define resolvers for only some fields of a type, and let the default resolvers handle the rest. 

<!-- Voisimme esimerkiksi määritellä, että kaikkien henkilöiden osoitteeksi tulisi <i>Manhattan New York</i> kovakoodaamalla seuraavat tyypin <i>Person</i> kenttien street ja city resolvereiksi: -->
We could for example define, that the address of all persons is 
<i>Manhattan New York</i> by hard coding the following to the resolvers of the street and city fields of the type <i>Person</i>.

```js
Person: {
  street: (root) => "Manhattan",
  city: (root) => "New York"
}
```

### Object within an object

<!-- Muutetaan skeemaa hiukan -->
Lets modify the scheme a bit

```js
  // highlight-start
type Address {
  street: String!
  city: String! 
}
  // highlight-end

type Person {
  name: String!
  phone: String
  address: Address!   // highlight-line
  id: ID!
}

type Query {
  personCount: Int!
  allPersons: [Person!]!
  findPerson(name: String!): Person
}
```

<!-- eli henkilöllä on nyt kenttä, jonka tyyppi on <i>Address</i>, joka koostuu kadusta ja kaupungista.  -->
so a person now has a field with the type <i>Address</i>, which contains the street and the city. 

<!-- Osoitetta tarvitsevat kyselyt muuttuvat muotoon -->
The queries requiring the address change into

```js
query {
  findPerson(name: "Arto Hellas") {
    phone 
    address {
      city 
      street
    }
  }
}
```

<!-- vastauksena on henkilö-olio, joka <i>sisältää</i> osoite-olion: -->
and the response now is an person object, which <i>contains</i> an address object. 

```js
{
  "data": {
    "findPerson": {
      "phone": "040-123543",
      "address":  {
        "city": "Espoo",
        "street": "Tapiolankatu 5 A"
      }
    }
  }
}
```

<!-- Talletetaan henkilöt palvelimella edelleen samassa muodossa kuin aiemmin. -->
We still save the persons in the server the same way we did before. 

```js
let persons = [
  {
    name: "Arto Hellas",
    phone: "040-123543",
    street: "Tapiolankatu 5 A",
    city: "Espoo",
    id: "3d594650-3436-11e9-bc57-8b80ba54c431"
  },
  // ...
]
```

<!-- Nyt siis palvelimen tallettamat henkilö-oliot eivät ole muodoltaan täysin samanlaisia kuin GraphQL-skeeman määrittelemät tyypin <i>Person</i> -oliot.  -->
Sp the person-objects saved in the server are not exactly the same as GraphQL type <i>Person</i> objects described in the schema. 

<!-- Toisin kuin tyypille <i>Person</i> ei tyypille <i>Address</i> ole määritelty <i>id</i>-kenttää, sillä osoitteita ei ole talletettu palvelimella omaan tietorakenteeseensa. -->
Contrary to the type <i>Person</i>, the <i>Address</i> type does not have a <i>id</i>-field, because they are not saved into their own data structure in the server. 

<!-- Koska taulukkoon talletetuilla olioilla ei ole kenttää <i>address</i> oletusarvoinen resolveri ei enää riitä. Lisätään resolveri tyypin <i>Person</i> kentälle <i>address</i>: -->
Because the objects saved in the array do not have a field <i>address</i>, the default resolver is not sufficient enough. 
Lets add a resolver for the field <i>address</i> of type <i>Person</i>: 

```js
const resolvers = {
  Query: {
    personCount: () => persons.length,
    allPersons: () => persons,
    findPerson: (root, args) =>
      persons.find(p => p.name === args.name)
  },
  // highlight-start
  Person: {
    address: (root) => {
      return { 
        street: root.street,
        city: root.city
      }
    }
  }
  // highlight-end
}
```

<!-- Eli aina palautettaessa <i>Person</i>-oliota, palautetaan niiden kentät <i>name</i>, <i>phone</i> sekä <i>id</i> käyttäen oletusarvoista resolveria, kenttä <i>address</i> muodostetaan itse määritellyn resolverin avulla. Resolverifunktion parametrina _root_ on käsittelyssä oleva henkilö-olio, eli osoitteen katu ja kaupunki saadaan sen kentistä. -->
So every time a <i>Person</i> object is returned, the fields <i>name</i>, <i>phone</i> and <i>id</i> are returned using their default resolvers, but the field <i>address</i> is formed by using a self defined resolver. The parameter _root_ of the resolver function is the person-object, so the street and the city of the address can be taken from its fields. 

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-1), branchissa <i>part8-1</i>. -->
The current code of the application can be found from [ github](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-1), branch <i>part8-1</i>.

### Mutations

<!-- Laajennetaan sovellusta siten, että puhelinluetteloon on mahdollista lisätä uusia henkilöitä. GraphQL:ssä kaikki muutoksen aiheuttavat operaatiot tehdään [mutaatioiden](https://graphql.org/learn/queries/#mutations) avulla. Mutaatiot määritellään skeemaan tyypin <i>Mutation</i> avaimina. -->
Lets add functionality for adding new persons to the phonebook. In GraphQL, all operations which cause a change are done with [mutations](https://graphql.org/learn/queries/#mutations). Mutations are described in the schema as the keys of type <i>Mutation</i>.

<!-- Käyttäjän lisäävä mutaation skeema näyttää seuraavalta -->
The schema for a mutation for adding a new person looks as follows: 

```js
type Mutation {
  addPerson(
    name: String!
    phone: String
    street: String!
    city: String!
  ): Person
}
```

<!-- Mutaatio siis saa parametreina käyttäjän tiedot. Parametreista <i>phone</i> on ainoa, jolle ei ole pakko asettaa arvoa. Mutaatioilla on parametrien lisäksi paluuarvo. Paluuarvo on nyt tyyppiä <i>Person</i>, ideana on palauttaa operaation onnistuessa lisätyn henkilön tiedot ja muussa tapauksessa <i>null</i>. Mutaatiossa ei anneta parametrina kentän <i>id</i> arvoa, sen luominen on parempi jättää palvelimen vastuulle. -->
The Mutation is given the details of the person as parameters. The parameter <i>phone</i> is the only one which is not non-null. The Mutation also has a return value. The return value is type <i>Person</i>, the idea being that the details of the added person are returned is the operation is successfull and if not, null. Value for the field <i>id</i> is not given as a parameter. Generating an id is better left for the server. 

<!-- Myös mutaatioita varten on määriteltävä resolveri: -->
Mutations also require a resolver: 

```js
const uuid = require('uuid/v1')

// ...

const resolvers = {
  // ...
  Mutation: {
    addPerson: (root, args) => {
      if (persons.find(p => p.name === args.name)) {
        throw new UserInputError('Name must be unique', {
          invalidArgs: args.name,
        })
      }

      const person = { ...args, id: uuid() }
      persons = persons.concat(person)
      return person
    }
  }
}
```

<!-- Mutaatio siis lisää parametreina _args_ saamansa olion taulukkoon _persons_ ja palauttaa lisätyn olion.  -->
The mutation adds the object given to it as a parameter _args_ to the array _persons_, and returns the object it added to the array. 

<!-- Kentälle <i>id</i> saadaan luotua uniikki tunniste kirjaston [uuid](https://github.com/kelektiv/node-uuid#readme) avulla.  -->
The <i>id</i> field is given an unique value using the [uuid](https://github.com/kelektiv/node-uuid#readme) library. 

<!-- Uusi henkilö voidaan lisätä seuraavalla mutaatiolla -->
A new person can be added with the following mutation

```js
mutation {
  addPerson(
    name: "Pekka Mikkola"
    phone: "045-2374321"
    street: "Vilppulantie 25"
    city: "Helsinki"
  ) {
    name
    phone
    address{
      city
      street
    }
    id
  }
}
```

<!-- Kannattaa huomata, että lisättävä henkilö talletetaan taulukkoon _persons_ muodossa -->
Note, that the person is saved to the _persons_ array as 

```js
{
  name: "Pekka Mikkola",
  phone: "045-2374321",
  street: "Vilppulantie 25",
  city: "Helsinki",
  id: "2b24e0b0-343c-11e9-8c2a-cb57c2bf804f"
}
```

<!-- Vastaus mutaatioon on kuitenkin -->
But the response to the mutation is 

```js
{
  "data": {
    "addPerson": {
      "name": "Pekka Mikkola",
      "phone": "045-2374321",
      "address": {
        "city": "Helsinki",
        "street": "Vilppulantie 25"
      },
      "id": "2b24e0b0-343c-11e9-8c2a-cb57c2bf804f"
    }
  }
}
```

<!-- eli tyypin <i>Person</i> kentän <i>address</i> resolveri muotoilee vastauksena palautettavan olion oikean muotoiseksi. -->
So the resolver of the <i>address</i> field of the <i>Person</i> type formats the response object to the right form. 

### Error handling

<!-- Jos yritämme luoda uuden henkilön, mutta parametrit eivät vastaa skeemassa määriteltyä (esim. katuosoite puuttuu), antaa palvelin virheilmoituksen:  -->
If we try to create a new person, but the parameters do not correspond with the schema description, the server gives a error message: 

![](../images/8/5.png)

<!-- GraphQL:n [validoinnin](https://graphql.org/learn/validation/) avulla pystytään siis jo automaattisesi hoitamaan osa virheenkäsittelyä.  -->
So some of the error handling can be automatically with graphQL [validation](https://graphql.org/learn/validation/).

<!-- Kaikkea GraphQL ei kuitenkaan pysty hoitamaan automaattisesti. Esimerkiksi tarkemmat säännöt mutaatiolla lisättävän datan kenttien muodolle on lisättävä itse. Niistä aiheutuvat virheet tulee hoitaa [GraphQL:n poikkeuskäsittelymekanismilla](https://www.apollographql.com/docs/apollo-server/features/errors.html). -->
However GraphQL cannot handle everything automatically. For example stricter rules for data sent to a Mutation has to be added by oneself. 
The errors from those rules are handled by [the error handling mechanism of GraphQL](https://www.apollographql.com/docs/apollo-server/features/errors.html).

<!-- Estetään saman nimen lisääminen puhelinluetteloon useampaan kertaan: -->
Lets block adding the same name to the phonebook multiple times: 

```js
const { ApolloServer, UserInputError, gql } = require('apollo-server') // highlight-line

// ...

const resolvers = {
  // ..
  Mutation: {
    addPerson: (root, args) => {
      // highlight-start
      if (persons.find(p => p.name === args.name)) {
        throw new UserInputError('Name must be unique', {
          invalidArgs: args.name,
        })
      }
      // highlight-end

      const person = { ...args, id: uuid() }
      persons = persons.concat(person)
      return person
    }
  }
}
```

<!-- Eli jos lisättävä nimi on jo luettelossa heitetään poikkeus _UserInputError_. -->
So if the name to be added already exists in the phonebook, throw _UserInputError_ error. 

![](../images/8/6.png)

<!-- Sovelluksen tämänhetkinen koodi on kokonaisuudessaan [githubissa](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-2), branchissa <i>part8-2</i>. -->
The current code of the application can be found from [ github](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-1), branch <i>part8-2</i>.

### Enum

<!-- Tehdään sovellukseen vielä sellainen lisäys, että kaikki henkilöt palauttavaa kyselyä voidaan säädellä parametrilla <i>phone</i> siten, että kysely palauttaa vain henkilöt, joilla on puhelinnumero -->
Lets add a possibility to filter the query returning all persons with the parameter <i>phone</i> so, that it returns only persons with a phone number

```js
query {
  allPersons(phone: YES) {
    name
    phone 
  }
}
```

<!-- tai henkilöt, joilla ei ole puhelinnumeroa -->
or persons without a phonenumber 

```js
query {
  allPersons(phone: NO) {
    name
  }
}
```

<!-- Skeema laajenee seuraavasti: -->
The schema changes like so: 

```js
// highlight-start
enum YesNo {
  YES
  NO
}
// highlight-end

type Query {
  personCount: Int!
  allPersons(phone: YesNo): [Person!]! // highlight-line
  findPerson(name: String!): Person
}
```

<!-- Tyyppi <i>YesNo</i> on GraphQL:n [enum](https://graphql.org/learn/schema/#enumeration-types), eli lueteltu tyyppi, jolla on kaksi mahdollista arvoa, <i>YES</i> ja <i>NO</i>. Kyselyssä _allPersons_ parametri _phone_ on tyypiltään <i>YesNo</i>, mutta sen arvo ei ole pakollinen. -->
The type <i>YesNo</i> is GraphQL [enum](https://graphql.org/learn/schema/#enumeration-types), or an enumerable, with two possible values <i>YES</i> or <i>NO</i>. In the query _allPersons_ the parameter _phone_  has the type <i>YesNo</i>, but is not non-null. 

<!-- Resolveri muuttuu seuraavasti -->
the resolver changes like so

```js
Query: {
  personCount: () => persons.length,
  // highlight-start
  allPersons: (root, args) => {
    if (!args.phone) {
      return persons
    }

    const byPhone = (person) =>
      args.phone === 'YES' ? person.phone : !person.phone

    return persons.filter(byPhone)
  },
  // highlight-end
  findPerson: (root, args) =>
    persons.find(p => p.name === args.name)
},
```

### Changing a phonenumber

<!-- Tehdään sovellukseen myös mutaatio, joka mahdollistaa henkilön puhelinnumeron muuttamisen. Mutaation skeema näyttää seuraavalta -->
Lets add a mutation for changing the phonenumber of a person. The schema of this mutation looks as follows

```js
type Mutation {
  addPerson(
    name: String!
    phone: String
    street: String!
    city: String!
  ): Person
  // highlight-start
  editNumber(
    name: String!
    phone: String!
  ): Person
  // highlight-end
}
```

<!-- ja sen toteuttaa seuraava resolveri: -->
and is done by a resolver:

```js
Mutation: {
  // ...
  editNumber: (root, args) => {
    const person = persons.find(p => p.name === args.name)
    if (!person) {
      return null
    }

    const updatedPerson = { ...person, phone: args.phone }
    persons = persons.map(p => p.name === args.name ? updatedPerson : p)
    return updatedPerson
  }   
}
```

<!-- Mutaatio hakee siis hakee kentän <i>name</i> perusteella henkilön, jonka numero päivitetään. -->
The mutation finds the person to be updated person by the field <i>name</i>.

The current code of the application can be found from [ github](https://github.com/fullstack-hy2019/graphql-phonebook-backend/tree/part8-1), branch <i>part8-3</i>.

### More on queries

<!-- GraphQL:ssä on yhteen kyselyyn mahdollista yhdistää monia tyypin <i>Query</i> kenttiä, eli "yksittäisiä kyselyitä". Esim. seuraava kysely palautta puhelinluettelon henkilöiden lukumäärän sekä nimet: -->
With GraphQl it is possible to combine multiple fields of type <i>Query</i>, or "separate queries" into one query. For example the following query returns both the amount of persons in the phonebook and their names: 

```js
query {
  personCount
  allPersons {
    name
  }
}
```

<!-- Vastaus näyttää seuraavalta -->
The response looks as follows

```js
{
  "data": {
    "personCount": 3,
    "allPersons": [
      {
        "name": "Arto Hellas"
      },
      {
        "name": "Matti Luukkainen"
      },
      {
        "name": "Venla Ruuska"
      }
    ]
  }
}
```

<!-- Yhdistetty kysely voi myös viitata useampaan kertaan samaan kyselyyn. Tällöin erillisille kyselyille on kuitenkin annettava vaihtoehtoiset nimet kaksoispistesyntaksilla -->
Combined query can also use the same query multiple times. You must however give the queries alternative names like so

```js
query {
  havePhone: allPersons(phone: YES){
    name
  }
  phoneless: allPersons(phone: NO){
    name
  }
}
```

<!-- Vastaus on muotoa -->
The response looks like

```js
{
  "data": {
    "havePhone": [
      {
        "name": "Arto Hellas"
      },
      {
        "name": "Matti Luukkainen"
      }
    ],
    "phoneless": [
      {
        "name": "Venla Ruuska"
      }
    ]
  }
}
```

<!-- Joissain tilanteissa voi myös olla hyötyä nimetä kyselyt. Näin on erityisesti tilanteissa, joissa kyselyillä tai mutaatiolla on [parametreja](https://graphql.org/learn/queries/#variables). Tutustumme parametreihin pian. -->
In some cases it might be beneficial to name the queries. This is the case especially when the queries or mutations have [parameters](https://graphql.org/learn/queries/#variables). We will go into parameters soon. 

<!-- Jos kyselyitä on useita, pyytää Playground valitsemaan mikä niistä suoritetaan: -->
If there are multiple queries, Playground asks you to choose which of them to run:

![](../images/8/7.png)

</div>

<div class="tasks">

### Exercises

<!-- Tehtävissä toteutetaan yksinkertaisen kirjaston GraphQL:ää tarjoava backend. Ota sovelluksesi lähtökohtaksi [tämä tiedosto](https://github.com/fullstack-hy2019/misc/blob/master/library-backend.js). Muista _npm init_ ja riippuvuuksien asentaminen! -->
Through the exercises we implement a GraphQl backend for a small library. 
Start with [this file](https://github.com/fullstack-hy2019/misc/blob/master/library-backend.js). Remember _npm init_ and to install dependencies!

#### 8.1: The number of books and authors

<!-- Toteuta kyselyt _bookCount_ ja _authorCount_ jotka palauttavat kirjojen ja kirjailijoiden lukumäärän. -->
Implement queries _bookCount_ and _authorCount_ which return the number of books and the number of authors. 

The query 

```js
query {
  bookCount
  authorCount
}
```

<!-- pitäisi alustavalla datalla tuottaa vastaus -->
should return

```js
{
  "data": {
    "bookCount": 7,
    "authorCount": 5
  }
}
```

#### 8.2: All books 

<!-- Toteuta kysely _allBooks_,  joka palauttavat kaikki kirjat. -->
Implement query _allBooks_, which returns the details of all books. 

<!-- Seurava kysely siis pitäisi pystyä tekemään -->
In the end user should be able to do the following query

```js
query {
  allBooks { 
    title 
    author
    published 
    genres
  }
}
```

#### 8.3: All authors

<!-- Toteuta kysely _allAuthors_ joka palauttaa kaikki kirjailijat. Kyselyn vastauksessa kirjailijoilla tulee myös olla kenttä _bookCount_ joka kertoo kirjailijan tekemien kirjojen määrän. -->
Implement query _allAuthors_, which returns the details of all authors. The response should include a field _bookCount_ containing the number of books the author has written. 

<!-- Esim. kyselyn -->
For example the query

```js
query {
  allAuthors {
    name
    bookCount
  }
}
```

<!-- vastauksen tulisi näyttää seuraavalta -->
should return

```js
{
  "data": {
    "allAuthors": [
      {
        "name": "Robert Martin",
        "bookCount": 2
      },
      {
        "name": "Martin Fowler",
        "bookCount": 1
      },
      {
        "name": "Fyodor Dostoevsky",
        "bookCount": 2
      },
      {
        "name": "Joshua Kerievsky",
        "bookCount": 1
      },
      {
        "name": "Sandi Metz",
        "bookCount": 1
      }
    ]
  }
}
```

#### 8.4: Books of an author

<!-- Laajenna kyselyä _allBooks_ siten, että sille voi antaa optionaalisen parametrin <i>author</i>, joka rajoittaa kirjalistan niihin, joiden author on parametrina annettu kirjailija. -->
Modify the _allBooks_ query so, that user can give an optional parameter <i>author</i>. The response should include only books written by that author. 

For example query

```js
query {
  allBooks(author: "Robert Martin") {
    title
  }
}
```

<!-- tulisi palauttaa -->
should return

```js
{
  "data": {
    "allBooks": [
      {
        "title": "Clean Code"
      },
      {
        "title": "Agile software development"
      }
    ]
  }
}
```

#### 8.5: Books by genre

<!-- Laajenna kyselyä _allBooks_ siten, että sille voi antaa optionaalisen parametrin <i>genre</i>, joka rajoittaa kirjalistan niihin, joiden genrejen joukossa on parametrina annettu genre. -->
Modify the query _allBooks_ so, that user can give optional parameter <i>genre</i>. The response should include only books of that genre. 

<!-- Esim. kyselyn -->
For example query

```js
query {
  allBooks(genre: "refactoring") {
    title
    author
  }
}
```

<!-- tulisi palauttaa -->
should return

```js
{
  "data": {
    "allBooks": [
      {
        "title": "Clean Code",
        "author": "Robert Martin"
      },
      {
        "title": "Refactoring, edition 2",
        "author": "Martin Fowler"
      },
      {
        "title": "Refactoring to patterns",
        "author": "Joshua Kerievsky"
      },
      {
        "title": "Practical Object-Oriented Design, An Agile Primer Using Ruby",
        "author": "Sandi Metz"
      }
    ]
  }
}
```

<!-- Kyselyn pitää toimia myös siinä tapauksessa, että se saa molemmat optionaaliset parametrit: -->
The query must work when both optional parameters are given: 

```js
query {
  allBooks(author: "Robert Martin", genre: "refactoring") {
    title
    author
  }
}
```

#### 8.6: Adding a book

<!-- Toteuta mutaatio _addBook_, jota voi käyttää seuraavasti -->
Implement mutation _addBook_, which can be used like this:

```js
mutation {
  addBook(
    title: "NoSQL Distilled",
    author: "Martin Fowler",
    published: 2012,
    genres: ["database", "nosql"]
  ) {
    title,
    author
  }
}
```

<!-- Mutaatio toimii myös niissä tilanteissa, joissa kirjoittaja ei ole ennestään palvelimen tiedossa: -->
The mutation works even if the author is not already saved to the server:

```js
mutation {
  addBook(
    title: "Pimeyden tango",
    author: "Reijo Mäki",
    published: 1997,
    genres: ["crime"]
  ) {
    title,
    author
  }
}
```

<!-- Jos näin on, lisätään uusi kirjailija järjestelmään. Kirjailijan syntymävuodesta ei ole tässä vaiheessa tietoa, eli kysely -->
If the author is not yet saved to the server, a new author is added to the system. The birth years of authors are not saved to the server yet, so the query

```js
query {
  allAuthors {
    name
    born
    bookCount
  }
}
```

<!-- palauttaa -->
returns

```js
{
  "data": {
    "allAuthors": [
      // ...
      {
        "name": "Reijo Mäki",
        "born": null,
        "bookCount": 1
      }
    ]
  }
}
```

#### 8.7: Updating the birth year of an author

<!-- Toteuta mutaatio _editAuthor_ jonka avulla on mahdollista asettaa kirjailijalle syntymävuosi. Mutaatiota käytetään seuraavasti -->
Implement mutation _editAuthor_, which can be used to set a birth year for an author. The mutation is used like so

```js
mutation {
  editAuthor(name: "Reijo Mäki", setBornTo: 1958) {
    name
    born
  }
}
```

<!-- Jos kirjailija löytyy, palauttaa operaatio editoidun kirjailijan: -->
If the correct author is found, the operation returns the edited author:

```js
{
  "data": {
    "editAuthor": {
      "name": "Reijo Mäki",
      "born": 1958
    }
  }
}
```

<!-- Olemattoman kirjailijan syntymävuoden editointiin reagoidaan palauttamalla <i>null</i>: -->
If the author is not in the system, <i>null</i> is returned: 

```js
{
  "data": {
    "editAuthor": null
  }
}
```

</div>
