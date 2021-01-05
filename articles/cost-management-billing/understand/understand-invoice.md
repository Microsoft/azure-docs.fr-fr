---
title: Comprendre votre facture Azure
description: Apprenez à lire et à comprendre l’utilisation et la facturation de votre abonnement Azure
author: bandersmsft
ms.reviewer: jureid
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: b7b115f909c57470bfbdab9cc44e6a258f85b8f1
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88685916"
---
# <a name="understand-terms-on-your-microsoft-azure-invoice"></a>Comprendre les termes figurant sur votre facture Microsoft Azure

La facture fournit un résumé de vos frais et fournit des instructions pour le paiement. Elle est disponible au téléchargement au format PDF (.pdf) à partir du [portail Azure](https://portal.azure.com/) ou peut être envoyée par courrier électronique. Pour plus d’informations, consultez [Comment obtenir votre facture Azure et vos données d’utilisation quotidienne](../manage/download-azure-invoice-daily-usage-date.md).

Points à noter :

-   Si vous utilisez un abonnement d’essai gratuit, vous pouvez obtenir vos informations d’utilisation quotidienne à partir du portail Azure, mais vous n’avez pas de facture.

-   Jusqu’à 24 heures d’utilisation à la fin de la période de facturation précédente peuvent s’afficher dans votre facture en cours.

-   Les frais répertoriés sur les relevés de facturation pour les clients internationaux sont uniquement fournis à titre d’estimation. Les banques peuvent appliquer des coûts différents selon les taux de conversion.

>[!VIDEO https://www.youtube.com/embed/jWG1lyJe3Mg]

## <a name="detailed-terms-and-descriptions-of-your-invoice"></a>Termes et descriptions détaillés de votre facture
Les sections suivantes répertorient les termes importants que vous voyez sur votre facture, ainsi qu’une description pour chacun d’entre-eux.

### <a name="account-information"></a>Informations sur le compte

La section Informations sur le compte de la facture se trouve en haut de la première page et affiche des informations sur votre profil et votre abonnement.

![Section Informations sur le compte](./media/understand-invoice/account-information-section.png)

| Terme | Description |
| --- | --- |
| Numéro de l’ordre d’achat du client |Numéro de commande facultatif, attribué par vous pour le suivi |
| Numéro de facture |Numéro de facture unique généré par Microsoft utilisé à des fins de suivi |
| Cycle de facturation |Plage de dates couverte par cette facture |
| Date de facture |Date à laquelle la facture a été générée, généralement un jour après la fin du cycle de facturation |
| Mode de paiement |Type de paiement utilisé sur le compte (facture ou carte de crédit). |
| Facturer à |Adresse de facturation répertoriée pour le compte |
| Offre d’abonnement (paiement à l’utilisation) |Type d’offre d’abonnement achetée (par exemple, paiement à l’utilisation, BizSpark Plus, Pass Azure, etc.). Pour plus d’informations, consultez [Types d’offre Azure](https://azure.microsoft.com/support/legal/offer-details/). |
| E-mail du propriétaire du compte | Adresse de messagerie du compte sous lequel le compte Microsoft Azure est enregistré. <br /><br />Pour modifier l’adresse e-mail, consultez [Comment modifier les informations de profil de votre compte Azure comme l’e-mail de contact, l’adresse et le numéro de téléphone](../manage/change-azure-account-profile.md). |

### <a name="understand-the-invoice-summary"></a>Comprendre le résumé de la facture
La section **Récapitulatif de la facture** répertorie le montant total des transactions depuis votre dernière facture et vos coûts d’utilisation actuels.

![Section Récapitulatif de la facture](./media/understand-invoice/invoice-summary-section.png)

Le nom d’abonnement (« Stockage en production ») est le nom de l’abonnement pour cette facture.

#### <a name="understand-the-previous-charges"></a>Comprendre les frais précédents
Les sections Solde précédent, Paiements et Solde restant dû de la facture récapitulent les transactions intervenues depuis votre dernière période de facturation.

| Terme | Description |
| --- | --- |
| Solde précédent |Montant total dû de votre dernière période de facturation |
| Paiements |Total des paiements et crédits appliqués à votre dernière période de facturation |
| Solde dû (depuis le cycle de facturation précédent) |Tout crédit ou solde restant appliqué à votre compte depuis votre dernière période de facturation |

#### <a name="understand-the-current-charges"></a>Comprendre les frais actuels
La section Frais actuels de la facture affiche les détails de vos frais mensuels pour la période de facturation actuelle.

| Terme | Description |
| --- | --- |
| Frais d'utilisation |Les frais d’utilisation correspondent au total des frais mensuels d’un abonnement pour la période de fabrication actuelle|
| Remises |Remises sur les services appliquées à votre période de facturation en cours|
| Ajustements |Divers crédits (utilisation gratuite, crédits, etc.) ou frais exceptionnels appliqués à votre période de facturation en cours.<br/><br/>Par exemple, si vous disposez de l’offre Visual Studio Enterprise avec MSDN, un crédit mensuel apparaît. Si vous annulez votre abonnement, vous verrez que les frais d’utilisation mensuelle dépassent le crédit mensuel dont vous bénéficiez avec votre offre d’abonnement. Les frais s’appliquent à partir du début de la période de facturation actuelle jusqu’à la date d’annulation de l’abonnement. |

#### <a name="sold-to-and-payment-instructions"></a>Instructions de vente et de paiement

Le tableau suivant décrit les instructions de vente et de paiement indiquées sur la seconde page de votre facture.

| Terme |Description |
| --- | --- |
| Vendu à |Adresse de profil qui se trouve sur le compte. <br/><br/>Pour modifier l’adresse, consultez [Comment modifier les informations de profil de votre compte Azure comme l’e-mail de contact, l’adresse et le numéro de téléphone](../manage/change-azure-account-profile.md).|
| Instructions de paiement |Instructions de paiement en fonction du mode de paiement (carte bancaire ou facture). |

#### <a name="usage-charges"></a>Frais d’utilisation

La section Frais d’utilisation de la facture affiche des informations de compteur concernant vos frais.

![Section Frais d’utilisation](./media/understand-invoice/usage-charges-section.png)

Le tableau suivant décrit les en-têtes de colonne Frais d’utilisation indiqués sur votre facture.

| Terme |Description |
| --- | --- |
| Name |Identifie le service de niveau supérieur pour l’utilisation |
| Type |Définit le type de service Azure pouvant affecter le tarif |
| Ressource |Identifie l’unité de mesure du compteur consommé |
| Région |Identifie l’emplacement du centre de données pour certains services dont le prix est basé sur cet emplacement |
| Consommé |Quantité du compteur utilisée au cours de la période de facturation |
| Inclus |Montant du compteur inclus gratuitement dans votre période de facturation actuelle |
| Facturable |Affiche la différence entre la quantité consommée et la quantité incluse. Ce montant vous est facturé. Pour les offres avec paiement à l’utilisation n’incluant aucun montant, ce total est identique à la quantité consommée. |
| Tarif |Montant qui vous est facturé par unité facturable |
| Valeur |Affiche le résultat de la multiplication de la colonne Quantité de dépassement par la colonne Taux. Si la quantité consommée ne dépasse pas la quantité incluse, aucun frais n’apparaît dans cette colonne. |
| Sous-total |Somme de tous vos frais avant TVA pour cette période de facturation |
| Total général |Somme de tous vos frais après TVA pour cette période de facturation |

## <a name="how-do-i-make-sure-that-the-charges-in-my-invoice-are-correct"></a>Comment m’assurer que les frais indiqués sur ma facture sont corrects ?
Si vous voulez plus de détails sur des frais indiqués sur votre facture, consultez [Comprendre votre facture Microsoft Azure.](review-individual-bill.md)

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).
