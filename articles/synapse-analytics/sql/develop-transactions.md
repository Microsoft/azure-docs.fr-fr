---
title: Utiliser des transactions
description: Conseils d’implémentation de transactions avec un pool SQL dédié dans Azure Synapse Analytics pour le développement de solutions.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: igorstan
ms.openlocfilehash: c4fe512ff6db24498148ffa724c3144a2f61823f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96451710"
---
# <a name="use-transactions-with-dedicated-sql-pool-in-azure-synapse-analytics"></a>Utiliser des transactions avec un pool SQL dédié dans Azure Synapse Analytics

Conseils d’implémentation de transactions avec un pool SQL dédié dans Azure Synapse Analytics pour le développement de solutions.

## <a name="what-to-expect"></a>À quoi s’attendre

En toute logique, un pool SQL dédié prend en charge les transactions dans le cadre de la charge de travail de l’entrepôt de données. Toutefois, pour garantir le maintien des performances du pool SQL dédié à grande échelle, certaines fonctionnalités sont limitées par rapport à SQL Server. Cet article identifie les différences et répertorie les autres éléments disponibles.

## <a name="transaction-isolation-levels"></a>Niveaux d’isolation des transactions

Le pool SQL dédié implémente les transactions ACID. Par défaut, le niveau d'isolation de la prise en charge transactionnelle est READ UNCOMMITTED.  Vous pouvez le remplacer par READ COMMITTED SNAPSHOT ISOLATION en activant l'option de base de données READ_COMMITTED_SNAPSHOT pour une base de données utilisateur lorsqu'elle est connectée à la base de données MASTER.  

Une fois activée, toutes les transactions de cette base de données sont exécutées sous READ COMMITTED SNAPSHOT ISOLATION et la définition de READ UNCOMMITTED au niveau de la session n'est pas honorée. Pour plus d'informations, consultez [Options ALTER DATABASE SET (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options?view=azure-sqldw-latest&preserve-view=true).

## <a name="transaction-size"></a>Taille de la transaction
Une transaction de modification de données unique est limitée en taille. La limite est appliquée par distribution. Par conséquent, l’allocation totale peut être calculée en multipliant la limite par le nombre de distributions. 

Pour évaluer approximativement le nombre maximal de lignes dans la transaction, divisez la limite de la distribution par la taille totale de chaque ligne. Pour les colonnes de longueur variable, pensez à prendre une longueur de colonne moyenne au lieu d’utiliser la taille maximale.

Dans le tableau ci-dessous, les hypothèses suivantes ont été formulées :

* Une distribution égale des données s’est produite
* La longueur de ligne moyenne est de 250 octets

## <a name="gen2"></a>Deuxième génération

| [DWU](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) | Limite par distribution (Go) | Nombre de distributions | Taille de transaction MAX (Go) | Nombre de lignes par distribution | Nombre de lignes max par transaction |
| --- | --- | --- | --- | --- | --- |
| DW100c |1 |60 |60 |4 000 000 |240 000 000 |
| DW200c |1.5 |60 |90 |6 000 000 |360 000 000 |
| DW300c |2.25 |60 |135 |9 000 000 |540 000 000 |
| DW400c |3 |60 |180 |12 000 000 |720 000 000 |
| DW500c |3,75 |60 |225 |15 000 000 |900 000 000 |
| DW1000c |7.5 |60 |450 |30 000 000 |1 800 000 000 |
| DW1500c |11,25 |60 |675 |45 000 000 |2 700 000 000 |
| DW2000c |15 |60 |900 |60 000 000 |3 600 000 000 |
| DW2500c |18,75 |60 |1 125 |75 000 000 |4 500 000 000 |
| DW3000c |22,5 |60 |1 350 |90 000 000 |5 400 000 000 |
| DW5000c |37.5 |60 |2 250 |150 000 000 |9 000 000 000 |
| DW6000c |45 |60 |2 700 |180 000 000 |10 800 000 000 |
| DW7500c |56,25 |60 |3 375 |225 000 000 |13 500 000 000 |
| DW10000c |75 |60 |4 500 |300 000 000 |18 000 000 000 |
| DW15000c |112,5 |60 |6 750 |450 000 000 |27 000 000 000 |
| DW30000c |225 |60 |13 500 |900 000 000 |54 000 000 000 |

## <a name="gen1"></a>Première génération

| [DWU](../sql-data-warehouse/sql-data-warehouse-overview-what-is.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) | Limite par distribution (Go) | Nombre de distributions | Taille de transaction MAX (Go) | Nombre de lignes par distribution | Nombre de lignes max par transaction |
| --- | --- | --- | --- | --- | --- |
| DW100 |1 |60 |60 |4 000 000 |240 000 000 |
| DW200 |1.5 |60 |90 |6 000 000 |360 000 000 |
| DW300 |2.25 |60 |135 |9 000 000 |540 000 000 |
| DW400 |3 |60 |180 |12 000 000 |720 000 000 |
| DW500 |3,75 |60 |225 |15 000 000 |900 000 000 |
| DW600 |4.5 |60 |270 |18 000 000 |1 080 000 000 |
| DW1000 |7.5 |60 |450 |30 000 000 |1 800 000 000 |
| DW1200 |9 |60 |540 |36 000 000 |2 160 000 000 |
| DW1500 |11,25 |60 |675 |45 000 000 |2 700 000 000 |
| DW2000 |15 |60 |900 |60 000 000 |3 600 000 000 |
| DW3000 |22,5 |60 |1 350 |90 000 000 |5 400 000 000 |
| DW6000 |45 |60 |2 700 |180 000 000 |10 800 000 000 |

La limite de taille de transaction est appliquée par transaction ou opération. Elle n’est pas appliquée à toutes les transactions simultanées. Par conséquent, chaque transaction est autorisée à écrire cette quantité de données dans le journal.

Pour optimiser et réduire la quantité de données écrites dans le journal, consultez l’article [Bonnes pratiques relatives aux transactions](../sql-data-warehouse/sql-data-warehouse-develop-best-practices-transactions.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

> [!WARNING]
> La taille de transaction maximale ne peut être obtenue que pour les tables distribuées HASH ou ROUND_ROBIN où la répartition des données est égale. Si la transaction écrit les données de manière asymétrique dans les distributions, alors la limite est susceptible d’être atteinte avant la taille de transaction maximale.
> <!--REPLICATED_TABLE-->

## <a name="transaction-state"></a>État des transactions

Le pool SQL dédié utilise la fonction XACT_STATE() pour signaler une transaction non réussie, avec la valeur −2. Cette valeur signifie que la transaction a échoué et est marquée pour une restauration uniquement.

> [!NOTE]
> L’association de la valeur -2 à la fonction XACT_STATE afin de signaler l’échec d’une transaction constitue un comportement différent par rapport à SQL Server. SQL Server utilise la valeur -1 pour représenter une transaction non validable. De plus, il peut tolérer la présence de certaines erreurs au sein d’une transaction sans pour autant signaler que cette dernière ne peut pas être validée. Par exemple, la valeur `SELECT 1/0` entraîne une erreur, mais ne fait pas passer une transaction à l’état non validable. Par ailleurs, SQL Server autorise également les lectures dans une transaction non validable. Toutefois, le pool SQL dédié ne vous permet pas d’effectuer cette opération. Si une erreur se produit dans une transaction de pool SQL dédié, celle-ci passe automatiquement à l’état -2 et il n’est pas possible d’utiliser d’autres instructions SELECT tant que l’instruction n’a pas été restaurée. Vous devez donc impérativement vérifier le code de votre application afin de vous assurer qu’il utilise la fonction XACT_STATE(), car des modifications du code peuvent être nécessaires.

Dans SQL Server par exemple, vous pouvez voir une transaction ressemblant à celle-ci :

```sql
SET NOCOUNT ON;
DECLARE @xact_state smallint = 0;

BEGIN TRAN
    BEGIN TRY
        DECLARE @i INT;
        SET     @i = CONVERT(INT,'ABC');
    END TRY
    BEGIN CATCH
        SET @xact_state = XACT_STATE();

        SELECT  ERROR_NUMBER()    AS ErrNumber
        ,       ERROR_SEVERITY()  AS ErrSeverity
        ,       ERROR_STATE()     AS ErrState
        ,       ERROR_PROCEDURE() AS ErrProcedure
        ,       ERROR_MESSAGE()   AS ErrMessage
        ;

        IF @@TRANCOUNT > 0
        BEGIN
            ROLLBACK TRAN;
            PRINT 'ROLLBACK';
        END

    END CATCH;

IF @@TRANCOUNT >0
BEGIN
    PRINT 'COMMIT';
    COMMIT TRAN;
END

SELECT @xact_state AS TransactionState;
```

Le code précédent génère le message d’erreur suivant :

Msg 111233, Level 16, State 1, Line 1 111233; La transaction active a été abandonnée. Les modifications en attente ont été restaurées. Cause : une transaction à un état de restauration uniquement n’a pas été explicitement restaurée avant une instruction DDL, DML ou SELECT.

Vous n’obtiendrez pas la sortie des fonctions ERROR_* non plus.

Dans le pool SQL dédié, le code doit être légèrement modifié :

```sql
SET NOCOUNT ON;
DECLARE @xact_state smallint = 0;

BEGIN TRAN
    BEGIN TRY
        DECLARE @i INT;
        SET     @i = CONVERT(INT,'ABC');
    END TRY
    BEGIN CATCH
        SET @xact_state = XACT_STATE();

        IF @@TRANCOUNT > 0
        BEGIN
            ROLLBACK TRAN;
            PRINT 'ROLLBACK';
        END

        SELECT  ERROR_NUMBER()    AS ErrNumber
        ,       ERROR_SEVERITY()  AS ErrSeverity
        ,       ERROR_STATE()     AS ErrState
        ,       ERROR_PROCEDURE() AS ErrProcedure
        ,       ERROR_MESSAGE()   AS ErrMessage
        ;
    END CATCH;

IF @@TRANCOUNT >0
BEGIN
    PRINT 'COMMIT';
    COMMIT TRAN;
END

SELECT @xact_state AS TransactionState;
```

Le comportement attendu est maintenant examiné. L’erreur dans la transaction est gérée, et les fonctions ERROR_* fournissent les valeurs attendues.

La seule chose qui a changé est que l’opération ROLLBACK de la transaction doit se produire avant la lecture des informations sur l’erreur, dans le bloc CATCH.

## <a name="error_line-function"></a>Fonction Error_Line()

Il est également important de signaler que le pool SQL dédié n’implémente ni ne prend en charge la fonction ERROR_LINE(). Si cette fonction figure dans votre code, vous devez la supprimer pour respecter les exigences du pool SQL dédié. Placez plutôt des libellés de requête dans votre code pour implémenter les fonctionnalités équivalentes. Pour plus d’informations, consultez l’article [LABEL](develop-label.md) :

## <a name="use-of-throw-and-raiserror"></a>Utilisation des paramètres THROW et RAISERROR

THROW est l’implémentation plus moderne du déclenchement d’exceptions dans le pool SQL dédié, mais RAISERROR est également pris en charge. Il existe cependant quelques différences, qu’il est préférable de prendre en compte.

* Les numéros associés aux messages d’erreur définis par l’utilisateur ne peuvent pas se trouver dans la plage de valeurs allant de 100 000 à 150 000 dans le cas du paramètre THROW.
* Les messages d’erreurs associés au paramètre RAISERROR sont définis sur la valeur fixe de 50 000.
* L’utilisation de l’élément sys.messages n’est pas prise en charge.

## <a name="limitations"></a>Limites

En ce qui concerne les transactions, le pool SQL dédié présente quelques restrictions supplémentaires. Les voici :

* Les transactions distribuées ne sont pas acceptées.
* Les transactions imbriquées ne sont pas autorisées.
* Les points de sauvegarde ne sont pas acceptés.
* Transactions sans nom
* Transactions sans marquage
* Aucune prise en charge de DDL comme CREATE TABLE dans une transaction définie par l’utilisateur

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’optimisation des transactions, consultez [Bonnes pratiques relatives aux transactions](../sql-data-warehouse/sql-data-warehouse-develop-best-practices-transactions.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json). Des guides supplémentaires sur les meilleures pratiques sont également fournis pour le [Pool SQL dédié](best-practices-sql-pool.md) et le [Pool SQL serverless](best-practices-sql-on-demand.md).
