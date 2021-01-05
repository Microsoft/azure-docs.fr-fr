---
title: Configurer Blockchain Data Manager avec Azure CLI - Azure Blockchain Service
description: Créez et gérez une instance Blockchain Data Manager pour Azure Blockchain Service avec Azure CLI
ms.date: 03/30/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: f067f4413f6ad8541cd36a7581f9243bed4e195f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87023736"
---
# <a name="configure-blockchain-data-manager-using-azure-cli"></a>Configurer Blockchain Data Manager avec Azure CLI

Configurez Blockchain Data Manager pour Azure Blockchain Service afin de capturer des données Blockchain et les envoyer à une rubrique Azure Event Grid.

Pour configurer une instance Data Manager Blockchain, vous devez :

* Créer une instance Blockchain Manager
* Créer une entrée sur un nœud de transaction Azure Blockchain Service
* Créer une sortie dans une rubrique Azure Event Grid
* Ajouter une application blockchain
* Démarrer une instance

## <a name="prerequisites"></a>Prérequis

* Installez la dernière version d’[Azure CLI](/cli/azure/install-azure-cli) et connectez-vous à l’aide de `az login`.
* Effectuer l’étape [Démarrage rapide : Utilisez Visual Studio Code pour vous connecter à un réseau de consortium Azure Blockchain Service](connect-vscode.md). Le service Azure Blockchain niveau *Standard* est recommandé lors de l’utilisation de Blockchain Data Manager.
* Créer une [rubrique Event Grid](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)
* En savoir plus les [Gestionnaires d’événements dans Azure Event Grid](../../event-grid/event-handlers.md)

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

Si vous préférez installer et utiliser l’interface de ligne de commande en local, ce guide de démarrage rapide nécessite au minimum la version 2.0.51 d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez l’installer ou le mettre à niveau, consultez [Installation d’Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myRG --location eastus
```

## <a name="create-instance"></a>Créer une instance

Une instance de Blockchain Data Manager surveille un nœud de transaction Azure Blockchain Service. Une instance capture toutes les données de bloc brut et de transaction brute à partir du nœud de transaction. Blockchain Data Manager publie un message **RawBlockAndTransactionMsg** qui est un sur-ensemble des informations retournées par les requêtes web3.eth [getBlock](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#getblock) et [getTransaction](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html#gettransaction).

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Blockchain Data Manager instance name> \
                   --resource-type Microsoft.blockchain/watchers \
                   --is-full-object \
                   --properties <watcher resource properties>
```

| Paramètre | Description |
|-----------|-------------|
| resource-group | Nom du groupe de ressources dans lequel créer l’instance Blockchain Data Manager. |
| name | Nom de l’instance Data Manager Blockchain. |
| resource-type | Le type de ressource est **Microsoft.blockchain/watchers** pour une instance Blockchain Data Manager. |
| is-full-object | Indique que les propriétés contiennent des options pour la ressource d’observateur. |
| properties | Chaîne au format JSON contenant les propriétés de la ressource d’observateur. Peut être passé en tant que chaîne ou fichier.  |

### <a name="create-instance-examples"></a>Créer des exemples d’instance

Exemple de configuration JSON pour créer une instance Blockchain Manager dans la région **USA Est**.

``` json
{
    "location": "eastus",
    "properties": {
    }
}
```

| Élément | Description |
|---------|-------------|
| location | Région dans laquelle créer la ressource d’observateur |
| properties | Propriétés à définir lors de la création de la ressource d’observateur |

Créez une instance Blockchain Data Manager nommée *mywatcher* à l’aide d’une chaîne JSON pour la configuration.

``` azurecli-interactive
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                     --is-full-object \
                     --properties '{"location":"eastus"}'
```

Créez une instance Blockchain Data Manager nommée *mywatcher* à l’aide d’un fichier de configuration JSON.

``` azurecli
az resource create \
                    --resource-group myRG \
                    --name mywatcher \
                    --resource-type Microsoft.blockchain/watchers \
                    --is-full-object \
                    --properties @watcher.json
```

## <a name="create-input"></a>Création d’une entrée

Une entrée connecte Blockchain Data Manager à un nœud de transaction Azure Blockchain Service. Seuls les utilisateurs ayant accès au nœud de transaction peuvent créer une connexion.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Input name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <input resource properties>
```

| Paramètre | Description |
|-----------|-------------|
| resource-group | Nom de groupe de ressources dans lequel créer la ressource d’entrée. |
| name | Nom de l’entrée. |
| espace de noms | Utilisez l’espace de noms du fournisseur **Microsoft.Blockchain**. |
| resource-type | **inputs** est le type de ressource d’une entrée de Data Manager Blockchain. |
| parent | Chemin d’accès à l’observateur auquel l’entrée est associée. Par exemple, **watchers/mywatcher**. |
| is-full-object | Indique que les propriétés contiennent des options pour la ressource d’entrée. |
| properties | Chaîne au format JSON contenant les propriétés de la ressource d’entrée. Peut être passé en tant que chaîne ou fichier. |

### <a name="input-examples"></a>Exemples d’entrée

Exemple de JSON de configuration permettant de créer une ressource d’entrée dans la région *USA Est* connectée au \<Blockchain member\>.

``` json
{
    "location": "eastus",
    "properties": {
        "inputType": "Ethereum",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/blockchainMembers/<Blockchain member>/transactionNodes/transaction-node"
        }
    }
}
```

| Élément | Description |
|---------|-------------|
| location | Région dans laquelle créer la ressource d’entrée. |
| inputType | Type de registre du membre du service Azure Blockchain. **Ethereum** est actuellement pris en charge. |
| resourceId | Nœud de transaction auquel l’entrée est connectée. Remplacez \<Subscription ID\>, \<Resource group\> et \<Blockchain member\> par les valeurs de la ressource de nœud de transaction. L’entrée se connecte au nœud de transaction par défaut pour le membre Azure Blockchain Service. |

Créez une entrée nommée *myInput* pour *mywatcher* à l’aide d’une chaîne JSON de configuration.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myInput \
                   --namespace Microsoft.Blockchain \
                   --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus", "properties":{"inputType":"Ethereum","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/BlockchainMembers/<Blockchain member>/transactionNodes/transaction-node"}}}'
```

Créez une entrée nommée *myInput* pour *mywatcher* à l’aide d’un fichier JSON de configuration.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name input \
                   --namespace Microsoft.Blockchain \ --resource-type inputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @input.json
```

## <a name="create-output"></a>Créer une sortie

Une connexion sortante envoie des données blockchain à Azure Event Grid. Vous pouvez envoyer des données blockchain à une ou plusieurs destinations. Blockchain Data Manager prend en charge plusieurs connexions sortantes de rubrique Event Grid pour une instance de Data Manager Blockchain donnée.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Output name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <output resource properties>
```

| Paramètre | Description |
|-----------|-------------|
| resource-group | Nom de groupe de ressources dans lequel créer la ressource de sortie. |
| name | Nom de la sortie. |
| espace de noms | Utilisez l’espace de noms du fournisseur **Microsoft.Blockchain**. |
| resource-type | **outputs** est le type de ressource d’une sortie de Data Manager Blockchain. |
| parent | Chemin d’accès à l’observateur auquel la sortie est associée. Par exemple, **watchers/mywatcher**. |
| is-full-object | Indique que les propriétés contiennent des options pour la ressource de sortie. |
| properties | Chaîne au format JSON contenant les propriétés de la ressource de sortie. Peut être passé en tant que chaîne ou fichier. |

### <a name="output-examples"></a>Exemples de sortie

Exemple de JSON de configuration permettant de créer une ressource de sortie dans la région *USA Est* connectée à une rubrique Event Grid nommée \<event grid topic\>.

``` json
{
    "location": "eastus",
    "properties": {
        "outputType": "EventGrid",
        "dataSource": {
            "resourceId": "/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"
        }
    }
}
```

| Élément | Description |
|---------|-------------|
| location | Région dans laquelle créer la ressource de sortie. |
| outputType | Type de sortie. Actuellement, **EventGrid** est pris en charge. |
| resourceId | Ressource à laquelle la sortie est connectée. Remplacez \<Subscription ID\>, \<Resource group\> et \<Blockchain member\> par les valeurs de la ressource Event Grid. |

Créez une sortie nommée *myoutput* pour *mywatcher* qui se connecte à une rubrique Event Grid à l’aide d’une chaîne de configuration JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"outputType":"EventGrid","dataSource":{"resourceId":"/subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.EventGrid/topics/<event grid topic>"}}}'
```

Créez une sortie nommée *myoutput* pour *mywatcher* qui se connecte à une rubrique Event Grid à l’aide d’un fichier de configuration JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myoutput \
                   --namespace Microsoft.Blockchain \
                   --resource-type outputs \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @output.json
```

## <a name="add-blockchain-application"></a>Ajouter une application blockchain

Si vous ajoutez une application blockchain, Blockchain Data Manager décode l’état de l’événement et de la propriété pour l’application. Dans le cas contraire, seules les données de bloc brut et de transaction brute sont envoyées. Blockchain Data Manager détecte également les adresses de contrat lors du déploiement du contrat. Vous pouvez ajouter plusieurs applications blockchain à une instance Blockchain Data Manager.


> [!IMPORTANT]
> Actuellement, les applications blockchain qui déclarent des [types de tableaux](https://solidity.readthedocs.io/en/v0.5.12/types.html#arrays) ou [types de mappages](https://solidity.readthedocs.io/en/v0.5.12/types.html#mapping-types) Solidity ne sont pas entièrement prises en charge. Les propriétés déclarées comme types de tableaux ou de mappages ne sont pas décodées dans les messages *ContractPropertiesMsg* ou *DecodedContractEventsMsg*.

``` azurecli
az resource create \
                   --resource-group <Resource group> \
                   --name <Application name> \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/<Watcher name> \
                   --is-full-object \
                   --properties <Application resource properties>
```

| Paramètre | Description |
|-----------|-------------|
| resource-group | Nom de groupe de ressources dans lequel créer la ressource d’application. |
| name | Nom de l’application. |
| espace de noms | Utilisez l’espace de noms du fournisseur **Microsoft.Blockchain**. |
| resource-type | **artifacts** est le type de ressource d’une application Data Manager Blockchain. |
| parent | Chemin d’accès à l’observateur auquel l’application est associée. Par exemple, **watchers/mywatcher**. |
| is-full-object | Indique que les propriétés contiennent des options pour la ressource d’application. |
| properties | Chaîne au format JSON contenant les propriétés de la ressource d’application. Peut être passé en tant que chaîne ou fichier. |

### <a name="blockchain-application-examples"></a>Exemples d’applications blockchain

Exemple de configuration JSON pour créer une ressource d’application dans la région *USA Est* surveillant un contrat intelligent défini par l’ABI et le bytecode du contrat.

``` json
{
    "location": "eastus",
    "properties": {
        "artifactType": "EthereumSmartContract",
        "content": {
            "abiFileUrl": "<ABI URL>",
            "bytecodeFileUrl": "<Bytecode URL>",
            "queryTargetTypes": [
                "ContractProperties",
                "ContractEvents"
            ]
        }
    }
}
```

| Élément | Description |
|---------|-------------|
| location | Région dans laquelle créer la ressource d’application. |
| artifactType | Type d’application. Actuellement, **EthereumSmartContract** est pris en charge. |
| abiFileUrl | URL du fichier JSON de l’ABI du contrat intelligent. Pour plus d’informations sur l’obtention de l’ABI du contrat et la création d’une URL, consultez [Obtenir l’ABI et le bytecode du contrat](data-manager-portal.md#get-contract-abi-and-bytecode) et [Créer l’ABI et le bytecode du contrat](data-manager-portal.md#create-contract-abi-and-bytecode-url). |
| bytecodeFileUrl | URL du fichier JSON pour le bytecode déployé du contrat intelligent. Pour plus d’informations sur l’obtention du bytecode déployé du contrat et la création d’une URL, consultez [Obtenir l’ABI et le bytecode du contrat](data-manager-portal.md#get-contract-abi-and-bytecode) et [Créer l’URL de l’ABI et du bytecode du contrat](data-manager-portal.md#create-contract-abi-and-bytecode-url). Remarque : Blockchain Data Manager exige le **bytecode déployé**. |
| queryTargetTypes | Types de messages publiés. **ContractProperties** publie le type de message *ContractPropertiesMsg*. **ContractEvents** publie le type de message *DecodedContractEventsMsg*. Remarque : Les types de message *RawBlockAndTransactionMsg* et *RawTransactionContractCreationMsg* sont toujours publiés. |

Créez une application nommée *myApplication* pour *mywatcher* surveillant un contrat intelligent défini par une chaîne JSON.

``` azurecli-interactive
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties '{"location":"eastus","properties":{"artifactType":"EthereumSmartContract","content":{"abiFileUrl":"<ABI URL>","bytecodeFileUrl":"<Bytecode URL>","queryTargetTypes":["ContractProperties","ContractEvents"]}}}'
```

Créez une application nommée *myApplication* pour *mywatcher* observant un contrat intelligent défini à l’aide d’un fichier de configuration JSON.

``` azurecli
az resource create \
                   --resource-group myRG \
                   --name myApplication \
                   --namespace Microsoft.Blockchain \
                   --resource-type artifacts \
                   --parent watchers/mywatcher \
                   --is-full-object \
                   --properties @artifact.json
```

## <a name="start-instance"></a>Démarrer l’instance

Lors de son exécution, une instance Blockchain Manager surveille les événements Blockchain à partir des entrées définies et envoie des données aux sorties définies.

``` azurecli
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Paramètre | Description |
|-----------|-------------|
| action | Démarrez l’observateur à l’aide de **start**. |
| ids | ID de ressource d’observateur. Remplacez \<Subscription ID\>, \<Resource group\> et \<Watcher name\> par les valeurs de la ressource d’observateur.|

### <a name="start-instance-example"></a>Démarrer un exemple d’instance

Démarrez une instance Data Manager Blockchain nommée *mywatcher*.

``` azurecli-interactive
az resource invoke-action \
                          --action start \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="stop-instance"></a>Arrêter l’instance

Arrêtez une instance Data Manager Blockchain.

``` azurecli
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/<Watcher name>
```

| Paramètre | Description |
|-----------|-------------|
| action | Utilisez **stop** pour arrêter l’observateur. |
| ids | Nom de l’observateur. Remplacez \<Subscription ID\>, \<Resource group\> et \<Watcher name\> par les valeurs de la ressource d’observateur. |

### <a name="stop-watcher-example"></a>Exemple d’arrêt de l’observateur

Arrêtez une instance nommée *mywatcher*.

``` azurecli-interactive
az resource invoke-action \
                          --action stop \
                          --ids /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Blockchain/watchers/mywatcher
```

## <a name="delete-instance"></a>Supprimer l'instance

Supprimez une instance Data Manager Blockchain.

``` azurecli
az resource delete \
                   --resource-group <Resource group> \
                   --name <Watcher name> \
                   --resource-type Microsoft.Blockchain/watchers
```

| Paramètre | Description |
|-----------|-------------|
| resource-group | Nom du groupe de ressources de l’observateur à supprimer. |
| name | Nom de l’observateur à supprimer. |
| resource-type | Le type de ressource est **Microsoft.blockchain/watchers** pour un observateur Blockchain Data Manager. |

### <a name="delete-instance-example"></a>Supprimer un exemple d’instance

Supprimez une instance nommée *mywatcher* dans le groupe de ressources *myRG*.

``` azurecli-interactive
az resource delete \
                   --resource-group myRG \
                   --name mywatcher \
                   --resource-type Microsoft.blockchain/watchers
```

## <a name="next-steps"></a>Étapes suivantes

Essayez le tutoriel suivant, qui consiste à créer un explorateur de messages des transactions blockchain à l’aide de Blockchain Data Manager et d’Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Utiliser Blockchain Data Manager pour envoyer des données à Azure Cosmos DB](data-manager-cosmosdb.md)
