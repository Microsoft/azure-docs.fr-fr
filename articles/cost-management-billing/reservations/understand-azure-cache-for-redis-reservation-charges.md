---
title: Comprendre comment la remise de réservation s’applique à Azure Cache pour Redis | Microsoft Docs
description: Découvrez comment la remise de réservation s’applique aux instances Azure Cache pour Redis.
author: yegu-ms
manager: maiye
ms.service: cache
ms.topic: conceptual
ms.date: 01/22/2020
ms.author: yegu
ms.openlocfilehash: 5f9e0a18db0920acd35ebd7b133ed3fe5d0eaee9
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352947"
---
# <a name="how-the-reservation-discount-is-applied-to-azure-cache-for-redis"></a>Comment la remise de réservation s’applique à Azure Cache pour Redis

Quand vous achetez une capacité de réserve Azure Cache pour Redis, la remise de réservation s’applique automatiquement aux instances de cache qui correspondent aux attributs et à la quantité de la réservation. Une réservation couvre uniquement les coûts de calcul de votre instance Azure Cache pour Redis. Les frais de stockage et de réseau vous sont facturés aux tarifs normaux. La capacité de réserve est disponible uniquement pour les caches de [niveau Premium](../../azure-cache-for-redis/quickstart-create-redis.md).

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Une remise de réservation repose sur le principe de ***capacité utilisée ou perdue** _. Ainsi, si vous ne disposez pas des ressources correspondantes pour une heure donnée, vous perdez une quantité de réservation pour cette heure. Vous ne pouvez pas reporter les heures réservées inutilisées.

Lorsque vous arrêtez une ressource, la remise de réservation s'applique automatiquement à une autre ressource correspondante dans l'étendue spécifiée. Si aucune ressource correspondante n’est trouvée dans l’étendue spécifiée, les heures réservées sont perdues.

## <a name="discount-applied-to-azure-cache-for-redis"></a>Remise appliquée à Azure Cache pour Redis

La remise de capacité de réserve Azure Cache pour Redis s’applique à vos caches toutes les heures. La réservation que vous achetez est rapprochée de l’utilisation de la capacité de calcul émise par les caches en cours d’exécution. Quand ces caches ne s’exécutent pas pendant une heure entière, la réservation s’applique automatiquement aux autres caches correspondant aux attributs de la réservation. La remise peut s’appliquer à des caches qui s’exécutent simultanément. Si, parmi les caches qui s’exécutent pendant une heure entière, aucun ne correspond aux attributs de la réservation, vous ne profitez pleinement de la remise de réservation pour cette heure.

Les exemples suivants montrent comment la remise de capacité de réserve Azure Cache pour Redis s’applique en fonction du nombre de caches que vous avez achetés et du moment où ils s’exécutent.

_ **Exemple 1** : Vous achetez une capacité de réserve Azure Cache pour Redis pour un cache de 6 Go. Si vous exécutez un cache de 13 Go qui correspond au reste des attributs de la réservation, vous êtes facturé au prix du paiement à l’utilisation pour 7 Go d’utilisation de la capacité de calcul de votre instance Azure Cache pour Redis et vous bénéficiez de la remise de réservation pour une heure d’utilisation de la capacité de calcul du cache de 6 Go.

Pour le reste de ces exemples, nous partons du principe que la capacité de réserve Azure Cache pour Redis que vous achetez est destinée à un cache de 26 Go et que les autres attributs de réservation correspondent au cache en cours d’exécution.

* **Exemple 2 :** Vous exécutez deux caches de 13 Go pendant une heure. La remise de réservation de 26 Go s’applique à l’utilisation de la capacité de calcul des deux caches.

* **Exemple 3** : Vous exécutez un cache de 26 Go de 13h00 à 13h30. Vous exécutez un autre cache de 26 Go de 13h30 à 14h00. Les deux sont couvertes par la remise de réservation.

* **Exemple 4** : Vous exécutez un cache de 26 Go de 13h00 à 13h45. Vous exécutez un autre cache de 26 Go de 13h30 à 14h00. Le prix du chevauchement de 15 minutes vous est facturé selon la méthode du paiement à l’utilisation. La remise de réservation s’applique aux calculs utilisés pendant la période restante.

Pour comprendre et voir l’application de vos réservations Azure dans les rapports de facturation d’utilisation, consultez [Comprendre l’utilisation des réservations Azure](./understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter
Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).