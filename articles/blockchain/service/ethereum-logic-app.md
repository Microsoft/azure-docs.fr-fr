---
title: Utiliser le connecteur Ethereum Blockchain avec Azure Logic Apps - Azure Blockchain Service
description: Utilisez le connecteur Ethereum Blockchain avec Azure Logic Apps pour déclencher des fonctions de contrat intelligent et répondre à des événements de contrat intelligent.
ms.date: 08/31/2020
ms.topic: how-to
ms.reviewer: caleteet
ms.openlocfilehash: 411337908553e58c252a0ed1a42d17f76195c720
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763789"
---
# <a name="use-the-ethereum-blockchain-connector-with-azure-logic-apps"></a>Utiliser le connecteur Ethereum Blockchain avec Azure Logic Apps

Utilisez le [connecteur Ethereum Blockchain](/connectors/blockchainethereum/) avec [Azure Logic Apps](../../logic-apps/index.yml) pour déclencher des fonctions de contrat intelligent et répondre à des événements de contrat intelligent. Cet article explique comment vous utiliser le connecteur Ethereum Blockchain pour envoyer des informations de blockchain à un autre service ou appeler une fonction de blockchain. Par exemple, supposons que vous souhaitiez créer un microservice basé sur REST qui retourne des informations à partir d’un registre blockchain. À l’aide d’une application logique, vous pouvez accepter les requêtes HTTP qui interrogent les informations stockées dans un registre blockchain.

## <a name="prerequisites"></a>Prérequis

- Effectuez le guide de démarrage rapide prérequis facultatif intitulé [Démarrage rapide : Utiliser Visual Studio Code pour se connecter à un réseau du consortium Azure Blockchain Service](connect-vscode.md). Le guide de démarrage rapide vous dirige tout au long de l’installation de l’[Azure Blockchain Development Kit for Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) et de la configuration de votre environnement de développement blockchain.
- Si vous débutez avec Azure Logic Apps, songez à consulter les modules Microsoft Learn [Présentation d’Azure Logic Apps](/learn/modules/intro-to-logic-apps/) et [Appeler une API à partir d’un flux de travail Logic Apps à l’aide d’un connecteur personnalisé](/learn/modules/logic-apps-and-custom-connectors/).

## <a name="create-a-logic-app"></a>Créer une application logique

Azure Logic Apps vous aide à planifier et à automatiser les processus et workflows métier quand vous avez besoin d’intégrer des systèmes et des services. Tout d’abord, vous créez une logique qui utilise le connecteur Ethereum Blockchain.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Intégration** > **Application logique**.
1. Sous **Créer une application logique**, indiquez où vous souhaitez créer votre application logique. Sélectionnez **Créer** quand vous avez terminé.

    Pour plus d’informations sur la création d’applications logiques, consultez [Créer votre premier workflow automatisé avec Azure Logic Apps](../../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Une fois qu’Azure a déployé votre application, sélectionnez votre ressource d’application logique.
1. Dans le Concepteur Logic Apps, sous **Modèles**, sélectionnez **Application logique vide**.

Chaque application logique doit démarrer avec un déclencheur, qui s’active lorsqu’un événement spécifique se produit ou lorsqu’une condition particulière est remplie. Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une instance d’application logique qui démarre et exécute votre flux de travail.

Le connecteur Ethereum Blockchain a un déclencheur et plusieurs actions. Le déclencheur ou l’action que vous utilisez dépend de votre scénario. Suivez la section de cet article qui correspond le mieux à votre scénario.

Si votre workflow :

* Se déclenche quand un événement se produit sur la blockchain : [Utiliser le déclencheur d’événements](#use-the-event-trigger).
* Interroge ou déploie un contrat intelligent : [Utiliser des actions](#use-actions).
* Suit un scénario courant : [Générer un workflow à l’aide du kit de développement](#generate-a-workflow).

## <a name="use-the-event-trigger"></a>Utiliser le déclencheur d’événements

Utilisez les déclencheurs d’événements Ethereum Blockchain quand vous souhaitez qu’une application logique s’exécute après un événement de contrat intelligent. Par exemple, vous souhaitez envoyer un e-mail quand une fonction de contrat intelligent est appelée.

1. Dans le Concepteur Logic Apps, sélectionnez le connecteur Ethereum Blockchain.
1. Sous l’onglet **Déclencheurs**, sélectionnez **Quand un événement de contrat intelligent se produit**.
1. Modifiez ou [créez une connexion d’API](#create-an-api-connection) à Azure Blockchain Service.
1. Entrez les détails sur le contrat intelligent dont vous souhaitez vérifier les événements.

    ![Concepteur Logic Apps avec les propriétés du déclencheur d’événements](./media/ethereum-logic-app/event-properties.png)

    | Propriété | Description |
    |----------|-------------|
    | **Contract ABI** | L’interface ABI (Application Binary Interface) du contrat définit les interfaces du contrat intelligent. Pour plus d’informations, consultez [Obtenir l’ABI du contrat](#get-the-contract-abi). |
    | **Smart contract address** | L’adresse du contrat est l’adresse de destination du contrat intelligent sur la blockchain Ethereum. Pour plus d’informations, consultez [Obtenir l’adresse du contrat](#get-the-contract-address). |
    | **Event name** | Sélectionnez un événement de contrat intelligent à vérifier. L’événement déclenche l’application logique. |
    | **Interval** et **Frequency** | Sélectionnez la fréquence à laquelle vous souhaitez vérifier l’événement. |

1. Sélectionnez **Enregistrer**.

Pour terminer votre application logique, vous pouvez ajouter une nouvelle étape qui effectue une action basée sur le déclencheur d’événements Ethereum Blockchain. par exemple envoyer un e-mail.

## <a name="use-actions"></a>Utiliser des actions

Utilisez les actions Ethereum Blockchain quand vous souhaitez qu’une application logique effectue une action sur le registre Blockchain. Par exemple, vous souhaitez créer un microservice basé sur REST qui appelle une fonction de contrat intelligent quand une requête HTTP est effectuée sur une application logique.

Les actions de connecteur nécessitent un déclencheur. Vous pouvez utiliser une action de connecteur Ethereum Blockchain comme étape suivante après un déclencheur, par exemple un déclencheur de requête HTTP pour un microservice.

1. Dans le Concepteur Logic Apps, sélectionnez **Nouvelle étape** à la suite d’un déclencheur.
1. Sélectionnez le connecteur Ethereum Blockchain.
1. Sous l’onglet **Actions**, sélectionnez l’une des actions disponibles.

    ![Concepteur Logic Apps avec les propriétés des actions](./media/ethereum-logic-app/action-properties.png)

1. Modifiez ou [créez une connexion d’API](#create-an-api-connection) à Azure Blockchain Service.
1. En fonction de l’action choisie, fournissez les informations suivantes sur votre fonction de contrat intelligent.

    | Propriété | Description |
    |----------|-------------|
    | **Contract ABI** | L’interface ABI du contrat définit les interfaces du contrat intelligent. Pour plus d’informations, consultez [Obtenir l’ABI du contrat](#get-the-contract-abi). |
    | **Contract bytecode** | Bytecode de contrat intelligent compilé. Pour plus d’informations, consultez [Obtenir le bytecode du contrat](#get-the-contract-bytecode). |
    | **Smart contract address** | L’adresse du contrat est l’adresse de destination du contrat intelligent sur la blockchain Ethereum. Pour plus d’informations, consultez [Obtenir l’adresse du contrat](#get-the-contract-address). |
    | **Smart contract function name** | Sélectionnez le nom de la fonction de contrat intelligent pour l’action. La liste est renseignée à partir des détails figurant dans le contrat ABI. |

    Une fois que vous avez sélectionné un nom de fonction de contrat intelligent, il est possible que des champs obligatoires pour les paramètres de fonction apparaissent. Entrez les valeurs ou le contenu dynamique requis pour votre scénario.

Vous pouvez maintenant utiliser votre application logique. Quand l’événement d’application logique est déclenché, l’action Ethereum Blockchain s’exécute. Par exemple, un déclencheur de requête HTTP exécute une action Ethereum Blockchain pour interroger une valeur d’état de contrat intelligent. Cette requête génère une réponse HTTP qui retourne la valeur.

## <a name="generate-a-workflow"></a>Générer un workflow

L’extension Visual Studio Code Azure Blockchain Development Kit for Ethereum peut générer des workflows d’application logique pour les scénarios courants. Quatre scénarios sont disponibles :

* Publication de données sur une instance Azure SQL Database
* Publication d’événements sur une instance d’Azure Event Grid ou d’Azure Service Bus
* Publication de rapports
* Microservice basé sur REST

 Azure Blockchain Development Kit utilise Truffle pour simplifier le développement de blockchain. Pour générer une application logique basée sur un contrat intelligent, vous avez besoin d’une solution Truffle pour le contrat intelligent. Vous avez également besoin d’une connexion à votre réseau de consortium Azure Blockchain Service. Pour plus d’informations, consultez [Démarrage rapide : Utiliser Visual Studio Code pour se connecter à un réseau du consortium Azure Blockchain Service](connect-vscode.md).

Par exemple, les étapes suivantes génèrent une application logique de microservice REST basée sur le contrat intelligent **HelloBlockchain** du guide de démarrage rapide :

1. Dans la barre latérale de l’Explorateur Visual Studio Code, développez le dossier **contracts** dans votre solution.
1. Cliquez avec le bouton droit sur **HelloBlockchain.sol** et sélectionnez **Generate Microservices for Smart Contracts** (Générer des microservices pour les contrats intelligents) dans le menu.

    ![Volet Visual Studio Code avec l’option Générer des microservices pour les contrats intelligents sélectionnée](./media/ethereum-logic-app/generate-logic-app.png)

1. Dans la palette de commandes, sélectionnez **Application logique**.
1. Entrez l’**adresse du contrat**. Pour plus d’informations, consultez [Obtenir l’adresse du contrat](#get-the-contract-address).
1. Sélectionnez l’abonnement Azure et le groupe de ressources pour l’application logique.

    La configuration de l’application logique et les fichiers de code sont générés dans le répertoire **generatedLogicApp**.

1. Affichez le répertoire **generatedLogicApp/HelloBlockchain**. Il existe un fichier JSON d’application logique pour chaque fonction, événement et propriété de contrat intelligent.
1. Ouvrez le fichier **generatedLogicApp/HelloBlockchain/Service/property.RequestMessage.logicapp.json** et copiez le contenu.

    ![Fichier JSON avec le code à copier](./media/ethereum-logic-app/requestmessage.png)

1. Dans votre application logique, sélectionnez **Affichage du code de l’application logique**. Remplacez le JSON existant par le JSON d’application logique généré.

    ![Affichage du code de l’application logique avec le nouveau code d’application remplacé](./media/ethereum-logic-app/code-view.png)

1. Sélectionnez **Concepteur** pour basculer vers le mode Concepteur.
1. L’application logique comprend les étapes de base du scénario. Toutefois, vous devez mettre à jour les détails de configuration du connecteur Ethereum Blockchain.
1. Sélectionnez l’étape **Connexions**, puis modifiez ou [créez une connexion d’API](#create-an-api-connection) à Azure Blockchain Service.

    ![Vue du concepteur avec l’étape Connexions sélectionnée](./media/ethereum-logic-app/microservice-logic-app.png)

1. Vous pouvez maintenant utiliser votre application logique. Pour tester le microservice basé sur REST, émettez une requête HTTP POST à l’URL de requête de l’application logique. Copiez le contenu de **URL HTTP POST** de l’étape **Quand une requête HTTP est reçue**.

    ![Volet du Concepteur Logic Apps avec l’URL HTTP POST](./media/ethereum-logic-app/post-url.png)

1. Utilisez cURL pour créer une requête HTTP POST. Remplacez le texte de l’espace réservé *\<HTTP POST URL\>* par l’URL obtenue à l’étape précédente.

    ``` bash
    curl -d "{}" -H "Content-Type: application/json" -X POST "<HTTP POST URL>"
    ```

    La commande cURL retourne une réponse à partir de l’application logique. En l’occurrence, la réponse est la sortie de la fonction de contrat intelligent **RequestMessage**.

    ![Sortie de code de la fonction de contrat intelligent RequestMessage](./media/ethereum-logic-app/curl.png)

Pour plus d’informations sur l’utilisation du kit de développement, consultez la [page du wiki Azure Blockchain Development Kit for Ethereum](https://github.com/Microsoft/vscode-azure-blockchain-ethereum/wiki).

## <a name="create-an-api-connection"></a>Créer une connexion d’API

Une connexion d’API à une blockchain est nécessaire pour le connecteur Blockchain Ethereum. Vous pouvez utiliser le connecteur d’API pour plusieurs applications logiques. Certaines propriétés sont obligatoires et d’autres dépendent de votre scénario.

> [!IMPORTANT]
> Une clé privée ou une adresse de compte et un mot de passe sont requis pour la création de transactions sur une blockchain. Une seule forme d’authentification est nécessaire. Vous n’avez pas besoin de fournir à la fois la clé privée et les détails du compte. L’interrogation des contrats ne nécessite pas de transaction. Si vous utilisez des actions qui interrogent l’état du contrat, la clé privée ou l’adresse du compte et le mot de passe ne sont pas nécessaires.

Pour vous aider à configurer une connexion à un membre Azure Blockchain Service, la liste suivante indique les propriétés dont vous pouvez avoir besoin en fonction de votre scénario.

| Propriété | Description |
|----------|-------------|
|**Nom de connexion** | Nom de la connexion d’API. Obligatoire. |
|**Ethereum RPC endpoint** | Adresse HTTP du nœud de transaction Azure Blockchain Service. Obligatoire. Pour plus d’informations, consultez [Obtenir le point de terminaison RPC](#get-the-rpc-endpoint). |
|**Private key** | Clé privée du compte Ethereum. Une clé privée ou une adresse de compte et un mot de passe sont requis pour les transactions. Pour plus d’informations, consultez [Obtenir la clé privée](#get-the-private-key). |
|**Adresse du compte** | Adresse du compte membre Azure Blockchain Service. Une clé privée ou une adresse de compte et un mot de passe sont requis pour les transactions. Pour plus d’informations, consultez [Obtenir l’adresse du compte](#get-the-account-address). |
|**Account password** | Le mot de passe du compte est défini lors de la création du membre. Pour plus d’informations sur la réinitialisation du mot de passe, consultez [Compte Ethereum](consortium.md#ethereum-account).|

## <a name="get-the-rpc-endpoint"></a>Obtenir le point de terminaison RPC

L’adresse du point de terminaison RPC Azure Blockchain Service est nécessaire pour se connecter à un réseau Blockchain. Vous pouvez obtenir l’adresse du point de terminaison à l’aide d’Azure Blockchain Development Kit for Ethereum ou du portail Azure.

**Pour utiliser le kit de développement :**

1. Sous **Azure Blockchain Service** dans Visual Studio Code, cliquez avec le bouton droit sur le consortium.
1. Sélectionnez **Copy RPC Endpoint Address** (Copier l’adresse du point de terminaison RPC).

    ![Volet Visual Studio Code présentant le consortium avec l’option Copier l’adresse du point de terminaison RPC sélectionnée](./media/ethereum-logic-app/devkit-rpc.png)

    Le point de terminaison RPC est copié dans le Presse-papiers.

**Pour utiliser le portail Azure :**

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez au membre Azure Blockchain Service. Sélectionnez **Nœuds de transaction**, puis le lien du nœud de transaction par défaut.

    ![Page des nœuds de transaction avec la sélection (nœud par défaut)](./media/ethereum-logic-app/transaction-nodes.png)

1. Sélectionnez **Chaînes de connexion** > **Clés d’accès**.
1. Copiez l’adresse du point de terminaison située sous **HTTPS (Clé d’accès 1)** ou **HTTPS (Clé d’accès 2)** .

    ![Portail Azure avec les clés d’accès à la chaîne de connexion](./media/ethereum-logic-app/connection-string.png)

    Le point de terminaison RPC est l’URL HTTPS, qui comprend l’adresse et la clé d’accès de votre nœud de transaction de membre Azure Blockchain Service.

## <a name="get-the-private-key"></a>Obtenir la clé privée

Vous pouvez utiliser la clé privée du compte Ethereum pour l’authentification lors de l’envoi d’une transaction à la blockchain. Les clés publiques et privées de votre compte Ethereum sont générées à partir d’un mnémonique de 12 mots. Azure Blockchain Development Kit for Ethereum génère un mnémonique quand vous vous connectez à un membre du consortium Azure Blockchain Service. Vous pouvez obtenir l’adresse du point de terminaison à l’aide de l’extension du kit de développement.

1. Dans Visual Studio Code, ouvrez la palette de commandes (F1).
1. Sélectionnez **Blockchain : Retrieve private key** (Récupérer la clé privée).
1. Sélectionnez le mnémonique que vous avez enregistré lors de la connexion au membre du consortium.

    ![Palette de commandes avec une option pour sélectionner le mnémonique](./media/ethereum-logic-app/private-key.png)

    La clé privée est copiée dans le Presse-papiers.

## <a name="get-the-account-address"></a>Obtenir l’adresse du compte

Vous pouvez utiliser le compte et le mot de passe du membre pour l’authentification lors de l’envoi d’une transaction à la blockchain. Le mot de passe est défini lors de la création du membre.

1. Dans le portail Azure, accédez à la page de vue d’ensemble Azure Blockchain Service.
1. Copiez l’adresse du **compte du membre**.

    ![Page de vue d’ensemble avec l’adresse du compte du membre](./media/ethereum-logic-app/member-account.png)

Pour plus d’informations sur l’adresse et le mot de passe du compte, consultez [Compte Ethereum](consortium.md#ethereum-account).

## <a name="get-the-contract-abi"></a>Obtenir l’ABI du contrat

L’interface ABI du contrat définit les interfaces du contrat intelligent. Elle décrit comment interagir avec le contrat intelligent. Vous pouvez obtenir l’ABI du contrat à l’aide d’Azure Blockchain Development Kit for Ethereum. Vous pouvez également l’obtenir à partir du fichier de métadonnées de contrat créé par le compilateur Solidity.

**Pour utiliser le kit de développement :**

Si vous avez utilisé le kit de développement ou Truffle pour créer votre contrat intelligent, vous pouvez utiliser l’extension pour copier l’ABI du contrat dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** de votre projet Solidity.
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Contract ABI** (Copier l’ABI du contrat).

    ![Volet Visual Studio Code avec l’option Copier l’ABI du contrat sélectionnée](./media/ethereum-logic-app/abi-devkit.png)

    L’ABI du contrat est copiée dans le Presse-papiers.

**Pour utiliser le fichier de métadonnées de contrat :**

1. Ouvrez le fichier de métadonnées de contrat contenu dans le dossier **build/contracts** de votre projet Solidity. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Recherchez la section **abi** dans le fichier JSON.
1. Copiez le tableau JSON **abi**.

    ![Code ABI dans le fichier de métadonnées de contrat](./media/ethereum-logic-app/abi-metadata.png)

## <a name="get-the-contract-bytecode"></a>Obtenir le bytecode du contrat

Le bytecode du contrat est le contrat intelligent compilé exécuté par la machine virtuelle Ethereum. Vous pouvez obtenir le bytecode du contrat à l’aide d’Azure Blockchain Development Kit for Ethereum. Vous pouvez également l’obtenir à partir du compilateur Solidity.

**Pour utiliser le kit de développement :**

Si vous avez utilisé le kit de développement ou Truffle pour créer votre contrat intelligent, vous pouvez utiliser l’extension pour copier le bytecode du contrat dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** de votre projet Solidity.
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Contract Bytecode** (Copier le bytecode du contrat).

    ![Volet Visual Studio Code avec l’option Copier le bytecode du contrat sélectionnée](./media/ethereum-logic-app/bytecode-devkit.png)

    Le bytecode du contrat est copié dans le Presse-papiers.

**Pour utiliser le fichier de métadonnées de contrat :**

1. Ouvrez le fichier de métadonnées de contrat contenu dans le dossier **build/contracts** de votre projet Solidity. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Recherchez l’élément **bytecode** dans le fichier JSON.
1. Copiez la valeur **bytecode**.

    ![Volet Visual Studio Code avec bytecode dans les métadonnées](./media/ethereum-logic-app/bytecode-metadata.png)

**Pour utiliser le compilateur Solidity :**

Utilisez la commande `solc --bin <smart contract>.sol` pour générer le bytecode du contrat.

## <a name="get-the-contract-address"></a>Obtenir l’adresse du contrat

L’adresse du contrat est l’adresse de destination du contrat intelligent sur la blockchain Ethereum. Vous utilisez cette adresse pour envoyer une transaction ou une requête d’état d’un contrat actif. Vous pouvez obtenir l’adresse du contrat à partir de la sortie de migration Truffle ou du fichier de métadonnées du contrat.

**Pour utiliser la sortie de migration Truffle :**

Truffle affiche l’adresse du contrat après le déploiement du contrat intelligent. Copiez l’**adresse du contrat** à partir de la sortie.

![Sortie de migration Truffle avec l’adresse du contrat dans Visual Studio Code](./media/ethereum-logic-app/contract-address-truffle.png)

**Pour utiliser le fichier de métadonnées de contrat :**

1. Ouvrez le fichier de métadonnées de contrat contenu dans le dossier **build/contracts** de votre projet Solidity. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Recherchez la section **networks** dans le fichier JSON.
1. Les réseaux privés sont identifiés par un ID de réseau entier. Recherchez la valeur d’adresse dans la section networks.
1. Copiez la valeur **address**.

![Métadonnées avec la valeur d’adresse dans Visual Studio Code](./media/ethereum-logic-app/contract-address-metadata.png)

## <a name="next-steps"></a>Étapes suivantes

Visionnez des scénarios courants dans la vidéo [Doing more with Logic Apps](https://channel9.msdn.com/Shows/Blocktalk/Doing-more-with-Logic-Apps?term=logic%20apps%20blockchain&lang-en=true).
