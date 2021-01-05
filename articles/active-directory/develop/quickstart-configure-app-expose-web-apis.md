---
title: 'Démarrage rapide : Inscrire et exposer une API web | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, vous allez inscrire une API web auprès de la plateforme d’identités Microsoft et configurer ses étendues, en l’exposant à des clients pour l’accès basé sur les autorisations aux ressources de l’API.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/03/2020
ms.author: marsma
ms.custom: aaddev, contperf-fy21q1
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 4ebae7e97f9128bb9302e9076e71dd3164f0c29e
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97030958"
---
# <a name="quickstart-configure-an-application-to-expose-a-web-api"></a>Démarrage rapide : Configurer une application pour exposer une API web

Dans ce guide de démarrage rapide, vous allez inscrire une API web auprès de la plateforme d’identités Microsoft et l’exposer à des applications clientes en ajoutant un exemple d’étendue. En inscrivant votre API web et en l’exposant par le biais d’étendues, vous pouvez fournir un accès basé sur les autorisations à ses ressources pour les utilisateurs autorisés et les applications clientes qui accèdent à votre API.

## <a name="prerequisites"></a>Prérequis

* Un compte Azure avec un abonnement actif : [créez gratuitement un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Exécution du [Démarrage rapide : Configurer un locataire](quickstart-create-new-tenant.md)

## <a name="register-the-web-api"></a>Inscrire l’API web

Pour fournir un accès délimité aux ressources de votre API web, vous devez d’abord inscrire l’API auprès de la plateforme d’identités Microsoft.

1. Effectuez les étapes décrites dans la section **Inscrire une application** du guide de [Démarrage rapide : Inscrire une application auprès de la plateforme d’identités Microsoft](quickstart-register-app.md).
1. Ignorez les sections **Ajouter un URI de redirection** et **Configurer les paramètres de plateforme**. Vous n’avez pas besoin de configurer un URI de redirection pour une API web, car aucun utilisateur n’est connecté de manière interactive.
1. Ignorez la section **Ajouter des informations d’identification** pour le moment. Votre API a besoin de ses propres informations d’identification uniquement si elle accède à une API en aval. Ce scénario n’est pas abordé dans cet article.

Une fois votre API web inscrite, vous êtes prêt à ajouter les étendues que le code de votre API peut utiliser pour fournir une autorisation précise aux consommateurs de cette dernière.

## <a name="add-a-scope"></a>Ajouter une étendue

Le code d’une application cliente demande l’autorisation d’effectuer des opérations définies par votre API web en transmettant un jeton d’accès avec ses demandes à la ressource protégée (l’API web). Votre API web effectue ensuite l’opération demandée uniquement si le jeton d’accès qu’elle reçoit contient les étendues nécessaires.

Tout d’abord, effectuez les étapes suivantes pour créer un exemple d’étendue nommé `Employees.Read.All` :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/quickstart-configure-app-expose-web-apis/portal-01-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire contenant l’inscription de votre application cliente.
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications**, puis l’inscription d’application de votre API.
1. Sélectionnez **Exposer une API** > **Ajouter une étendue**.

    :::image type="content" source="media/quickstart-configure-app-expose-web-apis/portal-02-expose-api.png" alt-text="Volet Exposer une API d’une inscription d’application dans le portail Azure":::

1. Vous êtes invité à définir un **URI d’ID d’application** si vous n’en avez pas encore configuré un.

   L’URI d’ID d’application, qui doit être globalement unique, fait office de préfixe pour les étendues que vous référencerez dans le code de votre API. Vous pouvez utiliser la valeur par défaut fournie, qui se présente sous la forme `api://<application-client-id>`, ou spécifier un URI plus lisible comme `https://contoso.com/api`.

1. Spécifiez ensuite les attributs de l’étendue dans le volet **Ajouter une étendue**. Pour cette procédure pas à pas, vous pouvez utiliser les exemples de valeurs ou spécifier les vôtres.

    | Champ | Description | Exemple |
    |-------|-------------|---------|
    | **Nom de l'étendue** | Nom de votre étendue. Une convention de nommage d’étendue courante est `resource.operation.constraint`. | `Employees.Read.All` |
    | **Qui peut donner son consentement** | Indique si cette étendue peut être consentie par des utilisateurs ou si le consentement d’un administrateur est nécessaire. Sélectionnez **administrateurs uniquement** pour des autorisations à privilèges élevés. | **Administrateurs et utilisateurs** |
    | **Nom d'affichage du consentement administrateur** | Courte description de l’objectif de l’étendue que seuls les administrateurs verront. | `Read-only access to Employee records` |
    | **Description du consentement de l'administrateur** | Description plus détaillée de l’autorisation accordée par l’étendue que seuls les administrateurs verront. | `Allow the application to have read-only access to all Employee data.` |
    | **Nom d'affichage du consentement de l'utilisateur** | Courte description de l’objectif de l’étendue. Affichée aux utilisateurs uniquement si vous définissez **Qui peut donner son consentement** sur **Administrateurs et utilisateurs**. | `Read-only access to your Employee records` |
    | **Description du consentement de l'utilisateur** | Description plus détaillée de l’autorisation accordée par l’étendue. Affichée aux utilisateurs uniquement si vous définissez **Qui peut donner son consentement** sur **Administrateurs et utilisateurs**. | `Allow the application to have read-only access to your Employee data.` |

1. Définissez l’**État** sur **Activé**, puis sélectionnez **Ajouter une étendue**.

1. (Facultatif) Pour supprimer les demandes de consentement des utilisateurs de votre application pour les étendues que vous avez définies, vous pouvez *préautoriser* l’application cliente à accéder à votre API web. Préautorisez *uniquement* les applications clientes que vous approuvez, car vos utilisateurs n’auront pas la possibilité de refuser le consentement.
    1. Sous **Applications clientes autorisées**, sélectionnez **Ajouter une application cliente**.
    1. Entrez l’**ID d’application (client)** de l’application cliente que vous souhaitez préautoriser, par exemple celui d’une application web que vous avez inscrite précédemment.
    1. Sous **Étendues autorisées**, sélectionnez les étendues pour lesquelles vous souhaitez supprimer les invites de consentement, puis sélectionnez **Ajouter une application**.

    Si vous avez effectué cette étape facultative, l’application cliente est désormais une application cliente préautorisée, et les utilisateurs ne sont pas invités à donner leur consentement quand ils s’y connectent.

## <a name="add-a-scope-requiring-admin-consent"></a>Ajouter une étendue nécessitant un consentement administrateur

Ajoutez ensuite un autre exemple d’étendue nommé `Employees.Write.All` auquel seuls les administrateurs peuvent donner leur consentement. Les étendues qui nécessitent le consentement de l’administrateur sont généralement utilisées pour fournir l’accès à des opérations avec des privilèges plus élevés, et souvent par des applications clientes qui s’exécutent en tant que services back-end ou en tant que démons qui ne connectent pas un utilisateur de manière interactive.

Pour ajouter l’exemple d’étendue `Employees.Write.All`, effectuez les étapes décrites dans la section [Ajouter une étendue](#add-a-scope), puis spécifiez ces valeurs dans le volet **Ajouter une étendue** :

| Champ                          | Valeur d'exemple                                                      |
|--------------------------------|--------------------------------------------------------------------|
| **Nom de l'étendue**                 | `Employees.Write.All`                                              |
| **Qui peut donner son consentement**            | **Administrateurs uniquement**                                                    |
| **Nom d'affichage du consentement administrateur** | `Write access to Employee records`                                 |
| **Description du consentement de l'administrateur**  | `Allow the application to have write access to all Employee data.` |
| **Nom d'affichage du consentement de l'utilisateur**  | *Aucune (laisser vide)*                                               |
| **Description du consentement de l'utilisateur**   | *Aucune (laisser vide)*                                               |

## <a name="verify-the-exposed-scopes"></a>Vérifier les étendues exposées

Si vous avez correctement ajouté les deux exemples d’étendues décrits dans les sections précédentes, ils apparaissent dans le volet **Exposer une API** de l’inscription d’application de votre API web, semblable à cette image :

:::image type="content" source="media/quickstart-configure-app-expose-web-apis/portal-03-scopes-list.png" alt-text="Capture d’écran du volet Exposer une API montrant deux étendues exposées.":::

Comme indiqué dans l’image, la chaîne complète d’une étendue est la concaténation de l’**URI d’ID d’application** de votre API web et du **Nom de l’étendue**.

Par exemple, si l’URI d’ID d’application de votre API web est `https://contoso.com/api`, et que le nom de l’étendue est `Employees.Read.All`, l’étendue complète est :

`https://contoso.com/api/Employees.Read.All`

## <a name="using-the-exposed-scopes"></a>Utilisation des étendues exposées

Dans l’article suivant de la série, vous allez configurer l’inscription d’une application cliente avec un accès à votre API web et les étendues que vous avez définies en suivant les étapes de cet article.

Une fois qu’une inscription d’application cliente est autorisée à accéder à votre API web, la plateforme d’identités Microsoft peut émettre un jeton d’accès OAuth 2.0 pour le client. Quand le client appelle l’API web, il présente un jeton d’accès dont la revendication d’étendue (`scp`) est définie sur les autorisations que vous avez spécifiées dans l’inscription d’application du client.

Vous pouvez exposer des étendues supplémentaires ultérieurement si nécessaire. Considérez que votre API web peut exposer plusieurs étendues associées à plusieurs opérations. Votre ressource peut contrôler l’accès à l’API web lors de l’exécution, en évaluant la ou les revendications de l’étendue (`scp`) dans le jeton d’accès OAuth 2.0 qu’elle reçoit.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez exposé votre API web en configurant ses étendues, configurez l’inscription de votre application cliente avec l’autorisation d’accès aux étendues.

> [!div class="nextstepaction"]
> [Configurer une inscription d’application pour l’accès à une API web](quickstart-configure-app-access-web-apis.md)

<!-- REF LINKS -->
[ms-graph-application]: /graph/api/resources/application
