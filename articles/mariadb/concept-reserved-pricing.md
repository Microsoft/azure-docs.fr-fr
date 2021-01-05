---
title: Prépayer les ressources de calcul avec une capacité de réserve - Azure Database for MariaDB
description: Prépayer les ressources de calcul Azure Database for MariaDB avec une capacité de réserve
author: mksuni
ms.author: sumuth
ms.service: mariadb
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: c28abc882635f7cd0815952fe0ae183bef7138d2
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241614"
---
# <a name="prepay-for-azure-database-for-mariadb-compute-resources-with-reserved-capacity"></a>Prépayer les ressources de calcul Azure Database for MariaDB avec une capacité de réserve

Faites des économies avec Azure Database for MariaDB en utilisant le prépaiement de ressources de calcul au lieu d’appliquer le tarif du paiement à l’utilisation. Avec une capacité de réserve Azure Database for MariaDB, vous prenez un engagement initial pour les serveurs MariaDB, sur une période d’un à trois ans, afin de bénéficier d’une remise importante sur les coûts de calcul. Pour acheter une capacité de réserve Azure Database for MariaDB, vous devez spécifier la région Azure, le type de déploiement, le niveau de performance et le terme. </br>

Vous n’avez pas besoin d’attribuer la réservation à des serveurs Azure Database for MariaDB spécifiques. Un serveur Azure Database for MariaDB déjà en cours d’exécution ou plusieurs récemment déployés, bénéficient automatiquement du tarif réservé. En achetant une réservation, vous payez d’avance les coûts de calcul pendant une durée d’un an ou de trois ans. Dès que vous achetez une réservation, les charges de calcul Azure Database for MariaDB qui correspondent aux attributs de la réservation ne sont plus facturées au tarif du paiement à l’utilisation. Une réservation ne couvre pas les frais de logiciels, de mise en réseau ou de stockage qui sont associés au serveur de base de données MariaDB. À l’issue de la période de réservation, la remise sur facturation expire les serveurs Azure Database for MariaDB sont facturés au tarif du paiement à l’utilisation. Les réservations ne se renouvellent pas automatiquement. Pour plus d’informations sur les tarifs, consultez l’[offre de capacité de réserve Azure Database for MariaDB](https://azure.microsoft.com/pricing/details/mariadb/). </br>

Vous pouvez acheter une capacité de réserve Azure Database for MariaDB sur le [Portail Azure](https://portal.azure.com/). Payez la réservation [à l’avance ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md). Pour acheter une capacité réservée :

* Vous devez disposer du rôle Propriétaire sur au moins un abonnement Entreprise ou individuel avec des tarifs de paiement à l’utilisation.
* Pour les abonnements Entreprise, **Add Reserved Instances** (Ajouter des instances réservées) doit être activé dans le [portal EA](https://ea.azure.com/). Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
* Pour ce qui est du programme des fournisseurs de solutions cloud (CSP), seuls les agents d’administration ou les agents commerciaux peuvent acheter une capacité de réserve Azure Database for MariaDB. </br>

Plus en savoir plus sur la facturation des achats de réservation pour les clients professionnels et les clients bénéficiant du paiement à l’utilisation, consultez les articles [Comprendre l’utilisation d’une réservation Azure pour votre inscription Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) et [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md).


## <a name="determine-the-right-server-size-before-purchase"></a>Déterminer la bonne taille de serveur avant l’achat

La taille de la réservation doit s’appuyer sur le nombre total de calculs utilisés par les instances de bases de données existantes ou sur le point d’être déployées dans une région spécifique et utilisant le même niveau de performance et la même génération de matériel.</br>

Par exemple, supposons que vous exécutez un usage général ; une base de données MariaDB de 32 vCore Gen5 et deux bases de données MariaDB 16 vCore Gen5 à mémoire optimisée. Admettons également que vous envisagez de déployer au cours du mois prochain un autre serveur de base de données Gen5 32 vCore à usage général et un serveur de base de données Gen5 16 vCore à mémoire optimisée. Imaginons que vous savez que ces ressources vous seront nécessaires pendant au moins 1 an. Dans ce cas, vous devez acheter une réservation de 64 (2x32) vCores pour une durée d’un an, pour un usage général de bases de données uniques - Gen5 et une réservation de 48 (2x16 + 16) vCores pour une durée d’un an pour une base de données unique à mémoire optimisée - Gen5


## <a name="buy-azure-database-for-mariadb-reserved-capacity"></a>Acheter une capacité de réserve Azure Database for MariaDB

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sélectionnez **Tous les services** > **Réservations**.
3.  Sélectionnez **Ajouter** , puis dans le volet Acheter des réservations, sélectionnez **Azure Database for MariaDB** pour acheter une nouvelle réservation pour vos bases de données MariaDB.
4.  Renseignez les champs obligatoires. Les bases de données existantes ou nouvelles, correspondant aux attributs que vous sélectionnez, peuvent prétendre à la remise de capacité réservée. Le nombre réel de vos serveurs Azure Database for MariaDB qui obtiennent la remise dépend de l’étendue et de la quantité sélectionnées.


![Aperçu de la tarification réservée](media/concepts-reserved-pricing/mariadb-reserved-price.png)


Le tableau suivant décrit les champs requis.

| Champ | Description |
| :------------ | :------- |
| Abonnement   | L’abonnement utilisé pour payer la réservation de capacité réservée Azure Database for MariaDB. Les coûts initiaux de la réservation de capacité réservée Azure Database for MariaDB sont facturés selon le mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P ou MS-AZR-0148p) ou un accord individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Pour un abonnement Entreprise, les frais sont déduits du solde d’engagement monétaire de l’inscription ou facturés comme un dépassement. Pour un abonnement individuel avec paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture sur l’abonnement.
| Étendue | L’étendue de la réservation vCore peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez : </br></br> **Partagé** : la remise de réservation vCore est appliquée aux serveurs Azure Database for MariaDB en cours d’exécution dans tous les abonnements de votre contexte de facturation. Pour les clients Entreprise, l'étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l'inscription. Pour les clients Paiement à l’utilisation, l’étendue partagée correspond à tous les abonnements Paiement à l’utilisation créés par l’administrateur de compte.</br></br> **Abonnement unique** : la remise de réservation vCore est appliquée aux serveurs Azure Database for MariaDB inclus dans cet abonnement. </br></br> **Groupe de ressources unique** : la remise de réservation est appliquée aux serveurs Azure Database for MariaDB inclus dans l’abonnement sélectionné et dans le groupe de ressources sélectionné dans cet abonnement.
| Région | La région Azure couverte par la réservation de capacité réservée Azure Database for MariaDB.
| Type de déploiement | Le type de ressource Azure Database for MariaDB pour laquelle vous voulez acheter la réservation.
| Niveau de performances | Le niveau de service des serveurs Azure Database for MariaDB.
| Terme | Une année
| Quantité | La quantité de ressources de calcul achetée au sein de la réservation de capacité réservée Azure Database for MariaDB. La quantité correspond au nombre de vCores de la région Azure et du niveau de performance sélectionnés qui est réservé et qui bénéficie de la remise de facturation. Par exemple, si vous exécutez ou envisagez d’exécuter des serveurs Azure Database for MariaDB avec la capacité de calcul totale de 16 vCores Gen5 dans la région USA Est, vous devez spécifier une quantité de 16 pour optimiser l’avantage pour tous les serveurs.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="vcore-size-flexibility"></a>Flexibilité de la taille vCore

La flexibilité de la taille vCore vous permet de vous mettre à l’échelle au sein d’un niveau de performances et d’une région, sans perdre le bénéfice de la capacité réservée. 

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

La remise de réservation vCore est appliquée automatiquement au nombre de serveurs Azure Database for MariaDB qui correspondent à l’étendue et aux attributs de la réservation de capacité réservée Azure Database pour MariaDB. Vous pouvez mettre à jour l’étendue de la réservation de capacité réservée Azure Database for MariaDB par le biais du Portail Azure, de PowerShell, de CLI ou de l’API. </br></br>
Pour savoir comment gérer la capacité réservée d’Azure Database pour MariaDB, consultez Gérer la capacité réservée d’Azure Database for MariaDB.

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

* [Qu’est-ce que des réservations Azure](../cost-management-billing/reservations/save-compute-costs-reservations.md) ?
* [Gérer les réservations Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)
* [Comprendre la remise sur réservation Azure](../cost-management-billing/reservations/understand-reservation-charges.md)
* [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reservation-charges-mariadb.md)
* [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
* [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)