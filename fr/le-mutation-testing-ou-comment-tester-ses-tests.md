

Garants de la pérennité des fonctionnalités développées, les tests sont un atout majeur dans la réalisation de logiciels de qualité. On peut même dire qu’ils sont indispensables et que leur intérêt n’est plus à prouver.

Pourtant, bien que sécurisants, les tests ne sont pas absolus. Développés principalement par des humains, le risque d’erreur lors de leur écriture persiste, menant par exemple à l’écriture de faux positifs. Des pratiques comme le TDD ou les code reviews ont permis de minimiser ce risque sans pour autant l’effacer. 

C’est ici que le _mutation testing_ (tests de mutations) entre en jeu. Inventée en 1971 par Richard Lipton, cette technique permet d’évaluer l’efficacité de nos tests et ce sans avoir à écrire une ligne de code (ou presque !).


# Comment ça marche ?


## Un peu de théorie

Le principe est très simple. Il s’agit de rendre le code “malade” à l’aide de mutations et d’observer la capacité de nos tests à diagnostiquer l’anomalie introduite.

Les mutations appliquées au code peuvent être de différentes formes comme :
*   la modification de la valeur d’une constante,
*   le remplacement d’opérateurs,
*   la suppression d’instructions,
*   et encore bien d’autres !


Si les tests restent au vert malgré les mutations du code, alors ils ne suffisent pas à détecter la régression amenée par le mutant. On parle dans ce cas de mutations survivantes. A l’inverse, si au moins un test passe au rouge lors de l'exécution d’un code muté, alors la mutation est dite tuée (sous entendu par le test).

On peut ainsi être en mesure de calculer un nouvel indicateur de qualité : le score de mutation qui vaut `(nb mutant tués / nb mutants générés) * 100`. Plus le score est élevé, plus nos tests sont robustes.

Trop abstrait ? C’est parti pour un exemple !


## Mutant, montre-moi ton vrai visage !

Prenons la classe suivante :

```java
public class User {

   public static int LAWFUL_AGE = 18;

   private int age;

   public User(int age) {
       this.age = age;
   }

   public int getAge() {
       return age;
   }

   public boolean isLawfulAge() {
       return age >= LAWFUL_AGE;
   }
}
```


Cette classe est testée à l’aide des tests unitaires suivants :

```java
class UserTest {

   @Test
   void is_lawful_age() {
       // Given
       User user = new User(18);
       
       // When
       boolean isLawfulAge = user.isLawfulAge();
       
       // Then
       Assertions.assertThat(isLawfulAge).isTrue();
   }

   @Test
   void is_under_age() {
       // Given
       User user = new User(17);
       
       // When
       boolean isLawfulAge = user.isLawfulAge();
       
       // Then
       Assertions.assertThat(isLawfulAge).isFalse();
   }
}
```


L’exemple étant des plus simplistes, il est possible de repérer le problème à la simple lecture du code : le cas où l’âge de l’utilisateur est supérieur à 18 n’est pas testé. Faisons comme si nous n’avions rien vu et transformons le code pour mettre en évidence le manque de robustesse de nos tests.


#### Mutant survivant

En remplaçant l’opérateur `>=` par `==` dans la méthode `isLawfulAge`, on observe alors que les 2 tests restent verts : notre mutant a survécu. On en conclut sans surprise que le test n’est pas assez robuste et que des régressions pourraient être passées sous silence.


#### Mutant tué

En décrémentant la constante `LAWFUL_AGE` de 1, on observe que le premier test reste vert tandis que le second passe au rouge. Le mutant a été tué par ce dernier.

En faisant varier le code manuellement dans le but de simuler un mutant, il a été possible de mettre en évidence les manquements des tests de l’exemple précédent. 

Voyons maintenant comment rendre ce processus plus fluide en automatisant son exécution et ce sans écrire une ligne de code (ou presque).


# À la découverte de Pitest

[Pitest](https://pitest.org/) est un outil permettant de générer automatiquement des mutants et de les exécuter pour vérifier le comportement des tests. Sa mise en place rapide et son utilisation simple font de Pitest un outil de choix. Seules quelques lignes de configuration dans le fichier de build permettent de tirer parti des tests de mutations.


### Du mutation testing sur RxJava

Pour nous en convaincre, nous allons voir comme il est simple de mettre en place le les tests de mutations sur un projet ayant déjà un bel historique. Prenons l’exemple de [RxJava](https://github.com/ReactiveX/RxJava).  Le projet utilisant Gradle, la suite traitera donc de la configuration de Pitest dans ce context, mais il est possible d’en faire de même pour des projets utilisant Maven.

Commençons par cloner le repository en local.

```bash
> git clone https://github.com/ReactiveX/RxJava.git
```

Vérifions maintenant que le projet build correctement :

```bash
> ./gradlew build
...
BUILD SUCCESSFUL in 8m 17s
23 actionable tasks: 19 executed, 4 up-to-date
...
```


Tout semble OK. Nous pouvons maintenant passer aux choses sérieuses et procéder à la mise en place de Pitest dans notre outil de build.

Pour cela, il est nécessaire d’installer le plugin `gradle-pitest-plugin` en le déclarant dans le fichier `build.gradle` :

```groovy
dependencies {
  ...
  classpath "info.solidsoft.gradle.pitest:gradle-pitest-plugin:1.5.1"
  ...
}
apply plugin: 'info.solidsoft.pitest'
```


Afin de réduire le temps d’exécution, il peut être intéressant d’utiliser plusieurs threads. Les tests de mutations se prêtent facilement au jeu du parallélisme. Pour cela, il est possible de spécifier le nombre de threads à utiliser lors des tests de mutations. Et nous allons vite voir que cela peut s’avérer nécessaire, voire indispensable.

``` groovy
pitest {
  threads = 4
}
```


Plus globalement, Pitest nous offre un bon nombre d’options et de possibilités de paramétrage. Il suffit alors de les déclarer elles aussi dans le bloc `pitest`. Nous en verrons quelques une intéressantes après. Pour l’instant, exécutons nos tests de mutations :

```bash
> ./gradlew pitest
```


![3nDhuXq](/content/images/2020/05/3nDhuXq.jpeg)

```bash
================================================================================
- Timings
================================================================================
> scan classpath : 1 seconds
> coverage and dependency analysis : 10 minutes and 33 seconds
> build mutation tests : 6 seconds
> run mutation analysis : 1 hours, 53 minutes and 15 seconds
--------------------------------------------------------------------------------
> Total  : 2 hours, 3 minutes and 57 seconds
--------------------------------------------------------------------------------
================================================================================
- Statistics
================================================================================
>> Generated 22339 mutations Killed 18928 (85%)
>> Ran 123314 tests (5.52 tests per mutation)
Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.0.1/userguide/command_line_interface.html#sec:command_line_warnings
BUILD SUCCESSFUL in 2h 6m 36s
4 actionable tasks: 1 executed, 3 up-to-date
14:59:21: Task execution finished 'pitest'.
```


Enfin ! Après plus de 2h d’exécution, le rapport tombe et RxJava arbore un généreux score de mutation de 85%. On notera plusieurs choses : 
*   malgré l’utilisation de 4 threads, la durée d’exécution des tests de mutations est considérablement plus longue que celle du build initialement effectué. Cette augmentation du temps de build est de l’ordre de 1500 % !
*   22 339 mutants ont été générés permettant d’exécuter 123 314 tests supplémentaires. 
*   un rapport HTML a été généré dans le dossier `./build/reports/pitest`. On y trouve les scores de chaque classe ainsi que les lignes pour lesquelles des mutations ont survécu.

Même si le nombre de cas testés est plutôt intéressant, le temps d’exécution excessivement long peut être un réel point de préoccupation.

Il est difficile d'imaginer, dans une chaîne de CI, d’avoir les tests de mutations prenant plusieurs heures à être exécutés. Comment réduire l'impact du Mutation testing sur la chaine de CI ?


## Améliorer le rendement des tests de mutations


### L’analyse incrémentale

Nous avons pour le moment utilisé Pitest de la manière la plus simple possible. Bien que nous ayons utilisé 4 threads, le temps d’exécution était bien trop long pour être acceptable.

Pitest propose un système d’[analyse incrémentale](https://pitest.org/quickstart/incremental_analysis/). Ce système se base sur un fichier contenant l’ancienne analyse à partir de laquelle il prendra la décision ou non de rejouer certains tests lors de la prochaine exécution des tests de mutations. Pour l’activer dans notre projet, il suffit de rajouter l’option `enableDefaultIncrementalAnalysis = true`. 

```groovy
pitest {
  threads = 4
  enableDefaultIncrementalAnalysis = true
}
```


> Si vous utilisez Maven, il est possible d’utiliser le goal `smcMutationCoverage` qui déléguera la détection des changements à votre VCS. Cela peut simplifier la construction du pipeline de CI qui n’aura alors plus besoin de garder le résultat de l’exécution précédente.

Pour générer le fichier d’analyse, nous allons devoir repasser par ces deux longues heures d’exécution…

```bash
> ./gradlew pitest
…
BUILD SUCCESSFUL in 1h 53m 15s
4 actionable tasks: 1 executed, 3 up-to-date
```


L’exécution est enfin terminée et le résultat de l’analyse se trouve dans  `./build/pitHistory.txt`. 

> Par défaut, le résultat sera enregistré dans `./build/pitHistory.txt`. Il est possible de spécifier l’emplacement du fichier à l’aide des options `historyInputLocation` et `historyOutputLocation`. 

Lançons à nouveau les tests de mutations :


```bash
> ./gradlew pitest
…
BUILD SUCCESSFUL in 827ms
4 actionable tasks: 4 up-to-date
```


Pitest a détecté que nous n’avions fait aucun changement dans le code et n’a donc pas exécuté les tests de mutations.


### Définir la politique de mutation

Il est possible d’affiner le scope des mutations à appliquer au code. Pour cela Pitest fournit différents [opérateurs de mutation](https://pitest.org/quickstart/mutators/) qu’il est possible de combiner comme bon nous semble.

Bien que réduire le nombre d’opérateurs puisse permettre de gagner en temps d’exécution, il peut être au contraire intéressant, en tirant profit de l’analyse incrémentale, d’augmenter le nombre d’opérateurs à appliquer.

Cela aura pour effet d’augmenter le nombre de mutants générés et donc le nombre de tests exécutés. 

Pour cela, il suffit de préciser la liste de mutateurs que l’on souhaite utiliser à l’aide de l’option `mutators`. Cette option accepte aussi bien des groupes (ex : `DEFAULTS`) que des noms (ex : `CRCR5`) de mutateurs.

```groovy
pitest {
  threads = 4
  enableDefaultIncrementalAnalysis = true
  mutators = [“DEFAULTS”, “CRCR5”, “CRCR6”]
}
```


# En bref

Des indicateurs comme le taux de couverture se voulaient être des indices quant à la qualité des tests mais ne pouvaient la garantir à eux seuls. Aujourd’hui, grâce au _mutation testing_, il est enfin possible de prendre pleine mesure de leur qualité à l’aide d’une critique automatisée de leur pertinence. 


Compte tenu du faible effort à fournir pour obtenir cette information, il semble très intéressant de se munir de cet indicateur qui viendra ajouter une couche d’assurance supplémentaire dans la qualité des livrables produits ou encore souligner efficacement des manquements. Son utilisation devra dépendre du contexte, le tout étant de ne pas tomber dans le piège de l’indicateur “Graal” à suivre aveuglément. 

 

Pour aller plus loin avec le mutation testing, il existe d’autres outils que nous n’avons pas abordés dans cet article comme [DSpot](https://github.com/STAMP-project/dspot). Le principe reste le même mais les mutations se font sur les tests eux mêmes: on parlera alors d’amplification de tests.

Annexe: l’exemple de cet article est disponible sur [GitHub](https://github.com/gelhaimer/RxJava)
