---
title: Enregistrer sur de grandes instances SAP HANA avec une réservation Azure
description: Comprenez ce que vous devez savoir avant d’acheter une réservation de grande instance SAP HANA et comment effectuer l’achat.
author: yashesvi
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: how-to
ms.date: 07/24/2020
ms.author: banders
ms.openlocfilehash: d6d0d0a4c4b3328fa50777b5106bac202c9972ef
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151510"
---
# <a name="save-on-sap-hana-large-instances-with-an-azure-reservation"></a>Enregistrer sur de grandes instances SAP HANA avec une réservation Azure

Vous pouvez enregistrer sur vos grandes instances SAP HANA (HLI) lorsque vous préachetez des réservations Azure pour une ou trois années. La remise sur réservation est appliquée à la référence (SKU) HLI approvisionnée qui correspond à l’instance réservée achetée. Cet article vous aide à comprendre ce que vous devez savoir avant d’acheter une réservation et comment effectuer l’achat.

En achetant une réservation, vous vous engagez à utiliser la HLI pendant une ou trois années. L’achat de la capacité de réserve de HLI couvre le calcul et le stockage NFS fournis avec la référence (SKU). La réservation n’inclut pas de coûts de licences de logiciels, tels que le système d’exploitation, SAP ou des coûts de stockage supplémentaires. La remise sur réservation s’applique automatiquement à la HLI SAP approvisionnée. Au terme de la réservation, les tarifs de paiement à l’utilisation s’appliquent à votre ressource approvisionnée.

## <a name="purchase-considerations"></a>Considérations relatives à l’achat

Une référence (SKU) HLI doit être approvisionnée avant de passer à l’achat de capacité de réserve. La réservation est payée à l’avance ou par mensualités. Les restrictions suivantes s’appliquent à la capacité de réserve de HLI :

- Des remises sur réservation s’appliquent uniquement aux abonnement de type Accord Entreprise et Contrat client Microsoft. Les autres abonnements ne sont pas pris en charge.
- La flexibilité de la taille d’instance n’est pas prise en charge pour la capacité de réserve de HLI. Une réservation s’applique uniquement à la référence (SKU) et à la région pour laquelle vous l’achetez.
- L’annulation et l’échange en libre-service ne sont pas pris en charge.
- L’étendue de la capacité de réserve étant unique, elle s’applique à un seul abonnement et un seul groupe de ressources. La capacité achetée ne peut pas être mise à jour pour être utilisée par un autre abonnement.
- Vous ne pouvez pas avoir d’étendue de réservation partagée pour une capacité de réserve HANA. Vous ne pouvez pas fractionner, fusionner ou mettre à jour une étendue de réservation.
- Vous pouvez acheter une seule HLI à la fois à l’aide des appels d’API de capacité de réserve. Effectuez des appels d’API supplémentaires pour acheter des quantités supplémentaires.

Vous pouvez acheter une capacité de réserve via le portail Azure ou à l’aide de l’[API REST](/rest/api/reserved-vm-instances/reservationorder/purchase).

## <a name="buy-a-hana-large-instance-reservation"></a>Acheter une réservation de grande instance HANA

Utilisez les informations suivantes pour acheter une réservation de HLI avec les [API REST d’ordre de réservation](/rest/api/reserved-vm-instances/reservationorder/purchase).

### <a name="get-the-reservation-order-and-price"></a>Obtenir l’ordre et le prix de réservation

Tout d’abord, récupérez l’ordre et le prix de réservation de la référence (SKU) de grande instance HANA approvisionnée à l’aide de l’API [Calculer le prix](/rest/api/reserved-vm-instances/reservationorder/calculate).

L’exemple suivant utilise la commande [armclient](https://github.com/projectkudu/ARMClient) pour effectuer des appels d’API REST avec PowerShell. Voici à quoi doivent ressembler l’ordre de réservation et la demande ainsi que le corps de la demande de l’API Calculer le prix de prix :

```azurepowershell-interactive
armclient post /providers/Microsoft.Capacity/calculatePrice?api-version=2018-06-01 "{
    'sku': {
        'name': 'SAP_HANA_On_Azure_S224om'
    },
    'location': 'eastus',
    'properties': {
        'reservedResourceType': 'SapHana',
        'billingScopeId': '/subscriptions/11111111-1111-1111-111111111111',
        'term': 'P1Y',
        'quantity': '1',
        'displayName': 'testreservation_S224om',
        'appliedScopes': ['/subscriptions/11111111-1111-1111-111111111111'],
        'appliedScopeType': 'Single',
       'instanceFlexibility': 'NotSupported'
    }
}"
```

Pour plus d’informations sur les champs de données et leurs descriptions, consultez [Champs de réservation de HLI](#hli-reservation-fields).

L’exemple de réponse suivant ressemble à ce qui est retourné. Notez la valeur que vous avez retournée pour `quoteId`.

```
{
  "properties": {
    "currencyCode": "USD",
    "netTotal": 313219.0,
    "taxTotal": 0.0,
    "isTaxIncluded": false,
    "grandTotal": 313219.0,
    "purchaseRequest": {
      "sku": {
        "name": "SAP_HANA_On_Azure_S224om"
      },
      "location": "eastus",
      "properties": {
        "billingScopeId": "/subscriptions/11111111-1111-1111-111111111111",
        "term": "P1Y",
        "billingPlan": "Upfront",
        "quantity": 1,
        "displayName": "testreservation_S224om",
        "appliedScopes": [
          "/subscriptions/11111111-1111-1111-111111111111"
        ],
        "appliedScopeType": "Single",
        "reservedResourceType": "SapHana",
        "instanceFlexibility": "NotSupported"
      }
    },
    "quoteId": "d0fd3a890795",
    "isBillingPartnerManaged": true,
    "reservationOrderId": "22222222-2222-2222-2222-222222222222",
    "skuTitle": "SAP HANA on Azure Large Instances - S224om - US East",
    "skuDescription": "SAP HANA on Azure Large Instances, S224om",
    "pricingCurrencyTotal": {
      "currencyCode": "USD",
      "amount": 313219.0
    }
  }
}
```

### <a name="make-your-purchase"></a>Effectuer votre achat

Effectuez votre achat à l’aide du `quoteId` retourné et du `reservationOrderId` que vous avez obtenus dans la section précédente [Obtenir l’ordre et le prix de réservation](#get-the-reservation-order-and-price).

Voici un exemple de demande :

```azurepowershell-interactive
armclient put /providers/Microsoft.Capacity/reservationOrders/22222222-2222-2222-2222-222222222222?api-version=2018-06-01 "{
    'sku': {
        'name': 'SAP_HANA_On_Azure_S224om'
    },
    'location': 'eastus',
    'properties': {
        'reservedResourceType': 'SapHana',
        'billingScopeId': '/subscriptions/11111111-1111-1111-111111111111',
        'term': 'P1Y',
        'quantity': '1',
        'displayName': ' testreservation_S224om',
        'appliedScopes': ['/subscriptions/11111111-1111-1111-111111111111/resourcegroups/123'],
        'appliedScopeType': 'Single',
       'instanceFlexibility': 'NotSupported',
       'renew': true,
       'quoteId': 'd0fd3a890795'
    }
}"
```

Voici un exemple de réponse. Si la commande a été passée correctement, la valeur `provisioningState` doit être `creating`.

```
{
  "id": "/providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222",
  "type": "Microsoft.Capacity/reservationOrders",
  "name": "22222222-2222-2222-2222-222222222222",
  "etag": 1,
  "properties": {
    "displayName": "testreservation_S224om",
    "requestDateTime": "2020-07-14T05:42:34.3528353Z",
    "term": "P1Y",
    "provisioningState": "Creating",
    "reservations": [
      {
        "sku": {
          "name": "SAP_HANA_On_Azure_S224om"
        },
        "id": "/providers/microsoft.capacity/reservationOrders22222222-2222-2222-2222-222222222222/reservations/33333333-3333-3333-3333-3333333333333",
        "type": "Microsoft.Capacity/reservationOrders/reservations",
        "name": "22222222-2222-2222-2222-222222222222/33333333-3333-3333-3333-3333333333333",
        "etag": 1,
        "location": "eastus”
        "properties": {
          "appliedScopes": [
            "/subscriptions/11111111-1111-1111-111111111111/resourcegroups/123"
          ],
          "appliedScopeType": "Single",
          "quantity": 1,
          "provisioningState": "Creating",
          "displayName": " testreservation_S224om",
          "effectiveDateTime": "2020-07-14T05:42:34.3528353Z",
          "lastUpdatedDateTime": "2020-07-14T05:42:34.3528353Z",
          "reservedResourceType": "SapHana",
          "instanceFlexibility": "NotSupported",
          "skuDescription": "SAP HANA on Azure Large Instances – S224om - US East",
          "renew": true
        }
      }
    ],
    "originalQuantity": 1,
    "billingPlan": "Upfront"
  }
}
```

### <a name="verify-purchase-status-success"></a>Vérifier la réussite du statut de l’achat

Exécutez la demande GET d’ordre de réservation pour afficher le statut du bon de commande. `provisioningState` doit être `Succeeded`.

```azurepowershell-interactive
armclient get /providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222?api-version=2018-06-01
```

La réponse doit ressembler à l’exemple suivant.

```
{
  "id": "/providers/microsoft.capacity/reservationOrders/44444444-4444-4444-4444-444444444444",
  "type": "Microsoft.Capacity/reservationOrders",
  "name": "22222222-2222-2222-2222-222222222222 ",
  "etag": 8,
  "properties": {
    "displayName": "testreservation_S224om",
    "requestDateTime": "2020-07-14T05:42:34.3528353Z",
    "createdDateTime": "2020-07-14T05:44:47.157579Z",
    "expiryDate": "2021-07-14",
    "term": "P1Y",
    "provisioningState": "Succeeded",
    "reservations": [
      {
        "id": "/providers/microsoft.capacity/reservationOrders/22222222-2222-2222-2222-222222222222/reservations/33333333-3333-3333-3333-3333333333333"
      }
    ],
    "originalQuantity": 1,
    "billingPlan": "Upfront"
  }
}
```

## <a name="hli-reservation-fields"></a>Champs de réservation de HLI

Les informations suivantes expliquent la signification des différents champs de réservation.

  **SKU** Nom de la référence (SKU) de HLI. Il ressemble à `SAP_HANA_On_Azure_<SKUname>`.

  **Emplacement** Régions de HLI disponibles. Pour connaître les régions disponibles, consultez [Références SKU pour SAP HANA sur Azure (grandes instances)](../../virtual-machines/workloads/sap/hana-available-skus.md). Pour obtenir le format de la chaîne d’emplacement, utilisez l’[appel d’API Obtenir des emplacements](/rest/api/resources/subscriptions/listlocations#locationlistresult).

  **Type de ressource réservés** `SapHana`

  **Abonnement** Abonnement utilisé pour payer la réservation. Les coûts de la réservation sont facturés au mode de paiement défini sur l’abonnement. Le type d’abonnement doit être Accord Entreprise (numéros de l’offre : MS-AZR-0017P ou MS-AZR-0148P) ou Contrat client Microsoft. Les frais sont déduits du solde de l’engagement financier, si disponibles, ou facturés comme un dépassement.

  **Étendue** L’étendue de la réservation doit être à portée unique.

  **Terme** Un ou trois ans. Il ressemble à `P1Y` ou à `P3Y`.

  **Quantité** Nombre d’instances achetées pour la réservation. La quantité à acheter est une HLI à la fois. Pour des réservations supplémentaires, répétez l’appel d’API avec les champs correspondants.

## <a name="troubleshoot-errors"></a>Résoudre les erreurs

Il se peut que vous receviez un message d’erreur semblable à l’exemple suivant lorsque vous achetez une réservation. La cause possible est que la HLI n’est pas approvisionnée pour l’achat. Dans ce cas, contactez votre équipe de compte Microsoft pour obtenir une HLI approvisionnée avant d’essayer d’acheter une réservation.

```
{
  "error": {
    "code": "BadRequest",
    "message": "Capacity check or quota check failed. Please select a different subscription or 
location. You can also go to https://aka.ms/corequotaincrease to learn about quota increase."
  }
} 
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [Comment appeler des API REST Azure avec Postman et cURL](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman).
- Consultez [Références SKU pour SAP HANA sur Azure (grandes instances)](../../virtual-machines/workloads/sap/hana-available-skus.md) pour la liste de références (SKU) disponibles et les régions.