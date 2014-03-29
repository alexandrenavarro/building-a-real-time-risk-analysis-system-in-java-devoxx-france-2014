# Building a real time risk analysis system in java devoxx france 2014

building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014
 	
Notre application de Trading d'Analyse de Risque temps réel doit maintenir une base de 100 millions de lignes (Xmx=64G) avec du flux de 5000 message/sec. Nos utilisateurs (répartis à Paris, Londres, NY et HK) suivent plus de 100 indicateurs tels que la sensibilité au marché et les estimations de gains : ils sont agrégés en temps réel avec des règles métiers complexes. Nous vous proposons un retour d’expérience afin d'expliquer nos choix technologiques : le design de l'application, la librairie ActivePivot pour agréger des indicateurs en temps réel et la diffusion des impacts en temps-réel.

## Presentation du métier, et objectifs du projet 
(Alexandre)
Problematique de l international, Traders.
 
## Fonctionnalites business avancees : 
(Alexandre)
- Grouping de deal
- Rebucketing
- Indicateur calculés temps reel PnL = sensi * (closing - last)

## Architecture global :
(Alexandre)
GREAT (Global Risk Exposure Analysis Tool) Server.
2 servers (main, DR) avec les mêmes données à NY, Paris, HK
Pas de proxy, client qui décident suivant.
Plusieurs type de cube, faulttolerance, pas de pricing.

## Stack technologique
(Alexandre)
### Server
- Webapp dans un Tomcat 7, jdk 7
- Classique (Spring/JavaConfig, CXF/WS avec Rest/Soap, JMS (tibco-ems), Guava)
- Moins classique (Disruptor, java.util.concurrent, reuters, flux direct des bourses, serialisation custom protobuf ou autre)
- Test : junit, mockito, assertj, wiremock.
- Outillage : JProfiler, MAT, maven / jenkins / sonar,

### Client
- UI dans excel ou GUI lourde C# via WS/Soap longpolling / ou REST.

## Indicateurs de volumetries 
(Alexandre) 
(nb faits, nb message de risk, nb msg flux, taille message, nb de deal, nb d instrument, nb users)

## ActivePivot: Aggregation et requete temps reel
(Benoit)
- Analogie F1
- Threading, long pooling, immutabilité dans activepivot / threading (1 requete , n thread)
//TODO

## Performance : 
(Benoit)
SLA (del), optimisations (tout est en asynchrone). Usage d un framework specifique? Non. (Queue type Disruptor, ExecutorService,)
- asynchronisement
- pool d'objets pour les parties flux temps avec sérialization delta, attention a l'allocation d'objet
- sampling dynamique pour le flux, insertion dans le cube.
- JVM: options, temps de GC (exemple du gars qui dit que l on est lent, pile au moment ou on a GC) (e.g. le mec qui a groupé en moins d une seconde). tuning Old / Young, Azul.

## Best practises / Design : 
(Alexandre/Benoit)
- Isolation (mode dégradé pour chacun des systemes sources), chargement de fichier, reload à chaud, Couplage faible message entre application avec serialization key/value (sans schema fort type corba).

- Configuration du projet: javaConfig a eu un fort impact (si xml, dans votre lib application.xml, web.xml, avec des référence à des classes, ce n'est pas correct)
- Monotoring REST jenkins health check, metrics
- Beta en UAT avec flux de prod puis DR pour les bétas Users.
- Package par feature pas technique, nombre de pom.xml, 1 par livrable ou lib avec cycle de vie différent.
- Tester/Tester/Tester (65/70 % de coverage).

## Future améliorations techniques dans les prochains mois
(Alexandre)
- Tomcat 8 embedded, jdk 1.8, Migration ActivePivot 5.0
- Deploiement automatique (deployit).
- Meilleur monitoring avec logstash/elasticsearch/kibana
- Refactoring package / livrable plus orienté feature pour avoir des cubes type genre Rates, Credit, ForexOption, XAsset ...
