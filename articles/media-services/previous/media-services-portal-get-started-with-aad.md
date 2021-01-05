---
title: Prise en main de l’authentification Azure AD à l’aide du portail Azure | Microsoft Docs
description: Découvrez comment utiliser le portail Azure pour accéder à l’authentification Azure Active Directory (Azure AD) en vue d’utiliser l’API Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 3d6c494232456b4819c9275d66fa6f7ab5de9b77
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89260728"
---
# <a name="get-started-with-azure-ad-authentication-by-using-the-azure-portal"></a>Prise en main de l’authentification Azure AD à l’aide du portail Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Découvrez comment utiliser le portail Azure pour accéder à l’authentification Azure Active Directory (Azure AD) en vue d’accéder à l’API Azure Media Services.

## <a name="prerequisites"></a>Conditions préalables requises

- Un compte Azure. Si vous n’avez pas de compte, commencez avec un [essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Un compte Media Services. Pour plus d’informations, consultez [Création d’un compte Azure Media Services à l’aide du portail Azure](media-services-portal-create-account.md).

Lorsque vous utilisez l’authentification Azure AD avec Azure Media Services, vous disposez de deux options d’authentification :

- **Authentification d’un principal de service**. Authentifiez un service. Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés : applications web, applications de fonction, applications logiques, API ou microservices.
- **Authentification utilisateur**. Authentifiez une personne qui utilise l’application pour interagir avec les ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct. 

## <a name="access-the-media-services-api"></a>Accéder à l’API Media Services

Cette page vous permet de sélectionner la méthode d’authentification que vous souhaitez utiliser pour vous connecter à l’API. Elle fournit également les valeurs dont vous avez besoin pour vous connecter à l’API.

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Media Services.
2. Choisissez comment vous connecter à l’API Media Services.
3. Sous **Se connecter à l’API Media Services**, sélectionnez la version de l’API Media Services à laquelle vous souhaitez vous connecter.

## <a name="service-principal-authentication--recommended"></a>Authentification d’un principal du service (recommandée)

Authentifie un service à l’aide d’une application Azure Active Directory (Azure AD) et d’un secret. Ceci est recommandé pour tous les services de niveau intermédiaire qui appellent l’API Media Services, par exemple les applications web, les fonctions, les applications logiques, les API ou les microservices. Il s’agit de la méthode d’authentification recommandée.

### <a name="manage-your-azure-ad-app-and-secret"></a>Gérer votre application Azure AD et votre secret

La section **Gérer votre application AAD et votre secret** vous permet de sélectionner ou de créer une application Azure AD et de générer un secret. Pour des raisons de sécurité, le secret ne peut pas être affiché après la fermeture du panneau. L’application utilise l’ID d’application et le secret pour l’authentification afin d’obtenir un jeton valide pour Media Services.

Veillez à avoir les autorisations suffisantes pour inscrire une application auprès de votre locataire Azure AD et pour attribuer l’application à un rôle dans votre abonnement Azure. Pour plus d’informations, consultez [Autorisations requises](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

### <a name="connect-to-media-services-api"></a>Se connecter à l’API Media Services

**Se connecter à l’API Media Services** fournit les valeurs que vous utilisez pour connecter votre application de principal du service. Vous pouvez récupérer des valeurs de texte ou copier les blocs JSON ou XML.

## <a name="user-authentication"></a>Authentification utilisateur

Cette option peut être utilisée pour authentifier un employé ou un membre d’un annuaire Azure Active Directory qui utilise une application pour interagir avec des ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Cette méthode d’authentification doit être utilisée uniquement pour les applications de gestion.

### <a name="connect-to-media-services-api"></a>Se connecter à l’API Media Services

Copiez vos informations d’identification pour connecter votre application utilisateur à partir de la section **Se connecter à l’API Media Services**. Vous pouvez récupérer des valeurs de texte ou copier les blocs JSON ou XML.

## <a name="next-steps"></a>Étapes suivantes

Commencez le [chargement de fichiers sur votre compte](media-services-portal-upload-files.md).
