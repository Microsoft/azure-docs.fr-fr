---
title: Tutoriel pour configurer Azure Active Directory B2C avec LexisNexis
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C à LexisNexis, service de profilage et de validation d’identité permettant de vérifier l’identification des utilisateurs et fournissant des évaluations de risques complètes basées sur l’appareil de l’utilisateur.
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/22/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 5f408699cae9580188a3780fc8f8654eaa97c26b
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108431"
---
# <a name="tutorial-for-configuring-lexisnexis-with-azure-active-directory-b2c"></a>Tutoriel pour configurer LexisNexis avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous donnons des conseils sur l’intégration d’Azure AD B2C à [LexisNexis](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd). LexisNexis fournit tout un éventail de solutions, que vous trouverez [ici](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd). Dans cet exemple de tutoriel, nous allons aborder la solution **ThreatMetrix** de LexisNexis. ThreatMetrix est un service de profilage et de validation d’identité. Il permet de vérifier l’identification des utilisateurs et fournit des évaluations de risques complètes basées sur l’appareil de l’utilisateur.

Cette intégration permet d’effectuer un profilage basé sur quelques informations sur l’utilisateur, fournies par ce dernier durant le processus d’inscription. ThreatMetrix détermine si l’utilisateur doit être autorisé ou non à poursuivre le processus de connexion. L’analyse des risques de ThreatMetrix prend en compte les attributs suivants :

- E-mail
- Numéro de téléphone
- Informations de profilage collectées à partir de l’ordinateur de l’utilisateur

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md) lié à votre abonnement Azure.

## <a name="scenario-description"></a>Description du scénario

L’intégration ThreatMetrix comprend les composants suivants :

- Azure AD B2C : il s’agit du serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur. Il est également connu sous le nom de fournisseur d’identité.

- ThreatMetrix : le service ThreatMetrix récupère les entrées de l’utilisateur et les associe à des informations de profilage collectées à partir de son ordinateur pour vérifier la sécurité de l’interaction avec l’utilisateur.

- API REST personnalisée : cette API implémente l’intégration entre Azure AD B2C et le service ThreatMetrix.

Le diagramme d’architecture suivant illustre l’implémentation.

![Capture d’écran - schéma d’architecture de LexisNexis](media/partner-lexisnexis/lexisnexis-architecture-diagram.png)

|Étape | Description |
|:--------------|:-------------|
|1. | L’utilisateur accède à une page de connexion. L’utilisateur sélectionne l’option d’inscription pour créer un compte et entrer des informations sur la page. Azure AD B2C collecte les attributs de l’utilisateur.
| 2. | Azure AD B2C appelle l’API de couche intermédiaire et transmet les attributs de l’utilisateur.
| 3. | L’API de couche intermédiaire collecte les attributs de l’utilisateur et les convertit dans un format consommable par l’API LexisNexis avant de les envoyer à LexisNexis.  
| 4. | LexisNexis consomme les informations et les traite pour vérifier l’identification de l’utilisateur en fonction de l’analyse des risques. Ensuite, il retourne le résultat à l’API de couche intermédiaire.
| 5. | L’API de couche intermédiaire traite les informations et renvoie les informations pertinentes à Azure AD B2C.
| 6. | Azure AD B2C reçoit les informations envoyées par l’API de couche intermédiaire. S’il affiche une réponse Échec, un message d’erreur est affiché à l’utilisateur. S’il affiche une réponse Réussite, l’utilisateur est authentifié et l’accès lui est accordé.

## <a name="onboard-with-lexisnexis"></a>Intégration avec LexisNexis

1. Pour créer un compte LexisNexis, contactez [LexisNexis](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd).

2. Créez une stratégie LexisNexis répondant à vos besoins. Utilisez la documentation disponible [ici](https://risk.lexisnexis.com/products/threatmetrix/?utm_source=bingads&utm_medium=ppc&utm_campaign=SEM%7CLNRS%7CUS%7CEN%7CTMX%7CBR%7CBing&utm_term=threat%20metrix&utm_network=o&utm_device=c&msclkid=1e85e32ec18c1ae9bbc1bc2998e026bd).

>[!NOTE]
> Le nom de la stratégie sera utilisé ultérieurement.

Une fois le compte créé, vous recevrez les informations dont vous avez besoin pour la configuration de l’API. Les sections suivantes décrivent le processus.

## <a name="configure-azure-ad-b2c-with-lexisnexis"></a>Configurer Azure AD B2C avec LexisNexis

### <a name="part-1---deploy-the-api"></a>Partie 1 : Déploiement de l’API

Déployez le [code d’API](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/ThreatMetrix/Api) fourni sur un service Azure. Le code peut être publié à partir de Visual Studio, en suivant ces [instructions](/visualstudio/deployment/quickstart-deploy-to-azure).

>[!NOTE]
>Vous aurez besoin de l’URL du service déployé pour configurer Azure AD avec les paramètres requis.

### <a name="part-2---configure-the-api"></a>Partie 2 : Configuration de l’API

Les paramètres d’application peuvent être [configurés dans le service d’application dans Azure](../app-service/configure-common.md#configure-app-settings).  Cette méthode permet de les configurer de façon sécurisée sans les archiver dans un référentiel. Vous devrez fournir les paramètres suivants à l’API REST :

| Paramètres de l’application | Source | Notes |
| :-------- | :------------| :-----------|
|ThreatMetrix: Url | Configuration du compte ThreatMetrix |     |
|ThreatMetrix:OrgId | Configuration du compte ThreatMetrix |     |
|ThreatMetrix:ApiKey |Configuration du compte ThreatMetrix|  |
|ThreatMetrix: Policy | Nom de la stratégie créée dans ThreatMetrix | |
| BasicAuth:ApiUsername |Définition d’un nom d’utilisateur pour l’API| Le nom d’utilisateur sera utilisé pour la configuration d’Azure AD B2C.
| BasicAuth:ApiPassword | Définition d’un mot de passe pour l’API | Le mot de passe sera utilisé pour la configuration d’Azure AD B2C.

### <a name="part-3---deploy-the-ui"></a>Partie 3 : Déploiement de l’interface utilisateur

Cette solution utilise des modèles d’IU personnalisés chargés par Azure AD B2C. Ces modèles d’IU effectuent le profilage qui est envoyé directement au service ThreatMetrix.

Reportez-vous à ces [instructions](./customize-ui-with-html.md#custom-page-content-walkthrough) pour déployer les [fichiers d’interface utilisateur](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/ThreatMetrix/ui-template) inclus dans un compte de stockage Blob. Les instructions traitent notamment de la configuration d’un compte de stockage Blob, de la configuration de CORS et de l’activation de l’accès public.

L’interface utilisateur est basée sur le [modèle Ocean Blue (Bleu océan)](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/ThreatMetrix/ui-template/ocean_blue). Tous les liens de l’interface utilisateur doivent être mis à jour pour faire référence à l’emplacement de déploiement. Dans le dossier de l’interface utilisateur, recherchez l’URL https://yourblobstorage/blobcontainer et remplacez-la par l’emplacement de déploiement.

### <a name="part-4---create-api-policy-keys"></a>Partie 4 : Création de clés de stratégie d’API

Consultez ce [document](./secure-rest-api.md#add-rest-api-username-and-password-policy-keys) et créez deux clés de stratégie : l’une pour le nom d’utilisateur de l’API et l’autre pour le mot de passe de l’API que vous avez défini plus haut.

L’exemple de stratégie utilise les noms de clés suivants :

- B2C_1A_RestApiUsername

- B2C_1A_RestApiPassword

### <a name="part-5---update-the-api-url"></a>Partie 5 : Mise à jour de l’URL de l’API

Dans la [stratégie TrustFrameworkExtensions](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/ThreatMetrix/policy/TrustFrameworkExtensions.xml) fournie, recherchez le profil technique nommé `Rest-LexisNexus-SessionQuery` et mettez à jour l’élément de métadonnées `ServiceUrl` avec l’emplacement de l’API déployée précédemment.

### <a name="part-6---update-ui-url"></a>Partie 6 : Mise à jour de l’URL de l’interface utilisateur

Dans la [stratégie TrustFrameworkExtensions](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/ThreatMetrix/policy/TrustFrameworkExtensions.xml) fournie, recherchez l’URL https://yourblobstorage/blobcontainer/ et remplacez-la par l’emplacement où les fichiers de l’interface utilisateur sont déployés.

>[!NOTE]
> Nous recommandons aux clients d’ajouter une notification de consentement sur la page de collection d’attributs. Informez les utilisateurs que des informations seront envoyées à des services tiers à des fins de vérification d’identité.

### <a name="part-7---configure-the-azure-ad-b2c-policy"></a>Partie 7 : Configuration de la stratégie Azure AD B2C

Reportez-vous à ce [document](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack) pour télécharger le [pack de démarrage pour comptes locaux](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts) et configurer la [stratégie](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/ThreatMetrix/policy) du locataire Azure AD B2C.

>[!NOTE]
>Mettez à jour les stratégies fournies en fonction de votre locataire spécifique.

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

7. Le puzzle ThreatMetrix s’affiche quand vous entrez **continuer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
