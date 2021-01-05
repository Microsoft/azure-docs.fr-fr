---
title: Utilisation de données JSON
description: Azure SQL Database et Azure SQL Managed Instance vous permettent d’analyser, d’interroger et de mettre en forme des données dans une notation JavaScript Object Notation (JSON).
services: sql-database
ms.service: sql-db-mi
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: ''
ms.date: 04/19/2020
ms.openlocfilehash: 4258c60335a13a6c0f27588aac1c6cdb185fcf6a
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782798"
---
# <a name="getting-started-with-json-features-in-azure-sql-database-and-azure-sql-managed-instance"></a>Prise en main des fonctionnalités JSON dans Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Azure SQL Database et Azure SQL Managed Instance vous permettent d’analyser et d’interroger des données représentées au format JavaScript Object Notation [(JSON)](https://www.json.org/) et d’exporter vos données relationnelles sous forme de texte JSON. Les scénarios JSON suivants sont disponibles :

- [Conversion de données relationnelles au format JSON](#formatting-relational-data-in-json-format) à l’aide de la clause `FOR JSON`.
- [Utilisation de données JSON](#working-with-json-data)
- [Interrogation de données JSON](#querying-json-data) à l’aide de fonctions scalaires JSON.
- [Conversion de JSON au format tabulaire](#transforming-json-into-tabular-format) à l’aide de la fonction `OPENJSON`.

## <a name="formatting-relational-data-in-json-format"></a>Conversion de données relationnelles au format JSON

Si vous disposez d’un service web qui prélève des données de la couche de base de données et fournit une réponse au format JSON, ou d’infrastructures ou bibliothèques JavaScript côté client qui acceptent des données au format JSON, vous pouvez convertir le contenu de votre base de données au format JSON directement dans une requête SQL. Vous n’avez plus besoin d’écrire de code d’application pour convertir les résultats d’Azure SQL Database ou d’Azure SQL Managed Instance au format JSON, ni d’inclure de bibliothèque de sérialisation JSON pour convertir des résultats de requête tabulaire, puis sérialiser des objets au format JSON. Au lieu de cela, vous pouvez utiliser la clause FOR JSON pour convertir les résultats de requête SQL au format JSON et les utiliser directement dans votre application.

Dans l’exemple suivant, les lignes de la table Sales.Customer sont converties au format JSON à l’aide de la clause FOR JSON :

```sql
select CustomerName, PhoneNumber, FaxNumber
from Sales.Customers
FOR JSON PATH
```

La clause FOR JSON PATH convertit les résultats de la requête au format de texte JSON. Les noms des colonnes sont utilisés en tant que clés, tandis que les valeurs des cellules sont générées en tant que valeurs JSON :

```json
[
{"CustomerName":"Eric Torres","PhoneNumber":"(307) 555-0100","FaxNumber":"(307) 555-0101"},
{"CustomerName":"Cosmina Vlad","PhoneNumber":"(505) 555-0100","FaxNumber":"(505) 555-0101"},
{"CustomerName":"Bala Dixit","PhoneNumber":"(209) 555-0100","FaxNumber":"(209) 555-0101"}
]
```

Le jeu de résultats est mis en forme de tableau JSON, où chaque ligne est formatée en tant qu’objet JSON séparé.

PATH indique que vous pouvez personnaliser le format de sortie de votre résultat JSON à l’aide d’une notation par points dans des alias de colonne. La requête suivante modifie le nom de la clé « CustomerName » dans le format JSON de sortie, et place les numéros de téléphone et de télécopie dans le sous-objet « Contact » :

```sql
select CustomerName as Name, PhoneNumber as [Contact.Phone], FaxNumber as [Contact.Fax]
from Sales.Customers
where CustomerID = 931
FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
```

Le résultat de cette requête ressemble à ceci :

```json
{
    "Name":"Nada Jovanovic",
    "Contact":{
           "Phone":"(215) 555-0100",
           "Fax":"(215) 555-0101"
    }
}
```

Dans cet exemple, nous renvoyons un objet JSON au lieu d’un tableau, en spécifiant l’option [WITHOUT_ARRAY_WRAPPER](/sql/relational-databases/json/remove-square-brackets-from-json-without-array-wrapper-option). Vous pouvez utiliser cette option si vous savez que vous retournez un objet unique en tant que résultat de requête.

La valeur principale de la clause FOR JSON est qu’elle vous permet de renvoyer des données hiérarchiques complexes à partir de votre base de données sous la forme de tableaux ou d’objets JSON imbriqués. L’exemple suivant montre comment inclure les lignes de la table `Orders` appartenant à `Customer` sous forme d’un tableau imbriqué de `Orders` :

```sql
select CustomerName as Name, PhoneNumber as Phone, FaxNumber as Fax,
        Orders.OrderID, Orders.OrderDate, Orders.ExpectedDeliveryDate
from Sales.Customers Customer
    join Sales.Orders Orders
        on Customer.CustomerID = Orders.CustomerID
where Customer.CustomerID = 931
FOR JSON AUTO, WITHOUT_ARRAY_WRAPPER
```

Au lieu d’envoyer des requêtes séparées pour obtenir des données du client, puis d’extraire une liste de commandes associées, vous pouvez obtenir toutes les données nécessaires à l’aide d’une requête unique, comme illustré dans l’exemple de sortie suivant :

```json
{
  "Name":"Nada Jovanovic",
  "Phone":"(215) 555-0100",
  "Fax":"(215) 555-0101",
  "Orders":[
    {"OrderID":382,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":395,"OrderDate":"2013-01-07","ExpectedDeliveryDate":"2013-01-08"},
    {"OrderID":1657,"OrderDate":"2013-01-31","ExpectedDeliveryDate":"2013-02-01"}
  ]
}
```

## <a name="working-with-json-data"></a>Utilisation de données JSON

Si vous n’avez pas de données strictement structurées, si vous avez des sous-objets, tableaux ou données hiérarchiques complexes, ou si vos structures de données évoluent au fil du temps, le format JSON peut vous aider à représenter toute structure de données complexe.

JSON est un format texte utilisable comme tout autre type de chaîne dans Azure SQL Database et Azure SQL Managed Instance. Vous pouvez envoyer ou stocker des données JSON en tant que type NVARCHAR standard :

```sql
CREATE TABLE Products (
  Id int identity primary key,
  Title nvarchar(200),
  Data nvarchar(max)
)
go
CREATE PROCEDURE InsertProduct(@title nvarchar(200), @json nvarchar(max))
AS BEGIN
    insert into Products(Title, Data)
    values(@title, @json)
END
```

Les données JSON utilisées dans cet exemple sont représentées à l’aide du type NVARCHAR(MAX). Des données JSON peuvent être insérées dans cette table ou fournies en tant qu’argument de la procédure stockée à l’aide de la syntaxe Transact-SQL standard, comme indiqué dans l’exemple suivant :

```sql
EXEC InsertProduct 'Toy car', '{"Price":50,"Color":"White","tags":["toy","children","games"]}'
```

Tout langage ou bibliothèque côté client qui fonctionne avec des données de chaîne dans Azure SQL Database et Azure SQL Managed Instance fonctionne également avec des données JSON. Des données JSON peuvent être stockées dans toute table prenant en charge le type NVARCHAR, par exemple, une table mémoire optimisée ou une table avec version système. JSON n’introduit aucune contrainte, que ce soit dans le code côté client ou dans la couche de base de données.

## <a name="querying-json-data"></a>Interrogation de données JSON

Si vous avez des données au format JSON stockées dans des tables SQL Azure, les fonctions JSON vous permettent d’utiliser ces données dans n’importe quelle requête SQL.

Les fonctions JSON disponibles dans Azure SQL Database et Azure SQL Managed Instance vous permettent de traiter des données au format JSON comme tout autre type de données SQL. Vous pouvez facilement extraire des valeurs du texte JSON et utiliser des données JSON dans toute requête :

```sql
select Id, Title, JSON_VALUE(Data, '$.Color'), JSON_QUERY(Data, '$.tags')
from Products
where JSON_VALUE(Data, '$.Color') = 'White'

update Products
set Data = JSON_MODIFY(Data, '$.Price', 60)
where Id = 1
```

La fonction JSON_VALUE extrait une valeur du texte JSON stocké dans la colonne Données. Cette fonction utilise un chemin d’accès de type JavaScript pour faire référence à une valeur en texte JSON à extraire. La valeur extraite peut être utilisée dans toute partie de la requête SQL.

La fonction JSON_QUERY est similaire à la fonction JSON_VALUE. Contrairement à la fonction JSON_VALUE, cette fonction extrait un sous-objet complexe, tel qu’un tableau ou un objet placés dans le texte JSON.

La fonction JSON_MODIFY permet de spécifier le chemin d’accès de la valeur dans le texte JSON à mettre à jour, ainsi qu’une nouvelle valeur pour remplacer l’ancienne. Vous pouvez ainsi facilement mettre à jour le texte JSON sans réanalyser la structure entière.

Étant donné que JSON est stocké dans un texte standard, il n’est nullement garanti que le format des valeurs stockées dans les colonnes de texte est correct. Vous pouvez vérifier que le texte stocké dans une colonne JSON est correctement mis en forme en utilisant des contraintes de vérification Azure SQL Database standard et la fonction ISJSON :

```sql
ALTER TABLE Products
    ADD CONSTRAINT [Data should be formatted as JSON]
        CHECK (ISJSON(Data) > 0)
```

Si le texte d’entrée est du JSON correctement mis en forme, la fonction ISJSON renvoie la valeur 1. À chaque insertion ou mise à jour de la colonne JSON, cette contrainte vérifie que nouvelle valeur de texte n’est pas dans du JSON mal formé.

## <a name="transforming-json-into-tabular-format"></a>Conversion de JSON en format tabulaire

Azure SQL Database et Azure SQL Managed Instance vous permettent également de convertir des collections JSON en format tabulaire, ainsi que de charger ou d’interroger des données JSON.

OPENJSON est une fonction table-valeur qui analyse le texte JSON, trouve un tableau d’objets JSON, opère une itération dans les éléments du tableau, et retourne une ligne dans le résultat de sortie pour chaque élément.

![Format tabulaire JSON](./media/json-features/image_2.png)

Dans l’exemple ci-dessus, nous pouvons spécifier où localiser le tableau JSON à ouvrir (dans le chemin d’accès $.Orders), les colonnes à retourner comme résultat, et où rechercher les valeurs JSON à renvoyer en tant que cellules.

Nous pouvons transformer un tableau JSON dans la variable @orders en un ensemble de lignes, analyser ce résultat ou insérer des lignes dans une table standard :

```sql
CREATE PROCEDURE InsertOrders(@orders nvarchar(max))
AS BEGIN

    insert into Orders(Number, Date, Customer, Quantity)
    select Number, Date, Customer, Quantity
    FROM OPENJSON (@orders)
     WITH (
            Number varchar(200),
            Date datetime,
            Customer varchar(200),
            Quantity int
     )
END
```

La collection de commandes mise en forme de tableau JSON et fournie en tant que paramètre à la procédure stockée peut être analysée et insérée dans la table Orders.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment intégrer JSON dans votre application, consultez les ressources suivantes :

Pour en savoir plus sur les différents scénarios d’intégration de JSON dans votre application, regardez les démonstrations de cette [vidéo Channel 9](https://channel9.msdn.com/Events/DataDriven/SQLServer2016/JSON-as-a-bridge-betwen-NoSQL-and-relational-worlds) ou recherchez un scénario correspondant à votre cas d’utilisation dans les [billets de blog de JSON](/archive/blogs/sqlserverstorageengine/json-in-sql-server-use-cases).