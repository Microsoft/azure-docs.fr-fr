---
title: Flux de modification dans l’API Azure Cosmos DB pour Cassandra
description: Découvrez comment utiliser le flux de modification dans l’API Azure Cosmos DB pour Cassandra afin d’obtenir les modifications apportées à vos données.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 11/25/2019
ms.author: thvankra
ms.openlocfilehash: 10f037dddcce43a1e023982af816660bd325d57f
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605091"
---
# <a name="change-feed-in-the-azure-cosmos-db-api-for-cassandra"></a>Flux de modification dans l’API Azure Cosmos DB pour Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

La prise en charge du [flux de modification](change-feed.md) dans l’API Azure Cosmos DB pour Cassandra est disponible par le biais des prédicats de requête en CQL (Cassandra Query Language). À l’aide de ces conditions de prédicat, vous pouvez interroger l’API de flux de modification. Les applications peuvent obtenir les modifications apportées à une table à l’aide de la clé primaire (également appelée clé de partition) comme le nécessite le langage CQL. Vous pouvez ensuite effectuer d’autres actions en fonction des résultats. Les modifications apportées aux lignes du tableau sont capturées dans l'ordre de leur modification et dans l'ordre de tri par clé de partition.

L’exemple suivant montre comment obtenir un flux de modification sur toutes les lignes d’une table d’espace de clés d’API Cassandra à l’aide de .NET. Le prédicat COSMOS_CHANGEFEED_START_TIME() est utilisé directement dans le langage CQL pour interroger les éléments du flux de modification à partir d’une heure de début spécifiée (dans ce cas, la date et l’heure actuelles). Vous pouvez télécharger l’exemple complet, pour C# [ici](/samples/azure-samples/azure-cosmos-db-cassandra-change-feed/cassandra-change-feed/) et pour Java [ici](https://github.com/Azure-Samples/cosmos-changefeed-cassandra-java).

Dans chaque itération, la requête reprend au dernier point auquel les modifications ont été lues, à l’aide de l’état de pagination. Nous pouvons voir un flux continu de nouvelles modifications apportées à la table dans l’espace de clés. Nous allons voir les modifications apportées aux lignes insérées ou mises à jour. La surveillance des opérations de suppression à l’aide d’un flux de modification dans l’API Cassandra n’est pas prise en charge actuellement.

# <a name="java"></a>[Java](#tab/java)

```java
    Session cassandraSession = utils.getSession();

    try {
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");  
        LocalDateTime now = LocalDateTime.now().minusHours(6).minusMinutes(30);  
        String query="SELECT * FROM uprofile.user where COSMOS_CHANGEFEED_START_TIME()='" 
            + dtf.format(now)+ "'";
        
        byte[] token=null; 
        System.out.println(query); 
        while(true)
        {
            SimpleStatement st=new  SimpleStatement(query);
            st.setFetchSize(100);
            if(token!=null)
                st.setPagingStateUnsafe(token);
            
            ResultSet result=cassandraSession.execute(st) ;
            token=result.getExecutionInfo().getPagingState().toBytes();
            
            for(Row row:result)
            {
                System.out.println(row.getString("user_name"));
            }
        }
    } finally {
        utils.close();
        LOGGER.info("Please delete your table after verifying the presence of the data in portal or from CQL");
    }
```

# <a name="c"></a>[C#](#tab/csharp)

```C#
    //set initial start time for pulling the change feed
     DateTime timeBegin = DateTime.UtcNow;

    //initialise variable to store the continuation token
    byte[] pageState = null;
    while (true)
    {
        try
        {

            //Return the latest change for all rows in 'user' table    
            IStatement changeFeedQueryStatement = new SimpleStatement(
            $"SELECT * FROM uprofile.user where COSMOS_CHANGEFEED_START_TIME() = '{timeBegin.ToString("yyyy-MM-ddTHH:mm:ss.fffZ", CultureInfo.InvariantCulture)}'");
            if (pageState != null)
            {
                changeFeedQueryStatement = changeFeedQueryStatement.SetPagingState(pageState);
            }
            Console.WriteLine("getting records from change feed at last page state....");
            RowSet rowSet = session.Execute(changeFeedQueryStatement);

            //store the continuation token here
            pageState = rowSet.PagingState;

            List<Row> rowList = rowSet.ToList();
            if (rowList.Count != 0)
            {
                for (int i = 0; i < rowList.Count; i++)
                {
                    string value = rowList[i].GetValue<string>("user_name");
                    int key = rowList[i].GetValue<int>("user_id");
                    // do something with the data - e.g. compute, forward to another event, function, etc.
                    // here, we just print the user name field
                    Console.WriteLine("user_name: " + value);
                }
            }
            else
            {
                Console.WriteLine("zero documents read");
            }
        }
        catch (Exception e)
        {
            Console.WriteLine("Exception " + e);
        }
    }

```
---

Pour obtenir les modifications apportées à une seule ligne par clé primaire, vous pouvez ajouter la clé primaire dans la requête. L’exemple suivant montre comment effectuer le suivi des modifications pour la ligne où « user_id = 1 ».

# <a name="c"></a>[C#](#tab/csharp)

```C#
    //Return the latest change for all row in 'user' table where user_id = 1
    IStatement changeFeedQueryStatement = new SimpleStatement(
    $"SELECT * FROM uprofile.user where user_id = 1 AND COSMOS_CHANGEFEED_START_TIME() = '{timeBegin.ToString("yyyy-MM-ddTHH:mm:ss.fffZ", CultureInfo.InvariantCulture)}'");

```

# <a name="java"></a>[Java](#tab/java)

```java
    String query="SELECT * FROM uprofile.user where user_id=1 and COSMOS_CHANGEFEED_START_TIME()='" 
                       + dtf.format(now)+ "'";
    SimpleStatement st=new  SimpleStatement(query);
```
---
## <a name="current-limitations"></a>Limites actuelles

Les limitations suivantes s’appliquent quand un flux de modification est utilisé avec l’API Cassandra :

* Les insertions et les mises à jour sont actuellement prises en charge. L’opération de suppression n’est pas encore prise en charge. Pour contourner ce problème, vous pouvez ajouter un marqueur réversible sur les lignes en cours de suppression. Par exemple, ajoutez un champ dans la ligne nommée « deleted » et affectez-lui la valeur « true ».
* La dernière mise à jour est conservée comme dans l’API SQL de base et les mises à jour intermédiaires de l’entité ne sont pas disponibles.


## <a name="error-handling"></a>Gestion des erreurs

Les codes d’erreur et messages suivants sont pris en charge quand un flux de modification est utilisé dans l’API Cassandra :

* **Code d’erreur HTTP 429** : quand la vitesse du flux de modification est limitée, une page vide est retournée.

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les ressources de l’API Cassandra Azure Cosmos DB à l’aide de modèles Azure Resource Manager](./templates-samples-cassandra.md)