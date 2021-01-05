---
title: Utilisation d’une réservation Azure pour un abonnement individuel
description: Découvrez comment analyser votre utilisation pour comprendre l’application de la réservation Azure à votre abonnement individuel avec tarifs de paiement à l’utilisation.
author: bandersmsft
ms.reviewer: yashar
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: banders
ms.openlocfilehash: 522b67e62e7ad8560bb48e8d056ca6daeeb45251
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350906"
---
# <a name="understand-azure-reservation-usage-for-your-individual-subscription-with-pay-as-you-go-rates-subscription"></a>Comprendre l’utilisation d’une réservation Azure pour votre abonnement individuel avec tarifs de paiement à l’utilisation

Pour évaluer l’utilisation d’une réservation, utilisez la [page Réservation](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=Reservations&Microsoft_Azure_Reservations=true#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade) et le fichier d’utilisation du [portail Comptes Azure](https://account.azure.com).

Si vous êtes un client avec un Contrat Entreprise, consultez [Comprendre l’utilisation des réservations pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md).

Cet article suppose que la réservation s’applique à un seul abonnement. Si la réservation s’applique à plusieurs abonnements, votre remise de réservation peut s’étendre à plusieurs fichiers CSV d’utilisation.

## <a name="usage-for-reserved-virtual-machine-instances"></a>Utilisation pour des instances de machine virtuelle réservées

Les sections suivantes supposent que vous exécutez une machine virtuelle Windows Standard_DS1_v2 dans la région USA Est et que les informations de vos instances de machine virtuelle réservées se présentent comme dans le tableau suivant :

| Champ | Valeur |
|---| :---: |
|ID de réservation |8117adfb-1d94-4675-be2b-f3c1bca808b6|
|Quantité |1|
|SKU | Standard_DS1_v2|
|Région | eastus |

La partie matérielle de la machine virtuelle est traitée, car la machine virtuelle déployée correspond aux attributs de la réservation. Pour savoir quels logiciels Windows ne sont pas couverts par l’instance de machine virtuelle réservée, consultez [Coûts des logiciels Windows dans les instances de machine virtuelle réservées Azure](reserved-instance-windows-software-costs.md).

### <a name="statement-section-of-csv-file-for-vms"></a>Section des relevés du fichier CSV pour les machines virtuelles

Cette section de votre fichier CSV montre l’utilisation totale de votre réservation. Appliquez le filtre au champ **Sous-catégorie de compteur** contenant **Réservation-** . Vous devez voir quelque chose de similaire à la capture d’écran suivante :

![Capture d’écran des détails et des frais d’utilisation d’une réservation après filtrage](./media/understand-reserved-instance-usage/billing-payg-reserved-instance-csv-statements.png)

La ligne **Instances réservées-Machine virtuelles de base** indique le nombre total d’heures couvertes par la réservation. Cette ligne affiche 0,00 dollar US, car la réservation la couvre. La ligne **Réservation-Windows Server (1 cœur)** couvre le coût des logiciels Windows.

### <a name="daily-usage-section-of-csv-file"></a>Section Utilisation quotidienne du fichier CSV

Filtrez les données sur **Informations supplémentaires**, puis tapez votre **ID de réservation**. La capture d’écran suivante affiche les champs associés à la réservation.

![Capture d’écran de l’utilisation quotidienne et des frais](./media/understand-reserved-instance-usage/billing-payg-reserved-instance-csv-details.png)

1. **L’ID de réservation** dans le champ **Infos supplémentaires** représente la réservation appliquée à la machine virtuelle.
2. Le **compteur de consommation** est l’ID du compteur de la machine virtuelle.
3. La ligne **Instances réservées-Machines virtuelles de base** dans **Sous-catégorie du compteur** correspond au coût de 0 dollar US dans la section du relevé. Le coût d’exécution de cette machine virtuelle est déjà payé par la réservation.
4. **L’ID du compteur** correspond à l’ID du compteur pour la réservation. Le coût de ce compteur est de 0 dollar US. Cet ID de compteur est indiqué pour toute machine virtuelle répondant aux critères de la remise de réservation.
5. Standard_DS1_v2 est une machine virtuelle à processeur virtuel qui est déployée sans Azure Hybrid Benefit. Ce compteur couvre donc les frais supplémentaires des logiciels Windows. Pour trouver le compteur correspondant à la machine virtuelle à 1 cœur de la série D, consultez [Coûts des logiciels Windows dans les instances de machine virtuelle réservées](reserved-instance-windows-software-costs.md). Si vous avez Azure Hybrid Benefit, ce coût supplémentaire n’est pas appliqué.

## <a name="usage-for-sql-database--cosmos-db-reservations"></a>Utilisation pour les réservations SQL Database et Cosmos DB

Les sections suivantes utilisent Azure SQL Database comme exemple pour décrire le rapport d’utilisation. Vous pouvez utiliser les mêmes étapes pour obtenir l’utilisation pour Azure Cosmos DB.

Supposez que vous exécutez une machine virtuelle SQL Database de 4e génération dans la région USA Est, et que vos informations de réservation ressemblent au contenu du tableau suivant :

| Champ | Valeur |
|---| --- |
|ID de réservation |446ec809-423d-467c-8c5c-bbd5d22906b1|
|Quantité |2|
|Produit| SQL Database 4e génération (2 cœurs)|
|Région | eastus |

### <a name="statement-section-of-csv-file"></a>Section des relevés du fichier CSV

Filtrez sur le nom de compteur **Utilisation des instances réservées** et choisissez la **catégorie du compteur** voulue : Azure SQL Database ou Azure Cosmos DB. Vous devez voir quelque chose de similaire à la capture d’écran suivante :

![Capture d’écran montrant une entrée Catégorie du compteur.](./media/understand-reserved-instance-usage/billing-payg-sql-db-reserved-capacity-csv-statements.png)

La ligne **Utilisation des instances réservées** indique le nombre total d’heures/cœur couvertes par la réservation. Le tarif est de 0 dollar US pour cette ligne puisque la réservation couvre le coût.

### <a name="detail-section-of-csv-file"></a>Section des détails du fichier CSV

Filtrez les données sur **Informations supplémentaires**, puis tapez votre **ID de réservation**. La capture d’écran suivante montre les champs associés à la réservation de capacité réservée SQL Database.

![Capture d’écran montrant les détails d’un fichier C S V pour une capacité de réserve.](./media/understand-reserved-instance-usage/billing-payg-sql-db-reserved-capacity-csv-details.png)

1. **L’ID de réservation** dans le champ **Infos supplémentaires** représente la réservation de capacité réservée SQL Database appliquée à la ressource SQL Database.
2. Le **compteur de consommation** représente l’ID du compteur de la ressource SQL Database.
3. **L’ID du compteur** est le compteur de réservation avec un coût de 0 dollar US. Le coût de ce compteur est de 0 dollar US. Cet ID de compteur est indiqué dans le fichier CSV pour toute ressource SQL Database répondant aux critères de la remise de réservation.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](save-compute-costs-reservations.md)
- [Prépayer des machines virtuelles avec des instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
- [Prépayer des ressources de calcul SQL Database avec une capacité réservée Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Gérer les réservations Azure](manage-reserved-vm-instance.md)
- [Comprendre comment la remise de réservation est appliquée](../manage/understand-vm-reservation-charges.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
- [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)