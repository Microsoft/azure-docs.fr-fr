---
title: Suivre le solde de crédit Azure d’un Contrat client Microsoft
description: Découvrez comment vérifier le solde de crédit Azure d’un Contrat client Microsoft.
author: bandersmsft
ms.reviewer: amberb
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 08/20/2019
ms.author: banders
ms.openlocfilehash: cf5772e9cf08eaa7f34acd59a9c96da6f20cd03d
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131409"
---
# <a name="track-microsoft-customer-agreement-azure-credit-balance"></a>Suivre le solde de crédit Azure d’un Contrat client Microsoft

Vous pouvez vérifier le solde de crédit Azure pour votre compte de facturation d’un Contrat client Microsoft dans le portail Azure ou via des API REST.

Dans le compte de facturation associé à un Contrat Client Microsoft, les crédits sont attribués à un profil de facturation. Chaque profil de facturation a ses propres crédits qui sont automatiquement appliqués aux frais sur sa facture. Vous devez avoir un rôle de propriétaire, contributeur, lecteur ou gestionnaire de factures sur le profil de facturation ou bien un rôle de propriétaire, contributeur ou lecteur sur le compte de facturation pour voir le solde de crédits Azure d’un profil de facturation. Pour en savoir plus sur les rôles, consultez [Présentation des rôles d’administrateur Azure dans le cadre des Contrats client Microsoft](understand-mca-roles.md).

> [!NOTE]
> Le nouveau crédit apparaît dans le portail Azure avec un délai pouvant atteindre 24 heures. Si vous obtenez un nouveau crédit et que vous ne le voyez pas dans le portail, patientez 24 heures.

Cet article s'applique à un compte de facturation associé à un contrat client Microsoft. [Vérifiez que vous avez accès à un contrat client Microsoft](#check-access-to-a-microsoft-customer-agreement).

## <a name="check-your-credit-balance"></a>Vérifier votre solde de crédit

### <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Recherchez **Gestion des coûts + facturation**.

    ![Capture d’écran montrant la recherche de « gestion des coûts + facturation » dans le portail.](./media/mca-check-azure-credits-balance/billing-search-cost-management-billing.png)

3. Dans la page des étendues de facturation, sélectionnez le compte de facturation pour lequel vous souhaitez suivre le solde de crédit. Le compte de facturation doit être de type **Contrat client Microsoft**.

    ![Capture d’écran montrant les étendues de facturation.](./media/mca-check-azure-credits-balance/list-of-scopes.png)

    > [!NOTE]
    >
    > Le portail Azure mémorise la dernière étendue de facturation à laquelle vous accédez et affiche l’étendue la prochaine fois que vous revenez à la page Gestion des coûts + facturation. Vous ne verrez pas la page des étendues de facturation si vous avez déjà consulté Gestion des coûts + facturation. Si c’est le cas, vérifiez que vous êtes dans l’[étendue appropriée](#check-access-to-a-microsoft-customer-agreement). Si ce n’est pas le cas, [changez d’étendue](view-all-accounts.md#switch-billing-scope-in-the-azure-portal) pour sélectionner le compte de facturation d’un Contrat client Microsoft.

3. Sélectionnez **Modes de paiement** sur le côté gauche, puis **Crédits Azure**.

   ![Capture d’écran des soldes estimé et actuel.](./media/mca-check-azure-credits-balance/mca-payment-methods.png)

4. La page Crédits Azure contient les sections suivantes :

   #### <a name="balance"></a>Balance

   La section Solde affiche le résumé de votre solde de crédit Azure.

   ![Capture d’écran du solde de crédit d’un profil de facturation](./media/mca-check-azure-credits-balance/mca-credit-balance.png)

   | Terme               | Définition                           |
   |--------------------|--------------------------------------------------------|
   | Solde estimé  | Montant estimé de crédits dont vous disposez après prise en compte de toutes les transactions facturées et en attente |
   | Solde actuel    | Montant des crédits depuis votre dernière facture. Les transactions en attente ne sont pas incluses. |

   Lorsque votre solde estimé atteint zéro (0), vous êtes facturé pour toute votre utilisation, y compris pour les produits éligibles aux crédits.

   #### <a name="credits-list"></a>Liste des crédits

   La section Liste des crédits affiche la liste des crédits Azure.

   ![Capture d’écran des listes de crédits d'un profil de facturation](./media/mca-check-azure-credits-balance/mca-credits-list.png)

   | Terme | Définition |
   |---|---|
   | Source | Source d’acquisition du crédit |
   | Date de début | Date d'acquisition du crédit |
   | Date d'expiration | Date d’expiration du crédit |
   | Solde actuel | Solde depuis votre dernière facture |
   | Montant d’origine | Montant d'origine du crédit |
   | Statut | État actuel du crédit. Cet état peut être actif, utilisé, expiré ou en cours d'expiration |

   #### <a name="transactions"></a>Transactions

   La section Transactions affiche toutes les transactions qui ont affecté le solde de vos crédits.

   ![Capture d’écran des transactions de crédits pour un profil de facturation](./media/mca-check-azure-credits-balance/mca-credits-transactions.png)

   | Terme | Définition |
   |---|---|
   | Date de la transaction | Date à laquelle la transaction a eu lieu |
   | Description | Description de la transaction |
   | Montant| Le montant de la transaction |
   | Balance | Solde après la transaction |

    > [!NOTE]
    >
    > Si vous ne voyez pas de crédits Azure dans la page des modes de paiement, soit vous n’avez pas de crédits, soit vous n’avez pas sélectionné l’étendue appropriée. Sélectionnez le compte de facturation qui contient des crédits ou un de ses profils de facturation. Pour savoir comment changer d’étendues, consultez [Changer d’étendues de facturation dans le portail Azure](view-all-accounts.md#switch-billing-scope-in-the-azure-portal).

5. Si vous affichez des crédits Azure au niveau de l’étendue du compte de facturation et que le compte de facturation possède plusieurs profils de facturation, la page Crédits Azure affiche un tableau avec un résumé des crédits Azure pour chaque profil de facturation. Sélectionnez un profil de facturation dans la liste, les modes de paiement, puis les crédits Azure pour afficher les détails d’un profil de facturation.

    ![Capture d’écran de la liste des crédits pour un compte de facturation](./media/mca-check-azure-credits-balance/mca-account-credit-list.png)

### <a name="rest-api"></a>[REST API](#tab/rest)

Vous pouvez utiliser les API [Consommation](/rest/api/billing/) et [Facturation Azure](/rest/api/consumption/) pour obtenir programmatiquement le solde de crédits de votre compte de facturation.

Les exemples ci-dessous utilisent les API REST. Actuellement, PowerShell et Azure CLI ne sont pas pris en charge.

### <a name="find-billing-profiles-you-have-access-to"></a>Rechercher les profils de facturation auxquels vous avez accès

```json
GET https://management.azure.com/providers/Microsoft.Billing/billingAccounts?$expand=billingProfiles&api-version=2019-10-01-preview
```
L’API retourne en réponse une liste de comptes de facturation et des profils de facturation associés.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx",
      "name": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx",
      "properties": {
        "accountId": "5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "accountStatus": "Active",
        "accountType": "Enterprise",
        "agreementType": "MicrosoftCustomerAgreement",
        "billingProfiles": [
          {
            "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx",
            "name": "PBFV-xxxx-xxx-xxx",
            "properties": {
              "address": {
                "addressLine1": "AddressLine1",
                "city": "City",
                "companyName": "CompanyName",
                "country": "Country",
                "postalCode": "xxxxx",
                "region": "Region"
              },
              "currency": "USD",
              "displayName": "Development",
              "hasReadAccess": true,
              "invoiceDay": 5,
              "invoiceEmailOptIn": true
            },
            "type": "Microsoft.Billing/billingAccounts/billingProfiles"
          }
        ],
        "displayName": "Contoso",
        "hasReadAccess": true,
      },
      "type": "Microsoft.Billing/billingAccounts"
    }
  ]
}
```

Utilisez la propriété `displayName` du profil de facturation pour identifier le profil de facturation pour lequel vous voulez vérifier le solde de crédit. Copiez la propriété `id` du profil de facturation. Par exemple, si vous souhaitez vérifier le solde de crédits du profil de facturation **Development**, vous copiez ```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```. Collez cette valeur quelque part, car vous en aurez besoin à l’étape suivante.

### <a name="get-azure-credit-balance"></a>Obtenir le solde de crédits Azure

Effectuez la requête suivante, en remplaçant `<billingProfileId>` par l’élément `id` copié lors de la première étape (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```).

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/credits/balanceSummary?api-version=2019-10-01
```

L’API retourne en réponse le solde estimé et le solde actuel pour le profil de facturation.

```json
{
  "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/credits/balanceSummary/57c2e8df-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "57c2e8df-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.Consumption/credits/balanceSummary",
  "eTag": null,
  "properties": {
    "balanceSummary": {
      "estimatedBalance": {
        "currency": "USD",
        "value": 996.13
      },
      "currentBalance": {
        "currency": "USD",
        "value": 997.87
      }
    },
    "pendingCreditAdjustments": {
      "currency": "USD",
      "value": 0.0
    },
    "expiredCredit": {
      "currency": "USD",
      "value": 0.0
    },
    "pendingEligibleCharges": {
      "currency": "USD",
      "value": -1.74
    }
  }
}
```

| Nom de l'élément  | Description                                                                           |
|---------------|---------------------------------------------------------------------------------------|
| `estimatedBalance` | Montant estimé de vos crédits après prise en compte de toutes les transactions facturées et en attente. |
| `currentBalance`   | Montant des crédits depuis votre dernière facture. Les transactions en attente ne sont pas prises en compte.    |
| `pendingCreditAdjustments`      | Ajustements tels que les remboursements qui ne sont pas encore facturés.  |
| `expiredCredit`      |  Crédits ayant expiré depuis votre dernière facture.  |
| `pendingEligibleCharges`  | Frais éligibles aux crédits qui ne sont pas encore facturés.   |

### <a name="get-list-of-credits"></a>Obtenir la liste des crédits

Effectuez la requête suivante, en remplaçant `<billingProfileId>` par l’élément `id` copié lors de la première étape (```/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```).

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/lots?api-version=2019-10-01
```
L’API retourne en réponse les listes de crédits Azure pour un profil de facturation.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/lots/f2ecfd94-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "f2ecfd94-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/lots",
      "eTag": null,
      "properties": {
        "originalAmount": {
          "currency": "USD",
          "value": 500.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 500.0
        },
        "source": "Azure Promotional Credit",
        "startDate": "09/18/2019 21:47:31",
        "expirationDate": "09/18/2020 21:47:30",
        "poNumber": ""
      }
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/xxxx-xxxx-xxx-xxx/providers/Microsoft.Consumption/lots/4ea40eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "4ea40eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/lots",
      "eTag": null,
      "properties": {
        "originalAmount": {
          "currency": "USD",
          "value": 500.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 497.87
        },
        "source": "Azure Promotional Credit",
        "startDate": "09/18/2019 21:47:31",
        "expirationDate": "09/18/2020 21:47:30",
        "poNumber": ""
      }
    }
  ]
}
```
| Nom de l'élément  | Description                                                                                               |
|---------------|-----------------------------------------------------------------------------------------------------------|
| `originalAmount` | Montant initial des crédits. |
| `closedBalance`   | Solde à la date de la dernière facture.    |
| `source`      | Source définissant qui a acquis les crédits. |
| `startDate`      |  Date d’activation des crédits.  |
| `expirationDate`  | Date d’expiration des crédits.   |
| `poNumber`  | Numéro de bon de commande de la facture sur laquelle les crédits ont été facturés.   |

### <a name="get-transactions-that-affected-credit-balance"></a>Obtenir les transactions ayant impacté le solde de crédits

Effectuez la requête suivante, en remplaçant `<billingProfileId>` par l’élément `id` copié lors de la première étape (```providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx```). Vous devez passer une valeur **startDate**  et une valeur **endDate** pour obtenir les transactions relevant de la période qui vous intéresse.

```json
GET https://management.azure.com<billingProfileId>/providers/Microsoft.Consumption/events?api-version=2019-10-01&startDate=2018-10-01T00:00:00.000Z&endDate=2019-10-11T12:00:00.000Z?api-version=2019-10-01
```
L’API retourne en réponse toutes les transactions qui ont impacté le solde de crédits de votre profil de facturation.

```json
{
  "value": [
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx`/providers/Microsoft.Consumption/events/e2032eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "e2032eb5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/events",
      "eTag": null,
      "properties": {
        "transactionDate": "10/11/2019",
        "description": "Credit eligible charges as of 10/11/2019",
        "newCredit": {
          "currency": "USD",
          "value": 0.0
        },
        "adjustments": {
          "currency": "USD",
          "value": 0.0
        },
        "creditExpired": {
          "currency": "USD",
          "value": 0.0
        },
        "charges": {
          "currency": "USD",
          "value": -1.74
        },
        "closedBalance": {
          "currency": "USD",
          "value": 998.26
        },
        "eventType": "PendingCharges",
        "invoiceNumber": ""
      }
    },
    {
      "id": "/providers/Microsoft.Billing/billingAccounts/5e98e158-xxxx-xxxx-xxxx-xxxxxxxxxxxx:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx_xxxx-xx-xx/billingProfiles/PBFV-xxxx-xxx-xxx/providers/Microsoft.Consumption/events/381efd80-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "name": "381efd80-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "type": "Microsoft.Consumption/events",
      "eTag": null,
      "properties": {
        "transactionDate": "09/18/2019",
        "description": "New credit added on 09/18/2019",
        "newCredit": {
          "currency": "USD",
          "value": 500.0
        },
        "adjustments": {
          "currency": "USD",
          "value": 0.0
        },
        "creditExpired": {
          "currency": "USD",
          "value": 0.0
        },
        "charges": {
          "currency": "USD",
          "value": 0.0
        },
        "closedBalance": {
          "currency": "USD",
          "value": 1000.0
        },
        "eventType": "PendingNewCredit",
        "invoiceNumber": ""
      }
    }
  ]
}
```
| Nom de l'élément  | Description                                                                                               |
|---------------|-----------------------------------------------------------------------------------------------------------|
| `transactionDate` | Date à laquelle la transaction a eu lieu. |
| `description` | Description de la transaction. |
| `adjustments`   | Ajustements de crédits pour la transaction.    |
| `creditExpired`      | Montant des crédits ayant expiré. |
| `charges`      |  Frais associés à la transaction.  |
| `closedBalance`  | Solde après la transaction.   |
| `eventType`  | Le type de transaction.   |
| `invoiceNumber`  | Numéro de la facture sur laquelle la transaction est facturée. Il reste vide si la transaction est en attente.   |

---

## <a name="how-credits-are-used"></a>Utilisation des crédits

Dans un compte de facturation pour un Contrat client Microsoft, vous utilisez les profils de facturation pour gérer vos factures et modes de paiement. Une facture mensuelle est générée pour chaque profil de facturation et les modes de paiement permettent de payer la facture.

Vous attribuez vos crédits acquis à un profil de facturation. Quand une facture est générée pour le profil de facturation, les crédits sont automatiquement appliqués au total des frais afin de calculer le montant que vous devez payer. Vous vous acquittez du montant restant en utilisant le mode de paiement choisi (chèque, virement ou carte de crédit).

## <a name="products-that-arent-covered-by-azure-credits"></a>Produits non couverts par les crédits Azure

 Les produits suivants ne sont pas couverts par vos crédits Azure. Ces produits vous sont facturés, quel que soit votre solde de crédit :

- Canonical
- Citrix XenApp Essentials
- Citrix XenDesktop
- Utilisateur inscrit
- Openlogic
- Utilisateur inscrit de Remote Access Rights XenApp Essentials
- Ubuntu Advantage
- Visual Studio Enterprise (mensuel)
- Visual Studio Enterprise (annuel)
- Visual Studio Professional (mensuel)
- Visual Studio Professional (annuel)
- Produits de la Place de marché Azure
- Plans de support Azure

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Vérifier l'accès à un contrat client Microsoft
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.

Si vous avez toujours besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème.

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre le compte de facturation associé à un Contrat client Microsoft](../understand/mca-overview.md)
- [Comprendre les termes indiqués sur votre facture du contrat client Microsoft](../understand/mca-understand-your-invoice.md)