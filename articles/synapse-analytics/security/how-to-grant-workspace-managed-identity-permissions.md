---
title: Accorder des autorisations à l’identité managée dans l’espace de travail Synapse
description: Cet article explique comment octroyer des autorisations pour l'identité managée dans l’espace de travail Azure Synapse.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 04/15/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 46fe27205a2c30d6cb2319bf620c6fe1ee34c31e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459007"
---
# <a name="grant-permissions-to-workspace-managed-identity"></a>Octroyer des autorisations à une identité managée de l’espace de travail

Cet article vous explique comment octroyer des autorisations pour l’identité managée dans l’espace de travail Azure Synapse. Les autorisations permettent, à leur tour, d’accéder aux pools SQL dédiés de l’espace de travail et au compte de stockage ADLS Gen2 via le portail Azure.

>[!NOTE]
>Cette identité managée de l’espace de travail sera appelée « identité managée » dans le reste de ce document.

## <a name="grant-managed-identity-permissions-to-the-dedicated-sql-pool"></a>Accorder des autorisations d’identité managée au pool SQL dédié

L’identité managée octroie des autorisations aux pools SQL dédiés de l’espace de travail. Une fois les autorisations octroyées, vous pouvez orchestrer des pipelines qui effectuent des activités en lien avec les pools SQL dédiés. Lorsque vous créez un espace de travail Azure Synapse à l’aide du portail Azure, vous pouvez accorder les autorisations CONTROL d’identité managée sur les pools SQL dédiés.

Sélectionnez **Sécurité** lors de la création de votre espace de travail Azure Synapse. Sélectionnez ensuite **Allow pipelines (running as workspace's system assigned identity) to access SQL pools** (Autoriser les pipelines [s’exécutant en tant qu’identité affectée par le système de l’espace de travail] pour accéder aux pools SQL).

![Autorisation CONTROL sur les pools SQL dédiés](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-16.png)

## <a name="grant-the-managed-identity-permissions-to-adls-gen2-storage-account"></a>Octroyer les autorisations de l'identité managée au compte de stockage ADLS Gen2

Un compte de stockage ADLS Gen2 est nécessaire pour créer un espace de travail Azure Synapse. Pour correctement lancer les pools Spark dans l’espace de travail Azure Synapse, l’identité managée Azure Synapse doit disposer du rôle *Contributeur aux données Blob du stockage* sur ce compte de stockage. L’orchestration de pipeline dans Azure Synapse tire également parti de ce rôle.

### <a name="grant-permissions-to-managed-identity-during-workspace-creation"></a>Octroyer des autorisations pour l'identité managée lors de la création de l'espace de travail

Azure Synapse tente d'octroyer le rôle Contributeur aux données Blob du stockage à l’identité managée une fois l'espace de travail Azure Synapse créé à l’aide du portail Azure. Vous fournissez les détails du compte de stockage ADLS Gen2 dans l’onglet **Informations de base**.

![Onglet Informations de base du flux de création de l'espace de travail](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-1.png)

Sélectionnez le compte de stockage ADLS Gen2 et le système de fichiers dans **Nom du compte** et **Nom du système de fichiers**.

![Détails d'un compte de stockage ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-2.png)

Si le créateur de l’espace de travail est également **propriétaire** du compte de stockage ADLS Gen2, Azure Synapse affecte le rôle *Contributeur aux données Blob du stockage* à l’identité managée. Le message suivant s’affiche sous les détails du compte de stockage que vous avez entrés.

![Affectation du rôle Contributeur aux données Blob du stockage réussie](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-3.png)

Si le créateur de l’espace de travail n’est pas propriétaire du compte de stockage ADLS Gen2, Azure Synapse n’affecte pas le rôle *Contributeur aux données Blob du stockage* à l’identité managée. Le message qui s’affiche sous les détails du compte de stockage informe le créateur de l’espace de travail qu’il ne dispose pas d'autorisations suffisantes pour octroyer le rôle *Contributeur aux données Blob du stockage* à l’identité managée.

![Affectation du rôle Contributeur aux données Blob du stockage non réussie](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-4.png)

Comme l’indique le message, vous ne pouvez pas créer de pools Spark tant que le rôle *Contributeur aux données Blob du stockage* n'est pas affecté à l’identité managée.

### <a name="grant-permissions-to-managed-identity-after-workspace-creation"></a>Octroyer des autorisations pour l'identité managée après création de l'espace de travail

Lors de la création de l’espace de travail, si vous n’affectez pas le rôle *Contributeur aux données Blob du stockage* à l’identité managée, le **propriétaire** du compte de stockage ADLS Gen2 s'en charge manuellement. Les étapes suivantes vous aideront à procéder à cette affectation manuellement.

#### <a name="step-1-navigate-to-the-adls-gen2-storage-account-in-azure-portal"></a>Étape 1 : Accéder au compte de stockage ADLS Gen2 dans le Portail Azure

Dans le Portail Microsoft Azure, ouvrez le compte de stockage ADLS Gen2 et sélectionnez **Vue d’ensemble** dans le volet de navigation gauche. Vous devez affecter le rôle *Contributeur aux données Blob du stockage* au niveau du conteneur ou du système de fichiers. Sélectionnez **Conteneurs**.  
![Vue d'ensemble du compte de stockage ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-5.png)

#### <a name="step-2-select-the-container"></a>Étape 2 : Sélection du conteneur

L’identité managée doit pouvoir accéder aux données du conteneur (système de fichiers) fourni lors de la création de l’espace de travail. Ce conteneur ou système de fichiers est disponible dans le portail Azure. Ouvrez l'espace de travail Azure Synapse dans le portail Azure et sélectionnez l'onglet **Vue d’ensemble** dans le volet de navigation gauche.
![Conteneur de comptes de stockage ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-7.png)


Sélectionnez le même conteneur ou système de fichiers pour octroyer le rôle *Contributeur aux données Blob du stockage* à l'identité managée.
![Capture d’écran montrant le conteneur ou système de fichiers que vous devez sélectionner.](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-6.png)

#### <a name="step-3-navigate-to-access-control"></a>Étape 3 : Accéder au contrôle d’accès

Sélectionnez **Contrôle d’accès (IAM)** .

![Contrôle d’accès (IAM)](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-8.png)

#### <a name="step-4-add-a-new-role-assignment"></a>Étape 4 : Ajouter une attribution de rôle

Sélectionnez **Ajouter**.

![Ajouter une attribution de rôle](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-9.png)

#### <a name="step-5-select-the-azure-role"></a>Étape 5 : Sélectionner le rôle Azure

Sélectionnez le rôle **Contributeur aux données Blob du stockage**.

![Sélectionner le rôle Azure](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-10.png)

#### <a name="step-6-select-the-azure-ad-security-principal"></a>Étape 6 : Sélectionner le principal de sécurité Azure AD

Sélectionnez **Utilisateur, groupe ou principal de service Azure AD** dans la liste déroulante **Attribuer l'accès à**.

![Sélectionner le principal de sécurité AAD](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-11.png)

#### <a name="step-7-search-for-the-managed-identity"></a>Étape 7 : Rechercher l’identité managée

Le nom de l’identité managée correspond également au nom de l’espace de travail. Recherchez votre identité managée en entrant le nom de l’espace de travail Azure Synapse dans **Sélectionner**. L'identité managée doit y être répertoriée.

![Rechercher l’identité managée](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-12.png)

#### <a name="step-8-select-the-managed-identity"></a>Étape 8 : Sélectionner l’identité managée

Sélectionnez l’identité managée pour **Membres sélectionnés**. Sélectionnez **Enregistrer** pour ajouter l’attribution de rôle.

![Sélectionner l’identité managée](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-13.png)

#### <a name="step-9-verify-that-the-storage-blob-data-contributor-role-is-assigned-to-the-managed-identity"></a>Étape 9 : Vérifiez que le rôle Contributeur aux données Blob du stockage est attribué à l’identité managée.

Sélectionnez **Contrôle d’accès (IAM)** , puis **Attributions de rôle**.

![Vérifier l'attribution de rôle](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-14.png)

Votre identité managée doit être répertoriée sous la section **Contributeur aux données Blob du stockage** et le rôle *Contributeur aux données Blob du stockage* doit lui avoir été attribué. 
![Sélection du conteneur de comptes de stockage ADLS Gen2](./media/how-to-grant-workspace-managed-identity-permissions/configure-workspace-managed-identity-15.png)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l'[identité managée de l'espace de travail](./synapse-workspace-managed-identity.md)
