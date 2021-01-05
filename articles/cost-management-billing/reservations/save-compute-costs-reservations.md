---
title: Qu’est-ce qu’une réservation Azure ?
description: Découvrez les réservations et tarifs Azure pour économiser sur vos instances réservées de machines virtuelles, les bases de données SQL, Azure Cosmos DB et les coûts d’autres ressources.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: overview
ms.date: 12/15/2020
ms.author: banders
ms.openlocfilehash: b79bde204d82b069b3314855a45a680b863c3f69
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560489"
---
# <a name="what-are-azure-reservations"></a>Qu’est-ce qu’une réservation Azure ?

Les réservations Azure vous permettent d’économiser en vous engageant sur des plans d’un ou trois ans pour plusieurs produits. L’engagement vous permet d’obtenir une remise sur les ressources que vous utilisez. Des réservations peuvent réduire sensiblement les coûts de ressources, jusqu’à hauteur de 72 % sur les prix de paiement à l’utilisation. Des réservations permettent de bénéficier d’une remise sur la facturation et n’ont aucune incidence sur l’état de runtime de vos ressources. Quand vous achetez une réservation, la remise s’applique automatiquement aux ressources correspondantes.

Vous pouvez payer une réservation à l’avance ou tous les mois. Une même réservation avec paiement initial et avec paiements mensuels a le même coût total : vous ne payez pas de frais supplémentaires si vous optez pour le paiement mensuel. Le paiement mensuel est disponible pour les réservations Azure, et non pour les produits tiers.

Vous pouvez acheter une réservation sur le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/ReservationsBrowseBlade).

## <a name="why-buy-a-reservation"></a>Pourquoi acheter une réservation ?

Si vous utilisez les ressources d’une façon systématique adaptée aux réservations, l’achat d’une réservation vous offre l’opportunité de réduire vos coûts. Par exemple, si vous exécutez en permanence des instances d’un service sans réservation, vous êtes facturé au tarif du paiement à l’utilisation. Quand vous achetez une réservation, vous bénéficiez immédiatement de la remise sur réservation. Les ressources ne sont plus facturées au tarif du paiement à l’utilisation.

## <a name="how-reservation-discount-is-applied"></a>Comment la remise de réservation est-elle appliquée ?

Après l’achat de la réservation, la remise sur réservation s’applique automatiquement à l’utilisation de ressources correspondant aux attributs que vous avez sélectionnés au moment de l’achat. Les attributs incluent la référence SKU, les régions (le cas échéant) et l’étendue. L’étendue de réservation détermine où les économies de la réservation s’appliquent.

Pour plus d’informations sur la façon dont la remise est appliquée, consultez [Application de la remise sur les instances réservées](reservation-discount-application.md).

Pour plus d’informations sur le fonctionnement de l’étendue de réservation, consultez [Réservations d’étendue](prepare-buy-reservation.md#scope-reservations).

## <a name="determine-what-to-purchase"></a>Déterminer ce qu’il faut acheter 

Toutes les réservations s’appliquent sur une base horaire sauf pour Azure Databricks. Pensez à acheter des réservations en fonction de votre utilisation régulière de base. Vous pouvez déterminer la réservation à acheter en analysant vos données d’utilisation ou en utilisant des recommandations de réservation. Vous trouverez des recommandations dans :

- Azure Advisor (machines virtuelles uniquement)
- Expérience d’achat de réservations dans le portail Azure.
- Application Power BI Cost Management
- API 

Pour plus d’informations, consultez [Déterminer la réservation à acheter](determine-reservation-purchase.md). 

## <a name="buying-a-reservation"></a>Achat d’une réservation 

Vous pouvez acheter des réservations à partir du portail Azure, d’API, de PowerShell et de l’interface CLI. 

Accédez au [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/Docs) pour effectuer des achats.

Pour plus d’informations, consultez [Acheter une réservation](prepare-buy-reservation.md).

## <a name="how-is-a-reservation-billed"></a>Comment une réservation est-elle facturée ? 

La réservation est facturée selon le mode de paiement associé à l’abonnement. Le coût de la réservation est déduit de votre solde d’engagement financier, le cas échéant. Lorsque ce solde ne couvre pas le coût des réservations, le dépassement vous est facturé. Si vous avez souscrit un abonnement à partir d’un forfait individuel avec paiement à l’utilisation, la carte de crédit associée à votre compte est facturée immédiatement pour les achats en paiement initial. Les paiements mensuels apparaissent sur votre facture et votre carte de crédit est facturée chaque mois. Lorsque vous réglez sur facture, les frais sont portés sur votre prochaine facture. 

## <a name="who-can-manage-a-reservation-by-default"></a>Qui peut gérer une réservation par défaut

Par défaut, les utilisateurs suivants peuvent voir et gérer des réservations :

- La personne qui achète une réservation et l’administrateur de compte de l’abonnement de facturation utilisé pour acheter la réservation sont ajoutés à l’ordre de réservation.
- Les administrateurs de facturation de l’Accord Entreprise et du Contrat client Microsoft.

Pour permettre à d’autres personnes de gérer des réservations, consultez [Gérer les réservations pour les ressources Azure](manage-reserved-vm-instance.md).

## <a name="get-reservation-details-and-utilization-after-purchase"></a>Obtenir les détails de la réservation et les données d’utilisation après l’achat

Si vous êtes autorisé à voir la réservation, vous pouvez examiner ses données d’utilisation dans le portail Azure. Vous pouvez également obtenir les données en utilisant des API. 

Pour plus d’informations sur la façon de voir des réservations dans le portail Azure, consultez [Afficher des réservations dans le portail Azure](view-reservations.md) 

## <a name="manage-reservations-after-purchase"></a>Gérer les réservations après l’achat 

Après avoir acheté une réservation Azure, vous pouvez mettre à jour l’étendue pour appliquer la réservation à un autre abonnement, modifier les personnes autorisées à gérer la réservation, diviser une réservation en parties plus petites ou changer la flexibilité de la taille d’instance. 

Pour plus d’informations, consultez [Gérer les réservations pour les ressources Azure](manage-reserved-vm-instance.md). 

## <a name="flexibility-with-azure-reservations"></a>Flexibilité des réservations Azure

Les réservations Azure offrent la flexibilité nécessaire pour répondre à vos besoins en constante évolution. Vous pouvez échanger une réservation pour une autre réservation du même type. Vous pouvez également obtenir le remboursement d’une réservation (jusqu’à 50 000 USD sur une période glissante de 12 mois) si vous n’en avez plus besoin. La limite maximale du remboursement s’applique à toutes les réservations dans le cadre de votre contrat avec Microsoft.

Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](exchange-and-refund-azure-reservations.md).


## <a name="charges-covered-by-reservation"></a>Frais couverts par la réservation

- **Instance de machine virtuelle réservée** : une réservation couvre uniquement les coûts de calcul de machine virtuelle et de services cloud. Une réservation ne couvre pas les frais de logiciels, de réseau, de stockage ou liés à Windows.
- **Capacité réservée de stockage Azure** : une réservation couvre la capacité de stockage pour les comptes de stockage standard pour le stockage d’objets blob ou le stockage Azure Data Lake Gen2. La réservation ne couvre pas les taux de bande passante ou de transaction.
- **La capacité réservée Azure Cosmos DB** : une réservation couvre le débit provisionné pour vos ressources. Elle ne couvre pas les frais de stockage et de réseau.
- **vCore réservé SQL Database** : couvre à la fois SQL Managed Instance et les pools élastiques/bases de données uniques SQL Database. seuls les coûts de calcul sont inclus dans une réservation. La licence SQL est facturée séparément. 
- **Azure Synapse Analytics** : une réservation couvre l’utilisation de cDWU. Elle ne couvre pas les frais de stockage ou de réseau associés à l’utilisation d’Azure Synapse Analytics.
- **Azure Databricks** : une réservation couvre uniquement l’utilisation de DBU. Les autres frais, comme le calcul, le stockage et le réseau, sont facturés séparément.
- **Frais de timbre App service** : Une réservation couvre l’utilisation des timbres. Elle ne s’applique pas aux workers, si bien que toutes les autres ressources associées aux timbres sont facturées séparément.
- **Azure Database pour MySQL** : seuls les coûts de calcul sont inclus dans une réservation. Une réservation ne couvre pas les frais de logiciels, de réseau ou de stockage qui sont associés au serveur de base de données MySQL.
- **Azure Database pour PostgreSQL** : seuls les coûts de calcul sont inclus dans une réservation. Une réservation ne couvre pas les frais de logiciels, de réseau ou de stockage qui sont associés aux serveurs de base de données PostgreSQL.
- **Azure Database pour MariaDB** : seuls les coûts de calcul sont inclus dans une réservation. Une réservation ne couvre pas les frais de logiciels, de réseau ou de stockage qui sont associés au serveur de base de données MariaDB.
- **Azure Data Explorer** : une réservation couvre les frais de majoration. Une réservation ne s’applique pas aux frais de calcul, de stockage ou de réseau associés à l’utilisation des clusters.
- **Azure Cache pour Redis** : seuls les coûts de calcul sont inclus dans une réservation. Une réservation ne couvre pas les frais de stockage ou de réseau associés aux instances de cache Redis.
- **Azure Dedicated Host** : seuls les coûts de calcul sont inclus avec l’hôte dédié.
- **Réservations de Stockage sur disque Azure** : une réservation couvre uniquement les disques SSD Premium de taille P30 ou supérieure. Elle ne couvre pas les autres types de disque ou les tailles inférieures à P30.

Abonnements logiciels :

- **SUSE Linux** : une réservation couvre les coûts de l’abonnement logiciel. Les remises s’appliquent uniquement aux compteurs SUSE et non à l’utilisation de la machine virtuelle.
- **Offres Red Hat** : une réservation couvre les coûts de l’abonnement logiciel. Les remises s’appliquent uniquement aux compteurs RedHat et non à l’utilisation de la machine virtuelle.
- **Azure VMware Solution by CloudSimple** : une réservation couvre les nœuds VMware CloudSimple. Des coûts logiciels supplémentaires sont toujours facturés.
- **Azure Red Hat OpenShift** : une réservation s’applique aux coûts OpenShift, et non aux coûts d’infrastructure Azure.

Pour les machines virtuelles Windows et SQL Database, la remise sur réservation ne s’applique pas aux coûts logiciels. Vous pouvez couvrir les coûts de licence avec [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
    - [Gérer les réservations Azure](manage-reserved-vm-instance.md)
    - [Comprendre l’utilisation d’une réservation pour votre abonnement avec paiement à l’utilisation](understand-reserved-instance-usage.md)
    - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](understand-reserved-instance-usage-ea.md)
    - [Coûts des logiciels Windows non inclus dans les réservations](reserved-instance-windows-software-costs.md)
    - [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)

- En savoir plus sur les réservations de plans de service :
    - [Machines virtuelles avec instances de machines virtuelles réservées Azure](../../virtual-machines/prepay-reserved-vm-instances.md)
    - [Ressources Azure Cosmos DB avec capacité réservée Azure Cosmos DB](../../cosmos-db/cosmos-db-reserved-capacity.md)
    - [Ressources de calcul SQL Database avec une capacité réservée Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
    - [Ressources de calcul Azure Cache pour Redis avec une capacité réservée Azure Cache pour Redis](../../azure-cache-for-redis/cache-reserved-pricing.md) En savoir plus sur les réservations pour les offres logicielles :
    - [Offres logicielles Red Hat dans Réservations Azure](../../virtual-machines/linux/prepay-suse-software-charges.md)
    - [Offres logicielles SUSE dans Réservations Azure](../../virtual-machines/linux/prepay-suse-software-charges.md)
