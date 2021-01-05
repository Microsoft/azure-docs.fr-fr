---
title: Optimiser les coûts de stockage d’objets blob avec une capacité réservée
titleSuffix: Azure Storage
description: Découvrez comment l’achat d’une capacité de réserve Stockage Azure peut réduire les coûts sur l’objet blob de blocs et les ressources Azure Data Lake Storage Gen2.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 10/08/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: 1c957330fbd54660367fa350d6985929f0bbd3d1
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95531387"
---
# <a name="optimize-costs-for-blob-storage-with-reserved-capacity"></a>Optimiser les coûts de stockage d’objets blob avec une capacité réservée

Vous pouvez réaliser des économies sur les coûts de stockage pour les données blob grâce à une capacité de réserve Stockage Azure. La capacité de réserve Stockage Azure offre une remise sur la capacité pour les objets blob de blocs et les données Azure Data Lake Storage Gen2 dans les comptes de stockage standard lorsque vous vous engagez à une réservation pour un an ou trois ans. Une réservation offre une quantité fixe de capacité de stockage pour la durée de la réservation.

La capacité de réserve Stockage Azure peut réduire de manière significative les coûts de capacité pour les objets blob de blocs et les données Azure Data Lake Storage Gen2. Les économies réalisées dépendent de la durée de votre réservation, de la capacité totale que vous choisissez de réserver, du niveau d’accès et du type de redondance que vous avez choisis pour votre compte de stockage. La capacité de réserve permet de bénéficier d’une remise, sans influencer l’état de vos ressources Stockage Azure.

Pour plus d’informations sur la tarification de la réservation de stockage Azure, consultez [Tarification des objets blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs/) et [Tarification Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

## <a name="reservation-terms-for-azure-storage"></a>Conditions de réservation pour Stockage Azure

Les sections suivantes décrivent les conditions d’une réservation de stockage Azure.

### <a name="reservation-capacity"></a>Capacité de réserve

Vous pouvez acheter une capacité de réserve Stockage Azure en unités de 100 Tio et 1 Pio par mois pour une période d’un an ou de trois ans.

### <a name="reservation-scope"></a>Étendue de la réservation

La capacité de réserve Stockage Azure est disponible pour un seul abonnement ou plusieurs abonnements (étendue partagée). En cas de limitation à un seul abonnement, la remise de réservation est appliquée à l’abonnement sélectionné uniquement. Lorsque plusieurs abonnements sont concernés, la remise de réservation est partagée entre ces abonnements dans la facturation du client.

Lorsque vous achetez une capacité de réserve Stockage Azure, vous pouvez utiliser votre réservation à la fois pour les objets blob de blocs et les données Azure Data Lake Storage Gen2. Une réservation s’applique à votre utilisation dans le cadre de l’étendue achetée et ne peut pas se limiter à un compte de stockage, un conteneur ou un objet spécifique au sein de l’abonnement.

Une réservation de stockage Azure couvre uniquement la quantité de données stockées dans un abonnement ou un groupe de ressources partagé. Les frais relatifs à une suppression anticipée, aux opérations, à la bande passante et au transfert de données ne sont pas inclus dans la réservation. Dès que vous achetez une réservation, les frais de capacité qui correspondent aux attributs de la réservation sont facturés au tarif de la réduction plutôt qu’à celui du paiement à l’utilisation. Pour plus d’informations sur les réservations Azure, consultez [Qu’est-ce qu’une réservation Azure ?](../../cost-management-billing/reservations/save-compute-costs-reservations.md).

### <a name="supported-account-types-tiers-and-redundancy-options"></a>Types de comptes, niveaux et options de redondance pris en charge

La capacité de réserve Stockage Azure est disponible pour les ressources dans les comptes de stockage standard, notamment les comptes v2 universels (GPv2) et ceux de stockage d’objets blob.

Tous les niveaux d’accès (chaud, froid et archive) sont pris en charge pour les réservations. Pour plus d’informations sur les niveaux d’accès, consultez [Stockage Blob Azure : niveaux d’accès chaud, froid et archive](storage-blob-storage-tiers.md).

Tous les types de redondance sont pris en charge pour les réservations. Pour plus d’informations sur les options de redondance, consultez [Redondance du stockage Azure](../common/storage-redundancy.md).

> [!NOTE]
> La capacité de réserve Stockage Azure n’est pas disponible pour les comptes de stockage Premium, les comptes de stockage v1 universel (GPv1), Azure Data Lake Storage Gen1, les objets blobs de pages, le Stockage File d’attente Azure, le stockage Tables Azure ou Azure Files.  

### <a name="security-requirements-for-purchase"></a>Exigences de sécurité pour l’achat

Pour acheter une capacité de réserve :

- Vous devez disposer du rôle **Propriétaire** sur au moins un abonnement Entreprise ou individuel avec des tarifs de paiement à l’utilisation.
- Pour les abonnements Entreprise, **Ajouter des instances réservées** doit être activé dans le portal EA. Si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
- Pour le programme Fournisseur de solutions cloud, seuls des agents d’administration ou des agents commerciaux peuvent acheter une capacité réservée de Stockage Blob Azure.

## <a name="determine-required-capacity-before-purchase"></a>Déterminer la capacité requise avant l’achat

Lorsque vous achetez une réservation de stockage Azure, vous devez choisir la région, le niveau d’accès et l’option de redondance pour la réservation. Votre réservation est valide uniquement pour les données stockées dans cette région, ce niveau d’accès et ce niveau de redondance. Par exemple, supposons que vous achetez une réservation de données dans la région USA Ouest pour le niveau chaud utilisant un stockage redondant interzone (ZRS). Vous ne pouvez pas utiliser la même réservation pour des données dans la région USA Est, au niveau archive ou dans un stockage géoredondant (GRS). Toutefois, vous pouvez acheter une autre réservation pour vos autres besoins.  

Les réservations sont disponibles aujourd’hui pour les blocs de 100 Tio ou 1 Pio, avec des remises plus élevées pour les blocs de 1 Pio. Lorsque vous achetez une réservation dans le Portail Azure, il se peut que Microsoft vous suggère des recommandations en fonction de votre précédente utilisation pour vous permettre de déterminer quelle réservation acheter.

## <a name="purchase-azure-storage-reserved-capacity"></a>Acheter une capacité de réserve Stockage Azure

Vous pouvez acheter une capacité de réserve Stockage Azure par l’intermédiaire du [Portail Azure](https://portal.azure.com). Payez la réservation comptant ou par paiements mensuels. Pour plus d’informations sur l’achat par mensualités, consultez [Acheter des réservations Azure avec des paiements comptants ou mensuels](../../cost-management-billing/reservations/prepare-buy-reservation.md).

Pour obtenir de l’aide sur l’identification des conditions de réservation appropriées pour votre scénario, consultez [Comprendre la remise sur la capacité de réserve Stockage Azure](../../cost-management-billing/reservations/understand-storage-charges.md).

Pour acheter une capacité de réserve, procédez comme suit :

1. Accédez au volet [Acheter des réservations](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/Browse_AddCommand) dans le Portail Azure.  
1. Sélectionnez **Stockage Blob Azure** pour acheter une nouvelle réservation.  
1. Renseignez les champs obligatoires, comme décrit dans le tableau suivant :

    ![Capture d’écran montrant comment acheter une capacité de réserve](media/storage-blob-reserved-capacity/select-reserved-capacity.png)

   |Champ  |Description  |
   |---------|---------|
   |**Portée**   |  Indique le nombre d’abonnements pouvant bénéficier de l’avantage de facturation associé à la réservation. Elle contrôle également la manière dont la réservation est appliquée à des abonnements spécifiques. <br/><br/> Si vous sélectionnez **Partagé**, la remise de réservation est appliquée à la capacité Stockage Azure dans tous les abonnements de votre contexte de facturation. Le contexte de facturation est basé sur la façon dont vous vous êtes inscrit dans Azure. Pour les clients Entreprise, l'étendue partagée correspond à l'inscription et inclut tous les abonnements compris dans l'inscription. Pour les clients avec paiement à l’utilisation, l’étendue partagée comprend tous les abonnements individuels avec des tarifs de paiement à l’utilisation créés par l’administrateur de compte.  <br/><br/>  Si vous sélectionnez **Abonnement unique**, la remise de réservation est appliquée à la capacité Stockage Azure de l’abonnement sélectionné. <br/><br/> Si vous sélectionnez **Groupe de ressources unique**, la remise de réservation est appliquée à la capacité Stockage Azure de l’abonnement sélectionné et du groupe de ressources sélectionné dans cet abonnement. <br/><br/> Vous pouvez changer l’étendue de la réservation après l’achat de la réservation.  |
   |**Abonnement**  | Abonnement utilisé pour payer la réservation de stockage Azure. Les coûts sont facturés selon le mode de paiement défini sur l’abonnement sélectionné. L’abonnement doit être de l’un des types suivants : <br/><br/>  Contrat Entreprise (références de l’offre : MS-AZR-0017P ou MS-AZR-0148P) : Pour un abonnement Entreprise, les frais sont déduits du solde de l’engagement financier de l’inscription, ou facturés comme un dépassement. <br/><br/> Abonnement individuel avec tarifs de paiement à l’utilisation (références de l’offre : MS-AZR-0003P ou MS-AZR-0023P) : Pour un abonnement individuel avec tarifs de paiement à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou par facture, défini sur l’abonnement.    |
   | **Région** | Région dans laquelle la réservation est en vigueur. |
   | **Niveau d’accès** | Niveau d’accès pour lequel la réservation est en vigueur. Les options incluent *Chaud*, *Froid* ou *Archive*. Pour plus d’informations sur les niveaux d’accès, consultez [Stockage d’objets blob Azure : niveaux d’accès chaud, froid et archive](storage-blob-storage-tiers.md). |
   | **Redondance** | Option de redondance pour la réservation. Les options incluent *LRS*, *ZRS*, *GRS*, *GZRS*, *RA-GRS* et *RA-GZRS*. Pour plus d’informations sur les options de redondance, consultez [Redondance du stockage Azure](../common/storage-redundancy.md). |
   | **Périodicité de facturation** | Indique la fréquence à laquelle le compte est facturé pour la réservation. Les options incluent *Mensuel* ou *Comptant*. |
   | **Taille** | Quantité de capacité à réserver. |
   |**Terme**  | Une année ou trois ans.   |

1. Une fois que vous avez sélectionné les paramètres de votre réservation, le Portail Azure affiche le coût. Le Portail affiche également le pourcentage de remise par rapport à la facturation du paiement à l’utilisation.

1. Vérifiez le coût total de la réservation dans le volet **Acheter des réservations**. Vous pouvez également fournir un nom pour la réservation.

    ![Capture d’écran montrant comment acheter une réservation](media/storage-blob-reserved-capacity/purchase-reservations.png)

Une fois que vous avez acheté une réservation, elle est automatiquement appliquée à tout objet blob de blocs Stockage Azure et à toutes les ressources Azure Data Lake Storage Gen2 qui correspondent aux conditions de la réservation. Si vous n’avez pas encore créé de ressources de stockage Azure, la réservation s’appliquera chaque fois que vous créez une ressource qui y correspond. Dans les deux cas, la durée de la réservation démarre une fois l’achat finalisé.

## <a name="exchange-or-refund-a-reservation"></a>Échanger ou rembourser une réservation

Vous pouvez échanger ou rembourser une réservation avec certaines limitations. Ces limitations sont décrites dans les sections suivantes.

Pour échanger ou rembourser une réservation, accédez aux détails de la réservation dans le Portail Azure. Sélectionnez **Échange** ou **Remboursement**, puis suivez les instructions pour soumettre une demande de support. Une fois la demande traitée, Microsoft vous envoie un e-mail pour confirmer qu’elle a été traitée.

Pour plus d’informations sur les stratégies de réservations Azure, consultez [Échanges et remboursements en libre-service de réservations Azure](../../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

### <a name="exchange-a-reservation"></a>Échanger une réservation

L’échange d’une réservation vous permet de recevoir un remboursement calculé au prorata en fonction de la partie inutilisée de la réservation. Vous pouvez ensuite appliquer le remboursement au prix d’achat d’une nouvelle réservation de stockage Azure.

Il n’y a pas de limite au nombre d’échanges que vous pouvez faire. De plus, un échange n’occasionne aucuns frais. La nouvelle réservation que vous achetez doit être supérieure ou égale à la valeur du crédit calculé au prorata de la réservation d’origine. Une réservation de stockage Azure peut être échangée uniquement pour une autre réservation de stockage Azure, et non pour une réservation dédiée à un autre service Azure.

### <a name="refund-a-reservation"></a>Rembourser une réservation

Vous pouvez annuler une réservation de stockage Azure à tout moment. Lorsque vous annulez, vous recevrez un remboursement calculé au prorata en fonction de la durée restante de la réservation, moins des frais de résiliation anticipé de 12 %. Le remboursement maximal par année est de 50 000 $.

L’annulation d’une réservation met immédiatement fin à la réservation et retourne les mois restants à Microsoft. Le solde restant calculé au prorata, moins les frais, est crédité sur votre moyen de paiement d’origine.

## <a name="expiration-of-a-reservation"></a>Expiration d’une réservation

Lorsqu’une réservation expire, toute capacité de stockage Azure que vous utilisez dans le cadre de cette réservation est facturée au tarif du paiement à l’utilisation. Les réservations ne sont pas renouvelées automatiquement.

Vous recevrez une notification par e-mail 30 jours avant l’expiration de la réservation, puis à nouveau au moment de l’expiration. Pour continuer à tirer parti des économies qu’offre une réservation, renouvelez-la au plus tard à la date d’expiration.

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Nous contacter

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Qu’est-ce qu’une réservation Azure ?](../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Comprendre comment la remise de réservation est appliquée à Stockage Azure](../../cost-management-billing/reservations/understand-storage-charges.md)