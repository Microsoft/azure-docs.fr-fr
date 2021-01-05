---
title: Migrer les API de rapports d’entreprise vers les API Azure Resource Manager
description: Cet article vous aide à comprendre les différences qui existent entre les API de création de rapports et les API Azure Resource Manager, ce qui se passe quand vous migrez vers les API Azure Resource Manager, et les nouvelles fonctionnalités qui sont disponibles avec les nouvelles API Azure Resource Manager.
author: bandersmsft
ms.reviewer: adwise
ms.service: cost-management-billing
ms.subservice: common
ms.topic: reference
ms.date: 11/19/2020
ms.author: banders
ms.openlocfilehash: 93dda4fc3a152b0a07a95ff327c9ea619f25787c
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96355814"
---
# <a name="migrate-from-enterprise-reporting-to-azure-resource-manager-apis"></a>Migrer les API de rapports d’entreprise vers les API Azure Resource Manager

Cet article aide les développeurs qui ont créé des solutions personnalisées à l’aide des [API de création de rapports Azure pour les clients Entreprise](../manage/enterprise-api.md) à migrer vers les API Azure Resource Manager pour Cost Management. La prise en charge du principal de service des nouvelles API Azure Resource Manager est en disponibilité générale. Les API Azure Resource Manager sont en cours de développement. Envisagez de migrer vers ces API plutôt que d’utiliser les anciennes API de création de rapports Azure pour les clients Entreprise. Les anciennes API sont en cours de dépréciation. Cet article vous aide à comprendre les différences qui existent entre les API de création de rapports et les API Azure Resource Manager, ce qui se passe quand vous migrez vers les API Azure Resource Manager, et les nouvelles fonctionnalités qui sont disponibles avec les nouvelles API Azure Resource Manager.

## <a name="api-differences"></a>Différences entre les API

Les informations suivantes décrivent les différences existant entre les anciennes API de création de rapports pour les clients Entreprise et les nouvelles API Azure Resource Manager.

| **Utilisation** | **API Contrat Entreprise** | **API Azure Resource Manager** |
| --- | --- | --- |
| Authentification | Clé API provisionnée dans le portail Contrat Entreprise (EA) | Authentification Azure Active Directory (Azure AD) au moyen de jetons utilisateur ou de principaux de service. Les principaux de service remplacent les clés API. |
| Étendues et autorisations | Toutes les requêtes se trouvent dans l’étendue de l’inscription. Les attributions d’autorisations de clé d’API détermineront si les données de l’ensemble de l’inscription, de l’intégralité d’un service ou d’un compte spécifique seront retournées. Aucune authentification utilisateur. | Les utilisateurs ou les principaux du service se voient attribuer un accès à l’étendue de l’inscription, du service ou du compte. |
| Point de terminaison d’URI | https://consumption.azure.com | https://management.azure.com |
| État du développement | En mode de maintenance. En voie de dépréciation. | En cours de développement. |
| API disponibles | Limitées à ce qui est actuellement disponible | Des API équivalentes sont disponibles pour remplacer chaque API EA. <p> D’autres API [Gestion des coûts](/rest/api/cost-management/) sont également à votre disposition, notamment : <p> <ul><li>Budgets</li><li>Alertes<li>Exports</li></ul> |

## <a name="migration-checklist"></a>Liste des éléments à vérifier pour la migration

- Familiarisez-vous avec les [API REST Azure Resource Manager](/rest/api/azure).
- Identifiez les API EA que vous utilisez et voyez vers quelles API Azure Resource Manager vous souhaitez les passer, à la section [Mappage d’API EA vers les nouvelles API Azure Resource Manager](#ea-api-mapping-to-new-azure-resource-manager-apis).
- Configurer l’authentification et l’autorisation du service pour les API Azure Resource Manager
  - Si vous n’utilisez pas déjà des API Azure Resource Manager, [inscrivez votre application cliente auprès d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad). L’inscription crée un principal de service que vous pouvez utiliser pour appeler les API.
  - Affectez l’accès du principal de service aux étendues nécessaires, comme indiqué ci-dessous.
  - Mettez à jour le code de programmation pour utiliser l’[authentification Azure AD](/rest/api/azure/#create-the-request) avec votre principal de service.
- Testez les API, puis mettez à jour le code de programmation pour remplacer les appels d’API EA par des appels d’API Azure Resource Manager.
- Mettez à jour la gestion des erreurs pour utiliser de nouveaux codes d’erreur. Entre autres considérations à prendre en compte :
  - Les API Azure Resource Manager présentent un délai d’expiration de 60 secondes.
  - Les API Azure Resource Manager comportent une limitation du débit en place. Une erreur de limitation 429 en découle en cas de dépassement. Créez vos solutions pour ne pas placer trop d’appels d’API sur une brève période.
- Passez en revue les autres API de gestion des coûts disponibles par le biais d’Azure Resource Manager et évaluez-les pour une utilisation ultérieure. Pour plus d’informations, consultez [Utiliser des API de gestion des coûts supplémentaires](#use-additional-cost-management-apis).

## <a name="assign-service-principal-access-to-azure-resource-manager-apis"></a>Affecter au principal de service l’accès aux API Azure Resource Manager

Après avoir créé un principal de service pour appeler par programme les API Azure Resource Manager, vous devez lui attribuer les autorisations appropriées pour autoriser et exécuter des requêtes dans Azure Resource Manager. Il existe deux frameworks d’autorisation pour différents scénarios.

### <a name="azure-billing-hierarchy-access"></a>Accès à la hiérarchie de facturation Azure

Pour affecter des autorisations de principal de service à vos étendues de compte de facturation d’entreprise, de services ou de compte d’inscription, utilisez les API [Autorisations de facturation](/rest/api/billing/2019-10-01-preview/billingpermissions), [Définitions de rôle de facturation](/rest/api/billing/2019-10-01-preview/billingroledefinitions) et [Attributions de rôle de facturation](/rest/api/billing/2019-10-01-preview/billingroleassignments).

- Utilisez les API d’autorisations de facturation pour identifier les autorisations déjà détenues par un principal de service sur une étendue donnée, telle qu’un service ou un compte de facturation.
- Utilisez les API de définitions de rôle de facturation pour répertorier les rôles disponibles qui peuvent être affectés à votre principal de service.
  - Seuls les rôles d’administrateur EA en lecture seule et les rôles d’administrateur de service en lecture seule peuvent être attribués aux principaux de service pour l’instant.
- Utilisez les API d’attributions de rôle de facturation pour affecter un rôle à votre principal de service.

L’exemple suivant montre comment appeler l’API Attributions de rôles pour accorder à un principal de service un accès à votre compte de facturation. Nous vous conseillons d’utiliser [PostMan](https://postman.com) pour effectuer ces configurations d’autorisation ponctuelles.

```json
POST https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/createBillingRoleAssignment?api-version=2019-10-01-preview
```

#### <a name="request-body"></a>Corps de la requête

```json
{
  "principalId": "00000000-0000-0000-0000-000000000000",
  "billingRoleDefinitionId": "/providers/Microsoft.Billing/billingAccounts/{billingAccountName}/providers/Microsoft.Billing/billingRoleDefinition/10000000-aaaa-bbbb-cccc-100000000000"
}

```

### <a name="azure-role-based-access-control"></a>Contrôle d'accès en fonction du rôle Azure

La nouvelle prise en charge du principal de service s’étend aux étendues propres à Azure, telles que les groupes d’administration, les abonnements et les groupes de ressources. Vous pouvez affecter des autorisations de principal de service à ces étendues, directement [dans le portail Azure](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) ou à l’aide d’[Azure PowerShell](../../active-directory/develop/howto-authenticate-service-principal-powershell.md#assign-the-application-to-a-role).

## <a name="ea-api-mapping-to-new-azure-resource-manager-apis"></a>Mappage d’API EA vers les nouvelles API Azure Resource Manager

Consultez le tableau ci-dessous pour identifier les API EA dont vous vous servez actuellement, et l’API Azure Resource Manager de substitution à utiliser à la place.

| **Scénario** | **API EA** | **API Azure Resource Manager** |
| --- | --- | --- |
| Récapitulatif du solde | [/balancesummary](/rest/api/billing/enterprise/billing-enterprise-api-balance-summary) |[Microsoft.Consumption/balances](/rest/api/consumption/balances/getbybillingaccount) |
| Price Sheet | [/pricesheet](/rest/api/billing/enterprise/billing-enterprise-api-pricesheet) | [Microsoft.Consumption/pricesheets/default](/rest/api/consumption/pricesheet) – utiliser pour les prix négociés <p> [API Prix de vente au détail](/rest/api/cost-management/retail-prices/azure-retail-prices) – utiliser pour les prix de vente au détail |
| Informations détaillées sur l’instance réservée | [/reservationdetails](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.CostManagement/generateReservationDetailsReport](/rest/api/cost-management/generatereservationdetailsreport) |
| Récapitulatif de l’instance réservée | [/reservationsummaries](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-usage) | [Microsoft.Consumption/reservationSummaries](/rest/api/consumption/reservationssummaries/list#reservationsummariesdailywithbillingaccountid) |
| Recommendations sur l’instance réservée | [/SharedReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation)<p>[/SingleReservationRecommendations](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation) | [Microsoft.Consumption/reservationRecommendations](/rest/api/consumption/reservationrecommendations/list) |
| Frais de l’instance réservée | [/reservationcharges](/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-charges) | [Microsoft.Consumption/reservationTransactions](/rest/api/consumption/reservationtransactions/list) |

## <a name="migration-details-by-api"></a>Détails de la migration par API

Les sections suivantes montrent les anciens exemples de requête d’API avec les nouveaux exemples d’API les remplaçant.

### <a name="balance-summary-api"></a>API de récapitulatif du solde

Utilisez les URI de requête suivants lors de l’appel de la nouvelle API de récapitulatif du solde. Votre numéro d’inscription doit être utilisé en tant que billingAccountId.

#### <a name="supported-requests"></a>Requêtes prises en charge

[Obtenir l’inscription](/rest/api/consumption/balances/getbybillingaccount)

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.Consumption/balances?api-version=2019-10-01
```

### <a name="response-body-changes"></a>Modifications du corps de la réponse

_Ancien corps de la réponse_ :

```json
{
        "id": "enrollments/100/billingperiods/201507/balancesummaries",
          "billingPeriodId": 201507,
          "currencyCode": "USD",
          "beginningBalance": 0,
          "endingBalance": 1.1,
          "newPurchases": 1,
          "adjustments": 1.1,
          "utilized": 1.1,
          "serviceOverage": 1,
          "chargesBilledSeparately": 1,
          "totalOverage": 1,
          "totalUsage": 1.1,
          "azureMarketplaceServiceCharges": 1,
          "newPurchasesDetails": [
            {
              "name": "",
              "value": 1
            }
          ],
          "adjustmentDetails": [
            {
              "name": "Promo Credit",
              "value": 1.1
            },
            {
              "name": "SIE Credit",
              "value": 1.0
            }
          ]
    }

```

_Nouveau corps de la réponse_ :

Les mêmes données sont désormais disponibles dans le champ `properties` de la réponse de la nouvelle API. Des modifications mineures de l’orthographe de certains noms de champs sont possibles.

```json
{
  "id": "/providers/Microsoft.Billing/billingAccounts/123456/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/balances/balanceId1",
  "name": "balanceId1",
  "type": "Microsoft.Consumption/balances",
  "properties": {
    "currency": "USD  ",
    "beginningBalance": 3396469.19,
    "endingBalance": 2922371.02,
    "newPurchases": 0,
    "adjustments": 0,
    "utilized": 474098.17,
    "serviceOverage": 0,
    "chargesBilledSeparately": 0,
    "totalOverage": 0,
    "totalUsage": 474098.17,
    "azureMarketplaceServiceCharges": 609.82,
    "billingFrequency": "Month",
    "priceHidden": false,
    "newPurchasesDetails": [
      {
        "name": "Promo Purchase",
        "value": 1
      }
    ],
    "adjustmentDetails": [
      {
        "name": "Promo Credit",
        "value": 1.1
      },
      {
        "name": "SIE Credit",
        "value": 1
      }
    ]
  }
}

```

### <a name="price-sheet"></a>Price Sheet

Utilisez les URI de requête suivants lors de l’appel de la nouvelle API de grille tarifaire.

#### <a name="supported-requests"></a>Requêtes prises en charge

 Vous pouvez appeler l’API à l’aide des étendues suivantes :

- Inscription : `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`
- Abonnement : `subscriptions/{subscriptionId}`

[_Obtenir la période de facturation actuelle_](/rest/api/consumption/pricesheet/get)


```json
https://management.azure.com/{scope}/providers/Microsoft.Consumption/pricesheets/default?api-version=2019-10-01
```

[_Obtenir la période de facturation spécifiée_](/rest/api/consumption/pricesheet/getbybillingperiod)

```json
https://management.azure.com/{scope}/providers/Microsoft.Billing/billingPeriods/{billingPeriodName}/providers/Microsoft.Consumption/pricesheets/default?api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Modifications du corps de la réponse

_Ancienne réponse_ :

```json
      [
        {
              "id": "enrollments/57354989/billingperiods/201601/products/343/pricesheets",
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
              "id": "enrollments/57354989/billingperiods/201601/products/2884/pricesheets",
              "billingPeriodId": "201404",
            "meterId": "dc210ecb-97e8-4522-8134-5385494233c0",
              "meterName": "Locally Redundant Storage Premium Storage - Snapshots - AU East",
              "unitOfMeasure": "100 GB",
              "includedQuantity": 0,
              "partNumber": "N9H-00402",
              "unitPrice": 0.00,
              "currencyCode": "USD"
        },
        ...
    ]

```

_Nouvelle réponse_  :

Les anciennes données se trouvent désormais dans le champ `pricesheets` de la réponse de la nouvelle API. Des informations détaillées sur le compteur sont également fournies.

```json
{
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Billing/billingPeriods/201702/providers/Microsoft.Consumption/pricesheets/default",
  "name": "default",
  "type": "Microsoft.Consumption/pricesheets",
  "properties": {
    "nextLink": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/providers/microsoft.consumption/pricesheets/default?api-version=2018-01-31&$skiptoken=AQAAAA%3D%3D&$expand=properties/pricesheets/meterDetails",
    "pricesheets": [
      {
        "billingPeriodId": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Billing/billingPeriods/201702",
        "meterId": "00000000-0000-0000-0000-000000000000",
        "unitOfMeasure": "100 Hours",
        "includedQuantity": 100,
        "partNumber": "XX-11110",
        "unitPrice": 0.00000,
        "currencyCode": "EUR",
        "offerId": "OfferId 1",
        "meterDetails": {
          "meterName": "Data Transfer Out (GB)",
          "meterCategory": "Networking",
          "unit": "GB",
          "meterLocation": "Zone 2",
          "totalIncludedQuantity": 0,
          "pretaxStandardRate": 0.000
        }
      }
    ]
  }
}

```

### <a name="reserved-instance-usage-details"></a>Détails de l’utilisation de l’instance réservée

Microsoft ne travaille pas activement sur les API des détails de la réservation synchrone. Nous vous conseillons de passer au modèle plus récent d’appel d’API asynchrone qui est pris en charge par SPN, dans le cadre de la migration. Les requêtes asynchrones permettent de mieux gérer les grandes quantités de données, et elles réduiront les erreurs de délai d’attente.

#### <a name="supported-requests"></a>Requêtes prises en charge

Utilisez les URI de requête suivants lors de l’appel de la nouvelle API des détails de la réservation asynchrone. Votre numéro d’inscription doit être utilisé en tant que `billingAccountId`. Vous pouvez appeler l’API avec les étendues suivantes :

- Inscription : `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`

#### <a name="sample-request-to-generate-a-reservation-details-report"></a>Exemple de requête pour générer un rapport sur les détails de la réservation

```json
POST 
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.CostManagement/generateReservationDetailsReport?startDate={startDate}&endDate={endDate}&api-version=2019-11-01

```

#### <a name="sample-request-to-poll-report-generation-status"></a>Exemple de requête pour interroger l’état de création du rapport

```json
GET
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.CostManagement/reservationDetailsOperationResults/{operationId}?api-version=2019-11-01

```

#### <a name="sample-poll-response"></a>Exemple de réponse d’interrogation

```json
{
  "status": "Completed",
  "properties": {
    "reportUrl": "https://storage.blob.core.windows.net/details/20200911/00000000-0000-0000-0000-000000000000?sv=2016-05-31&sr=b&sig=jep8HT2aphfUkyERRZa5LRfd9RPzjXbzB%2F9TNiQ",
    "validUntil": "2020-09-12T02:56:55.5021869Z"
  }
}

```

#### <a name="response-body-changes"></a>Modifications du corps de la réponse

La réponse de l’ancienne API des détails de la réservation synchrone est indiquée ci-dessous.

_Ancienne réponse_ :

```json
{
    "reservationOrderId": "00000000-0000-0000-0000-000000000000",
    "reservationId": "00000000-0000-0000-0000-000000000000",
    "usageDate": "2018-02-01T00:00:00",
    "skuName": "Standard_F2s",
    "instanceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/resourvegroup1/providers/microsoft.compute/virtualmachines/VM1",
    "totalReservedQuantity": 18.000000000000000,
    "reservedHours": 432.000000000000000,
    "usedHours": 400.000000000000000
}

```

_Nouvelle réponse_  :

La nouvelle API crée un fichier CSV pour vous. Voyez les champs de fichier suivants.

| **Ancienne propriété** | **Nouvelle propriété** | **Remarques** |
| --- | --- | --- |
|  | InstanceFlexibilityGroup | Nouvelle propriété pour la flexibilité d’instance. |
|  | InstanceFlexibilityRatio | Nouvelle propriété pour la flexibilité d’instance. |
| instanceId | InstanceName |  |
|  | Kind | C’est une nouvelle propriété. La valeur est `None`, `Reservation` ou `IncludedQuantity`. |
| reservationId | ID de réservation |  |
| reservationOrderId | ReservationOrderId |  |
| reservedHours | ReservedHours |  |
| skuName | skuName |  |
| totalReservedQuantity | TotalReservedQuantity |  |
| usageDate | UsageDate |  |
| usedHours | UsedHours |  |

### <a name="reserved-instance-usage-summary"></a>Résumé de l’utilisation de l’instance réservée

Utilisez les URI de requête suivants pour appeler la nouvelle API des récapitulatifs de la réservation.

#### <a name="supported-requests"></a>Requêtes prises en charge

 Appelez l’API avec les étendues suivantes :

- Inscription : `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`

[_Obtenir le récapitulatif de la réservation quotidiennement_](/rest/api/consumption/reservationssummaries/list#reservationsummariesdailywithbillingaccountid)

```json
https://management.azure.com/{scope}/Microsoft.Consumption/reservationSummaries?grain=daily&$filter=properties/usageDate ge 2017-10-01 AND properties/usageDate le 2017-11-20&api-version=2019-10-01
```

[_Obtenir le récapitulatif de la réservation mensuellement_](/rest/api/consumption/reservationssummaries/list#reservationsummariesmonthlywithbillingaccountid)

```json
https://management.azure.com/{scope}/Microsoft.Consumption/reservationSummaries?grain=daily&$filter=properties/usageDate ge 2017-10-01 AND properties/usageDate le 2017-11-20&api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Modifications du corps de la réponse

_Ancienne réponse_ :

```json
[
     {
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "reservationId": "00000000-0000-0000-0000-000000000000",
        "skuName": "Standard_F1s",
        "reservedHours": 24,
        "usageDate": "2018-05-01T00:00:00",
        "usedHours": 23,
        "minUtilizationPercentage": 0,
        "avgUtilizationPercentage": 95.83,
        "maxUtilizationPercentage": 100
    }
]

```

_Nouvelle réponse_  :

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/12345/providers/Microsoft.Consumption/reservationSummaries/reservationSummaries_Id1",
      "name": "reservationSummaries_Id1",
      "type": "Microsoft.Consumption/reservationSummaries",
      "tags": null,
      "properties": {
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "reservationId": "00000000-0000-0000-0000-000000000000",
        "skuName": "Standard_B1s",
        "reservedHours": 720,
        "usageDate": "2018-09-01T00:00:00-07:00",
        "usedHours": 0,
        "minUtilizationPercentage": 0,
        "avgUtilizationPercentage": 0,
        "maxUtilizationPercentage": 0
      }
    }
  ]
}

```

### <a name="reserved-instance-recommendations"></a>API des recommandations relatives à l’instance réservée

Utilisez les URI de requête suivants pour appeler la nouvelle API des recommandations relatives à la réservation.

#### <a name="supported-requests"></a>Requêtes prises en charge

 Appelez l’API avec les étendues suivantes :

- Inscription : `providers/Microsoft.Billing/billingAccounts/{billingAccountId}`
- Abonnement : `subscriptions/{subscriptionId}`
- Groupes de ressources : `subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`

[_Obtenir des recommandations_](/rest/api/consumption/reservationrecommendations/list)

La recommandation relative à l’étendue partagée et celle se rapportant à l’étendue unique sont disponibles par l’intermédiaire de cette API. Vous pouvez également filtrer sur l’étendue sous la forme d’un paramètre d’API facultatif.

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/123456/providers/Microsoft.Consumption/reservationRecommendations?api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Modifications du corps de la réponse

Les recommandations relatives à l’étendue partagée et à l’étendue unique sont combinées en une seule API.

_Ancienne réponse_ :

```json
[{
    "subscriptionId": "1111111-1111-1111-1111-111111111111",
    "lookBackPeriod": "Last7Days",
    "meterId": "2e3c2132-1398-43d2-ad45-1d77f6574933",
    "skuName": "Standard_DS1_v2",
    "term": "P1Y",
    "region": "westus",
    "costWithNoRI": 186.27634908960002,
    "recommendedQuantity": 9,
    "totalCostWithRI": 143.12931642978083,
    "netSavings": 43.147032659819189,
    "firstUsageDate": "2018-02-19T00:00:00"
}
]

```

_Nouvelle réponse_  :

```json
{
  "value": [
    {
      "id": "billingAccount/123456/providers/Microsoft.Consumption/reservationRecommendations/00000000-0000-0000-0000-000000000000",
      "name": "00000000-0000-0000-0000-000000000000",
      "type": "Microsoft.Consumption/reservationRecommendations",
      "location": "westus",
      "sku": "Standard_DS1_v2",
      "kind": "legacy",
      "properties": {
        "meterId": "00000000-0000-0000-0000-000000000000",
        "term": "P1Y",
        "costWithNoReservedInstances": 12.0785105,
        "recommendedQuantity": 1,
        "totalCostWithReservedInstances": 11.4899644807748,
        "netSavings": 0.588546019225182,
        "firstUsageDate": "2019-07-07T00:00:00-07:00",
        "scope": "Shared",
        "lookBackPeriod": "Last7Days",
        "instanceFlexibilityRatio": 1,
        "instanceFlexibilityGroup": "DSv2 Series",
        "normalizedSize": "Standard_DS1_v2",
        "recommendedQuantityNormalized": 1,
        "skuProperties": [
          {
            "name": "Cores",
            "value": "1"
          },
          {
            "name": "Ram",
            "value": "1"
          }
        ]
      }
    },
   ]
}

```

### <a name="reserved-instance-charges"></a>Frais de l’instance réservée

Utilisez les URI de requête suivants pour appeler la nouvelle API des frais de l’instance réservée.

#### <a name="supported-requests"></a>Requêtes prises en charge

[_Obtenir les frais de la réservation par plage de dates_](/rest/api/consumption/reservationtransactions/list)

```json
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/providers/Microsoft.Consumption/reservationTransactions?$filter=properties/eventDate+ge+2020-05-20+AND+properties/eventDate+le+2020-05-30&api-version=2019-10-01
```

#### <a name="response-body-changes"></a>Modifications du corps de la réponse

_Ancienne réponse_ :

```json
[
    {
        "purchasingEnrollment": "string",
        "armSkuName": "Standard_F1s",
        "term": "P1Y",
        "region": "eastus",
        "PurchasingsubscriptionGuid": "00000000-0000-0000-0000-000000000000",
        "PurchasingsubscriptionName": "string",
        "accountName": "string",
        "accountOwnerEmail": "string",
        "departmentName": "string",
        "costCenter": "",
        "currentEnrollment": "string",
        "eventDate": "string",
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "description": "Standard_F1s eastus 1 Year",
        "eventType": "Purchase",
        "quantity": int,
        "amount": double,
        "currency": "string",
        "reservationOrderName": "string"
    }
]

```
_Nouvelle réponse_  :

```json
{
  "value": [
    {
      "id": "/billingAccounts/123456/providers/Microsoft.Consumption/reservationtransactions/201909091919",
      "name": "201909091919",
      "type": "Microsoft.Consumption/reservationTransactions",
      "tags": {},
      "properties": {
        "eventDate": "2019-09-09T19:19:04Z",
        "reservationOrderId": "00000000-0000-0000-0000-000000000000",
        "description": "Standard_DS1_v2 westus 1 Year",
        "eventType": "Cancel",
        "quantity": 1,
        "amount": -21,
        "currency": "USD",
        "reservationOrderName": "Transaction-DS1_v2",
        "purchasingEnrollment": "123456",
        "armSkuName": "Standard_DS1_v2",
        "term": "P1Y",
        "region": "westus",
        "purchasingSubscriptionGuid": "11111111-1111-1111-1111-11111111111",
        "purchasingSubscriptionName": "Infrastructure Subscription",
        "accountName": "Microsoft Infrastructure",
        "accountOwnerEmail": "admin@microsoft.com",
        "departmentName": "Unassigned",
        "costCenter": "",
        "currentEnrollment": "123456",
        "billingFrequency": "recurring"
      }
    },
  ]
}

```

## <a name="use-additional-cost-management-apis"></a>Utiliser des API de gestion des coûts supplémentaires

Après la migration vers les API Azure Resource Manager pour vos scénarios de création de rapports existants, vous pouvez également utiliser de nombreuses autres API. Les API sont également disponibles par le biais d’Azure Resource Manager ; elles peuvent être automatisées à l’aide de l’authentification basée sur le principal de service. Voici un résumé rapide des nouvelles fonctionnalités que vous pouvez utiliser.

- [Budgets](/rest/api/consumption/budgets/createorupdate) – À utiliser pour définir des seuils afin de, à un stade précoce, superviser vos coûts, alerter les parties prenantes concernées et automatiser les actions en réponse à des dépassements de seuil.

- [Alertes](/rest/api/cost-management/alerts) – À utiliser pour afficher des informations sur des alertes, notamment mais sans s’y limiter sur des alertes budgétaires, de facturation, de crédit et de quota.

- [Exportations](/rest/api/cost-management/exports) – À utiliser pour planifier l’exportation périodique des données de vos frais vers un compte de stockage Azure de votre choix. C’est la solution recommandée pour les clients ayant une grande présence Azure, qui souhaitent analyser leurs données et les utiliser dans leurs propres systèmes internes.

## <a name="next-steps"></a>Étapes suivantes

- Familiarisez-vous avec les [API REST Azure Resource Manager](/rest/api/azure).
- Si nécessaire, identifiez les API EA que vous utilisez et voyez vers quelles API Azure Resource Manager vous souhaitez les passer, à la section [Mappage d’API EA vers les nouvelles API Azure Resource Manager](#ea-api-mapping-to-new-azure-resource-manager-apis).
- Si vous n’utilisez pas déjà des API Azure Resource Manager, [inscrivez votre application cliente auprès d’Azure AD](/rest/api/azure/#register-your-client-application-with-azure-ad).
- Au besoin, mettez à jour votre code de programmation pour utiliser l’[authentification Azure AD](/rest/api/azure/#create-the-request) avec votre principal de service.