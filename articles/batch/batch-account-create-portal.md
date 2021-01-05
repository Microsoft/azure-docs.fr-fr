---
title: Créer un compte dans le portail Azure
description: Apprenez à créer un compte Azure Batch dans le portail Azure pour exécuter des charges de travail parallèles à grande échelle dans le cloud
ms.topic: how-to
ms.date: 06/10/2020
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1205de2b800588b735aeb20d388ba4b64bc6b078
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84711338"
---
# <a name="create-a-batch-account-with-the-azure-portal"></a>Créer un compte Batch avec le portail Azure

Cette rubrique montre comment créer un compte Azure Batch dans le [portail Azure](https://portal.azure.com), en choisissant les propriétés du compte qui correspondent à votre scénario de calcul. Vous allez également découvrir où se trouvent les propriétés du compte importantes, telles que les clés d’accès et les URL du compte.

Pour plus d’informations sur les comptes et les scénarios Batch, consultez [Workflow et ressources du service Batch](batch-service-workflow-features.md).

## <a name="create-a-batch-account"></a>Création d’un compte Batch

[!INCLUDE [batch-account-mode-include](../../includes/batch-account-mode-include.md)]

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource**, puis sélectionnez **Calcul** et **Service Batch**.

1. Entrez les paramètres **Nouveau compte Batch**. Consultez les informations suivantes :

    ![Création d’un compte Batch][account_portal]

    a. **Abonnement**: Abonnement où créer le compte Batch. Si vous n’avez qu’un seul abonnement, il est sélectionné par défaut.

    b. **Groupe de ressources** : Sélectionnez un groupe de ressources existant pour votre nouveau compte Batch ; vous pouvez aussi en créer un.

    c. **Nom du compte** : Le nom que vous choisissez doit être unique dans la région Azure où le compte est créé (voir **Emplacement** ci-dessous). Le nom de compte peut contenir uniquement des caractères minuscules ou des chiffres et doit comporter entre 3 et 24 caractères.

    d. **Emplacement** : Région Azure où créer le compte Batch. Seules les régions prises en charge par votre abonnement et votre groupe de ressources sont affichées.

    e. **Compte de stockage** : Compte de Stockage Azure facultatif à associer à votre compte Batch. Un compte de stockage v2 universel est recommandé pour obtenir des performances optimales. Pour connaître les options de compte de stockage de Batch, consultez la [vue d’ensemble des fonctionnalités d’Azure Batch](accounts.md#azure-storage-accounts). Dans le portail, sélectionnez un compte de stockage existant ou créez-en un nouveau.

      ![Créez un compte de stockage.][storage_account]

    f. **Mode d’allocation de pool** : Sous l’onglet des paramètres **Avancé**, vous pouvez spécifier le mode d’allocation de pool **Service Batch** ou **Abonnement utilisateur**. Pour la plupart des scénarios, acceptez le **service Batch** par défaut.

      ![Mode d’allocation de pool Batch][pool_allocation]

1. Sélectionnez **Créer** pour créer le compte.

## <a name="view-batch-account-properties"></a>Afficher les propriétés du compte Batch

Une fois le compte créé, cliquez dessus pour accéder à ses propriétés et paramètres. Vous avez accès à tous les paramètres et propriétés du compte dans le menu de gauche.

> [!NOTE]
> Le nom du compte Batch correspond à son ID et ne peut pas être modifié. Si vous avez besoin de modifier le nom d’un compte Batch, vous devez supprimer le compte et en créer un nouveau avec le nom souhaité.

![Page du compte Batch dans le portail Azure][account_blade]

* **Nom, URL et clés du compte Batch** : Quand vous développez une application avec des [API Batch](batch-apis-tools.md#azure-accounts-for-batch-development), vous avez besoin d’une URL et d’une clé de compte pour accéder aux ressources Batch. Batch prend également en charge l’authentification Azure Active Directory.

    Pour afficher les informations d’accès du compte Batch, sélectionnez **Clés**.

    ![Clés de compte Batch dans le portail Azure][account_keys]

* Pour afficher le nom et les clés du compte de stockage associé à votre compte Batch, sélectionnez **Compte de stockage**.

* Pour afficher les quotas de ressources qui s’appliquent au compte Batch, sélectionnez **Quotas**. Pour plus de détails, consultez [Quotas et limites du service Batch](batch-quota-limit.md).

## <a name="additional-configuration-for-user-subscription-mode"></a>Configuration supplémentaire pour le mode d’abonnement utilisateur

Si vous choisissez de créer un compte Batch en mode d’abonnement utilisateur, suivez les étapes supplémentaires ci-dessous avant de créer le compte.

### <a name="allow-azure-batch-to-access-the-subscription-one-time-operation"></a>Autoriser Azure Batch à accéder à l’abonnement (opération ponctuelle)

Quand vous créez votre premier compte Batch en mode Abonnement utilisateur, vous devez inscrire votre abonnement auprès de Batch. (Si vous l’avez déjà fait, passez à la section suivante.)

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Tous les services** > **Abonnements**, puis cliquez sur l’abonnement que vous souhaitez utiliser pour le compte Batch.

1. Dans la page **Abonnement**, sélectionnez **Fournisseurs de ressources**et recherchez **Microsoft.Batch**. Vérifiez que le fournisseur de ressources **Microsoft.Batch** est enregistré dans l’abonnement. S’il n’est pas enregistré, cliquez sur le lien **Inscrire**.

    ![Inscrire le fournisseur Microsoft.Batch][register_provider]

1. Dans la page **Abonnement**, sélectionnez **Contrôle d’accès (IAM)**  > **Attributions de rôles** > **Ajouter une attribution de rôle**.

    ![Contrôle d’accès à l’abonnement][subscription_access]

1. Dans la page **Ajouter une attribution de rôle**, sélectionnez le rôle **Contributeur** ou **Propriétaire**, puis recherchez l’API Batch. Recherchez chacune de ces chaînes, jusqu’à ce que vous trouviez l’API :
    1. **MicrosoftAzureBatch**.
    1. **Microsoft Azure Batch**. Les locataires Azure AD les plus récents peuvent utiliser ce nom.
    1. La chaîne **ddbf3205-c6bd-46ae-8127-60eb93363864** correspond à l’ID de l’API Batch.

1. Une fois que vous avez trouvé l’API Batch, sélectionnez-la, puis cliquez sur **Enregistrer**.

    ![Ajouter des autorisations Batch][add_permission]

### <a name="create-a-key-vault"></a>Créer un coffre de clés

En mode abonnement utilisateur, un [coffre de clés Azure](../key-vault/general/overview.md) est nécessaire. Le coffre de clés doit se trouver dans la même région et le même abonnement que le compte Batch à créer. 

1. Dans le [portail Azure](https://portal.azure.com), cliquez sur **Nouveau** > **Sécurité** > **Key Vault**.

1. Dans la page **Créer un coffre de clés**, entrez un nom pour le coffre de clés et créez un groupe de ressources dans la région souhaitée pour votre compte Batch. Conservez les autres paramètres par défaut, puis cliquez sur **Créer**.

Lors de la création du compte Batch en mode abonnement utilisateur, spécifiez **Abonnement utilisateur** comme mode d’allocation de pool, sélectionnez le coffre de clés, puis cochez la case pour accorder à Azure Batch l’accès au coffre de clés.

Si vous préférez accorder manuellement l’accès au coffre de clés, accédez à la section **Stratégies d’accès** du coffre de clés, sélectionnez **Ajouter une stratégie d’accès**, puis recherchez **Microsoft Azure Batch**. Une fois les autorisations sélectionnées sous **Autorisations du secret**, vous devez les configurer à l’aide du menu déroulant. Azure Batch doit avoir au minimum les autorisations **Get**, **List**, **Set** et **Delete**.

![Autorisations de secret pour Azure Batch](./media/batch-account-create-portal/secret-permissions.png)

> [!NOTE]
> Vérifiez que les cases à cocher **Machines virtuelles Azure pour le déploiement** et **Azure Resource Manager pour le déploiement de modèles** sont activées sous **Stratégies d’accès** pour la ressource **Key Vault** liée.
>
> ![Créer une stratégie d’accès au coffre de clés obligatoire](./media/batch-account-create-portal/key-vault-access-policy.png)

### <a name="configure-subscription-quotas"></a>Configurer les quotas d’abonnement

Pour les comptes Batch d’abonnement utilisateur, les quotas de base doivent être définis manuellement. Les quotas de base Batch standard ne s’appliquent pas aux comptes en mode abonnement utilisateur.

1. Dans le [Portail Azure](https://portal.azure.com), sélectionnez votre compte Batch en mode d’abonnement utilisateur pour afficher ses paramètres et ses propriétés.
1. Dans le menu de gauche, sélectionnez **Quotas** pour afficher et configurer les quotas de base associés à votre compte Batch.

Pour plus d’informations sur les quotas de base en mode d’abonnement utilisateur, consultez [Quotas et limites du service Batch](batch-quota-limit.md).

## <a name="other-batch-account-management-options"></a>Autres options de gestion de comptes Batch

Vous pouvez créer et gérer des comptes Batch à l’aide du portail Azure, mais également avec les outils suivants :

* [Applets de commande PowerShell pour Batch](batch-powershell-cmdlets-get-started.md)
* [Azure CLI](batch-cli-get-started.md)
* [Gestion de lots .NET](batch-management-dotnet.md)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md) telles que les pools, les nœuds, les travaux et les tâches.
* Découvrez les bases du développement d’une application Batch à l’aide de la [bibliothèque Azure Batch pour .NET](quick-run-dotnet.md) ou [Python](quick-run-python.md). Ces démarrages rapides décrivent la création d’un exemple d’application qui utilise le service Batch pour exécuter une charge de travail sur plusieurs nœuds de calcul, et inclut l’utilisation d’Azure Storage pour la mise en attente et la récupération de fichiers de la charge de travail.

[account_blade]: ./media/batch-account-create-portal/batch_blade.png
[account_portal]: ./media/batch-account-create-portal/batch-account-portal.png
[pool_allocation]: ./media/batch-account-create-portal/batch-pool-allocation.png
[account_keys]: ./media/batch-account-create-portal/batch-account-keys.png
[storage_account]: ./media/batch-account-create-portal/storage_account.png
[subscription_access]: ./media/batch-account-create-portal/subscription_iam.png
[add_permission]: ./media/batch-account-create-portal/add_permission.png
[register_provider]: ./media/batch-account-create-portal/register_provider.png
