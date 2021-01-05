---
title: Migrer des tables Azure SQL Database vers Azure Cosmos DB avec Azure Data Factory
description: Prenez un schéma de base de données normalisé existant d’Azure SQL Database et migrez-le vers un conteneur dénormalisé Azure Cosmos DB avec Azure Data Factory.
services: data-factory
author: kromerm
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 04/29/2020
ms.author: makromer
ms.openlocfilehash: 5d93cb49c77a4c8164a8b4e9bca349a805f39678
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041623"
---
# <a name="migrate-normalized-database-schema-from-azure-sql-database-to-azure-cosmosdb-denormalized-container"></a>Migrer un schéma de base de données normalisé d’Azure SQL Database vers un conteneur dénormalisé Azure Cosmos DB

Ce guide explique comment prendre un schéma de base de données normalisé existant d’Azure SQL Database et le convertir en un schéma dénormalisé Azure Cosmos DB en vue de son chargement dans Azure Cosmos DB.

Les schémas SQL sont généralement modélisés à l’aide d’une troisième forme normale, pour aboutir à des schémas normalisés qui fournissent des niveaux élevés d’intégrité des données et moins de valeurs de données dupliquées. Des requêtes peuvent joindre des entités de différentes tables en vue de leur lecture. Cosmos DB est optimisé pour les transactions et requêtes super rapides dans une collection ou un conteneur via des schémas dénormalisés avec des données autonomes dans un document.

Nous allons utiliser Azure Data Factory pour créer un pipeline qui utilise un seul flux de données de mappage pour lire à partir de deux tables normalisées Azure SQL Database qui contiennent des clés primaires et étrangères en tant que relation d’entité. ADF va joindre ces tables dans un seul flux en utilisant le moteur Spark de flux de données, collecter les lignes jointes dans des tableaux et produire des documents nettoyés à insérer dans un nouveau conteneur Azure Cosmos DB.

Ce guide montre comment créer à la volée un conteneur nommé « orders » (commandes), qui utilisera les tables ```SalesOrderHeader``` et ```SalesOrderDetail``` de l’exemple de base de données standard SQL Server AdventureWorks. Ces tables représentent des transactions de vente jointes par ```SalesOrderID```. Chaque enregistrement de détail a sa propre clé primaire ```SalesOrderDetailID```. La relation entre l’en-tête et le détail est ```1:M```. Nous allons effectuer la jonction sur ```SalesOrderID``` dans ADF, puis reporter chaque enregistrement de détail associé dans un tableau nommé « detail ».

La requête SQL représentative pour ce guide est la suivante :

```
  SELECT
  o.SalesOrderID,
  o.OrderDate,
  o.Status,
  o.ShipDate,
  o.SalesOrderNumber,
  o.ShipMethod,
  o.SubTotal,
  (select SalesOrderDetailID, UnitPrice, OrderQty from SalesLT.SalesOrderDetail od where od.SalesOrderID = o.SalesOrderID for json auto) as OrderDetails
FROM SalesLT.SalesOrderHeader o;
```

Le conteneur Cosmos DB obtenu incorpore la requête interne dans un document et se présente comme suit :

![Collection](media/data-flow/cosmosb3.png)

## <a name="create-a-pipeline"></a>Créer un pipeline

1. Sélectionnez **+Nouveau pipeline** pour créer un pipeline.

2. Ajoutez une activité de flux de données.

3. Dans l’activité de flux de données, sélectionnez **Nouveau flux de données de mappage**.

4. Nous allons construire le graphique de données ci-dessous.

![Graphique de Data Flow](media/data-flow/cosmosb1.png)

5. Définissez la source pour « SourceOrderDetails ». Pour dataset, créez un jeu de données Azure SQL Database qui pointe vers la table ```SalesOrderDetail```.

6. Définissez la source pour « SourceOrderHeader ». Pour dataset, créez un jeu de données Azure SQL Database qui pointe vers la table ```SalesOrderHeader```.

7. Dans la source supérieure, ajoutez une transformation Colonne dérivée après « SourceOrderDetails ». Appelez la nouvelle transformation « TypeCast ». Nous devons arrondir la colonne ```UnitPrice``` et la caster vers un type de données Double pour Cosmos DB. Définissez la formule sur : ```toDouble(round(UnitPrice,2))```.

8. Ajoutez une autre colonne dérivée nommée « MakeStruct ». C’est là que nous allons créer une structure hiérarchique pour stocker les valeurs de la table des détails. N’oubliez pas que les détails sont un relation ```M:1``` à l’en-tête. Nommez la nouvelle structure ```orderdetailsstruct```, puis créez la hiérarchie de cette manière, en affectant à chaque sous-colonne le nom de la colonne entrante :

![Créer la structure](media/data-flow/cosmosb9.png)

9. Nous allons maintenant accéder à la source de l’en-tête Sales. Ajoutez une transformation de jointure. Pour le côté droit, sélectionnez « MakeStruct ». Conservez le paramétrage de jointure interne et choisissez ```SalesOrderID``` pour les deux côtés de la condition de jointure.

10. Cliquez sur l’onglet Aperçu des données dans la nouvelle jointure que vous avez ajoutée pour voir vos résultats jusqu’à ce point. Vous devriez voir toutes les lignes d’en-tête jointes avec les lignes de détails. Il s’agit du résultat de la jointure formée à partir de ```SalesOrderID```. Nous allons ensuite combiner les détails des lignes communes dans le struct de détails et agréger les lignes communes.

![Join](media/data-flow/cosmosb4.png)

11. Avant de créer les tableaux pour dénormaliser ces lignes, nous devons supprimer les colonnes inutiles et vérifier que les valeurs de données correspondent aux types de données Cosmos DB.

12. Ajoutez ensuite une transformation Select et définissez le mappage de champs pour qu’il ressemble à ceci :

![Nettoyage de colonne](media/data-flow/cosmosb5.png)

13. Nous allons maintenant caster à nouveau une colonne de devises, en l’occurrence ```TotalDue```. Comme à l’étape 7, définissez la formule sur : ```toDouble(round(TotalDue,2))```.

14. Ici, nous allons dénormaliser les lignes en les regroupant par la clé commune ```SalesOrderID```. Ajoutez une transformation d’agrégation et définissez Grouper par sur ```SalesOrderID```.

15. Dans la formule d’agrégation, ajoutez une nouvelle colonne nommée « details » et utilisez cette formule pour collecter les valeurs dans la structure que nous avons créée précédemment, nommée ```orderdetailsstruct``` : ```collect(orderdetailsstruct)```.

16. La transformation d’agrégation génère uniquement des colonnes qui font partie de formules d’agrégation ou de regroupement. Par conséquent, nous devons également inclure les colonnes de l’en-tête sales. Pour ce faire, ajoutez un modèle de colonne dans la même transformation d’agrégation. Ce modèle inclut toutes les autres colonnes de la sortie :

```instr(name,'OrderQty')==0&&instr(name,'UnitPrice')==0&&instr(name,'SalesOrderID')==0```

17. Utilisez la syntaxe « this » dans les autres propriétés afin de conserver les mêmes noms de colonne, et utilisez la fonction ```first()``` en tant qu’agrégat :

![Agrégat](media/data-flow/cosmosb6.png)

18. Nous sommes prêts à terminer le flux de migration en ajoutant une transformation de récepteur. Cliquez sur « new » (nouveau) en regard de dataset, puis ajoutez un jeu de données Cosmos DB qui pointe vers votre base de données Cosmos DB. Pour la collection, nous allons l’appeler « orders » (commandes) et elle n’aura ni schéma, ni document, car elle sera créée à la volée.

19. Dans Paramètres du récepteur, définissez la Clé de partition sur ```\SalesOrderID``` et l’action de collection sur « recréer ». Assurez-vous que l’onglet Mappage ressemble à ceci :

![Capture d’écran montrant l’onglet Mappage.](media/data-flow/cosmosb7.png)

20. Cliquez sur l’aperçu des données pour vous assurer que vous voyez ces 32 lignes prêtes pour insertion en tant que nouveaux documents dans votre nouveau conteneur :

![Capture d’écran montrant l’onglet Aperçu des données.](media/data-flow/cosmosb8.png)

Si tout semble correct, vous êtes prêt à créer un pipeline, à y ajouter cette activité de flux de données, et à l’exécuter. Vous pouvez l’exécuter à partir d’un débogage ou d’une exécution déclenchée. Après quelques minutes, vous devriez avoir un nouveau conteneur dénormalisé de commandes nommé « orders » dans votre base de données Cosmos DB.

## <a name="next-steps"></a>Étapes suivantes

* Créez le reste de votre logique de flux de données à l’aide de [transformations](concepts-data-flow-overview.md) de flux de données de mappage.
* [Téléchargez le modèle de pipeline terminé](https://github.com/kromerm/adfdataflowdocs/blob/master/sampledata/SQL%20Orders%20to%20CosmosDB.zip) pour ce didacticiel et importez-le dans votre fabrique.
