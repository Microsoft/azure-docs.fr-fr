---
title: Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données
titleSuffix: Azure Storage
description: Découvrez comment utiliser le Portail Azure pour attribuer des autorisations à un principal de sécurité Azure Active Directory avec le contrôle d’accès Azure en fonction du rôle (Azure RBAC). Stockage Azure prend en charge les rôles personnalisés Azure et intégrés pour l’authentification par le biais d’Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 40003db9c3bd2c736f9cedd73b8b7a31a77f625f
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854578"
---
# <a name="use-the-azure-portal-to-assign-an-azure-role-for-access-to-blob-and-queue-data"></a>Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente

Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Stockage Azure définit un ensemble de rôles intégrés Azure qui englobent les ensembles communs d’autorisations permettant d’accéder aux données blob ou de file d’attente.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, du compte de stockage ou d’un conteneur ou d’une file d’attente individuelle. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Cet article explique comment utiliser le portail Azure pour attribuer des rôles Azure. Le portail Azure fournit une interface simple pour attribuer des rôles Azure et gérer l’accès à vos ressources de stockage. Vous pouvez également attribuer des rôles Azure pour les ressources de blobs et de files d’attente à l’aide des outils de ligne de commande Azure ou des API de gestion Stockage Azure. Pour plus d’informations sur les rôles Azure pour les ressources de stockage, consultez [Authentifier l’accès aux objets blob et aux files d’attente Azure à l’aide d’Azure Active Directory](storage-auth-aad.md).

## <a name="azure-roles-for-blobs-and-queues"></a>Rôles Azure pour blobs et files d’attente

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Déterminer l’étendue de la ressource

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="assign-azure-roles-using-the-azure-portal"></a>Attribuer des rôles Azure à l’aide du portail Azure

Après avoir déterminé l’étendue appropriée pour une attribution de rôle, accédez à cette ressource dans le portail Azure. Affichez les paramètres **Contrôle d’accès (IAM)** pour la ressource et suivez ces instructions pour gérer les attributions de rôle :

1. Attribuez le rôle Azure Stockage Azure approprié pour accorder l’accès à un principal de sécurité Azure AD.

1. Attribuez le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) Azure Resource Manager aux utilisateurs qui ont besoin d’accéder aux conteneurs ou files d’attente via le portail Azure à l’aide de leurs informations d’identification Azure AD.

Les sections suivantes décrivent chacune de ces étapes plus en détail.

> [!NOTE]
> Lorsque vous créez un compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée via Azure AD. Vous devez vous attribuer explicitement un rôle Azure pour le Stockage Azure. Vous pouvez l’attribuer au niveau de votre abonnement, groupe de ressources, compte de stockage, conteneur ou file d’attente.
>
> Avant de vous attribuer un rôle pour l’accès aux données, vous pouvez accéder aux données de votre compte de stockage via le portail Azure, car ce dernier peut également utiliser la clé de compte pour l’accès aux données. Pour plus d’informations, consultez [Choisir comment autoriser l’accès à des données blobs dans le portail Azure](../blobs/authorize-data-operations-portal.md).

### <a name="assign-an-azure-built-in-role"></a>Attribuer un rôle intégré Azure

Avant d’attribuer un rôle à un principal de sécurité, veillez à tenir compte de l’étendue des autorisations que vous accordez. Passez en revue la section [Déterminer l’étendue de la ressource](#determine-resource-scope) pour décider de l’étendue appropriée.

La procédure illustrée ici attribue un rôle limité à un conteneur, mais vous pouvez suivre les mêmes étapes pour attribuer un rôle limité à une file d’attente :

1. Dans le [Portail Azure](https://portal.azure.com), accédez à votre compte de stockage, puis affichez l’**Aperçu** du compte.
1. Sous Services, sélectionnez **Blobs**.
1. Recherchez le conteneur pour lequel vous souhaitez attribuer un rôle, puis affichez les paramètres de ce conteneur.
1. Sélectionnez **Contrôle d’accès (IAM)** pour afficher les paramètres de contrôle d’accès du conteneur. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles.

    :::image type="content" source="media/storage-auth-aad-rbac-portal/portal-access-control-container.png" alt-text="Capture d’écran montrant les paramètres de contrôle d’accès du conteneur":::

1. Cliquez sur le bouton **Ajouter une attribution de rôle** pour ajouter un nouveau rôle.
1. Dans la fenêtre **Ajouter une attribution de rôle**, sélectionnez le rôle Stockage Azure que vous souhaitez attribuer. Recherchez ensuite le principal de sécurité auquel vous souhaitez attribuer ce rôle.

    :::image type="content" source="media/storage-auth-aad-rbac-portal/add-rbac-role.png" alt-text="Capture d’écran montrant comment attribuer un rôle Azure":::

1. Cliquez sur **Enregistrer**. L’identité à laquelle vous avez attribué le rôle apparaît sous ce dernier. Par exemple, l’image suivante montre que l’utilisateur ajouté a désormais des autorisations d’accès en lecture pour toutes les données du conteneur nommé *sample-container*.

    :::image type="content" source="media/storage-auth-aad-rbac-portal/container-scoped-role.png" alt-text="Capture d’écran montrant la liste des utilisateurs auxquels un rôle a été attribué":::

Vous pouvez suivre des étapes similaires pour attribuer un rôle dans l’étendue du compte de stockage, du groupe de ressources ou de l’abonnement.

### <a name="assign-the-reader-role-for-portal-access"></a>Attribuer le rôle Lecteur pour accéder au portail

Lorsque vous attribuez un rôle intégré ou personnalisé pour Stockage Azure à un principal de sécurité, vous accordez des autorisations à ce principal de sécurité pour effectuer des opérations sur les données dans votre compte de stockage. Les rôles **Lecteur de données** intégrés fournissent des autorisations de lecture pour les données dans un conteneur ou une file d’attente, tandis que les rôles **Contributeur aux données** intégrés permettent de lire, d’écrire et de supprimer des autorisations dans un conteneur ou une file d’attente. L’étendue des autorisations se limite à la ressource spécifiée.  
Par exemple, si vous attribuez le rôle **Contributeur aux données Blob du stockage** à l’utilisatrice Mary au niveau d’un conteneur nommé **exemple-container**, puis Mary se voit attribuer un accès en lecture, écriture et suppression à tous les objets blob dans ce conteneur.

Toutefois, si Mary souhaite afficher un objet blob dans le portail Azure, le rôle **Contributeur aux données Blob du stockage** ne fournit pas les autorisations suffisantes pour naviguer dans le portail et atteindre l’objet blob dans le but de le consulter. D’autres autorisations Azure AD sont requises pour naviguer dans le portail et afficher les autres ressources qui y sont visibles.

Si vos utilisateurs doivent être en mesure d’accéder aux blobs dans le portail Azure, attribuez-leur un rôle Azure supplémentaire, le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader), au niveau du compte de stockage ou au-dessus. Le rôle **Lecteur** est un rôle d’Azure Resource Manager qui permet aux utilisateurs d’afficher les ressources de compte de stockage, mais pas de les modifier. Il ne fournit pas d’autorisations en lecture pour les données dans Stockage Azure, mais uniquement pour les ressources de gestion de compte.

Suivez ces étapes pour attribuer le rôle **Lecteur** afin qu’un utilisateur puisse accéder aux objets blob à partir du portail Azure. Dans cet exemple, l’attribution est étendue au compte de stockage :

1. Dans le [Portail Azure](https://portal.azure.com), accédez à votre compte de stockage.
1. Sélectionnez **Contrôle d’accès (IAM)** pour afficher les paramètres de contrôle d’accès du compte de stockage. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles.
1. Dans la fenêtre **Ajouter une attribution de rôle**, sélectionnez le rôle **Lecteur**. 
1. Dans le champ **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou principal du service Azure AD**.
1. Recherchez le principal de sécurité auquel vous souhaitez attribuer le rôle.
1. Enregistrez l’attribution de rôle.

L’attribution du rôle **Lecteur** est uniquement nécessaire pour les utilisateurs qui doivent accéder aux blobs ou files d’attente à l’aide du Portail Azure.

> [!IMPORTANT]
> La préversion d’Explorateur Stockage dans le Portail Azure ne prend pas en charge l’utilisation d’informations d’identification Azure AD pour afficher et modifier des données blob ou de file d’attente. Explorateur Stockage dans le portail Azure utilise toujours les clés de compte pour accéder aux données. Pour utiliser Explorateur Stockage dans le Portail Azure, vous devez être affecté à un rôle qui comprend **Microsoft.Storage/storageAccounts/listkeys/action**.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les rôles Azure pour les ressources de stockage, consultez [Authentifier l’accès aux objets blob et aux files d’attente Azure à l’aide d’Azure Active Directory](storage-auth-aad.md). 
- Pour plus d’informations sur Azure RBAC, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md).
- Pour apprendre à attribuer et gérer les rôles Azure avec Azure PowerShell, Azure CLI ou l’API REST, consultez les articles suivants :
    - [Ajouter ou supprimer des attributions de rôles Azure à l’aide du module Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
    - [Ajouter ou supprimer des attributions de rôles Azure à l’aide de l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md)
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](../../role-based-access-control/role-assignments-rest.md)
- Pour apprendre à autoriser l’accès aux conteneurs et files d’attente à partir de vos applications de stockage, consultez [Utiliser Azure AD avec les applications de stockage Azure](storage-auth-aad-app.md).
