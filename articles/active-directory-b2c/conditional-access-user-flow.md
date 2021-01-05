---
title: Ajouter l’accès conditionnel à un flux d’utilisateur dans Azure AD B2C
description: Découvrez comment ajouter l’accès conditionnel à vos flux d’utilisateurs Azure AD B2C. Configurez des paramètres d’authentification multifacteur (MFA) et des stratégies d’accès conditionnel dans vos flux d’utilisateurs afin d’appliquer des stratégies et de limiter les connexions risquées.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: overview
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
author: msmimart
manager: celested
ms.collection: M365-identity-device-management
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: d6d5ab13c8997dffee42a053ba498376ccbcb6d8
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585256"
---
# <a name="add-conditional-access-to-user-flows-in-azure-active-directory-b2c"></a>Ajouter l’accès conditionnel à des flux d’utilisateurs dans Azure Active Directory B2C

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

L’accès conditionnel peut être ajouté à vos flux d’utilisateurs Azure Active Directory B2C pour gérer les connexions risquées à vos applications. L’intégration d’Identity Protection et de l’accès conditionnel dans Azure AD B2C vous permet de définir des stratégies qui identifient le comportement de connexion risqué et d’appliquer des stratégies qui exigent une action supplémentaire de la part de l’utilisateur ou de l’administrateur. Voici les composants qui activent l’accès conditionnel dans les flux d’utilisateurs Azure AD B2C :

- **Flux d’utilisateur**. Créez un flux d’utilisateur qui guide l’utilisateur tout au long du processus de connexion et d’inscription. Dans les paramètres de flux d’utilisateur, indiquez si les stratégies d’accès conditionnel doivent être activées lorsqu’un utilisateur suit le flux d’utilisateur.
- **Application qui dirige les utilisateurs vers le flux d’utilisateur**. Configurez votre application de façon à diriger les utilisateurs vers le flux d’utilisateur d’inscription et de connexion approprié en spécifiant le point de terminaison de flux d’utilisateur dans l’application.
- **Stratégie d’accès conditionnel**. [Créez une stratégie d’accès conditionnel](conditional-access-identity-protection-setup.md) et spécifiez les applications auxquelles vous souhaitez appliquer la stratégie. Lorsque l’utilisateur suit le flux d’utilisateur de connexion ou d’inscription de votre application, la stratégie d’accès conditionnel utilise des signaux Identity Protection pour identifier les connexions risquées, et présente l’action de correction appropriée si nécessaire.

L’accès conditionnel est pris en charge dans les versions les plus récentes des flux d’utilisateurs. Vous pouvez ajouter des stratégies d’accès conditionnel aux nouveaux flux d’utilisateurs au fur et à mesure de leur création, ou vous pouvez les ajouter à des flux d’utilisateurs existants tant que la version prend en charge l’accès conditionnel. Lorsque vous ajoutez l’accès conditionnel à un flux d’utilisateur existant, vous devez passer en revue deux paramètres :

- **Authentification multifacteur (MFA)**  : les utilisateurs peuvent désormais utiliser un code à usage unique par SMS ou voix, ou un mot de passe à usage unique par e-mail pour l’authentification multifacteur. Les paramètres MFA sont indépendants des paramètres d’accès conditionnel. Vous pouvez définir l’authentification MFA sur **Always On** afin qu’elle soit toujours requise quelle que soit la configuration de l’accès conditionnel, ou la définir sur **Conditionnelle** afin qu’elle soit requise uniquement lorsqu’une stratégie d’accès conditionnel active l’exige.

- **Accès conditionnel** : ce paramètre doit toujours être **Activé**. En général, vous ne **désactiverez** ce paramètre que lors du dépannage ou de la migration, ou pour les implémentations héritées.

Apprenez-en davantage sur [Identity Protection et l’accès conditionnel](conditional-access-identity-protection-overview.md) dans Azure AD B2C, ou découvrez [comment les configurer](conditional-access-identity-protection-setup.md).

## <a name="prerequisites"></a>Prérequis

- Azure AD B2C Premium 2 est nécessaire pour créer des stratégies de connexion risquée. Les locataires Premium P1 peuvent créer des stratégies basées sur l’emplacement, l’application ou le groupe.
- À des fins de test, vous pouvez [inscrire l’application web de test](tutorial-register-applications.md) `https://jwt.ms`, une application web Microsoft qui affiche le contenu décodé d’un jeton (le contenu du jeton ne quitte jamais votre navigateur). 
- Pour simuler une connexion risquée, téléchargez le navigateur TOR et essayez de vous connecter au point de terminaison du flux d’utilisateur.
- À l’aide des paramètres suivants, [créez une stratégie d’accès conditionnel](conditional-access-identity-protection-setup.md) :
   
  - Pour **Utilisateurs et groupes**, sélectionnez l’utilisateur de test (ne sélectionnez pas **Tous les utilisateurs**, car vous risqueriez de ne pas pouvoir vous connecter).
  - Pour **Applications ou actions cloud**, choisissez **Sélectionner les applications**, puis choisissez votre application par partie de confiance.
  - Pour Conditions, sélectionnez **Risque de connexion** et les niveaux de risque **Élevé**, **Moyen** et **Faible**.
  - Pour **Accorder**, choisissez **Bloquer l’accès**.

      ![Détections de risques](media/conditional-access-identity-protection-setup/test-conditional-access-policy.png)

::: zone pivot="b2c-user-flow"

## <a name="add-conditional-access-to-a-new-user-flow"></a>Ajouter l’accès conditionnel à un nouveau flux d’utilisateur

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Sous **Stratégies**, sélectionnez **Flux d’utilisateurs**, puis sélectionnez **Nouveau flux d’utilisateur**.
1. Dans la page **Créer un flux d’utilisateur**, sélectionnez le type de flux d’utilisateur.
1. Sous **Sélectionner une version**, sélectionnez **Recommandé**, puis **Créer**. ([Apprenez-en davantage](user-flow-versions.md) sur les version de flux utilisateur.)

    ![Page Créer un flux d'utilisateur du Portail Azure avec propriétés en surbrillance](./media/tutorial-create-user-flows/select-version.png)

1. Entrez un **Nom** pour le flux d’utilisateur. Par exemple, *signupsignin1*.
1. Dans la section **Fournisseurs d’identité**, sélectionnez les fournisseurs d’identité que vous souhaitez autoriser pour ce flux d’utilisateur.
2. Dans la section **Authentification multifacteur**, sélectionnez la **Méthode MFA** souhaitée puis, sous **Application MFA**, sélectionnez **Conditionnelle (recommandé)** .
 
   ![Configurer l’authentification multifacteur](media/conditional-access-user-flow/configure-mfa.png)

1. Dans la section **Accès conditionnel**, cochez la case **Appliquer les stratégies d’accès conditionnel**.

   ![Configurer les paramètres d’accès conditionnel](media/conditional-access-user-flow/configure-conditional-access.png)

1. Dans la section **Attributs utilisateur et revendications**, choisissez les revendications et les attributs à collecter et envoyer par l’utilisateur pendant l’inscription. Par exemple, sélectionnez **Afficher plus**, puis choisissez des attributs et des revendications pour **Pays/région** et **Nom d’affichage**. Sélectionnez **OK**.

    ![Page de sélection des attributs et revendications avec trois revendications sélectionnées](./media/conditional-access-user-flow/configure-user-attributes-claims.png)

1. Cliquez sur **Créer** pour ajouter le flux utilisateur. Un préfixe *B2C_1* est automatiquement ajouté au nom.

## <a name="add-conditional-access-to-an-existing-user-flow"></a>Ajouter l’accès conditionnel à un flux d’utilisateur existant

> [!NOTE]
> Le flux d’utilisateur existant doit être une version qui prend en charge l’accès conditionnel. Ces versions de flux d’utilisateurs sont étiquetées **Recommandé**.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.

1. Sous **Stratégies**, sélectionnez **Flux utilisateur**. Sélectionnez ensuite le flux d’utilisateur.

1. Sélectionnez **Propriétés** et vérifiez que le flux d’utilisateur prend en charge l’accès conditionnel en sélectionnant **Propriétés** et en recherchant le paramètre libellé **Accès conditionnel**.
 
   ![Configurer l’authentification multifacteur et l’accès conditionnel dans les propriétés](media/conditional-access-user-flow/add-conditional-access.png)

1. Dans la section **Authentification multifacteur**, sélectionnez la **Méthode MFA** souhaitée puis, sous **Application MFA**, sélectionnez **Conditionnelle (recommandé)** .
 
1. Dans la section **Accès conditionnel**, cochez la case **Appliquer les stratégies d’accès conditionnel**.

1. Sélectionnez **Enregistrer**.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

Pour tester l’accès conditionnel dans votre flux d’utilisateur, [créez une stratégie d’accès conditionnel](conditional-access-identity-protection-setup.md) et activez l’accès conditionnel dans votre flux d’utilisateur comme décrit ci-dessus. 


### <a name="run-the-user-flow"></a>Exécuter le flux d’utilisateur

1. Sélectionnez le flux utilisateur que vous avez créé pour ouvrir sa page de présentation, puis sélectionnez **Exécuter le flux d’utilisateur**. Sous **Application**, sélectionnez *webapp1*. L’**URL de réponse** doit être `https://jwt.ms`.

   ![Page Exécuter le flux d'utilisateur du portail avec bouton Exécuter le flux d'utilisateur en surbrillance](./media/tutorial-create-user-flows/signup-signin-run-now.PNG)

1. Copiez l’URL sous **Exécuter le point de terminaison du flux d’utilisateur**.

1. Pour simuler une connexion risquée, ouvrez le [navigateur Tor](https://www.torproject.org/download/) et utilisez l’URL que vous avez copiée à l’étape précédente pour vous connecter à l’application inscrite.

1. Entrez les informations demandées dans la page de connexion, puis essayez de vous connecter. Le jeton est envoyé à `https://jwt.ms` et vous devez le voir. Dans le jeton décodé jwt.ms, vous devez voir que la connexion a été bloquée :

   ![Tester une connexion bloquée](media/conditional-access-identity-protection-setup/test-blocked-sign-in.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="add-conditional-access-to-your-policy"></a>Ajouter l’accès conditionnel à votre stratégie

Vous trouverez un exemple de stratégie d’accès conditionnel sur [GitHub](https://github.com/azure-ad-b2c/samples/tree/master/policies/conditional-access).

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

[Personnaliser l’interface utilisateur dans un flux d’utilisateur Azure AD B2C](customize-ui-with-html.md)
