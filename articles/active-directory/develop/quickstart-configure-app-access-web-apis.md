---
title: 'Démarrage rapide : Configurer une application pour accéder à une API web | Azure'
titleSuffix: Microsoft identity platform
description: Dans ce guide de démarrage rapide, vous allez configurer une inscription d’application représentant une API web dans la plateforme d’identités Microsoft afin d’activer l’accès aux ressources délimité (autorisations) pour les applications clientes.
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
ms.reviewer: lenalepa, aragra, sureshja
ms.openlocfilehash: 2630984d54134ca8fca94fdc1960265ba71935a6
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031298"
---
# <a name="quickstart-configure-a-client-application-to-access-a-web-api"></a>Démarrage rapide : Configurer une application cliente pour accéder à une API web

Dans ce guide de démarrage rapide, vous fournissez à une application cliente inscrite auprès de la plateforme d’identités Microsoft un accès délimité basé sur les autorisations à votre propre API web. Vous fournissez également à l’application cliente l’accès à Microsoft Graph.

En spécifiant les étendues d’une API web dans l’inscription de votre application cliente, l’application cliente peut obtenir un jeton d’accès contenant ces étendues à partir de la plateforme d’identités Microsoft. Dans son code, l’API web peut ensuite fournir un accès basé sur les autorisations à ses ressources en fonction des étendues se trouvant dans le jeton d’accès.

## <a name="prerequisites"></a>Prérequis

* Un compte Azure avec un abonnement actif : [créez gratuitement un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Exécution du [Démarrage rapide : Inscrire une application](quickstart-register-app.md)
* Exécution du [Démarrage rapide : Configurer une application pour exposer une API web](quickstart-configure-app-expose-web-apis.md)

## <a name="add-permissions-to-access-your-web-api"></a>Ajouter des autorisations pour accéder à votre API web

Dans le premier scénario, vous accordez à une application cliente l’accès à votre propre API web, que vous devez avoir toutes les deux inscrites dans le cadre des prérequis. Si vous n’avez pas encore inscrit une application cliente et une API web, effectuez les étapes décrites dans les deux articles [Prérequis](#prerequisites).

Ce diagramme montre comment les deux inscriptions d’applications sont liées entre elles. Dans cette section, vous allez ajouter des autorisations à l’inscription de l’application cliente.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/diagram-01-app-permission-to-api-scopes.svg" alt-text="Diagramme linéaire montrant une API web avec, à droite, des étendues exposées et, à gauche, une application cliente où ces étendues sont sélectionnées comme autorisations" border="false":::

Une fois que vous avez inscrit votre application cliente et votre API web et que vous avez exposé l’API en créant des étendues, vous pouvez configurer les autorisations du client sur l’API en effectuant les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire contenant l’inscription de votre application cliente.
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications**, puis votre application cliente (et *pas* votre API web).
1. Sélectionnez **Autorisations des API** > **Ajouter une autorisation** > **Mes API**.
1. Sélectionnez l’API web que vous avez inscrite dans le cadre des prérequis.

    Les **autorisations déléguées** sont sélectionnées par défaut. Les autorisations déléguées sont appropriées pour les applications clientes qui accèdent à une API web en tant qu’utilisateur connecté, et dont l’accès doit être limité aux autorisations que vous sélectionnez à l’étape suivante. Laissez l’option **Autorisations déléguées** sélectionnée pour cet exemple.

    Les **autorisations d’application** sont pour les applications de type service ou démon qui doivent accéder à une API web de façon autonome, sans interaction utilisateur pour la connexion ou le consentement. Si vous n’avez pas défini de rôles d’application pour votre API web, cette option est désactivée.
1. Sous **Sélectionner des autorisations**, développez la ressource dont vous avez défini les étendues pour votre API web, puis sélectionnez les autorisations dont l’application cliente doit disposer pour le compte de l’utilisateur connecté.

    Si vous avez utilisé les exemples de noms d’étendues spécifiés dans le guide de démarrage rapide précédent, vous devez voir **Employees.Read.All** et **Employees.Write.All**.
    Sélectionnez **Employees.Read.All** ou une autre autorisation que vous avez créée quand vous avez rempli les prérequis.
1. Sélectionnez **Ajouter des autorisations** pour terminer le processus.

Une fois les autorisations ajoutées à votre API, vous devez voir les autorisations sélectionnées sous **Autorisations configurées**. L’image suivante montre l’exemple d’autorisation déléguée *Employees.Read.All* ajoutée à l’inscription de l’application cliente.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-02-configured-permissions-pane.png" alt-text="Volet Autorisations configurées du portail Azure présentant l’autorisation nouvellement ajoutée":::

Vous pouvez également remarquer l’autorisation *User.Read* pour l’API Microsoft Graph. Cette autorisation est automatiquement ajoutée quand vous inscrivez une application dans le portail Azure.

## <a name="add-permissions-to-access-microsoft-graph"></a>Ajouter des autorisations pour accéder à Microsoft Graph

En plus d’accéder à votre propre API web pour le compte de l’utilisateur connecté, votre application peut également avoir besoin d’accéder aux données de l’utilisateur (ou d’autres utilisateurs) stockées dans Microsoft Graph ou de les modifier. Ou vous disposez peut-être d’une application de service ou démon qui doit accéder eux-mêmes à Microsoft Graph, en effectuant les opérations sans aucune interaction utilisateur.

### <a name="delegated-permission-to-microsoft-graph"></a>Autorisation déléguée pour Microsoft Graph

Configurez l’autorisation déléguée pour Microsoft Graph afin de permettre à votre application cliente d’effectuer des opérations pour le compte de l’utilisateur connecté, par exemple lire ses e-mails ou modifier son profil. Par défaut, les utilisateurs de votre application cliente sont invités, au moment où ils se connectent, à donner leur consentement pour les autorisations déléguées que vous avez configurées pour celle-ci.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire contenant l’inscription de votre application cliente.
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications**, puis votre application cliente.
1. Sélectionnez **Autorisations de l’API** > **Ajouter une autorisation** > **Microsoft Graph**.
1. Sélectionnez **Autorisations déléguées**. Microsoft Graph expose de nombreuses autorisations, avec celles les plus couramment utilisées affichées en haut de la liste.
1. Sous **Sélectionner des autorisations**, sélectionnez les autorisations suivantes :

    | Autorisation       | Description                                         |
    |------------------|-----------------------------------------------------|
    | `email`          | Afficher l’adresse e-mail des utilisateurs                           |
    | `offline_access` | Conserver l’accès aux données auxquelles vous lui avez donné accès |
    | `openid`         | Connecter les utilisateurs                                       |
    | `profile`        | Afficher le profil de base des utilisateurs                           |
1. Sélectionnez **Ajouter des autorisations** pour terminer le processus.

Chaque fois que vous configurez des autorisations, les utilisateurs de votre application sont invités à se connecter afin de donner leur consentement pour autoriser votre application à accéder à l’API de ressource en leur nom.

En tant qu’administrateur, vous pouvez également accorder votre consentement pour le compte de *tous* les utilisateurs afin qu’ils ne soient pas invités à le faire. Le consentement administrateur est abordé plus loin dans la section [En savoir plus sur les autorisations d’API et le consentement administrateur](#more-on-api-permissions-and-admin-consent) de cet article.

### <a name="application-permission-to-microsoft-graph"></a>Autorisation d’application pour Microsoft Graph

Configurez des autorisations d’application pour une application qui doit s’authentifier de façon autonome, sans interaction ou consentement de l’utilisateur. Les autorisations d’application sont généralement utilisées par des services d’arrière-plan ou les applications démon qui accèdent à une API de manière « non interactive » et par des API web qui accèdent à une autre API (en aval).

Dans les étapes suivantes, vous accordez une autorisation à l’autorisation *Files.Read.All* de Microsoft Graph, par exemple.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/quickstart-configure-app-access-web-apis/portal-01-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire contenant l’inscription de votre application cliente.
1. Sélectionnez **Azure Active Directory** > **Inscriptions d’applications**, puis votre application cliente.
1. Sélectionnez **Autorisations de l’API** > **Ajouter une autorisation** > **Microsoft Graph** > **Autorisations de l’application**.
1. Toutes les autorisations exposées par Microsoft Graph sont affichées sous **Sélectionner des autorisations**.
1. Sélectionnez la ou les autorisations que vous voulez accorder à votre application. Par exemple, vous pouvez avoir une application démon qui analyse les fichiers de votre organisation, déclenchant des alertes sur un type ou un nom de fichier spécifique.

    Sous **Sélectionner des autorisations**, développez **Fichiers**, puis sélectionnez l’autorisation *Files.Read.All*.
1. Sélectionnez **Ajouter des autorisations**.

Certaines autorisations, comme *Files.Read.All* de Microsoft Graph, exigent un consentement administrateur. Vous accordez un consentement administrateur en sélectionnant le bouton **Accorder un consentement administrateur**, décrit plus loin dans la section [Bouton de consentement administrateur](#admin-consent-button).

### <a name="configure-client-credentials"></a>Configurer les informations d’identification du client

Les applications qui utilisent des autorisations d’application s’authentifient eux-mêmes en utilisant leurs propres informations d’identification, sans qu’aucune interaction utilisateur ne soit nécessaire. Pour que votre application (ou API) puisse accéder à Microsoft Graph, à votre propre API web ou à toute autre API en utilisant des autorisations d’application, vous devez configurer les informations d’identification de cette application cliente.

Pour plus d’informations sur la configuration des informations d’identification d’une application, consultez la section [Ajouter des informations d’identification](quickstart-register-app.md#add-credentials) du guide de [Démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](quickstart-register-app.md).

## <a name="more-on-api-permissions-and-admin-consent"></a>En savoir plus sur les autorisations d’API et le consentement administrateur

Le volet **Autorisations de l’API** d’une inscription d’application contient un tableau [Autorisations configurées](#configured-permissions), et peut également contenir un tableau [Autres autorisations accordées](#other-permissions-granted). Ces deux tableaux et le [bouton de consentement de l’administrateur](#admin-consent-button) sont décrits dans les sections suivantes.

### <a name="configured-permissions"></a>Autorisations configurées

Le tableau **Autorisations configurées** du volet **Autorisations de l’API** affiche la liste des autorisations dont votre application a besoin pour le fonctionnement de base : la liste d’*accès aux ressources nécessaires*. Les utilisateurs, ou leurs administrateurs, doivent donner leur consentement à ces autorisations avant d’utiliser votre application. D’autres autorisations facultatives peuvent être demandées ultérieurement au moment de l’exécution (à l’aide du consentement dynamique).

Il s’agit de la liste minimale des autorisations auxquelles les utilisateurs doivent donner leur consentement pour votre application. Il peut y en avoir d’autres, mais celles-ci sont toujours nécessaires. Pour des questions de sécurité et pour permettre aux utilisateurs et aux administrateurs de mieux maîtriser l’utilisation de votre application, ne demandez jamais quelque chose dont vous n’avez pas besoin.

Vous pouvez ajouter ou supprimer les autorisations qui s’affichent dans ce tableau en effectuant les étapes décrites ci-dessus ou celles mentionnées dans [Autres autorisations accordées](#other-permissions-granted) (dans la section suivante). En tant qu’administrateur, vous pouvez accorder un consentement administrateur pour l’ensemble complet des autorisations d’une API qui s’affichent dans le tableau et révoquer un consentement pour des autorisations individuelles.

### <a name="other-permissions-granted"></a>Autres autorisations accordées

Vous pouvez également voir un tableau intitulée **Autres autorisations accordées pour {votre locataire}** dans le volet **Autorisations de l’API**. Le tableau **Autres autorisations accordées pour {votre locataire}** présente les autorisations accordées pour le locataire et qui n’ont pas été explicitement configurées sur l’objet application. Ces autorisations ont été demandées et accordées dynamiquement. Cette section s’affiche uniquement s’il existe au moins une autorisation qui s’applique.

Vous pouvez ajouter l’ensemble complet des autorisations d’une API ou des autorisations individuelles s’affichant dans ce tableau au tableau **Autorisations configurées**. En tant qu’administrateur, vous pouvez révoquer un consentement administrateur pour des API ou pour des autorisations individuelles dans cette section.

### <a name="admin-consent-button"></a>Bouton de consentement administrateur

Le bouton **Accorder un consentement d’administrateur pour {votre locataire}** permet à un administrateur d’accorder le consentement administrateur aux autorisations configurées pour l’application. Quand vous sélectionnez ce bouton, une boîte de dialogue s’affiche pour vous demander de confirmer l’action de consentement.

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-03-grant-admin-consent-button.png" alt-text="Bouton Accorder un consentement d’administrateur mis en évidence dans le volet Autorisations configurées du portail Azure":::

Une fois le consentement accordé, les autorisations exigées par le consentement administrateur sont indiquées comme ayant reçu le consentement :

:::image type="content" source="media/quickstart-configure-app-access-web-apis/portal-04-admin-consent-granted.png" alt-text="Tableau Configurer les autorisations du portail Azure, montrant le consentement administrateur accordé pour l’autorisation Files.Read.All":::

Le bouton **Accorder un consentement administrateur** est *désactivé* si vous n’êtes pas administrateur ou si aucune autorisation n’a été configurée pour l’application. Si vous disposez d’autorisations qui ont été accordées mais qui n’ont pas encore été configurées, le bouton de consentement administrateur vous invite à gérer ces autorisations. Vous pouvez les ajouter aux autorisations configurées ou les supprimer.

## <a name="next-steps"></a>Étapes suivantes

Passez au démarrage rapide suivant de la série pour savoir comment configurer les types de compte pouvant accéder à votre application. Par exemple, vous souhaiterez peut-être limiter l’accès aux seuls utilisateurs de votre organisation (locataire unique) ou autoriser les utilisateurs d’autres locataires Azure AD (multilocataire) et ceux avec des comptes personnels Microsoft (MSA).

> [!div class="nextstepaction"]
> [Modifier les comptes pris en charge par une application](quickstart-modify-supported-accounts.md)