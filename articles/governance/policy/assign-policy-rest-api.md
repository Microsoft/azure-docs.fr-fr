---
title: 'Démarrage rapide : Nouvelle attribution de stratégie avec une API REST'
description: Dans ce démarrage rapide, vous allez utiliser une API REST pour créer une attribution Azure Policy afin d’identifier des ressources non conformes.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: ab05079c5bb319f0808a743a1d668649df51b1b3
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92074003"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-with-rest-api"></a>Démarrage rapide : Créer une attribution de stratégie pour identifier des ressources non conformes avec une API REST

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources.
Ce démarrage rapide vous guide pas à pas dans le processus de création d’une attribution de stratégie pour identifier les machines virtuelles qui n’utilisent pas de disques managés.

À la fin de ce processus, vous aurez identifié correctement les machines virtuelles qui n’utilisent pas de disques managés. Elles sont _non conformes_ à l’attribution de stratégie.

Une API REST est utilisée pour créer et gérer des ressources Azure. Ce guide utilise une API REST pour créer une attribution de stratégie et identifier des ressources non conformes dans votre environnement Azure.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- Si ce n’est pas déjà fait, installez [ARMClient](https://github.com/projectkudu/ARMClient). Il s’agit d’un outil qui envoie des requêtes HTTP aux API REST basées sur Azure Resource Manager. Vous pouvez également utiliser la fonctionnalité « Essayer » dans la documentation REST ou des outils tels que [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) ou [Postman](https://www.postman.com) de PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-policy-assignment"></a>Créer une affectation de stratégie

Dans ce démarrage rapide, vous allez créer une attribution de stratégie et attribuer la définition de stratégie **Auditer les machines virtuelles n’utilisant aucun disque managé.** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Cette stratégie identifie les ressources qui ne sont pas conformes aux conditions définies dans sa définition.

Exécutez la commande suivante pour créer une attribution de stratégie :

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
     ```

   - Corps de la requête

     ```json
     {
       "properties": {
         "displayName": "Audit VMs without managed disks Assignment",
         "description": "Shows all virtual machines not using managed disks",
         "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d",
       }
     }
     ```

Le point de terminaison et le corps de la demande précédents utilisent les informations suivantes :

URI DE L’API REST :
- **Scope** : une étendue détermine les ressources ou le regroupement de ressources sur lequel l’attribution de stratégie est appliquée. Cette étendue peut aller d’un groupe d’administration à une ressource individuelle. Veillez à remplacer `{scope}` par l’un des modèles suivants :
  - Groupe d’administration : `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Abonnement : `/subscriptions/{subscriptionId}`
  - Groupe de ressources : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`
  - Ressource : `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]{resourceType}/{resourceName}`
- **Name** : nom réel de l’attribution. Pour cet exemple, _audit-vm-manageddisks_ a été utilisé.

Corps de la demande :
- **DisplayName** : nom d’affichage pour l’attribution de stratégie. Dans ce cas, nous allons utiliser l’affectation _Audit VMs without managed disks_ (Auditer les machines virtuelles sans disques managés).
- **Description**  : explication plus détaillée de ce que fait la stratégie ou de la raison pour laquelle elle est attribuée à cette étendue.
- **policyDefinitionId** : ID de définition de stratégie, sur lequel vous vous basez pour créer l’attribution. Dans ce cas, il s’agit de l’ID de la définition de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_ .

## <a name="identify-non-compliant-resources"></a>Identifier des ressources non conformes

Pour afficher les ressources non conformes en vertu de cette attribution, exécutez la commande suivante pour obtenir les ID des ressources non conformes générées dans un fichier JSON :

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$filter=IsCompliant eq false and PolicyAssignmentId eq 'audit-vm-manageddisks'&$apply=groupby((ResourceId))"
```

Vos résultats doivent ressembler à l’exemple suivant :

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 3,
    "value": [{
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachineId>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionId>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine2Id>"
        },
        {
            "@odata.id": null,
            "@odata.context": "https://management.azure.com/subscriptions/<subscriptionId>/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
            "ResourceId": "/subscriptions/<subscriptionName>/resourcegroups/<rgname>/providers/microsoft.compute/virtualmachines/<virtualmachine3Id>"
        }

    ]
}
```

Les résultats sont comparables à ce que vous devriez généralement voir sous **Ressources non conformes** dans la vue du portail Azure.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez la commande suivante pour supprimer l’affectation créée :

```http
DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/audit-vm-manageddisks?api-version=2019-09-01
```

Remplacez `{scope}` par l’étendue que vous avez utilisée lors de la création initiale de l’attribution de stratégie.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie pour identifier les ressources non conformes de votre environnement Azure.

Pour en savoir plus sur l’affectation de stratégies visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)
