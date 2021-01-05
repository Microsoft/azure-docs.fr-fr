---
title: Disponibilité régionale d’Azure Media Services | Microsoft Docs
description: Cet article est une vue d’ensemble des fonctionnalités de Microsoft Azure Media Services et de la disponibilité régionale du service.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: inhenkel
ms.custom: references_regions
ms.openlocfilehash: cf84e45ff43643c7e9a983b867194a7920aa1b57
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93351835"
---
# <a name="media-services-regional-availability"></a>Disponibilité régionale de Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. Découvrez la dernière version, [Media Services v3](../latest/media-services-overview.md). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Microsoft Azure Media Services (AMS) vous permet de charger, de stocker, d’encoder et d’empaqueter en toute sécurité du contenu vidéo ou audio destiné à être diffusé à la demande ou en streaming en direct sur différents clients (par exemple, téléviseurs, PC et appareils mobiles).

AMS fonctionne dans plusieurs régions du monde entier, ce qui vous donne la possibilité de choisir où créer vos applications. Cet article est une vue d’ensemble des fonctionnalités de Microsoft Azure Media Services et de la disponibilité régionale du service.

Pour plus d’informations sur l’ensemble de l’infrastructure globale Azure, consultez [Zones géographiques Azure](https://azure.microsoft.com/global-infrastructure/geographies/).

## <a name="ams-accounts"></a>Comptes AMS

Utilisez la page [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=media-services&regions=all) pour déterminer si Media Services est disponible dans une région spécifique.

## <a name="streaming-endpoints"></a>Points de terminaison de streaming

En fonction de leurs besoins, les clients Media Services peuvent choisir un point de terminaison de streaming **Standard** ou **Premium**.

|Nom|Statut|Région
|---|---|---|
|standard|GA|Tous|
|Premium|GA|Tous|

## <a name="live-encoding"></a>Encodage en direct

Disponible dans toutes les régions, sauf : Allemagne, Brésil Sud, Inde Ouest, Inde Sud et Inde Centre.

## <a name="encoding-media-processors"></a>Processeurs multimédia d’encodage

AMS offre deux encodeurs à la demande : **Media Encoder Standard** et **Media Encoder Premium Workflow**. Pour plus d’informations, consultez [Vue d’ensemble et comparaison d’encodeurs multimédia à la demande Azure](media-services-encode-asset.md).

|Nom du processeur multimédia|Statut|Régions
|---|---|---|
|Media Encoder Standard|GA|Tous|
|Media Encoder Premium Workflow|GA|Tout sauf la Chine|

## <a name="analytics-media-processors"></a>Processeurs multimédias Analytics

Media Analytics est une collection de composants visuels et vocaux qui aident les organisations et les entreprises à extraire des connaissances exploitables de leurs fichiers vidéo. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Media Analytics](./legacy-components.md).

> [!NOTE]
> Certains Processeurs multimédias Analytics seront mis hors service. Pour connaître les dates de mise hors service, voir la rubrique [Composants hérités](legacy-components.md).

|Nom du processeur multimédia|Statut|Région
|---|---|---|
|Détecteur de visage Azure Media|PRÉVERSION|Tous|
|Azure Media Indexer|GA|Tous|
|Détecteur de mouvement Azure Media|PRÉVERSION|Tous|
|Azure Media OCR|PRÉVERSION|Tous|
|Azure Media Redactor|GA|Tous|
|Miniatures vidéo Azure Media|PRÉVERSION|Tous|

## <a name="protection"></a>Protection

Microsoft Azure Media Services vous permet de sécuriser votre contenu multimédia dès lors qu’il quitte votre ordinateur via le stockage, le traitement et la remise. Pour plus d’informations, consultez [Protection du contenu AMS](media-services-content-protection-overview.md).

|Chiffrement|Statut|Régions|
|---|---|---| 
|Stockage|GA|Tous|
|Clés AES-128|GA|Tous|
|Fairplay|GA|Tous|
|PlayReady|GA|Tous|
|Widevine|GA|Tous, sauf les régions Allemagne, Gouvernement fédéral et Chine.

> [!NOTE]
> Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="reserved-units-rus"></a>Unités réservées

Le nombre d’unités réservées approvisionnées détermine le nombre de tâches multimédias qui peuvent être traitées simultanément dans un compte donné.

Disponible dans toutes les régions.

## <a name="reserved-unit-ru-type"></a>Type d’unité réservée

Un compte Media Services est associé à un type d’unité réservé qui détermine la vitesse à laquelle vos tâches de traitement multimédia sont effectuées. Vous pouvez choisir entre les types d’unités réservées suivantes : S1, S2 ou S3.

|Nom du type d’unité réservée|Statut|Régions
|---|---|---|
|S1|GA|Tous|
|S2|GA|Tous, à l’exception des régions Brésil Sud et Inde Ouest|
|S3|GA|Tous, à l’exception de la région Inde Ouest|

## <a name="next-steps"></a>Étapes suivantes

[Migrer vers Media Services V3](../latest/media-services-overview.md)

## <a name="provide-feedback"></a>Fournir des commentaires

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]