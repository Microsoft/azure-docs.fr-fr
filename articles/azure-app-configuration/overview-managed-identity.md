---
title: Configurer des identités managées avec Azure App Configuration
description: Découvrez comment fonctionnent les identités managées dans Azure App Configuration et comment configurer une identité managée
author: barbkess
ms.topic: article
ms.date: 02/25/2020
ms.author: barbkess
ms.reviewer: lcozzens
ms.service: azure-app-configuration
ms.openlocfilehash: 270f8d471a95e67939a31357bc1cc056fe8ea4a1
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072796"
---
# <a name="how-to-use-managed-identities-for-azure-app-configuration"></a>Découvrez comment utiliser des identités managées pour Azure App Configuration

Cette rubrique explique comment créer une identité managée pour Azure App Configuration. Une identité managée issue d’Azure Active Directory (AAD) permet à Azure App Configuration d’accéder facilement à d’autres ressources protégées par AAD, telles qu’Azure Key Vault. L’identité est managée par la plateforme Azure. Vous n’avez pas besoin d’approvisionner des secrets ou d’assurer leur rotation. Pour plus d’informations sur les identités managées dans AAD, consultez [Identités gérées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

Deux types d’identité peuvent être accordés à votre application :

- Une **identité attribuée par le système** est liée à votre magasin de configuration. Elle est supprimée en cas de suppression de votre magasin de configuration. Un magasin de configuration ne peut avoir qu’une seule identité attribuée par le système.
- Une **identité attribuée par l’utilisateur** est une ressource Azure autonome qui peut être attribuée à votre magasin de configuration. Un magasin de configuration peut avoir plusieurs identités attribuées par l’utilisateur.

## <a name="adding-a-system-assigned-identity"></a>Ajout d’une identité attribuée par le système

La création d’un magasin App Configuration avec une identité attribuée par le système requiert la définition d’une propriété supplémentaire sur le magasin.

### <a name="using-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure (CLI)

Pour configurer une identité managée à l’aide d’Azure CLI, utilisez la commande [az appconfig identity assign] sur un magasin de configuration existant. Vous avez le choix entre trois options pour exécuter les exemples de cette section :

- Utiliser [Azure Cloud Shell](../cloud-shell/overview.md) à partir du portail Azure.
- Utiliser le service incorporé Azure Cloud Shell via le bouton « Essayer », en haut à droite de chaque bloc de code ci-dessous.
- [Installez la dernière version d’Azure CLI](/cli/azure/install-azure-cli) (2.1 ou ultérieure) si vous préférez utiliser une console CLI locale.

Les étapes suivantes vous guident dans la création d’un magasin App Configuration et l’attribution à celui-ci d’une identité à l’aide de l’interface de ligne de commande CLI :

1. Si vous utilisez l’interface de ligne de commande Azure dans une console locale, commencez par vous connecter à Azure avec [az login]. Utilisez un compte associé à votre abonnement Azure :

    ```azurecli-interactive
    az login
    ```

1. Créez un magasin App Configuration à l’aide de CLI. Pour plus d’exemples sur l’utilisation de l’interface CLI avec Azure App Configuration, voir [Exemples d’interface de ligne de commande App Configuration](scripts/cli-create-service.md) :

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Exécutez la commande [az appconfig identity assign] pour créer l’identité attribuée par le système pour ce magasin de configuration :

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup
    ```

## <a name="adding-a-user-assigned-identity"></a>Ajout d’une identité attribuée par l’utilisateur

La création d’un magasin App Configuration avec une identité attribuée par l’utilisateur nécessite la création de l’identité, puis l’attribution de son identificateur de ressource à votre magasin.

### <a name="using-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure (CLI)

Pour configurer une identité managée à l’aide d’Azure CLI, utilisez la commande [az appconfig identity assign] sur un magasin de configuration existant. Vous avez le choix entre trois options pour exécuter les exemples de cette section :

- Utiliser [Azure Cloud Shell](../cloud-shell/overview.md) à partir du portail Azure.
- Utiliser le service incorporé Azure Cloud Shell via le bouton « Essayer », en haut à droite de chaque bloc de code ci-dessous.
- [Installez la dernière version d’Azure CLI](/cli/azure/install-azure-cli) (2.0.31 ou ultérieure) si vous préférez utiliser une console CLI locale.

Les étapes suivantes vous guident dans la création d’une identité attribuée par l’utilisateur et d’un magasin App Configuration, puis l’attribution de l’identité au magasin à l’aide de l’interface CLI :

1. Si vous utilisez l’interface de ligne de commande Azure dans une console locale, commencez par vous connecter à Azure avec [az login]. Utilisez un compte associé à votre abonnement Azure :

    ```azurecli-interactive
    az login
    ```

1. Créez un magasin App Configuration à l’aide de CLI. Pour plus d’exemples sur l’utilisation de l’interface CLI avec Azure App Configuration, voir [Exemples d’interface de ligne de commande App Configuration](scripts/cli-create-service.md) :

    ```azurecli-interactive
    az group create --name myResourceGroup --location eastus
    az appconfig create --name myTestAppConfigStore --location eastus --resource-group myResourceGroup --sku Free
    ```

1. Créez une identité attribuée par l’utilisateur nommée `myUserAssignedIdentity` à l’aide de CLI.

    ```azurecli-interactive
    az identity create -resource-group myResourceGroup --name myUserAssignedIdentity
    ```

    Dans la sortie de cette commande, notez la valeur de la propriété `id`.

1. Exécutez la commande [az appconfig identity assign] pour affecter la nouvelle identité attribuée par l’utilisateur à ce magasin de configuration. Utilisez la valeur de la propriété `id` que vous avez notée à l’étape précédente.

    ```azurecli-interactive
    az appconfig identity assign --name myTestAppConfigStore --resource-group myResourceGroup --identities /subscriptions/[subscription id]/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity
    ```

## <a name="removing-an-identity"></a>Suppression d’une identité

Vous pouvez supprimer une identité attribuée par le système en désactivant la fonctionnalité à l’aide de la commande [az appconfig identity remove](/cli/azure/appconfig/identity?view=azure-cli-latest#az-appconfig-identity-remove) dans Azure CLI. Les identités attribuées par l’utilisateur peuvent être supprimées individuellement. Si vous supprimez une identité attribuée par le système de cette façon, vous la supprimez également d’AAD. Les identités attribuées par le système sont aussi automatiquement supprimées d’AAD lorsque la ressource d’application est supprimée.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application ASP.NET Core avec Azure App Configuration](quickstart-aspnet-core-app.md)

[az appconfig identity assign]: /cli/azure/appconfig/identity?view=azure-cli-latest#az-appconfig-identity-assign
[az login]: /cli/azure/reference-index#az-login