---
title: Redémarrer - Portail Azure - Azure Database pour PostgreSQL - Serveur flexible
description: Cet article explique comment effectuer des opérations de redémarrage dans Azure Database pour PostgreSQL à l’aide du Portail Azure.
author: sr-msft
ms.author: srranga
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 952bd6dddf9f276ed1a4a18f03799147f1902198
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90929793"
---
# <a name="restart-azure-database-for-postgresql---flexible-server"></a>Redémarrer Azure Database pour PostgreSQL - Serveur flexible

> [!IMPORTANT]
> Azure Database pour PostgreSQL - Serveur flexible est en préversion

Cet article indique la procédure pas-à-pas à suivre pour redémarrer le serveur flexible. Cette opération est utile pour appliquer les modifications de paramètres statiques nécessitant le redémarrage du serveur de base de données. La procédure est la même pour les serveurs configurés avec une haute disponibilité redondante dans une zone. 

> [!IMPORTANT]
> Lorsqu’ils sont configurés avec une haute disponibilité, les serveurs principal et de secours sont redémarrés en même temps.

## <a name="pre-requisites"></a>Conditions préalables

Pour utiliser ce guide pratique, il vous faut :

-   Vous devez disposer d’un serveur flexible.

## <a name="restart-your-flexible-server"></a>Redémarrez votre serveur flexible

Pour redémarrer votre serveur flexible, procédez comme suit.

1.  Dans le  [Portail Azure](https://portal.azure.com/), choisissez le serveur flexible à redémarrer.

2.  Cliquez sur **Vue d’ensemble** dans le volet gauche, puis cliquez sur **Redémarrer**.
   
     :::image type="content" source="./media/how-to-restart-server-portal/restart-base-page.png" alt-text="Redémarrer la sélection":::

3.  Un message de confirmation contextuel s’affiche.

4.  Cliquez sur **Oui** si vous souhaitez continuer.
   
     :::image type="content" source="./media/how-to-restart-server-portal/restart-pop-up.png" alt-text="Redémarrer la sélection":::
 
6.  Une notification indique que l’opération de redémarrage est lancée.

## <a name="next-steps"></a>Étapes suivantes

-   Découvrez la [continuité d’activité](./concepts-business-continuity.md)
-   Apprenez-en davantage sur la [haute disponibilité avec redondance interzone](./concepts-high-availability.md)
