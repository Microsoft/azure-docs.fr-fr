---
title: Tutoriel pour configurer Azure Active Directory B2C avec WhoIAM
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, découvrez comment intégrer l’authentification Azure AD B2C avec WhoIAM pour la vérification de l’utilisateur.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: d1b2c7513562e951e1098cf327780387ddf6a495
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953540"
---
# <a name="tutorial-for-configuring-whoiam-with-azure-active-directory-b2c"></a>Tutoriel de configuration de WhoIAM avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur la façon de configurer le système de gestion des identités personnalisé (BRIMS) [WhoIAM](https://www.whoiam.ai/brims/) dans votre environnement et de l’intégrer avec Active Directory B2C (Azure AD B2C).

Le système BRIMS est un ensemble d’applications et de services qui est déployé dans votre environnement. Il fournit une vérification de votre base d’utilisateurs par le biais de la voix, de SMS et d’adresses e-mails. Le système BRIMS fonctionne en liaison avec votre solution existante de gestion des identités et des accès et n’est dépendant d’aucune plateforme.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- [Un locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

- Un [compte d’évaluation gratuite](https://www.whoiam.ai/contact-us/) WhoIAM.

## <a name="scenario-description"></a>Description du scénario

L’intégration de WhoIAM inclut les composants suivants :

- Un locataire Azure AD B2C. Il s’agit du serveur d’autorisation qui vérifie les informations d’identification de l’utilisateur en fonction des stratégies personnalisées définies dans celui-ci. Également connu sous le nom de fournisseur d’identité.

- Un portail d’administration pour la gestion des clients et de leurs configurations.

- Un service d’API qui expose diverses fonctionnalités via des points de terminaison.  

- Azure Cosmos DB, qui fait office de back-end pour le portail d’administration du système BRIMS et le service d’API.

Le diagramme d’architecture suivant illustre l’implémentation.

![Schéma de l’architecture de l’intégration Azure AD B2C avec WhoIAM.](media/partner-whoiam/whoiam-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur arrive sur une page pour lancer une demande d’inscription ou de connexion à une application qui utilise Azure AD B2C comme fournisseur d’identité.
| 2. | Dans le cadre de l’authentification, l’utilisateur demande à vérifier la propriété de son adresse de messagerie ou de son téléphone, ou à utiliser sa voix comme facteur de vérification biométrique.  
| 3. | Azure AD B2C appelle le service d’API du système BRIMS et transmet l’adresse e-mail de l’utilisateur, son numéro de téléphone et l’enregistrement vocal.
| 4. | Le système BRIMS utilise des configurations prédéfinies telles que des modèles d’adresses e-mail et de SMS entièrement personnalisables pour interagir avec l’utilisateur dans sa langue respective, en toute cohérence avec le style de l’application.
| 5. | Une fois la vérification de l’identité d’un utilisateur terminée, le système renvoie à Azure AD B2C un jeton indiquant le résultat de la vérification. Azure AD B2C permet alors à l’utilisateur d’accéder à l’application ou fait échouer sa tentative d’authentification.  

## <a name="sign-up-with-whoiam"></a>S’inscrire avec WhoIAM

1. Contactez [WhoIAM](https://www.whoiam.ai/contact-us/) et créez un compte BRIMS.

2. Utilisez les instructions d’inscription mises à votre disposition et configurez les services Azure suivants :

    - [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) : utilisé pour le stockage sécurisé de mots de passe, tels que les mots de passe de services de messagerie.

    - [Azure App Service](https://azure.microsoft.com/services/app-service/) : utilisé pour héberger les services de l’API BRIMS et du portail d’administration.

    - [Azure Active Directory](https://azure.microsoft.com/services/active-directory/) : utilisé pour authentifier les utilisateurs administratifs pour le portail d’administration.

    - [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) : utilisé pour stocker et récupérer des paramètres.

    - [Application Insights](../azure-monitor/app/app-insights-overview.md) (facultatif) : utilisé pour se connecter à l’API et au portail d’administration.

3. Déployez l’API du système BRIMS et le portail d’administration de BRIMS dans votre environnement Azure.

4. Les modèles de stratégie personnalisée Azure AD B2C sont disponibles dans votre documentation d’inscription BRIMS. Suivez les instructions données dans la documentation pour configurer votre application et utiliser la plateforme BRIMS pour vérifier l’identité des utilisateurs.  

Pour plus d’informations sur le système BRIMS de WhoIAM, consultez la [documentation produit](https://www.whoiam.ai/brims/).

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C. Sous **Stratégies**, sélectionnez **Identity Experience Framework**.

2. Sélectionnez votre **SignUpSignIn** créé précédemment.

3. Sélectionnez **Exécuter le flux utilisateur**, puis :

   a. Pour **Application**, sélectionnez l’application inscrite (JWT dans cet exemple).

   b. Pour **URL de réponse**, sélectionnez l’**URL de redirection**.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte.

5. Le service BRIMS sera appelé pendant le flux, après la création de l’attribut d’utilisateur. Si le flux est incomplet, vérifiez que l’utilisateur n’est pas enregistré dans l’annuaire.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)