---
title: Migration avec temps d’arrêt minimal vers Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit comment effectuer une migration de base de données PostgreSQL vers Azure Database pour PostgreSQL - Serveur unique avec un temps d’arrêt minimal à l’aide du service Azure Database Migration Service.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: 27da5f1b731b2cdb0604f91f7f9e78b19ee2908b
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489793"
---
# <a name="minimal-downtime-migration-to-azure-database-for-postgresql---single-server"></a>Migration avec temps d’arrêt minimal vers Azure Database pour PostgreSQL - Serveur unique
[!INCLUDE[applies-to-postgres-single-flexible-server](includes/applies-to-postgres-single-flexible-server-hyperscale.md)]

Vous pouvez effectuer des migrations PostgreSQL vers Azure Database pour PostgreSQL avec un temps d’arrêt minimal à l’aide de la toute nouvelle **fonctionnalité de synchronisation continue** du service [Azure Database Migration Service](https://aka.ms/get-dms) (DMS). Cette fonctionnalité limite les temps d’arrêt affectant l’application.

## <a name="overview"></a>Vue d’ensemble
Azure DMS procède à une charge initiale de votre instance locale vers Azure Database pour PostgreSQL, puis synchronise en continu les nouvelles transactions vers Azure tandis que l’application s’exécute. Une fois les données à jour sur le côté Azure cible, vous arrêtez momentanément l’application (temps d’arrêt minimal), vous attendez que le dernier lot de données apparaisse côté cible (à partir du moment où vous arrêtez l’application jusqu’à ce qu’elle ne soit plus disponible pour accepter du trafic), puis vous mettez à jour votre chaîne de connexion pour qu’elle pointe vers Azure. Lorsque vous avez terminé, votre application sera disponible sur Azure !

:::image type="content" source="./media/howto-migrate-online/ContinuousSync.png" alt-text="Synchronisation continue avec le service Azure Database Migration Service":::

## <a name="next-steps"></a>Étapes suivantes
- Regardez la vidéo [App Modernization with Microsoft Azure](https://medius.studios.ms/Embed/Video/BRK2102?sid=BRK2102) (Modernisation d’applications avec Microsoft Azure) qui contient une démonstration sur la migration d’applications PostgreSQL vers Azure Database pour PostgreSQL.
- Consultez le tutoriel [Migrer PostgreSQL vers Azure Database pour PostgreSQL en ligne à l’aide de DMS](../dms/tutorial-postgresql-azure-postgresql-online.md).