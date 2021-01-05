---
title: TypingDNA avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C à TypingDNA afin de contribuer à la vérification et la confirmation d’identités basées sur le modèle de saisie de l’utilisateur, qui forcent l’authentification multifacteur et permettent de se conformer aux normes SCA pour la directive 2 sur les services de paiement (B2C).
author: gargi-sinha
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 68617d86fda940c5d3752f2389088a8c729aebec
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108346"
---
# <a name="tutorial-for-configuring-typingdna-with-azure-active-directory-b2c"></a>Didacticiel pour la configuration de TypingDNA avec Azure Active Directory B2C

Avec cette procédure, découvrez comment intégrer un exemple d’application de paiement en ligne dans Azure Active Directory B2C avec l’application TypingDNA. À l’aide de l’application TypingDNA, les clients Azure AD B2C peuvent se conformer aux exigences en matière de transactions [PSD2](https://www.typingdna.com/use-cases/sca-strong-customer-authentication) (Payment Services directive 2) par le biais de la dynamique de séquence de touches et de l’authentification client renforcée. Pour plus d’informations sur TypingDNA, rendez-vous [ici](https://www.typingdna.com/).

 Azure AD B2C utilise les technologies de TypingDNA pour capturer les caractéristiques de saisie des utilisateurs afin qu’elles soient enregistrées et analysées à des fins de familiarisation à chaque authentification. Cela ajoute une couche de protection liée au risque potentiel d’une authentification et évalue les niveaux de risque. Azure AD B2C peut invoquer d’autres mécanismes pour renforcer la confiance de l’utilisateur en appelant Azure AD MFA, en forçant la vérification par courrier électronique ou toute autre logique personnalisée pour votre scénario.

>[!NOTE]
> Cet exemple de stratégie est basé sur le pack de démarrage [SocialAndLocalAccountsWithMfa](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccountsWithMfa).

## <a name="scenario-description"></a>Description du scénario

![Capture d’écran du diagramme d’architecture TypingDNA](./media/partner-typingdna/typingdna-architecture-diagram.png)

### <a name="sign-up"></a>Inscription

1. Les pages Azure AD B2C utilisent la bibliothèque JavaScript de TypingDNA pour enregistrer le modèle de saisie de l’utilisateur. Par exemple, le nom d’utilisateur et le mot de passe sont enregistrés lors de l’enregistrement pour l’inscription initiale, puis à chaque connexion pour vérification.

2. Lorsque l’utilisateur soumet la page, la bibliothèque TypingDNA calcule la caractéristique de saisie de l’utilisateur. Après cela, insérez les informations dans un champ de texte masqué qu’Azure AD B2C a rendu. Ce champ est masqué avec CSS.  

    L’[exemple contient des fichiers HTML](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignUp.cshtml) avec les modifications JavaScript et CSS, et est référencé par les définitions de contenu `api.selfasserted.tdnasignin` et `api.selfasserted.tdnasignup`. Reportez-vous à [hébergement du contenu de la page](./customize-ui-with-html.md#hosting-the-page-content) pour héberger vos fichiers HTML.

3. Azure AD B2C compte maintenant le modèle de saisie dans le jeu de revendications lorsque l’utilisateur envoie ses informations d’identification. Il doit appeler une API (la vôtre) pour transmettre ces données au point de terminaison de l’API REST TypingDNA. Cette API est incluse dans l’[exemple (typingDNA-API-Interface)](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
4. L’API de couche intermédiaire transmet ensuite les données de modèle de saisie à l’API REST TypingDNA. Lors de l’inscription, le [point de terminaison de vérification de l’utilisateur](https://api.typingdna.com/index.html#api-API_Services-GetUser) est appelé pour confirmer que l’utilisateur n’existe pas, puis le point de terminaison du [modèle d’enregistrement](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern) est appelé pour enregistrer le premier modèle de saisie de l’utilisateur.

> [!NOTE]
> Tous les appels au point de terminaison de l’API REST TypingDNA envoient un UserId. Il doit s’agir d’une valeur hachée. Azure AD B2C utilise la `HashObjectIdWithEmail` transformation de revendications pour hacher l’e-mail avec un salt aléatoire et une valeur secrète.  

Les appels de l’API REST sont modelés avec `validationTechnicalProfiles` dans `LocalAccountSignUpWithLogonEmail-TDNA` :

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonEmail-TDNA" />
    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="true"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser"/>

</ValidationTechnicalProfiles>

```

### <a name="sign-in"></a>Connexion

Lors de la prochaine connexion, le modèle de saisie de l’utilisateur est calculé de la même manière qu’au moment de l’inscription à l’aide du [code HTML personnalisé](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/TypingDNA/source-code/selfAssertedSignIn.cshtml). Une fois que le profil de saisie se trouve dans le jeu de revendications Azure AD B2C, Azure AD B2C appellera votre API pour appeler le point de terminaison de l’API REST de TypingDNA. Le point de terminaison de l’[utilisateur de vérification](https://api.typingdna.com/index.html#api-API_Services-GetUser) est appelé pour confirmer que l’utilisateur existe. Ensuite, le point de terminaison [vérifier le modèle](https://api.typingdna.com/index.html#api-API_Services-verifyTypingPattern) est appelé pour retourner le `net_score`. Ce `net_score` indique à quel point le modèle de frappe était proche du modèle d’origine lors de l’inscription.

Ce modèle de saisie est modélisé avec `validationTechnicalProfiles` dans `SelfAsserted-LocalAccountSignin-Email-TDNA` :

```xml

<ValidationTechnicalProfiles>

    <ValidationTechnicalProfile ReferenceId="login-NonInteractive"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-CheckUser" ContinueOnError="false"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-VerifyUser"/>

    <ValidationTechnicalProfile ReferenceId="REST-TDNA-SaveUser">

        <Preconditions>

            <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>saveTypingPattern</Value>

            <Value>False</Value>

            <Action>SkipThisValidationTechnicalProfile</Action>

            </Precondition>

        </Preconditions>

    </ValidationTechnicalProfile>

</ValidationTechnicalProfiles>

```

 Si l’utilisateur obtient un modèle de saisie affichant un `net_score` élevé, vous pouvez l’enregistrer à l’aide du point de terminaison TypingDNA [enregistrer le type de saisie](https://api.typingdna.com/index.html#api-API_Services-saveUserPattern).  

Votre API doit retourner une revendication `saveTypingPattern` si vous souhaitez que le point de terminaison TypingDNA enregistrer le modèle de saisie soit appelé par Azure AD B2C (via votre API).

L’exemple dans le référentiel contient une API (TypingDNA-API-interface) qui est configurée avec les propriétés suivantes.

- Mode de formation : si l’utilisateur compte moins de deux modèles enregistrés, demandez toujours l’authentification multifacteur.

- Si l’utilisateur compte entre deux et cinq modèles enregistrés et que le `net_score` est inférieur à 50, demandez l’authentification multifacteur.

- Si l’utilisateur compte plus de cinq modèles enregistrés et que le `net_score` est inférieur à 65, demandez l’authentification multifacteur.

Ces seuils doivent être ajustés selon votre cas d’utilisation.

- Une fois que votre API a évalué le `net_score`, elle doit retourner une revendication booléenne à B2C - `promptMFA`.

- La revendication `promptMFA` est utilisée dans une condition préalable à l’exécution conditionnelle d’Azure AD MFA.

```xml

<OrchestrationStep Order="3" Type="ClaimsExchange">

    <Preconditions>

        <Precondition Type="ClaimsExist" ExecuteActionsIf="true">

            <Value>isActiveMFASession</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">

            <Value>promptMFA</Value>

            <Value>False</Value>

            <Action>SkipThisOrchestrationStep</Action>

        </Precondition>

    </Preconditions>

    <ClaimsExchanges>

        <ClaimsExchange Id="PhoneFactor-Verify" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" />

    </ClaimsExchanges>

</OrchestrationStep>

```

## <a name="onboard-with-typingdna"></a>Intégration avec TypingDNA

1. Inscription à TypingDNA [ici](https://www.typingdna.com/)
2. Connectez-vous au tableau de bord TypingDNA et obtenez une **clé API** et une **clé secrète API**. Cela sera nécessaire ultérieurement dans la configuration de l’interface API.

## <a name="integrate-typingdna-with-azure-ad-b2c"></a>Intégrer TypingDNA avec Azure AD B2C

1. Hébergez le code [TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) auprès du fournisseur d’hébergement de votre choix.
2. Remplacez toutes les instances de `apiKey` et `apiSecret` dans la solution [TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface) par les informations d’identification provenant de votre tableau de bord TypingDNA.
3. Hébergez les fichiers HTML auprès du fournisseur de votre choix en suivant les exigences CORS [ici](./customize-ui-with-html.md#3-configure-cors)
4. Remplacez les éléments LoadURI pour les définitions de contenu `api.selfasserted.tdnasignup` et `api.selfasserted.tdnasignin` dans le fichier `TrustFrameworkExtensions.xml` respectivement par l’URI de vos fichiers HTML hébergés.
5. Créez une clé de stratégie B2C sous infrastructure d’expérience d’identité dans le panneau Azure AD du **Portail Azure**. Utilisez l’option `Generate` et nommez cette clé `tdnaHashedId`.
6. Remplacer les TenantId dans les fichiers de stratégie
7. Remplacez les URL de services dans tous les profils techniques de l’API REST TypingDNA (REST-TDNA-VerifyUser, REST-TDNA-SaveUser, REST-TDNA-CheckUser) par le point de terminaison de votre [API TypingDNA-API-Interface](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/source-code/TypingDNA-API-Interface).
8. Téléchargez les [fichiers de stratégie](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/TypingDNA/policy) dans votre locataire.

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire B2C et choisissez l’infrastructure d’expérience d’identité.
2. Sélectionnez votre **Flux d’utilisateurs** précédemment créé.
3. Sélectionnez **Exécuter** le flux utilisateur

    a. **Application** : sélectionner l’application enregistrée (l’exemple concerne JWT)

    b. **URL de réponse** : sélectionnez l’URL de redirection

    c. Sélectionnez **Exécuter le flux utilisateur**.
  
4. Suivez le processus d’inscription et créez un compte
5. Se déconnecter
6. Suivez le processus d’inscription
7. Le résultat JWT affichera les résultats TypingDNA

## <a name="live-version"></a>Version en direct

• L’authentification multifacteur a été désactivée dans cette version test, mais vous pouvez voir si l’authentification MFA aurait été demandée par la revendication `promptMFA` après l’authentification.

• Inscrivez-vous [ici](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SU_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login) et connectez-vous [ici](https://b2cprod.b2clogin.com/b2cprod.onmicrosoft.com/oauth2/v2.0/authorize?p=B2C_1A_SI_TDNA&client_id=51d907f8-db14-4460-a1fd-27eaeb2a74da&nonce=defaultNonce&redirect_uri=https://jwt.ms/&scope=openid&response_type=id_token&prompt=login)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans AAD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans AAD B2C](./custom-policy-get-started.md?tabs=applications)