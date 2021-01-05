---
title: Créer une application blockchain - Azure Blockchain Workbench
description: Préversion du tutoriel sur la création d’une application blockchain pour Azure Blockchain Workbench.
ms.date: 08/24/2020
ms.topic: tutorial
ms.reviewer: ravastra
ms.openlocfilehash: add790a069d2e0ea66d84bbd632825cf9331fd38
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88784026"
---
# <a name="tutorial-create-a-blockchain-application-for-azure-blockchain-workbench"></a>Tutoriel : Créer une application blockchain pour Azure Blockchain Workbench

Vous pouvez utiliser Azure Blockchain Workbench pour créer des applications blockchain qui représentent des flux de travail à plusieurs parties définis par une configuration et du code de contrat intelligent.

Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer une application blockchain
> * Créer un fichier de code de contrat intelligent
> * Ajouter une application blockchain à Blockchain Workbench
> * Ajouter des membres à l’application blockchain

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Un déploiement Blockchain Workbench. Pour plus d’informations, consultez [Azure Blockchain Workbench deployment](deploy.md) (Déploiement d’Azure Blockchain Workbench) pour plus de détails sur le déploiement.
* Des utilisateurs Azure Active Directory dans le locataire associé à Blockchain Workbench. Pour plus d’informations, consultez [add Azure AD users in Azure Blockchain Workbench](manage-users.md#add-azure-ad-users) (Ajouter des utilisateurs Azure AD dans Azure Blockchain Workbench).
* Un compte d’administrateur Blockchain Workbench. Pour plus d’informations, consultez [add Blockchain Workbench administrators in Azure Blockchain Workbench](manage-users.md#manage-blockchain-workbench-administrators) (Ajouter des administrateurs Blockchain Workbench dans Azure Blockchain Workbench).

## <a name="hello-blockchain"></a>Hello, Blockchain !

Générons une application de base dans laquelle un demandeur envoie une requête et un répondeur envoie une réponse à la requête.
Par exemple, une requête peut être : « Hello, comment allez-vous ? », et la réponse peut être : « Je vais très bien ! ». La requête et la réponse sont enregistrées sur le blockchain sous-jacent.

Suivez les étapes pour créer les fichiers d’application, ou vous pouvez [télécharger l’exemple à partir de GitHub](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/hello-blockchain).

## <a name="configuration-file"></a>Fichier de configuration

Les métadonnées de configuration définissent les workflows généraux et le modèle d’interaction de l’application blockchain. Les métadonnées de configuration représentent les étapes de flux de travail et le modèle d’interaction de l’application blockchain. Pour plus d’informations sur le contenu des fichiers de configuration, consultez [Azure Blockchain Workflow configuration reference](configuration.md) (Référence de configuration de flux de travail Azure Blockchain).

1. Dans votre éditeur favori, créez un fichier nommé `HelloBlockchain.json`.
2. Ajoutez le texte JSON suivant pour définir la configuration de l’application blockchain.

    ``` json
    {
      "ApplicationName": "HelloBlockchain",
      "DisplayName": "Hello, Blockchain!",
      "Description": "A simple application to send request and get response",
      "ApplicationRoles": [
        {
          "Name": "Requestor",
          "Description": "A person sending a request."
        },
        {
          "Name": "Responder",
          "Description": "A person responding to a request"
        }
      ],
      "Workflows": [
        {
          "Name": "HelloBlockchain",
          "DisplayName": "Request Response",
          "Description": "A simple workflow to send a request and receive a response.",
          "Initiators": [ "Requestor" ],
          "StartState": "Request",
          "Properties": [
            {
              "Name": "State",
              "DisplayName": "State",
              "Description": "Holds the state of the contract.",
              "Type": {
                "Name": "state"
              }
            },
            {
              "Name": "Requestor",
              "DisplayName": "Requestor",
              "Description": "A person sending a request.",
              "Type": {
                "Name": "Requestor"
              }
            },
            {
              "Name": "Responder",
              "DisplayName": "Responder",
              "Description": "A person sending a response.",
              "Type": {
                "Name": "Responder"
              }
            },
            {
              "Name": "RequestMessage",
              "DisplayName": "Request Message",
              "Description": "A request message.",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "ResponseMessage",
              "DisplayName": "Response Message",
              "Description": "A response message.",
              "Type": {
                "Name": "string"
              }
            }
          ],
          "Constructor": {
            "Parameters": [
              {
                "Name": "message",
                "Description": "...",
                "DisplayName": "Request Message",
                "Type": {
                  "Name": "string"
                }
              }
            ]
          },
          "Functions": [
            {
              "Name": "SendRequest",
              "DisplayName": "Request",
              "Description": "...",
              "Parameters": [
                {
                  "Name": "requestMessage",
                  "Description": "...",
                  "DisplayName": "Request Message",
                  "Type": {
                    "Name": "string"
                  }
                }
              ]
            },
            {
              "Name": "SendResponse",
              "DisplayName": "Response",
              "Description": "...",
              "Parameters": [
                {
                  "Name": "responseMessage",
                  "Description": "...",
                  "DisplayName": "Response Message",
                  "Type": {
                    "Name": "string"
                  }
                }
              ]
            }
          ],
          "States": [
            {
              "Name": "Request",
              "DisplayName": "Request",
              "Description": "...",
              "PercentComplete": 50,
              "Value": 0,
              "Style": "Success",
              "Transitions": [
                {
                  "AllowedRoles": ["Responder"],
                  "AllowedInstanceRoles": [],
                  "Description": "...",
                  "Function": "SendResponse",
                  "NextStates": [ "Respond" ],
                  "DisplayName": "Send Response"
                }
              ]
            },
            {
              "Name": "Respond",
              "DisplayName": "Respond",
              "Description": "...",
              "PercentComplete": 90,
              "Value": 1,
              "Style": "Success",
              "Transitions": [
                {
                  "AllowedRoles": [],
                  "AllowedInstanceRoles": ["Requestor"],
                  "Description": "...",
                  "Function": "SendRequest",
                  "NextStates": [ "Request" ],
                  "DisplayName": "Send Request"
                }
              ]
            }
          ]
        }
      ]
    }
    ```

3. Enregistrez le fichier `HelloBlockchain.json`.

Le fichier de configuration comporte plusieurs sections. Les détails de chaque section sont les suivants :

### <a name="application-metadata"></a>Métadonnées d’application

Le début du fichier de configuration contient des informations sur l’application, notamment le nom et la description de l’application.

### <a name="application-roles"></a>Rôles d’application

La section des rôles d’application définit les rôles d’utilisateur autorisés à agir ou à participer dans l’application blockchain. Vous définissez un ensemble de rôles distincts basé sur la fonctionnalité. Dans le scénario de requête-réponse, il existe une distinction entre la fonctionnalité d’un demandeur en tant qu’entité formulant des requêtes et d’un répondeur en tant qu’entité formulant des réponses.

### <a name="workflows"></a>Workflows

Les flux de travail définissent une ou plusieurs étapes et actions du contrat. Dans le scénario de requête-réponse, la première étape (état) du flux de travail implique qu’un demandeur (rôle) effectue une action (transition) pour envoyer une requête (fonction). L’étape suivante (état) implique qu’un répondeur (rôle) effectue une action (transition) pour envoyer une réponse (fonction). Le flux de travail d’une application peut impliquer des propriétés, fonctions et états nécessaires pour décrire le flux d’un contrat.

## <a name="smart-contract-code-file"></a>Fichier de code de contrat intelligent

Les contrats intelligents représentent la logique métier de l’application blockchain. Actuellement, Blockchain Workbench prend en charge Ethereum comme registre blockchain. Ethereum utilise [Solidity](https://solidity.readthedocs.io) comme langage de programmation pour l’écriture de la logique métier à application automatique pour les contrats intelligents.

Les contrats intelligents dans Solidity sont semblables aux classes dans les langages orientés objet. Chaque contrat contient un état et des fonctions pour implémenter les étapes et les actions du contrat intelligent.

Dans votre éditeur favori, créez un fichier appelé `HelloBlockchain.sol`.

### <a name="version-pragma"></a>Pragma de version

En tant que meilleure pratique, indiquez la version de Solidity que vous ciblez. Spécifier la version permet d’éviter les problèmes d’incompatibilité avec les versions futures de Solidity.

Ajoutez le pragma de version suivant en haut du fichier de code de contrat intelligent `HelloBlockchain.sol`.

``` solidity
pragma solidity >=0.4.25 <0.6.0;
```

### <a name="configuration-and-smart-contract-code-relationship"></a>Configuration et relation de code de contrat intelligent

Blockchain Workbench utilise le fichier de configuration et le fichier de code de contrat intelligent pour créer une application blockchain. Il existe une relation entre ce qui est défini dans la configuration et le code dans le contrat intelligent. Les détails, fonctions, paramètres et types du contrat doivent correspondre pour créer l’application. Blockchain Workbench vérifie les fichiers avant la création de l’application.

### <a name="contract"></a>Contrat

Ajoutez l’en-tête **contract** à votre fichier de code de contrat intelligent `HelloBlockchain.sol`.

``` solidity
contract HelloBlockchain {
```

### <a name="state-variables"></a>Variables d’état

Les variables d’état stockent les valeurs d’état de chaque instance de contrat. Les variables d’état dans votre contrat doivent correspondre aux propriétés de flux de travail définies dans le fichier de configuration.

Ajoutez les variables d’état à votre contrat dans votre fichier de code de contrat intelligent `HelloBlockchain.sol`.

``` solidity
    //Set of States
    enum StateType { Request, Respond}

    //List of properties
    StateType public  State;
    address public  Requestor;
    address public  Responder;

    string public RequestMessage;
    string public ResponseMessage;
```

### <a name="constructor"></a>Constructeur

Le constructeur définit des paramètres d’entrée pour une nouvelle instance de contrat intelligent d’un flux de travail. Les paramètres requis pour le constructeur sont définis en tant que paramètres du constructeur dans le fichier de configuration. Le nombre, l’ordre et le type des paramètres doivent correspondre dans les deux fichiers.

Dans la fonction de constructeur, écrivez une logique métier à exécuter avant la création du contrat. Par exemple, initialisez les variables d’état avec des valeurs de départ.

Ajoutez la fonction de constructeur à votre contrat dans votre fichier de code de contrat intelligent `HelloBlockchain.sol`.

``` solidity
    // constructor function
    constructor(string memory message) public
    {
        Requestor = msg.sender;
        RequestMessage = message;
        State = StateType.Request;
    }
```

### <a name="functions"></a>Fonctions

Les fonctions sont les unités exécutables de logique métier dans un contrat. Les paramètres requis pour la fonction sont définis en tant que paramètres de fonction dans le fichier de configuration. Le nombre, l’ordre et le type des paramètres doivent correspondre dans les deux fichiers. Les fonctions sont associées à des transitions dans un flux de travail Blockchain Workbench dans le fichier de configuration. Une transition est une action effectuée pour passer à l’étape suivante du flux de travail d’une application tel que déterminé par le contrat.

Écrivez une logique métier à exécuter dans la fonction. Par exemple, la modification de la valeur d’une variable d’état.

1. Ajoutez les fonctions suivantes à votre contrat dans votre fichier de code de contrat intelligent `HelloBlockchain.sol`.

    ``` solidity
        // call this function to send a request
        function SendRequest(string memory requestMessage) public
        {
            if (Requestor != msg.sender)
            {
                revert();
            }
    
            RequestMessage = requestMessage;
            State = StateType.Request;
        }
    
        // call this function to send a response
        function SendResponse(string memory responseMessage) public
        {
            Responder = msg.sender;
    
            ResponseMessage = responseMessage;
            State = StateType.Respond;
        }
    }
    ```

2. Enregistrez votre fichier de code de contrat intelligent `HelloBlockchain.sol`.

## <a name="add-blockchain-application-to-blockchain-workbench"></a>Ajouter une application blockchain à Blockchain Workbench

Pour ajouter une application blockchain à Blockchain Workbench, vous chargez les fichiers de configuration et de contrat intelligent pour définir l’application.

1. Dans un navigateur web, accédez à l’adresse web de Blockchain Workbench. Par exemple, `https://{workbench URL}.azurewebsites.net/`. L’application web est créée lorsque vous déployez Blockchain Workbench. Pour plus d’informations sur la façon de trouver l’adresse web de Blockchain Workbench, consultez [URL Web Blockchain Workbench](deploy.md#blockchain-workbench-web-url)
2. Connectez-vous en tant [qu’administrateur Blockchain Workbench](manage-users.md#manage-blockchain-workbench-administrators).
3. Sélectionnez **Applications** > **Nouveau**. Le volet **Nouvelle application** s’affiche.
4. Sélectionnez **Upload the contract configuration (Charger la configuration du contrat)**  > **Parcourir** pour localiser le fichier de configuration **HelloBlockchain.json** que vous avez créé. Le fichier de configuration est automatiquement validé. Sélectionnez le lien **Afficher** pour afficher les erreurs de validation. Corrigez les erreurs de validation avant de déployer l’application.
5. Sélectionnez **Upload the contract code (Charger le code de contrat)**  > **Parcourir** pour localiser le fichier de code de contrat intelligent **HelloBlockchain.sol**. Le fichier de code est automatiquement validé. Sélectionnez le lien **Afficher** pour afficher les erreurs de validation. Corrigez les erreurs de validation avant de déployer l’application.
6. Sélectionnez **Déployer** pour créer l’application blockchain en fonction des fichiers de configuration et de contrat intelligent.

Le déploiement de l’application blockchain prend quelques minutes. Lorsque le déploiement est terminé, la nouvelle application s’affiche dans **Applications**. 

> [!NOTE]
> Vous pouvez également créer des applications blockchain à l’aide de l’[API REST Azure Blockchain Workbench](/rest/api/azure-blockchain-workbench).

## <a name="add-blockchain-application-members"></a>Ajouter des membres d’application blockchain

Ajoutez des membres d’application à votre application pour initier et effectuer des actions sur des contrats. Pour ajouter des membres d’application, vous devez être un [administrateur Blockchain Workbench](manage-users.md#manage-blockchain-workbench-administrators).

1. Sélectionnez **Applications** > **Hello, Blockchain!** .
2. Le nombre de membres associés à l’application s’affiche dans le coin supérieur droit de la page. Pour une nouvelle application, le nombre de membres sera de zéro.
3. Sélectionnez le lien **membres** dans le coin supérieur droit de la page. La liste actuelle des membres de l’application s’affiche.
4. Dans la liste des membres, sélectionnez **Ajouter des membres**.
5. Sélectionnez ou entrez le nom du membre à ajouter. Seuls les utilisateurs Azure AD existants dans le client Blockchain Workbench sont répertoriés. Si l’utilisateur est introuvable, vous devez [ajouter des utilisateurs Azure AD](manage-users.md#add-azure-ad-users).
6. Sélectionnez le **Rôle** du membre. Pour le premier membre, sélectionnez le rôle **Demandeur**.
7. Sélectionnez **Ajouter** pour ajouter le membre avec le rôle associé à l’application.
8. Ajoutez un autre membre à l’application avec le rôle **Répondeur**.

Pour plus d’informations sur la gestion des utilisateurs dans Blockchain Workbench, consultez [managing users in Azure Blockchain Workbench](manage-users.md) (Gestion des utilisateurs dans Azure Blockchain Workbench)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article sur les procédures, vous avez créé une application de demande et de réponse de base. Pour apprendre à utiliser l’application, passez à l’article suivant.

> [!div class="nextstepaction"]
> [Utilisation d’une application blockchain](use.md)
