# building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014


building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014


 	

Notre application de Trading d'Analyse de Risque temps réel doit maintenir une base de 100 millions de lignes (Xmx=64G) avec du flux de 5000 message/sec. Nos utilisateurs (répartis à Paris, Londres, NY et HK) suivent plus de 100 indicateurs tels que la sensibilité au marché et les estimations de gains : ils sont agrégés en temps réel avec des règles métiers complexes. Nous vous proposons un retour d’expérience afin d'expliquer nos choix technologiques : le design de l'application, la librairie ActivePivot pour agréger des indicateurs en temps réel et la diffusion des impacts en temps-réel.

# Presentation du métier, et objectifs du projet (problematique de l international)
 (Alexandre)
 
# (Alexandre) Indicateurs de volumetries (nb message, taille message, nb de deal, nb d instrument, nb users)

# Fonctionnalites business avancees: Grouping de deal, rebucketing, indicateur calculés temps reel PnL = sensi * (closing - last)

# Stack technologique (Tibco?, Java + UI en C#, ActivePivot, JProfiler, MAT, Spring, guava, disruptor)

## Server
- Classique (Spring/JavaConfig, CXF/WS avec Rest/Soap, JMS (tibco-ems), 
- Moins classique, 

## Client
- UI dans excel ou GUI lourde C#

# ActivePivot: Aggregation et requete temps reel (Benoit)
- Analogie F1

# Perfs: (benoit) 
SLA, optimisations (tout est en asynchrone). Usage d un framework specifique? Non
JVM: options, temps de GC (exemple du gars qui dit que l on est lent, pile au moment ou on a GC) (e.g. le mec qui a groupé en moins d une seconde).

# Design / Good practises: (Alexandre)
- Isolation (mode dégradé pour chacun des systemes sources), chargement de fichier, reload à chaud.
- Configuration du projet: javaConfig a eu un fort impact
- monotoring REST jenkins health check, metrics
  Logstash/Kibana
