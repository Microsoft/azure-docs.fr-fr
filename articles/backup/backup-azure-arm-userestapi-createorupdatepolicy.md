---
title: créer des stratégies de sauvegarde à l’aide de l’API REST
description: Dans cet article, vous apprendrez à créer et gérer des stratégies de sauvegarde (planification et rétention) à l’aide de l’API REST.
ms.topic: conceptual
ms.date: 08/21/2018
ms.assetid: 5ffc4115-0ae5-4b85-a18c-8a942f6d4870
ms.openlocfilehash: e4e6f5b5cf28c3830a91a494ea60680eee1546f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89179604"
---
# <a name="create-azure-recovery-services-backup-policies-using-rest-api"></a>Créer des stratégies de sauvegarde Azure Recovery Services à l’aide de l’API REST

Les étapes de création d’une stratégie de sauvegarde pour un coffre Azure Recovery Services sont décrites dans la [documentation sur l’API REST policy](/rest/api/backup/protectionpolicies/createorupdate). Nous allons utiliser ce document comme référence pour créer une stratégie pour la sauvegarde de machine virtuelle Azure.

## <a name="create-or-update-a-policy"></a>Créer ou mettre à jour une stratégie

Pour créer ou mettre à jour une stratégie de sauvegarde Azure, utilisez l’opération *PUT* suivante

```http
PUT https://management.azure.com/Subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.RecoveryServices/vaults/{vaultName}/backupPolicies/{policyName}?api-version=2019-05-13
```

`{policyName}` et `{vaultName}` sont fournis dans l’URI. Vous trouverez des informations supplémentaires dans le corps de la requête.

## <a name="create-the-request-body"></a>Créer le corps de la demande

Par exemple, voici les composants du corps de la requête pour une stratégie de sauvegarde de machine virtuelle Azure.

|Nom  |Obligatoire  |Type  |Description  |
|---------|---------|---------|---------|
|properties     |   True      |  ProtectionPolicy :[AzureIaaSVMProtectionPolicy](/rest/api/backup/protectionpolicies/createorupdate#azureiaasvmprotectionpolicy)      | Propriétés de ProtectionPolicyResource        |
|tags     |         | Object        |  Balises de ressource       |

Pour obtenir la liste complète des définitions de corps de la requête, reportez-vous au [document sur l’API REST backup policy](/rest/api/backup/protectionpolicies/createorupdate).

### <a name="example-request-body"></a>Exemple de corps de demande

Le corps de requête suivant définit une stratégie de sauvegarde pour les sauvegardes de machines virtuelles Azure.

La stratégie indique ce qui suit :

- Effectuer une sauvegarde hebdomadaire chaque lundi, mercredi, jeudi à 10 h 00, heure standard du Pacifique.
- Conserver les sauvegardes effectuées chaque lundi, mercredi, jeudi pendant une semaine.
- Conserver les sauvegardes effectuées chaque premier mercredi et chaque troisième jeudi de chaque mois pendant deux mois (annule les conditions de rétention précédentes, le cas échéant).
- Conserver les sauvegardes effectuées le quatrième lundi et le quatrième jeudi de février et de novembre pendant quatre ans (annule les conditions de rétention précédentes, le cas échéant).

```json
{
  "properties": {
    "backupManagementType": "AzureIaasVM",
    "timeZone": "Pacific Standard Time",
    "schedulePolicy": {
      "schedulePolicyType": "SimpleSchedulePolicy",
      "scheduleRunFrequency": "Weekly",
      "scheduleRunTimes": [
        "2018-01-24T10:00:00Z"
      ],
      "scheduleRunDays": [
        "Monday",
        "Wednesday",
        "Thursday"
      ]
    },
    "retentionPolicy": {
      "retentionPolicyType": "LongTermRetentionPolicy",
      "weeklySchedule": {
        "daysOfTheWeek": [
          "Monday",
          "Wednesday",
          "Thursday"
        ],
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 1,
          "durationType": "Weeks"
        }
      },
      "monthlySchedule": {
        "retentionScheduleFormatType": "Weekly",
        "retentionScheduleWeekly": {
          "daysOfTheWeek": [
            "Wednesday",
            "Thursday"
          ],
          "weeksOfTheMonth": [
            "First",
            "Third"
          ]
        },
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 2,
          "durationType": "Months"
        }
      },
      "yearlySchedule": {
        "retentionScheduleFormatType": "Weekly",
        "monthsOfYear": [
          "February",
          "November"
        ],
        "retentionScheduleWeekly": {
          "daysOfTheWeek": [
            "Monday",
            "Thursday"
          ],
          "weeksOfTheMonth": [
            "Fourth"
          ]
        },
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 4,
          "durationType": "Years"
        }
      }
    }
  }
}
```

> [!IMPORTANT]
> Les formats horaires de la planification et de la rétention prennent en charge uniquement DateTime. Ils ne prennent pas en charge le format Time uniquement.

## <a name="responses"></a>Réponses

La création/mise à jour de la stratégie de sauvegarde est une [opération asynchrone](../azure-resource-manager/management/async-operations.md). ce qui signifie qu’elle crée une autre opération qui doit faire l’objet d’un suivi distinct.

Elle retourne deux réponses : 202 (Accepté) lors de la création d’une autre opération, puis 200 (OK) quand cette opération est terminée.

|Nom  |Type  |Description  |
|---------|---------|---------|
|200 OK     |    [Protection PolicyResource](/rest/api/backup/protectionpolicies/createorupdate#protectionpolicyresource)     |  OK       |
|202 Accepté     |         |     Acceptée    |

### <a name="example-responses"></a>Exemples de réponses

Une fois que vous envoyez la demande *PUT* de création ou de mise à jour de stratégie, la réponse initiale est 202 (Accepté) avec un en-tête d’emplacement ou Azure-async-header.

```http
HTTP/1.1 202 Accepted
Pragma: no-cache
Retry-After: 60
Azure-AsyncOperation: https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SwaggerTestRg/providers/Microsoft.RecoveryServices/vaults/testVault/backupPolicies/testPolicy1/operations/00000000-0000-0000-0000-000000000000?api-version=2016-06-01
X-Content-Type-Options: nosniff
x-ms-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-client-request-id: e1f94eef-9b2d-45c4-85b8-151e12b07d03; e1f94eef-9b2d-45c4-85b8-151e12b07d03
Strict-Transport-Security: max-age=31536000; includeSubDomains
x-ms-ratelimit-remaining-subscription-writes: 1199
x-ms-correlation-request-id: db785be0-bb20-4598-bc9f-70c9428b170b
x-ms-routing-request-id: SOUTHINDIA:20180521T073907Z:db785be0-bb20-4598-bc9f-70c9428b170b
Cache-Control: no-cache
Date: Mon, 21 May 2018 07:39:06 GMT
Location: https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SwaggerTestRg/providers/Microsoft.RecoveryServices/vaults/testVault/backupPolicies/testPolicy1/operationResults/00000000-0000-0000-0000-000000000000?api-version=2019-05-13
X-Powered-By: ASP.NET
```

Effectuez ensuite le suivi de l’opération qui en résulte en utilisant l’en-tête d’emplacement ou Azure-AsyncOperation avec une commande *GET* simple.

```http
GET https://management.azure.com/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SwaggerTestRg/providers/Microsoft.RecoveryServices/vaults/testVault/backupPolicies/testPolicy1/operationResults/00000000-0000-0000-0000-000000000000?api-version=2019-05-13
```

Une fois l’opération terminée, elle retourne 200 (OK) avec le contenu de la stratégie dans le corps de la réponse.

```json
{
  "id": "/Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/SwaggerTestRg/providers/Microsoft.RecoveryServices/vaults/testVault/backupPolicies/testPolicy1",
  "name": "testPolicy1",
  "type": "Microsoft.RecoveryServices/vaults/backupPolicies",
  "properties": {
    "backupManagementType": "AzureIaasVM",
    "schedulePolicy": {
      "schedulePolicyType": "SimpleSchedulePolicy",
      "scheduleRunFrequency": "Weekly",
      "scheduleRunDays": [
        "Monday",
        "Wednesday",
        "Thursday"
      ],
      "scheduleRunTimes": [
        "2018-01-24T10:00:00Z"
      ],
      "scheduleWeeklyFrequency": 0
    },
    "retentionPolicy": {
      "retentionPolicyType": "LongTermRetentionPolicy",
      "weeklySchedule": {
        "daysOfTheWeek": [
          "Monday",
          "Wednesday",
          "Thursday"
        ],
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 1,
          "durationType": "Weeks"
        }
      },
      "monthlySchedule": {
        "retentionScheduleFormatType": "Weekly",
        "retentionScheduleWeekly": {
          "daysOfTheWeek": [
            "Wednesday",
            "Thursday"
          ],
          "weeksOfTheMonth": [
            "First",
            "Third"
          ]
        },
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 2,
          "durationType": "Months"
        }
      },
      "yearlySchedule": {
        "retentionScheduleFormatType": "Weekly",
        "monthsOfYear": [
          "February",
          "November"
        ],
        "retentionScheduleWeekly": {
          "daysOfTheWeek": [
            "Monday",
            "Thursday"
          ],
          "weeksOfTheMonth": [
            "Fourth"
          ]
        },
        "retentionTimes": [
          "2018-01-24T10:00:00Z"
        ],
        "retentionDuration": {
          "count": 4,
          "durationType": "Years"
        }
      }
    },
    "timeZone": "Pacific Standard Time",
    "protectedItemsCount": 0
  }
}
```

Si une stratégie est déjà utilisée pour protéger un élément, toute mise à jour de la stratégie entraîne la [modification de la protection](backup-azure-arm-userestapi-backupazurevms.md#changing-the-policy-of-protection) pour tous ces éléments associés.

## <a name="next-steps"></a>Étapes suivantes

[Activer la protection pour une machine virtuelle Azure non protégée](backup-azure-arm-userestapi-backupazurevms.md).

Pour plus d’informations sur les API REST de la Sauvegarde Azure, voir les documents suivants :

- [API REST du fournisseur Azure Recovery Services](/rest/api/recoveryservices/)
- [Bien démarrer avec l’API REST Azure](/rest/api/azure/)
