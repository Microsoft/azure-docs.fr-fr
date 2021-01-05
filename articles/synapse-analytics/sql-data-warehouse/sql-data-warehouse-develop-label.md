---
title: Utiliser des étiquettes pour instrumenter des requêtes
description: Conseils relatifs à l’utilisation d’étiquettes pour instrumenter les requêtes des pools SQL dédiés dans Azure Synapse Analytics.
services: synapse-analytics
author: MSTehrani
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: emtehran
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 7958caf71658dcdcbf31bac84697931e9049452f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462745"
---
# <a name="using-labels-to-instrument-queries-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Utilisation d’étiquettes pour instrumenter les requêtes des pools SQL dédiés dans Azure Synapse Analytics

Cet article inclut des conseils pour développer des solutions utilisant des étiquettes afin d’instrumenter les requêtes d’un pool SQL dédié.

## <a name="what-are-labels"></a>Que sont les étiquettes ?

Le pool SQL dédié prend en charge le concept d’étiquette de requête. Avant de poursuivre l’étude, examinons un exemple :

```sql
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query Label')
;
```

La dernière ligne balise la chaîne « Mon étiquette de requête » dans la requête. Cette balise est utile car l’étiquette peut être interrogée via des DMV.

L’interrogation d’étiquettes fournit un mécanisme de localisation des requêtes à problème qui permet d’identifier la progression via une ELT exécutée.

Une convention d’affectation de noms efficace s’avère très utile. Par exemple, démarrer l’étiquette par PROJET, PROCÉDURE, INSTRUCTION ou COMMENTAIRE identifie une requête de manière unique dans l’ensemble du code lors du contrôle de la source.

La requête suivante utilise une vue de gestion dynamique pour rechercher par étiquette :

```sql
SELECT  *
FROM    sys.dm_pdw_exec_requests r
WHERE   r.[label] = 'My Query Label'
;
```

> [!NOTE]
> Vous devez impérativement placer l’étiquette du mot entre crochets ou guillemets doubles lors de l’interrogation. Étiquette est un mot réservé et provoque une erreur lorsqu’il n’est pas délimité.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des conseils supplémentaires, consultez la [vue d’ensemble du développement](sql-data-warehouse-overview-develop.md).
