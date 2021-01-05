---
title: Comprendre la remise sur réservation dans Azure Cosmos DB
description: Découvrez comment la remise de réservation est appliquée au débit provisionné (RU/s) dans Azure Cosmos DB.
author: bandersmsft
ms.author: banders
ms.reviewer: sngun
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 02/12/2020
ms.openlocfilehash: ca4c1810912771f56661ca5b682b6991735f526e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023140"
---
# <a name="understand-how-the-reservation-discount-is-applied-to-azure-cosmos-db"></a>Comprendre comment la remise de réservation est appliquée à Azure Cosmos DB

Quand vous achetez une capacité réservée Azure Cosmos DB, la remise de réservation est automatiquement appliquée aux ressources Azure Cosmos DB qui correspondent aux attributs et à la quantité de la réservation. Une réservation couvre le débit provisionné pour les ressources Azure Cosmos DB. Elle ne couvre pas les frais de logiciel, de réseau, de stockage ou de conteneur prédéfini.

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de « *capacité utilisée ou perdue* ». Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n'est trouvée dans l'étendue spécifiée, les heures réservées sont *perdues*.

## <a name="reservation-discount-applied-to-azure-cosmos-db-accounts"></a>Remise de réservation appliquée aux comptes Azure Cosmos DB

Une remise de réservation est appliquée au [débit provisionné](../../cosmos-db/request-units.md) sous la forme d’unités de requête par seconde (RU/s) et calculée sur une base horaire. Dans le cas des ressources Azure Cosmos DB qui ne s’exécutent pas pendant une heure entière, la remise de réservation est automatiquement appliquée aux ressources Azure Cosmos DB qui correspondent aux attributs de la réservation. La remise peut être appliquée à des ressources Azure Cosmos DB qui s’exécutent simultanément. Si aucune ressource Cosmos DB ne s’exécute pendant une heure entière et ne correspond aux attributs de la réservation, vous ne bénéficiez pas pleinement de la remise de réservation pour cette heure.

Les remises sont hiérarchisées. Plus le nombre d’unités de requête est élevé pour une réservation, plus celle-ci bénéficie d’une remise importante.

L’achat de réservation applique des remises à toutes les régions selon le taux correspondant aux tarifs à la demande pour chaque région. Pour connaître les taux de remise de réservation dans chaque région, consultez la section [Remise de réservation par région](#reservation-discount-per-region) de cet article.

## <a name="reservation-discount-per-region"></a>Remise de réservation par région
La remise de réservation est appliquée aux coûts de débit d’Azure Cosmos DB sur une base horaire. Elle est appliquée à l’abonnement unique ou à l’étendue du compte/de l’inscription. La remise de réservation s’applique à l’utilisation de compteurs dans les différentes régions, selon les taux suivants :

|Description du compteur  |Région |Ratio  |
|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/heure - Asie-Pacifique Sud-Est  |  Asie-Pacifique Sud-Est    |   1      |
|Azure Cosmos DB - 100 RU/s/heure - Asie-Pacifique Est |   Asie-Pacifique Est   |    1     |
|Azure Cosmos DB - 100 RU/s/heure - Europe Nord|  Europe Nord       |    1     |
|Azure Cosmos DB - 100 RU/s/heure - Corée Sud|    Corée Sud     |     1    |
|Azure Cosmos DB - 100 RU/s/heure - Europe Ouest|    Europe Ouest     |      1   |
|Azure Cosmos DB - 100 RU/s/heure - Corée Centre|   Corée Centre    |       1  |
|Azure Cosmos DB - 100 RU/s/heure - Royaume-Uni Sud|   Sud du Royaume-Uni      |     1    |
|Azure Cosmos DB - 100 RU/s/heure - Royaume-Uni Ouest|   Ouest du Royaume-Uni      |    1     |
|Azure Cosmos DB - 100 RU/s/heure - Royaume-Uni Nord |   Nord du Royaume-Uni    |     1    |
|Azure Cosmos DB - 100 RU/s/heure - Royaume-Uni Sud 2|   Sud du Royaume-Uni 2      |     1    |
|Azure Cosmos DB - 100 RU/s/heure - USA Est 2|  USA Est 2     |     1    |
|Azure Cosmos DB - 100 RU/s/heure - USA Centre Nord|   USA Centre Nord      |     1    |
|Azure Cosmos DB - 100 RU/s/heure - USA Ouest|   USA Ouest      |     1    |
|Azure Cosmos DB - 100 RU/s/heure - USA Centre| USA Centre        |     1    |
|Azure Cosmos DB - 100 RU/s/heure - USA Ouest 2|   USA Ouest 2      |      1   |
|Azure Cosmos DB - 100 RU/s/heure - USA Centre-Ouest|   USA Centre-Ouest      |       1  |
|Azure Cosmos DB - 100 RU/s/heure - USA Est|   USA Est      |  1       |
|Azure Cosmos DB - 100 RU/s/heure - Afrique du Sud Nord|     Afrique du Sud Nord    |   1      |
|Azure Cosmos DB - 100 RU/s/heure - Afrique du Sud Ouest |    Afrique du Sud Ouest      |    1     |
|Azure Cosmos DB - 100 RU/s/heure - Inde Sud|    Inde Sud     |    1,0375    |
|Azure Cosmos DB - 100 RU/s/heure - Canada Est|   Canada Est      |    1.1      |
|Azure Cosmos DB - 100 RU/s/heure - Japon Est|   Japon Est      |    1,125     |
|Azure Cosmos DB - 100 RU/s/heure - Japon Ouest|     Japon Ouest    |   1,125       |
|Azure Cosmos DB - 100 RU/s/heure - Inde Ouest|     Inde Ouest    |    1,1375     |
|Azure Cosmos DB - 100 RU/s/heure - Inde Centre|    Inde Centre     |  1,1375       |
|Azure Cosmos DB - 100 RU/s/heure - Australie Est|     Australie Est    |   1,15       |
|Azure Cosmos DB - 100 RU/s/heure - Canada Centre|  Canada Centre       |   1.2       |
|Azure Cosmos DB - 100 RU/s/heure - France Centre|   France Centre      |    1,25      |
|Azure Cosmos DB - 100 RU/s/heure - Brésil Sud|  Brésil Sud       |   1.5      |
|Azure Cosmos DB - 100 RU/s/heure - Australie Centre|   Australie Centre      |   1.5      |
|Azure Cosmos DB - 100 RU/s/heure - Australie Centre 2| Australie Centre 2        |    1.5     |
|Azure Cosmos DB - 100 RU/s/heure - France Sud|    France Sud     |    1,625     |

## <a name="scenarios-that-show-how-the-reservation-discount-is-applied"></a>Scénarios qui montrent comment la remise de réservation est appliquée

Imaginons les exigences suivantes pour une réservation :

* Débit nécessaire : 50 000 RU/s  
* Régions utilisées : 2

Dans ce cas, votre total de frais à la demande correspond à une quantité de 500, par compteur de 100 RU/s dans ces deux régions. La consommation de RU/s totale est de 100 000 par heure.

**Scénario 1**

Par exemple, supposons que vous avez besoin de déployer Azure Cosmos DB dans les régions USA Centre Nord et USA Ouest. Chaque région dispose d’une consommation de débit de 50 000 RU/s. un achat de réservation de 100 000 RU/s couvrirait complètement vos frais à la demande.

La remise qu’une réservation couvre est calculée comme suit : consommation de débit * taux de la remise de réservation pour la région concernée. Pour les régions USA Centre Nord et USA Ouest, le taux de la remise de réservation est 1. Ainsi, la remise totale de RU/s est de 100 000. Cette valeur est calculée comme suit . 50 000 * 1 + 50 000 * 1 = 100 000 RU/s. Vous ne payez aucun frais supplémentaire au tarif habituel du paiement à l’utilisation.

|Description du compteur | Région |Consommation de débit (RU/s) |Remise de réservation appliquée aux RU/s |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/heure - USA Centre Nord  |   USA Centre Nord  | 50 000  | 50 000  |
|Azure Cosmos DB - 100 RU/s/heure - USA Ouest  |  USA Ouest   |  50 000  |  50 000 |

**Scénario 2**

Par exemple, supposons que vous avez besoin de déployer Azure Cosmos DB dans les régions Australie Centre 2 et France Sud. Chaque région dispose d’une consommation de débit de 50 000 RU/s. un achat de réservation de 100 000 RU/s serait applicable comme suit (en supposant que l’utilisation dans la région Australie Centre 2 a fait l’objet d’une remise en premier) :

|Description du compteur | Région |Consommation de débit (RU/s) |Remise de réservation appliquée aux RU/s |
|---------|---------|---------|---------|
|Azure Cosmos DB - 100 RU/s/heure - Australie Centre 2  |  Australie Centre 2   |  50 000  |  50 000   |
|Azure Cosmos DB - 100 RU/s/heure - France Sud  |  France Sud   |  50 000 |  15 384  |

Une utilisation de 50 000 unités dans la région Australie Centre 2 correspond à 75 000 RU/s d’utilisation facturable (ou utilisation normalisée). Cette valeur est calculée comme suit : consommation de débit * taux de la remise de réservation pour la région concernée. Le total est de 75 000 RU/s d’utilisation facturable ou normalisée. Cette valeur est calculée comme suit . 50 000 * 1.5 = 75 000 RU/s.

L’achat d’une réservation de 100 000 RU/s absorbe les 75 000 RU/s dans la région Australie Centre 2. Il reste 25 000 RU/s pour la région France Sud. Sur les 25 000 RU/s restantes, une remise de réservation de 15 384 RU/s est appliquée à la région France Sud. Cette valeur de remise est calculée comme suit : 25 000 / 1,625 = 15 384 RU/s. Les 34 616 RU/s restantes dans la région France Sud sont facturées au tarif habituel du paiement à l’utilisation.

Le système de facturation Azure fait bénéficier de l’avantage lié à la facturation de la réservation, la première instance qui est traitée et qui satisfait à la configuration de la réservation. En l’occurrence, dans notre exemple, Australie Centre 2.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports de facturation de l’utilisation, consultez [Comprendre l’utilisation des réservations Azure](understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

* [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)  
* [Prépayer des ressources Azure Cosmos DB avec une capacité réservée Azure Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md)  
* [Prépayer des ressources de calcul SQL Database avec une capacité réservée Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)  
* [Gérer les réservations pour Azure](manage-reserved-vm-instance.md)  
* [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)  
* [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
* [Comprendre l’utilisation d’une réservations pour les abonnements CSP](/partner-center/azure-reservations)