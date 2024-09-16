
# Grupo 6

## PIN 1

### Pasos

1. para ahorrar tiempo:
```sh
docker pull mguazzardo/pipe-seg:latest
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
jenkins: http://localhost:8090/view/Grupo%206/
registry: http://localhost:8089/

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
