---
title: Tutoriel – Développer un point de terminaison SCIM pour l’approvisionnement d’utilisateurs aux applications à partir d’Azure AD
description: Le système SCIM (Cross-domain Identity Management) normalise le provisionnement automatique des utilisateurs. Ce tutoriel explique comment développer un point de terminaison SCIM, intégrer votre API SCIM avec Azure Active Directory, et automatiser l’approvisionnement d’utilisateurs et de groupes dans vos applications cloud.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.custom: contperf-fy21q2
ms.openlocfilehash: a0abbacc55cff2f561323a22dd83311c87b1511d
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584491"
---
# <a name="tutorial---build-a-scim-endpoint-and-configure-user-provisioning-with-azure-ad"></a>Tutoriel – Créer un point de terminaison SCIM et configurer l’approvisionnement d’utilisateurs avec Azure AD

En tant que développeur d'applications, vous pouvez utiliser l'API de gestion des utilisateurs SCIM (System for Cross-Domain Identity Management) pour activer l'approvisionnement automatique des utilisateurs et des groupes entre votre application et Azure AD. Cet article explique comment créer un point de terminaison SCIM et l'intégrer au service d'approvisionnement Azure AD. La spécification SCIM fournit un schéma utilisateur commun pour l’approvisionnement. Utilisée conjointement avec des normes de fédération comme SAML ou OpenID Connect, la spécification SCIM offre aux administrateurs une solution de bout en bout basée sur des normes pour la gestion des accès.

La spécification SCIM est une définition normalisée de deux points de terminaison : un point de terminaison/Users et un point de terminaison/Groups. Elle utilise des verbes REST communs pour créer, mettre à jour et supprimer des objets, ainsi qu’un schéma prédéfini pour les attributs courants tels que le nom de groupe, le nom d’utilisateur, le prénom, le nom et l'adresse e-mail. Les applications proposant une API REST SCIM 2.0 peuvent réduire ou éliminer les difficultés liées à l’utilisation d’une API de gestion des utilisateurs propriétaires. Par exemple, tout client SCIM conforme sait comment effectuer une requête HTTP POST d'objet JSON sur le point de terminaison/Users afin de créer une nouvelle entrée d’utilisateur. Sans devoir utiliser une API légèrement différente pour les mêmes actions de base, les applications conformes à la norme SCIM peuvent instantanément tirer parti des clients, des outils et du code existants. 

![Approvisionnement d'Azure AD vers une application avec SCIM](media/use-scim-to-provision-users-and-groups/scim-provisioning-overview.png)

Le schéma d’objet utilisateur standard et les API REST de gestion définies dans SCIM 2.0 (RFC [7642](https://tools.ietf.org/html/rfc7642), [7643](https://tools.ietf.org/html/rfc7643), [7644](https://tools.ietf.org/html/rfc7644)) permettent aux fournisseurs d’identité et aux applications de s’intégrer plus facilement entre eux. Les développeurs d’applications qui créent un point de terminaison SCIM peuvent s’intégrer à n’importe quel client conforme à SCIM sans avoir à effectuer de travail personnalisé.

L'automatisation de l'approvisionnement d'une application nécessite la création d'un point de terminaison SCIM et l'intégration de celui-ci avec le client Azure AD SCIM. Procédez comme suit pour commencer à approvisionner des utilisateurs et des groupes dans votre application. 
    
  * **[Étape 1 : Concevoir votre schéma d'utilisateurs et de groupes.](#step-1-design-your-user-and-group-schema)** Identifiez les objets et les attributs dont votre application a besoin, et déterminez comment elles sont mappées au schéma des utilisateurs et des groupes pris en charge par l’implémentation SCIM Azure AD.

  * **[Étape 2 : Comprendre l'implémentation de SCIM Azure AD.](#step-2-understand-the-azure-ad-scim-implementation)** Comprenez la façon dont le client SCIM Azure AD est implémenté, et modélisez la gestion et les réponses de votre demande de protocole SCIM.

  * **[Étape 3 : Créer un point de terminaison SCIM.](#step-3-build-a-scim-endpoint)** Un point de terminaison doit être compatible avec SCIM 2.0 pour s’intégrer au service d’approvisionnement Azure AD. En guise d’option, vous pouvez utiliser des bibliothèques Microsoft Common Language Infrastructure (CLI) et des exemples de code pour créer votre point de terminaison. Ces exemples sont fournis à des fins de référence et de test uniquement. Nous vous déconseillons de coder votre application de production pour établir une dépendance.

  * **[Étape 4 : Intégrer votre point de terminaison SCIM au client SCIM Azure AD.](#step-4-integrate-your-scim-endpoint-with-the-azure-ad-scim-client)** Si votre organisation utilise une application tierce qui implémente le profil SCIM 2.0 pris en charge par Azure AD, vous pouvez commencer à automatiser l’approvisionnement et le désapprovisionnement d’utilisateurs et de groupes dès à présent.

  * **[Étape 5 : Publier votre application dans la galerie d’applications Azure AD.](#step-5-publish-your-application-to-the-azure-ad-application-gallery)** Permettez aux clients de découvrir facilement votre application et de configurer tout aussi facilement l’approvisionnement. 

![Procédure d’intégration d’un point de terminaison SCIM à Azure AD](media/use-scim-to-provision-users-and-groups/process.png)

## <a name="step-1-design-your-user-and-group-schema"></a>Étape 1 : Concevoir votre schéma d'utilisateurs et de groupes

Chaque application requiert des attributs différents pour créer un utilisateur ou un groupe. Commencez votre intégration en identifiant les objets (utilisateurs, groupes) et attributs (nom, responsable, fonction, etc.) nécessaires à votre application. La norme SCIM définit un schéma pour la gestion des utilisateurs et des groupes. Pour le schéma d’utilisateur de base, seuls ces trois attributs sont obligatoires : **id** (identificateur défini par le fournisseur de services), **externalId** (identificateur défini par le client) et **meta** (métadonnées en lecture seule gérées par le fournisseur de services). Tous les autres attributs sont facultatifs. Outre le schéma d’utilisateur de base, la norme SCIM définit une extension d’utilisateur d’entreprise et un modèle pour étendre le schéma d’utilisateur en fonction des besoins de votre application. Si, par exemple, votre application doit spécifier le responsable d’un utilisateur, vous pouvez utiliser le schéma d’entreprise pour collecter le responsable de l’utilisateur, et le schéma de base pour collecter l’adresse e-mail de l’utilisateur. Pour concevoir votre schéma, suivez les étapes ci-dessous :
  1. Listez tous les attributs dont votre application a besoin. Si cela vous aide, décomposez vos exigences en types d’attributs : les attributs requis pour l’authentification (par exemple, loginName et email), les attributs requis pour gérer le cycle de vie de l’utilisateur (par exemple, status/active) et les autres attributs nécessaires au fonctionnement de votre application (par exemple, manager, tag).
  2. Vérifiez si ces attributs sont déjà définis dans le schéma d’utilisateur de base ou le schéma d’utilisateur d’entreprise. Si des attributs dont vous avez besoin ne peuvent pas être définis dans les schémas d’utilisateur de base ou d’entreprise, vous devez définir une extension du schéma d’utilisateur qui couvre ces attributs particuliers. Dans l’exemple ci-dessous, nous avons ajouté une extension à l’utilisateur pour permettre le provisionnement d’un attribut « tag » sur un utilisateur. Nous vous conseillons d’utiliser au début seulement des schémas d’utilisateur de base et d’entreprise, avant de développer des schémas personnalisés supplémentaires.  
  3. Mappez les attributs SCIM aux attributs d’utilisateur dans Azure AD. Si l’un des attributs que vous avez définis dans votre point de terminaison SCIM n’a pas d’équivalent exact dans le schéma d’utilisateur Azure AD, il est probable que les données ne sont pas stockées sur l’objet utilisateur dans la plupart des locataires. Déterminez si cet attribut peut être facultatif pour la création d’un utilisateur. Si l’attribut est indispensable au fonctionnement de votre application, demandez à l’administrateur du locataire d’étendre son schéma ou d’utiliser un attribut d’extension comme indiqué ci-dessous pour la propriété « tags ».

### <a name="table-1-outline-the-attributes-that-you-need"></a>Tableau 1 : Lister les attributs dont vous avez besoin 
| Étape 1 : Déterminer les attributs requis par votre application| Étape 2 : Mapper les exigences de l’application à la norme SCIM| Étape 3 : Mapper les attributs SCIM aux attributs Azure AD|
|--|--|--|
|loginName|userName|userPrincipalName|
|firstName|name.givenName|givenName|
|lastName|name.lastName|lastName|
|workMail|Emails[type eq “work”].value|Messagerie|
|manager|manager|manager|
|tag|urn:ietf:params:scim:schemas:extension:2.0:CustomExtension:tag|extensionAttribute1|
|status|active|isSoftDeleted (valeur calculée non stockée sur l’utilisateur)|

Le schéma défini ci-dessus est représenté à l’aide de la charge utile JSON ci-dessous. Notez qu’en plus des attributs requis par l’application, la représentation JSON contient les attributs obligatoires `id`, `externalId` et `meta`.

```json
{
     "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User",
      "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
      "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:User"],
     "userName":"bjensen",
     "externalId":"bjensen",
     "name":{
       "familyName":"Jensen",
       "givenName":"Barbara"
     },
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
     "Manager": "123456"
   },
     "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User": {
     "tag": "701984",
   },
   "meta": {
     "resourceType": "User",
     "created": "2010-01-23T04:56:22Z",
     "lastModified": "2011-05-13T04:42:34Z",
     "version": "W\/\"3694e05e9dff591\"",
     "location":
 "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646"
   }
}   
```

### <a name="table-2-default-user-attribute-mapping"></a>Tableau 2 : Mappage d’attributs utilisateur par défaut
Vous pouvez ensuite utiliser le tableau ci-dessous pour comprendre comment les attributs requis par votre application peuvent être mappés à un attribut dans Azure AD et la RFC SCIM. Il est possible de [personnaliser](customize-application-attributes.md) la manière dont les attributs sont mappés entre Azure AD et votre point de terminaison SCIM. Notez que vous n'êtes pas tenu de prendre en charge les utilisateurs et groupes, ou tous les attributs affichés ci-dessous. Ils sont fournis à des fins de référence sur la façon dont les attributs d'Azure AD sont souvent mappés à des propriétés dans le protocole SCIM. 

| Utilisateur Azure Active Directory | "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" |
| --- | --- |
| IsSoftDeleted |active |
|department|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|
| displayName |displayName |
|employeeId|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|
| Facsimile-TelephoneNumber |phoneNumbers[type eq "fax"].value |
| givenName |name.givenName |
| jobTitle |title |
| mail |emails[type eq "work"].value |
| mailNickName |externalId |
| manager |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager |
| mobile |phoneNumbers[type eq "mobile"].value |
| postalCode |addresses[type eq "work"].postalCode |
| proxy-Addresses |emails[type eq "other"].Value |
| physical-Delivery-OfficeName |addresses[type eq "other"].Formatted |
| streetAddress |addresses[type eq "work"].streetAddress |
| surname |name.familyName |
| telephone-Number |phoneNumbers[type eq "work"].value |
| user-PrincipalName |userName |


### <a name="table-3-default-group-attribute-mapping"></a>Tableau 3 : Mappage d’attribut de groupe par défaut

| Groupe Azure Active Directory | urn:ietf:params:scim:schemas:core:2.0:Group |
| --- | --- |
| displayName |displayName |
| mail |emails[type eq "work"].value |
| mailNickName |displayName |
| membres |membres |
| objectId |externalId |
| proxyAddresses |emails[type eq "other"].Value |

Plusieurs points de terminaison sont définis dans la RFC SCIM. Vous pouvez commencer avec le point de terminaison /User et développer les autres points de terminaison à partir de celui-ci. Le point de terminaison /Schemas est utile si vous utilisez des attributs personnalisés ou si votre schéma change fréquemment. Il permet à un client de récupérer automatiquement le schéma le plus récent. Le point de terminaison /Bulk est particulièrement utile pour la prise en charge des groupes. Le tableau suivant décrit les différents points de terminaison définis dans la norme SCIM.
 
### <a name="table-4-determine-the-endpoints-that-you-would-like-to-develop"></a>Tableau 4 : Déterminer les points de terminaison à développer
|ENDPOINT|Description|
|--|--|
|/User|Permet d’effectuer des opérations CRUD sur un objet utilisateur.|
|/Group|Permet d’effectuer des opérations CRUD sur un objet groupe.|
|/ServiceProviderConfig|Fournit des détails sur les fonctionnalités de la norme SCIM prises en charge, par exemple les ressources prises en charge et la méthode d’authentification.|
|/ResourceTypes|Spécifie les métadonnées relatives à chaque ressource.|
|/Schemas|L’ensemble des attributs pris en charge par chaque client et chaque fournisseur de services peut varier. Un fournisseur de services peut inclure `name`, `title` et `emails`, tandis qu’un autre fournisseur de services utilise `name`, `title` et `phoneNumbers`. Le point de terminaison /Schemas permet la découverte des attributs pris en charge.|
|/Bulk|Les opérations en bloc vous permettent d’effectuer des opérations sur un grand nombre d’objets ressource à la fois (par exemple, pour mettre à jour les appartenances dans un grand groupe).|


## <a name="step-2-understand-the-azure-ad-scim-implementation"></a>Étape 2 : Comprendre l'implémentation de SCIM Azure AD
> [!IMPORTANT]
> La dernière mise à jour du comportement de l’implémentation SCIM d’Azure AD a été effectuée le 18 décembre 2018. Pour plus d’informations sur ce qui a changé, consultez [Conformité au protocole SCIM 2.0 du service de provisionnement des utilisateurs Azure AD](application-provisioning-config-problem-scim-compatibility.md).

Si vous créez une application prenant en charge une API de gestion utilisateur SCIM 2.0, cette section décrit en détail comment le client SCIM Azure AD est implémenté. Elle vous explique également comment modéliser la gestion et les réponses de la requête du protocole SCIM. Une fois que vous avez implémenté votre point de terminaison SCIM, vous pouvez le tester en suivant la procédure décrite dans la section précédente.

Dans la [spécification du protocole SCIM 2.0](http://www.simplecloud.info/#Specification), votre application doit satisfaire aux conditions suivantes :

* Prise en charge de la création d'utilisateurs et de groupes (facultatif), conformément à la section [3.3 du protocole SCIM](https://tools.ietf.org/html/rfc7644#section-3.3).  
* Prise en charge de la modification d'utilisateurs ou de groupes avec des demandes de correctifs, conformément à la [section 3.5.2 du protocole SCIM](https://tools.ietf.org/html/rfc7644#section-3.5.2). La prise en charge garantit que les groupes et les utilisateurs sont provisionnés de manière performante. 
* Prise en charge de la récupération d'une ressource connue pour un utilisateur ou un groupe créé plus tôt, conformément à la [section 3.4.1 du protocole SCIM](https://tools.ietf.org/html/rfc7644#section-3.4.1).  
* Prise en charge de l'exécution de requêtes d'utilisateurs ou de groupes, conformément à la [section 3.4.2 du protocole SCIM](https://tools.ietf.org/html/rfc7644#section-3.4.2).  Par défaut, les utilisateurs sont récupérés par leur `id` et interrogés via leur `username` et `externalId`, et les groupes sont interrogés via `displayName`.  
* Prise en charge de l'exécution de requêtes d'utilisateurs par ID et par gestion, conformément à la section 3.4.2 du protocole SCIM.  
* Prise en charge de l'exécution de requêtes de groupes par ID et par membre, conformément à la section 3.4.2 du protocole SCIM.  
* Prend en charge le filtre [excludedAttributes=members](#get-group) lors de l’interrogation de la ressource de groupe, conformément à la section 3.4.2.5 du protocole SCIM.
* Acceptation d’un jeton du porteur unique pour l’authentification et l’autorisation d’Azure AD dans votre application.
* Prise en charge de la suppression réversible d’un utilisateur `active=false` et de la restauration de l’utilisateur `active=true` (l’objet utilisateur doit être retourné dans une demande, que l’utilisateur soit actif ou non). La seule fois où l’utilisateur ne doit pas être retourné est lorsqu’il est supprimé définitivement de l’application. 

Suivez ces recommandations lors de l’implémentation d’un point de terminaison SCIM pour garantir la compatibilité avec Azure AD :

* `id` est une propriété nécessaire à toutes les ressources. Chaque réponse qui retourne une ressource doit garantir que chaque ressource dispose de cette propriété, sauf pour `ListResponse` sans membre.
* La réponse à une demande de requête/filtre doit toujours être `ListResponse`.
* Les groupes sont facultatifs mais uniquement pris en charge si l’implémentation SCIM prend en charge les demandes PATCH.
* Il n’est pas nécessaire d’inclure la ressource complète dans la réponse PATCH.
* Microsoft Azure AD utilise uniquement les opérateurs suivants :  
    - `eq`
    - `and`
* Ne nécessitez pas de correspondance sensible à la casse sur des éléments structurels dans SCIM, en particulier pour les valeurs d’opération `op` PATCH, comme défini dans https://tools.ietf.org/html/rfc7644#section-3.5.2. Azure AD émet les valeurs de 'op' en tant que `Add`, `Replace`, et `Remove`.
* Microsoft Azure AD effectue des requêtes pour extraire un utilisateur et un groupe aléatoires pour vous assurer que le point de terminaison et les informations d’identification sont valides. C’est également fait dans le cadre d’un flux de **Connexion test** dans le [portail Azure](https://portal.azure.com). 
* L’attribut sur lequel peuvent être interrogées les ressources doit être défini comme attribut correspondant sur l’application dans le [portail Azure](https://portal.azure.com). Pour plus d’informations, consultez [Personnalisation des mappages d’attributs d’approvisionnement d’utilisateurs pour les applications SaaS dans Azure Active Directory](customize-application-attributes.md)
* Prendre en charge HTTPS sur votre point de terminaison SCIM

### <a name="user-provisioning-and-deprovisioning"></a>Approvisionnement et déprovisionnement d'utilisateurs

L’illustration suivante contient les messages qu’Azure Active Directory envoie à un service SCIM pour gérer le cycle de vie d’un utilisateur dans le magasin d’identités de votre application.  

![Indique la séquence de provisionnement et de déprovisionnement d’utilisateurs](media/use-scim-to-provision-users-and-groups/scim-figure-4.png)<br/>
*Séquence de provisionnement et de déprovisionnement d’utilisateurs*

### <a name="group-provisioning-and-deprovisioning"></a>Approvisionnement et déprovisionnement de groupes

L’approvisionnement et le déprovisionnement de groupes sont facultatifs. L’illustration suivante contient les messages qu’Azure AD envoie à un service SCIM pour gérer le cycle de vie d’un utilisateur dans le cycle de vie d’un groupe du magasin d’identité de l'application.  Ces messages diffèrent des messages concernant les utilisateurs de deux manières :

* Les demandes d’extraction de groupes précisent que l’attribut des membres doit être exclu des ressources fournies en réponse à la demande.  
* Les requêtes permettant de déterminer si un attribut de référence a une certaine valeur sont des requêtes sur les attributs membres.  

![Indique la séquence de provisionnement et de déprovisionnement de groupes](media/use-scim-to-provision-users-and-groups/scim-figure-5.png)<br/>
*Séquence de provisionnement et de déprovisionnement de groupes*

### <a name="scim-protocol-requests-and-responses"></a>Demandes et réponses du protocole SCIM
Cette section fournit des exemples de requêtes SCIM émises par le client SCIM Azure AD et des exemples de réponses attendues. Pour de meilleurs résultats, vous devez coder votre application pour gérer ces demandes dans ce format et émettre les réponses attendues.

> [!IMPORTANT]
> Pour comprendre comment et quand le service d'approvisionnement d'utilisateurs d'Azure AD émet les opérations décrites ci-dessous, consultez la section [Cycles d'approvisionnement : cycle initial et cycle incrémentiel](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) de l'article [Fonctionnement de l'approvisionnement](how-provisioning-works.md).

[Opérations utilisateur](#user-operations)
  - [Create User](#create-user) ([Request](#request) / [Response](#response))
  - [Get User](#get-user) ([Request](#request-1) / [Response](#response-1))
  - [Get User by query](#get-user-by-query) ([Request](#request-2) / [Response](#response-2))
  - [Get User by query - Zero results](#get-user-by-query---zero-results) ([Request](#request-3) / [Response](#response-3))
  - [Update User [Multi-valued properties]](#update-user-multi-valued-properties) ([Request](#request-4) / [Response](#response-4))
  - [Update User [Single-valued properties]](#update-user-single-valued-properties) ([Request](#request-5) / [Response](#response-5)) 
  - [Désactiver l’utilisateur](#disable-user) ([Requête](#request-14) / [Réponse](#response-14))
  - [Delete User](#delete-user) ([Request](#request-6) / [Response](#response-6))


[Opérations de groupe](#group-operations)
  - [Create Group](#create-group) ([Request](#request-7) / [Response](#response-7))
  - [Get Group](#get-group) ([Request](#request-8) / [Response](#response-8))
  - [Get Group by displayName](#get-group-by-displayname) ([Request](#request-9) / [Response](#response-9))
  - [Update Group [Non-member attributes]](#update-group-non-member-attributes) ([Request](#request-10) / [Response](#response-10))
  - [Update Group [Add Members]](#update-group-add-members) ([Request](#request-11) / [Response](#response-11))
  - [Update Group [Remove Members]](#update-group-remove-members) ([Request](#request-12) / [Response](#response-12))
  - [Delete Group](#delete-group) ([Request](#request-13) / [Response](#response-13))

### <a name="user-operations"></a>Opérations utilisateur

* Les utilisateurs peuvent être interrogés via les attributs `userName` ou `email[type eq "work"]`.  

#### <a name="create-user"></a>Create User

###### <a name="request"></a>Requête

*POST /Users*
```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "active": true,
    "emails": [{
        "primary": true,
        "type": "work",
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com"
    }],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName"
    },
    "roles": []
}
```

##### <a name="response"></a>response

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "48af03ac28ad4fb88478",
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="get-user"></a>Obtenir un utilisateur

###### <a name="request"></a><a name="request-1"></a>Requête
*GET /Users/5d48a0a8e9f04aa38008* 

###### <a name="response-user-found"></a><a name="response-1"></a>Réponse (Utilisateur trouvé)
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5d48a0a8e9f04aa38008",
    "externalId": "58342554-38d6-4ec8-948c-50044d0a33fd",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_feed3ace-693c-4e5a-82e2-694be1b39934",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_22370c1a-9012-42b2-bf64-86099c2a1c22@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

###### <a name="request"></a>Requête
*GET /Users/5171a35d82074e068ce2* 

###### <a name="response-user-not-found-note-that-the-detail-is-not-required-only-status"></a>Réponse (Utilisateur introuvable. Notez que seul l’état est requis, et non les détails.)

```json
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:Error"
    ],
    "status": "404",
    "detail": "Resource 23B51B0E5D7AE9110A49411D@7cca31655d49f3640a494224 not found"
}
```

#### <a name="get-user-by-query"></a>Obtenir un utilisateur par requête

##### <a name="request"></a><a name="request-2"></a>Requête

*GET /Users?filter=userName eq "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081"*

##### <a name="response"></a><a name="response-2"></a>Réponse

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2441309d85324e7793ae",
        "externalId": "7fce0092-d52e-4f76-b727-3955bd72c939",
        "meta": {
            "resourceType": "User",
            "created": "2018-03-27T19:59:26.000Z",
            "lastModified": "2018-03-27T19:59:26.000Z"
            
        },
        "userName": "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081",
        "name": {
            "familyName": "familyName",
            "givenName": "givenName"
        },
        "active": true,
        "emails": [{
            "value": "Test_User_91b67701-697b-46de-b864-bd0bbe4f99c1@testuser.com",
            "type": "work",
            "primary": true
        }]
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="get-user-by-query---zero-results"></a>Obtenir un utilisateur par requête - sans résultat

##### <a name="request"></a><a name="request-3"></a>Requête

*GET /Users?filter=userName eq "non-existent user"*

##### <a name="response"></a><a name="response-3"></a>Réponse

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 0,
    "Resources": [],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="update-user-multi-valued-properties"></a>Mettre à jour un utilisateur [Propriétés à plusieurs valeurs]

##### <a name="request"></a><a name="request-4"></a>Requête

*PATCH /Users/6764549bef60420686bc HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
            {
            "op": "Replace",
            "path": "emails[type eq \"work\"].value",
            "value": "updatedEmail@microsoft.com"
            },
            {
            "op": "Replace",
            "path": "name.familyName",
            "value": "updatedFamilyName"
            }
    ]
}
```

##### <a name="response"></a><a name="response-4"></a>Réponse

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "6764549bef60420686bc",
    "externalId": "6c75de36-30fa-4d2d-a196-6bdcdb6b6539",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_fbb9dda4-fcde-4f98-a68b-6c5599e17c27",
    "name": {
        "formatted": "givenName updatedFamilyName",
        "familyName": "updatedFamilyName",
        "givenName": "givenName"
    },
    "active": true,
    "emails": [{
        "value": "updatedEmail@microsoft.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="update-user-single-valued-properties"></a>Mettre à jour un utilisateur [Propriétés à une seule valeur]

##### <a name="request"></a><a name="request-5"></a>Requête

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "userName",
        "value": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com"
    }]
}
```

##### <a name="response"></a><a name="response-5"></a>Réponse

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5171a35d82074e068ce2",
    "externalId": "aa1eca08-7179-4eeb-a0be-a519f7e5cd1a",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "userName": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_49dc1090-aada-4657-8434-4995c25a00f7@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

### <a name="disable-user"></a>Désactiver l’utilisateur

##### <a name="request"></a><a name="request-14"></a>Requête

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": false
        }
    ],
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ]
}
```

##### <a name="response"></a><a name="response-14"></a>Réponse

```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User"
    ],
    "id": "CEC50F275D83C4530A495FCF@834d0e1e5d8235f90a495fda",
    "userName": "deanruiz@testuser.com",
    "name": {
        "familyName": "Harris",
        "givenName": "Larry"
    },
    "active": false,
    "emails": [
        {
            "value": "gloversuzanne@testuser.com",
            "type": "work",
            "primary": true
        }
    ],
    "addresses": [
        {
            "country": "ML",
            "type": "work",
            "primary": true
        }
    ],
    "meta": {
        "resourceType": "Users",
        "location": "/scim/5171a35d82074e068ce2/Users/CEC50F265D83B4530B495FCF@5171a35d82074e068ce2"
    }
}
```
#### <a name="delete-user"></a>Supprimer l'utilisateur

##### <a name="request"></a><a name="request-6"></a>Requête

*DELETE /Users/5171a35d82074e068ce2 HTTP/1.1*

##### <a name="response"></a><a name="response-6"></a>Réponse

*HTTP/1.1 204 No Content*

### <a name="group-operations"></a>Opérations de groupe

* Les groupes doivent toujours être créés avec une liste de membre vide.
* Les groupes peuvent être interrogés via l’attribut `displayName`.
* La mise à jour de la requête PATCH de groupe doit renvoyer *HTTP 204 No Content* dans la réponse. Il n’est pas conseillé de renvoyer un corps avec une liste de tous les membres.
* Il n’est pas nécessaire de prendre en charge le renvoi de tous les membres du groupe.

#### <a name="create-group"></a>Créer un groupe

##### <a name="request"></a><a name="request-7"></a>Requête

*POST /Groups HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group", "http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/2.0/Group"],
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "displayName": "displayName",
    "meta": {
        "resourceType": "Group"
    }
}
```

##### <a name="response"></a><a name="response-7"></a>Réponse

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "927fa2c08dcb4a7fae9e",
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "displayName": "displayName",
    "members": []
}
```

#### <a name="get-group"></a>Obtenir un groupe

##### <a name="request"></a><a name="request-8"></a>Requête

*GET /Groups/40734ae655284ad3abcc?excludedAttributes=members HTTP/1.1*

##### <a name="response"></a><a name="response-8"></a>Réponse
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "40734ae655284ad3abcc",
    "externalId": "60f1bb27-2e1e-402d-bcc4-ec999564a194",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "displayName": "displayName",
}
```

#### <a name="get-group-by-displayname"></a>Obtenir un groupe par displayName

##### <a name="request"></a><a name="request-9"></a>Requête
*GET /Groups?excludedAttributes=members&filter=displayName eq "displayName" HTTP/1.1*

##### <a name="response"></a><a name="response-9"></a>Réponse

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "id": "8c601452cc934a9ebef9",
        "externalId": "0db508eb-91e2-46e4-809c-30dcbda0c685",
        "meta": {
            "resourceType": "Group",
            "created": "2018-03-27T22:02:32.000Z",
            "lastModified": "2018-03-27T22:02:32.000Z",
            
        },
        "displayName": "displayName",
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}
```

#### <a name="update-group-non-member-attributes"></a>Mettre à jour un groupe [Attributs non membre]

##### <a name="request"></a><a name="request-10"></a>Requête

*PATCH /Groups/fa2ce26709934589afc5 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "displayName",
        "value": "1879db59-3bdf-4490-ad68-ab880a269474updatedDisplayName"
    }]
}
```

##### <a name="response"></a><a name="response-10"></a>Réponse

*HTTP/1.1 204 No Content*

### <a name="update-group-add-members"></a>Mettre à jour un groupe [Ajouter des membres]

##### <a name="request"></a><a name="request-11"></a>Requête

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Add",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-11"></a>Réponse

*HTTP/1.1 204 No Content*

#### <a name="update-group-remove-members"></a>Mettre à jour un groupe [Supprimer des membres]

##### <a name="request"></a><a name="request-12"></a>Requête

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Remove",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-12"></a>Réponse

*HTTP/1.1 204 No Content*

#### <a name="delete-group"></a>Suppression d’un groupe

##### <a name="request"></a><a name="request-13"></a>Requête

*DELETE /Groups/cdb1ce18f65944079d37 HTTP/1.1*

##### <a name="response"></a><a name="response-13"></a>Réponse

*HTTP/1.1 204 No Content*

### <a name="security-requirements"></a>Spécifications de sécurité
**Versions du protocole TLS**

Les seules versions du protocole TLS autorisées sont TLS 1.2 et TLS 1.3. Aucune autre version de TLS n’est autorisée. Aucune version de SSL n’est autorisée. 
- Les clés RSA doivent comporter au moins 2 048 bits.
- Les clés ECC doivent avoir au moins 256 bits et être générées à l’aide d’une courbe elliptique approuvée


**Longueur des clés**

Tous les services doivent utiliser des certificats X.509 générés à l’aide de clés de chiffrement d’une longueur suffisante, à savoir :

**Suites de chiffrement**

Tous les services doivent être configurés pour utiliser les suites de chiffrement suivantes, dans l’ordre exact indiqué ci-dessous. Notez que si vous disposez uniquement d’un certificat RSA, les suites de chiffrement ECDSA n’ont aucun effet. </br>

Barre minimale des suites de chiffrement TLS 1.2 :

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

### <a name="ip-ranges"></a>Plages d’adresses IP
Le service de provisionnement Azure AD fonctionne actuellement dans le cadre des plages d’adresses IP pour AzureActiveDirectory, comme indiqué [ici](https://www.microsoft.com/download/details.aspx?id=56519&WT.mc_id=rss_alldownloads_all). Vous pouvez ajouter les plages d’adresses IP listées sous la balise AzureActiveDirectory pour autoriser le trafic en provenance du service de provisionnement Azure AD dans votre application. Notez que vous devez examiner attentivement la liste des plages d’adresses IP pour les adresses calculées. Une adresse telle que « 40.126.25.32 » peut être représentée dans la liste de plages d’adresses IP sous la forme « 40.126.0.0/18 ». Vous pouvez aussi récupérer programmatiquement la liste de plages d’adresses IP avec l’[API](/rest/api/virtualnetwork/servicetags/list) suivante.

## <a name="step-3-build-a-scim-endpoint"></a>Étape 3 : Créer un point de terminaison SCIM

Maintenant que vous avez conçu votre schéma et compris l'implémentation d'Azure AD SCIM, vous pouvez commencer à développer votre point de terminaison SCIM. Plutôt que de réaliser l’ensemble de la procédure et de créer l’implémentation entièrement par vous-même, vous pouvez vous appuyer sur un certain nombre de bibliothèques SCIM open source publiées par la communauté SCIM.

Le [code de référence](https://aka.ms/SCIMReferenceCode) .NET Core open source publié par l’équipe de provisionnement Azure AD est une ressource qui peut vous permettre de démarrer rapidement votre développement. Après avoir créé votre point de terminaison SCIM, il vous faudra le tester. Vous pouvez utiliser la collection de [tests Postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint) fournie dans le cadre du code de référence ou vous servir des exemples de requêtes/réponses fournis [ci-dessus](#user-operations).  

   > [!Note]
   > Le code de référence est destiné à vous permettre de commencer à créer votre point de terminaison SCIM et est fourni « tel quel ». Les contributions de la communauté sont très utiles pour la création et la gestion du code.

La solution est composée de deux projets, _Microsoft.SCIM_ et _Microsoft.SCIM.WebHostSample_.

Le projet _Microsoft.SCIM_ est la bibliothèque qui définit les composants du service web conforme à la spécification SCIM. Elle déclare l’interface _Microsoft.SCIM.IProvider_, les requêtes sont traduites en appels envoyés aux méthodes du fournisseur, lesquelles seraient programmées pour fonctionner sur un magasin d’identités.

![Décomposition : Une requête est traduite en appels vers les méthodes du fournisseur](media/use-scim-to-provision-users-and-groups/scim-figure-3.png)

Le projet _Microsoft.SCIM.WebHostSample_ est une application web ASP.NET Core Visual Studio qui est basée sur le modèle _Empty_. Cela permet de déployer l’exemple de code de manière autonome dans des conteneurs ou au sein d’Internet Information Services. Il implémente également l’interface _Microsoft.SCIM.IProvider_ en conservant les classes en mémoire comme exemple de magasin d’identités.

```csharp
    public class Startup
    {
        ...
        public IMonitor MonitoringBehavior { get; set; }
        public IProvider ProviderBehavior { get; set; }

        public Startup(IWebHostEnvironment env, IConfiguration configuration)
        {
            ...
            this.MonitoringBehavior = new ConsoleMonitor();
            this.ProviderBehavior = new InMemoryProvider();
        }
        ...
```

### <a name="building-a-custom-scim-endpoint"></a>Création d’un point de terminaison SCIM personnalisé

Le service SCIM doit avoir une adresse HTTP et un certificat d’authentification émanant du serveur dont l’autorité de certification racine est l’un des noms suivants :

* CNNIC
* Comodo
* CyberTrust
* DigiCert
* GeoTrust
* GlobalSign
* Go Daddy
* VeriSign
* WoSign

Le kit SDK .NET Core comprend un certificat de développement HTTPS qui peut être utilisé lors du développement, et le certificat est installé dans le cadre de la première exécution. Selon la façon dont vous exécutez l’application web ASP.NET Core, elle écoute un autre port :

* Microsoft.SCIM.WebHostSample : https://localhost:5001
* IIS Express : https://localhost:44359/

Pour plus d’informations sur le protocole HTTPS dans ASP.NET Core, utilisez le lien suivant : [Appliquer le protocole HTTPS dans ASP.NET Core](/aspnet/core/security/enforcing-ssl)

### <a name="handling-endpoint-authentication"></a>Gestion de l’authentification du point de terminaison

Les demandes d’Azure Active Directory incluent un jeton de support OAuth 2.0. Tout service recevant la demande doit authentifier l’émetteur comme étant Azure Active Directory pour le locataire Azure Active Directory attendu.

Dans le jeton, l’émetteur est identifié par une revendication iss, comme `"iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/"`. Dans cet exemple, l’adresse de base de la valeur de revendication, `https://sts.windows.net`, identifie Azure Active Directory comme l’émetteur, tandis que le segment d’adresse relative, _cbb1a5ac-f33b-45fa-9bf5-f37db0fed422_, est un identificateur unique du locataire Azure Active Directory au nom duquel le jeton a été émis.

L’audience du jeton sera l’ID de modèle d’application de l’application de la galerie. Chacune des applications inscrites dans un locataire unique peut recevoir la même revendication `iss` avec les requêtes SCIM. L’ID du modèle d’application pour toutes les applications personnalisées est _8adf8e6e-67b2-4cf2-a259-e3dc5476c621_. Le jeton généré par le service de provisionnement d’Azure AD doit être utilisé uniquement pour le test. Ne l’utilisez pas dans les environnements de production.

Dans l’exemple de code, les requêtes sont authentifiées à l’aide du package Microsoft.AspNetCore.Authentication.JwtBearer. Le code suivant impose que les requêtes envoyées à l’un des points de terminaison du service soient authentifiées à l’aide du jeton du porteur émis par Azure Active Directory pour un locataire spécifié :

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            if (_env.IsDevelopment())
            {
                ...
            }
            else
            {
                services.AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                    .AddJwtBearer(options =>
                    {
                        options.Authority = " https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/";
                        options.Audience = "8adf8e6e-67b2-4cf2-a259-e3dc5476c621";
                        ...
                    });
            }
            ...
        }

        public void Configure(IApplicationBuilder app)
        {
            ...
            app.UseAuthentication();
            app.UseAuthorization();
            ...
       }
```

Un jeton du porteur est également requis pour utiliser les [tests postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint) fournis et effectuer un débogage local à l’aide de localhost. L’exemple de code utilise des environnements ASP.NET Core pour modifier les options d’authentification pendant la phase de développement et activer l’utilisation d’un jeton autosigné.

Pour plus d’informations sur plusieurs environnements dans ASP.NET Core, utilisez le lien suivant : [Utiliser plusieurs environnements dans ASP.NET Core](
https://docs.microsoft.com/aspnet/core/fundamentals/environments)

Le code suivant impose que les requêtes envoyées à l’un des points de terminaison du service soient authentifiées à l’aide du jeton du porteur signé avec une clé personnalisée :

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            if (_env.IsDevelopment())
            {
                services.AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                    .AddJwtBearer(options =>
                    {
                        options.TokenValidationParameters =
                            new TokenValidationParameters
                            {
                                ValidateIssuer = false,
                                ValidateAudience = false,
                                ValidateLifetime = false,
                                ValidateIssuerSigningKey = false,
                                ValidIssuer = "Microsoft.Security.Bearer",
                                ValidAudience = "Microsoft.Security.Bearer",
                                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"))
                            };
                    });
            }
        ...
```

Envoyez une requête GET au contrôleur de jetons pour obtenir un jeton du porteur valide, la méthode _GenerateJSONWebToken_ est chargée de créer un jeton correspondant aux paramètres configurés pour le développement :

```csharp
        private string GenerateJSONWebToken()
        {
            // Create token key
            SymmetricSecurityKey securityKey =
                new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"));
            SigningCredentials credentials =
                new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

            // Set token expiration
            DateTime startTime = DateTime.UtcNow;
            DateTime expiryTime = startTime.AddMinutes(120);

            // Generate the token
            JwtSecurityToken token =
                new JwtSecurityToken(
                    "Microsoft.Security.Bearer",
                    "Microsoft.Security.Bearer",
                    null,
                    notBefore: startTime,
                    expires: expiryTime,
                    signingCredentials: credentials);

            string result = new JwtSecurityTokenHandler().WriteToken(token);
            return result;
        }
```

### <a name="handling-provisioning-and-deprovisioning-of-users"></a>Gestion du provisionnement et de l’annulation du provisionnement des utilisateurs

***Exemple 1. Interroger le service pour obtenir un utilisateur correspondant** _

Azure Active Directory interroge le service pour trouver un utilisateur avec une valeur d’attribut `externalId` correspondant à la valeur d’attribut mailNickname d’un utilisateur dans Azure AD. La requête est exprimée dans le protocole HTTP (Hypertext Transfer Protocol) comme dans cet exemple, jyoung étant un exemple de mailNickname d’utilisateur dans Azure Active Directory.

>[!NOTE]
> Il s'agit simplement d'un exemple. Tous les utilisateurs n’auront pas d’attribut mailNickname, et la valeur dont dispose un utilisateur peut ne pas être unique dans le répertoire. De plus, l’attribut utilisé pour la correspondance (dans ce cas, `externalId`) est configurable dans les [mappages d’attribut Azure AD](customize-application-attributes.md).

```
GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
 Authorization: Bearer ...
```

Dans l’exemple de code, la requête est traduite en un appel à la méthode QueryAsync du fournisseur du service. Voici la signature de cette méthode : 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource is defined in 
 // Microsoft.SCIM.Schemas.  
 // Microsoft.SCIM.IQueryParameters is defined in 
 // Microsoft.SCIM.Protocol.  

 Task<Resource[]> QueryAsync(IRequest<IQueryParameters> request);
```

Dans l’exemple de requête, pour un utilisateur avec une valeur d’attribut `externalId` donnée, les valeurs des arguments transmis à la méthode QueryAsync sont :

_ parameters.AlternateFilters.Count: 1
* parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
* parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"

***Exemple 2. Provisionner un utilisateur** _

Si la réponse à une requête du service web pour un utilisateur avec une valeur d’attribut `externalId` correspondant à la valeur d’attribut mailNickname d’un utilisateur ne renvoie aucun utilisateur, Azure Active Directory demande que le service approvisionne un utilisateur correspondant à celui d’Azure Active Directory.  Voici un exemple de requête : 

```
 POST https://.../scim/Users HTTP/1.1
 Authorization: Bearer ...
 Content-type: application/scim+json
 {
   "schemas":
   [
     "urn:ietf:params:scim:schemas:core:2.0:User",
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0User"],
   "externalId":"jyoung",
   "userName":"jyoung",
   "active":true,
   "addresses":null,
   "displayName":"Joy Young",
   "emails": [
     {
       "type":"work",
       "value":"jyoung@Contoso.com",
       "primary":true}],
   "meta": {
     "resourceType":"User"},
    "name":{
     "familyName":"Young",
     "givenName":"Joy"},
   "phoneNumbers":null,
   "preferredLanguage":null,
   "title":null,
   "department":null,
   "manager":null}
```

Dans l’exemple de code, la requête est traduite en un appel à la méthode CreateAsync du fournisseur du service. Voici la signature de cette méthode : 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource is defined in 
 // Microsoft.SCIM.Schemas.  

 Task<Resource> CreateAsync(IRequest<Resource> request);
```

Dans la requête d’approvisionnement d’un utilisateur, la valeur de l’argument de ressource est une instance de la classe Microsoft.SCIM.Core2EnterpriseUser, définie dans la bibliothèque Microsoft.SCIM.Schemas.  Si la requête d’approvisionnement de l’utilisateur réussit, l’implémentation de la méthode est supposée retourner une instance de la classe Microsoft.SCIM.Core2EnterpriseUser, avec la valeur de la propriété Identificateur définie sur l’identificateur unique de l’utilisateur nouvellement approvisionné.  

_*_Exemple 3. Interroger l'état actuel d’un utilisateur_*_ 

Pour mettre à jour un utilisateur qui existe dans un magasin d’identités avec SCIM frontal, Azure Active Directory continue en demandant au service l’état actuel de cet utilisateur avec une requête de type : 

```
 GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
 Authorization: Bearer ...
```

Dans l’exemple de code, la requête est traduite en un appel à la méthode RetrieveAsync du fournisseur du service. Voici la signature de cette méthode : 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource and 
 // Microsoft.SCIM.IResourceRetrievalParameters 
 // are defined in Microsoft.SCIM.Schemas 

 Task<Resource> RetrieveAsync(IRequest<IResourceRetrievalParameters> request);
```

Dans le cas d’une requête servant à récupérer l’état actuel d’un utilisateur, les valeurs des propriétés de l’objet fourni comme valeur d’argument des paramètres sont les suivantes : 
  
_ Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* SchemaIdentifier : "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

***Exemple 4. Interroger la valeur d’un attribut de référence à mettre à jour** _ 

Si un attribut de référence doit être mis à jour, Azure Active Directory interroge le service pour déterminer si la valeur actuelle de l’attribut de référence dans le magasin d’identités avec le service frontal correspond déjà à la valeur de cet attribut dans Azure Active Directory. Pour les utilisateurs, le seul attribut dont la valeur actuelle est interrogée de cette manière est l’attribut manager. Voici un exemple de requête visant à déterminer si l’attribut manager d’un objet utilisateur a actuellement une certaine valeur : Dans l’exemple de code, la requête est traduite en un appel à la méthode QueryAsync du fournisseur du service. La valeur des propriétés de l’objet fourni en tant que valeur d’argument des paramètres est la suivante : 
  
_ parameters.AlternateFilters.Count: 2
* parameters.AlternateFilters.ElementAt(x).AttributePath: « ID »
* parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
* parameters.AlternateFilters.ElementAt(y).AttributePath: "manager
* parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
* parameters.RequestedAttributePaths.ElementAt(0): « ID »
* parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

Ici, la valeur x de l’index peut être 0 et la valeur y de l’index peut être 1, ou la valeur x peut être 1 et la valeur y peut être 0, selon l’ordre des expressions de paramètre de requête du filtre.   

***Exemple 5. Demande Azure AD à un service SCIM pour mettre à jour un utilisateur** _ 

Voici un exemple de requête d’Azure Active Directory à un service SCIM pour mettre à jour un utilisateur : 

```
  PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
  Authorization: Bearer ...
  Content-type: application/scim+json
  {
    "schemas": 
    [
      "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations":
    [
      {
        "op":"Add",
        "path":"manager",
        "value":
          [
            {
              "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
              "value":"2819c223-7f76-453a-919d-413861904646"}]}]}
```

Dans l’exemple de code, la requête est traduite en un appel à la méthode UpdateAsync du fournisseur du service. Voici la signature de cette méthode : 

```csharp
 // System.Threading.Tasks.Tasks and 
 // System.Collections.Generic.IReadOnlyCollection<T>  // are defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in
 // Microsoft.SCIM.Service.
 // Microsoft.SCIM.IPatch, 
 // is defined in Microsoft.SCIM.Protocol. 

 Task UpdateAsync(IRequest<IPatch> request);
```

Dans le cas d’une demande de mise à jour d’un utilisateur, l’objet fourni comme valeur d’argument de correctif a les valeurs de propriété suivantes : 
  
_ ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier:  "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
* (PatchRequest as PatchRequest2).Operations.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName: OperationName.Add
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath: "manager"
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference: http://.../scim/Users/2819c223-7f76-453a-919d-413861904646
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value: 2819c223-7f76-453a-919d-413861904646

***Exemple 6. Déprovisionner un utilisateur** _

Pour déprovisionner un utilisateur à partir d’un magasin d’identités avec un service SCIM frontal, Azure AD envoie une demande similaire à celle-ci :

```
  DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
  Authorization: Bearer ...
```

Dans l’exemple de code, la requête est traduite en un appel à la méthode DeleteAsync du fournisseur du service. Voici la signature de cette méthode : 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.IResourceIdentifier, 
 // is defined in Microsoft.SCIM.Protocol. 

 Task DeleteAsync(IRequest<IResourceIdentifier> request);
```

L’objet fourni en tant que valeur d’argument resourceIdentifier présente ces valeurs de propriété dans le cas d’une demande de déprovisionnement d'un utilisateur : 

_ ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

## <a name="step-4-integrate-your-scim-endpoint-with-the-azure-ad-scim-client"></a>Étape 4 : Intégrer votre point de terminaison SCIM au client SCIM Azure AD

Azure AD peut être configuré pour attribuer automatiquement des utilisateurs et groupes dans les applications qui implémentent un profil spécifique du [protocole SCIM 2.0](https://tools.ietf.org/html/rfc7644). Les caractéristiques du profil sont détaillées à l'[Étape 2 : Comprendre l'implémentation de SCIM Azure AD](#step-2-understand-the-azure-ad-scim-implementation).

Vérifiez avec votre fournisseur d’application ou dans la documentation du fournisseur de votre application la conformité à ces exigences.

> [!IMPORTANT]
> L’implémentation de SCIM d’Azure AD repose sur le service d’approvisionnement d’utilisateur Azure AD, conçu pour conserver constamment les utilisateurs synchronisés avec Azure AD et l’application cible, et implémente un ensemble très spécifique d’opérations standard. Il est important de comprendre ces comportements afin de cerner le comportement du client SCIM d’Azure AD. Pour plus d'informations, consultez la section [Cycles d'approvisionnement : cycle initial et cycle incrémentiel](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) de l'article [Fonctionnement de l'approvisionnement](how-provisioning-works.md).

### <a name="getting-started"></a>Prise en main

Les applications qui prennent en charge le profil SCIM décrit dans cet article peuvent être connectées à Azure Active Directory à l’aide de la fonctionnalité « application ne figurant pas dans la galerie » dans la galerie d’applications Azure AD. Une fois connecté, Azure AD exécute toutes les 40 minutes un processus de synchronisation au cours duquel il interroge le point de terminaison de SCIM de l’application concernant les utilisateurs et les groupes assignés, et les crée ou les modifie en fonction des détails de l’attribution.

**Pour connecter une application qui prend en charge SCIM :**

1. Connectez-vous au [portail Azure Active Directory](https://aad.portal.azure.com). Notez que vous pouvez accéder à une version d’évaluation gratuite pour Azure Active Directory avec des licences P2 en vous inscrivant au [programme pour les développeurs](https://developer.microsoft.com/office/dev-program)
2. Dans le volet gauche, sélectionnez **Applications d’entreprise**. Une liste de toutes les applications configurées s’affiche, dont les applications qui ont été ajoutées à partir de la galerie.
3. Sélectionnez **+ Nouvelle application** > **Toutes** > **Application ne figurant pas dans la galerie**.
4. Entrez un nom pour votre application, puis sélectionnez **Ajouter** pour créer un objet d’application. La nouvelle application est ajoutée à la liste des applications d’entreprise et s’ouvre sur son écran de gestion d’application.

   ![Capture d’écran montrant la galerie d’applications Azure AD](media/use-scim-to-provision-users-and-groups/scim-figure-2a.png)<br/>
   *Galerie d’applications Azure AD*

5. Dans l’écran de gestion d’application, sélectionnez **Approvisionnement** dans le volet gauche.
6. Dans le menu **Mode d’approvisionnement**, sélectionnez **Automatique**.

   ![Exemple : Page de provisionnement d’une application dans le Portail Microsoft Azure](media/use-scim-to-provision-users-and-groups/scim-figure-2b.png)<br/>
   *Configuration du provisionnement dans le portail Azure*

7. Dans le champ **URL du locataire**, entrez l’URL du point de terminaison SCIM de l’application. Exemple : `https://api.contoso.com/scim/`
8. Si le point de terminaison SCIM requiert un jeton de porteur OAuth d’un émetteur autre qu’Azure AD, copiez le jeton de porteur OAuth requis dans le champ facultatif **Secret Token** (Jeton secret). Si ce champ est laissé vide, Azure AD inclut un jeton de porteur OAuth émis par Azure AD avec chaque requête. Les applications qui utilisent Azure AD comme fournisseur d'identité peuvent valider ce jeton émis par Azure AD. 
   > [!NOTE]
   > Il est **_déconseillé_* _ de laisser ce champ vide et d’utiliser un jeton généré par Azure AD. Cette option est principalement destinée à des fins de test.
9. Sélectionnez _ *Tester la connexion** pour qu’Azure Active Directory tente de se connecter au point de terminaison SCIM. Si la tentative échoue, des informations d’erreur s’affichent.  

    > [!NOTE]
    > **Tester la connexion** interroge le point de terminaison SCIM pour un utilisateur qui n’existe pas, en utilisant un GUID aléatoire en tant que propriété correspondante sélectionnée dans la configuration Azure AD. La réponse correcte attendue est HTTP 200 OK avec un message SCIM ListResponse vide.

10. Si la tentative de connexion à l’application réussit, sélectionnez **Enregistrer** pour enregistrer les informations d’identification d’administrateur.
11. La section **Mappages** présente deux ensembles sélectionnables de [mappages d’attributs](customize-application-attributes.md) : un pour les objets utilisateur et un autre pour les objets de groupe. Sélectionnez chacun d’eux pour consulter les attributs qui sont synchronisés entre Azure Active Directory et votre application. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les utilisateurs et les groupes dans votre application pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    > [!NOTE]
    > Vous pouvez éventuellement désactiver la synchronisation des objets de groupe en désactivant le mappage « Groupes ».

12. Sous **Paramètres**, le champ **Étendue** définit les utilisateurs et les groupes qui sont synchronisés. Sélectionnez **Sync only assigned users and groups (recommended)** (Synchroniser uniquement les utilisateurs et les groupes attribués (recommandé)) pour que seuls les utilisateurs et les groupes attribués soient synchronisés dans l’onglet **Utilisateurs et groupes**.
13. Une fois votre configuration terminée, définissez l’**état d’approvisionnement** sur **Activé**.
14. Sélectionnez **Enregistrer** pour démarrer le service d’approvisionnement Azure AD.
15. Si vous synchronisez uniquement les utilisateurs et les groupes attribués (recommandé), veillez à sélectionner l’onglet **Utilisateurs et groupes** et à attribuer les utilisateurs ou groupes que vous souhaitez synchroniser.

Une fois le cycle initial démarré, vous pouvez sélectionner **Journaux d’approvisionnement** dans le panneau de gauche pour suivre la progression, qui indique toutes les actions effectuées par le service d'approvisionnement de votre application. Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](check-status-user-account-provisioning.md).

> [!NOTE]
> Le cycle initial prend plus de temps que les synchronisations suivantes, qui se produisent environ toutes les 40 minutes, tant que le service est en cours d’exécution.

## <a name="step-5-publish-your-application-to-the-azure-ad-application-gallery"></a>Étape 5 : Publier votre application dans la galerie d’applications Azure AD

Si vous créez une application qui sera utilisée par plusieurs locataires, vous pouvez la rendre disponible dans la galerie d’applications Azure AD. Cela facilite pour les organisations la découverte de l’application et la configuration de l’approvisionnement. Vous pouvez facilement publier votre application dans la galerie Azure AD et mettre l’approvisionnement à la disposition d’autres utilisateurs. Découvrez les étapes [ici](../develop/v2-howto-app-gallery-listing.md). Microsoft vous aidera à intégrer votre application à la galerie, à tester votre point de terminaison et à publier la [documentation](../saas-apps/tutorial-list.md) d'intégration destinée aux clients.

### <a name="gallery-onboarding-checklist"></a>Liste de vérification d’intégration à la galerie
Suivez la liste de vérification ci-dessous pour assurer l’intégration rapide de votre application et une expérience de déploiement fluide pour les clients. Les informations seront collectées auprès de vous lors de l’intégration à la galerie. 
> [!div class="checklist"]
> * Prise en charge d’un point de terminaison de groupe et d’utilisateur [SCIM 2.0](#step-2-understand-the-azure-ad-scim-implementation) (un seul est obligatoire, mais il est recommandé de disposer des deux)
> * Prise en charge d’au moins 25 requêtes par seconde par locataire pour s’assurer que les utilisateurs et les groupes sont provisionnés et déprovisionnés sans délai (obligatoire)
> * Établissement de contacts d’ingénierie et de support pour guider l’intégration des clients à la galerie de publications (obligatoire)
> * Trois informations d’identification de test sans date d’expiration pour l’application (obligatoire)
> * Prise en charge de l’octroi de code d’autorisation OAuth ou d’un jeton de longue durée, comme décrit ci-dessous (obligatoire)
> * Établissement d’un point de contact d’ingénierie et de support pour la prise en charge de l’intégration des clients à la galerie de publications (obligatoire)
> * Prise en charge de la mise à jour de l’appartenance à plusieurs groupes avec un seul correctif (recommandé) 
> * Documentation publique de votre point de terminaison SCIM (recommandée) 
> * [Prise en charge de la détection de schéma](https://tools.ietf.org/html/rfc7643#section-6) (recommandée)


### <a name="authorization-for-provisioning-connectors-in-the-application-gallery"></a>Autorisation des connecteurs d’approvisionnement de connecteurs dans la galerie d’applications
La spécification SCIM ne définit pas de schéma spécifique à SCIM à des fins d'authentification et d'autorisation. Elle s’appuie sur l’utilisation des normes du secteur existantes. Le client d’approvisionnement Azure AD prend en charge deux méthodes d’autorisation pour les applications de la galerie. 

|Méthode d’autorisation|Avantages|Inconvénients|Support|
|--|--|--|--|
|Nom d’utilisateur et mot de passe (non recommandé ou pris en charge par Azure AD)|Facile à implémenter|Non sécurisé - [Votre Pa$$word n’a pas d’importance](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984)|Prise en charge au cas par cas pour les applications de la galerie. Pas de prise en charge pour les applications ne figurant pas dans la galerie.|
|Jeton de porteur de longue durée|Les jetons de longue durée ne nécessitent aucune intervention de la part d’un utilisateur. Ils sont simples à utiliser par les administrateurs qui configurent le provisionnement.|Les jetons de longue durée peuvent être difficiles à partager avec un administrateur sans utiliser de méthodes non sécurisées telles que la messagerie électronique. |Prise en charge pour toutes les applications (celles de la galerie et les autres). |
|Octroi du code d’autorisation OAuth|Les jetons d’accès ont une durée de vie beaucoup plus courte que les mots de passe. Ils comportent un mécanisme d’actualisation automatisée, ce qui n’est pas le cas des jetons de porteur de longue durée.  Un utilisateur doit être physiquement présent lors de l’autorisation initiale, ce qui ajoute un niveau de responsabilité. |Nécessite la présence d’un utilisateur. Si l’utilisateur quitte l’organisation, le jeton n’est plus valide et l’autorisation doit être recommencée.|Prise en charge pour les applications de la galerie, mais pas pour les autres. Toutefois, vous pouvez fournir un jeton d’accès dans l’interface utilisateur en tant que jeton secret à des fins de test à court terme. Notre backlog inclut une prise en charge de l’octroi de code OAuth sur une application ne figurant pas dans la galerie.|
|Octroi d’informations d’identification de client OAuth|Les jetons d’accès ont une durée de vie beaucoup plus courte que les mots de passe. Ils comportent un mécanisme d’actualisation automatisée, ce qui n’est pas le cas des jetons de porteur de longue durée. L’octroi du code d’autorisation et l’octroi d’informations d’identification du client permettent de créer le même type de jeton d’accès. L’utilisation de l’une ou l’autre de ces méthodes est donc transparente pour l’API.  Le provisionnement peut être entièrement automatisé, et de nouveaux jetons peuvent être demandés sans l’assistance d’un utilisateur. ||Pas de prise en charge pour les applications de la galerie ni pour les autres. La prise en charge est dans notre backlog.|

> [!NOTE]
> Il est déconseillé de laisser le champ du jeton vide dans l’interface utilisateur de l’application personnalisée de configuration du provisionnement Azure AD. Le jeton généré est principalement destiné à des fins de test.

**Flux d’octroi du code d’autorisation OAuth :** Le service d’approvisionnement prend en charge l'[octroi du code d’autorisation](https://tools.ietf.org/html/rfc6749#page-24). Une fois la demande de publication de votre application dans la galerie envoyée, notre équipe vous aidera à collecter les informations suivantes :
*  URL d’autorisation : URL client pour obtenir l’autorisation du propriétaire de la ressource via la redirection de l’agent utilisateur. L’utilisateur est redirigé vers cette URL pour autoriser l’accès. Notez que cette URL n’est pas configurable actuellement par le locataire.
*  URL d’échange de jeton : URL client pour échanger un octroi d’autorisation pour un jeton d’accès, généralement avec authentification du client. Notez que cette URL n’est pas configurable actuellement par le locataire.
*  ID client : Le serveur d’autorisation fournit au client inscrit un identifiant client, qui est une chaîne unique représentant les informations d’inscription fournies par le client.  L’identifiant client n’est pas secret ; il est exposé au propriétaire de la ressource et **ne doit pas** être utilisé seul à des fins d'authentification du client.  
*  Clé secrète client : La clé secrète client est une clé secrète générée par le serveur d’autorisation. Il doit s’agir d’une valeur unique exclusivement connue du serveur d’autorisation. 

Notez qu’OAuth v1 n’est pas pris en charge en raison de l’exposition de la clé secrète client. OAuth v2 est pris en charge.  

Meilleures pratiques (recommandées, mais pas obligatoires) :
* Prise en charge de plusieurs URL de redirection. Les administrateurs peuvent configurer l’approvisionnement depuis « portal.azure.com » et « aad.portal.azure.com ». La prise en charge de plusieurs URL de redirection permet aux utilisateurs d'autoriser l’accès depuis l’un ou l’autre des portails.
* Prise en charge de plusieurs clés secrètes pour assurer le bon renouvellement du secret, sans temps d’arrêt. 

Étapes du flux d’octroi de code OAuth :
1. L’utilisateur se connecte au portail Azure > Applications d’entreprise > Sélectionner une application > Approvisionnement, puis clique sur Autoriser.
2. Le portail Azure redirige l’utilisateur vers l’URL d’autorisation (page de connexion pour l’application tierce).
3. L’administrateur fournit des informations d’identification à l’application tierce. 
4. L’application tierce redirige l’utilisateur vers portail Azure et fournit le code d’octroi 
5. Les services d’approvisionnement Azure AD appellent l’URL du jeton et fournissent le code d’octroi. L’application tierce répond avec le jeton d’accès, le jeton d’actualisation et la date d’expiration
6. Lors du démarrage du cycle d’approvisionnement, le service vérifie si le jeton d’accès actuel est valide et l’échange contre un nouveau jeton si nécessaire. Le jeton d’accès est fourni dans chaque demande adressée à l’application et la validité de la demande est vérifiée avant chaque demande.

> [!NOTE]
> Bien qu’il ne soit pas actuellement possible de configurer OAuth sur l’application ne figurant pas dans la galerie, vous pouvez générer manuellement un jeton d’accès à partir de votre serveur d’autorisation et l’entrer dans le champ de jeton secret de l’application ne figurant pas dans la galerie. Cela vous permet de vérifier la compatibilité de votre serveur SCIM avec le client SCIM d’Azure AD avant de l’intégrer à la galerie d’applications qui prend en charge l’octroi de code OAuth.  

**Jetons de porteur OAuth à longue durée :** Si votre application ne prend pas en charge le flux d'octroi du code OAuth, vous pouvez générer un jeton de porteur OAuth à longue durée utilisable par un administrateur pour configurer l’intégration de l’approvisionnement. Le jeton doit être perpétuel, à défaut de quoi le travail d’approvisionnement sera [mis en quarantaine](application-provisioning-quarantine-status.md) à expiration du jeton. La taille de ce jeton doit être inférieure à 1 Ko.  

Si vous souhaitez davantage de méthodes d’authentification et d’autorisation, faites-le nous savoir sur [UserVoice](https://aka.ms/appprovisioningfeaturerequest).

### <a name="gallery-go-to-market-launch-check-list"></a>Liste de vérification du lancement de la galerie sur le marché
Pour contribuer à la sensibilisation et à la demande de notre intégration conjointe, nous vous recommandons de mettre à jour votre documentation existante et d’amplifier son intégration dans vos canaux marketing.  Voici un ensemble d’activités de liste de contrôle que nous vous recommandons d’effectuer à l’appui du lancement

* **Préparation du support client et des ventes.** Assurez-vous que vos équipes de vente et de support sont informées et capables de parler des fonctionnalités d’intégration. Informez vos équipes de vente et de support, mettez à leur disposition un FAQ, et incluez l’intégration dans vos documents de vente. 
* **Billet de blog et/ou communiqué de presse.** Élaborez un billet de blog ou un communiqué de presse décrivant l’intégration conjointe, les avantages et la prise en main. [Exemple : Communiqué de presse Imprivata et Azure Active Directory](https://www.imprivata.com/company/press/imprivata-introduces-iam-cloud-platform-healthcare-supported-microsoft) 
* **Réseaux sociaux.** Tirez parti de vos réseaux sociaux tels que Twitter, Facebook ou LinkedIn pour promouvoir l’intégration à vos clients. Veillez à inclure @AzureAD pour pouvoir retweeter votre publication. [Exemple : Publication Twitter d’Imprivata](https://twitter.com/azuread/status/1123964502909779968)
* **Site web marketing.** Créez ou mettez à jour vos pages marketing (par exemple, page d’intégration, page de partenaire, page de tarification, etc.) pour inclure la disponibilité de l’intégration conjointe. [Exemple : Page d’intégration Pingboard](https://pingboard.com/org-chart-for), [page d’intégration Smartsheet](https://www.smartsheet.com/marketplace/apps/microsoft-azure-ad), [page de tarification Monday.com](https://monday.com/pricing/) 
* **Documentation technique.** Créez un article du centre d’aide ou une documentation technique sur la façon dont les clients peuvent commencer. [Exemple : Envoi + Intégration de Microsoft Azure Active Directory.](https://envoy.help/en/articles/3453335-microsoft-azure-active-directory-integration/
) 
* **Communication du client.** Avertissez les clients de la nouvelle intégration dans le cadre de votre communication (bulletins mensuels, campagnes par courrier électronique, notes de publication de produits). 

## <a name="related-articles"></a>Articles connexes

* [Automatisation de l'approvisionnement et de l'annulation de l'approvisionnement des utilisateurs pour les applications SaaS](user-provisioning.md)
* [Personnaliser les mappages d’attributs pour l’approvisionnement des utilisateurs](customize-application-attributes.md)
* [Écriture d’expressions pour les mappages d’attributs](functions-for-customizing-application-data.md)
* [Filtres d’étendue pour le provisionnement des utilisateurs](define-conditional-rules-for-provisioning-user-accounts.md)
* [Notifications d’approvisionnement de comptes](user-provisioning.md)
* [Liste des didacticiels sur l’intégration des applications SaaS](../saas-apps/tutorial-list.md)

