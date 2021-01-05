---
title: Streaming en direct avec des encodeurs locaux qui créent des flux multidébits - Azure | Microsoft Docs
description: Cette rubrique explique comment configurer un canal qui reçoit un flux dynamique à débit binaire multiple en provenance d’un encodeur local.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: d9f0912d-39ec-4c9c-817b-e5d9fcf1f7ea
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 746fe9132dcb06678e2a0a975c8eed0aba6c3fad
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89269622"
---
# <a name="working-with-channels-that-receive-multi-bitrate-live-stream-from-on-premises-encoders"></a>Utilisation des canaux recevant un stream multidébit en direct d’encodeurs locaux

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> À partir du 12 mai 2018, les canaux en direct ne prendront plus en charge le protocole de réception du flux de transport RTP/MPEG-2. Effectuez la migration à partir de RTP/MPEG-2 vers le protocole de réception RTMP ou MP4 fragmenté (Smooth Streaming).

## <a name="overview"></a>Vue d’ensemble
Dans Azure Media Services, un *canal* représente un pipeline de traitement du contenu vidéo en flux continu. Un canal reçoit des flux d’entrée live de l’une des deux manières suivantes :

* Un encodeur live local envoie au canal un paquet RTMP ou Smooth Streaming (MP4 fragmenté) multidébit qui n’est pas activé pour effectuer un encodage live avec Media Services. Les flux reçus transitent par les canaux sans traitement supplémentaire. Cette méthode est appelée *pass-through*. Un encodeur live peut également envoyer un flux à débit binaire unique vers un canal qui n’est pas activé pour l’encodage en temps réel, mais ce n’est pas recommandé. Media Services fournit le flux aux clients qui le demandent.

  > [!NOTE]
  > L’utilisation d’une méthode pass-through est le moyen le plus économique de diffuser une vidéo en flux continu.


* Un encodeur live local envoie un flux à débit unique vers le canal activé pour effectuer un encodage en direct avec Media Services dans l'un des formats suivants : RTMP ou Smooth Streaming (MP4 fragmenté). Le canal procède ensuite à l’encodage en temps réel du flux à débit unique entrant en flux vidéo multidébit (adaptatif). Media Services fournit le flux aux clients qui le demandent.

À partir de la version Media Services 2.10, lorsque vous créez un canal, vous pouvez spécifier de quelle manière votre canal reçoit le flux d’entrée. Vous pouvez également spécifier si le canal doit procéder à l’encodage en temps réel de votre flux. Deux options s'offrent à vous :

* **Transmettre directement** : spécifiez cette valeur si vous envisagez d'utiliser un encodeur live local produisant des flux multidébits (un flux pass-through) en sortie. Le cas échéant, le flux entrant est transmis à la sortie sans encodage. Il s’agit du comportement d’un canal avant la version 2.10. Cet article fournit des détails sur l’utilisation des canaux de ce type.
* **Live Encoding** : choisissez cette valeur si vous envisagez d'utiliser Media Services pour encoder votre flux live à débit unique en flux multidébit. Laisser un canal d’encodage en temps réel dans l’état **En cours d’exécution** occasionne des frais de facturation. Nous vous recommandons d’arrêter immédiatement vos canaux en cours d’exécution une fois votre événement de streaming en direct terminé pour éviter des frais horaires supplémentaires. Media Services fournit le flux aux clients qui le demandent.

> [!NOTE]
> Cet article décrit les attributs des canaux qui ne sont pas activés pour effectuer un encodage live. Pour obtenir des informations sur l’utilisation des canaux qui sont activés pour effectuer l’encodage live, consultez [Streaming en direct avec Azure Media Services pour créer des flux multidébits](media-services-manage-live-encoder-enabled-channels.md).
>
>Pour plus d’informations sur les encodeurs locaux recommandés, consultez [Encodeurs locaux recommandés](media-services-recommended-encoders.md).

Le diagramme suivant décrit un workflow de streaming en direct utilisant un encodeur live local pour produire des flux multidébits au format MP4 fragmenté (Smooth Streaming) ou RMTP en sortie.

![Flux de travail live][live-overview]

## <a name="common-live-streaming-scenario"></a><a id="scenario"></a>Scénario courant de streaming en direct
Les étapes suivantes décrivent les tâches impliquées dans la création d’applications courantes de streaming en direct.

1. Connectez une caméra vidéo à un ordinateur. Démarrez et configurez un encodeur live local qui produit un flux multidébit au format MP4 fragmenté (Smooth Streaming) ou RTMP en sortie. Pour plus d’informations, voir [Prise en charge RTMP et encodeurs dynamiques dans Azure Media Services](https://go.microsoft.com/fwlink/?LinkId=532824).

    Vous pouvez également effectuer cette étape après la création du canal.
2. Créez et démarrez un canal.

3. Récupérez l'URL de réception du canal.

    L’encodeur live utilise l’URL de réception pour envoyer le flux au canal.
4. Récupérez l’URL d’aperçu du canal.

    Utilisez cette URL pour vérifier que votre canal reçoit correctement le flux dynamique.
5. Créez un programme.

    Lorsque vous utilisez le portail Azure, la création d’un programme crée également un élément multimédia.

    Lorsque vous utilisez le Kit SDK .NET ou de REST, vous devez créer une ressource et préciser son utilisation lors de la création d’un programme.
6. Publiez l’élément multimédia associé au programme.   

    >[!NOTE]
    >Une fois votre compte Azure Media Services créé, un point de terminaison de streaming **par défaut** est ajouté à votre compte à l’état **Arrêté**. Le point de terminaison à partir duquel vous souhaitez diffuser du contenu doit se trouver dans l’état **En cours d’exécution**.

7. Démarrez le programme dès que vous êtes prêt à lancer la diffusion en continu et l’archivage.

8. Un signal peut éventuellement être envoyé à l’encodeur dynamique pour qu’il démarre une publicité. La publicité est insérée dans le flux de sortie.

9. Arrêtez le programme chaque fois que vous voulez arrêter la diffusion et archiver l’événement.

10. Supprimez le programme (et éventuellement l’élément multimédia).     

## <a name="description-of-a-channel-and-its-related-components"></a><a id="channel"></a>Description d’un canal et de ses composants associés
### <a name="channel-input-ingest-configurations"></a><a id="channel_input"></a>Configurations de l’entrée (réception) des canaux
#### <a name="ingest-streaming-protocol"></a><a id="ingest_protocols"></a>Protocole de diffusion en continu de réception
Media Services prend en charge la réception des flux live en utilisant des flux au format MP4 fragmenté ou RTMP multidébit comme protocoles de diffusion en continu. Lorsque le protocole de streaming de réception RTMP est sélectionné, deux points de terminaison de réception (entrée) sont créés pour le canal :

* **URL principale** : spécifie l'URL complète du point de terminaison de réception RTMP principal du canal.
* **URL secondaire** (facultatif) : spécifie l'URL complète du point de terminaison de réception RTMP secondaire du canal.

Utilisez l’URL secondaire si vous désirez améliorer la durabilité et la tolérance de panne de votre flux de réception (ainsi que le basculement et la tolérance de panne de l’encodeur), tout spécialement dans les scénarios suivants :

- Double envoi d’encodeur unique vers des URL principales et secondaires :

    L’objectif principal de ce scénario est d’améliorer la résilience des fluctuations et gigues du réseau. Certains codeurs RTMP ne gèrent pas correctement les déconnexions du réseau. En cas de déconnexion du réseau, un encodeur peut arrêter le codage et ne plus envoyer les données en mémoire tampon lorsque la reconnexion se produit. Cela provoque parfois des discontinuités et des pertes de données. La déconnexion du réseau peut être due à un problème de réseau ou une maintenance côté Azure. Les URL principales et secondaires réduisent les problèmes de réseau et fournissent un processus de mise à niveau contrôlé. Chaque fois qu’une déconnexion réseau planifiée se produit, Media Services gère la déconnexion principale et secondaire et décale la déconnexion entre les deux. Les encodeurs ont suffisamment de temps pour envoyer des données, puis se reconnecter. L’ordre de la déconnexion peut être aléatoire, mais il y aura toujours un délai entre les URL principale/secondaire ou secondaire/principale. Dans ce scénario, l’encodeur est toujours le point de défaillance unique.

- Plusieurs encodeurs, chaque encodeur envoyant les données vers un point dédié :

    Ce scénario met en place une redondance d’encodeurs et d’ingestion. Dans ce scénario, l’encodeur 1 envoie les données à l’URL principale et l’encodeur 2 envoie les données à l’URL secondaire. Lorsqu’un encodeur est en échec, l’autre encodeur peut continuer à envoyer des données. La redondance des données peut être maintenue car Media Services ne déconnecte pas les URL principale et secondaire en même temps. Ce scénario part du principe que les encodeurs sont synchronisés et envoient exactement les mêmes données.  

- Double envoi de plusieurs encodeurs vers des URL principales et secondaires :

    Dans ce scénario, les deux encodeurs envoient des données aux URL principales et secondaires. Cela améliore la fiabilité et la tolérance de panne, ainsi que la redondance des données. Ce scénario peut tolérer une panne des deux encodeurs, ainsi que des déconnexions, même si l’un des encodeurs cesse de fonctionner. Il part du principe que les encodeurs sont synchronisés et envoient exactement les mêmes données.  

Pour plus d’informations sur les encodeurs live RTMP, consultez la page [Prise en charge RTMP et encodeurs live dans Azure Media Services](https://go.microsoft.com/fwlink/?LinkId=532824).

#### <a name="ingest-urls-endpoints"></a>URL (points de terminaison) de réception
Un canal fournit un point de terminaison d’entrée (URL de réception) que vous spécifiez dans l’encodeur live pour que ce dernier puisse transmettre les flux à vos canaux.   

Vous pouvez obtenir les URL de réception lors de la création du canal. Pour les obtenir, il n’est pas nécessaire que le canal soit à l’état **En cours d’exécution**. Lorsque vous êtes prêt à commencer l’envoi de données dans le canal, ce dernier doit être à l’état **En cours d’exécution**. Une fois que le canal commence à recevoir les données, vous pouvez prévisualiser votre flux via l’URL d’aperçu.

Vous avez la possibilité de recevoir un stream en direct au format MP4 fragmenté (Smooth Streaming) via une connexion TLS. Pour assurer l’ingestion via TLS, veillez à mettre à jour l’URL d’ingestion pour HTTPS. Il n’est pas possible d’ingérer un flux RTMP via TLS pour le moment.

#### <a name="keyframe-interval"></a><a id="keyframe_interval"></a>Intervalle d’image clé
Lorsque vous utilisez un encodeur live local pour générer un flux multidébit, l’intervalle d’image clé spécifie la durée du groupe d’images (GOP) tel qu’il est utilisé par cet encodeur externe. Une fois ce flux entrant reçu par le canal, vous pouvez distribuer votre flux live aux applications de lecture clientes dans un des formats suivants : Smooth Streaming, Dynamic Adaptive Streaming sur HTTP (DASH) et HTTP Live Streaming (HLS). Lors du streaming en direct, HLS est toujours empaquetée de façon dynamique. Par défaut, Media Services calcule automatiquement le coefficient d’empaquetage de segment HLS (fragments par segment) en fonction de l’intervalle d’image clé reçu de l’encodeur live.

Le tableau suivant montre le mode de calcul de la durée du segment :

| Intervalle d’image clé | Coefficient d’empaquetage de segment HLS (FragmentsPerSegment) | Exemple |
| --- | --- | --- |
| Inférieur ou égal à 3 secondes |3 |Si KeyFrameInterval (ou GOP) est égal à 2 secondes, le coefficient d’empaquetage de segment HLS est de 3 pour 1. Cela crée un segment HLS de 6 secondes. |
| 3 à 5 secondes |2:1 |Si KeyFrameInterval (ou GOP) est égal à 4 secondes, le coefficient d’empaquetage de segment HLS est de 2 pour 1. Cela crée un segment HLS de 8 secondes. |
| Supérieur à 5 secondes |1:1 |Si KeyFrameInterval (ou GOP) est égal à 6 secondes, le coefficient d’empaquetage de segment HLS est de 1 pour 1. Cela crée un segment HLS de 6 secondes. |

Vous pouvez modifier le coefficient de fragments par segment en configurant la sortie du canal et le paramètre FragmentsPerSegment sur ChannelOutputHls.

Vous pouvez également modifier la valeur de l'intervalle d'image clé en définissant la propriété KeyFrameInterval sur ChannelInput. Si vous définissez explicitement la valeur KeyFrameInterval, le coefficient d’empaquetage de segment HLS FragmentsPerSegment est calculé à l’aide des règles décrites précédemment.  

Si vous définissez explicitement les valeurs KeyFrameInterval et FragmentsPerSegment, Media Services utilise ces valeurs.

#### <a name="allowed-ip-addresses"></a>Adresses IP autorisées
Vous pouvez définir les adresses IP autorisées à publier du contenu vidéo sur ce canal. Une adresse IP autorisée peut être spécifiée de l’une des manières suivantes :

* Une adresse IP unique (par exemple, 10.0.0.1)
* une plage d’adresses IP utilisant une adresse IP et un masque de sous-réseau CIDR (par exemple, 10.0.0.1/22)
* une plage d’adresses IP utilisant une adresse IP et un masque de sous-réseau décimal séparé par des points (par exemple, 10.0.0.1(255.255.252.0))

Si aucune adresse IP n’est spécifiée et qu’il n’existe pas de définition de règle, alors aucune adresse IP n’est autorisée. Pour autoriser toutes les adresses IP, créez une règle et définissez la valeur 0.0.0.0/0.

### <a name="channel-preview"></a>Aperçu du canal
#### <a name="preview-urls"></a>URL d’aperçu
Les canaux fournissent un point de terminaison d’aperçu (URL d’aperçu) permettant de prévisualiser et de valider le flux avant de lui appliquer un traitement supplémentaire et de le distribuer.

Vous pouvez obtenir l’URL d’aperçu lors de la création du canal. Pour obtenir l’URL, il n’est pas nécessaire que le canal soit à l’état **En cours d’exécution**. Une fois que le canal commence à recevoir les données, vous pouvez prévisualiser votre flux.

Actuellement, le flux d’aperçu ne peut être distribué qu’au format MP4 fragmenté (Smooth Streaming), quel que soit le type d’entrée spécifié. Vous pouvez utiliser le lecteur [Smooth Streaming Health Monitor](https://playready.directtaps.net/smoothstreaming/) pour tester la diffusion au format Smooth Streaming. Vous pouvez également utiliser un lecteur hébergé dans le portail Azure pour afficher votre flux.

#### <a name="allowed-ip-addresses"></a>Adresses IP autorisées
Vous pouvez définir les adresses IP autorisées à se connecter au point de terminaison d’aperçu. Si aucune adresse IP n’est spécifiée, alors toutes les adresses IP sont autorisées. Une adresse IP autorisée peut être spécifiée de l’une des manières suivantes :

* Une adresse IP unique (par exemple, 10.0.0.1)
* une plage d’adresses IP utilisant une adresse IP et un masque de sous-réseau CIDR (par exemple, 10.0.0.1/22)
* une plage d’adresses IP utilisant une adresse IP et un masque de sous-réseau décimal séparé par des points (par exemple, 10.0.0.1(255.255.252.0))

### <a name="channel-output"></a>Sortie du canal
Pour plus d’informations sur le canal de sortie, consultez la section [Intervalle d’image clé](#keyframe_interval).

### <a name="channel-managed-programs"></a>Programmes gérés par canal
Un canal est associé à des programmes que vous pouvez utiliser pour contrôler la publication et le stockage des segments dans un flux dynamique. Les canaux gèrent les programmes. La relation entre canal et programme est similaire au contenu multimédia traditionnel où un canal a un flux de contenu constant et un programme est limité à un événement minuté sur ce canal.

Vous pouvez spécifier le nombre d’heures pendant lesquelles vous souhaitez conserver le contenu enregistré pour le programme en définissant la durée de la **fenêtre d’archivage** . Cette valeur peut être comprise entre 5 minutes et 25 heures. La durée de la fenêtre d’archivage détermine également la plage maximale de temps dans laquelle les clients peuvent effectuer des recherches en arrière à partir de la position dynamique actuelle. Les programmes peuvent durer davantage que le laps de temps spécifié, mais le contenu qui se situe en dehors de la longueur de fenêtre est ignoré en permanence. La valeur de cette propriété détermine également la longueur maximale que les manifestes de client peuvent atteindre.

Chaque programme est associé à élément multimédia qui stocke le contenu diffusé en continu. Un élément multimédia est mappé à un conteneur d’objets blob de blocs dans le compte de stockage Azure et les fichiers de l’élément multimédia sont stockés sous la forme d’objets blob dans ce conteneur. Pour publier le programme afin que vos clients puissent visionner le flux, vous devez créer un localisateur OnDemand pour la ressource associée. Vous pouvez utiliser ce localisateur pour générer une URL de streaming que vous pourrez fournir à vos clients.

Un canal prend en charge jusqu’à trois programmes exécutés simultanément, ce qui rend possible la création de plusieurs archives du même flux entrant. Vous pouvez publier et archiver différentes parties d’un événement en fonction des besoins. Par exemple, imaginez que vous devez archiver 6 heures d’un programme, mais diffuser uniquement les 10 dernières minutes. Pour ce faire, vous devez créer deux programmes exécutés simultanément. Un programme est configuré pour archiver six heures de l’événement, mais il n’est pas publié. L’autre programme est configuré pour archiver pendant 10 minutes et il est publié.

Vous ne devez pas réutiliser de programmes existants pour de nouveaux événements. Créez plutôt un programme pour chaque événement. Démarrez le programme dès que vous êtes prêt à lancer la diffusion en continu et l’archivage. Arrêtez le programme chaque fois que vous voulez arrêter la diffusion et archiver l’événement.

Pour supprimer du contenu archivé, arrêtez et supprimez le programme, puis supprimez l’élément multimédia associé. Un élément multimédia ne peut pas être supprimé si un programme l’utilise. Le programme doit d’abord être supprimé.

Même après l’arrêt et la suppression du programme, les utilisateurs peuvent lire votre contenu archivé en tant que vidéo à la demande, jusqu’à ce que vous ayez supprimé l’élément multimédia. Si vous souhaitez conserver le contenu archivé sans qu’il soit disponible pour le streaming, supprimez le localisateur de streaming.

## <a name="channel-states-and-billing"></a><a id="states"></a>États du canal et facturation
Les valeurs possibles de l’état actuel d’un canal sont les suivantes :

* **Arrêté** : il s'agit de l'état initial du canal après sa création. Dans cet état, les propriétés du canal peuvent être mises à jour, mais la diffusion en continu n’est pas autorisée.
* **Démarrage en cours** : le canal est en cours de démarrage. Aucune mise à jour ni aucun streaming ne sont autorisés durant cet état. Si une erreur se produit, le canal reprend l’état **Arrêté**.
* **En cours d’exécution** : le canal peut traiter les diffusions en direct.
* **En cours d'arrêt** : le canal est en cours d'arrêt. Aucune mise à jour ni aucun streaming ne sont autorisés durant cet état.
* **Suppression en cours** : le canal est en cours de suppression. Aucune mise à jour ni aucun streaming ne sont autorisés durant cet état.

Le tableau suivant montre comment les états du canal sont mappés au mode de facturation.

| État du canal | Indicateurs de l’interface utilisateur du portail | Facturation ? |
| --- | --- | --- |
| **Démarrage en cours** |**Démarrage en cours** |Aucun (état transitoire) |
| **Exécution** |**Prêt** (aucun programme en cours d’exécution)<p><p>or<p>**Diffusion en continu** (au moins un programme en cours d’exécution) |Oui |
| **En cours d’arrêt** |**En cours d’arrêt** |Aucun (état transitoire) |
| **Arrêté** |**Arrêté** |Non |

## <a name="closed-captioning-and-ad-insertion"></a><a id="cc_and_ads"></a>Sous-titrage codé et insertion de publicités
Le tableau suivant présente les normes de sous-titrage et d’insertion de publicités prises en charge.

| Standard | Notes |
| --- | --- |
| CEA-708 et EIA-608 (708/608) |CEA-708 et EIA-608 sont des normes de sous-titrage codé pour les États-Unis et le Canada.<p><p>Actuellement, le sous-titrage est uniquement pris en charge s’il est inclus dans le flux d’entrée encodé. Vous devez utiliser un encodeur multimédia live capable d’insérer des sous-titres 608 ou 708 dans le flux encodé qui est envoyé à Media Services. Media Services distribue le contenu avec les sous-titres insérés à vos utilisateurs. |
| TTML dans .ismt (pistes textuelles Smooth Streaming) |L'empaquetage dynamique de Media Services permet à vos clients de diffuser du contenu en continu dans l'un des formats suivants : DASH, HLS ou Smooth Streaming. Toutefois, si votre flux est au format MP4 fragmenté (Smooth Streaming) avec des sous-titres dans un fichier .ismt (pistes textuelles Smooth Streaming), vous pouvez distribuer le flux aux clients Smooth Streaming. |
| SCTE-35 |SCTE-35 est un système de signalisation numérique utilisé pour signaler l’insertion de publicités. Les récepteurs en aval utilisent le signal pour ajouter les publicités au flux pendant le temps alloué. La signalisation SCTE-35 doit être envoyée sous forme de piste fragmentée dans le flux d’entrée.<p><p>Actuellement, le seul format de flux d’entrée pris en charge transmettant les signaux publicitaires est le format MP4 fragmenté (Smooth Streaming), qui est aussi le seul format de sortie compatible. |

## <a name="considerations"></a><a id="considerations"></a>Considérations
Lorsque vous utilisez un encodeur live local pour envoyer un flux multidébit dans un canal, les contraintes suivantes s’appliquent :

* Assurez-vous que vous disposez d’une connectivité Internet libre suffisante pour envoyer des données aux points de réception.
* L’utilisation d’une URL de réception secondaire nécessite de la bande passante supplémentaire.
* Le flux multidébit entrant peut présenter un maximum de 10 niveaux de qualité vidéo (couches) et un maximum de 5 pistes audio.
* Le débit binaire moyen le plus élevé pour les niveaux de qualité vidéo doit être inférieur à 10 Mbit/s.
* La somme des débits binaires moyens de tous les flux vidéo et audio doit être inférieure à 25 Mbit/s.
* Vous ne pouvez pas modifier le protocole d’entrée pendant l’exécution du canal ou de ses programmes associés. Si vous avez besoin d’autres protocoles, vous devez créer des canaux distincts pour chaque protocole d’entrée.
* Vous pouvez recevoir un flux à débit binaire unique. Mais comme le flux de données n’est pas traité, les applications clientes recevront également un flux à débit binaire unique. (Nous ne recommandons pas cette option.)

Voici d’autres considérations liées à l’utilisation des canaux et des composants associés :

* Chaque fois que vous reconfigurez l’encodeur dynamique, appelez la méthode de réinitialisation **Reset** sur le canal. Avant de réinitialiser le canal, vous devez arrêter le programme. Une fois le canal réinitialisé, redémarrez le programme.

  > [!NOTE]
  > Lorsque vous redémarrez le programme, vous devez l’associer à un nouvel élément multimédia et créer un localisateur. 
  
* Un canal peut être arrêté uniquement lorsqu’il est dans l’état **En cours d’exécution** et que tous les programmes du canal ont été arrêtés.
* Par défaut, vous pouvez seulement ajouter cinq canaux à votre compte Media Services. Pour plus d’informations, consultez [Quotas et limitations](media-services-quotas-and-limitations.md).
* Vous êtes facturé uniquement lorsque votre canal est à l’état **En cours d’exécution**. Pour plus d’informations, consultez la section [États du canal et facturation](media-services-live-streaming-with-onprem-encoders.md#states).

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="suggestions-and-feedback"></a>Suggestions et commentaires

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-topics"></a>Rubriques connexes
[Encodeurs locaux recommandés](media-services-recommended-encoders.md)

[Spécification d’ingestion en direct au format MP4 fragmenté Azure Media Services](../media-services-fmp4-live-ingest-overview.md)

[Vue d’ensemble d’Azure Media Services et scénarios courants](media-services-overview.md)

[Concepts Media Services](media-services-concepts.md)

[live-overview]: ./media/media-services-manage-channels-overview/media-services-live-streaming-current.png
