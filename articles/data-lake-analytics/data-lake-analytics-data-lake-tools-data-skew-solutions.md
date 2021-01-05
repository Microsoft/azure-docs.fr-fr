---
title: Résoudre l’asymétrie des données - Azure Data Lake Tools pour Visual Studio
description: Solutions de dépannage potentielles pour les problèmes d'asymétrie des données à l’aide d’Azure Data Lake Tools pour Visual Studio.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 12/16/2016
ms.openlocfilehash: 0b82ce187c06afa69e54ea93931e1745f0d52674
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219188"
---
# <a name="resolve-data-skew-problems-by-using-azure-data-lake-tools-for-visual-studio"></a>Résolution de problèmes d'asymétrie des données à l’aide d'Azure Data Lake Tools pour Visual Studio

## <a name="what-is-data-skew"></a>Qu’est-ce que l'asymétrie des données ?

en résumé, l'asymétrie des données correspond à une valeur surreprésentée. Imaginez que vous avez affecté 50 contrôleurs fiscaux pour auditer des déclarations fiscales, un contrôleur pour chaque état américain. Le contrôleur affecté au Wyoming a peu de travail, car la population y est restreinte. En Californie, cependant, le contrôleur est très occupé en raison de la grande population de cet état.
    ![Exemple de problème d'asymétrie des données](./media/data-lake-analytics-data-lake-tools-data-skew-solutions/data-skew-problem.png)

Dans notre scénario, les données sont distribuées de manière inégale entre les contrôleurs fiscaux, ce qui signifie que certains d'entre eux doivent travailler plus que d’autres. Dans votre propre travail, vous rencontrez souvent des situations telles que celle-ci. En termes techniques, un vertex obtient beaucoup plus de données que ses pairs. Dans cette situation, le vertex doit travailler plus que les autres, ce qui ralentit toute la tâche. Qui plus est, la tâche peut échouer parce que les vertex sont, par exemple, limités à 5 heures d'exécution et à 6 Go de mémoire.

## <a name="resolving-data-skew-problems"></a>Résolution de problèmes liés à l'asymétrie des données

Azure Data Lake Tools pour Visual Studio peut aider à détecter un éventuel problème d'asymétrie des données dans votre tâche. En cas de problème, les solutions présentées dans cette section peuvent vous aider à les résoudre.

## <a name="solution-1-improve-table-partitioning"></a>Solution 1 : Améliorer le partitionnement des tables

### <a name="option-1-filter-the-skewed-key-value-in-advance"></a>Option 1 : Filtrer la valeur de clé décalée à l’avance

Si cela n’affecte pas votre logique métier, vous pouvez filtrer les valeurs plus fréquentes à l’avance. Par exemple, si la colonne GUID comprend un grand nombre de 000-000-000, vous n'agrégerez pas cette valeur. Avant l’agrégation, vous pouvez écrire “WHERE GUID != “000-000-000”” pour filtrer la valeur la plus fréquente.

### <a name="option-2-pick-a-different-partition-or-distribution-key"></a>Option n°2 : Sélectionner une clé de distribution ou de partition différente

Dans l’exemple précédent, si vous souhaitez uniquement vérifier la charge de travail des contrôles fiscaux partout dans le pays/la région, vous pouvez améliorer la distribution des données en sélectionnant le numéro d’ID comme clé. La sélection d'une clé de distribution ou partition différente permet parfois de distribuer les données de façon plus uniforme, mais vous devez vous assurer que cela n’affecte pas votre logique métier. Par exemple, pour calculer la somme des taxes pour chaque état, vous pouvez désigner _l'état_ comme clé de partition. Si vous continuez à rencontrer ce problème, essayez d’utiliser l’option 3.

### <a name="option-3-add-more-partition-or-distribution-keys"></a>Option 3 : Ajouter plus de clés de distribution ou de partition

Au lieu d’utiliser uniquement _l'état_ comme clé de partition, vous pouvez utiliser plusieurs clés pour le partitionnement. Par exemple, vous pouvez envisager le _code postal_ comme clé de partition supplémentaire pour réduire la taille des partitions de données et répartir les données de manière plus homogène.

### <a name="option-4-use-round-robin-distribution"></a>Option 4 : Utiliser la distribution par tourniquet (round robin)

Si vous ne trouvez pas de clé appropriée pour la partition et la distribution, vous pouvez essayer d’utiliser la distribution par tourniquet (round robin). La distribution par tourniquet (round robin) traite toutes les lignes de manière égale et les place au hasard dans des compartiments correspondants. Les données sont distribuées de façon égale, mais perdent les informations relatives à la localité, ce qui peut aussi réduire les performances de travail pour certaines opérations. De plus, si vous effectuez malgré tout une agrégation pour la clé décalée, le problème d'asymétrie des données persistera. Pour en savoir plus sur la distribution par tourniquet (round robin), consultez la section Distribution de table U-SQL dans [CREATE TABLE (SQL-U) : Création d’une table avec un schéma](/u-sql/ddl/tables/create/managed/create-table-u-sql-creating-a-table-with-schema#dis_sch).

## <a name="solution-2-improve-the-query-plan"></a>Solution 2 : Améliorer le plan de requête

### <a name="option-1-use-the-create-statistics-statement"></a>Option 1 : Utiliser l’instruction CREATE STATISTICS

U-SQL fournit l’instruction CREATE STATISTICS dans des tables. Cette instruction donne plus d’informations à l’optimiseur de requête sur les caractéristiques des données, notamment la distribution de la valeur, qui sont stockées dans une table. Pour la plupart des requêtes, l’optimiseur de requête génère déjà les statistiques nécessaires pour un plan de requête de haute qualité. Parfois, vous devrez peut-être améliorer les performances des requêtes en créant des statistiques supplémentaires avec CREATE STATISTICS ou en modifiant la conception des requêtes. Pour plus d’informations, consultez la page [CREATE STATISTICS (SQL-U)](/u-sql/ddl/statistics/create-statistics).

Exemple de code :

```usql
CREATE STATISTICS IF NOT EXISTS stats_SampleTable_date ON SampleDB.dbo.SampleTable(date) WITH FULLSCAN;
```

>[!NOTE]
>Les informations statistiques ne sont pas mises à jour automatiquement. Si vous mettez à jour les données dans une table sans recréer les statistiques, les performances des requêtes peuvent décliner.

### <a name="option-2-use-skewfactor"></a>Option n°2 : Utiliser SKEWFACTOR

Si vous souhaitez additionner les taxes pour chaque état, vous devez utiliser l'instruction GROUP BY, une approche qui n’évite pas le problème d'asymétrie des données. Toutefois, vous pouvez fournir un indicateur de données dans votre requête pour identifier l'asymétrie des données dans les clés, afin que l’optimiseur puisse préparer un plan d’exécution.

En règle générale, vous pouvez définir le paramètre sur 0,5 et 1, 0,5 indiquant une asymétrie peu importante et 1 une asymétrie conséquente. L’indicateur affectant l’optimisation du plan d’exécution de l’instruction en cours et de toutes les instructions en aval, il est important d'ajouter l’indicateur avant l’agrégation permettant d’identifier le décalage éventuel dans les clés.

```usql
SKEWFACTOR (columns) = x
```

Il indique que les colonnes données ont un facteur d’inclinaison x compris entre 0 (pas d’inclinaison) et 1 (très forte inclinaison).

Exemple de code :

```usql
//Add a SKEWFACTOR hint.
@Impressions =
    SELECT * FROM
    searchDM.SML.PageView(@start, @end) AS PageView
    OPTION(SKEWFACTOR(Query)=0.5)
    ;
//Query 1 for key: Query, ClientId
@Sessions =
    SELECT
        ClientId,
        Query,
        SUM(PageClicks) AS Clicks
    FROM
        @Impressions
    GROUP BY
        Query, ClientId
    ;
//Query 2 for Key: Query
@Display =
    SELECT * FROM @Sessions
        INNER JOIN @Campaigns
            ON @Sessions.Query == @Campaigns.Query
    ;
```

### <a name="option-3-use-rowcount"></a>Option 3 : Utiliser ROWCOUNT
Outre SKEWFACTOR, pour les cas de jonction de clés de décalage spécifiques, vous pouvez déterminer l’optimiseur en ajoutant un indicateur ROWCOUNT dans l’instruction U-SQL avant la jointure si vous savez que l’autre ensemble de lignes jointes est peu volumineux. De cette manière, l'optimiseur peut choisir une stratégie de diffusion conjointe pour améliorer les performances. N’oubliez pas que ROWCOUNT ne résout pas le problème d'asymétrie des données, mais peut proposer une aide supplémentaire.

```usql
OPTION(ROWCOUNT = n)
```

Identifiez un petit ensemble de lignes avant JOIN en fournissant une estimation du nombre de lignes (chiffre entier).

Exemple de code :

```usql
//Unstructured (24-hour daily log impressions)
@Huge   = EXTRACT ClientId int, ...
            FROM @"wasb://ads@wcentralus/2015/10/30/{*}.nif"
            ;
//Small subset (that is, ForgetMe opt out)
@Small  = SELECT * FROM @Huge
            WHERE Bing.ForgetMe(x,y,z)
            OPTION(ROWCOUNT=500)
            ;
//Result (not enough information to determine simple broadcast JOIN)
@Remove = SELECT * FROM Bing.Sessions
            INNER JOIN @Small ON Sessions.Client == @Small.Client
            ;
```

## <a name="solution-3-improve-the-user-defined-reducer-and-combiner"></a>Solution 3 : Améliorer le réducteur et le combinateur définis par l’utilisateur

Parfois, vous écrivez l’opérateur défini par l’utilisateur pour gérer une logique de processus complexe, et un combinateur et un réducteur bien écrits peuvent, dans certains cas, réduire les problèmes d'asymétrie des données.

### <a name="option-1-use-a-recursive-reducer-if-possible"></a>Option 1 : Utiliser un réducteur récursif si possible

Par défaut, le réducteur défini par l’utilisateur s’exécute en mode non récursif, ce qui signifie que le travail de réduction pour une clé est distribué dans un seul vertex. Mais, si vos données sont asymétriques, les ensembles de données volumineux peuvent être traités dans un seul vertex et cette exécution peut prendre un certain temps.

Pour améliorer les performances, vous pouvez ajouter un attribut dans votre code pour définir une exécution du réducteur en mode récursif. Ensuite, les grands ensembles de données peuvent être distribués sur plusieurs vertex et s’exécuter en parallèle, ce qui accélère votre travail.

Pour faire passer un réducteur en mode non récursif au mode récursif, vous devez vous assurer que votre algorithme est un algorithme associatif. Par exemple, la somme est associative, tandis qu’une valeur médiane ne l’est pas. Vous devez également vous assurer que l’entrée et la sortie du réducteur conservent le même schéma.

Attribut d'un réducteur récursif :

```usql
[SqlUserDefinedReducer(IsRecursive = true)]
```

Exemple de code :

```usql
[SqlUserDefinedReducer(IsRecursive = true)]
public class TopNReducer : IReducer
{
    public override IEnumerable<IRow>
        Reduce(IRowset input, IUpdatableRow output)
    {
        //Your reducer code goes here.
    }
}
```

### <a name="option-2-use-row-level-combiner-mode-if-possible"></a>Option n°2 : Utiliser le mode de combinateur au niveau des lignes si possible

À l’instar de l’indicateur ROWCOUNT pour les cas de jonction de clés de décalage spécifiques, le mode de combinateur essaie de distribuer les ensembles de valeurs de clés de décalage dans plusieurs vertex pour que le travail puisse être exécuté simultanément. Le mode de combinateur ne permet pas de résoudre le problème d'asymétrie des données, mais peut être utile pour traiter les ensembles de valeurs de clés de décalage volumineux.

Par défaut, le mode de combinateur est défini sur Full, ce qui signifie que l’ensemble de lignes de gauche et l’ensemble de lignes de droite ne peuvent pas être séparés. La définition du mode sur gauche/droite/interne permet une jointure au niveau des lignes. Le système sépare les ensembles de lignes correspondants et les répartit dans plusieurs vertex qui s’exécutent en parallèle. Toutefois, avant de configurer le mode de combinateur, assurez-vous que les ensembles de lignes correspondants peuvent être séparés.

L’exemple qui suit montre un ensemble de lignes gauche distinct. Chaque ligne de sortie dépend d’une seule ligne d’entrée de gauche, et potentiellement de toutes les lignes de droite avec la même valeur de clé. Si vous définissez le mode de combinateur sur gauche, le système sépare le grand ensemble de lignes de gauche en plusieurs petits ensembles et les distribue dans plusieurs vertex.

![Illustration du mode de combinateur](./media/data-lake-analytics-data-lake-tools-data-skew-solutions/combiner-mode-illustration.png)

>[!NOTE]
>Si vous définissez le mode de combinateur incorrect, la combinaison est moins efficace, et les résultats peuvent être erronés.

Attributs du mode de combinateur :

- SqlUserDefinedCombiner(Mode=CombinerMode.Full) : Chaque ligne de sortie dépend potentiellement de toutes les lignes d’entrée de gauche et de droite avec la même valeur de clé.

- SqlUserDefinedCombiner(Mode=CombinerMode.Left) : chaque ligne de sortie dépend d’une seule ligne d’entrée de gauche (et potentiellement de toutes les lignes de droite avec la même valeur de clé).

- qlUserDefinedCombiner(Mode=CombinerMode.Right) : chaque ligne de sortie dépend d’une seule ligne d’entrée de droite (et potentiellement de toutes les lignes de gauche avec la même valeur de clé).

- SqlUserDefinedCombiner(Mode=CombinerMode.Inner) : chaque ligne de sortie dépend d’une seule ligne d’entrée de gauche et droite avec la même valeur.

Exemple de code :

```usql
[SqlUserDefinedCombiner(Mode = CombinerMode.Right)]
public class WatsonDedupCombiner : ICombiner
{
    public override IEnumerable<IRow>
        Combine(IRowset left, IRowset right, IUpdatableRow output)
    {
    //Your combiner code goes here.
    }
}
```
