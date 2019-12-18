# Quarkus-utprøvingsgruppe

Quarkus er en stack for å bygge webapplikasjoner basert på java-biblioteker og -standarder.
En liste over alt som er offisielt støttet av utvidelser finnes på [quarkus.io/extensions](https://quarkus.io/extensions/).
Vi skal nå opprette en applikasjon, legge til noen av utvidelsene, og utforske både utivkling og generering av produksjonsbygg.

Det antas at du har docker og Maven >= 3.5.3. 
(`mvnw` blir lagt inn ved generering av prosjekt, så om du har for gammel versjon kan du jukse ved å 
[laste ned quarkus-quickstarts](https://github.com/quarkusio/quarkus-quickstarts/archive/master.zip)) og pakke ut `getting-started`.

Det er også mulig generere et prosjekt på [code.quarkus.io](https://code.quarkus.io).

## Opprett et prosjekt
Vi starter med [Quarkus oppstartsguide](https://quarkus.io/guides/getting-started). Dette er også en fin oversikt 
over hvilke ferdigbygde moduler som finnes.

Om du ønsker bruke Gradle kan du se på følge [disse stegene](https://quarkus.io/guides/gradle-tooling) 

Generer en applikasjon med 
```bash
mvn io.quarkus:quarkus-maven-plugin:1.0.1.Final:create \
    -DprojectGroupId=no.java.trd \
    -DprojectArtifactId=quarkus-starter \
    -DclassName="no.java.trd.quarkus.GreetingResource" \
    -Dpath="/hello"
```
og start den med `mvn compile quarkus:dev` (`./mvnw` kan også brukes) og besøk http://localhost:8080/ og http://localhost:8080/hello

Quarkus sjekker om noe har endret seg ved behandling av hver request. Åpne `GreetingResource` og endre teksten som returneres.

Ved generering ble det også opprettet en test for `/hello`. 
Endre `GreetingResourceTest` slik at den passerer og kjør den. (I IDE eller med mvn test)
Ved kjøring av `quarkus:dev` startes debug-lytter, så om du trenger steppe gjennom `GreetingResource.hello()`
er det bare å koble debugger til på port 5005.

Stopp `quarkus:dev` og bygg applikasjonen med `mvn package`. 
Den bygde applikasjonen kan kjøres med `java -jar target/quarkus-starter-1.0-SNAPSHOT-runner.jar`. 
Fra [getting-started](https://quarkus.io/guides/getting-started#packaging-and-run-the-application):
```
The Class-Path entry of the MANIFEST.MF from the runner jar explicitly lists the jars from the lib directory. 
So if you want to deploy your application somewhere, you need to copy the runner jar as well as the lib directory
```
Dette betyr at det er kun vår kode som ligger i jar-filen, og at alle avhengigheter må ligge i en `lib`-mappe i samme 
mappe som jar-filen. (`target/quarkus-starter-1.0-SNAPSHOT-runner.jar` + `target/lib`)

Men det er også mulig å bygge en überjar ved å legge til

```xml
<configuration>
  <uberJar>true</uberJar>
</configuration>
```
i pom.xml for `quarkus-maven-plugin`.
For å verifisere at dette er tilfelle kan du slette `target/lib` og prøve kjøre `java -jar target/quarkus-starter-1.0-SNAPSHOT-runner.jar`, 
så legge til `<uberJar>true</uberJar>` og bygge og kjøre på nytt.

## Komponenter og injiseringer
Nå kan du lage [en service](https://quarkus.io/guides/getting-started#using-injection) som tar seg av hilsning når 
vi tar inn et navn. Du trenger ikke stoppe applikasjonen for at endringene skal få effekt.
Oppdater `GreetingResourceTest` til å teste det nye endepunktet.

For mer om CDI [ta en titt her](https://quarkus.io/guides/cdi-reference).

Noen ganger er det ønskelig å [overstyre oppførsel ved testing](https://quarkus.io/guides/getting-started-testing#mock-support). 
Opprett en egen implementasjon av `GreetingService` som overstyrer hva `GreetingResource.greeting(name)` returnerer når testene kjører.
`MockGreetingService.greeting(name)` kan for eksempel legge til prefix «HALLO TEST!»

## Oppstart og stopping
Det er veldig lett å legge til kode som kjøres ved [oppstart og stopping](https://quarkus.io/guides/lifecycle#listening-for-startup-and-shutdown-events) av applikasjonen.

## Konfigurasjon
Quarkus bruker [MicroProfile Config](https://microprofile.io/project/eclipse/microprofile-config) til konfigurasjonshåndtering. 
[Denne guiden](https://quarkus.io/guides/config#injecting-configuration-value) viser 
forskjellige måter `@ConfigProperty` kan brukes for å ta inn verdier fra konfigurasjon. 

Endre `GreetingResource.hello` til å ta inn en prefix, suffix og melding. Endre gjerne `src/main/resources/application.properties`
under kjøring og se at output oppdateres ved kjøring.

Dersom du kopierer feltene annotert med `@ConfigProperty` til GreetingResourceTest kan du bruke dem til å sjekke at responsen er rett.
Dersom `src/test/resources/application.properties` finnes vil den bli brukt i stedet for `src/main/resources/application.properties`. 
Disse blir ikke flettet, så det ikke mulig å delvis overstyre konfig.

## JSON-støtte
Slik applikasjonen vår er nå støtter den ikke å generere JSON fra POJOs.

Legg til utvidelse for `jsonb` med 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-resteasy-jsonb"
```
Eller Jackson med 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-resteasy-jackson"
```
Om applikasjonen allerede kjører trenger du ikke stoppe eller restarte, det skjer automatisk.

Lag noen domeneklasser og endepunkt for å liste dem ut. Eller bare stjel dem [fra denne guiden](https://quarkus.io/guides/rest-json#creating-your-first-json-rest-service)
Legg til et felt i domeneklassen din og sjekk at endringen blir tatt i bruk uten restart.

Lag en test som tester utlisting og å legge til domeneobjekter, [som her](https://github.com/quarkusio/quarkus-quickstarts/blob/master/rest-json-quickstart/src/test/java/org/acme/rest/json/FruitResourceTest.java)

## REST-Klient
Microprofile Rest Client kan generere klienter basert på interfacer og domeneklasser. 
Legg til utvidelse for `rest-client` med 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-rest-client"
```

Kopier oppsett fra [denne guiden](https://quarkus.io/guides/rest-client#setting-up-the-model) for å konsumere [restcountries.eu](https://restcountries.eu/)
Legg til konfigurasjon for klienten i `application.properties`
```properties
no.java.trd.quarkus.CountriesService/mp-rest/url=https://restcountries.eu/rest
```

Sjekk at alt virker ved å åpne http://localhost:8080/country/name/norway eller et annet land.

## Websockets
Demoapplikasjonen vår trenger å implementere chat. 
Legg til Websocketstøtte med 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-undertow-websockets"
```
Kopier `ChatSocket.java`(til src/main/java/no/java/trd/quarkus/) og `index.html`(til src/main/resources/META-INF/resources) 
fra [Websocket-guiden](https://quarkus.io/guides/websockets#handling-web-sockets)

Eksempel på hvordan teste et Websocket-endepunkt kan du se [her](https://github.com/quarkusio/quarkus-quickstarts/blob/master/websockets-quickstart/src/test/java/org/acme/websocket/ChatTestCase.java)

## Database
[Databaseaksess](https://quarkus.io/guides/datasource)
[Reactive-databaseaksess](https://quarkus.io/guides/reactive-sql-clients)

## Dokumentasjon med OpenAPI 3 og SwaggerUI
Microprofile OpenAPI støtter å generere OpenAPI 3-dokumenter basert på annotasjoner og domeneklasser. 
Legg til utvidelsen og restart applikasjonen.
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-smallrye-openapi"
``` 
Og sjekk [guide](https://quarkus.io/guides/openapi-swaggerui).
Nå er dokumentasjonen for alle endepunktene i applikasjonen tilgjengelig på http://localhost:8080/openapi. 
Ved lokalt kjøring er Swagger tilgjengelig på http://localhost:8080/swagger-ui. 
Om swagger-ui skal være med i produksjon kan du legge til `quarkus.swagger-ui.always-include=true` i konfigurasjonen.

## Helsesjekker
Det er kjekt å vite om applikasjonen vi har laget er frisk eller ikke. Det er det så klart en utvidelse for!
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-smallrye-health"
```
Når du nå starter applikasjonen kan du sjekke helsesjekkene på http://localhost:8080/health.
Det finnes ikke noen sjekker ennå, så opprett en [som alltid er frisk](https://quarkus.io/guides/microprofile-health#creating-your-first-health-check)

Utvid med en helsesjekk som sporadisk feiler.

## Metrikker
Fra [Metrics Guide](https://quarkus.io/guides/microprofile-metrics)
```text
MicroProfile Metrics allows applications to gather various metrics and statistics that provide insights into what is happening inside the application.

The metrics can be read remotely using JSON format or the OpenMetrics format, so that they can be processed by additional tools such as Prometheus, and stored for analysis and visualisation. 
```
Legg til utvidelsen 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-smallrye-metrics"
```
og dryss `@Counted`, `@Timed`, og `@Gauge` på endepunktene i applikasjonen. 
Gjør noen requester til alle endepunktene. 

Metrikkene er tilgjengelige på http://localhost:8080/metrics.
De metrikkene vi har definert er på http://localhost:8080/metrics/application.
Andre innebygde metrikker finnes på http://localhost:8080/metrics/vendor og http://localhost:8080/metrics/base. 

Når du åpner disse i nettleseren er det på [Prometheus](https://prometheus.io)-format.

Last ned `prometheus.yml`og start Prometheus med 
```bash
docker run --net="host" -p 9090:9090 -v /FULLSTI_HER/prometheus.yml:/etc/prometheus/prometheus.yml \
       prom/prometheus
```
og åpne localhost:9090 og lag regler for grafing av metrikkene.
 
Grafana kan [importere fra Prometheus](https://prometheus.io/docs/visualization/grafana/)
Start Grafana med 
```bash
docker run --net="host" --name=grafana -p 3000:3000 grafana/grafana
```
og lag grafer!

## Opentracing
[MicroProfile OpenTracing](https://quarkus.io/guides/opentracing) instrumenterer koden vår og eksponerer det til 
[Jaeger](https://www.jaegertracing.io).

Legg til utvidelsen
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-smallrye-opentracing"
```
og legg innstillinger i `application.properties`
```properties
quarkus.jaeger.service-name=quarkus-starter
quarkus.jaeger.sampler-type=const
quarkus.jaeger.sampler-param=1
quarkus.jaeger.endpoint=http://localhost:14268/api/traces
```

Start Jaeger med 
```bash
docker run -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 -p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp -p 5778:5778 -p 16686:16686 -p 14268:14268 -p 9411:9411 jaegertracing/all-in-one:latest
```
Jaeger kan nå åpnes i nettleseren på http://localhost:16686/search

Gjør noen kall til alle endepunktene.

## Feiltoleranse
[MicroProfile Fault toleranse](https://github.com/eclipse/microprofile-fault-tolerance/) gir et sett annotasjoner vi
kan bruke til å definere «fallback»-metoder dersom noe bruker for lang tid eller feiler.
Disse er definert:
- `@TimeOut`: «Define a duration for timeout»
- `@RetryPolicy`: «Define a criteria on when to retry»
- `@Fallback`: «Provide an alternative solution for a failed execution»
- `@Bulkhead`: «Isolate failures in part of the system while the rest part of the system can still function»
- `@CircuitBreaker`: «Offer a way of fail fast by automatically failing execution to prevent the system overloading and indefinite wait or timeout by the clients»

Legg til utvidelsen 
```bash
mvn quarkus:add-extension -Dextensions="io.quarkus:quarkus-smallrye-fault-tolerance"
```

[Denne](https://www.tomitribe.com/blog/tomee-a-tutorial-on-microprofile-fault-tolerance/) og 
[denne](https://developers.redhat.com/blog/wp-content/uploads/2017/11/microprofile-fault-tolerance-spec.pdf) siden går gjennom hvordan 
de forskjellige annotasjonene brukes.

## Hva nå?
* Se om du finner noe annet interessant i [listen over guider](https://quarkus.io/guides/)
* Kikk på [Quarkus Cheat-Sheet](https://lordofthejars.github.io/quarkus-cheat-sheet/#quarkuscheatsheet)
* Gå gjennom en [annen workshop](https://quarkus.io/quarkus-workshops/super-heros/)
