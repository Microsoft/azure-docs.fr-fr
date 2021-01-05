---
title: Authentifier une application pour accéder aux ressources Azure Event Hubs
description: Cet article fournit des informations sur l’authentification d’une application avec Azure Active Directory pour accéder aux ressources Azure Event Hubs
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: 25ec5f11ca7b5e801e18155f1a3da6474c8e66e2
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913311"
---
# <a name="authenticate-an-application-with-azure-active-directory-to-access-event-hubs-resources"></a>Authentifier une application avec Azure Active Directory pour accéder aux ressources Azure Event Hubs
Microsoft Azure offre la gestion du contrôle d’accès intégré pour les ressources et les applications basées sur Azure Active Directory (Azure AD). L’un des principaux avantages de l’utilisation d’Azure AD avec Azure Event Hubs est que vous n’avez plus besoin de stocker vos informations d’identification dans le code. Au lieu de cela, vous pouvez demander un jeton d’accès OAuth 2.0 à partir de la Plateforme d’identité Microsoft. Le nom de la ressource pour demander un jeton est `https://eventhubs.azure.net/`. Il est identique pour tous les clouds/locataires (pour les clients Kafka, il s’agit de `https://<namespace>.servicebus.windows.net`). Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne un jeton d’accès à l’application et l’application peut ensuite l’utiliser pour autoriser les demandes vers les ressources Azure Event Hubs.

Lorsqu’un rôle est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, de l’espace de noms Event Hubs ou de toute ressource sous-jacente. Un principal de sécurité Azure AD peut attribuer des rôles à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md). 

> [!NOTE]
> Une définition de rôle est une collection d’autorisations. Le contrôle d’accès en fonction du rôle Azure (RBAC Azure) contrôle la façon dont ces autorisations sont appliquées par le biais de l’attribution de rôle. Une attribution de rôle se compose de trois éléments : un principal de sécurité, une définition de rôle et une étendue. Pour plus d’informations, consultez [Comprendre les différents rôles](../role-based-access-control/overview.md).

## <a name="built-in-roles-for-azure-event-hubs"></a>Rôles intégrés pour Azure Event Hubs
Azure fournit les rôles intégrés Azure suivants pour autoriser l’accès aux données Event Hubs à l’aide d’Azure AD et d’OAuth :

- [Propriétaire de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner) : Utilisez ce rôle pour accorder un accès complet aux ressources Event Hubs.
- [Expéditeur de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender) : Utilisez ce rôle pour accorder un accès en envoi aux ressources Event Hubs.
- [Récepteur de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver) : Utilisez ce rôle pour accorder un accès en réception aux ressources Event Hubs.   

Pour les rôles intégrés du registre de schémas, consultez [Rôles du registre de schémas](schema-registry-overview.md#azure-role-based-access-control).

> [!IMPORTANT]
> Notre préversion prenait en charge l’ajout de privilèges d’accès aux données Event Hubs au rôle Propriétaire ou Contributeur. Toutefois, les privilèges d’accès aux données pour le rôle Propriétaire et le rôle Contributeur ne sont plus respectés. Si vous utilisez le rôle Propriétaire ou Contributeur, passez à l’utilisation du rôle Propriétaire de données Azure Event Hubs.


## <a name="authenticate-from-an-application"></a>Authentifier à partir d’une application
Le principal avantage d’utiliser Azure AD avec Event Hubs est que vos informations d’identification n’ont plus besoin d’être stockées dans votre code. À la place, vous pouvez demander un jeton d’accès OAuth 2.0 sur la plateforme d’identités Microsoft. Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne le jeton d’accès à l’application et l’application peut ensuite l’utiliser pour autoriser les demandes vers Azure Event Hubs.

Les sections suivantes vous montrent comment configurer votre application native ou une application web pour l’authentification avec la plateforme d’identités Microsoft 2.0. Pour plus d’informations sur la plateforme d’identité Microsoft 2.0, veuillez consulter l’article [Présentation de la plateforme d’identités Microsoft (v2.0)](../active-directory/develop/v2-overview.md).

Pour avoir une vue d’ensemble du flux d’octroi de code OAuth 2.0, consultez [Autoriser l’accès aux applications web Azure Active Directory à l’aide du flux d’octroi de code OAuth 2.0](../active-directory/develop/v2-oauth2-auth-code-flow.md).

### <a name="register-your-application-with-an-azure-ad-tenant"></a>Inscrire votre application à un locataire Azure AD
La première étape d’utilisation d’Azure AD pour autoriser l’accès aux ressources Event Hubs est d’inscrire votre application cliente avec un locataire Azure AD à partir du [portail Microsoft Azure](https://portal.azure.com/). Lorsque vous inscrivez votre application cliente, vous fournissez des informations la concernant à Azure AD. Azure AD fournit ensuite un ID de client (appelé aussi ID d’application) que vous pouvez utiliser pour associer votre application au runtime Azure AD. Pour en savoir plus sur l’ID de client, consultez [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md). 

Les illustrations suivantes montrent les étapes d’inscription d’une application web :

![Inscrire une application](./media/authenticate-application/app-registrations-register.png)

> [!Note]
> Si vous inscrivez votre application comme une application native, vous pouvez spécifier n’importe quel URI valide pour l’URI de redirection. Pour les applications natives, cette valeur ne devra pas être une URL réelle. Pour les applications web, l’URI de redirection doit être un URI valide, car il spécifie l’URL à laquelle les jetons sont fournis.

Une fois votre application inscrite, **l’ID (client) d’application** se trouve sous **Paramètres** :

![ID de l’application inscrite](./media/authenticate-application/application-id.png)

Pour plus d’informations sur l’inscription d’une application dans Azure AD, consultez [Intégration d’applications à Azure Active Directory](../active-directory/develop/quickstart-register-app.md).


### <a name="create-a-client-secret"></a>Créer une clé secrète client   
L’application a besoin d’une clé secrète client pour prouver son identité lors de la requête de jeton. Pour ajouter le secret client, effectuez ces étapes.

1. Accédez à votre inscription d’application dans le Portail Microsoft Azure.
1. Sélectionnez le paramètre **Certificats et clés secrètes**.
1. Sous **Secrets client** , sélectionnez **Nouveau secret client** pour créer un secret.
1. Fournissez une description pour le secret, puis choisissez un intervalle d’expiration.
1. Copiez immédiatement la valeur de la nouvelle clé secrète dans un emplacement sécurisé. La valeur de remplissage s’affiche pour vous une seule fois.

    ![Clé secrète client](./media/authenticate-application/client-secret.png)


## <a name="assign-azure-roles-using-the-azure-portal"></a>Attribuer des rôles Azure à l’aide du portail Azure  
Après avoir inscrit l'application, vous devez attribuer un rôle Azure AD Event Hubs au principal de service de l'application, comme décrit dans la section [Rôles intégrés pour Azure Event Hubs](#built-in-roles-for-azure-event-hubs). 

1. Dans le [portail Azure](https://portal.azure.com/), accédez à votre espace de noms Event Hubs.
2. Dans la page **Vue d’ensemble** , sélectionnez le hub d’événements auquel vous souhaitez attribuer un rôle.

    ![Sélectionnez votre hub d'événements](./media/authenticate-application/select-event-hub.png)
1. Sélectionnez **Contrôle d’accès (IAM)** pour afficher les paramètres de contrôle d’accès du hub d'événements. 
1. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles. Sélectionnez le bouton **Ajouter** dans la barre d’outils, puis sélectionnez **Ajouter une attribution de rôle**. 

    ![Ajouter un bouton à la barre d’outils](./media/authenticate-application/role-assignments-add-button.png)
1. Sur la page **Ajouter une attribution de rôle** , procédez comme suit :
    1. Sélectionnez le **rôle Event Hubs** que vous souhaitez attribuer. 
    1. Recherchez le **principal de sécurité** (utilisateur, groupe, principal de service) auquel vous souhaitez attribuer le rôle. Sélectionnez l' **application inscrite** dans la liste. 
    1. Sélectionnez **Enregistrer** pour enregistrer l’attribution de rôle. 

        ![Attribuer un rôle à un utilisateur](./media/authenticate-application/assign-role-to-user.png)
    4. Passez à l'onglet **Attributions des rôles** et confirmez l'attribution du rôle. Par exemple, dans l'illustration suivante, l'application **mywebapp** dispose du rôle **Expéditeur de données Azure Event Hubs**. 
        
        ![Utilisateur dans la liste](./media/authenticate-application/user-in-list.png)

Vous pouvez suivre des étapes similaires pour attribuer un rôle dans l’étendue de l’espace de noms Event Hubs, du groupe de ressources ou de l’abonnement. Une fois que vous avez défini le rôle et son étendue, vous pouvez tester ce comportement à l’aide d’exemples [dans cet emplacement GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). Pour en savoir plus sur la gestion de l’accès aux ressources Azure à l’aide d’Azure RBAC et du portail Azure, consultez [cet article](..//role-based-access-control/role-assignments-portal.md). 


### <a name="client-libraries-for-token-acquisition"></a>Bibliothèques clientes pour l’acquisition de jeton  
Lorsque vous avez inscrit votre application et lui avez accordé les autorisations pour envoyer/recevoir des données dans Azure Event Hubs, vous pouvez ajouter du code à votre application pour authentifier un principal de sécurité et acquérir un jeton OAuth 2.0. Pour l’authentification et l’acquisition du jeton, vous pouvez utiliser l’une des [bibliothèques d’authentification de la plateforme d’identités Microsoft](../active-directory/develop/reference-v2-libraries.md) ou toute autre bibliothèque open source prenant en charge OpenID ou Connect 1.0. Votre application peut alors utiliser le jeton d’accès pour autoriser une requête sur Azure Event Hubs.

Pour obtenir la liste de scénarios pour lesquels l’acquisition de jetons est pris en charge, veuillez consulter la section [Scénarios](https://aka.ms/msal-net-scenarios) du référentiel GitHub [Microsoft Authentication Library (MSAL) pour .NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet).

## <a name="samples"></a>Exemples
- [Exemples Microsoft.Azure.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    Ces exemples utilisent l'ancienne bibliothèque **Microsoft.Azure.EventHubs** , mais vous pouvez facilement mettre celle-ci à jour à l'aide de la dernière bibliothèque **Azure.Messaging.EventHubs**. Pour migrer l'exemple de l'ancienne bibliothèque vers la nouvelle, consultez le [Guide de migration de Microsoft.Azure.EventHubs vers Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [Exemples Azure.Messaging.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)

    Cet exemple a été mis à jour pour utiliser la dernière bibliothèque **Azure.Messaging.EventHubs**.

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur Azure RBAC, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) ?
- Pour apprendre à attribuer et gérer les rôles Azure avec Azure PowerShell, Azure CLI ou l’API REST, consultez les articles suivants :
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../role-based-access-control/role-assignments-cli.md)
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](../role-based-access-control/role-assignments-rest.md)
    - [Ajouter des attributions de rôle Azure à l’aide de modèles Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

Consultez les articles associés suivants :
- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs](authenticate-managed-identity.md)
- [Authentifier les requêtes adressées à Azure Event Hubs à l’aide de signatures d’accès partagé](authenticate-shared-access-signature.md)
- [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md)
- [Authentifier l’accès aux ressources Event Hubs avec des signatures d’accès partagé](authorize-access-shared-access-signature.md)
