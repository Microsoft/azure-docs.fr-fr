---
title: Ajouter des rôles d’application et les obtenir à partir d’un jeton | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment ajouter des rôles d’application à une application inscrite dans Azure Active Directory, comment affecter des utilisateurs et des groupes à ces rôles et les recevoir dans la revendication Rôles au sein du jeton.
services: active-directory
author: kalyankrishna1
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 11/13/2020
ms.author: kkrishna
ms.reviewer: marsma, kkrishna, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 96c52c46a75d6d5810dfddf91439c275d14e85f1
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616135"
---
# <a name="how-to-add-app-roles-to-your-application-and-receive-them-in-the-token"></a>Procédure : ajouter des rôles d’application dans votre application et les recevoir dans le jeton

Le contrôle d’accès en fonction du rôle (RBAC) est un mécanisme populaire pour appliquer l’autorisation dans des applications. En utilisant RBAC, un administrateur accorde des autorisations aux rôles, pas à des utilisateurs individuels ou à des groupes. L’administrateur peut alors attribuer des rôles aux différents utilisateurs et groupes pour contrôler l’accès au contenu et aux fonctionnalités.

En utilisant le RBAC avec des rôles d’application et des revendications de rôle, les développeurs peuvent mettre en place des autorisations en toute sécurité dans leurs applications sans beaucoup d’efforts.

Une autre approche consiste à utiliser des groupes Azure AD et des revendications de groupe, comme indiqué dans l’exemple de code [active-directory-aspnetcore-webapp-openidconnect-v2](https://aka.ms/groupssample) sur GitHub. Les groupes Azure AD et les rôles d’application ne sont pas mutuellement exclusifs et peuvent être utilisés conjointement pour offrir un contrôle d’accès encore plus précis.

## <a name="declare-roles-for-an-application"></a>Déclarer des rôles pour une application

Vous définissez des rôles d’application à l’aide du [portail Azure](https://portal.azure.com). Quand un utilisateur se connecte à l’application, Azure AD émet une revendication `roles` pour chaque rôle qui lui a été accordé individuellement ou de par son appartenance à un groupe.

Il existe deux façons de déclarer des rôles d’application à l’aide du portail Azure :

* [Interface utilisateur des rôles d’application](#app-roles-ui--preview) | Préversion
* [Éditeur de manifeste d’application](#app-manifest-editor)

Le nombre de rôles que vous ajoutez est compté pour les limites définies pour un manifeste d’application par Azure Active Directory. Pour plus d’informations sur ces limites, consultez la section [Limites du manifeste](./reference-app-manifest.md#manifest-limits) de [Référence du manifeste d’application Azure Active Directory](reference-app-manifest.md).

### <a name="app-roles-ui--preview"></a>Interface utilisateur des rôles d’application | Préversion

> [!IMPORTANT]
> Fonctionnalité de l’interface utilisateur du portail des rôles d’application [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

Pour créer un rôle d’application à l’aide de l’interface utilisateur du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **répertoire + abonnement** dans le menu supérieur, puis choisissez l’abonné Azure Active Directory qui contient l’inscription de l’application à laquelle vous souhaitez ajouter un rôle d’application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications**, puis sélectionnez l’application dans laquelle vous souhaitez définir des rôles d’application.
1. Sélectionnez **Rôles d’application | Aperçu**, puis **Créer un rôle d’application**.

   :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-overview-pane.png" alt-text="Volet Rôles d’application d’une inscription d’application dans le portail Azure":::
1. Dans le volet **Créer un rôle d’application** , entrez les paramètres du rôle. Le tableau qui suit l’image décrit chaque réglage et leurs paramètres.

    :::image type="content" source="media/howto-add-app-roles-in-azure-ad-apps/app-roles-create-context-pane.png" alt-text="Volet contextuel de création de rôles d’application d’une inscription d’application dans le portail Azure":::

    | Champ | Description | Exemple |
    |-------|-------------|---------|
    | **Nom complet** | Nom d'affichage du rôle d’application qui apparaît lors du consentement de l'administrateur et de l'affectation de l'application. Cette valeur peut contenir des espaces. | `Survey Writer` |
    | **Types de membres autorisés** | Spécifie si ce rôle d’application peut être attribué aux utilisateurs, aux applications ou aux deux.<br/><br/>Lorsqu’ils sont disponibles pour `applications`, les rôles d’application s’affichent en tant que permissions d’application sous la section **Gérer** > **Autorisations d’API > Ajouter une autorisation > Mes API > Choisir une API > Permissions d’application**, lors de l’inscription d’une application. | `Users/Groups` |
    | **Valeur** | Spécifie la valeur de la revendication des rôles que l'application doit attendre dans le jeton. La valeur doit correspondre exactement à la chaîne référencée dans le code de l’application. La valeur ne peut pas contenir d’espaces. | `Survey.Create` |
    | **Description** | Description plus détaillée du rôle d’application affiché pendant les expériences d’affectation et de consentement des applications d’administration. | `Writers can create surveys.` |
    | **Voulez-vous activer ce rôle d'application ?** | Spécifie si le rôle d’application est activé. Pour supprimer un rôle d’application, décochez cette case et appliquez la modification avant de tenter l’opération de suppression. | *Activée* |

1. Sélectionnez **Appliquer** pour enregistrer vos modifications.

### <a name="app-manifest-editor"></a>Éditeur de manifeste d’application

Pour ajouter des rôles en modifiant directement le manifeste :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **répertoire + abonnement** dans le menu supérieur, puis choisissez l’abonné Azure Active Directory qui contient l’inscription de l’application à laquelle vous souhaitez ajouter un rôle d’application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications**, puis sélectionnez l’application dans laquelle vous souhaitez définir des rôles d’application.
1. Là encore, sous **Gérer**, sélectionnez **Manifeste**.
1. Modifiez le manifeste d’application en recherchant le paramètre `appRoles` et en ajoutant vos rôles d'application. Vous pouvez définir des rôles d’application qui ciblent `users`, `applications`, ou les deux. Les extraits de code JSON suivants montrent des exemples pour les deux.
1. Enregistrez le manifeste.

Chaque définition de rôle d’application dans le manifeste doit avoir un GUID unique pour sa valeur `id`.

La propriété `value` de chaque définition de rôle d’application doit correspondre exactement aux chaînes qui sont utilisées dans le code de l’application. La propriété `value` ne peut pas contenir d’espaces. Si tel est le cas, vous recevrez une erreur lors de l’enregistrement du manifeste.

#### <a name="example-user-app-role"></a>Exemple : Rôle d’application Utilisateur

Cet exemple définit un rôle d’application nommé `Writer` que vous pouvez assigner à un `User` :

```json
"appId": "8763f1c4-0000-0000-0000-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "Writer",
      "id": "d1c2ade8-0000-0000-0000-6d06b947c66f",
      "isEnabled": true,
      "description": "Writers Have the ability to create tasks.",
      "value": "Writer"
    }
  ],
"availableToOtherTenants": false,
```

#### <a name="example-application-app-role"></a>Exemple : Rôle d’application Application

Lorsqu’ils sont disponibles pour `applications`, les rôles d’application s’affichent en tant que permissions d’application sous la section **Gérer** > **Autorisations d’API > Ajouter une autorisation > Mes API > Choisir une API > Permissions d’application**, lors de l’inscription d’une application.

Cet exemple montre un rôle d’application ciblé sur une `Application` :

```json
"appId": "8763f1c4-0000-0000-0000-158e9ef97d6a",
"appRoles": [
    {
      "allowedMemberTypes": [
        "Application"
      ],
      "displayName": "ConsumerApps",
      "id": "47fbb575-0000-0000-0000-0f7a6c30beac",
      "isEnabled": true,
      "description": "Consumer apps have access to the consumer data.",
      "value": "Consumer"
    }
  ],
"availableToOtherTenants": false,
```

## <a name="assign-users-and-groups-to-roles"></a>Affecter des utilisateurs et des groupes à des rôles

Une fois que vous avez ajouté des rôles d’application à votre application, vous pouvez leur affecter des utilisateurs et des groupes. L’attribution des utilisateurs et des groupes aux rôles peut être effectuée via l’interface utilisateur du portail ou par programmation à l’aide de [Microsoft Graph](/graph/api/user-post-approleassignments). Lorsque les utilisateurs assignés aux différents rôles d’application se connectent à l’application, leurs jetons portent les rôles qui leur ont été attribués dans la revendication `roles`.

Pour attribuer des rôles aux utilisateurs et aux groupes à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans **Azure Active Directory**, sélectionnez **Applications d’entreprise** dans le menu de navigation de gauche.
1. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications. Si votre application n’apparaît pas dans la liste, utilisez les filtres présents en haut de la liste **Toutes les applications** pour restreindre la liste ou bien faites-la défiler vers le bas pour localiser votre application.
1. Sélectionnez l’application dans laquelle vous souhaitez assigner des utilisateurs ou un groupe de sécurité aux rôles.
1. Sous **Gérer**, sélectionnez **Utilisateurs et groupes**.
1. Sélectionnez **Ajouter un membre** pour ouvrir le volet **Ajouter une attribution**.
1. Sélectionnez le sélecteur **Utilisateurs et groupes** à partir du volet **Ajouter une attribution**. Une liste d’utilisateurs et de groupes de sécurité s’affiche. Vous pouvez rechercher un utilisateur ou un groupe et sélectionner plusieurs utilisateurs et groupes qui s’affichent dans la liste.
1. Une fois que vous avez sélectionné des utilisateurs et des groupes, sélectionnez le bouton **Sélectionner** pour continuer.
1. Sélectionnez **Sélectionner un rôle** dans le volet **Ajouter une attribution**. Tous les rôles que vous avez définis pour l’application s’affichent.
1. Choisissez un rôle et sélectionnez **Sélectionner**.
1. Appuyez sur le bouton **Attribuer** pour terminer l’attribution des utilisateurs et des groupes à l’application.

Vérifiez que les utilisateurs et les groupes ajoutés figurent dans la liste **Utilisateurs et groupes**.

## <a name="assign-app-roles-to-applications"></a>Attribuer des rôles d’application aux applications

Une fois que vous avez ajouté des rôles d’application dans votre application, vous pouvez attribuer un rôle d’application à une application cliente à l’aide du portail Azure ou par programmation à l’aide de [Microsoft Graph](/graph/api/user-post-approleassignments).

Lorsque vous attribuez des rôles d’application à une application, vous créez des *autorisations d’application*. Les autorisations d’application sont généralement utilisées par les applications démon ou les services back-end qui doivent s’authentifier et effectuer des appels d’API autorisés de façon autonome, sans interaction d’un utilisateur.

Pour attribuer des rôles d’application à une application à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans **Azure Active Directory**, sélectionnez **Inscriptions d'applications** dans le menu de navigation de gauche.
1. Sélectionnez **Toutes les applications** pour afficher la liste complète de vos applications. Si votre application n’apparaît pas dans la liste, utilisez les filtres présents en haut de la liste **Toutes les applications** pour restreindre la liste ou bien faites-la défiler vers le bas pour localiser votre application.
1. Sélectionnez l’application à laquelle vous souhaitez attribuer un rôle d’application.
1. Sélectionnez **Autorisations de l’API** > **Ajouter une autorisation**.
1. Sélectionnez l’onglet **Mes API**, puis l’application pour laquelle vous avez défini des rôles d’application.
1. Sélectionnez **Autorisations de l’application**.
1. Sélectionnez le ou les rôles vous souhaitez affecter.
1. Sélectionnez le bouton **Ajouter des autorisations** pour terminer l’ajout du ou des rôles.

Les nouveaux rôles ajoutés doivent apparaître dans le volet **Autorisations des API** lors de l’inscription de votre application.

#### <a name="grant-admin-consent"></a>Accorder un consentement administrateur

Étant donné qu’il s’agit *d’autorisations d’application*, et non d’autorisations déléguées, un administrateur doit accorder l’autorisation d’utiliser les rôles d’application attribués à l’application.

1. Dans le volet **Autorisations des API** lors de l’inscription de l’application, sélectionnez **Accorder un consentement d’administrateur pour \<tenant name\>** .
1. Sélectionnez **Oui** quand vous êtes invité à accorder un consentement pour les autorisations demandées.

La colonne **État** devrait indiquer **Accordé pour \<tenant name\>** .

## <a name="use-app-roles-in-your-web-api"></a>Utiliser des rôles d’application dans votre API Web

Une fois que vous avez défini des rôles d’application et que vous les avez affectés à un utilisateur, un groupe ou une application, l’étape suivante consiste à ajouter à votre API web un code qui vérifie ces rôles lorsque l’API est appelée. Autrement dit, lorsqu’une application cliente demande une opération d’API pour laquelle une autorisation est nécessaire, le code de votre API doit vérifier que les étendues se trouvent dans le jeton d’accès présenté dans l’appel de l’application cliente.

Pour savoir comment ajouter une autorisation à votre API web, consultez [API web protégée : Vérifier les étendues et les rôles d’application](scenario-protected-web-api-verification-scope-app-roles.md).

## <a name="app-roles-vs-groups"></a>Rôles d’application et groupes d’application

Même si vous pouvez utiliser des rôles ou des groupes d’application pour l’autorisation, les principales différences entre ces deux options peuvent influencer votre choix pour votre scénario.

| Rôles d'application                                                                          | Groupes                                                      |
|------------------------------------------------------------------------------------|-------------------------------------------------------------|
| Les groupes sont spécifiques à une application et définis dans l’inscription de l’application. Ils suivent l’application. | Ils ne sont pas spécifiques à une application, mais à un locataire Azure AD. |
| Les rôles d’application sont supprimés lorsque leur inscription d’application est supprimée.                      | Les groupes restent intacts même si l’application est supprimée.            |
| Fournis dans la revendication `roles`.                                                     | Fournis dans la revendication `groups`.                                 |

Les développeurs peuvent utiliser des rôles d’application pour contrôler si un utilisateur peut se connecter à une application ou si une application peut obtenir un jeton d’accès pour une API web. Pour étendre ce contrôle de sécurité aux groupes, les développeurs et les administrateurs peuvent également attribuer des groupes de sécurité à des rôles d’application.

Les rôles d’application sont préférés par les développeurs lorsqu’ils souhaitent décrire et contrôler eux-mêmes les paramètres d’autorisation dans leur application. Par exemple, une application qui utilise des groupes pour l’autorisation s’arrêtera au locataire suivant, car l’ID de groupe et le nom peuvent être différents. Une application utilisant des rôles d’application reste sûre. En fait, l’affectation de groupes à des rôles d’application est courante avec les applications SaaS pour les mêmes raisons.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les rôles des applications, consultez les ressources suivantes.

* Exemples de code sur GitHub
  * [Ajouter une autorisation à une application web ASP.NET Core à l’aide de groupes et revendications de groupes](https://aka.ms/groupssample)
  * [Application monopage Angular (SPA) appelant une API web .NET Core et utilisant des rôles d’application et des groupes de sécurité](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups/blob/master/README.md)
* Documentation de référence
  * [Manifeste de l’application Azure AD](./reference-app-manifest.md)
  * [Jetons d’accès Azure AD](access-tokens.md)
  * [Jetons d’ID Azure AD](id-tokens.md)
  * [Fournir des revendications facultatives à votre application](active-directory-optional-claims.md)
* Vidéo : [Implémenter l’autorisation dans vos applications avec la plateforme d’identité Microsoft (vidéo)](https://www.youtube.com/watch?v=LRoc-na27l0) (1:01:15)
