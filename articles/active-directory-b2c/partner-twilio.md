---
title: Application Twilio Verify avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer un exemple d’application de paiement en ligne dans Azure AD B2C avec l’API Twilio Verify. Conformez-vous à la directive PSD2 (Payment Services Directive 2) sur les transactions grâce à la liaison dynamique et à l’authentification forte du client.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 953653a758577ed3d48ca2d81403b4cb363ea294
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994041"
---
# <a name="integrating-twilio-verify-app-with-azure-active-directory-b2c"></a>Intégration de l’application Twilio Verify à Azure Active Directory B2C

Dans ce guide pas à pas, découvrez comment intégrer un exemple d’application de paiement en ligne dans Azure AD B2C (Azure Active Directory B2C) avec l’API Twilio Verify. L’API Twilio Verify aide les clients Azure AD B2C à se conformer à la directive PSD2 (Payment Services Directive 2) sur les transactions par le biais de la liaison dynamique et de l’authentification forte du client.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* Un [locataire Azure AD B2C](tutorial-create-tenant.md) qui est lié à votre abonnement Azure.
* Un [compte d’essai](https://www.twilio.com/try-twilio) sur Twilio.

## <a name="scenario-description"></a>Description du scénario

La solution Twilio se compose des éléments suivants :

- Une [application web de démonstration .NET PSD2](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App), qui permet de se connecter ou de s’inscrire et d’effectuer une transaction factice à haut risque.
- Une [stratégie d’inscription et de connexion](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy) Azure AD B2C combinée.
- Une stratégie Azure AD B2C intégrée à l’API Twilio Verify, qui utilise un jeton `id_token_hint`.
- Une application web .NET, qui héberge un point de terminaison OpenIdConnect `.well-known` utilisé pour valider `id_token_hint`.


    ![flux twilio](media/partner-twilio/twilio-flow.png)

| Étape | Description |
|------|------|
| 1     | Un utilisateur tente de se connecter à l’application de démonstration PSD2 ou de s’y inscrire. L’utilisateur est authentifié au moyen de la stratégie Azure AD B2C combinée d’inscription et de connexion. Un jeton est renvoyé à l’application. À l’inscription, le numéro de téléphone de l’utilisateur est vérifié par SMS/téléphone, puis enregistré dans le compte Azure AD B2C associé.     |
| 2     | L’utilisateur démarre une transaction à haut risque, par exemple, un virement de 50,00 $. Le jeton d’accès actif de l’utilisateur est évalué pour le PolicyId afin de déterminer si l’utilisateur s’est déjà authentifié par le biais d’une stratégie personnalisée renforcée (step-up).     |
| 3     | L’application enregistre la valeur (50,00 $) et le bénéficiaire (John Doe) de la transaction, puis génère un jeton signé. Ce jeton, appelé `id_token_hint`, contient la revendication `amount:$500, payee:john doe`. `id_token_hint` est envoyé avec la demande à la stratégie personnalisée Azure AD B2C, qui est intégrée à Twilio.     |
| 4     | Azure AD B2C vérifie la signature du jeton id_token_hint en contrôlant le point de terminaison OpenId Connect `/.well-known` des applications. Après la vérification, il extrait les revendications de ce jeton, à savoir `amount` et `payee`. L’utilisateur voit alors une page qui permet de vérifier son numéro de téléphone mobile par SMS.     |
| 5     | L’utilisateur fait la demande de vérification de son numéro de téléphone par SMS, et Azure AD B2C envoie une demande d’API REST au point de terminaison de l’API Twilio Verify. Il envoie également les revendications `amount` et `payee` de la transaction dans le cadre du processus PSD2 pour générer le code secret à usage unique (OTP). Twilio envoie un SMS au numéro de téléphone enregistré de l’utilisateur.     |
| 6     |  L’utilisateur entre le code secret à usage unique qu’il a reçu par SMS et le soumet à Azure AD B2C. Azure AD B2C effectue une demande d’API avec ce code secret à l’API Twilio Verify pour vérifier que le code secret est correct. Pour finir, un jeton est envoyé à l’application, avec un nouveau PolicyId qui signifie que l’utilisateur a bien effectué l’authentification renforcée.    |
|      |      |

## <a name="onboard-with-twilio"></a>Intégration de Twilio

1. Obtenez un [compte d’essai](https://www.twilio.com/try-twilio) sur Twilio.

2. Achetez un numéro de téléphone sur Twilio comme cela est décrit dans [cet article](https://support.twilio.com/hc/articles/223135247-How-to-Search-for-and-Buy-a-Twilio-Phone-Number-from-Console)

3. Accédez à l’[API Verify](https://www.twilio.com/console/verify/services) dans la console Twilio et suivez les [instructions](https://www.twilio.com/docs/verify/verifying-transactions-psd2) données pour créer un service et activer l’option PSD2.  

## <a name="configure-the-psd2-demo-app"></a>Configurer l’application de démonstration PSD2

1. Ouvrez la solution  B2C-WebAPI-DotNet et remplacez les valeurs suivantes par les valeurs propres à votre locataire dans le fichier web.config :

    ```xml
   <add key="ida:Tenant" value="yourtenant.onmicrosoft.com" />
   <add key="ida:TenantId" value="d6f33888-0000-4c1f-9b50-1590f171fc70" />
   <add key="ida:ClientId" value="6bd98cc8-0000-446a-a05e-b5716ef2651b" />
   <add key="ida:ClientSecret" value="secret" />
   <add key="ida:AadInstance" value="https://yourtenant.b2clogin.com/tfp/{0}/{1}" />
   <add key="ida:RedirectUri" value="https://your hosted psd2 demo app url/" />
   ```

2. L’[application web](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/source-code/PSD2%20Demo%20App) héberge également le générateur de jetons id_token_hint et le point de terminaison des métadonnées.
   - Créez votre certificat de signature comme cela est expliqué dans cet [exemple](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#creating-a-signing-certificate).
   - Modifiez les lignes suivantes pour votre certificat dans le fichier web.config :
   
   ```xml
   <add key="ida:SigningCertThumbprint" value="4F39D6014818082CBB763E5BA5F230E545212E89" />
   <add key="ida:SigningCertAlgorithm" value="RS256" />
   ```

3. Chargez l’application de démonstration sur le fournisseur d’hébergement de votre choix. Vous trouverez des conseils pour Azure App Service dans  [cet exemple](https://github.com/azure-ad-b2c/samples/tree/master/policies/invite#hosting-the-application-in-azure-app-service), y compris des instructions pour le chargement de votre certificat.

4. Modifiez votre inscription dans l’application Azure AD B2C en ajoutant une URL de réponse correspondant à l’URL où l’application est hébergée.

5. Ouvrez les [fichiers de stratégie](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Twilio-VerifyAPI/policy) et remplacez toutes les instances de  `contoso` par le nom de votre locataire.

6. Recherchez le profil technique de l’API REST de Twilio, **Custom-SMS-Enroll**. Remplacez la valeur `ServiceURL` par votre AccountSID Twilio et la valeur From par le numéro de téléphone que vous avez acheté.

7. Recherchez les profils techniques de l’API REST de Twilio, **TwilioRestAPI-Verify-Step1** et **TwilioRestAPI-Verify-Step2**, puis remplacez la valeur `ServiceURL` par votre AccountSID Twilio.

## <a name="integrate-with-azure-ad-b2c"></a>Intégrer avec Azure AD B2C

Ajoutez les fichiers de stratégie à Azure AD B2C :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général de votre locataire Azure AD B2C.

2. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.

3. Choisissez **Tous les services** dans le coin supérieur gauche du Portail Azure, recherchez et sélectionnez **Azure Active Directory B2C**.

4. Accédez à **Azure AD B2C** > **Identity Experience Framework** > **Clés de stratégie**.

5. Ajoutez une nouvelle clé appelée **B2cRestTwilioClientId**. Sélectionnez **manuel** et spécifiez la valeur de l’AccountSID Twilio.

6. Ajoutez une autre clé appelée **B2cRestTwilioClientSecret**. Sélectionnez **manuel** et spécifiez la valeur du jeton AUTH TOKEN Twilio.

7. Chargez tous les fichiers de stratégie sur votre locataire.

8. Personnalisez la chaîne dans la transformation de revendications GenerateOTPMessageEnrol à utiliser comme texte du SMS d’inscription.

## <a name="test-the-solution"></a>Test de la solution

* Accédez à votre application et testez les actions de connexion, d’inscription et d’envoi d’argent.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Exemples de code d’intégration de Twilio](https://github.com/azure-ad-b2c/samples/tree/master/policies/twilio-mfa-psd2) sur GitHub  

- [Stratégies personnalisées dans AAD B2C](custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans AAD B2C](custom-policy-get-started.md?tabs=applications)
