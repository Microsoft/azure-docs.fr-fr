---
title: Codes d’erreur courants pour Azure Key Vault | Microsoft Docs
description: Codes d’erreur courants pour Azure Key Vault
services: key-vault
author: sebansal
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: reference
ms.date: 09/29/2020
ms.author: mbaldwin
ms.openlocfilehash: 9ae13b88d767e43c425ceb86d0be455cebc0e6ac
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462530"
---
# <a name="common-error-codes-for-azure-key-vault"></a>Codes d’erreur courants pour Azure Key Vault

Les codes d’erreur répertoriés dans le tableau suivant peuvent être renvoyés par une opération sur un coffre de clés Azure.

| Code d'erreur | Message utilisateur |
|--|--|
| VaultAlreadyExists |  Échec de la tentative de création d’un coffre de clés avec le nom spécifié, car le nom est déjà utilisé. Si vous avez récemment supprimé un coffre de clés portant ce nom, il est possible qu’il soit encore à l’état de suppression réversible. Vous pouvez vérifier s’il existe à l’état de suppression réversible [ici](./key-vault-recovery.md?tabs=azure-portal#list-recover-or-purge-a-soft-deleted-key-vault). |
| VaultNameNotValid |  Le nom du coffre doit comporter 24 caractères alphanumériques et commencer par une lettre. |
| AccessDenied |  Il se peut qu’il vous manque des autorisations dans la stratégie d’accès pour effectuer cette opération. |
| ForbiddenByFirewall |  L’adresse du client n’est pas autorisée et l’appelant n’est pas un service approuvé. |
| ConflictError |  Vous demandez plusieurs opérations sur le même élément.  |
| RegionNotSupported |  La région Azure spécifiée n’est pas prise en charge pour cette ressource. |
| SkuNotSupported |  Le type de SKU spécifié n’est pas pris en charge pour cette ressource. |
| ResourceNotFound |  La ressource Azure spécifiée est introuvable. |
| ResourceGroupNotFound | Le groupe de ressources Azure spécifié est introuvable. |
| CertificateExpired |  Vérifiez la date d’expiration et la période de validité du certificat. |


## <a name="next-steps"></a>Étapes suivantes

- Consulter le [Guide du développeur Azure Key Vault](developers-guide.md)
- En savoir plus sur [l’authentification auprès du coffre de clés](authentication.md)