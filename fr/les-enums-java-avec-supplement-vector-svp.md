# Les enums Java ? Avec supplément Visitor s’il vous plaît !


Les enums peuvent être vues comme un regroupement de constantes fortement typées. Elles trouvent leur utilité dans de nombreux usages : attribuer une sémantique forte à des valeurs, borner et valider les valeurs possibles d’une donnée, améliorer la lisibilité du code, etc. L’utilisation d’enums peut cependant devenir acrobatique lorsqu’il s’agit de baser des décisions sur leurs valeurs. A mesure que l’enum évolue, chaque endroit où celle-ci a été utilisée en tant que condition doit être revérifié. Si la valeur d’une enum est activement exploitée dans la logique métier, maintenir la base de code dans son ensemble peut devenir un cauchemar : un oubli de vérification peut entraîner une corruption du système dans son ensemble.

N'existe-t-il pas alors un moyen de réduire l'impact des évolutions d'une enum sur le code métier ?
## L’approche switch-case

Considérons l’enum `AssetClass` représentant les différents groupes de matières premières échangeables :

```java
public enum AssetClass {
   METAL,
   ENERGY,
   AGRICULTURAL,
}
```

Cette enum pourra être utilisée afin de décider des stratégies, mapper, ou tout autre comportement dépendant de la valeur de `AssetClass` utilisée. Imaginons utiliser cette enum pour définir une stratégie de trading automatique à utiliser. Cela ressemblera généralement à quelque chose comme :

```java
public AutomatedTradingStrategy getAutomatedTradingStrategy(AssetClass assetClass) {
    switch (assetClass) {
        case METAL:return new HedgingStrategy();
        case ENERGY: return new SwingTradingStrategy();
        case AGRICULTURAL:
        default: return DayTradingStrategy();
    }
}
```

Le switch-case est probablement la manière la plus simple et la plus directe de faire. Cela a cependant plusieurs défauts.

La méthode `getAutomatedTradingStrategy()` retourne un comportement en fonction de la valeur de `AssetClass`. Définir un comportement par défaut devient alors obligatoire, même si, dans cet exemple, l’ensemble des valeurs de l’enum sont traitées. Pour cela, nous pouvons soit retourner une implémentation de `AutomatedTradingStrategy` par défaut, soit retourner null ou alors jeter une exception.

L’utilisation de ce comportement par défaut rendra silencieux l’ajout d’une nouvelle valeur à l’enum. Il faudra alors penser à re-vérifier chaque endroit où `AssetClass` est impliquée dans des règles métiers. Et rien ne nous protège d'un oubli.

Le problème suivant est probablement le moins évident. L’utilisation du switch-case crée ici un couplage fort entre la logique métier et les valeurs de l’enum brisant ainsi le principe ouvert/fermé: le code doit être ouvert à l’extension mais fermé à la modification. Ici, modifier l’enum implique de modifier chaque bloc de code qui reposait sur ses valeurs.
Pourtant, dans le cas du switch-case, nous n’avons aucun intérêt à savoir si une asset est représentée par une enum, un objet ou autre. Seule sa sémantique compte.
Par exemple, les métaux pourraient être divisés en deux sous-catégories : les métaux précieux et les métaux de base. Tout code reposant sur `AssetClass.METAL` devra alors être retravaillé afin de prendre en compte ces deux nouvelles valeurs. Le refactoring de l'existant n'apportera aucune nouvelle valeur métier mais exposera un code déjà opérationnel à des risques de régressions.

## Le pattern Visitor à la rescousse

Comment pouvons-nous alors briser ce couplage tout en offrant la possibilité de contextualiser la logique métier aux valeurs de l'enum ? La réponse est dans le titre : utilisons le pattern Visitor.

Créons dans un premier temps l’interface qui servira de contrat entre notre enum et le code souhaitant interagir avec.

```java
public interface AssetClassVisitor<T> {
   T visitMetal();
   T visitEnergy();
   T visitAgricultural();
}
```

L’interface est générique afin que celle-ci puisse permettre des implémentations dont l’objectif diffère selon son contexte d’utilisation.

Il est maintenant nécessaire de modifier l’enum afin que celle-ci accepte toute demande respectant le contrat porté par `AssetClassVisitor` :

```java
public enum AssetClass {
   METAL {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitMetal();
       }
   },
   ENERGY {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitEnergy();
       }
   },
   AGRICULTURAL {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitAgricultural();
       }
   };

   public abstract <E> E accept(AssetClassVisitor<E> visitor);
}
```

Il ne reste alors plus qu’à l’utiliser. Remplaçons notre switch-case par une implémentation du visitor :

```java
public AutomatedTradingStrategy getAutomatedTradingStrategy(AssetClass assetClass) {
    return assetClass.accept(new AssetClassVisitor<AutomatedTradingStrategy>() {
        @Override
        public AutomatedTradingStrategy visitMetal() {
            return new HedgingStrategy();
        }

        @Override
        public AutomatedTradingStrategy visitEnergy() {
            return new SwingTradingStrategy();
        }

        @Override
        public AutomatedTradingStrategy visitAgricultural() {
            return new DayTradingStrategy();
        }
    });
}
```


Comme on peut le constater, chaque valeur de `AssetClass` porte la responsabilité d’appeler la méthode du visitor appropriée. Il est désormais inutile de connaître les valeurs ou l’implémentation d’AssetClass. `AssetClass.AGRICULTURAL` pourrait alors en `AssetClass.AGRI` sans avoir à modifier quoi que ce soit au niveau de la logique métier.
Il est par ailleurs devenu inutile de gérer des comportements par défaut. Les possibilités sont désormais restreintes à celles fournies par l’interface.

## Ajoutons une nouvelle asset class

Notre business évolue et nous devons désormais étendre nos activités aux bétails et viandes. Il suffit alors simplement d’ajouter la valeur à notre enum et de mettre à jour notre contrat d’interface.

```java
public enum AssetClass {
   METAL {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitMetal();
       }
   },
   ENERGY {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitEnergy();
       }
   },
   AGRICULTURAL {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitAgricultural();
       }
   },
   // La nouvelle valeur
   LIVESTOCK_AND_MEAT {
       @Override
       public <E> E accept(AssetClassVisitor<E> visitor) {
           return visitor.visitLiveStockAndMeat();
       }
   };

   public abstract <E> E accept(AssetClassVisitor<E> visitor);
}
```

```java
public interface AssetClassVisitor<T> {
   T visitMetal();
   T visitEnergy();
   T visitAgricultural();
   // La nouvelle méthode
   T visitLiveStockAndMeat();
}
```

Après cela, le code va s’allumer comme un sapin de Noël : plus rien ne compile. Et le compilateur devrait être remercié d’avoir fait un si bon travail ! Toutes ces erreurs mises en évidence de toute part nous montrent que certaines partie du code ne sont pas en mesure de répondre à cette nouvelle valeur. Corrigeons donc cela en utilisant une exception : les  fonctionnalités existantes ne sont pas encore disponibles pour les bétails et viandes.

```java
public AutomatedTradingStrategy getAutomatedTradingStrategy(AssetClass assetClass) {
    return assetClass.accept(new AssetClassVisitor<AutomatedTradingStrategy>() {
        @Override
        public AutomatedTradingStrategy visitMetal() {
            return new HedgingStrategy();
        }

        @Override
        public AutomatedTradingStrategy visitEnergy() {
            return new SwingTradingStrategy();
        }

        @Override
        public AutomatedTradingStrategy visitAgricultural() {
            return new DayTradingStrategy();
        }

        @Override
        public AutomatedTradingStrategy visitLiveStockAndMeat() {
            throw new AutomatedTradingNotSupported("Automated trading for Livestock and meat is not allowed.")
        }
    });
}
```

## En bref

Lors d’une de mes missions, l'équipe a été confrontée à un nombre conséquent d’enum et de logique métier basée sur leurs valeurs. Le pattern Visitor était notre bouclier contre les cas à la marge au point d'en devenir notre standard dans la gestion des enums.

Utiliser ce pattern n'est pas nécessaire si les enums sont purement descriptives. Cependant, sortir l’artillerie lourde vaut définitivement le coût de développement supplémentaire. Briser le couplage entre la valeur d’une enum et la logique métier offre une souplesse d’évolution supplémentaire tandis que le compilateur réduit la boucle de feedback en mettant en lumière les oublis potentiels.


