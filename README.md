1. Workflow CI para el proyecto de frontend

Debes crear un nuevo workflow que se dispare cuando haya cambios en el proyecto hangman-front y exista una nueva pull request 
(deben darse las dos condiciones a la vez). El workflow ejecutará las siguientes operaciones:

    Build del proyecto
    Ejecución de los test unitarios

Lo primero es tener el directorio .github/wokflows en la raiz del proyecto. En esa ruta tendremos para esta primera 
parte del laboratorio el fichero ci.yaml que va a encargarse del workflow de integración continua.

<pre>
name: Integración continua   #nombre del workflow

on:    #eventos que disparan el workflow
  push:    #cuando cambie el código de la parte hagman-front y se haga un push
    branches: [ main ]
    paths: [ 'hangman-front/**' ]
  pull_request: #cuando cuando tenga una pull request
    branches: [ main ]
    paths: [ 'hangman-front/**' ]

jobs:
  audit:
    runs-on: ubuntu-latest #sistema donde de realizarán las acciones

    steps:
      - name: Checkout
        uses: actions/checkout@v6  #descarga el código en el runner
      - name: Build and test       
        working-directory: ./hangman-front  #indica el directorio de trabajo desde donde debe ejecutar los siguientes comandos
        run: |
          npm ci         #instala dependencias 
          npm run build --if-present  #ejecuta el script build si existe
          npm run test                #ejecuta los tests
</pre>

He creado un pull-request y al lazar las Actions ha fallado el test.Muestro logs

<pre>
  [18:17] > hangman-front@1.0.0 test
> jest -c ./config/test/jest.js
FAIL src/components/start-game.spec.tsx
 StartGame component specs
 ✕ should display a list of topics (59 ms)
 ● StartGame component specs › should display a list of topics
 expect(received).toHaveLength(expected)
 Expected length: 1
 Received length: 2
 Received array: [<li>topic A</li>, <li>topic B</li>]
 14 | const items = await screen.findAllByRole('listitem');
 15 |
 > 16 | expect(items).toHaveLength(1);    #LINEA DONDE SE PRODUCE EL EROR EN EL CÓDIGO
 | ^
 17 | expect(getTopicsStub).toHaveBeenCalled();
 18 | });
 19 | });
 at _callee$ (src/components/start-game.spec.tsx:16:19)
 at tryCatch (src/components/start-game.spec.tsx:2:1)
 at Generator._invoke (src/components/start-game.spec.tsx:2:1)
 at Generator.next (src/components/start-game.spec.tsx:2:1)
 at asyncGeneratorStep (src/components/start-game.spec.tsx:2:1)
 at _next (src/components/start-game.spec.tsx:2:1)
Test Suites: 1 failed, 1 total
Tests: 1 failed, 1 total
Snapshots: 0 total
Time: 1.216 s
Ran all test suites.
Error: Process completed with exit code 1.
</pre>

Una vez cambiada la línea 16 del fichero src/components/start-game.spec.tsx
expect(items).toHaveLength(1); por
expect(items).toHaveLength(2);  que es lo que espera el test el workflow de integración continua se realiza sin problemas.

Hago merge a main de la rama y elimino la rama.


