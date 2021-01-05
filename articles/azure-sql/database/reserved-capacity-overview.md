---
title: Économiser sur les coûts de calcul grâce à une capacité de réserve
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Découvrez comment acheter une capacité de réserve Azure SQL Database et SQL Managed Instance pour réduire vos coûts de calcul.
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: sstein
ms.date: 10/13/2020
ms.openlocfilehash: b4a8b809fc135a4af26fae10912890f2017153f4
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780918"
---
# <a name="save-costs-for-resources-with-reserved-capacity---azure-sql-database--sql-managed-instance"></a>Réduire les coûts des ressources grâce à une capacité de réserve – Azure SQL Database et SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)] 

Faites des économies avec Azure SQL Database et SQL Managed Instance en procédant à une réservation des ressources de calcul au lieu de payer le tarif du paiement à l’utilisation. Avec une capacité de réserve, vous prenez l’engagement d’utiliser SQL Database ou SQL Managed Instance sur une période d’un an ou de trois ans afin de bénéficier de remises importantes sur les coûts de calcul. Pour acheter une capacité de réserve, vous devez spécifier la région Azure, le type de déploiement, le niveau de performance et la durée.

Vous n’avez pas besoin d’attribuer la réservation à une base de données ou une instance managée spécifique. Les déploiements existants correspondants, qui sont déjà en cours d’exécution ou récemment déployés, bénéficient automatiquement de cet avantage. En achetant une réservation, vous vous engagez à payer les coûts de calcul pendant une durée d’un an ou de trois ans. Dès que vous achetez une réservation, les frais de calcul qui correspondent aux attributs de la réservation ne sont plus facturés au tarif du paiement à l’utilisation. 

Une réservation s’applique aux réplicas de calcul principaux et secondaires facturables, mais ne couvre pas les frais de logiciels, de mise en réseau ou de stockage associés au service. À l’issue de la période de réservation, la remise sur facturation cesse de s’appliquer et la base de données ou l’instance managée est facturée au tarif du paiement à l’utilisation. Les réservations ne sont pas renouvelées automatiquement. Pour plus d’informations sur les tarifs, consultez l’[offre de capacité de réserve](https://azure.microsoft.com/pricing/details/sql-database/managed/).

Vous pouvez acheter une capacité de réserve sur le [portail Azure](https://portal.azure.com). Payez la réservation [à l’avance ou par paiements mensuels](../../cost-management-billing/reservations/prepare-buy-reservation.md). Pour acheter une capacité réservée :

- Vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise ou individuel avec des tarifs de paiement à l’utilisation.
- Pour les abonnements Entreprise, **Add Reserved Instances** (Ajouter des instances réservées) doit être activé dans le [portal EA](https://ea.azure.com). Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement. Capacité réservée.

Pour en savoir plus sur la facturation des achats de réservation pour les clients professionnels et les clients payant à l’utilisation, consultez les articles [Comprendre l’utilisation d’une réservation Azure pour votre inscription Entreprise](../../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) et [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec paiement à l’utilisation](../../cost-management-billing/reservations/understand-reserved-instance-usage.md).

> [!NOTE]
> L’achat d’une capacité de réserve ne préalloue ni ne réserve de ressources d’infrastructure spécifiques (machines virtuelles ou nœuds) à votre usage.

## <a name="determine-correct-size-before-purchase"></a>Déterminer la taille correcte avant l’achat

La taille de la réservation doit s’appuyer sur le nombre total de calculs utilisés par la base de données ou instance managée existante ou sur le point d’être déployée dans une région spécifique et utilisant le même niveau de performance et la même génération de matériel.

Par exemple, supposez que vous exécutiez un pool élastique Gen5 à usage général de 16 vCores et deux bases de données uniques Gen5 critiques pour l’entreprise de quatre vCores. Imaginez ensuite que vous envisagiez de déployer au cours du mois suivant un pool élastique Gen5 à usage général de 16 vCores supplémentaire et un pool élastique Gen5 critique pour l’entreprise de 32 vCores. Enfin, imaginons que vous savez que ces ressources vous seront nécessaires pendant au moins 1 an. Dans ce cas, vous devez acheter une réservation de 32 (2x16) vCores pour une durée d’un an, avec l’option Usage général pour pool élastique/base de données unique - Gen5, et une réservation de 40 (2x4 + 32) vCores pour une durée d’un an, avec l’option Critique pour l’entreprise pour pool élastique/base de données unique - Gen5.

## <a name="buy-reserved-capacity"></a>Acheter une capacité réservée

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services** > **Réservations** .
3. Sélectionnez **Ajouter** , puis, dans le volet **Acheter des réservations** , choisissez **SQL Database** pour acheter une nouvelle réservation de bases de données SQL.
4. Renseignez les champs obligatoires. Les bases de données existantes de SQL Database et SQL Managed Instance qui correspondent aux attributs que vous sélectionnez peuvent prétendre à la remise de capacité de réserve. Le nombre réel de vos bases de données ou instances managées qui obtiennent la remise dépend de l’étendue et de la quantité sélectionnées.

    ![Capture d’écran avant de soumettre l’achat de la capacité réservée](./media/reserved-capacity-overview/sql-reserved-vcores-purchase.png)

    Le tableau suivant décrit les champs requis.
    
    | Champ      | Description|
    |------------|--------------|
    |Abonnement|Abonnement utilisé pour payer la réservation de capacité. Les coûts initiaux de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être un contrat Entreprise (numéro de l’offre MS-AZR-0017P ou MS-AZR-0148P) ou un contrat individuel avec paiement à l’utilisation (numéro de l’offre MS-AZR-0003P ou MS-AZR-0023P). Pour un abonnement Entreprise, les frais sont déduits du solde d’engagement monétaire de l’inscription ou facturés comme un dépassement. Pour un abonnement individuel avec paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture sur l’abonnement.|
    |Étendue       |L’étendue de la réservation vCore peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez <br/><br/>**Partagée** , la remise de réservation vCore est appliquée à la base de données ou instance managée en cours d’exécution dans tous les abonnements de votre contexte de facturation. Pour les clients Entreprise, l'étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l'inscription. Pour les clients Paiement à l’utilisation, l’étendue partagée correspond à tous les abonnements Paiement à l’utilisation créés par l’administrateur de compte.<br/><br/>**Abonnement unique** , la remise de réservation vCore est appliquée à la base de données ou instance managée incluse dans cet abonnement. <br/><br/>**Groupe de ressources unique** , la remise de réservation est appliquée à la base de données ou instance managée incluse dans l’abonnement sélectionné et dans le groupe de ressources sélectionné au sein de cet abonnement.|
    |Région      |Région Azure couverte par la réservation de capacité.|
    |Type de déploiement|Type de ressource SQL pour laquelle vous voulez acheter la réservation.|
    |Niveau de performances|Niveau de service pour les bases de données ou instances managées. |
    |Terme        |Une année ou trois ans.|
    |Quantité    |Quantité de ressources de calcul achetées au sein de la réservation de capacité. La quantité correspond au nombre de vCores de la région Azure et du niveau de performance sélectionnés qui est réservé et qui bénéficie de la remise de facturation. Par exemple, si vous exécutez ou envisagez d’exécuter plusieurs bases de données avec la capacité de calcul totale de 16 vCores Gen5 dans la région USA Est, vous devez spécifier une quantité de 16 pour optimiser l’avantage pour toutes les bases de données. |

1. Étudiez de plus près le coût de la réservation de capacité dans la section **Coûts** .
1. Sélectionnez **Achat** .
1. Sélectionnez **Afficher cette réservation** pour connaître l’état de votre achat.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="vcore-size-flexibility"></a>Flexibilité de la taille vCore

La flexibilité de la taille vCore vous permet de vous mettre à l’échelle au sein d’un niveau de performances et d’une région, sans perdre le bénéfice de la capacité réservée. La capacité de réserve vous donne également la possibilité de déplacer temporairement vos bases de données chaudes vers et à partir des pools élastiques (au sein du même niveau de performances et de la même région) dans le cadre de vos opérations normales, sans perdre l’avantage de la capacité de réserve. En conservant une mémoire tampon non appliquée dans votre réservation, vous pouvez gérer efficacement les pics de performances sans dépasser votre budget.

## <a name="limitation"></a>Limitation

Vous ne pouvez pas réserver de bases de données basées sur des DTU (De base, Standard ou Premium) dans SQL Database.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

La remise de réservation vCore est appliquée automatiquement au nombre de bases de données ou d’instances managées qui correspondent à l’étendue et aux attributs de la réservation de capacité. Vous pouvez mettre à jour l’étendue de la réservation de capacité par le biais du [portail Azure](https://portal.azure.com), de PowerShell, d’Azure CLI ou de l’API.

Pour savoir comment gérer la réservation de capacité, consultez [Gérer la capacité de réserve](../../cost-management-billing/reservations/manage-reserved-vm-instance.md).

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Gérer les réservations Azure](../../cost-management-billing/reservations/manage-reserved-vm-instance.md)
- [Comprendre la remise sur réservation Azure](../../cost-management-billing/reservations/understand-reservation-charges.md)
- [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../../cost-management-billing/reservations/understand-reserved-instance-usage.md)
- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
- [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)