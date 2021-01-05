---
title: Migrer un Contrat Entreprise vers des API Contrat client Microsoft – Azure
description: Cet article vous aide à comprendre les conséquences de la migration d’un Contrat Entreprise Microsoft vers un Contrat Client Microsoft.
author: bandersmsft
ms.author: banders
ms.date: 07/24/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.openlocfilehash: 811b2cb7fd9a4f664e7a643f5a8e192a51416888
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689097"
---
# <a name="migrate-from-enterprise-agreement-to-microsoft-customer-agreement-apis"></a>Migrer depuis des API Enterprise Agreement vers des API Microsoft Customer Agreement

Cet article vous aide à comprendre la structure de données, l’API et les autres différences d’intégration de système entre les comptes de Contrat Entreprise Microsoft (ou EA pour Enterprise Agreement) et de Contrat Client Microsoft (ou MCA, pour Microsoft Customer Agreement). Azure Cost Management prend en charge les API pour ces deux types de compte. Avant de continuer, veuillez lire l’article [Découvrez comment configurer votre compte de facturation associé à un Contrat Client Microsoft](../manage/mca-setup-account.md).

Nous recommandons aux organisations ayant déjà un compte EA de lire attentivement cet article lors de la phase de configuration de leur compte MCA. Auparavant, le renouvellement d’un compte EA nécessitait un minimum de travail pour passer d’une ancienne inscription vers une nouvelle. Quant à la migration vers un compte MCA, elle nécessitait encore plus d’efforts. Ces efforts supplémentaires étaient dus aux modifications du sous-système de facturation sous-jacent, impactant toutes les API liées et les offres de service liés aux coûts.

## <a name="mca-apis-and-integration"></a>API et intégration MCA

La nouvelle intégration et les API MCA vous permettent de :

- Disposer d’une disponibilité complète via des API Azure natives.
- Configurer plusieurs factures dans un seul compte de facturation.
- Pouvoir accéder à une API combinée avec l’utilisation des services Azure, ainsi qu’avec l’utilisation et les achats d’une place de marché tierce.
- Pouvoir afficher les coûts sur des profils de facturation (tout comme pour les inscriptions) à l’aide d’Azure Cost Management.
- Pouvoir accéder aux nouvelles API pour afficher les coûts, être averti lorsque les coûts dépassent des seuils prédéfinis, et exporter des données brutes automatiquement.

## <a name="migration-checklist"></a>Liste des éléments à vérifier pour la migration

Les éléments suivants vous aideront à d’effectuer votre transition vers les API MCA.

- Familiarisez-vous avec le contenu de l’article [Prise en main de votre compte de facturation dans le cadre d’un Contrat Client Microsoft](../understand/mca-overview.md).
- Déterminez les API que vous utilisez et affichez celles qui sont remplacées dans la section suivante.
- Familiarisez-vous avec les [API REST Azure Resource Manager](/rest/api/azure).
- Si vous n’utilisez pas déjà des API Azure Resource Manager, [inscrivez votre application cliente auprès d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad).
- Mettez à jour le code de programmation pour [utiliser l’authentification Azure AD](/rest/api/azure/#create-the-request).
- Mettez à jour le code de programmation pour remplacer les appels d’API EA par les appels d’API MCA.
- Mettez à jour la gestion des erreurs pour utiliser de nouveaux codes d’erreur.
- Passez en revue les offres d’intégration supplémentaires, telles que Cloudyn et Power BI, pour d’autres actions requises.

## <a name="ea-apis-replaced-with-mca-apis"></a>API EA remplacées par les API MCA

Les API EA utilisent une clé d’API pour l’authentification et l’autorisation. Les API MCA utilisent l’authentification Azure AD.

| Objectif | API EA | API MCA |
| --- | --- | --- |
| Solde et crédits | [/balancesummary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) | Microsoft.Billing/billingAccounts/billingProfiles/availableBalanceussae |
| Utilisation (JSON) | [/usagedetails](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)[/usagedetailsbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format) | [Microsoft.Consumption/usageDetails](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Utilisation (CSV) | [/usagedetails/download](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)[/usagedetails/submit](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format) | [Microsoft.Consumption/usageDetails/download](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Utilisation de la Place de marché (CSV) | [/marketplacecharges](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)[/marketplacechargesbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge) | [Microsoft.Consumption/usageDetails/download](/rest/api/consumption/usagedetails)<sup>1</sup> |
| Billing periods | [/billingperiods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) | Microsoft.Billing/billingAccounts/billingProfiles/invoices |
| Price sheet | [/pricesheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) | Microsoft.Billing/billingAccounts/billingProfiles/pricesheet/default/download format=json|csv Microsoft.Billing/billingAccounts/…/billingProfiles/…/invoices/… /pricesheet/default/download format=json|csv Microsoft.Billing/billingAccounts/../billingProfiles/../providers/Microsoft.Consumption/pricesheets/download  |
| Achats de réservation | [/reservationcharges](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges) | Microsoft.Billing/billingAccounts/billingProfiles/transactions |
| Reservation recommendations | [/SharedReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-shared-reserved-instance-recommendations)[/](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations)[SingleReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations) | [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list) |
| Utilisation de la réservation | [/reservationdetails](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#request-for-reserved-instance-usage-details)[/reservationsummaries](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.Consumption/reservationDetails](/rest/api/consumption/reservationsdetails)[Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries) |

<sup>1</sup> Le service Azure et l’utilisation de la Place de marché par des tiers sont disponibles avec l’[API Détails d’utilisation](/rest/api/consumption/usagedetails).

Les API suivantes sont disponibles pour les comptes de facturation MCA :

| Objectif | API Microsoft Customer Agreement (MCA) |
| --- | --- |
| Compte de facturation<sup>2</sup> | Microsoft.Billing/billingAccounts |
| Profils de facturation<sup>2</sup> | Microsoft.Billing/billingAccounts/billingProfiles |
| Sections de facture<sup>2</sup> | Microsoft.Billing/billingAccounts/invoiceSections |
| Factures | Microsoft.Billing/billingAccounts/billingProfiles/invoices |
| Abonnements de facturation | {scope}/billingSubscriptions |

<sup>2</sup> Les APIs renvoient des listes d’objets, qui sont des étendues, où fonctionnent les expériences Cost Management dans le Portail Microsoft Azure, ainsi que les API. Pour plus d’informations sur les étendues Cost Management, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).

Si vous utilisez déjà des API EA, vous devez les mettre à jour pour prendre en charge les comptes de facturation MCA. Le tableau suivant présente les autres modifications de l’intégration :

| Objectif | Ancienne offre | Nouvelle offre |
| --- | --- | --- |
| Cloudyn | [Cloudyn.com](https://www.cloudyn.com) | [Azure Cost Management](https://azure.microsoft.com/services/cost-management/) |
| Power BI | [Microsoft Consumption Insights](/power-bi/desktop-connect-azure-consumption-insights) pack de contenu et connecteur |  [Connecteur Azure Consumption Insights](/power-bi/desktop-connect-azure-consumption-insights) |

## <a name="apis-to-get-balance-and-credits"></a>API pour obtenir le solde et crédits

L’API [Get Balance Summary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) (Obtenir le résumé du solde) vous fournit un résumé mensuel des :

- Balances
- Nouveaux achats
- Frais du service Place de marché Azure
- Ajustements
- Frais de dépassement de service

Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request).

L’API Get Balance Summary est remplacée par l’API Available Balance (Solde disponible) (Microsoft.Billing/billingAccounts/billingProfiles/availableBalance).

Pour obtenir les soldes disponibles avec l’API Available Balance :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/availableBalances?api-version=2018-11-01-preview` |

## <a name="apis-to-get-cost-and-usage"></a>API pour obtenir le coût et utilisation

Permet d’obtenir une décomposition quotidienne des coûts à partir de l’utilisation des services Azure, de l’utilisation de la Place de marché par des tiers et d’autres achats de la Place de marché avec les API suivantes. Les API suivantes ont été fusionnées pour les services Azure et l’utilisation de la Place de marché par des tiers. Les anciennes API sont remplacés par l’API Usage Details (Détails d’utilisation) ([Microsoft.Consumption/usageDetails](/rest/api/consumption/usagedetails)). Elle ajoute les achats de la Place de marché, qui étaient précédemment affichés uniquement dans le résumé du solde.

- [Get usage detail/download](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)
- [Obtenir les détails de l’utilisation/envoyer](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#csv-format)
- [Get usage detail/usagedetails](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)
- [Get usage detail/usagedetailsbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail#json-format)
- [Get marketplace store charge/marketplacecharges](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)
- [Get marketplace store charge/marketplacechargesbycustomdate](/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge)

Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request).

Toutes les API précédentes sont remplacées par l’API Consumption/Usage Details.

Pour obtenir des détails d’utilisation avec l’API Usage Details :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/{scope}/providers/Microsoft.Consumption/usageDetails?api-version=2019-01-01` |

L’API Usage Details, comme avec toutes les API Cost Management, est disponible pour plusieurs étendues. Pour les coûts facturés, utilisez l’étendue du profil de facturation, comme vous le feriez normalement pour un niveau d’accord de mise en œuvre.  Pour plus d’informations sur les étendues Cost Management, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).

| Type | Format d’ID |
| --- | --- |
| Compte de facturation | `/Microsoft.Billing/billingAccounts/{billingAccountId}` |
| Profil de facturation | `/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}` |
| Subscription | `/subscriptions/{subscriptionId}` |
| Resource group | `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}` |

Utilisez les paramètres de chaîne de requête suivant pour mettre à jour le code de programmation.

| Anciens paramètres | Nouveaux paramètres |
| --- | --- |
| `billingPeriod={billingPeriod}` | Non pris en charge |
| `endTime=yyyy-MM-dd` | `endDate=yyyy-MM-dd` |
| `startTime=yyyy-MM-dd` | `startDate=yyyy-MM-dd` |

Le corps de la réponse est également modifié.

Ancien corps de la réponse :

```
{
  "id": "string",
  "data": [{...}, ...],
  "nextLink": "string"
}
```

Nouveau corps de la réponse :

```
{
  "value": [{
    "id": "{scope}/providers/Microsoft.Consumption/usageDetails/###",
    "name": "###",
    "type": "Microsoft.Consumption/usageDetails",
    "tags": {...},
    "properties": [{...}, ...],
    "nextLink": "string"
  }, ...]
}
```

Le nom de propriété contenant le tableau d’enregistrements d’utilisation est passé de data à _values_. Chaque enregistrement est utilisé pour avoir une liste plate des propriétés détaillées. Toutefois, tous les détails de chaque enregistrement se trouvent maintenant dans une propriété imbriquée nommée _properties_, à l’exception des balises. La nouvelle structure est cohérente avec d’autres API Azure. Certains noms de propriété ont changé. Le tableau suivant présente les propriétés correspondantes.

| Ancienne propriété | Nouvelle propriété | Notes |
| --- | --- | --- |
| AccountId | N/A | Le créateur de l’abonnement n’est pas suivi. Utilisez invoiceSectionId (identique à departmentId). |
| AccountNameAccountOwnerId et AccountOwnerEmail | N/A | Le créateur de l’abonnement n’est pas suivi. Utilisez invoiceSectionName (identique à departmentName). |
| AdditionalInfo | additionalInfo | &nbsp;  |
| ChargesBilledSeparately | isAzureCreditEligible | Notez que ces propriétés sont opposées. Si isAzureCreditEnabled a la valeur true, ChargesBilledSeparately a la valeur false. |
| ConsumedQuantity | quantité | &nbsp; |
| ConsumedService | consumedService | Les valeurs précises des chaînes peuvent différer. |
| ConsumedServiceId | None | &nbsp; |
| CostCenter | costCenter | &nbsp; |
| Date et usageStartDate | Date | &nbsp;  |
| jour | None | Analyse le jour à partir de la date. |
| DepartmentId | invoiceSectionId | Les valeurs précises diffèrent. |
| DepartmentName | invoiceSectionName | Les valeurs précises des chaînes peuvent différer. Permet de configurer les sections de vos factures en fonction des services impliqués, si nécessaire. |
| ExtendedCost et Cost | costInBillingCurrency | &nbsp;  |
| InstanceId | resourceId | &nbsp;  |
| Is Recurring Charge | None | &nbsp;  |
| Location | location | &nbsp;  |
| MeterCategory | meterCategory | Les valeurs précises des chaînes peuvent différer. |
| ID du compteur | meterId | Les valeurs précises des chaînes diffèrent. |
| MeterName | meterName | Les valeurs précises des chaînes peuvent différer. |
| MeterRegion | meterRegion | Les valeurs précises des chaînes peuvent différer. |
| MeterSubCategory | meterSubCategory | Les valeurs précises des chaînes peuvent différer. |
| Month | None | Analyse le mois à partir de la date. |
| Nom de l’offre | None | Utilisez publisherName et productOrderName. |
| OfferID | None | &nbsp;  |
| Order Number | None | &nbsp;  |
| PartNumber | None | Utilisez meterId et productOrderName pour identifier les prix de manière unique. |
| Nom du plan | productOrderName | &nbsp;  |
| Produit | Produit |   |
| ProductId | productId | Les valeurs précises des chaînes diffèrent. |
| Nom de l’éditeur | publisherName | &nbsp;  |
| ResourceGroup | resourceGroupName | &nbsp;  |
| ResourceGuid | meterId | Les valeurs précises des chaînes diffèrent. |
| ResourceLocation | resourceLocation | &nbsp;  |
| ResourceLocationId | None | &nbsp;  |
| ResourceRate | effectivePrice | &nbsp;  |
| ServiceAdministratorId | N/A | &nbsp;  |
| ServiceInfo1 | serviceInfo1 | &nbsp;  |
| ServiceInfo2 | serviceInfo2 | &nbsp;  |
| NomService | meterCategory | Les valeurs précises des chaînes peuvent différer. |
| ServiceTier | meterSubCategory | Les valeurs précises des chaînes peuvent différer. |
| StoreServiceIdentifier | N/A | &nbsp;  |
| SubscriptionGuid | subscriptionId | &nbsp;  |
| SubscriptionId | subscriptionId | &nbsp;  |
| SubscriptionName | subscriptionName | &nbsp;  |
| Balises | tags | La propriété des balises s’appliquent à l’objet racine, pas à la propriété de propriétés imbriquées. |
| UnitOfMeasure | unitOfMeasure | Les valeurs précises des chaînes diffèrent. |
| usageEndDate | Date | &nbsp;  |
| Year | None | Analyse l’année à partir de la date. |
| (nouveau) | billingCurrency | Devise utilisée pour les frais. |
| (nouveau) | billingProfileId | ID unique pour le profil de facturation (identique à l’inscription). |
| (nouveau) | billingProfileName | Nom du profil de facturation (identique à l’inscription). |
| (nouveau) | chargeType | Permet de différencier l’utilisation des services Azure, l’utilisation de la Place de marché et les achats. |
| (nouveau) | invoiceId | ID unique de la facture. Vide pour le mois ouvert en cours. |
| (nouveau) | publisherType | Type de serveur de publication pour les achats. Vide pour l’utilisation. |
| (nouveau) | serviceFamily | Type d’achat. Vide pour l’utilisation. |
| (nouveau) | servicePeriodEndDate | Date de fin pour le service acheté. |
| (nouveau) | servicePeriodStartDate | Date de début pour le service acheté. |

## <a name="billing-periods-api-replaced-by-invoices-api"></a>Remplacement de l’API Billing Periods par l’API Invoices

Les comptes de facturation MCA n’utilisent pas de périodes de facturation. Au lieu de cela, ils utilisent des factures pour définir l’étendue des coûts de périodes de facturation spécifiques. L’API [Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) (Périodes de facturation) est remplacé par l’API Invoices (Factures). Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request).

Pour obtenir des factures avec l’API Invoices :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/invoices?api-version=2018-11-01-preview` |

## <a name="price-sheet-apis"></a>API Price Sheet

Cette section décrit les versions de l’API Price Sheet (Grille tarifaire) existantes et fournit des suggestions pour utiliser l’API Price Sheet avec les contrats client Microsoft (MCA). Elle décrit également l’API Price Sheet pour les contrats client Microsoft (MCA) et présente les champs des grilles tarifaires. Les API [Enterprise Get Price Sheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) et [Enterprise Get Billing Periods](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) sont remplacés par l’API Price Sheet pour les contrats client Microsoft (Microsoft.Billing/billingAccounts/billingProfiles/pricesheet). La nouvelle API prend en charge les formats JSON et CSV, dans des formats REST asynchrones. Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request).

### <a name="billing-enterprise-apis"></a>Versions de l’API Billing Enterprise (Facturation d’entreprise)

Les versions de l’API Billing Enterprise étaient utilisées conjointement aux accords de mise en œuvre Entreprise pour obtenir des informations sur les prix et la période de facturation. Les jetons web Azure Active Directory étaient utilisés pour l’authentification et l’autorisation.

Pour obtenir les prix applicables pour l’accord de mise en œuvre d’entreprise spécifié avec les API Price Sheet et Billing Period :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://consumption.azure.com/v2/enrollments/{enrollmentNumber}/pricesheet` |
| GET | `https://consumption.azure.com/v2/enrollments/{enrollmentNumber}/billingPeriods/{billingPeriod}/pricesheet` |

### <a name="price-sheet-api-for-microsoft-customer-agreements"></a>API Price Sheet pour les contrats client Microsoft

Utilisez l’API Price Sheet pour les contrats client Microsoft, afin d’afficher les prix pour tous les services Azure Consumption, ainsi que la consommation de la Place de marché. Les prix indiqués dans le profil de facturation s’appliquent à tous les abonnements qui appartiennent au profil de facturation.

Utiliser l’API Price Sheet pour afficher toutes les données de grille tarifaire Azure Consumption au format CSV :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&startDate=2019-01-01&endDate=2019-01-31&format=csv` |

Utiliser l’API Price Sheet pour afficher toutes les données de grille tarifaire Azure Consumption au format JSON :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&startDate=2019-01-01&endDate=2019-01-31&format=json` |

Utilise l’API pour retourner la grille tarifaire de l’intégralité du compte. Toutefois, vous pouvez également obtenir une version condensée de la grille tarifaire au format PDF. Ce résumé inclut les services Azure Consumption, ainsi que la consommation de la Place de marché, qui sont facturés pour une facture spécifique. La facture est identifiée par le un élément {invoiceId}, qui correspond au **Numéro de facture** indiqué dans les fichiers PDF Invoice Summary (Récapitulatif de la facture). Voici un exemple.

![Capture d’écran représentant le numéro de facture correspondant à l’élément InvoiceId](./media/migrate-cost-management-api/invoicesummary.png)

Pour afficher les informations de facturation avec l’API Price Sheet au format CSV :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/2909cffc-b0a2-5de1-bb7b-5d3383764184/billingProfiles/2dcffe0c-ee92-4265-8647-515b8fe7dc78/invoices/{invoiceId}/pricesheet/default/download?api-version=2018-11-01-preview&format=csv` |

Pour afficher les informations de facturation avec l’API Price Sheet au format JSON :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/2909cffc-b0a2-5de1-bb7b-5d3383764184/billingProfiles/2dcffe0c-ee92-4265-8647-515b8fe7dc78/invoices/{invoiceId}/pricesheet/default/download?api-version=2018-11-01-preview&format=json` |

Vous pouvez également voir les prix estimés pour n’importe quel service Azure Consumption, ainsi que la consommation de la Place de marché, dans le cycle de facturation ou la période du service en cours.

Pour afficher les prix estimés pour les services de consommation avec l’API Price Sheet au format CSV :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billing AccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&format=csv` |

Pour afficher les prix estimés pour les services de consommation avec l’API Price Sheet au format JSON :

| Méthode | URI de demande |
| --- | --- |
| POST | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billing AccountId}/billingProfiles/{billingProfileId}/pricesheet/default/download?api-version=2018-11-01-preview&format=json` |

Les version de l’API Microsoft Customer Agreement Price Sheet sont des *API REST asynchrones*. Les réponses pour les API ont changé par rapport aux API synchrones précédentes. Le corps de la réponse de l’API a aussi changé.

#### <a name="old-response-body"></a>Ancien corps de la réponse

Voici un exemple de la réponse de l’API REST synchrone :

```
[
        {
            "id": "enrollments/573549891/billingperiods/2016011/products/343/pricesheets",
            "billingPeriodId": "201704",
            "meterId": "dc210ecb-97e8-4522-8134-2385494233c0",
            "meterName": "A1 VM",
            "unitOfMeasure": "100 Hours",
            "includedQuantity": 0,
            "partNumber": "N7H-00015",
            "unitPrice": 0.00,
            "currencyCode": "USD"
        },
        {
    ]
```

#### <a name="new-response-body"></a>Nouveau corps de la réponse

Les API prennent en charge le format [Azure REST asynchrone](../../azure-resource-manager/management/async-operations.md). Si vous appelez l’API à l’aide de la méthode GET, vous recevez la réponse suivante :

```
No Response Body

HTTP Status 202 Accepted
```

Les en-têtes suivants sont envoyés à l’emplacement de la sortie :

```
Location:https://management.azure.com/providers/Microsoft.Consumption/operationresults/{operationId}?sessiontoken=XZDFSnvdkbkdsb==

Azure-AsyncOperation:https://managment.azure.com/providers/Microsoft.Consumption/operationStatus/{operationId}?sessiontoken=XZDFSnvdkbkdsb==

Retry-After: 10

OData-EntityId: {operationId}

```

Effectuez un autre appel GET vers l’emplacement. La réponse de l’appel GET est la même, jusqu’à ce que l’opération atteigne un état d’achèvement ou d’échec. Une fois achevé, la réponse à l’appel GET vers l’emplacement retourne l’URL de téléchargement. Comme si l’opération était exécutée simultanément. Voici un exemple :

```
HTTP Status 200

{
  "id": "providers/Microsoft.Consumption/operationresults/{operationId}",
  "name": {operationId},
  "type": “Microsoft.Consumption/operationResults",
  "properties" : {
    "downloadUrl": {urltoblob},
    "validTill": "Date"
  }
}
```

Le client peut également effectuer un appel GET pour `Azure-AsyncOperation`. Le point de terminaison retourne l’état de l’opération.

Le tableau suivant affiche les champs de l’ancienne grille tarifaire de l’API Enterprise Get. Il inclut les champs correspondants dans la nouvelle grille tarifaire des contrats client Microsoft :

| Ancienne propriété | Nouvelle propriété | Notes |
| --- | --- | --- |
| billingPeriodId  | _Non applicable_ | Non applicable. Pour les contrats client Microsoft, la facture et grille tarifaire associée ont remplacé le concept de billingPeriodId. |
| meterId  | meterId | &nbsp;  |
| unitOfMeasure  | unitOfMeasure | Les valeurs précises des chaînes peuvent différer. |
| includedQuantity  | includedQuantity | Non applicable pour les services dans les contrats client Microsoft. |
| partNumber  | _Non applicable_ | Au lieu de cela, utilisez une combinaison de productOrderName (identique à offerID) et meterID. |
| unitPrice  | unitPrice | L’élément unitPrice (prix unitaire) est applicable pour les services utilisés dans les contrats client Microsoft. |
| currencyCode  | pricingCurrency | Les contrats client Microsoft représentent les prix dans les devises de tarification et de facturation. L’élément currencyCode (Code de la devise) correspond à l’élément pricingCurrency (Devise de tarification) dans les contrats client Microsoft. |
| offerID | productOrderName | Au lieu de OfferID, vous pouvez utiliser productOrderName. Cependant, cet élément n’est pas identique à OfferID. Toutefois, les éléments productOrderName et meter déterminent la tarification dans les contrats client Microsoft liées aux éléments meterId et OfferID dans les accords de mise en œuvre hérités. |

## <a name="consumption-price-sheet-api-operations"></a>Opérations de l’API Consumption Price Sheet

Pour les contrats Entreprise, vous avez utilisé les opérations [Get](/rest/api/consumption/pricesheet/get) (Obtenir) et [Get By Billing Period](/rest/api/consumption/pricesheet/getbybillingperiod) (Obtenir par période de facturation) de l’API Consumption Price Sheet pour une étendue selon l’élément subscriptionId ou pour établir une période de facturation. L’API utilise l’authentification Azure Resource Management.

Pour obtenir les informations grille tarifaire tarification pour une étendue avec l’API Price Sheet :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Consumption/pricesheets/default?api-version=2018-10-01` |

Pour obtenir des informations de grille tarifaire avec l’API Price Sheet :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodName}/providers/Microsoft.Consumption/pricesheets/default?api-version=2018-10-01` |

Au lieu des points de terminaison d’API ci-dessus, utilisez les options suivantes pour les contrats client Microsoft :

**API Price Sheet pour les contrats client Microsoft (API REST asynchrone)**

Cette API est pour les contrats client Microsoft et fournit des attributs supplémentaires.

**API Price Sheet pour une étendue de profil de facturation dans un compte de facturation**

Cette API est l’API existante. Elle a été mise à jour afin de fournir la grille tarifaire pour un profil de facturation dans un compte de facturation.

## <a name="price-sheet-for-a-scope-by-billing-account"></a>API Price Sheet pour une étendue par compte de facturation

L’authentification Azure Resource Manager est utilisée lorsque vous obtenez l’API Price Sheet au niveau de l’étendue d’accord de mise en œuvre dans un compte de facturation.

Pour obtenir l’API Price Sheet pour le compte d’accord de mise en œuvre dans une table de tarification dans un compte de facturation :

| Méthode | URI de demande |
| --- | --- |
| GET | `/providers/Microsoft.Billing/billingAccounts/65085863/providers/Microsoft.Consumption/pricesheets/download?api-version=2019-01-01` |

Pour un contrat client Microsoft, utilisez les informations dans la section suivante. Vous y trouverez les propriétés de champ utilisées pour les contrats client Microsoft.

### <a name="price-sheet-for-a-billing-profile-scope-in-a-billing-account"></a>API Price Sheet pour une étendue de profil de facturation dans un compte de facturation

L’API Price Sheet mise à jour par l’API Billing Account obtient la grille tarifaire au format CSV. Pour obtenir la grille tarifaire dans l’étendue de profil de facturation pour un MCA :

| Méthode | URI de demande |
| --- | --- |
| GET | `/providers/Microsoft.Billing/billingAccounts/28ae4b7f-41bb-581e-9fa4-8270c857aa5f/billingProfiles/ef37facb-cd6f-437a-9261-65df15b673f9/providers/Microsoft.Consumption/pricesheets/download?api-version=2019-01-01` |

Au niveau de la portée de l’accord de mise en œuvre, la réponse et les propriétés de l’API sont identiques. Les propriétés correspondent aux mêmes propriétés MCA.

Le tableau suivant répertorie les anciennes propriétés des versions de l’API [Azure Resource Manager Price Sheet](/rest/api/consumption/pricesheet) et les nouvelles propriétés équivalentes.

| Ancienne propriété de l’API Azure Resource Manager Price Sheet  | Nouvelle propriété de l’API Microsoft Customer Agreement Price Sheet   | Description |
| --- | --- | --- |
| ID du compteur | _meterId_ | Identificateur unique du compteur. Identique à meterID. |
| Nom du compteur | meterName | Nom du compteur. Le compteur représente la ressource déployable d’un service Azure. |
| Catégorie du compteur  | service | Nom de la catégorie de classification du compteur. Identique au service de Microsoft Customer Agreement Price Sheet. Les valeurs précises des chaînes diffèrent. |
| Sous-catégorie du compteur | meterSubCategory | Nom de la catégorie de sous-classification du compteur. Basé sur la classification de la différenciation des jeux de caractéristiques de haut niveau dans le service. Par exemple, base de données SQL de Base et base de données SQL standard. |
| Région du compteur | meterRegion | &nbsp;  |
| Unité | _Non applicable_ | Peut être analysé à partir de l’élément unitOfMeasure. |
| Unité de mesure | unitOfMeasure | &nbsp;  |
| Numéro de référence | _Non applicable_ | Au lieu d’un numéro de référence, utilisez productOrderName et MeterID pour identifier de façon unique le prix pour un profil de facturation. Les champs sont répertoriés sur la facture MCA plutôt que le numéro de référence dans les factures MCA. |
| Prix unitaire | unitPrice | Prix unitaire du contrat client Microsoft. |
| Code devise | pricingCurrency | Les contrats client Microsoft représentent les prix dans les devises de tarification et de facturation. Le code de la devise correspond à l’élément pricingCurrency (Devise de tarification) dans les contrats client Microsoft. |
| Quantité incluse | includedQuantity | Non applicable aux services dans les contrats client Microsoft. S’affiche avec des valeurs égales à zéro. |
|  ID de l’offre  | productOrderName | Au lieu de OfferID, utilisez productOrderName. Diffère de l’élément OfferID. Cependant les éléments productOrderName et meter (compteur) déterminent la tarification dans les contrats client Microsoft. Lié aux éléments meterId et OfferID dans les accords de mise en œuvre hérités. |

Le prix pour les contrats client Microsoft n’est pas défini comme celui des contrats Entreprise. Le prix des services lors de l’accord de mise en œuvre Entreprise est unique pour le produit, le numéro de référence, la mesure et l’offre. Le numéro de référence n’est pas utilisé dans les contrats client Microsoft.

Le prix du service Azure Consumption qui fait partie d’un Contrat client Microsoft est unique pour les éléments productOrderName et meterID. Ils représentent le compteur de service et l’offre du produit.

Pour concilier la grille tarifaire et son utilisation dans l’API Usage Details, vous pouvez utiliser les éléments productOrderName et meterID.

Les utilisateurs qui ont des droits de propriétaire de profil de facturation, de contributeur, de lecteur et de gestionnaire de facture peuvent télécharger la grille tarifaire.

La grille tarifaire inclut les prix des services basés sur leur utilisation. Les services incluent Azure Consumption et la consommation de la Place de marché. Le dernier prix à la fin de chaque période de service est verrouillé et appliqué à l’utilisation au cours d’une seule période de service. Pour les services Azure Consumption, la période de service est généralement d’un mois calendaire.

### <a name="retired-price-sheet-api-fields"></a>Champs de l’API Price Sheet supprimés

Les champs suivants ne sont pas disponibles dans l’API Microsoft Customer Agreement Price Sheet ou correspondent à des champs identiques.

|Champ supprimé| Description|
|---|---|
| billingPeriodId | Non applicable. Correspond à l’élément InvoiceId pour un MCA. |
| offerID | Non applicable. Correspond à l’élément productOrderName pour un MCA. |
| meterCategory  | Non applicable. Correspond au à l’élément Service pour un MCA. |
| unité | Non applicable. Peut être analysé à partir de l’élément unitOfMeasure. |
| currencyCode | Identique à l’élément pricingCurrency pour un MCA. |
| meterLocation | Identique à l’élément meterRegion pour un MCA. |
| partNumber (numéro de référence) | Non applicable, car le numéro de référence n’est pas répertorié dans les factures MCA. Au lieu du numéro de référence, utilisez la combinaison des éléments meterId et productOrderName pour identifier les prix de manière unique. |
| totalIncludedQuantity | Non applicable. |
| pretaxStandardRate  | Non applicable. |

## <a name="reservation-instance-charge-api-replaced"></a>Remplacement de l’API Reservation Instance Charge

L’[API Reserved Instance Charge](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges) (Frais d’instance réservés) permet d’obtenir les transactions de facturation pour les achats de réservation. La nouvelle API inclut tous les achats, y compris des offres tierces de la Place de marché. Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request). L’API Reserved Instance Charge est remplacée par l’API Transactions.

Pour obtenir des transactions d’achats de réservations avec l’API Transactions :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}/transactions?api-version=2018-11-01-preview` |

## <a name="recommendations-apis-replaced"></a>Remplacement des versions de l’API Recommendations

Les versions de l’API Reserved Instance Purchase Recommendations fournissent des informations sur l’utilisation de la machine virtuelle sur les 7, 30 ou 60 derniers jours. Les versions de cette API fournissent également des recommandations d’achats de réservations. Ils comprennent :

- [API Shared Reserved Instance Recommendation](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-shared-reserved-instance-recommendations) (Recommandation relative à l’instance réservée partagée)
- [API Single Reserved Instance Recommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation#request-for-single-reserved-instance-recommendations) (Recommandation relative à l’instance réservée unique)

Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request). Les API de recommandations de réservations répertoriées précédemment sont remplacées par l’API [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list).

Pour obtenir des suggestions de réservations avec l’API Reservation Recommendations :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationRecommendations?api-version=2019-01-01` |

## <a name="reservation-usage-apis-replaced"></a>Remplacement de l’API Reservation Usage

L’API Reserved Instance Usage permet d’obtenir des informations sur l’utilisation des réservations dans un accord de mise en œuvre. S’il existe plusieurs instances réservées dans un accord de mise en œuvre, vous pouvez également obtenir l’utilisation de tous les achats de l’instance réservée à l’aide de cette API.

Ils comprennent :

- [Détails de l’utilisation de l’instance réservée](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage#request-for-reserved-instance-usage-details)
- [Résumé de l’utilisation de l’instance réservée](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage)

Toutes les API de consommation sont remplacées par des API Azure natives qui utilisent Azure AD pour l’authentification et l’autorisation. Pour plus d’informations sur l’appel des API REST Azure, consultez [Prise en main de REST](/rest/api/azure/#create-the-request). Les API de suggestions de réservation répertoriées précédemment sont remplacées par les API [Microsoft.Consumption/reservationDetails](/rest/api/consumption/reservationsdetails) et [Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries).

Pour obtenir les détails des réservations avec l’API Reservation Details :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationDetails?api-version=2019-01-01` |

Pour obtenir des résumés des réservations avec l’API Reservation Summaries :

| Méthode | URI de demande |
| --- | --- |
| GET | `https://management.azure.com/providers/Microsoft.Consumption/reservationSummaries?api-version=2019-01-01` |



## <a name="move-from-cloudyn-to-cost-management"></a>Migrer de Cloudyn à Cost Management

Nous recommandons aux organisations qui utilisent [Cloudyn](https://cloudyn.com) de commencer à utiliser [Azure Cost Management](https://azure.microsoft.com/services/cost-management/) pour répondre à leurs besoins de gestion des coûts. Cost Management est disponible dans le Portail Microsoft Azure sans intégration et avec une latence de huit heures. Pour plus d’informations, consultez la page [Documentation de Cost Management](../index.yml).

Grâce à Azure Cost Management, vous pouvez :

- Afficher les coûts au fil du temps par rapport à un budget prédéfini. Analyser tous les jours des modèles de coûts pour identifier et arrêter les dépenses anormales. Décomposer les coûts par balises, groupe de ressources, service et emplacement.
- Créer des budgets pour définir des limites d’utilisation et de coûts, mais aussi recevoir une notification lors de l’approche de seuils importants. Automatiser des groupes d’actions pour déclencher des événements personnalisés et appliquer des limites strictes selon vos conditions.
- Optimiser les coûts et l’utilisation grâce à des suggestions d’Azure Advisor. Découvrir des optimisations d’achat de réservations, éviter la sous-utilisation de vos machines virtuelles et supprimer des ressources inutilisées pour respecter vos budgets.
- Planifier quotidiennement une exportation de données de coût et d’utilisation pour publier un fichier CSV vers votre compte de stockage. Automatiser l’intégration avec des systèmes externes pour conserver les données de facturation synchronisées et à jour.

## <a name="power-bi-integration"></a>Intégration Power BI

Vous pouvez également utiliser Power BI pour la création de rapports de coûts. Le [connecteur Azure Cost Management](/power-bi/desktop-connect-azure-cost-management) pour Power BI Desktop permet de créer des rapports puissants et personnalisés qui vous aideront à mieux comprendre vos dépenses Azure. Le connecteur Azure Cost Management prend actuellement en charge les clients disposant d’un Contrat client Microsoft ou d’un Contrat Entreprise.

## <a name="next-steps"></a>Étapes suivantes

- Consultez la [documentation de Cost Management](../index.yml) pour apprendre à surveiller et contrôler les dépenses Azure. Cette documentation est aussi utile si vous souhaitez optimiser l’utilisation de vos ressources avec Cost Management.
