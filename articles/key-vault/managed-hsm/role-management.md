---
title: Gestion des rôles de plan de données HSM managé - Azure Key Vault | Microsoft Docs
description: Utilisez cet article pour gérer les affectations de rôles pour votre HSM managé
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 78d4d9a8b5023731530c5e348f5c9ba72815d410
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445640"
---
# <a name="managed-hsm-role-management"></a>Gestion du rôle HSM managé

> [!NOTE]
> Key Vault prend en charge deux types de ressources : les coffres et les HSM managés. Cet article concerne le **HSM managé**. Si vous souhaitez apprendre à gérer un coffre, consultez [Gérer Key Vault à l’aide de l’interface de ligne de commande Azure](../general/manage-with-cli2.md).

Pour obtenir une vue d’ensemble de Managed HSM, consultez [Qu’est-ce que Managed HSM ?](overview.md). Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Cet article vous explique comment gérer les rôles pour un plan de données HSM managé. Pour en savoir plus sur le modèle de contrôle d’accès HSM managé, consultez [Contrôle d’accès HSM managé](access-control.md).

Pour permettre à un principal de sécurité (par exemple un utilisateur, un principal du service, un groupe ou une identité managée) d’effectuer des opérations de plan de données HSM managé, il doit se voir attribuer un rôle qui autorise l’exécution de ces opérations. Par exemple, si vous souhaitez permettre à une application d’effectuer une opération de signature à l’aide d’une clé, elle doit recevoir un rôle contenant« Microsoft.KeyVault/managedHSM/keys/sign/action » comme l’une des actions de données. Un rôle peut être attribué au niveau d’une étendue spécifique. Le RBAC local HSM managé prend en charge deux étendues, à l’échelle du HSM (`/` ou `/keys`) et par clé (`/keys/<keyname>`).

Pour obtenir la liste de tous les rôles intégrés au HSM managé et les opérations qu’ils autorisent, consultez [Rôles intégrés au HSM managé](built-in-roles.md).

## <a name="prerequisites"></a>Prérequis

Pour utiliser les commandes Azure CLI de cet article, vous devez disposer des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 ou version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).
* HSM managé dans votre abonnement. Consultez [Démarrage rapide : Provisionner et activer un HSM managé à l’aide d’Azure CLI](quick-create-cli.md) pour provisionner et activer un HSM managé.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli
az login
```

Pour plus d’informations sur les options de connexion via l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)

## <a name="create-a-new-role-assignment"></a>Créer une attribution de rôle

### <a name="assign-roles-for-all-keys"></a>Attribuer des rôles pour toutes les clés

Utilisez la commande `az keyvault role assignment create` pour attribuer un rôle **Responsable du chiffrement du HSM managé** à l’utilisateur identifié par le nom d’utilisateur principal **user2\@contoso.com** pour toutes les **clés** (étendue`/keys`) dans ContosoHSM.

```azurecli-interactive
az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys
```

### <a name="assign-role-for-a-specific-key"></a>Affecter un rôle pour une clé spécifique

Utilisez la commande `az keyvault role assignment create` pour attribuer un rôle **Responsable du chiffrement du HSM managé** à l’utilisateur identifié par le nom d’utilisateur principal **user2\@contoso.com** pour une clé spécifique nommée **myrsakey**.

```azurecli-interactive
az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys/myrsakey
```

## <a name="list-existing-role-assignments"></a>Lister les attributions de rôles existantes

Utilisez `az keyvault role assignment list` pour lister les attributions de rôles.

Toutes les attributions de rôles au niveau de l’étendue / (valeur par défaut si aucune étendue n’est spécifiée) pour tous les utilisateurs (valeur par défaut quand aucun destinataire n’est spécifié)

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM
```

Toutes les attributions de rôles au niveau du HSM pour un utilisateur spécifique **user1@contoso.com** .

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user@contoso.com
```

Toutes les attributions de rôles pour un utilisateur spécifique **user2@contoso.com** et une clé spécifique **myrsakey**.

```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user2@contoso.com --scope /keys/myrsakey
```

Une attribution de rôle spécifique pour le rôle **Responsable du chiffrement du HSM managé** pour un utilisateur spécifique **user2@contoso.com** et une clé spécifique **myrsakey**


```azurecli-interactive
az keyvault role assignment list --hsm-name ContosoMHSM --assignee user2@contoso.com --scope /keys/myrsakey --role "Managed HSM Crypto Officer"
```

## <a name="delete-a-role-assignment"></a>Supprimer une attribution de rôle

Utilisez la commande `az keyvault role assignment delete` pour supprimer un rôle **Responsable du chiffrement du HSM managé** attribué à l’utilisateur **user2\@contoso.com** pour la clé **myrsakey2**.

```azurecli-interactive
az keyvault role assignment delete --hsm-name ContosoMHSM --role "Managed HSM Crypto Officer" --assignee user2@contoso.com  --scope /keys/myrsakey2
```

## <a name="list-all-available-role-definitions"></a>Lister toutes les définitions de rôles disponibles

Utilisez la commande `az keyvault role definition list` pour lister toutes les définitions de rôles.

```azurecli-interactive
az keyvault role definition list --hsm-name ContosoMHSM
```

## <a name="next-steps"></a>Étapes suivantes

- Lisez la présentation du [Contrôle d’accès en fonction du rôle (Azure RBAC)](../../role-based-access-control/overview.md).
- Suivre un tutoriel sur la [gestion du rôle HSM managé](role-management.md)
- Découvrir le [modèle de contrôle d’accès HSM managé](access-control.md)
- Voir tous les [rôles intégrés pour le RBAC local HSM managé](built-in-roles.md)
