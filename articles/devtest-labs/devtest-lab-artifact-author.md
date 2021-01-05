---
title: Créer des artefacts personnalisés pour votre machine virtuelle DevTest Labs | Microsoft Docs
description: Découvrez comment créer des artefacts à utiliser avec Azure DevTest Labs pour déployer et configurer des applications après avoir approvisionner une machine virtuelle.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 85acfcc3811e671e58fadab08a23951778e1323d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88270680"
---
# <a name="create-custom-artifacts-for-your-devtest-labs-virtual-machine"></a>Créer des artefacts personnalisés pour votre machine virtuelle DevTest Labs

Regardez la vidéo suivante pour avoir une vue d’ensemble de la procédure décrite dans cet article :

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/how-to-author-custom-artifacts/player]
>
>

## <a name="overview"></a>Vue d’ensemble
Vous pouvez utiliser des *artefacts* pour déployer et configurer votre application après avoir approvisionné une machine virtuelle. Un artefact se compose d’un fichier de définition d’artefact et autres fichiers de script qui sont stockés dans un dossier de dépôt Git. Les fichiers de définition d'artefact se composent de JSON et d'expressions que vous pouvez utiliser pour spécifier ce que vous voulez installer sur une machine virtuelle. Par exemple, vous pouvez définir le nom d’un artefact, une commande à exécuter et des paramètres disponibles quand la commande est exécutée. Vous pouvez faire référence à d'autres fichiers de script dans le fichier de définition d'artefact par nom.

## <a name="artifact-definition-file-format"></a>Format de fichier de définition d'artefact
L'exemple suivant indique les sections qui composent la structure de base d'un fichier de définition :

```json
  {
    "$schema": "https://raw.githubusercontent.com/Azure/azure-devtestlab/master/schemas/2016-11-28/dtlArtifacts.json",
    "title": "",
    "description": "",
    "iconUri": "",
    "targetOsType": "",
    "parameters": {
      "<parameterName>": {
        "type": "",
        "displayName": "",
        "description": ""
      }
    },
    "runCommand": {
      "commandToExecute": ""
    }
  }
```

| Nom de l'élément | Requis ? | Description |
| --- | --- | --- |
| $schema |Non |Emplacement du fichier de schéma JSON. Le fichier de schéma JSON peut vous aider à tester la validité du fichier de définition. |
| title |Oui |Nom de l'artefact affiché dans le laboratoire. |
| description |Oui |Description de l'artefact affiché dans le laboratoire. |
| iconUri |Non |URI de l’icône affichée dans le laboratoire. |
| targetOsType |Oui |Système d’exploitation de la machine virtuelle où l’artefact est installé. Les options prises en charge sont Windows et Linux. |
| parameters |Non |Valeurs fournies quand la commande d’installation d’artefact est exécutée sur une machine. Cela vous permet de personnaliser votre artefact. |
| runCommand |Oui |Commande d'installation d'artefact qui est exécutée sur une machine virtuelle. |

### <a name="artifact-parameters"></a>Paramètres d'artefact
Dans la section des paramètres du fichier de définition, spécifiez les valeurs qu’un utilisateur peut entrer quand il installe un artefact. Vous pouvez faire référence à ces valeurs dans la commande d'installation d'artefact.

Pour définir des paramètres, utilisez la structure suivante :

```json
  "parameters": {
    "<parameterName>": {
      "type": "<type-of-parameter-value>",
      "displayName": "<display-name-of-parameter>",
      "description": "<description-of-parameter>"
    }
  }
```

| Nom de l'élément | Requis ? | Description |
| --- | --- | --- |
| type |Oui |Type de la valeur du paramètre. Consultez la liste suivante des types autorisés. |
| displayName |Oui |Nom du paramètre qui est affiché à un utilisateur dans le laboratoire. |
| description |Oui |Description du paramètre qui est affiché dans le laboratoire. |

Les types autorisés sont :

* string (n’importe quelle chaîne JSON valide)
* int (n’importe quel entier JSON valide)
* bool (n’importe quel booléen JSON valide)
* array (n’importe quel tableau JSON valide)

## <a name="secrets-as-secure-strings"></a>Secrets sous forme de chaînes sécurisées
Déclarez des secrets sous forme de chaînes sécurisées. Voici la syntaxe de déclaration d’un paramètre de chaîne sécurisée dans la section `parameters` du fichier **artifactfile.json** :

```json

    "securestringParam": {
      "type": "securestring",
      "displayName": "Secure String Parameter",
      "description": "Any text string is allowed, including spaces, and will be presented in UI as masked characters.",
      "allowEmpty": false
    },
```

Pour la commande d’installation d’artefact, exécutez le script PowerShell qui accepte la chaîne sécurisée créée à l’aide de la commande ConvertTo-SecureString. 

```json
  "runCommand": {
    "commandToExecute": "[concat('powershell.exe -ExecutionPolicy bypass \"& ./artifact.ps1 -StringParam ''', parameters('stringParam'), ''' -SecureStringParam (ConvertTo-SecureString ''', parameters('securestringParam'), ''' -AsPlainText -Force) -IntParam ', parameters('intParam'), ' -BoolParam:$', parameters('boolParam'), ' -FileContentsParam ''', parameters('fileContentsParam'), ''' -ExtraLogLines ', parameters('extraLogLines'), ' -ForceFail:$', parameters('forceFail'), '\"')]"
  }
```

Pour obtenir l’exemple de fichier artifactfile.json complet et le fichier artifact.ps1 (script PowerShell), consultez [cet exemple sur GitHub](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts/windows-test-paramtypes).

Un autre point important à retenir est de ne pas journaliser les secrets dans la console, car la sortie est capturée en vue du débogage utilisateur. 

## <a name="artifact-expressions-and-functions"></a>Expressions et fonctions d'artefact
Vous pouvez utiliser des expressions et des fonctions pour construire la commande d’installation d’artefact.
Les expressions sont placées entre crochets ([ et ]) et sont évaluées au moment où l'artefact est installé. Les expressions peuvent apparaître n’importe où dans une valeur de chaîne JSON. Les expressions retournent toujours une autre valeur JSON. Si vous avez besoin d’utiliser une chaîne littérale qui commence par un crochet ([), vous devez utiliser deux crochets ([[).
En général, vous utilisez des expressions avec des fonctions pour construire une valeur. Exactement comme dans JavaScript, les appels de fonctions sont mis en forme de la manière suivante : **functionName(arg1, arg2, arg3)** .

La liste suivante indique les fonctions courantes :

* **parameters(parameterName)** : retourne une valeur de paramètre qui est fournie à l’exécution de la commande de l’artefact.
* **concat(arg1, arg2, arg3,….. )** : combine plusieurs valeurs de chaîne. Cette fonction peut prendre de nombreux arguments.

L’exemple suivant indique comment utiliser les fonctions et les expressions pour construire une valeur :

```json
  runCommand": {
      "commandToExecute": "[concat('powershell.exe -ExecutionPolicy bypass \"& ./startChocolatey.ps1'
  , ' -RawPackagesList ', parameters('packages')
  , ' -Username ', parameters('installUsername')
  , ' -Password ', parameters('installPassword'))]"
  }
```

## <a name="create-a-custom-artifact"></a>Création d'un artefact personnalisé

1. Installer un éditeur JSON. Vous avez besoin d’un éditeur JSON pour utiliser les fichiers de définition d’artefact. Nous vous recommandons d’utiliser [Visual Studio Code](https://code.visualstudio.com/), qui est disponible pour Windows, Linux et OS X.
2. Obtenir un exemple de fichier de définition artifactfile.json. Découvrez les artefacts créés par l’équipe DevTest Labs dans notre [dépôt GitHub](https://github.com/Azure/azure-devtestlab). Nous avons créé une bibliothèque d’artefacts riche qui vous permet de créer vos propres artefacts. Téléchargez un fichier de définition d'artefact et modifiez-le pour créer vos propres artefacts.
3. Utiliser IntelliSense. Utilisez IntelliSense pour voir des éléments valides que vous pouvez utiliser pour construire un fichier de définition d’artefact. Vous pouvez également voir les différentes options pour les valeurs d’un élément. Par exemple, quand vous modifiez l’élément **targetOsType**, IntelliSense vous montre deux options, pour Windows ou Linux.
4. Stockez l’artefact dans le [dépôt Git public pour DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/Artifacts) ou [votre propre dépôt Git](devtest-lab-add-artifact-repo.md). Dans le dépôt public, vous pouvez afficher les artefacts partagés par d’autres utilisateurs et vous pouvez les utiliser directement ou les personnaliser en fonction de vos besoins.
   
   1. Créez un répertoire distinct pour chaque artefact. Le nom du répertoire doit être le même que le nom de l’artefact.
   2. Stockez le fichier de définition d’artefact (artifactfile.json) dans le répertoire que vous avez créé.
   3. Stockez les scripts qui sont référencés à partir de la commande d'installation d'artefact.
      
      Voici un exemple de dossier d'artefact :
      
      ![Exemple de dossier d’artefact](./media/devtest-lab-artifact-author/git-repo.png)
5. Si vous utilisez votre propre dépôt pour stocker les artefacts, ajoutez le dépôt au lab en suivant les instructions fournies dans l’article : [Ajouter un dépôt Git des artefacts et des modèles](devtest-lab-add-artifact-repo.md).

## <a name="related-articles"></a>Articles connexes
* [Guide pratique pour diagnostiquer les échecs d’artefact dans DevTest Labs](devtest-lab-troubleshoot-artifact-failure.md)
* [Joindre une machine virtuelle à un domaine Active Directory existant à l’aide d’un modèle Resource Manager dans DevTest Labs](https://www.visualstudiogeeks.com/blog/DevOps/Join-a-VM-to-existing-AD-domain-using-ARM-template-AzureDevTestLabs)

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [ajouter un dépôt d’artefacts Git à un laboratoire](devtest-lab-add-artifact-repo.md).
