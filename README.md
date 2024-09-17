
# Grupo 6

## PIN 1

### Modificaciones

**Jenkinsfile**

se elimina la linea 15: <https://github.com/EducacionMundose/PIN1/blob/master/Jenkinsfile#L15>, quedando en el fork: <https://github.com/hdbarrios/devops-g6-pin1/blob/master/Jenkinsfile>

<table>
  <tr>
    <th>Code original</th>
    <th>Code modificado</th>
  </tr>
  <tr>
    <td>
      <pre>
stage('Building image') {
    steps {
        sh '''
        cd webapp
        docker build -t testapp .
        '''
    }
}
      </pre>
    </td>
    <td>
      <pre>
stage('Building image') {
    steps {
        sh '''
        docker build -t testapp .
        '''
    }
}
      </pre>
    </td>
  </tr>
</table>

**Jenkins.seg**
- se elimina el `cd webapp` ya que no estice en el repo
- se modifica el stage "Vulnerability Scan - Docker", ademas se adiciona timeout y un combobox para seleccionar el nivel de severity de la validacion de la imagen
<table>
  <tr>
    <th>Code original</th>
    <th>Code modificado</th>
  </tr>
  <tr>
    <td>
      <pre>
stage('Building image') {
    steps {
        sh '''
        cd webapp
        docker build -t testapp .
        '''
    }
}
      </pre>
    </td>
    <td>
      <pre>
stage('Building image') {
    steps {
        sh '''
        docker build -t testapp .
        '''
    }
}
      </pre>
    </td>
  </tr>
  <tr>
    <td>
      <pre>
stage('Vulnerability Scan - Docker ') {
    steps {
        sh "docker run  -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image --severity=critical 127.0.0.1:5000/mguazzardo/testapp"        
    }
}
     </pre>
    </td>
    <td>
      <pre>
stage('Vulnerability Scan - Docker ') {
    steps {
        timeout(time: 20, unit: 'MINUTES') {
          script {
            def severity = ''
            if (params.SEVERITY == 'ALL') {
            severity = 'UNKNOWN,LOW,MEDIUM,HIGH,CRITICAL'
            } else {
                severity = params.SEVERITY
            }
            sh "docker run  -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.55.1 image --severity=${severity} 127.0.0.1:5000/mguazzardo/testapp"
            }
        }
    }
}
      </pre>
    </td>
    </tf>
</table>

```groovy
  parameters {
    choice(
      name: 'SEVERITY',
      choices: ['LOW', 'MEDIUM', 'HIGH', 'CRITICAL', 'ALL', 'UNKNOWN' ],
      description: 'Select the severity level for Trivy scan'
    )
  }
```
---

### comprobar images mas recientes de node-alpine:

editar el Dockerfile del repo del PIN y cambiar la linea:

`FROM node:11.1.0-alpine`

por una version mas reciente:

`FROM node:18.20.4-alpine` [ver](https://hub.docker.com/_/node/tags)

de esta forma se testea el stage **Vulnerability Scan - Docker**

otro ejemplo de busqueda:
```sh 
$ curl -s "https://hub.docker.com/v2/repositories/library/node/tags/?page_size=20" | jq -r '.results[] | "node:\(.name)"'
node:hydrogen-alpine3.20
node:hydrogen-alpine
node:18.20.4-alpine3.20
node:18.20.4-alpine
node:18.20-alpine3.20
node:18.20-alpine
node:18-alpine3.20
node:18-alpine
node:lts-alpine3.20
node:lts-alpine3.19
node:lts-alpine
node:iron-alpine3.20
node:iron-alpine3.19
node:iron-alpine
node:hydrogen-alpine3.19
node:current-alpine3.20
node:current-alpine3.19
node:current-alpine
node:alpine3.20
node:alpine3.19

```

### Pasos para levantar ambiente:

1. para ahorrar tiempo:
    ```sh
    docker pull docker.io/hdbarrios/hdbarrios/jenkins-grupo-6-pin
    -- alternativo: docker pull mguazzardo/pipe-seg:latest
    docker pull registry:2
    docker pull joxit/docker-registry-ui:latest
    docker pull aquasec/trivy:0.55.1
    ```

2. clona el fork: https://github.com/hdbarrios/devops-g6-pin1.git

   ```sh
   git clone https://github.com/hdbarrios/devops-g6-pin1.git
   cd devops-g6-pin1
   ```

3. ejecutar docker compose:

    ```sh
    $ docker compose up -d
    [+] Running 4/4
     ⠿ Network devops-g6-pin1_pin1-network  Created                         0.1s
     ⠿ Container registry-ui                Started                         0.5s
     ⠿ Container jenkins-curso              Started                         0.4s
     ⠿ Container docker-registry            Started                         0.5s

    $ docker compose ps
    NAME                COMMAND                  SERVICE             STATUS              PORTS
    docker-registry     "/entrypoint.sh /etc…"   registry            running             0.0.0.0:5000->5000/tcp, :::5000->5000/tcp
    jenkins-curso       "/usr/bin/tini -- /u…"   jenkins             running             0.0.0.0:8088->8080/tcp, :::8088->8080/tcp
    registry-ui         "/docker-entrypoint.…"   registry-ui         running             0.0.0.0:8089->80/tcp, :::8089->80/tcp

    ```

Luego abrir en el browser:

Jenkins: <http://localhost:8088/view/Grupo%206/>

Registry: <http://localhost:8089/>

---

### crear una imagen personalizada:

```sh
# download imagen de jenkins
docker pull docker.io/hdbarrios/jenkins-grupo-6-pin
# ejecutar una contenedor con la imagen de jenkins (temporalmente)
docker run -d --name jenkins-grupo-6-pin -p 8090:8080 hdbarrios/jenkins-grupo-6-pin
# descargar el home de jenkins: (en un data local o del repo)
docker cp $(docker ps | grep jenkins-grupo-6-pin | awk '{print$1}'):/var/jenkins_home ./data/

docker stop $(docker ps | grep jenkins-grupo-6-pin | awk '{print$1}')
docker rm $(docker ps | grep jenkins-grupo-6-pin | awk '{print$1}')

#ejecutar un contenedor atachando un volumen al .data/jenkins_home:
docker run -d --name jenkins-grupo-6-pin -p 8090:8080 -v ./data/jenkins_home:/var/jenkins_home hdbarrios/jenkins-grupo-6-pin

#hacer todas las configuraciones necesarias en el jenkins:
#abrir en el browser: http://localhost:8090

#al terminar:

docker commit $(docker ps | grep jenkins-grupo-6-pin | awk '{print$1}')

docker tag hdbarrios/jenkins-grupo-6-pin:latest hdbarrios/jenkins-grupo-6-pin:latest

docker push hdbarrios/jenkins-grupo-6-pin:latest

```

---

### como borrar una imagen del registry local:
```sh
$ curl -I -H "Accept: application/vnd.docker.distribution.manifest.v2+json" -X GET http://localhost:5000/v2/mguazzardo/testapp/manifests/latest
    HTTP/1.1 200 OK
    Access-Control-Allow-Headers: Authorization
    Access-Control-Allow-Headers: Content-Type
    Access-Control-Allow-Methods: GET
    Access-Control-Allow-Methods: PUT
    Access-Control-Allow-Methods: POST
    Access-Control-Allow-Methods: DELETE
    Access-Control-Allow-Methods: OPTIONS
    Access-Control-Allow-Origin: http://localhost:8089
    Content-Length: 1786
    Content-Type: application/vnd.docker.distribution.manifest.v2+json
    Docker-Content-Digest: sha256:4322bde9991b43a05f2607a9f1e2c379a374eb44ec7c636a464d47c17eda9aa3  # <<<< este es el digest >>>>
    Docker-Distribution-Api-Version: registry/2.0
    Etag: "sha256:4322bde9991b43a05f2607a9f1e2c379a374eb44ec7c636a464d47c17eda9aa3"
    X-Content-Type-Options: nosniff
    Date: Mon, 16 Sep 2024 22:13:29 GMT

$ curl -X DELETE http://localhost:5000/v2/mguazzardo/testapp/manifests/sha256:4322bde9991b43a05f2607a9f1e2c379a374eb44ec7c636a464d47c17eda9aa3
```
