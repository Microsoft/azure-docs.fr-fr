---
title: Économiser grâce à l’instance réservée de Azure VMware Solution
description: Découvrez comment acheter une instance réservée pour Azure VMware Solution.
ms.topic: how-to
ms.date: 11/12/2020
ms.openlocfilehash: 80780e9718001893b6960220b5004c1c29b2fec5
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327395"
---
# <a name="save-costs-with-azure-vmware-solution"></a>Économiser grâce à Azure VMware Solution

Lorsque vous validez une instance réservée d’[Azure VMware Solution](introduction.md), vous économisez de l’argent.  La remise de réservation est appliquée automatiquement aux hôtes Azure VMware Solution en cours d’exécution qui correspondent à l’étendue et aux attributs de la réservation. Un achat d’instance réservée couvre uniquement la partie calcul de votre utilisation et inclut les coûts liés aux licences logicielles. 

## <a name="purchase-restriction-considerations"></a>Considérations en matière de restriction d’achat

Les instances réservées sont disponibles avec quelques exceptions près :

-   **Clouds** : les réservations sont disponibles uniquement dans les régions répertoriées sur la page [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=azure-vmware).

-   **Quota insuffisant** : une réservation étendue à un abonnement unique/partagé doit avoir un quota d’hôtes disponible dans l’abonnement pour la nouvelle instance réservée. Pour résoudre ce problème, vous pouvez [créer une demande d’augmentation de quota](enable-azure-vmware-solution.md).

-   **Éligibilité de l’offre** : vous aurez besoin d’un [Contrat Entreprise (EA) Azure](../cost-management-billing/manage/ea-portal-agreements.md) avec Microsoft.

-   **Restrictions de capacité** : dans de rares cas, Azure limite l’achat de nouvelles réservations pour les SKU d’hôtes Azure VMware Solution en raison d’une faible capacité dans une région donnée.

## <a name="buy-a-reservation"></a>Acheter une réservation

Vous pouvez acheter une instance réservée pour une instance d’hôte Azure VMware Solution dans le [portail Azure](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/documentation/filters/%7B%22reservedResourceType%22%3A%22VirtualMachines%22%7D).

Vous pouvez payer la réservation [comptant ou par paiements mensuels](../cost-management-billing/reservations/prepare-buy-reservation.md).

Ces conditions s’appliquent à l’achat d’une instance réservée d’hôte dédié :

-   Vous devez avoir le rôle *Propriétaire* pour au moins un abonnement EA ou un abonnement assorti d’un paiement à l’utilisation.

-   Pour les abonnements EA, vous devez activer l’option **Ajouter des instances réservées** dans le [portail EA](https://ea.azure.com/). Si elle est désactivée, vous devez être administrateur EA de l’abonnement pour l’activer.

-   Pour un abonnement dans le cadre d’un plan Azure du fournisseur de solutions Cloud (CSP), le partenaire doit acheter les instances réservées dans le portail Azure pour le client. 

### <a name="buy-reserved-instances-for-an-ea-subscription"></a>Acheter des instances réservées pour un abonnement EA

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez **Tous les services** > **Réservations**.

3. Sélectionnez **Acheter maintenant**, puis sélectionnez **Azure VMware Solution**.

4. Renseignez les champs obligatoires. Les attributs sélectionnés qui correspondent aux hôtes Azure VMware Solution en cours d’exécution sont éligibles pour la remise de réservation.  Les attributs incluent la référence SKU, les régions (le cas échéant) et l’étendue. L’étendue de réservation détermine où les économies de la réservation s’appliquent.

   Si vous avez un contrat EA, vous pouvez utiliser l’option **Ajouter** pour ajouter rapidement des instances. L’option n’est pas disponible pour les autres types d’abonnements.

   | Champ        |  Description |
   | ------------ | ------------ |
   | Abonnement | Abonnement utilisé pour payer la réservation. Les coûts de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P or MS-AZR-0148P), Contrat client Microsoft ou un abonnement individuel avec paiement à l’utilisation (numéros de l’offre : MS-AZR-0003P ou MS-AZR-0023P). Les frais sont déduits du solde de l’engagement financier, si disponibles, ou facturés comme un dépassement. Pour un abonnement payable à l’utilisation, les frais sont facturés sur la carte de crédit de l’abonnement ou sur un mode de paiement par facture. |
   | Étendue        | L’étendue de la réservation peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez :<br><ul><li><b>Étendue de groupe de ressources unique</b> : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.</li><li><b>Étendue d’abonnement unique</b> : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.</li><li><b>Étendue partagée</b> : applique la remise de réservation aux ressources correspondantes des abonnements éligibles inclus dans le contexte de facturation. Pour les clients EA, le contexte de facturation est défini par l’inscription. Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.</li></ul>       |
   | Région       | Région Azure couverte par la réservation.   |
   | Taille de l’hôte    | AV36    |
   | Terme         | Une année ou trois ans.  |
   | Quantité     | Nombre d’instances à acheter au sein de la réservation. La quantité correspond au nombre d’hôtes Azure VMware Solution en cours d’exécution pouvant bénéficier de la remise.    |

### <a name="buy-reserved-instances-for-a-csp-subscription"></a>Acheter des instances réservées pour un abonnement CSP

Les fournisseurs de solutions Cloud qui souhaitent acheter des instances réservées pour leurs clients doivent utiliser la procédure **Administrer au nom de** (AOBO) figurant dans la [documentation d’Espace partenaires](/partner-center/azure-plan-manage). Pour plus d’informations, consultez la vidéo [Admin on behalf of (AOBO)](https://channel9.msdn.com/Series/cspdev/Module-11-Admin-On-Behalf-Of-AOBO).

1. Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com).

2. Sélectionnez **CSP** pour accéder à la zone **Clients**.

3. Développez les détails du client, puis sélectionnez **Portail de gestion Microsoft Azure**. 

   :::image type="content" source="media/reserved-instances/csp-partner-center-aobo.png" alt-text="Zone Clients de l’Espace partenaires Microsoft" lightbox="media/reserved-instances/csp-partner-center-aobo.png":::

4. Dans le portail Azure, sélectionnez **Tous les services** > **Réservations**.

5. Sélectionnez **Acheter maintenant**, puis sélectionnez **Azure VMware Solution**.

   :::image type="content" source="media/reserved-instances/csp-buy-ri-azure-portal.png" alt-text="Réservations dans Portail Microsoft Azure" lightbox="media/reserved-instances/csp-buy-ri-azure-portal.png":::

6. Renseignez les champs obligatoires. Les attributs sélectionnés qui correspondent aux hôtes Azure VMware Solution en cours d’exécution sont éligibles pour la remise de réservation.  Les attributs incluent la référence SKU, les régions (le cas échéant) et l’étendue. L’étendue de réservation détermine où les économies de la réservation s’appliquent.

   | Champ        |  Description |
   | ------------ | ------------ |
   | Abonnement | Abonnement utilisé pour payer la réservation. Les coûts de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être un abonnement éligible qui, dans ce cas, est un abonnement CSP.|
   | Étendue        | L’étendue de la réservation peut couvrir un seul abonnement ou plusieurs abonnements (étendue partagée). Si vous sélectionnez :<br><ul><li><b>Étendue de groupe de ressources unique</b> : applique la remise de réservation aux ressources correspondantes incluses dans le groupe de ressources sélectionné uniquement.</li><li><b>Étendue d’abonnement unique</b> : applique la remise de réservation aux ressources correspondantes incluses dans l’abonnement sélectionné.</li><li><b>Étendue partagée</b> : applique la remise de réservation aux ressources correspondantes des abonnements éligibles inclus dans le contexte de facturation. Pour les clients EA, le contexte de facturation est défini par l’inscription. Pour les abonnements individuels utilisant les tarifs du paiement à l’utilisation, l’étendue de facturation correspond à tous les abonnements éligibles créés par l’administrateur de compte.</li></ul>       |
   | Région       | Région Azure couverte par la réservation.   |
   | Taille de l’hôte    | AV36    |
   | Terme         | Une année ou trois ans.  |
   | Quantité     | Nombre d’instances à acheter au sein de la réservation. La quantité correspond au nombre d’hôtes Azure VMware Solution en cours d’exécution pouvant bénéficier de la remise.     |

Pour en savoir plus sur l’affichage des réservations achetées pour votre client, consultez l’article [Afficher les réservations Azure en tant que fournisseur de solutions cloud (CSP)](../cost-management-billing/reservations/how-to-view-csp-reservations.md).

## <a name="usage-data-and-reservation-usage"></a>Données d’utilisation et utilisation de la réservation

Votre utilisation qui bénéficie d’une remise de réservation a un prix effectif de zéro. Vous pouvez voir quelle instance Azure VMware Solution a reçu la remise de réservation pour chaque réservation.

Pour plus d’informations sur l’affichage des remises de réservation dans les données d’utilisation :

- Pour les clients EA, consultez [Comprendre l’utilisation d’une réservation Azure pour votre accord de mise en œuvre Entreprise](../cost-management-billing/reservations/understand-reserved-instance-usage-ea.md).
- Pour les abonnements individuels, consultez [Comprendre l’utilisation d’une réservation Azure pour votre abonnement avec paiement à l’utilisation](../cost-management-billing/reservations/understand-reserved-instance-usage.md).

## <a name="change-a-reservation-after-purchase"></a>Modifier une réservation après achat

Vous pouvez apporter ces modifications à une réservation après achat :

-   Mettez à jour l’étendue de la réservation

-   Flexibilité de taille d’instance (le cas échéant)

-   Propriété

Vous pouvez également fractionner une réservation en blocs plus petits ou fusionner des réservations. Aucune des modifications n’entraîne de nouvelle transaction commerciale ou de changement de la date de fin de la réservation.

Pour plus d’informations sur les réservations gérées par CSP, consultez [Vendre des réservations Microsoft Azure à des clients à l’aide d’Espace partenaires, du portail Azure ou d’API](/partner-center/azure-reservations).



>[!NOTE]
>Une fois que vous avez acheté votre réservation, vous ne serez pas en mesure d’apporter directement les types de modifications suivants :
>
> - Région d’une réservation existante
> - SKU
> - Quantité
> - Duration
>
>Toutefois, vous pouvez *échanger* une réservation si vous souhaitez y apporter des modifications.

## <a name="cancel-exchange-or-refund-reservations"></a>Annuler, échanger ou rembourser des réservations

Vous pouvez annuler, échanger ou rembourser des réservations avec certaines limitations. Pour plus d’informations, consultez [Échanges et remboursements en libre-service pour les réservations Azure](../cost-management-billing/reservations/exchange-and-refund-azure-reservations.md).

Les CSP peuvent annuler, échanger ou rembourser, avec certaines restrictions, les réservations achetées pour leur client. Pour plus d’informations, consultez [Gérer, annuler, échanger ou rembourser des réservations Microsoft Azure pour les clients](/partner-center/azure-reservations-manage).