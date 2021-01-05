---
title: 'Interroger avec l’API Gremlin d’Azure Cosmos DB et la console TinkerPop Gremlin : Didacticiel'
description: Démarrage rapide d’Azure Cosmos DB pour créer des vertex, des arêtes et des requêtes à l’aide de l’API Gremlin Azure Cosmos DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: quickstart
ms.date: 07/10/2020
ms.author: chrande
ms.openlocfilehash: 20cb4cea52317022aea8a5b9e4b8280f8b88ff85
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93361509"
---
# <a name="quickstart-create-query-and-traverse-an-azure-cosmos-db-graph-database-using-the-gremlin-console"></a>Démarrage rapide : Créer, interroger et parcourir une base de données de graphe Azure Cosmos DB à l’aide de la console Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

> [!div class="op_single_selector"]
> * [Console Gremlin](create-graph-gremlin-console.md)
> * [.NET](create-graph-dotnet.md)
> * [Java](create-graph-java.md)
> * [Node.JS](create-graph-nodejs.md)
> * [Python](create-graph-python.md)
> * [PHP](create-graph-php.md)
>  

Azure Cosmos DB est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale. Rapidement, vous avez la possibilité de créer et d’interroger des documents, des paires clé/valeur, et des bases de données orientées graphe, profitant tous de la distribution à l’échelle mondiale et des capacités de mise à l’échelle horizontale au cœur d’Azure Cosmos DB. 

Ce guide de démarrage rapide explique comment créer un compte, une base de données et un graphe (conteneur) [API Gremlin](graph-introduction.md) Azure Cosmos DB à l’aide du portail Azure, puis comment utiliser la [console Gremlin](https://tinkerpop.apache.org/docs/current/reference/#gremlin-console) d’[Apache TinkerPop](https://tinkerpop.apache.org) pour travailler avec les données API Gremlin. Dans ce didacticiel, vous créez et interrogez des vertex et des bords, mettez à jour une propriété de vertex, interrogez des vertex, parcourez le graphique et supprimez un vertex.

:::image type="content" source="./media/create-graph-gremlin-console/gremlin-console.png" alt-text="Azure Cosmos DB à partir de la console Apache Gremlin":::

La console Gremlin est basée sur Groovy/Java et s’exécute sous Linux, Mac et Windows. Vous pouvez la télécharger à partir du [site Apache TinkerPop](https://tinkerpop.apache.org/downloads.html).

## <a name="prerequisites"></a>Prérequis

Vous devez posséder un abonnement Azure pour créer un compte Azure Cosmos DB pour ce démarrage rapide.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Vous devez également installer la [console Gremlin](https://tinkerpop.apache.org/downloads.html). **La version recommandée est la version 3.4.3** ou antérieure. (Pour utiliser la console Gremlin sur Windows, vous devez installer [Java Runtime](https://www.oracle.com/technetwork/java/javase/overview/index.html).)

## <a name="create-a-database-account"></a>Création d’un compte de base de données

[!INCLUDE [cosmos-db-create-dbaccount-graph](../../includes/cosmos-db-create-dbaccount-graph.md)]

## <a name="add-a-graph"></a>Ajout d’un graphique

[!INCLUDE [cosmos-db-create-graph](../../includes/cosmos-db-create-graph.md)]

## <a name="connect-to-your-app-servicegraph"></a><a id="ConnectAppService"></a>Connexion à votre service d’application/graphe

1. Avant de démarrer la console Gremlin, créez ou modifiez le fichier config remote-secure.yaml dans le répertoire `apache-tinkerpop-gremlin-console-3.2.5/conf`.
2. Renseignez vos configurations *host* , *port* , *username* , *password* , *connectionPool* et *serializer* telles que définies dans le tableau suivant :

    Paramètre|Valeur suggérée|Description
    ---|---|---
    hôtes|[ *account-name*. **gremlin**.cosmos.azure.com]|Voir la capture d’écran suivante. Il s’agit de la valeur **URI Gremlin** sur la page Vue d’ensemble du portail Azure, entre crochets, avec la fin : 443/ supprimée. Remarque : Veillez à utiliser la valeur Gremlin, et **non** l’URI qui se termine par [ *account-name*.documents.azure.com], afin d’éviter que l’exécution ultérieure de requêtes Gremlin se solde par une exception indiquant que l’hôte n’a pas répondu à temps. 
    port|443|Définissez la valeur sur 443.
    username|*Votre nom d’utilisateur*|Ressource sous la forme `/dbs/<db>/colls/<coll>`, où `<db>` est le nom de votre base de données et `<coll>` le nom de votre collection.
    mot de passe|*Votre clé primaire*| Reportez-vous à la deuxième capture d’écran ci-dessous. Il s’agit de votre clé primaire, que vous pouvez récupérer à partir de la page Clés du portail Azure, dans la zone Clé primaire. Utilisez le bouton Copier sur le côté gauche de la zone pour copier la valeur.
    connectionPool|{enableSsl: true}|Votre paramètre de pool de connexions pour TLS.
    serializer|{ className: org.apache.tinkerpop.gremlin.<br>driver.ser.GraphSONMessageSerializerV2d0,<br> config: {serializeResultToString: true }}|Définissez le paramètre sur cette valeur et supprimez tous les sauts de ligne `\n` quand vous collez la valeur.

   Pour la valeur de Hôtes, copiez la valeur de l’ **URI Gremlin** depuis la page **Vue d’ensemble**  :

   :::image type="content" source="./media/create-graph-gremlin-console/gremlin-uri.png" alt-text="Afficher et copier la valeur d’URI Gremlin sur la page Vue d’ensemble du portail Azure":::

   Pour la valeur du mot de passe, copiez la **Clé principale** depuis la page **Clés**  :

   :::image type="content" source="./media/create-graph-gremlin-console/keys.png" alt-text="Afficher et copier votre clé principale dans le portail Azure, page Clés":::

   Votre fichier remote-secure.yaml doit ressembler à ceci :

   ```yaml
   hosts: [your_database_server.gremlin.cosmos.azure.com] 
   port: 443
   username: /dbs/your_database_account/colls/your_collection
   password: your_primary_key
   connectionPool: {
     enableSsl: true
   }
   serializer: { className: org.apache.tinkerpop.gremlin.driver.ser.GraphSONMessageSerializerV2d0, config: { serializeResultToString: true }}
   ```

   Veillez à inclure la valeur du paramètre d’hôtes entre crochets []. 

1. Dans votre terminal, exécutez `bin/gremlin.bat` ou `bin/gremlin.sh` pour démarrer la [console Gremlin](https://tinkerpop.apache.org/docs/3.2.5/tutorials/getting-started/).

1. Dans votre terminal, exécutez `:remote connect tinkerpop.server conf/remote-secure.yaml` pour vous connecter à votre service d’application.

    > [!TIP]
    > Si vous recevez l’erreur `No appenders could be found for logger`, vérifiez que vous avez mis à jour la valeur de sérialiseur dans le fichier remote-secure.yaml comme décrit à l’étape 2. Si votre configuration est correcte, cet avertissement peut être ignoré en toute sécurité, car il ne doit pas impacter l’utilisation de la console. 

1. Exécutez ensuite `:remote console` pour rediriger toutes les commandes de la console au serveur distant.

   > [!NOTE]
   > Si vous n’exécutez pas la commande `:remote console` et que vous souhaitez rediriger toutes les commandes de la console au serveur distant, vous devez faire précéder la commande par `:>`. Par exemple, vous devez exécuter la commande comme ceci : `:> g.V().count()`. Ce préfixe, qui fait partie de la commande, est important quand vous utilisez la console Gremlin avec Azure Cosmos DB. Si vous n’indiquez pas ce préfixe, la console doit exécuter la commande en local, le plus souvent sur un graphe en mémoire. L’utilisation du préfixe `:>` indique à la console qu’elle doit exécuter une commande à distance, sur Azure Cosmos DB (l’émulateur localhost ou une instance Azure) dans le cas présent.

Parfait ! L’installation étant terminée, exécutons quelques commandes de console.

Essayons une simple commande count(). Lorsque l’invite de commandes s’affiche, saisissez ce qui suit dans la console :

```java
g.V().count()
```

## <a name="create-vertices-and-edges"></a>Création de vertex et de bords

Commençons par ajouter des vertex pour cinq personnes : *Thomas* , *Mary Kay* , *Robin* , *Ben* et *Jack*.

Entrée (Thomas) :

```java
g.addV('person').property('firstName', 'Thomas').property('lastName', 'Andersen').property('age', 44).property('userid', 1).property('pk', 'pk')
```

Sortie :

```bash
==>[id:796cdccc-2acd-4e58-a324-91d6f6f5ed6d,label:person,type:vertex,properties:[firstName:[[id:f02a749f-b67c-4016-850e-910242d68953,value:Thomas]],lastName:[[id:f5fa3126-8818-4fda-88b0-9bb55145ce5c,value:Andersen]],age:[[id:f6390f9c-e563-433e-acbf-25627628016e,value:44]],userid:[[id:796cdccc-2acd-4e58-a324-91d6f6f5ed6d|userid,value:1]]]]
```

Entrée (Mary Kay) :

```java
g.addV('person').property('firstName', 'Mary Kay').property('lastName', 'Andersen').property('age', 39).property('userid', 2).property('pk', 'pk')

```

Sortie :

```bash
==>[id:0ac9be25-a476-4a30-8da8-e79f0119ea5e,label:person,type:vertex,properties:[firstName:[[id:ea0604f8-14ee-4513-a48a-1734a1f28dc0,value:Mary Kay]],lastName:[[id:86d3bba5-fd60-4856-9396-c195ef7d7f4b,value:Andersen]],age:[[id:bc81b78d-30c4-4e03-8f40-50f72eb5f6da,value:39]],userid:[[id:0ac9be25-a476-4a30-8da8-e79f0119ea5e|userid,value:2]]]]

```

Entrée (Robin) :

```java
g.addV('person').property('firstName', 'Robin').property('lastName', 'Wakefield').property('userid', 3).property('pk', 'pk')
```

Sortie :

```bash
==>[id:8dc14d6a-8683-4a54-8d74-7eef1fb43a3e,label:person,type:vertex,properties:[firstName:[[id:ec65f078-7a43-4cbe-bc06-e50f2640dc4e,value:Robin]],lastName:[[id:a3937d07-0e88-45d3-a442-26fcdfb042ce,value:Wakefield]],userid:[[id:8dc14d6a-8683-4a54-8d74-7eef1fb43a3e|userid,value:3]]]]
```

Entrée (Ben) :

```java
g.addV('person').property('firstName', 'Ben').property('lastName', 'Miller').property('userid', 4).property('pk', 'pk')

```

Sortie :

```bash
==>[id:ee86b670-4d24-4966-9a39-30529284b66f,label:person,type:vertex,properties:[firstName:[[id:a632469b-30fc-4157-840c-b80260871e9a,value:Ben]],lastName:[[id:4a08d307-0719-47c6-84ae-1b0b06630928,value:Miller]],userid:[[id:ee86b670-4d24-4966-9a39-30529284b66f|userid,value:4]]]]
```

Entrée (Jack) :

```java
g.addV('person').property('firstName', 'Jack').property('lastName', 'Connor').property('userid', 5).property('pk', 'pk')
```

Sortie :

```bash
==>[id:4c835f2a-ea5b-43bb-9b6b-215488ad8469,label:person,type:vertex,properties:[firstName:[[id:4250824e-4b72-417f-af98-8034aa15559f,value:Jack]],lastName:[[id:44c1d5e1-a831-480a-bf94-5167d133549e,value:Connor]],userid:[[id:4c835f2a-ea5b-43bb-9b6b-215488ad8469|userid,value:5]]]]
```


Ensuite, ajoutons des bords pour les relations entre ces personnes.

Entrée (Thomas -> Mary Kay) :

```java
g.V().hasLabel('person').has('firstName', 'Thomas').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Mary Kay'))
```

Sortie :

```bash
==>[id:c12bf9fb-96a1-4cb7-a3f8-431e196e702f,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:0d1fa428-780c-49a5-bd3a-a68d96391d5c,outV:1ce821c6-aa3d-4170-a0b7-d14d2a4d18c3]
```

Entrée (Thomas -> Robin) :

```java
g.V().hasLabel('person').has('firstName', 'Thomas').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Robin'))
```

Sortie :

```bash
==>[id:58319bdd-1d3e-4f17-a106-0ddf18719d15,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:3e324073-ccfc-4ae1-8675-d450858ca116,outV:1ce821c6-aa3d-4170-a0b7-d14d2a4d18c3]
```

Entrée (Robin -> Ben) :

```java
g.V().hasLabel('person').has('firstName', 'Robin').addE('knows').to(g.V().hasLabel('person').has('firstName', 'Ben'))
```

Sortie :

```bash
==>[id:889c4d3c-549e-4d35-bc21-a3d1bfa11e00,label:knows,type:edge,inVLabel:person,outVLabel:person,inV:40fd641d-546e-412a-abcc-58fe53891aab,outV:3e324073-ccfc-4ae1-8675-d450858ca116]
```

## <a name="update-a-vertex"></a>Mise à jour d’un vertex

Nous allons mettre à jour le vertex *Thomas* avec un nouvel âge : *45*.

Entrée :
```java
g.V().hasLabel('person').has('firstName', 'Thomas').property('age', 45)
```
Sortie :

```bash
==>[id:ae36f938-210e-445a-92df-519f2b64c8ec,label:person,type:vertex,properties:[firstName:[[id:872090b6-6a77-456a-9a55-a59141d4ebc2,value:Thomas]],lastName:[[id:7ee7a39a-a414-4127-89b4-870bc4ef99f3,value:Andersen]],age:[[id:a2a75d5a-ae70-4095-806d-a35abcbfe71d,value:45]]]]
```

## <a name="query-your-graph"></a>Interrogation de votre graphique

À présent, exécutons différentes requêtes sur votre graphique.

Tout d’abord, essayons une requête avec un filtre pour retourner uniquement les personnes de plus de 40 ans.

Entrée (requête de filtre) :

```java
g.V().hasLabel('person').has('age', gt(40))
```

Sortie :

```bash
==>[id:ae36f938-210e-445a-92df-519f2b64c8ec,label:person,type:vertex,properties:[firstName:[[id:872090b6-6a77-456a-9a55-a59141d4ebc2,value:Thomas]],lastName:[[id:7ee7a39a-a414-4127-89b4-870bc4ef99f3,value:Andersen]],age:[[id:a2a75d5a-ae70-4095-806d-a35abcbfe71d,value:45]]]]
```

Ensuite, projetons le premier nom parmi les personnes de plus de 40 ans.

Entrée (filtre + requête de projection) :

```java
g.V().hasLabel('person').has('age', gt(40)).values('firstName')
```

Sortie :

```bash
==>Thomas
```

## <a name="traverse-your-graph"></a>Traversée du graphique

Parcourons le graphique pour retourner tous les amis de Thomas.

Entrée (amis de Thomas) :

```java
g.V().hasLabel('person').has('firstName', 'Thomas').outE('knows').inV().hasLabel('person')
```

Sortie : 

```bash
==>[id:f04bc00b-cb56-46c4-a3bb-a5870c42f7ff,label:person,type:vertex,properties:[firstName:[[id:14feedec-b070-444e-b544-62be15c7167c,value:Mary Kay]],lastName:[[id:107ab421-7208-45d4-b969-bbc54481992a,value:Andersen]],age:[[id:4b08d6e4-58f5-45df-8e69-6b790b692e0a,value:39]]]]
==>[id:91605c63-4988-4b60-9a30-5144719ae326,label:person,type:vertex,properties:[firstName:[[id:f760e0e6-652a-481a-92b0-1767d9bf372e,value:Robin]],lastName:[[id:352a4caa-bad6-47e3-a7dc-90ff342cf870,value:Wakefield]]]]
```

Ensuite, nous allons obtenir la couche suivante de vertex. Parcourons le graphique pour retourner tous les amis des amis de Thomas.

Entrée (amis des amis de Thomas) :

```java
g.V().hasLabel('person').has('firstName', 'Thomas').outE('knows').inV().hasLabel('person').outE('knows').inV().hasLabel('person')
```
Sortie :

```bash
==>[id:a801a0cb-ee85-44ee-a502-271685ef212e,label:person,type:vertex,properties:[firstName:[[id:b9489902-d29a-4673-8c09-c2b3fe7f8b94,value:Ben]],lastName:[[id:e084f933-9a4b-4dbc-8273-f0171265cf1d,value:Miller]]]]
```

## <a name="drop-a-vertex"></a>Suppression d’un vertex

Nous allons à présent supprimer un vertex de la base de données des graphiques.

Entrée (suppression du vertex de Jack) :

```java
g.V().hasLabel('person').has('firstName', 'Jack').drop()
```

## <a name="clear-your-graph"></a>Effacement de tout le contenu de votre graphique

Enfin, nous allons supprimer tous les vertex et bords de la base de données.

Entrée :

```java
g.E().drop()
g.V().drop()
```

Félicitations ! Vous avez terminé ce tutoriel Azure Cosmos DB : Tutoriel sur l’API Gremlin !

## <a name="review-slas-in-the-azure-portal"></a>Vérification des contrats SLA dans le portail Azure

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à créer un compte Azure Cosmos DB, à créer un graphique à l’aide de l’Explorateur de données, à créer des vertex et des bords et à parcourir votre graphique à l’aide de la console Gremlin. Vous pouvez maintenant générer des requêtes plus complexes et implémenter une logique de traversée de graphique puissante, à l’aide de Gremlin. 

> [!div class="nextstepaction"]
> [Interroger à l’aide de Gremlin](tutorial-query-graph.md)
