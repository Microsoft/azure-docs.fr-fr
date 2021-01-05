---
title: Comprendre le détail de l’utilisation et des frais
description: Découvrez comment lire et comprendre votre fichier détaillé sur l’utilisation et les frais. Affichez la liste des termes et descriptions utilisés dans le fichier.
author: bandersmsft
ms.reviewer: micflan
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 559fe43bbd9b8951a97733ae1250046f65f10404
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92149359"
---
# <a name="understand-the-terms-in-your-azure-usage-and-charges-file"></a>Comprendre les termes du fichier sur l’utilisation et les frais Azure

Le fichier détaillé sur l’utilisation et les frais contient l’utilisation quotidienne évaluée sur la base des tarifs négociés, des achats (par exemple, réservations, dépenses sur l Place de marché) et des remboursements au cours de la période spécifiée.
Les frais ne comprennent les crédits, taxes, ou d’autres frais ou remises.
Le tableau suivant couvre les frais inclus pour chaque type de compte.

Type de compte | Utilisation d’Azure | Utilisation de la Place de marché | Achats | Remboursements
--- | --- | --- | --- | ---
Contrat Entreprise (EA) | Oui | Oui | Oui | Non
Contrat client Microsoft (MCA) | Oui | Oui | Oui | Oui
Paiement à l’utilisation | Oui | Oui | Non | Non

Pour en savoir plus sur les commandes passées sur la Place de marché (également appelées services externes), voir [Comprendre vos frais de service externe Azure](understand-azure-marketplace-charges.md).

Pour obtenir des instructions de téléchargement, consultez [Guide pratique pour obtenir votre facture Azure et vos données d’utilisation quotidienne](../manage/download-azure-invoice-daily-usage-date.md).
Vous pouvez ouvrir votre fichier CSV sur l’utilisation et les frais dans Microsoft Excel ou une autre application de feuille de calcul.

## <a name="list-of-terms-and-descriptions"></a>Liste de termes et descriptions

Le tableau suivant présente les termes importants utilisés dans la dernière version du fichier sur l’utilisation et les frais Azure.
La liste couvre les comptes associés à un paiement à l’utilisation (PAYG), à un Contrat Entreprise (EA) et à un Contrat client Microsoft (MCA).

Terme | Type de compte | Description
--- | --- | ---
AccountName | EA, PAYG | Nom complet du compte d’inscription EA ou du compte de facturation PAYG.
AccountOwnerId<sup>1</sup> | EA, PAYG | Identificateur unique pour le compte d’inscription EA ou le compte de facturation PAYG.
AdditionalInfo | Tous | Métadonnées relatives au service. Par exemple, le type d’image d’une machine virtuelle.
BillingAccountId<sup>1</sup> | Tous | Identificateur unique pour le compte de facturation racine.
BillingAccountName | Tous | Nom du compte de facturation.
BillingCurrency | Tous | Devise associée au compte de facturation.
BillingPeriod | EA, PAYG | Période de facturation des frais.
BillingPeriodEndDate | Tous | Date de fin de la période de facturation.
BillingPeriodStartDate | Tous | La date de début de la période de facturation.
BillingProfileId<sup>1</sup> | Tous | Identificateur unique de l’inscription EA, de l’abonnement PAYG, du profil de facturation MCA ou du compte regroupé AWS.
BillingProfileName | Tous | Nom de l’inscription EA, de l’abonnement PAYG, du profil de facturation MCA ou du compte regroupé AWS.
ChargeType | Tous | Indique si les frais ont trait à l’utilisation (**Usage**), à un achat (**Purchase**) ou à un remboursement (**Refund**).
ConsumedService | Tous | Nom du service auquel les frais sont associés.
CostCenter<sup>1</sup> | EA, MCA | Centre de coûts défini pour l’abonnement au fins de suivi des coûts (uniquement disponible dans les des périodes de facturation ouvertes pour les comptes MCA).
Coût | EA, PAYG | Voir CostInBillingCurrency.
CostInBillingCurrency | MCA | Coût des frais dans la devise de facturation avant crédits ou taxes.
CostInPricingCurrency | MCA | Coût des frais dans la devise de tarification avant crédits ou taxes.
Devise | EA, PAYG | Voir BillingCurrency.
Date<sup>1</sup> | Tous | Date d’achat ou d’utilisation à laquelle les frais correspondent.
EffectivePrice | Tous | Prix unitaire unifié pour la période. Les prix unifiés établissent la moyenne des fluctuations du prix unitaire, comme un nivellement progressif, ce qui réduit le prix à mesure que la quantité augmente au fil du temps.
ExchangeRateDate | MCA | Date à laquelle que le taux de change a été établi.
ExchangeRatePricingToBilling | MCA | Taux de change utilisé pour convertir le coût de la devise de tarification vers la devise de facturation.
Fréquence | Tous | Indique si un coût est supposé se répéter. Les coûts peut être ponctuels (**OneTime**), se répéter mensuellement ou annuellement (**Recurring**) ou être basés sur l’utilisation (**UsageBased**).
InvoiceId | PAYG, MCA | Référence de document unique figurant sur la facture au format PDF.
InvoiceSection | MCA | Voir InvoiceSectionName.
InvoiceSectionId<sup>1</sup> | EA, MCA | Identificateur unique du service EA ou de la section de facture MCA.
InvoiceSectionName | EA, MCA | Nom du service EA ou de la section de facture MCA.
IsAzureCreditEligible | Tous | Indique si les frais sont éligibles pour un paiement à l’aide de crédits Azure (valeurs : True, False).
Emplacement | MCA | Emplacement du centre de données où la ressource est exécutée.
MeterCategory | Tous | Nom de la catégorie de classification du compteur. Par exemple, *Services cloud* ou *Mise en réseau*.
MeterId<sup>1</sup> | Tous | Identificateur unique du compteur.
MeterName | Tous | Nom du compteur.
MeterRegion | Tous | Nom de l’emplacement du centre de données pour des services tarifés en fonction de l’emplacement. Voir Location.
MeterSubCategory | Tous | Nom de la catégorie de sous-classification du compteur.
OfferId<sup>1</sup> | Tous | Nom de l’offre achetée.
PayGPrice | Tous | Prix de vente au détail de la ressource.
PartNumber<sup>1</sup> | EA, PAYG | Identificateur utilisé pour obtenir la tarification du compteur spécifique.
PlanName | EA, PAYG | Nom du plan sur la Place de marché.
PreviousInvoiceId | MCA | Référence à une facture d’origine si cette ligne correspond à un remboursement.
PricingCurrency | MCA | Devise utilisée lors de l’évaluation basée sur les prix négociés.
PricingModel | Tous | Identificateur indiquant le mode de facturation utilisé par le compteur. (Valeurs : à la demande, réservation, Spot)
Produit | Tous | Nom du produit.
ProductId<sup>1</sup> | MCA | Identificateur unique pour le produit.
ProductOrderId | Tous | Identificateur unique de la commande du produit.
ProductOrderName | Tous | Nom unique de la commande du produit.
PublisherName | Tous | Éditeur pour les services de la Place de marché.
PublisherType | Tous | Type d’éditeur (valeurs : **Azure**, **AWS**, **Place de marché**).
Quantité | Tous | Nombre d’unités achetées ou consommés.
ID de réservation | EA, MCA | Identificateur unique de l’instance de réservation achetée.
ReservationName | EA, MCA | Nom de l’instance de réservation achetée.
ResourceGroup | Tous | Nom du [groupe de ressources](../../azure-resource-manager/management/overview.md) dans lequel figure la ressource. Tous les frais ne proviennent pas de ressources déployées sur des groupes de ressources. Les frais qui n’ont pas de groupe de ressources seront indiqués comme étant nuls ou vides, **autres**ou **non applicable**.
ResourceId<sup>1</sup> | Tous | Identificateur unique de la ressources [Azure Resource Manager](/rest/api/resources/resources).
ResourceLocation | Tous | Emplacement du centre de données où la ressource est exécutée. Voir Location.
Nom_ressource | EA, PAYG | Nom de la ressource. Tous les frais ne proviennent pas de ressources déployées. Les frais qui n’ont pas de type de ressource s’affichent comme étant nuls ou vides, **autres**ou **non applicable**.
ResourceType | MCA | Type d’instance de ressource. Tous les frais ne proviennent pas de ressources déployées. Les frais qui n’ont pas de type de ressource s’affichent comme étant nuls ou vides, **autres**ou **non applicable**.
ServiceFamily | MCA | Famille de services à laquelle appartient le service.
ServiceInfo1 | Tous | Métadonnées relatives au service.
ServiceInfo2 | Tous | Champ hérité avec des métadonnées facultatives spécifiques du service.
ServicePeriodEndDate | MCA | Date de fin de la période d’évaluation utilisée pour définir et fixer la tarification correspondant au service consommé ou acheté.
ServicePeriodStartDate | MCA | Date de début de la période d’évaluation utilisée pour définir et fixer la tarification correspondant au service consommé ou acheté.
SubscriptionId<sup>1</sup> | Tous | Identificateur unique de l’abonnement Azure.
SubscriptionName | Tous | Nom de l’abonnement Azure.
Tags<sup>1</sup> | Tous | Balises affectées à la ressource. N’inclut pas les balises de groupe de ressources. Peut être utilisée pour grouper ou répartir des coûts de refacturation interne. Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](https://azure.microsoft.com/updates/organize-your-azure-resources-with-tags/).
Terme | Tous | Affiche le terme de validité de l’offre. Par exemple : Dans le cas d’instances réservées, le terme est de 12 mois. Pour les achats ponctuels ou les achats récurrents, le terme est de 1 mois (SaaS, support Marketplace). Cela ne s’applique pas à la consommation Azure.
UnitOfMeasure | Tous | Unité de mesure pour la facturation du service. Par exemple, les services de calcul sont facturés à l’heure.
UnitPrice | EA, PAYG | Prix unitaire des frais.

_<sup>**1**</sup> Champs utilisés pour générer un ID unique pour un enregistrement de coût unique._

Notez que certains champs peuvent différer par leur casse ou leur espacement d’un type de compte à l’autre.
Les versions antérieures des fichiers sur le paiement à l’utilisation comportent des sections séparées pour le relevé et l’utilisation quotidienne.

### <a name="list-of-terms-from-older-apis"></a>Liste de termes des anciennes API
Le tableau suivant établit les correspondances entre les termes utilisés dans les anciennes API et les nouveaux termes. Reportez-vous au tableau ci-dessus pour connaître ces descriptions.

Ancien terme | Nouveau terme
--- | ---
ConsumedQuantity | Quantité
IncludedQuantity | N/A
InstanceId | ResourceId
Tarif | EffectivePrice
Unité | UnitOfMeasure
UsageDate | Date
UsageEnd | Date
UsageStart | Date


## <a name="ensure-charges-are-correct"></a>Vérifier que les frais sont corrects

Pour en savoir plus sur les détails de l’utilisation et des frais, découvrez comment comprendre votre facture de [paiement à l’utilisation](review-individual-bill.md) ou de [Contrat client Microsoft](review-customer-agreement-bill.md).

## <a name="need-help-contact-us"></a>Vous avez besoin d’aide ? Contactez-nous.

Si vous avez des questions ou besoin d’aide, [créez une demande de support](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Étapes suivantes

- [Afficher et télécharger votre facture Microsoft Azure](download-azure-invoice.md)
- [Afficher et télécharger votre utilisation et vos frais Microsoft Azure](download-azure-daily-usage.md)