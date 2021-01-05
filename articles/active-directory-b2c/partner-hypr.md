---
title: Tutoriel pour configurer Azure Active Directory B2C avec HYPR
titleSuffix: Azure AD B2C
description: Tutoriel pour configurer Azure Active Directory B2C avec HYPR pour une véritable authentification client forte et sans mot de passe
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/27/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: b80b1a4b3f9bcde6cf01b0e0e59425c6783bd5d9
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953761"
---
# <a name="tutorial-for-configuring-hypr-with-azure-active-directory-b2c"></a>Tutoriel de configuration de HYPR avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur la façon de configurer Azure AD B2C avec [HYPR](https://get.hypr.com). Grâce à Azure AD B2C comme fournisseur d’identité, vous pouvez intégrer HYPR à toutes vos applications clientes pour fournir une véritable authentification sans mot de passe à vos utilisateurs. HYPR remplace les mots de passe par des chiffrements à clé publique, éliminant les fraudes, le hameçonnage et la réutilisation des informations d’identification.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md). Le locataire est lié à votre abonnement Azure.

- Un client cloud HYPR ; vous pouvez utiliser un [compte d’essai gratuit](https://get.hypr.com/free-trial).

- L’appareil mobile d’un utilisateur inscrit à l’aide des API REST HYPR ou de Gestionnaire d’appareils HYPR dans votre client HYPR. Par exemple, vous pouvez utiliser le [Kit de développement logiciel (SDK) Java HYPR](https://docs.hypr.com/integratinghypr/docs/hypr-java-web-sdk) pour accomplir cette tâche.

## <a name="scenario-description"></a>Description du scénario

L’intégration de HYRP inclut les composants suivants :

- Azure AD B2C : il s’agit du serveur d’autorisation chargé de vérifier les informations d’identification de l’utilisateur. Il est également connu sous le nom de fournisseur d’identité.

- Applications web et mobiles : vos applications web ou mobiles que vous choisissez de protéger avec HYPR et Azure AD B2C. HYPR fournit un Kit de développement logiciel (SDK) mobile robuste ainsi qu’une application mobile que vous pouvez utiliser sur les plateformes iOS et Android pour effectuer une véritable authentification sans mot de passe.

- Application mobile HYPR : l’application mobile HYPR peut être utilisée pour exécuter cet exemple si vous préférez ne pas utiliser les Kits de développement logiciel (SDK) mobiles dans vos propres applications mobiles.

- API REST HYPR : vous pouvez utiliser les API HYPR pour effectuer l’inscription et l’authentification des appareils utilisateur. Ces API sont accessibles [ici](https://apidocs.hypr.com).

Le diagramme d’architecture suivant illustre l’implémentation.

![Capture d’écran du diagramme d’architecture de HYPR](media/partner-hypr/hypr-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur accède à une page de connexion. Les utilisateurs sélectionnent Connexion/Inscription et entrent le nom d’utilisateur dans la page.
| 2. | L’application envoie les attributs utilisateur à Azure AD B2C pour identifier la vérification.
| 3. | Azure AD B2C collecte les attributs utilisateur et envoie les attributs à HYPR pour authentifier l’utilisateur par le biais de l’application mobile HYPR.
| 4. | HYPR envoie une notification Push à l’appareil mobile de l’utilisateur inscrit pour une authentification certifiée FIDO (Fast Identity Online). Il peut s’agir d’un code décentralisé, biométrique ou d’empreinte digitale de l’utilisateur.  
| 5. | Une fois que l’utilisateur a confirmé la notification Push, il est autorisé ou non à accéder à l’application cliente en fonction des résultats de la vérification.

## <a name="configure-the-azure-ad-b2c-policy"></a>Configurer la stratégie Azure AD B2C

1. Accédez à la [stratégie HYPR d’Azure AD B2C](https://github.com/HYPR-Corp-Public/Azure-AD-B2C-HYPR-Sample/tree/master/policy) dans le dossier Stratégie.

2. Suivez ce [document](./custom-policy-get-started.md?tabs=applications#custom-policy-starter-pack) pour télécharger le [Pack de démarrage LocalAccounts](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/LocalAccounts)

3. Configurez la stratégie pour le locataire Azure AD B2C.

>[!NOTE]
>Mettez à jour les stratégies fournies en fonction de votre locataire spécifique.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C locataire, puis, sous Stratégies, sélectionnez **Identity Experience Framework**.

2. Sélectionnez votre **SignUpSignIn** créé précédemment.

3. Sélectionnez **Exécuter le flux d’utilisateurs**, puis sélectionnez les paramètres :

   a. **Application** : sélectionnez l’application inscrite (JWT dans l’exemple).

   b. **URL de réponse** : sélectionnez l’**URL de redirection**.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte

5. HYPR est appelé pendant le flux, après la création de l’attribut utilisateur. Si le flux est incomplet, vérifiez que l’utilisateur n’est pas enregistré dans le répertoire.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)