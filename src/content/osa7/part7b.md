---
mainImage: ../../images/part-7.svg
part: 7
letter: b
---

<div class="content">

<!-- Osassa 2 on jo katsottu kahta tapaa tyylien lisäämiseen eli vanhan koulukunnan [yksittäistä CSS](/osa2#tyylien-lisääminen)-tiedostoa, [inline-tyylejä](/osa6#inline-tyylit). Katsotaan tässä osassa vielä muutamaa tapaa. -->
In part 2 we examined two different ways of adding styles to our application: the old-school [single CSS](/osa2#tyylien-lisääminen) file and [inline-styles](/osa6#inline-tyylit). In this part we will take a look at a few other ways. 

<!-- ### Valmiit käyttöliittymätyylikirjastot -->
### Ready-made UI libraries

<!-- Eräs lähestymistapa sovelluksen tyylien määrittelyyn on valmiin "UI frameworkin", eli suomeksi ehkä käyttöliittymätyylikirjaston käyttö. -->
One approach to defining styles for an application is to use a ready-made "UI framework".

<!-- Ensimmäinen laajaa kuuluisuutta saanut UI framework oli Twitterin kehittämä [Bootstrap](https://getbootstrap.com/), joka lienee edelleen UI frameworkeista eniten käytetty. Viime aikoina UI frameworkeja on noussut kuin sieniä sateella. Valikoima on niin iso, ettei tässä kannata edes yrittää tehdä tyhjentävää listaa. -->
One of the first widely popular UI frameworks was the [Bootstrap](https://getbootstrap.com/) toolkit created by Twitter, that may still be the most popular framework. Recently there has been an explosion in the number of new UI frameworks that have entered the arena. In fact, the selection is so vast that there is little hope of creating an exhaustive list of options.

<!-- Monet UI-frameworkit sisältävät web-sovellusten käyttöön valmiiksi määriteltyjä teemoja sekä "komponentteja", kuten painikkeita, menuja, taulukkoja. Termi komponentti on edellä kirjotettu hipsuissa sillä kyse ei ole samasta asiasta kuin React-komponentti. Useimmiten UI-frameworkeja käytetään sisällyttämällä sovellukseen frameworkin määrittelemät CSS-tyylitiedostot sekä Javascript-koodi. -->
Many UI frameworks provide developers of web applications with ready-made themes and "components" like buttons, menus, and tables. We write components in quotes as in this context we are not talking about React components. Usually UI frameworks are used by including the CSS stylesheets and JavaScript code of the framework with the application.

<!-- Monesta UI-frameworkista on tehty React-ystävällisiä versiota, joissa UI-frameworkin avulla määritellyistä "komponenteista" on tehty React-komponentteja. Esim. Bootstrapista on olemassa parikin React-versiota [reactstrap](http://reactstrap.github.io/) ja [react-bootstrap](https://react-bootstrap.github.io/). -->
There are React-friendly versions available of many UI frameworks, where the framework's "components" have been transformed into React components. There are a few different React versions of Boostrap like [reactstrap](http://reactstrap.github.io/) and [react-bootstrap](https://react-bootstrap.github.io/).

<!-- Katsotaan seuraavaksi kahta UI-framworkia bootstrapia ja [semantic ui](https://semantic-ui.com/):ta. Lisätään molempien avulla samantapaiset tyylit luvun [React-router](/osa7/react_router) sovellukseen. -->
Next we will take a closer look at two UI frameworks, Bootstrap and [Semantic UI](https://semantic-ui.com/). We will use both frameworks to add similar styles to the application we made in the [React-router](/osa7/react_router) section of the course material.

<!-- ### react bootstrap -->
### React Bootstrap

<!-- Aloitetaan bootstrapista, käytetään kirjastoa [react-bootstrap](https://react-bootstrap.github.io/). -->
Let's start by taking a look at Bootstrap with the help of the [react-bootstrap](https://react-bootstrap.github.io/) package.

<!-- Asennetaan kirjasto suorittamalla komento -->
Let's install the package with the command:

```js
npm install --save react-bootstrap
```

<!-- Lisätään sitten sovelluksen tiedostoon <i>public/index.html</i> tagin <i>head</i> sisään bootstrapin css-määrittelyt lataava rivi: -->
Then let's add a link for loading the CSS stylesheet for Bootstrap inside of the <i>head</i> tag in the <i>public/index.html</i> file of the application:

```js
<head>
<link
  rel="stylesheet"
  href="https://maxcdn.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css"
  integrity="sha384-GJzZqFGwb1QTTN6wy59ffF1BuGJpLSa9DkKMp0DgiMDm4iYMj70gZWKYbI706tWS"
  crossorigin="anonymous"
/>
  // ...
</head>
```

<!-- Kun sovellus ladataan uudelleen, näyttää se jo aavistuksen tyylikkäämmältä: -->
When we reload the application we notice that it already looks a bit more stylish:

![](../images/7/5.png)

<!-- Bootstrapissa koko sivun sisältö renderöidään yleensä [container](https://getbootstrap.com/docs/4.1/layout/overview/#containers):ina, eli käytännössä koko sovelluksen ympäröivä _div_-elementti merkitään luokalla _container_: -->
In Bootstrap, all of the content of the application is typically rendered inside of a [container](https://getbootstrap.com/docs/4.1/layout/overview/#containers). In practice this is accomplished by giving the root _div_ element of the application the  _container_ class attribute:

```js
const App = () => {
  // ...

  return (
    <div class="container"> // highlight-line
      // ...
    </div>
  )
}
```

<!-- Sovelluksen ulkoasu muuttuu siten, että sisältö ei ole enää yhtä kiinni selaimen reunoissa: -->
We notice that this already has an effect on the appearance of the application. The content is no longer as close to the edges of the browser as it was earlier:

![](../images/7/6.png)

<!-- Muutetaan seuraavaksi komponenttia <i>Notes</i> siten, että se renderöi muistiinpanojen listan [taulukkona](https://getbootstrap.com/docs/4.1/content/tables/). React bootstrap tarjoaa valmiin komponentin [Table](https://react-bootstrap.github.io/components/table/), joten CSS-luokan käyttöön ei ole tarvetta. -->
Next, let's make some changes to the <i>Notes</i> component so that it renders the list of notes as a [table](https://getbootstrap.com/docs/4.1/content/tables/). React Bootstrap provides a built-in [Table](https://react-bootstrap.github.io/components/table/) component for this purpose, so there is no need to define CSS classes separately.

```js
const Notes = (props) => (
  <div>
    <h2>Notes</h2>
    <Table striped>
      <tbody>
        {props.notes.map(note =>
          <tr key={note.id}>
            <td>
              <Link to={`/notes/${note.id}`}>
                {note.content}
              </Link>
            </td>
            <td>
              {note.user}
            </td>
          </tr>
        )}
      </tbody>
    </Table>
  </div>
)
```

<!-- Ulkoasu on varsin tyylikäs: -->
The appearance of the application is quite stylish:

![](../images/7/7.png)

<!-- Huomaa, että koodissa käytettävät React bootstrapin komponentit täytyy importata, eli koodiin on lisättävä: -->
Notice that the React Bootstrap components have to be imported separately from the library as shown below:

```js
import { Table } from 'react-bootstrap'
```

<!-- #### Lomake -->
#### Forms

<!-- Parannellaan seuraavaksi näkymän <i>Login</i> kirjautumislomaketta Bootstrapin [lomakkeiden](https://getbootstrap.com/docs/4.1/components/forms/) avulla. -->
Let's improve the form in the <i>Login</i> view with the help of Bootstrap [forms](https://getbootstrap.com/docs/4.1/components/forms/).

<!-- React bootstrap tarjoaa valmiit [komponentit](https://react-bootstrap.github.io/components/forms/) myös lomakkeiden muodostamiseen (dokumentaatio tosin ei ole paras mahdollinen): -->
React Bootstrap provides built-in [components](https://react-bootstrap.github.io/components/forms/) for creating forms (although the documentation for them is slightly lacking):

```js
let Login = (props) => {
  // ...
  return (
    <div>
      <h2>login</h2>
      <Form onSubmit={onSubmit}>
        <Form.Group>
          <Form.Label>username:</Form.Label>
          <Form.Control
            type="text"
            name="username"
          />
          <Form.Label>password:</Form.Label>
          <Form.Control
            type="password"
          />
          <Button variant="primary" type="submit">
            login
          </Button>
        </Form.Group>
      </Form>
    </div>
)}
```

<!-- Importoitavien komponenttien määrä kasvaa: -->
The number of components we need to import increases:

```js
import { Table, Form, Button } from 'react-bootstrap'
```

<!-- Lomake näyttää parantelun jälkeen seuraavalta: -->
After switching over to the Bootstrap form, our improved application looks like this:

![](../images/7/8.png)

<!-- #### Notifikaatio -->
#### Notification

<!-- Toteutetaan sovellukseen kirjautumisen jälkeinen <i>notifikaatio</i>: -->
Now that the login form is in better shape, let's take a look at improving our application's notifications:

![](../images/7/9.png)

<!-- Asetetaan notifikaatio kirjautumisen yhteydessä komponentin <i>App</i> tilan muuttujaan _message_: -->
Let's add a message for the notification when a user logs in to the application. We will store it in the _message_ variable in the <i>App</i> component's state:

```js
const App = () => {
  const [notes, setNotes] = useState([
    // ...
  ])

  const [user, setUser] = useState(null)
  const [message, setMessage] = useState(null) // highlight-line

  const login = (user) => {
    setUser(user)
    // highlight-start
    setMessage(`welcome ${user}`)
    setTimeout(() => {
      setMessage(null)
    }, 10000)
    // highlight-end
  }
  // ...
}
```

<!-- ja renderöidään viesti Bootstrapin [Alert](https://getbootstrap.com/docs/4.1/components/alerts/)-komponentin avulla. React bootstrap tarjoaa tähän jälleen valmiin [React-komponentin](https://react-bootstrap.github.io/components/alerts/): -->
We will render the message as a Bootstrap [Alert](https://getbootstrap.com/docs/4.1/components/alerts/) component. Once again, the React Bootstrap library provides us with a matching [React component](https://react-bootstrap.github.io/components/alerts/): 

```js
<div className="container">
  <Router>
    <div>
    // highlight-start
      {(message &&
        <Alert variant="success">
          {message}
        </Alert>
      )}
      // highlight-end
    //...
)}
```

<!-- #### Navigaatiorakenne -->
#### Navigation structure

<!-- Muutetaan vielä lopuksi sovelluksen navigaatiomenu käyttämään Bootstrapin [Navbaria](https://getbootstrap.com/docs/4.1/components/navbar/). Tähänkin React bootstrap tarjoaa [valmiit komponentit](https://react-bootstrap.github.io/components/navbar/#navbars-mobile-friendly), dokumentaatio on hieman kryptistä, mutta trial and error johtaa lopulta toimivaan ratkaisuun: -->
Lastly, let's alter the application's navigation menu to use Bootstrap's [Navbaria](https://getbootstrap.com/docs/4.1/components/navbar/) component. The React Bootstrap library provides us with [matching built-in components](https://react-bootstrap.github.io/components/navbar/#navbars-mobile-friendly). Through trial and error, we end up with a working solution in spite of the cryptic documentation:

```js
<Navbar collapseOnSelect expand="lg" bg="dark" variant="dark">
  <Navbar.Toggle aria-controls="responsive-navbar-nav" />
  <Navbar.Collapse id="responsive-navbar-nav">
    <Nav className="mr-auto">
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/">home</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/notes">notes</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        <Link style={padding} to="/users">users</Link>
      </Nav.Link>
      <Nav.Link href="#" as="span">
        {user
          ? <em>{user} logged in</em>
          : <Link to="/login">login</Link>
        }
    </Nav.Link>
    </Nav>
  </Navbar.Collapse>
</Navbar>
```

<!-- Ulkoasu on varsin tyylikäs -->
The resulting layout has a very clean and pleasing appearance:

![](../images/7/10.png)

<!-- Jos selaimen kokoa kaventaa, huomaamme että menu "kollapsoituu" ja sen saa näkyville vain klikkaamalla: -->
If the viewport of the browser is narrowed, we notice that the menu "collapses" and it can be expanded by clicking the "hamburger" button:

![](../images/7/11a.png)

<!-- Bootstrap ja valtaosa tarjolla olevista UI-frameworkeista tuottavat [responsiivisia](https://en.wikipedia.org/wiki/Responsive_web_design) näkymiä, eli sellaisia jotka renderöityvät vähintään kohtuullisesti monen kokoisilla näytöillä. -->
Bootstrap and a large majority of existing UI frameworks produce [responsive](https://en.wikipedia.org/wiki/Responsive_web_design) designs, meaning that the resulting applications render well on a variety of different screen sizes.

<!-- Chromen developer-konsolin avulla on mahdollista simuloida sovelluksen käyttöä erilaisilla mobiilipäätteillä -->
Chrome developer tools makes it possible to simulate using our application in the browser of different mobile clients:

![](../images/7/12.png)


<!-- Esimerkin sovelluksen koodi kokonaisuudessaan [täällä](https://github.com/fullstack-hy2019/misc/blob/master/notes-bootstrap.js) -->
You can find the complete code for the application [here](https://github.com/fullstack-hy2019/misc/blob/master/notes-bootstrap.js).

### Semantic UI

<!-- Olen käyttänyt bootstrapia vuosia, mutta reilu vuosi sitten siirryin [Semantic UI](https://semantic-ui.com/):n käyttäjäksi. Kurssin tehtävien [palautusovellus](https://studies.cs.helsinki.fi/courses) on tehty Semanticilla ja kokemukset ovat olleet rohkaisevia, erityisesti semanticin [React-tuki](https://react.semantic-ui.com) on ensiluokkainen ja dokumentaatiokin huomattavasti parempi kuin bootstrapissa. -->
I have used Bootstrap for years but approximately a year ago I switched over to using [Semantic UI](https://semantic-ui.com/). The [exercise submission system](https://studies.cs.helsinki.fi/courses) that is used in this course has been made with Semantic and my experience has been reassuring. The [support for React](https://react.semantic-ui.com) is world class and the documentation is leagues above Bootstrap.

<!-- Lisätään nyt [React-router](/osa6/#react-router)-sovellukselle edellisen luvun tapaan tyylit semanticilla. -->
Let's continue working with the [React-router](/osa6/#react-router) example application we just styled with Bootstrap, but this time style it with Semantic UI.

<!-- Aloitetaan asentamalla [semantic-ui-react](https://react.semantic-ui.com)-kirjasto: -->
We will start by installing the [semantic-ui-react](https://react.semantic-ui.com) package:

```js
npm install --save semantic-ui-react
```

<!-- Lisätään sitten sovelluksen tiedostoon <i>public/index.html</i> head-tagin sisään semanticin css-määrittelyt lataava rivi (joka löytyy [tästä](https://react.semantic-ui.com/usage#content-delivery-network-cdn)): -->

Then let's add the link to the CSS stylesheet for Semantic UI inside the head tag of the application's <i>public/index.html</i> file (the link can be found [here](https://react.semantic-ui.com/usage#content-delivery-network-cdn)):

```js
<head>
  <link
    rel="stylesheet"
    href="//cdn.jsdelivr.net/npm/semantic-ui@2.4.2/dist/semantic.min.css"
  />
  // ...
</head>
```

<!-- Sijoitetaan koko sovelluksen renderöimä sisältö Semanticin komponentin [Container](https://react.semantic-ui.com/elements/container) sisälle. -->
We render all of the application's content inside of Semantic's [Container](https://react.semantic-ui.com/elements/container) component.

<!-- Semanticin dokumentaatio sisältää jokaisesta komponentista useita esimerkkikoodinpätkiä, joiden avulla komponenttien käytön periaatteet on helppo omaksua: -->
The documentation for Semantic UI contains several code examples for each component that makes it easy to see how each component is used in practice:

![](../images/7/13.png)

<!-- Muutetaan komponentin App uloin <i>div</i>-elementti komponentiksi <i>Container</i>: -->
Let's swap the out root <i>div</i> element of App for a <i>Container</i> component:

```js
import { Container } from 'semantic-ui-react'

// ...

const App = () => {
  // ...
  return (
    <Container>
      // ...
    </Container>
  )
}
```

<!-- Sivun sisältö ei ole enää reunoissa kiinni: -->
The content of the application is no longer attached to the edges of the browser:

![](../images/7/14.png)

<!-- Edellisen luvun tapaan, renderöidään muistiinpanot taulukkona, komponentin [Table](https://react.semantic-ui.com/collections/table) avulla. Koodi näyttää seuraavalta -->
Just like we did with Bootstrap, let's render the notes as a table with Semantic's [Table](https://react.semantic-ui.com/collections/table) component. The resulting code looks like this:

```js
import { Table } from 'semantic-ui-react'

const Notes = (props) => (
  <div>
    <h2>Notes</h2>
    <Table striped celled>
      <Table.Body>
        {props.notes.map(note =>
          <Table.Row key={note.id}>
            <Table.Cell>
              <Link to={`/notes/${note.id}`}>
                {note.content}
              </Link>
            </Table.Cell>
            <Table.Cell>
              {note.user}
            </Table.Cell>
          </Table.Row>
        )}
      </Table.Body>
    </Table>
  </div>
```

<!-- Muistiinpanojen lista näyttää seuraavalta: -->
After making these changes, the list of notes looks like this:

![](../images/7/14.png)

<!-- #### Lomake -->
#### Form

<!-- Otetaan kirjautumissivulla käyttöön Semanticin [Form](https://react.semantic-ui.com/collections/form)-komponentti: -->
Let's use Semantic's [Form](https://react.semantic-ui.com/collections/form) component in the login view of the application:

```js
import { Form, Button } from 'semantic-ui-react'

let Login = (props) => {
  const onSubmit = (event) => {
    // ...
  }

  return (
    <Form onSubmit={onSubmit}>
      <Form.Field>
        <label>username</label>
        <input name='username' />
      </Form.Field>
      <Form.Field>
        <label>password</label>
        <input type='password' />
      </Form.Field>
      <Button type='submit'>login</Button>
    </Form>
  )
}
```

<!-- Ulkoasu näyttää seuraavalta: -->
The appearance of the new login view looks like this:

![](../images/7/15.png)

<!-- #### Notifikaatio -->
#### Notification

<!-- Edellisen luvun tapaan, toteutetaan sovellukseen kirjautumisen jälkeinen <i>notifikaatio</i>: -->
Just like we did with the Bootstrap version, let's implement a styled <i>notification</i> that is displayed after a user logs in to the application:

![](../images/7/6.png)

<!-- Kuten edellisessä luvussa, asetetaan notifikaatio kirjautumisen yhteydessä komponentin <i>App</i> tilan muuttujaan _message_: -->
As we did previously, let's store the message of the notification in the _message_ variable in the <i>App</i> component's state:

```js
const App = () => {
  // ...
  const [message, setMessage] = useState(null)

  const login = (user) => {
    setUser(user)
    setMessage(`welcome ${user}`)
    setTimeout(() => {
      setMessage(null)
    }, 10000)
  }

  // ...
}
```

<!-- ja renderöidään viesti käyttäen komponenttia [Message](https://react.semantic-ui.com/collections/message): -->
And let's render the notification by using Semantic's [Message](https://react.semantic-ui.com/collections/message) component:

```js
<Container>
  {(message &&
    <Message success>
      {message}
    </Message>
  )}
  // ...
</Conteiner>
```

<!-- #### Navigaatiorakenne -->
#### Navigation structure

<!-- Navigaatiorakenne toteutetaan komponentin [Menu](https://react.semantic-ui.com/collections/menu) avulla: -->
The navigation bar of the application will be implemented with Semantic's [Menu](https://react.semantic-ui.com/collections/menu) component:

```js
<Router>
  <div>
    <Menu inverted>
      <Menu.Item link>
        <Link to="/">home</Link>
      </Menu.Item>
      <Menu.Item link>
        <Link to="/notes">notes</Link>
      </Menu.Item>
      <Menu.Item link>
        <Link to="/users">users</Link>
      </Menu.Item>
      <Menu.Item link>
        {user
          ? <em>{user} logged in</em>
          : <Link to="/login">login</Link>
        }
      </Menu.Item>
    </Menu>
    // ...
  </div>
</Router>
```

<!-- Lopputulos näyttää seuraavalta: -->
The result looks like this:

![](../images/7/17.png)

<!-- Esimerkin sovelluksen koodi kokonaisuudessaan [täällä](https://github.com/fullstack-hy2019/misc/blob/master/notes-semantic.js). -->
You can find the complete code for the application [here](https://github.com/fullstack-hy2019/misc/blob/master/notes-semantic.js).

<!-- ### Loppuhuomioita -->
### Closing thoughts

<!-- Ero react-bootstrapin ja semantic-ui-reactin välillä ei ole suuri. On makuasia kummalla tuotettu ulkoasu on tyylikkäämpi. Oma vuosia kestäneen bootstrapin käytön jälkeinen siirtymiseni semanticiin johtuu semanticin saumattomammasta React-tuesta, laajemmasta valmiiden komponenttien valikoimasta ja paremmasta sekä selkeämmästä dokumentaatiosta. Semantic UI projektin kehitystyön jatkuvuuden suhteen on kuitenkin viime aikoina ollut ilmoilla muutamia [kysymysmerkkejä](https://github.com/Semantic-Org/Semantic-UI/issues/6109), ja tilannetta kannattaakin seurata. -->
The different between React-Bootstrap and Semantic-UI-React is not that big. Determining which one produces more aesthetically pleasing results comes down to a matter of taste. After years of using Bootstrap, the reasons that made me switch over to Semantic UI were its seamless integration with React, its wider selection of built-in components, and its overall better documentation. There has been some [uncertainty](https://github.com/Semantic-Org/Semantic-UI/issues/6109) regarding the future of Semantic UI, and it's recommended to keep your ear on the ground.

<!-- Esimerkissä käytettiin UI-frameworkeja niiden React-integraatiot tarjoavien kirjastojen kautta. -->
In the two previous examples we used the UI frameworks with the help of React-integration libraries.

<!-- Sen sijaan että käytimme kirjastoa [React bootstrap](https://react-bootstrap.github.io/), olisimme voineet aivan yhtä hyvin käyttää Bootstrapia suoraan, liittämällä HTML-elementteihin CSS-luokkia. Eli sen sijaan että määrittelimme esim. taulukon komponentin <i>Table</i> avulla -->
Instead of using the [React Bootstrap](https://react-bootstrap.github.io/) library, we could have just as well used Bootstrap directly by defining CSS classes to our application's HTML elements. Instead of defining the table with the <i>Table</i> component:

```js
<Table striped>
  // ...
</Table>
```

<!-- olisimme voineet käyttää normaalia HTML:n taulukkoa <i>table</i> ja CSS-luokkaa -->
We could have used a regular HTML <i>table</i> and added the required CSS class:

```js
<table className="table striped">
  // ...
</table>
```

<!-- Taulukon määrittelyssä React bootstrapin tuoma etu ei ole suuri. -->
The benefit of using the React Bootstrap library is not that evident from this example.

<!-- Tiiviimmän ja ehkä paremmin luettavissa olevan kirjoitusasun lisäksi toinen etu React-kirjastoina olevissa UI-frameworkeissa on se, että kirjastojen mahdollisesti käyttämä Javascript-koodi on sisällytetty React-komponentteihin. Esim. osa Bootstrapin komponenteista edellyttää toimiakseen muutamaakin ikävää [Javascript-riippuvuutta](https://getbootstrap.com/docs/4.1/getting-started/introduction/#js) joita emme mielellään halua React-sovelluksiin sisällyttää. -->
In addition to making the frontend code more compact and readable, another benefit of using React UI framework libraries is that they include the JavaScript that is needed to make specific components work. Some Bootstrap components require a few unpleasant [JavaScript dependencies](https://getbootstrap.com/docs/4.1/getting-started/introduction/#js) that we would prefer not to include in our React applications.

<!-- React-kirjastoina tarjottavien UI-frameworkkien ikävä puoli verrattuna frameworkin "suoraan käyttöön" on React-kirjastojen API:n mahdollinen epästabiilius ja osittain huono dokumentaatio. Tosin [react-semanticin](https://react.semantic-ui.com) suhteen tilanne on paljon parempi kuin monien muiden UI-frameworkien sillä kyseessä on virallinen React-integraatio. -->
Some potential downsides to using UI frameworks through integration libraries instead of using them "directly", are that integration libraries may have unstable API's and poor documentation. The situation with [Semantic UI React](https://react.semantic-ui.com) is a lot better than with many other UI frameworks, as it is an official React integration library.

<!-- Kokonaan toinen kysymys on se kannattaako UI-frameworkkeja ylipäätän käyttää. Kukin muodostakoon oman mielipiteensä, mutta CSS:ää taitamattomalle ja puutteellisilla design-taidoilla varustetulle ne ovat varsin käyttökelpoisia työkaluja. -->
There is also the question of whether or not UI framework libraries should be used in the first place. It is up to everyone to form their own opinion, but for people lacking knowledge in CSS and web design they are very useful tools.

<!-- ### Muita UI-frameworkeja -->
### Other UI frameworks

<!-- Luetellaan tässä kaikesta huolimatta muitakin UI-frameworkeja. Jos oma suosikkisi ei ole mukana, tee pull request -->
Here are some other UI frameworks for your consideration. If you do not see your favorite UI framework in the list, please make a pull request to the course material.

- <http://www.material-ui.com/>
- <https://bulma.io/>
- <https://ant.design/>
- <https://foundation.zurb.com/>

### Styled components

<!-- Tapoja liittää tyylejä React-sovellukseen on jo näkemiämme lisäksi [muitakin](https://blog.bitsrc.io/5-ways-to-style-react-components-in-2019-30f1ccc2b5b). -->
There are also [other ways](https://blog.bitsrc.io/5-ways-to-style-react-components-in-2019-30f1ccc2b5b) of styling React applications that we have not yet taken a look at.

<!-- Mielenkiintoisen näkökulman tyylien määrittelyyn tarjoaa ES6:n [tagged template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) -syntaksia hyödyntävä [styled components](https://www.styled-components.com/) -kirjasto. -->
The [styled components](https://www.styled-components.com/) library offers an interesting approach for defining styles through [tagged template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) that were introduced in ES6.

<!-- Tehdään styled-componentsin avulla esimerkkisovellukseemme muutama tyylillinen muutos. Tehdään ensin kaksi tyylimäärittelyyn käytettävää komponenttia: -->
Let's make a few changes to the styles of our application with the help of styled components. First, let's define two components for defining styles:

```js
import styled from 'styled-components'

const Button = styled.button`
  background: Bisque;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid Chocolate;
  border-radius: 3px;
`

const Input = styled.input`
  margin: 0.25em;
`
```

<!-- Koodi luo HTML:n elementeistä <i>button</i> ja <i>input</i> tyyleillä rikastetut versiot ja sijoitetaan ne muuttujiin <i>Button</i> ja <i>Input</i>: -->
The code above creates styled versions of the <i>button</i> and <i>input</i> HTML elements and then assigns them to the <i>Button</i> and <i>Input</i> variables.

<!-- Tyylien määrittelyn syntaksi on varsin mielenkiintoinen, css-määrittelyt asetetaan backtick-hipsujen sisään. -->
The syntax for defining the styles is quite interesting, as the CSS rules are defined inside of backticks.

<!-- Määritellyt komponentit toimivat kuten normaali <i>button</i> ja <i>input</i> ja sovelluksessa käytetään niitä  normaaliin tapaan: -->
The styled components that we defined work exactly like regular <i>button</i> and <i>input</i> elements, and they can be used the same way:

```js
const Login = (props) => {
  // ...
  return (
    <div>
      <h2>login</h2>
      <form onSubmit={onSubmit}>
        <div>
          username:
          <Input /> // highlight-line
        </div>
        <div>
          password:
          <Input type='password' /> // highlight-line
        </div>
        <Button type="submit" primary=''>login</Button> // highlight-line
      </form>
    </div>
  )
}
```

<!-- Määritellään vielä seuraavat tyylien lisäämiseen tarkoitetut komponentit, jotka ovat kaikki rikastettuja versioita <i>div</i>-elementistä: -->
Let's create a few more components for styling that application, that are styled versions of <i>div</i> elements:

```js
const Page = styled.div`
  padding: 1em;
  background: papayawhip;
`

const Navigation = styled.div`
  background: BurlyWood;
  padding: 1em;
`

const Footer = styled.div`
  background: Chocolate;
  padding: 1em;
  margin-top: 1em;
`
```

<!-- Otetaan uudet komponentit käyttöön sovelluksessa: -->
Let's use the components in our application:

```js
const App = () => {
  // ...

  return (
    <Page> // highlight-line
      <Router>
        <div>
          <Navigation> // highlight-line
            <Link style={padding} to="/">home</Link>
            <Link style={padding} to="/notes">notes</Link>
            <Link style={padding} to="/users">users</Link>
            {user
              ? <em>{user} logged in</em>
              : <Link to="/login">login</Link>
            }
          </Navigation>

          <Route exact path="/" render={() => <Home />} />
          <Route exact path="/notes" render={() =>
            <Notes notes={notes} />}
          />
          <Route exact path="/notes/:id" render={({ match }) =>
            <Note note={noteById(match.params.id)} />}
          />
          <Route path="/users" render={() =>
            user ? <Users /> : <Redirect to="/login" />
          } />
          <Route path="/login" render={() =>
            <Login onLogin={login} />}
          />
        </div>
      </Router>
      <Footer> // highlight-line
        <em>Note app, Department of Computer Science 2019</em>
      </Footer>
    </Page>
  )
}
```

<!-- Lopputulos on seuraavassa: -->
The appearance of the resulting application is shown below:

![](../images/7/18.png)

<!-- Styled components on nostanut tasaisesti suosiotaan viime aikoina ja tällä hetkellä näyttääkin, että se on melko monien mielestä paras tapa React-sovellusten tyylien määrittelyyn. -->
Styled components have seen a consistent growth in popularity in recent times, and quite a lot of people consider it to be the best way of defining styles to React applications.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Tämän luvun asioihin liittyvät tehtävät ovat osan lopun [blogilistaa laajentavassa tehtäväsarjassa](/osa7/tehtavia_blogilistan_laajennus). -->
The exercises related to the topics presented here, can be found at the end of this course material section in the exercise set [for extending the blog list application](/osa7/tehtavia_blogilistan_laajennus).

</div>


<!--

### Komponenttikohtainen CSS ja CSS-moduulit

Tapoja liittää tyylejä React-sovellukseen on jo näkemiämme lisäksi [muitakin](https://blog.bitsrc.io/5-ways-to-style-react-components-in-2019-30f1ccc2b5b). Katsotaan vielä lyhyestä muutamaa tapaa.

On melko yleistä, että React-sovelluksen CSS määritellään useassa eri tiedostossa. Koko sovellusta koskeva CSS saattaa tiedostossa <i>index.js</i> ja sen lisäksi eri komponentteja koskevat määrittelyt tehdään komponenttikohtaisiin tiedostoihin.

Voisimme määritellä esimerkkisovelukselle fontin ja taustavärin tiedostossa <i>index.css</i>

```css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
  background: lightgrey;
}
```

ja importata tyylit sovellukseen

```js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'
import './index.css'

ReactDOM.render(<App />, document.getElementById('root'));
```

Tämän lisäksi voisimme määritellä komponentille <i>Login</i> oman tyylimäärityksen

```css
button {
  background: Bisque;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid Chocolate;
  border-radius: 3px;
}

input {
  margin: 0.25em;
}
```

ja importata sen komponentin koodissa

```js
import React from 'react'
import { withRouter } from 'react-router-dom'
import './login.css' // highlight-line

const Login = (props) => {
  // ...
}


export default withRouter(Login)
```

Yksi CSS:n keskeisistä ongelmista on se, että CSS-määrittelyt ovat <i>globaaleja</i>. Suurissa tai jo keskikokoisissakin sovelluksissa tämä aiheuttaa ongelmia, sillä tiettyihin komponentteihin vaikuttavat monissa paikoissa määritellyt tyylit ja lopputulos voi olla vaikeasti ennakoitavissa.

Oletetaan, että nyt haluaisimme lisätä komponenttiin <i>Note</i> painikkeen jolla on oma tyylinsä. Määritellään tyyli tiedostoon <i>note.css</i>

```css
button {
  font-size: 2em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid black;
  border-radius: 1px;
}
```

ja importataan se komponentin koodista

```css
import React from 'react'
import './note.css' // highlight-line

const Note = ({ note }) => {
  return (
    <div>
      <h2>{note.content}</h2>
      <div>{note.user}</div>
      <div><strong>{note.important ? 'tärkeä' : ''}</strong></div>
      <button>merkitse tärkeäksi</button> // highlight-line
    </div>
  )
}
```

Lopputulos ei kuitenkaan ole haluamamme kaltainen. Koska molemmat komponentit asettavat tyylit button-elementille, käykin niin että myöhemmin määritelty ylikirjoittaa aiemmin määritellyn, ja molempien tyyli on sama.

Perinteinen tapa kiertää ongelma on ollut käyttää monimutkaisempia CSS-luokan nimiä, esim. _Login_button_ ja _Note_button_, tämä muuttuu kuitenkin jossain vaiheessa varsin hankalaksi.

[CSS-moduulit](https://github.com/css-modules/css-modules) tarjoaa tähän erään ratkaisun.

Lyhyesti ilmaisten periaatteena on tehdä CSS-määrittelyistä lähtökohtaisesti lokaaleja, vain yhden komponentin kontekstissa voimassa olevia, joka taas mahdollistaa luontevien CSS-luokkanimien käytön. Käytännössä tämä lokaalius toteutetaan generoimalla konepellin alla CSS-luokille uniikit luokkanimet.

Muutetaan tyylejä käyttäviä komponentteja hiukan:

```js
import styles from './Hello.css'

const Hello = ({ counter }) => (
  <p className={styles.content}>
    hello webpack {counter} clicks!
  </p>
)

export default Hello
```

Erona siis edelliseen on se, että tyyliit "sijoitetaan muuttujaan" _styles_

```js
import styles from './Hello.css';
```

Nyt tyylitiedoston määrittelelyihin voi viitata muuttujan _styles_ kautta, ja CSS-luokan liittäminen tapahtuu seuraavasti

```js
<p className={styles.content}>
```

-->
