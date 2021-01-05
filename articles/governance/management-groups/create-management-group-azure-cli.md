---
title: 'Démarrage rapide : Créer un groupe d’administration avec Azure CLI'
description: Dans ce guide de démarrage rapide, vous utilisez Azure CLI pour créer un groupe d’administration afin d’organiser vos ressources dans une hiérarchie de ressources.
ms.date: 08/31/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 54d7afbaa4bfe2799ca0d68927394a38d9845c53
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920659"
---
# <a name="quickstart-create-a-management-group-with-the-azure-cli"></a>Démarrage rapide : Créer un groupe d’administration avec Azure CLI

Les groupes d’administration sont des conteneurs qui vous aident à gérer l’accès, la stratégie et la conformité dans plusieurs abonnements. Créez ces conteneurs pour construire une hiérarchie efficace utilisable avec [Azure Policy](../policy/overview.md) et les [contrôles d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md). Pour plus d’informations sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](overview.md).

La création du premier groupe d’administration dans l’annuaire peut nécessiter jusqu’à 15 minutes. En effet, des processus s’exécutent la première fois pour configurer le service des groupes d’administration dans Azure pour votre annuaire. Vous recevez une notification quand le processus est terminé. Pour plus d’informations, consultez la [configuration initiale des groupes d’administration](./overview.md#initial-setup-of-management-groups).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- Ce guide de démarrage rapide nécessite que vous exécutiez la version 2.0.76 (ou ultérieure) d’Azure CLI pour installer et utiliser l’interface de ligne de commande localement. Pour connaître la version de l’interface, exécutez `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

- Tout utilisateur Azure AD dans le locataire peut créer un groupe d’administration sans l’autorisation d’écriture du groupe d’administration attribuée à cet utilisateur si la [protection de la hiérarchie](./how-to/protect-resource-hierarchy.md#setting---require-authorization) n’est pas activée. Ce nouveau groupe d’administration devient alors un enfant du groupe d’administration racine ou le [groupe d’administration par défaut](./how-to/protect-resource-hierarchy.md#setting---default-management-group), et le rôle « Propriétaire » est attribué au créateur. Le service de groupe d’administration offre cette possibilité afin que les attributions de rôles ne soient pas nécessaires au niveau de la racine. Aucun utilisateur n’a accès au groupe d’administration racine lors de sa création. Pour éviter l’obstacle que constitue la recherche des administrateurs généraux Azure AD pour commencer à utiliser des groupes d’administration, nous autorisons la création des groupes d’administration initiaux au niveau racine.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

### <a name="create-in-the-azure-cli"></a>Créer dans Azure CLI

Dans Azure CLI, utilisez la commande [az account management-group create](/cli/azure/account/management-group#az_account_management_group_create) pour créer un groupe d’administration. Dans cet exemple, la valeur **name** du groupe d’administration est _Contoso_.

```azurecli-interactive
az account management-group create --name 'Contoso'
```

**name** est un identificateur unique créé. Cet ID est utilisé par d’autres commandes pour faire référence à ce groupe et il ne peut pas être modifié ultérieurement.

Si vous voulez afficher un autre nom pour le groupe d’administration dans le Portail Azure, ajoutez le paramètre **display-name**. Par exemple, pour créer un groupe d’administration dont l’identificateur GroupName est Contoso et le nom d’affichage « Contoso Group », utilisez la commande suivante :

```azurecli-interactive
az account management-group create --name 'Contoso' --display-name 'Contoso Group'
```

Dans les exemples précédents, le groupe d’administration est créé sous le groupe d’administration racine. Pour spécifier un autre groupe d’administration en tant que parent, utilisez le paramètre **parent** et fournissez le nom du groupe parent.

```azurecli-interactive
az account management-group create --name 'ContosoSubGroup' --parent 'Contoso'
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer le groupe d’administration créé ci-dessus, utilisez la commande [az account management-group delete](/cli/azure/account/management-group#az_account_management_group_delete) :

```azurecli-interactive
az account management-group delete --name 'Contoso'
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un groupe d’administration pour organiser votre hiérarchie de ressources. Le groupe d’administration peut contenir des abonnements ou d’autres groupes d’administration.

Pour en savoir plus sur les groupes d’administration et sur la gestion de votre hiérarchie de ressources, consultez :

> [!div class="nextstepaction"]
> [Gérer vos ressources avec des groupes d’administration](./manage.md)