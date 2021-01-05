---
title: Bien démarrer avec l’authentification Azure AD
description: Découvrez comment accéder à l’authentification Azure Active Directory (Azure AD) pour consommer l’API Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 00808c25ac84da852cce6169fb210767ee2b56cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89265879"
---
# <a name="get-credentials-to-access-media-services-api"></a>Obtenir des informations d’identification pour accéder à l’API Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Quand vous utilisez l’authentification Azure AD pour accéder à l’API Azure Media Services, vous disposez de deux options d’authentification :

- **Authentification d’un principal du service** (recommandée)

    Authentifiez un service. Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés : applications web, applications de fonction, applications logiques, API ou microservices.
- **Authentification utilisateur**

    Authentifiez une personne qui utilise l’application pour interagir avec les ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct. 

Cet article décrit les étapes permettant d’obtenir des informations d’identification pour accéder à l’API Media Services. Choisissez parmi les onglets suivants.

## <a name="prerequisites"></a>Prérequis

- Un compte Azure. Si vous n’avez pas de compte, commencez avec un [essai gratuit Azure](https://azure.microsoft.com/pricing/free-trial/). 
- Un compte Media Services. Pour plus d’informations, consultez [Création d’un compte Azure Media Services à l’aide du portail Azure](create-account-howto.md).

## <a name="portal"></a>[Portail](#tab/portal/)

### <a name="api-access"></a>Accès d’API

La page **Accès d’API** vous permet de sélectionner la méthode d’authentification que vous souhaitez utiliser pour vous connecter à l’API. Elle fournit également les valeurs dont vous avez besoin pour vous connecter à l’API.

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre compte Media Services.
2. Sélectionnez le panneau **Accès d’API (nouveau)** dans la barre de navigation de gauche.
3. Sous **Se connecter à l’API Media Services**, sélectionnez la version de l’API Media Services à laquelle vous souhaitez vous connecter (V3 est la dernière version du service).

### <a name="service-principal-authentication--recommended"></a>Authentification d’un principal du service (recommandée)

Authentifie un service à l’aide d’une application Azure Active Directory (Azure AD) et d’un secret. Ceci est recommandé pour tous les services de niveau intermédiaire qui appellent l’API Media Services, par exemple les applications web, les fonctions, les applications logiques, les API ou les microservices. Il s’agit de la méthode d’authentification recommandée.

#### <a name="manage-your-azure-ad-app-and-secret"></a>Gérer votre application Azure AD et votre secret

La section **Gérer votre application AAD et votre secret** vous permet de sélectionner ou de créer une application Azure AD et de générer un secret. Pour des raisons de sécurité, le secret ne peut pas être affiché après la fermeture du panneau. L’application utilise l’ID d’application et le secret pour l’authentification afin d’obtenir un jeton valide pour Media Services.

Veillez à avoir les autorisations suffisantes pour inscrire une application auprès de votre locataire Azure AD et pour attribuer l’application à un rôle dans votre abonnement Azure. Pour plus d’informations, consultez [Autorisations requises](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

#### <a name="connect-to-media-services-api"></a>Se connecter à l’API Media Services

**Se connecter à l’API Media Services** fournit les valeurs que vous utilisez pour connecter votre application de principal du service. Vous pouvez récupérer des valeurs de texte ou copier les blocs JSON ou XML.

### <a name="user-authentication"></a>Authentification utilisateur

Cette option peut être utilisée pour authentifier un employé ou un membre d’un annuaire Azure Active Directory qui utilise une application pour interagir avec des ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Cette méthode d’authentification doit être utilisée uniquement pour les applications de gestion.

#### <a name="connect-to-media-services-api"></a>Se connecter à l’API Media Services

Copiez vos informations d’identification pour connecter votre application utilisateur à partir de la section **Se connecter à l’API Media Services**. Vous pouvez récupérer des valeurs de texte ou copier les blocs JSON ou XML.

## <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/cli/)

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

---

## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Charger, encoder et diffuser des vidéos avec Media Services v3](stream-files-tutorial-with-api.md).
