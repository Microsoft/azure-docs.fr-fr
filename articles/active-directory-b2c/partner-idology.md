---
title: Intégration d’IDology à Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer un exemple d’application de paiement en ligne dans Azure AD B2C avec IDology. IDology est un fournisseur de vérification d’identité et de vérification qui propose plusieurs solutions.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2b6001932b6f53a60eda76b6136611a10011391a
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928679"
---
# <a name="tutorial-for-configuring-idology-with-azure-active-directory-b2c"></a>Tutoriel de configuration d’IDology avec Azure Active Directory B2C 

Dans cet exemple de tutoriel, nous donnons des conseils pour l’intégration [d’IDology](https://www.idology.com/solutions/) à Azure AD B2C. IDology est un fournisseur de vérification d’identité et de vérification qui propose plusieurs solutions. Dans cet exemple, nous allons aborder la solution ExpectID par IDology.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un [locataire Azure AD B2C](tutorial-create-tenant.md) lié à votre abonnement Azure.

## <a name="scenario-description"></a>Description du scénario

L’intégration IDology comprend les composants suivants :

- Azure AD B2C : serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur, également connu sous le nom de fournisseur d’identité.
- IDology : le service IDology prend en entrée les données fournies par l’utilisateur et vérifie son identité.
- API REST personnalisée : cette API implémente l’intégration entre Azure AD et le service IDology.

Le diagramme d’architecture suivant illustre l’implémentation.

![Diagramme de l’architecture d’IDology](media/partner-idology/idology-architecture-diagram.png)

| Étape | Description |
|------|------|
|1     | Un utilisateur arrive sur la page de connexion. |
|2     | L’utilisateur sélectionne l’option d’inscription pour créer un compte et entrer des informations sur la page. Azure AD B2C collecte les attributs de l’utilisateur. |
|3     | Azure AD B2C appelle l’API de couche intermédiaire et transmet les attributs de l’utilisateur. |
|4     | L’API de couche intermédiaire collecte les attributs de l’utilisateur et les transforme dans un format consommable par l’API IDology. Ensuite, elle envoie les informations à IDology. |
|5     | IDology consomme les informations et les traite, puis envoie le résultat à l’API de couche intermédiaire. |
|6     | L’API de couche intermédiaire traite les informations et renvoie les informations pertinentes à Azure AD B2C. |
|7     | Azure AD B2C reçoit les informations envoyées par l’API de couche intermédiaire. S’il affiche une réponse **Échec**, un message d’erreur est affiché à l’utilisateur. S’il affiche une réponse **Réussite**, l’utilisateur est authentifié et écrit dans le répertoire. |
|      |      |

> [!NOTE]
> Azure AD B2C peut également demander au client d’effectuer une authentification avec passage à une édition supérieure, mais ce cas de figure n’est pas abordé dans ce tutoriel.

## <a name="onboard-with-idology"></a>Intégration avec IDology

1. IDology fournit tout un éventail de solutions, que vous trouverez [ici](https://www.idology.com/solutions/). Dans cet exemple, nous utilisons ExpectID.

2. Pour créer un compte IDology, contactez [IDology](https://www.idology.com/request-a-demo/microsoft-integration-signup/).

3. Une fois le compte créé, vous recevrez les informations dont vous avez besoin pour la configuration de l’API. Les sections suivantes décrivent le processus.

## <a name="integrate-with-azure-ad-b2c"></a>Intégrer avec Azure AD B2C

### <a name="part-1---deploy-the-api"></a>Partie 1 : Déploiement de l’API

Déployez le [code d’API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/IDology/Api) fourni sur un service Azure. Le code peut être publié à partir de Visual Studio, en suivant ces [instructions](/visualstudio/deployment/quickstart-deploy-to-azure).

Vous aurez besoin de l’URL du service déployé pour configurer Azure AD avec les paramètres requis.

### <a name="part-2---configure-the-api"></a>Partie 2 : Configuration de l’API 

Les paramètres d’application peuvent être [configurés dans App Service dans Azure](../app-service/configure-common.md#configure-app-settings). Cette méthode permet de les configurer de façon sécurisée sans les archiver dans un référentiel. Vous devrez fournir les paramètres suivants à l’API REST :

| Paramètres de l’application | Source | Notes |
| :-------- | :------------| :-----------|
|IdologySettings:ApiUsername | Configuration du compte IDology |     |
|IdologySettings:ApiPassword | Configuration du compte IDology |     |
|WebApiSettings:ApiUsername |Définition d’un nom d’utilisateur pour l’API| Utilisé dans la configuration ExtId |
|WebApiSettings:ApiPassword | Définition d’un mot de passe pour l’API | Utilisé dans la configuration ExtId

### <a name="part-3---create-api-policy-keys"></a>Partie 3 : Création de clés de stratégie d’API

Suivez ce [document](secure-rest-api.md#add-rest-api-username-and-password-policy-keys) pour créer deux clés de stratégie : l’une pour le nom d’utilisateur de l’API et l’autre pour le mot de passe de l’API que vous avez défini ci-dessus.

L’exemple de stratégie utilise les noms de clés suivants :

* B2C_1A_RestApiUsername
* B2C_1A_RestApiPassword

### <a name="part-4---configure-the-azure-ad-b2c-policy"></a>Partie 4 : Configuration de la stratégie Azure AD B2C

1. Suivez ce [document](custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack) pour télécharger le [pack de démarrage LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts) et configurer la stratégie du locataire Azure AD B2C. Suivez les instructions jusqu’à la fin de la section **Tester la stratégie personnalisée**.

2. Téléchargez les deux exemples de stratégies [ici](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/IDology/policy).

3. Mettez à jour les deux exemples de stratégies :

   1. Ouvrez les deux stratégies :

      1. Dans la section `Idology-ExpectId-API`, mettez à jour l’élément de métadonnées `ServiceUrl` avec l’emplacement de l’API déployée ci-dessus.

      1. Remplacez `yourtenant` par le nom de votre locataire Azure AD B2C.
      Par exemple, si le nom de votre locataire Azure AD B2C est  `contosotenant`, remplacez toutes les instances de  `yourtenant.onmicrosoft.com` par `contosotenant.onmicrosoft.com`.

   1. Ouvrez le fichier TrustFrameworkExtensions.xml :

      1. Recherchez l’élément  `<TechnicalProfile Id="login-NonInteractive">`. Remplacez les deux instances de  `IdentityExperienceFrameworkAppId` par l’ID de l’application IdentityExperienceFramework que vous avez créée.

      1. Remplacez les deux instances de  `ProxyIdentityExperienceFrameworkAppId` par l’ID de l’application ProxyIdentityExperienceFramework que vous avez créée.

4. Remplacez SignInorSignUp.xml et TrustFrameworkExtensions.xml, chargés dans Azure AD B2C à l’étape 1, par les deux exemples de stratégies mis à jour.

> [!NOTE]
> Nous recommandons aux clients d’ajouter une notification de consentement sur la page de collection d’attributs. Informez les utilisateurs que des informations seront envoyées à des services tiers à des fins de vérification d’identité.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C, puis sélectionnez **Flux d’utilisateurs** sous **Stratégies**.

2. Sélectionnez le **Flux d’utilisateurs** que vous avez créé.

3. Sélectionnez **Exécuter le flux d’utilisateurs**, puis sélectionnez les paramètres :

   1. **Application** : sélectionnez l’application enregistrée (JWT dans l’exemple).

   1. **URL de réponse** : sélectionnez **l’URL de redirection**.

   1. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte.

5. Déconnectez-vous.

6. Suivez le processus de connexion.

7. Le puzzle IDology s’affiche une fois que vous avez entré **Continuer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](custom-policy-get-started.md?tabs=applications)
