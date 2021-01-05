---
title: Coûts des logiciels de réservations pour Azure
description: Découvrez les compteurs de logiciels qui ne sont pas inclus dans les coûts d’instance Azure Reserved VM Instance.
author: yashesvi
ms.reviewer: yashar
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: banders
ms.openlocfilehash: 625c40ea17cf20449f1be241a1605a80bf6a73f1
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352998"
---
# <a name="software-costs-not-included-with-azure-reserved-vm-instances"></a>Coûts des logiciels non inclus dans les instances Azure Reserved VM Instances

Les remises liées à l’instance réservée de machine virtuelle et à la capacité réservée SQL s’appliquent uniquement aux coûts d’infrastructure et non aux coûts des logiciels. Si vous n’utilisez pas de machines virtuelles Windows et que vous ne bénéficiez pas d’Azure Hybrid Benefit sur vos instances de machine virtuelle réservées, nous vous facturons les compteurs de logiciels listés dans la section suivante. Pour les déploiements SQL PaaS, le coût IP continue d’être facturé à l’aide d’un compteur distinct si Azure Hybrid Benefit n’est pas sélectionné.

## <a name="windows-software-meters-not-included-in-reservation-cost"></a>Compteurs de logiciels Windows non inclus dans le coût des réservations

| ID du compteur | Nom du compteur dans le fichier d’utilisation | Utilisé par la machine virtuelle |
| ------- | ------------------------| --- |
| e7e152ac-f29c-4cce-ad6e-026192c01ef2 | Réservation-Windows Server Rafale (1 cœur) | Série B |
| cac255a2-9f0f-4c62-8bd6-f0fa449c5f76 | Réservation-Windows Server Rafale (2 cœurs) | Série B |
| 09756b58-3fb5-4390-976d-9ddd14f9ed18 | Réservation-Windows Server Rafale (4 cœurs) | Série B |
| e828cb37-5920-4dc7-b30f-664e4dbcb6c7 | Réservation-Windows Server Rafale (8 cœurs) | Série B |
| f65a06cf-c9c3-47a2-8104-f17a8542215a | Réservation-Windows Server (1 cœur) | Tout sauf la série B |
| b99d40ae-41fe-4d1d-842b-56d72f3d15ee | Réservation-Windows Server (2 cœurs) | Tout sauf la série B |
| 1cb88381-0905-4843-9ba2-7914066aabe5 | Réservation-Windows Server (4 cœurs) | Tout sauf la série B |
| 07d9e10d-3e3e-4672-ac30-87f58ec4b00a | Réservation-Windows Server (6 cœurs) | Tout sauf la série B |
| 603f58d1-1e96-460b-a933-ce3775ac7e2e | Réservation-Windows Server (8 cœurs) | Tout sauf la série B |
| 36aaadda-da86-484a-b465-c8b5ab292d71 | Réservation-Windows Server (12 cœurs) | Tout sauf la série B |
| 02968a6b-1654-4495-ada6-13f378ba7172 | Réservation-Windows Server (16 cœurs) | Tout sauf la série B |
| 175434d8-75f9-474b-9906-5d151b6bed84 | Réservation-Windows Server (20 cœurs) | Tout sauf la série B |
| 77eb6dd0-88f5-4a16-ab39-05d1742efb25 | Réservation-Windows Server (24 cœurs) | Tout sauf la série B |
| 0d5bdf46-b719-4b1f-a780-b9bdfffd0591 | Réservation-Windows Server (32 cœurs) | Tout sauf la série B |
| f1214b5c-cc16-445f-be6c-a3bb75f8395a | Réservation-Windows Server (40 cœurs) | Tout sauf la série B |
| 637b7c77-65ad-4486-9cc7-dc7b3e9a8731 | Réservation-Windows Server (64 cœurs) | Tout sauf la série B |
| da612742-e7cc-4ca3-9334-0fb7234059cd | Réservation-Windows Server (72 cœurs) | Tout sauf la série B |
| a485cb8c-069b-4cf3-9a8e-ddd84b323da2 | Réservation-Windows Server (128 cœurs) | Tout sauf la série B |
| 904c5c71-1eb7-43a6-961c-d305a9681624 | Réservation-Windows Server (256 cœurs) | Tout sauf la série B |
| 6fdab81b-4284-4df9-8939-c237cc7462fe | Réservation-Windows Server (96 cœurs) | Tout sauf la série B |

## <a name="cloud-services-software-meters-not-included-in-reservation-cost"></a>Compteurs de services cloud non inclus dans le coût des réservations

| ID du compteur | Nom du compteur dans le fichier d’utilisation |
| ------- | ------------------------|
|ac9d47ff-ff68-4afc-a145-0c321cf8d0d5|Licence Cloud Services 1 processeur virtuel|
|e0434559-19ee-4132-9c46-05ad4044f3f7|Licence Cloud Services 2 processeurs virtuels|
|6ecc834e-39b3-48b3-8d10-cc5626bacb66|Licence Cloud Services 4 processeurs virtuels|
|13103090-ca72-4825-ab12-7f16c4931d95|Licence Cloud Services 8 processeurs virtuels|
|ecd2bb6e-45a5-49aa-a58b-3947ba21c364|Licence Cloud Services 16 processeurs virtuels|
|de2c7f1d-06dc-4b16-bc8b-c2ec5f4c8aee|Licence Cloud Services 20 processeurs virtuels|
|ca1af837-4b35-47f5-8d14-b1988149c4ca|Licence Cloud Services 32 processeurs virtuels|
|dc72ee45-2ab7-4698-b435-e2cf10d1f9f6|Licence Cloud Services 64 processeurs virtuels|
|7a803026-244c-4659-834c-11e6b2d6b76f|Licence Cloud Services 80 processeurs virtuels|

## <a name="get-rates-for-azure-meters"></a>Obtenir les tarifs des compteurs Azure

Vous pouvez obtenir le coût de chacun de ces compteurs via l’API Azure RateCard. Pour en savoir plus sur la façon de se procurer les tarifs d’un compteur Azure, consultez [Obtenir des informations sur les prix et les métadonnées des ressources utilisées dans un abonnement Azure](/previous-versions/azure/reference/mt219004(v=azure.100)).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les réservations pour Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
- [Prépayer des machines virtuelles avec des instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
- [Gérer les réservations pour Azure](manage-reserved-vm-instance.md)
- [Comprendre comment la remise de réservation est appliquée](../manage/understand-vm-reservation-charges.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).