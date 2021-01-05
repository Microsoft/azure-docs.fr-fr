---
title: Ressources du modèle Gestion des API Azure | Microsoft Docs
description: Découvrez les types de ressources utilisables dans les modèles du portail des développeurs dans la Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 51a1b4c6-a9fd-4524-9e0e-03a9800c3e94
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: c3b6123c63bf530463379a175745ef86baf2c5a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84689985"
---
# <a name="azure-api-management-template-resources"></a>Ressources du modèle Gestion des API Azure
La Gestion des API Azure fournit les types de ressources suivants, utilisables dans les modèles du portail des développeurs.  
  
-   [Ressources de chaînes](#strings)  
  
-   [Ressources de glyphes](#glyphs)  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
##  <a name="string-resources"></a><a name="strings"></a> Ressources de chaînes  
 La Gestion des API fournit un ensemble complet de ressources de chaînes utilisables dans le portail des développeurs. Ces ressources sont traduites dans toutes les langues prises en charge par la Gestion des API. L’ensemble de modèles par défaut utilise ces ressources pour les étiquettes, les en-têtes de pages et les éventuelles chaînes constantes affichées dans le portail des développeurs. Pour utiliser une ressource de chaîne dans vos modèles, fournir le préfixe de la chaîne de ressource suivi du nom de la chaîne, comme le montre l’exemple suivant.  
  
```  
{% localized "Prefix|Name" %}  
  
```  
  
 L’exemple suivant est issu du modèle Liste de produits et affiche **Produits** en haut de la page.  
  
```  
<h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
  
```  
  
Les options de localisation suivantes sont prises en charge :

| Paramètres régionaux    | Langage               |
|-----------|------------------------|
| « en »      | « English »              |
| « cs »      | « Čeština »              |
| « de »      | « Deutsch »              |
| « es »      | « Español »              |
| « fr »      | « Français »             |
| « hu »      | « Magyar »               |
| « it »      | « Italiano »             |
| « ja-JP »   | « 日本語 »                |
| « ko »      | « 한국어 »                |
| « nl »      | « Nederlands »           |
| « pl »      | « Polski »               |
| « pt-br »   | « Português (Brasil) »   |
| « pt-pt »   | « Português (Portugal) » |
| « ru »      | « Русский »              |
| « sv »      | « Svenska »              |
| « tr »      | « Türkçe »               |
| « zh-hans » | « 中文(简体) »           |
| « zh-hant » | « 中文(繁體) »           |

 Reportez-vous aux tableaux suivants pour connaître les ressources de chaînes utilisables dans vos modèles sur le portail des développeurs. Utilisez le nom de la table comme préfixe des ressources de chaînes dans ce tableau.  
  
-   [ApisStrings](#ApisStrings)  
  
-   [ApplicationListStrings](#ApplicationListStrings)  
  
-   [AppDetailsStrings](#AppDetailsStrings)  
  
-   [AppStrings](#AppStrings)  
  
-   [CommonResources](#CommonResources)  
  
-   [CommonStrings](#CommonStrings)  
  
-   [Documentation](#Documentation)  
  
-   [ErrorPageStrings](#ErrorPageStrings)  
  
-   [IssuesStrings](#IssuesStrings)  
  
-   [NotFoundStrings](#NotFoundStrings)  
  
-   [ProductDetailsStrings](#ProductDetailsStrings)  
  
-   [ProductsStrings](#ProductsStrings)  
  
-   [ProviderInfoStrings](#ProviderInfoStrings)  
  
-   [SigninResources](#SigninResources)  
  
-   [SigninStrings](#SigninStrings)  
  
-   [SignupStrings](#SignupStrings)  
  
-   [SubscriptionListStrings](#SubscriptionListStrings)  
  
-   [SubscriptionStrings](#SubscriptionStrings)  
  
-   [UpdateProfileStrings](#UpdateProfileStrings)  
  
-   [UserProfile](#UserProfile)  
  
###  <a name="apisstrings"></a><a name="ApisStrings"></a> ApisStrings  
  
|Name|Texte|  
|----------|----------|  
|PageTitleApis|API|  
  
###  <a name="appdetailsstrings"></a><a name="AppDetailsStrings"></a> AppDetailsStrings  
  
|Name|Texte|  
|----------|----------|  
|WebApplicationsDetailsTitle|Aperçu de l’application.|  
|WebApplicationsRequirementsHeader|Spécifications|  
|WebApplicationsScreenshotAlt|Capture d'écran|  
|WebApplicationsScreenshotsHeader|Captures d’écran.|  
  
###  <a name="applicationliststrings"></a><a name="ApplicationListStrings"></a> ApplicationListStrings  
  
|Name|Texte|  
|----------|----------|  
|WebDevelopersAppDeleteConfirmation|Voulez-vous vraiment supprimer l’application ?|  
|WebDevelopersAppNotPublished|Non publié.|  
|WebDevelopersAppNotSubmitted|Non soumis.|  
|WebDevelopersAppTableCategoryHeader|Category|  
|WebDevelopersAppTableNameHeader|Name|  
|WebDevelopersAppTableStateHeader|State|  
|WebDevelopersEditLink|Modifier|  
|WebDevelopersRegisterAppLink|Inscription de l’application|  
|WebDevelopersRemoveLink|Supprimer|  
|WebDevelopersSubmitLink|Submit|  
|WebDevelopersYourApplicationsHeader|Vos applications|  
  
###  <a name="appstrings"></a><a name="AppStrings"></a> AppStrings  
  
|Name|Texte|  
|----------|----------|  
|WebApplicationsHeader|Applications|  
  
###  <a name="commonresources"></a><a name="CommonResources"></a> CommonResources  
  
|Name|Texte|  
|----------|----------|  
|NoItemsToDisplay|Aucun résultat trouvé.|  
|GeneralExceptionMessage|Il y a un problème. Il peut s’agir d’un dysfonctionnement temporaire ou d’un bogue. Réessayez.|  
|GeneralJsonExceptionMessage|Il y a un problème. Il peut s’agir d’un dysfonctionnement temporaire ou d’un bogue. Rechargez la page et réessayez.|  
|ConfirmationMessageUnsavedChanges|Certaines modifications n’ont pas été enregistrées. Voulez-vous vraiment annuler et ignorer les modifications ?|  
|AzureActiveDirectory|Azure Active Directory|  
|HttpLargeRequestMessage|Le texte de la requête HTTP est trop long.|  
  
###  <a name="commonstrings"></a><a name="CommonStrings"></a> CommonStrings  
  
|Name|Texte|  
|----------|----------|  
|ButtonLabelCancel|Annuler|  
|ButtonLabelSave|Enregistrer|  
|GeneralExceptionMessage|Il y a un problème. Il peut s’agir d’un dysfonctionnement temporaire ou d’un bogue. Réessayez.|  
|NoItemsToDisplay|Il n’y a aucun élément à afficher.|  
|PagerButtonLabelFirst|Premier|  
|PagerButtonLabelLast|Dernier|  
|PagerButtonLabelNext|Suivant|  
|PagerButtonLabelPrevious|Précédent|  
|PagerLabelPageNOfM|Page {0} sur {1}|  
|PasswordTooShort|Le mot de passe est trop court.|  
|EmailAsPassword|N’utilisez pas votre adresse de messagerie comme mot de passe.|  
|PasswordSameAsUserName|Votre mot de passe ne peut pas contenir votre nom d’utilisateur.|  
|PasswordTwoCharacterClasses|Utilisez différentes classes de caractères.|  
|PasswordTooManyRepetitions|Trop de répétitions.|  
|PasswordSequenceFound|Votre mot de passe contient des séquences.|  
|PagerLabelPageSize|Taille de la page|  
|CurtainLabelLoading|Chargement en cours...|  
|TablePlaceholderNothingToDisplay|Aucune donnée pour la période et l’étendue sélectionnées.|  
|ButtonLabelClose|fermez|  
  
###  <a name="documentation"></a><a name="Documentation"></a> Documentation  
  
|Name|Texte|  
|----------|----------|  
|WebDocumentationInvalidHeaderErrorMessage|En-tête non valide « {0} »|  
|WebDocumentationInvalidRequestErrorMessage|URL de demande non valide.|  
|TextboxLabelAccessToken|Jeton d’accès *|  
|DropdownOptionPrimaryKeyFormat|Primary-{0}|  
|DropdownOptionSecondaryKeyFormat|Secondary-{0}|  
|WebDocumentationSubscriptionKeyText|Votre clé d’abonnement|  
|WebDocumentationTemplatesAddHeaders|Ajoutez les en-têtes HTTP requis.|  
|WebDocumentationTemplatesBasicAuthSample|Exemple d’autorisation de base|  
|WebDocumentationTemplatesCurlForBasicAuth|Pour une autorisation de base, utilisez : --user {nom_utilisateur}:{mot_de_passe}.|  
|WebDocumentationTemplatesCurlValuesForPath|Fournissez des valeurs aux paramètres de chemin d’accès (indiqués entre {...}), à votre clé d’abonnement et aux paramètres de requête.|  
|WebDocumentationTemplatesDeveloperKey|Spécifiez votre clé d’abonnement.|  
|WebDocumentationTemplatesJavaApache|Cet exemple utilise le client HTTP Apache à partir de composants HTTP (http://hc.apache.org/httpcomponents-client-ga/)|  
|WebDocumentationTemplatesOptionalParams|Fournissez des valeurs aux paramètres facultatifs, en fonction des besoins.|  
|WebDocumentationTemplatesPhpPackage|Cet exemple utilise le package HTTP_Request2. (pour plus d’informations : https://pear.php.net/package/HTTP_Request2)|  
|WebDocumentationTemplatesPythonValuesForPath|Fournissez des valeurs aux paramètres de chemin d’accès (indiqués entre {...}) et au corps de la demande si nécessaire.|  
|WebDocumentationTemplatesRequestBody|Spécifiez le corps de la demande.|  
|WebDocumentationTemplatesRequiredParams|Donnez des valeurs aux paramètres obligatoires suivants.|  
|WebDocumentationTemplatesValuesForPath|Donnez des valeurs aux paramètres de chemin d’accès (indiqués entre {...}).|  
|OAuth2AuthorizationEndpointDescription|Le point de terminaison d’autorisation est utilisé pour interagir avec le propriétaire de la ressource et obtenir une autorisation.|  
|OAuth2AuthorizationEndpointName|Point de terminaison d’autorisation|  
|OAuth2TokenEndpointDescription|Le point de terminaison de jeton est utilisé par le client pour obtenir un jeton d’accès en présentant son autorisation ou son jeton d’actualisation.|  
|OAuth2TokenEndpointName|Point de terminaison de jeton|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Description|<p\>         Le client lance le flux en dirigeant l’agent utilisateur du propriétaire de la ressource vers le point de terminaison d’autorisation.  Le client inclut son identificateur client, l’étendue demandée, l’état local et un URI de redirection auquel le serveur d’autorisation renverra l’agent utilisateur une fois l’accès accordé (ou refusé).     </p\>     <p\>         Le serveur d’autorisation authentifie le propriétaire de la ressource (par l’intermédiaire de l’agent utilisateur) et détermine si le propriétaire de la ressource accorde ou refuse la demande d’accès du client.     </p\>     <p\>         En supposant que le propriétaire de la ressource accorde l’accès, le serveur d’autorisation redirige l’agent utilisateur vers le client à l’aide de l’URI de redirection fourni précédemment (dans la demande ou lors de l’inscription du client).  L’URI de redirection comprend un code d’autorisation et les éventuels états locaux fournis précédemment par le client.     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ErrorDescription|<p\>     Si l’utilisateur refuse la demande d’accès ou que la demande n’est pas valide, le client sera informé en ajoutant les paramètres suivants à la redirection : </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_Name|Demande d’autorisation.|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_RequestDescription|<p\>         L’application cliente doit envoyer l’utilisateur vers le point de terminaison d’autorisation afin de lancer le processus OAuth.          Au point de terminaison d’autorisation, l’utilisateur s’authentifie, puis accorde ou refuse l’accès à l’application.     </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_AuthorizationRequest_ResponseDescription|<p\>     En supposant que le propriétaire de la ressource accorde l’accès, le serveur d’autorisation redirige l’agent utilisateur vers le client à l’aide de l’URI de redirection fourni précédemment (dans la demande ou lors de l’inscription du client).  L’URI de redirection comprend un code d’autorisation et les éventuels états locaux fournis précédemment par le client. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_Description|<p\>  Le client demande un jeton d’accès au point de terminaison du jeton du serveur d’autorisation en incluant le code d’autorisation reçu à l’étape précédente.  Lors de la demande, le client s’authentifie auprès du serveur d’autorisation.  Le client inclut l’URI de redirection utilisé pour obtenir le code d’autorisation à des fins de vérification. </p\> <p\>     Le serveur d’autorisation authentifie le client, valide le code d’autorisation et vérifie que l’URI de redirection reçu correspond à l’URI utilisé pour rediriger le client à l’étape (C).  S’il est valide, le serveur d’autorisation répond en envoyant un jeton d’accès et, éventuellement, un jeton d’actualisation. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ErrorDescription|<p\>     Si la demande d’authentification du client échoue ou n’est pas valide, le serveur d’autorisation répond en envoyant le code d’état HTTP 400 (demande incorrecte) (sauf indication contraire) et ajoute les paramètres suivants à la réponse. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_RequestDescription|<p\>   Le client effectue une demande auprès du point de terminaison de jeton en envoyant les paramètres suivants au format « application/x-www-forme-urlencodee » avec l’encodage de caractères UTF-8 dans le corps d’entité de la requête HTTP. </p\>|  
|OAuth2Flow_AuthorizationCodeGrant_Step_TokenRequest_ResponseDescription|<p\>  Le serveur d’autorisation émet un jeton d’accès et un jeton d’actualisation facultatif et construit la réponse en ajoutant les paramètres suivants au corps d’entité de la réponse HTTP avec un code d’état 200 (OK). </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_Description|<p\>  Le client s’authentifie auprès du serveur d’autorisation et demande un jeton d’accès au point de terminaison de jeton. </p\> <p\>  Le serveur d’autorisation authentifie le client et, s’il est valide, émet un jeton d’accès. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\>     Si la demande d’authentification du client échoue ou n’est pas valide, le serveur d’autorisation répond en envoyant le code d’état HTTP 400 (demande incorrecte) (sauf indication contraire) et ajoute les paramètres suivants à la réponse. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_RequestDescription|<p\>   Le client effectue une demande auprès du point de terminaison de jeton en ajoutant les paramètres suivants au format « application/x-www-forme-urlencodee » avec l’encodage de caractères UTF-8 dans le corps d’entité de la requête HTTP. </p\>|  
|OAuth2Flow_ClientCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\>  Si la demande de jeton d’accès est valide et autorisée, le serveur d’autorisation émet un jeton d’accès et un jeton d’actualisation facultatif et construit la réponse en ajoutant les paramètres suivants au corps d’entité de la réponse HTTP avec un code d’état 200 (OK). </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_Description|<p\>         Le client lance le flux en dirigeant l’agent utilisateur du propriétaire de la ressource vers le point de terminaison d’autorisation.  Le client inclut son identificateur client, l’étendue demandée, l’état local et un URI de redirection auquel le serveur d’autorisation renverra l’agent utilisateur une fois l’accès accordé (ou refusé). </p\>     <p\>         Le serveur d’autorisation authentifie le propriétaire de la ressource (par l’intermédiaire de l’agent utilisateur) et détermine si le propriétaire de la ressource accorde ou refuse la demande d’accès du client. </p\>     <p\>         En supposant que le propriétaire de la ressource accorde l’accès, le serveur d’autorisation redirige l’agent utilisateur vers le client à l’aide de l’URI de redirection fourni précédemment.  L’URI de redirection inclut le jeton d’accès dans le fragment d’URI. </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ErrorDescription|<p\>     Si le propriétaire de la ressource refuse la demande d’accès ou que la demande échoue pour des raisons autres qu’un URI de redirection manquant ou non valide, le serveur d’autorisation en informe le client en ajoutant les paramètres suivants au composant de fragment de l’URI de redirection au format « application/x-www-forme-urlencodee ». </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_RequestDescription|<p\>         L’application cliente doit envoyer l’utilisateur vers le point de terminaison d’autorisation afin de lancer le processus OAuth.      Au point de terminaison d’autorisation, l’utilisateur s’authentifie, puis accorde ou refuse l’accès à l’application. </p\>|  
|OAuth2Flow_ImplicitGrant_Step_AuthorizationRequest_ResponseDescription|<p\>     Si le propriétaire de la ressource accorde la demande d’accès, le serveur d’autorisation émet un jeton d’accès et le remet au client en ajoutant les paramètres suivants au composant de fragment de l’URI de redirection au format « application/x-www-forme-urlencodee ». </p\>|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Description|Le flux de code d’autorisation est optimisé pour les clients capables de préserver la confidentialité de leurs identifiants (par exemple, les applications de serveur web implémentées en PHP, Java, Python, Ruby, ASP.NET, etc.).|  
|OAuth2Flow_ObtainAuthorization_AuthorizationCodeGrant_Name|Octroi du code d’autorisation.|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Description|Le flux d’informations d’identification du client est approprié dans les cas où le client (votre application) demande l’accès aux ressources protégées sous son contrôle. Le client est considéré comme propriétaire de la ressource, par conséquent aucune intervention de l’utilisateur final n’est requise.|  
|OAuth2Flow_ObtainAuthorization_ClientCredentialsGrant_Name|Octroi des identifiants du client.|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Description|Le flux implicite est optimisé pour les clients dans l’incapacité de garantir la confidentialité de leurs identifiants connus pour exploiter un URI de redirection en particulier. Ces clients sont généralement implémentés dans un navigateur avec un langage de script comme JavaScript.|  
|OAuth2Flow_ObtainAuthorization_ImplicitGrant_Name|Octroi implicite.|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Description|Le flux d’informations de mot de passe du propriétaire de la ressource est approprié dans les cas où le propriétaire de la ressource a une relation de confiance avec le client (votre application), par exemple le système d’exploitation de l’appareil ou une application à privilèges élevés. Ce flux est adapté aux clients capables d’obtenir les identifiants du propriétaire de la ressource (nom d’utilisateur et mot de passe, généralement à l’aide d’un formulaire interactif).|  
|OAuth2Flow_ObtainAuthorization_ResourceOwnerPasswordCredentialsGrant_Name|Octroi des informations de mot de passe du propriétaire de la ressource.|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_Description|<p\>   Le propriétaire de la ressource fournit au client son nom d’utilisateur et son mot de passe. </p\> <p\> Le client demande un jeton d’accès au point de terminaison du jeton du serveur d’autorisation en incluant les identifiants reçus de la part du propriétaire de la ressource.  Lors de la demande, le client s’authentifie auprès du serveur d’autorisation. </p\> <p\>  Le serveur d’autorisation authentifie le client et valide les identifiants du propriétaire de la ressource ; s’ils sont valides, il émet un jeton d’accès. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ErrorDescription|<p\>     Si la demande d’authentification du client échoue ou n’est pas valide, le serveur d’autorisation répond en envoyant le code d’état HTTP 400 (demande incorrecte) (sauf indication contraire) et ajoute les paramètres suivants à la réponse. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_RequestDescription|<p\>   Le client effectue une demande auprès du point de terminaison de jeton en ajoutant les paramètres suivants au format « application/x-www-forme-urlencodee » avec l’encodage de caractères UTF-8 dans le corps d’entité de la requête HTTP. </p\>|  
|OAuth2Flow_ResourceOwnerPasswordCredentialsGrant_Step_TokenRequest_ResponseDescription|<p\>  Si la demande de jeton d’accès est valide et autorisée, le serveur d’autorisation émet un jeton d’accès et un jeton d’actualisation facultatif et construit la réponse en ajoutant les paramètres suivants au corps d’entité de la réponse HTTP avec un code d’état 200 (OK). </p\>|  
|OAuth2Step_AccessTokenRequest_Name|Demande de jeton d’accès.|  
|OAuth2Step_AuthorizationRequest_Name|Demande d’autorisation.|  
|OAuth2AccessToken_AuthorizationCodeGrant_TokenResponse|OBLIGATOIRE. Jeton d’accès émis par le serveur d’autorisation.|  
|OAuth2AccessToken_ClientCredentialsGrant_TokenResponse|OBLIGATOIRE. Jeton d’accès émis par le serveur d’autorisation.|  
|OAuth2AccessToken_ImplicitGrant_AuthorizationResponse|OBLIGATOIRE. Jeton d’accès émis par le serveur d’autorisation.|  
|OAuth2AccessToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|OBLIGATOIRE. Jeton d’accès émis par le serveur d’autorisation.|  
|OAuth2ClientId_AuthorizationCodeGrant_AuthorizationRequest|OBLIGATOIRE. Identificateur client.|  
|OAuth2ClientId_AuthorizationCodeGrant_TokenRequest|OBLIGATOIRE si le client ne s’authentifie pas auprès du serveur d’autorisation.|  
|OAuth2ClientId_ImplicitGrant_AuthorizationRequest|OBLIGATOIRE. Identificateur client.|  
|OAuth2Code_AuthorizationCodeGrant_AuthorizationResponse|OBLIGATOIRE. Code d’autorisation généré par le serveur d’autorisation.|  
|OAuth2Code_AuthorizationCodeGrant_TokenRequest|OBLIGATOIRE. Code d’autorisation reçu de la part du serveur d’autorisation.|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_AuthorizationErrorResponse|FACULTATIF. Texte ASCII lisible fournissant des informations supplémentaires.|  
|OAuth2ErrorDescription_AuthorizationCodeGrant_TokenErrorResponse|FACULTATIF. Texte ASCII lisible fournissant des informations supplémentaires.|  
|OAuth2ErrorDescription_ClientCredentialsGrant_TokenErrorResponse|FACULTATIF. Texte ASCII lisible fournissant des informations supplémentaires.|  
|OAuth2ErrorDescription_ImplicitGrant_AuthorizationErrorResponse|FACULTATIF. Texte ASCII lisible fournissant des informations supplémentaires.|  
|OAuth2ErrorDescription_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|FACULTATIF. Texte ASCII lisible fournissant des informations supplémentaires.|  
|OAuth2ErrorUri_AuthorizationCodeGrant_AuthorizationErrorResponse|FACULTATIF. URI qui identifie une page web lisible avec des informations sur l’erreur.|  
|OAuth2ErrorUri_AuthorizationCodeGrant_TokenErrorResponse|FACULTATIF. URI qui identifie une page web lisible avec des informations sur l’erreur.|  
|OAuth2ErrorUri_ClientCredentialsGrant_TokenErrorResponse|FACULTATIF. URI qui identifie une page web lisible avec des informations sur l’erreur.|  
|OAuth2ErrorUri_ImplicitGrant_AuthorizationErrorResponse|FACULTATIF. URI qui identifie une page web lisible avec des informations sur l’erreur.|  
|OAuth2ErrorUri_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|FACULTATIF. URI qui identifie une page web lisible avec des informations sur l’erreur.|  
|OAuth2Error_AuthorizationCodeGrant_AuthorizationErrorResponse|OBLIGATOIRE. Code d’erreur ASCII unique de la liste suivante : invalid_request, unauthorized_client, access_denied, unsupported_response_type, invalid_scope, server_error, temporarily_unavailable.|  
|OAuth2Error_AuthorizationCodeGrant_TokenErrorResponse|OBLIGATOIRE. Code d’erreur ASCII unique de la liste suivante : invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2Error_ClientCredentialsGrant_TokenErrorResponse|OBLIGATOIRE. Code d’erreur ASCII unique de la liste suivante : invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2Error_ImplicitGrant_AuthorizationErrorResponse|OBLIGATOIRE. Code d’erreur ASCII unique de la liste suivante : invalid_request, unauthorized_client, access_denied, unsupported_response_type, invalid_scope, server_error, temporarily_unavailable.|  
|OAuth2Error_ResourceOwnerPasswordCredentialsGrant_TokenErrorResponse|OBLIGATOIRE. Code d’erreur ASCII unique de la liste suivante : invalid_request, invalid_client, invalid_grant, unauthorized_client, unsupported_grant_type, invalid_scope.|  
|OAuth2ExpiresIn_AuthorizationCodeGrant_TokenResponse|RECOMMANDÉ. Durée de vie en secondes du jeton d’accès.|  
|OAuth2ExpiresIn_ClientCredentialsGrant_TokenResponse|RECOMMANDÉ. Durée de vie en secondes du jeton d’accès.|  
|OAuth2ExpiresIn_ImplicitGrant_AuthorizationResponse|RECOMMANDÉ. Durée de vie en secondes du jeton d’accès.|  
|OAuth2ExpiresIn_ResourceOwnerPasswordCredentialsGrant_TokenResponse|RECOMMANDÉ. Durée de vie en secondes du jeton d’accès.|  
|OAuth2GrantType_AuthorizationCodeGrant_TokenRequest|OBLIGATOIRE. DOIT avoir la valeur « authorization_code ».|  
|OAuth2GrantType_ClientCredentialsGrant_TokenRequest|OBLIGATOIRE. DOIT avoir la valeur « client_credentials ».|  
|OAuth2GrantType_ResourceOwnerPasswordCredentialsGrant_TokenRequest|OBLIGATOIRE. DOIT avoir la valeur « password ».|  
|OAuth2Password_ResourceOwnerPasswordCredentialsGrant_TokenRequest|OBLIGATOIRE. Mot de passe de propriétaire de la ressource.|  
|OAuth2RedirectUri_AuthorizationCodeGrant_AuthorizationRequest|FACULTATIF. L’URI du point de terminaison de redirection doit être un URI absolu.|  
|OAuth2RedirectUri_AuthorizationCodeGrant_TokenRequest|OBLIGATOIRE si le paramètre « redirect_uri » a été inclus dans la demande d’autorisation ; leurs valeurs DOIVENT être identiques.|  
|OAuth2RedirectUri_ImplicitGrant_AuthorizationRequest|FACULTATIF. L’URI du point de terminaison de redirection doit être un URI absolu.|  
|OAuth2RefreshToken_AuthorizationCodeGrant_TokenResponse|FACULTATIF. Jeton d’actualisation, qui peut être utilisé pour obtenir de nouveaux jetons d’accès.|  
|OAuth2RefreshToken_ClientCredentialsGrant_TokenResponse|FACULTATIF. Jeton d’actualisation, qui peut être utilisé pour obtenir de nouveaux jetons d’accès.|  
|OAuth2RefreshToken_ResourceOwnerPasswordCredentialsGrant_TokenResponse|FACULTATIF. Jeton d’actualisation, qui peut être utilisé pour obtenir de nouveaux jetons d’accès.|  
|OAuth2ResponseType_AuthorizationCodeGrant_AuthorizationRequest|OBLIGATOIRE. DOIT avoir la valeur « code ».|  
|OAuth2ResponseType_ImplicitGrant_AuthorizationRequest|OBLIGATOIRE. DOIT avoir la valeur « token ».|  
|OAuth2Scope_AuthorizationCodeGrant_AuthorizationRequest|FACULTATIF. Étendue de la demande d’accès.|  
|OAuth2Scope_AuthorizationCodeGrant_TokenResponse|FACULTATIF si identique à l’étendue demandée par le client ; sinon, obligatoire.|  
|OAuth2Scope_ClientCredentialsGrant_TokenRequest|FACULTATIF. Étendue de la demande d’accès.|  
|OAuth2Scope_ClientCredentialsGrant_TokenResponse|FACULTATIF si identique à l’étendue demandée par le client ; sinon, obligatoire.|  
|OAuth2Scope_ImplicitGrant_AuthorizationRequest|FACULTATIF. Étendue de la demande d’accès.|  
|OAuth2Scope_ImplicitGrant_AuthorizationResponse|FACULTATIF si identique à l’étendue demandée par le client ; sinon, obligatoire.|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenRequest|FACULTATIF. Étendue de la demande d’accès.|  
|OAuth2Scope_ResourceOwnerPasswordCredentialsGrant_TokenResponse|FACULTATIF si identique à l’étendue demandée par le client ; sinon, obligatoire.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationErrorResponse|OBLIGATOIRE si le paramètre « state » était présent dans la demande d’autorisation du client.  Valeur exacte reçue de la part du client.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationRequest|RECOMMANDÉ. Valeur opaque utilisée par le client pour maintenir l’état entre la demande et le rappel.  Le serveur d’autorisation inclut cette valeur lors de la redirection de l’agent utilisateur vers le client.  Le paramètre DOIT DE PRÉFÉRENCE être utilisé pour empêcher la falsification de demandes intersites.|  
|OAuth2State_AuthorizationCodeGrant_AuthorizationResponse|OBLIGATOIRE si le paramètre « state » était présent dans la demande d’autorisation du client.  Valeur exacte reçue de la part du client.|  
|OAuth2State_ImplicitGrant_AuthorizationErrorResponse|OBLIGATOIRE si le paramètre « state » était présent dans la demande d’autorisation du client.  Valeur exacte reçue de la part du client.|  
|OAuth2State_ImplicitGrant_AuthorizationRequest|RECOMMANDÉ. Valeur opaque utilisée par le client pour maintenir l’état entre la demande et le rappel.  Le serveur d’autorisation inclut cette valeur lors de la redirection de l’agent utilisateur vers le client.  Le paramètre DOIT DE PRÉFÉRENCE être utilisé pour empêcher la falsification de demandes intersites.|  
|OAuth2State_ImplicitGrant_AuthorizationResponse|OBLIGATOIRE si le paramètre « state » était présent dans la demande d’autorisation du client.  Valeur exacte reçue de la part du client.|  
|OAuth2TokenType_AuthorizationCodeGrant_TokenResponse|OBLIGATOIRE. Type du jeton émis.|  
|OAuth2TokenType_ClientCredentialsGrant_TokenResponse|OBLIGATOIRE. Type du jeton émis.|  
|OAuth2TokenType_ImplicitGrant_AuthorizationResponse|OBLIGATOIRE. Type du jeton émis.|  
|OAuth2TokenType_ResourceOwnerPasswordCredentialsGrant_TokenResponse|OBLIGATOIRE. Type du jeton émis.|  
|OAuth2UserName_ResourceOwnerPasswordCredentialsGrant_TokenRequest|OBLIGATOIRE. Nom d’utilisateur du propriétaire de la ressource.|  
|OAuth2UnsupportedTokenType|Le type de jeton « {0} » n’est pas pris en charge.|  
|OAuth2InvalidState|Réponse non valide du serveur d’autorisation.|  
|OAuth2GrantType_AuthorizationCode|Code d’autorisation.|  
|OAuth2GrantType_Implicit|Implicite|  
|OAuth2GrantType_ClientCredentials|Informations d'identification du client|  
|OAuth2GrantType_ResourceOwnerPassword|Mot de passe de propriétaire de la ressource.|  
|WebDocumentation302Code|302 Trouvé.|  
|WebDocumentation400Code|400 (Demande incorrecte).|  
|OAuth2SendingMethod_AuthHeader|En-tête d’autorisation.|  
|OAuth2SendingMethod_QueryParam|Paramètre de requête.|  
|OAuth2AuthorizationServerGeneralException|Une erreur s’est produite pendant l’autorisation d’accès via {0}|  
|OAuth2AuthorizationServerCommunicationException|Impossible d’établir une connexion HTTP au serveur d’autorisation ou connexion fermée de façon inattendue.|  
|WebDocumentationOAuth2GeneralErrorMessage|Une erreur inattendue s'est produite.|  
|AuthorizationServerCommunicationException|Une exception de communication du serveur d’autorisation s’est produite. Contactez l’administrateur.|  
|TextblockSubscriptionKeyHeaderDescription|Clé d’abonnement qui fournit l’accès à cette API. Trouvée dans votre <a href='/developer'\>profil</a\>.|  
|TextblockOAuthHeaderDescription|Jeton d’accès OAuth 2.0 obtenu de <i\>{0}</i\>. Types d’autorisation pris en charge : <i\>{1}</i\>.|  
|TextblockContentTypeHeaderDescription|Type de média du corps envoyé à l’API.|  
|ErrorMessageApiNotAccessible|L’API que vous essayez d’appeler n’est pas accessible pour l’instant. Contactez l’éditeur de l’API <a href="/issues"\>ici</a\>.|  
|ErrorMessageApiTimedout|L’API que vous essayez d’appeler met plus de temps que d’habitude à obtenir une réponse. Contactez l’éditeur de l’API <a href="/issues"\>ici</a\>.|  
|BadRequestParameterExpected|« Le paramètre « {0} » est attendu »|  
|TooltipTextDoubleClickToSelectAll|Double-cliquez pour tout sélectionner.|  
|TooltipTextHideRevealSecret|Afficher/masquer|  
|ButtonLinkOpenConsole|Essayer|  
|SectionHeadingRequestBody|Corps de la demande|  
|SectionHeadingRequestParameters|Paramètres de la demande|  
|SectionHeadingRequestUrl|URL de la demande|  
|SectionHeadingResponse|response|  
|SectionHeadingRequestHeaders|En-têtes de requête|  
|FormLabelSubtextOptional|facultatif|  
|SectionHeadingCodeSamples|Exemples de code|  
|TextblockOpenidConnectHeaderDescription|Jeton d’ID OpenID Connect obtenu de <i\>{0}</i\>. Types d’autorisation pris en charge : <i\>{1}</i\>.|  
  
###  <a name="errorpagestrings"></a><a name="ErrorPageStrings"></a> ErrorPageStrings  
  
|Name|Texte|  
|----------|----------|  
|LinkLabelBack|Retour|  
|LinkLabelHomePage|page d'accueil|  
|LinkLabelSendUsEmail|Envoyez-nous un e-mail.|  
|PageTitleError|Désolé, un problème est survenu dans le traitement de la page demandée.|  
|TextblockPotentialCauseIntermittentIssue|Il peut s’agir d’un problème intermittent d’accès aux données qui a déjà disparu.|  
|TextblockPotentialCauseOldLink|Il est possible que le lien sur lequel vous avez cliqué soit ancien et ne pointe plus vers l’emplacement correct.|  
|TextblockPotentialCauseTechnicalProblem|Un problème technique a pu se produire de notre côté.|  
|TextblockPotentialSolutionRefresh|Essayez de réactualiser la page.|  
|TextblockPotentialSolutionStartOver|Recommencez à partir de notre {0}.|  
|TextblockPotentialSolutionTryAgain|Accédez à {0} et réessayez l’action effectuée.|  
|TextReportProblem|{0} décrivant la cause du problème et nous nous en occuperons dès que possible.|  
|TitlePotentialCause|Cause potentielle.|  
|TitlePotentialSolution|Il peut simplement s’agir d’un problème temporaire, quelques actions à tenter.|  
  
###  <a name="issuesstrings"></a><a name="IssuesStrings"></a> IssuesStrings  
  
|Name|Texte|  
|----------|----------|  
|WebIssuesIndexTitle|Problèmes|  
|WebIssuesNoActiveSubscriptions|Vous ne disposez d’aucun abonnement actif. Vous devez vous abonner à un produit pour signaler un problème.|  
|WebIssuesNotSignin|Vous n’êtes pas connecté. {0} pour signaler un problème ou poster un commentaire.|  
|WebIssuesReportIssueButton|Signaler un problème|  
|WebIssuesSignIn|se connecter|  
|WebIssuesStatusReportedBy|État: {0} &#124; Signalé par {1}|  
  
###  <a name="notfoundstrings"></a><a name="NotFoundStrings"></a> NotFoundStrings  
  
|Name|Texte|  
|----------|----------|  
|LinkLabelHomePage|page d'accueil|  
|LinkLabelSendUsEmail|Envoyez-nous un e-mail.|  
|PageTitleNotFound|Désolé, nous ne trouvons pas la page que vous recherchez.|  
|TextblockPotentialCauseMisspelledUrl|Vous avez peut-être mal orthographié l’URL si vous l’avez tapée.|  
|TextblockPotentialCauseOldLink|Il est possible que le lien sur lequel vous avez cliqué soit ancien et ne pointe plus vers l’emplacement correct.|  
|TextblockPotentialSolutionRetype|Essayez de retaper l’URL.|  
|TextblockPotentialSolutionStartOver|Recommencez à partir de notre {0}.|  
|TextReportProblem|{0} décrivant la cause du problème et nous nous en occuperons dès que possible.|  
|TitlePotentialCause|Cause potentielle.|  
|TitlePotentialSolution|Solution potentielle.|  
  
###  <a name="productdetailsstrings"></a><a name="ProductDetailsStrings"></a> ProductDetailsStrings  
  
|Name|Texte|  
|----------|----------|  
|WebProductsAgreement|En m’abonnant au produit {0}, j’accepte les `<a data-toggle='modal' href='#legal-terms'\>Terms of Use</a\>`.|  
|WebProductsLegalTermsLink|Conditions d’utilisation|  
|WebProductsSubscribeButton|S'abonner|  
|WebProductsUsageLimitsHeader|Limites d’utilisation|  
|WebProductsYouAreNotSubscribed|Vous êtes abonné à ce produit.|  
|WebProductsYouRequestedSubscription|Vous avez demandé l’abonnement à ce produit.|  
|ErrorYouNeedToAgreeWithLegalTerms|Vous devez accepter les conditions d’utilisation pour pouvoir continuer.|  
|ButtonLabelAddSubscription|Ajouter un abonnement|  
|LinkLabelChangeSubscriptionName|Modifier|  
|ButtonLabelConfirm|Confirmer|  
|TextblockMultipleSubscriptionsCount|Vous avez {0} abonnements à ce produit :|  
|TextblockSingleSubscriptionsCount|Vous avez {0} abonnement à ce produit :|  
|TextblockSingleApisCount|Ce produit contient l’API {0} :|  
|TextblockMultipleApisCount|Ce produit contient les API {0} :|  
|TextblockHeaderSubscribe|S’abonner au produit|  
|TextblockSubscriptionDescription|Un nouvel abonnement sera créé ainsi :|  
|TextblockSubscriptionLimitReached|Limite d’abonnements atteinte.|  
  
###  <a name="productsstrings"></a><a name="ProductsStrings"></a> ProductsStrings  
  
|Name|Texte|  
|----------|----------|  
|PageTitleProducts|Products|  
  
###  <a name="providerinfostrings"></a><a name="ProviderInfoStrings"></a> ProviderInfoStrings  
  
|Name|Texte|  
|----------|----------|  
|TextboxExternalIdentitiesDisabled|La connexion est désactivée par les administrateurs pour l’instant.|  
|TextboxExternalIdentitiesSigninInvitation|Vous pouvez également vous connecter avec|  
|TextboxExternalIdentitiesSigninInvitationPrimary|Connectez-vous avec :|  
  
###  <a name="signinresources"></a><a name="SigninResources"></a> SigninResources  
  
|Name|Texte|  
|----------|----------|  
|PrincipalNotFound|Principal introuvable ou signature non valide.|  
|ErrorSsoAuthenticationFailed|Échec de l’authentification SSO.|  
|ErrorSsoAuthenticationFailedDetailed|Jeton fourni non valide ou signature non vérifiable.|  
|ErrorSsoTokenInvalid|Jeton SSO non valide.|  
|ValidationErrorSpecificEmailAlreadyExists|E-mail « {0} » déjà inscrit|  
|ValidationErrorSpecificEmailInvalid|E-mail « {0} » non valide|  
|ValidationErrorPasswordInvalid|Le mot de passe n’est pas valide. Corrigez les erreurs, puis réessayez.|  
|PropertyTooShort|{0} est trop court|  
|WebAuthenticationAddresserEmailInvalidErrorMessage|Adresse de messagerie non valide.|  
|ValidationMessageNewPasswordConfirmationRequired|Confirmez le nouveau mot de passe.|  
|ValidationErrorPasswordConfirmationRequired|Le mot de passe de confirmation est vide.|  
|WebAuthenticationEmailChangeNotice|Un e-mail de confirmation du changement a été envoyé à {0}. Suivez les instructions qu’il contient pour confirmer votre nouvelle adresse de messagerie. Si le message n’est pas arrivé dans votre boîte de réception d’ici quelques minutes, vérifiez votre dossier Courrier indésirable.|  
|WebAuthenticationEmailChangeNoticeHeader|Votre demande de modification de l’adresse de messagerie a été traitée.|  
|WebAuthenticationEmailChangeNoticeTitle|Modification de l’adresse de messagerie demandée.|  
|WebAuthenticationEmailHasBeenRevertedNotice|Votre adresse de messagerie existe déjà. La demande a été annulée.|  
|ValidationErrorEmailAlreadyExists|L’adresse de messagerie existe déjà.|  
|ValidationErrorEmailInvalid|Adresse de messagerie non valide.|  
|TextboxLabelEmail|Email|  
|ValidationErrorEmailRequired|L’adresse de messagerie est obligatoire.|  
|WebAuthenticationErrorNoticeHeader|Error|  
|WebAuthenticationFieldLengthErrorMessage|{0} doit avoir une longueur maximale de {1}|  
|TextboxLabelEmailFirstName|Prénom|  
|ValidationErrorFirstNameRequired|Le prénom est obligatoire.|  
|ValidationErrorFirstNameInvalid|Prénom non valide.|  
|NoticeInvalidInvitationToken|Veuillez noter que les liens de confirmation ne sont valides que pendant 48 heures. Si vous vous trouvez toujours dans cette plage de temps, vérifiez que votre lien est correct. Si votre lien a expiré, renouvelez l’action que vous tentez de confirmer.|  
|NoticeHeaderInvalidInvitationToken|Jeton d’invitation non valide.|  
|NoticeTitleInvalidInvitationToken|Erreur de confirmation.|  
|WebAuthenticationLastNameInvalidErrorMessage|Nom non valide.|  
|TextboxLabelEmailLastName|Nom|  
|ValidationErrorLastNameRequired|Le nom est obligatoire.|  
|WebAuthenticationLinkExpiredNotice|Le lien de confirmation qui vous a été envoyé a expiré. `<a href={0}?token={1}>Resend confirmation email.</a\>`|  
|NoticePasswordResetLinkInvalidOrExpired|Votre lien de réinitialisation du mot de passe n’est pas valide ou a expiré.|  
|WebAuthenticationLinkExpiredNoticeTitle|Lien envoyé.|  
|WebAuthenticationNewPasswordLabel|Nouveau mot de passe|  
|ValidationMessageNewPasswordRequired|Un nouveau mot de passe est requis.|  
|TextboxLabelNotificationsSenderEmail|Adresse de messagerie de l’expéditeur des notifications|  
|TextboxLabelOrganizationName|Nom de l’organisation|  
|WebAuthenticationOrganizationRequiredErrorMessage|Le nom de l’organisation est vide.|  
|WebAuthenticationPasswordChangedNotice|Votre mot de passe a bien été mis à jour.|  
|WebAuthenticationPasswordChangedNoticeTitle|Mot de passe mis à jour.|  
|WebAuthenticationPasswordCompareErrorMessage|Les mots de passe ne correspondent pas.|  
|WebAuthenticationPasswordConfirmLabel|Confirmer le mot de passe|  
|ValidationErrorPasswordInvalidDetailed|Le mot de passe est trop faible.|  
|WebAuthenticationPasswordLabel|Mot de passe|  
|ValidationErrorPasswordRequired|Le mot de passe est requis.|  
|WebAuthenticationPasswordResetSendNotice|Un e-mail de confirmation du changement de mot de passe a été envoyé à {0}. Suivez les instructions décrites dans l’e-mail pour poursuivre la procédure de modification du mot de passe.|  
|WebAuthenticationPasswordResetSendNoticeHeader|Votre demande de réinitialisation de mot de passe a bien été traitée.|  
|WebAuthenticationPasswordResetSendNoticeTitle|Réinitialisation de mot de passe demandée.|  
|WebAuthenticationRequestNotFoundNotice|Demande introuvable.|  
|WebAuthenticationSenderEmailRequiredErrorMessage|L’adresse de messagerie de l’expéditeur des notifications est vide.|  
|WebAuthenticationSigninPasswordLabel|Confirmez la modification en entrant un mot de passe.|  
|WebAuthenticationSignupConfirmNotice|Un e-mail de confirmation d’inscription a été envoyé à {0}.<br /\> Suivez les instructions décrites dans l’e-mail pour activer votre compte.<br /\> Si le message n’arrive pas dans votre boîte de réception d’ici quelques minutes, vérifiez votre dossier Courrier indésirable.|  
|WebAuthenticationSignupConfirmNoticeHeader|Votre compte a bien été créé.|  
|WebAuthenticationSignupConfirmNoticeRepeatHeader|L’e-mail de confirmation d’inscription a été renvoyé.|  
|WebAuthenticationSignupConfirmNoticeTitle|Compte créé|  
|WebAuthenticationTokenRequiredErrorMessage|Le jeton est vide.|  
|WebAuthenticationUserAlreadyRegisteredNotice|Il semble qu’un utilisateur associé à cette adresse de messagerie soit déjà inscrit dans le système. Si vous avez oublié votre mot de passe, essayez de le restaurer ou contactez notre équipe de support.|  
|WebAuthenticationUserAlreadyRegisteredNoticeHeader|Utilisateur déjà inscrit.|  
|WebAuthenticationUserAlreadyRegisteredNoticeTitle|Déjà inscrit.|  
|ButtonLabelChangePassword|Modifier le mot de passe|  
|ButtonLabelChangeAccountInfo|Modifier les informations du compte|  
|ButtonLabelCloseAccount|Fermer le compte|  
|WebAuthenticationInvalidCaptchaErrorMessage|Le texte entré ne correspond pas au texte de l’image. Réessayez.|  
|ValidationErrorCredentialsInvalid|L’adresse de messagerie ou le mot de passe n’est pas valide. Corrigez les erreurs, puis réessayez.|  
|WebAuthenticationRequestIsNotValid|La demande n’est pas valide.|  
|WebAuthenticationUserIsNotConfirm|Veuillez confirmer votre inscription avant de tenter de vous connecter.|  
|WebAuthenticationInvalidEmailFormated|E-mail non valide : {0}|  
|WebAuthenticationUserNotFound|Utilisateur non trouvé.|  
|WebAuthenticationTenantNotRegistered|Votre compte appartient à un locataire Azure Active Directory qui n’est pas autorisé à accéder à ce portail.|  
|WebAuthenticationAuthenticationFailed|L’authentification a échoué.|  
|WebAuthenticationGooglePlusNotEnabled|L’authentification a échoué. Si vous avez autorisé l’application, contactez l’administrateur pour vérifier que l’authentification Google est configurée correctement.|  
|ValidationErrorAllowedTenantIsRequired|Le client autorisé est requis.|  
|ValidationErrorTenantIsNotValid|Le locataire Azure Active Directory « {0} » n’est pas valide.|  
|WebAuthenticationActiveDirectoryTitle|Azure Active Directory|  
|WebAuthenticationLoginUsingYourProvider|Connectez-vous à l’aide de votre compte {0}|  
|WebAuthenticationUserLimitNotice|Ce service a atteint le nombre maximal d’utilisateurs autorisés. Veuillez `<a href="mailto:{0}"\>contact the administrator</a\>` pour mettre à niveau leur service et réactiver l’inscription de l’utilisateur.|  
|WebAuthenticationUserLimitNoticeHeader|Inscription des utilisateurs désactivée.|  
|WebAuthenticationUserLimitNoticeTitle|Inscription des utilisateurs désactivée.|  
|WebAuthenticationUserRegistrationDisabledNotice|L’inscription des utilisateurs a été désactivée par l’administrateur. Connectez-vous avec un fournisseur d’identité externe.|  
|WebAuthenticationUserRegistrationDisabledNoticeHeader|Inscription des utilisateurs désactivée.|  
|WebAuthenticationUserRegistrationDisabledNoticeTitle|Inscription des utilisateurs désactivée.|  
|WebAuthenticationSignupPendingConfirmationNotice|Pour pouvoir terminer la création de votre compte, nous devons vérifier votre adresse de messagerie. Nous avons envoyé un e-mail à {0}. Suivez les instructions décrites dans l’e-mail pour activer votre compte. Si le message n’est pas arrivé d’ici quelques minutes, vérifiez votre dossier Courrier indésirable.|  
|WebAuthenticationSignupPendingConfirmationAccountFoundNotice|Nous avons trouvé un compte non confirmé associé à l’e-mail {0}. Pour pouvoir terminer la création de votre compte, nous devons vérifier votre adresse de messagerie. Nous avons envoyé un e-mail à {0}. Suivez les instructions décrites dans l’e-mail pour activer votre compte. Si le message n’est pas arrivé d’ici quelques minutes, vérifiez votre dossier Courrier indésirable.|  
|WebAuthenticationSignupConfirmationAlmostDone|C’est presque fini.|  
|WebAuthenticationSignupConfirmationEmailSent|Nous avons envoyé un e-mail à {0}. Suivez les instructions décrites dans l’e-mail pour activer votre compte. Si le message n’est pas arrivé d’ici quelques minutes, vérifiez votre dossier Courrier indésirable.|  
|WebAuthenticationEmailSentNotificationMessage|E-mail envoyé à {0}|  
|WebAuthenticationNoAadTenantConfigured|Aucun client Azure Active Directory n’est configuré pour le service.|  
|CheckboxLabelUserRegistrationTermsConsentRequired|J’accepte les `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use</a\>`.|  
|TextblockUserRegistrationTermsProvided|Veuillez consulter les `<a data-toggle="modal" href="#" data-target="#terms"\>Terms of Use.</a\>`.|  
|DialogHeadingTermsOfUse|Conditions d’utilisation|  
|ValidationMessageConsentNotAccepted|Vous devez accepter les conditions d’utilisation pour pouvoir continuer.|  
  
###  <a name="signinstrings"></a><a name="SigninStrings"></a> SigninStrings  
  
|Name|Texte|  
|----------|----------|  
|WebAuthenticationForgotPassword|Vous avez oublié votre mot de passe ?|  
|WebAuthenticationIfAdministrator|Si vous êtes un administrateur, vous devez vous connecter `<a href="{0}"\>here</a\>`.|  
|WebAuthenticationNotAMember|Vous n’êtes pas encore membre ? `<a href="/signup"\>Sign up now</a\>`|  
|WebAuthenticationRemember|Se souvenir de moi sur cet ordinateur|  
|WebAuthenticationSigininWithPassword|Connectez-vous avec votre nom d’utilisateur et votre mot de passe.|  
|WebAuthenticationSigninTitle|Se connecter|  
|WebAuthenticationSignUpNow|ici|  
  
###  <a name="signupstrings"></a><a name="SignupStrings"></a> SignupStrings  
  
|Name|Texte|  
|----------|----------|  
|PageTitleSignup|Inscription|  
|WebAuthenticationAlreadyAMember|Déjà membre ?|  
|WebAuthenticationCreateNewAccount|Créer un nouveau compte Gestion des API|  
|WebAuthenticationSigninNow|Se connecter|  
|ButtonLabelSignup|Inscription|  
  
###  <a name="subscriptionliststrings"></a><a name="SubscriptionListStrings"></a> SubscriptionListStrings  
  
|Name|Texte|  
|----------|----------|  
|SubscriptionCancelConfirmation|Voulez-vous vraiment annuler cet abonnement ?|  
|SubscriptionRenewConfirmation|Voulez-vous vraiment renouveler cet abonnement ?|  
|WebDevelopersManageSubscriptions|Gérer les abonnements|  
|WebDevelopersPrimaryKey|Clé primaire|  
|WebDevelopersRegenerateLink|Régénérer|  
|WebDevelopersSecondaryKey|Clé secondaire|  
|ButtonLabelShowKey|Afficher|  
|ButtonLabelRenewSubscription|Renouveler|  
|WebDevelopersSubscriptionRequested|Demandé sur {0}|  
|WebDevelopersSubscriptionRequestedState|Demandé|  
|WebDevelopersSubscriptionTableNameHeader|Name|  
|WebDevelopersSubscriptionTableStateHeader|State|  
|WebDevelopersUsageStatisticsLink|Rapports d’analyse|  
|WebDevelopersYourSubscriptions|Vos abonnements|  
|SubscriptionPropertyLabelRequestedDate|Demandé le|  
|SubscriptionPropertyLabelStartedDate|Démarré le|  
|PageTitleRenameSubscription|Renommer l’abonnement|  
|SubscriptionPropertyLabelName|Nom d’abonnement|  
  
###  <a name="subscriptionstrings"></a><a name="SubscriptionStrings"></a> SubscriptionStrings  
  
|Name|Texte|  
|----------|----------|  
|SectionHeadingCloseAccount|Vous souhaitez fermer votre compte ?|  
|PageTitleDeveloperProfile|Profil|  
|ButtonLabelHideKey|Masquer|  
|ButtonLabelRegenerateKey|Régénérer|  
|InformationMessageKeyWasRegenerated|Voulez-vous vraiment régénérer cette clé ?|  
|ButtonLabelShowKey|Afficher|  
  
###  <a name="updateprofilestrings"></a><a name="UpdateProfileStrings"></a> UpdateProfileStrings  
  
|Name|Texte|  
|----------|----------|  
|ButtonLabelUpdateProfile|Mettre à jour le profil|  
|PageTitleUpdateProfile|Mettre à jour les informations du compte|  
  
###  <a name="userprofile"></a><a name="UserProfile"></a> UserProfile  
  
|Name|Texte|  
|----------|----------|  
|ButtonLabelChangeAccountInfo|Modifier les informations du compte|  
|ButtonLabelChangePassword|Modifier le mot de passe|  
|ButtonLabelCloseAccount|Fermer le compte|  
|TextboxLabelEmail|Email|  
|TextboxLabelEmailFirstName|Prénom|  
|TextboxLabelEmailLastName|Nom|  
|TextboxLabelNotificationsSenderEmail|Adresse de messagerie de l’expéditeur des notifications|  
|TextboxLabelOrganizationName|Nom de l’organisation|  
|SubscriptionStateActive|Actif|  
|SubscriptionStateCancelled|Annulé|  
|SubscriptionStateExpired|Expiré|  
|SubscriptionStateRejected|Rejeté|  
|SubscriptionStateRequested|Demandé|  
|SubscriptionStateSuspended|Interrompu|  
|DefaultSubscriptionNameTemplate|{0} (valeur par défaut)|  
|SubscriptionNameTemplate|Accès développeurs #{0}|  
|TextboxLabelSubscriptionName|Nom d’abonnement|  
|ValidationMessageSubscriptionNameRequired|Le nom d'abonnement ne peut pas être vide.|  
|ApiManagementUserLimitReached|Ce service a atteint le nombre maximal d’utilisateurs autorisés. Veuillez passer à un niveau tarifaire supérieur.|  
  
##  <a name="glyph-resources"></a><a name="glyphs"></a> Ressources de glyphes  
 Les modèles du portail des développeurs de la Gestion des API peuvent utiliser les glyphes [Glyphicons de Bootstrap](https://getbootstrap.com/components/#glyphicons). Cet ensemble de glyphes contient plus de 250 glyphes au format de police provenant de l’ensemble Halflings [Glyphicon](https://glyphicons.com/). Pour utiliser un glyphe de cet ensemble, utilisez la syntaxe suivante.  
  
```html  
<span class="glyphicon glyphicon-user">  
```  
  
 Pour connaître la liste complète des glyphes, consultez [Glyphicons de Bootstrap](https://getbootstrap.com/components/#glyphicons).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’utilisation de modèles, consultez la page [Guide pratique de personnalisation du portail des développeurs Gestion des API à l’aide de modèles](api-management-developer-portal-templates.md).
