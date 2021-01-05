---
title: Comprendre la remise de réservation - Serveur unique Azure Database pour PostgreSQL
description: Découvrez comment une remise de réservation est appliquée à des serveurs uniques Azure Database pour PostgreSQL.
author: mksuni
ms.author: sumuth
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: ace362872f0b7ba8e2f3d0302c887e2465c62982
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240339"
---
# <a name="how-a-reservation-discount-is-applied-to-azure-database-for-postgresql-single-server"></a>Comment une remise de réservation est-elle appliquée à un serveur unique Azure Database pour PostgreSQL ?

Lorsque vous achetez une capacité réservée de serveur unique Azure Database pour PostgreSQL, la remise de réservation est automatiquement appliquée aux bases de données de serveurs uniques PostgreSQL qui correspondent aux attributs et à la quantité de la réservation. Une réservation couvre uniquement les coûts de calcul de votre serveur unique Azure Database pour PostgreSQL. Les frais de stockage et de réseau vous sont facturés aux tarifs normaux.

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de * **capacité utilisée ou perdue** _. Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.</br>

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n’est trouvée dans l’étendue spécifiée, les heures réservées sont perdues.

## <a name="discount-applied-to-azure-database-for-postgresql-single-server"></a>Remise appliquée à un serveur unique Azure Database pour PostgreSQL

La remise de capacité réservée de serveur unique Azure Database pour PostgreSQL est appliquée à votre serveur unique PostgreSQL en cours d’exécution sur une base horaire. La réservation que vous achetez est associée au calcul utilisé par le serveur unique Azure Database pour PostgreSQL en cours d’exécution. Pour des serveurs uniques PostgreSQL qui ne s’exécutent pas pendant une heure entière, la réservation est automatiquement appliquée aux autres serveurs uniques Azure Database pour PostgreSQL qui correspondent aux attributs de la réservation. La remise peut être appliquée à des serveurs uniques Azure Database pour PostgreSQL qui s’exécutent simultanément. Si aucun serveur unique PostgreSQL s’exécutant pendant une heure entière ne correspond aux attributs de la réservation, vous ne bénéficiez pas pleinement de la remise de réservation pour cette heure.

Les exemples suivants montrent comment la remise de capacité réservée de serveur unique Azure Database pour PostgreSQL s’applique en fonction du nombre de cœurs achetés et du moment où ils s’exécutent.

_ **Exemple 1**  : Vous achetez une capacité réservée de serveur unique Azure Database pour PostgreSQL pour un 8 vCore. Si vous exécutez un serveur unique Azure Database pour PostgreSQL 16 vCore qui correspond au reste des attributs de la réservation, vous êtes facturé au prix du paiement à l’utilisation pour 8 vCore de votre utilisation de calcul de serveur unique PostgreSQL et vous bénéficiez de la remise de réservation pour une heure de calcul utilisé de serveur unique PostgreSQL 8 vCore.</br>

Dans le reste de ces exemples, nous supposons que la capacité réservée de serveur unique Azure Database pour PostgreSQL que vous achetez est pour un serveur unique Azure Database pour PostgreSQL 16 vCore et que le reste des attributs de réservation correspond aux serveurs uniques PostgreSQL en cours d’exécution.

* **Exemple 2 :** Vous exécutez deux serveurs uniques Azure Database pour PostgreSQL avec 8 vCore chacun pendant une heure. La remise de réservation pour 16 vCore est appliquée aux calculs utilisés par le serveur unique Azure Database pour PostgreSQL 8 vCore.

* **Exemple 3**  : Vous exécutez un serveur unique Azure Database pour PostgreSQL 16 vCore entre 13h et 13h30. Vous exécutez un autre serveur unique Azure Database pour PostgreSQL 16 vCore entre 13h30 et 14h. Les deux sont couvertes par la remise de réservation.

* **Exemple 4**  : Vous exécutez un serveur unique Azure Database pour PostgreSQL 16 vCore entre 13h et 13h45. Vous exécutez un autre serveur unique Azure Database pour PostgreSQL 16 vCore entre 13h30 et 14h. Le prix du chevauchement de 15 minutes vous est facturé selon la méthode du paiement à l’utilisation. La remise de réservation s’applique aux calculs utilisés pendant la période restante.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports de facturation d’utilisation, consultez [Comprendre l’utilisation des réservations Azure](./understand-reserved-instance-usage-ea.md).

## <a name="next-steps"></a>Étapes suivantes

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).