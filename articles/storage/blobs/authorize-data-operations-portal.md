---
title: Choisir comment autoriser l’accès à des données de blobs dans le portail Azure
titleSuffix: Azure Storage
description: Lorsque vous accédez aux données d’objet blob à l’aide du Portail Azure, le portail fait des requêtes à Azure Storage en arrière-plan. Ces requêtes au stockage Azure peuvent être authentifiées et autorisées à l’aide de votre compte Azure AD ou de la clé d’accès au compte de stockage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 09/08/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.custom: contperf-fy21q1
ms.openlocfilehash: 8c963f11a34217253f02cb5d116d66cdbf8bcc19
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033955"
---
# <a name="choose-how-to-authorize-access-to-blob-data-in-the-azure-portal"></a>Choisir comment autoriser l’accès à des données de blobs dans le portail Azure

Lorsque vous accédez aux données d’objet blob à l’aide du [Portail Azure](https://portal.azure.com), le portail fait des requêtes à Azure Storage en arrière-plan. Une requête au stockage Azure peut être autorisée à l’aide de votre compte Azure AD ou de la clé d’accès au compte de stockage. Le portail indique quelle méthode vous utilisez et vous permet de basculer entre les deux si vous disposez des autorisations appropriées.  

Vous pouvez également préciser de quelle manière une opération individuelle de chargement d'objets blob doit être autorisée sur le portail Azure. Par défaut, le portail applique la méthode que vous utilisez déjà pour autoriser une opération de chargement d'objets blob, mais vous pouvez modifier ce paramètre au moment du chargement d'un objet blob.

## <a name="permissions-needed-to-access-blob-data"></a>Autorisations nécessaires pour accéder aux données d’objet blob

Selon la façon dont vous souhaitez autoriser l’accès aux données blob dans le portail Azure, vous devrez disposer d’autorisations particulières. Dans la plupart des cas, ces autorisations sont fournies via le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Pour plus d’informations sur Azure RBAC, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md)

### <a name="use-the-account-access-key"></a>Utiliser la clé d'accès au compte

Pour accéder aux données de blob avec la clé d’accès au compte, vous devez disposer d’un rôle Azure qui vous est attribué et qui inclut l’action Azure RBAC **Microsoft.Storage/storageAccounts/listkeys/action**. Ce rôle Azure peut être intégré ou il peut s’agir d’un rôle personnalisé. Les rôles intégrés qui prennent en charge **Microsoft.Storage/storageAccounts/listkeys/action** incluent :

- Le rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager
- Le rôle [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) d’Azure Resource Manager
- Le rôle [Contributeur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

Lorsque vous tentez d’accéder aux données d’objet blob dans le Portail Microsoft Azure, le portail commence par vérifier si un rôle RBAC vous a été attribué avec **Microsoft.Storage/storageAccounts/listkeys/action**. Si un rôle avec cette action vous a été attribué, le portail utilise la clé de compte pour l’accès aux données d’objet blob. Si un rôle avec cette action ne vous a pas été attribué, le portail tente d’accéder aux données à l’aide de votre compte Azure AD.

> [!NOTE]
> Les rôles d’administrateur d’abonnement classique Administrateur de service et Co-administrateur incluent l’équivalent du rôle [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) d’Azure Resource Manager. Le rôle **Propriétaire** inclut toutes les actions, y compris **Microsoft.Storage/storageAccounts/listkeys/action**, pour qu’un utilisateur avec l’un de ces rôles d’administration puisse accéder également aux données d’objet blob avec la clé de compte. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles).

### <a name="use-your-azure-ad-account"></a>Utiliser votre compte Azure AD

Pour accéder aux données d’objet blob à partir du Portail Microsoft Azure à l’aide de votre compte Azure AD, vous devez remplir ces deux conditions :

- Le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader) Azure Resource Manager vous a été attribué, au moins, limité au niveau du compte de stockage ou à un niveau supérieur. Le rôle **Lecteur** octroie les autorisations les plus restreintes, mais l’utilisation d’un autre rôle Azure Resource Manager accordant l’accès aux ressources de gestion de compte de stockage est également acceptable.
- Un rôle intégré ou personnalisé qui fournit l’accès aux données d’objet blob vous a été attribué.

L’attribution de rôle **Lecteur** ou l’attribution d’un autre rôle Azure Resource Manager est nécessaire pour que l’utilisateur puisse afficher et parcourir les ressources de gestion du compte de stockage dans le Portail Microsoft Azure. Les rôles Azure qui accordent l’accès aux données d’objet blob n’accordent pas d’accès aux ressources de gestion du compte de stockage. Pour accéder aux données d’objet blob dans le portail, l’utilisateur a besoin d’autorisations pour parcourir les ressources de compte de stockage. Pour plus d’informations sur cette exigence, consultez [Attribuer le rôle Lecteur pour accéder au portail](../common/storage-auth-aad-rbac-portal.md#assign-the-reader-role-for-portal-access).

Les rôles intégrés qui prennent en charge les accès à vos données d’objet blob sont les suivants :

- [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) : Pour le contrôle d’accès dans Azure Data Lake Storage Gen2.
- [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor) : Autorisations en lecture/écriture/suppression pour les objets blob.
- [Lecteur des données blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) : Autorisations en lecture seule pour les objets blob.

Les rôles personnalisés peuvent prendre en charge différentes combinaisons des mêmes autorisations fournies par les rôles intégrés. Pour plus d’informations sur la création de rôles Azure personnalisés, consultez [Rôles personnalisés Azure](../../role-based-access-control/custom-roles.md) et [Présentation des définitions de rôles pour les ressources Azure](../../role-based-access-control/role-definitions.md).

> [!IMPORTANT]
> La préversion d’Explorateur Stockage dans le portail Azure ne prend pas en charge l’utilisation d’informations d’identification Azure AD pour visualiser et modifier des données blob. Explorateur Stockage dans le portail Azure utilise toujours les clés de compte pour accéder aux données. Pour utiliser Explorateur Stockage dans le portail Azure, un rôle qui comprend **Microsoft.Storage/storageAccounts/listkeys/action** doit vous être attribué.

## <a name="navigate-to-blobs-in-the-azure-portal"></a>Accédez aux objets blob dans le Portail Azure

Pour afficher les données d’objet blob sur le portail, accédez à la **Vue d’ensemble** pour votre compte de stockage, puis cliquez sur les liens pour **Objet blob**. Vous pouvez également naviguer jusqu’à la section **Service Blob** dans le menu.

:::image type="content" source="media/anonymous-read-access-configure/blob-public-access-portal.png" alt-text="Capture d’écran montrant comment accéder aux données d’objet blob dans le portail Azure":::

## <a name="determine-the-current-authentication-method"></a>Déterminer la méthode d’authentification actuelle

Lorsque vous accédez à un conteneur, le portail Azure indique si vous utilisez actuellement la clé d’accès au compte ou votre compte Azure AD pour vous authentifier.

### <a name="authenticate-with-the-account-access-key"></a>S'authentifier à l'aide de la clé d'accès au compte

Si vous vous authentifiez à l’aide de la clé d’accès au compte, vous verrez la méthode **Clé d’accès** spécifiée comme méthode d’authentification dans le portail :

:::image type="content" source="media/authorize-data-operations-portal/auth-method-access-key.png" alt-text="Capture d’écran montrant l’utilisateur qui accède actuellement aux conteneurs à l’aide de la clé de compte":::

Pour passer à l’utilisation du compte Azure AD, cliquez sur le lien mis en surbrillance dans l’image. Si vous disposez des autorisations appropriées via les rôles Azure qui vous sont attribués, vous pourrez poursuivre. Toutefois, si vous n’avez pas les autorisations appropriées, vous verrez un message d’erreur semblable à celui-ci :

:::image type="content" source="media/authorize-data-operations-portal/auth-error-azure-ad.png" alt-text="Erreur affichée si le compte Azure AD ne prend pas en charge l’accès":::

Notez qu’aucun objet blob n’apparaît dans la liste si votre compte Azure AD ne dispose pas des autorisations pour les afficher. Cliquez sur le lien **Basculer sur clé d’accès** pour utiliser à nouveau la clé d’accès pour l’authentification.

### <a name="authenticate-with-your-azure-ad-account"></a>S'authentifier à l'aide du compte Azure AD

Si vous vous authentifiez à l’aide de votre compte Azure AD, vous verrez la méthode **Clé d’accès** spécifiée comme méthode d’authentification dans le portail :

:::image type="content" source="media/authorize-data-operations-portal/auth-method-azure-ad.png" alt-text="Capture d’écran montrant l’utilisateur qui accède actuellement aux conteneurs à l’aide d’un compte Azure AD":::

Pour passer à l’utilisation d’une clé d’accès au compte, cliquez sur le lien mis en surbrillance dans l’image. Si vous avez accès à la clé de compte, vous pourrez poursuivre. Toutefois, si vous n’avez pas accès à la clé de compte, vous verrez un message d’erreur semblable à celui-ci :

:::image type="content" source="media/authorize-data-operations-portal/auth-error-access-key.png" alt-text="Erreur affichée si vous n’avez pas accès à la clé de compte":::

Notez qu’aucun objet blob n’apparaît dans la liste si vous n’avez pas accès aux clés de compte. Cliquez sur le lien **Basculer sur Compte d’utilisateur Azure AD** pour vous authentifier à nouveau.

## <a name="specify-how-to-authorize-a-blob-upload-operation"></a>Spécifier comment autoriser une opération de chargement d'objets blob

Lorsque vous chargez un objet blob à partir du portail Azure, vous pouvez spécifier si cette opération doit être authentifiée et autorisée à l'aide de la clé d'accès au compte ou de vos informations d'identification Azure AD. Par défaut, le portail utilise la méthode d'authentification actuelle, comme indiqué dans [Déterminer la méthode d'authentification actuelle](#determine-the-current-authentication-method).

Pour spécifier comment autoriser une opération de chargement d'objets blob, procédez comme suit :

1. Sur le portail Azure, accédez au conteneur dans lequel vous souhaitez charger un objet blob.
1. Cliquez sur le bouton **Charger**.
1. Développez la section **Avancé** pour afficher les propriétés avancées de l'objet blob.
1. Dans le champ **Type d'authentification**, indiquez si vous souhaitez autoriser l'opération de chargement à l'aide de votre compte Azure AD ou de la clé d'accès au compte, comme illustré ci-dessous :

    :::image type="content" source="media/authorize-data-operations-portal/auth-blob-upload.png" alt-text="Capture d'écran montrant comment modifier la méthode d'autorisation au moment du chargement d'un objet blob":::

## <a name="next-steps"></a>Étapes suivantes

- [Authentifier l’accès aux objets blob et aux files d’attente Azure à l’aide d’Azure Active Directory](../common/storage-auth-aad.md)
- [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-portal.md)
- [Utiliser Azure CLI afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-cli.md)
- [Utiliser le module Azure PowerShell afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](../common/storage-auth-aad-rbac-powershell.md)
