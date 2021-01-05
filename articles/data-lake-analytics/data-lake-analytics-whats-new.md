---
title: Modifications récentes de Data Lake Analytics
description: Cet article fournit la liste actualisée des modifications récentes apportées à Data Lake Analytics.
author: xujiang1
ms.service: data-lake-analytics
ms.topic: overview
ms.author: xujiang1
ms.date: 07/31/2020
ms.openlocfilehash: 6f4885594e4f67d91c595d8afdddc15e8ed4ae75
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88263178"
---
# <a name="whats-new-in-data-lake-analytics"></a>Nouveautés de Data Lake Analytics

Azure Data Lake Analytics est mis à jour de façon irrégulière pour certains composants. Pour vous tenir informé des mises à jour les plus récentes, cet article vous fournit des informations sur :

- La notification des préversions bêta des composants clés
- Des informations importantes sur les versions des composants, par exemple : la liste des versions disponibles des composants, leur version actuelle par défaut, etc.


## <a name="notification-of-key-component-beta-preview"></a>Notification des préversions bêta des composants clés

Aucune version bêta d’un composant clé n’est disponible en préversion. 

## <a name="u-sql-runtime"></a>Runtime U-SQL

Le runtime U-SQL Azure Data Lake, y compris le compilateur, l’optimiseur et le gestionnaire de travaux, est ce qui traite votre code U-SQL.

Quand vous soumettez le travail Azure Data Lake Analytics depuis n’importe quel outil, votre travail utilise le runtime par défaut actuellement disponible dans l’environnement de production. 

La version du runtime sera mise à jour de façon non périodique. Le runtime précédent restera disponible pendant un certain temps. Dès qu’une nouvelle version bêta sera prête pour la préversion, elle sera également disponible ici.

> [!CAUTION]
> - Le choix d’un runtime différent de la valeur par défaut est susceptible d’interrompre vos travaux U-SQL. Il est fortement recommandé de ne pas utiliser les versions autres que celles par défaut pour la production, mais à des fins de test uniquement.
> - La version du common language runtime, qui n’est pas définie par défaut, a un cycle de vie fixe. Elle expirera automatiquement.

La version suivante est la version actuelle du common language runtime par défaut.

- release-20200124live_adl_16283022_2

Pour savoir comment résoudre les échecs du runtime U-SQL, reportez-vous à [Résoudre les échecs du runtime U-SQL](runtime-troubleshoot.md).

## <a name="net-framework"></a>.NET Framework

Azure Data Lake Analytics utilise actuellement le **.NET Framework v4.7.2**. 

Si le code de votre code U-SQL Azure Data Lake Analytics utilise des assemblys personnalisés et que ceux-ci utilisent des bibliothèques .NET, vérifiez si votre code continue de fonctionner correctement.

Pour savoir comment résoudre les problèmes liés à une mise à niveau de .NET, utilisez [Résoudre les problèmes d’une mise à niveau .NET](runtime-troubleshoot.md).

## <a name="release-note"></a>Note de publication

Pour obtenir des informations récentes sur les mises à jour, reportez-vous à la [Note de publication d’Azure Data Lake Analytics](https://github.com/Azure/AzureDataLake/tree/master/docs/Release_Notes).


## <a name="next-steps"></a>Étapes suivantes

* Prise en main de Data Lake Analytics à l’aide du [portail Azure](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli.md)

