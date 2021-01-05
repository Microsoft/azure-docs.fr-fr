---
title: Comprendre la remise de réservation - Azure Database pour MySQL
description: Découvrez comment une remise de réservation est appliquée à vos serveurs Azure Database pour MySQL.
author: mksuni
ms.author: sumuth
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 85a8c59cce74787a7b1723850915ec133b34b3b0
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240373"
---
# <a name="how-a-reservation-discount-is-applied-to-azure-database-for-mysql"></a>Comment une remise de réservation est-elle appliquée à Azure Database pour MySQL ?

Lorsque vous achetez une capacité réservée Azure Database pour MySQL, la remise de réservation est automatiquement appliquée aux serveurs MySQL qui correspondent aux attributs et à la quantité de la réservation. Une réservation couvre uniquement les coûts de calcul de votre Azure Database pour MySQL. Les frais de stockage et de réseau vous sont facturés aux tarifs normaux.

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de * **capacité utilisée ou perdue** _. Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.</br>

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n’est trouvée dans l’étendue spécifiée, les heures réservées sont perdues.

## <a name="discount-applied-to-azure-database-for-mysql"></a>Remise appliquée à Azure Database pour MySQL

La remise de capacité réservée Azure Database pour MySQL est appliquée aux serveurs MySQL en cours d’exécution sur une base horaire. La réservation que vous achetez est associée au calcul utilisé par les serveurs Azure Database pour MySQL en cours d’exécution. Pour les serveurs MySQL qui ne s’exécutent pas pendant une heure entière, la réservation est automatiquement appliquée aux autres serveurs Azure Database pour MySQL qui correspondent aux attributs de la réservation. La remise peut être appliquée à des serveurs Azure Database pour MySQL qui s’exécutent simultanément. Si aucun serveur MySQL s’exécutant pendant une heure entière ne correspond aux attributs de la réservation, vous ne bénéficiez pas pleinement de la remise de réservation pour cette heure.

Les exemples suivants montrent comment la remise de capacité réservée Azure Database pour MySQL s’applique en fonction du nombre de cœurs achetés et du moment où ils s’exécutent.

_ **Exemple 1**  : Vous achetez une capacité réservée Azure Database pour MySQL pour un 8 vCore. Si vous exécutez un serveur Azure Database pour MySQL 16 vCore qui correspond au reste des attributs de la réservation, vous êtes facturé au prix du paiement à l’utilisation pour 8 vCore de votre utilisation de calcul de serveur MySQL et vous bénéficiez de la remise de réservation pour une heure de calcul utilisé de serveur MySQL 8 vCore.</br>

Dans le reste de ces exemples, nous supposons que la capacité réservée Azure Database pour MySQL que vous achetez est pour un serveur Azure Database pour MySQL 16 vCore et que le reste des attributs de réservation correspond aux serveurs MySQL en cours d’exécution.

* **Exemple 2 :** Vous exécutez deux serveurs Azure Database pour MySQL avec 8 vCore chacun pendant une heure. La remise de réservation pour 16 vCore est appliquée aux calculs utilisés par le serveur Azure Database pour MySQL 8 vCore.

* **Exemple 3**  : Vous exécutez un serveur Azure Database pour MySQL 16 vCore entre 13h et 13h30. Vous exécutez un autre serveur Azure Database pour MySQL 16 vCore entre 13h30 et 14h. Les deux sont couvertes par la remise de réservation.

* **Exemple 4**  : Vous exécutez un serveur Azure Database pour MySQL 16 vCore entre 13h et 13h45. Vous exécutez un autre serveur Azure Database pour MySQL 16 vCore entre 13h30 et 14h. Le prix du chevauchement de 15 minutes vous est facturé selon la méthode du paiement à l’utilisation. La remise de réservation s’applique aux calculs utilisés pendant la période restante.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports de facturation d’utilisation, consultez [Comprendre l’utilisation des réservations Azure](./understand-reserved-instance-usage-ea.md).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).