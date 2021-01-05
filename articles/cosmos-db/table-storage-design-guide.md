---
title: Concevoir des tables Azure Cosmos DB scalables et performantes
description: 'Guide de conception Stockage Table Azure : tables scalables et performantes dans Azure Cosmos DB et Stockage Table Azure'
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 06/19/2020
author: sakash279
ms.author: akshanka
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 709b83ad3e71a932202cebb9c9cb6187feae4ed7
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080003"
---
# <a name="azure-table-storage-table-design-guide-scalable-and-performant-tables"></a>Guide de conception de table de stockage Table Azure : tables scalables et performantes
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

[!INCLUDE [storage-table-cosmos-db-tip-include](../../includes/storage-table-cosmos-db-tip-include.md)]

Pour concevoir des tables scalables et performantes, vous devez prendre en compte divers facteurs, dont le coût. Si vous avez déjà conçu des schémas pour des bases de données relationnelles, ces considérations vous seront familières. Toutefois, bien qu’il existe des similarités entre le stockage Table Azure et les modèles relationnels, il existe également de nombreuses différences importantes. Ces différences conduisent généralement à des conceptions différentes qui peuvent sembler absurdes ou incorrectes à une personne ayant une bonne connaissance des bases de données relationnelles, mais qui sont logiques pour une personne menant une conception pour un magasin de paires clé/valeur NoSQL comme celui du stockage Table.

Le stockage Table est conçu pour prendre en charge des applications à l’échelle du cloud qui peuvent contenir des milliards d’entités (« lignes » dans la terminologie de base de données relationnelle) de données, ou pour des jeux de données devant prendre en charge des volumes de transactions très élevés. Vous devez donc concevoir différemment la façon dont vous stockez vos données et comprendre comment fonctionne le stockage Table. Un magasin de données NoSQL bien conçu améliore la scalabilité de votre solution (pour un coût inférieur) par rapport à une solution qui utilise une base de données relationnelle. Ce guide fournit une aide relative à ces sujets.  

## <a name="about-azure-table-storage"></a>À propos de Stockage Table Azure
Cette section présente certaines des principales fonctionnalités du stockage Table qui sont particulièrement adaptées aux conceptions orientées vers l’amélioration des performances et de la scalabilité. Si vous ne connaissez pas Stockage Azure et le stockage Table, consultez [Présentation du Stockage Microsoft Azure](../storage/common/storage-introduction.md) et [Bien démarrer avec le stockage Table Azure à l’aide de .NET](./tutorial-develop-table-dotnet.md) avant de lire le reste de cet article. Bien que ce guide porte sur le stockage Table, il aborde également le stockage File d’attente Azure et le stockage Blob Azure, en expliquant comment les utiliser avec le stockage Table dans une solution.  

Le stockage Table utilise un format tabulaire pour stocker les données. Selon la terminologie standard, chaque ligne de la table représente une entité et les colonnes stockent les différentes propriétés de cette entité. Chaque entité a une paire de clés qui permet de l’identifier de manière unique et une colonne d’horodatage que le stockage Table utilise pour suivre les dernières mises à jour de l’entité. Le champ d’horodatage est ajouté automatiquement, et vous ne pouvez pas le remplacer manuellement par une valeur arbitraire. Le stockage Table utilise le dernier horodatage modifié (ou LMT, pour Last Modified Timestamp) afin de gérer l’accès concurrentiel optimiste.  

> [!NOTE]
> Les opérations d’API REST du stockage Table retournent également une valeur `ETag` dérivée du dernier horodatage modifié (LMT). Dans ce document, nous utilisons indifféremment les termes ETag et LMT, car ils font référence aux mêmes données sous-jacentes.  
> 
> 

L'exemple suivant présente la conception d'une table simple pour stocker des entités relatives à des employés (Employee) ainsi qu'à leurs services (Department). Plusieurs des exemples présentés ultérieurement dans ce guide sont basés sur cette conception simple.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td>Marketing</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Juin</td>
<td>Cao</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>department</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Marketing</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>Ventes</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


Jusqu’à présent, cette conception ressemble à une table dans une base de données relationnelle. Les principales différences sont les colonnes obligatoires et la possibilité de stocker plusieurs types d’entité dans la même table. En outre, chacune des propriétés définies par l’utilisateur, telles que **FirstName** ou **Age** , est caractérisée par un type de données, par exemple un nombre entier ou une chaîne, tout comme une colonne dans une base de données relationnelle. Toutefois, contrairement à une base de données relationnelle, la nature sans schéma du stockage Table signifie qu’une propriété n’a pas nécessairement besoin d’avoir les mêmes types de données pour chaque entité. Pour stocker des types de données complexes dans une seule propriété, vous devez utiliser un format sérialisé comme JSON ou XML. Pour plus d’informations, consultez [Présentation du modèle de données du stockage Table](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model).

Le choix de la valeur de `PartitionKey` et `RowKey` est fondamental pour une bonne conception de table. Toutes les entités stockées dans une table doivent avoir une combinaison unique de `PartitionKey` et `RowKey`. Comme avec les clés dans une table de base de données relationnelle, les valeurs de `PartitionKey` et `RowKey` sont indexées pour créer un index cluster qui permet d’effectuer des recherches rapides. Toutefois, le stockage Table ne crée pas d’index secondaires. Il s’agit donc des deux seules propriétés indexées (certains des modèles décrits plus loin montrent comment contourner cette limitation apparente).  

Une table est constituée d’une ou plusieurs partitions, et la plupart des décisions de conception que vous prenez consistent à choisir une valeur de `PartitionKey` et `RowKey` convenable pour optimiser votre solution. Une solution peut être constituée d’une seule table contenant toutes vos entités organisées en partitions, mais généralement, une solution a plusieurs tables. Les tables vous aident à organiser logiquement vos entités et à gérer l’accès aux données à l’aide de listes de contrôle d’accès. Vous pouvez supprimer une table entière en une seule opération de stockage.  

### <a name="table-partitions"></a>Partitions de table
Le nom du compte, le nom de la table et la valeur de `PartitionKey` identifient la partition dans le service de stockage où le stockage Table stocke l’entité. Tout en appartenant au schéma d’adressage des entités, les partitions définissent une étendue pour les transactions (voir la section [Transactions de groupe d’entités](#entity-group-transactions) plus loin dans cet article) et constituent la base de la méthode de mise à l’échelle du stockage Table. Pour plus d’informations sur les partitions, consultez [Check-list des performances et de la scalabilité pour le stockage Table](../storage/tables/storage-performance-checklist.md).  

Dans le stockage Table, un nœud individuel traite une ou plusieurs partitions complètes, et le service se met à l’échelle en équilibrant la charge des partitions de manière dynamique parmi les nœuds. Si un nœud est en cours de chargement, le stockage Table peut fractionner la plage de partitions servie par ce nœud sur des nœuds différents. Quand le trafic diminue, le stockage Table peut refusionner les plages de partitions des nœuds calmes sur un nœud unique.  

Pour plus d’informations sur les détails internes du stockage Table, et notamment la façon dont il gère les partitions, consultez [Stockage Microsoft Azure : service de stockage cloud hautement disponible à cohérence forte](/archive/blogs/windowsazurestorage/sosp-paper-windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency).  

### <a name="entity-group-transactions"></a>Transactions de groupe d’entités
Dans le stockage Table, les transactions de groupe d’entités (EGT) constituent l’unique mécanisme intégré pour effectuer des mises à jour atomiques entre plusieurs entités. Les EGT sont également appelées *transactions par lots*. Les transactions EGT peuvent uniquement utiliser des entités stockées dans la même partition (partageant la même clé de partition dans une table donnée). Par conséquent, quand vous avez besoin d’un comportement transactionnel atomique entre plusieurs entités, vérifiez que ces entités sont dans la même partition. Ceci justifie souvent la conservation de plusieurs types d’entité dans la même table (et partition) plutôt que l’utilisation de plusieurs tables pour différents types d’entité. Une seule EGT peut traiter jusqu'à 100 entités.  Si vous envoyez plusieurs EGT simultanées pour traitement, vérifiez bien que ces EGT n’opèrent pas sur des entités communes aux différentes EGT. Sinon, le traitement risque d’être retardé.

Les EGT représentent également un éventuel compromis à prendre en compte dans votre conception. L’utilisation de plusieurs partitions augmente la scalabilité de votre application, car Azure a plus d’occasions d’équilibrer la charge des requêtes parmi les nœuds. Toutefois, cela peut limiter la capacité de votre application à effectuer des transactions atomiques et à assurer une cohérence forte pour vos données. Par ailleurs, il s’agit d’objectifs de scalabilité spécifiques au niveau d’une partition qui peuvent limiter le débit de transactions attendu pour un nœud unique.

Pour plus d’informations sur les objectifs de scalabilité pour les comptes Stockage Azure, consultez [Objectifs de scalabilité pour les comptes de stockage standard](../storage/common/scalability-targets-standard-account.md). Pour plus d’informations sur les objectifs de scalabilité pour le stockage Table, consultez [Objectifs de scalabilité et de performances pour le stockage Table](../storage/tables/scalability-targets.md). Les sections ultérieures de ce guide présenteront différentes stratégies de conception pour vous aider à gérer les compromis tels que celui-ci et décrivent les meilleures méthodes pour choisir votre clé de partition en fonction des exigences spécifiques de votre application cliente.  

### <a name="capacity-considerations"></a>Considérations relatives à la capacité
Le tableau suivant présente certaines des valeurs de clés à connaître quand vous concevez une solution de stockage Table :  

| Capacité totale d'un compte de stockage Azure | 500 To |
| --- | --- |
| Nombre de tables dans un compte de stockage Azure |Limité uniquement par la capacité du compte de stockage |
| Nombre de partitions dans une table |Limité uniquement par la capacité du compte de stockage |
| Nombre d'entités dans une partition |Limité uniquement par la capacité du compte de stockage |
| Taille d'une entité individuelle |Jusqu’à 1 Mo, avec un maximum de 255 propriétés (y compris `PartitionKey`, `RowKey` et `Timestamp`) |
| Taille de la `PartitionKey` |Chaîne jusqu’à 1 Ko |
| Taille de la `RowKey` |Chaîne jusqu’à 1 Ko |
| Taille d’une transaction ETG |Une transaction peut inclure au plus 100 entités, et la charge utile doit être inférieure à 4 Mo. Une transaction EGT ne peut mettre à jour une entité qu'une seule fois. |

Pour plus d’informations, consultez [Présentation du modèle de données du service de Table](/rest/api/storageservices/Understanding-the-Table-Service-Data-Model).  

### <a name="cost-considerations"></a>Considérations relatives au coût
Le stockage Table est relativement peu coûteux, mais vous devez y inclure les estimations de coût pour l’utilisation des capacités et la quantité de transactions dans le cadre de l’évaluation d’une solution qui utilise le stockage Table. Toutefois, dans de nombreux scénarios, le stockage de données dénormalisées ou dupliquées afin d’améliorer les performances ou la scalabilité de votre solution est une approche appropriée. Pour plus d’informations sur la tarification, consultez la page [Tarification Azure Storage](https://azure.microsoft.com/pricing/details/storage/).  

## <a name="guidelines-for-table-design"></a>Conseils pour la conception de table
Ces listes récapitulent certaines des principales recommandations à prendre en compte lors de la conception de vos tables. Elles sont toutes traitées en détail plus loin dans ce guide. Ces recommandations sont différentes de celles généralement prodiguées pour la conception d’une base de données relationnelle.  

Conception de votre stockage Table pour une *lecture* efficace :

* **Pensez votre conception pour l’interrogation dans des applications à lecture intensive.** Quand vous concevez vos tables, pensez aux requêtes que vous allez exécuter (en particulier celles sensibles à la latence) avant de réfléchir à la méthode de mise à jour de vos entités. Cela permet généralement d’élaborer une solution efficace et performante.  
* **Spécifiez à la fois `PartitionKey` et `RowKey` dans vos requêtes.** Les *requêtes de pointage* telles que celles-ci sont les requêtes de stockage Table les plus efficaces.  
* **Envisagez de stocker des copies dupliquées des entités.** Le stockage Table est bon marché. Vous pourriez donc stocker la même entité plusieurs fois (avec différentes clés) pour rendre les requêtes plus efficaces.  
* **Envisagez de dénormaliser vos données.** Le stockage Tables est bon marché. Nous vous recommandons donc d’envisager la dénormalisation de vos données. Par exemple, stockez des entités de résumé pour que les requêtes d’agrégation de données aient seulement besoin d’accéder à une entité unique.  
* **Utilisez des valeurs de clé composées.** Les seules clés que vous avez sont `PartitionKey` et `RowKey`. Par exemple, utilisez des valeurs de clé composées pour activer les chemins d’accès de clé de substitution pour les entités.  
* **Utilisez la projection de requête.** Vous pouvez réduire la quantité de données que vous transférez sur le réseau en utilisant des requêtes qui sélectionnent uniquement les champs dont vous avez besoin.  

Conception de votre stockage Table pour une *écriture* efficace :  

* **Ne créez pas de partitions à chaud.** Choisissez des clés qui vous permettent de répartir vos requêtes sur plusieurs partitions à tout moment.  
* **Évitez les pics de trafic.** Distribuez le trafic sur une période de temps raisonnable et évitez les pics de trafic.
* **Ne créez pas nécessairement une table distincte pour chaque type d’entité.** Lorsque vous avez besoin d’utiliser des transactions atomiques sur des types d’entité, vous pouvez stocker ces types d’entité multiples dans la même partition de la même table.
* **Prévoyez le débit maximal nécessaire.** Vous devez connaître les objectifs de scalabilité du stockage Table, et vous assurer que votre conception n’entraînera pas de dépassement.  

Plus loin dans ce guide, vous rencontrerez des exemples mettant tous ces principes en pratique.  

## <a name="design-for-querying"></a>Conception pour l'interrogation
Le stockage Table peut lire ou écrire de façon intensive, ou effectuer une combinaison des deux. Cette section traite de la conception pour prendre en charge efficacement les opérations de lecture. En règle générale, une conception qui prend en charge les opérations de lecture de manière efficace le sera également pour des opérations d'écriture. Toutefois, il existe d’autres éléments à prendre en compte lors de la conception pour prendre en charge les opérations d’écriture. Celles-ci sont présentées dans la section suivante, [Conception pour la modification de données](#design-for-data-modification).

Un bon point de départ afin de vous permettre de lire les données efficacement consiste à vous demander quelles requêtes votre application devra exécuter pour extraire les données dont elle aura besoin.  

> [!NOTE]
> Avec le stockage Table, il est important de mettre en place une conception efficace dès le début, car il est difficile et coûteux de la modifier ultérieurement. Par exemple, dans une base de données relationnelle, il est souvent possible de résoudre les problèmes de performances simplement en ajoutant des index à une base de données existante. Cela n’est pas possible avec le stockage Table.  

### <a name="how-your-choice-of-partitionkey-and-rowkey-affects-query-performance"></a>Comment votre choix de `PartitionKey` et `RowKey` affecte les performances des requêtes
Les exemples suivants partent du principe que le stockage Table stocke les entités relatives aux employés (employee) en utilisant la structure suivante (la plupart des exemples omettent la propriété `Timestamp` par souci de clarté) :  

| Nom de la colonne | Type de données |
| --- | --- |
| `PartitionKey` (nom du service) |String |
| `RowKey` (ID d’employé) |String |
| `FirstName` |String |
| `LastName` |String |
| `Age` |Integer |
| `EmailAddress` |String |

Voici quelques recommandations générales pour la conception de requêtes de stockage Table. La syntaxe de filtre utilisée dans les exemples suivants provient de l’API REST de stockage Table. Pour plus d’informations, consultez [Entités de requêtes](/rest/api/storageservices/Query-Entities).  

* Une *requête de pointage* constitue la méthode de recherche la plus efficace. Elle est recommandée pour les recherches sur de gros volumes ou des recherches nécessitant la latence la plus faible. Une telle requête peut utiliser les index pour localiser une entité individuelle efficacement en spécifiant les valeurs de `PartitionKey` et `RowKey`. Par exemple : `$filter=(PartitionKey eq 'Sales') and (RowKey eq '2')`.  
* La deuxième solution consiste à utiliser une *requête de plage de données*. Elle utilise la `PartitionKey` et filtre sur une plage de valeurs `RowKey` pour retourner plusieurs entités. La valeur de `PartitionKey` identifie une partition spécifique, tandis que la valeur de `RowKey` identifie un sous-ensemble des entités de cette partition. Par exemple : `$filter=PartitionKey eq 'Sales' and RowKey ge 'S' and RowKey lt 'T'`.  
* La troisième solution consiste à effectuer une *analyse de partition*. Elle utilise la `PartitionKey` et filtre sur une autre propriété non-clé, et peut retourner plusieurs entités. La valeur de `PartitionKey` identifie une partition spécifique, et les valeurs des propriétés sélectionnent un sous-ensemble d’entités dans cette partition. Par exemple : `$filter=PartitionKey eq 'Sales' and LastName eq 'Smith'`.  
* Une *analyse de table* n’inclut pas la valeur de `PartitionKey`, et s’avère inefficace car elle lance une recherche sur toutes les partitions qui composent la table pour toutes les entités correspondantes. Elle effectue une analyse de table, que votre filtre utilise la valeur de `RowKey` ou non. Par exemple : `$filter=LastName eq 'Jones'`.  
* Les requêtes de stockage Table Azure qui retournent plusieurs entités les trient par ordre de `PartitionKey` et `RowKey`. Pour éviter un nouveau tri des entités dans le client, sélectionnez une valeur de `RowKey` qui définit l’ordre de tri le plus répandu. Les résultats de la requête renvoyés par l’API Table Azure dans Azure Cosmos DB ne sont pas triés par clé de partition ou clé de ligne. Pour obtenir la liste détaillée des différences de fonctionnalités, consultez [Différences entre l'API Table dans Azure Cosmos DB et Stockage Table Azure](table-api-faq.md#table-api-vs-table-storage).

L’utilisation d’un connecteur «  **or**  » pour spécifier un filtre selon les valeurs de `RowKey` déclenche une analyse de partition, et n’est pas traitée en tant que requête de plage de données. Par conséquent, évitez les requêtes qui utilisent des filtres tels que : `$filter=PartitionKey eq 'Sales' and (RowKey eq '121' or RowKey eq '322')`.  

Pour obtenir des exemples de code côté client qui utilisent la bibliothèque de client de stockage pour exécuter des requêtes efficaces, consultez :  

* [Exécuter une requête de pointage à l’aide de la bibliothèque de client de stockage](#run-a-point-query-by-using-the-storage-client-library)
* [Récupérer plusieurs entités à l’aide de LINQ](#retrieve-multiple-entities-by-using-linq)
* [Projection côté serveur](#server-side-projection)  

Pour obtenir des exemples de code côté client pouvant gérer plusieurs types d'entité stockés dans la même table, consultez la page :  

* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)  

### <a name="choose-an-appropriate-partitionkey"></a>Choisir une `PartitionKey` appropriée
Votre choix de `PartitionKey` peut équilibrer le besoin d’utiliser des transactions EGT (pour garantir la cohérence) avec l’exigence de distribution de vos entités sur plusieurs partitions (pour assurer une solution scalable).  

À une extrémité, vous pouvez stocker toutes vos entités dans une partition unique. Mais cela peut limiter la scalabilité de votre solution et empêcher le stockage Table de pouvoir équilibrer la charge des requêtes. À l’autre extrémité, vous pouvez stocker une entité par partition. Cette solution est hautement scalable et permet au stockage Table d’équilibrer la charge des requêtes, mais vous empêche d’utiliser des transactions EGT.  

Une `PartitionKey` idéale vous permet d’utiliser des requêtes efficaces, et a des partitions suffisantes pour garantir la scalabilité de votre solution. En règle générale, vous vous rendrez compte que vos entités ont une propriété appropriée qui les distribue sur des partitions suffisantes.

> [!NOTE]
> Par exemple, dans un système qui stocke des informations sur des utilisateurs ou des employés, `UserID` peut être une bonne `PartitionKey`. Vous pouvez avoir plusieurs entités qui utilisent un `UserID` spécifique comme clé de partition. Chaque entité qui stocke des données relatives à un utilisateur est regroupée dans une partition unique. Ces entités sont accessibles par le biais d’EGT, tout en restant hautement scalables.
> 
> 

Au moment de choisir la valeur de `PartitionKey`, vous devez vous demander comment vous allez insérer, mettre à jour et supprimer des entités. Pour plus d’informations, consultez [Conception pour la modification de données](#design-for-data-modification) plus loin dans cet article.  

### <a name="optimize-queries-for-table-storage"></a>Optimiser les requêtes pour le stockage Table
Le stockage Table indexe automatiquement vos entités en utilisant les valeurs `PartitionKey` et `RowKey` dans un index cluster unique. C’est la raison pour laquelle les requêtes de pointage sont les plus efficaces. Toutefois, il n’existe aucun autre index que celui de l’index en cluster sur la `PartitionKey` et la `RowKey`.

De nombreuses conceptions doivent répondre aux conditions requises pour permettre la recherche d'entités basée sur plusieurs critères. Par exemple, la localisation des entités relatives aux employés (employee) en fonction de leurs adresses e-mail, de leur ID employé ou de leur nom. Les modèles suivants de la section [Modèles de conception de table](#table-design-patterns) permettent de gérer ces types d’exigences. Les modèles décrivent également comment contourner le fait que le stockage Table ne fournit pas d’index secondaires.  

* [Modèle d’index secondaire intra-partition](#intra-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` (dans la même partition). Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  
* [Modèle d’index secondaire entre les partitions](#inter-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` dans des partitions ou des tables distinctes. Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  
* [Modèle d’entités d’index](#index-entities-pattern) : Permet de mettre à jour des entités d'index pour mener des recherches efficaces renvoyant des listes d'entités.  

### <a name="sort-data-in-table-storage"></a>Trier des données dans le stockage Table

Le stockage Table retourne les résultats de requête triés par ordre croissant, en fonction de `PartitionKey` puis de `RowKey`.

> [!NOTE]
> Les résultats de la requête renvoyés par l’API Table Azure dans Azure Cosmos DB ne sont pas triés par clé de partition ou clé de ligne. Pour obtenir la liste détaillée des différences de fonctionnalités, consultez [Différences entre l'API Table dans Azure Cosmos DB et Stockage Table Azure](table-api-faq.md#table-api-vs-table-storage).

Les clés dans le stockage Table sont des valeurs de chaîne. Pour être sûr que les valeurs numériques sont triées correctement, vous devez les convertir en une longueur fixe et les remplir avec des zéros. Par exemple, si la valeur d’ID d’un employé que vous utilisez comme `RowKey` est une valeur de nombre entier, vous devez convertir l’ID de cet employé, **123** , en **00000123**. 

De nombreuses applications ont des conditions d'utilisation pour l'utilisation des données triées dans différents ordres : par exemple, le tri des employés par nom ou par date d'arrivée. Les modèles suivants de la section [Modèles de conception de table](#table-design-patterns) permettent de comprendre comment alterner les ordres de tri pour vos entités :  

* [Modèle d’index secondaire intra-partition](#intra-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` (dans la même partition). Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  
* [Modèle d’index secondaire entre les partitions](#inter-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` dans des partitions ou des tables distinctes. Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.
* [Modèle de fin de journal](#log-tail-pattern) : récupérer les *n* entités récemment ajoutées à une partition en utilisant une valeur de `RowKey` qui effectue un tri dans l’ordre inverse de la date et de l’heure.  

## <a name="design-for-data-modification"></a>Conception pour la modification de données
Cette section se concentre sur les considérations de conception pour optimiser les insertions, les mises à jour et les suppressions. Dans certains cas, vous devrez évaluer le compromis entre les conceptions qui optimisent les requêtes et celles qui optimisent la modification des données. Cette évaluation est similaire à ce que vous faites dans les conceptions des bases de données relationnelles (bien que les techniques de gestion des compromis de conception soient différentes dans une base de données relationnelle). La section [Modèles de conception de table](#table-design-patterns) décrit plusieurs modèles de conception détaillés pour le stockage Table, et apporte des informations sur certains de ces compromis. En pratique, vous constaterez que les nombreuses conceptions optimisées pour l’interrogation des entités fonctionnent aussi bien pour la modification des entités.  

### <a name="optimize-the-performance-of-insert-update-and-delete-operations"></a>Optimiser les performances des opérations d'insertion, de mise à jour et de suppression
Pour mettre à jour ou supprimer une entité, vous devez être en mesure de l’identifier à l’aide des valeurs de `PartitionKey` et `RowKey`. À cet égard, votre choix de `PartitionKey` et `RowKey` pour la modification des entités doit suivre des critères similaires à ceux de votre choix concernant la prise en charge des requêtes de pointage. Il faut identifier les entités le plus efficacement possible. Il vaut mieux ne pas utiliser d’analyse de table ou de partition inefficace pour localiser une entité afin de découvrir les valeurs de `PartitionKey` et `RowKey` que vous devez mettre à jour ou supprimer.  

Les modèles suivants de la section [Modèles de conception de table](#table-design-patterns) permettent d’optimiser les performances de vos opérations d’insertion, de mise à jour et de suppression :  

* [Modèle de suppression de volume élevé](#high-volume-delete-pattern) : activer la suppression d’un volume élevé d’entités en stockant toutes les entités pour les supprimer simultanément dans leur propre table distincte. Vous supprimez les entités en supprimant la table.  
* [Modèle de série de données](#data-series-pattern) : Stockez des séries de données complètes dans une entité unique pour réduire votre nombre de demandes.  
* [Modèle d’entités larges](#wide-entities-pattern) : Utilisez plusieurs entités physiques pour stocker des entités logiques ayant plus de 252 propriétés.  
* [Modèle d’entités volumineuses](#large-entities-pattern) : Utilisez le stockage d'objets blob pour stocker des valeurs de propriétés volumineuses.  

### <a name="ensure-consistency-in-your-stored-entities"></a>Assurer la cohérence de vos entités stockées
L'autre facteur déterminant pour votre choix de clés en vue de l'optimisation des modifications de données consiste à assurer la cohérence à l'aide de transactions atomiques. Vous pouvez uniquement utiliser une EGT pour mener des opérations sur des entités stockées dans la même partition.  

Les modèles suivants de la section [Modèles de conception de table](#table-design-patterns) permettent de gérer la cohérence :  

* [Modèle d’index secondaire intra-partition](#intra-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` (dans la même partition). Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  
* [Modèle d’index secondaire entre les partitions](#inter-partition-secondary-index-pattern) : stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` dans des partitions ou des tables distinctes. Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  
* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern) : Permet de conserver un comportement cohérent entre les limites de partition ou les limites du système de stockage à l'aide des files d'attente Azure.
* [Modèle d’entités d’index](#index-entities-pattern) : Permet de mettre à jour des entités d'index pour mener des recherches efficaces renvoyant des listes d'entités.  
* [Modèle de dénormalisation](#denormalization-pattern) : combiner des données connexes dans une entité unique pour pouvoir récupérer toutes les données dont vous avez besoin avec une seule requête de pointage.  
* [Modèle de série de données](#data-series-pattern) : stocker des séries de données complètes dans une entité unique pour réduire le nombre de requêtes effectuées.  

Pour plus d’informations, consultez [Transactions de groupe d’entités](#entity-group-transactions) plus loin dans cet article.  

### <a name="ensure-your-design-for-efficient-modifications-facilitates-efficient-queries"></a>Faire en sorte que votre conception pour des modifications efficaces facilite les requêtes efficaces
Dans de nombreux cas, une conception visant à obtenir une interrogation efficace entraîne des modifications efficaces, mais vous devez toujours évaluer si cela est le cas pour votre scénario. Certains des modèles de la section [Modèles de conception de table](#table-design-patterns) évaluent clairement les compromis entre l’interrogation et la modification des entités. Vous devez toujours prendre en compte le nombre de chaque type d’opération.  

Les modèles suivants de la section [Modèles de conception de table](#table-design-patterns) répondent aux compromis entre la conception en vue d’une interrogation efficace et la conception en vue d’une modification de données efficace :  

* [Modèle de clé composée](#compound-key-pattern) : utiliser des valeurs de `RowKey` composées pour permettre à un client de rechercher des données connexes en utilisant une seule requête de pointage.  
* [Modèle de fin de journal](#log-tail-pattern) : récupérer les *n* entités récemment ajoutées à une partition en utilisant une valeur de `RowKey` qui effectue un tri dans l’ordre inverse de la date et de l’heure.  

## <a name="encrypt-table-data"></a>Chiffrer des données de table
La bibliothèque de client .NET Stockage Azure prend en charge le chiffrement des propriétés de l’entité de chaîne pour les opérations d’insertion et de remplacement. Les chaînes chiffrées sont stockées sur le service en tant que propriétés binaires, et elles sont converties en chaînes après le déchiffrement.    

Pour les tables, outre la stratégie de chiffrement, les utilisateurs doivent spécifier les propriétés à chiffrer. Spécifiez un attribut `EncryptProperty` (pour les entités POCO qui dérivent de `TableEntity`) ou un programme de résolution de chiffrement dans les options de requête. Un programme de résolution de chiffrement est un délégué qui prend une clé de partition, une clé de ligne et un nom de propriété, puis retourne une valeur booléenne indiquant si cette propriété doit être chiffrée. Au cours du chiffrement, la bibliothèque de client utilise ces informations pour décider si une propriété doit être chiffrée lors de l’écriture en ligne. Le délégué fournit également la possibilité de définir la manière dont les propriétés sont chiffrées l’aide d’un programme logique. (Par exemple, si X, alors chiffrer la propriété A ; sinon chiffrer les propriétés A et B.) Il n’est pas nécessaire de fournir ces informations pendant la lecture ou l’interrogation des entités.

La fusion n’est pas prise en charge actuellement. Un sous-ensemble de propriétés ayant pu être chiffré précédemment à l’aide d’une clé différente, la fusion des nouvelles propriétés et la mise à jour des métadonnées entraîneront une perte de données. L’opération de fusion nécessite d’effectuer des appels de service supplémentaires pour lire l’entité pré-existante à partir du service, ou d’utiliser une nouvelle clé par propriété. Pour des raisons de performances, aucune de ces options n’est adaptée.     

Pour plus d’informations sur le chiffrement des données de table, consultez [Chiffrement côté client et coffre de clés Azure pour Microsoft Azure Storage](../storage/common/storage-client-side-encryption.md).  

## <a name="model-relationships"></a>Relations de modèles
La création de modèles de domaine est une étape importante pour concevoir des systèmes complexes. En règle générale, le processus de modélisation permet d’identifier les entités et les relations entre elles, pour mieux comprendre le domaine de l’entreprise et concevoir votre système. Cette section est axée sur la manière dont vous pouvez traduire certains des types de relations courantes présents dans les modèles du domaine pour les conceptions du stockage Table. Le processus de mappage d’un modèle de données logique vers un modèle de données NoSQL physique est différent de celui utilisé pour la conception d’une base de données relationnelle. La conception des bases de données relationnelles suppose généralement un processus de normalisation des données optimisé pour limiter la redondance. Une telle conception suppose également une fonctionnalité d’interrogation déclarative qui fait abstraction de l’implémentation du fonctionnement de la base de données.  

### <a name="one-to-many-relationships"></a>Relations un-à-plusieurs
Les relations un-à-plusieurs entre des objets de domaine d’entreprise se produisent fréquemment : par exemple, un service a de nombreux employés. Il existe plusieurs méthodes pour implémenter des relations un-à-plusieurs dans le stockage Table. Chacune d’elles a des inconvénients et des avantages qui peuvent être pertinents pour un scénario particulier.  

Prenons l’exemple d’une grande entreprise multinationale avec des dizaines de milliers de services et d’entités d’employés. Chaque service a de nombreux employés, et chaque employé est associé à un service spécifique. Une approche consiste à stocker séparément les entités relatives aux services (department) et aux employés (employee), comme ceci :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE01.png" alt-text="Graphique présentant une entité Department et une entité Employee":::

Cet exemple montre une relation un-à-plusieurs implicite entre les types, basée sur la valeur de `PartitionKey`. Chaque service peut avoir de nombreux employés.  

Cet exemple montre également une entité de service (department) et ses entités d'employés (employee) dans la même partition. Vous pouvez choisir d’utiliser des partitions, des tables ou même des comptes de stockage différents pour les différents types d’entité.  

Une autre approche consiste à dénormaliser vos données et à stocker uniquement les entités d’employés avec les données de services dénormalisées, comme indiqué dans l’exemple suivant. Cette approche dénormalisée n’est peut-être pas la meilleure pour ce scénario si vous devez pouvoir modifier les détails d’un responsable de service. Pour ce faire, il vous faudrait mettre à jour chaque employé du service.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE02.png" alt-text="Graphique de l’entité Employé montrant comment dénormaliser vos données et stocker uniquement les entités d’employés avec les données de services dénormalisées.":::

Pour plus d’informations, consultez la section [Modèle de dénormalisation](#denormalization-pattern) plus loin dans ce guide.  

Le tableau suivant récapitule les avantages et les inconvénients de chacune des approches pour le stockage des entités d’employés et de services qui ont une relation un-à-plusieurs. Vous devez également prendre en compte la fréquence à laquelle vous prévoyez d’effectuer diverses opérations. Il peut être acceptable d’avoir une conception qui comprend une opération coûteuse si cette opération ne se produit que rarement.  

<table>
<tr>
<th>Approche</th>
<th>Avantages</th>
<th>Inconvénients</th>
</tr>
<tr>
<td>Séparation des types d'entités, même partition, même table</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service en une seule opération.</li>
<li>Vous pouvez utiliser une EGT pour maintenir la cohérence si vous avez besoin de modifier une entité de service à chaque mise à jour/insertion/suppression d'une entité d'employé. Par exemple, si vous conservez un nombre d’employés de service pour chaque service.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer à la fois une entité d’employé et de service pour certaines activités du client.</li>
<li>Les opérations de stockage se produisent dans la même partition. Si les volumes de transactions sont élevés, cela peut générer une zone sensible.</li>
<li>Vous ne pouvez pas déplacer un employé vers un nouveau service à l’aide d’une EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Types d’entités distincts, différentes partitions ou différentes tables ou différents comptes de stockage</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service ou une entité d'employé avec une seule opération.</li>
<li>Si les volumes de transactions sont élevés, ceci peut permettre de répartir la charge sur plusieurs partitions.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer à la fois une entité d’employé et de service pour certaines activités du client.</li>
<li>Vous ne pouvez pas utiliser des EGT pour maintenir la cohérence lors d’une mise à jour/insertion/suppression d’employé et d’une mise à jour de service. Par exemple, la mise à jour d'un nombre d'employés dans une entité de service.</li>
<li>Vous ne pouvez pas déplacer un employé vers un nouveau service à l’aide d’une EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Dénormaliser en type d'entité unique</td>
<td>
<ul>
<li>Vous pouvez récupérer toutes les informations dont vous avez besoin en utilisant une seule demande.</li>
</ul>
</td>
<td>
<ul>
<li>Il peut être coûteux de maintenir la cohérence si vous devez mettre à jour les informations d’un service (cela vous obligerait à mettre à jour tous les employés d’un service).</li>
</ul>
</td>
</tr>
</table>

Votre choix parmi ces options, ainsi que la détermination des avantages et des inconvénients les plus significatifs, dépendent de votre scénario d’application. Par exemple, à quelle fréquence modifiez-vous les entités de service ? Toutes les requêtes de vos employés ont-elles besoin des informations supplémentaires sur les services ? Êtes-vous proche de vos limites de scalabilité sur vos partitions ou votre compte de stockage ?  

### <a name="one-to-one-relationships"></a>Relations un à un
Les modèles de domaines peuvent inclure des relations un-à-un entre les entités. Si vous devez implémenter une relation un-à-un dans le stockage Table, vous devez également choisir comment lier les deux entités associées quand vous avez besoin de récupérer les deux. Ce lien peut être implicite (basé sur une convention dans les valeurs de clé) ou explicite (si l’on stocke un lien sous forme de valeurs de `PartitionKey` et `RowKey` dans chaque entité et son entité associée). Pour savoir quand stocker les entités associées dans la même partition, consultez la section [Relations un à plusieurs](#one-to-many-relationships).  

Certaines considérations sur l’implémentation peuvent vous conduire à implémenter des relations un-à-un dans le stockage Table :  

* Gestion des entités volumineuses (pour plus d’informations, consultez [Modèle d’entités volumineuses](#large-entities-pattern)).  
* L’implémentation de contrôles d’accès (pour plus d’informations, consultez [Contrôler l’accès avec des signatures d’accès partagé](#control-access-with-shared-access-signatures)).  

### <a name="join-in-the-client"></a>Joindre le client
Bien qu’il existe des façons de modéliser des relations dans le stockage Table, n’oubliez pas que les deux principaux motifs pour utiliser le stockage Table sont la scalabilité et les performances. Si vous devez modéliser plusieurs relations pouvant compromettre les performances et la scalabilité de votre solution, demandez-vous s’il est nécessaire de générer toutes les relations de données dans votre conception de table. Vous pouvez peut-être simplifier la conception, et améliorer la scalabilité et les performances de votre solution, si vous laissez votre application cliente effectuer les jointures nécessaires.  

Par exemple, si vous avez des petites tables qui contiennent des données qui ne changent pas souvent, vous pouvez récupérer ces données une fois et les mettre en cache sur le client. Cela peut éviter des allers-retours répétés pour récupérer les mêmes données. Dans les exemples que nous avons étudiés dans ce guide, il est probable que l’ensemble des services d’une petite organisation soit peu volumineux et rarement modifié. Cela en fait un bon candidat pour les données qu’une application cliente peut télécharger une seule fois et mettre en cache en tant que données de recherche.  

### <a name="inheritance-relationships"></a>Relations d'héritage
Si votre application cliente utilise un ensemble de classes qui font partie d’une relation d’héritage pour représenter des entités métier, vous pouvez facilement conserver ces entités dans le stockage Table. Par exemple, l’ensemble de classes suivant peut être défini dans votre application cliente, `Person` étant une classe abstraite.

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE03.png" alt-text="Diagramme des relations d’héritage":::

Vous pouvez rendre persistantes les instances des deux classes concrètes dans le stockage Table en utilisant une seule table `Person`. Utilisez des entités ressemblant à ce qui suit :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE04.png" alt-text="Graphique montrant l’entité Client et l’entité Employee":::

Pour plus d’informations sur l’utilisation de plusieurs types d’entité dans la même table dans le code client, consultez [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types) plus loin dans ce guide. Vous y trouverez des exemples montrant comment reconnaître le type d'entité dans le code client.  

## <a name="table-design-patterns"></a>Modèles de conception de table
Dans les sections précédentes, vous avez découvert comment optimiser votre conception de table pour la récupération des données d’entité à l’aide de requêtes, et pour l’insertion, la mise à jour et la suppression des données d’entité. Cette section décrit certains modèles appropriés pour une utilisation avec le stockage Table. En outre, vous verrez comment traiter certains problèmes et compromis abordés précédemment dans ce guide. Le diagramme suivant récapitule les relations entre les différents modèles :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE05.png" alt-text="Diagramme des modèles de conception de tables":::

Le plan des modèles met en évidence les relations entre les modèles (bleus) et les anti-modèles (orange) qui sont décrits dans ce guide. Il existe bien sûr bien d'autres modèles qui méritent votre attention. Par exemple, l’un des principaux scénarios pour un stockage Table consiste à utiliser des [modèles d’affichages matérialisés](/previous-versions/msp-n-p/dn589782(v=pandp.10)) à partir du modèle [Répartition de la responsabilité de requête de commande](/previous-versions/msp-n-p/jj554200(v=pandp.10)).  

### <a name="intra-partition-secondary-index-pattern"></a>Modèle d’index secondaire intra-partition
stocker plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` (dans la même partition). Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`. La cohérence des mises à jour entre les copies peut être assurée à l’aide d’EGT.  

#### <a name="context-and-problem"></a>Contexte et problème
Le stockage Table indexe automatiquement les entités en utilisant les valeurs `PartitionKey` et `RowKey`. Ainsi, une application cliente peut récupérer une entité efficacement à l’aide de ces valeurs. Par exemple, à l’aide de la structure de table suivante, une application cliente peut utiliser une requête de pointage pour récupérer une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (valeurs `PartitionKey` et `RowKey`). Un client peut aussi récupérer des entités triées par ID d’employé au sein de chaque service.

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE06.png" alt-text="Graphique de l’entité Employé où une application cliente peut utiliser une requête de pointage pour récupérer une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (valeurs PartitionKey et RowKey).":::

Si vous voulez également trouver une entité d’employé en fonction de la valeur d’une autre propriété, comme l’adresse e-mail, vous devez utiliser une analyse de partition moins efficace pour rechercher une correspondance. En effet, le stockage Table ne fournit pas d’index secondaires. De plus, vous ne pouvez pas demander une liste des employés triés dans un ordre différent de celui de `RowKey`.  

#### <a name="solution"></a>Solution
Pour contourner l’absence d’index secondaires, vous pouvez stocker plusieurs copies de chaque entité, chaque copie utilisant une valeur différente de `RowKey`. Si vous stockez une entité avec les structures suivantes, vous pouvez récupérer efficacement des entités d’employés d’après leur adresse e-mail ou leur ID d’employé. Les valeurs de préfixe pour `RowKey`, `empid_` et `email_` permettent d’interroger un seul employé, ou une plage d’employés, à l’aide d’une plage d’adresses e-mail ou d’ID d’employés.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE07.png" alt-text="Graphique montrant une entité Employee avec différentes valeurs RowKey":::

Les deux critères de filtre suivants (l’un recherchant d’après l’ID d’employé et l’autre d’après l’adresse e-mail) spécifient des requêtes de pointage :  

* $filter=(PartitionKey eq ’Sales’) and (RowKey eq ’empid_000223’)  
* $filter=(PartitionKey eq 'Sales') and (RowKey eq email_jonesj@contoso.com  

Si vous interrogez un ensemble d’entités d’employés, vous pouvez spécifier une plage triée par ID d’employé ou une plage triée par adresse e-mail. Recherchez les entités avec le préfixe approprié dans la `RowKey`.  

* Pour rechercher tous les employés du service des ventes avec un ID d’employé compris entre 000100 et 000199, utilisez : $filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000100') and (RowKey le 'empid_000199')  
* Pour rechercher tous les employés du service des ventes dont l’adresse e-mail commence par la lettre « a », utilisez : $filter=(PartitionKey eq 'Sales') and (RowKey ge 'email_a') and (RowKey lt 'email_b')  
  
La syntaxe de filtre utilisée dans les exemples précédents provient de l’API REST de stockage Table. Pour plus d’informations, consultez [Entités de requêtes](/rest/api/storageservices/Query-Entities).  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* L’utilisation du stockage Table est relativement bon marché. Le coût réel du stockage des données en double ne doit donc pas être une préoccupation majeure. Toutefois, vous devez toujours évaluer le coût de la conception en fonction de vos besoins en stockage anticipés, et ajouter uniquement des entités en double pour prendre en charge les requêtes que votre application cliente exécutera.  
* Les entités d’index secondaires étant stockées dans la même partition que les entités d’origine, vérifiez que vous ne dépassez pas les objectifs de scalabilité pour une partition spécifique.  
* Vous pouvez maintenir la cohérence de vos entités en double en utilisant des EGT pour mettre à jour de façon atomique les deux copies d'une même entité. Cela implique un stockage de toutes les copies d'une entité dans la même partition. Pour plus d’informations, consultez [Utiliser des transactions de groupe d’entités](#entity-group-transactions).  
* La valeur de la `RowKey` doit être unique pour chaque entité. Nous vous conseillons d'utiliser des valeurs de clé composée.  
* Le remplissage des valeurs numériques dans `RowKey` (par exemple l’ID d’employé 000223) permet de corriger le tri et le filtrage en fonction des limites inférieure et supérieure.  
* Vous n’avez pas forcément besoin de dupliquer toutes les propriétés de votre entité. Par exemple, si les requêtes de recherche des entités à l’aide de l’adresse e-mail dans la `RowKey` n’ont jamais besoin de l’âge de l’employé, ces entités peuvent présenter la structure suivante :

  :::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE08.png" alt-text="Graphique de l’entité Employee":::

* En général, il est préférable de stocker les données en double et de vous assurer que vous pouvez récupérer toutes les données dont vous avez besoin avec une seule requête, plutôt que d’utiliser une requête pour rechercher une entité et une autre pour rechercher les données requises.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle dans les situations suivantes :

- Votre application cliente doit récupérer des entités à l’aide de différentes clés.
- Votre client doit récupérer des entités dans différents ordres de tri.
- Vous pouvez identifier chaque entité à l’aide d’une variété de valeurs uniques.

Toutefois, veillez à ne pas dépasser les limites de scalabilité de partition quand vous effectuez des recherches d’entité à l’aide des différentes valeurs de `RowKey`.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle d’index secondaire entre les partitions](#inter-partition-secondary-index-pattern)
* [Modèle de clé composée](#compound-key-pattern)
* [Transactions de groupe d’entités](#entity-group-transactions)
* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)

### <a name="inter-partition-secondary-index-pattern"></a>Modèle d’index secondaire entre les partitions
Stockez plusieurs copies de chaque entité en utilisant différentes valeurs de `RowKey` dans des partitions ou des tables distinctes. Cela permet d’effectuer des recherches rapides et efficaces, ainsi que d’autres ordres de tri à l’aide de différentes valeurs de `RowKey`.  

#### <a name="context-and-problem"></a>Contexte et problème
Le stockage Table indexe automatiquement les entités en utilisant les valeurs `PartitionKey` et `RowKey`. Ainsi, une application cliente peut récupérer une entité efficacement à l’aide de ces valeurs. Par exemple, à l’aide de la structure de table suivante, une application cliente peut utiliser une requête de pointage pour récupérer une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (valeurs `PartitionKey` et `RowKey`). Un client peut aussi récupérer des entités triées par ID d’employé au sein de chaque service.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE09.png" alt-text="Graphique de la structure de l’entité Employé, qui lorsqu’elle est employée, permet à une application cliente d’utiliser une requête de pointage pour récupérer une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (valeurs PartitionKey et RowKey).":::[9]

Si vous voulez également pouvoir trouver une entité d'employé en fonction de la valeur d'une autre propriété, comme l'adresse de messagerie, vous devez utiliser une analyse de partition moins efficace pour rechercher une correspondance. En effet, le stockage Table ne fournit pas d’index secondaires. De plus, vous ne pouvez pas demander une liste des employés triés dans un ordre différent de celui de `RowKey`.  

Vous prévoyez un volume élevé de transactions sur ces entités et vous souhaitez réduire le risque de limitation du débit de votre client par le stockage Table.  

#### <a name="solution"></a>Solution
Pour contourner l’absence d’index secondaires, vous pouvez stocker plusieurs copies de chaque entité, chaque copie utilisant des valeurs différentes de `PartitionKey` et `RowKey`. Si vous stockez une entité avec les structures suivantes, vous pouvez récupérer efficacement des entités d’employés d’après leur adresse e-mail ou leur ID d’employé. Les valeurs de préfixe pour `PartitionKey`, `empid_` et `email_` permettent d’identifier l’index à utiliser pour une requête.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE10.png" alt-text="Graphique montrant l’entité Employee avec index primaire et l’entité Employee avec index secondaire":::

Les deux critères de filtre suivants (l’un recherchant d’après l’ID d’employé et l’autre d’après l’adresse e-mail) spécifient des requêtes de pointage :  

* $filter=(PartitionKey eq ’empid_Sales’) and (RowKey eq ’000223’)
* $filter=(PartitionKey eq ’email_Sales’) and (RowKey eq jonesj@contoso.com  

Si vous interrogez un ensemble d’entités d’employés, vous pouvez spécifier une plage triée par ID d’employé ou une plage triée par adresse e-mail. Recherchez les entités avec le préfixe approprié dans la `RowKey`.  

* Pour rechercher tous les employés du service des ventes avec un ID d’employé compris entre **000100** et **000199** , utilisez : $filter=(PartitionKey eq 'empid_Sales') and (RowKey ge '000100') and (RowKey le '000199')  
* Pour rechercher tous les employés du service des ventes ayant une adresse e-mail qui commence par « a », triés par ordre d’adresse e-mail, utilisez : $filter=(PartitionKey eq 'email_Sales') and (RowKey ge 'a') and (RowKey lt 'b')  

Notez que la syntaxe de filtre utilisée dans les exemples précédents provient de l’API REST de stockage Table. Pour plus d’informations, consultez [Entités de requêtes](/rest/api/storageservices/Query-Entities).  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Vous pouvez conserver la cohérence de vos entités en double en utilisant le [modèle de transactions cohérentes](#eventually-consistent-transactions-pattern) pour gérer les entités d’index primaire et secondaire.  
* L’utilisation du stockage Table est relativement bon marché. Le coût réel du stockage des données en double ne doit donc pas être une préoccupation majeure. Toutefois, évaluez toujours le coût de la conception en fonction de vos besoins en stockage anticipés, et ajoutez uniquement des entités en double pour prendre en charge les requêtes que votre application cliente exécutera.  
* La valeur de la `RowKey` doit être unique pour chaque entité. Nous vous conseillons d'utiliser des valeurs de clé composée.  
* Le remplissage des valeurs numériques dans `RowKey` (par exemple l’ID d’employé 000223) permet de corriger le tri et le filtrage en fonction des limites inférieure et supérieure.  
* Vous n’avez pas forcément besoin de dupliquer toutes les propriétés de votre entité. Par exemple, si les requêtes de recherche des entités à l’aide de l’adresse e-mail dans la `RowKey` n’ont jamais besoin de l’âge de l’employé, ces entités peuvent présenter la structure suivante :
  
  :::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE11.png" alt-text="Graphique montrant l’entité Employee avec index secondaire":::

* En général, il est préférable de stocker les données en double et de vous assurer que vous pouvez récupérer toutes les données dont vous avez besoin en utilisant une seule requête, plutôt que d’utiliser une requête pour rechercher une entité à l’aide de l’index secondaire et une autre pour rechercher les données requises dans l’index primaire.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle dans les situations suivantes :

- Votre application cliente doit récupérer des entités à l’aide de différentes clés.
- Votre client doit récupérer des entités dans différents ordres de tri.
- Vous pouvez identifier chaque entité à l’aide d’une variété de valeurs uniques.

Utilisez ce modèle si vous voulez éviter le dépassement des limites de scalabilité de partition quand vous effectuez des recherches d’entité à l’aide des différentes valeurs de `RowKey`.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern)  
* [Modèle d’index secondaire intra-partition](#intra-partition-secondary-index-pattern)  
* [Modèle de clé composée](#compound-key-pattern)  
* [Transactions de groupe d’entités](#entity-group-transactions)  
* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)  

### <a name="eventually-consistent-transactions-pattern"></a>Modèle de transactions cohérentes
Permet de conserver un comportement cohérent entre les limites de partition ou les limites du système de stockage à l'aide des files d'attente Azure.  

#### <a name="context-and-problem"></a>Contexte et problème
Les EGT activent les transactions atomiques de plusieurs entités qui partagent la même clé de partition. Pour des raisons de scalabilité et de performances, vous pouvez décider de stocker des entités ayant des exigences de cohérence dans des partitions distinctes ou dans un système de stockage distinct. Dans ce scénario, vous ne pouvez pas utiliser les EGT pour maintenir la cohérence. Par exemple, vous pouvez avoir besoin de maintenir la cohérence entre :  

* des entités stockées dans deux partitions différentes de la même table, dans des tables différentes ou dans différents comptes de stockage ;  
* Une entité stockée dans le stockage Table et un objet blob stocké dans le stockage d’objets blob.  
* Une entité stockée dans le stockage Table et un fichier dans un système de fichiers.  
* Une entité stockée dans le stockage Table mais indexée à l’aide de Recherche cognitive Azure.  

#### <a name="solution"></a>Solution
À l'aide des files d'attente Azure, vous pouvez implémenter une solution cohérente entre plusieurs partitions ou systèmes de stockage.

Pour illustrer cette approche, supposez que vous avez besoin d’archiver des entités sur les anciens employés. Ces entités sont rarement interrogées, et doivent être exclues de toutes les activités impliquant des employés actuels. Pour implémenter cette exigence, vous stockez les employés actifs dans la table **Current** et les anciens employés dans la table **Archive**. L’archivage d’un employé nécessite la suppression de son entité de la table **Current** et son ajout à la table **Archive**.

Toutefois, vous ne pouvez pas utiliser une EGT pour effectuer ces deux opérations Pour éviter le risque qu'une défaillance provoque l'apparition d'une entité dans les deux tables ou dans aucune d'elles, l'opération d'archivage doit être cohérente. Le diagramme de séquence suivant décrit les étapes de cette opération.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE12.png" alt-text="Diagramme de solution pour la cohérence finale":::

Un client lance l’opération d’archivage en plaçant un message dans une file d’attente Azure (dans cet exemple, pour archiver l’employé n°456). Un rôle de travail interroge la file d'attente à la recherche de nouveaux messages ; lorsqu'il en trouve un, il le lit et laisse une copie masquée dans la file d'attente. Le rôle de travail extrait ensuite une copie de l’entité à partir de la table **Current** , insère une copie dans la table **Archive** et supprime l’original de la table **Current**. Enfin, si aucune erreur n'est survenue lors des étapes précédentes, le rôle de travail supprime le message masqué de la file d'attente.  

Dans cet exemple, l’étape 4 du diagramme permet d’insérer l’employé dans la table **Archive** . L’employé peut être ajouté à un objet blob dans le stockage Blob ou à un fichier dans un système de fichiers.  

#### <a name="recover-from-failures"></a>Récupérer suite à des échecs
Il est important que les opérations des étapes 4-5 du diagramme soient *idempotentes* au cas où le rôle de travail nécessite un redémarrage de l’opération d’archivage. Si vous utilisez le stockage Table, à l’étape 4 vous devez utiliser une opération « insérer ou remplacer » ; à l’étape 5, vous devez faire appel à une opération « supprimer si existe » dans la bibliothèque de client que vous utilisez. Si vous utilisez un autre système de stockage, vous devez utiliser une opération idempotent appropriée.  

Si le rôle de travail ne termine jamais l’étape 6, après un délai d’attente le message réapparaît dans la file d’attente, prêt pour le rôle de travail qui tentera de le retraiter. Le rôle de travail peut vérifier le nombre de fois où un message de file d’attente a été lu et, si nécessaire, l’indiquer comme message « incohérent » en vue d’une investigation en l’envoyant vers une file d’attente distincte. Pour plus d’informations sur la lecture des messages de la file d’attente et la vérification du nombre de retraits, consultez [Obtenir des messages](/rest/api/storageservices/Get-Messages).  

Certaines erreurs provenant du stockage Table et du stockage File d’attente sont des erreurs temporaires, et votre application cliente doit inclure une logique de nouvelle tentative appropriée pour les gérer.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Cette solution ne fournit pas d’isolation des transactions. Par exemple, un client peut lire les tables **Current** et **Archive** quand le rôle de travail est entre les étapes 4-5 du diagramme, et obtenir une vue incohérente des données. Les données seront cohérentes par la suite.  
* Vous pouvez être amené à vérifier que les étapes 4-5 sont idempotent afin d’assurer la cohérence.  
* Vous pouvez mettre à l'échelle la solution en utilisant plusieurs files d'attente et instances de rôle de travail.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous souhaitez maintenir une cohérence entre des entités qui existent dans différentes partitions ou tables. Vous pouvez étendre ce modèle pour garantir la cohérence des opérations entre le stockage Table et le stockage Blob, et d’autres sources de données différentes de Stockage Azure, comme une base de données ou un système de fichiers.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Transactions de groupe d’entités](#entity-group-transactions)  
* [Fusion ou remplacement](#merge-or-replace)  

> [!NOTE]
> Si l’isolation des transactions est importante pour votre solution, redéfinissez vos tables pour pouvoir utiliser des EGT.  
> 
> 

### <a name="index-entities-pattern"></a>Modèle d’entités d’index
Permet de mettre à jour des entités d'index pour mener des recherches efficaces renvoyant des listes d'entités.  

#### <a name="context-and-problem"></a>Contexte et problème
Le stockage Table indexe automatiquement les entités en utilisant les valeurs `PartitionKey` et `RowKey`. Ainsi, une application cliente peut récupérer une entité efficacement à l’aide d’une requête de pointage. Par exemple, à l’aide de la structure de table suivante, une application cliente peut récupérer efficacement une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (valeurs `PartitionKey` et `RowKey`).  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE13.png" alt-text="Graphique de la structure de l’entité Employé où une application cliente peut récupérer efficacement une entité d’employé individuelle en utilisant le nom de service et l’ID d’employé (PartitionKey et RowKey).":::

Si vous voulez également récupérer la liste des entités d’employés en fonction de la valeur d’une autre propriété qui n’est pas unique (par exemple son nom), vous devez utiliser une analyse de partition moins efficace. Cette analyse recherche des correspondances au lieu d’utiliser un index pour rechercher directement. En effet, le stockage Table ne fournit pas d’index secondaires.  

#### <a name="solution"></a>Solution
Pour permettre la recherche par nom de famille en utilisant la structure d’entité précédente, vous devez tenir à jour des listes d’ID d’employés. Si vous voulez récupérer les entités d’employés ayant un nom donné (comme Jones), vous devez d’abord localiser la liste d’ID d’employés dont le nom est Jones, puis récupérer ces entités d’employés. Il existe trois méthodes principales pour stocker les listes d’ID d’employés :  

* L’utilisation du stockage Blob.  
* La création d'entités d'index dans la même partition que les entités des employés.  
* La création d'entités d'index dans une table ou une partition séparée.  

Option 1 : Utiliser le stockage Blob  

Créez un objet blob pour chaque nom unique et, dans chaque magasin d’objets blob, stockez une liste des valeurs de `PartitionKey` (service) et `RowKey` (ID d’employé) pour les employés ayant ce nom. Quand vous ajoutez ou supprimez un employé, vérifiez que le contenu de l’objet blob adéquat est cohérent avec les entités d’employés.  

Option n°2 : Créer des entités d’index dans la même partition  

Utilisez des entités d’index qui stockent les données suivantes :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE14.png" alt-text="Graphique montrant l’entité Employee, avec une chaîne contenant une liste des ID d’employés ayant le même nom de famille":::

La propriété `EmployeeIDs` contient une liste d’ID d’employés pour les employés portant le nom stocké dans la `RowKey`.  

Les étapes suivantes décrivent le processus à suivre quand vous ajoutez un nouvel employé. Dans cet exemple, nous ajoutons au service des ventes un employé ayant l’ID 000152 et dont le nom de famille est Jones :  

1. Récupérez l’entité de l’index ayant la valeur `PartitionKey` « Sales » et la valeur `RowKey` « Jones ». Enregistrez l'ETag de cette entité pour l'utiliser lors de l'étape 2.  
2. Créez une EGT (c’est-à-dire une opération de traitement par lot) qui insère la nouvelle entité d’employé (valeur `PartitionKey` « sales » et valeur `RowKey` « 000152 ») et met à jour l’entité d’index (valeur `PartitionKey` « sales » et valeur `RowKey` « Jones »). Pour ce faire, l’EGT ajoute le nouvel ID d’employé à la liste dans le champ EmployeeIDs. Pour plus d’informations sur les EGT, consultez [Transactions de groupe d’entités](#entity-group-transactions).  
3. Si l’EGT échoue en raison d’une erreur d’accès concurrentiel optimiste (autrement dit, quelqu’un d’autre a modifié l’entité d’index), vous devez recommencer à l’étape 1.  

Vous pouvez adopter une approche similaire pour supprimer un employé si vous utilisez la deuxième option. La modification du nom d’un employé est légèrement plus complexe, car vous devrez exécuter une EGT qui met à jour trois entités : l’entité d’employé, l’entité d’index pour l’ancien nom et l’entité d’index pour le nouveau nom. Vous devez récupérer chaque entité avant d’apporter des modifications, afin de récupérer les valeurs ETag que vous pouvez ensuite utiliser pour effectuer les mises à jour à l’aide de l’accès concurrentiel optimiste.  

Les étapes suivantes décrivent le processus à suivre quand vous devez rechercher tous les employés ayant un nom spécifique dans un service. Dans cet exemple, nous recherchons tous les employés dont le nom est Jones et qui travaillent dans le service des ventes :  

1. Récupérez l’entité de l’index ayant la valeur `PartitionKey` « Sales » et la valeur `RowKey` « Jones ».  
2. Analysez la liste des ID d’employés dans le champ `EmployeeIDs`.  
3. Si vous avez besoin de plus d’informations sur chacun de ces employés (par exemple leurs adresses e-mail), récupérez chacune des entités d’employés à l’aide de la valeur `PartitionKey` « Sales » et des valeurs `RowKey` de la liste des employés obtenue à l’étape 2.  

Option 3 : Créer des entités d’index dans une table ou une partition séparée  

Pour cette méthode, utilisez les entités d’index qui stockent les données suivantes :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE15.png" alt-text="Capture d’écran qui montre l’entité d’index Employé qui contient une liste des ID d’employés pour les employés dont le nom de famille est stocké dans RowKey et PartitionKey.":::

La propriété `EmployeeIDs` contient une liste d’ID d’employés pour les employés portant le nom stocké dans la `RowKey` et `PartitionKey`.  

Vous ne pouvez pas utiliser des EGT pour maintenir la cohérence, car les entités d’index sont dans une partition distincte des entités d’employés. Vérifiez que les entités d’index sont cohérentes avec les entités d’employé.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Cette solution nécessite au moins deux requêtes pour récupérer des entités correspondantes : une pour interroger les entités d’index afin d’obtenir la liste des valeurs `RowKey`, puis des requêtes pour récupérer chaque entité dans la liste.  
* Étant donné qu’une entité a une taille maximale de 1 Mo, l’utilisation des méthodes n°2 et n°3 dans la solution partent du principe que la liste d’ID d’employés pour un nom donné n’est jamais supérieure à 1 Mo. Si la liste d’ID d’employés est susceptible d’être supérieure à 1 Mo, appliquez la méthode n°1 et stockez les données d’index dans le stockage Blob.  
* Si vous appliquez la méthode n°2 (utilisation d’EGT pour gérer l’ajout et la suppression des employés et la modification du nom d’un employé), vous devez évaluer si le volume des transactions atteint les limites de scalabilité dans une partition donnée. Si c’est le cas, vous devez envisager d’adopter une solution cohérente (option 1 ou option 3). Celles-ci utilisent des files d’attente pour gérer les requêtes de mise à jour, et vous permettent de stocker vos entités d’index dans une partition distincte des entités d’employés.  
* L’option 2 de cette solution part du principe que vous souhaitez effectuer une recherche par nom de famille dans un service. Par exemple, vous souhaitez récupérer une liste d’employés dont le nom de famille est Jones dans le service des ventes. Si vous souhaitez pouvoir rechercher dans toute l’organisation tous les employés portant le nom de famille Jones, appliquez la méthode n°1 ou n°3.
* Vous pouvez implémenter une solution basée sur une file d’attente qui assure la cohérence finale. Pour plus d’informations, consultez [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern).  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous souhaitez rechercher un jeu d’entités qui partagent toutes une valeur de propriété courante, comme l’ensemble des employés dont le nom de famille est Jones.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle de clé composée](#compound-key-pattern)  
* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern)  
* [Transactions de groupe d’entités](#entity-group-transactions)  
* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)  

### <a name="denormalization-pattern"></a>Modèle de dénormalisation
Combinez des données connexes dans une entité unique pour pouvoir récupérer toutes les données dont vous avez besoin avec une seule requête de pointage.  

#### <a name="context-and-problem"></a>Contexte et problème
Dans une base de données relationnelle, vous normalisez généralement des données pour supprimer les doublons générés quand des requêtes extraient des données provenant de plusieurs tables. Si vous normalisez des données dans les tables Azure, vous devez effectuer plusieurs allers-retours à partir du client vers le serveur pour récupérer des données associées. Par exemple, avec la structure de table suivante, vous avez besoin de deux allers-retours pour récupérer les détails d’un service. Un aller-retour extrait l’entité Department qui comprend l’ID du responsable, et le deuxième aller-retour extrait les détails du responsable dans une entité Employee.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE16.png" alt-text="Graphique d’entité Department et d’entité Employee":::

#### <a name="solution"></a>Solution
Au lieu de stocker les données dans les deux entités distinctes, dénormalisez les données et conservez une copie des détails du responsable dans l’entité du service. Par exemple :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE17.png" alt-text="Graphique d’entité Department dénormalisée et combinée":::

Une fois les entités de service stockées avec ces propriétés, vous pouvez récupérer toutes les informations nécessaires sur un service à l’aide d’une requête de pointage.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Des coûts réels sont associés au stockage des données à deux reprises. L’avantage en matière de performances résultant d’un nombre réduit de requêtes au stockage Table compense généralement l’augmentation marginale des coûts de stockage. En outre, ce coût est partiellement compensé par une réduction du nombre de transactions dont vous avez besoin pour extraire les détails d’un service.  
* Vous devez conserver la cohérence des deux entités qui stockent des informations sur les responsables. Vous pouvez gérer le problème de cohérence en faisant appel à des EGT pour mettre à jour plusieurs entités en une seule transaction atomique. Dans ce cas, l’entité Department et l’entité Employee pour le responsable du service sont stockées dans la même partition.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous devez fréquemment rechercher des informations connexes. Ce modèle réduit le nombre de requêtes que votre client doit effectuer pour récupérer les données requises.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle de clé composée](#compound-key-pattern)  
* [Transactions de groupe d’entités](#entity-group-transactions)  
* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)

### <a name="compound-key-pattern"></a>Modèle de clé composée
utiliser des valeurs de `RowKey` composées pour permettre à un client de rechercher des données connexes en utilisant une seule requête de pointage.  

#### <a name="context-and-problem"></a>Contexte et problème
Dans une base de données relationnelle, il est naturel d’utiliser des jointures dans les requêtes pour retourner des données connexes au client dans une seule requête. Par exemple, vous pouvez utiliser l’ID d’employé pour rechercher une liste d’entités associées qui contiennent des données de performances et d’évaluation de cet employé.  

Supposez que vous stockez des entités relatives aux employés dans le stockage Table à l’aide de la structure suivante :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE18.png" alt-text="Graphique de la structure d’entité Employé que vous devez utiliser pour stocker les entités d’employés dans le stockage Table.":::

Vous devez également stocker les données historiques relatives aux évaluations et aux performances de chaque année durant laquelle l’employé a travaillé pour votre organisation, et vous devez être en mesure d’accéder à ces informations par année. Une option consiste à créer une autre table qui stocke les entités avec la structure suivante :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE19.png" alt-text="Graphique d’entité d’évaluation d’employé":::

Avec cette approche, vous pouvez décider de dupliquer certaines informations (telles que le prénom et le nom) dans la nouvelle entité afin de pouvoir récupérer vos données avec une requête unique. Cependant, vous ne pouvez pas conserver une cohérence forte, car vous ne pouvez pas utiliser une EGT pour mettre à jour les deux entités de manière atomique.  

#### <a name="solution"></a>Solution
Stockez un nouveau type d’entité dans votre table d’origine à l’aide d’entités structurées comme suit :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE20.png" alt-text="Graphique d’entité Employee avec clé composée":::

Notez que la `RowKey` est désormais une clé composée de l’ID d’employé et de l’année des données d’évaluation. Cela vous permet de récupérer les performances de l’employé et d’examiner les données avec une seule requête pour une seule entité.  

L'exemple suivant indique comment vous pouvez récupérer toutes les données d'évaluation d'un employé donné (par exemple, l'employé 000123 du service des ventes) :  

$filter=(PartitionKey eq 'Sales') and (RowKey ge 'empid_000123') and (RowKey lt 'empid_000124')&$select=RowKey,Manager Rating,Peer Rating,Comments  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Vous devez utiliser un caractère de séparation approprié pour faciliter l’analyse des valeurs de `RowKey` : par exemple, **000123_2012**.  
* Vous stockez également cette entité dans la même partition que les autres entités qui contiennent des données associées au même employé. Cela signifie que vous pouvez utiliser des EGT pour maintenir une forte cohérence.
* Vous devez prendre en compte la fréquence à laquelle vous interrogez les données afin de déterminer si ce modèle est approprié. Par exemple, si vous accédez rarement aux données d’évaluation et souvent aux données principales de l’employé, vous devez les conserver en tant qu’entités distinctes.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous devez stocker une ou plusieurs entités connexes que vous interrogez fréquemment.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Transactions de groupe d’entités](#entity-group-transactions)  
* [Utiliser des types d’entités hétérogènes](#work-with-heterogeneous-entity-types)  
* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern)  

### <a name="log-tail-pattern"></a>Modèle de fin de journal
Récupérez les *n* dernières entités ajoutées à une partition en utilisant une valeur de `RowKey` qui effectue un tri dans l’ordre inverse de la date et de l’heure.  

> [!NOTE]
> Les résultats de la requête renvoyés par l’API Table Azure dans Azure Cosmos DB ne sont pas triés par clé de partition ou clé de ligne. Ainsi, ce modèle convient au stockage Table, mais pas à Azure Cosmos DB. Pour obtenir la liste détaillée des différences de fonctionnalités, consultez [Différences entre l'API Table dans Azure Cosmos DB et Stockage Table Azure](table-api-faq.md#table-api-vs-table-storage).

#### <a name="context-and-problem"></a>Contexte et problème
Une exigence courante est de pouvoir récupérer les toutes dernières entités créées, par exemple les dix dernières notes de frais soumises par un employé. Les requêtes de table prennent en charge une opération de requête `$top` pour retourner les *n* premières entités d’un ensemble. Il n’existe aucune opération de requête équivalente pour retourner les *n* dernières entités d’un ensemble.  

#### <a name="solution"></a>Solution
Stockez les entités en utilisant une `RowKey` qui trie naturellement par ordre inverse de date et d’heure, afin que l’entrée la plus récente soit toujours la première dans la table.  

Par exemple, pour être en mesure de récupérer les dix notes de frais les plus récentes soumises par un employé, vous pouvez utiliser une valeur de graduation inverse dérivée de l'heure actuelle. L’exemple de code C# suivant montre une méthode de création d’une valeur de « graduation inverse » adéquate pour une `RowKey` qui procède à un tri du plus récent au plus ancien :  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

Vous pouvez récupérer la valeur de date-heure à l’aide du code suivant :  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

La requête de table ressemble à ceci :  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Vous devez faire précéder la valeur de graduation inverse par des zéros non significatifs pour garantir que la valeur de chaîne trie comme prévu.  
* Vous devez être conscient des objectifs d'évolutivité au niveau d'une partition. Veillez à ne pas créer des partitions de zone sensible.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle quand vous avez besoin d’accéder aux entités dans l’ordre inverse de date-heure, ou quand vous devez accéder aux entités ajoutées récemment.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Ajouter un anti-modèle ou un préfixe d’anti-modèle](#prepend-append-anti-pattern)  
* [Récupérer des entités](#retrieve-entities)  

### <a name="high-volume-delete-pattern"></a>Modèle de suppression de volume élevé
activer la suppression d’un volume élevé d’entités en stockant toutes les entités pour les supprimer simultanément dans leur propre table distincte. Vous supprimez les entités en supprimant la table.  

#### <a name="context-and-problem"></a>Contexte et problème
De nombreuses applications suppriment les anciennes données qui n’ont plus besoin d’être disponibles pour une application cliente ou archivées par l’application sur un autre support de stockage. En général, vous identifiez ces données par date. Par exemple, vous devez supprimer les enregistrements de toutes les requêtes de connexion datant de plus de 60 jours.  

Une conception possible consiste à utiliser la date et l’heure de la requête de connexion dans la `RowKey` :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE21.png" alt-text="Graphique d’entité de tentative de connexion":::

Cette approche évite les zones sensibles de partition, car l’application peut insérer et supprimer des entités de connexion pour chaque utilisateur dans une partition distincte. Toutefois, cette approche peut s’avérer coûteuse et fastidieuse si vous avez un grand nombre d’entités. Tout d’abord, vous devez effectuer une analyse de table afin d’identifier toutes les entités à supprimer, puis vous devez supprimer chaque entité ancienne. Vous pouvez réduire le nombre d’allers-retours vers le serveur requis pour supprimer les anciennes entités en traitant par lots plusieurs demandes de suppression dans les TGE.  

#### <a name="solution"></a>Solution
Utilisez une table distincte pour chaque jour de tentative de connexion. Vous pouvez utiliser la conception d’entité précédente pour éviter les zones sensibles quand vous insérez des entités. La suppression des anciennes entités consiste désormais à simplement supprimer une table tous les jours (une seule opération de stockage) au lieu de rechercher et de supprimer des centaines de milliers d’entités de connexion individuelle chaque jour.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Votre conception prend-elle en charge les autres méthodes de traitement des données que votre application va utiliser, telles que la recherche d’entités spécifiques, les liaisons avec d’autres données ou la génération d’informations d’agrégation ?  
* Votre conception évite-t-elle les zones sensibles lorsque vous insérez de nouvelles entités ?  
* Vous devez attendre un délai si vous voulez réutiliser le même nom de table après l'avoir supprimée. Il est préférable de toujours utiliser des noms de table uniques.  
* Attendez-vous à une limitation du débit lors de l’utilisation initiale d’une nouvelle table, pendant que le stockage Table assimile les modèles d’accès et distribue les partitions parmi les nœuds. Vous devez réfléchir à la fréquence à laquelle vous devez créer des tables.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous avez un volume élevé d'entités, que vous devez supprimer en même temps.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Transactions de groupe d’entités](#entity-group-transactions)
* [Modifier des entités](#modify-entities)  

### <a name="data-series-pattern"></a>Modèle de série de données
Stockez des séries de données complètes dans une entité unique pour réduire votre nombre de demandes.  

#### <a name="context-and-problem"></a>Contexte et problème
Il arrive qu'une application stocke une série de données qu'elle doit fréquemment récupérer en une seule fois : Par exemple, votre application peut enregistrer combien de messages de messagerie instantanée chaque employé envoie toutes les heures, puis utiliser ces informations pour tracer le nombre de messages envoyés par chaque utilisateur dans les 24 heures précédentes. Une conception peut consister à stocker 24 entités pour chaque employé :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE22.png" alt-text="Graphique d’entité de statistiques de message":::

Grâce à cette conception, vous pouvez facilement rechercher et mettre à jour l'entité mise à jour pour chaque employé chaque fois que l'application doit mettre à jour la valeur de nombre de messages. Cependant, pour récupérer les informations pour tracer un graphique de l'activité des 24 heures précédentes, vous devez récupérer les 24 entités.  

#### <a name="solution"></a>Solution
Utilisez la conception suivante, avec une propriété distincte pour stocker le nombre de messages pour chaque heure :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE23.png" alt-text="Graphique montrant l’entité de statistiques de message avec des propriétés distinctes":::

Grâce à cette conception, vous pouvez utiliser une opération de fusion pour mettre à jour le nombre de messages pour un employé pour une heure spécifique. À présent, vous pouvez récupérer toutes les informations dont vous avez besoin pour tracer le graphique à l’aide d’une requête pour une seule entité.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Si votre série de données complète ne tient pas dans une seule entité (une entité peut avoir jusqu’à 252 propriétés), utilisez un autre magasin de données, comme un objet blob.  
* Si vous avez plusieurs clients qui mettent à jour une entité simultanément, utilisez le **ETag** pour implémenter l’accès concurrentiel optimiste. Si vous avez de nombreux clients, vous pouvez observer une contention élevée.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous devez mettre à jour et récupérer une série de données associée à une entité individuelle.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle d'entités volumineuses](#large-entities-pattern)  
* [Fusion ou remplacement](#merge-or-replace)  
* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern) (si vous stockez la série de données dans un objet blob)  

### <a name="wide-entities-pattern"></a>Modèle d’entités larges
Utilisez plusieurs entités physiques pour stocker des entités logiques ayant plus de 252 propriétés.  

#### <a name="context-and-problem"></a>Contexte et problème
Une entité individuelle ne peut pas avoir plus de 252 propriétés (à l’exception des propriétés système obligatoires) et ne peut pas stocker plus de 1 Mo de données au total. Dans une base de données relationnelle, vous contourneriez généralement les limites de taille d’une ligne en ajoutant une nouvelle table et en appliquant une relation un-à-un entre elles.  

#### <a name="solution"></a>Solution
À l’aide du stockage Table, vous pouvez stocker plusieurs entités pour représenter un objet métier volumineux unique ayant plus de 252 propriétés. Par exemple, si vous souhaitez stocker le nombre de messages instantanés envoyés par chaque employé durant les 365 derniers jours, vous pouvez utiliser la conception suivante qui utilise deux entités avec des schémas différents :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE24.png" alt-text="Graphique montrant l’entité de statistiques de message avec RowKey 01 et l’entité de statistiques de message avec RowKey 02":::

Si vous souhaitez apporter une modification qui nécessite la mise à jour des deux entités pour les garder mutuellement synchronisées, vous pouvez utiliser une EGT. Sinon, vous pouvez utiliser une opération de fusion pour mettre à jour le nombre de messages pour un jour spécifique. Pour récupérer toutes les données d’un employé, vous devez récupérer les deux entités. Vous pouvez effectuer cette opération avec deux requêtes efficaces qui utilisent à la fois une `PartitionKey` et une valeur `RowKey`.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants quand vous choisissez comment implémenter ce modèle :  

* La récupération d'une entité logique complète implique au moins deux transactions de stockage : une pour récupérer chaque entité physique.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle quand vous avez besoin de stocker des entités dont la taille ou le nombre de propriétés dépassent les limites d’une entité individuelle dans le stockage Table.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Transactions de groupe d’entités](#entity-group-transactions)
* [Fusion ou remplacement](#merge-or-replace)

### <a name="large-entities-pattern"></a>Modèle d’entités volumineuses
Utilisez le stockage Blob pour stocker des valeurs de propriétés volumineuses.  

#### <a name="context-and-problem"></a>Contexte et problème
Une entité individuelle ne peut pas stocker plus de 1 Mo de données au total. Si une ou plusieurs des propriétés stockent des valeurs qui provoquent un dépassement de la taille totale de votre entité, vous ne pouvez pas stocker l’intégralité de l’entité dans le stockage Table.  

#### <a name="solution"></a>Solution
Si votre entité dépasse 1 Mo car une ou plusieurs propriétés contiennent une grande quantité de données, vous pouvez stocker des données dans le stockage Blob et stocker ensuite l’adresse de l’objet blob dans une propriété de l’entité. Par exemple, vous pouvez stocker la photo d’un employé dans le stockage Blob, et stocker un lien vers la photo dans la propriété `Photo` de votre entité d’employé :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE25.png" alt-text="Graphique montrant l’entité Employee avec chaîne Photo qui pointe vers le stockage Blob":::

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* Pour maintenir la cohérence entre l’entité du stockage Table et les données du stockage Blob, utilisez le [modèle de transactions cohérentes](#eventually-consistent-transactions-pattern) pour tenir à jour vos entités.
* La récupération d'une entité complète implique au moins deux transactions de stockage : une pour récupérer l'entité et l'autre pour récupérer les données blob.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle quand vous avez besoin de stocker des entités dont la taille dépasse les limites d’une entité individuelle dans le stockage Table.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle de transactions cohérentes](#eventually-consistent-transactions-pattern)  
* [Modèle d’entités larges](#wide-entities-pattern)

<a name="prepend-append-anti-pattern"></a>

### <a name="prependappend-anti-pattern"></a>Ajouter un anti-modèle ou un préfixe d'anti-modèle
Quand vous avez un volume élevé d’insertions, augmentez la scalabilité en répartissant les insertions parmi plusieurs partitions.  

#### <a name="context-and-problem"></a>Contexte et problème
L'ajout d'entité ou de suffixe d'entité à vos entités stockées pousse généralement l'application à ajouter de nouvelles entités à la première ou à la dernière partition d'une séquence. Dans ce cas, toutes les insertions à un moment donné se produisent dans la même partition, ce qui crée une zone sensible. Cela empêche le stockage Table d’effectuer des insertions d’équilibrage de charge sur plusieurs nœuds, et peut éventuellement obliger votre application à atteindre les objectifs de scalabilité pour la partition. Par exemple, considérez le cas d’une application qui journalise l’accès au réseau et aux ressources par les employés. Une structure d’entité telle que la suivante peut entraîner la saturation de la partition de l’heure actuelle, si le volume des transactions atteint l’objectif de scalabilité pour une partition individuelle :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE26.png" alt-text="Graphique d’une structure d’entité qui peut entraîner la saturation de la partition de l’heure actuelle, si le volume des transactions atteint l’objectif de scalabilité pour une partition individuelle.":::

#### <a name="solution"></a>Solution
La structure d’entité alternative suivante permet d’éviter une zone sensible dans n’importe quelle partition particulière tandis que l’application journalise les événements :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE27.png" alt-text="Graphique montrant l’entité Employee avec RowKey combinant l’année, le mois, le jour, l’heure et l’ID d’événement":::

Dans cet exemple, notez que `PartitionKey` et `RowKey` sont toutes deux des clés composées. `PartitionKey` utilise les ID de service et d’employé pour distribuer la journalisation sur plusieurs partitions.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous choisissez comment implémenter ce modèle :  

* La structure clé alternative qui évite de créer efficacement des partitions sensibles sur des insertions prend-elle en charge efficacement les requêtes effectuées par votre application cliente ?  
* Le volume prévu de transactions signifie-t-il que vous êtes susceptible d’atteindre les objectifs de scalabilité pour une partition individuelle et d’être limité par le stockage Table ?  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Évitez l’ajout d’anti-modèle ou de suffixe d’anti-modèle quand votre volume de transactions est susceptible d’entraîner la limitation du débit par le stockage Table quand vous accédez à une partition sensible.  

#### <a name="related-patterns-and-guidance"></a>Conseils et modèles connexes
Les modèles et les conseils suivants peuvent aussi présenter un intérêt quand il s’agit d’implémenter ce modèle :  

* [Modèle de clé composée](#compound-key-pattern)  
* [Modèle de fin de journal](#log-tail-pattern)  
* [Modifier des entités](#modify-entities)  

### <a name="log-data-anti-pattern"></a>Journalisation de l'anti-modèle de données
En règle générale, vous devez utiliser le stockage Blob plutôt que le stockage Table pour stocker les données du journal.  

#### <a name="context-and-problem"></a>Contexte et problème
Un cas d’usage courant pour les données de journal consiste à récupérer une sélection d’entrées de journal pour une plage de dates/heures spécifique. Par exemple, vous souhaitez rechercher tous les messages d’erreur et messages critiques enregistrés par votre application entre 15:04 et 15:06 à une date spécifique. Vous ne souhaitez pas utiliser la date et l’heure du message de journal pour déterminer la partition sur laquelle vous enregistrez les entités de journal. Cela aboutit à une partition sensible, car à tout moment donné, toutes les entités du journal partagent la même valeur de `PartitionKey` (voir [Ajouter un anti-modèle ou un préfixe d’anti-modèle](#prepend-append-anti-pattern)). Par exemple, le schéma d’entité suivant d’un message de journal génère une partition sensible, car l’application écrit tous les messages de journal sur la partition pour la date et l’heure actuelles :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE28.png" alt-text="Graphique montrant un schéma d’entité pour un message de journalisation qui aboutit à une partition sensible.":::

Dans cet exemple, la `RowKey` contient la date et l’heure du message de journal pour s’assurer que les messages du journal sont triés par ordre de date/heure. La `RowKey` comprend également un ID de message, pour le cas où plusieurs messages de journal partageraient les mêmes date et heure.  

Une autre approche consiste à utiliser une valeur de `PartitionKey` qui garantit que l’application écrit des messages dans une plage de partitions. Par exemple, si la source du message de journal offre un moyen de distribuer les messages sur plusieurs partitions, vous pouvez utiliser le schéma d’entité suivant :  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE29.png" alt-text="Graphique d’entité de message de journal":::

Toutefois, le problème avec ce schéma est que, pour récupérer tous les messages de journal pour un intervalle de temps spécifique, vous devez rechercher sur chaque partition dans la table.

#### <a name="solution"></a>Solution
La section précédente a présenté le problème posé par la tentative d’utilisation du stockage Table pour stocker des entrées de journal et a suggéré deux conceptions, peu satisfaisantes. Une solution a conduit à une partition sensible, avec le risque de faibles performances d’écriture de messages de journal. L’autre solution a entraîné des performances médiocres, en raison de la nécessité d’analyser chaque partition dans la table pour récupérer les messages du journal pour une période spécifique. Le stockage Blob offre une meilleure solution pour ce type de scénario. Voici comment Azure Storage Analytics stocke les données de journal collectées.  

Cette section décrit comment Storage Analytics stocke les données de journal dans le stockage Blob, en guise d’illustration de cette approche pour le stockage des données que vous interrogez généralement par plage.  

Storage Analytics stocke les messages de journal dans un format délimité dans plusieurs objets blob. Ce format facilite l'analyse des données du message de journalisation pour une application cliente.  

Storage Analytics utilise une convention de nommage pour les objets blob qui vous permet de localiser le ou les objets blob contenant les messages de journal que vous recherchez. Par exemple, un objet blob nommé « queue/2014/07/31/1800/000001.log » contient des messages de journal liés au service de File d’attente dont l’heure de début est à 18h00 le 31 juillet 2014. Le « 000001 » indique qu'il s'agit du premier fichier journal pour cette période. Storage Analytics enregistre également les horodatages du premier et du dernier messages stockés dans le fichier dans le cadre des métadonnées de l’objet blob. L’API pour le stockage Blob vous permet de localiser des objets blob dans un conteneur en fonction d’un préfixe de nom. Pour localiser tous les objets blob qui contiennent des données de journal de file d’attente pour l’heure commençant à 18:00, vous pouvez utiliser le préfixe « queue/2014/07/31/1800 ».  

Storage Analytics met en mémoire tampon les messages de journal en interne, puis met à jour de façon périodique l’objet blob adéquat ou en crée un autre avec le dernier lot d’entrées de journal. Cela réduit le nombre d’écritures qu’il doit exécuter vers le stockage Blob.  

Si vous implémentez une solution similaire dans votre propre application, réfléchissez à la manière de gérer le compromis entre la fiabilité, le coût et la scalabilité. En d’autres termes, évaluez l’effet de l’écriture de chaque entrée de journal dans le stockage Blob, par rapport à la mise en mémoire tampon des mises à jour dans votre application et leur écriture dans le stockage Blob par lots.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Prenez en compte les points suivants lorsque vous décidez de la manière de stocker des données de journalisation :  

* Si vous créez une conception de table qui évite les risques de partitions sensibles, il est possible que l’accès aux données de journal ne soit pas très efficace.  
* Pour traiter les données de journalisation, un client doit souvent charger de nombreux enregistrements.  
* Bien que les données de journal soient souvent structurées, le stockage Blob peut être une meilleure solution.  

### <a name="implementation-considerations"></a>Considérations relatives à l’implémentation
Cette section décrit certaines des considérations à prendre en compte lorsque vous implémentez les modèles décrits dans les sections précédentes. La plupart de cette section utilise des exemples rédigés en C#, ainsi que la bibliothèque cliente de stockage (version 4.3.0 au moment de la rédaction).  

### <a name="retrieve-entities"></a>Récupérer des entités
Comme indiqué dans la section [Conception pour l’interrogation](#design-for-querying), la requête la plus efficace est une requête de pointage. Toutefois, dans certains scénarios vous devrez peut-être récupérer plusieurs entités. Cette section décrit certaines des approches courantes pour récupérer des entités à l’aide de la bibliothèque de client de stockage.  

#### <a name="run-a-point-query-by-using-the-storage-client-library"></a>Exécuter une requête de pointage à l’aide de la bibliothèque de client de stockage
Le moyen le plus simple d’exécuter une requête de pointage est d’utiliser l’opération de table **Retrieve**. Comme indiqué dans l’extrait de code C# suivant, cette opération récupère une entité avec une `PartitionKey` ayant la valeur « sales » et une `RowKey` ayant la valeur « 212 » :  

```csharp
TableOperation retrieveOperation = TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
var retrieveResult = employeeTable.Execute(retrieveOperation);
if (retrieveResult.Result != null)
{
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
}  
```

Notez que cet exemple part du principe que l’entité récupérée doit être de type `EmployeeEntity`.  

#### <a name="retrieve-multiple-entities-by-using-linq"></a>Récupérer plusieurs entités à l’aide de LINQ
Vous pouvez récupérer plusieurs entités à l’aide de LINQ avec la bibliothèque de client de stockage, en spécifiant une requête avec une clause **where**. Pour éviter une analyse de table, vous devez toujours inclure la valeur de `PartitionKey` dans la clause where, et si possible la valeur de `RowKey` afin d’éviter les analyses de table et de partition. Le stockage Table prend en charge un ensemble limité d’opérateurs de comparaison (supérieur à, supérieure ou égal à, inférieur ou égal à, égal, et différent de). L’extrait de code C# suivant recherche tous les employés dont le nom commence par « B » (en supposant que la `RowKey` stocke le nom de famille) dans le service Sales (en supposant que la `PartitionKey` stocke le nom du service) :  

```csharp
TableQuery<EmployeeEntity> employeeQuery = employeeTable.CreateQuery<EmployeeEntity>();
var query = (from employee in employeeQuery
            where employee.PartitionKey == "Sales" &&
            employee.RowKey.CompareTo("B") >= 0 &&
            employee.RowKey.CompareTo("C") < 0
            select employee).AsTableQuery();
var employees = query.Execute();  
```

Notez que la requête spécifie à la fois une `RowKey` et une `PartitionKey` pour garantir des performances optimales.  

L’exemple de code suivant montre une fonctionnalité équivalente faisant appel à l’API Fluent (pour plus d’informations sur cette API, consultez [Bonnes pratiques pour la conception d’une API Fluent](https://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx)) :  

```csharp
TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
    TableQuery.CombineFilters(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition(
    "PartitionKey", QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.GenerateFilterCondition(
    "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
),
TableOperators.And,
TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
    )
);
var employees = employeeTable.ExecuteQuery(employeeQuery);  
```

> [!NOTE]
> L’exemple imbrique plusieurs méthodes `CombineFilters` pour inclure les trois conditions de filtre.  
> 
> 

#### <a name="retrieve-large-numbers-of-entities-from-a-query"></a>Récupérer un grand nombre d’entités à partir d’une requête
Une requête optimale retourne une entité individuelle basée sur une valeur de `PartitionKey` et une valeur de `RowKey`. Toutefois, dans certains scénarios, vous pouvez être obligé de retourner de nombreuses entités à partir de la même partition ou même de plusieurs partitions. Vous devez toujours tester entièrement les performances de votre application dans de tels scénarios.  

Une requête sur le stockage Table peut retourner un maximum de 1000 entités à la fois, et s’exécuter pendant un maximum de cinq secondes. Le stockage Table retourne un jeton de continuation pour permettre à l’application cliente de demander le jeu d’entités suivant, si l’une des conditions suivantes est vraie :

- Le jeu de résultats contient plus de 1000 entités.
- La requête ne s’est pas terminée dans les cinq secondes.
- La requête franchit les limites de la partition. 

Pour plus d’informations sur le fonctionnement des jetons de continuation, consultez [Délai de requête et pagination](/rest/api/storageservices/Query-Timeout-and-Pagination).  

Si vous utilisez la bibliothèque de client de stockage, celle-ci peut gérer automatiquement les jetons de continuation pour vous lors du retour des entités à partir du stockage Table. Par exemple, le code C# suivant gère automatiquement les jetons de continuation si le stockage Table les retourne dans une réponse :  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

var employees = employeeTable.ExecuteQuery(employeeQuery);
foreach (var emp in employees)
{
        ...
}  
```

Le code C# suivant gère les jetons de continuation de manière explicite :  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

TableContinuationToken continuationToken = null;

do
{
        var employees = employeeTable.ExecuteQuerySegmented(
        employeeQuery, continuationToken);
    foreach (var emp in employees)
    {
    ...
    }
    continuationToken = employees.ContinuationToken;
} while (continuationToken != null);  
```

En utilisant des jetons de continuation de manière explicite, vous pouvez contrôler le moment où votre application extrait le segment suivant des données. Par exemple, si votre application cliente permet aux utilisateurs de parcourir les entités stockées dans une table, un utilisateur peut décider de ne pas parcourir toutes les entités récupérées par la requête. Votre application utiliserait uniquement un jeton de continuation pour récupérer le segment suivant quand l’utilisateur aurait terminé de parcourir toutes les entités du segment actuel. Cette approche présente plusieurs avantages :  

* Vous pouvez limiter la quantité de données à récupérer à partir du stockage Table et que vous placez sur le réseau.  
* Vous pouvez effectuer des E/S asynchrones dans .NET.  
* Vous pouvez sérialiser le jeton de continuation dans un stockage permanent, afin de pouvoir continuer même si l’application se plante.  

> [!NOTE]
> En général, un jeton de continuation retourne un segment contenant 1000 entités, bien qu’il puisse y en avoir moins. C’est également le cas si vous limitez le nombre d’entrées retournées par une requête à l’aide de **Take** pour retourner les n premières entités qui correspondent à vos critères de recherche. Le stockage Table peut retourner un segment contenant moins de n entités, ainsi qu’un jeton de continuation pour vous permettre de récupérer les entités restantes.  
> 
> 

Le code C# suivant montre comment modifier le nombre d'entités renvoyées à l'intérieur d'un segment :  

```csharp
employeeQuery.TakeCount = 50;  
```

#### <a name="server-side-projection"></a>Projection côté serveur
Une seule entité peut avoir jusqu'à 255 propriétés et une taille allant jusqu'à 1 Mo. Quand vous interrogez la table et récupérez des entités, il est possible que vous n’ayez pas besoin de toutes les propriétés et que vous puissiez éviter de transférer des données quand ça n’est pas nécessaire (ce qui permet de réduire la latence et les coûts). Vous pouvez utiliser la projection côté serveur pour transférer uniquement les propriétés que vous avez besoin. L’exemple suivant récupère uniquement la propriété `Email` (avec les valeurs de `PartitionKey`, `RowKey`, `Timestamp` et `ETag`) à partir des entités sélectionnées par la requête.  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
List<string> columns = new List<string>() { "Email" };
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

var entities = employeeTable.ExecuteQuery(employeeQuery);
foreach (var e in entities)
{
        Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
}  
```

Notez que la valeur de `RowKey` est disponible même si elle ne figure pas dans la liste de propriétés à récupérer.  

### <a name="modify-entities"></a>Modifier des entités
La bibliothèque de client de stockage vous permet de modifier les entités stockées dans le stockage Table pour les insérer, les supprimer et les mettre à jour. Vous pouvez utiliser des EGT pour traiter par lots plusieurs opérations d’insertion, mise à jour et suppression, afin de réduire le nombre d’allers-retours requis et d’améliorer les performances de votre solution.  

Les exceptions levées quand la bibliothèque de client de stockage exécute une EGT incluent généralement l’index de l’entité qui a provoqué l’échec du lot. Cela est utile lorsque vous déboguez du code qui utilise des EGT.  

Nous vous conseillons de réfléchir également à la façon dont votre conception affecte la méthode de votre application cliente pour gérer les opérations d'accès concurrentiel et de mises à jour.  

#### <a name="managing-concurrency"></a>Gérer l'accès concurrentiel
Par défaut, le stockage Table implémente des contrôles d’accès concurrentiel optimiste au niveau des entités individuelles pour les opérations d’insertion, de fusion et de suppression, bien qu’il soit possible pour un client de forcer le stockage Table à ignorer ces contrôles. Pour plus d’informations, consultez [Gestion de l’accès concurrentiel dans Microsoft Azure Storage](../storage/common/storage-concurrency.md).  

#### <a name="merge-or-replace"></a>Fusion ou remplacement
La méthode `Replace` de la classe `TableOperation` remplace toujours l’entité complète dans le stockage Table. Si vous n’incluez pas une propriété dans la requête quand cette propriété existe dans l’entité stockée, la requête supprime cette propriété de l’entité stockée. Si vous ne souhaitez pas supprimer une propriété explicitement à partir d'une entité stockée, vous devez inclure chaque propriété dans la demande.  

Vous pouvez utiliser la méthode `Merge` de la classe `TableOperation` pour réduire la quantité de données que vous envoyez au stockage Table quand vous souhaitez mettre à jour une entité. La méthode `Merge` remplace toutes les propriétés de l’entité stockée par des valeurs de propriété de l’entité incluse dans la requête. Cette méthode laisse intactes toutes les propriétés de l’entité stockée qui ne sont pas incluses dans la requête. Cela est utile si vous avez des entités volumineuses et que vous avez seulement besoin de mettre à jour un petit nombre de propriétés dans une requête.  

> [!NOTE]
> Les méthodes `*Replace` et `Merge` échouent si l’entité n’existe pas. En guise d’alternative, vous pouvez utiliser les méthodes `InsertOrReplace`et `InsertOrMerge`, qui créent une entité si elle n’existe pas.  
> 
> 

### <a name="work-with-heterogeneous-entity-types"></a>Utiliser des types d’entités hétérogènes
Le stockage Table est un magasin de tables *sans schéma*. Cela signifie qu’une table peut stocker des entités de plusieurs types afin d’améliorer la flexibilité de votre conception. L'exemple suivant présente une table qui stocke les entités de service et d'employé :  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

Chaque entité doit toujours avoir des valeurs `PartitionKey`, `RowKey` et `Timestamp`, mais elle peut avoir n’importe quel ensemble de propriétés. De plus, il n’y a rien pour indiquer le type d’une entité, sauf si vous choisissez de stocker cette information quelque part. Il existe deux options pour identifier le type d'une entité :  

* Ajout d’un préfixe de type d’entité à la `RowKey` (ou éventuellement à la `PartitionKey`). Par exemple, `EMPLOYEE_000123` ou `DEPARTMENT_SALES` comme valeurs de `RowKey`.  
* Utilisation d’une propriété distincte pour enregistrer le type d’entité, comme indiqué dans le tableau ci-dessous.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>department</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>FirstName</th>
<th>LastName</th>
<th>Age</th>
<th>E-mail</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

La première option, qui consiste à ajouter un préfixe de type d’entité à la `RowKey`, est utile au cas où deux entités de types différents se retrouvent avec la même valeur de clé. Il regroupe également les entités du même type dans la partition.  

Les techniques abordées dans cette section s’appliquent particulièrement à la discussion sur les [relations d’héritage](#inheritance-relationships).  

> [!NOTE]
> Pensez à inclure un numéro de version dans la valeur de type d’entité, pour permettre aux applications clientes de faire évoluer des objets POCO et de fonctionner avec différentes versions.  
> 
> 

Le reste de cette section décrit certaines des fonctionnalités de la bibliothèque cliente de stockage qui facilitent l'utilisation de plusieurs types d'entités dans la même table.  

#### <a name="retrieve-heterogeneous-entity-types"></a>Récupérer des types d’entités hétérogènes
Si vous utilisez la bibliothèque de client de stockage, vous avez trois options pour travailler avec plusieurs types d’entité.  

Si vous connaissez le type de l’entité stockée avec des valeurs `RowKey` et `PartitionKey` spécifiques, vous pouvez spécifier le type d’entité quand vous récupérez l’entité. Vous avez vu cela dans les deux exemples précédents qui récupèrent les entités de type `EmployeeEntity` : [Exécuter une requête de pointage à l’aide de la bibliothèque de client de stockage](#run-a-point-query-by-using-the-storage-client-library) et [Récupérer plusieurs entités à l’aide de LINQ](#retrieve-multiple-entities-by-using-linq).  

La deuxième option consiste à utiliser le type `DynamicTableEntity` (un conteneur de propriétés) au lieu d’un type d’entité POCO concret. Cette option peut également améliorer les performances, car il n’est pas nécessaire de sérialiser et de désérialiser l’entité en types .NET. Le code C# suivant récupère plusieurs entités de types différents à partir de la table, mais retourne toutes les entités en tant qu’instances de `DynamicTableEntity`. Il utilise ensuite la propriété `EntityType` pour déterminer le type de chaque entité :  

```csharp
string filter = TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("PartitionKey",
    QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("RowKey",
                    QueryComparisons.GreaterThanOrEqual, "B"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey",
        QueryComparisons.LessThan, "F")
    )
);
TableQuery<DynamicTableEntity> entityQuery =
    new TableQuery<DynamicTableEntity>().Where(filter);
var employees = employeeTable.ExecuteQuery(entityQuery);

IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
foreach (var e in entities)
{
EntityProperty entityTypeProperty;
if (e.Properties.TryGetValue("EntityType", out entityTypeProperty))
{
    if (entityTypeProperty.StringValue == "Employee")
    {
        // Use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
        }
    }
}  
```

Pour récupérer d’autres propriétés, vous devez utiliser la méthode `TryGetValue` sur la propriété `Properties` de la classe `DynamicTableEntity`.  

Une troisième option consiste à combiner l’utilisation du type `DynamicTableEntity` et d’une instance `EntityResolver`. Cela vous permet de résoudre plusieurs types POCO dans la même requête. Dans cet exemple, le délégué `EntityResolver` utilise la propriété `EntityType` pour faire la distinction entre les deux types d’entité retournés par la requête. La méthode `Resolve` utilise le délégué `resolver` pour résoudre les instances `DynamicTableEntity` en instances `TableEntity`.  

```csharp
EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
{

        TableEntity resolvedEntity = null;
        if (props["EntityType"].StringValue == "Department")
        {
        resolvedEntity = new DepartmentEntity();
        }
        else if (props["EntityType"].StringValue == "Employee")
        {
        resolvedEntity = new EmployeeEntity();
        }
        else throw new ArgumentException("Unrecognized entity", "props");

        resolvedEntity.PartitionKey = pk;
        resolvedEntity.RowKey = rk;
        resolvedEntity.Timestamp = ts;
        resolvedEntity.ETag = etag;
        resolvedEntity.ReadEntity(props, null);
        return resolvedEntity;
};

string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<DynamicTableEntity> entityQuery =
        new TableQuery<DynamicTableEntity>().Where(filter);

var entities = employeeTable.ExecuteQuery(entityQuery, resolver);
foreach (var e in entities)
{
        if (e is DepartmentEntity)
        {
    ...
        }
        if (e is EmployeeEntity)
        {
    ...
        }
}  
```

#### <a name="modify-heterogeneous-entity-types"></a>Modifier des types d’entités hétérogènes
Vous n’avez pas besoin de connaître le type d’une entité pour la supprimer, et vous connaissez toujours le type d’une entité quand vous l’insérez. Toutefois, vous pouvez utiliser le type `DynamicTableEntity` pour mettre à jour une entité sans connaître son type et sans utiliser de classe d’entité POCO. L’exemple de code suivant récupère une entité unique et vérifie que la propriété `EmployeeCount` existe avant de la mettre à jour.  

```csharp
TableResult result =
        employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
DynamicTableEntity department = (DynamicTableEntity)result.Result;

EntityProperty countProperty;

if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
{
        throw new
        InvalidOperationException("Invalid entity, EmployeeCount property not found.");
}
countProperty.Int32Value += 1;
employeeTable.Execute(TableOperation.Merge(department));  
```

### <a name="control-access-with-shared-access-signatures"></a>Contrôler l’accès avec des signatures d’accès partagé
Vous pouvez utiliser des signature d’accès partagé (SAS) pour permettre aux applications clientes de modifier (et d’interroger) des entités de table directement, sans avoir besoin de s’authentifier directement auprès du stockage Table. En règle générale, il existe trois principaux avantages à l'utilisation de SAP dans votre application :  

* Vous n’avez pas besoin de distribuer votre clé de compte de stockage sur une plateforme non sécurisée (par exemple un appareil mobile) pour permettre à cet appareil d’accéder à des entités dans le stockage Table et de les modifier.  
* Vous pouvez décharger une partie du travail effectué par les rôles web et de travail dans la gestion de vos entités. Vous pouvez décharger ce travail à des appareils clients tels que les appareils mobiles et les ordinateurs des utilisateurs finaux.  
* Vous pouvez affecter un ensemble d’autorisations contraintes et limitées dans le temps à un client (par exemple pour autoriser l’accès en lecture seule à des ressources spécifiques).  

Pour plus d’informations sur l’utilisation de jetons SAS avec le stockage Table, consultez [Utilisation des signatures d’accès partagé (SAS)](../storage/common/storage-sas-overview.md).  

Toutefois, vous devez toujours générer les jetons SAS qui accordent à une application cliente l’accès aux entités dans le stockage Table. Vous devez le faire dans un environnement qui dispose d’un accès sécurisé à vos clés de compte de stockage. En règle générale, vous utilisez un rôle web ou de travail pour générer les jetons SAP et les transmettre vers les applications clientes qui ont besoin d'accéder à vos entités. Comme il existe toujours une surcharge impliquée dans la génération et l'envoi de jetons SAP aux clients, vous devez envisager la meilleure méthode pour réduire cette surcharge, en particulier dans les scénarios à volumes élevés.  

Il est possible de générer un jeton SAS qui accorde l’accès à un sous-ensemble d’entités dans une table. Par défaut, vous créez un jeton SAS pour une table entière. Toutefois, il est également possible de spécifier que le jeton SAS accorde l’accès à une plage de valeurs `PartitionKey`, ou à une plage de valeurs `PartitionKey` et `RowKey`. Vous pouvez choisir de générer des jetons SAS pour des utilisateurs spécifiques de votre système, de sorte que chaque jeton SAS d’un utilisateur lui permette uniquement d’accéder à ses propres entités dans le stockage Table.  

### <a name="asynchronous-and-parallel-operations"></a>Opérations asynchrones et parallèles
Si vous effectuez la diffusion de vos demandes sur plusieurs partitions, vous pouvez améliorer le débit et la réactivité du client en utilisant des requêtes asynchrones ou parallèles.
Par exemple, vous pouvez avoir plusieurs instances de rôle de travail accédant à vos tables en parallèle. Vous pouvez avoir des rôles de travail responsables d’ensembles particuliers de partitions, ou simplement plusieurs instances de rôle de travail, chacune étant en mesure d’accéder à toutes les partitions d’une table.  

Dans une instance cliente, vous pouvez améliorer le débit en exécutant des opérations de stockage en mode asynchrone. La bibliothèque cliente de stockage facilite l'écriture des modifications et des requêtes asynchrones. Par exemple, vous pouvez commencer avec la méthode synchrone qui récupère toutes les entités d’une partition, comme illustré dans le code C# suivant :  

```csharp
private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

        TableContinuationToken continuationToken = null;

        do
        {
        var employees = employeeTable.ExecuteQuerySegmented(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
    {
        ...
    }
        continuationToken = employees.ContinuationToken;
        } while (continuationToken != null);
}  
```

Vous pouvez facilement modifier ce code afin que la requête s’exécute de façon asynchrone, comme suit :  

```csharp
private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);
        TableContinuationToken continuationToken = null;

        do
        {
        var employees = await employeeTable.ExecuteQuerySegmentedAsync(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
        {
            ...
        }
        continuationToken = employees.ContinuationToken;
            } while (continuationToken != null);
}  
```

Dans cet exemple asynchrone, vous pouvez voir les modifications suivantes par rapport à la version synchrone :  

* La signature de méthode inclut désormais le modificateur `async` et retourne une instance `Task`.  
* Au lieu d’appeler la méthode `ExecuteSegmented` pour récupérer les résultats, la méthode appelle maintenant la méthode `ExecuteSegmentedAsync`. La méthode utilise le modificateur `await` pour récupérer les résultats de façon asynchrone.  

L’application cliente peut appeler cette méthode plusieurs fois, avec des valeurs différentes pour le paramètre `department`. Chaque requête s’exécute sur un thread distinct.  

Il n’existe aucune version asynchrone de la méthode `Execute` dans la classe `TableQuery`, car l’interface `IEnumerable` ne prend pas en charge l’énumération asynchrone.  

Vous pouvez également insérer, mettre à jour et supprimer des entités de façon asynchrone. L'exemple C# suivant indique une méthode simple et synchrone pour insérer ou remplacer une entité d'employé :  

```csharp
private static void SimpleEmployeeUpsert(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = employeeTable
        .Execute(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

Vous pouvez facilement modifier ce code pour que la mise à jour s’exécute de façon asynchrone, comme suit :  

```csharp
private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = await employeeTable
        .ExecuteAsync(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

Dans cet exemple asynchrone, vous pouvez voir les modifications suivantes par rapport à la version synchrone :  

* La signature de méthode inclut désormais le modificateur `async` et retourne une instance `Task`.  
* Au lieu d’appeler la méthode `Execute` pour mettre à jour l’entité, la méthode appelle maintenant la méthode `ExecuteAsync`. La méthode utilise le modificateur `await` pour récupérer les résultats de façon asynchrone.  

L’application cliente peut appeler plusieurs méthodes asynchrones comme celle-ci, et chaque appel de méthode s’exécute sur un thread distinct.