---
title: Utiliser des messages pour l’intégration à Azure Blockchain Workbench
description: Vue d’ensemble de l’utilisation des messages pour intégrer Azure Blockchain Workbench Preview avec d’autres systèmes.
ms.date: 09/05/2019
ms.topic: article
ms.reviewer: brendal
ms.openlocfilehash: 14bd0f84bc9490d95d3dbe0b9f122882f0d2059d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74324505"
---
# <a name="azure-blockchain-workbench-messaging-integration"></a>Intégration des messages Azure Blockchain Workbench

En plus de fournir une API REST, Azure Blockchain Workbench propose une intégration basée sur la messagerie. Workbench publie des événements orienté registre via Azure Event Grid, ce qui permet aux consommateurs en aval d’ingérer des données ou d’entreprendre des actions basées sur ces événements. Pour les clients qui ont besoin d’une messagerie fiable, Azure Blockchain Workbench transmet également les messages à un point de terminaison Azure Service Bus.

## <a name="input-apis"></a>API d’entrée

Si vous souhaitez initier des transactions à partir de systèmes externes pour créer des utilisateurs, créer des contrats et mettre à jour des contrats, vous pouvez utiliser les API d’entrée de messagerie pour effectuer des transactions sur un registre. Consultez les [exemples d’intégration de messagerie](https://aka.ms/blockchain-workbench-integration-sample) pour obtenir un exemple qui illustre les API d’entrée.

Les API d’entrée actuellement disponibles sont les suivantes.

### <a name="create-user"></a>Créer un utilisateur

Crée un nouvel utilisateur.

La requête requiert les champs suivants :

| **Nom**             | **Description**                                      |
|----------------------|------------------------------------------------------|
| requestId            | GUID fourni par le client                                |
| firstName            | Prénom de l’utilisateur                              |
| lastName             | Nom de l’utilisateur                               |
| emailAddress         | Adresse e-mail de l’utilisateur                           |
| externalId           | ID d’objet Azure AD de l’utilisateur                      |
| connectionId         | Identificateur unique de la connexion blockchain |
| messageSchemaVersion | Version du schéma de messagerie                            |
| messageName          | **CreateUserRequest**                               |

Exemple :

``` json
{
    "requestId": "e2264523-6147-41fc-bbbb-edba8e44562d",
    "firstName": "Ali",
    "lastName": "Alio",
    "emailAddress": "aa@contoso.com",
    "externalId": "6a9b7f65-ffff-442f-b3b8-58a35abd1bcd",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateUserRequest"
}
```

Blockchain Workbench retourne une réponse avec les champs suivants :

| **Nom**              | **Description**                                                                                                             |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------------|
| requestId             | GUID fourni par le client |
| userId                | ID de l’utilisateur qui a été créé. |
| userChainIdentifier   | Adresse de l’utilisateur qui a été créé sur le réseau blockchain. Dans Ethereum, il s’agit de l’adresse **on-chain** de l’utilisateur. |
| connectionId          | Identificateur unique de la connexion blockchain|
| messageSchemaVersion  | Version du schéma de messagerie |
| messageName           | **CreateUserUpdate** |
| status                | État de la demande de création d’utilisateur.  En cas de réussite, la valeur est **Success**. En cas d’échec, la valeur est **Failure**.     |
| additionalInformation | Informations supplémentaires fournies en fonction de l’état |

Exemple de réponse **create user** réussie de Blockchain Workbench :

``` json
{ 
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Success", 
    "additionalInformation": { } 
} 
```

Si la demande a échoué, les détails sur l’échec sont inclus dans les informations supplémentaires.

``` json
{
    "requestId": "e2264523-6147-41fc-bb59-edba8e44562d", 
    "userId": 15, 
    "userChainIdentifier": null, 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateUserUpdate", 
    "status": "Failure", 
    "additionalInformation": { 
        "errorCode": 4000, 
        "errorMessage": "User cannot be provisioned on connection." 
    }
}
```

### <a name="create-contract"></a>Créer un contrat

Crée un nouveau contrat.

La requête requiert les champs suivants :

| **Nom**             | **Description**                                                                                                           |
|----------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId            | GUID fourni par le client |
| userChainIdentifier  | Adresse de l’utilisateur qui a été créé sur le réseau blockchain. Dans Ethereum, il s’agit de l’adresse **on-chain** de l’utilisateur. |
| applicationName      | Nom de l’application |
| version              | Version de l'application. Obligatoire si plusieurs versions de l'application sont activées. Sinon, la version est facultative. Pour plus d'informations sur les versions des applications, consultez [Versions des applications Azure Blockchain Workbench](version-app.md). |
| workflowName         | Nom du flux de travail |
| parameters           | Entrée de paramètres pour la création de contrat |
| connectionId         | Identificateur unique de la connexion blockchain |
| messageSchemaVersion | Version du schéma de messagerie |
| messageName          | **CreateContractRequest** |

Exemple :

``` json
{ 
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211", 
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398", 
    "applicationName": "AssetTransfer",
    "version": "1.0",
    "workflowName": "AssetTransfer", 
    "parameters": [ 
        { 
            "name": "description", 
            "value": "a 1969 dodge charger" 
        }, 
        { 
            "name": "price", 
            "value": "12345" 
        } 
    ], 
    "connectionId": 1, 
    "messageSchemaVersion": "1.0.0", 
    "messageName": "CreateContractRequest" 
}
```

Blockchain Workbench retourne une réponse avec les champs suivants :

| **Nom**                 | **Description**                                                                   |
|--------------------------|-----------------------------------------------------------------------------------|
| requestId                | GUID fourni par le client                                                             |
| contractId               | Identificateur unique du contrat dans Azure Blockchain Workbench |
| contractLedgerIdentifier | Adresse du contrat sur le registre                                            |
| connectionId             | Identificateur unique de la connexion blockchain                               |
| messageSchemaVersion     | Version du schéma de messagerie                                                         |
| messageName              | **CreateContractUpdate**                                                      |
| status                   | État de la demande de création de contrat.  Valeurs possibles : **Envoyé**, **Validé**, **Échec**.  |
| additionalInformation    | Informations supplémentaires fournies en fonction de l’état                              |

Exemple d’une réponse **create contract** envoyée à partir de Blockchain Workbench :

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Submitted",
    "additionalInformation": { }
}
```

Exemple d’une réponse **create contract** validée à partir de Blockchain Workbench :

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Committed",
    "additionalInformation": { }
}
```

Si la demande a échoué, les détails sur l’échec sont inclus dans les informations supplémentaires.

``` json
{
    "requestId": "ce3c429b-a091-4baa-b29b-5b576162b211",
    "contractId": 55,
    "contractLedgerIdentifier": null,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractUpdate",
    "status": "Failure",
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract cannot be provisioned on connection."
    }
}
```

### <a name="create-contract-action"></a>Créer une action de contrat

Crée une nouvelle action de contrat.

La requête requiert les champs suivants :

| **Nom**                 | **Description**                                                                                                           |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------|
| requestId                | GUID fourni par le client |
| userChainIdentifier      | Adresse de l’utilisateur qui a été créé sur le réseau blockchain. Dans Ethereum, il s’agit de l’adresse **on-chain** de l’utilisateur. |
| contractLedgerIdentifier | Adresse du contrat sur le registre |
| version                  | Version de l'application. Obligatoire si plusieurs versions de l'application sont activées. Sinon, la version est facultative. Pour plus d'informations sur les versions des applications, consultez [Versions des applications Azure Blockchain Workbench](version-app.md). |
| workflowFunctionName     | Nom de la fonction du flux de travail |
| parameters               | Entrée de paramètres pour la création de contrat |
| connectionId             | Identificateur unique de la connexion blockchain |
| messageSchemaVersion     | Version du schéma de messagerie |
| messageName              | **CreateContractActionRequest** |

Exemple :

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "userChainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f248398",
    "contractLedgerIdentifier": "0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe",
    "version": "1.0",
    "workflowFunctionName": "modify",
    "parameters": [
        {
            "name": "description",
            "value": "a 1969 dodge charger"
        },
        {
            "name": "price",
            "value": "12345"
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionRequest"
}
```

Blockchain Workbench retourne une réponse avec les champs suivants :

| **Nom**              | **Description**                                                                   |
|-----------------------|-----------------------------------------------------------------------------------|
| requestId             | GUID fourni par le client|
| contractId            | Identificateur unique du contrat dans Azure Blockchain Workbench |
| connectionId          | Identificateur unique de la connexion blockchain |
| messageSchemaVersion  | Version du schéma de messagerie |
| messageName           | **CreateContractActionUpdate** |
| status                | État de la demande d’action de contrat. Valeurs possibles : **Envoyé**, **Validé**, **Échec**.                         |
| additionalInformation | Informations supplémentaires fournies en fonction de l’état |

Exemple d’une réponse **create contract action** envoyée à partir de Blockchain Workbench :

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Submitted",
    "additionalInformation": { }
}
```

Exemple d’une réponse **create contract action** validée à partir de Blockchain Workbench :

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Committed",
    "additionalInformation": { }
}
```

Si la demande a échoué, les détails sur l’échec sont inclus dans les informations supplémentaires.

``` json
{
    "requestId": "a5530932-9d6b-4eed-8623-441a647741d3",
    "contractId": 105,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "CreateContractActionUpdate",
    "status": "Failure",
    "additionalInformation": {
        "errorCode": 4000,
        "errorMessage": "Contract action cannot be provisioned on connection."
    }
}
```

### <a name="input-api-error-codes-and-messages"></a>Messages et codes d’erreur d’API d’entrée

**Code d’erreur 4000 : Erreur de demande incorrecte**
- connectionId non valide
- Échec de la désérialisation de CreateUserRequest
- Échec de la désérialisation de CreateContractRequest
- Échec de la désérialisation de CreateContractActionRequest
- L’application {identifiée par le nom de l’application} n’existe pas
- L’application {identifiée par le nom de l’application} n’a pas de flux de travail
- UserChainIdentifier n’existe pas
- Le contrat {identifié par l’identificateur du registre} n’existe pas
- Le contrat {identifié par l’identificateur du registre} n’a pas de fonction {nom de fonction de flux de travail}
- UserChainIdentifier n’existe pas

**Code d’erreur 4090 : Erreur de conflit**
- L’utilisateur existe déjà
- Le contrat existe déjà
- L’action de contrat existe déjà

**Code d’erreur 5000 : Erreur interne du serveur**
- Messages d’exception

## <a name="event-notifications"></a>Notifications d’événement

Vous pouvez utiliser des notifications d’événements pour informer les utilisateurs et les systèmes en aval des événements qui se produisent dans Blockchain Workbench et sur le réseau blockchain auquel il est connecté. Les notifications d’événements peuvent être utilisées directement dans le code ou avec des outils comme Logic Apps et Flow pour déclencher des flux de données vers des systèmes en aval.

Consultez la section [Référence sur les messages de notification](#notification-message-reference) pour plus d’informations sur les différents messages qui peuvent être reçus.

### <a name="consuming-event-grid-events-with-azure-functions"></a>Consommation d’événements Event Grid avec Azure Functions

Si un utilisateur souhaite utiliser Event Grid pour être informé des événements qui se produisent dans Blockchain Workbench, vous pouvez utiliser des événements d’Event Grid à l’aide d’Azure Functions.

1. Créez une **application Azure Function** dans le portail Azure
2. Créez une fonction.
3. Recherchez le modèle pour Event Grid. Le code du modèle de base pour lire le message est affiché. Modifiez le code en fonction de vos besoins.
4. Enregistrez la fonction. 
5. Sélectionnez Event Grid dans le groupe de ressources de Blockchain Workbench.

### <a name="consuming-event-grid-events-with-logic-apps"></a>Consommation d’événements Event Grid avec Logic Apps

1. Créez une **application logique Azure** dans le portail Azure.
2. Lorsque vous ouvrez l’application logique Azure dans le portail, vous êtes invité à sélectionner un déclencheur. Sélectionnez **Azure Event Grid--When a resource event occurs** (Azure Event Grid--Quand un événement de ressource se produit).
3. Lorsque le Concepteur de flux de travail s’affiche, vous êtes invité à vous connecter.
4. Sélectionnez l’abonnement. Ressource comme **Microsoft.EventGrid.Topics**. Sélectionnez le **nom de la ressource** à partir du nom de la ressource du groupe de ressources Azure Blockchain Workbench.
5. Sélectionnez l’événement Event Grid dans le groupe de ressources de Blockchain Workbench.

## <a name="using-service-bus-topics-for-notifications"></a>Utilisation des rubriques Service Bus pour les notifications

Des rubriques Service Bus peuvent être utilisées pour informer les utilisateurs des événements qui se produisent dans Blockchain Workbench. 

1. Accédez à Service Bus dans le groupe de ressources de Workbench.
2. Sélectionnez **Rubriques**.
3. Sélectionnez **egress-topic**.
4. Créez un abonnement à cette rubrique. Récupérez une clé pour celle-ci.
5. Créez un programme qui s’abonne aux événements de cet abonnement.

### <a name="consuming-service-bus-messages-with-logic-apps"></a>Consommation des messages Service Bus avec Logic Apps

1. Créez une **application logique Azure** dans le portail Azure.
2. Lorsque vous ouvrez l’application logique Azure dans le portail, vous êtes invité à sélectionner un déclencheur. Tapez **Service Bus** dans la zone de recherche et sélectionnez le déclencheur approprié pour le type d’interaction que vous souhaitez avoir avec Service Bus. Par exemple, **Service Bus -- Quand une rubrique abonnement reçoit un message (saisie semi-automatique)** .
3. Lorsque le Concepteur de flux de travail s’affiche, spécifiez les informations de connexion à Service Bus.
4. Sélectionnez votre abonnement et spécifiez la rubrique de **workbench-external**.
5. Développez la logique de votre application qui utilise le message à partir de ce déclencheur.

## <a name="notification-message-reference"></a>Référence sur les messages de notification

En fonction de **messageName**, les messages de notification ont l’un des types suivants.

### <a name="block-message"></a>Message de bloc

Contient des informations sur des blocs individuels. *BlockMessage* inclut une section avec des informations de niveau bloc et une section avec des informations sur la transaction.

| Nom | Description |
|------|-------------|
| block | Contient des [informations sur les blocs](#block-information) |
| transactions | Contient une collection d’[informations sur les transactions](#transaction-information) pour le bloc |
| connectionId | Identificateur unique de la connexion |
| messageSchemaVersion | Version du schéma de messagerie |
| messageName | **BlockMessage** |
| additionalInformation | Informations supplémentaires fournies |

#### <a name="block-information"></a>Informations de bloc

| Nom              | Description |
|-------------------|-------------|
| blockId           | Identificateur unique du bloc dans Azure Blockchain Workbench |
| blockNumber       | Identificateur unique pour un bloc sur le registre |
| blockHash         | Code de hachage du bloc |
| previousBlockHash | Hachage du bloc précédent |
| blockTimestamp    | Timestamp du bloc |

#### <a name="transaction-information"></a>Informations sur les transactions

| Nom               | Description |
|--------------------|-------------|
| transactionId      | Identificateur unique de la transaction dans Azure Blockchain Workbench |
| transactionHash    | Code de hachage de la transaction sur le registre |
| de               | Identificateur unique sur le registre pour l’origine de la transaction |
| to                 | Identificateur unique sur le registre pour la destination de la transaction |
| provisioningStatus | Identifie l’état actuel du processus de configuration pour la transaction. Les valeurs possibles sont les suivantes : </br>0 : la transaction a été créée par l’API dans la base de données</br>1 : la transaction a été envoyée au registre</br>2 : la transaction a été validée correctement dans le registre</br>3 ou 4 : échec de la validation de la transaction dans le registre</br>5 : la transaction a été validée correctement dans le registre |

Exemple de *BlockMessage* de Blockchain Workbench :

``` json
{
    "block": {
        "blockId": 123,
        "blockNumber": 1738312,
        "blockHash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
        "previousBlockHash": null,
        "blockTimestamp": "2018-10-09T23:35:58Z",
    },
    "transactions": [
        {
            "transactionId": 234,
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": null,
            "provisioningStatus": 1
        },
        {
            "transactionId": 235,
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xadd97e1e595916e29ea94fda894941574000ffff",
            "to": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff",
            "provisioningStatus": 2
        }
    ],
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "BlockMessage",
    "additionalInformation": {}
}
```

### <a name="contract-message"></a>Message de contrat

Contient des informations sur un contrat. Le message inclut une section avec des propriétés du contrat et une section avec des informations sur les transactions. Toutes les transactions qui ont modifié le contrat pour le bloc particulier sont incluses dans la section des transactions.

| Nom | Description |
|------|-------------|
| blockId | Identificateur unique du bloc dans Azure Blockchain Workbench |
| blockHash | Hachage du bloc |
| modifyingTransactions | [Transactions ayant modifié](#modifying-transaction-information) le contrat |
| contractId | Identificateur unique du contrat dans Azure Blockchain Workbench |
| contractLedgerIdentifier | Identificateur unique du contrat sur le registre |
| contractProperties | [Propriétés du contrat](#contract-properties) |
| isNewContract | Indique si ce contrat a été ou non récemment créé. Les valeurs possibles sont : true - ce contrat était un contrat nouvellement créé. false : ce contrat est une mise à jour du contrat. |
| connectionId | Identificateur unique de la connexion |
| messageSchemaVersion | Version du schéma de messagerie |
| messageName | **ContractMessage** |
| additionalInformation | Informations supplémentaires fournies |

#### <a name="modifying-transaction-information"></a>Modification des informations sur les transactions

| Nom               | Description |
|--------------------|-------------|
| transactionId | Identificateur unique de la transaction dans Azure Blockchain Workbench |
| transactionHash | Code de hachage de la transaction sur le registre |
| de | Identificateur unique sur le registre pour l’origine de la transaction |
| to | Identificateur unique sur le registre pour la destination de la transaction |

#### <a name="contract-properties"></a>Propriétés du contrat

| Nom               | Description |
|--------------------|-------------|
| workflowPropertyId | Identificateur unique de la propriété du workflow dans Azure Blockchain Workbench |
| name | Nom de la propriété du workflow |
| value | Valeur de la propriété du workflow |

Exemple de *ContractMessage* de Blockchain Workbench :

``` json
{
    "blockId": 123,
    "blockhash": "0x03a39411e25e25b47d0ec6433b73b488554a4a5f6b1a253e0ac8a200d13fffff",
    "modifyingTransactions": [
        {
            "transactionId": 234,
            "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07ffff"
        },
        {
            "transactionId": 235,
            "transactionHash": "0xa4d9c95b581f299e41b8cc193dd742ef5a1d3a4ddf97bd11b80d123fec27ffff",
            "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
            "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
        }
    ],
    "contractId": 111,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "contractProperties": [
        {
            "workflowPropertyId": 1,
            "name": "State",
            "value": "0"
        },
        {
            "workflowPropertyId": 2,
            "name": "Description",
            "value": "1969 Dodge Charger"
        },
        {
            "workflowPropertyId": 3,
            "name": "AskingPrice",
            "value": "30000"
        },
        {
            "workflowPropertyId": 4,
            "name": "OfferPrice",
            "value": "0"
        },
        {
            "workflowPropertyId": 5,
            "name": "InstanceAppraiser",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 6,
            "name": "InstanceBuyer",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 7,
            "name": "InstanceInspector",
            "value": "0x0000000000000000000000000000000000000000"
        },
        {
            "workflowPropertyId": 8,
            "name": "InstanceOwner",
            "value": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
        },
        {
            "workflowPropertyId": 9,
            "name": "ClosingDayOptions",
            "value": "[21,48,69]"
        }
    ],
    "isNewContract": false,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "ContractMessage",
    "additionalInformation": {}
}
```

### <a name="event-message-contract-function-invocation"></a>Message d’événement : Appel de fonction du contrat

Contient des informations indiquant quand une fonction de contrat est appelée, comme le nom de la fonction, les paramètres en entrée et l’appelant de la fonction.

| Nom | Description |
|------|-------------|
| eventName                   | **ContractFunctionInvocation** |
| caller                      | [Informations de l’appelant](#caller-information) |
| contractId                  | Identificateur unique du contrat dans Azure Blockchain Workbench |
| contractLedgerIdentifier    | Identificateur unique du contrat sur le registre |
| functionName                | Nom de la fonction |
| parameters                  | [Informations sur les paramètres](#parameter-information) |
| transaction                 | Informations sur les transactions |
| inTransactionSequenceNumber | Numéro de séquence de la transaction dans le bloc |
| connectionId                | Identificateur unique de la connexion |
| messageSchemaVersion        | Version du schéma de messagerie |
| messageName                 | **EventMessage** |
| additionalInformation       | Informations supplémentaires fournies |

#### <a name="caller-information"></a>Informations de l’appelant

| Nom | Description |
|------|-------------|
| type | Type de l’appelant, comme un utilisateur ou un contrat |
| id | Identificateur unique de l’appelant dans Azure Blockchain Workbench |
| ledgerIdentifier | Identificateur unique pour l’appelant sur le registre |

#### <a name="parameter-information"></a>Informations sur les paramètres

| Nom | Description |
|------|-------------|
| name | Nom du paramètre |
| value | Valeur du paramètre |

#### <a name="event-message-transaction-information"></a>Informations sur les transactions des messages d’événement

| Nom               | Description |
|--------------------|-------------|
| transactionId      | Identificateur unique de la transaction dans Azure Blockchain Workbench |
| transactionHash    | Code de hachage de la transaction sur le registre |
| de               | Identificateur unique sur le registre pour l’origine de la transaction |
| to                 | Identificateur unique sur le registre pour la destination de la transaction |

Exemple de *EventMessage ContractFunctionInvocation* de Blockchain Workbench :

``` json
{
    "eventName": "ContractFunctionInvocation",
    "caller": {
        "type": "User",
        "id": 21,
        "ledgerIdentifier": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60ffff"
    },
    "contractId": 34,
    "contractLedgerIdentifier": "0xf8559473b3c7197d59212b401f5a9f07b429ffff",
    "functionName": "Modify",
    "parameters": [
        {
            "name": "description",
            "value": "a new description"
        },
        {
            "name": "price",
            "value": "4567"
        }
    ],
    "transaction": {
        "transactionId": 234,
        "transactionHash": "0x5c1fddea83bf19d719e52a935ec8620437a0a6bdaa00ecb7c3d852cf92e1ffff",
        "from": "0xd85e7262dd96f3b8a48a8aaf3dcdda90f60dffff",
        "to": "0xf8559473b3c7197d59212b401f5a9f07b429ffff"
    },
    "inTransactionSequenceNumber": 1,
    "connectionId": 1,
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

### <a name="event-message-application-ingestion"></a>Message d’événement : Ingestion d’application

Contient des informations indiquant quand une application est chargée sur Workbench, comme le nom et la version de l’application chargée.

| Nom | Description |
|------|-------------|
| eventName | **ApplicationIngestion** |
| applicationId | Identificateur unique pour l’application dans Azure Blockchain Workbench |
| applicationName | Nom de l'application |
| applicationDisplayName | Nom d’affichage de l’application |
| applicationVersion | Version de l’application |
| applicationDefinitionLocation | URL de l’emplacement du fichier de configuration de l’application |
| contractCodes | Collection de [codes de contrat](#contract-code-information) pour l’application |
| applicationRoles | Collection de [rôles de contrat](#application-role-information) pour l’application |
| applicationWorkflows | Collection de [workflows de contrat](#application-workflow-information) pour l’application |
| connectionId | Identificateur unique de la connexion |
| messageSchemaVersion | Version du schéma de messagerie |
| messageName | **EventMessage** |
| additionalInformation | Les informations supplémentaires fournies ici incluent les états des workflows et les informations de transition de l’application. |

#### <a name="contract-code-information"></a>Informations de code de contrat

| Nom | Description |
|------|-------------|
| id | Identificateur unique du fichier de code de contrat dans Azure Blockchain Workbench |
| ledgerId | Identificateur unique du registre dans Azure Blockchain Workbench |
| location | URL de l’emplacement du fichier de code de contrat |

#### <a name="application-role-information"></a>Informations des rôles de l’application

| Nom | Description |
|------|-------------|
| id | Identificateur unique pour le rôle d’application dans Azure Blockchain Workbench |
| name | Nom du rôle d’application |

#### <a name="application-workflow-information"></a>Informations de workflow d’application

| Nom | Description |
|------|-------------|
| id | Identificateur unique pour le workflow d’application dans Azure Blockchain Workbench |
| name | Nom du workflow d’application |
| displayName | Nom d’affichage du workflow d’application |
| functions | Collection de [fonctions pour le workflow d’application](#workflow-function-information)|
| unis | Collection d’[états pour le workflow d’application](#workflow-state-information) |
| properties | [Informations des propriétés de workflow](#workflow-property-information) d’application |

##### <a name="workflow-function-information"></a>Informations de fonction de workflow

| Nom | Description |
|------|-------------|
| id | Identificateur unique pour la fonction de workflow d’application dans Azure Blockchain Workbench |
| name | Nom de la fonction |
| parameters | Paramètres pour la fonction |

##### <a name="workflow-state-information"></a>Informations d’état du workflow

| Nom | Description |
|------|-------------|
| name | Nom de l’état |
| displayName | Nom d’affichage de l’état |
| style | Style d’état (réussite ou échec) |

##### <a name="workflow-property-information"></a>Informations de propriété de workflow

| Nom | Description |
|------|-------------|
| id | Identificateur unique pour la propriété de workflow d’application dans Azure Blockchain Workbench |
| name | Nom de la propriété |
| type | Type de propriété |

Exemple de *EventMessage ApplicationIngestion* de Blockchain Workbench :

``` json
{
    "eventName": "ApplicationIngestion",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": "1.0",
    "applicationDefinitionLocation": "http://url",
    "contractCodes": [
        {
            "id": 23,
            "ledgerId": 1,
            "location": "http://url"
        }
    ],
    "applicationRoles": [
            {
                "id": 134,
                "name": "Buyer"
            },
            {
                "id": 135,
                "name": "Seller"
            }
       ],
    "applicationWorkflows": [
        {
            "id": 89,
            "name": "AssetTransfer",
            "displayName": "Asset Transfer",
            "functions": [
                {
                    "id": 912,
                    "name": "",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                },
                {
                    "id": 913,
                    "name": "modify",
                    "parameters": [
                        {
                            "name": "description",
                            "type": {
                                "name": "string"
                             }
                        },
                        {
                            "name": "price",
                            "type": {
                                "name": "int"
                            }
                        }
                    ]
                }
            ],
            "states": [ 
                 {
                      "name": "Created",
                      "displayName": "Created",
                      "style" : "Success"
                 },
                 {
                      "name": "Terminated",
                      "displayName": "Terminated",
                      "style" : "Failure"
                 }
            ],
            "properties": [
                {
                    "id": 879,
                    "name": "Description",
                    "type": {
                        "name": "string"
                     }
                },
                {
                    "id": 880,
                    "name": "Price",
                    "type": {
                        "name": "int"
                     }
                }
            ]
        }
    ],
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation":
        {
            "states" :
            [
                {
                    "Name": "BuyerAccepted",
                    "Transitions": [
                        {
                            "DisplayName": "Accept",
                            "AllowedRoles": [ ],
                            "AllowedInstanceRoles": [ "InstanceOwner" ],
                            "Function": "Accept",
                            "NextStates": [ "SellerAccepted" ]
                        }
                    ]
                }
            ]
        }
}
```

### <a name="event-message-role-assignment"></a>Message d’événement : Attribution de rôle

Contient des informations indiquant quand un rôle est attribué à un utilisateur dans Workbench, comme la personne ayant effectué l’attribution de rôle, et le nom du rôle et de l’application correspondante.

| Nom | Description |
|------|-------------|
| eventName | **RoleAssignment** |
| applicationId | Identificateur unique pour l’application dans Azure Blockchain Workbench |
| applicationName | Nom de l'application |
| applicationDisplayName | Nom d’affichage de l’application |
| applicationVersion | Version de l’application |
| applicationRole        | Informations sur le [rôle d’application](#roleassignment-application-role) |
| assigner               | Informations sur la [personne ayant effectué l’attribution](#roleassignment-assigner) |
| assignee               | Informations sur la [personne ayant reçu l’attribution](#roleassignment-assignee) |
| connectionId           | Identificateur unique de la connexion |
| messageSchemaVersion   | Version du schéma de messagerie |
| messageName            | **EventMessage** |
| additionalInformation  | Informations supplémentaires fournies |

#### <a name="roleassignment-application-role"></a>Rôle d’application RoleAssignment

| Nom | Description |
|------|-------------|
| id | Identificateur unique pour le rôle d’application dans Azure Blockchain Workbench |
| name | Nom du rôle d’application |

#### <a name="roleassignment-assigner"></a>Personne ayant effectué l’attribution RoleAssignment

| Nom | Description |
|------|-------------|
| id | Identificateur unique de l’utilisateur dans Azure Blockchain Workbench |
| type | Type de personne ayant effectué l’attribution |
| chainIdentifier | Identificateur unique de l’utilisateur sur le registre |

#### <a name="roleassignment-assignee"></a>Personne ayant reçu l’attribution RoleAssignment

| Nom | Description |
|------|-------------|
| id | Identificateur unique de l’utilisateur dans Azure Blockchain Workbench |
| type | Type de la personne ayant reçu l’attribution |
| chainIdentifier | Identificateur unique de l’utilisateur sur le registre |

Exemple de *EventMessage RoleAssignment* de Blockchain Workbench :

``` json
{
    "eventName": "RoleAssignment",
    "applicationId": 31,
    "applicationName": "AssetTransfer",
    "applicationDisplayName": "Asset Transfer",
    "applicationVersion": "1.0",
    "applicationRole": {
        "id": 134,
        "name": "Buyer"
    },
    "assigner": {
        "id": 1,
        "type": null,
        "chainIdentifier": "0xeFFC7766d38aC862d79706c3C5CEEf089564ffff"
    },
    "assignee": {
        "id": 3,
        "type": null,
        "chainIdentifier": "0x9a8DDaCa9B7488683A4d62d0817E965E8f24ffff"
    },
    "connectionId": [ ],
    "messageSchemaVersion": "1.0.0",
    "messageName": "EventMessage",
    "additionalInformation": { }
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Modèles d’intégration de contrat intelligent](integration-patterns.md)
