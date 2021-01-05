---
title: Télémétrie d’Azure Media Services | Microsoft Docs
description: Cet article donne une vue d’ensemble des données de télémétrie de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 95c20ec4-c782-4063-8042-b79f95741d28
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 0701e9c6428283d45cf4b4a2e24c8de99d9a286b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89265896"
---
# <a name="azure-media-services-telemetry"></a>Télémétrie Azure Media Services  

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Azure Media Services (AMS) vous permet d’accéder aux données de télémétrie/mesure pour ses services. La version actuelle de AMS vous permet de collecter les données de télémétrie pour les entités en temps réel **Channel**, **StreamingEndpoint** et **Archive**. 

Les données de télémétrie sont écrites dans une table de stockage dans un compte de stockage Azure que vous spécifiez (en général, vous utilisez le compte de stockage associé à votre compte AMS). 

Le système de télémétrie ne gère pas la rétention des données. Vous pouvez supprimer les anciennes données de télémétrie en supprimant les tables de stockage.

Cette rubrique explique comment configurer et utiliser la télémétrie AMS.

## <a name="configuring-telemetry"></a>Configuration de la télémétrie

Vous pouvez configurer la télémétrie sur une granularité au niveau du composant. Il existe deux niveaux de détail : « Normal » et « Détaillé ». Actuellement, les deux niveaux renvoient les mêmes informations. Il est recommandé d’utiliser la valeur Normal. 

Les rubriques suivantes montrent comment activer la télémétrie :

[Activation de la télémétrie avec .NET](media-services-dotnet-telemetry.md) 

[Activation de la télémétrie avec REST](media-services-rest-telemetry.md)

## <a name="consuming-telemetry-information"></a>informations sur l’utilisation de la télémétrie

La télémétrie est écrite dans une table de stockage Azure dans le compte de stockage que vous avez spécifié lorsque vous avez configuré la télémétrie pour le compte Media Services. Cette section décrit les tables de stockage pour les mesures.

Vous pouvez utiliser les données de télémétrie ainsi :

- Lire les données à partir d’Azure Table Storage directement (par exemple en utilisant le SDK Storage). Pour obtenir la description des tables de stockage de télémétrie, consultez les **informations sur l’utilisation de la télémétrie** dans [cette](/previous-versions/azure/mt742089(v=azure.100)) rubrique.

ou

- Utilisez le support dans le SDK .NET Media Services pour lire les données de stockage, comme décrit dans [cette](media-services-dotnet-telemetry.md) rubrique. 


Le schéma de télémétrie décrit ci-dessous est conçu pour offrir de bonnes performances dans les limites du stockage de table Azure :

- Les données sont partitionnées par ID de compte et ID de service pour permettre que les données de télémétrie de chaque service soient interrogées indépendamment.
- Les partitions contiennent la date pour donner une limite supérieure raisonnable sur la taille de la partition.
- Les clés de ligne sont dans l’ordre horaire inverse pour autoriser les éléments de télémétrie les plus récents à être interrogés pour un service donné.

Cela devrait permettre d’optimiser la plupart des requêtes courantes :

- Téléchargement parallèle, indépendant de données pour des services distincts.
- Récupération de toutes les données d’un service donné dans une plage de dates.
- Récupération des données les plus récentes pour un service.

### <a name="telemetry-table-storage-output-schema"></a>Schéma de sortie de stockage de table de données de télémétrie

Les données de télémétrie sont stockées en agrégat dans une table, « TelemetryMetrics20160321 », où « 20160321 » est la date de création de la table. Le système de télémétrie crée une table distincte pour chaque nouveau jour basé sur l’heure UTC 00:00. La table est utilisée pour stocker des valeurs récurrentes telles que comme la vitesse de transmission de réception dans une fenêtre de temps donnée, les octets envoyés, etc. 

Propriété|Valeur|Exemples/notes
---|---|---
PartitionKey|{account ID}_{entity ID}|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66<br/<br/>L’ID de compte est inclus dans la clé de partition afin de simplifier les workflows dans lesquels plusieurs comptes Media Services écrivent sur le même compte de stockage.
RowKey|{secondes avant minuit}_{valeur aléatoire}|01688_00199<br/><br/>La clé de ligne commence par le nombre de secondes avant minuit pour autoriser les requêtes supérieures de style n au sein d’une partition. Pour plus d’informations, consultez [cet](../../cosmos-db/table-storage-design-guide.md#log-tail-pattern) article. 
Timestamp|Date/Heure|Timestamp automatique à partir de la table Azure 2016-09-09T22:43:42.241Z
Type|Le type de l’entité offrant les données de télémétrie|Channel/StreamingEndpoint/Archive<br/><br/>Le type d’événement est simplement une valeur de chaîne.
Name|Le nom de l’événement de télémétrie|ChannelHeartbeat/StreamingEndpointRequestLog
ObservedTime|L’heure à laquelle l’événement de télémétrie est survenu (UTC)|2016-09-09T22:42:36.924Z<br/><br/>L’heure observée est fournie par l’entité envoyant les données de télémétrie (par exemple, un canal). Il peut y avoir des problèmes de synchronisation de l’heure entre les composants, cette valeur est donc approximative
ServiceID|{ID de service}|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
Propriétés spécifiques à une entité|Comme définies par l’événement|StreamName: stream1, Bitrate 10123, …<br/><br/>Les propriétés restantes sont définies pour le type d’événement donné. Le contenu de la table Azure est constitué de paires clé / valeur.  (Autrement dit, différentes lignes de la table possèdent différents ensembles de propriétés.)

### <a name="entity-specific-schema"></a>Schéma spécifique à une entité

Il existe trois types d’entrées de données télémétriques spécifiques à une entité, chacune envoyée selon la fréquence suivante :

- Points de terminaison de diffusion en continu : toutes les 30 secondes
- Canaux en temps réel : toutes les minutes
- Archive en temps réel : toutes les minutes

**Point de terminaison de diffusion en continu**

Propriété|Valeur|Exemples
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Timestamp automatique à partir de la table Azure 2016-09-09T22:43:42.241Z
Type|Type|StreamingEndpoint
Name|Name|StreamingEndpointRequestLog
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|ID de service|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
HostName|Nom d’hôte du point de terminaison|builddemoserver.origin.mediaservices.windows.net
StatusCode|État HTTP des enregistrements|200
ResultCode|Détails du code de résultat|S_OK
RequestCount|Demande totale dans l’agrégation|3
BytesSent|Octets agrégés envoyés|2987358
ServerLatency|Latence moyenne du serveur (stockage inclus)|129
E2ELatency|Latence moyenne de bout en bout|250

**Canal en temps réel**

Propriété|Valeur|Exemples/notes
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Timestamp automatique à partir de la table Azure 2016-09-09T22:43:42.241Z
Type|Type|Channel
Name|Name|ChannelHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|ID de service|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
TrackType|Type de piste vidéo/audio/texte|vidéo/audio
TrackName|Nom de la piste|vidéo/audio_1
Bitrate|Suivre la vitesse de transmission|785000
CustomAttributes||   
IncomingBitrate|Vitesse de transmission entrante réelle|784548
OverlapCount|Chevauchement dans la réception|0
DiscontinuityCount|Discontinuité de piste|0
LastTimestamp|Timestamp des dernières données reçues|1800488800
NonincreasingCount|Nombre de fragments rejetés en raison d’un timestamp n’augmentant pas|2
UnalignedKeyFrames|Si nous avons reçu des fragments (parmi les différents niveaux de qualité) où les trames-clés ne sont pas alignées |True
UnalignedPresentationTime|Si nous avons reçu des fragments (parmi les différents niveaux/pistes de qualité) où l’heure de présentation n’est pas alignée|True
UnexpectedBitrate|True, si la vitesse de transmission réelle/calculée pour la piste audio/vidéo > 40 000 bits/s et IncomingBitrate == 0 ou IncomingBitrate et actualBitrate diffèrent de 50 % |True
Healthy|True, si <br/>overlapCount, <br/>DiscontinuityCount, <br/>NonIncreasingCount, <br/>UnalignedKeyFrames, <br/>UnalignedPresentationTime, <br/>UnexpectedBitrate<br/> sont tous 0|True<br/><br/>Healthy est une fonction composite qui retourne la valeur false lorsque l’une des conditions suivantes contient :<br/><br/>- OverlapCount > 0<br/>- DiscontinuityCount > 0<br/>- NonincreasingCount > 0<br/>- UnalignedKeyFrames == True<br/>- UnalignedPresentationTime == True<br/>- UnexpectedBitrate == True

**Archive en temps réel**

Propriété|Valeur|Exemples/notes
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Timestamp automatique à partir de la table Azure 2016-09-09T22:43:42.241Z
Type|Type|Archivage
Name|Name|ArchiveHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|ID de service|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
ManifestName|URL du programme|asset-eb149703-ed0a-483c-91c4-e4066e72cce3/a0a5cfbf-71ec-4bd2-8c01-a92a2b38c9ba.ism
TrackName|Nom de la piste|audio_1
TrackType|Type de la piste|Audio/vidéo
CustomAttribute|Chaîne hexadécimale qui fait la distinction entre des pistes différentes avec les mêmes nom et vitesse de transmission (angles multiples de la caméra)|
Bitrate|Suivre la vitesse de transmission|785000
Healthy|True si FragmentDiscardedCount == 0 && ArchiveAcquisitionError == False|True (ces deux valeurs ne sont pas présentes dans la mesure, mais elles figurent dans l’événement source)<br/><br/>Healthy est une fonction composite qui retourne la valeur false lorsque l’une des conditions suivantes contient :<br/><br/>- FragmentDiscardedCount > 0<br/>- ArchiveAcquisitionError == True

## <a name="general-qa"></a>Questions et réponses d’ordre général

### <a name="how-to-consume-metrics-data"></a>Comment utiliser des données de mesures ?

Les données de mesures sont stockées comme une série de tables Azure dans le compte de stockage du client. Ces données peuvent être utilisées à l’aide des outils suivants :

- Kit de développement logiciel (SDK) AMS
- L’explorateur de stockage Microsoft Azure (prend en charge l’exportation vers un format CSV et traité dans Excel)
- API REST

### <a name="how-to-find-average-bandwidth-consumption"></a>Comment trouver l’utilisation moyenne de bande passante ?

L’utilisation moyenne bande passante est la moyenne des octets envoyés sur un intervalle de temps.

### <a name="how-to-define-streaming-unit-count"></a>Comment définir le nombre d’unités de diffusion en continu ?

Le nombre d’unités de diffusion en continu peut être défini comme le débit maximal à partir des points de terminaison de streaming du service divisé par le débit maximal d’un point de terminaison. Le débit utilisable maximal d’un point de terminaison est de 160 Mbits/s.
Par exemple, supposons que le débit maximal du service d’un client est de 40 Mbits/s (la valeur maximale d’octets envoyés sur un intervalle de temps). Le nombre d’unités de diffusion en continu est égal à (40 Mbits/s) * (8 bits/octet) /(160 Mbits/s) = 2 unités de diffusion en continu.

### <a name="how-to-find-average-requestssecond"></a>Comment trouver des demandes/seconde moyennes ?

Pour trouver le nombre moyen de demandes/seconde, calculez le nombre moyen de demandes (nombre de demandes) sur un intervalle de temps.

### <a name="how-to-define-channel-health"></a>Comment définir l’intégrité du canal ?

L’intégrité du canal peut être définie comme une fonction booléenne composite, telle que fausse lorsque l’une des conditions suivantes contient :

- OverlapCount > 0
- DiscontinuityCount > 0
- NonincreasingCount > 0
- UnalignedKeyFrames == True 
- UnalignedPresentationTime == True 
- UnexpectedBitrate == True


### <a name="how-to-detect-discontinuities"></a>Comment détecter des discontinuités ?

Pour détecter des discontinuités, recherchez toutes les entrées de données de canal où DiscontinuityCount > 0. Le timestamp ObservedTime correspondant indique les heures auxquelles les discontinuités sont survenues.

### <a name="how-to-detect-timestamp-overlaps"></a>Comment détecter les chevauchements de timestamp ?

Pour détecter les chevauchements de timestamp, recherchez toutes les entrées de données de canal où OverlapCount > 0. Le timestamp ObservedTime correspondant indique les heures auxquelles le chevauchement de timestamp est survenu.

### <a name="how-to-find-streaming-request-failures-and-reasons"></a>Comment rechercher les raisons et les échecs de demandes de diffusion en continu ?

Pour rechercher les raisons et les échecs des demandes de diffusion en continu, recherchez toutes les entrées de données de point de terminaison de streaming où ResultCode n’est pas égal à S_OK. Le champ StatusCode correspondant indique la raison de l’échec de la demande.

### <a name="how-to-consume-data-with-external-tools"></a>Comment utiliser des données avec des outils externes ?

Les données télémétriques peuvent être traitées et affichées avec les outils suivants :

- PowerBI
- Application Insights
- Azure Monitor (anciennement Shoebox)
- Tableau de bord dynamique AMS
- Portail Azure (en attente de publication)

### <a name="how-to-manage-data-retention"></a>Comment gérer la rétention des données ?

Le système de télémétrie ne fournit pas de gestion de rétention des données ou de suppression automatique des anciens enregistrements. Par conséquent, vous devez gérer et supprimer manuellement les anciens enregistrements de la table de stockage. Consultez le stockage SDK pour connaître la marche à suivre.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
