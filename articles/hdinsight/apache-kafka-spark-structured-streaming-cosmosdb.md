---
title: Apache Spark et Apache Kafka vers Azure Cosmos DB – Azure HDInsight
description: Découvrez comment utiliser Apache Spark Structured Streaming pour lire des données à partir d’Apache Kafka puis les stocker dans Azure Cosmos DB. Dans cet exemple, vous diffusez des données à l’aide d’un bloc-notes Jupyter à partir de Spark sur HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 11/18/2019
ms.openlocfilehash: 387eb4f4c73b2103a7461c0d06c4d0e0562ec9db
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842462"
---
# <a name="use-apache-spark-structured-streaming-with-apache-kafka-and-azure-cosmos-db"></a>Utiliser Apache Spark Structured Streaming avec Apache Kafka et Azure Cosmos DB

Découvrez comment utiliser [Apache Spark](https://spark.apache.org/) [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html) pour lire des données à partir d’[Apache Kafka](https://kafka.apache.org/) sur Azure HDInsight, puis les stocker dans Azure Cosmos DB.

[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) est une base de données multimodèle distribuée à l’échelle mondiale. Cet exemple utilise un modèle de base de données d’API SQL. Pour plus d’informations, consultez le document [Bienvenue dans Azure Cosmos DB](../cosmos-db/introduction.md).

Spark Structured Streaming est un moteur de traitement de flux basé sur Spark SQL. Il vous permet d’exprimer des calculs de diffusion en continu de la même façon que pour les calculs de lot sur les données statiques. Pour plus d’informations sur Structured Streaming, consultez le [Guide de programmation Structured Streaming](https://spark.apache.org/docs/2.2.0/structured-streaming-programming-guide.html) sur le site Apache.org.

> [!IMPORTANT]  
> Cet exemple utilise Spark 2.2 sur HDInsight 3.6.
>
> Les étapes décrites dans ce document créent un groupe de ressources Azure qui contient à la fois un Spark sur HDInsight et un Kafka sur un cluster HDInsight. Ces clusters sont tous deux situés dans un réseau virtuel Azure, ce qui permet au cluster Spark de communiquer directement avec le cluster Kafka.
>
> Lorsque vous avez terminé les étapes décrites dans ce document, n’oubliez pas de supprimer les clusters pour éviter les frais supplémentaires.

## <a name="create-the-clusters"></a>Création des clusters

Apache Kafka sur HDInsight ne donne pas accès aux répartiteurs Kafka sur l’Internet public. Tout ce qui communique avec Kafka doit se trouver sur le même réseau virtuel Azure que les nœuds du cluster Kafka. Pour cet exemple, les clusters Kafka et Spark sont situés dans un réseau virtuel Azure. Le diagramme suivant illustre le flux de communication entre les clusters :

![Diagramme des clusters Spark et Kafka dans un réseau virtuel Azure](./media/apache-kafka-spark-structured-streaming-cosmosdb/apache-spark-kafka-vnet.png)

> [!NOTE]  
> Le service Kafka est limité à la communication au sein du réseau virtuel. L’accès aux autres services sur le cluster, tels que SSH et Ambari, se fait via Internet. Pour plus d’informations sur les ports publics disponibles avec HDInsight, consultez [Ports et URI utilisés par HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Même si vous pouvez créer un réseau virtuel Azure, et des clusters Kafka et Spark manuellement, il est plus facile d’utiliser un modèle Azure Resource Manager. Utilisez les étapes suivantes pour déployer un réseau virtuel Azure et des clusters Kafka et Spark sur votre abonnement Azure.

1. Utilisez le bouton suivant pour vous connecter à Azure et ouvrir le modèle dans le portail Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-scala-kafka-cosmosdb%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="./media/apache-kafka-spark-structured-streaming-cosmosdb/resource-manager-deploy.png" alt="Deploy to Azure"/>
    </a>

    Le modèle Azure Resource Manager se trouve dans le référentiel GitHub pour ce projet ([https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb](https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb)).

    Ce modèle crée les ressources suivantes :

   * Un cluster Kafka sur HDInsight 3.6.

   * Un cluster Spark sur HDInsight 3.6.

   * Un réseau virtuel Azure, qui contient les clusters HDInsight. Le réseau virtuel créé par le modèle utilise l’espace d’adressage 10.0.0.0/16.

   * Une base de données d’API SQL Azure Cosmos DB.

    > [!IMPORTANT]  
    > Le bloc-notes de diffusion en continu structurée utilisé dans cet exemple nécessite Spark sur HDInsight 3.6. Si vous utilisez une version antérieure de Spark sur HDInsight, vous recevez des erreurs lors de l’utilisation du bloc-notes.

1. Utilisez les informations suivantes pour remplir les entrées dans la section **Déploiement personnalisé** :

    |Propriété |Valeur |
    |---|---|
    |Abonnement|Sélectionnez votre abonnement Azure.|
    |Resource group|créez un groupe ou sélectionnez un groupe existant. Ce groupe contient le cluster HDInsight.|
    |Nom de compte Azure Cosmos DB|cette valeur est utilisée comme nom du compte Cosmos DB. Le nom peut uniquement contenir des lettres minuscules, des chiffres et le caractère de trait d’union (-). Sa longueur doit être comprise entre 3 et 31 caractères.|
    |Nom du cluster de base|cette valeur est utilisée comme nom de base pour les clusters Spark et Kafka. Par exemple, si vous entrez **myhdi**, un cluster Spark nommé __spark-hdi__ et un cluster Kafka nommé **kafka-hdi** sont créés.|
    |Version de cluster|Version du cluster HDInsight Cet exemple est testé avec HDInsight 3.6 et peuvent ne pas fonctionne avec d’autres types de cluster.|
    |Nom d’utilisateur de connexion au cluster|nom de l’utilisateur administrateur pour les clusters Spark et Kafka.|
    |Mot de passe de connexion au cluster|mot de passe de l’utilisateur administrateur pour les clusters Spark et Kafka.|
    |Nom d’utilisateur SSH|utilisateur SSH à créer pour les clusters Spark et Kafka.|
    |Mot de passe SSH|mot de passe de l’utilisateur SSH pour les clusters Spark et Kafka.|

    ![Valeurs pour le déploiement HDInsight personnalisé](./media/apache-kafka-spark-structured-streaming-cosmosdb/hdi-custom-parameters.png)

1. Passez en revue les **termes et conditions**, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.

1. Enfin, sélectionnez **Achat**. La création des clusters, du réseau virtuel et du compte Cosmos DB peut prendre jusqu’à 45 minutes.

## <a name="create-the-cosmos-db-database-and-collection"></a>Créer la base de données et la collection Cosmos DB

Le projet utilisé dans ce document stocke des données dans Cosmos DB. Avant d’exécuter le code, vous devez d’abord créer une _base de données_ et une _collection_ dans votre instance Cosmos DB. Vous devez également récupérer le point de terminaison du document et la _clé_ utilisés pour authentifier les requêtes sur Cosmos DB.

L’une des méthodes possibles est d’utiliser [Azure CLI](/cli/azure/). Le script suivant crée une base de données nommée `kafkadata` et une collection nommée `kafkacollection`. Il retourne ensuite la clé primaire.

```azurecli
#!/bin/bash

# Replace 'myresourcegroup' with the name of your resource group
resourceGroupName='myresourcegroup'
# Replace 'mycosmosaccount' with the name of your Cosmos DB account name
name='mycosmosaccount'

# WARNING: If you change the databaseName or collectionName
#          then you must update the values in the Jupyter notebook
databaseName='kafkadata'
collectionName='kafkacollection'

# Create the database
az cosmosdb sql database create --account-name $name --name $databaseName --resource-group $resourceGroupName

# Create the collection
az cosmosdb sql container create --account-name $name --database-name $databaseName --name $collectionName --partition-key-path "/my/path" --resource-group $resourceGroupName

# Get the endpoint
az cosmosdb show --name $name --resource-group $resourceGroupName --query documentEndpoint

# Get the primary key
az cosmosdb keys list --name $name --resource-group $resourceGroupName --type keys
```

Le point de terminaison du document et les informations de clé primaire est semblable au texte suivant :

```text
# endpoint
"https://mycosmosaccount.documents.azure.com:443/"
# key
"YqPXw3RP7TsJoBF5imkYR0QNA02IrreNAlkrUMkL8EW94YHs41bktBhIgWq4pqj6HCGYijQKMRkCTsSaKUO2pw=="
```

> [!IMPORTANT]  
> Enregistrez le point de terminaison et les valeurs clés, car ils sont nécessaires dans Notebooks Jupyter.

## <a name="get-the-notebooks"></a>Obtenir les blocs-notes

Le code de l’exemple décrit dans ce document est disponible à l’adresse [https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb](https://github.com/Azure-Samples/hdinsight-spark-scala-kafka-cosmosdb).

## <a name="upload-the-notebooks"></a>Charger les blocs-notes

Pour charger les blocs-notes à partir du projet vers votre cluster Spark sur HDInsight, procédez comme suit :

1. Dans votre navigateur web, connectez-vous au bloc-notes Jupyter sur votre cluster Spark. Dans l’URL suivante, remplacez `CLUSTERNAME` par le nom de votre cluster __Spark__ :

    ```http
    https://CLUSTERNAME.azurehdinsight.net/jupyter
    ```

    Lorsque vous y êtes invité, entrez l’identifiant de connexion (admin) et le mot de passe du cluster utilisés lors de la création du cluster.

2. Dans la partie supérieure droite de la page, utilisez le bouton __Charger__ pour charger le fichier __Stream-taxi-data-to-kafka.ipynb__ dans le cluster. Sélectionnez __Ouvrir__ pour démarrer le chargement.

3. Recherchez l’entrée __Stream-taxi-data-to-kafka.ipynb__ dans la liste des blocs-notes, puis sélectionnez le bouton __Charger__ en regard de celle-ci.

4. Répétez les étapes 1 à 3 pour charger le bloc-notes __Stream-data-from-Kafka-to-Cosmos-DB.ipynb__.

## <a name="load-taxi-data-into-kafka"></a>Charger des données de taxi dans Kafka

Une fois que les fichiers ont été chargés, sélectionnez l’entrée __Stream-taxi-data-to-kafka.ipynb__ pour ouvrir le bloc-notes. Suivez les étapes dans le bloc-notes pour charger des données dans Kafka.

## <a name="process-taxi-data-using-spark-structured-streaming"></a>Traiter des données de taxi à l’aide de Spark Structured Streaming

Dans la page d’accueil [Jupyter Notebook](https://jupyter.org/), sélectionnez l’entrée __Stream-data-from-Kafka-to-Cosmos-DB.ipynb__. Suivez les étapes dans le bloc-notes pour transmettre en continu des données à partir de Kafka dans Azure Cosmos DB à l’aide de Spark Structured Streaming.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à utiliser Apache Spark Structured Streaming, consultez les documents suivants pour en savoir plus sur l’utilisation d’Apache Spark, d’Apache Kafka et d’Azure Cosmos DB :

* [Comment utiliser le streaming Apache Spark (DStream) avec Apache Kafka](hdinsight-apache-spark-with-kafka.md)
* [Bien démarrer avec Jupyter Notebook et Apache Spark sur HDInsight](spark/apache-spark-jupyter-spark-sql.md)
* [Bienvenue dans Azure Cosmos DB](../cosmos-db/introduction.md)
