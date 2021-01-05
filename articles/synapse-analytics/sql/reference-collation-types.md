---
title: Prise en charge du classement
description: Prise en charge des types de classements pour Synapse SQL dans Azure Synapse Analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: reference
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 436dbac814197556385a33d956928f97fd4716bf
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93311919"
---
# <a name="database-collation-support-for-synapse-sql-in-azure-synapse-analytics"></a>Prise en charge du classement de bases de données pour Synapse SQL dans Azure Synapse Analytics 

Les classements fournissent les règles de paramètres régionaux, de page de codes, d’ordre de tri et de respect des caractères pour les types de données basés sur des caractères. Une fois que vous avez effectué votre sélection, toutes les colonnes et expressions nécessitant des informations de classement héritent du classement choisi dans le paramètre de base de données. L’héritage par défaut peut être substitué en déclarant explicitement un classement différent pour un type de données basé sur des caractères.

Sur le Portail Azure, il est possible de modifier le classement par défaut d’une base de données de pool SQL dédié lors de sa création. Grâce à cette fonctionnalité, il est encore plus facile de créer une base de données à l’aide de l’un des 3800 classements de base de données pris en charge.

Vous pouvez spécifier le classement par défaut de la base de données de pools SQL serverless au moment de sa création à l’aide de l’instruction CREATE DATABASE.

## <a name="change-collation"></a>Modifier le classement
Pour changer le classement par défaut de la base de données de pools SQL dédiés, vous devez mettre à jour le champ Classement dans l’expérience d’approvisionnement. Par exemple, si vous souhaitez modifier le classement par défaut en respectant la casse, vous devez renommer le classement SQL_Latin1_General_CP1_CI_AS en SQL_Latin1_General_CP1_CS_AS. 

Pour modifier le classement par défaut de la base de données de pools SQL serverless, vous pouvez utiliser l’instruction ALTER DATABASE.

## <a name="list-of-unsupported-collation-types"></a>Liste des types de classement pris en charge
*    Japanese_Bushu_Kakusu_140_BIN
*    Japanese_Bushu_Kakusu_140_BIN2
*    Japanese_Bushu_Kakusu_140_CI_AI_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_VSS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS_VSS
*    Japanese_Bushu_Kakusu_140_CI_AI
*    Japanese_Bushu_Kakusu_140_CI_AI_WS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS
*    Japanese_Bushu_Kakusu_140_CI_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS
*    Japanese_Bushu_Kakusu_140_CI_AS_WS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS
*    Japanese_Bushu_Kakusu_140_CI_AS_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AI
*    Japanese_Bushu_Kakusu_140_CS_AI_WS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS
*    Japanese_Bushu_Kakusu_140_CS_AI_KS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS
*    Japanese_Bushu_Kakusu_140_CS_AS_WS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS
*    Japanese_Bushu_Kakusu_140_CS_AS_KS_WS
*    Japanese_XJIS_140_BIN
*    Japanese_XJIS_140_BIN2
*    Japanese_XJIS_140_CI_AI_VSS
*    Japanese_XJIS_140_CI_AI_WS_VSS
*    Japanese_XJIS_140_CI_AI_KS_VSS
*    Japanese_XJIS_140_CI_AI_KS_WS_VSS
*    Japanese_XJIS_140_CI_AS_VSS
*    Japanese_XJIS_140_CI_AS_WS_VSS
*    Japanese_XJIS_140_CI_AS_KS_VSS
*    Japanese_XJIS_140_CI_AS_KS_WS_VSS
*    Japanese_XJIS_140_CS_AI_VSS
*    Japanese_XJIS_140_CS_AI_WS_VSS
*    Japanese_XJIS_140_CS_AI_KS_VSS
*    Japanese_XJIS_140_CS_AI_KS_WS_VSS
*    Japanese_XJIS_140_CS_AS_VSS
*    Japanese_XJIS_140_CS_AS_WS_VSS
*    Japanese_XJIS_140_CS_AS_KS_VSS
*    Japanese_XJIS_140_CS_AS_KS_WS_VSS
*    Japanese_XJIS_140_CI_AI
*    Japanese_XJIS_140_CI_AI_WS
*    Japanese_XJIS_140_CI_AI_KS
*    Japanese_XJIS_140_CI_AI_KS_WS
*    Japanese_XJIS_140_CI_AS
*    Japanese_XJIS_140_CI_AS_WS
*    Japanese_XJIS_140_CI_AS_KS
*    Japanese_XJIS_140_CI_AS_KS_WS
*    Japanese_XJIS_140_CS_AI
*    Japanese_XJIS_140_CS_AI_WS
*    Japanese_XJIS_140_CS_AI_KS
*    Japanese_XJIS_140_CS_AI_KS_WS
*    Japanese_XJIS_140_CS_AS
*    Japanese_XJIS_140_CS_AS_WS
*    Japanese_XJIS_140_CS_AS_KS
*    Japanese_XJIS_140_CS_AS_KS_WS

De plus, le pool SQL dédié ne prend pas en charge les types de classements suivants :

*    SQL_EBCDIC1141_CP1_CS_AS
*    SQL_EBCDIC277_2_CP1_CS_AS
*    UTF-8

## <a name="check-the-current-collation"></a>Vérifier le classement actuel
Pour vérifier le classement actuel de la base de données, vous pouvez exécuter l’extrait de code T-SQL suivant :
```sql
SELECT DATABASEPROPERTYEX(DB_NAME(), 'Collation') AS Collation;
```
Quand « Classement » est transmis en tant que paramètre de propriété, la fonction DatabasePropertyEx retourne le classement actuel de la base de données spécifiée. Vous pouvez en savoir plus sur la fonction DatabasePropertyEx sur MSDN.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les bonnes pratiques concernant les pools SQL dédiés et les pools SQL serverless, consultez les articles suivants :

- [Bonnes pratiques pour les pools SQL dédiés](best-practices-sql-pool.md)
- [Bonnes pratiques pour les pools SQL serverless](best-practices-sql-on-demand.md)


