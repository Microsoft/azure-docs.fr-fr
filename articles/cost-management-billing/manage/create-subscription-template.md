---
title: Créer des abonnements Azure programmatiquement avec un modèle Azure Resource Manager
description: Découvrez comment créer des abonnements Azure programmatiquement en utilisant un modèle Azure Resource Manager.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 11/17/2020
ms.reviewer: andalmia
ms.author: banders
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 92dd129444800dc44e4418fb12d2e4df4aebc02d
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97826146"
---
# <a name="programmatically-create-azure-subscriptions-with-an-azure-resource-manager-template"></a>Créer des abonnements Azure programmatiquement avec un modèle Azure Resource Manager

Cet article vous aide à créer des abonnements Azure programmatiquement en utilisant un modèle Azure Resource Manager (modèle ARM). Les clients Azure disposant d’un compte de facturation pour les types de contrat suivants peuvent créer des abonnements à l’aide d’un modèle ARM :

- Contrat Entreprise (EA)
    - Pour plus d’informations, consultez [API REST Azure Enterprise](ea-portal-rest-apis.md).
- Contrat client Microsoft (MCA)
    - [API REST de facturation Azure](/rest/api/billing)
    - [API d’abonnement Azure](/rest/api/subscription)
- Contrat Partenaire Microsoft (MPA)
    - [API REST de facturation Azure](/rest/api/billing)
    - [API d’abonnement Azure](/rest/api/subscription)

Quand vous créez un abonnement Azure programmatiquement à l’aide d’un modèle, cet abonnement est régi par le contrat dans le cadre duquel vous avez obtenu les services Azure auprès de Microsoft ou d’un revendeur agréé. Pour plus d’informations, consultez [Informations légales relatives à Microsoft Azure](https://azure.microsoft.com/support/legal/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="create-subscriptions-using-arm-templates"></a>Créer des abonnements à l’aide de modèles ARM

Vous pouvez créer des abonnements avec un modèle Azure Resource Manager (modèle ARM), ce qui vous permet d’automatiser vos processus de déploiement de production et de test. Dans l’exemple suivant, vous allez utiliser un modèle ARM pour créer un abonnement et un groupe de ressources Azure.

## <a name="prerequisites"></a>Prérequis

Pour créer des abonnements, vous devez avoir l’un des rôles suivants : 

- Propriétaire de l’abonnement Azure sur une section de facture
- Contributeur de l’abonnement Azure sur une section de facture
- Créateur de l’abonnement Azure sur une section de facture
- Propriétaire de l’abonnement Azure sur un profil de facturation ou un compte de facturation
- Contributeur de l’abonnement Azure sur un profil de facturation ou un compte de facturation

Pour plus d’informations, consultez [Rôles et tâches liés à la facturation des abonnements](understand-mca-roles.md#subscription-billing-roles-and-tasks).

En outre, étant donné que vous effectuez le déploiement d’un modèle ARM, vous devez disposer d’autorisations en écriture sur l’objet racine. Pour créer le déploiement ARM sous un groupe d’administration, vous devez disposer d’autorisations en écriture sur le groupe d’administration. L’action consiste uniquement à créer un déploiement ARM. Si un abonnement est créé, il est créé uniquement dans le groupe d’administration spécifié dans le modèle ARM.

Les exemples suivants utilisent les API REST. Actuellement, PowerShell et Azure CLI ne sont pas pris en charge.

## <a name="find-billing-accounts-that-you-have-access-to"></a>Rechercher les comptes de facturation auxquels vous avez accès

Exécutez la requête suivante pour lister tous les comptes de facturation.

### <a name="rest"></a>[REST](#tab/rest-getBillingAccounts)

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingaccounts/?api-version=2020-05-01
```

En réponse, l’API retourne la liste de tous les comptes de facturation auxquels vous avez accès.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "name": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "billingProfiles": {
          "hasMoreResults": false
        },
        "displayName": "Contoso",
        "hasReadAccess": false
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}
```

Utilisez la propriété `displayName` afin d’identifier le compte de facturation pour lequel vous souhaitez créer des abonnements. Vérifiez que le compte affiche *MicrosoftCustomerAgreement* comme agreementType. Copiez la valeur `name` du compte. Par exemple, si vous souhaitez créer un abonnement pour le compte de facturation `Contoso`, copiez `5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx`. Collez cette valeur quelque part, de façon à pouvoir l’utiliser à l’étape suivante.

<!--
### [PowerShell](#tab/azure-powershell-getBillingAccounts)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.
-->

<!--
### [Azure CLI](#tab/azure-cli-getBillingAccounts)

we're still working on enabling CLI SDK for billing APIs. Check back soon.
-->

---

## <a name="find-billing-profiles--invoice-sections-to-create-subscriptions"></a>Rechercher les profils de facturation et les sections de facture pour créer des abonnements

Les frais de votre abonnement sont indiqués dans une section de la facture d’un profil de facturation. Utilisez l’API suivante pour obtenir la liste des profils de facturation et des sections de facture pour lesquels vous avez l’autorisation de créer des abonnements Azure.

Tout d’abord, vous pouvez obtenir la liste des profils de facturation dans le compte de facturation auquel vous avez accès.

### <a name="rest"></a>[REST](#tab/rest-getBillingProfiles)

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingaccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingprofiles/?api-version=2020-05-01
```

En réponse, l’API retourne la liste de tous les profils de facturation auxquels vous avez accès pour créer des abonnements :

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx",
      "name": "AW4F-xxxx-xxx-xxx",
      "properties": {
        "billingRelationshipType": "Direct",
        "billTo": {
          "addressLine1": "One Microsoft Way",
          "city": "Redmond",
          "companyName": "Contoso",
          "country": "US",
          "email": "kenny@contoso.com",
          "phoneNumber": "425xxxxxxx",
          "postalCode": "98052",
          "region": "WA"
        },
        "currency": "USD",
        "displayName": "Contoso Billing Profile",
        "enabledAzurePlans": [
          {
            "skuId": "0002",
            "skuDescription": "Microsoft Azure Plan for DevTest"
          },
          {
            "skuId": "0001",
            "skuDescription": "Microsoft Azure Plan"
          }
        ],
        "hasReadAccess": true,
        "invoiceDay": 5,
        "invoiceEmailOptIn": false,
        "invoiceSections": {
          "hasMoreResults": false
        },
        "poNumber": "001",
        "spendingLimit": "Off",
        "status": "Active",
        "systemId": "AW4F-xxxx-xxx-xxx",
        "targetClouds": []
      },
      "type": "Microsoft.Billing/billingAccounts/billingProfiles"
    }
  ]
}
```

 Copiez l’`id` pour identifier les sections de facture sous le profil de facturation. Par exemple, copiez `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx` et appelez l’API suivante.

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx/invoicesections?api-version=2020-05-01
```

### <a name="response"></a>response

```json
{
  "totalCount": 1,
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx/invoiceSections/SH3V-xxxx-xxx-xxx",
      "name": "SH3V-xxxx-xxx-xxx",
      "properties": {
        "displayName": "Development",
        "state": "Active",
        "systemId": "SH3V-xxxx-xxx-xxx"
      },
      "type": "Microsoft.Billing/billingAccounts/billingProfiles/invoiceSections"
    }
  ]
}
```

Utilisez la propriété `id` afin d’identifier la section de facture pour laquelle vous souhaitez créer des abonnements. Copiez la chaîne entière. Par exemple, `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx/invoiceSections/SH3V-xxxx-xxx-xxx`. 

<!--
### [PowerShell](#tab/azure-powershell-getBillingProfiles)

we're still working on enabling PowerShell SDK for billing APIs. Check back soon.

### [Azure CLI](#tab/azure-cli-getBillingProfiles)

we're still working on enabling CLI SDK for billing APIs. Check back soon.
-->

---

## <a name="create-a-subscription-and-resource-group-with-a-template"></a>Créer un abonnement et un groupe de ressources à l’aide d’un modèle

Le modèle ARM suivant crée un abonnement nommé *Dev Team subscription* pour la section de facture *Development*. Cet abonnement est facturé au profil de facturation *Contoso Billing Profile* et s’affiche dans la section *Development* de la facture correspondante. Utilisez l’étendue de facturation copiée à l’étape précédente : `/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx/invoiceSections/SH3V-xxxx-xxx-xxx`. 

### <a name="request"></a>Requête

```rest
PUT https://management.azure.com/providers/Microsoft.Resources/deployments/sampleTemplate?api-version=2019-10-01
```

### <a name="request-body"></a>Corps de la demande

```json
{
  "properties":
    {
      "location": "westus",
      "properties": {
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {
            "uniqueAliasName": "sampleAlias"
          },
          "resources": [
            {
              "type": "Microsoft.Resources/deployments",
              "apiVersion": "2019-10-01",
              "name": "sampleTemplate",
              "location": "westus",
              "properties": {
                "expressionEvaluationOptions": {
                  "scope": "inner"
                },
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                  "contentVersion": "1.0.0.0",
                  "variables": {
                    "uniqueAliasName": "sampleAlias"
                  },
                  "resources": [
                    {
                      "name": "[variables('uniqueAliasName')]",
                      "type": "Microsoft.Subscription/aliases",
                      "apiVersion": "2020-09-01",
                      "properties": {
                        "workLoad": "Production",
                        "displayName": "Dev Team subscription",
                        "billingScope": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx_xxxx-xx-xx/billingProfiles/AW4F-xxxx-xxx-xxx/invoiceSections/SH3V-xxxx-xxx-xxx"
                      },
                      "dependsOn": [],
                      "tags": {}
                    }
                  ],
                  "outputs": {
                    "subscriptionId": {
                      "type": "string",
                      "value": "[replace(reference(variables('uniqueAliasName')).subscriptionId, 'invalidrandom/', '')]"
                    }
                  }
                }
              }
            },
            {
              "name": "sampleOuterResource",
              "type": "Microsoft.Resources/deployments",
              "apiVersion": "2019-10-01",
              "location": "westus",
              "properties": {
                "expressionEvaluationOptions": {
                  "scope": "inner"
                },
                "mode": "Incremental",
                "parameters": {
                  "subscriptionId": {
                    "value": "[reference('sampleTemplate').outputs.subscriptionId.value]"
                  }
                },
                "template": {
                  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
              "contentVersion": "1.0.0.0",
                  "parameters": {
                    "subscriptionId": {
                      "type": "string"
                    }
                  },
                  "variables": {},
                  "resources": [
                    {
                      "name": "sampleInnerResource",
                      "type": "Microsoft.Resources/deployments",
                      "subscriptionId": "[parameters('subscriptionId')]",
                      "apiVersion": "2019-10-01",
                      "location": "westus",
                      "properties": {
                        "expressionEvaluationOptions": {
                          "scope": "inner"
                        },
                        "mode": "Incremental",
                        "parameters": {},
                        "template": {
                          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                          "contentVersion": "1.0.0.0",
                          "parameters": {},
                          "variables": {},
                          "resources": [
                            {
                              "type": "Microsoft.Resources/resourceGroups",
                              "apiVersion": "2020-05-01",
                              "location": "[deployment().location]",
                              "name": "sampleRG",
                              "properties": {},
                              "tags": {}
                            }
                          ],
                          "outputs": {}
                        }
                      }
                    }
                  ],
                  "outputs": {}
                }
              }
            }
          ],
          "outputs": {
            "messageFromLinkedTemplate": {
              "type": "string",
              "value": "[reference('sampleTemplate').outputs.subscriptionId.value]"
            }
          }
        },
        "mode": "Incremental"
      }
    }
}
```

### <a name="response"></a>response

```json
{
  "id": "/providers/Microsoft.Resources/deployments/sampleTemplate",
  "name": "sampleTemplate",
  "type": "Microsoft.Resources/deployments",
  "location": "westus",
  "properties": {
    "templateHash": "16005880870587497948",
    "parameters": {},
    "mode": "Incremental",
    "provisioningState": "Accepted",
    "timestamp": "2020-10-07T19:06:34.110811Z",
    "duration": "PT0.1345459S",
    "correlationId": "2b57ddf6-7e27-42cb-90b4-90eeccd11a28",
    "providers": [
      {
        "namespace": "Microsoft.Resources",
        "resourceTypes": [
          {
            "resourceType": "deployments",
            "locations": [
              "westus"
            ]
          }
        ]
      }
    ],
    "dependencies": [
      {
        "dependsOn": [
          {
            "id": "/providers/Microsoft.Resources/deployments/sampleTemplate",
            "resourceType": "Microsoft.Resources/deployments",
            "resourceName": "anuragTemplate1"
          }
        ],
        "id": "/providers/Microsoft.Resources/deployments/sampleOuterResource",
        "resourceType": "Microsoft.Resources/deployments",
        "resourceName": "sampleOuterResource"
      }
    ]
  }
}
```

Vous pouvez connaître l’état du déploiement pour en superviser la progression.

```json
GET https://management.azure.com/providers/Microsoft.Resources/deployments/sampleTemplate?api-version=2019-10-01
```

### <a name="response"></a>response

```json
{
  "id": "/providers/Microsoft.Resources/deployments/sampleDeployment5",
  "name": "sampleDeployment5",
  "type": "Microsoft.Resources/deployments",
  "location": "westus",
  "properties": {
    "templateHash": "16005880870587497948",
    "parameters": {},
    "mode": "Incremental",
    "provisioningState": "Succeeded",
    "timestamp": "2020-10-07T19:07:20.8007311Z",
    "duration": "PT46.824466S",
    "correlationId": "2b57ddf6-7e27-42cb-90b4-90eeccd11a28",
    "providers": [
      {
        "namespace": "Microsoft.Resources",
        "resourceTypes": [
          {
            "resourceType": "deployments",
            "locations": [
              "westus"
            ]
          }
        ]
      }
    ],
    "dependencies": [
      {
        "dependsOn": [
          {
            "id": "/providers/Microsoft.Resources/deployments/sampleTemplate",
            "resourceType": "Microsoft.Resources/deployments",
            "resourceName": "sampleTemplate"
          }
        ],
        "id": "/providers/Microsoft.Resources/deployments/sampleOuterResource",
        "resourceType": "Microsoft.Resources/deployments",
        "resourceName": "sampleOuterResource"
      }
    ],
    "outputs": {
      "messageFromLinkedTemplate": {
        "type": "String",
        "value": "16edf959-11fd-48bb-9a46-85190963ead9"
      }
    },
    "outputResources": [
      {
        "id": "/providers/Microsoft.Subscription/aliases/sampleAlias"
      },
      {
        "id": "/subscriptions/16edf959-11fd-48bb-9a46-85190963ead9/resourceGroups/sampleRG"
      }
    ]
  }
}
```

Dans l’exemple précédent, vous pouvez voir que l’abonnement créé était `16edf959-11fd-48bb-9a46-85190963ead9` et que le groupe de ressources créé était `sampleRG`.

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que vous avez créé un abonnement, vous pouvez accorder cette capacité à d’autres utilisateurs et principaux de service. Pour plus d’informations, consultez [Accorder l’accès pour créer des abonnements Azure Enterprise (préversion)](grant-access-to-create-subscription.md).
* Pour plus d’informations sur la gestion d’un grand nombre d’abonnements à l’aide de groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../governance/management-groups/overview.md).
