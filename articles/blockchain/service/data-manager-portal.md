---
title: Configurer Blockchain Data Manager avec le portail Azure - Azure Blockchain Service
description: Créez et gérez Blockchain Data Manager pour Azure Blockchain Service avec le portail Azure.
ms.date: 03/30/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: a151c28d31bf0bb7f21185fb161315d42f9563d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85200679"
---
# <a name="configure-blockchain-data-manager-using-the-azure-portal"></a>Configurer Blockchain Data Manager avec le Portail Azure

Configurez Blockchain Data Manager pour Azure Blockchain Service afin de capturer des données Blockchain et les envoyer à une rubrique Azure Event Grid.

Pour configurer une instance Data Manager Blockchain, vous devez :

* Créer une instance Blockchain Data Manager pour un nœud de transaction Azure Blockchain Service
* Ajouter vos applications blockchain

## <a name="prerequisites"></a>Prérequis

* Effectuer l’étape [Démarrage rapide : Créer un membre blockchain à l’aide du portail Azure](create-member.md) ou [Démarrage rapide : Créer un membre blockchain Azure Blockchain Service à l’aide de l’interface Azure CLI](create-member-cli.md). Le service Azure Blockchain niveau *Standard* est recommandé lors de l’utilisation de Blockchain Data Manager.
* Créer une [rubrique Event Grid](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)
* En savoir plus les [Gestionnaires d’événements dans Azure Event Grid](../../event-grid/event-handlers.md)

## <a name="create-instance"></a>Créer une instance

Une instance Blockchain Data Manager se connecte à un nœud de transaction Azure Blockchain Service et le surveille. Seuls les utilisateurs ayant accès au nœud de transaction peuvent créer une connexion. Une instance capture toutes les données de bloc brut et de transaction brute à partir du nœud de transaction. Blockchain Data Manager publie un message **RawBlockAndTransactionMsg** qui est un sur-ensemble des informations retournées par les requêtes web3.eth [getBlock](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#getblock) et [getTransaction](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#gettransaction).

Une connexion sortante envoie des données blockchain à Azure Event Grid. Vous configurez une seule connexion sortante lorsque vous créez l’instance. Blockchain Data Manager prend en charge plusieurs connexions sortantes de rubrique Event Grid pour une instance de Data Manager Blockchain donnée. Vous pouvez envoyer des données blockchain à une ou plusieurs destinations. Pour ajouter une autre destination, ajoutez simplement des connexions sortantes à l’instance.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez au membre Azure Blockchain Service que vous souhaitez connecter à Blockchain Data Manager. Sélectionnez **Blockchain Data Manager**.
1. Sélectionnez **Ajouter**.

    ![Ajouter Blockchain Data Manager](./media/data-manager-portal/add-instance.png)

    Entrez les informations suivantes :

    Paramètre | Description
    --------|------------
    Nom | Entrez un nom unique pour un Blockchain Data Manager. Le nom du Blockchain Data Manager peut contenir des lettres minuscules et des chiffres et ne doit pas dépasser 20 caractères.
    Nœud de transaction | Choisissez un nœud de transaction. Seuls les nœuds de transaction pour lesquels vous disposez d’un accès en lecture sont répertoriés.
    Nom de la connexion | Entrez un nom unique pour la connexion sortante dans laquelle les données de transaction blockchain sont envoyées.
    Point de terminaison Event Grid | Choisissez une rubrique Event Grid dans le même abonnement que l’instance Blockchain Data Manager.

1. Sélectionnez **OK**.

    La création d’une instance Blockchain Data Manager prend moins d’une minute. Une fois l’instance déployée, elle est automatiquement démarrée. Une instance de Blockchain Data Manager en cours d’exécution capture les événements blockchain à partir du nœud de transaction et envoie les données aux connexions sortantes.

    La nouvelle instance s’affiche dans la liste des instances Blockchain Data Manager pour le membre Azure Blockchain Service.

    ![Liste des instances de membre de données Blockchain](./media/data-manager-portal/instance-list.png)

## <a name="add-blockchain-application"></a>Ajouter une application blockchain

Si vous ajoutez une application blockchain, Blockchain Data Manager décode l’état de l’événement et de la propriété pour l’application. Dans le cas contraire, seules les données de bloc brut et de transaction brute sont envoyées. Blockchain Data Manager détecte également les adresses de contrat lors du déploiement du contrat. Vous pouvez ajouter plusieurs applications blockchain à une instance Blockchain Data Manager.

> [!IMPORTANT]
> Actuellement, les applications blockchain qui déclarent des [types de tableaux](https://solidity.readthedocs.io/en/v0.5.12/types.html#arrays) ou [types de mappages](https://solidity.readthedocs.io/en/v0.5.12/types.html#mapping-types) Solidity ne sont pas entièrement prises en charge. Les propriétés déclarées comme types de tableaux ou de mappages ne sont pas décodées dans les messages *ContractPropertiesMsg* ou *DecodedContractEventsMsg*.

Blockchain Data Manager nécessite un fichier bytecode déployé et ABI de contrat intelligent pour ajouter l’application.

### <a name="get-contract-abi-and-bytecode"></a>Obtenir l’ABI et le bytecode du contrat

L’interface ABI du contrat définit les interfaces du contrat intelligent. Elle décrit comment interagir avec le contrat intelligent. Vous pouvez utiliser le [kit de développement Azure Blockchain pour l’extension Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) pour copier l’ABI du contrat dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** de votre projet Solidity.
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Contract ABI** (Copier l’ABI du contrat).

    ![Volet Visual Studio Code avec l’option Copier l’ABI du contrat sélectionnée](./media/data-manager-portal/abi-devkit.png)

    L’ABI du contrat est copiée dans le Presse-papiers.

1. Enregistrez le tableau **abi** sous forme de fichier JSON. Par exemple, *abi.json*. Vous utiliserez ce fichier ultérieurement.

Blockchain Data Manager a besoin du bytecode déployé pour le contrat intelligent. Le bytecode déployé est différent du bytecode de contrat intelligent. Vous pouvez utiliser l’extension du kit de développement Azure Blockchain pour copier le bytecode dans le Presse-papiers.

1. Dans le volet de l’Explorateur Visual Studio Code, développez le dossier **build/contracts** de votre projet Solidity.
1. Cliquez avec le bouton droit sur le fichier JSON des métadonnées de contrat. Le nom du fichier est le nom du contrat intelligent suivi de l’extension **.json**.
1. Sélectionnez **Copy Transaction Bytecode** (Copier le bytecode de transaction).

    ![Volet Visual Studio Code avec l’option permettant de copier le bytecode de transaction sélectionnée](./media/data-manager-portal/bytecode-devkit.png)

    Le bytecode est copié dans le Presse-papiers.

1. Enregistrez la valeur **bytecode** en tant que fichier JSON. Par exemple, *bytecode.json*. Vous utiliserez ce fichier ultérieurement.

L’exemple suivant montre les fichiers *abi.json* et *bytecod.json* ouverts dans l’éditeur de VS Code. Vos fichiers devraient être similaires.

![Exemple de fichiers abi.json et bytecode.json](./media/data-manager-portal/contract-files.png)

### <a name="create-contract-abi-and-bytecode-url"></a>Créer l’ABI et le bytecode du contrat

Blockchain Data Manager requiert que les fichiers ABI et bytecode du contrat soient accessibles par une URL lors de l’ajout d’une application. Vous pouvez utiliser un compte de stockage Azure pour fournir une URL accessible en privé.

#### <a name="create-storage-account"></a>Créer un compte de stockage

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

#### <a name="upload-contract-files"></a>Charger les fichiers de contrat

1. Créez un nouveau conteneur pour le compte de stockage. Sélectionnez **Conteneurs > Conteneur**.

    ![Créer un conteneur de compte de stockage](./media/data-manager-portal/create-container.png)

    | Champ | Description |
    |-------|-------------|
    | Nom  | Nommez le conteneur. Par exemple, *smartcontract* |
    | Niveau d'accès public | Choisissez *Privé (aucun accès anonyme)* |

1. Sélectionnez **OK** pour créer le conteneur.
1. Sélectionnez le conteneur, puis sélectionnez **Télécharger**.
1. Sélectionnez les deux fichiers JSON que vous avez créés dans la section [Obtenir l’ABI et le bytecode du contrat](#get-contract-abi-and-bytecode).

    ![Charger l’objet blob](./media/data-manager-portal/upload-blobs.png)

    Sélectionnez **Télécharger**.

#### <a name="generate-url"></a>Générer une URL

Pour chaque blob, générez une signature d’accès partagé.

1. Sélectionnez le blob ABI JSON.
1. Sélectionnez **Générer une signature d'accès partagé**.
1. Définissez l’expiration de la signature d’accès souhaitée, puis sélectionnez **Générer un jeton et une URL SAP d'objet blob**.

    ![Générer un jeton SAS](./media/data-manager-portal/generate-sas.png)

1. Copiez l’**URL SAS d’objet blob** et enregistrez-la pour la section suivante.
1. Répétez les étapes [Générer une URL](#generate-url) pour l’objet blob JSON de bytecode.

### <a name="add-application-to-instance"></a>Ajouter l’application à l’instance

1. Sélectionnez votre instance Data Manager Blockchain dans la liste des instances.
1. Sélectionnez **Applications Blockchain**.
1. Sélectionnez **Ajouter**.

    ![Ajouter une application blockchain](./media/data-manager-portal/add-application.png)

    Entrez le nom de l’application blockchain et les URL d’ABI et de bytecode du contrat intelligent.

    Paramètre | Description
    --------|------------
    Nom | Entrez un nom unique pour l’application blockchain à suivre.
    ABI du contrat | Chemin d’accès de l’URL au fichier ABI du contrat. Pour plus d’informations, consultez [Créer l’ABI et le bytecode du contrat](#create-contract-abi-and-bytecode-url).
    Bytecode du contrat | Chemin de l’URL du fichier bytecode. Pour plus d’informations, consultez [Créer l’ABI et le bytecode du contrat](#create-contract-abi-and-bytecode-url).

1. Sélectionnez **OK**.

    Une fois l’application créée, l’application s’affiche dans la liste des applications blockchain.

    ![Liste des applications blockchain](./media/data-manager-portal/artifact-list.png)

Vous pouvez supprimer le compte de stockage Azure ou l’utiliser pour configurer d’autres applications blockchain. Si vous souhaitez supprimer le compte de stockage Azure, vous pouvez supprimer le groupe de ressources. La suppression du groupe de ressources efface également le compte de stockage associé et d’autres ressources liées au groupe de ressources.

## <a name="stop-instance"></a>Arrêter l’instance

Arrêtez l’instance Blockchain Manager lorsque vous souhaitez arrêter la capture des événements blockchain et l’envoi de données aux connexions sortantes. Lorsque l’instance est arrêtée, aucun frais n’est facturé pour Blockchain Data Manager. Pour plus d’informations, voir la [tarification](https://azure.microsoft.com/pricing/details/blockchain-service).

1. Accédez à **Présentation** et sélectionnez **Arrêter**.

    ![Arrêter l’instance](./media/data-manager-portal/stop-instance.png)

## <a name="next-steps"></a>Étapes suivantes

Essayez le tutoriel suivant, qui consiste à créer un explorateur de messages des transactions blockchain à l’aide de Blockchain Data Manager et d’Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Utiliser Blockchain Data Manager pour envoyer des données à Azure Cosmos DB](data-manager-cosmosdb.md)