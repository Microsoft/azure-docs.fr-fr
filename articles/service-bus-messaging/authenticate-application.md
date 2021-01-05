---
title: Authentifier une application pour accéder aux entités Azure Service Bus
description: Cet article fournit des informations sur l’authentification d’une application avec Azure Active Directory pour accéder aux entités Azure Service Bus (files d’attente, rubriques, etc.)
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: c4e19c0ab26d491ba0b95159e274383431aefaee
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518226"
---
# <a name="authenticate-and-authorize-an-application-with-azure-active-directory-to-access-azure-service-bus-entities"></a>Authentifier et autoriser une application avec Azure Active Directory pour accéder aux entités Azure Service Bus
Azure Service Bus prend en charge l’utilisation d’Azure Active Directory (Azure AD) pour autoriser les requêtes d’accès aux entités Service Bus (files d’attente, rubriques, abonnements ou filtres). Avec Azure AD, vous pouvez utiliser le contrôle d’accès en fonction du rôle d’Azure (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur, un groupe ou un principal de service d’application. Pour en savoir plus sur les rôles et les attributions de rôles, consultez [Comprendre les différents rôles](../role-based-access-control/overview.md).

## <a name="overview"></a>Vue d’ensemble
Quand un principal de sécurité (un utilisateur, un groupe ou une application) tente d’accéder à une entité Service Bus, la requête doit être autorisée. Avec Azure AD, l’accès à une ressource est un processus en deux étapes. 

 1. Pour commencer, l’identité du principal de sécurité est authentifiée, et un jeton OAuth 2.0 est renvoyé. Le nom de ressource à utiliser pour demander un jeton est `https://servicebus.azure.net`.
 1. Ensuite, ce jeton est transmis dans une requête adressée au service Service Bus pour autoriser l’accès à la ressource spécifiée.

L’étape d’authentification nécessite qu’une requête d’application contienne un jeton d’accès OAuth 2.0 au moment de l’exécution. Si une application s’exécute à partir d’une entité Azure telle qu’une machine virtuelle Azure, un groupe de machines virtuelles identiques ou une application Azure Function, elle peut utiliser une identité managée pour accéder aux ressources. Pour plus d’informations sur l’authentification des requêtes adressées par une identité managée au service Service Bus, consultez [Authentifier l’accès aux ressources Azure Service Bus avec Azure Active Directory et les identités managées pour les ressources Azure](service-bus-managed-service-identity.md). 

L’étape d’autorisation exige qu’un ou plusieurs rôles Azure soient attribués au principal de sécurité. Azure Service Bus fournit des rôles Azure qui englobent des ensembles d’autorisations pour les ressources Service Bus. Les rôles qui sont attribués à un principal de sécurité déterminent les autorisations dont disposera le principal. Pour plus d’informations sur l’attribution de rôles Azure dans Azure Service Bus, consultez [Rôles Azure intégrés pour Azure Service Bus](#azure-built-in-roles-for-azure-service-bus). 

Les applications natives et applications web qui adressent des requêtes à Service Bus peuvent également autoriser l’accès avec Azure AD. Cet article explique comment demander un jeton d’accès et l’utiliser pour autoriser les demandes de ressources Service Bus. 


## <a name="assigning-azure-roles-for-access-rights"></a>Attribution de rôles Azure pour les droits d’accès
Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais [d’Azure RBAC](../role-based-access-control/overview.md). Azure Service Bus définit un ensemble de rôles Azure intégrés qui englobent les ensembles d’autorisations couramment utilisés pour accéder aux entités Service Bus. Il est également possible de définir des rôles personnalisés pour accéder aux données.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources ou de l’espace de noms Service Bus. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un groupe, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-service-bus"></a>Rôles Azure intégrés pour Azure Service Bus
Pour Azure Service Bus, la gestion des espaces de noms et de toutes les ressources associées via le Portail Azure et l’API de gestion des ressources Azure est déjà protégée à l’aide du modèle Azure RBAC. Azure fournit les rôles Azure intégrés ci-dessous pour autoriser l’accès à un espace de noms Service Bus :

- [Propriétaire de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) : permet l’accès aux données de l’espace de noms Service Bus et ses entités (files d’attente, rubriques, abonnements et filtres).
- [Expéditeur de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-sender) : utilisez ce rôle pour autoriser l’accès en envoi à l’espace de noms Service Bus et à ses entités.
- [Récepteur de données Azure Service Bus](../role-based-access-control/built-in-roles.md#azure-service-bus-data-receiver) : utilisez ce rôle pour autoriser l’accès en réception à l’espace de noms Service Bus et à ses entités. 

## <a name="resource-scope"></a>Étendue des ressources 
Avant d’attribuer un rôle Azure à un principal de sécurité, déterminez l’étendue de l’accès dont doit disposer le principal de sécurité. Selon les bonnes pratiques, il est toujours préférable d’accorder la plus petite étendue possible.

La liste suivante décrit les niveaux auxquels vous pouvez étendre l’accès aux ressources Service Bus, en commençant par la plus petite étendue :

- **File d’attente** , **rubrique** ou **abonnement**  : l’attribution de rôle s’applique à l’entité Service Bus spécifique. Actuellement, le Portail Azure ne prend pas en charge l’affectation d’utilisateurs, de groupes ou d’identités managées aux rôles Azure Service Bus au niveau de l’abonnement. 
- **Espace de noms Service Bus**  : l’attribution de rôle s’étend à toute la topologie de Service Bus sous l’espace de noms et au groupe de consommateurs qui lui est associé.
- **Groupe de ressources**  : l’attribution de rôle s’applique à toutes les ressources Service Bus sous le groupe de ressources.
- **Abonnement** : l’attribution de rôle s’applique à toutes les ressources Service Bus dans tous les groupes de ressources de l’abonnement.

> [!NOTE]
> Gardez à l’esprit que la propagation des attributions de rôles Azure peut prendre cinq minutes. 

Pour plus d’informations sur la définition des rôles intégrés, consultez [Comprendre les définitions de rôles](../role-based-access-control/role-definitions.md#management-and-data-operations). Pour plus d’informations sur la création de rôles personnalisés Azure, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).


## <a name="assign-azure-roles-using-the-azure-portal"></a>Attribuer des rôles Azure à l’aide du portail Azure  
Pour en savoir plus sur la gestion de l’accès aux ressources Azure à l’aide d’Azure RBAC et du portail Azure, consultez [cet article](..//role-based-access-control/role-assignments-portal.md). 

Après avoir déterminé l’étendue appropriée pour une attribution de rôle, accédez à cette ressource dans le portail Azure. Affichez les paramètres Contrôle d’accès (IAM) pour la ressource et suivez ces instructions pour gérer les attributions de rôle :

> [!NOTE]
> Les étapes décrites ci-dessous vous permettent d’attribuer un rôle à votre espace de noms Service Bus. Vous pouvez effectuer les mêmes étapes pour attribuer un rôle à d’autres étendues prises en charge (groupe de ressources, abonnement, etc.).

1. Dans le [portail Azure](https://portal.azure.com/), accédez à votre espace de noms Service Bus. Sélectionnez **Contrôle d’accès (IAM)** dans le menu de gauche pour afficher les paramètres du contrôle d’accès pour l’espace de noms. Si vous devez créer un espace de noms Service Bus, suivez les instructions données dans cet article : [Créer un espace de noms Service Bus Messaging](service-bus-create-namespace-portal.md).

    ![Sélectionner Contrôle d’accès dans le menu de gauche](./media/authenticate-application/select-access-control-menu.png)
1. Sélectionnez l’onglet **Attributions de rôles** pour afficher la liste des attributions de rôles. Sélectionnez le bouton **Ajouter** dans la barre d’outils, puis sélectionnez **Ajouter une attribution de rôle**. 

    ![Ajouter un bouton à la barre d’outils](./media/authenticate-application/role-assignments-add-button.png)
1. Sur la page **Ajouter une attribution de rôle** , procédez comme suit :
    1. Sélectionnez le **rôle Service Bus** que vous souhaitez attribuer. 
    1. Recherchez le **principal de sécurité** (utilisateur, groupe, principal de service) auquel vous souhaitez attribuer le rôle.
    1. Sélectionnez **Enregistrer** pour enregistrer l’attribution de rôle. 

        ![Attribuer un rôle à un utilisateur](./media/authenticate-application/assign-role-to-user.png)
    4. L’identité à laquelle vous avez attribué le rôle apparaît sous ce dernier. Par exemple, dans l’image suivante, Azure-users a le rôle Propriétaire de données Azure Service Bus. 
        
        ![Utilisateur dans la liste](./media/authenticate-application/user-in-list.png)

Vous pouvez suivre des étapes similaires pour attribuer un rôle dans l’étendue d’un groupe de ressources ou d’un abonnement. Une fois que vous avez défini le rôle et son étendue, vous pouvez tester ce comportement à l’aide des [exemples disponibles sur GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl).


## <a name="authenticate-from-an-application"></a>Authentifier à partir d’une application
L’un des principaux avantages d’utiliser Azure AD avec Service Bus est que vous n’avez plus besoin de stocker vos informations d’identification dans votre code. À la place, vous pouvez demander un jeton d’accès OAuth 2.0 sur la plateforme d’identités Microsoft. Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne le jeton d’accès à l’application, laquelle peut ensuite l’utiliser pour autoriser les requêtes adressées à Azure Service Bus.

Les sections suivantes vous montrent comment configurer votre application native ou une application web pour l’authentification avec la plateforme d’identités Microsoft 2.0. Pour plus d’informations sur la plateforme d’identité Microsoft 2.0, veuillez consulter l’article [Présentation de la plateforme d’identités Microsoft (v2.0)](../active-directory/develop/v2-overview.md).

Pour avoir une vue d’ensemble du flux d’octroi de code OAuth 2.0, consultez [Autoriser l’accès aux applications web Azure Active Directory à l’aide du flux d’octroi de code OAuth 2.0](../active-directory/develop/v2-oauth2-auth-code-flow.md).

### <a name="register-your-application-with-an-azure-ad-tenant"></a>Inscrire votre application à un locataire Azure AD
La première étape d’utilisation d’Azure AD pour autoriser l’accès aux entités Service Bus est d’inscrire votre application cliente avec un locataire Azure AD à partir du [portail Azure](https://portal.azure.com/). Lorsque vous inscrivez votre application cliente, vous fournissez des informations la concernant à Azure AD. Azure AD fournit ensuite un ID de client (appelé aussi ID d’application) que vous pouvez utiliser pour associer votre application au runtime Azure AD. Pour en savoir plus sur l’ID de client, consultez [Objets application et principal du service dans Azure Active Directory](../active-directory/develop/app-objects-and-service-principals.md). 

Les illustrations suivantes montrent les étapes d’inscription d’une application web :

![Inscrire une application](./media/authenticate-application/app-registrations-register.png)

> [!Note]
> Si vous inscrivez votre application comme une application native, vous pouvez spécifier n’importe quel URI valide pour l’URI de redirection. Pour les applications natives, cette valeur ne devra pas être une URL réelle. Pour les applications web, l’URI de redirection doit être un URI valide, car il spécifie l’URL à laquelle les jetons sont fournis.

Une fois votre application inscrite, **l’ID (client) d’application** se trouve sous **Paramètres** :

![ID de l’application inscrite](./media/authenticate-application/application-id.png)

Pour plus d’informations sur l’inscription d’une application dans Azure AD, consultez [Intégration d’applications à Azure Active Directory](../active-directory/develop/quickstart-register-app.md).

> [!IMPORTANT]
> Notez les valeurs de **TenantId** et **ApplicationId** , car vous en aurez besoin pour exécuter l’application.

### <a name="create-a-client-secret"></a>Créer une clé secrète client   
L’application a besoin d’une clé secrète client pour prouver son identité lors de la requête de jeton. Pour ajouter le secret client, effectuez ces étapes.

1. Accédez à la page d’inscription de votre application dans le portail Azure si vous n’y êtes pas déjà.
1. Dans le menu de gauche, sélectionnez **Certificats et secrets**.
1. Sous **Secrets client** , sélectionnez **Nouveau secret client** pour créer un secret.

    ![Nouveau secret client (bouton)](./media/authenticate-application/new-client-secret-button.png)
1. Entrez une description pour le secret, choisissez un intervalle d’expiration, puis sélectionnez **Ajouter**.

    ![Ajouter un secret client (page)](./media/authenticate-application/add-client-secret-page.png)
1. Copiez immédiatement la valeur de la nouvelle clé secrète dans un emplacement sécurisé. La valeur de remplissage s’affiche pour vous une seule fois.

    ![Clé secrète client](./media/authenticate-application/client-secret.png)

### <a name="permissions-for-the-service-bus-api"></a>Autorisations pour l’API Service Bus
Si votre application est une application console, vous devez inscrire une application native et ajouter des autorisations d’API pour **Microsoft.ServiceBus** dans l’ensemble des **autorisations requises**. Les applications natives ont également besoin d’un **URI de redirection** dans Azure AD, qui servira d’identificateur. Cet URI n’est pas obligatoirement une destination réseau. Utilisez `https://servicebus.microsoft.com` pour cet exemple, étant donné que l’exemple de code utilise déjà cet URI.

### <a name="client-libraries-for-token-acquisition"></a>Bibliothèques clientes pour l’acquisition de jeton  
Une fois que vous avez inscrit votre application et lui avez accordé les autorisations pour envoyer/recevoir des données dans Azure Service Bus, ajoutez le code nécessaire à votre application pour authentifier un principal de sécurité et acquérir un jeton OAuth 2.0. Pour l’authentification et l’acquisition du jeton, vous pouvez utiliser l’une des [bibliothèques d’authentification de la plateforme d’identités Microsoft](../active-directory/develop/reference-v2-libraries.md) ou toute autre bibliothèque open source prenant en charge OpenID ou Connect 1.0. Votre application peut alors utiliser le jeton d’accès pour autoriser une requête adressée à Azure Service Bus.

Pour obtenir la liste de scénarios pour lesquels l’acquisition de jetons est pris en charge, veuillez consulter la section [Scénarios](https://aka.ms/msal-net-scenarios) du référentiel GitHub [Microsoft Authentication Library (MSAL) pour .NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet).

## <a name="sample-on-github"></a>Exemple sur GitHub
Consultez l’exemple suivant sur GitHub : [Contrôle d'accès en fonction du rôle Azure pour Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl). 

Utilisez l’option de **connexion avec le secret client** , et non l’option de **connexion de l’utilisateur interactif**. Si vous utilisez l’option du secret client, vous ne voyez pas de fenêtre indépendante s’afficher. L’application utilise l’ID de locataire et l’ID d’application pour l’authentification. 

### <a name="run-the-sample"></a>Exécution de l'exemple

Avant d’exécuter l’exemple, modifiez le fichier **app.config** et, selon votre scénario, définissez les valeurs suivantes :

- `tenantId`: défini avec la valeur **TenantId**.
- `clientId`: défini avec la valeur **ApplicationId**.
- `clientSecret`: si vous souhaitez vous connecter à l’aide du secret client, créez-le dans Azure AD. En outre, utilisez une application web ou une API au lieu d’une application native. Ajoutez également l’application sous **Contrôle d’accès (IAM)** dans l’espace de noms que vous avez créé précédemment.
- `serviceBusNamespaceFQDN`: défini sur le nom DNS complet de l’espace de noms Service Bus que vous venez de créer, par exemple, `example.servicebus.windows.net`.
- `queueName`: défini sur le nom de la file d’attente que vous avez créé.
- URI de redirection que vous avez spécifié dans votre application lors des étapes précédentes.

Quand vous exécutez l’application console, vous êtes invité à sélectionner un scénario. Sélectionnez l’option de **connexion de l’utilisateur interactif** en tapant le numéro de l’utilisateur et en appuyant sur Entrée. L’application affiche une fenêtre de connexion, vous demande l’autorisation d’accéder à Service Bus et utilise ensuite le service à exécuter via le scénario d’envoi/réception à l’aide de l’identité de connexion.


## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur Azure RBAC, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) ?
- Pour apprendre à attribuer et gérer les rôles Azure avec Azure PowerShell, Azure CLI ou l’API REST, consultez les articles suivants :
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure PowerShell](../role-based-access-control/role-assignments-powershell.md)  
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../role-based-access-control/role-assignments-cli.md)
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](../role-based-access-control/role-assignments-rest.md)
    - [Ajouter ou supprimer des attributions de rôle Azure à l’aide de modèles Azure Resource Manager](../role-based-access-control/role-assignments-template.md)

Pour en savoir plus sur la messagerie Service Bus, voir les rubriques suivantes.

- [Exemples Azure RBAC pour Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/RoleBasedAccessControl)
- [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
- [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
- [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)