---
title: Erreurs liées à des ressources introuvables
description: Explique comment résoudre les erreurs liées à une ressource introuvable. L’erreur peut survenir pendant le déploiement d’un modèle Azure Resource Manager ou au cours d’opérations de gestion.
ms.topic: troubleshooting
ms.date: 06/10/2020
ms.openlocfilehash: 224af4ce0fe5053201f25d8207f4ca8cdc73e638
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84667945"
---
# <a name="resolve-resource-not-found-errors"></a>Résoudre les erreurs liées à des ressources introuvables

Cet article décrit l’erreur que vous rencontrez quand une ressource est introuvable au cours d’une opération. En règle générale, vous obtenez cette erreur pendant le déploiement de ressources. Elle peut aussi survenir quand vous effectuez des tâches de gestion et que Azure Resource Manager ne trouve pas la ressource nécessaire. Par exemple, si vous essayez d’ajouter des étiquettes à une ressource qui n’existe pas, vous obtenez cette erreur.

## <a name="symptom"></a>Symptôme

Deux codes d’erreur indiquent que la ressource est introuvable. L’erreur **NotFound** retourne un résultat qui se présente comme ceci :

```
Code=NotFound;
Message=Cannot find ServerFarm with name exampleplan.
```

L’erreur **ResourceNotFound** retourne un résultat qui se présente comme ceci :

```
Code=ResourceNotFound;
Message=The Resource 'Microsoft.Storage/storageAccounts/{storage name}' under resource
group {resource group name} was not found.
```

## <a name="cause"></a>Cause

Resource Manager a besoin de récupérer les propriétés d’une ressource, mais ne trouve pas la ressource dans vos abonnements.

## <a name="solution-1---check-resource-properties"></a>Solution 1 : Vérifier les propriétés de la ressource

Quand vous obtenez cette erreur pendant que vous effectuez une tâche de gestion, vérifiez les valeurs que vous spécifiez pour la ressource. Les trois valeurs à vérifier sont les suivantes :

* Nom de la ressource
* Nom de groupe ressources
* Abonnement

Si vous utilisez PowerShell ou Azure CLI, vérifiez que vous exécutez bien la commande dans l’abonnement qui contient la ressource. Vous pouvez changer d’abonnement avec [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext) ou [az account set](/cli/azure/account#az-account-set). De nombreuses commandes proposent également un paramètre d’abonnement qui vous permet de spécifier un abonnement différent du contexte actuel.

Si vous ne parvenez pas à vérifier les propriétés, connectez-vous au [portail](https://portal.azure.com). Recherchez la ressource que vous essayez d’utiliser et examinez le nom de la ressource, le groupe de la ressource et l’abonnement.

## <a name="solution-2---set-dependencies"></a>Solution 2 : Définir des dépendances

Si vous obtenez cette erreur pendant le déploiement d’un modèle, il peut être nécessaire d’ajouter une dépendance. Resource Manager optimise le déploiement en créant des ressources en parallèle, quand cela est possible. Si une ressource doit être déployée après une autre ressource, vous devez utiliser l’élément **dependsOn** de votre modèle. Par exemple, quand vous déployez une application web, le plan App Service doit être présent. Si vous n’avez pas spécifié que l’application web est dépendante du plan App Service, Resource Manager crée les deux ressources en même temps. Vous obtenez une erreur indiquant que la ressource du plan App Service est introuvable, car elle n’existe pas encore au moment où vous tentez de définir une propriété dans l’application web. Vous pouvez éviter cette erreur en définissant la dépendance dans l’application web.

```json
{
  "type": "Microsoft.Web/sites",
  "apiVersion": "2015-08-01",
  "dependsOn": [
    "[variables('hostingPlanName')]"
  ],
  ...
}
```

Il vaut mieux éviter de définir des dépendances qui ne sont pas nécessaires. Lorsque vous avez des dépendances inutiles, vous prolongez la durée du déploiement en empêchant les ressources qui ne sont pas interdépendantes d’être déployées en parallèle. Par ailleurs, il se peut que créiez des dépendances circulaires qui bloquent le déploiement. La fonction [reference](template-functions-resource.md#reference) et les fonctions [list*](template-functions-resource.md#list) créent une dépendance implicite envers la ressource référencée, lorsque cette ressource est déployée dans le même modèle et référencée par son nom (et non l’ID de la ressource). Il se peut donc que vous ayez des dépendances en plus des dépendances spécifiées dans la propriété **dependsOn**. La fonction [resourceId](template-functions-resource.md#resourceid) ne crée pas de dépendance implicite pas plus qu’elle ne valide l’existence de la ressource. La fonction [reference](template-functions-resource.md#reference) et les fonctions [list*](template-functions-resource.md#list) ne créent pas de dépendance implicite lorsque la ressource est référencée par son ID. Pour créer une dépendance implicite, transmettez le nom de la ressource qui est déployée dans le même modèle.

Si vous observez des problèmes de dépendance, vous devez déterminer l’ordre de déploiement des ressources. Pour afficher l’ordre des opérations de déploiement :

1. Sélectionnez l’historique de déploiement de votre groupe de ressources.

   ![sélectionner l’historique de déploiement](./media/error-not-found/select-deployment.png)

2. Sélectionnez un déploiement dans l’historique, puis sélectionnez **Événements**.

   ![sélectionner les événements de déploiement](./media/error-not-found/select-deployment-events.png)

3. Examinez la séquence d’événements de chaque ressource. Faites attention à l’état de chaque opération. Par exemple, l’image suivante montre trois comptes de stockage déployés en parallèle. Remarquez que les trois comptes de stockage sont démarrés en même temps.

   ![déploiement en parallèle](./media/error-not-found/deployment-events-parallel.png)

   L’image suivante montre trois comptes de stockage non déployés en parallèle. Le deuxième compte de stockage dépend du premier compte de stockage, et le troisième du deuxième. Le premier compte de stockage est démarré, accepté et terminé avant que le suivant ne démarre.

   ![déploiement séquentiel](./media/error-not-found/deployment-events-sequence.png)

## <a name="solution-3---get-external-resource"></a>Solution 3 : Obtenir une ressource externe

Quand vous déployez un modèle et que vous devez obtenir une ressource qui existe dans un abonnement ou un groupe de ressources différent, utilisez la [fonction resourceId](template-functions-resource.md#resourceid). Cette fonction retourne le nom complet de la ressource.

Les paramètres d’abonnement et de groupe de ressources de la fonction resourceId sont facultatifs. Si vous ne les spécifiez pas, ils indiquent par défaut l’abonnement et le groupe de ressources actuels. Quand vous utilisez une ressource située dans un groupe de ressources ou un abonnement différent, veillez à fournir ces valeurs.

L’exemple suivant obtient l’ID d’une ressource qui appartient à un groupe de ressources différent.

```json
"properties": {
  "name": "[parameters('siteName')]",
  "serverFarmId": "[resourceId('plangroup', 'Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
}
```

## <a name="solution-4---get-managed-identity-from-resource"></a>Solution 4 : Obtenir une identité managée à partir d'une ressource

Si vous déployez une ressource qui crée implicitement une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md), vous devez attendre que cette ressource soit déployée avant de récupérer des valeurs sur l'identité managée. Si vous transmettez le nom de l'identité managée à la fonction [reference](template-functions-resource.md#reference), Resource Manager tente de résoudre la référence avant que la ressource et l'identité ne soient déployées. Transmettez plutôt le nom de la ressource à laquelle l'identité est appliquée. Cette approche garantit que la ressource et l'identité gérée seront déployées avant que Resource Manager ne résolve la fonction reference.

Dans la fonction reference, utilisez `Full` pour obtenir toutes les propriétés, y compris l'identité managée.

Le modèle est :

`"[reference(resourceId(<resource-provider-namespace>, <resource-name>, <API-version>, 'Full').Identity.propertyName]"`

> [!IMPORTANT]
> N’utilisez pas le modèle :
>
> `"[reference(concat(resourceId(<resource-provider-namespace>, <resource-name>),'/providers/Microsoft.ManagedIdentity/Identities/default'),<API-version>).principalId]"`
>
> Votre modèle échouera.

Par exemple, pour obtenir l'ID de principal d'une identité managée appliquée à une machine virtuelle, utilisez :

```json
"[reference(resourceId('Microsoft.Compute/virtualMachines', variables('vmName')),'2019-12-01', 'Full').identity.principalId]",
```

Ou, pour obtenir l'ID de locataire d'une identité managée appliquée à un groupe de machines virtuelles identiques, utilisez :

```json
"[reference(resourceId('Microsoft.Compute/virtualMachineScaleSets',  variables('vmNodeType0Name')), 2019-12-01, 'Full').Identity.tenantId]"
```

## <a name="solution-5---check-functions"></a>Solution 5 : Vérifier les fonctions

Quand vous déployez un modèle, recherchez des expressions qui utilisent les fonctions [reference](template-functions-resource.md#reference) ou [listKeys](template-functions-resource.md#listkeys). Les valeurs que vous fournissez varient selon si la ressource fait partie des mêmes modèle, groupe de ressources et abonnement. Vérifiez que vous fournissez les valeurs de paramètres nécessaires pour votre scénario. Si la ressource se trouve dans un autre groupe de ressources, fournissez l’ID complet de la ressource. Par exemple, pour faire référence à un compte de stockage dans un autre groupe de ressources, utilisez :

```json
"[reference(resourceId('exampleResourceGroup', 'Microsoft.Storage/storageAccounts', 'myStorage'), '2017-06-01')]"
```