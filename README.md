building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014
====================================================================

building-a-real-time-risk-analysis-system-in-java-devoxx-france-2014


 	

Notre application de Trading d'Analyse de Risque temps réel doit maintenir une base de 100 millions de lignes (Xmx=64G) avec du flux de 5000 message/sec. Nos utilisateurs (répartis à Paris, Londres, NY et HK) suivent plus de 100 indicateurs tels que la sensibilité au marché et les estimations de gains : ils sont agrégés en temps réel avec des règles métiers complexes. Nous vous proposons un retour d’expérience afin d'expliquer nos choix technologiques : le design de l'application, la librairie ActivePivot pour agréger des indicateurs en temps réel et la diffusion des impacts en temps-réel.



1) Presentation du métier, et objectifs du projet (problematique de l internationale)

2) Indicqteurs de volumetries (nb message, taille message, nb de deal, nb d instrument, nb users)

3) Stack technologique (Tibco?, Java + UI en C#, ActivePivot, JProfiler, MAT)

4) ActivePivot: Aggregation et requete temps reel



5) Perfs: SLA, optimisations (tout est en asynchrone). Usage d un framework specifique? Non

6) JVM: options, temps de GC (exemple du gars qui dit que l on est lent, pile au moment ou on a GC) (e.g. le mec qui a groupé en moins d une seconde).

7) Configuration du projet: javaConfig a eu un fort impact

8) Design / isolation (mode dégradé pour chacun des systemes sources)
