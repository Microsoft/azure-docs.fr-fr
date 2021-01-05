---
title: Utiliser le package NuGet CI/CD Azure Stream Analytics
description: Cet article explique comment utiliser le package NuGet CI/CD Azure Stream Analytics pour configurer un processus de déploiement et d’intégration continus.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/15/2019
ms.openlocfilehash: c05db2d9ba184da89665a236994c851355cc2644
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019852"
---
# <a name="use-the-azure-stream-analytics-cicd-nuget-package-for-integration-and-development"></a>Utiliser le package NuGet CI/CD Azure Stream Analytics pour l’intégration et le développement 
Cet article explique comment utiliser le package NuGet CI/CD Azure Stream Analytics pour configurer un processus de déploiement et d’intégration continus.

Utilisez la version 2.3.0000.0 ou une version ultérieure des [outils Stream Analytics pour Visual Studio](./stream-analytics-quick-create-vs.md) pour que MSBuild soit pris en charge.

Un package NuGet est disponible : [Microsoft.Azure.Stream Analytics.CICD](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/). Il fournit MSBuild, une série locale et des outils de déploiement qui prennent en charge le processus de déploiement et d’intégration continus des [projets Visual Studio Stream Analytics](stream-analytics-vs-tools.md). 
> [!NOTE]
> Le package NuGet peut uniquement être utilisé avec la version 2.3.0000.0 ou une version ultérieure des outils Stream Analytics pour Visual Studio. Si vous avez des projets créés dans les versions précédentes des outils Visual Studio, ouvrez-les simplement avec la version 2.3.0000.0 ou une version ultérieure, et enregistrez-les. Ensuite, les nouvelles fonctionnalités sont activées. 

Pour plus d’informations, consultez la page [Outils Stream Analytics pour Visual Studio](./stream-analytics-quick-create-vs.md).

## <a name="msbuild"></a>MSBuild
Comme l’expérience standard Visual Studio MSBuild, vous avez deux options pour générer un projet. Vous pouvez cliquer sur le projet avec le bouton droit, puis choisir **Générer**. Vous pouvez également utiliser **MSBuild** dans le package NuGet à partir de la ligne de commande.
```
./build/msbuild /t:build [Your Project Full Path] /p:CompilerTaskAssemblyFile=Microsoft.WindowsAzure.StreamAnalytics.Common.CompileService.dll  /p:ASATargetsFilePath="[NuGet Package Local Path]\build\StreamAnalytics.targets"

```

Quand un projet Visual Studio Stream Analytics est correctement généré, il crée les deux fichiers de modèles Azure Resource Manager suivants sous le dossier **bin/[Debug/Retail]/Deploy** : 

* Fichier de modèle Resource Manager

   `[ProjectName].JobTemplate.json`

* Fichier de paramètres Resource Manager
   
   `[ProjectName].JobTemplate.parameters.json`

Les paramètres par défaut dans le fichier parameters.json proviennent des paramètres de votre projet Visual Studio. Si vous voulez effectuer un déploiement dans un autre environnement, remplacez les paramètres en conséquence.

> [!NOTE]
> Pour toutes les informations d’identification, les valeurs par défaut sont Null. Vous **devez** les définir avant le déploiement dans le cloud.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```
Découvrez comment [effectuer un déploiement avec un fichier de modèle Resource Manager et Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md). Découvrez comment [utiliser un objet en tant que paramètre dans un modèle Resource Manager](/azure/architecture/building-blocks/extending-templates/objects-as-parameters).

Pour utiliser une identité managée pour Azure Data Lake Store Gen1 comme récepteur de sortie, vous devez fournir l’accès au principal de service à l’aide de PowerShell avant de déployer sur Azure. Découvrez plus d’informations sur le [déploiement d’ADLS Gen1 avec une identité managée et le modèle Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).


## <a name="command-line-tool"></a>Outil en ligne de commande

### <a name="build-the-project"></a>Créer le projet
Le package NuGet contient un outil en ligne de commande appelé **SA.exe**. Il prend en charge la génération du projet et le test local sur un ordinateur arbitraire, que vous pouvez utiliser dans votre processus de diffusion et d’intégration continues. 

Par défaut, les fichiers de déploiement sont placés sous le répertoire actif. Vous pouvez spécifier le chemin de sortie à l’aide du paramètre -OutputPath suivant :

```
./tools/SA.exe build -Project [Your Project Full Path] [-OutputPath <outputPath>] 
```

### <a name="test-the-script-locally"></a>Tester le script localement

Si votre projet a spécifié des fichiers d’entrée en local dans Visual Studio, vous pouvez exécuter un test de script automatisé à l’aide de la commande *localrun*. Le résultat de la sortie est placé sous le répertoire actif.
 
```
localrun -Project [ProjectFullPath]
```

### <a name="generate-a-job-definition-file-to-use-with-the-stream-analytics-powershell-api"></a>Générer un fichier de définition de travail à utiliser avec l’API PowerShell Stream Analytics

La commande *arm* prend le modèle de travail et les fichiers de paramètres de modèle de travail générés par la build en tant qu’entrée. Elle les combine ensuite dans un fichier JSON de définition de travail qui peut être utilisé avec l’API PowerShell Stream Analytics.

```powershell
arm -JobTemplate <templateFilePath> -JobParameterFile <jobParameterFilePath> [-OutputFile <asaArmFilePath>]
```
Exemple :
```powershell
./tools/SA.exe arm -JobTemplate "ProjectA.JobTemplate.json" -JobParameterFile "ProjectA.JobTemplate.parameters.json" -OutputFile "JobDefinition.json" 
```



## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un travail cloud Azure Stream Analytics dans Visual Studio](stream-analytics-quick-create-vs.md)
* [Tester des requêtes Stream Analytics localement avec Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Explorer des travaux Azure Stream Analytics avec Visual Studio](stream-analytics-vs-tools.md)