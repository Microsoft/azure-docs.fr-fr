---
title: Supprimer l’accès à une délégation
description: Découvrez comment supprimer l’accès aux ressources qui ont été déléguées à un fournisseur de services pour Azure Lighthouse.
ms.date: 12/03/2020
ms.topic: how-to
ms.openlocfilehash: d3442bb9fd2f6e7423fd4bf28cace1f7fd91ad80
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608450"
---
# <a name="remove-access-to-a-delegation"></a>Supprimer l’accès à une délégation

Une fois que le groupe de ressources ou l’abonnement d’un client a été délégué à un fournisseur de services pour [Azure Lighthouse](../overview.md), la délégation peut être supprimée si nécessaire. Une fois qu’une délégation est supprimée, l’accès à [la gestion des ressources déléguées Azure Lighthouse](../concepts/azure-delegated-resource-management.md) précédemment accordé aux utilisateurs du locataire du fournisseur de services ne s’appliquera plus.

La suppression d’une délégation peut être effectuée par un utilisateur dans le locataire du client ou dans le locataire du fournisseur de services, tant que l’utilisateur dispose des autorisations appropriées.

> [!TIP]
> Même si nous faisons référence aux fournisseurs de services et aux clients dans cette rubrique, les [entreprises gérant plusieurs locataires](../concepts/enterprise.md) peuvent utiliser les mêmes processus.

## <a name="customers"></a>Clients

Les utilisateurs du locataire du client qui disposent du [rôle intégré Propriétaire](../../role-based-access-control/built-in-roles.md#owner) pour un abonnement peuvent supprimer l’accès du fournisseur de services à cet abonnement (ou aux groupes de ressources de cet abonnement). Pour ce faire, un utilisateur du locataire du client peut accéder à la [page des fournisseurs de services](view-manage-service-providers.md#add-or-remove-service-provider-offers) du Portail Azure, rechercher l’offre sur l’écran des **offres du fournisseur de services**, puis sélectionner l’icône de la corbeille sur la ligne de cette offre.

Une fois la suppression confirmée, aucun utilisateur du locataire du fournisseur de services ne pourra accéder aux ressources précédemment déléguées.

## <a name="service-providers"></a>Fournisseurs de services

Les utilisateurs d’un locataire gérant peuvent supprimer l’accès aux ressources déléguées s’ils ont reçu le [rôle Supprimer l’attribution de l’inscription des services gérés](../../role-based-access-control/built-in-roles.md#managed-services-registration-assignment-delete-role) pour les ressources du client. Si ce rôle n’a été attribué à aucun utilisateur du fournisseur de services, la délégation peut être supprimée uniquement par un utilisateur du locataire du client.

L’exemple ci-dessous montre une affectation octroyant le **rôle Supprimer l’attribution de l’inscription des services gérés** pouvant être inclus dans un fichier de paramètres pendant le [processus d’intégration](onboard-customer.md) :

```json
    "authorizations": [ 
        { 
            "principalId": "cfa7496e-a619-4a14-a740-85c5ad2063bb", 
            "principalIdDisplayName": "MSP Operators", 
            "roleDefinitionId": "91c1777a-f3dc-4fae-b103-61d183457e46" 
        } 
    ] 
```

Ce rôle peut également être sélectionné dans une **autorisation** lors de la [création d’une offre de service géré](../../marketplace/partner-center-portal/create-new-managed-service-offer.md#authorization) à publier sur la Place de marché Azure.

Un utilisateur disposant de cette autorisation peut supprimer une délégation de l’une des manières suivantes.

### <a name="azure-portal"></a>Portail Azure

1. Accédez à la [page Mes clients](view-manage-customers.md).
2. Sélectionnez **Délégations**.
3. Recherchez la délégation que vous souhaitez supprimer, puis sélectionnez l’icône de corbeille qui apparaît dans sa ligne.

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
# Log in first with Connect-AzAccount if you're not using Cloud Shell

# Sign in as a user from the managing tenant directory 

Login-AzAccount

# Select the subscription that is delegated - or contains the delegated resource group(s)

Select-AzSubscription -SubscriptionName "<subscriptionName>"

# Get the registration assignment

Get-AzManagedServicesAssignment -Scope "/subscriptions/{delegatedSubscriptionId}"

# Delete the registration assignment

Remove-AzManagedServicesAssignment -ResourceId "/subscriptions/{delegatedSubscriptionId}/providers/Microsoft.ManagedServices/registrationAssignments/{assignmentGuid}"
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Sign in as a user from the managing tenant directory

az login

# Select the subscription that is delegated – or contains the delegated resource group(s)

az account set -s <subscriptionId/name>

# List registration assignments

az managedservices assignment list

# Delete the registration assignment

az managedservices assignment delete --assignment <id or full resourceId>
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [Gestion des ressources déléguées Azure](../concepts/azure-delegated-resource-management.md).
- [Affichez et gérez les clients](view-manage-customers.md) en accédant à **Mes clients** sur le portail Azure.
- Apprenez à [mettre à jour une délégation précédente](update-delegation.md).
