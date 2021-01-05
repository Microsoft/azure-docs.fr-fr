---
title: 'Démarrage rapide : Inscrire une application dans la plateforme d’identités Microsoft | Azure'
description: Dans ce guide de démarrage rapide, vous allez découvrir comment inscrire une application auprès de la plateforme d’identités Microsoft.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/03/2020
ms.author: marsma
ms.custom: aaddev, identityplatformtop40, contperf-fy21q1, contperf-fy21q2
ms.reviewer: aragra, lenalepa, sureshja
ms.openlocfilehash: 4e9e6c9179c7fbf405cf8fc661a209da3c9d3b61
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97031264"
---
# <a name="quickstart-register-an-application-with-the-microsoft-identity-platform"></a>Démarrage rapide : Inscrire une application avec la plateforme d’identités Microsoft

Dans ce guide de démarrage rapide, vous allez inscrire une application dans le portail Azure afin que la plateforme d’identités Microsoft puisse fournir des services d’authentification et d’autorisation pour votre application et ses utilisateurs.

Chaque application pour laquelle vous voulez que la plateforme d’identités Microsoft assure la gestion des identités et des accès (IAM) doit être inscrite. Qu’il s’agisse d’une application cliente telle qu’une application web ou mobile, ou d’une API web sur laquelle repose une application cliente, son inscription établit une relation d’approbation entre votre application et le fournisseur d’identité, à savoir la plateforme d’identités Microsoft.

## <a name="prerequisites"></a>Prérequis

* Un compte Azure avec un abonnement actif : [créez gratuitement un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Exécution du [Démarrage rapide : Configurer un locataire](quickstart-create-new-tenant.md)

## <a name="register-an-application"></a>Inscrire une application

L’inscription de votre application établit une relation d’approbation entre votre application et la plateforme d’identités Microsoft. L’approbation est unidirectionnelle : votre application approuve la plateforme d’identités Microsoft, et non le contraire.

Effectuez les étapes suivantes pour créer l’inscription d’application :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Spécifiez qui peut utiliser l’application. Ces personnes sont parfois appelées *audience de connexion*.

    | Types de comptes pris en charge | Description |
    |-------------------------|-------------|
    | **Comptes dans cet annuaire organisationnel uniquement** | Sélectionnez cette option si vous générez une application destinée à être utilisée uniquement par des utilisateurs (ou des invités) dans *votre* locataire.<br><br>Souvent appelée application *métier* (LOB), il s’agit d’une application **monolocataire** dans la plateforme d’identités Microsoft. |
    | **Comptes dans un annuaire organisationnel** | Sélectionnez cette option si vous souhaitez que les utilisateurs de *n’importe quel* locataire Azure AD puisse utiliser votre application. Cette option est appropriée si, par exemple, vous générez une application Logiciel en tant que service (SaaS) que vous envisagez de fournir à plusieurs organisations.<br><br>C’est une application dite **multilocataire** dans la plateforme d’identités Microsoft. |
    | **Comptes dans un annuaire organisationnel et comptes personnels Microsoft** | Sélectionnez cette option pour cibler l’ensemble le plus large de clients.<br><br>En sélectionnant cette option, vous inscrivez une application **multilocataire** qui peut également prendre en charge les utilisateurs disposant de **comptes Microsoft** (MSA) personnels. |
    | **Comptes Microsoft personnels** | Sélectionnez cette option si vous générez une application destinée à être utilisée uniquement par des utilisateurs disposant de comptes Microsoft personnels. Les comptes Microsoft personnels incluent les comptes Skype, Xbox, Live et Hotmail. |

1. N’entrez rien pour **URI de redirection (facultatif)** . Vous allez en configurer un dans la prochaine section.
1. Sélectionnez **Inscrire** pour procéder à l’inscription d’application initiale.

    :::image type="content" source="media/quickstart-register-app/portal-02-app-reg-01.png" alt-text="Capture d’écran du portail Azure dans un navigateur web montrant le volet Inscrire une application.":::

Une fois l’inscription terminée, le portail Azure affiche le volet **Vue d’ensemble** de l’inscription d’application, qui comprend son **ID d’application (client)** . Aussi simplement appelée *ID client*, cette valeur identifie de manière unique votre application dans la plateforme d’identités Microsoft.

Le code de votre application, ou plus généralement une bibliothèque d’authentification utilisée dans votre application, utilise également l’ID client comme un aspect de la validation des jetons de sécurité reçus de la plateforme d’identités.

:::image type="content" source="media/quickstart-register-app/portal-03-app-reg-02.png" alt-text="Capture d’écran du portail Azure dans un navigateur web montrant le volet Vue d’ensemble d’une inscription d’application.":::

## <a name="add-a-redirect-uri"></a>Ajouter un URI de redirection

Un URI de redirection est l’emplacement où la plateforme d’identités Microsoft redirige le client d’un utilisateur et envoie des jetons de sécurité après authentification.

Dans une application web de production, par exemple, l’URI de redirection est souvent un point de terminaison public sur lequel votre application s’exécute, comme `https://contoso.com/auth-response`. Pendant le développement, il est courant d’ajouter également le point de terminaison où vous exécutez votre application localement, comme `https://127.0.0.1/auth-response` ou `http://localhost/auth-response`.

Pour ajouter et modifier des URI de redirection pour vos applications inscrites, vous devez configurer leurs [paramètres de plateforme](#configure-platform-settings).

### <a name="configure-platform-settings"></a>Configurer des paramètres de plateforme

Les paramètres de chaque type d’application, dont les URI de redirection, sont configurés dans **Configurations de plateforme** dans le portail Azure. Certaines plateformes, comme le **web** et les **applications monopages**, nécessitent de spécifier manuellement un URI de redirection. Pour les autres plateformes comme les plateformes mobiles et de bureau, vous pouvez sélectionner des URI de redirection générés automatiquement quand vous configurez leurs autres paramètres.

Pour configurer des paramètres d’application en fonction de la plateforme ou de l’appareil ciblé :

1. Sélectionnez votre application dans les **Inscriptions d’applications** dans le portail Azure.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Sous **Configurations de plateformes**, sélectionnez **Ajouter une plateforme**.
1. Dans **Configurer des plateformes**, sélectionnez la vignette correspondant à votre type d’application (plateforme) pour configurer ses paramètres.

    :::image type="content" source="media/quickstart-register-app/portal-04-app-reg-03-platform-config.png" alt-text="Capture d’écran du volet Configuration de la plateforme dans le portail Azure" border="false":::

    | Plateforme | Paramètres de configuration |
    | -------- | ---------------------- |
    | **Web** | Entrez un **URI de redirection** pour votre application, qui est l’emplacement où la plateforme d’identités Microsoft redirige le client d’un utilisateur et envoie des jetons de sécurité après authentification.<br/><br/>Sélectionnez cette plateforme pour les applications web standard qui s’exécutent sur un serveur. |
    | **Application monopage** | Entrez un **URI de redirection** pour votre application, qui est l’emplacement où la plateforme d’identités Microsoft redirige le client d’un utilisateur et envoie des jetons de sécurité après authentification.<br/><br/>Sélectionnez cette plateforme si vous générez une application web côté client dans JavaScript ou avec un framework comme Angular, Vue.js, React.js ou Blazor WebAssembly. |
    | **iOS / macOS** | Entrez l’**ID de bundle** de l’application, qui se trouve dans Xcode dans *Info.plist* ou Paramètres de build.<br/><br/>Un URI de redirection est automatiquement généré quand vous spécifiez un ID de bundle. |
    | **Android** | Entrez le **Nom du package** de l’application, que vous trouverez dans le fichier *AndroidManifest.xml*. Générez et entrez ensuite le **hachage de signature**.<br/><br/>Un URI de redirection est automatiquement généré quand vous spécifiez ces paramètres. |
    | **Applications de bureau et mobiles** | Sélectionnez l’un des **URI de redirection suggérés** ou spécifiez un **URI de redirection personnalisé**.<br/>Pour les applications de bureau, nous vous recommandons d’effectuer les opérations suivantes :<br/>`https://login.microsoftonline.com/common/oauth2/nativeclient`<br/><br/>Sélectionnez cette plateforme pour les applications mobiles qui n’utilisent pas la dernière bibliothèque d’authentification Microsoft (MSAL) ou qui n’utilisent pas de répartiteur. Sélectionnez également cette plateforme pour les applications de bureau. |
1. Sélectionnez **Configurer** pour effectuer la configuration de la plateforme.

### <a name="redirect-uri-restrictions"></a>Restrictions applicables aux URI de redirection

Certaines restrictions s’appliquent au format des URI de redirection que vous ajoutez à une inscription d’application. Pour plus d’informations sur ces restrictions, consultez [Limitations et restrictions des URI de redirection (URL de réponse)](reply-url.md).

## <a name="add-credentials"></a>Ajouter les informations d’identification

Les informations d’identification sont utilisées par les applications clientes confidentielles qui accèdent à une API web. Les applications web, les autres API web ou les applications de type service et démon sont des exemples de clients confidentiels. Les informations d’identification permettent à votre application de s’authentifier de façon autonome, sans qu’aucune interaction utilisateur ne soit nécessaire au moment de l’exécution.

Vous pouvez ajouter des certificats et des secrets clients (une chaîne) en tant qu’informations d’identification à votre inscription d’application cliente confidentielle.

:::image type="content" source="media/quickstart-register-app/portal-05-app-reg-04-credentials.png" alt-text="Capture d’écran du portail Azure montrant le volet Certificats et secrets dans une inscription d’application":::

### <a name="add-a-certificate"></a>Ajouter un certificat

Parfois appelés *clé publique*, les certificats sont le type d’informations d’identification recommandé, car ils fournissent un niveau d’assurance plus élevé qu’un secret client.

1. Sélectionnez votre application dans les **Inscriptions d’applications** dans le portail Azure.
1. Sélectionnez **Certificats et secrets** > **Charger le certificat**.
1. Sélectionnez le fichier que vous voulez charger. Il doit s’agir d’un fichier de type .cer, .pem ou .crt.
1. Sélectionnez **Ajouter**.

### <a name="add-a-client-secret"></a>Ajouter un secret client

Le secret client, également appelé *mot de passe d’application*, est une valeur de chaîne que votre application peut utiliser à la place d’un certificat pour s’identifier. Souvent utilisé pendant le développement, c’est le plus simple à utiliser des deux types d’informations d’identification, mais il est considéré comme moins sécurisé qu’un certificat. Vous devez utiliser des certificats dans vos applications qui s’exécutent en production.

1. Sélectionnez votre application dans les **Inscriptions d’applications** dans le portail Azure.
1. Sélectionnez **Certificats et secrets** >  **Nouveau secret client**.
1. Ajoutez une description pour votre clé secrète client.
1. Sélectionnez une durée.
1. Sélectionnez **Ajouter**.
1. **Enregistrez la valeur du secret** pour une utilisation dans le code de votre application cliente. Une fois que vous avez quitté cette page, elle *ne s’affiche plus*.

## <a name="next-steps"></a>Étapes suivantes

Les applications clientes doivent généralement accéder aux ressources d’une API web. La plateforme d’identités Microsoft permet d’assurer la protection de votre application cliente, mais également d’autoriser l’accès délimité basé sur les autorisations à votre API web.

Passez au guide démarrage rapide suivant de la série afin de créer une autre inscription d’application pour votre API web et d’exposer ses étendues.

> [!div class="nextstepaction"]
> [Configurer une application pour exposer une API web](quickstart-configure-app-expose-web-apis.md)
