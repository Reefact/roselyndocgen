# Documentation de l'outils `docgen`

## Présentation

`docgen` est un outil en ligne de commande permettant de générer automatiquement la documentation des projets .NET en analysant leur code source via Roslyn. Il permet une configuration flexible à l’aide d’un fichier JSON (`docgen.json`).

## Installation

### Prérequis

- .NET x ou supérieur doit être installé.
- `docgen` doit être ajouté à votre projet ou installé en tant qu'outil global/local.

### Installation globale

```sh
dotnet tool install -g docgen
```

### Installation locale dans un projet

```sh
dotnet new tool-manifest
```

```sh
dotnet tool install docgen
```

📌 **Après avoir cloné un projet utilisant un manifest d’outils, il faut exécuter :**

```sh
dotnet tool restore
```

## Commandes disponibles

### `docgen generate`

Génère la documentation d’un projet ou d’une solution.

#### Syntaxe

```sh
docgen generate --project <path> [options]
docgen generate --solution <path> [options]
```

#### Options

| Option                  | Mode Projet (`--project`) | Mode Solution (`--solution`) | Description                                                                        |
| ----------------------- | ------------------------- | ---------------------------- | ---------------------------------------------------------------------------------- |
| `--project <path>`      | ✅ **Obligatoire**         | ❌ Non applicable             | Spécifie le projet pour lequel générer la documentation.                           |
| `--solution <path>`     | ❌ Non applicable          | ✅ **Obligatoire**            | Génère la documentation pour tous les projets d'une solution.                      |
| `--config <file>`       | ✅                         | ❌                            | Spécifie un fichier de configuration JSON différent du `docgen.json` par défaut.   |
| `--output <path>`       | ✅                         | ✅                            | Définit la base du chemin de sortie.                                               |
| `--flat`                | ✅                         | ✅                            | Désactive la création automatique des sous-dossiers projet. ⚠ Risque d'écrasement. |
| `--only <generators>`   | ✅                         | ❌                            | Génère uniquement certains générateurs configurés.                                 |
| `--except <generators>` | ✅                         | ❌                            | Génère tous les générateurs sauf ceux spécifiés.                                   |

#### Exemples d’utilisation

**Générer la documentation pour un projet spécifique :**

```sh
docgen generate --project src/MyProjectA
```

**Générer la documentation pour un projet avec un fichier de configuration spécifique :**

```sh
docgen generate --project src/MyProjectA --config custom-docgen.json
```

**Générer la documentation de tous les projets d'une solution :**

```sh
docgen generate --solution path/to/MySolution.sln
```

**Forcer la sortie des fichiers dans un répertoire commun :**

```sh
docgen generate --solution path/to/MySolution.sln --output "CommonDocs"
```

→ Cela génère dans `CommonDocs/{NomDuProjet}/` (sauf si `--flat` est utilisé).

## Configuration de la génération de documentation

Le fichier `docgen.json` permet de configurer la génération de la documentation d’un projet. Il définit les documents à générer et leurs options spécifiques.

### Exemple de `docgen.json`

```json
{
  "documents": {
    "classDiagramWithPrivateMembers": {
      "description": "Diagramme de classe qui inclut les membres privés",
      "type": "ClassDiagram",
      "includePrivateMembers": true,
      "fileName": "class-diagram-1.mmd"
    },
    "classDiagramWithoutPrivateMembers": {
      "description": "Diagramme de classe qui n'inclut pas les membres privés",
      "type": "ClassDiagram",
      "includePrivateMembers": false,
      "fileName": "class-diagram-2.mmd"
    }
  }
}
```

## Gestion des générateurs de documentation

### Convention de nommage des générateurs

Nous recommandons que les classes de générateurs n'aient pas le suffixe `Generator` dans leur nom. Cela permet d'obtenir une configuration plus lisible et orientée document, plutôt que centrée sur l'implémentation technique du générateur (voir exemple ci-dessus). De plus, la détection des générateurs repose sur l’attribut `[GeneratorAttribute]`, ce qui garantit qu'ils sont identifiés sans ambiguïté, même sans ce suffixe.

### Ajout de générateurs personnalisés

Les développeurs peuvent utiliser les générateurs fournis par `RoselynDocGen.Generators.Common` ou créer leurs propres générateurs en suivant ces étapes :

1. **Référencer la bibliothèque du générateur dans le projet**
   ```xml
   <ItemGroup>
     <PackageReference Include="RoselynDocGen.Generators.MyTeam" Version="1.0.0" PrivateAssets="all" IncludeAssets="runtime" />
   </ItemGroup>
   ```
2. **Créer un générateur personnalisé** avec l’attribut `[GeneratorAttribute]` :
   ```csharp
   [GeneratorAttribute]
   public class CustomClassDiagram : CSharpSyntaxWalker {
       // ...
   }
   ```
3. **Configurer `docgen.json` avec le nom simple ou complet de la classe du générateur** :
   ```json
   {
     "outputPath": "docs/",
     "documents": {
       "customEntityDoc": {
         "type": "RoselynDocGen.Generators.MyTeam.ClassList"
       }
     }
   }
   ```

⚠ **Règles de détection des générateurs**

- `docgen` **ne détecte que les générateurs flaggés avec l'attribut `[GeneratorAttribute]`**.
- **La recherche de générateurs est insensible à la casse**, permettant d’écrire `classDiagram` au lieu de `ClassDiagram` dans `docgen.json`.

## Intégration CI/CD

### Exemple avec GitHub Actions

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
        run: docgen generate --project src/MyProjectA
```