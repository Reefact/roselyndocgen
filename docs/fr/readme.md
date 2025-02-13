# RoselynDocGen

## Présentation

**RoselynDocGen** est une bibliothèque modulaire qui permet aux équipes de documenter leur architecture logicielle à partir du code source d’une solution .NET. Elle s’intègre naturellement aux workflows de développement et d’intégration continue.

Chaque équipe peut facilement développer ses propres générateurs de documentation en fonction de ses besoins spécifiques. La bibliothèque permet de concevoir ces générateurs grâce à une structure claire, des modèles simplifiés pour les cas courants et des abstractions sur Roslyn offrant une flexibilité avancée lorsque nécessaire. De plus, l'intégration des générateurs est facilitée par un système de configuration fluide, rendant leur activation et leur personnalisation intuitives.

La génération de documentation peut être déclenchée à la demande, soit localement par un développeur, soit dans un pipeline CI/CD en s'intégrant ainsi naturellement aux processus de déploiement et d’intégration continue.

## Architecture générale

RoselynDocGen est composé de deux éléments principaux :

### Core

Le cœur du système est responsable de l'analyse syntaxique du code et de la gestion des générateurs de documentation. Il repose sur Roslyn et fonctionne de la manière suivante :

- Implémente un `CSharpSyntaxWalker` global qui visite le code et délègue l'exécution aux générateurs enregistrés.
- Fournit des abstractions pour simplifier l'extraction des informations syntaxiques fréquentes.
- Gère la configuration des générateurs.
- Exécute les générateurs en parallèle pour améliorer les performances.
- Le `CSharpSyntaxWalker` global du Core parcourt l'ensemble du code une seule fois, tandis que les traitements effectués par les générateurs, qui fonctionnent indépendamment les uns des autres, sont parallélisés.

### Générateurs

Les générateurs sont des modules indépendants, chacun étant spécialisé dans un type de documentation.

Chaque générateur :
- Implémente son propre `CSharpSyntaxWalker`.
- Génère un fichier de documentation sous un format spécifique.
- Peut être ajouté ou retiré via la configuration.

## Configuration et exécution

### Configuration

La configuration de la génération de documentation se fait via un fichier `docgen.json` placé à la racine du projet. Ce fichier décrit les documents à générer, leur type et leurs options spécifiques.

📌 Exemple de configuration :
```json
{
  "documents": {
    "classDiagramWithoutPrivateMembers": {
      "type": "ClassDiagram",
      "description": "Diagramme de classe sans les membres privés",
      "outputFile": "class-diagram.mmd",
      "includePrivateMembers": false
    }
  }
}
```

📌 Principes :
- Chaque document est défini par un nom unique.
- Le champ `type` indique le type de générateur utilisé.
- Une description optionnelle peut être fournie pour faciliter l’analyse des logs.
- Les options spécifiques de ce type de document.

### Exécution

Une fois configuré, la génération de documentation peut être déclenchée via la CLI en utilisant [`docgen`](docgen.md) :

```sh
docgen generate --project src/MyProjectA
```

### Intégration CI/CD

RoselynDocGen peut être utilisé dans un pipeline CI/CD.

📌 Exemple avec GitHub Actions :
```yaml
jobs:
  generate-docs:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Restore dependencies
        run: dotnet restore

      - name: Build project
        run: dotnet build src/MyProjectA --configuration Release

      - name: Generate documentation
        run: docgen generate --project src/MyProjectA --output docs
```

Cette approche permet d’automatiser la génération de documentation tout en garantissant une cohérence entre les environnements de développement et de production.