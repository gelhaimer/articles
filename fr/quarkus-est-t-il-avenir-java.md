# Quarkus est-il l’avenir de Java ?

[Quarkus](https://quarkus.io/) est le petit dernier du monde Java. Cela fait maintenant quelques temps que ce framework open source initié par Redhat fait parler de lui. Temps de démarrage amélioré, diminution du coût de run, productivité accrue, sont autant de promesses qui ont permis à Quarkus d’être déjà considéré par certains comme le futur de Java.


## La promesse

Quarkus a été imaginé pour permettre le développement d’applications Java dites cloud-natives, ou “kubernetes native” pour reprendre leurs termes. Pour ce faire, Quarkus a pour principal objectif de réduire le temps de démarrage des applications ainsi que leur empreinte mémoire. 

Au delà de ces objectifs de performance, le framework affiche une volonté forte de simplifier la vie des développeurs en leur proposant notamment une configuration unifiée et le retour du live reload.


## Le livereload, une fausse bonne idée ?

Quarkus fait renaître une fonctionnalité qui avait plus ou moins disparu avec l’arrivée de Spring Boot : le livereload.

Lorsque nous développons en Java, il est nécessaire de lancer une compilation et de redémarrer l'application afin que les changements effectués soient pris en compte. Le livereload a permis d’éliminer ce besoin de redémarrage et donc indirectement de réduire la boucle de feedback.

Cela ne fonctionne cependant pas correctement avec des frameworks comme Spring Boot qui construisent un contexte applicatif au démarrage du programme en utilisant la réflexion. Un changement dans le code peut amener une modification du contexte et donc nécessiter un redémarrage. Ce redémarrage a alors un coût indirect et nous pourrions naïvement croire à un impact négatif sur la productivité.

Mais avons-nous vraiment besoin de démarrer l’application pour développer et vérifier le fonctionnement de notre code ?

Sans surprise, la réponse est non. Enfin, du moins pas suffisamment souvent pour en faire un point de frustration !

Le [TDD](https://blog.ippon.fr/2020/02/12/apprendre-le-tdd/) (Test Driven Development) ainsi qu’une bonne base de tests sont amplement suffisants pour obtenir un retour rapide sur le comportement de l'application.

Si un changement dans le code casse l’application sans que les tests ne l’aient montré, alors aucune nouvelle modification ne devrait être apportée tant qu’un nouveau test n’a pas été implémenté.

Ainsi, bien que le livereload semble fournir un gain significatif de productivité, ce gain est, à mon avis, négligeable face à l’utilisation de pratiques de développement centrées sur la qualité logiciel. Et si gain de productivité doit être recherché, alors c’est clairement sur ces axes indépendants de la technologie utilisée qu’il faut travailler.


## Compilation native : des coûts cachés ?

Quarkus permet de réduire d’environ 99% le temps de boot et d’environ 86% l’empreinte mémoire des applications Java classiques en utilisant la compilation native proposée par GraalVM (pourcentages calculés à partir des données disponibles sur [Quarkus.io](https://quarkus.io/)). C’est autant de ressources en moins consommées et donc d’économies réalisées.

Une partie importante est cependant négligée : le coût du build.

Afin d’obtenir ces améliorations, Quarkus tire profit de GraalVM et des conteneurs afin de construire des exécutables Java natifs. L’exécutable est alors optimisé pour tourner dans un environnement défini. Pour cela, il faut construire l’exécutable d’une certaine manière. C’est là que la compilation AOT (ahead of time) entre en jeu.

Cette opération est gourmande en ressources, aussi bien mémoire que CPU, et va potentiellement s’exécuter un nombre important de fois dans la chaîne de CI, selon la taille de l’équipe, le nombre de commits, etc.

Nous avons alors d’une part un gain potentiel sur le coût du run, et de l’autre, une augmentation potentielle du coût de la phase de build. En l’état, il est très difficile de tirer des conclusions quant aux avantages économiques apportés par Quarkus dans la mesure où nous ne savons pas vraiment de quel côté penche la balance. Cela dépendra sans doute fortement du contexte projet.


## A quelle problématique répond vraiment Quarkus ?

Quarkus tente aujourd’hui de répondre aux problèmes liés à Java et son utilisation dans le Cloud. Cependant, d’autres alternatives existent déjà comme Golang ou encore NodeJS qui permettent nativement de créer des applications légères et au démarrage rapide.

Ces technologies ne sont malheureusement pas maîtrisées par beaucoup de développeurs et d’entreprises. Cette contrainte, couplée à une pénurie de compétences rendent risquées des migrations de Java vers Golang par exemple.

C’est sur ce point que Quarkus apporte, selon moi, une réelle solution. En essayant de pallier les manquements de Java dans des contextes Cloud, Quarkus tend à proposer aux entreprises une alternative en capitalisant sur les compétences déjà acquises.

En tant que développeur, il devient alors possible de répondre aux exigences des applications modernes sans nécessiter l’apprentissage complet d’un nouveau langage et de son écosystème.


## Finalement, quel avenir pour Quarkus ?

Quarkus doit encore faire ses preuves. La frontière entre gains et coûts étant encore floue, je ne conseillerais pas, aujourd’hui, son utilisation. Il est pour l’heure préférable de privilégier des technologies qui ont déjà fait leurs preuves dans des environnements Cloud exigeants et de ne pas tomber dans le piège du [Hype Driven Development](https://www.youtube.com/watch?v=nfd3wh-GOQI) que l’on peut actuellement observer avec Kubernetes (qui sert d’ailleurs au branding de Quarkus).

Il est cependant indéniable que les avancées de Quarkus sont à suivre. Il s’agit ici d’une réelle remise en question de Java et de sa capacité à répondre aux problématiques cloud-native tout en considérant la difficulté des entreprises à construire leur expertise IT. 

L’enthousiasme de la communauté Java pour Quarkus montre un intérêt certain pour les améliorations proposées et présage ainsi de belles avancées.