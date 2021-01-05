---
title: Prépayer des hôtes Azure Dedicated Host pour réaliser des économies
description: Découvrez comment acheter des instances réservées Azure Dedicated Host pour réduire vos coûts de calcul.
services: virtual-machines
author: yashar
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 02/28/2020
ms.author: banders
ms.openlocfilehash: d7af95b9fe2f6d31faa239985f8e8165fd968372
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978592"
---
# <a name="save-costs-with-azure-dedicated-host-reservations"></a>Réduisez les coûts avec les réservations Azure Dedicated Host

Lorsque vous validez une instance réservée d’hôtes Azure Dedicated Host, vous pouvez économiser de l’argent. La remise de réservation est appliquée automatiquement au nombre d’hôtes dédiés en cours d’exécution qui correspondent à l’étendue et aux attributs de la réservation. Vous n’avez pas besoin d’attribuer une réservation à un hôte dédié pour obtenir les remises. Un achat d’instance réservée couvre uniquement la partie calcul de votre utilisation et inclut les coûts liés aux licences logicielles. Consultez l’article [Vue d’ensemble des hôtes Azure Dedicated Host pour machines virtuelles](./dedicated-hosts.md).

## <a name="determine-the-right-dedicated-host-sku-before-you-buy"></a>Déterminer la référence SKU appropriée de l’hôte dédié avant de l’acheter


Avant d’acheter une réservation, vous devez déterminer l’hôte dédié dont vous avez besoin. Une référence SKU est définie pour un hôte dédié représentant la série et le type de la machine virtuelle. 

Commencez par passer en revue les tailles prises en charge pour la [machine virtuelle Windows](./sizes.md) ou [Linux](./sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) afin d’identifier la série de machine virtuelle.

Ensuite, vérifiez qu’elle est prise en charge sur les hôtes Azure Dedicated Host. La page de [tarification des hôtes Azure Dedicated Host](https://aka.ms/ADHPricing) contient la liste complète des références SKU d’hôtes dédiés, les informations de l’UC et diverses options de tarification (y compris les instances réservées).

Vous pouvez trouver plusieurs références SKU prenant en charge la série de votre machine virtuelle (avec différents types). Identifiez la meilleure référence SKU en comparant la capacité de l’hôte (nombre de processeurs virtuels). Notez que vous serez en mesure d’appliquer la réservation à plusieurs références SKU d’hôtes dédiés prenant en charge la même série de machines virtuelles (par exemple, DSv3_Type1 et DSv3_Type2), mais pas sur des séries différentes de machines virtuelles (comme DSv3 et ESv3).



## <a name="purchase-restriction-considerations"></a>Considérations en matière de restriction d’achat

Les instances réservées sont disponibles pour la plupart des tailles d’hôtes dédiés, à quelques exceptions près.

Les remises de réservation ne s’appliquent pas dans les cas suivants :

- **Clouds** : les réservations ne sont pas disponibles à l’achat dans les régions Allemagne et Chine.

- **Quota insuffisant** : une réservation qui est limitée à un seul abonnement doit avoir un quota de processeurs virtuels disponible dans l’abonnement pour la nouvelle instance réservée. Par exemple, si l’abonnement cible a une limite de quota de 10 processeurs virtuels pour la série DSv3, vous ne pouvez pas acheter de réservation d’hôtes dédiés prenant en charge cette série. La vérification du quota pour les réservations inclut les machines virtuelles et les hôtes dédiés déjà déployés dans l’abonnement. Pour résoudre ce problème, vous pouvez  [créer une demande d’augmentation de quota](../azure-portal/supportability/resource-manager-core-quotas-request.md). 

- **Restrictions de capacité** : dans de rares cas, Azure limite l’achat de nouvelles réservations pour un sous-ensemble de références SKU d’hôtes dédiés, en raison d’une faible capacité dans une région donnée.

## <a name="buy-a-reservation"></a>Acheter une réservation

Vous pouvez acheter une instance réservée pour une instance Azure Dedicated Host dans le  [Portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D).

Payez la réservation  [comptant ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md). Ces conditions s’appliquent à l’achat d’une instance Azure Dedicated Host réservée :

- Vous devez avoir le rôle Propriétaire pour au moins un abonnement EA ou un abonnement assorti d’un paiement à l’utilisation.

- Pour les abonnements EA, l’option  **Ajouter des instances réservées** doit être activée dans le  [portail EA](https://ea.azure.com/). Ou, si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.

- Pour le programme CSP (Cloud Solution Provider), seuls les agents administrateurs ou les agents commerciaux peuvent acheter les réservations.

Pour acheter une instance :

1. Connectez-vous au  [portail Azure](https://portal.azure.com/).

2. Sélectionnez  **Tous les services** \> **Réservations**.

3. Sélectionnez  **Ajouter** pour acheter une nouvelle réservation, puis cliquez sur  **Hôtes dédiés**.

4. Complétez les champs obligatoires. Les instances d’Azure Dedicated Host en cours d’exécution qui correspondent aux attributs que vous avez sélectionnés peuvent bénéficier de la remise de réservation. Le nombre réel de vos instances Dedicated Host qui bénéficient de la remise dépend de l’étendue et de la quantité sélectionnées.

Si vous avez un Accord Entreprise, vous pouvez utiliser l’option  **Ajouter** pour ajouter rapidement des instances supplémentaires. L’option n’est pas disponible pour les autres types d’abonnements.

| **Champ**           | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Abonnement        | Abonnement utilisé pour payer la réservation. Les coûts de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P or MS-AZR-0148P) ou Microsoft Customer Agreement (MCA) ou un abonnement individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Les frais sont déduits du solde de l’engagement financier, si disponibles, ou facturés comme un dépassement. Pour un abonnement payable à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou facture défini sur l’abonnement. |
| Étendue               | L’étendue de la réservation peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez :                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Région              | Région Azure couverte par la réservation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Taille d’hôte dédié | Taille des instances Dedicated Host.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Terme                | Une année ou trois ans.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Quantité            | Nombre d’instances achetées au sein de la réservation. La quantité correspond au nombre d’instances Dedicated Host en cours d’exécution pouvant bénéficier de la remise.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

- **Étendue de groupe de ressources unique** : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.

- **Étendue d’abonnement unique** : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.

- **Étendue partagée** : applique la remise de réservation aux ressources correspondantes des abonnements éligibles inclus dans le contexte de facturation. Pour les clients EA, le contexte de facturation est défini par l’inscription. Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.

## <a name="usage-data-and-reservation-utilization"></a>Données d’utilisation et utilisation de la réservation

Vos données d’utilisation ont un prix effectif de zéro pour l’utilisation qui bénéficie d’une remise de réservation. Vous pouvez voir l’instance de machine virtuelle qui a reçu la remise de réservation pour chaque réservation.

Pour plus d’informations sur l’affichage des remises de réservation dans les données d’utilisation, consultez  [Comprendre l’utilisation de la réservation Azure pour l’inscription de votre entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) si vous êtes un client EA. Si vous avez un abonnement individuel, consultez  [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md).

## <a name="change-a-reservation-after-purchase"></a>Modifier une réservation après achat

Vous pouvez apporter les modifications suivantes à une réservation après achat :

- Mettez à jour l’étendue de la réservation

- Flexibilité de taille d’instance (le cas échéant)

- Propriété

Vous pouvez également fractionner une réservation en segments plus petits ou fusionner des réservations fractionnées. Aucune des modifications n’entraîne de nouvelle transaction commerciale ou de changement de la date de fin de la réservation.

Vous ne pouvez apporter directement les modifications suivantes après achat :

- Région d’une réservation existante

- SKU

- Quantité

- Duration

Toutefois, vous pouvez  *échanger* une réservation si vous souhaitez y apporter des modifications.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez  [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide,  [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir comment gérer une réservation, consultez  [Gérer les réservations Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md).

Pour plus d’informations sur les réservations Azure, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md)

- [Utilisation d’hôtes dédiés Azure](./dedicated-hosts.md)

- [Tarifs des hôtes dédiés](https://azure.microsoft.com/pricing/details/virtual-machines/dedicated-host/)

- [Gérer les réservations dans Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)

- [Comprendre comment la remise de réservation est appliquée](../cost-management-billing/manage/understand-vm-reservation-charges.md)

- [Comprendre l’utilisation d’une réservation pour un abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md)

- [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)

- [Coûts des logiciels Windows non inclus dans les réservations](../cost-management-billing/reservations/reserved-instance-windows-software-costs.md)

- [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)