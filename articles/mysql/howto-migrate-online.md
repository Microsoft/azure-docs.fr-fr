---
title: Migration avec temps d’arrêt minimal - Azure Database pour MySQL
description: Cet article décrit comment effectuer une migration de base de données MySQL vers Azure Database pour MySQL avec un temps d’arrêt minimal à l’aide du service Azure Database Migration Service.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/30/2020
ms.openlocfilehash: 75b548b53de4c0f4ea39959c3d70e9ddb361d64c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540329"
---
# <a name="minimal-downtime-migration-to-azure-database-for-mysql"></a>Migration avec temps d’arrêt minimal vers Azure Database pour MySQL
[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Vous pouvez effectuer des migrations MySQL vers Azure Database pour MySQL avec un temps d’arrêt minimal à l’aide de la toute nouvelle **fonctionnalité de synchronisation continue** du service [Azure Database Migration Service](https://aka.ms/get-dms) (DMS). Cette fonctionnalité limite les temps d’arrêt affectant l’application.

Vous pouvez également consulter le [guide de la migration de base de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide) pour obtenir des informations détaillées et des cas d’utilisation sur la migration de base de données vers Azure Database pour MySQL. Ce guide fournit des conseils pour réussir la planification et l’exécution d’une migration MySQL vers Azure.

## <a name="overview"></a>Vue d’ensemble
Azure DMS procède à une charge initiale de votre instance locale vers Azure Database pour MySQL, puis synchronise en continu les nouvelles transactions vers Azure tandis que l’application s’exécute. Une fois les données à jour sur le côté Azure cible, vous arrêtez momentanément l’application (temps d’arrêt minimal), vous attendez que le dernier lot de données apparaisse côté cible (à partir du moment où vous arrêtez l’application jusqu’à ce qu’elle ne soit plus disponible pour accepter du trafic), puis vous mettez à jour votre chaîne de connexion pour qu’elle pointe vers Azure. Lorsque vous avez terminé, votre application sera disponible sur Azure !

:::image type="content" source="./media/howto-migrate-online/ContinuousSync.png" alt-text="Synchronisation continue avec le service Azure Database Migration Service":::

## <a name="next-steps"></a>Étapes suivantes
- Pour plus d’informations sur la migration de bases de données vers Azure Database pour MySQL, consultez le [Guide de migration des bases de données](https://github.com/Azure/azure-mysql/tree/master/MigrationGuide).
- Regardez la vidéo [Easily migrate MySQL/PostgreSQL apps to Azure managed service](https://medius.studios.ms/Embed/Video/THR2201?sid=THR2201) (Migration facile des applications MySQL/PostgreSQL vers un service géré Azure) qui comprend une démonstration sur la migration des applications MySQL vers Azure Database pour MySQL.
- Consultez le tutoriel [Migration de MySQL vers Azure Database pour MySQL en ligne à l’aide de DMS](../dms/tutorial-mysql-azure-mysql-online.md).