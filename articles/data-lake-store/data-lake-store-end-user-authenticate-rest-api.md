---
title: Authentification des utilisateurs finaux - REST avec Data Lake Storage Gen1 - Azure
description: Découvrez comment authentifier les utilisateurs finaux auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory et de l’API REST
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 462cd06c9da3b1f0a57c293d52c59181372b709b
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103745"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-rest-api"></a>Authentification des utilisateurs finaux auprès d’Azure Data Lake Storage Gen1 avec l’API REST
> [!div class="op_single_selector"]
> * [À l’aide de Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Utilisation du kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Utilisation de Python](data-lake-store-end-user-authenticate-python.md)
> * [Utilisation de l'API REST](data-lake-store-end-user-authenticate-rest-api.md)
> 
>  

Dans cet article, vous allez apprendre à utiliser l’API REST pour authentifier les utilisateurs finaux auprès d’Azure Data Lake Storage Gen1. Pour plus d’informations sur l’authentification de service à service auprès de Data Lake Storage Gen1 à l’aide de l’API REST, consultez [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-service-to-service-authenticate-rest-api.md).

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Créez une application « native » Azure Active Directory**. Vous devez avoir suivi la procédure dans [End-user authentication with Data Lake Storage Gen1 using Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md) (Authentification des utilisateurs finaux avec Data Lake Storage Gen1 à l’aide d’Azure Active Directory).

* **[cURL](https://curl.haxx.se/)** . Cet article utilise cURL pour montrer comment effectuer des appels d’API REST sur un compte Data Lake Storage Gen1.

## <a name="end-user-authentication"></a>Authentification des utilisateurs finaux
L’authentification de l’utilisateur final est l’approche recommandée si vous souhaitez qu’un utilisateur se connecte à votre application à l’aide d’Azure AD. Votre application est en mesure d’accéder aux ressources Azure avec le même niveau d’accès que l’utilisateur qui s’est connecté. L’utilisateur doit fournir ses informations d’identification régulièrement pour que votre application maintienne l’accès.

Conséquence de la connexion de l’utilisateur final : votre application reçoit un jeton d’accès et un jeton d’actualisation. Le jeton d’accès est lié à chaque requête adressée au Data Lake Storage Gen1 ou à Data Lake Analytics et, par défaut, il est valide pendant une heure. Le jeton d’actualisation peut être utilisé pour obtenir un nouveau jeton d’accès et, par défaut, il est valide pendant deux semaines au maximum, s’il est régulièrement utilisé. Vous pouvez utiliser deux approches différentes pour la connexion de l’utilisateur final.

Dans ce scénario, l’application invite l’utilisateur à se connecter. Toutes les opérations sont effectuées dans le contexte de l’utilisateur. Procédez comme suit :

1. Dans votre application, redirigez l’utilisateur vers l’URL suivante :

    `https://login.microsoftonline.com/<TENANT-ID>/oauth2/authorize?client_id=<APPLICATION-ID>&response_type=code&redirect_uri=<REDIRECT-URI>`

   > [!NOTE]
   > \<REDIRECT-URI> doit être encodé pour être utilisé dans une URL. Donc, pour https://localhost , utilisez `https%3A%2F%2Flocalhost` )

    Pour les besoins de ce didacticiel, vous pouvez remplacer les valeurs d’espace réservé de l’URL ci-dessus et la coller dans la barre d’adresse d’un navigateur web. Vous serez redirigé pour vous authentifier à l’aide de vos informations de connexion Azure. Lorsque vous êtes connecté, la réponse s’affiche dans la barre d’adresse du navigateur. La réponse présente le format suivant :

    `http://localhost/?code=<AUTHORIZATION-CODE>&session_state=<GUID>`

2. Capturez le code d’autorisation de la réponse. Pour ce didacticiel, vous pouvez copier le code d’autorisation de la barre d’adresse du navigateur web et le transmettre dans la demande POST au point de terminaison de jeton, comme indiqué dans l’extrait de code suivant :

    ```console
    curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token \
    -F redirect_uri=<REDIRECT-URI> \
    -F grant_type=authorization_code \
    -F resource=https://management.core.windows.net/ \
    -F client_id=<APPLICATION-ID> \
    -F code=<AUTHORIZATION-CODE>
    ```

   > [!NOTE]
   > Dans ce cas, il n’est pas nécessaire d’encoder \<REDIRECT-URI>.
   > 
   > 

3. La réponse est un objet JSON contenant un jeton d’accès (par exemple, `"access_token": "<ACCESS_TOKEN>"`) et un jeton d’actualisation (par exemple, `"refresh_token": "<REFRESH_TOKEN>"`). Votre application utilise le jeton d’accès pour accéder à Azure Data Lake Storage Gen1 et le jeton d’actualisation pour obtenir un autre jeton d’accès lorsque l’un d’eux expire.

    ```json
    {"token_type":"Bearer","scope":"user_impersonation","expires_in":"3599","expires_on":"1461865782","not_before":    "1461861882","resource":"https://management.core.windows.net/","access_token":"<REDACTED>","refresh_token":"<REDACTED>","id_token":"<REDACTED>"}
    ```

4. Quand le jeton d’accès arrive à expiration, vous pouvez en demander un nouveau à l’aide du jeton d’actualisation, comme indiqué dans l’extrait de code suivant :

    ```console
    curl -X POST https://login.microsoftonline.com/<TENANT-ID>/oauth2/token  \
         -F grant_type=refresh_token \
         -F resource=https://management.core.windows.net/ \
         -F client_id=<APPLICATION-ID> \
         -F refresh_token=<REFRESH-TOKEN>
    ```

Pour plus d’informations sur l’authentification utilisateur interactive, consultez [Flux d’octroi d’un code d’autorisation](/previous-versions/azure/dn645542(v=azure.100)).

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à utiliser l’authentification de service à service auprès d’Azure Data Lake Storage Gen1 avec l’API REST. Vous pouvez à présent consulter les articles ci-après, qui expliquent comment utiliser l’API REST pour travailler avec Azure Data Lake Storage Gen1.

* [Opérations de gestion des comptes sur Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-get-started-rest-api.md)
* [Opérations sur les données dans Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-data-operations-rest-api.md)