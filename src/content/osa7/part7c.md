---
mainImage: ../../images/part-7.svg
part: 7
letter: c
---

<div class="content">

<!-- React oli aiemmin jossain määrin kuuluisa siitä, että sovelluskehityksen edellyttämien työkalujen konfigurointi on ollut hyvin hankalaa. Kiitos [create-react-app](https://github.com/facebookincubator/create-react-app):in, sovelluskehitys Reactilla on kuitenkin nykyään tuskatonta, parempaa työskentelyflowta on tuskin ollut koskaan Javascriptillä tehtävässä selainpuolen sovelluskehityksessä. -->
Developing with React was notorious for requiring tools that were very difficult to configure. These days, getting started with React development is almost painless thanks to [create-react-app](https://github.com/facebookincubator/create-react-app). A better development workflow has probably never existed for browser-side JavaScript development.

<!-- Emme voi kuitenkaan turvautua ikuisesti create-react-app:in magiaan ja nyt onkin aika selvittää mitä kaikkea taustalla on. Avainasemassa React-sovelluksen toimintakuntoon saattamisessa on [webpack](https://webpack.js.org/)-niminen työkalu. -->
We can not rely on the black magic of create-react-app forever and it's time for us to take a look under the hood. One of the key players in making React applications functional is a tool called [webpack](https://webpack.js.org/).

<!-- ### bundlaus -->
### Bundling

<!-- Olemme toteuttaneet sovelluksia jakamalla koodin moduuleihin, joita on <i>importattu</i> niitä tarvitseviin paikkoihin. Vaikka ES6-moduulit ovatkin Javascript-standardissa määriteltyjä, ei mikään selain vielä osaa käsitellä moduuleihin jaettua koodia. -->
We have implemented our applications by dividing our code into separate modules that have been <i>imported</i> to places that require them. Even though ES6 modules are defined in the ECMAScript standard, no browser actually knows how to handle code that is divided into modules.

<!-- Selainta varten moduuleissa oleva koodi <i>bundlataan</i>, eli siitä muodostetaan yksittäinen, kaiken koodin sisältävä tiedosto. Kun veimme Reactilla toteutetun frontendin tuotantoon osan 3 luvussa [Frontendin tuotantoversio](/osa3/sovellus_internetiin#frontendin-tuotantoversio), suoritimme bundlauksen komennolla _npm run build_. Konepellin alla kyseinen npm-skripti suorittaa bundlauksen webpackia hyväksi käyttäen. Tuloksena on joukko hakemistoon <i>build</i> sijoitettavia tiedostoja: -->
For this reason, code that is divided into modules must be <i>bundled</i> for browsers, meaning that all of the source code files are transformed into a single file that contains all of the application code. When we deployed our React frontend to production in [part 3](/osa3/sovellus_internetiin#frontendin-tuotantoversio), we performed the bundling of our application with the _npm run build_ command. Under the hood, the npm script bundles the source code using webpack which produces the following collection of files in the <i>build</i> directory:

<pre>
├── asset-manifest.json
├── favicon.ico
├── index.html
├── manifest.json
├── precache-manifest.8082e70dbf004a0fe961fc1f317b2683.js
├── service-worker.js
└── static
    ├── css
    │   ├── main.f9a47af2.chunk.css
    │   └── main.f9a47af2.chunk.css.map
    └── js
        ├── 1.578f4ea1.chunk.js
        ├── 1.578f4ea1.chunk.js.map
        ├── main.8209a8f2.chunk.js
        ├── main.8209a8f2.chunk.js.map
        ├── runtime~main.229c360f.js
        └── runtime~main.229c360f.js.map
</pre>

<!-- Hakemiston juuressa oleva sovelluksen "päätiedosto" <i>index.html</i> lataa <i>script</i>-tagin avulla bundlatun Javascript-tiedoston (jos ollaan tarkkoja, on bundlattuja Javascript-tiedostoja kaksi kappaletta): -->
The <i>index.html</i> file located at the root of the build directory is the "main file" of the application, that loads the bundled JavaScript file with a <i>script</i> tag (in fact there are two bundled JavaScript files):

```html
<!doctype html><html lang="en">
<head>
  <meta charset="utf-8"/>
  <title>React App</title>
  <link href="/static/css/main.f9a47af2.chunk.css" rel="stylesheet"></head>
<body>
  <div id="root"></div>
  <script src="/static/js/1.578f4ea1.chunk.js"></script>
  <script src="/static/js/main.8209a8f2.chunk.js"></script>
</body>
</html>
```

<!-- Kuten esimerkistä näemme, create-react-app:illa tehdyssä sovelluksessa bundlataan Javascriptin lisäksi sovelluksen CSS-määrittelyt tiedostoon <i>/static/css/main.f9a47af2.chunk.css</i> -->
As we can see from the example application that was created with create-react-app, the build script also bundles the application's CSS files into a single <i>/static/css/main.f9a47af2.chunk.css</i> file.

<!-- Käytännössä bundlaus tapahtuu siten, että sovelluksen Javascriptille määritellään alkupiste, usein tiedosto <i>index.js</i>, ja bundlauksen yhteydessä webpack ottaa mukaan kaiken koodin mitä alkupiste importtaa, sekä importattujen koodien importtaamat koodit, jne. -->
In practice, bundling is done so that we define an entry point for the application, which typically is the <i>index.js</i> file. When webpack bundles the code, it includes all of the code that the entry point imports, and the code that its imports import, and so on.

<!-- Koska osa importeista on kirjastoja, kuten React, Redux ja Axios, bundlattuun Javascript-tiedostoon tulee myös kaikkien näiden sisältö. -->
Since part of imported files are packages like React, Redux, and Axios; the bundled JavaScript file will also contain the contents of each of these libraries.

<!-- > Vanha tapa jakaa sovelluksen koodi moneen tiedostoon perustui siihen, että <i>index.html</i> latasi kaikki sovelluksen tarvitsemat erilliset Javascript-tiedostot script-tagien avulla. Tämä on kuitenkin tehotonta, sillä jokaisen tiedoston lataaminen aiheuttaa pienen overheadin ja nykyään pääosin suositaankin koodin bundlaamista yksittäiseksi tiedostoksi. -->
> The old way of dividing the application's code into multiple files was based on the fact that the <i>index.html</i> file loaded all of the separate JavaScript files of the application with the help of script tags. This resulted in  decreased performance, since the loading of each separate file results in some overhead. For this reason, these days the preferred method is to bundle the code into a single file.

<!-- Tehdään nyt React-projektille sopiva webpack-konfiguraatio kokonaan käsin. -->
Next, we will create a suitable webpack configuration for a React application by hand from scratch.

<!-- Luodaan projektia varten hakemisto ja sen sisälle seuraavat hakemistot (<i>build</i> ja <i>src</i>) sekä seuraavat tiedostot: -->
Let's create a new directory for the project with the following subdirectories (<i>build</i> and <i>src</i>) and files:

<pre>
├── build
├── package.json
├── src
│   └── index.js
└── webpack.config.js
</pre>

<!-- Tiedoston <i>package.json</i> sisältö voi olla esim. seuraava: -->
The contents of the <i>package.json</i> file can e.g. be the following:

```json
{
  "name": "webpack-osa7",
  "version": "0.0.1",
  "description": "practising webpack",
  "scripts": {},
  "license": "MIT"
}
```

<!-- Asennetaan webpack komennolla -->
Let's install webpack with the command:

```js
npm install --save-dev webpack webpack-cli
```

<!-- Webpackin toiminta konfiguroidaan tiedostoon <i>webpack.config.js</i>, laitetaan sen alustavaksi sisällöksi seuraava -->
We define the functionality of webpack in the <i>webpack.config.js</i> file, which we initialize with the following content:

```js
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  }
}
module.exports = config
```

<!-- Määritellään sitten npm-skripti <i>build</i> jonka avulla bundlaus suoritetaan -->
We will then define a new npm script called <i>build</i> that will execute the bundling with webpack:

```js
// ...
"scripts": {
  "build": "webpack --mode=development"
},
// ...
```

<!-- Lisätään hieman koodia tiedostoon <i>src/index.js</i>: -->
Let's add some more code to the <i>src/index.js</i> file:

```js
const hello = name => {
  console.log(`hello ${name}`);
};
```

<!-- Kun nyt suoritamme komennon _npm run build_ webpack bundlaa koodin. Tuloksena on hakemistoon <i>build</i> sijoitettava tiedosto <i>main.js</i>: -->
When we execute the _npm run build_ command our application code will be bundled by webpack. The operation will produce a new <i>main.js</i> file that is added under the <i>build</i> directory:

![](../images/7/19.png)

<!-- Tiedostossa on paljon erikoisen näköistä tavaraa. Lopussa on mukana myös kirjoittamamme koodi. -->
The file contains a lot of stuff that looks quite interesting. We can also see the code we wrote earlier at the end of the file.

<!-- Lisätään hakemistoon <i>src</i> tiedosto <i>App.js</i> ja sille sisältö -->
Let's add a <i>App.js</i> file under the <i>src</i> directory with the following content:

```js
const App = () => {
  return null
}

export default App
```

<!-- Importataan ja käytetään modulia <i>App</i> tiedostossa <i>index.js</i> -->
Let's import and use the <i>App</i> module in the <i>index.js</i> file:

```js
import App from './App';

const hello = name => {
  console.log(`hello ${name}`)
};

App()
```

<!-- Kun nyt suoritamme bundlauksen komennolla _npm run build_ huomaamme webpackin havainneen molemmat tiedostot: -->
When we bundle the application again with the _npm run build_ command, we notice that webpack has acknowledged both files:

![](../images/7/20.png)

<!-- Kirjoittamamme koodi löytyy erittäin kryptisesti muotoiltuna bundlen lopusta: -->
Our application code can be found at the end of the bundle file in a rather obscure format:

```js
/***/ "./src/App.js":
/*!********************!*\
  !*** ./src/App.js ***!
  \********************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\nconst App = () => {\n  return null\n}\n\n/* harmony default export */ __webpack_exports__[\"default\"] = (App);\n\n//# sourceURL=webpack:///./src/App.js?");

/***/ }),

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _App__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./App */ \"./src/App.js\");\n\n\nconst hello = name => {\n  console.log(`hello ${name}`)\n};\n\nObject(_App__WEBPACK_IMPORTED_MODULE_0__[\"default\"])()\n\n//# sourceURL=webpack:///./src/index.js?");

/***/ })
```

<!-- ### Konfiguraatiotiedosto -->
### Configuration file

<!-- Katsotaan nyt tarkemmin konfiguraation <i>webpack.config.js</i> tämänhetkistä sisältöä: -->
Let's take a closer at the contents of our current <i>webpack.config.js</i> file:

```js
const path = require('path')

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js'
  }
}
module.exports = config
```

<!-- Konfiguraatio on Javascriptia ja tapahtuu eksporttaamalla määrittelyt sisältävä olio Noden moduulisyntaksilla. -->
The configuration file has been written in JavaScript and the configuration object is exported by using Node's module syntax. 

<!-- Tämän hetkinen minimaalinen määrittely on aika ilmeinen, kenttä [entry](https://webpack.js.org/concepts/#entry) kertoo sen tiedoston, mistä bundlaus aloitetaan. -->
Our minimal configuration definition almost explains itself. The [entry](https://webpack.js.org/concepts/#entry) property of the configuration object specifies the file that will serve as the entry point for bundling the application.

<!-- Kenttä [output](https://webpack.js.org/concepts/#output) taas kertoo minne muodostettu bundle sijoitetaan. Kohdehakemisto täytyy määritellä <i>absoluuttisena polkuna</i>, se taas onnistuu helposti [path.resolve](https://nodejs.org/docs/latest-v8.x/api/path.html#path_path_resolve_paths)-metodilla. [\_\_dirname](https://nodejs.org/docs/latest/api/globals.html#globals_dirname) on Noden globaali muuttuja, joka viittaa nykyiseen hakemistoon. -->
The [output](https://webpack.js.org/concepts/#output) property defines the location where the bundled code will be stored. The target directory must be defined as an <i>absolute path</i> which is easy to create with the [path.resolve](https://nodejs.org/docs/latest-v8.x/api/path.html#path_path_resolve_paths) method. We also use [\_\_dirname](https://nodejs.org/docs/latest/api/globals.html#globals_dirname) which is a global variable in Node that stores the path to the current directory.

<!-- ### Reactin bundlaaminen -->
### Bundling React

<!-- Muutetaan sitten sovellus minimalistiseksi React-sovellukseksi. Asennetaan tarvittavat kirjastot -->
Next, let's transform our application into a minimal React application. Let's install the required libraries:

```js
npm install --save react react-dom
```

<!-- Liitetään tavanomaiset loitsut tiedostoon <i>index.js</i> -->
And let's turn our application into a React application by adding the familiar definitions in the <i>index.js</i> file:

```js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

ReactDOM.render(<App />, document.getElementById('root'))
```

<!-- ja muutetaan <i>App.js</i> muotoon -->
We will also make the following changes to the <i>App.js</i> file:

```js
import React from 'react'

const App = () => (
  <div>hello webpack</div>
)

export default App
```

<!-- Tarvitsemme sovellukselle myös "pääsivuna" toimivan tiedoston <i>build/index.html</i> joka lataa <i>script</i>-tagin avulla bundlatun Javascriptin: -->
We still need the <i>build/index.html</i> file  that will serve as the "main page" of our application that will load our bundled JavaScript code with a <i>script</i> tag:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="text/javascript" src="./main.js"></script>
  </body>
</html>
```

<!-- Kun bundlaamme sovelluksen, törmäämme kuitenkin ongelmaan -->
When we bundle our application, we run into the following problem:

![](../images/7/21.png)

<!-- ### Loaderit -->
### Loaders

<!-- Webpack mainitsee että saatamme tarvita <i>loaderin</i> tiedoston <i>App.js</i> käsittelyyn. Webpack ymmärtää itse vain Javascriptia ja vaikka se saattaa meiltä matkan varrella olla unohtunutkin, käytämme Reactia ohjelmoidessamme [JSX](https://facebook.github.io/jsx/):ää näkymien renderöintiin, eli esim. seuraava -->
The error message from webpack states that we may need an appropriate <i>loader</i> to bundle the <i>App.js</i> file correctly. By default, webpack only knows how to deal with plain JavaScript. Although we may have become unaware of it, we are actually using [JSX](https://facebook.github.io/jsx/) for rendering our views in React. To illustrate this, the following code is not regular JavaScript:

```js
const App = () => (
  <div>hello webpack</div>
)
```

<!-- ei ole "normaalia" Javascriptia, vaan JSX:n tarjoama syntaktinen oikotie määritellä <i>div</i>-tagiä vastaava React-elementti. -->
The syntax used above comes from JSX and it provides us with an alternative way of defining a React element for an html <i>div</i> tag.

<!-- [Loaderien](https://webpack.js.org/concepts/loaders/) avulla on mahdollista kertoa webpackille miten tiedostot tulee käsitellä ennen niiden bundlausta. -->
We can use [loaders](https://webpack.js.org/concepts/loaders/) to inform webpack of the files that need to be processed before they are bundled.

<!-- Määritellään projektiimme Reactin käyttämän JSX:n normaaliksi Javascriptiksi muuntava loaderi: -->
Let's configure a loader to our application that transforms the JSX code into regular JavaScript:

```js
const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        query: {
          presets: ['react'],
        },
      },
    ],
  },
}
```

<!-- Loaderit määritellään kentän <i>module</i> alle sijoitettavaan taulukkoon <i>rules</i>. -->
Loaders are defined under the <i>module</i> property in the <i>rules</i> array.

<!-- Yksittäisen loaderin määrittely on kolmiosainen: -->
The definition for a single loader consists of three parts:

```js
{
  test: /\.js$/,
  loader: 'babel-loader',
  query: {
    presets: ['@babel/preset-react']
  }
}
```

<!-- Kenttä <i>test</i> määrittelee että käsitellään <i>.js</i>-päätteisiä tiedostoja, <i>loader</i> kertoo että käsittely tapahtuu [babel-loader](https://github.com/babel/babel-loader):illa. Kenttä <i>query</i> taas antaa loaderille sen toimintaa ohjaavia parametreja. -->
The <i>test</i> property specifies that the loader is for files that have names ending with <i>.js</i>. The <i>loader</i> property specifies that the processing for those files will be done with [babel-loader](https://github.com/babel/babel-loader). The <i>query</i> property is used for specifying parameters for the loader, that configure its functionality.

<!-- Asennetaan loader ja sen tarvitsemat kirjastot <i>kehitysaikaiseksi riippuvuudeksi</i>: -->
Let's install the loader and its required packages as a <i>development dependency</i>:

```js
npm i @babel/core babel-loader @babel/preset-react --save-dev
```

<!-- Nyt bundlaus onnistuu. -->
Bundling the application will now succeed.

<!-- Jos katsomme bundlattua koodia ja editoimme hieman koodin ulkoasua, huomaamme, että komponentti <i>App</i> on muuttunut muotoon -->
If we make some changes to the <i>App</i> component and take a look at the bundled code, we notice that the bundled version of the component looks like this:

```js
const App = () =>
  react__WEBPACK_IMPORTED_MODULE_0___default.a.createElement(
    'div',
    null,
    'hello webpack'
  )
```

<!-- Eli JSX-syntaksin sijaan komponentit luodaan pelkällä Javascriptilla käyttäen Reactin funktiota [createElement](https://reactjs.org/docs/react-without-jsx.html). -->
As we can see from example above, the React elements that were written in JSX are now created with regular JavaScript by using React's [createElement](https://reactjs.org/docs/react-without-jsx.html) function.

<!-- Sovellusta voi nyt kokeilla avaamalla tiedoston <i>build/index.html</i>  selaimen <i>open file</i> -toiminnolla: -->
You can test the bundled application by opening the <i>build/index.html</i> file with the <i>open file</i> functionality of your browser:

![](../images/7/22.png)

<!-- On kuitenkin huomionarvoista, että jos sovelluksemme sisältää <i>async/await</i> -toiminnallisuutta, selaimeen ei piirry mitään. [Konsoliin saapuneen virheviestin googlaaminen](https://stackoverflow.com/questions/33527653/babel-6-regeneratorruntime-is-not-defined) valaisee asiaa. Asian korjaamiseksi on asennettava vielä yksi puuttuva riippuvuus, [@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill). -->
It's worth noting that that if the bundled application's source code uses <i>async/await</i>, the browser will not render anything. [Googling the error message in the console](https://stackoverflow.com/questions/33527653/babel-6-regeneratorruntime-is-not-defined) will shed some light on the issue. We have to install one more missing dependency, that is [@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill):

```
npm install --save-dev @babel/polyfill
```

<!-- Muutetaan vielä tiedostoon <i>webpack.config.js</i> entry-kohdan määrittelyä seuraavasti: -->
Let's make the following changes to the <i>entry</i> property of the webpack configuration object in the <i>webpack.config.js</i> file:

```js
  entry: ['@babel/polyfill', './src/index.js']
```

<!-- Tässä on jo melkein kaikki mitä tarvitsemme React-sovelluskehitykseen. -->
Our configuration contains nearly everything that we need for React development.

<!-- ### Transpilaus -->
### Transpilers

<!-- Prosessista, joka muuttaa Javascriptia muodosta toiseen käytetään englanninkielistä termiä [transpiling](https://en.wiktionary.org/wiki/transpile), joka taas on termi, joka viittaa koodin kääntämiseen (compile) sitä muuntamalla (transform). Suomenkielisen termin puuttuessa käytämme prosessista tällä kurssilla nimitystä <i>transpilaus</i>. -->
The process of transforming code from one form of JavaScript to another is called [transpiling](https://en.wiktionary.org/wiki/transpile). The general definition of the term is to compile source code code by transforming it from one language to another.

<!-- Edellisen luvun konfiguraation avulla siis <i>transpiloimme</i> JSX:ää sisältävän Javascriptin normaaliksi Javascriptiksi tämän hetken johtavan työkalun [babelin](https://babeljs.io/) avulla. -->
By using the configuration from the previous section we are <i>transpiling</i> the code containing JSX into regular JavaScript with the help of [babel](https://babeljs.io/) which is currently the most popular tool for the job.

<!-- Kuten osassa 1 jo mainittiin, läheskään kaikki selaimet eivät vielä osaa Javascriptin uusimpien versioiden ES6:n ja ES7:n ominaisuuksia ja tämän takia koodi yleensä transpiloidaan käyttämään vanhempaa Javascript-syntaksia ES5:ttä. -->
As mentioned in part 1, most browsers do not support the latest features that were introduced in ES6 and ES7, and for this reason the code is usually transpiled to a version of JavaScript that implements the ES5 standard.

<!-- Babelin suorittama transpilointiprosessi määritellään <i>pluginien</i> avulla. Käytännössä useimmiten käytetään valmiita [presetejä](https://babeljs.io/docs/plugins/), eli useamman sopivan pluginin joukkoja. -->
The transpilation process that is executed by Babel is defined with <i>plugins</i>. In practice, most developers use ready-made [presets](https://babeljs.io/docs/plugins/) that are groups of pre-configured plugins.

<!-- Tällä hetkellä sovelluksemme transpiloinnissa käytetään presetiä [@babel/preset-react](https://babeljs.io/docs/plugins/preset-react/): -->
Currently we are using the [@babel/preset-react](https://babeljs.io/docs/plugins/preset-react/) preset for transpiling the source code of our application:

```js
{
  test: /\.js$/,
  loader: 'babel-loader',
  query: {
    presets: ['@babel/preset-react']
  }
}
```

<!-- Otetaan käyttöön preset [@babel/preset-env](https://babeljs.io/docs/plugins/preset-env/), joka sisältää kaiken hyödyllisen, minkä avulla uusimman standardin mukainen koodi saadaan transpiloitua ES5-standardin mukaiseksi koodiksi: -->
Let's add the [@babel/preset-env](https://babeljs.io/docs/plugins/preset-env/) plugin that contains everything needed to take code using all of the latest features and transpile it to code that is compatible with the ES5 standard:

```js
{
  test: /\.js$/,
  loader: 'babel-loader',
  query: {
    presets: ['@babel/preset-env', '@babel/preset-react'] // highlight-line
  }
}
```

<!-- Preset asennetaan komennolla -->
Let's install the preset with the command:

```js
npm install @babel/preset-env --save-dev
```

<!-- Kun nyt transpiloimme koodin, muuttuu se vanhan koulukunnan Javascriptiksi. Komponentin <i>App</i> määrittely näyttää seuraavalta: -->
When we transpile the code it gets transformed into old-school JavaScript. The definition of the transformed <i>App</i> component looks like this:

```js
var App = function App() {
  return _react2.default.createElement('div', null, 'hello webpack')
};
```

<!-- Muuttujan määrittely tapahtuu avainsanan _var_ avulla, sillä ES5 ei tunne avainsanaa _const_. Myöskään nuolifunktiot eivät ole käytössä, joten funktiomäärittely käyttää avainsanaa _function_. -->
As we can see, variables are declared with the _var_ keyword as ES5 JavaScript does not understand the _const_ keyword. Arrow functions are also not used, which is why the function definition used the _function_ keyword.

### CSS

<!-- Lisätään sovellukseemme hieman CSS:ää. Tehdään tiedosto <i>src/index.css</i> -->
Let's add some CSS to our application. Let's create a new <i>src/index.css</i> file:

```css
.container {
  margin: 10;
  background-color: #dee8e4;
}
```

<!-- Määritellään tyyli käytettäväksi komponentissa <i>App</i> -->
Then let's use the style in the <i>App</i> component:

```js
const App = () => (
  <div className="container">
    hello webpack
  </div>
)
```

<!-- ja importataan se tiedostossa <i>index.js</i> -->
And we import the style in the <i>index.js</i> file:

```js
import './index.css'
```

<!-- Transpilointi hajoaa -->
This will cause the transpilation process to break:

![](../images/7/23.png)

<!-- CSS:ää varten onkin otettava käyttöön [css](https://webpack.js.org/loaders/css-loader/)- ja [style](https://webpack.js.org/loaders/style-loader/)-loaderit: -->
When using CSS, we have to use [css](https://webpack.js.org/loaders/css-loader/) and [style](https://webpack.js.org/loaders/style-loader/) loaders:

```js
{
  rules: [
    {
      test: /\.js$/,
      loader: 'babel-loader',
      query: {
        presets: ['@babel/preset-react', '@babel/preset-env'],
      },
    },
    // highlight-start
    {
      test: /\.css$/,
      loaders: ['style-loader', 'css-loader'],
    },
    // highlight-end
  ];
}
```

<!-- [css-loaderin](https://webpack.js.org/loaders/css-loader/) tehtävänä on ladata <i>CSS</i>-tiedostot, ja [style-loader](https://webpack.js.org/loaders/style-loader/) generoi koodiin CSS:t sisältävän <i>style</i>-elementin. -->
The job of the [css loader](https://webpack.js.org/loaders/css-loader/) is to load the <i>CSS</i> files and the job of the [style loader](https://webpack.js.org/loaders/style-loader/) is to generate and inject a <i>style</i> element that contains all of the styles of the application.

<!-- Näin konfiguroituna CSS-määrittelyt sisällytetään sovelluksen Javascriptin sisältävään tiedostoon <i>main.js</i>. Sovelluksen päätiedostossa <i>index.html</i> ei siis ole tarvetta erikseen ladata CSS:ää. -->
With this configuration the CSS definitions are included in the <i>main.js</i> file of the application. For this reason there is no need to separately import the <i>CSS</i> styles in the main <i>index.html</i> file of the application.

<!-- CSS voidaan tarpeen vaatiessa myös generoida omaan tiedostoonsa esim. [mini-css-extract-pluginin](https://github.com/webpack-contrib/mini-css-extract-plugin) avulla. -->
If needed, the application's CSS can also be generated into its own separate file by using the [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin).

<!-- Kun loaderit asennetaan -->
When we install the loaders:

```js
npm install style-loader css-loader --save-dev
```

<!-- bundlaus toimii taas ja sovellus saa uudet tyylit. -->
The bundling will succeed once again and the application gets new styles. 

### Webpack-dev-server

<!-- Sovelluskehitys onnistuu jo, mutta development workflow on suorastaan hirveä (alkaa jo muistuttaa Javalla tapahtuvaa sovelluskehitystä...), muutosten jälkeen koodin on bundlattava ja selain uudelleenladattava jos haluamme testata koodia. -->
The current configuration makes it possible to develop our application but the workflow is awful (to the point where it resembles the development workflow with Java). Every time we make a change to the code we have to bundle it and refresh the browser in order to test the code.

<!-- Ratkaisun tarjoaa [webpack-dev-server](https://webpack.js.org/guides/development/#using-webpack-dev-server). Asennetaan se komennolla -->
The [webpack-dev-server](https://webpack.js.org/guides/development/#using-webpack-dev-server) offers a solution to our problems. Let's install it with the command:

```js
npm install --save-dev webpack-dev-server
```

<!-- Määritellään dev-serverin käynnistävä npm-skripti: -->
Let's define an npm script for starting the dev-server:

```js
{
  // ...
  "scripts": {
    "build": "webpack --mode=development",
    "start": "webpack-dev-server --mode=development" // highlight-line
  },
  // ...
}
```

<!-- Lisätään tiedostoon <i>webpack.config.js</i> kenttä <i>devServer</i> -->
Let's also add a new <i>devServer</i> property to the configuration object in the <i>webpack.config.js</i> file:

```js
const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'main.js',
  },
  // highlight-start
  devServer: {
    contentBase: path.resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
  },
  // highlight-end
  // ...
};
```

<!-- Komento _npm start_ käynnistää nyt dev-serverin porttiin, eli sovelluskehitys tapahtuu avaamalla tuttuun tapaan selain osoitteeseen <http://localhost:3000>. Kun teemme koodiin muutoksia, reloadaa selain automaattisesti itsensä. -->
The _npm start_ command will now start the dev-server at the port 3000, meaning that our application will be available by visiting <http://localhost:3000> in the browser. When we make changes to the code, the browser will automatically refresh the page.

<!-- Päivitysprosessi on nopea, dev-serveriä käytettäessä webpack ei bundlaa koodia normaaliin tapaan tiedostoksi <i>main.js</i>, bundlauksen tuotos on olemassa ainoastaan keskusmuistissa. -->
The process for updating the code is fast. When we use the dev-server, the code is not bundled the usual way into the <i>main.js</i> file. The result of the bundling exists only in memory.

<!-- Laajennetaan koodia muuttamalla komponentin <i>App</i> määrittelyä seuraavasti: -->
Let's extend the code by changing the definition of the <i>App</i> component as shown below:

```js
import React, {useState} from 'react'

const App = () => {
  const [counter, setCounter] = useState(0)

  return (
    <div className="container">
      hello webpack {counter} clicks
      <button onClick={() => setCounter(counter + 1)} >press</button>
    </div>
  )
}

export default App
```

<!-- Kannattaa huomata, että virheviestit eivät renderöidy selaimeen kuten create-react-app:illa tehdyissä sovelluksissa, eli on seurattava tarkasti konsolia: -->
It's worth noticing that the error messages don't show up the same way as they did with our applications that were made using create-react-app. For this reason we have to pay more attention to the console:

![](../images/7/24.png)

<!-- Sovellus toimii hyvin ja kehitys on melko sujuvaa. -->
The application works nicely and the development workflow is quite smooth.

<!-- ### Sourcemappaus -->
### Source maps

<!-- Erotetaan napin klikkauksenkäsittelijä omaksi funktioksi ja talletetaan tilaan <i>values</i> aiemmat laskurin arvot: -->
Let's extract the click handler into its own function and store the previous value of the counter into its own <i>values</i> state:

```js
const App = () => {
  const [counter, setCounter] = useState(0)
  const [values, setValues] = useState() // highlight-line

  const handleClick = () => {
    setCounter(counter + 1)
    setValues(values.concat(counter)) // highlight-line
  }

  return (
    <div className="container">
      hello webpack {counter} clicks
      <button onClick={handleClick} >press</button>
    </div>
  )
}
```

<!-- Sovellus ei enää toimi, ja konsoli kertoo virheestä -->
The application no longer works and the console will display the following error:

![](../images/7/25.png)

<!-- Tiedämme tietenkin nyt että virhe on metodissa onClick, mutta jos olisi kyse suuremmasta sovelluksesta, on virheilmoitus sikäli hyvin ikävä, että sen ilmoittama paikka: -->
We know that the error is in the onClick method, but if the application was any larger the error message would be quite difficult to track down:

<pre>
App.js:27 Uncaught TypeError: Cannot read property 'concat' of undefined
    at handleClick (App.js:27)
</pre>

<!-- ei vastaa alkuperäisen koodin virheen sijaintia. Jos klikkaamme virheilmoitusta, huomaamme, että näytettävä koodi on jotain ihan muuta kuin kirjoittamamme koodi: -->
The location of the error indicated in the message does not match the actual location of the error in our source code. If we click the error message, we notice that the displayed source code does not resemble our application code:

![](../images/7/26.png)

<!-- Haluamme tietenkin, että virheilmoitusten yhteydessä näytetään kirjoittamamme koodi. -->
Of course, we want to see our actual source code in the error message.

<!-- Korjaus on onneksi hyvin helppo, pyydetään webpackia generoimaan bundlelle ns. [source map](https://webpack.js.org/configuration/devtool/), jonka avulla bundlea suoritettaessa tapahtuva virhe on mahdollista <i>mäpätä</i> alkuperäisen koodin vastaavaan kohtaan. -->
Luckily fixing the error message in this respect is quite easy. We will ask webpack to generate a so-called [source map](https://webpack.js.org/configuration/devtool/) for the bundle, that makes it possible to <i>map errors</i> that occur during the execution of the bundle to the corresponding part in the original source code.

<!-- Source map saadaan generoitua lisäämällä konfiguraatioon kenttä <i>devtool</i> ja sen arvoksi 'source-map': -->
The source map can be generated by adding a new <i>devtool</i> property to the configuration object with the value 'source-map':

```js
const config = {
  entry: './src/index.js',
  output: {
    // ...
  },
  devServer: {
    // ...
  },
  devtool: 'source-map', // highlight-line
  // ..
};
```

<!-- Konfiguraatioiden muuttuessa webpack tulee käynnistää uudelleen, on tosin mahdollista konfiguroida webpack tarkkailemaan konfiguraatioiden muutoksia, mutta emme tee sitä. -->
Webpack has to be restarted when we make changes to its configuration. It is also possible to make webpack watch for changes made to itself but we will not do that this time.

<!-- Nyt virheilmoitus on hyvä -->
The error message is now a lot better:

![](../images/7/27.png)

<!-- Source mapin käyttö mahdollistaa myös chromen debuggerin luontevan käytön -->
Generating the source map also makes it possible to the Chrome debugger:

![](../images/7/28.png)

<!-- Korjataan bugi alustamalla tila <i>values</i> tyhjäksi taulukoksi: -->
Let's fix the bug by initializing the state of <i>values</i> as an empty array:

```js
const App = () => {
  const [counter, setCounter] = useState(0)
  const [values, setValues] = useState([])
  // ...
}
```

<!-- ### Koodin minifiointi -->
### Minifying the code

<!-- Kun sovellus viedään tuotantoon, on siis käytössä tiedostoon <i>main.js</i> webpackin generoima koodi. Vaikka sovelluksemme sisältää omaa koodia vain muutaman rivin, on tiedoston <i>main.js</i> koko 807027 tavua, sillä se sisältää myös kaiken React-kirjaston koodin. Tiedoston koollahan on sikäli väliä, että selain joutuu lataamaan tiedoston kun sovellusta aletaan käyttämään. Nopeilla internetyhteyksillä 807027 tavua ei sinänsä ole ongelma, mutta jos mukaan sisällytetään enemmän kirjastoja, alkaa sovelluksen lataaminen pikkuhiljaa hidastua etenkin mobiilikäytössä. -->
When we deploy the application to production, we are using the <i>main.js</i> code bundle that is generated by webpack. The size of the <i>main.js</i> file is 807027 bytes even though our application only contains a few lines of our own code. The large file size is due to the fact that the bundle also contains the source code for the entire React library. The size of the bundled code matters since the browser has to load the code when the application is first used. With high-speed internet connections 807027 bytes is not an issue, but if we were to keep adding more external dependencies, loading speeds could become an issue particularly for mobile users.

<!-- Jos tiedoston sisältöä tarkastelee, huomaa että sitä voisi optimoida huomattavasti koon suhteen esim. poistamalla kommentit. Tiedostoa ei kuitenkaan kannata lähteä optimoimaan käsin, sillä tarkoitusta varten on olemassa monia työkaluja. -->
If we inspect the contents of the bundle file, we notice that it could be greatly optimized in terms of file size by removing all of the comments. There's no point in manually optimizing these files, as there are many existing tools for the job.

<!-- Javascript-tiedostojen optimointiprosessista käytetään nimitystä <i>minifiointi</i>. Alan johtava työkalu tällä hetkellä lienee [UglifyJS](http://lisperator.net/uglifyjs/). -->
The optimization process for JavaScript files is called <i>minification</i>. One of the leading tools intended for this purpose is [UglifyJS](http://lisperator.net/uglifyjs/).

<!-- Webpackin versiosta 4 alkaen pluginia ei ole tarvinnut konfiguroida erikseen, riittää että muutetaan tiedoston <i>package.json</i> määrittelyä siten, että koodin bundlaus tapahtuu <i>production</i>-moodissa: -->
Starting from version 4 of webpack, the minification plugin does not require additional configuration to be used. It is enough to modify the npm script in the <i>package.json</i> file to specify that webpack will execute the bundling of the code in <i>production</i> mode:

```json
{
  "name": "webpack-osa7",
  "version": "0.0.1",
  "description": "practising webpack",
  "scripts": {
    "build": "webpack --mode=production", // highlight-line
    "start": "webpack-dev-server --mode=development"
  },
  "license": "MIT",
  "dependencies": {
    // ...
  },
  "devDependencies": {
    // ...
  }
}
```

<!-- Kun sovellus bundlataan uudelleen, pienenee tuloksena oleva <i>main.js</i> mukavasti -->
When we bundle the application again, the size of the resulting <i>main.js</i> decreases substantially:

```js
$ ls -l main.js
-rw-r--r--  1 mluukkai  984178727  126388 Feb  6 18:27 main.js
```

<!-- Minifioinnin lopputulos on kuin vanhan liiton c-koodia, kommentit ja jopa turhat välilyönnit ja rivinvaihdot on poistettu ja muuttujanimet ovat yksikirjaimisia: -->
The output of the minification process resembles old-school C code; all of the comments and even unnecessary whitespace and newline characters have been removed, and variable names have been replaced with a single character.

```js
function h(){if(!d){var e=u(p);d=!0;for(var t=c.length;t;){for(s=c,c=[];++f<t;)s&&s[f].run();f=-1,t=c.length}s=null,d=!1,function(e){if(o===clearTimeout)return clearTimeout(e);if((o===l||!o)&&clearTimeout)return o=clearTimeout,clearTimeout(e);try{o(e)}catch(t){try{return o.call(null,e)}catch(t){return o.call(this,e)}}}(e)}}a.nextTick=function(e){var t=new Array(arguments.length-1);if(arguments.length>1)
```

<!-- ### Sovelluskehitys- ja tuotantokonfiguraatio -->
### Development and production configuration

<!-- Lisätään sovellukselle backend. Käytetään jo tutuksi käynyttä muistiinpanoja tarjoavaa palvelua. -->
Next, let's add a backend to our application and by repurposing the now-familiar note application backend.

<!-- Talletetaan seuraava sisältö tiedostoon <i>db.json</i> -->
Let's store the following content in the <i>db.json</i> file:

```json
{
  "notes": [
    {
      "important": true,
      "content": "HTML on helppoa",
      "id": "5a3b8481bb01f9cb00ccb4a9"
    },
    {
      "important": false,
      "content": "Mongo osaa tallettaa oliot",
      "id": "5a3b920a61e8c8d3f484bdd0"
    }
  ]
}
```

<!-- Tarkoituksena on konfiguroida sovellus webpackin avulla siten, että paikallisesti sovellusta kehitettäessä käytetään backendina portissa 3001 toimivaa json-serveriä. -->
Our goal is to configure the application with webpack in such a way, that when used locally the application uses the json-server available in port 3001 as its backend.

<!-- Bundlattu tiedosto laitetaan sitten käyttämään todellista, osoitteessa <https://radiant-plateau-25399.herokuapp.com/api/notes> olevaa backendia. -->
The bundled file will then be configured to use the backend available at the <https://radiant-plateau-25399.herokuapp.com/api/notes> url.

<!-- Asennetaan <i>axios</i>, käynnistetään json-server ja tehdään tarvittavat lisäykset sovellukseen. Vaihtelun vuoksi muistiinpanojen hakeminen palvelimelta on toteutettu [custom hookin](/osa5/custom_hookit) _useNotes_ avulla: -->
We will install <i>axios</i>, start the json-server, and then make the necessary changes to the application. For the sake of changing things up, we will fetch the notes from the backend with our [custom hook](/osa5/custom_hookit) called _useNotes_:

```js
import React, { useState, useEffect } from 'react'
import axios from 'axios';

// highlight-start
const useNotes = (url) => {
  const [notes, setNotes] = useState([])

  useEffect(() => {
    axios.get(url).then(response => {
      setNotes(response.data)
    })
  }, [url])

  return notes
}
// highlight-end

const App = () => {
  const [counter, setCounter] = useState(0)
  const [values, setValues] = useState([])
  const url = 'http://localhost:3001/notes'
  const notes = useNotes(url) // highlight-line

  const handleClick = () => {
    setCounter(counter + 1)
    setValues(values.concat(counter))
  }

  return (
    <div className="container">
      hello webpack {counter} clicks
      <button onClick={handleClick} >press</button>
      <div>{notes.length} notes on server {url}</div> // highlight-line
    </div>
  )
}

export default App
```

<!-- Koodissa on nyt kovakoodattuna sovelluskehityksessä käytettävän palvelimen osoite. Miten saamme osoitteen hallitusti muutettua osoittamaan internetissä olevaan backendiin bundlatessamme koodin? -->
The address of the backend server is currently hardcoded in the application code. How can we change the address in a controlled fashion to point to the production backend server when the code is bundled for production?

<!-- Muutetaan <i>webpack.config.js</i> oliosta [funktioksi](https://webpack.js.org/configuration/configuration-types/#exporting-a-function): -->
Let's change the configuration object in the <i>webpack.config.js</i> file to be a function instead of an object:

```js
const path = require('path');

const config = (env, argv) => {
  return {
    entry: './src/index.js',
    output: {
      // ...
    },
    devServer: {
      // ...
    },
    devtool: 'source-map',
    module: {
      // ...
    },
    plugins: [
      // ...
    ],
  }
}

module.exports = config
```

<!-- Määrittely on muuten täysin sama, mutta aiemmin eksportattu olio on nyt määritellyn funktion paluuarvo. Funktio saa parametrit <i>env</i> ja <i>argv</i>, joista jälkimmäisen avulla saamme selville npm-skriptissä määritellyn <i>moden</i>. -->
The definition remains almost exactly the same, except for the fact that the configuration object is now returned by the function. The function receives the two parameters, <i>env</i> and <i>argv</i>, the second of which can be used for accessing the <i>mode</i> that is defined in the npm script. 

<!-- Webpackin [DefinePlugin](https://webpack.js.org/plugins/define-plugin/):in avulla voimme määritellä globaaleja <i>vakioarvoja</i>, joita on mahdollista käyttää bundlattavassa koodissa. Määritellään nyt vakio <i>BACKEND\_URL</i>, joka saa eri arvon riippuen siitä ollaanko kehitysympäristössä vai tehdäänkö tuotantoon sopivaa bundlea: -->
We can also use webpack's [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) for defining <i>global default constants</i> that can be used in the bundled code. Let's define a new global constant <i>BACKEND\_URL</i>, that gets a different value depending on the environment that the code is being bundled for:

```js
const path = require('path')
const webpack = require('webpack') // highlight-line

const config = (env, argv) => {
  console.log('argv', argv.mode)

  // highlight-start
  const backend_url = argv.mode === 'production'
    ? 'https://radiant-plateau-25399.herokuapp.com/api/notes'
    : 'http://localhost:3001/notes'
  // highlight-end

  return {
    entry: './src/index.js',
    output: {
      path: path.resolve(__dirname, 'build'),
      filename: 'main.js'
    },
    devServer: {
      contentBase: path.resolve(__dirname, 'build'),
      compress: true,
      port: 3000,
    },
    devtool: 'source-map',
    module: {
      // ...
    },
    // highlight-start
    plugins: [
      new webpack.DefinePlugin({
        BACKEND_URL: JSON.stringify(backend_url)
      })
    ]
    // highlight-end
  }
}

module.exports = config
```

<!-- Määriteltyä vakiota käytetään koodissa seuraavasti: -->
The global constant is used in the following way in the code:

```js
const App = () => {
  const [counter, setCounter] = useState(0)
  const [values, setValues] = useState([])
  const notes = useNotes(BACKEND_URL) // highlight-line

  // ...
  return (
    <div className="container">
      hello webpack {counter} clicks
      <button onClick={handleClick} >press</button>
      <div>{notes.length} notes on server {BACKEND_URL}</div> // highlight-line
    </div>
  )
}
```

<!-- Jos kehitys- ja tuotantokonfiguraatio eriytyvät paljon, saattaa olla hyvä idea [eriyttää konfiguraatiot](https://webpack.js.org/guides/production/) omiin tiedostoihinsa. -->
If the configuration for development and production differs a lot, it may be a good idea to [separate the configuration](https://webpack.js.org/guides/production/) of the two into their own files.

<!-- Tuotantoversiota eli bundlattua sovellusta on mahdollista kokeilla lokaalisti suorittamalla komento -->
We can inspect the bundled production version of the application locally by executing the following command in the <i>build</i> directory:

```js
npx static-server
```

<!-- hakemistossa <i>build</i>, jolloin sovellus käynnistyy oletusarvoisesti osoitteeseen <http://localhost:9080>. -->
By default the bundled application will be available at <http://localhost:9080>.

### Polyfill

<!-- Sovelluksemme on valmis ja toimii muiden selaimien kohtuullisen uusilla versiolla, mutta Internet Explorerilla sovellus ei toimi. Syynä tähän on se, että _axiosin_ ansiosta koodissa käytetään [Promiseja](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), mikään IE:n versio ei kuitenkaan niitä tue: -->
Our application is finished and works with all relatively recent versions of modern browsers, with the exception of Internet Explorer. The reason for this is that because of _axios_ our code uses [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise), and no existing version of IE supports them:

![](../images/7/29.png)

<!-- On paljon muutakin standardissa määriteltyjä asioita, joita IE ei tue, esim. niinkin harmiton komento kuin taulukoiden [find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) ylittää IE:n kyvyt: -->
There are many other things in the standard that IE does not support. Something as harmless as the [find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) method of JavaScript arrays exceeds the capabilities of IE:

![](../images/7/30.png)

<!-- Tälläisessä tilanteessa normaali koodin transpilointi ei auta, sillä transpiloinnissa koodia käännetään uudemmasta Javascript-syntaksista vanhempaan, selaimien paremmin tukemaan syntaksiin. Promiset ovat syntaktisesti täysin IE:n ymmärrettävissä, IE:ltä vain puuttuu toteutus promisesta, samoin on tilanne taulukoiden suhteen, IE:llä taulukoiden _find_ on arvoltaan <i>undefined</i>. -->
In these situations it is not enough to transpile the code, as transpilation simply transforms the code from a newer version of JavaScript to an older one with wider browser support. IE understands Promises syntactically but it simply has not implemented their functionality. The _find_ property of arrays in IE is simply <i>undefined</i>.

<!-- Jos haluamme sovelluksen IE-yhteensopivaksi, tarvitsemme [polyfilliä](https://remysharp.com/2010/10/08/what-is-a-polyfill), eli koodia, joka lisää puuttuvan toiminnallisuuden vanhempiin selaimiin. -->
If we want the application to be IE-compatible we need to add a [polyfill](https://remysharp.com/2010/10/08/what-is-a-polyfill), which is code that adds the missing functionality to older browsers.

<!-- Polyfillaus on mahdollista hoitaa [Webpackin ja Babelin avulla](https://babeljs.io/docs/usage/polyfill/) tai asentamalla yksi monista tarjolla olevista polyfill-kirjastoista. -->
Polyfills can be added with the help of [webpack and Babel](https://babeljs.io/docs/usage/polyfill/) or by installing one of many existing polyfill libraries.

<!-- Esim. kirjaston [promise-polyfill](https://www.npmjs.com/package/promise-polyfill) tarjoaman polyfillin käyttö on todella helppoa, koodiin lisätään seuraava: -->
The polyfill provided by the [promise-polyfill](https://www.npmjs.com/package/promise-polyfill) library is easy to use, we simply have to add the following to our existing application code:

```js
import PromisePolyfill from 'promise-polyfill'

if (!window.Promise) {
  window.Promise = PromisePolyfill
}
```

<!-- Jos globaalia _Promise_-olioa ei ole olemassa, eli selain ei tue promiseja, sijoitetaan polyfillattu promise globaaliin muuttujaan. Jos polyfillattu promise on hyvin toteutettu, muun koodin pitäisi toimia ilman ongelmia. -->
If the global _Promise_ object does not exist, meaning that the browser does not support Promises, the polyfilled Promise is stored in the global variable. If the polyfilled Promise is implemented well enough, the rest of the code should work without issues.

<!-- Kattavahko lista olemassaolevista polyfilleistä löytyy [täältä](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills). -->
One exhaustive list of existing polyfills can be found [here](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-browser-Polyfills).

<!-- Selaimien yhteensopivuus käytettävien API:en suhteen kannattaakin tarkistaa esim. [https://caniuse.com](https://caniuse.com)-sivustolta tai [Mozillan sivuilta](https://developer.mozilla.org/en-US/). -->
The browser compatibility of different API's can be checked by visiting [https://caniuse.com](https://caniuse.com) or [Mozilla's website](https://developer.mozilla.org/en-US/).

### Eject

<!-- Create-react-app käyttää taustalla webpackia. Jos peruskonfiguraatio ei riitä, on projektit mahdollista [ejektoida](https://github.com/facebook/create-react-app/blob/master/packages/react-sscripts/template/README.md#npm-run-eject), jolloin kaikki konepellin alla oleva magia häviää, ja konfiguraatiot tallettuvat hakemistoon <i>config</i> ja muokattuun <i>package.json</i>-tiedostoon. -->
The create-react-app tool uses webpack behind the scenes. If the default configuration is not enough, it is possible to [eject](https://github.com/facebook/create-react-app/blob/master/packages/react-sscripts/template/README.md#npm-run-eject) the project which will get rid of all of the black magic, and the default configuration files will be stored in the <i>config</i> directory and in a modified <i>package.json</i> file.

<!-- Jos create-react-app:illa tehdyn sovelluksen ejektoi, paluuta ei ole, sen jälkeen kaikesta konfiguroinnista on huolehdittava itse. Konfiguraatiot eivät ole triviaaleimmasta päästä ja create-react-appin ja ejektoinnin sijaan parempi vaihtoehto saattaa joskus olla tehdä itse koko webpack-konfiguraatio. -->
If you eject an application created with create-react-app, there is no return and all of the configuration will have to be maintained manually. The default configuration is not trivial, and instead of ejecting from a create-react-app application, a better alternative may be to write your own webpack configuration from the get-go.

<!-- Ejektoidun sovelluksen konfiguraatioiden lukeminen on suositeltavaa ja sangen opettavaista! -->
Going through and reading the configuration files of an ejected application is still recommended and extremely educational.

</div>

<div class="tasks">

<!-- ### Tehtäviä -->
### Exercises

<!-- Koko osan lopun [blogilistaa laajentavassa tehtäväsarjassa](/osa7/tehtavia_blogilistan_laajennus) on myös yksi webpackiin liittyvä tehtävä. -->
One exercise related to the topics presented here, can be found at the end of this course material section in the exercise set [for extending the blog list application](/osa7/tehtavia_blogilistan_laajennus).


</div>
