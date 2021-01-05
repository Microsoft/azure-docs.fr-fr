---
title: Configurer des identités managées sur un groupe de machines virtuelles identiques – Azure AD
description: Instructions détaillées pour configurer des identités managées pour des ressources Azure sur un groupe de machines virtuelles identiques en utilisant le portail Azure.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: 73c2f4167d6ce5465a879f6b2ef7e96d91065c4d
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590917"
---
# <a name="configure-managed-identities-for-azure-resources-on-a-virtual-machine-scale-set-vmss-using-the-azure-portal"></a>Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques (VMSS) en utilisant le portail Azure

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent des services Azure avec une identité managée automatiquement dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code. 

Cet article explique comment effectuer les opérations d’identités managées pour ressources Azure suivantes sur un groupe de machines virtuelles identiques en utilisant PowerShell :

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md).
- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.
- Pour effectuer les opérations de gestion dans cet article, votre compte doit disposer des attributions des rôles Azure suivants :

    > [!NOTE]
    > Aucune attribution de rôle d'annuaire Azure AD supplémentaire n’est requise.

    - [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) pour activer et supprimer l’identité managée affectée par le système à partir d’un groupe de machines virtuelles identiques.

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, vous allez découvrir comment activer et désactiver l’identité managée affectée par le système en utilisant le portail Azure.

### <a name="enable-system-assigned-managed-identity-during-creation-of-a-virtual-machine-scale-set"></a>Activer une identité managée affectée par le système lors de la création d’un groupe de machines virtuelles identiques

Actuellement, le portail Azure ne prend pas en charge l’activation de l’identité managée attribuée par le système lors de la création d’un groupe de machines virtuelles identiques. Au lieu de cela, consultez l’article de démarrage rapide suivant sur la création de groupes de machines virtuelles identiques pour commencer par créer un groupe de machines virtuelles identiques, puis passez à la section suivante pour plus d’informations sur l’activation de l’identité managée attribuée par le système sur un groupe de machines virtuelles identiques :

- [Créer un groupe de machines virtuelles identiques dans le portail Azure](../../virtual-machine-scale-sets/quick-create-portal.md)  

### <a name="enable-system-assigned-managed-identity-on-an-existing-virtual-machine-scale-set"></a>Activer une identité managée affectée par le système sur un groupe de machines virtuelles identiques existant

Pour activer l’identité managée affectée par le système sur un groupe de machines virtuelles identiques ayant été initialement configuré sans :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques.

2. Accédez au groupe de machines virtuelles identiques souhaité.

3. Sous **Attribuée par le système**, **État**, sélectionnez **Activé** puis cliquez sur **Enregistrer** :

   ![La capture d’écran montre la page « Identité (préversion) » avec l’option « Affectée par le système » sélectionnée, l’état « Activé » et le bouton « Enregistrer » mis en évidence.](./media/msi-qs-configure-portal-windows-vmss/create-windows-vmss-portal-configuration-blade.png) 

### <a name="remove-system-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>Supprimer une identité managée affectée par le système d’un groupe de machines virtuelles identiques

Si vous disposez d’un groupe de machines virtuelles identiques qui n’a plus besoin d’une identité managée affectée par le système :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques. Vérifiez également que votre compte appartient à un rôle qui vous donne des autorisations en écriture sur le groupe de machines virtuelles identiques.

2. Accédez au groupe de machines virtuelles identiques souhaité.

3. Sous **Attribuée par le système**, **État**, sélectionnez **Désactivé** puis cliquez sur **Enregistrer** :

   ![Capture d’écran de la page Configuration](./media/msi-qs-configure-portal-windows-vmss/disable-windows-vmss-portal-configuration-blade.png)

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, vous découvrez comment ajouter et supprimer une identité managée attribuée par l’utilisateur à partir d’un groupe de machines virtuelles identiques en utilisant le portail Azure.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-a-virtual-machine-scale-set"></a>Attribuer une identité managée affectée par l’utilisateur lors de la création d’un groupe de machines virtuelles identiques

Actuellement, le portail Azure ne prend pas en charge l’attribution d’une identité managée attribuée par l’utilisateur lors de la création d’un ensemble de machines virtuelles identiques. Au lieu de cela, consultez l’article de démarrage rapide suivant sur la création d’un groupe de machines virtuelles identiques pour commencer par créer un groupe de machines virtuelles identiques, puis passez à la section suivante pour plus d’informations sur l’attribution d’une identité managée affectée par l’utilisateur à ce groupe  :

- [Créer un groupe de machines virtuelles identiques dans le portail Azure](../../virtual-machine-scale-sets/quick-create-portal.md)

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-virtual-machine-scale-set"></a>Attribuer une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques existant

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte associé à l’abonnement Azure qui contient le groupe de machines virtuelles identiques.
2. Accédez au groupe de machines virtuelles identiques souhaité, cliquez sur **Identité**, **Attribuée par l’utilisateur**, puis sur **\+Ajouter**.

   ![Capture d’écran montrant la page « Identité » avec l’option « Attribuée par l’utilisateur » sélectionnée et le bouton « Ajouter » mis en évidence.](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vmss-screenshot1.png)

3. Cliquez sur l’identité affectée par l’utilisateur que vous souhaitez ajouter au groupe de machines virtuelles identiques, puis cliquez sur **Ajouter**.
   
   ![Ajouter une identité affectée par l’utilisateur à un groupe de machines virtuelles identiques](./media/msi-qs-configure-portal-windows-vm/add-user-assigned-identity-vm-screenshot2.png)

### <a name="remove-a-user-assigned-managed-identity-from-a-virtual-machine-scale-set"></a>Supprimer une identité managée affectée par l’utilisateur d’un groupe de machines virtuelles identiques

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte associé à l’abonnement Azure qui contient la machine virtuelle.
2. Accédez au groupe de machines virtuelles identiques souhaité, cliquez sur **Identité**, puis sur **Affecté(e) par l'utilisateur**. Cliquez ensuite sur le nom de l’identité managée attribuée par l’utilisateur que vous souhaitez supprimer, puis sur **Supprimer** (cliquez sur **Oui** dans le volet de confirmation).

   ![Supprimer l’identité affectée par l’utilisateur d’un groupe de machines virtuelles identiques](./media/msi-qs-configure-portal-windows-vm/remove-user-assigned-identity-vmss-screenshot.png)


## <a name="next-steps"></a>Étapes suivantes

- Dans le portail Azure, accordez à l’identité managée d’un groupe de machines virtuelles identiques Azure l’[accès à une autre ressource Azure](howto-assign-access-portal.md).
