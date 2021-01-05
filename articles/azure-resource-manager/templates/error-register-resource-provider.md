---
title: Erreurs d’inscription du fournisseur de ressources
description: Décrit comment résoudre les erreurs d’inscription de fournisseurs de ressources Azure quand vous déployez des ressources avec Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 02/15/2019
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 143cf03a33739f43a29af94fc2f8a336fb3aef8f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89076665"
---
# <a name="resolve-errors-for-resource-provider-registration"></a>Résoudre les erreurs d’inscription de fournisseurs de ressources

Cet article décrit les erreurs que vous pouvez rencontrer quand vous utilisez un fournisseur de ressources que vous n’avez pas jamais utilisé auparavant dans votre abonnement.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="symptom"></a>Symptôme

Lorsque vous déployez une ressource, vous pouvez recevoir le message et le code d’erreur suivants :

```
Code: NoRegisteredProviderFound
Message: No registered resource provider found for location {location}
and API version {api-version} for type {resource-type}.
```

Vous pouvez sinon recevoir un message similaire indiquant :

```
Code: MissingSubscriptionRegistration
Message: The subscription is not registered to use namespace {resource-provider-namespace}
```

Le message d’erreur doit fournir des suggestions pour les emplacements et les versions d’API pris en charge. Vous pouvez changer votre modèle en l’une des valeurs suggérées. La plupart des fournisseurs sont inscrits automatiquement par le portail Azure ou l’interface de ligne de commande que vous utilisez, mais pas tous. Si vous n’avez pas déjà utilisé un certain fournisseur de ressources, vous devrez peut-être l’inscrire.

Ou, lors de la désactivation de l’arrêt automatique pour les machines virtuelles, vous pouvez recevoir un message d’erreur semblable au suivant :

```
Code: AuthorizationFailed
Message: The client '<identifier>' with object id '<identifier>' does not have authorization to perform action 'Microsoft.Compute/virtualMachines/read' over scope ...
```

## <a name="cause"></a>Cause :

Ces erreurs apparaissent pour l’une des raisons suivantes :

* Le fournisseur de ressources requis n’a pas été inscrit pour votre abonnement
* La version de l’API n’est pas prise en charge pour le type de ressource.
* L’emplacement n’est pas pris en charge pour le type de ressource.
* Pour l’arrêt automatique des machines virtuelles, le fournisseur de ressources Microsoft.DevTestLab doit être inscrit.

## <a name="solution-1---powershell"></a>Solution 1 : PowerShell

Pour PowerShell, utilisez **Get-AzResourceProvider** pour afficher l'état de votre inscription.

```powershell
Get-AzResourceProvider -ListAvailable
```

Pour inscrire un fournisseur, utilisez **Register-AzResourceProvider** et indiquez le nom du fournisseur de ressources que vous souhaitez inscrire.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Cdn
```

Pour obtenir les emplacements pris en charge d’un type particulier de ressource, utilisez la commande suivante :

```powershell
((Get-AzResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
```

Pour obtenir les versions d’API prises en charge d’un type particulier de ressource, utilisez la commande suivante :

```powershell
((Get-AzResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
```

## <a name="solution-2---azure-cli"></a>Solution 2 : Azure CLI

Pour voir si le fournisseur est inscrit, utilisez la commande `az provider list` .

```azurecli-interactive
az provider list
```

Pour inscrire un fournisseur de ressources, utilisez la commande `az provider register` et indiquez *l’espace de noms* à inscrire.

```azurecli-interactive
az provider register --namespace Microsoft.Cdn
```

Pour afficher les emplacements et les versions d’API pris en charge pour un type de ressources, utilisez la commande suivante :

```azurecli-interactive
az provider show -n Microsoft.Web --query "resourceTypes[?resourceType=='sites'].locations"
```

## <a name="solution-3---azure-portal"></a>Solution 3 : Portail Azure

Vous pouvez voir l’état de l’inscription et inscrire un espace de noms de fournisseur de ressources par le biais du portail.

1. À partir du portail Azure, sélectionnez **Tous les services**.

   ![Sélectionner Tous les services](./media/error-register-resource-provider/select-all-services.png)

1. Sélectionnez **Abonnements**.

   ![Sélectionner des abonnements](./media/error-register-resource-provider/select-subscriptions.png)

1. Dans la liste des abonnements, sélectionnez l’abonnement que vous souhaitez utiliser pour inscrire le fournisseur de ressources.

   ![Sélectionner l’abonnement pour inscrire le fournisseur de ressources](./media/error-register-resource-provider/select-subscription-to-register.png)

1. Sur votre abonnement, sélectionnez **Fournisseurs de ressources**.

   ![Sélectionner les fournisseurs de ressources](./media/error-register-resource-provider/select-resource-provider.png)

1. Passez en revue la liste des fournisseurs de ressources et, si nécessaire, sélectionnez le lien **Inscrire** pour inscrire le fournisseur de ressources du type que vous voulez déployer.

   ![Répertorier les fournisseurs de ressources](./media/error-register-resource-provider/list-resource-providers.png)
