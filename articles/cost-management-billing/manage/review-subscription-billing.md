---
title: Passer en revue les données de facturation d’un abonnement Azure avec l’API REST
description: Découvrez comment utiliser les API REST Azure pour passer en revue les détails de facturation d’un abonnement. Vous pouvez utiliser des filtres pour mieux personnaliser les résultats.
author: lleonard-msft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: article
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: b4d6502e49fdd30a68188a1e580a1c137984c89f
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132378"
---
# <a name="review-subscription-billing-using-rest-apis"></a>Passer en revue la facturation d’un abonnement avec les API REST

Les API de création de rapports Azure vous aident à passer en revue et à gérer vos coûts Azure.

Des filtres permettent de personnaliser les résultats en fonction de vos besoins.

Vous pouvez découvrir ici comment utiliser les API REST pour retourner les détails de facturation d’un abonnement sur une plage de dates donnée.

``` http
GET https://management.azure.com/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}/providers/Microsoft.Consumption/usageDetails?$filter=properties/usageEnd ge '${startDate}' AND properties/usageEnd le '${endDate}'
Content-Type: application/json
Authorization: Bearer
```

## <a name="build-the-request"></a>Générer la demande

Le paramètre `{subscriptionID}` est obligatoire et identifie l’abonnement cible.

Le paramètre `{billingPeriod}` est obligatoire et spécifie une [période de facturation](/rest/api/billing/enterprise/billing-enterprise-api-billing-periods) active.

Les paramètres `${startDate}` et `${endDate}` sont obligatoires pour cet exemple, mais facultatifs pour le point de terminaison. Ils spécifient la plage de dates sous forme de chaînes, au format AAAA-MM-JJ (exemples : `'20180501'` et `'20180615'`).

Les en-têtes suivants sont requis :

|En-tête de requête|Description|
|--------------------|-----------------|
|*Content-Type :*|Obligatoire. Défini sur `application/json`.|
|*Authorization :*|Obligatoire. Défini sur un [jeton d’accès](/rest/api/azure/#authorization-code-grant-interactive-clients) `Bearer` valide. |

## <a name="response"></a>response

Le code d’état 200 (OK) est retourné pour une réponse correcte, qui contient une liste de coûts détaillés pour votre compte.

``` json
{
  "value": [
    {
      "id": "/subscriptions/{$subscriptionID}/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/usageDetails/{$detailsID}",
      "name": "{$detailsID}",
      "type": "Microsoft.Consumption/usageDetails",
      "properties": {
        "billingPeriodId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/billingPeriods/${billingPeriod}",
        "invoiceId": "/subscriptions/${subscriptionID}/providers/Microsoft.Billing/invoices/${invoiceID}",
        "usageStart": "${startDate}}",
        "usageEnd": "${endDate}",
        "currency": "USD",
        "usageQuantity": "${usageQuantity}",
        "billableQuantity": "${billableQuantity}",
        "pretaxCost": "${cost}",
        "meterId": "${meterID}",
        "meterDetails": "${meterDetails}"
      }
    }
  ],
  "nextLink": "${nextLinkURL}"
}
```

Chaque élément de **value** représente une information détaillée concernant l’utilisation d’un service :

|Propriété de la réponse|Description|
|----------------|----------|
|**subscriptionGuid** | ID global unique pour l’abonnement. |
|**startDate** | Date à laquelle l’utilisation a démarré. |
|**endDate** | Date à laquelle l’utilisation s’est terminée. |
|**useageQuantity** | Quantité utilisée. |
|**billableQuantity** | Quantité réellement facturée. |
|**pretaxCost** | Coût facturé, avant application des taxes. |
|**meterDetails** | Informations détaillées sur l’utilisation. |
|**nextLink**| Quand elle est définie, spécifie une URL pour la « page » suivante d’informations détaillées. Vide quand la page est la dernière. |

Cet exemple est abrégé ; consultez [Liste des détails d’utilisation](/rest/api/consumption/usagedetails/list#usagedetailslistforbillingperiod-legacy) pour une description complète de chaque champ de la réponse.

Les autres codes d’état indiquent les conditions d’erreur. Dans ces cas, l’objet de réponse explique pourquoi la demande a échoué.

``` json
{
  "error": [
    {
      "code": "Error type.",
      "message": "Error response describing why the operation failed."
    }
  ]
}
```

## <a name="next-steps"></a>Étapes suivantes
- Consulter [Présentation des rapports d’entreprise](./enterprise-api.md)
- Examiner [l’API REST de facturation d’entreprise](/rest/api/billing/)
- [Bien démarrer avec l’API REST Azure](/rest/api/azure/)