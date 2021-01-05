---
title: Autoriser l’accès aux objets blob et aux files d’attente avec Active Directory
titleSuffix: Azure Storage
description: Autorisez l’accès aux blobs et aux files d’attente Azure à l’aide d’Azure Active Directory (Azure AD). Attribuez des rôles Azure pour les droits d’accès. Accédez aux données avec un compte Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: 9d03496634c5d30d30b23a76b5b47b1e810af288
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94635396"
---
# <a name="authorize-access-to-blobs-and-queues-using-azure-active-directory"></a>Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory

Le Stockage Azure prend en charge l’utilisation d’Azure Active Directory (Azure AD) pour autoriser les demandes au stockage Blob et File d’attente. Avec Azure AD, vous pouvez utiliser le contrôle d’accès en fonction du rôle d’Azure (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur, un groupe ou un principal de service d’application. Le principal de sécurité est authentifié par Azure AD pour retourner un jeton OAuth 2.0. Le jeton peut ensuite être utilisé pour autoriser une demande auprès du stockage Blob ou File d’attente.

Autoriser les demandes auprès du Stockage Azure avec Azure AD offre davantage de sécurité et de facilité d’utilisation sur l’autorisation de clé partagée. Microsoft recommande d’utiliser, si possible, l’autorisation Azure AD avec vos applications d’objets blob et de file d’attente afin de minimiser les failles de sécurité potentielles inhérentes à la clé partagée.

L’autorisation avec Azure AD est disponible pour tous les comptes de stockage universels et comptes de stockage Blob, dans la totalité des régions publiques et clouds nationaux. Seuls les comptes de stockage créés avec le modèle de déploiement Azure Resource Manager prennent en charge l’autorisation Azure AD.

Le stockage Blob prend également en charge la création de signatures d’accès partagé (SAS) signées avec des informations d’identification Azure AD. Pour plus d’informations, consultez [Accorder un accès limité aux données avec des signatures d’accès partagé](storage-sas-overview.md).

Azure Files prend uniquement en charge l’autorisation avec AD (en préversion) ou Azure AD DS (généralement disponible) sur SMB uniquement pour des machines virtuelles jointes au domaine. Pour en savoir plus sur l’utilisation d’AD (en préversion) ou d’Azure AD DS (généralement disponible) sur SMB pour Azure Files, voir [Vue d’ensemble de la prise en charge de l’authentification basée sur l’identité Azure Files pour l’accès SMB](../files/storage-files-active-directory-overview.md).

L’autorisation avec Azure AD n’est pas prise en charge pour Stockage Table Azure. Utilisez la clé partagée pour autoriser les demandes au stockage Table.

## <a name="overview-of-azure-ad-for-blobs-and-queues"></a>Vue d’ensemble d’Azure AD pour les objets blob et les files d’attente

Lorsqu’un principal de sécurité (utilisateur, groupe ou application) tente d’accéder à une ressource d’objet blob ou de file d’attente, cette requête doit être autorisée, sauf s’il s’agit d’un objet blob disponible pour l’accès anonyme. Avec Azure AD, l’accès à une ressource est un processus en deux étapes. Pour commencer, l’identité du principal de sécurité est authentifiée, et un jeton OAuth 2.0 est renvoyé. Ensuite, ce jeton est transmis dans le cadre d’une requête adressée au service BLOB ou au service de File d’attente et utilisé par le service pour autoriser l’accès à la ressource spécifiée.

L’étape d’authentification nécessite qu’une application demande un jeton d’accès OAuth 2.0 au moment de l’exécution. Si une application s’exécute à partir d’une entité Azure telle qu’une machine virtuelle Azure, un groupe de machines virtuelles identiques ou une application Azure Functions, elle peut utiliser une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md) pour accéder aux objets blob ou aux files d’attente. Pour savoir comment autoriser des requêtes adressées au service Azure Blob ou au service de File d’attente par une identité managée, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory et les identités managées pour les ressources Azure](storage-auth-aad-msi.md).

L’étape d’autorisation exige qu’un ou plusieurs rôles Azure soient attribués au principal de sécurité. Le service Stockage Azure fournit des rôles Azure qui englobent des ensembles communs d’autorisations pour les données de blob et de file d’attente. Les rôles qui sont attribués à un principal de sécurité déterminent les autorisations dont disposera le principal. Pour plus d’informations sur l’attribution de rôles Azure pour le service Stockage Azure, consultez [Gérer les droits d’accès aux données de stockage avec le contrôle RBAC Azure](./storage-auth-aad-rbac-portal.md).

Les applications natives et les applications web qui adressent des requêtes au service Azure Blob ou au service de File d’attente peuvent également autoriser l’accès avec Azure AD. Pour savoir comment demander un jeton d’accès et l’utiliser pour autoriser les requêtes qui interrogent des données d’objet blob ou de file d’attente, consultez [Autoriser l’accès au stockage Azure avec Azure AD à partir d’une application de stockage Azure](storage-auth-aad-app.md).

## <a name="assign-azure-roles-for-access-rights"></a>Attribuer des rôles Azure pour les droits d’accès

Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md). Stockage Azure définit un ensemble de rôles intégrés Azure qui englobent les ensembles communs d’autorisations permettant d’accéder aux données blob et de file d’attente. Vous pouvez également définir des rôles personnalisés pour l’accès aux données d’objet blob et de file d’attente.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, du compte de stockage ou d’un conteneur ou d’une file d’attente individuelle. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

### <a name="azure-built-in-roles-for-blobs-and-queues"></a>Rôles intégrés Azure pour les objets blob et les files d’attente

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

Pour plus d’informations sur l’attribution d’un rôle intégré Azure à un principal de sécurité, consultez l’un des articles suivants :

- [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-portal.md)
- [Utiliser Azure CLI afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-cli.md)
- [Utiliser le module Azure PowerShell afin d’attribuer un rôle Azure pour l’accès aux données de blob et de file d’attente](storage-auth-aad-rbac-powershell.md)

Pour plus d’informations sur la définition des rôles intégrés pour le Stockage Azure, consultez [Comprendre les définitions de rôles](../../role-based-access-control/role-definitions.md#management-and-data-operations). Pour plus d’informations sur la création de rôles personnalisés Azure, consultez [Rôles personnalisés Azure](../../role-based-access-control/custom-roles.md).

### <a name="access-permissions-for-data-operations"></a>Autorisations d’accès pour les opérations de données

Pour plus d’informations sur les autorisations requises pour l’appel d’opérations propres aux services Azure BLOB ou de File d’attente, consultez la section [Permissions for calling blob and queue data operations](/rest/api/storageservices/authorize-with-azure-active-directory#permissions-for-calling-blob-and-queue-data-operations) (Autorisations pour l’appel d’opérations de données d’objet blob et de file d’attente).

## <a name="resource-scope"></a>Étendue des ressources

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="access-data-with-an-azure-ad-account"></a>Accéder aux données avec un compte Azure AD

L’accès aux données d’objet blob ou de file d’attente par le biais du portail Azure, de PowerShell ou d’Azure CLI peut être autorisé à l’aide du compte Azure AD de l’utilisateur, ou au moyen des clés d’accès au compte (autorisation de clé partagée).

### <a name="data-access-from-the-azure-portal"></a>Accès aux données à partir du Portail Azure

Le Portail Azure peut utiliser soit votre compte Azure AD, soit les clés d’accès au compte pour accéder aux données d’objet blob et de file d’attente dans un compte de stockage Azure. Le schéma d’autorisation utilisé par le portail Azure dépend des rôles Azure qui vous sont attribués.

Lorsque vous tentez d’accéder aux données de blob ou de file d’attente, le portail Azure commence par vérifier si vous avez reçu un rôle Azure avec **Microsoft.Storage/storageAccounts/listkeys/action**. Si vous avez été doté d’un rôle avec cette action, le Portail Azure utilise la clé de compte pour l’accès aux données d’objet blob et de file d’attente par le biais de l’autorisation par clé partagée. Si vous n’avez pas reçu un rôle avec cette action, le Portail Azure tente d’accéder aux données à l’aide de votre compte Azure AD.

Pour accéder aux données d’objet blob ou de file d’attente à partir du Portail Azure à l’aide de votre compte Azure AD, vous devez disposer d’autorisations pour accéder à ces données, ainsi que pour parcourir les ressources de compte de stockage dans le Portail Azure. Les rôles intégrés fournis par le service Stockage Azure octroient l’accès aux ressources d’objet blob et de file d’attente, mais n’accordent pas d’autorisations pour les ressources de compte de stockage. C’est la raison pour laquelle l’accès au portail requiert également l’attribution d’un rôle Azure Resource Manager, tel que le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader), étendu au niveau du compte de stockage ou à un niveau supérieur. Le rôle **Lecteur** octroie les autorisations les plus restreintes, mais l’utilisation d’un autre rôle Azure Resource Manager accordant l’accès aux ressources de gestion de compte de stockage est également acceptable. Pour en savoir plus sur l’attribution d’autorisations à des utilisateurs pour l’accès aux données dans le portail Azure avec un compte Azure AD, consultez [Utiliser le portail Azure afin d’attribuer un rôle Azure pour l’accès aux données Blob et de file d’attente](storage-auth-aad-rbac-portal.md).

Le Portail Azure indique le schéma d’autorisation en cours d’utilisation lorsque vous accédez à un conteneur ou à une file d’attente. Pour plus d’informations sur l’accès aux données dans le portail, consultez [Choisir comment autoriser l’accès à des données d’objets blob dans le portail Azure](../blobs/authorize-data-operations-portal.md) et [Choisir comment autoriser l’accès à des données en file d’attente dans le portail Azure](../queues/authorize-data-operations-portal.md).

### <a name="data-access-from-powershell-or-azure-cli"></a>Accès aux données à partir de PowerShell ou d’Azure CLI

Azure CLI et PowerShell prennent en charge la connexion avec des informations d’identification Azure AD. Une fois que vous vous êtes connecté, votre session s’exécute sous ces informations d’identification. Pour plus d’informations, consultez l’article [Exécuter des commandes Azure CLI ou PowerShell avec des informations d’identification Azure AD pour accéder aux données d’objet blob ou de file d’attente](../blobs/authorize-data-operations-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

- [Autoriser l’accès aux objets blob et files d’attente avec Azure Active Directory et les identités managées pour les ressources Azure](storage-auth-aad-msi.md)
- [S'authentifier auprès d'Azure Active Directory à partir d'une application pour accéder aux objets blob et aux files d'attente](storage-auth-aad-app.md)
- [Azure Storage support for Azure Active Directory based access control generally available](https://azure.microsoft.com/blog/azure-storage-support-for-azure-ad-based-access-control-now-generally-available/) (Disponibilité générale de la prise en charge par le service Stockage Azure du contrôle d’accès basé sur Azure Active Directory)