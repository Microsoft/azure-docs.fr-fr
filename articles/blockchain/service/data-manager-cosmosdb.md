---
title: Utiliser Blockchain Data Manager pour mettre à jour Azure Cosmos DB - Azure Blockchain Service
description: Utiliser Blockchain Data Manager pour Azure Blockchain Service pour envoyer des données blockchain à Azure Cosmos DB
ms.date: 03/08/2020
ms.topic: tutorial
ms.reviewer: chroyal
ms.openlocfilehash: 69790787bc888448f2f40178bd12ee7058cc5892
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91283449"
---
# <a name="tutorial-use-blockchain-data-manager-to-send-data-to-azure-cosmos-db"></a>Tutoriel : Utiliser Blockchain Data Manager pour envoyer des données à Azure Cosmos DB

Dans ce didacticiel, vous utilisez Blockchain Data Manager pour Azure Blockchain Service afin d’enregistrer des données de transaction blockchain dans Azure Cosmos DB. Blockchain Data Manager capture, transforme et fournit des données de registre blockchain à des rubriques Azure Event Grid. Depuis Azure Event Grid, utilisez un connecteur d’application logique Azure pour créer des documents dans une base de données Azure Cosmos DB. Lorsque vous aurez terminé ce didacticiel, vous pourrez parcourir les données de transaction blockchain dans l’Explorateur de données Azure Cosmos DB.

[![Capture d’écran montrant les détails de la transaction blockchain.](./media/data-manager-cosmosdb/raw-msg.png)](./media/data-manager-cosmosdb/raw-msg.png#lightbox)

Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * Créer une instance Data Manager Blockchain
> * Ajouter une application blockchain pour décoder des propriétés de transaction et des événements
> * Créer un compte et une base de données Azure Cosmos DB pour stocker des données de transaction
> * Créer une application logique Azure pour connecter une rubrique Azure Event Grid à Azure Cosmos DB
> * Transmettre une transaction vers un registre blockchain
> * Afficher les données de transaction décodées dans Azure Cosmos DB

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Effectuer l’étape [Démarrage rapide : Créer un membre blockchain à l’aide du portail Azure](create-member.md) ou [Démarrage rapide : Créer un membre blockchain Azure Blockchain Service à l’aide de l’interface Azure CLI](create-member-cli.md)
* Effectuer l’étape [Démarrage rapide : Utiliser Visual Studio Code pour se connecter à un réseau du consortium Azure Blockchain Service](connect-vscode.md). Le guide de démarrage rapide vous dirige tout au long de l’installation de l’[Azure Blockchain Development Kit for Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) et de la configuration de votre environnement de développement blockchain.
* Mener à bien le [didacticiel : Utiliser Visual Studio Code pour créer, générer et déployer des contrats intelligents](send-transaction.md). Ce didacticiel vous guide pendant la création d’un exemple de contrat intelligent.
* Créer une [rubrique Event Grid](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)
* En savoir plus les [Gestionnaires d’événements dans Azure Event Grid](../../event-grid/event-handlers.md)

## <a name="create-instance"></a>Créer une instance

Une instance Blockchain Data Manager se connecte à un nœud de transaction Azure Blockchain Service et le surveille. Une instance capture toutes les données de bloc brut et de transaction brute à partir du nœud de transaction. Une connexion sortante envoie des données blockchain à Azure Event Grid. Vous configurez une seule connexion sortante lorsque vous créez l’instance.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez au membre Azure Blockchain Service que vous avez créé en suivant le [Guide de démarrage rapide prérequis : Créer un membre de blockchain à l’aide du Portail Azure](create-member.md). Sélectionnez **Blockchain Data Manager**.
1. Sélectionnez **Ajouter**.

    ![Ajouter Blockchain Data Manager](./media/data-manager-cosmosdb/add-instance.png)

    Entrez les informations suivantes :

    Paramètre | Exemple | Description
    --------|---------|------------
    Nom | mywatcher | Entrez un nom unique pour un Blockchain Data Manager.
    Nœud de transaction | myblockchainmember | Sélectionnez le nœud de transaction par défaut du membre Azure Blockchain Service que vous avez créé lors de l’étape prérequise.
    Nom de la connexion | cosmosdb | Entrez un nom unique pour la connexion sortante dans laquelle les données de transaction blockchain sont envoyées.
    Point de terminaison Event Grid | myTopic | Sélectionnez la rubrique Event Grid que vous avez créée lors de l’étape prérequise. Remarque : L’instance de Blockchain Data Manager et la rubrique Event Grid doivent être associées au même abonnement.

1. Sélectionnez **OK**.

    La création d’une instance Blockchain Data Manager prend moins d’une minute. Une fois l’instance déployée, elle est automatiquement démarrée. Une instance de Blockchain Data Manager en cours d’exécution capture les événements blockchain à partir du nœud de transaction et envoie les données à Event Grid.

## <a name="add-application"></a>Ajouter l’application

Ajoutez l’application blockchain **helloblockchain** pour que Blockchain Data Manager décode l’état de l’événement et de la propriété. Blockchain Data Manager nécessite le fichier ABI et bytecode de contrat intelligent pour ajouter l’application.

### <a name="get-contract-abi-and-bytecode"></a>Obtenir l’ABI et le bytecode du contrat

L’interface ABI du contrat définit les interfaces du contrat intelligent. Elle décrit comment interagir avec le contrat intelligent. Vous pouvez utiliser le [kit de développement Azure Blockchain pour l’extension Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) pour copier l’ABI du contrat dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** du projet Solidity **helloblockchain** que vous avez créé en suivant le [didacticiel prérequis : Utiliser Visual Studio Code pour créer, générer et déployer des contrats intelligents](send-transaction.md).
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Contract ABI** (Copier l’ABI du contrat).

    ![Volet Visual Studio Code avec l’option Copier l’ABI du contrat sélectionnée](./media/data-manager-cosmosdb/abi-devkit.png)

    L’ABI du contrat est copiée dans le Presse-papiers.

1. Enregistrez le tableau **abi** sous forme de fichier JSON. Par exemple, *abi.json*. Vous utiliserez ce fichier ultérieurement.

Blockchain Data Manager a besoin du bytecode déployé pour le contrat intelligent. Le bytecode déployé est différent du bytecode de contrat intelligent. Vous pouvez utiliser l’extension du kit de développement Azure Blockchain pour copier le bytecode dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** de votre projet Solidity.
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Transaction Bytecode** (Copier le bytecode de transaction).

    ![Volet Visual Studio Code avec l’option permettant de copier le bytecode de transaction sélectionnée](./media/data-manager-cosmosdb/bytecode-devkit.png)

    Le bytecode est copié dans le Presse-papiers.

1. Enregistrez la valeur **bytecode** en tant que fichier JSON. Par exemple, *bytecode.json*. Vous utiliserez ce fichier ultérieurement.

L’exemple suivant montre les fichiers *abi.json* et *bytecod.json* ouverts dans l’éditeur de VS Code. Vos fichiers devraient être similaires.

![Exemple de fichiers abi.json et bytecode.json](./media/data-manager-cosmosdb/contract-files.png)

### <a name="create-contract-abi-and-bytecode-url"></a>Créer l’ABI et le bytecode du contrat

Blockchain Data Manager requiert que les fichiers ABI et bytecode du contrat soient accessibles par une URL lors de l’ajout d’une application. Vous pouvez utiliser un compte de stockage Azure pour fournir une URL accessible en privé.

#### <a name="create-storage-account"></a>Créer un compte de stockage

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

#### <a name="upload-contract-files"></a>Charger les fichiers de contrat

1. Créez un nouveau conteneur pour le compte de stockage. Sélectionnez **Conteneurs > Conteneur**.

    ![Créer un conteneur de compte de stockage](./media/data-manager-cosmosdb/create-container.png)

    | Paramètre | Description |
    |---------|-------------|
    | Nom  | Nommez le conteneur. Par exemple, *smartcontract* |
    | Niveau d'accès public | Choisissez *Privé (aucun accès anonyme)* |

1. Sélectionnez **OK** pour créer le conteneur.
1. Sélectionnez le conteneur, puis sélectionnez **Télécharger**.
1. Sélectionnez les deux fichiers JSON que vous avez créés dans la section [Obtenir l’ABI et le bytecode du contrat](#get-contract-abi-and-bytecode).

    ![Charger l’objet blob](./media/data-manager-cosmosdb/upload-blobs.png)

    Sélectionnez **Télécharger**.

#### <a name="generate-url"></a>Générer une URL

Pour chaque blob, générez une signature d’accès partagé.

1. Sélectionnez le blob ABI JSON.
1. Sélectionnez **Générer une signature d'accès partagé**.
1. Définissez l’expiration de la signature d’accès souhaitée, puis sélectionnez **Générer un jeton et une URL SAP d'objet blob**.

    ![Générer un jeton SAS](./media/data-manager-cosmosdb/generate-sas.png)

1. Copiez l’**URL SAS d’objet blob** et enregistrez-la pour la section suivante.
1. Répétez les étapes [Générer une URL](#generate-url) pour l’objet blob JSON de bytecode.

### <a name="add-helloblockchain-application-to-instance"></a>Ajouter l’application helloblockchain à l’instance

1. Sélectionnez votre instance Data Manager Blockchain dans la liste des instances.
1. Sélectionnez **Applications Blockchain**.
1. Sélectionnez **Ajouter**.

    ![Ajouter une application blockchain](./media/data-manager-cosmosdb/add-application.png)

    Entrez le nom de l’application blockchain et les URL d’ABI et de bytecode du contrat intelligent.

    Paramètre | Description
    --------|------------
    Nom | Entrez un nom unique pour l’application blockchain à suivre.
    ABI du contrat | Chemin d’accès de l’URL au fichier ABI du contrat. Pour plus d’informations, consultez [Créer l’ABI et le bytecode du contrat](#create-contract-abi-and-bytecode-url).
    Bytecode du contrat | Chemin de l’URL du fichier bytecode. Pour plus d’informations, consultez [Créer l’ABI et le bytecode du contrat](#create-contract-abi-and-bytecode-url).

1. Sélectionnez **OK**.

    Une fois l’application créée, l’application s’affiche dans la liste des applications blockchain.

    ![Liste des applications blockchain](./media/data-manager-cosmosdb/artifact-list.png)

Vous pouvez supprimer le compte de stockage Azure ou l’utiliser pour configurer d’autres applications blockchain. Si vous souhaitez supprimer le compte de stockage Azure, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le compte de stockage associé et d’autres ressources liées au groupe de ressources.

## <a name="create-azure-cosmos-db"></a>Créer une base de données Azure Cosmos

[!INCLUDE [cosmos-db-create-storage-account](../../../includes/cosmos-db-create-dbaccount.md)]

### <a name="add-a-database-and-container"></a>Ajouter une base de données et un conteneur

Vous pouvez utiliser l’Explorateur de données du portail Azure pour créer une base de données et un conteneur.

1. Dans la page de votre compte Azure Cosmos DB, sur la gauche, sélectionnez **Explorateur de données**, puis **Nouveau conteneur**.
1. Dans le volet **Ajouter un conteneur**, entrez les paramètres du nouveau conteneur.

    ![Ajouter les paramètres du conteneur](./media/data-manager-cosmosdb/add-container.png)

    | Paramètre | Description
    |---------|-------------|
    | ID de base de données | Entrez **blockchain-data** comme nom de la nouvelle base de données. |
    | Débit | Laissez le débit sur **400** unités de requête par seconde (RU/s). Si vous souhaitez réduire la latence, vous pourrez augmenter le débit par la suite.|
    | ID de conteneur | Entrez **Messages** comme nom de votre nouveau conteneur. |
    | Clé de partition | Utilisez **/MessageType** comme clé de partition. |

1. Sélectionnez **OK**. L’Explorateur de données présente la nouvelle base de données et le conteneur que vous avez créé.

## <a name="create-logic-app"></a>Créer une application logique

Azure Logic Apps vous aide à planifier et à automatiser les processus et workflows métier quand vous avez besoin d’intégrer des systèmes et des services. Vous pouvez utiliser une application logique pour connecter Event Grid à Azure Cosmos DB.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Intégration** > **Application logique**.
1. Indiquez où vous souhaitez créer votre application logique. Sélectionnez **Créer** quand vous avez terminé.

    Pour plus d’informations sur la création d’applications logiques, consultez [Créer votre premier workflow automatisé avec Azure Logic Apps](../../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Une fois qu’Azure a déployé votre application, sélectionnez votre ressource d’application logique.
1. Dans le Concepteur Logic Apps, sous **Modèles**, sélectionnez **Application logique vide**.

### <a name="add-event-grid-trigger"></a>Ajouter un déclencheur Event Grid

Chaque application logique doit démarrer avec un déclencheur, qui s’active lorsqu’un événement spécifique se produit ou lorsqu’une condition particulière est remplie. Chaque fois que le déclencheur s’active, le moteur Logic Apps crée une instance d’application logique qui démarre et exécute votre flux de travail. Utilisez un déclencheur Azure Event Grid pour envoyer les données de transaction blockchain de Event Grid vers Cosmos DB.

1. Dans le concepteur de Logic Apps, recherchez et sélectionnez le connecteur **Azure Event Grid**.
1. Sous l’onglet **Déclencheurs**, sélectionnez **Quand un événement de ressource se produit**.
1. Créez une connexion d’API à votre rubrique Event Grid.

    ![Paramètres du déclencheur Event Grid](./media/data-manager-cosmosdb/event-grid-trigger.png)

    | Paramètre | Description
    |---------|-------------|
    | Abonnement | Sélectionnez l’abonnement Azure qui contient la rubrique Event Hub. |
    | Type de ressource | Sélectionnez **Microsoft.EventGrid.Topics**. |
    | Nom de la ressource | Sélectionnez le nom de la rubrique Event Grid dans laquelle Blockchain Data Manager envoie ses messages de données de transaction. |

### <a name="add-cosmos-db-action"></a>Ajouter une action Cosmos DB

Ajoutez une action de création de document dans Cosmos DB pour chaque transaction. Utilisez le type de message de transaction comme clé de partition pour catégoriser les messages.

1. Sélectionnez **Nouvelle étape**.
1. Dans **Choisir une action**, recherchez **Azure Cosmos DB**.
1. Sélectionnez **Azure Cosmos DB > Actions > Créer ou mettre à jour un document**.
1. Créez une connexion d’API à votre base de données Cosmos DB.

    ![Paramètres de la connexion à Cosmos DB](./media/data-manager-cosmosdb/cosmosdb-connection.png)

    | Paramètre | Description
    |---------|-------------|
    | Nom de connexion | Sélectionnez l’abonnement Azure qui contient la rubrique Event Hub. |
    | Compte DocumentDB | Sélectionnez le compte DocumentDB que vous avez créé dans la section [Créer un compte Azure Cosmos DB](#create-azure-cosmos-db). |

1. Entrez l’**ID de base de données** et l’**ID de la collection** de l’instance Azure Cosmos DB que vous avez créée en suivant la section [Ajouter une base de données et un conteneur](#add-a-database-and-container).

1. Sélectionnez le paramètre **Document**. Dans la liste déroulante *Ajouter du contenu dynamique*, sélectionnez **Expression**, puis copiez et collez l’expression suivante :

    ```
    addProperty(triggerBody()?['data'], 'id', utcNow())
    ```

    L’expression obtient la partie de données du message et définit l’ID sur une valeur d’horodatage.

1. Sélectionnez **Ajouter un nouveau paramètre**, puis sélectionnez **Valeur de clé de partition**.
1. Définissez la **Valeur de clé de partition** sur `"@{triggerBody()['data']['MessageType']}"`. La valeur doit être encadrée par des guillemets doubles.

    ![Concepteur Logic Apps paramétré pour Cosmos DB](./media/data-manager-cosmosdb/create-action.png)

    La valeur définit la clé de partition sur le type de message de transaction.

1. Sélectionnez **Enregistrer**.

L’application logique surveille la rubrique Event Grid. Elle crée un document dans Cosmos DB pour chaque nouveau message de transaction transmis par Blockchain Data Manager.

## <a name="send-a-transaction"></a>Envoyer une transaction

L’étape suivante consiste à tester ce que vous venez de créer, en transmettant une transaction vers le registre blockchain. Utilisez la fonction **SendRequest** du contrat **HelloBlockchain** que vous avez créée dans le [tutoriel prérequis : Utiliser Visual Studio Code pour créer, générer et déployer des contrats intelligents](send-transaction.md).

1. Utilisez la page d’interaction du contrat intelligent du kit de développement Azure Blockchain pour appeler la fonction **SendRequest**. Cliquez avec le bouton droit sur **HelloBlockchain.sol** et choisissez **Show Smart Contract Interaction Page** (Afficher la page d’interaction du contrat intelligent) dans le menu.

    ![Choix de l’option permettant d’afficher la page d’interaction du contrat intelligent](./media/data-manager-cosmosdb/contract-interaction.png)

1. Choisissez l’action de contrat **SendRequest** et entrez **Hello, Blockchain!** pour le paramètre **requestMessage**. Sélectionnez **Execute** (Exécuter) pour appeler la fonction **SendRequest** par le biais d’une transaction.

    ![Exécuter l’action SendRequest](./media/data-manager-cosmosdb/sendrequest-action.png)

La fonction SendRequest définit les champs **RequestMessage** et **State**. L’état actuel de **RequestMessage** correspond à l’argument que vous avez passé (**Hello, Blockchain**). La valeur du champ **State** reste **Request**.

## <a name="view-transaction-data"></a>Afficher les données de transaction

Maintenant que vous avez connecté votre instance de Blockchain Data Manager à Azure Cosmos DB, vous pouvez afficher les messages de transaction blockchain dans l’Explorateur de données Cosmos DB.

1. Accédez à l’Explorateur de données Cosmos DB. Par exemple, **cosmosdb-blockchain > Explorateur de données > blockchain-data > Messages > Éléments**.

    ![Explorateur de données Cosmos DB](./media/data-manager-cosmosdb/data-explorer.png)

    L’Explorateur de données répertorie les messages de données blockchain créés dans la base de données Cosmos DB.

1. Parcourez les messages en sélectionnant leurs ID d’élément et recherchez le message doté du code de hachage de transaction correspondant.

    [![Capture d’écran montrant les détails de la transaction blockchain d’un élément sélectionné.](./media/data-manager-cosmosdb/raw-msg.png)](./media/data-manager-cosmosdb/raw-msg.png#lightbox)

    Le message de transaction brut contient des détails sur la transaction. Toutefois, ses informations de propriété sont chiffrées.

    Comme vous avez ajouté le contrat intelligent HelloBlockchain à votre instance Blockchain Data Manager, un type de message **ContractProperties** est également envoyé. Il contient des informations sur les propriétés décodées.

1. Recherchez le message **ContractProperties** correspondant à la transaction. Il doit s’agir du message suivant dans la liste.

    [![Détail de la transaction blockchain](./media/data-manager-cosmosdb/properties-msg.png)](./media/data-manager-cosmosdb/properties-msg.png#lightbox)

    Le tableau **DecodedProperties** contient les propriétés de la transaction.

Félicitations ! Vous venez de créer un explorateur de messages des transactions blockchain à l’aide de Blockchain Data Manager et d’Azure Cosmos DB.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en aurez plus besoin, vous pourrez supprimer les ressources et groupes de ressources que vous avez utilisés dans le cadre de ce didacticiel. Pour supprimer un groupe de ressources :

1. Dans le portail Azure, accédez à **Groupe de ressources** dans le volet de navigation de gauche et sélectionnez le groupe de ressources que vous souhaitez supprimer.
1. Sélectionnez **Supprimer le groupe de ressources**. Validez la suppression en entrant le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’intégration aux registres blockchain.

> [!div class="nextstepaction"]
> [Utiliser le connecteur Ethereum Blockchain avec Azure Logic Apps](ethereum-logic-app.md)
