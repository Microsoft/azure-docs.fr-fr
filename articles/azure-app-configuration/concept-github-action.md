---
title: Synchroniser votre dépôt GitHub avec App Configuration
description: Utilisez GitHub Actions pour mettre automatiquement à jour votre instance App Configuration quand vous mettez à jour votre dépôt GitHub.
author: lisaguthrie
ms.author: lcozzens
ms.date: 02/20/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 66d0e32e7dfdd5ab2abee5108ac8ce54c5222747
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87371819"
---
# <a name="sync-your-github-repository-to-app-configuration"></a>Synchroniser votre dépôt GitHub avec App Configuration

Les équipes qui souhaitent continuer à utiliser leurs pratiques de contrôle de code source peuvent utiliser GitHub Actions pour synchroniser automatiquement leur référentiel GitHub avec leur magasin App Configuration. Cela vous permet d’apporter des modifications à vos fichiers config comme vous le feriez normalement, tout en obtenant des avantages App Configuration comme : <br>
&nbsp;&nbsp;&nbsp;&nbsp;•   Configuration centralisée en dehors de votre code <br>
&nbsp;&nbsp;&nbsp;&nbsp;•   Mise à jour de la configuration sans redéploiement de l’intégralité de votre application <br>
&nbsp;&nbsp;&nbsp;&nbsp;•   Intégration à des services comme Azure App Service et Azure Functions 

Un [workflow](https://help.github.com/articles/about-github-actions#workflow) GitHub Actions définit un processus automatisé dans un dépôt GitHub. L’action *Synchronisation Azure App Configuration* déclenche les mises à jour d’une instance App Configuration lorsque des modifications sont apportées au référentiel source. Elle utilise un fichier YAML (.yml) trouvé sous le chemin `/.github/workflows/` de votre référentiel pour définir les étapes et les paramètres. Vous pouvez déclencher des mises à jour de configuration lorsque vous envoyez (push), révisez ou branchez des fichiers de configuration d’application, comme vous le feriez avec du code d’application.

La [documentation](https://help.github.com/actions/automating-your-workflow-with-github-actions/configuring-a-workflow) GitHub fournit une vue détaillée des flux de travail et actions GitHub. 

## <a name="enable-github-actions-in-your-repository"></a>Activer GitHub Actions dans votre dépôt
Pour commencer à utiliser cette action GitHub, accédez à votre dépôt et sélectionnez l’onglet **Actions**. Sélectionnez **Nouveau workflow**, puis **Configurer un workflow vous-même**. Enfin, recherchez « Azure App Configuration Sync » sur la Place de marché.
> [!div class="mx-imgBorder"]
> ![Sélectionner l’onglet Action](media/find-github-action.png)

> [!div class="mx-imgBorder"]
> ![Sélectionner l’action Azure App Configuration Sync](media/app-configuration-sync-action.png)

## <a name="sync-configuration-files-after-a-push"></a>Synchroniser les fichiers de configuration après un push
Cette action synchronise les fichiers Azure App Configuration lors du push d’une modification vers `appsettings.json`. Quand un développeur envoie (push) une modification vers `appsettings.json`, l’action App Configuration Sync met à jour l’instance App Configuration avec les nouvelles valeurs.

La première section de ce flux de travail spécifie que l’action se déclenche *sur* un *push* contenant `appsettings.json` vers la branche *maître*. La deuxième section répertorie les travaux exécutés une fois que l’action déclenchée. L’action extrait les fichiers appropriés et met à jour l’instance App Configuration à l’aide de la chaîne de connexion stockée en tant que secret dans le dépôt.  Pour plus d’informations sur l’utilisation des secrets dans GitHub, consultez [cet article GitHub](https://help.github.com/actions/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets) sur la création et l’utilisation des secrets chiffrés.

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your                        
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }} 
          separator: ':' 
```

## <a name="use-strict-sync"></a>Utiliser une synchronisation stricte
Par défaut, l’action GitHub n’active pas le mode strict, ce qui signifie que la synchronisation ajoutera uniquement les valeurs de clés du fichier de configuration à l’instance App Configuration (aucune paire clé-valeur n’est supprimée). L’activation du mode strict signifie que les paires clé-valeur qui ne se trouvent pas dans le fichier de configuration sont supprimées de l’instance App Configuration, afin qu’elles correspondent au fichier de configuration. Si vous effectuez une synchronisation à partir de plusieurs sources ou à l’aide d’Azure Key Vault avec App Configuration, vous pouvez utiliser des préfixes ou des étiquettes différents avec une synchronisation stricte afin d’éviter d’effacer les paramètres de configuration d’autres fichiers (voir les exemples ci-dessous). 

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: 'Label' 
          prefix: 'Prefix:' 
          strict: true 
```
## <a name="sync-multiple-files-in-one-action"></a>Synchroniser plusieurs fichiers en une seule action 

Si votre configuration se trouve dans plusieurs fichiers, vous pouvez utiliser le modèle ci-dessous pour déclencher une synchronisation lorsque l’un des fichiers est modifié. Ce modèle utilise la bibliothèque Glob https://www.npmjs.com/package/glob. Notez que si le nom de votre fichier config contient une virgule, vous pouvez utiliser une barre oblique inverse pour éviter la virgule. 

```json
on:
  push:
    branches:
      - 'master'
    paths:
      - 'appsettings.json'
      - 'appsettings2.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: '{appsettings.json,appsettings2.json}'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
```

## <a name="sync-by-prefix-or-label"></a>Synchroniser par préfixe ou étiquette
Si vous spécifiez des préfixes ou des étiquettes dans votre action de synchronisation, vous synchroniserez uniquement cet ensemble particulier. Cela est important pour l’utilisation de la synchronisation stricte avec plusieurs fichiers. Selon la configuration utilisée, un préfixe ou une étiquette peut être associé à chaque fichier, puis chaque préfixe ou étiquette peut être synchronisé séparément afin que rien ne soit remplacé. En règle générale, les préfixes sont utilisés pour différentes applications ou les services et étiquettes sont utilisés pour différents environnements. 

Synchronisation par préfixe : 

```json
on:
  push:
    branches:
      - 'master'
    paths:
      - 'appsettings.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          prefix: 'Prefix::'
```

Synchronisation par étiquette : 

```json
on:
  push:
    branches:
      - 'master'
    paths:
      - 'appsettings.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          label: 'Label'

```

## <a name="use-a-dynamic-label-on-sync"></a>Utiliser une étiquette dynamique lors d’une synchronisation
L’action suivante insère une étiquette dynamique à chaque synchronisation. De cette façon, chaque synchronisation peut être identifiée et les modifications du code peuvent être mappées aux modifications de configuration.

La première section de ce flux de travail spécifie que l’action se déclenche *sur* un *push* contenant `appsettings.json` vers la branche *maître*. La deuxième section exécute un travail qui crée une étiquette pour la mise à jour de configuration en fonction du hachage de validation. Le travail met ensuite à jour l’instance App Configuration avec les nouvelles valeurs et l’étiquette unique pour cette mise à jour.

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # Creates a label based on the branch name and the first 8 characters          
      # of the commit hash 
      - id: determine_label 
        run: echo ::set-output name=LABEL::"${GITHUB_REF#refs/*/}/${GITHUB_SHA:0:8}" 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          label: ${{ steps.determine_label.outputs.LABEL }} 
```

## <a name="use-azure-key-vault-with-github-action"></a>Utiliser Azure Key Vault avec l’action GitHub
Les développeurs qui utilisent Azure Key Vault avec AppConfiguration doivent utiliser deux fichiers distincts, généralement un fichier appSettings.json et un fichier secretreferences.json. Le fichier secretreferences.json contient l’URL du secret du coffre de clés.

{ "mySecret": "{\"uri\":\"https://myKeyVault.vault.azure.net/secrets/mySecret "}" }

L’action GitHub peut ensuite être configurée pour effectuer une synchronisation stricte sur appSettings.json, suivie d’une synchronisation non stricte sur secretreferences.json. L’exemple suivant déclenche une synchronisation lorsque l’un des fichiers est mis à jour :

```json
on:
  push:
    branches:
      - 'master'
    paths:
      - 'appsettings.json'
      - 'secretreferences.json'

jobs:
  syncconfig:
    runs-on: ubuntu-latest
    steps:
      # checkout done so that files in the repo can be read by the sync
      - uses: actions/checkout@v1
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'appsettings.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          strict: true
      - uses: azure/appconfiguration-sync@v1
        with:
          configurationFile: 'secretreferences.json'
          format: 'json'
          # Replace <ConnectionString> with the name of the secret in your repository
          connectionString: ${{ secrets.<ConnectionString> }}
          separator: ':'
          contentType: 'application/vnd.microsoft.appconfig.keyvaultref+json;charset=utf-8'

```

## <a name="use-max-depth-to-limit-github-action"></a>Utiliser une profondeur maximale pour limiter une action GitHub
Le comportement par défaut pour les attributs JSON imbriqués consiste à aplatir l’objet entier.  Le JSON ci-dessous définit cette paire clé-valeur :

| Clé | Valeur |
| --- | --- |
| Object:Inner:InnerKey | InnerValue |

```json
{ "Object": 
    { "Inner":
        {
        "InnerKey": "InnerValue"
        }
    }
}
```

Si l’objet imbriqué est destiné à être la valeur envoyée par push à l’instance Configuration, vous pouvez utiliser la valeur de profondeur (*depth*) pour arrêter l’aplatissement à la profondeur appropriée. 

```json
on: 
  push: 
    branches: 
      - 'master' 
    paths: 
      - 'appsettings.json' 
 
jobs: 
  syncconfig: 
    runs-on: ubuntu-latest 
    steps: 
      # checkout done so that files in the repo can be read by the sync 
      - uses: actions/checkout@v1 
      - uses: azure/appconfiguration-sync@v1 
        with: 
          configurationFile: 'appsettings.json' 
          format: 'json' 
          # Replace <ConnectionString> with the name of the secret in your 
          # repository 
          connectionString: ${{ secrets.<ConnectionString> }}  
          separator: ':' 
          depth: 2 
```

Pour une profondeur de 2, l’exemple ci-dessus retourne maintenant la paire clé-valeur suivante :

| Clé | Valeur |
| --- | --- |
| Object:Inner | {"InnerKey":"InnerValue"} |

## <a name="understand-action-inputs"></a>Comprendre les entrées d’action
Les paramètres d’entrée spécifient les données que l’action utilise pendant le runtime.  Le tableau suivant contient les paramètres d’entrée acceptés par la synchronisation d’App Configuration et les valeurs attendues pour chacun d’eux.  Pour plus d’informations sur les entrées d’action pour GitHub Actions, voir la [documentation](https://help.github.com/actions/automating-your-workflow-with-github-actions/metadata-syntax-for-github-actions#inputs) de GitHub.

> [!Note]
> Les ID d’entrée ne respectent pas la casse.


| Nom d’entrée | Requis ? | Valeur |
|----|----|----|
| configurationFile | Oui | Chemin relatif du fichier de configuration dans le dépôt.  Les modèles Glob sont pris en charge et peuvent inclure plusieurs fichiers. |
| format | Oui | Format du fichier de configuration.  Les formats valides sont les suivants : JSON, YAML, propriétés. |
| connectionString | Oui | Chaîne de connexion pour l’instance App Configuration. La chaîne de connexion doit être stockée en tant que secret dans le dépôt GitHub, et seul le nom du secret doit être utilisé dans le flux de travail. |
| separator | Oui | Séparateur utilisé lors de l’aplatissement du fichier de configuration en paires clé-valeur.  Les valeurs valides sont les suivantes : . , ; : - _ __ / |
| prefix | Non | Préfixe à ajouter au début des clés. |
| label | Non | Étiquette utilisée lors de la définition de paires clé-valeur. Si elle n’est pas spécifiée, une étiquette null est utilisée. |
| strict | Non | Valeur booléenne déterminant si le mode strict est activé. La valeur par défaut est false. |
| depth | Non | Profondeur maximale pour l’aplatissement du fichier de configuration.  La profondeur doit être un nombre positif.  La valeur par défaut n’a pas de profondeur maximale. |
| tags | Non | Spécifie la balise définie sur les paires clé-valeur.  Le format attendu est une forme convertie en chaîne d’un objet JSON similaire à la suivante : { [propertyName : string]: string; } Chaque nom-valeur de propriété devient une balise. |

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris l’action GitHub de synchronisation d’App Configuration et la façon de l’utiliser pour automatiser les mises à jour de votre instance App Configuration. Pour savoir comment Azure App Configuration réagit aux modifications des paires clé-valeur, passez à l’[article](./concept-app-configuration-event.md) suivant.
