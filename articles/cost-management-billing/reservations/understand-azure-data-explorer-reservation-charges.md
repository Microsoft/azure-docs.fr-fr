---
title: Comprendre comment la remise de réservation est appliquée à Azure Data Explorer
description: Découvrez comment la remise de réservation est appliquée à Azure Data Explorer.
author: avneraa
ms.author: avnera
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/12/2020
ms.openlocfilehash: b222e4ff519fbe2fbb4fa58b39555ac9fead0917
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352913"
---
# <a name="understand-how-the-reservation-discount-is-applied-to-azure-data-explorer"></a>Comprendre comment la remise de réservation est appliquée à Azure Data Explorer

Lorsque vous achetez une capacité réservée Azure Data Explorer, la remise de réservation est automatiquement appliquée aux ressources Azure Data Explorer qui correspondent aux attributs et à la quantité de la réservation. Une réservation inclut les frais de majoration d’Azure Data Explorer. Elle n’inclut pas les ressources de calcul, de mise en réseau, de stockage ou toute autre ressource Azure utilisée pour exploiter le cluster Azure Data Explorer. Les réservations pour ces ressources doivent être achetées séparément.

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de base de « *capacité utilisée ou perdue* ». Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n'est trouvée dans l'étendue spécifiée, les heures réservées sont *perdues*.

## <a name="reservation-discount-applied-to-azure-data-explorer-clusters"></a>Remise de réservation appliquée aux clusters Azure Data Explorer

Une remise de réservation est appliquée à la consommation de majoration d’Azure Data Explorer sur une base horaire. Dans le cas des ressources Azure Data Explorer qui ne s’exécutent pas pendant une heure entière, la remise de réservation est automatiquement appliquée aux autres ressources Azure Data Explorer qui correspondent aux attributs de la réservation. La remise peut être appliquée à des ressources Azure Data Explorer qui s’exécutent simultanément. Si aucune ressource Azure Data Explorer ne s’exécute pendant une heure entière et ne correspond aux attributs de la réservation, vous ne bénéficiez pas pleinement de la remise de réservation pour cette heure.

> [!NOTE]
> * Il est **vivement recommandé** d’acheter une [capacité de réservation](../../virtual-machines/prepay-reserved-vm-instances.md) pour les machines virtuelles utilisées pour le cluster Azure Data Explorer afin d’optimiser les économies de capacité réservée.
> * L’achat de réservation appliquera les remises à toutes les régions.

## <a name="examples"></a>Exemples

Les exemples suivants montrent comment la remise de capacité réservée Azure Data Explorer est appliquée en fonction du nombre d’unités de majoration achetées et du moment où elles s’exécutent.
Par exemple, pour une taille de cluster du moteur : **2 machines virtuelles D11_v2**, le montant total de vos frais à la demande est de quatre unités de majoration d’Azure Data Explorer par heure.

**Scénario 1**

Vous achetez une capacité réservée Azure Data Explorer pour 8 unités de majoration d’Azure Data Explorer. Vous exécutez un cluster du moteur de deux machines virtuelles D13_v2 avec un total de 16 cœurs pour une facturation de 16 unités de majoration d’Azure Data Explorer par heure et qui corresponde au reste des attributs de la réservation. Vous êtes facturé au tarif du paiement à l’utilisation pour huit cœurs de calcul utilisé d’Azure Data Explorer et bénéficiez de la remise de réservation pour une heure d’utilisation de huit cœurs d’unité de majoration d’Azure Data Explorer.

Dans le reste de ces exemples, nous supposons que la capacité réservée Azure Data Explorer que vous achetez est pour un cluster Azure Data Explorer à 16 cœurs et que le reste des attributs de réservation correspond au cluster Azure Data Explorer en cours d’exécution.

**Scénario 2**

Vous exécutez deux clusters du moteur Azure Data Explorer avec huit cœurs, chacun pendant une heure dans deux régions différentes. La remise de réservation de 16 cœurs est appliquée pour le cluster et les 16 unités de majoration d’Azure Data Explorer qu’elles consomment.

**Scénario 3**

Vous exécutez un cluster du moteur Azure Data Explorer à 16 cœurs entre 13h et 13h30. Vous exécutez un autre cluster du moteur Azure Data Explorer à 16 cœurs entre 13h30 et 14h. Les deux sont couvertes par la remise de réservation.

**Scénario 4**

Vous exécutez un cluster du moteur Azure Data Explorer à 16 cœurs entre 13h et 13h40. Vous exécutez un autre cluster du moteur Azure Data Explorer à 16 cœurs entre 13h30 et 14h. Le prix du chevauchement de 15 minutes vous est facturé selon la méthode du paiement à l’utilisation. La remise de réservation s’applique à la majoration d’Azure Data Explorer utilisée pendant la période restante.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports de facturation d’utilisation, consultez [Comprendre l’utilisation des réservations Azure](understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

* [Prépayer des ressources de calcul Azure Data Explorer avec une capacité réservée Azure Data Explorer](/azure/data-explorer/pricing-reserved-capacity)  
* [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)  
* [Gérer les réservations Azure](manage-reserved-vm-instance.md)  
* [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
* [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
* [Comprendre l’utilisation d’une réservations pour les abonnements CSP](/partner-center/azure-reservations)