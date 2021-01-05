---
title: "Récupérer l'API d'une offre spécifique : Place de marché Azure"
description: L’API récupère l’offre spécifiée dans l’espace de noms du serveur de publication.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 9f3ba6b2f13b9f2bb1d538db84723e3a9baaef12
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87271839"
---
# <a name="retrieve-a-specific-offer"></a>Récupérer une offre en particulier

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les changements répertoriés dans les [informations de référence relatives aux API de Portail Cloud Partner](./cloud-partner-portal-api-overview.md) pour vous assurer que votre code continue de fonctionner après la transition vers Espace partenaires. Les API de Portail Cloud Partner doivent uniquement être utilisées pour les produits existants intégrés avant la transition vers Espace partenaires. Les nouveaux produits doivent utiliser les API de soumission d’Espace partenaires.

Récupère l’offre spécifiée dans l’espace de noms du serveur de publication.  

Vous pouvez également récupérer une version spécifique de l’offre ou récupérer l’offre dans les emplacements brouillon, vue ou production. Si aucun emplacement n’est indiqué, la valeur par défaut est `draft`. Toute tentative de récupération d’une offre qui n’a pas été prévisualisée ou publiée entraîne une erreur `404 Not Found`.

> [!WARNING]
> Les valeurs de secret des champs secrets ne seront pas récupérées par cette API.

``` http
    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/versions/<version>?api-version=2017-10-31

    GET https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/slot/<slotId>?api-version=2017-10-31

```

## <a name="uri-parameters"></a>Paramètres URI

| **Nom**    | **Description**                                                                          | **Type de données** |
|-------------|------------------------------------------------------------------------------------------|---------------|
| publisherId | publisherId. Par exemple, Contoso                                                        | String        |
| offerId     | Guid qui identifie de façon unique l’offre.                                                 | String        |
| version     | Version de l’offre en cours de récupération. Par défaut, la dernière version de l’offre est récupérée. | Integer       |
| slotId      | L’emplacement à partir duquel l’offre doit être récupérée. La valeur peut être une des valeurs suivantes :      <br/>  - `Draft` (par défaut) récupère l’offre actuellement en brouillon.  <br/>  -  `Preview` (par défaut) récupère l’offre actuellement en préversion.     <br/>  -  `Production` (par défaut) récupère l’offre actuellement en production.          |      enum |
| api-version | Dernière version de l’API                                                                    | Date          |
|  |  |  |

## <a name="header"></a>En-tête

|  **Nom**          |   **Valeur**            |
|  ---------------   |  --------------        |
|  Content-Type      | `application/json`     |
|  Autorisation     | `Bearer YOUR_TOKEN`    |
|  |  |

## <a name="body-example"></a>Exemple de corps

### <a name="response"></a>response

``` json
{
    "offerTypeId": "microsoft-azure-virtualmachines",
    "publisherId": "contoso",
    "status": "failed",
    "id": "059afc24-07de-4126-b004-4e42a51816fe",
    "version": 5,
    "definition": {
        "displayText": "Contoso Virtual Machine Offer",
        "offer": {
            "microsoft-azure-marketplace-testdrive.enabled": false,
            "microsoft-azure-marketplace-testdrive.videos": [],
            "microsoft-azure-marketplace.title": "Contoso App",
            "microsoft-azure-marketplace.summary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.longSummary": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.description": "Contoso App makes dev ops a breeze",
            "microsoft-azure-marketplace.offerMarketingUrlIdentifier": "contosoapp",
            "microsoft-azure-marketplace.allowedSubscriptions": [
                "59160c40-2e25-4dcf-a2fd-6514cb08bf08"
            ],
            "microsoft-azure-marketplace.usefulLinks": [
            {
                "linkTitle": "Contoso App for Azure",
                "linkUrl": "https://azuremarketplace.microsoft.com"
            }
            ],
                "microsoft-azure-marketplace.categoryMap": [
                {
                "categoryL1": "analytics",
                "categoryL2-analytics": [
                "visualization-and-reporting"
                ]
                },
                {
                "categoryL1": "ai-plus-machine-learning",
                "categoryL2-ai-plus-machine-learning": [
                "bot-services",
                "cognitive-services",
                "other"
                ]
                }
            ],
            "microsoft-azure-marketplace.smallLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/6218c455-9cbc-450c-9920-f2e7a69ee132.png?sv=2014-02-14&sr=b&sig=6O8MM9dgiJ48VK0MwddkyVbprRAnBszyhVkVHGShhkI%3D&se=2019-03-28T19%3A46%3A50Z&sp=r",
            "microsoft-azure-marketplace.mediumLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/557e714b-2f31-4e12-b0cc-e48dd840edf4.png?sv=2014-02-14&sr=b&sig=NwL67NTQf9Gc9VScmZehtbHXpYmxhwZc2foy3o4xavs%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.largeLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/142485da-784c-44cb-9523-d4f396446258.png?sv=2014-02-14&sr=b&sig=xaMxhwx%2FlKYfz33mJGIg8UBdVpsOwVvqhjTJ883o0iY%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.wideLogo": "https://publishingapistore.blob.core.windows.net/testcontent/D6191_publishers_contoso/contosovirtualmachine/48af9013-1df7-4c94-8da8-4626e5039ce0.png?sv=2014-02-14&sr=b&sig=%2BnN7f2tprkrqb45ID6JlT01zXcy1PMTkWXtLKD6nfoE%3D&se=2019-03-28T19%3A46%3A49Z&sp=r",
            "microsoft-azure-marketplace.screenshots": [],
            "microsoft-azure-marketplace.videos": [],
            "microsoft-azure-marketplace.leadDestination": "None",
            "microsoft-azure-marketplace.tableLeadConfiguration": {},
            "microsoft-azure-marketplace.blobLeadConfiguration": {},
            "microsoft-azure-marketplace.salesForceLeadConfiguration": {},
            "microsoft-azure-marketplace.crmLeadConfiguration": {},
            "microsoft-azure-marketplace.httpsEndpointLeadConfiguration": {},
            "microsoft-azure-marketplace.marketoLeadConfiguration": {},
            "microsoft-azure-marketplace.privacyURL": "https://azuremarketplace.microsoft.com",
            "microsoft-azure-marketplace.termsOfUse": "Terms of use",
            "microsoft-azure-marketplace.engineeringContactName": "Jon Doe",
            "microsoft-azure-marketplace.engineeringContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.engineeringContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.supportContactName": "Jon Doe",
            "microsoft-azure-marketplace.supportContactEmail": "jondoe@outlook.com",
            "microsoft-azure-marketplace.supportContactPhone": "555-555-5555",
            "microsoft-azure-marketplace.publicAzureSupportUrl": "",
            "microsoft-azure-marketplace.fairfaxSupportUrl": ""
            },
            "plans": [
            {
                "planId": "contososkuidentifier",
                "microsoft-azure-virtualmachines.skuTitle": "Contoso App",
                "microsoft-azure-virtualmachines.skuSummary": "Contoso App makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.skuDescription": "This is a description for the Contoso App that makes dev ops a breeze.",
                "microsoft-azure-virtualmachines.hideSKUForSolutionTemplate": false,
                "microsoft-azure-virtualmachines.cloudAvailability": [
                    "PublicAzure"
                ],
                "microsoft-azure-virtualmachines.certificationsFairfax": [],
                "virtualMachinePricing": {
                    "isByol": true,
                    "freeTrialDurationInMonths": 0
                },
                "microsoft-azure-virtualmachines.operatingSystemFamily": "Windows",
                "microsoft-azure-virtualmachines.windowsOSType": "Other",
                "microsoft-azure-virtualmachines.operationSystem": "Contoso App",
                "microsoft-azure-virtualmachines.recommendedVMSizes": [
                    "a0-basic",
                    "a0-standard",
                    "a1-basic",
                    "a1-standard",
                    "a2-basic",
                    "a2-standard"
                ],
                "microsoft-azure-virtualmachines.openPorts": [],
                "microsoft-azure-virtualmachines.vmImages": {
                    "1.0.1": {
                    "osVhdUrl": "http://contosoteststorage.blob.core.windows.net/test/contosoVM.vhd?sv=2014-02-14&sig=WlDo6Q4xwYH%2B5QEJbItPUVdgHhBcrVxPBmntZ2vU96w%3D&st=2016-06-25T18%3A30%3A00Z&se=2017-06-25T18%3A30%3A00Z&sp=rl",
                    "lunVhdDetails": []
                    }
                },
                "regions": [
                    "DZ",
                    "AR"
                ]
            }
            ]
        },
        "changedTime": "2017-06-07T06:15:39.7349221Z"
    }
}
```

### <a name="response-body-properties"></a>Propriétés du corps de réponse

|  **Nom**       |   **Description**                                                                                                               |
|  -------------  |   -----------------------------------------------------------------------------------------------------                         |
|  offerTypeId    | Identifie le type d’offre                                                                                                    |
|  publisherId    | Identificateur unique de l’éditeur                                                                                              |
|  status         | État de l’offre. Pour la liste des valeurs possibles, consultez [État de l’offre](#offer-status) ci-dessous.                                  |
|  Id             | GUID qui identifie de façon unique l’offre                                                                                         |
|  version        | Version actuelle de l’offre. La propriété de version ne peut pas être modifiée par le client. Elle est incrémentée après chaque publication.    |
|  Définition     | Définition réelle de la charge de travail                                                                                               |
|  changedTime    | Date/heure UTC de la dernière modification de l’offre                                                                                   |
|  |  |

### <a name="response-status-codes"></a>Codes d’état de réponse

| **Code**  | **Description**                                                                                                                 |
|  ------   | ------------------------------------------------------------------------------------------------------------------------------- |
|  200      | `OK` : la requête a été correctement traitée et toutes les offres de l’éditeur ont été retournées au client.               |
|  400      | `Bad/Malformed request` : le corps de la réponse d’erreur peut contenir plus d’informations.                                                 |
|  403      | `Forbidden` : le client n’a pas accès à l’espace de noms spécifié.                                                        |
|  404      | `Not found` : l’entité spécifiée n’existe pas. Le client doit vérifier les valeurs publisherId et offerId, ainsi que la version (si elle est spécifiée).      |
|  |  |

### <a name="offer-status"></a>État de l’offre

|  **Nom**                   |   **Description**                             |
| --------------------------- |  -------------------------------------------- |
|  NeverPublished             | L’offre n’a jamais été publiée.               |
|  NotStarted                 | L’offre est nouvelle, mais n’a pas démarré.              |
|  WaitingForPublisherReview  | L’offre est en attente d’approbation du serveur de publication.      |
|  Exécution en cours                    | La soumission de l’offre est en cours de traitement.          |
|  Opération réussie                  | La soumission de l’offre a été traitée.    |
|  Opération annulée                   | La soumission de l’offre a été annulée.                |
|  Échec                     | La soumission de l’offre a échoué.                      |
|  |  |
