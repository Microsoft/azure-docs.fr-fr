---
title: Considérations sur les API – Place de marché Azure
description: Gestion des versions, des erreurs et des problèmes d’autorisation lors de l’utilisation des API de la Place de marché.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 0920bec160874f27c8b1b6f2132951b57719b31c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88035278"
---
# <a name="api-considerations"></a>Considérations relatives aux API

<a name="api-versioning"></a>Contrôle de version d’API
--------------

> [!NOTE]
> Les API de Portail Cloud Partner sont intégrées à Espace partenaires et continueront d’y fonctionner. La transition introduit de légères modifications. Passez en revue les changements répertoriés dans les [informations de référence relatives aux API de Portail Cloud Partner](./cloud-partner-portal-api-overview.md) pour vous assurer que votre code continue de fonctionner après la transition vers Espace partenaires. Les API de Portail Cloud Partner doivent uniquement être utilisées pour les produits existants intégrés avant la transition vers Espace partenaires. Les nouveaux produits doivent utiliser les API de soumission d’Espace partenaires.

Plusieurs versions de l’API peuvent être disponibles simultanément. Les clients doivent indiquer la version qu’ils souhaitent invoquer en fournissant le paramètre `api-version` en tant que partie de la chaîne de requête.

   `GET https://cloudpartner.azure.com/api/offerTypes?api-version=2017-10-31`

Le code HTTP 400 est émis en cas de réponse à une requête avec une version d’API inconnue ou invalide. Cette erreur renvoie la collection des versions connues de l’API dans le corps de la réponse.

``` json
    {
        "error": { 
            "code":"InvalidAPIVersion",
            "message":"Invalid api version. Allowed values are [2016-08-01-preview]"
        }
    }
```            

<a name="errors"></a>Erreurs
------

L’API répond aux erreurs avec les codes d’état HTTP correspondants et renvoie, éventuellement, des informations supplémentaires dans la réponse sérialisée au format JSON.
Lorsque vous recevez une erreur, en particulier une erreur de classe 400, ne réexécutez pas la requête avant d’avoir corrigé la cause sous-jacente. Par exemple, dans l’exemple de réponse ci-dessus, corrigez le paramètre de version de l’API avant de renvoyer la requête.

<a name="authorization-header"></a>En-tête d’autorisation.
--------------------

Pour toutes les API abordées dans cette référence, vous devez transmettre l’en-tête d’autorisation ainsi que le jeton du porteur obtenu auprès d’Azure Active Directory (Azure AD). Cet en-tête est nécessaire pour recevoir une réponse valide ; s’il n’est pas fourni, une erreur `401 Unauthorized` est renvoyée. 

``` HTTP
  GET https://cloudpartner.azure.com/api/offerTypes?api-version=2016-08-01-preview

    Accept: application/json 
    Authorization: Bearer <YOUR_TOKEN> 
```
