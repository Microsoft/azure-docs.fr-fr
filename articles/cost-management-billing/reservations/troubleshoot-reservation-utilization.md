---
title: Résoudre les problèmes d’utilisation de réservations Azure
description: Cet article vous aide à comprendre et à résoudre les problèmes de réservations Azure qui n’indiquent aucune utilisation ou une utilisation nulle (0) dans le portail Azure. L’utilisation qui ne correspond pas est également expliquée.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: reservations
ms.author: banders
ms.reviewer: yashar
ms.topic: troubleshooting
ms.date: 10/19/2020
ms.openlocfilehash: 9d5706843e8131110566ad8f955415b6db29f7ba
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207429"
---
# <a name="troubleshoot-reservation-utilization"></a>Résoudre les problèmes d’utilisation de réservations

Cet article vous aide à comprendre et à résoudre les problèmes de réservations Azure qui n’indiquent aucune utilisation ou une utilisation nulle (0) dans le portail Azure. L’utilisation qui ne correspond pas est également expliquée.

## <a name="symptoms"></a>Symptômes

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à **Réservations** .
1. Dans la liste des réservations, recherchez la quantité d’utilisation d’une réservation dans la colonne **Utilisation (%)** . Elle peut être de 0 %.
1. Sélectionnez la réservation.
1. Dans la page de vue d’ensemble de la réservation, le pourcentage utilisé dans le graphique peut ne pas correspondre à la valeur indiquée dans la liste des réservations.

## <a name="cause"></a>Cause

La colonne **Utilisation (%)** dans le portail Azure indique la valeur du jour. Cette valeur est calculée lorsque les données d’utilisation arrivent à partir de là où les ressources s’exécutent. Azure se sert de l’utilisation pour calculer le pourcentage d’utilisation.

Certaines ressources signalent l’utilisation plus lentement que d’autres. De plus, certains types de produits comme les bases de données SQL sont lents à signaler leurs données d’utilisation.

En raison de la latence, le calcul de l’utilisation peut indiquer des valeurs inférieures à l’utilisation réelle. La différence est perceptible à la limite de la journée. Dans ce cas, si Azure n’obtient pas les données d’utilisation de quatre à huit heures, il calcule une valeur de 0 %. La valeur 0 % est indiquée parce que les données d’utilisation ne sont pas parvenues, et il semble que la réservation n’applique aucun avantage à aucune ressource.

Lorsque les données d’utilisation arrivent, la valeur change jusqu’à atteindre le pourcentage correct. Lorsque toutes les données d’utilisation sont collectées, la valeur correcte est déterminée et affichée avec précision dans le graphique.

## <a name="solution"></a>Solution

Si vous constatez que vos valeurs d’utilisation ne correspondent pas à vos attentes, passez en revue le graphique pour obtenir une vue d’ensemble de l’utilisation réelle. Toute valeur de point datant de plus de deux jours doit être exacte. Les moyennes à plus long terme, de sept à 30 jours, doivent être exactes.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la gestion des réservations, consultez [Gérer les réservations pour les ressources Azure](manage-reserved-vm-instance.md).