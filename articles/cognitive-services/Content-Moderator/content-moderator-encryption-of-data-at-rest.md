---
title: Chiffrement des données au repos de Content Moderator
titleSuffix: Azure Cognitive Services
description: Chiffrement des données au repos de Content Moderator.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: egeaney
ms.openlocfilehash: 1401108a594e30790e842ec379724603f11d493f
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913702"
---
# <a name="content-moderator-encryption-of-data-at-rest"></a>Chiffrement des données au repos de Content Moderator

Content Moderator chiffre automatiquement vos données lorsqu'elles sont conservées dans le cloud, ce qui vous permet de satisfaire aux exigences de votre organisation en matière de sécurité et de conformité.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Les clés gérées par le client sont disponibles uniquement au niveau tarifaire E0. Pour demander la possibilité d’utiliser des clés gérées par le client, complétez et envoyez le [formulaire de demande de clé gérée par le client Content Moderator](https://aka.ms/cogsvc-cmk). Comptez environ 3 à 5 jours ouvrables pour obtenir des nouvelles sur le statut de votre demande. Selon la demande, vous pouvez être placé dans une file d’attente et approuvé lorsque de l’espace devient disponible. Une fois approuvé pour l’utilisation d’une clé CMK avec le service Content Moderator, vous devez créer une ressource Content Moderator et sélectionner E0 comme niveau tarifaire. Une fois que votre ressource Content Moderator avec le niveau tarifaire E0 est créée, vous pouvez utiliser Azure Key Vault pour configurer votre identité managée.

[!INCLUDE [cognitive-services-cmk](../includes/cognitive-services-cmk-regions.md)]

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="enable-data-encryption-for-your-content-moderator-team"></a>Activer le chiffrement des données pour votre équipe Content Moderator

Afin d’activer le chiffrement des données pour votre équipe de révision Content Moderator, consultez [Démarrage rapide : Essayer Content Moderator sur le web](quick-start.md#create-a-review-team).  

> [!NOTE]
> Vous devez fournir un _ID de ressource_ avec le niveau tarifaire Content Moderator E0.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir la liste complète des services qui prennent en charge CMK, consultez [Clés gérées par le client pour Cognitive Services](../encryption/cognitive-services-encryption-keys-portal.md)
* [Qu’est-ce qu’Azure Key Vault ?](../../key-vault/general/overview.md)
* [Formulaire de demande de clé gérée par le client Cognitive Services](https://aka.ms/cogsvc-cmk)