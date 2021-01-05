---
title: Accéder à une API Azure Media Services avec l’authentification Azure Active Directory | Microsoft Docs
description: Découvrez les concepts et les étapes à suivre pour utiliser Azure Active Directory (Azure AD) afin d’authentifier l’accès à l’API Azure Media Services.
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
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 9dae3b1adc1f7fecf6a34fc51d2a35bf4c98ea65
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92425916"
---
# <a name="access-the-azure-media-services-api-with-azure-ad-authentication"></a>Accéder à l’API Azure Media Services avec l’authentification Azure AD 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

L’API Azure Media Services est une API RESTful. Vous pouvez l’utiliser pour effectuer des opérations sur les ressources médias à l’aide d’une API REST ou à l’aide de Kits de développement logiciel (SDK) clients disponibles. Azure Media Services propose un Kit de développement logiciel (SDK) client Media Services pour Microsoft .NET. Pour être autorisé à accéder aux ressources Media Services et à l’API Media Services, vous devez tout d’abord être authentifié. 

Media Services prend en charge l’[authentification avec Azure Active Directory (Azure AD)](../../active-directory/fundamentals/active-directory-whatis.md). Le service REST Media Azure nécessite que l’utilisateur ou l’application effectuant les requêtes de l’API REST ait le rôle **Collaborateur** ou **Propriétaire** pour accéder aux ressources. Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md)  

Ce document donne une vue d’ensemble de l’accès à l’API Media Services à l’aide des API REST ou .NET.

> [!NOTE]
> L’autorisation Access Control a été dépréciée le 1er juin 2018.

## <a name="access-control"></a>Contrôle d’accès

Pour que la demande Azure Media REST réussisse, l’utilisateur appelant doit avoir un rôle Collaborateur ou Propriétaire pour le compte Media Services auquel il tente d’accéder.  
Seul un utilisateur avec le rôle Propriétaire peut donner accès aux ressources médias (compte) pour les nouveaux utilisateurs ou les nouvelles applications. Le rôle Collaborateur ne peut accéder qu’à la ressource média.
Les requêtes non autorisées échouent et génèrent le code d’état 401. Si vous voyez ce code d’erreur, vérifiez que le rôle Collaborateur ou Propriétaire est assigné à votre utilisateur pour le compte Media Services de l’utilisateur. Vous pouvez le vérifier dans le portail Azure. Recherchez votre compte média, puis cliquez sur l’onglet **Contrôle d’accès** . 

![Onglet Contrôle d’accès](./media/media-services-use-aad-auth-to-access-ams-api/media-services-access-control.png)

## <a name="types-of-authentication"></a>Types d’authentification 
 
Lorsque vous utilisez l’authentification Azure AD avec Azure Media Services, vous disposez de deux options d’authentification :

- **Authentification utilisateur** . Authentifiez une personne qui utilise l’application pour interagir avec les ressources Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct. 
- **Authentification d’un principal de service** . Authentifiez un service. Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés, par exemple les applications web, les applications de fonction, les applications logiques, les API ou les microservices.

### <a name="user-authentication"></a>Authentification utilisateur 

Les applications qui doivent utiliser la méthode d’authentification utilisateur sont les applications natives de gestion ou de surveillance : applications mobiles, applications Windows et applications de console. Ce type de solution est utile lorsque vous souhaitez une interaction humaine avec le service dans l’un des scénarios suivants :

- Tableau de bord de surveillance pour vos travaux d’encodage.
- Tableau de bord de surveillance pour vos flux en direct.
- Application de gestion pour les utilisateurs d’ordinateurs de bureau ou les utilisateurs mobiles afin de gérer les ressources dans un compte Media Services.

> [!NOTE]
> Cette méthode d’authentification ne doit pas être utilisée pour les applications accessibles aux consommateurs. 

Une application native doit tout d’abord acquérir un jeton d’accès à partir d’Azure AD, puis l’utiliser lorsque vous effectuez des requêtes HTTP auprès de l’API REST Media Services. Ajoutez le jeton d’accès à l’en-tête de la requête. 

Le diagramme suivant illustre un flux d’authentification d’application interactive classique : 

![Diagramme d’applications natives](./media/media-services-use-aad-auth-to-access-ams-api/media-services-native-aad-app1.png)

Dans le diagramme précédent, les nombres représentent le flux des requêtes dans l’ordre chronologique.

> [!NOTE]
> Lorsque vous utilisez la méthode d’authentification utilisateur, toutes les applications partagent le même ID client d’application native (par défaut) et le même URI de redirection d’application native. 

1. Demandez à un utilisateur ses informations d’identification.
2. Demandez un jeton d’accès Azure AD avec les paramètres suivants :  

   * Point de terminaison de locataire Azure AD.

       Les informations relatives au locataire peuvent être récupérées à partir du portail Azure. Placez votre curseur sur le nom de l’utilisateur connecté, en haut à droite.
   * URI de ressource Media Services. 

       Cet URI est identique pour les comptes Media Services qui se trouvent dans le même environnement Azure (par exemple, https:\//rest.media.azure.net).

   * ID client d’application Media Services (natif).
   * URI de redirection d’application Media Services (natif).
   * URI de ressource pour REST Media Services.
        
       L’URI représente le point de terminaison d’API REST (par exemple, https://test03.restv2.westus.media.azure.net/api/) ).

     Pour obtenir les valeurs de ces paramètres, consultez [Prise en main de l’authentification Azure AD à l’aide du portail Azure](media-services-portal-get-started-with-aad.md) par le biais de l’option d’authentification utilisateur.

3. Le jeton d’accès Azure AD est envoyé au client.
4. Le client envoie une requête à l’API REST Azure Media avec le jeton d’accès Azure AD.
5. Le client récupère les données de Media Services.

Pour plus d’informations sur l’utilisation de l’authentification Azure AD dans l’optique de communiquer avec les requêtes REST par le biais du Kit de développement logiciel (SDK) client Media Services .NET, consultez [Use Azure AD authentication to access Azure Media Services API with .NET](media-services-dotnet-get-started-with-aad.md) (Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec .NET). 

Si vous n’utilisez pas le Kit de développement logiciel (SDK) client de Media Services .NET, vous devez créer manuellement une demande de jeton d’accès Azure AD en utilisant les paramètres décrits à l’étape 2. Pour plus d’informations, consultez [Bibliothèques d’authentification d’Azure Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md).

### <a name="service-principal-authentication"></a>Authentification d’un principal du service

Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services de niveau intermédiaire et des travaux planifiés : applications web, applications de fonction, applications logiques, API et microservices. Cette méthode d’authentification est également adaptée aux applications interactives dans lesquelles vous souhaitez peut-être utiliser un compte de service pour gérer les ressources.

Lorsque vous utilisez la méthode d’authentification du principal de service pour créer des scénarios de clients, l’authentification est généralement gérée au niveau intermédiaire (par le biais d’une API) et non directement dans une application mobile ou une application de bureau. 

Pour utiliser cette méthode, créez une application Azure AD et un principal de service dans son propre locataire. Après avoir créé l’application, assignez à l’application le rôle Collaborateur ou Propriétaire pour accéder au compte Media Services. Vous pouvez le faire dans le portail Azure, à l’aide d’Azure CLI ou d’un script PowerShell. Vous pouvez aussi utiliser une application Azure AD existante. Vous pouvez inscrire et gérer votre application Azure AD et votre principal de service [dans le portail Azure](media-services-portal-get-started-with-aad.md). Vous pouvez également le faire à l’aide d’[Azure CLI](media-services-use-aad-auth-to-access-ams-api.md) ou de [PowerShell](media-services-powershell-create-and-configure-aad-app.md). 

![Applications de niveau intermédiaire](./media/media-services-use-aad-auth-to-access-ams-api/media-services-principal-service-aad-app1.png)

Après avoir créé votre application Azure AD, vous obtenez les valeurs correspondant aux paramètres suivants. Vous avez besoin de ces valeurs pour l’authentification :

- ID client 
- Clé secrète client 

Dans la figure précédente, les nombres représentent le flux des requêtes dans l’ordre chronologique :
    
1. Une application de niveau intermédiaire (API web ou application web) nécessite un jeton d’accès Azure AD qui possède les paramètres suivants :  

   * Point de terminaison de locataire Azure AD.

       Les informations relatives au locataire peuvent être récupérées à partir du portail Azure. Placez votre curseur sur le nom de l’utilisateur connecté, en haut à droite.
   * URI de ressource Media Services. 

       Cet URI est identique pour les comptes Media Services qui se trouvent dans le même environnement Azure (par exemple, https:\//rest.media.azure.net).

   * URI de ressource pour REST Media Services.

       L’URI représente le point de terminaison d’API REST (par exemple, https://test03.restv2.westus.media.azure.net/api/) ).

   * Valeurs de l’application Azure AD : ID client et clé secrète client.
    
     Pour obtenir les valeurs de ces paramètres, consultez [Prise en main de l’authentification Azure AD à l’aide du portail Azure](media-services-portal-get-started-with-aad.md) par le biais de l’option d’authentification utilisateur du principal de serveur.

2. Le jeton d’accès Azure AD est envoyé au niveau intermédiaire.
4. Le niveau intermédiaire envoie une requête à l’API REST Azure Media avec le jeton Azure AD.
5. Le niveau intermédiaire récupère les données de Media Services.

Pour plus d’informations sur l’utilisation de l’authentification Azure AD dans l’optique de communiquer avec les requêtes REST par le biais du Kit de développement logiciel (SDK) client Media Services .NET, consultez [Use Azure AD authentication to access Azure Media Services API with .NET](media-services-dotnet-get-started-with-aad.md) (Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec .NET). 

Si vous n’utilisez pas le Kit de développement logiciel (SDK) client Media Services .NET, vous devez créer manuellement une demande de jeton Azure AD en utilisant les paramètres décrits à l’étape 1. Pour plus d’informations, consultez [Bibliothèques d’authentification d’Azure Active Directory](../../active-directory/azuread-dev/active-directory-authentication-libraries.md).

## <a name="troubleshooting"></a>Dépannage

Exception : « Le serveur distant a retourné une erreur : (401) Non autorisé. »

Solution : Pour que la requête Media Services REST réussisse, l’utilisateur appelant doit avoir un rôle Contributeur ou Propriétaire pour le compte Media Services auquel il tente d’accéder. Pour plus d’informations, consultez la section [Contrôle d’accès](media-services-use-aad-auth-to-access-ams-api.md#access-control).

## <a name="resources"></a>Ressources

Les articles suivants sont des vues d’ensemble des concepts d’authentification Azure AD : 

- [Scénarios d’authentification pour Azure AD](../../active-directory/develop/authentication-vs-authorization.md)
- [Add, update, or remove an application in Azure AD](../../active-directory/develop/quickstart-register-app.md) (Ajouter, mettre à jour ou supprimer une application dans Azure AD)
- [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)

## <a name="next-steps"></a>Étapes suivantes

* Utilisez le portail Azure pour [accéder à l’authentification Azure AD en vue d’utiliser l’API Azure Media Services](media-services-portal-get-started-with-aad.md).
* Utilisez l’authentification Azure AD pour [accéder à l’API Azure Media Services avec .NET](media-services-dotnet-get-started-with-aad.md).
