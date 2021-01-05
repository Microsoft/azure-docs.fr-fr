---
title: Lancement de l’intégration de l’intégrité - Manager de déploiement Azure
description: Explique comment déployer un service sur plusieurs régions avec Azure Deployment Manager. Montre des pratiques de déploiement sécurisé consistant à vérifier la stabilité d’un déploiement avant sa propagation à toutes les régions.
author: mumian
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: jgao
ms.openlocfilehash: 8e13f18ef9c2ec71a351d8a80d72d600bf8e9715
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607403"
---
# <a name="introduce-health-integration-rollout-to-azure-deployment-manager-public-preview"></a>Introduire le lancement de l’intégration de l’intégrité dans Azure Deployment Manager (préversion publique)

[Azure Deployment Manager](./deployment-manager-overview.md) vous permet d’effectuer les lancements intermédiaires des ressources Azure Resource Manager. Les ressources sont déployées région par région de manière ordonnée. La vérification d’intégrité intégrée d’Azure Deployment Manager peut surveiller les lancements et interrompre automatiquement ceux qui posent problème, de sorte que vous puissiez les résoudre et ainsi limiter leur impact. Cette fonctionnalité peut limiter l’indisponibilité du service provoquée par des régressions dans les mises à jour.

## <a name="health-monitoring-providers"></a>Fournisseurs de surveillance de l’intégrité

Pour faciliter l’intégration de l’intégrité, Microsoft collabore avec certaines des principales sociétés de supervision de l’intégrité de service pour vous fournir une solution copier/coller simple vous permettant d’intégrer des contrôles d’intégrité à vos déploiements. Si vous n’utilisez pas déjà un analyseur d’intégrité, ces solutions sont excellentes pour démarrer :

| ![fournisseur analyseur d’intégrité azure deployment manager azure monitor](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-azure-monitor.svg)| ![datadog fournisseur analyseur d’intégrité azure deployment manager](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-datadog.svg) | ![fournisseur analyseur d’intégrité azure deployment manager site24x7](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-site24x7.svg) | ![wavefront fournisseur analyseur d’intégrité azure deployment manager](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-wavefront.svg) |
|-----|-----|------|------|
|Azure Monitor, la plateforme d'observabilité complète de la pile de Microsoft dédiée à la surveillance et à l'analytique en modes natif Cloud et hybride. |DataDog, plateforme leader de surveillance et d’analytique pour des environnements de cloud modernes. Découvrez [comment Datadog s’intègre avec Azure Deployment Manager](https://www.datadoghq.com/azure-deployment-manager/).|Site24x7, solution tout-en-un pour la surveillance des services du cloud privé et public. Découvrez [comment Site24x7 s’intègre avec Azure Deployment Manager](https://www.site24x7.com/azure/adm.html).| Wavefront, la plateforme de surveillance et d’analytique pour les environnements d’application multi-cloud. Découvrez [comment Wavefront s’intègre avec Azure Deployment Manager](https://go.wavefront.com/wavefront-adm/).|

## <a name="how-service-health-is-determined"></a>Évaluation de l’intégrité du service

[Les fournisseurs de surveillance de l’intégrité](#health-monitoring-providers) offre plusieurs moyens pour surveiller des services et vous alerter en cas de problèmes d’intégrité. [Azure Monitor](../../azure-monitor/overview.md) en fait partie. Azure Monitor peut être utilisé pour créer des alertes lorsque certains seuils sont dépassés. Par exemple, l’utilisation de votre mémoire et de votre processeur monte en flèche et dépasse les niveaux prévus lors du déploiement d’une nouvelle mise à jour sur votre service. Lorsque vous êtes notifié, vous pouvez entreprendre les actions correctives.

Ces fournisseurs d’analyse d’intégrité offrent en général des API REST afin que le statut des analyseurs de vos services puissent être examinés par programmation. Les API REST peuvent revenir avec un simple signal sain/non sain (déterminé par le code de réponse HTTP) et/ou avec des informations détaillées sur les signaux reçus.

La nouvelle étape *healthCheck* dans Azure Deployment Manager vous permet de déclarer des codes HTTP qui indiquent un service sain ou, pour des résultats REST plus complexes, vous pouvez même spécifier des expressions régulières qui, si correspondance il y a, indiquent une réponse « sain ».

Le flux de la configuration des vérifications de l’intégrité avec Azure Deployment Manager :

1. Créez vos analyseurs d’intégrité via un fournisseur d’analyse d’intégrité de votre choix.
1. Créez une ou plusieurs étapes healthCheck dans le cadre de votre lancement d’Azure Deployment Manager. Remplissez les étapes healthCheck avec les informations suivantes :

    1. L’URI de l’API REST de vos analyseurs d’intégrité (comme définis par votre fournisseur).
    1. Informations d'authentification. Actuellement, seule l’authentification par clé API est prise en charge. Pour Azure Monitor, le type d'authentification doit être défini sur « RolloutIdentity » car l'identité managée affectée par l'utilisateur qui est utilisée pour le lancement d'Azure Deployment Manager s'étend à Azure Monitor.
    1. Les [codes de statut HTTP](https://www.wikipedia.org/wiki/List_of_HTTP_status_codes) ou les expressions régulières qui définissent une réponse « sain ». Remarque : vous pouvez fournir des expressions régulières qui doivent TOUTES correspondre afin que la réponse soit considérée comme « sain », ou vous pouvez fournir des expressions dans lesquelles N’IMPORTE LAQUELLE doit correspondre pour que la réponse soit considérée comme « sain ». Les deux méthodes sont prises en charge.

    Le code JSON suivant est un exemple d'intégration d'Azure Monitor à Azure Deployment Manager qui utilise RolloutIdentity et établit un contrôle d'intégrité dans lequel un lancement se poursuit s'il n'y a aucune alerte. La seule API prise en charge par Azure Monitor est : [Alertes - Tout obtenir](/rest/api/monitor/alertsmanagement/alerts/getall).

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT1M",
          "maxElasticDuration": "PT1M",
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
                    "type": "RolloutIdentity"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "\"value\":\\[\\]"
                    ],
                    "matchQuantifier": "All"
                  }
                }
              }
            ]
          }
        }
      }
    }
    ```

    Le code JSON suivant est un exemple qui s'applique à tous les autres fournisseurs de contrôle d'intégrité :

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

1. Appelez les étapes healthCheck au moment adéquat dans votre lancement d’Azure Deployment Manager. Dans l’exemple suivant, une étape de vérification d’intégrité est appelée dans **postDeploymentSteps** de **stepGroup2**.

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

Pour parcourir un exemple, consultez [Tutoriel : Utiliser le contrôle d’intégrité dans Azure Deployment Manager](./deployment-manager-health-check.md).

## <a name="phases-of-a-health-check"></a>Phases d’une vérification d’intégrité

À ce stade, Azure Deployment Manager sait comment interroger l’intégrité de votre service et lors de quelles phases de votre lancement le faire. Toutefois, Azure Deployment Manager permet aussi des configurations poussées de la période de ces vérifications. Une étape healthCheck est exécutée en 3 phases séquentielles, avec des durées configurables :

1. Wait

    1. Après la fin d’une opération de déploiement, les machines virtuelles peut redémarrer, se reconfigurer en fonction des nouvelles données ou même démarrer pour la première fois. Cela prend du temps avant que les services ne commencent à envoyer des signaux d’intégrité afin qu’ils soient regroupés par le fournisseur d’analyse d’intégrité en quelque chose d’utile. Pendant ce processus houleux, il n’est pas très utile de vérifier l’intégrité du service, car la mise à jour n’a pas encore de statut stable. En effet, le service peut passer d’un état sain à non sain le temps que les ressources s’installent.
    1. Pendant la phase d’attente, l’intégrité du service n’est pas surveillé. Cela permet aux ressources d’avoir le temps de s’installer avant le début du processus de vérification de l’intégrité.
1. Elastic

    1. Comme il est impossible de savoir dans tous les cas le temps que mettront les ressources à s’installer avant d’être stables, la phase Elastic offre une période flexible entre le moment où les ressources sont potentiellement instables et le moment où elles doivent maintenant un état stable et sain.
    1. Lorsque la phase Elastic débute, Azure Deployment Manager commence à interroger périodiquement l’intégrité du service via le point de terminaison REST fourni. L’intervalle d’interrogation est configurable.
    1. Si l’analyseur d’intégrité revient avec des signaux indiquant que le service n’est pas sain, ces signaux sont ignorés, la phase Elastic se poursuit et l’interrogation continue.
    1. Dès que l’analyseur d’intégrité revient avec des signaux indiquant que le service est sain, la phase Elastic prend fin et la phase HealthyState débute.
    1. Ainsi, la durée spécifiée pour la phase Elastic représente la durée maximale qui peut être passée à interroger l’intégrité du service avant qu’une réponse « sain » soit considérée obligatoire.
1. HealthyState

    1. Pendant la phase HealthyState, l’intégrité du service est interrogé en permanence à la même fréquence que la phase Elastique.
    1. Le service maintient des signaux sains depuis le fournisseur d’analyse d’intégrité pour toute la durée spécifiée.
    1. Si, à tout moment, une réponse « non sain » est détectée, Azure Deployment Manager interrompt l’ensemble du lancement et retourne la réponse REST contenant les signaux du service non sain.
    1. Une fois que la période HealthyState terminée, la phase healthCheck est terminée, et le déploiement passe à l’étape suivante.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à intégrer la supervision de l’intégrité à Azure Deployment Manager. Passez à l’article suivant pour apprendre à déployer avec Deployment Manager.

> [!div class="nextstepaction"]
> [Tutoriel : intégrer la supervision de l’intégrité à Azure Deployment Manager](./deployment-manager-tutorial-health-check.md)