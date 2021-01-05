---
title: Remises sur les réservations pour Azure App Service
description: Découvrez comment les remises de réservation s’appliquent aux timbres isolés Azure App Service. Les remises sont automatiquement appliquées aux frais de timbres dans une région.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: f7d3134652d75cb4ce40b3e9366d3083f5d760df
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682618"
---
# <a name="how-reservation-discounts-apply-to-azure-app-service-isolated-stamps"></a>Comment les remises de réservation s’appliquent aux timbres isolés Azure App Service

Après l’achat d’une capacité réservée de timbres isolés App Service, la remise de réservation est automatiquement appliquée aux frais de timbres dans une région. La remise de réservation s’applique à l’utilisation émise par le compteur de frais de timbre isolé. Les workers, les frontaux supplémentaires et toute autre ressource associée au timbre continuent d’être facturés au tarif régulier.

## <a name="reservation-discount-application"></a>Application de la remise de réservation

La remise sur les frais de timbres isolés de l’App Service s’applique aux timbres isolés en cours d’exécution sur une base horaire. Si vous n’avez pas de tampon déployé pendant une heure, la capacité réservée est gaspillée pour cette heure. Elle n’est pas reconduite.

Après l’achat, la réservation que vous achetez est jumelée à un timbre isolé courant dans une région spécifique. Si vous arrêtez ce tampon, les remises de réservation sont automatiquement appliquées à tous les autres tampons en cours d’exécution dans la région. Lorsqu’il n’existe aucun tampon, la réservation est appliquée au tampon suivant créé dans la région.

Lorsque les tampons ne s’exécutent pas pendant une heure complète, la réservation s’applique automatiquement à d’autres tampons correspondants dans la même région pendant la même heure.

## <a name="choose-a-stamp-type---windows-or-linux"></a>Choisir un type de tampon - Windows ou Linux

Un tampon isolé vide émet le compteur de tampon Windows par défaut. Par exemple, quand aucun rôle de travail n’est déployé. Il continue à émettre le compteur lorsque les rôles de travail Windows sont déployés. Si vous déployez un rôle de travail Linux, le compteur prend la valeur du compteur de tampon Linux. Le tampon émet le compteur Windows lorsque les rôles de travail Linux et Windows sont déployés.

Par conséquent, le compteur de tampons peut changer entre Windows et Linux pendant toute la durée de vie du tampon. Pendant ce temps, les réservations sont spécifiques au système d’exploitation. Vous devez acheter une réservation qui prend en charge les rôles de travail que vous envisagez de déployer sur le tampon. Les tampons spécifiques à Windows et les tampons mixtes utilisent la réservation Windows. Les tampons avec des rôles de travail Linux utilisent la réservation Linux.

Le seul cas où vous devez acheter une réservation Linux est lorsque vous envisagez de _n’avoir que_ des rôles de travail Linux dans le tampon.

## <a name="discount-examples"></a>Exemples de remises

Les exemples suivants montrent comment la remise d’instance réservée de frais de timbre isolé s’applique en fonction des déploiements.

- **Exemple 1** : Vous achetez une instance de capacité de tampon réservé isolé dans une région sans tampon isolé App Service. Vous déployez un nouveau tampon dans la région et payez des tarifs réservés pour ce tampon.
- **Exemple 2 :** Vous achetez une instance de capacité de tampon réservé isolé dans une région qui a déjà déployé un tampon isolé App Service. Vous commencez à recevoir le tarif réservé pour le tampon déployé.
- **Exemple 3** : Vous achetez une instance de capacité de tampon réservé isolé dans une région qui a déjà déployé un tampon isolé App Service. Vous commencez à recevoir le tarif réservé pour le tampon déployé. Par la suite, vous supprimez le tampon et en déployez un nouveau. Vous recevez le tarif réservé pour le nouveau tampon. Les remises ne sont pas reportées pour les durées sans tampons déployés.
- **Exemple 4** : Vous achetez une instance de capacité de tampon réservé Linux isolé dans une région puis déployez un nouveau tampon dans la région. Lorsque le tampon est initialement déployé sans les rôles de travail, il émet le compteur de tampons Windows. Aucune remise n’est reçue. Lorsque le premier rôle de travail Linux déploie le tampon, il émet le compteur de tampon Linux et la remise de réservation s’applique. Si un rôle de travail Windows est déployé ultérieurement sur le tampon, le compteur de tampons revient à Windows. Vous ne recevez plus de remise pour la réservation de tampon réservé Linux isolée.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](manage-reserved-vm-instance.md).
- Pour en savoir plus sur l’achat anticipé d’une capacité réservée de timbres isolés App Service pour économiser de l’argent, voir [Paiement anticipé des frais de timbres isolés App Service Azure avec capacité réservée](prepay-app-service-isolated-stamp.md).
- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
  - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
  - [Gérer les réservations dans Azure](manage-reserved-vm-instance.md)
  - [Comprendre l’utilisation d’une réservation pour un abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
  - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
