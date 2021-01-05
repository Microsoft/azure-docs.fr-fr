---
title: Statistiques en temps réel dans Azure CDN | Microsoft Docs
description: Les statistiques en temps réel fournissent des données en temps réel sur les performances du CDN Azure lors de la diffusion de contenu à vos clients.
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: c7989340-1172-4315-acbb-186ba34dd52a
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 3af2e849aa6658e539b0b5bdbda4428cc28e5ce5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84887240"
---
# <a name="real-time-stats-in-microsoft-azure-cdn"></a>Statistiques en temps réel dans Microsoft Azure CDN
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>Vue d’ensemble
Ce document explique les statistiques en temps réel dans Microsoft Azure CDN.  Cette fonctionnalité fournit des données en temps réel (par exemple, relatives à la bande passante, aux états du cache et aux connexions simultanées) à votre profil CDN lors de la diffusion de contenu à vos clients. Elle permet une surveillance continue de l’intégrité de votre service à tout moment, y compris lors des événements de mise en service.

Les graphiques suivants sont disponibles :

* [Bande passante](#bandwidth)
* [Codes d’état](#status-codes)
* [États du cache](#cache-statuses)
* [Connexions](#connections)

## <a name="accessing-real-time-stats"></a>Accès à des statistiques en temps réel
1. Dans le [portail Azure](https://portal.azure.com), accédez à votre profil CDN.
   
    ![Panneau du profil CDN](./media/cdn-real-time-stats/cdn-profile-blade.png)
2. Dans le panneau Profil CDN, cliquez sur le bouton **Gérer** .
   
    ![Bouton de gestion du panneau de profil CDN](./media/cdn-real-time-stats/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.
3. Pointez sur l’onglet **Analyse**, puis sur le menu volant **Statistiques en temps réel**.  Cliquez sur **HTTP Large Object**.
   
    ![Portail de gestion CDN](./media/cdn-real-time-stats/cdn-premium-portal.png)
   
    Les graphiques de statistiques en temps réel s’affichent.

Chacun de ces graphiques affiche des statistiques en temps réel pour la période sélectionnée. L’affichage débute dès le chargement de la page.  Les graphiques se mettent automatiquement à jour après quelques secondes.  Le bouton **Actualiser le graphique**, le cas échéant, permet d’effacer le graphique. En cliquant sur ce bouton, seules les données sélectionnées s’afficheront.

## <a name="bandwidth"></a>Bande passante
![Graphique Bande passante](./media/cdn-real-time-stats/cdn-bandwidth.png)

Le graphique **Bande passante** montre la quantité de bande passante utilisée pour la plateforme actuelle sur l’intervalle de temps sélectionné. La partie ombrée du graphique indique l’utilisation de la bande passante. La quantité exacte de bande passante en cours d’utilisation s’affiche directement sous le graphique linéaire.

## <a name="status-codes"></a>Codes d’état
![Graphique Code d’état](./media/cdn-real-time-stats/cdn-status-codes.png)

Le graphique **Codes d’état** indique la fréquence à laquelle certains codes de réponse HTTP se produisent sur l’intervalle de temps sélectionné.

> [!TIP]
> Pour obtenir une description de chaque option de code d’état HTTP, consultez la page [Azure CDN HTTP Status Codes](/previous-versions/azure/mt759238(v=azure.100))(Codes d’état HTTP d’Azure CDN).
> 
> 

La liste des codes d’état HTTP s’affiche directement au-dessus du graphique. Cette liste indique chaque code d’état pouvant être inclus dans le graphique linéaire et le nombre actuel d’occurrences par seconde pour ce code d’état. Par défaut, une ligne s’affiche pour chacun de ces codes d’état dans le graphique. Toutefois, vous pouvez choisir de surveiller uniquement les codes d’état qui ont une importance spéciale pour votre configuration CDN. Pour ce faire, cochez les codes d’état souhaité et désactivez toutes les autres options, puis cliquez sur **Actualiser le graphique**. 

Vous pouvez masquer temporairement les données consignées pour un code d’état spécifique.  Dans la légende qui se trouve directement sous le graphique, cliquez sur le code d’état que vous souhaitez masquer. Le code d’état est immédiatement masqué dans le graphique. Pour le faire réapparaître, cochez de nouveau cette option de code d’état.

## <a name="cache-statuses"></a>États du cache
![Graphique États du cache](./media/cdn-real-time-stats/cdn-cache-status.png)

Le graphique **États du cache** indique la fréquence à laquelle certains types d’états du cache se produisent sur l’intervalle de temps sélectionné. 

> [!TIP]
> Pour obtenir une description de chaque option de code d’état du cache, consultez la page [Azure CDN Cache Status Codes](/previous-versions/azure/mt759237(v=azure.100))(Codes d’état du cache d’Azure CDN).
> 
> 

La liste des codes d’état du cache s’affiche directement au-dessus du graphique. Cette liste indique chaque code d’état pouvant être inclus dans le graphique linéaire et le nombre actuel d’occurrences par seconde pour ce code d’état. Par défaut, une ligne s’affiche pour chacun de ces codes d’état dans le graphique. Toutefois, vous pouvez choisir de surveiller uniquement les codes d’état qui ont une importance spéciale pour votre configuration CDN. Pour ce faire, cochez les codes d’état souhaité et désactivez toutes les autres options, puis cliquez sur **Actualiser le graphique**. 

Vous pouvez masquer temporairement les données consignées pour un code d’état spécifique.  Dans la légende qui se trouve directement sous le graphique, cliquez sur le code d’état que vous souhaitez masquer. Le code d’état est immédiatement masqué dans le graphique. Pour le faire réapparaître, cochez de nouveau cette option de code d’état.

## <a name="connections"></a>Connexions
![Graphique Connexions](./media/cdn-real-time-stats/cdn-connections.png)

Ce graphique indique le nombre de connexions qui ont été établies pour vos serveurs Edge. Chaque demande de ressource qui traverse notre CDN entraîne une connexion.

## <a name="next-steps"></a>Étapes suivantes
* Tenez-vous informé en consultant la page [Real-time alerts in Azure CDN](cdn-real-time-alerts.md)
* Allez plus loin en vous familiarisant avec les [rapports HTTP avancés](cdn-advanced-http-reports.md)
* Analysez les [modes d’utilisation](cdn-analyze-usage-patterns.md)

