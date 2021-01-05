---
title: Accorder à des applications l’autorisation d’accéder à un coffre de clés Azure en utilisant le RBAC Azure | Microsoft Docs
description: Découvrez comment donner accès aux clés, secrets et certificats à l’aide d’un contrôle d’accès en fonction du rôle Azure.
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 8/30/2020
ms.author: mbaldwin
ms.openlocfilehash: be2fa0fa85da7c0f02567580c50968b7418e4cde
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445181"
---
# <a name="provide-access-to-key-vault-keys-certificates-and-secrets-with-an-azure-role-based-access-control-preview"></a>Donnez accès aux clés, certificats et secrets du coffre de clés avec un contrôle d’accès en fonction du rôle Azure (préversion)

> [!NOTE]
> Le fournisseur de ressources Key Vault prend en charge deux types de ressources : les **coffres** et les **HSM managés**. Le contrôle d’accès décrit dans cet article s’applique uniquement aux **coffres**. Pour en savoir plus sur le contrôle d’accès pour le HSM managé, consultez [Contrôle d’accès HSM managé](../managed-hsm/access-control.md).

Le contrôle d’accès en fonction du rôle (RBAC Azure) est un système d’autorisation basé sur [Azure Resource Manager](../../azure-resource-manager/management/overview.md), qui permet une gestion précise des accès des ressources Azure.

Le RBAC Azure permet aux utilisateurs de gérer les autorisations de clé, de secrets et de certificats. Il fournit un emplacement unique pour gérer toutes les autorisations sur tous les coffres de clés. 

Le modèle de RBAC Azure offre la possibilité de définir des autorisations pour différents niveaux d’étendue : groupe d’administration, abonnement, groupe de ressources ou ressources individuelles.  Le RBAC Azure pour le coffre de clés offre également la possibilité d’avoir des autorisations distinctes sur des clés, des secrets et des certificats individuels.

Pour plus d’informations, consultez [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)

## <a name="best-practices-for-individual-keys-secrets-and-certificates"></a>Meilleures pratiques pour les clés, secrets et certificats individuels

Nous recommandons d’utiliser un coffre par application et par environnement (développement, préproduction et production).

Vous ne devez utiliser des autorisations sur des clés, secrets et certificats individuels que pour des scénarios spécifiques :

-   Applications multicouches qui doivent séparer le contrôle d’accès entre les couches

-   Partage d’un secret individuel entre plusieurs applications

Pour des instructions sur la gestion d’Azure Key Vault, consultez :

- [Meilleurs pratiques relatives à Azure Key Vault](best-practices.md)
- [Limites du service Azure Key Vault](service-limits.md)

## <a name="azure-built-in-roles-for-key-vault-data-plane-operations-preview"></a>Rôles intégrés Azure pour les opérations de plan de données d’Azure Key Vault (préversion)
> [!NOTE]
> Le rôle `Key Vault Contributor` permet aux opérations de plan de gestion de gérer les coffres de clés. Il n’autorise pas l’accès aux clés, aux secrets et aux certificats.

| Rôle intégré | Description | id |
| --- | --- | --- |
| Administrateur de Key Vault (préversion) | Permet d’effectuer toutes les opérations du plan de données sur un coffre de clés et tous les objets qu’il contient, y compris les certificats, les clés et les secrets. Ne peut pas gérer les ressources du coffre de clés ni gérer les attributions de rôles. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | 00482a5a-887f-4fb3-b363-3b7fe8e74483 |
| Responsable des certificats de Key Vault (préversion) | Permet d’effectuer une action sur les certificats d’un coffre de clés, à l’exception des autorisations de gestion. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | a4417e6f-fecd-4de8-b567-7b0420556985 |
| Responsable du chiffrement de Key Vault (préversion)| Permet d’effectuer une action sur les clés d’un coffre de clés, à l’exception des autorisations de gestion. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | 14b46e9e-c2b7-41b4-b07b-48a6ebf60603 |
| Service de chiffrement de Key Vault (préversion) | Permet de lire les métadonnées des clés et d’effectuer des opérations visant à envelopper/désenvelopper. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | e147488a-f6f5-4113-8e2d-b22465e65bf6 |
| Utilisateur de chiffrement de Key Vault (préversion) | Permet d’effectuer des opérations de chiffrement à l’aide de clés. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | 12338af0-0e69-4776-bea7-57ae8d297424 |
| Lecteur de Key Vault (préversion)| Permet de lire les métadonnées de coffres de clés et de leurs certificats, clés et secrets. Ne peut pas lire les valeurs sensibles, telles que les contenus secrets ou les documents clés. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | 21090545-7ca7-4776-b22c-e363652d74d2 |
| Responsable des secrets de Key Vault (préversion)| Permet d’effectuer une action sur les secrets d’un coffre de clés, à l’exception des autorisations de gestion. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | b86a8fe4-44ce-4948-aee5-eccb2c155cd7 |
| Utilisateur de secrets de Key Vault (préversion)| Permet de lire le contenu du secret. Fonctionne uniquement pour les coffres de clés qui utilisent le modèle d’autorisation « Contrôle d’accès en fonction du rôle Azure ». | 4633458b-17de-408a-b874-0445c86b69e6 |

Pour plus d’informations sur les définitions de rôles intégrés Azure, consultez [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md).

## <a name="using-azure-rbac-secret-key-and-certificate-permissions-with-key-vault"></a>Utilisation des autorisations de secret, de clé et de certificat de RBAC Azure avec Key Vault

Le nouveau modèle d’autorisation de RBAC Azure pour le coffre de clés fournit une alternative au modèle d’autorisations de stratégie d’accès au coffre. 

### <a name="prerequisites"></a>Prérequis

Pour ajouter des attributions de rôles, vous devez disposer :

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- d’autorisations `Microsoft.Authorization/roleAssignments/write` et `Microsoft.Authorization/roleAssignments/delete`, telles que [Administrateur de l’accès utilisateur](../../role-based-access-control/built-in-roles.md#user-access-administrator) ou [Propriétaire de l’accès utilisateur](../../role-based-access-control/built-in-roles.md#owner)

### <a name="enable-azure-rbac-permissions-on-key-vault"></a>Activer les autorisations de RBAC Azure sur Key Vault

> [!IMPORTANT]
> La définition d’un modèle d’autorisation de RBAC Azure invalide toutes les autorisations des stratégies d’accès. Cela peut entraîner des interruptions lorsque des rôles Azure équivalents ne sont pas attribués.

1.  Activer des autorisations de RBAC Azure sur un nouveau coffre de clés :

    ![Activer des autorisations Azure RBAC – Nouveau coffre](../media/rbac/image-1.png)

2.  Activez des autorisations de RBAC Azure sur un coffre de clés existant :

    ![Activer des autorisations Azure RBAC – Coffre existant](../media/rbac/image-2.png)

### <a name="assign-role"></a>Affecter le rôle

> [!Note]
> Il est recommandé d’utiliser l’ID de rôle unique à la place du nom de rôle dans les scripts. Par conséquent, si un rôle est renommé, vos scripts continuent de fonctionner. Lors de la période de préversion, chaque rôle portera le suffixe « (préversion) », qui sera supprimé par la suite. Dans ce document, le nom de rôle est utilisé uniquement à des fins de lisibilité.

Commande Azure CLI pour créer une attribution de rôle :

```azurecli
az role assignment create --role <role_name_or_id> --assignee <assignee> --scope <scope>
```

Dans le portail Azure, l’écran Attribution de rôle Azure est disponible pour toutes les ressources sous l’onglet Contrôle d’accès (IAM).

![Attribution de rôle – Onglet (IAM)](../media/rbac/image-3.png)

### <a name="resource-group-scope-role-assignment"></a>Attribution de rôle d’étendue de groupe de ressources

1.  Accédez au Groupe de ressources du coffre de clés.
    ![Attribution de rôle – Groupe de ressources](../media/rbac/image-4.png)

2.  Cliquez sur Contrôle d’accès (IAM) \> Ajouter une attribution de rôle \> Ajouter.

3.  Créez le rôle « Lecteur de Key Vault (préversion) » pour l’utilisateur actuel.

    ![Ajouter un rôle – Groupe de ressources](../media/rbac/image-5.png)

Azure CLI :
```azurecli
az role assignment create --role "Key Vault Reader (preview)" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}
```

L’attribution de rôle ci-dessus permet de répertorier les objets figurant dans le coffre de clés.

### <a name="key-vault-scope-role-assignment"></a>Attribution de rôle d’étendue de coffre de clés

1. Accéder à Key Vault \> onglet Contrôle d’accès (IAM).

2. Cliquez sur Ajouter une attribution de rôle \> Ajouter.

3. Créez le rôle « Responsable des secrets de Key Vault (préversion) » pour l’utilisateur actuel.

    ![Attribution de rôle – Coffre de clés](../media/rbac/image-6.png)

 Azure CLI :

```azurecli
az role assignment create --role "Key Vault Secrets Officer (preview)" --assignee {i.e jalichwa@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}
```

Après avoir créé l’attribution de rôle ci-dessus, vous pouvez créer/mettre à jour/supprimer des secrets.

4. Créez un secret (Secrets \> +Générer/Importer) pour tester l’attribution de rôle de niveau secret.

    ![Ajouter un rôle – Coffre de clés](../media/rbac/image-7.png)

### <a name="secret-scope-role-assignment"></a>Attribution de rôle d’étendue de secret

1. Ouvrez l’un des secrets créés précédemment. Repérez Vue d’ensemble et Contrôle d’accès (IAM) (préversion).

2. Cliquez sur l’onglet Contrôle d’accès (IAM) (préversion).

    ![Attribution de rôle – Secret](../media/rbac/image-8.png)

3. Créez le rôle « Responsable des secrets de Key Vault (préversion) » pour l’utilisateur actuel, comme ci-dessus pour le coffre de clés.

Azure CLI :

```azurecli
az role assignment create --role "Key Vault Secrets Officer (preview)" --assignee {i.e user@microsoft.com} --scope /subscriptions/{subscriptionid}/resourcegroups/{resource-group-name}/providers/Microsoft.KeyVault/vaults/{key-vault-name}/secrets/RBACSecret
```

### <a name="test-and-verify"></a>Tester et vérifier

> [!NOTE]
> Les navigateurs utilisant une mise en cache, une actualisation de la page est requise après la suppression d’attributions de rôles.<br>
> Laissez passer quelques minutes, le temps que les attributions de rôles soient actualisées.

1. Validez l’ajout du nouveau secret sans le rôle « Responsable des secrets de Key Vault » au niveau du coffre de clés.

Accédez à l’onglet Contrôle d’accès (IAM) du coffre de clés, puis supprimez l’attribution de rôle « Responsable des secrets de Key Vault (préversion) » pour cette ressource.

![Supprimer une attribution – Coffre de clés](../media/rbac/image-9.png)

Accédez au secret créé précédemment. Vous pouvez voir toutes les propriétés du secret.

![Affichage de secret avec accès](../media/rbac/image-10.png)

La page Créer un secret (Secrets \> +Générer/Importer) doit afficher l’erreur suivante :

   ![Créer un secret](../media/rbac/image-11.png)

2.  Validez la modification du secret sans le rôle « Responsable des secrets de Key Vault » au niveau du secret.

-   Accédez à l’onglet Contrôle d’accès (IAM) du secret créé précédemment, puis supprimez l’attribution de rôle « Responsable des secrets de Key Vault (préversion) » pour cette ressource.

-   Accédez au secret créé précédemment. Vous pouvez voir les propriétés du secret.

   ![Affichage de secret sans accès](../media/rbac/image-12.png)

3. Valider les secrets lus sans le rôle de lecteur au niveau du coffre de clés.

-   Accédez à l’onglet Contrôle d’accès (IAM) du groupe de ressources, puis supprimez l’attribution de rôle « Lecteur de Key Vault (préversion) ».

-   La navigation vers l’onglet Secrets du coffre de clés doit afficher l’erreur suivante :

   ![Onglet Secret – Erreur](../media/rbac/image-13.png)

### <a name="creating-custom-roles"></a>Création de rôles personnalisés 

[Commande az role definition create](/cli/azure/role/definition#az-role-definition-create)

**(Script Bash de CLI)</br>**
```azurecli
az role definition create --role-definition '{ \
   "Name": "Backup Keys Operator", \
   "Description": "Perform key backup/restore operations", \
    "Actions": [ 
    ], \
    "DataActions": [ \
        "Microsoft.KeyVault/vaults/keys/read ", \
        "Microsoft.KeyVault/vaults/keys/backup/action", \
         "Microsoft.KeyVault/vaults/keys/restore/action" \
    ], \
    "NotDataActions": [ 
   ], \
    "AssignableScopes": ["/subscriptions/{subscriptionId}"] \
}'
```

Pour plus d’informations sur la manière de créer des rôles personnalisés, consultez :

[Rôle personnalisés Azure](../../role-based-access-control/custom-roles.md)

## <a name="known-limits-and-performance"></a>Limites et performances connues

-   2000 attributions de rôles Azure par abonnement.

-   Latence des attributions de rôles : au niveau des performances attendues actuelles, jusqu’à 10 minutes (600 secondes) peuvent s’écouler après une modification des attributions de rôles, avant que le rôle soit appliqué.

## <a name="learn-more"></a>En savoir plus

- [Vue d’ensemble d’Azure RBAC](../../role-based-access-control/overview.md)
- [Tutoriel Rôles personnalisés](../../role-based-access-control/tutorial-custom-role-cli.md)