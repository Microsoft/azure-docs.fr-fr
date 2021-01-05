---
title: Fournisseurs et types de ressources
description: Décrit les fournisseurs de ressources qui prennent en charge Azure Resource Manager. Il décrit leurs schémas, les versions d’API disponibles et les régions qui peuvent héberger les ressources.
ms.topic: conceptual
ms.date: 12/04/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6d114fdfae12dd9ee96a23e4dafc3847c6429d0c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745112"
---
# <a name="azure-resource-providers-and-types"></a>Fournisseurs et types de ressources Azure

Lorsque vous déployez des ressources, vous devez fréquemment récupérer des informations sur les fournisseurs et les types de ressources. Par exemple, si vous voulez stocker des clés et des secrets, vous utilisez le fournisseur de ressources Microsoft.KeyVault. Ce fournisseur de ressources offre un type de ressource appelé vaults pour créer le coffre de clés.

Le nom d’un type de ressource est au format : **{fournisseur de ressources}/{type de ressource}**. Le type de ressource pour un coffre de clés est **Microsoft.KeyVault/vaults**.

Dans cet article, vous apprendrez comment :

* Afficher tous les fournisseurs de ressources dans Azure
* Vérifier l’état de l’inscription d’un fournisseur de ressources
* Inscrire un fournisseur de ressources
* Afficher les types de ressources pour un fournisseur de ressources
* Afficher les emplacements valides pour un type de ressource
* Afficher les versions d’API valides pour un type de ressource

Vous pouvez effectuer ces étapes via le portail Azure, Azure PowerShell ou Azure CLI.

Pour obtenir la liste qui mappe les fournisseurs de ressources aux services Azure, consultez [Fournisseurs de ressources pour les services Azure](azure-services-resource-providers.md).

## <a name="register-resource-provider"></a>S’inscrire auprès du fournisseur de ressources

Pour pouvoir utiliser un fournisseur de ressources, votre abonnement Azure doit être inscrit pour le fournisseur de ressources. L’inscription permet de configurer votre abonnement pour qu’il fonctionne avec le fournisseur de ressources. Certains fournisseurs de ressources sont inscrits par défaut. D’autres fournisseurs de ressources sont inscrits automatiquement lorsque vous effectuez certaines actions. Par exemple, lorsque vous créez une ressource via le portail, le fournisseur de ressources est généralement inscrit pour vous. Pour d’autres scénarios, il se peut que vous deviez inscrire manuellement un fournisseur de ressources. Pour obtenir la liste des fournisseurs de ressources inscrits par défaut, consultez [Fournisseurs de ressources pour les services Azure](azure-services-resource-providers.md).

Cet article vous montre comment vérifier l’état d’inscription d’un fournisseur de ressources et comment l’inscrire si nécessaire. Vous devez être autorisé à effectuer l’opération `/register/action` pour le fournisseur de ressources. Cette autorisation est incluse dans les rôles Contributeur et Propriétaire.

> [!IMPORTANT]
> Inscrivez un fournisseur de ressources uniquement au moment où vous êtes prêt à l’utiliser. L’étape d’inscription vous permet de conserver les privilèges minimaux dans votre abonnement. Un utilisateur malveillant ne peut pas utiliser de fournisseurs de ressources qui ne sont pas inscrits.

Votre code d’application ne doit pas bloquer la création de ressources pour un fournisseur de ressources qui est **en cours d’inscription**. Lorsque vous inscrivez le fournisseur de ressources, l’opération est effectuée individuellement pour chaque région prise en charge. Pour créer des ressources dans une région, l’inscription doit uniquement être effectuée dans cette région. En ne bloquant pas le fournisseur de ressources à l’état d’inscription en cours, votre application peut poursuivre beaucoup plus tôt qu’en attendant la fin de l’inscription pour toutes les régions.

Vous ne pouvez pas annuler l’inscription d’un fournisseur de ressources quand vous avez encore des types de ressources de ce fournisseur de ressources dans votre abonnement.

## <a name="azure-portal"></a>Portail Azure

### <a name="register-resource-provider"></a>S’inscrire auprès du fournisseur de ressources

Pour afficher tous les fournisseurs de ressources et l'état d'inscription de votre abonnement :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le menu du portail Azure, recherchez **Abonnements**. Sélectionnez-le parmi les options disponibles.

   :::image type="content" source="./media/resource-providers-and-types/search-subscriptions.png" alt-text="Recherche des abonnements":::

1. Sélectionnez l’abonnement que vous souhaitez afficher.

   :::image type="content" source="./media/resource-providers-and-types/select-subscription.png" alt-text="sélectionner les abonnements":::

1. Dans le menu de gauche, sous **Paramètres**, sélectionnez **Fournisseurs de ressources**.

   :::image type="content" source="./media/resource-providers-and-types/select-resource-providers.png" alt-text="Sélection des fournisseurs de ressources":::

6. Recherchez le fournisseur de ressources que vous souhaitez inscrire, puis sélectionnez **Inscrire**. Pour conserver les privilèges minimaux dans votre abonnement, inscrivez uniquement les fournisseurs de ressources que vous êtes prêt à utiliser.

   :::image type="content" source="./media/resource-providers-and-types/register-resource-provider.png" alt-text="Inscription des fournisseurs de ressources":::

### <a name="view-resource-provider"></a>Afficher un fournisseur de ressources

Pour afficher des informations pour un fournisseur de ressources particulier :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le menu du portail Azure, sélectionnez **Tous les services**.
3. Dans la zone **Tous les services**, entrez **Explorateur de ressources**, puis sélectionnez **Explorateur de ressources**.

    ![sélectionner Tous les services](./media/resource-providers-and-types/select-resource-explorer.png)

4. Développez **Fournisseurs** en sélectionnant la flèche droite.

    ![sélectionner les fournisseurs](./media/resource-providers-and-types/select-providers.png)

5. Développez le fournisseur de ressources et le type de ressource que vous souhaitez afficher.

    ![sélectionner le type de ressource](./media/resource-providers-and-types/select-resource-type.png)

6. Resource Manager est pris en charge dans toutes les régions, mais il est possible que certaines ressources que vous déployez ne soient pas prises en charge dans toutes les régions. En outre, il peut y avoir des limitations sur votre abonnement qui vous empêchent d’utiliser certaines régions prenant en charge la ressource. L’Explorateur de ressources affiche les emplacements valides pour le type de ressource.

    ![afficher les emplacements](./media/resource-providers-and-types/show-locations.png)

7. La version de l'API correspond à une version des opérations de l'API REST publiées par le fournisseur de ressources. Lorsqu'un fournisseur de ressources active de nouvelles fonctionnalités, une nouvelle version de l'API REST sera publiée. L’Explorateur de ressources affiche les versions d’API valides pour le type de ressource.

    ![afficher les versions d’API](./media/resource-providers-and-types/show-api-versions.png)

## <a name="azure-powershell"></a>Azure PowerShell

Pour afficher tous les fournisseurs de ressources dans Azure et l’état de l’inscription de votre abonnement, utilisez :

```azurepowershell-interactive
Get-AzResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
```

Qui retourne des résultats semblables à :

```output
ProviderNamespace                RegistrationState
-------------------------------- ------------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Pour afficher tous les fournisseurs de ressources inscrits pour votre abonnement, utilisez la commande suivante :

```azurepowershell-interactive
 Get-AzResourceProvider -ListAvailable | Where-Object RegistrationState -eq "Registered" | Select-Object ProviderNamespace, RegistrationState | Sort-Object ProviderNamespace
```

Pour conserver les privilèges minimaux dans votre abonnement, inscrivez uniquement les fournisseurs de ressources que vous êtes prêt à utiliser. Pour inscrire un fournisseur de ressources, utilisez :

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Qui retourne des résultats semblables à :

```output
ProviderNamespace : Microsoft.Batch
RegistrationState : Registering
ResourceTypes     : {batchAccounts, operations, locations, locations/quotas}
Locations         : {West Europe, East US, East US 2, West US...}
```

Pour afficher des informations pour un fournisseur de ressources particulier, utilisez :

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Batch
```

Qui retourne des résultats semblables à :

```output
{ProviderNamespace : Microsoft.Batch
RegistrationState : Registered
ResourceTypes     : {batchAccounts}
Locations         : {West Europe, East US, East US 2, West US...}

...
```

Pour afficher les types de ressources pour un fournisseur de ressources, utilisez :

```azurepowershell-interactive
(Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes.ResourceTypeName
```

Résultat :

```output
batchAccounts
operations
locations
locations/quotas
```

La version de l'API correspond à une version des opérations de l'API REST publiées par le fournisseur de ressources. Lorsqu'un fournisseur de ressources active de nouvelles fonctionnalités, une nouvelle version de l'API REST sera publiée.

Pour obtenir les versions d’API disponibles pour un type de ressource, utilisez :

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).ApiVersions
```

Résultat :

```output
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

Resource Manager est pris en charge dans toutes les régions, mais il est possible que certaines ressources que vous déployez ne soient pas prises en charge dans toutes les régions. En outre, il peut y avoir des limitations sur votre abonnement qui vous empêchent d’utiliser certaines régions prenant en charge la ressource.

Pour obtenir les emplacements pris en charge pour un type de ressource, utilisez :

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

Résultat :

```output
West Europe
East US
East US 2
West US
...
```

## <a name="azure-cli"></a>Azure CLI

Pour afficher tous les fournisseurs de ressources dans Azure et l’état de l’inscription de votre abonnement, utilisez :

```azurecli-interactive
az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
```

Qui retourne des résultats semblables à :

```output
Provider                         Status
-------------------------------- ----------------
Microsoft.ClassicCompute         Registered
Microsoft.ClassicNetwork         Registered
Microsoft.ClassicStorage         Registered
Microsoft.CognitiveServices      Registered
...
```

Pour afficher tous les fournisseurs de ressources inscrits pour votre abonnement, utilisez la commande suivante :

```azurecli-interactive
az provider list --query "sort_by([?registrationState=='Registered'].{Provider:namespace, Status:registrationState}, &Provider)" --out table
```

Pour conserver les privilèges minimaux dans votre abonnement, inscrivez uniquement les fournisseurs de ressources que vous êtes prêt à utiliser. Pour inscrire un fournisseur de ressources, utilisez :

```azurecli-interactive
az provider register --namespace Microsoft.Batch
```

Qui retourne un message indiquant que l’inscription est en cours.

Pour afficher des informations pour un fournisseur de ressources particulier, utilisez :

```azurecli-interactive
az provider show --namespace Microsoft.Batch
```

Qui retourne des résultats semblables à :

```output
{
    "id": "/subscriptions/####-####/providers/Microsoft.Batch",
    "namespace": "Microsoft.Batch",
    "registrationsState": "Registering",
    "resourceTypes:" [
        ...
    ]
}
```

Pour afficher les types de ressources pour un fournisseur de ressources, utilisez :

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[*].resourceType" --out table
```

Résultat :

```output
Result
---------------
batchAccounts
operations
locations
locations/quotas
```

La version de l'API correspond à une version des opérations de l'API REST publiées par le fournisseur de ressources. Lorsqu'un fournisseur de ressources active de nouvelles fonctionnalités, une nouvelle version de l'API REST sera publiée.

Pour obtenir les versions d’API disponibles pour un type de ressource, utilisez :

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].apiVersions | [0]" --out table
```

Résultat :

```output
Result
---------------
2017-05-01
2017-01-01
2015-12-01
2015-09-01
2015-07-01
```

Resource Manager est pris en charge dans toutes les régions, mais il est possible que certaines ressources que vous déployez ne soient pas prises en charge dans toutes les régions. En outre, il peut y avoir des limitations sur votre abonnement qui vous empêchent d’utiliser certaines régions prenant en charge la ressource.

Pour obtenir les emplacements pris en charge pour un type de ressource, utilisez :

```azurecli-interactive
az provider show --namespace Microsoft.Batch --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" --out table
```

Résultat :

```output
Result
---------------
West Europe
East US
East US 2
West US
...
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la création de modèles Resource Manager, consultez [Création de modèles Azure Resource Manager](../templates/template-syntax.md). 
* Pour afficher les schémas liés aux modèles de fournisseurs de ressources, consultez [Référence au modèle](/azure/templates/).
* Pour obtenir la liste qui mappe les fournisseurs de ressources aux services Azure, consultez [Fournisseurs de ressources pour les services Azure](azure-services-resource-providers.md).
* Pour afficher les opérations pour un fournisseur de ressources, consultez [API REST Azure](/rest/api/).
