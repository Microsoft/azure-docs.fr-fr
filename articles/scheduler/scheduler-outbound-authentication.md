---
title: Authentification sortante
description: Découvrez comment configurer ou supprimer l’authentification sortante pour Azure Scheduler
services: scheduler
ms.service: scheduler
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: article
ms.date: 08/15/2016
ms.openlocfilehash: 0a8d79af9f45731971cb1be1f39fc193f9d0f0d9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80878967"
---
# <a name="outbound-authentication-for-azure-scheduler"></a>Authentification sortante pour Azure Scheluler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

Les travaux Azure Scheduler peuvent avoir à appeler des services qui requièrent une authentification, tels que d’autres services Azure, Salesforce.com, Facebook et des sites web personnalisés sécurisés. Le service appelé peut déterminer si le travail Scheduler peut accéder aux ressources demandées. 

Scheduler prend en charge les modèles d’authentification suivants : 

* Authentification par *certificat client* lors de l’utilisation de certificats clients SSL/TLS
* Authentification *de base*
* Authentification *OAuth Active Directory*

## <a name="add-or-remove-authentication"></a>Ajouter ou supprimer l’authentification

* Pour ajouter l’authentification à un travail Scheduler, lorsque vous créez ou mettez à jour le travail, ajoutez l’élément enfant JSON (JavaScript Object Notation) `authentication` à l’élément `request`. 

  Les réponses ne renvoient jamais les secrets transmis au service Scheduler via une demande PUT, PATCH ou POST dans l'objet `authentication`. 
  Les réponses définissent les informations secrètes sur la valeur null ou peuvent utiliser un jeton public qui représente l'entité authentifiée. 

* Pour supprimer l’authentification d’un travail Scheduler, exécutez explicitement une demande PUT ou PATCH sur le travail et définissez l’objet `authentication` sur null. La réponse ne contiendra aucune propriété d’authentification.

## <a name="client-certificate"></a>Certificat client

### <a name="request-body---client-certificate"></a>Corps de la demande - Certificat Client

Lorsque vous ajoutez l'authentification à l'aide du modèle `ClientCertificate`, spécifiez les éléments supplémentaires suivants dans le corps de la demande.  

| Élément | Obligatoire | Description |
|---------|----------|-------------|
| **authentication** (élément parent) | Objet d’authentification pour l’utilisation d’un certificat client SSL/TLS |
| **type** | Oui | Type d’authentification. Pour les certificats clients SSL/TLS, la valeur est `ClientCertificate`. |
| **pfx** | Oui | Contenu codé en base64 du fichier PFX |
| **mot de passe** | Oui | Mot de passe pour accéder au fichier PFX |
||| 

### <a name="response-body---client-certificate"></a>Corps de la réponse - Certificat client 

Lorsqu'une demande est envoyée avec des informations d'authentification, la réponse contient les éléments d’authentification suivants.

| Élément | Description | 
|---------|-------------| 
| **authentication** (élément parent) | Objet d’authentification pour l’utilisation d’un certificat client SSL/TLS |
| **type** | Type d’authentification. Pour les certificats clients SSL/TLS, la valeur est `ClientCertificate`. |
| **certificateThumbprint** |Empreinte numérique du certificat |
| **certificateSubjectName** |Nom unique du sujet du certificat |
| **certificateExpiration** | Date d'expiration du certificat |
||| 

### <a name="sample-rest-request---client-certificate"></a>Exemple de demande REST - Certificat client

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "clientcertificate",
          "password": "password",
          "pfx": "pfx key"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---client-certificate"></a>Exemple de réponse REST - Certificat client

```json
HTTP/1.1 200 OKCache-Control: no-cache
Pragma: no-cache
Content-Length: 858
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 56c7b40e-721a-437e-88e6-f68562a73aa8
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 1075219e-e879-4030-bc81-094e54fbabce
x-ms-routing-request-id: WESTUS:20160316T190424Z:1075219e-e879-4030-bc81-094e54fbabce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:04:23 GMT

{
  "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
  "type": "Microsoft.Scheduler/jobCollections/jobs",
  "name": "southeastasiajc/httpjob",
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "certificateThumbprint": "88105CG9DF9ADE75B835711D899296CB217D7055",
          "certificateExpiration": "2021-01-01T07:00:00Z",
          "certificateSubjectName": "CN=Scheduler Mgmt",
          "type": "ClientCertificate"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled",
    "status": {
      "nextExecutionTime": "2016-03-16T19:05:00Z",
      "executionCount": 0,
      "failureCount": 0,
      "faultedCount": 0
    }
  }
}
```

## <a name="basic"></a>De base

### <a name="request-body---basic"></a>Corps de la demande - De base

Lorsque vous ajoutez l'authentification à l'aide du modèle `Basic`, spécifiez les éléments supplémentaires suivants dans le corps de la demande.

| Élément | Obligatoire | Description |
|---------|----------|-------------|
| **authentication** (élément parent) | Objet d'authentification pour l’utilisation de l’authentification de base | 
| **type** | Oui | Type d’authentification. Pour l'authentification de base, la valeur est `Basic`. | 
| **username** | Oui | Nom d'utilisateur à authentifier | 
| **mot de passe** | Oui | Mot de passe à authentifier |
|||| 

### <a name="response-body---basic"></a>Corps de la réponse - De base

Lorsqu'une demande est envoyée avec des informations d'authentification, la réponse contient les éléments d’authentification suivants.

| Élément | Description | 
|---------|-------------|
| **authentication** (élément parent) | Objet d'authentification pour l’utilisation de l’authentification de base |
| **type** | Type d’authentification. Pour l'authentification de base, la valeur est `Basic`. |
| **username** | Nom d'utilisateur authentifié |
||| 

### <a name="sample-rest-request---basic"></a>Exemple de demande REST - De base

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 562
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "type": "basic",
          "username": "user",
          "password": "password"
        }
      },
      "type": "http"
    },
    "recurrence": {
      "frequency": "minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "enabled"
  }
}
```

### <a name="sample-rest-response---basic"></a>Exemple de réponse REST - De base

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 701
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: a2dcb9cd-1aea-4887-8893-d81273a8cf04
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 7816f222-6ea7-468d-b919-e6ddebbd7e95
x-ms-routing-request-id: WESTUS:20160316T190506Z:7816f222-6ea7-468d-b919-e6ddebbd7e95
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:05:06 GMT

{  
   "id":"/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type":"Microsoft.Scheduler/jobCollections/jobs",
   "name":"southeastasiajc/httpjob",
   "properties":{  
      "startTime":"2015-05-14T14:10:00Z",
      "action":{  
         "request":{  
            "uri":"https://mywebserviceendpoint.com",
            "method":"GET",
            "headers":{  
               "x-ms-version":"2013-03-01"
            },
            "authentication":{  
               "username":"user1",
               "type":"Basic"
            }
         },
         "type":"Http"
      },
      "recurrence":{  
         "frequency":"Minute",
         "endTime":"2016-04-10T08:00:00Z",
         "interval":1
      },
      "state":"Enabled",
      "status":{  
         "nextExecutionTime":"2016-03-16T19:06:00Z",
         "executionCount":0,
         "failureCount":0,
         "faultedCount":0
      }
   }
}
```

## <a name="active-directory-oauth"></a>OAuth Active Directory

### <a name="request-body---active-directory-oauth"></a>Corps de la demande - OAuth Active Directory 

Lorsque vous ajoutez l'authentification à l'aide du modèle `ActiveDirectoryOAuth`, spécifiez les éléments supplémentaires suivants dans le corps de la demande.

| Élément | Obligatoire | Description |
|---------|----------|-------------|
| **authentication** (élément parent) | Oui | Objet d'authentification pour l'utilisation de l’authentification ActiveDirectoryOAuth |
| **type** | Oui | Type d’authentification. Pour l’authentification ActiveDirectoryOAuth, la valeur est `ActiveDirectoryOAuth`. |
| **client** | Oui | L’identificateur de client pour le client Azure AD. Pour trouver l’identificateur de locataire pour le locataire Azure AD, exécutez `Get-AzureAccount` dans Azure PowerShell. |
| **public ciblé** | Oui | Cette valeur est définie sur `https://management.core.windows.net/`. | 
| **clientId** | Oui | Identifiant client de l’application Azure AD | 
| **secret** | Oui | Secret du client qui demande le jeton | 
|||| 

### <a name="response-body---active-directory-oauth"></a>Corps de la réponse - OAuth Active Directory

Lorsqu'une demande est envoyée avec des informations d'authentification, la réponse contient les éléments d’authentification suivants.

| Élément | Description |
|---------|-------------|
| **authentication** (élément parent) | Objet d'authentification pour l'utilisation de l’authentification ActiveDirectoryOAuth |
| **type** | Type d’authentification. Pour l’authentification ActiveDirectoryOAuth, la valeur est `ActiveDirectoryOAuth`. | 
| **client** | L’identifiant de locataire pour le locataire Azure AD |
| **public ciblé** | Cette valeur est définie sur `https://management.core.windows.net/`. |
| **clientId** | Identifiant client de l’application Azure AD |
||| 

### <a name="sample-rest-request---active-directory-oauth"></a>Exemple de demande REST - OAuth Active Directory

```json
PUT https://management.azure.com/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobcollections/southeastasiajc/jobs/httpjob?api-version=2016-01-01 HTTP/1.1
User-Agent: Fiddler
Host: management.azure.com
Authorization: Bearer sometoken
Content-Length: 757
Content-Type: application/json; charset=utf-8

{
  "properties": {
    "startTime": "2015-05-14T14:10:00Z",
    "action": {
      "request": {
        "uri": "https://mywebserviceendpoint.com",
        "method": "GET",
        "headers": {
          "x-ms-version": "2013-03-01"
        },
        "authentication": {
          "tenant":"microsoft.onmicrosoft.com",
          "audience":"https://management.core.windows.net/",
          "clientId":"dc23e764-9be6-4a33-9b9a-c46e36f0c137",
          "secret": "G6u071r8Gjw4V4KSibnb+VK4+tX399hkHaj7LOyHuj5=",
          "type":"ActiveDirectoryOAuth"
        }
      },
      "type": "Http"
    },
    "recurrence": {
      "frequency": "Minute",
      "endTime": "2016-04-10T08:00:00Z",
      "interval": 1
    },
    "state": "Enabled"
  }
}
```

### <a name="sample-rest-response---active-directory-oauth"></a>Exemple de réponse REST - OAuth Active Directory

```json
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 885
Content-Type: application/json; charset=utf-8
Expires: -1
x-ms-request-id: 86d8e9fd-ac0d-4bed-9420-9baba1af3251
Server: Microsoft-IIS/8.5
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
x-ms-ratelimit-remaining-subscription-resource-requests: 599
x-ms-correlation-request-id: 5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
x-ms-routing-request-id: WESTUS:20160316T191003Z:5183bbf4-9fa1-44bb-98c6-6872e3f2e7ce
Strict-Transport-Security: max-age=31536000; includeSubDomains
Date: Wed, 16 Mar 2016 19:10:02 GMT

{
   "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/CS-SoutheastAsia-scheduler/providers/Microsoft.Scheduler/jobCollections/southeastasiajc/jobs/httpjob",
   "type": "Microsoft.Scheduler/jobCollections/jobs",
   "name": "southeastasiajc/httpjob",
   "properties": {
      "startTime": "2015-05-14T14:10:00Z",
      "action": {  
         "request": {
            "uri": "https://mywebserviceendpoint.com",
            "method": "GET",
            "headers": {  
               "x-ms-version": "2013-03-01"
            },
            "authentication": {  
               "tenant": "microsoft.onmicrosoft.com",
               "audience": "https://management.core.windows.net/",
               "clientId": "dc23e764-9be6-4a33-9b9a-c46e36f0c137",
               "type": "ActiveDirectoryOAuth"
            }
         },
         "type": "Http"
      },
      "recurrence": {  
         "frequency": "minute",
         "endTime": "2016-04-10T08:00:00Z",
         "interval": 1
      },
      "state": "Enabled",
      "status": {  
         "lastExecutionTime": "2016-03-16T19:10:00.3762123Z",
         "nextExecutionTime": "2016-03-16T19:11:00Z",
         "executionCount": 5,
         "failureCount": 5,
         "faultedCount": 1
      }
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Limites, valeurs par défaut et codes d’erreur d’Azure Scheluler](scheduler-limits-defaults-errors.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)