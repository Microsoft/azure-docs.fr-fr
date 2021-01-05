---
title: Authentification, requêtes et réponses
description: Découvrez comment Azure Key Vault utilise des requêtes et réponses au format JSON et l’authentification requise pour l’utilisation d’un coffre de clés.
services: key-vault
author: amitbapat
manager: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 58616b647affd33e96357e556ab61f85d1c62129
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752275"
---
# <a name="authentication-requests-and-responses"></a>Authentification, requêtes et réponses

Azure Key Vault fournit deux types de conteneurs pour stocker et gérer les secrets de vos applications cloud :

|Type de conteneur|Types d’objets pris en charge|Point de terminaison de plan de données|
|--|--|--|
| **Coffres**|<ul><li>Clés protégées par logiciel</li><li>Clés protégées par HSM (avec la référence (SKU) Premium)</li><li>Certificats</li><li>Clés de compte de stockage</li></ul> | https://{vault-name}.vault.azure.net
|**HSM managé** |<ul><li>Clés protégées par HSM</li></ul> | https://{hsm-name}.managedhsm.azure.net

Voici les suffixes d’URL utilisés pour accéder à chaque type d’objet

|Type d'objet|URL suffix (Suffixe de l’URL)|
|--|--|
|Clés protégées par logiciel| /keys |
|Clés protégées par HSM| /keys |
|Secrets|/secrets|
|Certificats| /certificates|
|Clés de compte de stockage|/storageaccounts
||

Azure Key Vault prend en charge les requêtes et les réponses au format JSON. Les requêtes effectuées auprès d’Azure Key Vault sont dirigées vers une URL Azure Key Vault valide par le biais de la technologie HTTPS, avec des paramètres d’URL et des corps de requête et de réponse encodés au format JSON.

Cette rubrique traite des caractéristiques du service Azure Key Vault. Pour obtenir des informations générales sur l’utilisation des interfaces REST Azure, y compris l’authentification/autorisation et l’acquisition de jetons d’accès, consultez les [informations de référence sur l’API REST Azure](/rest/api/azure).

## <a name="request-url"></a>URL de la demande  
 Les opérations de gestion des clés utilisent HTTP DELETE, GET, PATCH, PUT et HTTP POST, tandis que les opérations de chiffrement sur les objets clés existants utilisent HTTP POST. Les clients qui ne prennent pas en charge certains verbes HTTP peuvent également utiliser HTTP POST à l’aide de l’en-tête X-HTTP-REQUEST pour spécifier le verbe souhaité. Les requêtes ne nécessitant généralement pas de corps doivent inclure un corps vide lors de l’utilisation de HTTP POST, par exemple lors de l’utilisation de POST au lieu de DELETE.  

 Pour utiliser des objets présents dans Azure Key Vault, voici quelques exemples d’URL :  

- Pour CRÉER une clé nommée TESTKEY dans Key Vault, utilisez : `PUT /keys/TESTKEY?api-version=<api_version> HTTP/1.1`  

- Pour IMPORTER une clé nommée IMPORTEDKEY dans Key Vault, utilisez : `POST /keys/IMPORTEDKEY/import?api-version=<api_version> HTTP/1.1`  

- Pour OBTENIR un secret nommé MYSECRET dans Key Vault, utilisez : `GET /secrets/MYSECRET?api-version=<api_version> HTTP/1.1`  

- Pour SIGNER un code de hachage utilisant une clé nommée TESTKEY dans Key Vault, utilisez : `POST /keys/TESTKEY/sign?api-version=<api_version> HTTP/1.1`  

- L’autorité associée à la requête effectuée auprès de Key Vault se présente toujours comme suit : 
  - Pour les coffres : `https://{keyvault-name}.vault.azure.net/`
  - Pour les HSM managés : `https://{HSM-name}.managedhsm.azure.net/`

  Les clés sont toujours stockées sous le chemin d’accès/keys, tandis que les secrets sont toujours stockés sous le chemin d’accès /secrets.  

## <a name="api-version"></a>Version de l'API  
 Le service Azure Key Vault prend en charge le contrôle de version de protocole pour assurer la compatibilité avec les clients de niveau inférieur, bien que certaines fonctionnalités ne soient pas disponibles pour ces clients. Les clients doivent utiliser le paramètre de chaîne de requête `api-version` pour spécifier la version du protocole qu’ils prennent en charge, car il n’existe aucune valeur par défaut.  

 Les versions de protocole Azure Key Vault suivent le schéma de date {YYYY}.{MM}.{JJ}.  

## <a name="request-body"></a>Corps de la requête  
 Conformément à la spécification HTTP, les opérations GET NE doivent PAS avoir un corps de requête, contrairement aux opérations POST et PUT. Le corps des opérations DELETE est facultatif dans HTTP.  

 Sauf indication contraire dans la description de l’opération, le type de contenu du corps de requête doit être application/json et il doit comporter un objet JSON sérialisé conforme au type de contenu.  

 Sauf indication contraire dans la description de l’opération, l’en-tête de requête Accept doit contenir le type de média application/json.  

## <a name="response-body"></a>Corps de la réponse  
 Sauf indication contraire dans la description de l’opération, le type de contenu du corps de réponse des opérations ayant réussi et échoué est application/json. Il comporte des informations d’erreur détaillées.  

## <a name="using-http-post"></a>Utilisation de HTTP POST  
 Certains clients peuvent ne pas être en mesure d’utiliser certains verbes HTTP, comme PATCH ou DELETE. Azure Key Vault prend en charge HTTP POST comme alternative pour ces clients, à condition que le client inclue également l’en-tête « X-HTTP-METHOD » pour spécifier le verbe HTTP d’origine. La prise en charge de HTTP POST est indiquée pour chaque API définie dans ce document.  

## <a name="error-responses"></a>Réponses d’erreur  
 La gestion des erreurs utilise des codes d’état HTTP. Les résultats types sont :  

- 2xx - Réussite : utilisé pour un fonctionnement normal. Le corps de réponse contient le résultat attendu  

- 3xx - Redirection : le message 304 « Non modifié » peut être retourné pour répondre à une opération GET conditionnelle. D’autres codes 3xx pourront indiquer les futures modifications de DNS et de chemin d’accès.  

- 4xx - Erreur du client : utilisé pour les requêtes incorrectes, les clés manquantes, les erreurs de syntaxe, les paramètres non valides, les erreurs d’authentification, etc. Le corps de réponse contient une explication détaillée de l’erreur.  

- 5xx – Erreur de serveur : utilisé pour les erreurs internes du serveur. Le corps de réponse contient une synthèse des informations d’erreur.  

  Le système est conçu pour fonctionner derrière un pare-feu ou un proxy. Par conséquent, un client peut recevoir d’autres codes d’erreur.  

  Azure Key Vault renvoie également des informations d’erreur dans le corps de réponse lorsqu’un problème survient. Le corps de réponse est au format JSON et prend la forme suivante :  

```  

{  
  "error":  
  {  
    "code": "BadArgument",  
    "message":  

      "’Foo’ is not a valid argument for ‘type’."  
    }  
  }  
}  

```  

## <a name="authentication"></a>Authentification  
 Toutes les requêtes auprès d’Azure Key Vault DOIVENT être authentifiées. Azure Key Vault prend en charge les jetons d’accès Azure Active Directory qui peuvent être obtenus à l’aide d’OAuth2 [[RFC6749](https://tools.ietf.org/html/rfc6749)]. 
 
 Pour plus d’informations sur l’enregistrement de votre application et l’authentification permettant d’utiliser Azure Key Vault, consultez [Register your client application with Azure AD (Inscrire votre application cliente avec Azure AD)](/rest/api/azure/index#register-your-client-application-with-azure-ad).
 
 Les jetons d’accès doivent être envoyés au service avec l’en-tête d’autorisation HTTP :  

```  
PUT /keys/MYKEY?api-version=<api_version>  HTTP/1.1  
Authorization: Bearer <access_token>  

```  

 Lorsqu’un jeton d’accès n’est pas fourni, ou lorsqu’un jeton n’est pas accepté par le service, une erreur HTTP 401 est renvoyée au client. Elle inclut l’en-tête WWW-Authenticate, par exemple :  

```  
401 Not Authorized  
WWW-Authenticate: Bearer authorization="…", resource="…"  

```  

 Les paramètres de l’en-tête WWW-Authenticate sont :  

-   authorization : adresse du service d’autorisation OAuth2 qui peut être utilisé afin d’obtenir un jeton d’accès pour la requête.  

-   resource : nom de la ressource (`https://vault.azure.net`) à utiliser dans la requête d'autorisation.

> [!NOTE]
> Les clients du Kit de développement logiciel (SDK) Key Vault pour les secrets, les certificats et les clés dans le premier appel à Key Vault ne fournissent pas de jeton d’accès pour récupérer les informations du locataire. Cet appel est supposé recevoir une réponse HTTP 401 utilisant le client du SDK Key Vault où le coffre de clés montre à l’application l’en-tête WWW-Authenticate contenant la ressource et le locataire auquel il doit accéder et demander le jeton. Si tout est correctement configuré, le deuxième appel de l’application à Key Vault contiendra un jeton valide et réussira. 
