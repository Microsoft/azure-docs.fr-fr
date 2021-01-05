---
title: Authentification de service à service avec le flux On-Behalf-Of Auth2.0 | Microsoft Docs
description: Cet article explique comment utiliser des messages HTTP pour implémenter l’authentification de service à service avec le flux On-Behalf-Of OAuth 2.0.
services: active-directory
documentationcenter: .net
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 08/5/2020
ms.author: ryanwi
ms.reviewer: hirsin, nacanuma
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: f746cc654934464d907c6ad669eb7470e4dcaeeb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88117734"
---
# <a name="service-to-service-calls-that-use-delegated-user-identity-in-the-on-behalf-of-flow"></a>Appels de service à service utilisant l’identité utilisateur déléguée dans le flux On-Behalf-Of

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Le flux On-Behalf-Of (OBO) OAuth 2.0 permet à une application qui appelle un service ou une API web de passer l’authentification utilisateur à un autre service ou à une autre API web. Le flux OBO propage l’identité et les autorisations de l’utilisateur délégué à travers la chaîne des demandes. Pour que le service de niveau intermédiaire puisse faire des demandes authentifiées au service en aval, il doit sécuriser un jeton d’accès d’Azure Active Directory (Azure AD) pour le compte de l’utilisateur.

> [!IMPORTANT]
> À compter de mai 2018, vous ne pouvez plus utiliser un `id_token` pour le flux On-Behalf-Of.  Les applications à une seule page doivent passer un jeton d’accès à un client confidentiel de la couche intermédiaire pour effectuer des flux OBO. Pour plus d’informations sur les clients pouvant effectuer des appels On-Behalf-Of, consultez [Limitations](#client-limitations).

## <a name="on-behalf-of-flow-diagram"></a>Diagramme du flux Pour le compte de

Le flux OBO commence après que l’utilisateur a été authentifié sur une application qui utilise le [flux d’octroi de code d’autorisation OAuth 2.0](v1-protocols-oauth-code.md). À ce stade, l’application envoie un jeton d’accès (jeton A) à l’API web de niveau intermédiaire (API A) contenant les revendications de l’utilisateur et le consentement pour accéder à l’API A. Ensuite, l’API A fait une demande authentifiée à l’API web en aval (API B).

Ces étapes constituent le flux On-Behalf-Of : ![Indique les étapes dans le flux On-Behalf-Of OAuth 2.0](./media/v1-oauth2-on-behalf-of-flow/active-directory-protocols-oauth-on-behalf-of-flow.png)

1. L’application cliente fait une demande à l’API A avec le jeton A.
1. L’API A s’authentifie auprès du point de terminaison d’émission de jeton Azure AD et demande un jeton pour accéder à l’API B.
1. Le point de terminaison d’émission de jeton Azure AD valide les informations d’identification de l’API A avec le jeton A et émet le jeton d’accès pour l’API B (jeton B).
1. La demande adressée à l’API B contient le jeton B dans l’en-tête d’autorisation.
1. L’API B retourne les données depuis la ressource sécurisée.

>[!NOTE]
>La revendication d’audience dans un jeton d’accès utilisé pour demander un jeton pour un service en aval doit être l’ID du service qui effectue la demande OBO. Le jeton doit également être signé avec la clé de signature globale Azure Active Directory (qui est la valeur par défaut pour les applications inscrites via **Inscriptions d’applications** dans le portail).

## <a name="register-the-application-and-service-in-azure-ad"></a>Inscrire le service et l’application dans Azure AD

Inscrivez le service de niveau intermédiaire et l’application cliente dans Azure AD.

### <a name="register-the-middle-tier-service"></a>Inscrire le service de niveau intermédiaire

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre du haut, sélectionnez votre compte et, sous la liste **Annuaire**, sélectionnez un locataire Active Directory pour votre application.
1. Dans le volet gauche, sélectionnez **Plus de services** et choisissez **Azure Active Directory**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un nom convivial pour l’application, puis sélectionnez le type d’application.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
1. Définir l’URI de redirection sur l’URL de base.
1. Sélectionnez **Inscrire** pour créer l’application.
1. Dans le portail Azure, choisissez votre application, puis sélectionnez **Certificats et secrets**.
1. Sélectionnez **Nouvelle clé secrète client** et ajouter un secret avec une durée d’un ou deux ans.
1. Lorsque vous enregistrez cette page, le portail Azure affiche la valeur du secret. Copiez et enregistrez la valeur du secret dans un endroit sûr.
1. Créez une étendue sur votre application sur la page **Exposer une API** pour votre application, puis cliquez sur « Ajouter une étendue ».  Le portail peut vous obliger à créer également un URI d’ID d’application. 

> [!IMPORTANT]
> Vous avez besoin du secret pour configurer les paramètres de l’application dans votre implémentation. Cette valeur de secret ne sera plus affichée et elle n’est récupérable par aucun autre moyen. Enregistrez-la dès qu’elle est visible dans le portail Azure.

### <a name="register-the-client-application"></a>Inscrire l’application cliente

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre du haut, sélectionnez votre compte et, sous la liste **Annuaire**, sélectionnez un locataire Active Directory pour votre application.
1. Dans le volet gauche, sélectionnez **Plus de services** et choisissez **Azure Active Directory**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un nom convivial pour l’application, puis sélectionnez le type d’application.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft**.
1. Définir l’URI de redirection sur l’URL de base.
1. Sélectionnez **Inscrire** pour créer l’application.
1. Configurez les autorisations pour votre application. Dans **Autorisations des API**, sélectionnez **Ajouter une autorisation**, puis **Mes API**.
1. Tapez le nom du service de niveau intermédiaire dans le champ de texte.
1. Choisissez **Sélectionner les autorisations** puis sélectionnez l’étendue que vous avez créée lors de la dernière étape de l’inscription du niveau intermédiaire.

### <a name="configure-known-client-applications"></a>Configurer les applications clientes connues

Dans ce scénario, le service de niveau intermédiaire doit obtenir le consentement de l’utilisateur pour accéder à l’API en aval sans interaction de l’utilisateur. L’option d’accorder l’accès à l’API en aval doit être présentée au préalable lors de l’étape de consentement pendant l’authentification.

Effectuez les étapes ci-dessous pour lier explicitement l’inscription de l’application cliente dans Azure AD avec l’inscription du service de niveau intermédiaire. Cette opération fusionne le consentement exigé par le client et par le niveau intermédiaire dans une même boîte de dialogue.

1. Accédez à l’inscription du service de niveau intermédiaire, puis sélectionnez **Manifeste** pour ouvrir l’éditeur de manifeste.
1. Recherchez la propriété de tableau `knownClientApplications`, puis ajoutez l’ID client de l’application cliente en tant qu’élément.
1. Enregistrez le manifeste en sélectionnant **Enregistrer**.

## <a name="service-to-service-access-token-request"></a>Demande de jeton d’accès de service à service

Pour demander un jeton d’accès, faites une requête HTTP POST au point de terminaison Azure AD spécifique au locataire, avec les paramètres suivants :

```
https://login.microsoftonline.com/<tenant>/oauth2/token
```

L’application cliente est sécurisée par un secret partagé ou par un certificat.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Premier cas : Requête de jeton d’accès avec un secret partagé

Lorsque l’application utilise un secret partagé, la demande de jeton d’accès de service à service contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| grant_type |Obligatoire | Type de la demande de jeton. Une demande OBO utilise un jeton d’accès JWT : la valeur doit donc être **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |Obligatoire | Valeur du jeton d’accès utilisé dans la requête. |
| client_id |Obligatoire | ID d’application affecté au service appelant lors de l’inscription auprès d’Azure AD. Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, choisissez l’annuaire, puis sélectionnez le nom de l’application. |
| client_secret |Obligatoire | Clé enregistrée pour le service appelant dans Azure AD. Vous devez avoir noté cette valeur au moment de l’inscription. |
| resource |Obligatoire | URI de l’ID d’application du service de destination (ressource sécurisée). Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, puis choisissez l’annuaire. Sélectionnez le nom de l’application, choisissez **Tous les paramètres**, puis sélectionnez **Propriétés**. |
| requested_token_use |Obligatoire | Spécifie comment la demande doit être traitée. Dans le flux Pour le compte de, la valeur doit être **on_behalf_of**. |
| scope |Obligatoire | Liste des étendues (séparées par des espaces) pour la demande de jeton. Pour OpenID Connect, l’étendue **openid** doit être spécifiée.|

#### <a name="example"></a>Exemple

La requête HTTP POST suivante demande un jeton d’accès pour l’API web https://graph.microsoft.com. `client_id` identifie le service qui demande le jeton d’accès.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_secret=0Y1W%2BY3yYb3d9N8vSjvm8WrGzVZaAaHbHHcGbcgG%2BoI%3D
&resource=https%3A%2F%2Fgraph.microsoft.com
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.ewogICJhdWQiOiAiaHR0cHM6Ly9ncmFwaC5taWNyb3NvZnQuY29tIiwKICAiaXNzIjogImh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLAogICJpYXQiOiAxNDkzNDIzMTY4LAogICJuYmYiOiAxNDkzNDIzMTY4LAogICJleHAiOiAxNDkzNDY2OTUxLAogICJhY3IiOiAiMSIsCiAgImFpbyI6ICJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsCiAgImFtciI6IFsKICAgICJwd2QiCiAgXSwKICAiYXBwaWQiOiAiNjI1MzkxYWYtYzY3NS00M2U1LThlNDQtZWRkM2UzMGNlYjE1IiwKICAiYXBwaWRhY3IiOiAiMSIsCiAgImVfZXhwIjogMzAyNjgzLAogICJmYW1pbHlfbmFtZSI6ICJUZXN0IiwKICAiZ2l2ZW5fbmFtZSI6ICJOYXZ5YSIsCiAgImlwYWRkciI6ICIxNjcuMjIwLjEuMTc3IiwKICAibmFtZSI6ICJOYXZ5YSBUZXN0IiwKICAib2lkIjogIjFjZDRiY2FjLWI4MDgtNDIzYS05ZTJmLTgyN2ZiYjFiYjczOSIsCiAgInBsYXRmIjogIjMiLAogICJwdWlkIjogIjEwMDMzRkZGQTEyRUQ3RkUiLAogICJzY3AiOiAiVXNlci5SZWFkIiwKICAic3ViIjogIjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLAogICJ0aWQiOiAiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwKICAidW5pcXVlX25hbWUiOiAibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLAogICJ1cG4iOiAibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLAogICJ1dGkiOiAieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsCiAgInZlciI6ICIxLjAiCn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ
&requested_token_use=on_behalf_of
&scope=openid
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Deuxième cas : Requête de jeton d’accès avec un certificat

Une demande de jeton d’accès de service à service avec un certificat contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| grant_type |Obligatoire | Type de la demande de jeton. Une demande OBO utilise un jeton d’accès JWT : la valeur doit donc être **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |Obligatoire | Valeur du jeton utilisé dans la demande. |
| client_id |Obligatoire | ID d’application affecté au service appelant lors de l’inscription auprès d’Azure AD. Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, choisissez l’annuaire, puis sélectionnez le nom de l’application. |
| client_assertion_type |Obligatoire |La valeur doit être `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| client_assertion |Obligatoire | Jeton web JSON que vous créez et que vous signez avec le certificat inscrit comme informations d’identification pour votre application. Pour plus d’informations sur le format de l’assertion et sur la façon d’inscrire votre certificat, consultez [Informations d’identification des certificats](../develop/active-directory-certificate-credentials.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).|
| resource |Obligatoire | URI de l’ID d’application du service de destination (ressource sécurisée). Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, puis choisissez l’annuaire. Sélectionnez le nom de l’application, choisissez **Tous les paramètres**, puis sélectionnez **Propriétés**. |
| requested_token_use |Obligatoire | Spécifie comment la demande doit être traitée. Dans le flux Pour le compte de, la valeur doit être **on_behalf_of**. |
| scope |Obligatoire | Liste des étendues (séparées par des espaces) pour la demande de jeton. Pour OpenID Connect, l’étendue **openid** doit être spécifiée.|

Ces paramètres sont presque les mêmes que pour la demande par secret partagé, sauf que le `client_secret parameter` est remplacé par deux paramètres : `client_assertion_type` et `client_assertion`.

#### <a name="example"></a>Exemple

La requête HTTP POST suivante demande un jeton d’accès pour l’API web https://graph.microsoft.com avec un certificat. `client_id` identifie le service qui demande le jeton d’accès.

```
// line breaks for legibility only

POST /oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&resource=https%3A%2F%2Fgraph.microsoft.com
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=openid
```

## <a name="service-to-service-access-token-response"></a>Réponse du jeton d’accès de service à service

Une réponse correspondant à une réussite est une réponse JSON OAuth 2.0 avec les paramètres suivants :

| Paramètre | Description |
| --- | --- |
| token_type |Indique la valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le **jeton porteur**. Pour plus d’informations sur les jetons du porteur, consultez le [Framework d’autorisation OAuth 2.0 : Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt). |
| scope |Étendue de l’accès accordé dans le jeton. |
| expires_in |Durée de validité du jeton d’accès (en secondes). |
| expires_on |L’heure d’expiration du jeton d’accès. La date est représentée en nombre de secondes à partir du 1er janvier 1970 (1970-01-01T0:0:0Z) UTC jusqu’au moment de l’expiration. Cette valeur est utilisée pour déterminer la durée de vie des jetons en cache. |
| resource |URI de l’ID d’application du service de destination (ressource sécurisée). |
| access_token |Le jeton d’accès demandé. Le service web appelant peut utiliser ce jeton pour s’authentifier auprès du service destinataire. |
| id_token |Jeton d’ID demandé. Le service appelant peut utiliser ce jeton pour vérifier l’identité de l’utilisateur et démarrer une session avec lui. |
| refresh_token |Jeton d’actualisation pour le jeton d’accès demandé. Le service appelant peut utiliser ce jeton pour demander un autre jeton d’accès après l’expiration du jeton d’accès actuel. |

### <a name="success-response-example"></a>Exemple de réponse correspondant à une réussite

L’exemple suivant illustre une réponse affirmative à une demande de jeton d’accès pour l’API web https://graph.microsoft.com.

```json
{
    "token_type":"Bearer",
    "scope":"User.Read",
    "expires_in":"43482",
    "ext_expires_in":"302683",
    "expires_on":"1493466951",
    "not_before":"1493423168",
    "resource":"https://graph.microsoft.com",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0IiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiLyIsImlhdCI6MTQ5MzQyMzE2OCwibmJmIjoxNDkzNDIzMTY4LCJleHAiOjE0OTM0NjY5NTEsImFjciI6IjEiLCJhaW8iOiJBU1FBMi84REFBQUE1NnZGVmp0WlNjNWdBVWwrY1Z0VFpyM0VvV2NvZEoveWV1S2ZqcTZRdC9NPSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJhcHBpZGFjciI6IjEiLCJlX2V4cCI6MzAyNjgzLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJwdWlkIjoiMTAwMzNGRkZBMTJFRDdGRSIsInNjcCI6IlVzZXIuUmVhZCIsInN1YiI6IjNKTUlaSWJlYTc1R2hfWHdDN2ZzX0JDc3kxa1l1ekZKLTUyVm1Zd0JuM3ciLCJ0aWQiOiIyNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IiLCJ1bmlxdWVfbmFtZSI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXBuIjoibmF2eWFAZGRvYmFsaWFub3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJ4Q3dmemhhLVAwV0pRT0x4Q0dnS0FBIiwidmVyIjoiMS4wIn0.cqmUVjfVbqWsxJLUI1Z4FRx1mNQAHP-L0F4EMN09r8FY9bIKeO-0q1eTdP11Nkj_k4BmtaZsTcK_mUygdMqEp9AfyVyA1HYvokcgGCW_Z6DMlVGqlIU4ssEkL9abgl1REHElPhpwBFFBBenOk9iHddD1GddTn6vJbKC3qAaNM5VarjSPu50bVvCrqKNvFixTb5bbdnSz-Qr6n6ACiEimiI1aNOPR2DeKUyWBPaQcU5EAK0ef5IsVJC1yaYDlAcUYIILMDLCD9ebjsy0t9pj_7lvjzUSrbMdSCCdzCqez_MSNxrk1Nu9AecugkBYp3UVUZOIyythVrj6-sVvLZKUutQ",
    "refresh_token":"AQABAAAAAABnfiG-mA6NTae7CdWW7QfdjKGu9-t1scy_TDEmLi4eLQMjJGt_nAoVu6A4oSu1KsRiz8XyQIPKQxSGfbf2FoSK-hm2K8TYzbJuswYusQpJaHUQnSqEvdaCeFuqXHBv84wjFhuanzF9dQZB_Ng5za9xKlUENrNtlq9XuLNVKzxEyeUM7JyxzdY7JiEphWImwgOYf6II316d0Z6-H3oYsFezf4Xsjz-MOBYEov0P64UaB5nJMvDyApV-NWpgklLASfNoSPGb67Bc02aFRZrm4kLk-xTl6eKE6hSo0XU2z2t70stFJDxvNQobnvNHrAmBaHWPAcC3FGwFnBOojpZB2tzG1gLEbmdROVDp8kHEYAwnRK947Py12fJNKExUdN0njmXrKxNZ_fEM33LHW1Tf4kMX_GvNmbWHtBnIyG0w5emb-b54ef5AwV5_tGUeivTCCysgucEc-S7G8Cz0xNJ_BOiM_4bAv9iFmrm9STkltpz0-Tftg8WKmaJiC0xXj6uTf4ZkX79mJJIuuM7XP4ARIcLpkktyg2Iym9jcZqymRkGH2Rm9sxBwC4eeZXM7M5a7TJ-5CqOdfuE3sBPq40RdEWMFLcrAzFvP0VDR8NKHIrPR1AcUruat9DETmTNJukdlJN3O41nWdZOVoJM-uKN3uz2wQ2Ld1z0Mb9_6YfMox9KTJNzRzcL52r4V_y3kB6ekaOZ9wQ3HxGBQ4zFt-2U0mSszIAA",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiI2MjUzOTFhZi1jNjc1LTQzZTUtOGU0NC1lZGQzZTMwY2ViMTUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8yNjAzOWNjZS00ODlkLTQwMDItODI5My01YjBjNTEzNGVhY2IvIiwiaWF0IjoxNDkzNDIzMTY4LCJuYmYiOjE0OTM0MjMxNjgsImV4cCI6MTQ5MzQ2Njk1MSwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidXRpIjoieEN3ZnpoYS1QMFdKUU9MeENHZ0tBQSIsInZlciI6IjEuMCJ9."
}
```

### <a name="error-response-example"></a>Exemple de réponse d’erreur

Le point de terminaison de jeton Azure AD renvoie une réponse d’erreur quand il tente d’acquérir un jeton d’accès pour une API en aval qui est définie avec une stratégie d’accès conditionnel (par exemple l’authentification multifacteur). Le service de niveau intermédiaire doit faire apparaître cette erreur à l’application cliente afin que celle-ci puisse fournir une interaction utilisateur pour satisfaire la stratégie d’accès conditionnel.

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Utiliser le jeton d’accès pour accéder à la ressource sécurisée

Le service de niveau intermédiaire peut utiliser le jeton obtenu pour faire des demandes authentifiées à l’API web en aval, en définissant le jeton dans l’en-tête `Authorization`.

### <a name="example"></a>Exemple

```
GET /me?api-version=2013-11-08 HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Assertions SAML obtenues avec un flux OBO OAuth 2.0

Certains services web basés sur OAuth doivent accéder à d’autres API de service web qui acceptent les instructions d’assertion SAML dans des flux non interactifs. Azure Active Directory peut fournir une instruction d’assertion SAML en réponse à un flux On-Behalf-Of qui utilise un service web basé sur SAML comme ressource cible.

>[!NOTE]
>Il s’agit d’une extension non standard pour le flux On-Behalf-Of OAuth 2.0 qui permet à une application OAuth2 d’accéder aux points de terminaison d’API du service web qui consomment des jetons SAML.

> [!TIP]
> Quand vous appelez un service web protégé par SAML à partir d’une application web front-end, vous pouvez simplement appeler l’API et lancer un flux d’authentification interactif normal avec la session existante de l’utilisateur. Vous devez seulement utiliser un flux OBO quand un appel de service à service nécessite un jeton SAML pour fournir le contexte de l’utilisateur.

### <a name="obtain-a-saml-token-by-using-an-obo-request-with-a-shared-secret"></a>Obtenir un jeton SAML en utilisant une demande OBO avec un secret partagé

Une demande de service à service pour obtenir une assertion SAML contient les paramètres suivants :

| Paramètre | Type | Description |
| --- | --- | --- |
| grant_type |Obligatoire | Type de la demande de jeton. Pour une demande qui utilise un JWT, la valeur doit être **urn:ietf:params:oauth:grant-type:jwt-bearer**. |
| assertion |Obligatoire | Valeur du jeton d’accès utilisé dans la requête.|
| client_id |Obligatoire | ID d’application affecté au service appelant lors de l’inscription auprès d’Azure AD. Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, choisissez l’annuaire, puis sélectionnez le nom de l’application. |
| client_secret |Obligatoire | Clé enregistrée pour le service appelant dans Azure AD. Vous devez avoir noté cette valeur au moment de l’inscription. |
| resource |Obligatoire | URI de l’ID d’application du service de destination (ressource sécurisée). Il s’agit de la ressource qui sera l’audience du jeton SAML. Pour rechercher l’ID d’application dans le portail Azure, sélectionnez **Active Directory**, puis choisissez l’annuaire. Sélectionnez le nom de l’application, choisissez **Tous les paramètres**, puis sélectionnez **Propriétés**. |
| requested_token_use |Obligatoire | Spécifie comment la demande doit être traitée. Dans le flux Pour le compte de, la valeur doit être **on_behalf_of**. |
| requested_token_type | Obligatoire | Spécifie le type de jeton demandé. La valeur peut être **urn:ietf:params:oauth:token-type:saml2** ou **urn:ietf:params:oauth:token-type:saml1**, en fonction des exigences de la ressource. |

La réponse contient un jeton SAML encodé en UTF8 et Base64url.

- **SubjectConfirmationData pour une assertion SAML provenant d’un appel OBO** : si l’application cible nécessite une valeur de destinataire dans **SubjectConfirmationData**, la valeur doit être une URL de réponse sans caractère générique dans la configuration de la ressource d’application.
- **Le nœud SubjectConfirmationData** : le nœud ne peut pas contenir d’attribut **InResponseTo**, dans la mesure où il ne fait pas partie d’une réponse SAML. L’application qui reçoit le jeton SAML doit pouvoir accepter l’assertion SAML sans attribut **InResponseTo**.

- **Consentement** : un consentement doit avoir été accordé pour recevoir un jeton SAML contenant des données utilisateur sur un flux OAuth. Pour plus d’informations sur les autorisations et sur l’obtention d’un consentement de l’administrateur, consultez [Autorisations et consentement dans le point de terminaison Azure Active Directory v1.0](./v1-permissions-consent.md).

### <a name="response-with-saml-assertion"></a>Réponse avec instruction d’assertion SAML

| Paramètre | Description |
| --- | --- |
| token_type |Indique la valeur du type de jeton. Le seul type de jeton pris en charge par Azure AD est le **jeton porteur**. Pour plus d’informations sur les jetons du porteur, consultez le [Framework d’autorisation OAuth 2.0 : Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt). |
| scope |Étendue de l’accès accordé dans le jeton. |
| expires_in |Durée de validité du jeton d’accès (en secondes). |
| expires_on |L’heure d’expiration du jeton d’accès. La date est représentée en nombre de secondes à partir du 1er janvier 1970 (1970-01-01T0:0:0Z) UTC jusqu’au moment de l’expiration. Cette valeur est utilisée pour déterminer la durée de vie des jetons en cache. |
| resource |URI de l’ID d’application du service de destination (ressource sécurisée). |
| access_token |Paramètre qui retourne l’assertion SAML. |
| refresh_token |Le jeton d’actualisation. Le service appelant peut utiliser ce jeton pour demander un autre jeton d’accès après l’expiration de l’instruction d’assertion SAML actuelle. |

- token_type : Support
- expires_in : 3296
- ext_expires_in : 0
- expires_on : 1529627844
- resource : `https://api.contoso.com`
- access_token : \<SAML assertion\>
- issued_token_type : urn:ietf:params:oauth:token-type:saml2
- refresh_token : \<Refresh token\>

## <a name="client-limitations"></a>Limitations du client

Les clients publics ayant des URL de réponse avec des caractères génériques ne peuvent pas utiliser un `id_token` pour les flux OBO. Cependant, un client confidentiel peut toujours échanger les jetons d’**accès** obtenus via le flux d’octroi implicite, même si le client public a un URI de redirection avec des caractères génériques inscrit.

## <a name="next-steps"></a>Étapes suivantes

Découvrez plus d’informations sur le protocole OAuth 2.0 et une autre méthode pour effectuer l’authentification de service à service utilisant les informations d’identification du client :

* [Authentification de service à service utilisant l’octroi d’informations d’identification du client OAuth 2.0 dans Azure AD](v1-oauth2-client-creds-grant-flow.md)
* [OAuth 2.0 dans Azure AD](v1-protocols-oauth-code.md)