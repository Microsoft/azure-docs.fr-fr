---
title: Tutoriel pour configurer Azure Active Directory B2C avec Zscaler
titleSuffix: Azure AD B2C
description: Dans ce tutoriel, découvrez comment intégrer l’authentification Azure AD B2C avec Zscaler
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/09/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: ff51c2a71dfcaec580733a92e265628ac816e229
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095815"
---
# <a name="tutorial-to-configure-zscaler-private-access-with-azure-active-directory-b2c-for-secure-hybrid-access"></a>Tutoriel pour configurer l’accès privé Zscaler avec Azure Active Directory B2C pour un accès hybride sécurisé

Dans ce tutoriel, découvrez comment intégrer l’authentification Azure AD B2C avec [Zscaler Private Access (ZPA)](https://www.zscaler.com/products/zscaler-private-access). ZPA fournit un accès sécurisé et basé sur des stratégies aux applications privées et aux ressources, sans les risques liés aux coûts, à la complexité ou à la sécurité d’un réseau privé virtuel (VPN). L’offre d’accès hybride sécurisé de Zscaler active une surface d’attaque nulle pour les applications accessibles aux consommateurs lorsqu’elle est associée à Azure AD B2C.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure. Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/tutorial-create-tenant) lié à votre abonnement Azure.

- Un [abonnement ZPA](https://azuremarketplace.microsoft.com/marketplace/apps/aad.zscalerprivateaccess?tab=Overview)

## <a name="scenario-description"></a>Description du scénario

L’intégration de ZPA inclut les composants suivants :

- Azure AD B2C : fournisseur d’identité chargé de vérifier les informations d’identification de l’utilisateur, également chargé de l’inscription d’un nouvel utilisateur.

- ZPA : service responsable de la sécurisation de l’application web en appliquant l’[accès Confiance Zéro](https://www.microsoft.com/security/blog/2018/12/17/zero-trust-part-1-identity-and-access-management/#:~:text=Azure%20Active%20Directory%20%28Azure%20AD%29%20provides%20the%20strong%2C,to%20express%20their%20access%20requirements%20in%20simple%20terms.).

- Application web : héberge le service auquel l’utilisateur tente d’accéder.

Le diagramme suivant décrit comment ZPA s’intègre à Azure AD B2C.

![L’image montre le diagramme d’architecture Zscaler](media/partner-zscaler/zscaler-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur arrive sur un portail utilisateur ZPA ou une application d’accès au navigateur ZPA.
| 2. | ZPA requiert les informations de contexte utilisateur avant de pouvoir décider s’il faut autoriser l’utilisateur à accéder à l’application web. Pour authentifier l’utilisateur, ZPA effectue une redirection SAML vers la page de connexion Azure AD B2C.  
| 3. | L’utilisateur arrive sur la page de connexion Azure AB B2C. S’il s’agit d’un nouvel utilisateur, celui-ci s’inscrit pour créer un compte. Un utilisateur existant se connecte à l’aide de ses informations d’identification. Azure AD B2C valide l’identité de l’utilisateur.
| 4. | Une fois l’authentification réussie, Azure AD B2C redirige l’utilisateur vers ZPA avec l’assertion SAML. ZPA vérifie l’assertion SAML et définit le contexte utilisateur.
| 5. | ZPA évalue les stratégies d’accès de l’utilisateur. Si l’utilisateur est autorisé à accéder à l’application web, la connexion est autorisée.

## <a name="onboard-to-zpa"></a>Intégration à ZPA

Ce tutoriel part du principe que vous disposez déjà d’une installation fonctionnelle de ZPA. Si vous commencez tout juste à utiliser ZPA, reportez-vous au [Guide de configuration pas à pas pour ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa).

## <a name="integrate-with-azure-ad-b2c"></a>Intégrer avec Azure AD B2C

### <a name="part-1---configure-azure-ad-b2c-as-an-idp-on-zpa"></a>Partie 1 : Configurer Azure AD B2C en tant que fournisseur d’identité sur ZPA

Pour configurer Azure AD B2C en tant que [fournisseur d’identité sur ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign) :

1. Connectez-vous au [portail d’administration ZPA](https://admin.private.zscaler.com).

2. Accédez à **Administration** > **Configuration du fournisseur d’identité**.

3. Sélectionnez **Ajouter une configuration de fournisseur d’identité**.

4. Pour **1 Informations sur le fournisseur d’identité**, entrez ce qui suit :

   a. **Nom** : Azure AD B2C

   b. **Authentification unique** : Sélectionner un utilisateur

   c. **Domaines** : Choisissez les domaines d’authentification que vous souhaitez associer à ce fournisseur d’identité, puis sélectionnez **Terminé**.

   ![L’image montre les informations du fournisseur d’identité](media/partner-zscaler/add-idp-configuration.png)

5. Sélectionnez **Suivant**.

6. Pour **2 Métadonnées du fournisseur de service** :

   a. Copiez l’URL du fournisseur de service et notez-la pour une utilisation ultérieure.

   b. Copiez l’ID d’entité du fournisseur de service et notez-le pour une utilisation ultérieure.

   ![L’image affiche les informations de métadonnées du fournisseur de service](media/partner-zscaler/sp-metadata.png)

7. Sélectionnez **Suspendre**.

Le reste de la configuration du fournisseur d’identité reprend après la configuration d’Azure AD B2C.

### <a name="part-2---configure-custom-policy-in-azure-ad-b2c"></a>Partie 2 : Configurer une stratégie personnalisée dans Azure AD B2C

>[!Note]
>Cette étape est requise uniquement si vous n’avez pas encore de stratégies personnalisées. Si vous disposez déjà d’une ou de plusieurs stratégies personnalisées, vous pouvez ignorer cette étape.

L’article [Prise en main des stratégies personnalisées dans Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started) fournit des instructions sur la façon de configurer des stratégies personnalisées sur votre locataire Azure AD B2C.

### <a name="part-3---register-zpa-as-a-saml-application-in-azure-ad-b2c"></a>Partie 3 - Inscrire ZPA en tant qu’application SAML dans Azure AD B2C

L’article [Inscrire une application SAML dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/connect-with-saml-service-providers) fournit des instructions sur la configuration d’une application SAML dans Azure AD B2C. Dans l’[étape 3.2](https://docs.microsoft.com/azure/active-directory-b2c/connect-with-saml-service-providers#32-upload-and-test-your-policy-metadata), vous recevrez l’URL de métadonnées SAML de fournisseur d’identité utilisée par Azure AD B2C. Vous en aurez besoin par la suite.

Suivez les instructions jusqu’à l’[étape 4.2](https://docs.microsoft.com/azure/active-directory-b2c/connect-with-saml-service-providers#42-update-the-app-manifest). À l’étape 4.2, vous devez mettre à jour le manifeste de l’application. Mettez à jour les propriétés comme suit :

- **identifierUris** : utilisez l’ID d’entité du fournisseur de service mentionné dans **Partie 1, étape 6A**.

- **samlMetadataUrl** : ignorez cette propriété, car ZPA n’héberge pas d’URL de métadonnées SAML.

- **replyUrlsWithType** : utilisez l’URL du fournisseur de service notée dans **Partie 1, étape 6B**.

- **logoutUrl** : ignorez cette propriété, car ZPA ne prend pas en charge une URL de déconnexion.

Le reste des étapes ne s’applique pas à ce tutoriel.

### <a name="part-4---extract-the-idp-saml-metadata-from-azure-ad-b2c"></a>Partie 4 : Extraction des métadonnées SAML du fournisseur d’identité à partir d’Azure AD B2C

De l’étape précédente, vous devez obtenir une URL de métadonnées SAML au format suivant :

```https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/Samlp/metadata```

où `<tenant-name>` est le nom de votre locataire Azure AD B2C et `<policy-name>` est le nom de la stratégie SAML personnalisée que vous avez créée à la dernière étape.

Par exemple : https://safemarch.b2clogin.com/safemarch.onmicrosoft.com/B2C_1A_signup_signin_saml//Samlp/metadata

Ouvrez un navigateur web et accédez à l’URL des métadonnées SAML. Lorsque la page se charge, cliquez avec le bouton droit n’importe où sur la page. Sélectionnez **Enregistrer la page sous** et enregistrez le fichier sur votre ordinateur, car vous allez l’utiliser dans la partie suivante.

### <a name="part-5---complete-idp-configuration-on-zpa"></a>Partie 5 : Terminer la configuration du fournisseur d’identité sur ZPA

Terminez la [configuration du fournisseur d’identité dans le portail d’administration ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign) partiellement effectuée dans la partie 1 :

1. Connectez-vous au [portail d’administration ZPA](https://admin.private.zscaler.com).

2. Accédez à **Administration** > **Configuration du fournisseur d’identité**.

3. Sélectionnez le fournisseur d’identité configuré dans la partie 1, puis sélectionnez **Reprendre**.

4. Pour **3 Créer le fournisseur d’identité**

   a. Accédez à **Fichier de métadonnées du fournisseur d’identité** > **Sélectionner le fichier** et chargez le fichier de métadonnées que vous avez enregistré dans la partie 4.

   b. Vérifiez que l’**état** de la configuration du fournisseur d’identité est **Activé**.

   c. Sélectionnez **Enregistrer**.

      ![L’image montre la création des informations du fournisseur d’identité](media/partner-zscaler/create-idp.png)

## <a name="test-the-solution"></a>Test de la solution

Accédez à un portail utilisateur ZPA ou à une application d’accès au navigateur et testez le processus d’inscription ou de connexion. Cela doit aboutir à une authentification SAML réussie.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Bien démarrer avec les stratégies personnalisées dans Azure Active Directory B2C](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started)

- [Inscrire une application SAML dans Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/connect-with-saml-service-providers)

- [Guide de configuration pas à pas pour ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa)

- [Configuration d’un fournisseur d’identité pour l’authentification unique](https://help.zscaler.com/zpa/configuring-idp-single-sign)
