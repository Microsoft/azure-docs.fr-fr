---
title: Processus d’application d’une remise pré-achat Azure Databricks
description: Découvrez comment s’applique une remise pré-achat Azure Databricks. Vous pouvez utiliser ces Databricks à tout moment pendant la période d’achat.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: f5a2102cedfa2b8631c1eb7ff6a3ff2452d315e3
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88690525"
---
# <a name="how-azure-databricks-pre-purchase-discount-is-applied"></a>Processus d’application d’une remise pré-achat Azure Databricks

Vous pouvez utiliser des unités de validation Azure Databricks (DBCU) pré-achetées à tout moment au cours de la période d’achat. Toutes les Azure Databricks utilisent automatiquement les déductions des DBCU pré-achetées.

Contrairement aux machines virtuelles, les unités Pré-achetées n’expirent pas sur une base horaire. Vous pouvez les utiliser à tout moment pendant la durée de l’achat. Pour obtenir les remises pré-achat, vous n’avez pas besoin de redéployer ou d’affecter un plan pré-installé à vos espaces de travail Azure Databricks pour l’utilisation.

La remise pré-achat s’applique uniquement à l’utilisation de l’unité de Azure Databricks (DBU). Les autres frais liés au calcul, au stockage et à la mise en réseau sont facturés séparément.

## <a name="pre-purchase-discount-application"></a>Application d’une remise pré-achat

Le pré-achat de Databricks s’applique à toutes les charges de travail et les niveaux Databricks. Vous pouvez considérer le pré-achat comme un pool d’unités de validation Databricks pré-payé. L’utilisation est déduite du pool, quelle que soit la charge de travail ou le niveau. L’utilisation est déduite dans le rapport suivant :

| **Charge de travail** | **Rapport d’application DBU : niveau Standard** | **Rapport d’application DBU : niveau Premium** |
| --- | --- | --- |
| Analytique des données | 0.4 | 0,55 |
| Engineering données | 0.15 | 0.30 |
| Engineering données allégé | 0,07 | 0,22 |

Par exemple, lorsqu’une quantité d’analytique données (niveau Standard) est utilisée, les unités de validation Databricks pré-achetées sont déduites de 0,4 unités. Lorsqu’une quantité Engineering données allégé (niveau Standard) est utilisée, les unités de validation Databricks pré-achetées sont déduites de 0,07 unités

## <a name="determine-plan-use"></a>Déterminer l’utilisation du plan

Pour déterminer l’utilisation de votre plan DBCU, dans le Portail Microsoft Azure, accédez à **Réservations**, puis cliquez sur le plan Databricks acheté. L’utilisation à ce jour est indiquée avec les unités restantes. Pour plus d’informations sur la détermination de l’utilisation de votre réservation, consultez la section [Utilisation de la réservation](reservation-apis.md#see-reservation-usage).

## <a name="how-discount-application-shows-in-usage-data"></a>Affichage de l’application de remise dans les données d’utilisation

Lorsque la remise pré-achat s’applique à votre utilisation de Databricks, les frais à la demande apparaissent comme nuls dans les données d’utilisation. Pour plus d’informations sur les coûts de réservation et l’utilisation, voir [Obtenir les données d’utilisation et de coûts de la réservation pour les Contrats Entreprise](understand-reserved-instance-usage-ea.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](manage-reserved-vm-instance.md).
- Pour en savoir plus sur le préachat d’Azure Databricks pour faire des économies, veuillez consulter [Optimiser les coûts de Azure Databricks avec un pré-achat](prepay-databricks-reserved-capacity.md).
- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
  - [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
  - [Gérer les réservations dans Azure](manage-reserved-vm-instance.md)
  - [Comprendre l’utilisation d’une réservation pour un abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
  - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
