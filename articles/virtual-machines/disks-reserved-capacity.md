---
title: Optimiser les coûts de Stockage sur disque Azure avec des réservations
description: Apprenez-en davantage sur l’achat de réservations Stockage sur disque Azure pour réduire les coûts liés aux disques managés SSD Premium.
author: roygara
ms.author: rogarana
ms.date: 01/30/2020
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: e3642a621253b79fdfcdd8cb2b648dee534c6758
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91968683"
---
# <a name="reduce-costs-with-azure-disks-reservation"></a>Réduire les coûts avec la réservation de disques Azure

Économisez sur votre utilisation du Stockage sur disque Azure avec une capacité de réserve. Des réservations de Stockage sur disque Azure associées au service Azure Reserved Virtual Machine Instances vous permettent de réduire le coût total de vos machines virtuelles. La remise sur la réservation est appliquée automatiquement aux disques correspondants dans l’étendue de réservation sélectionnée. En raison de cette application automatique, vous n’avez pas besoin d’attribuer une réservation à un disque managé pour bénéficier les remises.

Les remises sont appliquées toutes les heures en fonction de l’utilisation du disque. La capacité de réserve inutilisée n’est pas reportée. Les remises sur réservation de Stockage sur disque Azure ne s’appliquent pas aux disques non managés, aux disques Ultra ou à la consommation d’objets blobs de pages.

## <a name="determine-your-storage-needs"></a>Déterminer vos besoins en stockage

Avant d’acheter une réservation, déterminez vos besoins en matière de stockage. Actuellement, les réservations de Stockage sur disque Azure sont disponibles uniquement pour des références SKU de disques SSD Premium Azure. La référence SKU d’un disque SSD Premium détermine la taille et les performances du disque.

Lorsque vous déterminez vos besoins de stockage, ne pensez pas aux disques en vous basant uniquement sur la capacité. Par exemple, vous ne pouvez pas avoir de réservation pour un disque P40 et l’utiliser pour payer deux disques P30 plus petits. Lors de l’achat d’une réservation, vous achetez celle-ci uniquement pour le nombre total de disques par référence SKU.

Une réservation de disque est effectuée par référence SKU de disque. Par conséquent, la consommation de la réservation est basée sur l’unité des références SKU de disque et non sur la taille fournie.

Par exemple, supposons que vous réservez un disque P40 avec 2 Tio de capacité de stockage approvisionnée. Supposons également que vous n’allouez que deux disques P30. Dans ce cas, la réservation du disque P40 ne compte pas pour la consommation des disques P30 pour lesquels vous êtes facturé au tarif de paiement à l’utilisation.
<br/>
<br/>
[!INCLUDE [disk-storage-premium-ssd-sizes](../../includes/disk-storage-premium-ssd-sizes.md)]

## <a name="purchase-considerations"></a>Considérations relatives à l’achat

Nous vous recommandons de suivre les pratiques suivantes lorsque vous envisagez d’acheter une réservation de disque :

- Pour faciliter la détermination des réservations à acheter, vous devez analyser vos informations d’utilisation. Veillez à suivre l’utilisation des SKU de disque plutôt que la capacité de disque approvisionnée ou utilisée.
- Examinez votre réservation de disque en même temps que votre réservation de machine virtuelle. Pour réaliser une économie maximale, nous vous recommandons vivement d’effectuer des réservations tant pour l’utilisation des machines virtuelles que pour l’utilisation des disques. Vous pouvez commencer par déterminer la réservation de machine virtuelle appropriée, puis évaluer la réservation de disque. En règle générale, vous disposez d’une configuration standard pour chacune de vos charges de travail. Par exemple, un serveur SQL Server peut avoir deux disques de données P40 et un disque de système d’exploitation P30.
  
  Cet type de configuration peut vous aider à déterminer la quantité réservée à acheter. Cette approche peut simplifier le processus d’évaluation et vous assurer de disposer d’un plan aligné tant pour votre machine virtuelle que pour vos disques. Le plan inclut des considérations telles que les abonnements ou les régions.

## <a name="purchase-restrictions"></a>Restrictions d’achat

Les remises sur réservation sont actuellement indisponibles pour les composants suivants :

- Disques non managés ou objets blobs de pages.
- Disques durs SSD standard ou lecteurs de disque dur (HDD) standard.
- Références (SKU) de SSD Premium inférieures à P30 : Références (SKU) de SSD P1, P2, P3, P4, P6, P10, P15 et P20.
- Disques dans les régions Azure Government, Azure Allemagne ou Azure Chine.

Dans de rares cas, Azure limite l’achat de nouvelles réservations à un sous-ensemble de références SKU de disque, en raison d’une faible capacité dans une région donnée.

## <a name="buy-a-disk-reservation"></a>Acheter une réservation de disque

Vous pouvez acheter des réservations de Stockage sur disque Azure via le [portail Azure](https://portal.azure.com/). Vous pouvez payer la réservation au comptant ou en plusieurs mensualités. Pour plus d’informations sur l’achat par mensualités, consultez [Acheter des réservations par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md#buy-reservations-with-monthly-payments).

Pour acheter une capacité de réserve, procédez comme suit :

1. Accédez au volet [Acheter des réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/Browse_AddCommand) dans le portail Azure.

1. Sélectionnez **Disques managés Azure** pour acheter une réservation.

    ![Volet pour l’achat de réservations](media/disks-reserved-capacity/disks-reserved-purchase-reservation.png) 

1. Spécifiez les valeurs requises décrites dans le tableau suivant :

   |Élément  |Description  |
   |---------|---------|
   |**Étendue**   |  Nombre d’abonnements pouvant bénéficier de l’avantage de facturation associé à la réservation. Celle valeur spécifie également la manière dont la réservation est appliquée à des abonnements spécifiques. <br/><br/> Si vous sélectionnez **Partagé**, la remise de réservation est appliquée à la capacité de Stockage Azure de chaque abonnement dans votre contexte de facturation. Le contexte de facturation est basé sur la façon dont vous vous êtes inscrit dans Azure. Pour les clients Entreprise, l'étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l'inscription. Pour les clients avec paiement à l’utilisation, l’étendue partagée comprend tous les abonnements individuels avec des tarifs de paiement à l’utilisation créés par l’administrateur de compte.  <br/><br/>  Si vous sélectionnez **Abonnement unique**, la remise de réservation est appliquée à la capacité Stockage Azure de l’abonnement sélectionné. <br/><br/> Si vous sélectionnez **Groupe de ressources unique**, la remise de réservation est appliquée à la capacité de Stockage Azure de l’abonnement sélectionné et du groupe de ressources sélectionné dans cet abonnement. <br/><br/> Vous pouvez changer l’étendue de la réservation après l’achat de la réservation.  |
   |**Abonnement**  | Abonnement que vous utilisez pour payer la réservation de Stockage Azure. Les coûts sont facturés selon le mode de paiement défini sur l’abonnement sélectionné. L’abonnement doit être de l’un des types suivants :<br/><ul><li> Accord Entreprise (numéros d’offre MS-AZR-0017P et MS-AZR-0148P). Pour un abonnement Entreprise, les frais sont déduits du solde de l’engagement financier de l’inscription, ou facturés comme un dépassement.</li><br/><li>Abonnement individuel avec taux de paiement à l’utilisation (numéros d’offre MS-AZR-0003P et MS-AZR-0023P). Pour un abonnement individuel avec tarifs de paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture, défini sur l’abonnement.</li></ul>    |
   | **Disques** | Référence (SKU) que vous souhaitez créer. |
   | **Région** | Région dans laquelle la réservation est en vigueur. |
   | **Périodicité de facturation** | Fréquence à laquelle le compte est facturé pour la réservation. Les options sont **Mensuel** ou **Comptant**. |

    ![Volet de sélection du produit que vous souhaitez acheter.png](media/disks-reserved-capacity/premium-ssd-reserved-purchase-selection.png)

1. Une fois que vous avez spécifié les valeurs de votre réservation, le portail Azure affiche le coût. Le Portail affiche également le pourcentage de remise par rapport à la facturation du paiement à l’utilisation. Sélectionnez **Suivant** pour accéder au volet **Acheter des réservations**.

1. Dans le volet **Acheter des réservations**, vous pouvez nommer votre réservation et sélectionner la quantité totale de réservations à faire. Le nombre de réservations est mappé au nombre de disques. Par exemple, si vous souhaitez réserver une centaine de disques, entrez la valeur **Quantité** **100**.

1. Vérifiez le coût total de la réservation.

    ![Volet Acheter des réservations](media/disks-reserved-capacity/premium-ssd-reserved-selecting-sku-total-purchase.png)

Une fois que vous avez acheté une réservation, celle-ci est automatiquement appliquée aux ressources de Stockage sur disque correspondant aux conditions de la réservation. Si vous n’avez pas encore créé de ressources de Stockage sur disque, la réservation s’applique chaque fois que vous créez une ressource correspondant aux conditions de la réservation. Dans les deux cas, la durée de la réservation démarre une fois l’achat finalisé.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations dans certaines limites. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="expiration-of-a-reservation"></a>Expiration d’une réservation

Quand une réservation expire, toute capacité de Stockage sur disque Azure utilisée en lien avec cette réservation est facturée au tarif du paiement à l’utilisation. Les réservations ne sont pas renouvelées automatiquement.

Vous recevrez une notification par e-mail 30 jours avant l’expiration de la réservation, puis à nouveau le jour de l’expiration. Pour continuer à tirer parti des économies qu’offre une réservation, renouvelez-la au plus tard à la date d’expiration.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Comprendre comment la remise de réservation est appliquée à Stockage sur disque Azure](../cost-management-billing/reservations/understand-disk-reservations.md)