1. Workflow CI para el proyecto de frontend

Debes crear un nuevo workflow que se dispare cuando haya cambios en el proyecto hangman-front y exista una nueva pull request 
(deben darse las dos condiciones a la vez). El workflow ejecutará las siguientes operaciones:

    Build del proyecto
    Ejecución de los test unitarios

Andes de nada creo una rama para incluir el workflow de integración continua.

Luego creamos el directorio .github/wokflows en la raiz del proyecto. En esa ruta tendremos para esta primera 
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

Además de crear el fichero ci.yaml para que el workflow se ejecutase automáticamente he tenido que incluir un comentario en el index.html para que incluya un cambio
en el proyecto hangman-front ya que de lo contrario no se ejecutaría.
Una vez hecho lo anterior, he creado un pull-request y al lazar las Actions ha fallado el test.Muestro logs

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
expect(items).toHaveLength(2);  que es lo que espera el test y subir los cambios al repositorio,  en el apartado Actions de GitHub el workflow de integración continua se ve que se realiza sin problemas.

<img width="1912" height="642" alt="9_1_Captura de pantalla_2026-05-08_16-49-16" src="https://github.com/user-attachments/assets/627a49f5-6239-4332-b769-569f5a49d6f0" />


Por último, hago merge a main y elimino la rama usada para la parte de integración continua 

<b>2. Workflow CD para el proyecto de frontend</b>

Crea un nuevo workflow que se dispare manualmente y haga lo siguiente:

    Crear una nueva imagen de Docker
    Publicar dicha imagen en el container registry de GitHub

Para realizar esta parte creo otra rama para continuos delivery.
En esta rama, creo el fichero cd.yaml con el siguiente contenido

 <pre>
 
name: Despliegue continuo (usando acciones de docker)

on:
  workflow_dispatch:  #esto indica que se ejecutará manualmente mediante un botón en el apartado Actios de GitHub

jobs:
  buildAndPushImage:
    runs-on: ubuntu-latest  #runner

    steps:
      - name: Checkout
        uses: actions/checkout@v6  #descarga el código en el runner
      - name: Login into Docker Hub  
        uses: docker/login-action@v4  #esta Accion definida por GitHub nos permite iniciar sesion en dockerHub con nuestras credenciales.
        with:
          username: andresluque     #nombre de usuario de Docker Hub
          password: ${{ secrets.DOCKER_PASSWORD }}   #contraseña del usuario de Docker Hub, para ello creamos un Secret en GitHub, en Settings del repositorio.
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v4  #action que hace build del proyecto
      - name: Build and push Docker Image
        uses: docker/build-push-action@v7    #action que hace el push en docker hub
        with:
          context: ./hangman-front           #indica que la ruta del contexto
          push: true                         #indica que queremos que se haga push a docker hub
          tags: andresluque/hangman-api-cep-grupo-a:latest    nombre de la imagen en docker hub
          file: ./hangman-front/Dockerfile     #ruta del fichero Dockerfile
 </pre>

Una vez que tenemos el fichero, subimos los cambios al repositorio de Github (no se ejecutará nada en Actios)
Para que salga el botón y podamos ejecutarlo manualmente, debemos hacer una pull-request y hacer merge a main.
Una vez realizado lo anterior ya veremos en la pestaña Actions el nuevo workflow. También se muestra en la imgagen el botón para ejecutar el workflow manualmente.
<img width="1931" height="585" alt="9_Captura de pantalla_2026-05-08_16-35-54" src="https://github.com/user-attachments/assets/813477c8-345d-46ca-acd3-2c364217e02e" />

Al pulsarlo comienza la ejecución

<img width="1912" height="897" alt="9_Captura de pantalla_2026-05-08_16-36-39" src="https://github.com/user-attachments/assets/17d98aa2-723e-46c4-b9df-7021b96e3a9b" />

Por útlimo vemos que se ha realizado correctamente el build y el push a docker hub.

<img width="1889" height="1043" alt="9_Captura de pantalla_2026-05-08_16-38-13" src="https://github.com/user-attachments/assets/3e50071e-0bba-4171-9ad9-73a4cd11afd5" />

Borramos la rama creada para la continuos delivery.



