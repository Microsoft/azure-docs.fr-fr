---
title: Arrêt/Démarrage - Portail Azure - Serveur flexible Azure Database pour MySQL
description: Cet article explique comment arrêter/démarrer des opérations dans Azure Database pour MySQL à l’aide du portail Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/29/2020
ms.openlocfilehash: 15b08ea67afe0d307470b5a4fb0f7d26e0f4ea82
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241920"
---
# <a name="stopstart-an-azure-database-for-mysql---flexible-server-preview"></a>Arrêter/Démarrer un serveur flexible Azure Database pour MySQL (préversion)

> [!IMPORTANT]
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Cet article indique la procédure pas-à-pas à suivre pour arrêter et démarrer le serveur flexible.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

-   Vous devez disposer d’un serveur flexible Azure Database pour MySQL.

## <a name="stop-a-running-server"></a>Arrêter un serveur en cours d’exécution

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez le serveur flexible à arrêter.

2.  Dans la page **Vue d’ensemble** , cliquez sur le bouton **Arrêter** dans la barre d’outils.
    
    :::image type="content" source="media/how-to-stop-start-server-portal/stop-server.png" alt-text="Arrêtez le serveur flexible."::: 

3.  Cliquez sur **Oui** pour confirmer l’arrêt de votre serveur.

    :::image type="content" source="media/how-to-stop-start-server-portal/confirm-stop.png" alt-text="Confirmez l’arrêt du serveur flexible."::: 

> [!NOTE]
> Une fois que le serveur est arrêté, les autres opérations de gestion ne sont pas disponibles pour le serveur flexible.

## <a name="start-a-stopped-server"></a>Démarrer un serveur arrêté

1.  Dans le [portail Azure](https://portal.azure.com/), choisissez le serveur flexible à démarrer.

2.  Dans la page **Vue d’ensemble** , cliquez sur le bouton **Démarrer** dans la barre d’outils.

    :::image type="content" source="media/how-to-stop-start-server-portal/start-server.png" alt-text="Démarrez le serveur flexible.":::  

> [!NOTE]
> Une fois que le serveur a démarré, toutes les opérations de gestion sont disponibles pour le serveur flexible.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [réseaux dans un serveur flexible Azure Database pour MySQL](./concepts-networking.md)
- [Créer et gérer un réseau virtuel de serveur flexible Azure Database pour MySQL avec le portail Azure](./how-to-manage-virtual-network-portal.md).

