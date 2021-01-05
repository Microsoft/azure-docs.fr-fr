---
title: Prépayer les machines virtuelles Azure pour réaliser des économies
description: Découvrez comment acheter des instances Azure Reserved Virtual Machine Instances pour réduire vos coûts de calcul.
author: vikramdesai01
manager: vikramdesai01
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 10/30/2017
ms.author: vikdesai
ms.openlocfilehash: e513bdaa7efa2f1d0f18ff3548d9abfdc5e085d7
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007396"
---
# <a name="save-costs-with-azure-reserved-vm-instances"></a>Économiser grâce aux instances de machines virtuelles Azure réservées



Vous pouvez faire des économies en optant pour une instance de machine virtuelle réservée Azure. La remise de réservation est appliquée automatiquement au nombre de machines virtuelles en cours d’exécution qui correspondent à l’étendue et aux attributs de la réservation. Vous n’avez pas besoin d’attribuer une réservation à une machine virtuelle pour obtenir les remises. Un achat d’instance réservée couvre uniquement la partie Calcul de l’utilisation de votre machine virtuelle. Pour les machines virtuelles Windows, le compteur d’utilisation est divisé en deux compteurs distincts. Il existe un compteur de calcul, qui est le même que le compteur Linux et un compteur IP Windows. Les frais que vous voyez lorsque vous effectuez l’achat sont uniquement pour les coûts de calcul. Les frais n’incluent pas les coûts des logiciels Windows. Pour plus d’informations sur les coûts liés aux logiciels, consultez [Coûts des logiciels non inclus dans Azure Reserved VM Instances](../cost-management-billing/reservations/reserved-instance-windows-software-costs.md).

## <a name="determine-the-right-vm-size-before-you-buy"></a>Déterminer la taille de machine virtuelle adaptée avant l’achat

Avant d’acheter une réservation, vous devez déterminer la taille de la machine virtuelle dont vous avez besoin. Les sections suivantes vous aideront à déterminer la taille de machine virtuelle appropriée.

### <a name="use-reservation-recommendations"></a>Utiliser les suggestions de réservation

Vous pouvez utiliser les suggestions de réservation pour déterminer les réservations à acheter.

- Les suggestions d’achat et une quantité recommandée s’affichent lorsque vous achetez une instance réservée de machine virtuelle sur le portail Azure.
- Azure Advisor fournit des suggestions d’achat pour les divers abonnements.  
- Vous pouvez utiliser les API pour obtenir des suggestions d’achat tant pour l’étendue partagée que pour l’étendue d’un abonnement unique. Pour plus d’informations, voir [API de suggestion d’achat d’instance réservée pour les entreprises](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation).
- Pour les clients Accord Entreprise (EA) et Microsoft Customer Agreement (MCA), des suggestions d’achat pour des étendues d’abonnement simples et partagées sont disponibles dans le [pack de contenu Power BI Azure Consumption Insights](/power-bi/service-connect-to-azure-consumption-insights).

### <a name="services-that-get-vm-reservation-discounts"></a>Services qui obtiennent des remises de réservation de machines virtuelles

Vos réservations de machine virtuelle peuvent s’appliquer à l’utilisation de machines virtuelles émise à partir de plusieurs services, pas seulement pour vos déploiements de machine virtuelle. Les ressources qui obtiennent des remises de réservation varient en fonction du paramètre de flexibilité de la taille d’instance.

#### <a name="instance-size-flexibility-setting"></a>Paramètre de flexibilité de la taille d’instance

Le paramètre de flexibilité de la taille d’instance détermine les services qui obtiennent les remises d’instance réservée.

Que le paramètre soit activé ou désactivé, les remises de réservation s’appliquent automatiquement à toute utilisation de machine virtuelle correspondante lorsque la valeur *ConsumedService* est `Microsoft.Compute`. Par conséquent, vérifiez vos données d’utilisation pour la valeur *ConsumedService*. Voici quelques exemples :

- Machines virtuelles
- Groupes identiques de machines virtuelles
- Service de conteneur
- Déploiements Azure Batch (en mode Abonnement utilisateur)
- Azure Kubernetes Service (AKS)
- Service Fabric

Lorsque le paramètre est activé, les remises de réservation s’appliquent automatiquement à l’utilisation de la machine virtuelle correspondante lorsque la valeur *ConsumedService* est l’un des éléments suivants :

- Microsoft.Compute
- Microsoft.ClassicCompute
- Microsoft.Batch
- Microsoft.MachineLearningServices
- Microsoft.Kusto

Vérifiez la valeur *ConsumedService* dans vos données d’utilisation pour déterminer si l’utilisation est éligible pour les remises de réservation.

Pour plus d’informations à propos de la flexibilité de la taille d’instance, consultez [Flexibilité en termes de taille de machine virtuelle avec des instances de machines virtuelles réservées](reserved-vm-instance-size-flexibility.md).

### <a name="analyze-your-usage-information"></a>Analyser vos informations d’utilisation

Pour faciliter la détermination des réservations à acheter, vous devez analyser vos informations d’utilisation. Des données d’utilisation sont disponibles dans le fichier d’utilisation et les API. Combinez-les pour déterminer la réservation à acheter. Pour déterminer la quantité de réservations à acheter, vous devez identifier des instances de machine virtuelle très utilisées quotidiennement. Évitez la sous-catégorie `Meter` et les champs `Product` dans les données d’utilisation. Ils ne font pas de distinction entre les tailles de machines virtuelles qui utilisent un stockage Premium. Si vous vous basez sur ces champs pour déterminer la taille de machine virtuelle en vue d’un achat de réservation, vous risquez d’opter pour une taille incorrecte. Ensuite, vous n’obtiendrez la remise sur réservation que vous attendez. Pour déterminer la taille correcte de machine virtuelle, consultez plutôt le champ `AdditionalInfo` dans le fichier ou l’API d’utilisation.

Votre fichier d’utilisation indique vos frais par période de facturation et l’utilisation quotidienne. Pour plus d’informations sur le téléchargement de votre fichier d’utilisation, consultez [Afficher et télécharger vos informations d’utilisation et vos frais Azure](../cost-management-billing/understand/download-azure-daily-usage.md). Ensuite, en utilisant les informations du fichier d’utilisation, vous pouvez [déterminer la réservation à acheter](../cost-management-billing/reservations/determine-reservation-purchase.md).

### <a name="purchase-restriction-considerations"></a>Considérations en matière de restriction d’achat

Des instances de machines virtuelles réservées sont disponibles pour la plupart des tailles de machine virtuelle, à quelques exceptions près. Les remises sur réservation ne s’appliquent pas aux machines virtuelles suivantes :

- **Séries de machines virtuelles** : A, Av2 ou G.

- **Machines virtuelles préversion et promo** : n’importe quelle série ou tailles de machines virtuelles en préversion ou qui utilise un compteur promotionnel.

- **Clouds** : les réservations ne sont pas disponibles à l’achat dans les régions Allemagne et Chine.

- **Quota insuffisant** : une réservation dont l’étendue est limitée à un seul abonnement doit avoir un quota de processeurs virtuels disponible dans l’abonnement pour la nouvelle instance réservée. Par exemple, si l’abonnement cible a un quota limite de 10 processeurs virtuels pour la série D, vous ne pouvez pas acheter de réservation pour 11 instances Standard_D1. La vérification du quota pour les réservations inclut les machines virtuelles déjà déployées dans l’abonnement. Par exemple, si l’abonnement a un quota de 10 processeurs virtuels pour la série D et a deux instances Standard_D1 déployées, vous pouvez acheter une réservation pour 10 instances Standard_D1 dans cet abonnement. Pour résoudre ce problème, vous pouvez [créer une demande d’augmentation de quota](../azure-portal/supportability/resource-manager-core-quotas-request.md).

- **Restrictions de capacité** : dans de rares cas, Azure limite l’achat de nouvelles réservations pour certaines tailles de machine virtuelle, en raison d’une faible capacité dans une région donnée.

## <a name="buy-a-reserved-vm-instance"></a>Acheter une instance de machine virtuelle réservée

Vous pouvez acheter une instance de machine virtuelle réservée dans le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D). Payez la réservation [à l’avance ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md).
Ces conditions s’appliquent à l’achat d’une instance de machine virtuelle réservée :

- Vous devez avoir le rôle Propriétaire pour au moins un abonnement EA ou un abonnement assorti d’un paiement à l’utilisation.
- Pour les abonnements EA, l’option **Ajouter des instances réservées** doit être activée sur le [portail EA](https://ea.azure.com/). Ou, si ce paramètre est désactivé, vous devez être administrateur EA de l’abonnement.
- Pour le programme CSP (Cloud Solution Provider), seuls les agents administrateurs ou les agents commerciaux peuvent acheter les réservations.

Pour acheter une instance :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Tous les services** > **Réservations**.
1. Sélectionnez **Ajouter** pour acheter une nouvelle réservation, puis cliquez sur **Machine virtuelle**.
1. Complétez les champs obligatoires. Les instances de machines virtuelles en cours d’exécution qui correspondent aux attributs que vous sélectionnez se qualifient pour bénéficier de la remise de réservation. Le nombre réel de vos instances de machines virtuelles qui obtiennent la remise dépend de l’étendue et de la quantité sélectionnées.

Si vous avez un Accord Entreprise, vous pouvez utiliser l’option **Ajouter** pour ajouter rapidement des instances supplémentaires. L’option n’est pas disponible pour les autres types d’abonnements.


| Champ      | Description|
|------------|--------------|
|Abonnement|Abonnement utilisé pour payer la réservation. Les coûts de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P or MS-AZR-0148P) ou Microsoft Customer Agreement (MCA) ou un abonnement individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Les frais sont déduits du solde de l’engagement financier, si disponibles, ou facturés comme un dépassement. Pour un abonnement payable à l’utilisation, les frais sont facturés sur le mode de paiement par carte de crédit ou facture défini sur l’abonnement.|    
|Étendue       |L’étendue de la réservation peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez : <ul><li>**Étendue de groupe de ressources unique** : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.</li><li>**Étendue d’abonnement unique** : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.</li><li>**Étendue partagée** : applique la remise de réservation aux ressources correspondantes dans les abonnements éligibles inclus dans le contexte de facturation. Pour les clients EA, le contexte de facturation est défini par l’inscription. Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.</li></ul>|
|Région    |Région Azure couverte par la réservation.|    
|Taille de la machine virtuelle     |Taille des instances de machines virtuelles.|
|Optimiser pour     |La flexibilité de la taille d’instance de machine virtuelle est sélectionnée par défaut. Cliquez sur **Paramètres avancés** pour modifier la valeur de la flexibilité de taille d’instance afin d’appliquer la remise de réservation à d’autres machines virtuelles dans le même [Groupe de taille de machine virtuelle](reserved-vm-instance-size-flexibility.md). La priorité de capacité priorise la capacité de centre de données pour vos déploiements. Cela offre une assurance supplémentaire dans votre capacité à lancer les instances de machines virtuelles quand vous en avez besoin. La priorité de capacité est uniquement disponible si l’étendue de réservation comprend un seul abonnement. |
|Terme        |Une année ou trois ans. Une durée de 5 ans est également disponible uniquement pour les machines virtuelles HBv2 VMs.|
|Quantité    |Nombre d’instances achetées au sein de la réservation. La quantité correspond au nombre d’instances de machines virtuelles en cours d’exécution pouvant bénéficier de la remise de facturation. Par exemple, si vous exécutez 10 machines virtuelles Standard_D2 dans la région USA Est, vous devez spécifier 10 comme quantité pour optimiser l’avantage pour toutes les machines virtuelles en cours d’exécution. |

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2PjmT]

## <a name="usage-data-and-reservation-utilization"></a>Données d’utilisation et utilisation de la réservation

Vos données d’utilisation ont un prix effectif de zéro pour l’utilisation qui bénéficie d’une remise de réservation. Vous pouvez voir l’instance de machine virtuelle qui a reçu la remise de réservation pour chaque réservation.

Pour plus d’informations sur l’affichage des remises de réservation dans les données d’utilisation, consultez [Comprendre l’utilisation de la réservation Azure pour l’inscription de votre entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md) si vous êtes un client EA. Si vous avez un abonnement individuel, consultez [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md).

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

Toutefois, vous pouvez *échanger* une réservation si vous souhaitez y apporter des modifications.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir comment gérer une réservation, consultez [Gérer les réservations Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md).
- Pour plus d’informations sur les réservations Azure, consultez les articles suivants :
    - [Qu’est-ce qu’une réservation Azure ?](../cost-management-billing/reservations/save-compute-costs-reservations.md)
    - [Gérer les réservations dans Azure](../cost-management-billing/reservations/manage-reserved-vm-instance.md)
    - [Comprendre comment la remise de réservation est appliquée](../cost-management-billing/manage/understand-vm-reservation-charges.md)
    - [Comprendre l’utilisation d’une réservation pour un abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md)
    - [Comprendre l’utilisation d’une réservation pour votre Accord de Mise en Œuvre Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md)
    - [Coûts des logiciels Windows non inclus dans les réservations](../cost-management-billing/reservations/reserved-instance-windows-software-costs.md)
    - [Réservations Azure dans le cadre du programme Fournisseur de solutions Cloud de l’Espace partenaires](/partner-center/azure-reservations)