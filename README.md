# Building a real time risk analysis system in java devoxx france 2014

building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014
 	
Notre application de Trading d'Analyse de Risque temps réel doit maintenir une base de 100 millions de lignes (Xmx=64G) avec du flux de 5000 message/sec. Nos utilisateurs (répartis à Paris, Londres, NY et HK) suivent plus de 100 indicateurs tels que la sensibilité au marché et les estimations de gains : ils sont agrégés en temps réel avec des règles métiers complexes. Nous vous proposons un retour d’expérience afin d'expliquer nos choix technologiques : le design de l'application, la librairie ActivePivot pour agréger des indicateurs en temps réel et la diffusion des impacts en temps-réel.

## Presentation du métier, et objectifs du projet 
(Alexandre)
Comment etait implementé un tel projet auparavant? 
Quelles technos? Quelles difficultés? Limitations? (Typiquement pas temps reel, et moins de trade intraday). 
Anaylsis Services, cube avec notification tibcorv.
Problèmatique, inutilisable à l'internation (Paris centrique), pas de, performance non (retard de quelques secondes à plusieurs minutes).
Comment va evoluer le projet dans le futur ? (Accroissement du nombre de trade, et de leur variété (Cross Assets))
1M de faits dans le cube au début, aujourd'hui, 50M-100M de faits dans le cube alimenté par plusieurs système.
Criticité de la production.

## Fonctionnalites business avancees : 
(Alexandre)
- Indicateur préagrégé en temps tel que la sensi, position ou bookvalue envoyé par des calculateurs de risque
- Indicateur calculés temps reel PnL = sensi * (closing - last) pour avoir des estimations temps.
- Bucketing par Maturité, parametrable par utilisateur
- Grouping de deal, parametrable par utilisateur (seul partie en écriture, sinon tout en lecture)
- Limite / StressTest (en cours de dev).

## Architecture global :
(Alexandre)
GREAT (Global Risk Exposure Analysis Tool) Server est avec un cube en mémoire temps réel qui contient les risques.
2 servers (main, DR) avec les mêmes données à NY, Paris, HK
Pas de proxy type Apache, NGinx ou HAProxy, client qui décident suivant.

Recoit des risques depuis un calculateur, souscrit au last Price et closing.
Plusieurs type de cube (Rates, XAsset, ... ), pas de pricing, que de l'estimation / approximation.

## Stack technologique
(Alexandre)
### Server
- Webapp dans un Tomcat 7, jdk 7
- Classique (Spring/JavaConfig, CXF/WS avec Rest/Soap, JMS (tibco-ems), Guava)
- Moins classique (Disruptor, java.util.concurrent, reuters, flux direct des bourses, serialisation custom interne ou  protobuf)
- Test : junit, mockito, assertj, wiremock.
- Outillage : maven / jenkins / sonar et JProfiler, MAT.

### Client
- UI dans excel ou GUI lourde C# via WS/Soap longpolling / ou REST.

## Indicateurs de volumetries 
(Alexandre) 
(nb faits 50/100 M pour 2 jours, nb message de risk /jour (todo), nb msg flux 3-5k/msg, taille message (todo), nb de deal = 50k, nb d instrument 10k, nb de soucription , nb users 100 paris, 20 NY / 20 HK, machine 24 core avec 256Go de ram utilisé de l'ordre de 50% en Cpu et mémoire mis -Xmx64g)

## ActivePivot: Aggregation et requete temps reel
(Benoit)
- Composant Technologique In Memory: enabled by new Hardware and new technos (Java GCs)
- Analogie F1: Si la technologie est tres puissante, elle reste difficile a mettre en oeuvre. OLAP apporte son lot de difficulte car tres naturel pour sujet simple, mais complexe pour les sujet avancee. La maturite acquises le long des projets ammene a trouver des solutions simples meme pour les sujets avancees... mais il est difficile de trouver ces solutions simples.
- Threading, long pooling, immutabilité dans activepivot / threading (1 requete , n thread)
- S'appuie sur la ForkJoinPool: backporté depuis JDK7 dans JDK6, QuartetFS est un des gros contributeurs a la JSR-166. Beaucoup de retour, bug-spotting et fixing, optimisation pour machine jusque 64 coeurs, 
- Optimisation pour machine avec jusque plusieurs TB de data (tests actuellement sur machine Bull 4TeraByte)
### Optim memoire
- Optimisation pour architecture NUMA
- Direct Memory pour diminuer la taille du heap
- Sharding (deploiement avec 6*1TB), architecture hsared nothing
- Explication du moteur temps reeel. pas de maintiens de vues temps reel. Calcul de l impact d une transaction au vues des questions posees.
 
## Performance : 
(Benoit)
SLA (del), optimisations (tout est en asynchrone). Usage d un framework specifique? Non. (Queue type Disruptor, ExecutorService,)
- asynchronisement
- pool d'objets pour les parties flux temps avec sérialization delta, attention a l'allocation d'objet
- sampling dynamique pour le flux, insertion dans le cube.
- JVM: options, temps de GC (exemple du gars qui dit que l on est lent, pile au moment ou on a GC) (e.g. le mec qui a groupé en moins d une seconde). tuning Old / Young, Azul., gc logs.

### Resolution des problemes:
- JProfiler
- MAT memory heap dump amaysis (over heaps up to 150GB, takes 1 or 2 hours)
- Thread Dump en prod.

## Best practises / Design : 
(Alexandre/Benoit)
- Isolation (mode dégradé pour chacun des systemes sources), chargement de fichier, reload à chaud (JMX, fichier), Couplage faible message entre application avec serialization key/value (sans schema fort type corba), pas de livraison simultané.
- Configuration du projet: javaConfig a eu un fort impact (si xml, dans votre lib application.xml, web.xml, avec des référence à des classes, ce n'est pas correct)
- Monotoring REST jenkins health check, metrics
- Beta en UAT avec flux de prod puis DR pour les bétas Users, release toutes le 3 semaines environ.
- Package par feature pas technique, nombre de pom.xml, 1 par livrable ou lib avec cycle de vie différent.
- Maven (javaformat, appassembler, assembly, release, site,)
- Interface / Impl simplifie le test dans et permet pas forcément de changement d'implementation (ex : cache HashMap / RelionalStore, lib temps réel Abstraction Subscriber entre flux floor, ).
- Tester/Tester/Tester (65/70 % de coverage).
- Savoir dire non sur certains choix si vous pensez que cela peut avoir des conséquences négatifs (analogie MST).

## Future améliorations techniques dans les prochains mois
(Alexandre)
- Tomcat 8 embedded, jdk 1.8, Migration ActivePivot 5.0
- Deploiement automatique (deployit).
- Meilleur monitoring avec logstash/elasticsearch/kibana
- Refactoring package / livrable plus orienté feature pour avoir des cubes type genre Rates, XAsset, Credit, ForexOption ...


Criticite de la production? Si les 2 serveurs sont down, il ne peuvent pas très traités mais pas seulement, il ne peuvent pas voir l'impact des évalotuions sur leurs positions actuelles. 
