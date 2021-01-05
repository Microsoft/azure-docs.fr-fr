---
title: Transférer un abonnement Azure vers une autre instance Azure AD Directory
description: Découvrez comment transférer un abonnement Azure et les ressources associées connues vers un autre annuaire Azure Active Directory (Azure AD).
services: active-directory
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.workload: identity
ms.date: 12/10/2020
ms.author: rolyon
ms.openlocfilehash: 92b27690ab1f2ca8d98eb2231c5a27bc508613f8
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095421"
---
# <a name="transfer-an-azure-subscription-to-a-different-azure-ad-directory"></a>Transférer un abonnement Azure vers une autre instance Azure AD Directory

Les organisations peuvent avoir plusieurs abonnements Azure. Chaque abonnement est associé à un annuaire Azure Active Directory (Azure AD) spécifique. Pour faciliter la gestion, vous souhaiterez peut-être transférer un abonnement vers un autre annuaire Azure AD. Lorsque vous transférez un abonnement à un autre annuaire Azure AD, certaines ressources ne sont pas transférées vers l’annuaire cible. Par exemple, toutes les attributions de rôles et tous les rôles personnalisés dans le contrôle d’accès en fonction du rôle (RBAC) Azure sont **définitivement** supprimés de l’annuaire source et ne sont pas transférés vers l’annuaire cible.

Cet article décrit les étapes de base que vous pouvez suivre pour transférer un abonnement vers un autre annuaire Azure AD et recréer certaines des ressources après le transfert.

> [!NOTE]
> Pour les abonnements aux fournisseurs de solutions cloud (CSP) Azure, le changement d’annuaire Azure AD pour l’abonnement n’est pas pris en charge.

## <a name="overview"></a>Vue d’ensemble

Le transfert d’un abonnement Azure vers un autre annuaire Azure AD est un processus complexe qui doit être soigneusement planifié et exécuté. De nombreux services Azure requièrent des principaux de sécurité (identités) pour fonctionner normalement ou même gérer d’autres ressources Azure. Cet article tente de couvrir la plupart des services Azure qui dépendent fortement des principaux de sécurité, mais n’est pas exhaustif.

> [!IMPORTANT]
> Dans certains scénarios, le transfert d’un abonnement peut nécessiter un temps d’arrêt pour effectuer le processus. Effectuez une planification soignée pour évaluer si un temps d’arrêt est à prévoir pour votre transfert.

Le diagramme suivant illustre les étapes de base que vous devez suivre lorsque vous transférez un abonnement à un autre annuaire.

1. Préparer le transfert

1. Transfert de l’abonnement Azure vers un autre annuaire

1. Recréer les ressources dans l’annuaire cible, comme les attributions de rôles, les rôles personnalisés et les identités managées

    ![Diagramme de transfert d’abonnement](./media/transfer-subscription/transfer-subscription.png)

### <a name="deciding-whether-to-transfer-a-subscription-to-a-different-directory"></a>Déterminer la possibilité de transférer un abonnement vers un autre annuaire

Voici quelques raisons pour lesquelles vous pouvez souhaiter transférer un abonnement :

- En raison d’une fusion ou d’une acquisition d’entreprise, vous souhaitez gérer un abonnement acquis dans votre annuaire Azure AD principal.
- Une personne de votre organisation a créé un abonnement et vous souhaitez consolider la gestion sur un annuaire Azure AD particulier.
- Vous avez des applications qui dépendent d’un ID d’abonnement ou d’une URL spécifique, et il n’est pas facile de modifier la configuration ou le code de l’application.
- Une partie de votre entreprise a été divisée en une société distincte et vous devez déplacer certaines de vos ressources dans un autre annuaire Azure AD.
- Vous souhaitez gérer certaines de vos ressources dans un annuaire Azure AD différent pour des raisons d’isolation de sécurité.

### <a name="alternate-approaches"></a>Autres approches

Le transfert d’un abonnement nécessite un temps d’arrêt pour terminer le processus. Selon votre scénario, vous pouvez envisager les autres approches suivantes :

- Recréez les ressources et copiez les données vers l’annuaire et l’abonnement cibles.
- Adoptez une architecture à plusieurs annuaires et laissez l’abonnement dans l’annuaire source. Utilisez Azure Lighthouse pour déléguer des ressources afin que les utilisateurs dans l’annuaire cible puissent accéder à l’abonnement dans l’annuaire source. Pour plus d’informations, consultez [Azure Lighthouse dans les scénarios d’entreprise](../lighthouse/concepts/enterprise.md).

### <a name="understand-the-impact-of-transferring-a-subscription"></a>Comprendre l’impact du transfert d’un abonnement

Plusieurs ressources Azure dépendent d’un abonnement ou d’un annuaire. Selon votre situation, le tableau suivant répertorie l’impact connu du transfert d’un abonnement. En effectuant les étapes décrites dans cet article, vous pouvez recréer certaines des ressources qui existaient avant le transfert de l’abonnement.

> [!IMPORTANT]
> Cette section répertorie les services ou ressources Azure connus qui dépendent de votre abonnement. Étant donné que les types de ressources dans Azure évoluent constamment, il peut y avoir des dépendances supplémentaires non répertoriées ici, ce qui peut entraîner un changement cassant de votre environnement. 

| Service ou ressource | Affecté | Récupérable | Êtes-vous affecté ? | Ce que vous pouvez faire |
| --------- | --------- | --------- | --------- | --------- |
| Affectations de rôles | Oui | Oui | [Répertorier les attributions de rôles](#save-all-role-assignments) | Toutes les attributions de rôles sont définitivement supprimées. Vous devez mapper les utilisateurs, groupes et principaux du service aux objets correspondants dans l’annuaire cible. Vous devez recréer les attributions de rôles. |
| Rôles personnalisés | Oui | Oui | [Répertorier les rôles personnalisés](#save-custom-roles) | Tous les rôles personnalisés sont définitivement supprimés. Vous devez recréer les rôles personnalisés et les attributions de rôles. |
| Identités managées attribuées par le système | Oui | Oui | [Répertorier les identités managées](#list-role-assignments-for-managed-identities) | Vous devez désactiver et réactiver les identités managées. Vous devez recréer les attributions de rôles. |
| Identités managées attribuées par l’utilisateur | Oui | Oui | [Répertorier les identités managées](#list-role-assignments-for-managed-identities) | Vous devez supprimer, recréer et attacher les identités managées à la ressource appropriée. Vous devez recréer les attributions de rôles. |
| Azure Key Vault | Oui | Oui | [Répertorier les stratégies d’accès Key Vault](#list-key-vaults) | Vous devez mettre à jour l’ID de locataire associé aux coffres de clés. Vous devez supprimer et ajouter de nouvelles stratégies d’accès. |
| Bases de données Azure SQL avec l’intégration de l’authentification Azure AD activée | Oui | Non | [Vérifier les bases de données SQL Azure avec l’authentification Azure AD](#list-azure-sql-databases-with-azure-ad-authentication) |  |  |
| Stockage Azure et Azure Data Lake Storage Gen2 | Oui | Oui |  | Vous devez recréer toutes les listes de contrôle d’accès. |
| Azure Data Lake Storage Gen1 | Oui | Oui |  | Vous devez recréer toutes les listes de contrôle d’accès. |
| Azure Files | Oui | Oui |  | Vous devez recréer toutes les listes de contrôle d’accès. |
| Azure File Sync | Oui | Oui |  |  |
| Azure Disques managés | Oui | Oui |  |  Si vous utilisez des jeux de chiffrement de disque pour chiffrer les disques managés avec des clés gérées par le client, vous devez désactiver et réactiver les identités affectées par le système associées aux jeux de chiffrement du disque. De plus, vous devez recréer les attributions de rôles, c’est-à-dire accorder à nouveau les autorisations requises sur les jeux de chiffrement de disque dans les coffres de clés. |
| Azure Kubernetes Service | Oui | Oui |  |  |
| Azure Policy | Oui | Non | Tous les objets Azure Policy, y compris les définitions personnalisées, les affectations, les exemptions et les données de conformité. | Vous devez [exporter](../governance/policy/how-to/export-resources.md), importer et réaffecter les définitions. Créez ensuite de nouvelles affectations de stratégie et toutes les [exemptions de stratégie](../governance/policy/concepts/exemption-structure.md) nécessaires. |
| Azure Active Directory Domain Services | Oui | Non |  |  |
| Inscriptions des applications | Oui | Oui |  |  |

> [!WARNING]
> Si vous utilisez le chiffrement au repos pour une ressource, comme un compte de stockage ou une base de données SQL, qui a une dépendance sur un coffre de clés qui n’est **pas** dans l’abonnement en cours de transfert, cela peut entraîner un scénario irrécupérable. Si vous rencontrez cette situation, vous devez prendre les mesures nécessaires pour utiliser un coffre de clés différent ou pour désactiver temporairement les clés gérées par le client afin d’éviter ce scénario irrécupérable.

## <a name="prerequisites"></a>Prérequis

Pour effectuer cette procédure, vous avez besoin de :

- [Bash Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure CLI](/cli/azure)
- Administrateur de compte de l’abonnement que vous souhaitez transférer dans l’annuaire source
- Rôle de [Propriétaire](built-in-roles.md#owner) dans l’annuaire cible

## <a name="step-1-prepare-for-the-transfer"></a>Étape 1 : Préparer le transfert

### <a name="sign-in-to-source-directory"></a>Se connecter à l’annuaire source

1. Connectez-vous à Azure en tant qu’administrateur.

1. Obtenez une liste de vos abonnements à l’aide de la commande [az account list](/cli/azure/account#az_account_list).

    ```azurecli
    az account list --output table
    ```

1. Utilisez [az account set](/cli/azure/account#az_account_set) pour définir l’abonnement actif que vous souhaitez transférer.

    ```azurecli
    az account set --subscription "Marketing"
    ```

### <a name="install-the-azure-resource-graph-extension"></a>Installer l’extension Azure Resource Graph

 L’extension Azure CLI pour [Azure Resource Graph](../governance/resource-graph/index.yml), *resource-graph*, vous permet d’utiliser la commande [az graph](/cli/azure/ext/resource-graph/graph) pour interroger les ressources gérées par Azure Resource Manager. Vous utiliserez cette commande dans les étapes ultérieures.

1. Utilisez [az extension list](/cli/azure/extension#az_extension_list) pour voir si l’extension *resource-graph* est installée.

    ```azurecli
    az extension list
    ```

1. Si ce n’est pas le cas, installez l’extension *resource-graph*.

    ```azurecli
    az extension add --name resource-graph
    ```

### <a name="save-all-role-assignments"></a>Enregistrer toutes les affectations de rôle

1. Utilisez [az role assignment list](/cli/azure/role/assignment#az_role_assignment_list) pour répertorier toutes les attributions de rôles (y compris les attributions de rôles héritées).

    Pour faciliter la consultation de la liste, vous pouvez exporter la sortie au format JSON, TSV ou dans une table. Pour plus d’informations, consultez [Lister les attributions de rôles à l’aide du RBAC Azure et d’Azure CLI](role-assignments-list-cli.md).

    ```azurecli
    az role assignment list --all --include-inherited --output json > roleassignments.json
    az role assignment list --all --include-inherited --output tsv > roleassignments.tsv
    az role assignment list --all --include-inherited --output table > roleassignments.txt
    ```

1. Enregistrez la liste des attributions de rôles.

    Lorsque vous transférez un abonnement, toutes les attributions de rôle sont **définitivement** supprimées. Il est donc important d’enregistrer une copie.

1. Passez en revue la liste des attributions de rôles. Il peut y avoir des attributions de rôle dont vous n’avez pas besoin dans l’annuaire cible.

### <a name="save-custom-roles"></a>Enregistrer des rôles personnalisés

1. Utilisez [az role definition list](/cli/azure/role/definition#az_role_definition_list) pour répertorier vos rôles personnalisés. Pour plus d’informations, consultez [Créer ou mettre à jour des rôles personnalisés Azure à l’aide d’Azure CLI](custom-roles-cli.md).

    ```azurecli
    az role definition list --custom-role-only true --output json --query '[].{roleName:roleName, roleType:roleType}'
    ```

1. Enregistrez chaque rôle personnalisé dont vous aurez besoin dans l’annuaire cible sous la forme d’un fichier JSON distinct.

    ```azurecli
    az role definition list --name <custom_role_name> > customrolename.json
    ```

1. Effectuez des copies des fichiers de rôle personnalisés.

1. Modifiez chaque copie pour utiliser le format suivant.

    Vous utiliserez ces fichiers ultérieurement pour recréer les rôles personnalisés dans l’annuaire cible.

    ```json
    {
      "Name": "",
      "Description": "",
      "Actions": [],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": []
    }
    ```

### <a name="determine-user-group-and-service-principal-mappings"></a>Déterminer les mappages d’utilisateurs, de groupes et de principaux de service

1. En fonction de votre liste d’attributions de rôles, déterminez les utilisateurs, groupes et principaux de service à mapper dans l’annuaire cible.

    Vous pouvez identifier le type de principal en examinant la propriété `principalType` dans chaque attribution de rôle.

1. Si nécessaire, dans l’annuaire cible, créez les utilisateurs, groupes ou principaux de service dont vous aurez besoin.

### <a name="list-role-assignments-for-managed-identities"></a>Répertorier les attributions de rôles pour les identités managées

Les identités managées ne sont pas mises à jour lorsqu’un abonnement est transféré vers un autre annuaire. Par conséquent, toutes les identités managées existantes affectées par le système ou par l’utilisateur seront rompues. Après le transfert, vous pouvez réactiver toutes les identités managées affectées par le système. Pour les identités managées affectées par l’utilisateur, vous devez les recréer et les joindre dans l’annuaire cible.

1. Passez en revue la [liste des services Azure qui prennent en charge les identités managées](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) pour noter où vous pouvez utiliser des identités managées.

1. Utilisez [az ad sp list](/cli/azure/ad/sp#az_ad_sp_list) pour répertorier vos identités managées affectées par le système et affectées par l’utilisateur.

    ```azurecli
    az ad sp list --all --filter "servicePrincipalType eq 'ManagedIdentity'"
    ```

1. Dans la liste des identités managées, déterminez celles qui sont affectées par le système et celles qui sont affectées par l’utilisateur. Vous pouvez utiliser les critères suivants pour déterminer le type.

    | Critères | Type d'identité managée |
    | --- | --- |
    | La propriété `alternativeNames` inclut `isExplicit=False` | Attribué par le système |
    | La propriété `alternativeNames` n’inclut pas `isExplicit` | Attribué par le système |
    | La propriété `alternativeNames` inclut `isExplicit=True` | Affecté par l’utilisateur |

    Vous pouvez également utiliser [az identity list](/cli/azure/identity#az_identity_list) pour répertorier uniquement les identités managées affectées à l’utilisateur. Pour plus d’informations, consultez [Créer, répertorier ou supprimer une identité managée affectée par l’utilisateur en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md).

    ```azurecli
    az identity list
    ```

1. Obtenez la liste des valeurs `objectId` pour vos identités managées.

1. Effectuez une recherche dans votre liste d’attributions de rôles pour voir s’il existe des attributions de rôles pour vos identités managées.

### <a name="list-key-vaults"></a>List key vaults (Afficher la liste des Key Vaults)

Lorsque vous créez un coffre de clés, celui-ci est automatiquement lié à l’ID de client Azure Active Directory par défaut pour l’abonnement dans lequel il est créé. Toutes les entrées de stratégie d’accès sont également liées à cet ID de client. Pour plus d’informations, consultez [Déplacement d’un coffre Azure Key Vault vers un nouvel abonnement](../key-vault/general/move-subscription.md).

> [!WARNING]
> Si vous utilisez le chiffrement au repos pour une ressource, comme un compte de stockage ou une base de données SQL, qui a une dépendance sur un coffre de clés qui n’est **pas** dans l’abonnement en cours de transfert, cela peut entraîner un scénario irrécupérable. Si vous rencontrez cette situation, vous devez prendre les mesures nécessaires pour utiliser un coffre de clés différent ou pour désactiver temporairement les clés gérées par le client afin d’éviter ce scénario irrécupérable.

- Si vous avez un coffre de clés, utilisez [az keyvault show](/cli/azure/keyvault#az_keyvault_show) pour répertorier les stratégies d’accès. Pour plus d’informations, consultez [Attribuer une stratégie d’accès Key Vault](../key-vault/general/assign-access-policy-cli.md).

    ```azurecli
    az keyvault show --name MyKeyVault
    ```

### <a name="list-azure-sql-databases-with-azure-ad-authentication"></a>Répertorier les bases de données SQL Azure avec authentification Azure AD

- Utilisez [az sql server ad-admin list](/cli/azure/sql/server/ad-admin#az_sql_server_ad_admin_list) et l’extension [az graph](/cli/azure/ext/resource-graph/graph) pour déterminer si vous utilisez des bases de données Azure AD avec l’intégration de l’authentification Azure AD activée. Pour plus d’informations, consultez [Configurer et gérer l’authentification Azure Active Directory avec SQL](../azure-sql/database/authentication-aad-configure.md).

    ```azurecli
    az sql server ad-admin list --ids $(az graph query -q 'resources | where type == "microsoft.sql/servers" | project id' -o tsv | cut -f1)
    ```

### <a name="list-acls"></a>Répertorier les ACL

1. Si vous utilisez Azure Data Lake Storage Gen1, répertoriez les listes de contrôle d’accès qui sont appliquées à un fichier à l’aide du portail Azure ou de PowerShell.

1. Si vous utilisez Azure Data Lake Storage Gen2, répertoriez les listes de contrôle d’accès qui sont appliquées à un fichier à l’aide du portail Azure ou de PowerShell.

1. Si vous utilisez Azure Files, répertoriez les listes de contrôle d’accès qui sont appliquées à n’importe quel fichier.

### <a name="list-other-known-resources"></a>Répertorier les autres ressources connues

1. Utilisez [az account show](/cli/azure/account#az_account_show) pour obtenir votre ID d’abonnement.

    ```azurecli
    subscriptionId=$(az account show --query id | sed -e 's/^"//' -e 's/"$//')
    ```

1. Utilisez l'extension [az graph](/cli/azure/ext/resource-graph/graph) pour répertorier d’autres ressources Azure avec des dépendances connues au répertoire Azure AD.

    ```azurecli
    az graph query -q \
    'resources | where type != "microsoft.azureactivedirectory/b2cdirectories" | where  identity <> "" or properties.tenantId <> "" or properties.encryptionSettingsCollection.enabled == true | project name, type, kind, identity, tenantId, properties.tenantId' \
    --subscriptions $subscriptionId --output table
    ```

## <a name="step-2-transfer-the-subscription"></a>Étape 2 : Transférer l’abonnement

Dans cette étape, vous transférez l’abonnement de l’annuaire source vers l’annuaire cible. Les étapes sont différentes si vous souhaitez également transférer la propriété de facturation.

> [!WARNING]
> Lorsque vous transférez l’abonnement, toutes les attributions de rôles dans l’annuaire source sont **définitivement** supprimées et ne peuvent pas être restaurées. Vous ne pouvez pas revenir en arrière une fois que vous avez transféré l’abonnement. Veillez à effectuer les étapes précédentes avant d’effectuer cette étape.

1. Déterminez si vous souhaitez également transférer la propriété de facturation à un autre compte.

1. Transférer l’abonnement vers un autre annuaire.

    - Si vous voulez conserver la propriété de facturation actuelle, effectuez les étapes décrites dans [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).
    - Si vous souhaitez également transférer la propriété de facturation, effectuez les étapes indiquées dans [Transférer la propriété de facturation d’un abonnement Azure à un autre compte](../cost-management-billing/manage/billing-subscription-transfer.md). Pour transférer l’abonnement vers un autre annuaire, vous devez cocher la case **Locataire Azure AD de l’abonnement**.

1. Une fois que vous avez terminé le transfert de l’abonnement, revenez à cet article pour recréer les ressources dans l’annuaire cible.

## <a name="step-3-re-create-resources"></a>Étape 3 : Recréer les ressources

### <a name="sign-in-to-target-directory"></a>Se connecter à l’annuaire cible

1. Dans l’annuaire cible, connectez-vous en tant qu’utilisateur qui a accepté la demande de transfert.

    Seul l’utilisateur dans le nouveau compte qui accepte la demande de transfert a accès à la gestion des ressources.

1. Obtenez une liste de vos abonnements à l’aide de la commande [az account list](/cli/azure/account#az_account_list).

    ```azurecli
    az account list --output table
    ```

1. Utilisez [az account set](/cli/azure/account#az_account_set) pour définir l’abonnement actif que vous souhaitez utiliser.

    ```azurecli
    az account set --subscription "Contoso"
    ```

### <a name="create-custom-roles"></a>Créer des rôles personnalisées
        
- Utilisez [az role definition create](/cli/azure/role/definition#az_role_definition_create) pour créer chaque rôle personnalisé à partir des fichiers que vous avez créés précédemment. Pour plus d’informations, consultez [Créer ou mettre à jour des rôles personnalisés Azure à l’aide d’Azure CLI](custom-roles-cli.md).

    ```azurecli
    az role definition create --role-definition <role_definition>
    ```

### <a name="create-role-assignments"></a>Créer des attributions de rôles

- Utilisez [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) pour créer les attributions de rôle pour les utilisateurs, les groupes et les principaux de service. Pour plus d'informations, consultez [Ajouter ou supprimer des attributions de rôles à l'aide du RBAC Azure et d’Azure CLI](role-assignments-cli.md).

    ```azurecli
    az role assignment create --role <role_name_or_id> --assignee <assignee> --resource-group <resource_group>
    ```

### <a name="update-system-assigned-managed-identities"></a>Mettre à jour les identités managées attribuées par le système

1. Désactivez et réactivez les identités managées affectées par le système.

    | Service Azure | Informations complémentaires | 
    | --- | --- |
    | Machines virtuelles | [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#system-assigned-managed-identity) |
    | Groupes identiques de machines virtuelles | [Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vmss.md#system-assigned-managed-identity) |
    | Autres services | [Services prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md) |

1. Utilisez [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) pour créer les attributions de rôle pour les identités managées affectées par le système. Pour plus d’informations, consultez [Affecter à une identité managée l’accès à une ressource à l’aide d’Azure CLI](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).

    ```azurecli
    az role assignment create --assignee <objectid> --role '<role_name_or_id>' --scope <scope>
    ```

### <a name="update-user-assigned-managed-identities"></a>Mettre à jour les identités managées affectées par l'utilisateur

1. Supprimer, recréer et attacher des identités managées affectées par l’utilisateur.

    | Service Azure | Informations complémentaires | 
    | --- | --- |
    | Machines virtuelles | [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) |
    | Groupes identiques de machines virtuelles | [Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vmss.md#user-assigned-managed-identity) |
    | Autres services | [Services prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)<br/>[Créer, répertorier ou supprimer une identité managée affectée par l’utilisateur en utilisant Azure CLI](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) |

1. Utilisez [az role assignment create](/cli/azure/role/assignment#az_role_assignment_create) pour créer les attributions de rôle pour les identités managées affectées par l’utilisateur. Pour plus d’informations, consultez [Affecter à une identité managée l’accès à une ressource à l’aide d’Azure CLI](../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).

    ```azurecli
    az role assignment create --assignee <objectid> --role '<role_name_or_id>' --scope <scope>
    ```

### <a name="update-key-vaults"></a>Mettre à jour des coffres de clés

Cette section décrit les étapes de base pour mettre à jour vos coffres de clés. Pour plus d’informations, consultez [Déplacement d’un coffre Azure Key Vault vers un nouvel abonnement](../key-vault/general/move-subscription.md).

1. Mettez à jour l’ID de locataire associé à tous les coffres de clés existants dans l’abonnement sur le répertoire cible.

1. supprimer toutes les entrées de stratégie d’accès existantes ;

1. Ajouter de nouvelles entrées de stratégie d’accès associées au répertoire cible.

### <a name="update-acls"></a>Mettre à jour les ACL

1. Si vous utilisez Azure Data Lake Storage Gen1, affectez les listes de contrôle d’accès appropriées. Pour plus d’informations, consultez [Sécurisation des données stockées dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-secure-data.md).

1. Si vous utilisez Azure Data Lake Storage Gen2, affectez les listes de contrôle d’accès appropriées. Pour plus d’informations, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-access-control.md).

1. Si vous utilisez Azure Files, affectez les listes de contrôle d’accès appropriées.

### <a name="review-other-security-methods"></a>Réviser les autres méthodes de sécurité

Même si les attributions de rôles sont supprimées pendant le transfert, les utilisateurs du compte du propriétaire d’origine peuvent continuer à avoir accès à l’abonnement par le biais d’autres méthodes de sécurité, notamment :

- Touches d’accès rapide pour les services tels que Storage.
- [Certificats de gestion](../cloud-services/cloud-services-certs-create.md) accordant à l’utilisateur l’accès administrateur sur les ressources d’abonnement.
- Informations d’identification d’accès à distance pour les services tels que les machines virtuelles Azure.

Si vous avez l’intention de supprimer l’accès des utilisateurs dans l’annuaire source afin qu’ils n’aient pas accès à l’annuaire cible, vous devez envisager de faire pivoter les informations d’identification. Tant que les informations d’identification ne sont pas mises à jour, les utilisateurs continueront à avoir accès après le transfert.

1. Faites pivoter les clés d’accès au compte de stockage. Pour plus d’informations, consultez [Gérer les clés d’accès au compte de stockage](../storage/common/storage-account-keys-manage.md).

1. Si vous utilisez des clés d’accès pour d’autres services, comme Azure SQL Database ou la messagerie Azure Service Bus, faites pivoter les clés d’accès.

1. Pour les ressources qui utilisent des secrets, ouvrez les paramètres de la ressource et mettez à jour la clé secrète.

1. Pour les ressources qui utilisent des certificats, mettez à jour le certificat.

## <a name="next-steps"></a>Étapes suivantes

- [Transfert de la propriété de facturation d’un abonnement Azure à un autre compte](../cost-management-billing/manage/billing-subscription-transfer.md)
- [Transférer des abonnements Azure entre des abonnés et des CSP](../cost-management-billing/manage/transfer-subscriptions-subscribers-csp.md)
- [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)
- [Azure Lighthouse dans les scénarios d’entreprise](../lighthouse/concepts/enterprise.md)