---
title: Surveiller et gérer la création de certificats
description: Scénarios illustrant un éventail d’options permettant de créer un certificat, de surveiller le processus de création du certificat et d’interagir avec ce dernier avec Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: conceptual
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: 02e13ce81ed2f11c0bb69015a4864c4a1ad55593
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81427365"
---
# <a name="monitor-and-manage-certificate-creation"></a>Surveiller et gérer la création de certificats
S’applique à : Azure

Scénarios/Opérations décrits dans cet article :

- Demander un certificat Key Vault avec un émetteur pris en charge
- Obtenir une demande en attente : état de la demande « inProgress »
- Obtenir une demande en attente : état de la demande « complete »
- Obtenir une demande en attente : état de la demande en attente « canceled » ou « failed »
- Obtenir une demande en attente : état de la demande en attente « deleted » ou « overwritten »
- Créer (ou importer) lorsqu’une demande en attente existe : état « inProgress »
- Procéder à une fusion lorsqu’une demande en attente est créée avec un émetteur (DigiCert, par exemple)
- Demander une annulation alors que l’état de la demande en attente est « inProgress »
- Supprimer un objet de demande en attente
- Créer manuellement un certificat Key Vault
- Procéder à une fusion lors de la création d’une demande en attente : création manuelle d’un certificat

## <a name="request-a-kv-certificate-with-a-supported-issuer"></a>Demander un certificat Key Vault avec un émetteur pris en charge 

|Méthode|URI de demande|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

Les exemples suivants nécessitent qu’un objet nommé « mydigicert » soit déjà disponible dans votre coffre de clés, le fournisseur de l’émetteur étant DigiCert. L’émetteur du certificat est une entité représentée dans Azure Key Vault (KV) en tant que ressource CertificateIssuer. Il fournit des informations sur la source d’un certificat KV, ainsi que le nom de l’émetteur, le fournisseur, les informations d’identification et d’autres détails administratifs.

### <a name="request"></a>Requête

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert",
      "cty": "OV-SSL",
    }
  }
}
```

### <a name="response"></a>response

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "mydigicert"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "InProgress",
  "status_details": "Pending certificate created. Certificate request is in progress. This may take some time based on the issuer provider. Please check again later",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-inprogress"></a>Obtenir une demande en attente : état de la demande « inProgress »

|Méthode|URI de demande|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Requête
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

> [!NOTE]
> Si l’élément *request_id* est spécifié dans la requête, il fait office de filtre. Si les éléments *request_id* de la requête et de l’objet en attente sont différents, un code d’état HTTP 404 est retourné.

### <a name="response"></a>response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="get-pending-request---request-status-is-complete"></a>Obtenir une demande en attente : état de la demande « complete »

### <a name="request"></a>Requête

|Méthode|URI de demande|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "completed",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "target": “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
}

```

## <a name="get-pending-request---pending-request-status-is-canceled-or-failed"></a>Obtenir une demande en attente : état de la demande en attente « canceled » ou « failed »

### <a name="request"></a>Requête

|Méthode|URI de demande|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "failed",
  "status_details": "",
  "request_id": "a76827a18b63421c917da80f28e9913d",
  "error": {
    "code": "<errorcode>",
    "message": "<message>"
  }
}

```

> [!NOTE]
> la valeur *errorcode* peut être « Certificate issuer error » (Erreur de l’émetteur de certificat) ou « Request rejected » (Demande rejetée) selon que l’erreur relève de l’émetteur ou de l’utilisateur.

## <a name="get-pending-request---pending-request-status-is-deleted-or-overwritten"></a>Obtenir une demande en attente : état de la demande en attente « deleted » ou « overwritten »
Un objet en attente peut être supprimé ou remplacé par une opération de création/importation lorsque son état n’est pas « inProgress ».

|Méthode|URI de demande|
|------------|-----------------|
|GET|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Requête
GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

GET `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>response

```
StatusCode: 404, ReasonPhrase: 'Not Found'
{
  "error": {
    "code": "PendingCertificateNotFound",
    "message": "…"
  }
}

```

## <a name="create-or-import-when-pending-request-exists---status-is-inprogress"></a>Créer (ou importer) lorsqu’une demande en attente existe : état « inProgress »
Quatre états sont possibles pour un objet en attente : « inprogress », « canceled », « failed » ou « completed ».

Si l’état d’une demande en attente est « inprogress », les opérations de création (et d’importation) échouent avec un code d’état HTTP 409 (conflit).

Pour résoudre un conflit :

- Si le certificat Key Vault est créé manuellement, vous pouvez terminer sa création en effectuant une fusion ou une suppression de l’objet en attente.

- Si le certificat est créé avec un émetteur, vous pouvez patienter jusqu’à son achèvement, son échec ou son annulation. Vous pouvez également supprimer l’objet en attente.

> [!NOTE]
> La suppression d’un objet en attente peut ou non annuler la demande de certificat X509 auprès du fournisseur.

|Méthode|URI de demande|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Requête

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "mydigicert"
    }
  }
}
```

### <a name="response"></a>response

```
StatusCode: 409, ReasonPhrase: 'Conflict'
{
  "error": {
    "code": "Forbidden",
    "message": "A new key vault certificate can not be created or imported while a pending key vault certificate's status is inProgress."
  }
}

```

## <a name="merge-when-pending-request-is-created-with-an-issuer"></a>Procéder à une fusion lorsqu’une demande en attente est créée avec un émetteur
La fusion n’est pas autorisée quand un objet en attente est créé avec un émetteur, mais elle l’est lorsque son état est « inProgress ». 

Si la demande de création du certificat X509 échoue ou est annulée pour une raison quelconque et si un certificat X509 peut être récupéré par des moyens hors bande, une opération de fusion peut être effectuée pour terminer la création du certificat Key Vault.

|Méthode|URI de demande|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Requête

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

### <a name="response"></a>response

```json
StatusCode: 403, ReasonPhrase: 'Forbidden'
{
  "error": {
    "code": "Forbidden",
    "message": "Merge is forbidden on pending object created with issuer : <issuer-name> while it is in progess."
  }
}

```

## <a name="request-a-cancellation-while-the-pending-request-status-is-inprogress"></a>Demander une annulation alors que l’état de la demande en attente est « inProgress »
Une annulation peut uniquement être demandée. Une demande peut ou non être annulée. Si l’état d’une demande n’est pas « inProgress », une erreur d’état HTTP 400 (demande incorrecte) est retournée.

|Méthode|URI de demande|
|------------|-----------------|
|PATCH|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Requête
PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

PATCH `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

```json
{
  "cancellation_requested": true
}

```

### <a name="response"></a>response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": true,
  "status": "inProgress",
  "status_details": "…",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}
```

## <a name="delete-a-pending-request-object"></a>Supprimer un objet de demande en attente

> [!NOTE]
> La suppression de l’objet en attente peut ou non annuler la demande de certificat X509 auprès du fournisseur.

|Méthode|URI de demande|
|------------|-----------------|
|Suppression|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}`|

### <a name="request"></a>Requête
DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"`

OR

DELETE `“https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}"`

### <a name="response"></a>response

```
StatusCode: 200, ReasonPhrase: 'OK'
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "{issuer-name}"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "cancellation_requested": false,
  "status": "inProgress",
  "request_id": "a76827a18b63421c917da80f28e9913d",
}
```

## <a name="create-a-kv-certificate-manually"></a>Créer manuellement un certificat Key Vault
Vous pouvez créer un certificat émis par une autorité de certification de votre choix via un processus de création manuelle. Définissez le nom de l’émetteur sur « Unknown » ou ne spécifiez pas le champ de l’émetteur.

|Méthode|URI de demande|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/create?api-version={api-version}`|

### <a name="request"></a>Requête

```json
{
  "policy": {
    "x509_props": {
      "subject": "CN=MyCertSubject1"
    },
    "issuer": {
      "name": "Unknown"
    }
  }
}

```

### <a name="response"></a>response

```
StatusCode: 202, ReasonPhrase: 'Accepted'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1/pending?api-version={api-version}&request_id=a76827a18b63421c917da80f28e9913d"
{
  "id": “https://mykeyvault.vault.azure.net/certificates/mycert1/pending",
  "issuer": {
    "name": "Unknown"
  },
  "csr": "MIICq......DD5Lp5cqXg==",
  "status": "inProgress",
  "status_details": "Pending certificate created. Please Perform Merge to complete the request.",
  "request_id": "a76827a18b63421c917da80f28e9913d"
}

```

## <a name="merge-when-a-pending-request-is-created---manual-certificate-creation"></a>Procéder à une fusion lors de la création d’une demande en attente : création manuelle d’un certificat

|Méthode|URI de demande|
|------------|-----------------|
|POST|`https://mykeyvault.vault.azure.net/certificates/mycert1/pending/merge?api-version={api-version}`|

### <a name="request"></a>Requête

```json
{
  "x5c": [ "MIICxTCCAbi………………………trimmed for brevitiy……………………………………………EPAQj8=" ]
}

```

|Nom de l'élément|Obligatoire|Type|Version|Description|
|------------------|--------------|----------|-------------|-----------------|
|x5c|Oui|tableau|\<introducing version>|Chaîne d’approbation X509 en tant que table de chaînes en base 64.|

### <a name="response"></a>response

```
StatusCode: 201, ReasonPhrase: 'Created'
Location: “https://mykeyvault.vault.azure.net/certificates/mycert1?api-version={api-version}"
{
    "id": "https mykeyvault.vault.azure.net/certificates/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "kid": "https:// mykeyvault.vault.azure.net/keys/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "sid": " mykeyvault.vault.azure.net/secrets/mycert1/f366e1a9dd774288ad84a45a5f620352",
    "cer": "……de34534……",
    "x5t": "n14q2wbvyXr71Pcb58NivuiwJKk",
    "attributes": {
        "enabled": true,
        "exp": 1530394215,
        "nbf": 1435699215,
        "created": 1435699919,
        "updated": 1435699919
    },
    "pending": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/pending"
    },
    "policy": {
        "id": "https:// mykeyvault.vault.azure.net/certificates/mycert1/policy",
        "key_props": {
            "exportable": false,
            "kty": "RSA",
            "key_size": 2048,
            "reuse_key": false
        },
        "secret_props": {
            "contentType": "application/x-pkcs12"
        },
        "x509_props": {
            "subject": "CN=Mycert1",
            "ekus": ["1.3.6.1.5.5.7.3.1", "1.3.6.1.5.5.7.3.2"],
            "validity_months": 12
        },
        "lifetime_actions": [{
            "trigger": {
                "lifetime_percentage": 80
            },
            "action": {
                "action_type": "EmailContacts"
            }
        }],
        "issuer": {
            "name": "Unknown"
        },
        "attributes": {
            "enabled": true,
            "created": 1435699811,
            "updated": 1435699811
        }
    }
}

```
