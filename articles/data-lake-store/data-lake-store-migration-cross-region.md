---
title: Migration entre les régions Azure Data Lake Storage Gen1 | Microsoft Docs
description: Découvrez les éléments à prendre en compte lors de la planification et de l’exécution d’une migration vers Azure Data Lake Storage Gen1 dès qu’elle est disponible dans les nouvelles régions.
services: data-lake-store
documentationcenter: ''
author: twooley
manager: amitkul
editor: swums
ms.assetid: ebde7b9f-2e51-4d43-b7ab-566417221335
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 01/27/2017
ms.author: twooley
ms.openlocfilehash: c6520036f3ddb8799025129391330268b0604886
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723804"
---
# <a name="migrate-azure-data-lake-storage-gen1-across-regions"></a>Migrer Azure Data Lake Storage Gen1 entre les régions

La disponibilité d’Azure Data Lake Storage Gen1 s’étendant à davantage de régions, vous pouvez décider d’effectuer une migration à usage unique pour tirer parti d’une nouvelle région. Découvrez ce que vous devez prendre en compte lorsque vous planifiez et effectuez la migration.

## <a name="prerequisites"></a>Conditions préalables requises

* **Un abonnement Azure**. Pour plus d’informations, consultez [Créer votre compte Azure gratuit](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Data Lake Storage Gen1 dans deux régions différentes**. Pour plus d’informations, consultez [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Azure Data Factory**. Pour plus d’informations, consultez [Présentation d’Azure Data Factory](../data-factory/introduction.md).


## <a name="migration-considerations"></a>Considérations relatives à la migration

Tout d’abord, identifiez la stratégie de migration appropriée pour votre application qui écrit, lit ou traite des données dans Data Lake Storage Gen1. Au moment de choisir une stratégie, tenez compte des exigences de disponibilité de votre application et du temps d’arrêt qui se produit pendant une migration. Par exemple, votre approche la plus simple peut être d’utiliser le modèle de migration dans le cloud de type « lift-and-shift ». Dans cette approche, vous suspendez l’application dans votre région existante et toutes vos données sont copiées dans la nouvelle région. Une fois le processus de copie terminé, vous reprenez votre application dans la nouvelle région, puis supprimez l’ancien compte Data Lake Storage Gen1. La migration nécessite un temps d’arrêt.

Pour réduire ce temps d’arrêt, vous pouvez immédiatement commencer à ingérer les nouvelles données dans la nouvelle région. Une fois que vous avez les données minimales requises, exécutez votre application dans la nouvelle région. En arrière-plan, continuez à copier les données plus anciennes du compte Data Lake Storage Gen1 existant vers le nouveau compte Data Lake Storage Gen1 dans la nouvelle région. Cette approche vous permet de passer à la nouvelle région avec un temps d’arrêt minimal. Une fois que toutes les anciennes données ont été copiées, supprimez l’ancien compte Data Lake Storage Gen1.

D’autres informations importantes à prendre en compte lorsque vous planifiez votre migration sont :

* **Volume de données**. Le volume de données (nombre de gigaoctets, nombre de fichiers et dossiers, etc.) a un impact sur le temps et les ressources nécessaires à la migration.

* **Nom du compte Data Lake Storage Gen1**. Le nouveau nom de compte dans la nouvelle région doit être globalement unique. Par exemple, le nom de votre ancien compte Data Lake Storage Gen1 dans la région USA Est 2 peut être contosoeastus2.azuredatalakestore.net. Vous pouvez nommer votre nouveau compte Data Lake Storage Gen1 dans la région Europe Nord contosonortheu.azuredatalakestore.net.

* **Outils**. Nous vous recommandons d’utiliser l’[activité de copie Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) pour copier les fichiers Data Lake Storage Gen1. Data Factory prend en charge le déplacement de données avec une fiabilité optimale et de hautes performances. N’oubliez pas que Data Factory copie uniquement l’arborescence de dossiers et le contenu des fichiers. Vous devez appliquer manuellement les listes de contrôle d’accès que vous avez utilisées dans l’ancien compte vers le nouveau compte. Pour plus d’informations, y compris sur les objectifs de performance pour les scénarios optimistes, consultez le [Guide sur les performances et le réglage de l’activité de copie](../data-factory/copy-activity-performance.md). Si vous souhaitez copier les données plus rapidement, vous devrez peut-être utiliser des unités de déplacement des données dans le cloud supplémentaires. D’autres outils, tels que AdlCopy, ne permettent pas de copier de données entre différentes régions.  

* **Frais liés à la bande passante**. Des [frais liés à la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/) s’appliquent, car les données sont transférées en dehors d’une région Azure.

* **Listes de contrôle d’accès sur vos données**. Sécurisez vos données dans la nouvelle région en appliquant des listes de contrôle d’accès aux fichiers et dossiers. Pour plus d’informations, consultez [Sécurisation des données stockées dans Azure Data Lake Storage Gen1](data-lake-store-secure-data.md). Nous vous recommandons de profiter de la migration pour mettre à jour et ajuster vos listes de contrôle d’accès. Vous pouvez utiliser des paramètres similaires à vos paramètres actuels. Vous pouvez afficher les listes de contrôle d’accès dans n’importe quel fichier à l’aide du portail Azure, des [applets de commande PowerShell](/powershell/module/az.datalakestore/get-azdatalakestoreitempermission) ou des Kits de développement logiciel (SDK).  

* **Emplacement des services d’analyse**. Pour des performances optimales, vos services d’analyse, tels qu’Azure Data Lake Analytics ou Azure HDInsight, doivent se situer dans la même région que vos données.  

## <a name="next-steps"></a>Étapes suivantes
* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
