---
title: Utiliser le contrôle d’intégrité d’Azure Deployment Manager
description: Utilisez le contrôle d’intégrité pour déployer des ressources Azure de manière sécurisée avec Azure Deployment Manager.
author: mumian
ms.date: 10/09/2019
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: 3c7b74d31bc3c4e2276cd52c8e6450630dc99bcd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86058025"
---
# <a name="tutorial-use-health-check-in-azure-deployment-manager-public-preview"></a>Tutoriel : Utiliser le contrôle d’intégrité dans Azure Deployment Manager (préversion publique)

Découvrez comment intégrer le contrôle d’intégrité dans [Azure Deployment Manager](./deployment-manager-overview.md). Ce tutoriel est basé sur le tutoriel [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md). Vous devez suivre ce dernier avant de commencer celui-ci.

Dans le modèle de lancement utilisé dans [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md), vous avez utilisé une étape d’attente. Dans ce tutoriel, vous remplacez l’étape d’attente par une étape de contrôle d’intégrité.

> [!IMPORTANT]
> Si votre abonnement est marqué pour les Canaries pour les tests de nouvelles fonctionnalités Azure, vous pouvez utiliser Azure Deployment Manager uniquement en vue du déploiement dans la région des Canaries. 

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Créer un simulateur de service de contrôle d’intégrité
> * Réviser le modèle de lancement
> * Déployer la topologie
> * Déployer le déploiement avec l’état non sain
> * Vérifier le déploiement du lancement
> * Déployer le lancement avec état sain
> * Vérifier le déploiement du lancement
> * Nettoyer les ressources

Ressources supplémentaires :

* Les [informations de référence de l’API REST Azure Deployment Manager](/rest/api/deploymentmanager/).
* [Un exemple d’Azure Deployment Manager](https://github.com/Azure-Samples/adm-quickstart).

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Effectuez le tutoriel [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md).

## <a name="install-the-artifacts"></a>Installer les artefacts

Téléchargez [les modèles et les artefacts ](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMTutorial.zip) et décompressez-les localement si ce n’est déjà fait. Exécutez ensuite le script PowerShell qui se trouve dans [Préparer les artefacts](./deployment-manager-tutorial.md#prepare-the-artifacts). Le script crée un groupe de ressources, crée un conteneur de stockage, crée un conteneur d’objets blob, charge les fichiers téléchargés, puis crée un jeton SAS.

Effectuez une copie de l’URL avec le jeton SAS. Cette URL est nécessaire pour remplir un champ dans les deux fichiers de paramètres, le fichier de paramètres de topologie et le fichier de paramètres de lancement.

Ouvrez CreateADMServiceTopology.Parameters.json et mettez à jour les valeurs de **projectName** et d’**artifactSourceSASLocation**.

Ouvrez CreateADMRollout.Parameters.json et mettez à jour les valeurs de **projectName** et d’**artifactSourceSASLocation**.

## <a name="create-a-health-check-service-simulator"></a>Créer un simulateur de service de contrôle d’intégrité

En production, vous utilisez généralement un ou plusieurs fournisseurs de supervision. Pour faciliter l’intégration de l’intégrité, Microsoft collabore avec certaines des principales sociétés de supervision de l’intégrité de service pour vous fournir une solution copier/coller simple vous permettant d’intégrer des contrôles d’intégrité à vos déploiements. Pour obtenir la liste de ces sociétés, consultez [Fournisseurs de contrôle d’intégrité](./deployment-manager-health-check.md#health-monitoring-providers). Pour les besoins de ce tutoriel, vous créez une [fonction Azure](../../azure-functions/index.yml) afin de simuler un service de supervision de l’intégrité. Cette fonction prend un code d’état et retourne le même code. Votre modèle Azure Deployment Manager utilise le code d’état pour déterminer comment effectuer le déploiement.

Les deux fichiers suivants sont utilisés pour le déploiement de la fonction Azure. Vous n’avez pas besoin de télécharger ces fichiers pour suivre le tutoriel.

* Un modèle Resource Manager situé sur [https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json). Vous déployez ce modèle pour créer une fonction Azure.
* Un fichier zip contenant le code source de la fonction Azure, [https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMHCFunction0417.zip](https://github.com/Azure/azure-docs-json-samples/raw/master/tutorial-adm/ADMHCFunction0417.zip). Ce fichier zip est appelé par le modèle Resource Manager.

Pour déployer la fonction Azure, sélectionnez **Try it** afin d’ouvrir Azure Cloud shell, puis collez le script suivant dans la fenêtre de l’interpréteur de commandes.  Pour coller le code, cliquez avec le bouton droit sur la fenêtre de l’interpréteur de commandes, puis sélectionnez **Coller**.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/tutorial-adm/deploy_hc_azure_function.json" -projectName $projectName
```

Pour vérifier et tester la fonction Azure

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Ouvrez le groupe de ressources.  Le nom par défaut est le nom du projet avec **rg** ajouté.
1. Sélectionnez le service d’application dans le groupe de ressources.  Le nom par défaut du service d’application est le nom du projet avec **webapp** ajouté.
1. Développez **Fonctions**, puis sélectionnez **HttpTrigger1**.

    ![Fonction Azure de contrôle d’intégrité Azure Deployment Manager](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-function.png)

1. Sélectionnez **&lt;/> Obtenir l’URL de la fonction**.
1. Sélectionnez **Copier** pour copier l’URL dans le Presse-papiers.  L’URL ressemble à celle-ci :

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/{healthStatus}?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    Remplacez `{healthStatus}` dans l’URL par un code d’état. Dans ce tutoriel, utilisez **unhealthy** pour tester le scénario non sain, et utilisez **healthy** ou **warning** pour tester le scénario sain. Créez deux URL, une avec un état non sain et l’autre avec un état sain. Exemples :

    ```url
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    https://myhc0417webapp.azurewebsites.net/api/healthStatus/healthy?code=hc4Y1wY4AqsskAkVw6WLAN1A4E6aB0h3MbQ3YJRF3XtXgHvooaG0aw==
    ```

    Vous avez besoin des deux URL pour effectuer ce tutoriel.

1. Pour tester le simulateur de supervision de l’intégrité, ouvrez les URL que vous avez créées à l’étape précédente.  Les résultats pour l’état non sain doivent ressembler à ceci :

    ```
    Status: unhealthy
    ```

## <a name="revise-the-rollout-template"></a>Réviser le modèle de lancement

L’objectif de cette section est de vous montrer comment inclure une étape de contrôle d’intégrité dans le modèle de lancement.

1. Ouvrez **CreateADMRollout.json** que vous avez créé dans [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md). Ce fichier JSON fait partie du téléchargement.  Consultez les [Conditions préalables](#prerequisites).
1. Ajoutez deux autres paramètres :

    ```json
    "healthCheckUrl": {
      "type": "string",
      "metadata": {
        "description": "Specifies the health check URL."
      }
    },
    "healthCheckAuthAPIKey": {
      "type": "string",
      "metadata": {
          "description": "Specifies the health check Azure Function function authorization key."
      }
    }
    ```

1. Remplacez la définition de ressource d’étape d’attente par une définition de ressource d’étape de contrôle d’intégrité :

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

    En fonction de la définition, le lancement se poursuit si l’état d’intégrité est *healthy* ou *warning*.

1. Mettez à jour le **dependsON** de la définition de lancement pour inclure l’étape de contrôle d’intégrité qui vient d’être définie :

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.DeploymentManager/artifactSources', variables('rolloutArtifactSource').name)]",
      "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
    ],
    ```

1. Mettez à jour **stepGroups** pour inclure l’étape de contrôle d’intégrité. Le **healthCheckStep** est appelé dans **postDeploymentSteps** de **stepGroup2**. **stepGroup3** et **stepGroup4** sont déployés uniquement si l’état sain est *healthy* ou *warning*.

    ```json
    "stepGroups": [
      {
        "name": "stepGroup1",
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
          "postDeploymentSteps": []
        },
        {
          "name": "stepGroup2",
          "dependsOnStepGroups": ["stepGroup1"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
          "postDeploymentSteps": [
            {
              "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
            }
          ]
        },
        {
          "name": "stepGroup3",
          "dependsOnStepGroups": ["stepGroup2"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
           "postDeploymentSteps": []
        },
        {
          "name": "stepGroup4",
          "dependsOnStepGroups": ["stepGroup3"],
          "preDeploymentSteps": [],
          "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
          "postDeploymentSteps": []
        }
    ]
    ```

    Si vous comparez la section **stepGroup3** avant et après sa révision, cette section dépend désormais de **stepGroup2**.  C’est nécessaire quand **stepGroup3** et les groupes d’étapes ultérieurs dépendent des résultats de la supervision de l’intégrité.

    La capture d’écran suivante illustre les zones modifiées ainsi que le mode d’utilisation de l’étape de contrôle d’intégrité :

    ![Modèle de contrôle d’intégrité Azure Deployment Manager](./media/deployment-manager-tutorial-health-check/azure-deployment-manager-hc-rollout-template.png)

## <a name="deploy-the-topology"></a>Déployer la topologie

Exécutez le script PowerShell suivant pour déployer la topologie. Vous avez besoin des fichiers **CreateADMServiceTopology.json** et **CreateADMServiceTopology.Parameters.json** que vous avez utilisés dans [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md).

```azurepowershell
# Create the service topology
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "$filePath\ADMTemplates\CreateADMServiceTopology.json" `
    -TemplateParameterFile "$filePath\ADMTemplates\CreateADMServiceTopology.Parameters.json"
```

Vérifiez que la topologie de service et les ressources soulignées ont été créées avec succès à l’aide du portail Azure :

![Ressources de topologie de service déployées de didacticiel Azure Deployment Manager](./media/deployment-manager-tutorial/azure-deployment-manager-tutorial-deployed-topology-resources.png)

L’option **Afficher les types masqués** doit être sélectionnée pour voir les ressources.

## <a name="deploy-the-rollout-with-the-unhealthy-status"></a>Déployer le lancement avec l’état non sain

Utilisez l’URL d’état non sain créée dans [Créer un simulateur de service de contrôle d’intégrité](#create-a-health-check-service-simulator). Vous avez besoin du fichier **CreateADMServiceTopology.json** révisé et du fichier **CreateADMServiceTopology.Parameters.json** que vous avez utilisé dans [Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md).

```azurepowershell-interactive
$healthCheckUrl = Read-Host -Prompt "Enter the health check Azure function URL"
$healthCheckAuthAPIKey = $healthCheckUrl.Substring($healthCheckUrl.IndexOf("?code=")+6, $healthCheckUrl.Length-$healthCheckUrl.IndexOf("?code=")-6)
$healthCheckUrl = $healthCheckUrl.Substring(0, $healthCheckUrl.IndexOf("?"))

# Create the rollout
New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "$filePath\ADMTemplates\CreateADMRollout.json" `
    -TemplateParameterFile "$filePath\ADMTemplates\CreateADMRollout.Parameters.json" `
    -healthCheckUrl $healthCheckUrl `
    -healthCheckAuthAPIKey $healthCheckAuthAPIKey
```

> [!NOTE]
> `New-AzResourceGroupDeployment` est un appel asynchrone. Le message de réussite signifie uniquement que le déploiement a pu commencer. Pour vérifier le déploiement, utilisez `Get-AZDeploymentManagerRollout`.  Voir la procédure suivante.

Pour vérifier la progression du lancement, utilisez le script PowerShell suivant :

```azurepowershell
$projectName = Read-Host -Prompt "Enter the same project name used earlier in this tutorial"
$resourceGroupName = "${projectName}rg"
$rolloutName = "${projectName}Rollout"

# Get the rollout status
Get-AzDeploymentManagerRollout `
    -ResourceGroupName $resourceGroupName `
    -Name $rolloutName `
    -Verbose
```

L’exemple de sortie suivant montre que le déploiement a échoué en raison de l’état non sain :

```output
Service: myhc0417ServiceWUSrg
    TargetLocation: WestUS
    TargetSubscriptionId: <Subscription ID>

    ServiceUnit: myhc0417ServiceWUSWeb
        TargetResourceGroup: myhc0417ServiceWUSrg

        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
Get-AzDeploymentManagerRollout :
Service: myhc0417ServiceWUSrg
    ServiceUnit: myhc0417ServiceWUSWeb
        Step: RestHealthCheck/healthCheckStep.PostDeploy
            Status: Failed
            StepGroup: stepGroup2
            Operation Info:
                Start Time: 05/06/2019 17:58:31
                End Time: 05/06/2019 17:58:32
                Total Duration: 00:00:01
                Error:
                    Code: ResourceReportedUnhealthy
                    Message: Health checks failed as the following resources were unhealthy: '05/06/2019 17:58:32 UTC: Health check 'appHealth' failed with the following errors: Response from endpoint 'https://myhc0417webapp.azurewebsites.net/api/healthStatus/unhealthy' does not match the regex pattern(s): 'Status: healthy, Status: warning.'. Response content: "Status: unhealthy"..'.
At line:1 char:1
+ Get-AzDeploymentManagerRollout `
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : NotSpecified: (:) [Get-AzDeploymentManagerRollout], Exception
+ FullyQualifiedErrorId : RolloutFailed,Microsoft.Azure.Commands.DeploymentManager.Commands.GetRollout


ResourceGroupName       : myhc0417rg
BuildVersion            : 1.0.0.0
ArtifactSourceId        : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/artifactSources/myhc0417ArtifactSourceRollout
TargetServiceTopologyId : /subscriptions/<Subscription ID>/resourceGroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/serviceTopologies/myhc0417ServiceTopology
Status                  : Failed
TotalRetryAttempts      : 0
Identity                : Microsoft.Azure.Commands.DeploymentManager.Models.PSIdentity
OperationInfo           : Microsoft.Azure.Commands.DeploymentManager.Models.PSRolloutOperationInfo
Services                : {myhc0417ServiceWUS, myhc0417ServiceWUSrg}
Name                    : myhc0417Rollout
Type                    : Microsoft.DeploymentManager/rollouts
Location                : centralus
Id                      : /subscriptions/<Subscription ID>/resourcegroups/myhc0417rg/providers/Mi
                          crosoft.DeploymentManager/rollouts/myhc0417Rollout
Tags                    :
```

Une fois le lancement terminé, vous verrez un groupe de ressources supplémentaire créé pour la région USA Ouest.

## <a name="deploy-the-rollout-with-the-healthy-status"></a>Déployer le lancement avec l’état sain

Répétez cette section pour redéployer le lancement avec l’URL d’état sain.  Une fois le lancement terminé, vous verrez un groupe de ressources supplémentaire créé pour la région USA Est.

## <a name="verify-the-deployment"></a>Vérifier le déploiement

1. Ouvrez le [portail Azure](https://portal.azure.com).
2. Accédez aux applications web nouvellement créées sous les nouveaux groupes de ressources créés par le déploiement du lancement.
3. Ouvrez l’application web dans un navigateur web. Vérifiez l’emplacement et la version sur le fichier index.html.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Utilisez le champ **Filter by name** (Filtrer par nom) afin de réduire les groupes de ressources créés dans ce didacticiel. Il doivent être au nombre de 3-4 :

    * **&lt;projectName>rg** : contient les ressources Deployment Manager.
    * **&lt;projectName>ServiceWUSrg** : contient les ressources définies par ServiceWUS.
    * **&lt;projectName>ServiceEUSrg** : contient les ressources définies par ServiceEUS.
    * Le groupe de ressources pour l’identité managée définie par l’utilisateur.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.
5. Répétez les deux dernières étapes pour supprimer d’autres groupes de ressources créés lors de ce didacticiel.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser la fonctionnalité de contrôle d’intégrité d’Azure Deployment Manager. Pour en savoir plus, consultez la [Documentation Azure Resource Manager](../index.yml).
