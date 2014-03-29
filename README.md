# Building a real time risk analysis system in java devoxx france 2014

building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014
 	
Notre application de Trading d'Analyse de Risque temps réel doit maintenir une base de 100 millions de lignes (Xmx=64G) avec du flux de 5000 message/sec. Nos utilisateurs (répartis à Paris, Londres, NY et HK) suivent plus de 100 indicateurs tels que la sensibilité au marché et les estimations de gains : ils sont agrégés en temps réel avec des règles métiers complexes. Nous vous proposons un retour d’expérience afin d'expliquer nos choix technologiques : le design de l'application, la librairie ActivePivot pour agréger des indicateurs en temps réel et la diffusion des impacts en temps-réel.

## Presentation du métier, et objectifs du projet (problematique de l international)
(Alexandre)
 
## Fonctionnalites business avancees: Grouping de deal, rebucketing, indicateur calculés temps reel PnL = sensi * (closing - last)
(Alexnadre)

## Architecture global :
(Alexandre)
GREAT (Global Risk Exposure Analysis Tool) Server.
2 servers avec les mêmes données, internatial, plusieurs type de cube, faulttolerance, pas de pricing

## Stack technologique (Tibco?, Java + UI en C#, ActivePivot, JProfiler, MAT, Spring, guava, disruptor)
(Alexandre)
### Server
- Webapp dans Tomcat 7, jdk 7
- Classique (Spring/JavaConfig, CXF/WS avec Rest/Soap, JMS (tibco-ems), Guava)
- Moins classique (Disruptor, java.util.concurrent, reuters, flux direct des bourses, serialisation custom protobuf ou autre)
- Test : mockito, assertj, wiremock.
- Outillage : JProfiler, MAT, maven / jenkins / sonar,

### Client
- UI dans excel ou GUI lourde C# via WS/Soap longpolling / ou REST.

## Indicateurs de volumetries 
(Alexandre) 
(nb message, taille message, nb de deal, nb d instrument, nb users)

## ActivePivot: Aggregation et requete temps reel (Benoit)
- Analogie F1
(Benoit)

## Performance : 
(benoit) 
SLA (del), optimisations (tout est en asynchrone). Usage d un framework specifique? Non
- asynchronisement
- pool d'objets pour les parties flux temps avec delta.
- sampling dynamique 
JVM: options, temps de GC (exemple du gars qui dit que l on est lent, pile au moment ou on a GC) (e.g. le mec qui a groupé en moins d une seconde). tuning Old / Young.

## Best practises / Design : (Alexandre)
- Isolation (mode dégradé pour chacun des systemes sources), chargement de fichier, reload à chaud.
- Configuration du projet: javaConfig a eu un fort impact (si xml, dans votre lib application.xml, web.xml, avec des référence à des classes, ce n'est pas correct)
- Monotoring REST jenkins health check, metrics
- Beta en UAT avec flux de prod puis DR pour les bétas Users.
- Package par feature pas technique, nombre de pom.xml, 1 par livrable ou lib avec cycle de vie différent.
- Tester/Tester/Tester (65/70 % de coverage).

## Future améliorations techniques dans les prochains mois
(Alexandre)
- Tomcat 8 embedded, jdk 1.8.
- Migration ActivePivot 5.0
- Deploiement automatique (deployit).
- Meilleur monitoring avec logstash/elasticsearch/kibana
- Refactoring plus orienté feature pour avoir des cubes type.
