---
title: 'Azure Peering Service : comment mesurer les données de télémétrie des connexions '
description: Dans ce tutoriel, découvrez comment mesurer les données de télémétrie des connexions.
services: peering-service
author: derekolo
ms.service: peering-service
ms.topic: tutorial
ms.date: 05/18/2020
ms.author: derekol
Customer intent: Customer wants to measure their connection telemetry per prefix to Microsoft services with Azure Peering Service.
ms.openlocfilehash: abbe69ebbaed56ed416f85fafa7b77a1740fabe7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022494"
---
# <a name="tutorial-measure-peering-service-connection-telemetry"></a>Tutoriel : Mesurer la télémétrie de la connexion Peering Service

 Dans ce tutoriel, vous allez découvrir comment mesurer les données de télémétrie relatives à vos connexions Peering Service.
 
 La télémétrie des connexions fournit des insights collectés au sujet de la connectivité entre l’endroit où se trouve le client et le réseau Microsoft. Cet article explique comment consulter le rapport de latence d’une connexion Azure Peering Service spécifique. 

Pour mesurer les données de télémétrie des connexions Peering Service, vous devez inscrire une connexion Peering Service dans le portail Azure. Pour savoir comment inscrire une connexion, consultez [Inscrire une connexion Peering Service - Portail Azure](azure-portal.md).


## <a name="view-a-latency-report"></a>Afficher un rapport de latence

Pour afficher un rapport de latence au sujet d’une connexion Peering Service spécifique, effectuez ces étapes.

1. Sélectionnez **Toutes les ressources** dans le volet gauche, puis sélectionnez la connexion Peering Service. Sélectionnez ensuite **Ouvrir** sous **Préfixes**. 

   ![Sélectionner la connexion Peering Service](./media/peering-service-measure/peering-service-measure-menu.png)

2. Une page du rapport de latence pour tous les préfixes associés à cette connexion Peering Service s’affiche. 

      ![Page du rapport de latence](./media/peering-service-measure/peering-service-latency-report.png)

3. Par défaut, le rapport est mis à jour toutes les heures qui sont indiquées dans cette page. Si vous souhaitez afficher le rapport selon une autre chronologie, choisissez l’option appropriée dans **Afficher les données du/de la dernier(ère)** . 

4. Pour voir les événements liés à un préfixe particulier, sélectionnez le nom du préfixe et sélectionnez **Événements de préfixe** dans le volet gauche. Les événements capturés s’affichent.


   ![Événements de préfixe](./media/peering-service-measure/peering-service-prefix-event.png)

 Voici quelques événements possibles qui peuvent être capturés dans la liste **Événements de préfixe**.

| **Événements de préfixe** | **Type d’événement**|**Raisonnement**|
|-----------|---------|---------|
| PrefixAnnouncementEvent |Information|Réception de l’annonce du préfixe|
| PrefixWithdrawalEvent|Avertissement| Réception du retrait du préfixe |
| PrefixBackupRouteAnnouncementEvent |Information|Réception de l’annonce de la route de sauvegarde du préfixe |
| PrefixBackupRouteWithdrawalEvent|Avertissement|Réception du retrait de la route de sauvegarde du préfixe |
| PrefixActivePath |Information| Route active du préfixe actuel   |
| PrefixBackupPath | Information|Route de sauvegarde du préfixe actuel   |
| PrefixOriginAsChangeEvent|Critique| Réception du préfixe exact avec un numéro de système autonome d’origine différent (pour la route active)| 
| PrefixBackupRouteOriginAsChangeEvent  | Error|Réception du préfixe avec un numéro de système autonome d’origine différent (pour la route de sauvegarde)  |

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la connexion Peering Service, consultez [Connexion Peering Service](connection.md).
- Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).