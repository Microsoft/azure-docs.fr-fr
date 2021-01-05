---
title: Inscrire une application SaaS – Place de marché Azure
description: Découvrez comment inscrire une application SaaS à partir du portail Azure et recevoir un jeton de sécurité Azure Active Directory.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 06/10/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: 39a0830806d2d9c7358d175cae703e9c81c45b02
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130015"
---
# <a name="register-a-saas-application"></a>Inscrire une application SaaS

Cet article explique comment inscrire une application SaaS à l’aide du [Portail Microsoft Azure](https://portal.azure.com/) et comment obtenir le jeton d’accès de l’éditeur (jeton d’accès Azure Active Directory). L’éditeur utilisera ce jeton pour authentifier l’application SaaS en appelant les API d’approvisionnement SaaS.  Celles-ci s’appuient sur les informations d’identification du client OAuth 2.0 pour accorder un flux sur les points de terminaison Azure Active Directory (v1.0) afin d’effectuer une demande de jeton d’accès de service à service.

La Place de marché Azure n’impose aucune contrainte sur la méthode d’authentification utilisée par votre service SaaS pour les utilisateurs finaux. Le flux ci-dessous n’est nécessaire que pour l’authentification du service SaaS sur la Place de marché Azure.

Pour plus d’informations sur Azure AD (Active Directory), consultez [Qu’est-ce que l’authentification ?](../../active-directory/develop/authentication-vs-authorization.md).

## <a name="register-an-azure-ad-secured-app"></a>Inscrire une application sécurisée Azure AD

Toute application qui souhaite utiliser les fonctionnalités d’Azure AD doit d’abord être enregistrée dans un locataire Azure AD. Ce processus d’inscription implique de donner à Azure AD quelques informations sur votre application. Pour inscrire une nouvelle application à l’aide du portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Si votre compte vous propose plusieurs accès, cliquez sur votre compte en haut à droite et définissez votre session de portail pour le locataire Azure AD souhaité.
3. Dans le volet de navigation gauche, cliquez sur le service **Azure Active Directory** , cliquez sur **Inscriptions des applications** , puis cliquez sur **Nouvelle inscription d’application**.

    ![Inscriptions des applications SaaS](./media/saas-offer-app-registration-v1.png)

4. Sur la page, saisissez les informations d\'inscription de votre application :
    -   **Name**  : saisissez un nom d’application explicite
    -   **Type d’application**  :  
        
        Sélectionnez **Application web/API** pour les [applications clientes](../../active-directory/develop/developer-glossary.md#client-application) et les [ressources/applications API](../../active-directory/develop/developer-glossary.md#resource-server) installées sur un serveur sécurisé. Ce paramètre est utilisé pour les [clients web](../../active-directory/develop/developer-glossary.md#web-client) confidentiels OAuth et les [clients basés sur un agent utilisateur](../../active-directory/develop/developer-glossary.md#user-agent-based-client) publics.
        La même application peut également exposer un client et une ressource/API.

        Pour obtenir des exemples spécifiques d’applications web, consultez les configurations guidées de démarrage rapide disponibles dans la section [Bien démarrer](../../active-directory/develop/quickstart-create-new-tenant.md) du [Guide du développeur Azure AD](../../active-directory/develop/index.yml).

5. Lorsque vous avez terminé, cliquez sur **Inscrire**.  Azure AD affecte un *ID d’application* unique à votre nouvelle application. Nous vous recommandons d’inscrire une seule application qui accède à l’API, et en tant que locataire unique.

6. Pour créer la clé secrète client, accédez à la page **Certificats et secrets** , puis cliquez sur **+ Nouveau secret client**.  Veillez à copier la valeur du secret pour pouvoir l’utiliser dans votre code.

**L’ID d’application Azure AD** est associé à votre ID d’éditeur. Veillez donc à utiliser le même *ID d’application* dans toutes vos offres.

>[!Note]
>Si un éditeur possède deux comptes différents dans l’Espace partenaires, il doit utiliser deux ID d’application Azure AD différents.  Chaque compte partenaire de l’Espace partenaires doit avoir recours à un ID d’application Azure AD unique pour toutes les offres SaaS publiées par son intermédiaire.

## <a name="how-to-get-the-publishers-authorization-token"></a>Procédure à suivre pour récupérer le jeton d’autorisation de l’éditeur

Une fois que vous avez inscrit votre application, vous pouvez demander programmatiquement le jeton d’autorisation de l’éditeur (jeton d’accès Azure AD, à l’aide du point de terminaison v1 Azure AD). L’éditeur doit utiliser ce jeton pour appeler les différentes API d’approvisionnement SaaS. Ce jeton n’est valide que pendant une heure. 

Pour plus d’informations sur ces jetons, consultez [Jetons d’accès Azure Active Directory](../../active-directory/develop/access-tokens.md).  Notez que, dans les versions antérieures à la v1 du flux, c’est le jeton de point de terminaison qui est utilisé.

### <a name="get-the-token-with-an-http-post"></a>Récupération du jeton avec une requête HTTP POST

#### <a name="http-method"></a>Méthode HTTP

Post<br>

##### <a name="request-url"></a>*URL de requête* 

`https://login.microsoftonline.com/*{tenantId}*/oauth2/token`

##### <a name="uri-parameter"></a>*Paramètre URI*

|  Nom du paramètre    |  Obligatoire         |  Description |
|  ---------------   |  ---------------  | ------------ |
|  `tenantId`        |  True      |  ID de locataire de l’application AAD inscrite. |

##### <a name="request-header"></a>*En-tête de requête*

|  Nom de l’en-tête       |  Obligatoire         |  Description |
|  ---------------   |  ---------------  | ------------ |
|  `content-type`    |  True      |  Type de contenu associé à la requête. La valeur par défaut est `application/x-www-form-urlencoded`. |

##### <a name="request-body"></a>*Corps de la demande*

|  Nom de la propriété     |  Obligatoire         |  Description |
|  ---------------   |  ---------------  | ------------ |
|  `grant_type`      |  True      |  Type d’autorisation. Utilisez `"client_credentials"`. |
|  `client_id`       |  True      |  Identificateur du client/de l’application associé à l’application Azure AD. |
|  `client_secret`   |  True      |  Secret associé à l’application Azure AD. |
|  `resource`        |  True      |  Ressource cible pour laquelle le jeton est demandé. Utilisez `20e940b3-4c77-4b0b-9a53-9e16a1b010a7`, car l’API SaaS de la Place de marché est toujours la ressource cible dans ce cas. |

##### <a name="response"></a>*Réponse*

|  Nom     |  Type         |  Description |
|  ------   |  ---------------  | ------------ |
|  200 OK   |  TokenResponse    |  Demande réussie. |

##### <a name="tokenresponse"></a>*TokenResponse*

Exemple de réponse :

```json
{
      "token_type": "Bearer",
      "expires_in": "3600",
      "ext_expires_in": "0",
      "expires_on": "15251…",
      "not_before": "15251…",
      "resource": "20e940b3-4c77-4b0b-9a53-9e16a1b010a7",
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayIsImtpZCI6ImlCakwxUmNxemhpeTRmcHhJeGRacW9oTTJZayJ9…"
  }
```

La valeur du champ `"access_token"` dans la réponse est le `<access_token>` que vous donnerez comme paramètre d’autorisation pour appeler toutes les API d’approvisionnement SaaS et de mesure de la Place de marché.

## <a name="next-steps"></a>Étapes suivantes

Votre application sécurisée Azure AD peut maintenant utiliser [l’API de traitement des commandes SaaS version 2](./pc-saas-fulfillment-api-v2.md).