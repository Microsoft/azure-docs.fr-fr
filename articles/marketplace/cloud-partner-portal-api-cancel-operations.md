---
title: API d’annulation d’opération – Place de marché commerciale Microsoft
description: API permettant d’annuler une opération en cours sur l’offre
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 095d9cf68374ee81cafdb6e7a32df29a57d122cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87287930"
---
# <a name="cancel-operation"></a>Annuler une opération

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les changements répertoriés dans les [informations de référence relatives aux API de Portail Cloud Partner](./cloud-partner-portal-api-overview.md) pour vous assurer que votre code continue de fonctionner après la transition vers Espace partenaires. Les API de Portail Cloud Partner doivent uniquement être utilisées pour les produits existants intégrés avant la transition vers Espace partenaires. Les nouveaux produits doivent utiliser les API de soumission d’Espace partenaires.

Cette API annule une opération qui est en cours sur l’offre. Utilisez l’[API de récupération d’opérations](./cloud-partner-portal-api-retrieve-operations.md) pour obtenir un `operationId` à transmettre à cette API. L’annulation est généralement une opération synchrone, mais, dans certains scénarios complexes, une nouvelle opération peut être nécessaire pour annuler une opération existante. Dans ce cas, le corps de la réponse HTTP contient l’emplacement de l’opération qui doit être utilisé pour interroger l’état de la requête.

  `POST https://cloudpartner.azure.com/api/publishers/<publisherId>/offers/<offerId>/cancel?api-version=2017-10-31`

## <a name="uri-parameters"></a>Paramètres URI

--------------

|  **Nom**    |      **Description**                                  |    **Type de données**  |
| ------------ |     ----------------                                  |     -----------   |
| publisherId  |  Identificateur du serveur de publication, par exemple `contoso`         |   String          |
| offerId      |  Identificateur de l’offre                                     |   String          |
| api-version  |  Version actuelle de l’API                               |    Date           |
|  |  |  |

## <a name="header"></a>En-tête
------

|  **Nom**              |  **Valeur**         |
|  ---------             |  ----------        |
|  Content-Type          |  application/json  |
|  Autorisation         |  Porteur de VOTRE JETON |
|  |  |

## <a name="body-example"></a>Exemple de corps
------------

### <a name="request"></a>Requête

``` json
{
   "metadata": {
     "notification-emails": "jondoe@contoso.com"
    }
}     
```

### <a name="request-body-properties"></a>Propriétés du corps de la requête

|  **Nom**                |  **Description**                                               |
|  --------                |  ---------------                                               |
|  e-mails de notification     | Liste d’ID d’e-mail séparés par des virgules pour informer de la progression de l’opération de publication. |
|  |  |

### <a name="response"></a>response

#### <a name="migrated-offers"></a>Offres migrées

`Location: /api/publishers/contoso/offers/contoso-offer/operations/56615b67-2185-49fe-80d2-c4ddf77bb2e8?api-version=2017-10-31`

#### <a name="non-migrated-offers"></a>Offres non migrées

`Location: /api/operations/contoso$contoso-offer$2$preview?api-version=2017-10-31`

### <a name="response-header"></a>En-tête de réponse

|  **Nom**             |    **Valeur**                       |
|  ---------            |    ----------                      |
| Emplacement    | Chemin relatif permettant de récupérer l’état de cette opération. |
|  |  |

### <a name="response-status-codes"></a>Codes d’état de réponse

| **Code**  |  **Description**                                                                       |
|  ------   |  ------------------------------------------------------------------------               |
|  200      | OK. La requête a été traitée avec succès et l’opération est annulée de manière synchrone. |
|  202      | Accepté. La requête a été traitée avec succès et l’opération est en cours d’annulation. L’emplacement de l’opération d’annulation est renvoyé dans l’en-tête de la réponse. |
|  400      | Requête inappropriée/mal formée. Le corps de la réponse d’erreur pourrait fournir plus d’informations.  |
|  403      | Accès interdit. Le client n’a pas accès à l’espace de noms spécifié dans la requête. |
|  404      | Introuvable. L’entité spécifiée n’existe pas. |
|  |  |
