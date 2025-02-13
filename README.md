# RoselynDocGen

## 1. Présentation

**RoselynDocGen** est une bibliothèque modulaire qui permet aux équipes de documenter leur architecture logicielle à partir du code source d’une solution .NET. Elle s’intègre naturellement aux workflows de développement et d’intégration continue.

Chaque équipe peut facilement développer ses propres générateurs de documentation en fonction de ses besoins spécifiques. La bibliothèque permet de concevoir ces générateurs grâce à une structure claire, des modèles simplifiés pour les cas courants et des abstractions sur Roslyn offrant une flexibilité avancée lorsque nécessaire. De plus, l'intégration des générateurs est facilitée par un système de configuration fluide, rendant leur activation et leur personnalisation intuitives.

La génération de documentation peut être déclenchée à la demande, soit localement par un développeur, soit dans un pipeline CI/CD en s'intégrant ainsi naturellement aux processus de déploiement et d’intégration continue.

## 2. Architecture générale

RoselynDocGen est composé de deux éléments principaux :

### Core

Le cœur du système est responsable de l'analyse syntaxique du code et de la gestion des générateurs de documentation. Il repose sur Roslyn et fonctionne de la manière suivante :

- Implémente un `CSharpSyntaxWalker` global qui visite le code et délègue l'exécution aux générateurs enregistrés.
- Fournit des abstractions pour simplifier l'extraction des informations syntaxiques fréquentes.
- Gère la configuration des générateurs.
- Exécute les générateurs en parallèle pour améliorer les performances.
- Le `CSharpSyntaxWalker` global du Core parcourt l'ensemble du code une seule fois, tandis que les traitements effectués par les générateurs, qui fonctionnent indépendamment les uns des autres, sont parallélisés.

### **Générateurs**

Les générateurs sont des modules indépendants, chacun étant spécialisé dans un type de documentation.

Chaque générateur :

- Implémente son propre `CSharpSyntaxWalker`.
- Génère un fichier de documentation sous un format spécifique.
- Peut être ajouté ou retiré via la configuration.

## 3. Configuration et exécution

### **Configuration**

La configuration se fait directement en C# :

```csharp
var config = DocumentationConfig.Configure()
    .UseGenerator<ClassDiagramGenerator>(options => 
    {
        options.SetOutputPath("docs/class-diagram.mmd");
        options.IncludePrivateMembers = true;
    });

DocumentationGenerator.Run(config);
```

### **Exécution via CLI**

Une fois configuré, RoselynDocGen peut être exécuté avec la commande suivante :

```shell
docgen generate
```

Cette approche permet de déclencher la génération à la demande, sans impacter les temps de compilation.
