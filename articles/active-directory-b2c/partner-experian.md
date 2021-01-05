---
title: Tutoriel pour configurer Azure Active Directory B2C avec Experian
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C à Experian pour la vérification et la confirmation de l’identification en fonction des attributs de l’utilisateur afin d’éviter les fraudes.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/22/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 2058e5362a65b6cd5f3e5b7cb9c20ce32d020d30
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928696"
---
# <a name="tutorial-for-configuring-experian-with-azure-active-directory-b2c"></a>Tutoriel de configuration d’Experian avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous donnons des conseils pour l’intégration d’Azure AD B2C à [Experian](https://www.experian.com/decision-analytics/account-opening-fraud/microsoft-integration). Experian fournit tout un éventail de solutions, que vous trouverez [ici](https://www.experian.com/).

Dans cet exemple, l’identité numérique intégrée d’Experian et la plateforme de lutte contre les risques de fraude **CrossCore** sont utilisées. CrossCore est un service de vérification d’ID qui est utilisé pour vérifier l’identification des utilisateurs. Il effectue une analyse des risques sur la base de plusieurs informations fournies par l’utilisateur lors de l’inscription. CrossCore est utilisé pour détermine si l’utilisateur doit être autorisé ou non à poursuivre le processus de connexion. Les attributs suivants peuvent être utilisés dans l’analyse des risques CrossCore :

- E-mail
- Adresse IP
- Prénom
- Middle Name
- Surname
- Adresse postale
- City
- État/province
- Code postal
- Pays/région
- Numéro de téléphone

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

## <a name="scenario-description"></a>Description du scénario

L’intégration Experian comprend les composants suivants :

- Azure AD B2C : il s’agit du serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur. Il est également connu sous le nom de fournisseur d’identité.

- Experian : le service Experian accepte les entrées fournies par l’utilisateur et vérifie l’identité de ce dernier.

- API REST personnalisée : cette API implémente l’intégration entre Azure AD B2C et le service Experian.

Le diagramme d’architecture suivant illustre l’implémentation.

![Capture d’écran du schéma d’architecture d’Experian](media/partner-experian/experian-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur accède à une page de connexion. L’utilisateur sélectionne l’option d’inscription pour créer un compte et entre des informations sur la page. Azure AD B2C collecte les attributs de l’utilisateur.
| 2. | Azure AD B2C appelle l’API de couche intermédiaire et transmet les attributs de l’utilisateur.
| 3. | L’API de couche intermédiaire collecte les attributs de l’utilisateur et les convertit dans un format consommable par l’API Experian avant de les envoyer à Experian.
| 4. | Experian consomme les informations et les traite pour vérifier l’identification de l’utilisateur en fonction de l’analyse des risques. Ensuite, il retourne le résultat à l’API de couche intermédiaire.
| 5. | L’API de couche intermédiaire traite les informations et renvoie les informations pertinentes dans le format JSON approprié à Azure AD B2C.
| 6. | Azure AD B2C reçoit les informations envoyées par l’API de couche intermédiaire. S’il affiche une réponse Échec, un message d’erreur est affiché à l’utilisateur. S’il affiche une réponse Réussite, l’utilisateur est authentifié et écrit dans le répertoire.

## <a name="onboard-with-experian"></a>Intégration d’Experian

1. Pour créer un compte Experian, contactez [Experian](https://www.experian.com/decision-analytics/account-opening-fraud/microsoft-integration).

2. Une fois le compte créé, vous recevrez les informations dont vous avez besoin pour la configuration de l’API. Les sections suivantes décrivent le processus.

## <a name="configure-azure-ad-b2c-with-experian"></a>Configurer Azure AD B2C avec Experian

### <a name="part-1---deploy-the-api"></a>Partie 1 : Déploiement de l’API

Déployez le [code d’API](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Experian/CrossCoreIntegrationApi/CrossCoreIntegrationApi.sln) fourni sur un service Azure. Le code peut être publié à partir de Visual Studio, en suivant ces [instructions](/visualstudio/deployment/quickstart-deploy-to-azure).

>[!NOTE]
>Vous aurez besoin de l’URL du service déployé pour configurer Azure AD avec les paramètres requis.

### <a name="part-2---deploy-the-client-certificate"></a>Partie 2 : Déploiement du certificat client

L’appel de l’API Experian est protégé par un certificat client. Ce certificat client sera fourni par Experian. Conformément aux instructions mentionnées dans ce [document](../app-service/environment/certificates.md#private-client-certificate), le certificat doit être chargé vers le service Azure App. L’exemple de stratégie utilise ces étapes clés dans le processus :

- Téléchargement du certificat

- Définissez la clé `WEBSITE_LOAD_ROOT_CERTIFICATES` avec l’empreinte du certificat.

### <a name="part-3---configure-the-api"></a>Partie 3 : Configuration de l’API

Les paramètres d’application peuvent être [configurés dans le service d’application dans Azure](../app-service/configure-common.md#configure-app-settings). Cette méthode permet de les configurer de façon sécurisée sans les archiver dans un référentiel. Vous devrez fournir les paramètres suivants à l’API REST :

| Paramètres de l’application | Source | Notes |
| :-------- | :------------| :-----------|
|CrossCoreConfig:TenantId | Configuration du compte Experian |     |
|CrossCoreConfig:OrgCode | Configuration du compte Experian |     |
|CrossCore:ApiEndpoint |Configuration du compte Experian|  |
|CrossCore:ClientReference | Configuration du compte Experian | |
| CrossCore:ModelCode |Configuration du compte Experian|
| CrossCore:OrgCode | Configuration du compte Experian |
| CrossCore:SignatureKey  | Configuration du compte Experian |
| CrossCore:TenantId  | Configuration du compte Experian |
| CrossCore:CertificateThumbprint | Certificat Experian |
| BasicAuth:ApiUsername | Définition d’un nom d’utilisateur pour l’API | Utilisé dans la configuration ExtId |
| BasicAuth:ApiPassword | Définition d’un mot de passe pour l’API | Utilisé dans la configuration ExtId

### <a name="part-4---create-api-policy-keys"></a>Partie 4 : Création de clés de stratégie d’API

Consultez ce [document](./secure-rest-api.md#add-rest-api-username-and-password-policy-keys) et créez deux clés de stratégie : l’une pour le nom d’utilisateur de l’API et l’autre pour le mot de passe de l’API que vous avez défini plus haut pour l’authentification HTTP de base.

>[!NOTE]
>Vous aurez besoin des clés pour la configuration des stratégies ultérieurement.

### <a name="part-5---replace-the-configuration-values"></a>Partie 5 : Remplacement des valeurs de configuration

Dans les [stratégies personnalisées](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Experian/policy) fournies, recherchez les espaces réservés suivants et remplacez-les par les valeurs correspondantes de votre instance.

|                      Espace réservé                       |                                   Remplacer par la valeur                                 |                   Exemple                    |
| ------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------- |
| {your_tenant_name}                                     | Nom abrégé de votre locataire                                                           | « yourtenant » de yourtenant.onmicrosoft.com |
| {your_trustframeworkbase_policy}                       | Nom Azure AD B2C de votre stratégie TrustFrameworkBase                  | B2C_1A_experian_TrustFrameworkBase           |
| {your_tenant_IdentityExperienceFramework_appid}        | ID de l’application IdentityExperienceFramework configuré dans votre locataire Azure AD B2C      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_ ProxyIdentityExperienceFramework _appid} | ID de l’application ProxyIdentityExperienceFramework configuré dans votre locataire Azure AD B2C | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_appid}                         | ID d’application de l’application de stockage de votre locataire                                      | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_tenant_extensions_app_objectid}                  | ID d’objet de l’application de stockage de votre locataire                                   | 01234567-89ab-cdef-0123-456789abcdef         |
| {your_api_username_key_name}                           | Nom de la clé de nom d’utilisateur que vous avez créée [ici](#part-4---create-api-policy-keys)             | B2C\_1A\_RestApiUsername                     |
| {your_api_password_key_name}                           | Nom de la clé de mot de passe que vous avez créée [ici](#part-4---create-api-policy-keys)             | B2C\_1A\_RestApiPassword                     |
| {your_app_service_URL}                                 | URL du service d’application que vous avez configurée                                             | `https://yourapp.azurewebsites.net`          |

### <a name="part-6---configure-the-azure-ad-b2c-policy"></a>Partie 6 : Configuration de la stratégie Azure AD B2C

Pour obtenir des instructions sur la configuration de votre client Azure AD B2C et de stratégies, reportez-vous à ce [document](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack).

>[!NOTE]
>Cet exemple de stratégie est basé sur le [pack de démarrage de comptes locaux](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts).

>[!NOTE]
> Nous recommandons aux clients d’ajouter une notification de consentement sur la page de collection d’attributs. Informez les utilisateurs que des informations seront envoyées à des services tiers à des fins de vérification d’identité.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C, puis sélectionnez **Flux d’utilisateurs** sous Stratégies.

2. Sélectionnez le **Flux d’utilisateurs** que vous avez créé.

3. Sélectionnez **Exécuter le flux d’utilisateurs**, puis sélectionnez les paramètres :

   a. **Application** : sélectionnez l’application inscrite (JWT dans l’exemple).

   b. **URL de réponse** : sélectionnez l’**URL de redirection**.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte

5. Déconnectez-vous.

6. Suivez le processus d’inscription  

7. Le puzzle CrossCore s’affiche quand vous entrez **Continuer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
