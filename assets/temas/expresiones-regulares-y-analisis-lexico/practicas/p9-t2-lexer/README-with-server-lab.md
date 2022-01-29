## Práctica: Analizador Léxico para Un Subconjunto de JavaScript (Variante de antigua p4-t2-lexer)

Vamos a trabajar a partir de este repo de Douglas Crockford:

-  [https://github.com/douglascrockford/TDOP](https://github.com/douglascrockford/TDOP)
-  Autor: [Douglas Crockford](http://www.crockford.com/), [douglas@crockford.com en la Wikipedia](https://en.wikipedia.org/wiki/Douglas_Crockford)
-  Fecha que figura en el repo: 2010-11-12
-  Vea una versión modificada en funcionamiento en [http://crguezl.github.io/ull-etsii-grado-pl-minijavascript/](http://crguezl.github.io/ull-etsii-grado-pl-minijavascript/). En esta versión el analizador léxico usa expresiones regulares.


### Descripción:

-   [tdop.html](http://crguezl.github.io/ull-etsii-grado-pl-minijavascript/tdop.html) contains a description of Vaughn Pratt’s Top Down Operator
    Precedence, and describes the parser whose lexer we are going to
    write in this lab. Is a simplified version of JavaScript.
-   The file [`index.html`](https://github.com/douglascrockford/TDOP/blob/master/index.html) parses [`parse.js`](https://github.com/douglascrockford/TDOP/blob/master/parse.js) and displays its AST.
-   The page depends on on [`parse.js`](https://github.com/douglascrockford/TDOP/blob/master/parse.js) and [`tokens.js`](https://github.com/douglascrockford/TDOP/blob/master/tokens.js).
-   The file [`parse.js`](https://github.com/douglascrockford/TDOP/blob/master/parse.js) contains the Simplified JavaScript parser.
-   [tokens.js](https://github.com/douglascrockford/TDOP/blob/master/tokens.js) produces an array of token objects from a string. 

### Requisitos

1. Douglas Crockford escribió este analizador léxico sin usar expresiones
regulares. Reescriba el analizador léxico en [tokens.js](https://github.com/douglascrockford/TDOP/blob/master/tokens.js) usando expresiones regulares.
2.  Evite que se hagan copias de la cadena siendo procesada. Muévase
    dentro de la misma cadena usando `lastIndex`. Quizá usar la opción sticky le ayude.
    Tiene una solución dada por el profesor en 
    - [https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/blob/gh-pages/tokens.js](https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/blob/gh-pages/tokens.js)
3. Modifique la solución de Crockford usado regexps en [tokens.js](https://github.com/douglascrockford/TDOP/blob/master/tokens.js)
4.  Añada un server ([vea aquí un ejemplo](https://github.com/ULL-ESIT-PL-1617/evaluar-manejo-de-cookies-y-sessions-en-expressjs-alu0100825510)) para el HTML y haga el despliegue de su aplicación en la máquina virtual del [iaas](https://github.com/SYTW/iaas-ull-es) o en [Heroku](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/recursos/heroku.html)
6. Opcional: Use [sessions](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/cookies/) para controlar quien accede a la aplicación. Puede ver un ejemplo de como hacerlo en los ficheros:
  - [ULL-ESIT-PL-1617/evaluar-manejo-de-cookies-y-sessions-en-expressjs](https://github.com/ULL-ESIT-PL-1617/evaluar-manejo-de-cookies-y-sessions-en-expressjs-alu0100825510)
  - En el módulo npm [@ull-esit-pl/auth](https://www.npmjs.com/package/@ull-esit-pl/auth) encontrará la solución al problema explicada en clase
7. En el `README.md` escriba un tutorial con lo que ha aprendido en esta práctica
8. Cuando haga la entrega indique los enlaces a los repos (analizador) así como a los despliegues. Ponga también el enlace al despliegue en el README de su repo.

### Recursos

* Lexical Analysis
  * Una solución incompleta  de esta práctica se encuentra en:
     - [https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/tree/gh-pages](https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/tree/gh-pages) en github.
     - Veala en funcionamiento en [http://crguezl.github.io/ull-etsii-grado-pl-minijavascript/](http://crguezl.github.io/ull-etsii-grado-pl-minijavascript/)
  * El método `tokens` retorna el array de tokens [https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/blob/gh-pages/tokens.js](https://github.com/crguezl/ull-etsii-grado-pl-minijavascript/blob/gh-pages/tokens.js)
* Programación Web
  * [ejs](https://ejs.co/)
  * Cookies, Sessions, Authentication
    * Ejemplo de server con cookies y sessions: [ULL-ESIT-PL-1617/evaluar-manejo-de-cookies-y-sessions-en-expressjs](https://github.com/ULL-ESIT-PL-1617/evaluar-manejo-de-cookies-y-sessions-en-expressjs-alu0100825510).
    * [Cookies (apuntes 16/17)](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/cookies/)
    * [Sessions y Authentication (apuntes 16/17)](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/authentication/)
  * Express
    * [Apuntes de Express 1617](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/express/)
  * Despliegues
    * [Como Desplegar una Aplicación Web en iaas.ull.es](https://github.com/SYTW/iaas-ull-es)
    * [Apuntes de Heroku](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/recursos/heroku.html)
  * Webpack
    * [Webpack guide: "getting started"](https://webpack.js.org/guides/getting-started/)
    * [Youtube video Webpack 4 por Fatz](https://youtu.be/vF2emKbaP4M)
* Expresiones Regulares
  * [Apuntes: Expresiones Regulares](../apuntes/regexp/README.md)
  * [Eloquent JS: Chapter 9: Regular Expressions](http://eloquentjavascript.net/09_regexp.html)
* XRegexp
  * [XRegexp](http://xregexp.com/) 
    - [XRegexp en GitHub](https://github.com/slevithan/xregexp)
    - [xregexp-all.js](https://unpkg.com/xregexp/xregexp-all.js)
  * [JavaScript y Unicode](https://github.com/ULL-ESIT-PL/unicode-js) (Repo en GitHub unicode-js)
  * [Repositorio con ejemplos de uso de XRegExp](https://github.com/ULL-ESIT-GRADOII-PL/xregexp-example) 
  * [Ejemplos de extensiones de XRegExp para Unicode](https://github.com/ULL-ESIT-GRADOII-PL/xregexp-example/blob/gh-pages/unicode.js)
  * [Módulo @ull-esit-pl/uninums](https://www.npmjs.com/package/@ull-esit-pl/uninums)

    ```js
    > uninums = require("@ull-esit-pl/uninums")
    { normalSpaces: [Function: normalSpaces],
      normalDigits: [Function: normalDigits],
      parseUniInt: [Function: parseUniInt],
      parseUniFloat: [Function: parseUniFloat],
      sortNumeric: [Function: sortNumeric] }
    > uninums.parseUniInt('६.६')
    6
    > uninums.parseUniFloat('६.६')
    6.6
    > uninums.parseUniFloat('६.६e६')
    6600000
    > uninums.sortNumeric(['٣ dogs','١٠ cats','٢ mice']) 
    [ '٢ mice', '٣ dogs', '١٠ cats' ]
    > uninums.normalDigits('٢ mice')
    '2 mice'
    > uninums.normalDigits('٣ dog')
    '3 dog'
    > uninums.normalDigits('١٠ cats')
    '10 cats'
    > uninums.normalDigits('٠۴६')
    '046'
    ```
* Diseño
  * [Apuntes: Code Smells](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/patterns/codesmell.html)
  * [Principios de Diseño](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/patterns/designprinciples.html)
  * [Patrones de Diseño](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/patterns/)
  * [Strategy Pattern](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/patterns/strategypattern.html)
  * [Práctica: Eliminando Switch Smell](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/practicas/practicanoswitchsmell.html)
* Creación de Módulos
  * Véase la sección
    - [Creación de Paquetes y Módulos en NodeJS](https://crguezl.github.io/ull-esit-1617/_book/apuntes/npm/nodejspackages.html) GitHub
    - [Creación de Paquetes y Módulos en NodeJS](https://casianorodriguezleon.gitbooks.io/ull-esit-1617/content/apuntes/npm/nodejspackages.html) GitBook
  * [Ejemplo de módulo npm: ULL-ESIT-DSI-1617/scapegoat](https://github.com/ULL-ESIT-DSI-1617/scapegoat)
  * [prueba-scapegoat. Ejemplo de programa cliente](https://github.com/ULL-ESIT-DSI-1617/prueba-scapegoat)
  * [Repo combinado librería + cliente de prueba](https://github.com/ULL-ESIT-DSI-1617/create-a-npm-module)
    - [Video que explica como hacer un repo combinado](https://youtu.be/17cZY3na3As)
  * [Solución:  repo ULL-ESIT-GRADOII-PL/modulos](https://github.com/ULL-ESIT-GRADOII-PL/modulos/tree/master)
  * [Como funciona *require*](https://youtu.be/qffmnSCRR3c) Video del profesor
  * [Best practice: Specify global dependencies in your gulpfile](https://stackoverflow.com/questions/14657170/installing-global-npm-dependencies-via-package-json)
  * [Node.js — How to test your new NPM module without publishing it every 5 minutes](https://medium.com/@the1mills/how-to-test-your-npm-module-without-publishing-it-every-5-minutes-1c4cb4b369be)
  * [Best practice: Better local require() paths for Node.js](https://gist.github.com/branneman/8048520):
     - When the directory structure of your Node.js **application** (not library!) has some depth, you end up with a lot of annoying relative paths in your require calls like:
      ```
       var Article = require('../../../models/article');
      ```
     Those suck for maintenance and they're ugly.




### Notas para el Profesor

```bash
[~/srcPLgrado/lexical_analysis_top_down_operator_precedence(gh-pages)]$ pwd -P
/Users/casiano/local/src/javascript/PLgrado/lexical_analysis_top_down_operator_precedence
[~/campus-virtual/1819/pl1819/introduccion/tema2-expresiones-regulares-y-analisis-lexico/practicas/p4-t2-lexer/pl1718-solutions(master)]$ pwd -P
/Users/casiano/local/academica/centros/ETSII/GRADO/PL/campus-virtual/tema2-regexp-y-lexico/practica-analisis-lexico-tdop/solutions

```
* sol-david dibad
  * [sol-davafons](https://github.com/ULL-ESIT-PL-1819/p4-t2-lexer-Dibad)
* sol-cas
  * [sol-cas](https://github.com/ULL-ESIT-PL-1819/analizador-lexico-para-js)
* sol-ai
  * [sol-ai auth](https://github.com/ULL-ESIT-PL-1718/authentication-angeligareta)
  * [sol-angeli](https://github.com/ULL-ESIT-PL-1718/analizador-lexico-para-js-angeligareta)
* sol-carlos
  * [sol-ca-auth](https://github.com/ULL-ESIT-PL-1718/alu0100966589-AuthModule)
  * [sol-ca](https://github.com/ULL-ESIT-PL-1718/analizador-lexico-para-js-alu0100966589)
* sol-daute
  * [sol-da-auth](https://github.com/ULL-ESIT-PL-1718/auth-alu0100973914)
  * [sol-da](https://github.com/ULL-ESIT-PL-1718/analizador-lexico-para-js-alu0100973914)
* sol-cristian
  * [sol-cri](https://github.com/ULL-ESIT-PL-1718/analizador-lexico-para-js-alu0100945850)
  * [sol-cri-auth](https://github.com/ULL-ESIT-PL-1718/auth-alu0100945850)




