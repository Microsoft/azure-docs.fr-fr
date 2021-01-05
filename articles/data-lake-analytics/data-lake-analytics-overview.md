---
title: Vue d’ensemble d’Azure Data Lake Analytics
description: Data Lake Analytics permet de gérer votre entreprise grâce aux informations obtenues dans vos données cloud, à n’importe quelle échelle.
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.service: data-lake-analytics
ms.topic: overview
ms.date: 06/23/2017
ms.openlocfilehash: 4a6ef2821080982e2b34108703bd80574443244a
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445531"
---
# <a name="what-is-azure-data-lake-analytics"></a>Définition d'Azure Data Lake Analytics

Azure Data Lake Analytics est un service de travaux d’analyse à la demande, qui simplifie les Big Data. Au lieu de déployer, de configurer et d’optimiser le matériel, vous écrivez des requêtes pour transformer vos données et en extraire des informations pertinentes. Le service d’analyse peut traiter les travaux instantanément, quelle qu’en soit l’échelle, en définissant le compteur sur la puissance requise. Vous payez les travaux uniquement lorsque ceux-ci sont exécutés, ce qui rend le service économique. 

## <a name="azure-data-lake-analytics-recent-update-information"></a>Informations sur les mises à jour récentes d’Azure Data Lake Analytics

Le service Azure Data Lake Analytics est mis à jour de façon non périodique pour différentes raisons. Nous continuons à fournir la prise en charge de ce service avec la mise à jour des composants, les préversions bêta des composants, etc. 

- Pour obtenir des informations générales sur les mises à jour récentes, reportez-vous à [Nouveautés de Data Lake Analytics](data-lake-analytics-whats-new.md).
- Pour obtenir des détails sur chaque mise à jour, reportez-vous à la [Note de publication d’Azure Data Lake Analytics](https://github.com/Azure/AzureDataLake/tree/master/docs/Release_Notes).

## <a name="dynamic-scaling"></a>Mise à l’échelle dynamique
  
Data Lake Analytics approvisionne dynamiquement les ressources et vous permet d’analyser plusieurs téraoctets voire pétaoctets de données. Vous ne payez que la puissance de traitement que vous utilisez. Il n’est pas nécessaire de réécrire le code à mesure que vous augmentez ou diminuez la taille des données stockées ou le volume des ressources de calcul utilisé. 

## <a name="develop-faster-debug-and-optimize-smarter-using-familiar-tools"></a>Développez plus rapidement, déboguez et optimisez de manière plus intelligente à l’aide d’outils familiers
  
Data Lake Analytics s’intègre en profondeur avec Visual Studio. Vous pouvez utiliser des outils que vous connaissez pour exécuter, déboguer et ajuster votre code. Les visualisations de vos travaux U-SQL vous permettent de voir la façon dont votre code est exécuté à l'échelle. Vous pouvez ainsi facilement identifier les goulots d'étranglement en matière de performances et optimiser les coûts.

## <a name="u-sql-simple-and-familiar-powerful-and-extensible"></a>U-SQL : simple et familier, puissant, et extensible
  
Data Lake Analytics inclut U-SQL, un langage de requête qui étend la nature connue, simple, déclarative de SQL avec la puissance d'expression de C#. Le langage U-SQL utilise le même runtime distribué qui alimente le service Microsoft interne Data Lake à l’échelle d’exaoctet. Les développeurs SQL et .NET peuvent maintenant traiter et analyser leurs données avec des compétences qu’ils possèdent déjà.

## <a name="integrates-seamlessly-with-your-it-investments"></a>S’intègre de façon transparente avec vos investissements informatiques
  
Data Lake Analytics utilise vos investissements informatiques existants pour l’identité, la gestion et la sécurité. Cette approche simplifie la gouvernance des données et facilite l’extension de vos applications de données actuelles. Data Lake Analytics est intégré à Active Directory pour les autorisations et la gestion des utilisateurs. Il est fourni avec des capacités de surveillance et d’audit intégrées.

## <a name="affordable-and-cost-effective"></a>Abordable et économique

Data Lake Analytics est une solution économique permettant d'exécuter vos charges de travail Big Data. Vous payez pour chaque travail au cours duquel des données sont traitées. Aucun matériel, aucune licence ni aucun contrat de support technique propre au service ne sont requis. Le système est mis à l’échelle automatiquement au démarrage du travail puis à la fin de son exécution de sorte que vous ne payez que ce dont vous avez réellement besoin. [En savoir plus sur le contrôle des coûts et les économies](https://aka.ms/adlasavemoney).

## <a name="works-with-all-your-azure-data"></a>Fonctionne avec toutes vos données Azure
  
Data Lake Analytics fonctionne avec Azure Data Lake Storage pour fournir le niveau de performance, le débit et la parallélisation les plus élevés possibles. Il fonctionne avec les blobs Stockage Azure, Azure SQL Database et Azure Synapse Analytics.


## <a name="next-steps"></a>Étapes suivantes

* Découvrez les mises à jour récentes d’Azure Data Lake Analytics en utilisant [Nouveautés d’Azure Data Lake Analytics](data-lake-analytics-whats-new.md)
* Prise en main de Data Lake Analytics à l’aide du [portail Azure](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli.md)
* Gérer Data Lake Analytics à l’aide du [portail Azure](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) | [Kit de développement logiciel (SDK) Azure .NET](data-lake-analytics-manage-use-dotnet-sdk.md) | [Node.js](data-lake-analytics-manage-use-nodejs.md)
* [How to control costs and save money with Data Lake Analytics](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c) (Comment contrôler les coûts et réaliser des économies avec Data Lake Analytics)
