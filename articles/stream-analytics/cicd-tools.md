---
title: Automatiser les builds, les tests et les déploiements d’une tâche Azure Stream Analytics à l’aide des outils CI/CD
description: Cet article explique comment utiliser les outils CI/CD d’Azure Stream Analytics pour générer, tester et déployer automatiquement un projet Azure Stream Analytics.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: aa75a553ffc131f4827aa045849f1317d894ddc5
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123148"
---
# <a name="automate-builds-tests-and-deployments-of-an-azure-stream-analytics-job-using-cicd-tools"></a>Automatiser les builds, les tests et les déploiements d’une tâche Azure Stream Analytics à l’aide des outils CI/CD

Vous pouvez utiliser le package npm CI/CD d’Azure Stream Analytics pour générer, tester et déployer automatiquement vos projets Azure Stream Analytics pour Visual Studio Code ou Visual Studio. Les projets peuvent être créés à l’aide d’outils de développement ou exportés à partir de tâches Stream Analytics existantes. Cet article explique comment utiliser le package npm avec n’importe quel système CI/CD. Pour déployer avec Azure Pipelines, consultez [Utiliser Azure DevOps pour créer un pipeline CI/CD pour une tâche Stream Analytics](set-up-cicd-pipeline.md).

## <a name="installation"></a>Installation

Vous pouvez [télécharger le package](https://www.npmjs.com/package/azure-streamanalytics-cicd) directement, ou l’installer [globalement](https://docs.npmjs.com/downloading-and-installing-packages-globally) à l’aide de la commande `npm install -g azure-streamanalytics-cicd`. Nous recommandons d’utiliser la commande, qui peut également être utilisée dans une tâche de script PowerShell ou Azure CLI d’un pipeline de build dans **Azure Pipelines**.

## <a name="build-the-project"></a>Créer le projet

Le package npm **asa-streamanalytics-cicd** fournit les outils pour générer les modèles Azure Resource Manager des projets Stream Analytics pour [Visual Studio Code](./quick-create-visual-studio-code.md) ou [Visual Studio](stream-analytics-quick-create-vs.md). Vous pouvez également utiliser le package npm sur Windows, macOS et Linux sans installer Visual Studio Code ou Visual Studio.

Une fois que vous avez installé le package, utilisez la commande suivante pour générer vos projets Stream Analytics.

```powershell
azure-streamanalytics-cicd build -project <projectFullPath> [-outputPath <outputPath>]
```

La commande *build* effectue une vérification syntaxique de mot clé et génère le modèle Azure Resource Manager.

| Paramètre | Description |
|---|---|
| `-project` | Chemin d'accès absolu du fichier **asaproj.json** pour votre projet Visual Studio Code ou **[Nom de votre projet].asaproj** pour le projet Visual Studio. |
| `-outputPath` | Chemin d’accès du dossier de sortie pour les modèles Azure Resource Manager. S’il n’est pas spécifié, les modèles sont placés dans le répertoire actif. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd build -project "/Users/username/projects/samplejob/samplejob.asaproj"
```

--- 

Quand un projet Stream Analytics est généré avec succès, il génère les deux fichiers suivants sous le dossier de sortie :

* Fichier de modèle Azure Resource Manager

   `[ProjectName].JobTemplate.json`

* Fichier de paramètres Azure Resource Manager

   `[ProjectName].JobTemplate.parameters.json`

Les paramètres par défaut dans le fichier parameters.json proviennent des paramètres de votre projet Visual Studio Code ou Visual Studio. Si vous voulez effectuer un déploiement dans un autre environnement, remplacez les paramètres en conséquence.

Les valeurs par défaut de toutes les informations d’identification sont **null**. Vous devez définir les valeurs avant le déploiement sur Azure.

```json
"Input_EntryStream_sharedAccessPolicyKey": {
      "value": null
    },
```

Pour utiliser une identité gérée pour Azure Data Lake Store Gen1 comme récepteur de sortie, vous devez fournir l’accès au principal de service à l’aide de PowerShell avant de déployer sur Azure. Découvrez plus d’informations sur le [déploiement d’ADLS Gen1 avec une identité managée et le modèle Resource Manager](stream-analytics-managed-identities-adls.md#resource-manager-template-deployment).

## <a name="local-run"></a>Exécution locale

Si votre projet a spécifié des fichiers d’entrée locaux, vous pouvez exécuter un script Stream Analytics localement à l’aide de la commande `localrun`.

```powershell
azure-streamanalytics-cicd localrun -project <projectFullPath> [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Paramètre | Description |
|---|---|
| `-project` | Chemin d'accès du fichier **asaproj.json** pour votre projet Visual Studio Code ou **[Nom de votre projet].asaproj** pour le projet Visual Studio. |
| `-outputPath` | Chemin d’accès du dossier de sortie. S’il n’est pas spécifié, les fichiers de résultats de sortie sont placés dans le répertoire actif. |
| `-customCodeZipFilePath` | Chemin d’accès du fichier zip du code personnalisé C#, tel qu’un fichier UDF ou un désérialiseur, s’ils sont utilisés. Regroupez les DLL dans un fichier zip et indiquez ce chemin. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd localrun -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```

---

> [!Note] 
> UDF JavaScript fonctionne uniquement sur Windows.

## <a name="automated-test"></a>Test automatisé

Vous pouvez utiliser le package npm CI/CD pour configurer et exécuter des tests automatisés pour votre script Stream Analytics.

### <a name="add-a-test-case"></a>Ajouter un cas de test

Les cas de test sont décrits dans un fichier de configuration de test. Pour commencer, utilisez la commande `addtestcase` pour ajouter un modèle de cas de test au fichier de configuration de test. Si le fichier de configuration de test n’existe pas, il est créé par défaut.

```powershell
azure-streamanalytics-cicd addtestcase -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>]
```

| Paramètre | Description |
|---|---|
| `-project` | Chemin d'accès du fichier **asaproj.json** pour votre projet Visual Studio Code ou **[Nom de votre projet].asaproj** pour le projet Visual Studio. |
| `-testConfigPath` | Chemin d'accès du fichier de configuration de test. S’il n’est pas spécifié, le fichier est recherché dans **\test** sous le répertoire actif du fichier **asaproj.json** , avec le nom de fichier par défaut **testConfig.json**. S’il n’existe pas, in nouveau fichier est créé. |

#### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/asaproj.json"
```

#### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
azure-streamanalytics-cicd addtestcase -project "/Users/roger/projects/samplejob/samplejob.asaproj"
```
---

Si le fichier de configuration de test est vide, le contenu suivant est écrit dans le fichier. Dans le cas contraire, un cas de test est ajouté au tableau de **TestCases**. Les configurations d’entrée nécessaires sont automatiquement remplies en fonction des fichiers de configuration d’entrée, s’ils existent. Dans le cas contraire, les valeurs par défaut sont configurées. Le **chemin d’accès au fichier** de chaque entrée et sortie attendue doit être spécifié avant d’exécuter le test. Vous pouvez modifier la configuration manuellement.

Si vous souhaitez que la validation de test ignore une sortie donnée, définissez le champ **obligatoire** de cette sortie attendue sur **false**.

```json
{
  "Script": "",
  "TestCases": [
    {
      "Name": "Case 1",
      "Inputs": [
        {
          "InputAlias": [Input alias string],
          "Type": "Data Stream",
          "Format": "JSON",
          "FilePath": [Required],
          "ScriptType": "InputMock"
        }
      ],
      "ExpectedOutputs": [
        {
          "OutputAlias": [Output alias string],
          "FilePath": "Required",
          "Required": true
        }
      ]
    }
  ]
}
```

### <a name="run-a-unit-test"></a>Exécutez un test unitaire

Vous pouvez utiliser la commande suivante pour exécuter plusieurs cas de test pour votre projet. Un résumé des résultats des tests est généré dans le dossier de sortie. Le processus se termine avec le code **0** pour tous les tests réussis ; **-1** pour les exceptions ; **-2** pour les tests échoués.

```powershell
azure-streamanalytics-cicd test -project <projectFullPath> [-testConfigPath <testConfigFileFullPath>] [-outputPath <outputPath>] [-customCodeZipFilePath <zipFilePath>]
```

| Paramètre | Description |
|---|---|
| `-project` | Chemin d'accès du fichier **asaproj.json** pour votre projet Visual Studio Code ou **[Nom de votre projet].asaproj** pour le projet Visual Studio. |
| `-testConfigPath` | Chemin d'accès au fichier de configuration de test. S’il n’est pas spécifié, le fichier est recherché dans **\test** sous le répertoire actif du fichier **asaproj.json** , avec le nom de fichier par défaut **testConfig.json**.
| `-outputPath` | Chemin d’accès au dossier de sortie des résultats des tests. S’il n’est pas spécifié, les fichiers de résultats de sortie sont placés dans le répertoire actif. |
| `-customCodeZipFilePath` | Chemin d’accès du fichier zip du code personnalisé, tel qu’un fichier UDF ou un désérialiseur, s’ils sont utilisés. |

Une fois tous les tests terminés, un résumé des résultats des tests au format JSON est généré dans le dossier de sortie. Le fichier résumé est nommé **testResultSummary.json**.

```json
{
  "Total": (integer) total_number_of_test_cases,
  "Passed": (integer) number_of_passed_test_cases,
  "Failed": (integer) number_of_failed_test_cases,
  "Script": (string) absolute_path_to_asaql_file,
  "Results": [ (array) detailed_results_of_test_cases
    {
      "Name": (string) name_of_test_case,
      "Status": (integer) 0(passed)_or_1(failed),
      "Time": (string) time_span_of_running_test_case,
      "OutputMatched": [ (array) records_of_actual_outputs_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputNotEqual": [ (array) records_of_actual_outputs_not_equal_to_expected_outputs
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputMissing": [ (array) records_of_actual_outputs_missing
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": ""
        }
      ],
      "OutputUnexpected": [ (array) records_of_actual_outputs_unexpected
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": "",
          "Output": (string) path_to_the_actual_output_file
        }
      ],
      "OutputUnrequired": [ (array) records_of_actual_outputs_unrequired_to_be_checked
        {
          "OutputAlias": (string) output_alias,
          "ExpectedOutput": (string) path_to_the_expected_output_file,
          "Output": (string) path_to_the_actual_output_file
        }
      ]
    }
  ],
  "Time": (string) time_span_of_running_all_test_cases,
}
```

## <a name="deploy-to-azure"></a>Déployer sur Azure

Vous pouvez utiliser le modèle Azure Resource Manager et les fichiers de paramètres générés à partir du build pour [déployer votre tâche sur Azure](../azure-resource-manager/templates/template-tutorial-use-parameter-file.md?tabs=azure-powershell#deploy-template).

## <a name="next-steps"></a>Étapes suivantes

* [Intégration et déploiement continus sur Azure Stream Analytics](cicd-overview.md)
* [Configurer un pipeline CI/CD pour une tâche Stream Analytics à l’aide d’Azure Pipelines](set-up-cicd-pipeline.md)