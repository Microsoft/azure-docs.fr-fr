---
title: Déploiement sécurisé entre les régions - Azure Deployment Manager
description: Découvrez comment déployer un service sur plusieurs régions avec Azure Deployment Manager et découvrez des pratiques de déploiement sécurisées.
ms.topic: conceptual
ms.date: 11/21/2019
ms.custom: seodec18
ms.openlocfilehash: 8b950fdc36fe3fbea1ce9436bdd7f7372c64c055
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333203"
---
# <a name="enable-safe-deployment-practices-with-azure-deployment-manager-public-preview"></a>Permettre des pratiques sûres de déploiement avec Azure Deployment Manager (préversion publique)

Afin de déployer votre service dans de nombreuses régions et vous assurer qu’il s’exécute comme prévu dans chaque région, vous pouvez utiliser Azure Deployment Manager et coordonner un lancement par étapes du service. Tout comme vous le feriez pour n’importe quel déploiement Azure, vous définissez les ressources de votre service dans les [modèles Resource Manager](template-syntax.md). Une fois les modèles créés, vous utilisez Deployment Manager pour décrire la topologie de votre service, et la façon dont il doit être lancé.

Deployment Manager est une fonction de Resource Manager. Elle étend vos fonctionnalités au cours du déploiement. Utilisez Deployment Manager lorsque vous avez un service complexe à déployer sur plusieurs régions. Avec un déploiement intermédiaire de votre service, vous pouvez rechercher des problèmes potentiels avant le déploiement dans toutes les régions. Si les précautions supplémentaires apportées par un lancement par étapes ne vous sont pas utiles, préférez les [options de déploiement](deploy-portal.md) standard pour Resource Manager. Deployment Manager s’intègre parfaitement à tous les outils tiers existants qui prennent en charge des déploiements Resource Manager, telles que les offres d’intégration continue et de livraison continue (CI/CD).

Azure Deployment Manager est en préversion. Aidez-nous à améliorer cette fonctionnalité en apportant des [commentaires](https://aka.ms/admfeedback).

Pour utiliser Deployment Manager, vous devez créer quatre fichiers :

* Modèle de topologie
* Modèle de lancement
* Fichier de paramètres pour la topologie
* Fichier de paramètres pour le lancement

Vous déployez le modèle de topologie avant de déployer le modèle de lancement.

Ressources supplémentaires :

- Les [informations de référence de l’API REST Azure Deployment Manager](/rest/api/deploymentmanager/).
- [Tutoriel : Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md).
- [Tutoriel : Utiliser le contrôle d’intégrité dans Azure Deployment Manager](./deployment-manager-tutorial-health-check.md).
- [Un exemple d’Azure Deployment Manager](https://github.com/Azure-Samples/adm-quickstart).

## <a name="identity-and-access"></a>Identité et accès

Avec Deployment Manager, une [identité managée affectée à l’utilisateur](../../active-directory/managed-identities-azure-resources/overview.md) effectue les actions de déploiement. Vous créez cette identité avant de commencer votre déploiement. Elle doit avoir accès à l’abonnement Azure sur lequel vous déployez le service, et disposer des autorisations suffisantes pour procéder au déploiement complet. Pour plus d’informations sur les actions accordées par le biais des rôles, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

L’identité doit résider dans le même emplacement que le déploiement.

## <a name="topology-template"></a>Modèle de topologie

Le modèle de topologie décrit les ressources Azure constituant votre service, et l’emplacement où les déployer. Le schéma suivant illustre cette topologie dans un exemple de service :

![Hiérarchie, de la topologie de service aux services et unités de service](./media/deployment-manager-overview/service-topology.png)

Le modèle de topologie comporte les ressources suivantes :

* Source d’artefact - Emplacement où sont stockés vos modèles et paramètres Resource Manager
* Topologie de service - Pointe vers la source d’artefact
  * Services - Spécifie l’emplacement et l’ID d’abonnement Azure
    * Unités de service - Spécifie le groupe de ressources, le mode de déploiement et le chemin au modèle et au fichier de paramètres

Pour comprendre ce qui se passe à chaque niveau, il est utile de voir quelles valeurs vous fournissez.

![Valeurs de chaque niveau](./media/deployment-manager-overview/topology-values.png)

### <a name="artifact-source-for-templates"></a>Source d’artefact pour les modèles

Dans votre modèle de topologie, vous créez une source d’artefact contenant les fichiers de paramètres et les modèles. La source d’artefact constitue un moyen d’extraction des fichiers pour le déploiement. Une autre source d’artefact est présentée pour les fichiers binaires plus loin dans cet article.

L’exemple suivant montre le format général de la source d’artefact.

```json
{
  "type": "Microsoft.DeploymentManager/artifactSources",
  "apiVersion": "2018-09-01-preview",
  "name": "<artifact-source-name>",
  "location": "<artifact-source-location>",
  "properties": {
    "sourceType": "AzureStorage",
    "artifactRoot": "<root-folder-for-templates>",
    "authentication": {
      "type": "SAS",
      "properties": {
        "sasUri": "<SAS-URI-for-storage-container>"
      }
    }
  }
}
```

Pour plus d’informations, consultez [Informations de référence sur le modèle artifactSources](/azure/templates/Microsoft.DeploymentManager/artifactSources).

### <a name="service-topology"></a>Topologie de service

L’exemple suivant montre le format général de la ressource de topologie de service. Vous fournissez l’ID de ressource de la source d’artefact qui contient les fichiers de paramètres et les modèles. La topologie de service comprend toutes les ressources de service. Pour vous assurer que la source d’artefact est disponible, la topologie de service en dépend.

```json
{
  "type": "Microsoft.DeploymentManager/serviceTopologies",
  "apiVersion": "2018-09-01-preview",
  "name": "<topology-name>",
  "location": "<topology-location>",
  "dependsOn": [
    "<artifact-source>"
  ],
  "properties": {
    "artifactSourceId": "<resource-ID-artifact-source>"
  },
  "resources": [
    {
      "type": "services",
        ...
        }
    ]
}
```

Pour plus d’informations, consultez [Informations de référence sur le modèle serviceTopologies](/azure/templates/Microsoft.DeploymentManager/serviceTopologies).

### <a name="services"></a>Services

L’exemple suivant montre le format général de la ressource des services. Dans chaque service, vous fournissez l’emplacement et l’ID d’abonnement Azure à utiliser pour le déploiement de votre service. Pour déployer sur plusieurs régions, vous définissez un service pour chaque région. Le service dépend de la topologie de service.

```json
{
  "type": "services",
  "apiVersion": "2018-09-01-preview",
  "name": "<service-name>",
  "location": "<service-location>",
  "dependsOn": [
      "<service-topology>"
  ],
  "properties": {
    "targetSubscriptionId": "<subscription-ID>",
    "targetLocation": "<location-of-deployed-service>"
  },
  "resources": [
    {
      "type": "serviceUnits",
            ...
        }
    ]
}
```

Pour plus d’informations, consultez [Informations de référence sur le modèle services](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services).

### <a name="service-units"></a>Unités de service

L’exemple suivant montre le format général de la ressource des unités de service. Dans chaque unité de service, vous spécifiez le groupe de ressources, le [mode de déploiement](deployment-modes.md) à utiliser pour le déploiement et le chemin au modèle et au fichier de paramètres. Si vous spécifiez un chemin relatif pour le modèle et les paramètres, le chemin complet est construit à partir du dossier racine dans la source d’artefacts. Vous pouvez spécifier un chemin absolu pour le modèle et les paramètres, mais vous perdez la possibilité de générer facilement la version de vos mises en production. L’unité de service dépend du service.

```json
{
  "type": "serviceUnits",
  "apiVersion": "2018-09-01-preview",
  "name": "<service-unit-name>",
  "location": "<service-unit-location>",
  "dependsOn": [
    "<service>"
  ],
  "tags": {
    "serviceType": "Service West US Web App"
  },
  "properties": {
    "targetResourceGroup": "<resource-group-name>",
    "deploymentMode": "Incremental",
    "artifacts": {
      "templateArtifactSourceRelativePath": "<relative-path-to-template>",
      "parametersArtifactSourceRelativePath": "<relative-path-to-parameter-file>"
    }
  }
}
```

Chaque modèle doit inclure les ressources associées que vous souhaitez déployer en une seule étape. Par exemple, une unité de service peut disposer d’un modèle qui déploie toutes les ressources du serveur frontal de votre service.

Pour plus d’informations, consultez [Informations de référence sur le modèle serviceUnits](/azure/templates/Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits).

## <a name="rollout-template"></a>Modèle de lancement

Le modèle de lancement décrit les étapes à suivre lors du déploiement de votre service. Vous spécifiez la topologie de service à utiliser et définissez l’ordre de déploiement des unités de service. Il comprend une source d’artefact pour le stockage des fichiers binaires du déploiement. Dans votre modèle de lancement, vous définissez la hiérarchie suivante :

* Source d’artefact
* Étape
* Lancement
  * Groupes d’étape
    * Opérations de déploiement

L’illustration suivante détaille la hiérarchie du modèle de lancement :

![Hiérarchie, du lancement aux étapes](./media/deployment-manager-overview/Rollout.png)

Chaque lancement peut contenir plusieurs groupes d’étape. Chaque groupe d’étape contient une opération de déploiement qui pointe vers une unité de service dans la topologie de service.

### <a name="artifact-source-for-binaries"></a>Source d’artefact pour les fichiers binaires

Dans le modèle de lancement, vous créez une source d’artefact pour les fichiers binaires que vous devez déployer sur le service. Cette source d’artefact est similaire à la [source d’artefact des modèles](#artifact-source-for-templates), à ceci près qu’elle contient les scripts, les pages web, le code compilé ou encore d’autres fichiers nécessaires à votre service.

### <a name="steps"></a>Étapes

Vous pouvez définir une étape à effectuer avant ou après votre opération de déploiement. Actuellement, seules les étapes `wait` et « healthCheck » sont disponibles.

Cette étape d’attente suspend le déploiement avant de poursuivre. Elle vous permet de vérifier que votre service s’exécute comme prévu avant de déployer l’unité de service suivante. L’exemple suivant montre le format général d’une étape d’attente.

```json
{
    "type": "Microsoft.DeploymentManager/steps",
    "apiVersion": "2018-09-01-preview",
    "name": "waitStep",
    "location": "<step-location>",
    "properties": {
        "stepType": "wait",
        "attributes": {
          "duration": "PT1M"
        }
    }
},
```

La propriété duration utilise la [norme ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations). L’exemple précédent indique une attente d’une minute.

Pour plus d’informations sur l’étape de contrôle d’intégrité, consultez [Introduire le lancement de l’intégration de l’intégrité dans Azure Deployment Manager](./deployment-manager-health-check.md) et [Tutoriel : Utiliser le contrôle d’intégrité dans Azure Deployment Manager](./deployment-manager-tutorial-health-check.md).

Pour plus d’informations, consultez [Informations de référence sur le modèle des étapes](/azure/templates/Microsoft.DeploymentManager/steps).

### <a name="rollouts"></a>Lancements

Pour vous assurer que la source d’artefact est disponible, le lancement en est tributaire. Le lancement définit les groupes d’étape de chaque unité de service déployée. Vous pouvez définir les actions à entreprendre avant ou après le déploiement. Par exemple, vous pouvez spécifier que le déploiement attende la fin du déploiement de l’unité de service. Vous pouvez définir l’ordre des groupes d’étape.

L’objet d’identité spécifie l’[identité managée affectée à l’utilisateur](#identity-and-access) qui effectue les actions de déploiement.

L’exemple suivant montre le format général du lancement.

```json
{
  "type": "Microsoft.DeploymentManager/rollouts",
  "apiVersion": "2018-09-01-preview",
  "name": "<rollout-name>",
  "location": "<rollout-location>",
  "Identity": {
    "type": "userAssigned",
    "identityIds": [
      "<managed-identity-ID>"
    ]
  },
  "dependsOn": [
    "<artifact-source>"
  ],
  "properties": {
    "buildVersion": "1.0.0.0",
    "artifactSourceId": "<artifact-source-ID>",
    "targetServiceTopologyId": "<service-topology-ID>",
    "stepGroups": [
      {
        "name": "stepGroup1",
        "dependsOnStepGroups": ["<step-group-name>"],
        "preDeploymentSteps": ["<step-ID>"],
        "deploymentTargetId":
          "<service-unit-ID>",
        "postDeploymentSteps": ["<step-ID>"]
      },
            ...
        ]
    }
}
```

Pour plus d’informations, consultez [Informations de référence sur les lancements](/azure/templates/Microsoft.DeploymentManager/rollouts).

## <a name="parameter-file"></a>Fichier de paramètres

Vous créez deux fichiers de paramètres. L’un des fichiers de paramètres est utilisé au cours du déploiement de la topologie de service, l’autre sert au déploiement du lancement. Il existe certaines valeurs dont vous devez vous assurer qu’elles soient les mêmes dans les deux fichiers de paramètres.

## <a name="containerroot-variable"></a>Variable containerRoot

Avec les déploiements par version, le chemin à vos artefacts change avec chaque nouvelle version. La première fois que vous exécutez un déploiement, le chemin peut être `https://<base-uri-blob-container>/binaries/1.0.0.0`. La deuxième fois, il peut être `https://<base-uri-blob-container>/binaries/1.0.0.1`. Deployment Manager simplifie l’obtention du chemin racine correct pour le déploiement actif à l’aide de la variable `$containerRoot`. Cette valeur change avec chaque version et elle n’est pas connue avant le déploiement.

Utilisez la variable `$containerRoot` dans le fichier de paramètres pour que le modèle déploie les ressources Azure. Au moment du déploiement, cette variable est remplacée par les valeurs réelles au lancement.

Par exemple, au cours du lancement, vous créez une source d’artefact pour les artefacts binaires.

```json
{
  "type": "Microsoft.DeploymentManager/artifactSources",
  "apiVersion": "2018-09-01-preview",
  "name": "[variables('rolloutArtifactSource').name]",
  "location": "[parameters('azureResourceLocation')]",
  "properties": {
    "sourceType": "AzureStorage",
      "artifactRoot": "[parameters('binaryArtifactRoot')]",
      "authentication" :
        {
          "type": "SAS",
          "properties": {
            "sasUri": "[parameters('artifactSourceSASLocation')]"
          }
        }
  }
},
```

Remarquez les propriétés `artifactRoot` et `sasUri`. La racine de l’artefact peut être définie avec une valeur telle que `binaries/1.0.0.0`. L’URI SAS est l’URI menant à votre conteneur de stockage, avec un jeton SAS pour l’accès. Deployment Manager construit automatiquement la valeur de la variable `$containerRoot`. Il combine ces valeurs dans le format `<container>/<artifactRoot>`.

Votre modèle et votre fichier de paramètre doivent connaître le chemin correct pour l’obtention des fichiers binaires par version. Par exemple, pour déployer les fichiers d’une application web, créez le fichier de paramètres suivant avec la variable $containerRoot. Vous devez utiliser deux barres obliques inverses (`\\`) pour le chemin, car la première barre est un caractère d’échappement.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "deployPackageUri": {
      "value": "$containerRoot\\helloWorldWebAppWUS.zip"
    }
  }
}
```

Utilisez ensuite ce paramètre dans votre modèle :

```json
{
  "name": "MSDeploy",
  "apiVersion": "2015-08-01",
  "type": "extensions",
  "location": "[parameters('location')]",
  "dependsOn": [
    "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
  ],
  "tags": {
    "displayName": "WebAppMSDeploy"
  },
  "properties": {
    "packageUri": "[parameters('deployPackageURI')]"
  }
}
```

Vous gérez les déploiements avec version en créant des dossiers et en transmettant cette racine pendant le lancement. Le chemin transite jusqu’au modèle qui déploie les ressources.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert Deployment Manager. Passez à l’article suivant pour apprendre à déployer avec Deployment Manager.

> [!div class="nextstepaction"]
> [Tutoriel : Utiliser Azure Deployment Manager avec des modèles Resource Manager](./deployment-manager-tutorial.md)
>
> [Démarrage rapide : Essayez Azure Deployment Manager en seulement quelques minutes](https://github.com/Azure-Samples/adm-quickstart)
