---
title: Mettre à jour les ressources managées
description: Décrit comment utiliser des ressources dans le groupe de ressources managé pour une application managée Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/26/2017
ms.author: tomfitz
ms.openlocfilehash: 3dbb88d53a961e277e102d7eec580309951c73b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649429"
---
# <a name="work-with-resources-in-the-managed-resource-group-for-azure-managed-application"></a>Utiliser des ressources dans le groupe de ressources managé pour une application managée Azure

Cet article décrit comment mettre à jour des ressources qui sont déployées dans le cadre d’une application managée. En tant qu’éditeur d’une application managée, vous avez accès aux ressources dans le groupe de ressources managé. Pour mettre à jour ces ressources, vous devez trouver le groupe de ressources managé associé à une application managée et accéder à la ressource souhaitée dans ce groupe de ressources.

Cet article part du principe que vous avez déployé l’application managée dans l’exemple de projet [Application web managée (IaaS) avec services de gestion Azure](https://github.com/Azure/azure-managedapp-samples/tree/master/Managed%20Application%20Sample%20Packages/201-managed-web-app). Cette application managée comprend une machine virtuelle **Standard_D1_v2**. Si vous n’avez pas déployé cette application managée, vous pouvez quand même utiliser cet article pour vous familiariser avec les étapes de mise à jour d’un groupe de ressources managé.

L’illustration suivante montre l’application managée déployée.

![Application managée déployée](./media/update-managed-resources/deployed.png)

Dans cet article, vous utilisez Azure CLI pour :

* Identifier l’application managée.
* Identifier le groupe de ressources managé.
* Identifier les ressources de machines virtuelles dans le groupe de ressources managé.
* Changer la taille des machines virtuelles (la réduire si vous ne les utilisez pas, ou l’augmenter pour prendre en charge une charge plus élevée).
* Affecter une stratégie au groupe de ressources managé qui spécifie les emplacements autorisés.

## <a name="get-managed-application-and-managed-resource-group"></a>Obtenir une application managée et un groupe de ressources managé

Pour obtenir les applications managées dans un groupe de ressources, utilisez :

```azurecli-interactive
az managedapp list --query "[?contains(resourceGroup,'DemoApp')]"
```

Pour obtenir l’ID du groupe de ressources managé, utilisez :

```azurecli-interactive
az managedapp list --query "[?contains(resourceGroup,'DemoApp')].{ managedResourceGroup:managedResourceGroupId }"
```

## <a name="resize-vms-in-managed-resource-group"></a>Redimensionner des machines virtuelles dans le groupe de ressources managé

Pour afficher les machines virtuelles dans le groupe de ressources managé, spécifiez le nom du groupe de ressources managé.

```azurecli-interactive
az vm list -g DemoApp6zkevchqk7sfq --query "[].{VMName:name,OSType:storageProfile.osDisk.osType,VMSize:hardwareProfile.vmSize}"
```

Pour mettre à jour la taille des machines virtuelles, utilisez :

```azurecli-interactive
az vm resize --size Standard_D2_v2 --ids $(az vm list -g DemoApp6zkevchqk7sfq --query "[].id" -o tsv)
```

Une fois l’opération terminée, vérifiez que l’application s’exécute sur Standard D2 v2.

![Application managée utilisant Standard D2 v2](./media/update-managed-resources/upgraded.png)

## <a name="apply-policy-to-managed-resource-group"></a>Appliquer une stratégie au groupe de ressources managé

Obtenez le groupe de ressources managé et attribuez une stratégie dans cette portée. La stratégie **e56962a6-4747-49cd-b67b-bf8b01975c4c** est une stratégie intégrée permettant de spécifier des emplacements autorisés.

```azurecli-interactive
managedGroup=$(az managedapp show --name <app-name> --resource-group DemoApp --query managedResourceGroupId --output tsv)

az policy assignment create --name locationAssignment --policy e56962a6-4747-49cd-b67b-bf8b01975c4c --scope $managedGroup --params '{
                            "listofallowedLocations": {
                                "value": [
                                    "northeurope",
                                    "westeurope"
                                ]
                            }
                        }'
```

Pour afficher les emplacements autorisés, utilisez :

```azurecli-interactive
az policy assignment show --name locationAssignment --scope $managedGroup --query parameters.listofallowedLocations.value
```

L’attribution de stratégie s’affiche dans le portail.

![Afficher l’attribution de stratégie](./media/update-managed-resources/assignment.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation des applications gérées, consultez [Vue d’ensemble des applications gérées](overview.md).
* Pour voir des exemples de projets, consultez [Exemples de projets pour des applications managées Azure](sample-projects.md).
