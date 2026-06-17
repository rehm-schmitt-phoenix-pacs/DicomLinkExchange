DICOM Link Exchange (DLX)
=========================

A collaborative project to work on the specifications for the DIN workgroup "QR-Codes/Online-Bereitstellung von Bilddaten" resp. DIN-Normenausschuss Radiologie (NAR) NA 080-00-04-10 AK "Online Bereitstellung von Bilddaten".

The main goal is to provide an [OpenAPI](https://www.openapis.org/) specification to support the specified workflow.

## OpenApi

This project uses [OpenAPI](https://www.openapis.org/) (v3) for all specifications. 

To edit and preview the specification, the `dicomLinkExchange.yaml` can be opened with e.g. the [Swagger Editor](https://editor.swagger.io/). 

## Demo Server

This project includes a minimal demo server for prototyping of the api and specifications.

**THE IMPLEMENTATION OF THE DEMO SERVER IN NOT COMPLETE AND ONLY FOR DEMO PURPOSES!**

[![No Maintenance Intended](http://unmaintained.tech/badge.svg)](http://unmaintained.tech/)

### Available token

Following demo token are available (hard coded).

###### Birthday-Token: ABC-S1Z-98A   

    Question: "Wann haben Sie Geburtstag?"
    Answer: "19700101"
    
###### Password-Token: DF2-11Z-KS4    

    Question: "Wie lautet Ihr Einmal-Passwort?"
    Answer: "132-238-252"

###### Custom-Token: HDF-34F-HK6

    Question 1: "Wann war ihre Untersuchung?"
    Answer 1: "20240419"
 
    Question 2:  "Wie heißt Ihr behandelnder Arzt?"
    Answer 2: "Dr. Mayer"

##### Building (maven/java)

To generate the api using the yaml-specifications and build the server use

    mvn clean compile package
    
### Running (java)

To start the server, launch the main class `de.fschili.dlx.DlxDemoServer` from your IDE

    de.fschili.dlx.DlxDemoServer
    
or via spring-boot with

    mvn spring-boot:run  
    
respectively lauch the generated jar from commandline using

    java -jar dicom-link-exchange.jar

The demo server will run at 

    http://localhost:3000/
    
and will respond to the defined requests.

Konfiguration can be made at

    resources/application.properties

### Documentation (Swagger)

The documentation can be browsed at 

    http://localhost:3000/swagger-ui/index.html

## Docker

Der Demo-Server kann als Docker-Container gebaut und gestartet werden.

### Bauen

    docker compose build

### Starten

    docker compose up -d

Der Server läuft dann auf http://localhost:3000/.

### Auf einem entfernten Linux-Host nutzen

1. **Image exportieren und übertragen** (falls kein Zugriff auf eine Registry besteht):

   Auf dem Entwicklungsrechner:

       docker save dicomlinkexchange-dlx-server | gzip > dlx-server.tar.gz
       scp dlx-server.tar.gz user@zielhost:/tmp/

   Auf dem Zielhost:

       docker load < /tmp/dlx-server.tar.gz

2. **Oder: Image in eine Registry pushen** (z.B. Docker Hub, Gitea Container Registry):

       docker tag dicomlinkexchange-dlx-server registry.example.com/dicom-link-exchange:latest
       docker push registry.example.com/dicom-link-exchange:latest

   Auf dem Zielhost dann:

       docker pull registry.example.com/dicom-link-exchange:latest
       docker run -d -p 3000:3000 --name dlx-server registry.example.com/dicom-link-exchange:latest

3. **Direkt per docker-compose auf dem Zielhost** (wenn das Image in einer Registry liegt):

   `docker-compose.yml` auf den Zielhost kopieren und die `build`-Zeile durch `image` ersetzen:

       services:
         dlx-server:
           image: registry.example.com/dicom-link-exchange:latest
           container_name: dicom-link-exchange
           ports:
             - "3000:3000"
           restart: unless-stopped

   Dann starten mit:

       docker compose up -d

4. **Zugriff von extern prüfen:**

       curl http://<zielhost-ip>:3000/dlx/v1/api_info
