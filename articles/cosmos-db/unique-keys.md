---
title: Utiliser des clés uniques dans Azure Cosmos DB
description: Découvrez comment définir et utiliser des clés uniques pour une base de données Azure Cosmos. Cet article explique également comment les clés uniques ajoutent une couche d’intégrité des données.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/23/2020
ms.reviewer: sngun
ms.openlocfilehash: 165fb2937db5edfa4f51f62033afaf87cfff83ef
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353100"
---
# <a name="unique-key-constraints-in-azure-cosmos-db"></a>Contraintes des clés uniques dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les clés uniques renforcent l’intégrité des données dans un conteneur Azure Cosmos. Les stratégies de clé unique s’établissent lors de la création d'un conteneur Azure Cosmos. Les clés uniques permettent de certifier qu’une ou plusieurs valeurs d’une partition logique sont uniques. Vous pouvez également garantir l’unicité par [clé de partition](partitioning-overview.md).

Une fois qu’un conteneur a été généré avec une stratégie de clé unique, la création ou la mise à jour d’un élément engendrant un doublon au sein d’une partition logique sont impossibles, conformément à la contrainte de clé unique. La combinaison clé de partition-clé unique garantit l'unicité d'un élément à l’échelle d'un conteneur.

Prenons l'exemple d'un conteneur Azure Cosmos dont la contrainte de clé unique porte sur l’adresse e-mail et la clé de partition correspond à `CompanyID`. Dès lors que l'adresse e-mail de l'utilisateur est configurée avec une clé unique, chaque élément possède une adresse e-mail unique dans un `CompanyID` donné. Il est impossible de créer deux éléments avec des adresses e-mail en double et avec la même valeur de clé de partition. Dans l’API SQL (Core) d’Azure Cosmos DB, les éléments sont stockés sous forme de valeurs JSON. Ces valeurs JSON respectent la casse. Lorsque vous choisissez une propriété comme clé unique, vous pouvez insérer des valeurs respectant la casse pour cette propriété. Par exemple, si vous avez une clé unique définie sur la propriété du nom, « Gaby » est différent de « gaby » et vous pouvez insérer les deux dans le conteneur.

Pour créer des éléments comportant la même adresse e-mail, mais un prénom, un nom et une adresse e-mail différents, ajoutez d’autres chemins d’accès à la stratégie de clé unique. Au lieu de créer une clé unique basée uniquement sur l’adresse e-mail, vous pouvez en créer une qui combine à la fois le prénom, le nom et l’adresse e-mail. Cette clé est appelée clé unique composite. Dans ce cas, chaque combinaison unique des trois valeurs est autorisée dans un `CompanyID` donné. 

Par exemple, le conteneur peut inclure des éléments présentant les valeurs suivantes, chacun respectant la contrainte de clé unique.

|CompanyID|Prénom|Nom|Adresse de messagerie|
|---|---|---|---|
|Contoso|Gaby|Duperre|gaby@contoso.com |
|Contoso|Gaby|Duperre|gaby@fabrikam.com|
|Fabrikam|Gaby|Duperre|gaby@fabrikam.com|
|Fabrikam|Ivan|Duperre|gaby@fabrikam.com|
|Fabrikam|   |Duperre|gaby@fabraikam.com|
|Fabrikam|   |   |gaby@fabraikam.com|

Si vous tentez d’insérer un autre élément présentant une des combinaisons du tableau précédent, vous recevez une erreur indiquant que la contrainte de clé unique n’a pas été remplie. Vous recevez un `Resource with specified ID or name already exists` ou `Resource with specified ID, name, or unique index already exists` comme message de retour. 

## <a name="define-a-unique-key"></a>Définir une clé unique

Les clés uniques ne peuvent être définies qu'au moment de la création d'un conteneur Azure Cosmos. Une clé unique est associée à une partition logique. Dans l'exemple précédent, si vous partitionnez le conteneur en fonction du code postal, vous obtenez des doublons dans chaque partition logique. Lorsque vous créez des clés uniques, tenez compte des propriétés suivantes :

* Il n’est pas possible de mettre à jour un conteneur existant pour qu'il utilise une autre clé unique. Autrement dit, une fois qu'un conteneur a été créé avec une stratégie de clé unique, celle-ci n'est pas modifiable.

* Pour définir la clé unique d’un conteneur existant, créez un nouveau conteneur avec la contrainte de clé unique. Utilisez un outil de migration des données adapté pour déplacer les données du conteneur existant vers le nouveau. Avec les conteneurs SQL, utilisez [l'Outil de migration de données ](import-data.md) pour déplacer des données. Sur les conteneurs MongoDB, utilisez [mongoimport.exe ou mongorestore.exe](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json) pour déplacer des données.

* Une stratégie de clé unique peut comporter au maximum 16 valeurs de chemin d’accès, Par exemple, les valeurs peuvent être `/firstName`, `/lastName`, et `/address/zipCode`. Chaque stratégie de clé unique peut contenir un maximum de 10 combinaisons ou contraintes de clé unique. Les chemins d’accès combinés de chaque contrainte d’index unique ne doivent pas dépasser 60 octets. Dans l’exemple précédent, la combinaison du prénom, du nom et de l’adresse e-mail représente une seule contrainte, qui utilise 3 des 16 chemins possibles.

* Lorsqu’un conteneur dispose d’une stratégie de clé unique, le coût en [unités de requête (RU)](request-units.md) de la création, de la mise à jour et de la suppression d’un élément est légèrement supérieur.

* Les clés uniques partiellement allouées ne sont pas prises en charge. S’il manque les valeurs de certains chemins d’accès uniques, elles sont traitées comme des valeurs Null, qui font partie intégrante de la contrainte d’unicité. C’est pourquoi il ne peut y avoir qu'un seul élément comportant une valeur Null pour satisfaire à cette contrainte.

* Les noms de clés uniques sont sensibles à la casse. Prenons l’exemple d’un conteneur doté d’une contrainte de clé unique définie sur `/address/zipcode`. Si vos données comportent un champ intitulé `ZipCode`, Azure Cosmos DB insère « null » comme clé unique, car `zipcode` n’est pas identique à `ZipCode`. En raison de ce respect de la casse, aucun autre enregistrement contenant ZipCode ne peut être inséré ; en effet, un doublon sur la valeur « null » violerait la contrainte de clé unique.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [partitions logiques](partitioning-overview.md)
* Explorer [comment définir des clés uniques](how-to-define-unique-keys.md) lors de la création d’un conteneur