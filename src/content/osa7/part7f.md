---
mainImage: ../../images/part-7.svg
part: 7
letter: f
---

<div class="content">

<!-- Kurssin seitsemännessä osassa on luvun [React router](/osa7/react_router) kolmen tehtävän lisäksi 17 tehtävää, joissa jatketaan osissa 4 ja 5 tehtyä Bloglist-sovellusta.  Osa seuraavassa olevista tehtävistä on toisistaan riippumattomia "featureita", eli tehtäviä ei tarvitse tehdä järjestyksessä, voit jättää osan aivan hyvin toteuttamatta. -->
In addition to the three exercises in the [React router](/osa7/react_router) section of this seventh part of the course material, there are 17 exercises that continue our work on the Bloglist application that we worked on in parts four and five of the course material. Some of the following exercises are "features" that are independent of one another, meaning that there is no need to finish the exercises in any particular order. You are free to skip over a part of the exercises if you wish to do so.

<!-- Voit ottaa pohjaksi oman sovelluksesi sijaan myös mallivastauksen koodin. -->
If you do not want to use your own Bloglist application, you are free to use the code from the model solution as a starting point for these exercises. 

<!-- Useimmat tämän osan tehtävistä vaativat olemassaolevan koodin refaktoroimista. Tämä on tilanne käytännössä aina sovelluksia laajennettaessa, eli vaikka refaktorointi voi olla hankalaa ja ikävääkin, on kyseessä oleellinen taito. -->
Many of the exercises in this part of the course material will require the refactoring of existing code. This is a common reality of extending existing applications, meaning that refactoring is an important and necessary skill even if it may feel difficult and unpleasant at times.

<!-- Hyvä neuvo niin refaktorointiin kuin uudenkin koodin kirjoittamiseen on <i>pienissä askelissa eteneminen</i>, koodia ei kannata hajottaa totaalisesti refaktorointia tehdessä pitkäksi aikaa, se on käytännössä varma resepti hermojen menettämiseen. -->
One good piece of advice for both refactoring and writing new code is to take <i> baby steps</i>. Losing your sanity is almost guaranteed if you leave the application in a completely broken state for long periods of time while refactoring.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

#### 7.4: redux, step1

<!-- Siirry käyttämään React-komponenttien tilan sijaan Reduxia sovelluksen tilan hallintaan. -->
Refactor the application from using internal React component state to using Redux for the application's state management.

<!-- Muuta tässä tehtävässä notifikaatio käyttämään Reduxia. -->
Additionally, change the application's notifications to use Redux at this point of the exercise set.

#### 7.5 redux, step2

<!-- Siirrä blogien tietojen talletus Reduxiin. -->
Store the information about blog posts in the Redux store.

<!-- Kirjautumisen ja uuden blogin luomisen lomakkeiden tilaa voit halutessasi hallita edelleen Reactin tilan avulla.  -->
You are free to manage the state for logging in and creating new blog posts by using the internal state of React components.

<!-- Tämä ja seuraava osa ovat kohtuullisen työläitä, mutta erittäin opettavaisia. -->
This and the next exercise are quite laborious but incredibly educational.

#### 7.6 redux, step3

<!-- Siirrä myös kirjautuneen käyttäjän tietojen talletus Reduxiin. -->
Store the information about the signed in user in the Redux store.

<!-- #### 7.7 käyttäjien näkymä -->
#### 7.7 Users view

<!-- Tee sovellukseen näkymä, joka näyttää kaikkiin käyttäjiin liittyvät perustietot: -->
Implement a view to the application that displays all of the basic information related to users:

![](../images/7/41.png)

<!-- #### 7.8 yksittäisen käyttäjän näkymä -->
#### 7.8 Individual user view

<!-- Tee sovellukseen yksittäisen käyttäjän näkymä, jolta selviää mm. käyttäjän lisäämät blogit -->
Implement a view for individual users, that displays all of the blog posts added by that user:

![](../images/7/44.png)

<!-- Näkymään päästään klikkaamalla nimeä kaikkien käyttäjien näkymästä -->
You can access the view by clicking the name of the user in the view that lists all users.

![](../images/7/43.png)

<i>**Huom1:**</i> jos sovelluksesi käyttää tilanhallintaan Reduxia, saattaa tässä tehtävässä olla hyödyksi käyttää funktion _mapStateToProps_ toista parametria [ownPropsia](https://react-redux.js.org/api/connect#mapstatetoprops-state-ownprops-object), joka on dokumentaation hienoisesta kryptisyydestä huolimatta aika [helppokäyttöinen](https://stackoverflow.com/questions/41198842/what-is-the-use-of-the-ownprops-arg-in-mapstatetoprops-and-mapdispatchtoprops).

<i>**Huom2:**</i> törmäät tätä tehtävää tehdessäsi lähes varmasti seuraavaan virheeseen

![](../images/7/42a.png)

vika ilmenee jos uudelleenlataat sivun ollessasi yksittäisen käyttäjän sivulla. 

Vian syynä on se, että jos mennään suoraan jonkin käyttäjän sivulle, eivät käyttäjien tiedot ole vielä ehtineet palvelimelta React-sovellukseen. Ongelman voi kiertää ehdollisella renderöinnillä esim. seuraavasti:

```js
const User = (props) => {
  // highlight-start
  if ( props.user === undefined) { 
    return null
  }
  // highlight-end

  return (
    <div>
      <h2>{props.user.name}</h2>

      <h3>added blogs</h3>
      // ...
    </div>
  )
}
```

#### 7.9 blogin näkymä

Toteuta sovellukseen oma näkymä yksittäisille blogeille. Näkymä voi näyttää seuraavalta

![](../images/7/45.png)

Näkymään päästään klikkaamalla blogin nimeä kaikkien blogien näkymästä

![](../images/7/46.png)

Tämän tehtävän jälkeen tehtävässä 5.6 toteutettua toiminnallisuutta ei enää tarvita, eli kaikkien blogien näkymässä yksittäisten blogien detaljien ei enää tarvitse avautua klikattaessa.

#### 7.10 navigointi

Tee sovellukseen navigaatiomenu

![](../images/7/47.png)

#### 7.11 kommentit, step1

Tee sovellukseen mahdollisuus blogien kommentointiin:

![](../images/7/48.png)

Kommentit ovat anonyymejä, eli ne eivät liity järjestelmän käyttäjiin.

Tässä tehtävässä riittää, että frontend osaa näyttää blogilla olevat backendin kautta lisätyt kommentit.

Sopiva rajapinta kommentin luomiseen on osoitteeseen <i>api/blogs/:id/comments</i> tapahtuva HTTP POST -pyyntö.

#### 7.12 kommentit, step2

Laajenna sovellusta siten, että kommentointi onnistuu frontendista käsin:

![](../images/7/49.png)

#### 7.13 tyylit, step1

Tee sovelluksesi ulkoasusta tyylikkäämpi jotain kurssilla esiteltyä tapaa käyttäen

#### 7.14 tyylit, step2

Jos käytät tyylien lisäämiseen noin tunnin aikaa, merkkaa myös tämä tehtävä tehdyksi.

#### 7.15 ESLint

Konfiguroi frontend käyttämään ESLintiä

#### 7.16 Webpack

Tee sovellukselle sopiva webpack-konfiguraatio

#### 7.17 End to end -testaus, step1

Tee sovellukselle ainakin kaksi E2E-testiä [Cypress-kirjaston](/osa7/luokkakomponentit_e_2_e_testaus#sovelluksen-end-to-end-testaus) avulla. Sopiva testattava asia on esim. käyttäjän kirjautuminen sovellukseen.

Tämän tehtävän testeissä voit olettaa, että tietokannan tila on testien kannalta sopiva, eli että tietokannassa on olemassa ainakin yksi käyttäjä. 

Kannattanee käyttää hetki aikaa Cypressin dokumentaation silmäilemiseen, erityisesti 
[best practices](https://docs.cypress.io/guides/references/best-practices.html) sisältää monia asioita, joita on hyvä pitää mielessä testejä kirjoittaessa.

#### 7.18 End to end -testaus, step2

Laajenna E2E-testejä siten, että testit [alustavat tietokannan](/osa7/luokkakomponentit_e_2_e_testaus#tietokannan-tilan-kontrollointi) aina ennen testien suorittamista. Tee myös ainakin yksi testi, joka muokkaa sovelluksen tietokantaa, esim. lisää sovellukseen blogin.

#### 7.19 End to end -testaus, step3

Laajenna vielä E2E-testejäsi. Voit merkitä tehtävän, jos käytät laajentamiseen vähintään 30 minuuttia aikaa.

#### 7.20 Kurssipalaute

Anna kurssille palautetta Moodlessa.

</div>
