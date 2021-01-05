---
title: Spécification d’ingestion en direct au format MP4 fragmenté Azure Media Services | Microsoft Docs
description: Cette spécification décrit le protocole et le format requis pour l’ingestion de streaming en direct basée sur le format MP4 fragmenté pour Microsoft Azure Media Services. Ce document explique également les pratiques recommandées pour créer des mécanismes d’ingestion en direct extrêmement redondants et robustes.
services: media-services
documentationcenter: ''
author: cenkdin
manager: femila
editor: ''
ms.assetid: 43fac263-a5ea-44af-8dd5-cc88e423b4de
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 7323ae611431e1d91fd1a8471914be388fcc4712
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019509"
---
# <a name="azure-media-services-fragmented-mp4-live-ingest-specification"></a>Spécification d’ingestion en direct au format MP4 fragmenté Azure Media Services 

Cette spécification décrit le protocole et le format requis pour l’ingestion de streaming en direct basée sur le format MP4 fragmenté pour Microsoft Azure Media Services. Media Services fournit le service de streaming en direct que les clients peuvent utiliser pour diffuser en continu des événements en direct et diffuser du contenu en temps réel en utilisant Azure en tant que plateforme cloud. Ce document explique également les pratiques recommandées pour créer des mécanismes d’ingestion en direct extrêmement redondants et robustes.

## <a name="1-conformance-notation"></a>1. Notation de conformité
Les mots clés « MUST », « MUST NOT », « REQUIRED », « SHALL », « SHALL NOT », « SHOULD », « SHOULD NOT », « RECOMMENDED », « MAY » et « OPTIONAL » figurant dans ce document doivent être interprétés tels qu’ils sont décrits dans le document RFC 2119.

## <a name="2-service-diagram"></a>2. Diagramme du service
Le diagramme suivant illustre l’architecture générale du service de streaming en direct dans Media Services :

1. Un encodeur en direct transmet les flux en direct à des canaux créés et approvisionnés via le SDK Azure Media Services.
1. Les canaux, les programmes et les points de terminaison de streaming dans Media Services gèrent toutes les fonctionnalités de streaming en direct, notamment l’ingestion, le formatage, le DVR cloud, la sécurité, la scalabilité et la redondance.
1. Les clients peuvent éventuellement choisir de déployer une couche Azure Content Delivery Network entre le point de terminaison de streaming et les points de terminaison client.
1. Les points de terminaison client diffusent à partir du point de terminaison de streaming à l’aide des protocoles de diffusion en continu adaptative HTTP. Parmi les exemples citons Microsoft Smooth Streaming, Dynamic Adaptive Streaming sur HTTP (DASH ou MPEG-DASH) et Apple HTTP Live Streaming (HLS).

![Flux d’ingestion][image1]

## <a name="3-bitstream-format--iso-14496-12-fragmented-mp4"></a>3. Format de flux binaire – MP4 fragmenté ISO 14496-12
Le format câble utilisé pour l’ingestion du streaming en direct décrite dans ce document repose sur la norme [ISO 14496-12]. Pour obtenir une explication détaillée du format MP4 fragmenté et des extensions de fichiers vidéo à la demande et d’ingestion de streaming en direct, consultez [[MS-SSTR]](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251).

### <a name="live-ingest-format-definitions"></a>Définitions de format de réception en temps réel
La liste suivante décrit des définitions de formats spéciaux qui s’appliquent à l’ingestion en direct dans Azure Media Services :

1. Les zones **ftyp**, **Live Server Manifest Box** et **moov** DOIVENT être envoyées avec chaque requête (HTTP POST). Ces zones DOIVENT être envoyées au début du flux et chaque fois que l’encodeur doit se reconnecter pour reprendre l’ingestion du flux. Pour plus d’informations, consultez la section 6 dans [1].
1. La section 3.3.2 dans [1] définit une zone facultative appelée **StreamManifestBox** pour l’ingestion en direct. En raison de la logique de routage de l’équilibreur de charge Azure, l’utilisation de cette zone est dépréciée. La zone NE DOIT PAS être présente au moment de l’ingestion dans Media Services. Si cette zone est présente, Media Services l’ignore en mode silencieux.
1. La zone **TrackFragmentExtendedHeaderBox** définie à la section 3.2.3.2 dans [1] doit être présente pour chaque fragment.
1. La version 2 de la zone **TrackFragmentExtendedHeaderBox** DOIT être utilisée pour générer des segments de médias ayant des URL identiques dans plusieurs centres de données. Le champ d’index de fragment est OBLIGATOIRE pour le basculement entre centres de données des formats de streaming basé sur les index comme Apple HLS et MPEG-DASH basée sur les index. Pour permettre un basculement entre centres de données, l’index de fragment DOIT être synchronisé entre plusieurs encodeurs et être augmenté de 1 pour chaque fragment multimédia successif, même entre les redémarrages ou échecs de l’encodeur.
1. La section 3.3.6 dans [1] définit la zone appelée **MovieFragmentRandomAccessBox** (**mfra**) qui PEUT être envoyée à la fin de la réception en direct pour indiquer la fin du flux (EOS, End-of-Stream) au canal. En raison de la logique d’ingestion d’Azure Media Services, l’utilisation d’EOS est dépréciée et la zone **mfra** pour l’ingestion en direct ne DOIT PAS être envoyée. Si elle l’est, Media Services l’ignore en mode silencieux. Pour réinitialiser l’état du point d’ingestion, nous vous recommandons d’utiliser la [réinitialisation de canal](/rest/api/media/operations/channel#reset_channels). Nous vous recommandons également d’utiliser [l’arrêt de programme](/rest/api/media/operations/program#stop_programs) pour terminer une présentation et un flux.
1. La durée du fragment MP4 DOIT être une constante, pour réduire la taille des manifestes de client. En outre, une durée de fragment MP4 constante améliore la méthode heuristique de téléchargement client par le biais de l’utilisation de balises de répétition. La durée PEUT varier pour compenser les fréquences d’images non entières.
1. La durée du fragment MP4 DOIT être comprise entre environ 2 et 6 secondes.
1. Les horodatages et index du fragment MP4 (**TrackFragmentExtendedHeaderBox** `fragment_ absolute_ time` et `fragment_index`) DOIVENT normalement arriver dans l’ordre croissant. Bien que Media Services ne duplique pas les fragments, il est capable, de façon limitée, de réorganiser les fragments en fonction de la chronologie du média.

## <a name="4-protocol-format--http"></a>4. Format de protocole – HTTP
L’ingestion en direct basée sur le format MP4 fragmenté ISO pour Media Services utilise une longue requête HTTP POST standard pour transmettre au service des données multimédias encodées qui sont empaquetées au format MP4 fragmenté. Chaque requête HTTP POST envoie un flux binaire (« stream ») MP4 fragmenté en commençant par les zones d’en-tête (zones **ftyp**, **Live Server Manifest Box** et **moov**) et en continuant avec une séquence de fragments (zones **moof** et **mdat**). Pour connaître la syntaxe URL de la requête HTTP POST, consultez la section 9.2 dans [1]. Voici un exemple d'URL POST : 

`http://customer.channel.mediaservices.windows.net/ingest.isml/streams(720p)`

### <a name="requirements"></a>Spécifications
Voici les spécifications détaillées :

1. L’encodeur DOIT démarrer la diffusion en envoyant une requête HTTP POST avec un « corps » vide (longueur de contenu nulle) à l’aide de la même URL d’ingestion. Cela peut aider l’encodeur à détecter rapidement si le point de terminaison d’ingestion en direct est valide et s’il existe une authentification ou d’autres conditions requises. Conformément au protocole HTTP, le serveur ne peut pas renvoyer une réponse HTTP tant que la requête entière, y compris le corps POST, n’est pas reçue. Étant donné la longue durée d’un événement en direct, sans cette étape, l’encodeur risque de ne pas pouvoir détecter d’éventuelles erreurs tant qu’il n’a pas terminé d’envoyer toutes les données.
1. L’encodeur DOIT gérer toutes les erreurs ou demandes d’authentification liées à (1). Si (1) réussit avec une réponse 200, continuez.
1. L’encodeur DOIT démarrer une nouvelle requête HTTP POST avec le flux MP4 fragmenté. La charge utile DOIT commencer par les zones d’en-tête, suivies des fragments. Notez que les zones **ftyp**, **Live Server Manifest Box** et **moov** (dans cet ordre) DOIVENT être envoyées avec chaque requête, même si l’encodeur doit se reconnecter, car la requête précédente a été interrompue avant la fin du flux. 
1. L’encodeur DOIT utiliser l’encodage de transfert en bloc pour le chargement, car il est impossible de prédire la longueur totale du contenu de l’événement en direct.
1. Quand l’événement est terminé, après l’envoi du dernier fragment, l’encodeur DOIT gracieusement mettre fin à la séquence de message de l’encodage de transfert en bloc (la plupart des piles de client HTTP gère cela automatiquement). L’encodeur DOIT attendre que le service retourne le code de réponse finale, puis mettre fin à la connexion. 
1. L’encodeur NE DOIT PAS utiliser le nom `Events()` comme décrit à la section 9.2 dans [1] pour l’ingestion en direct dans Media Services.
1. Si la requête HTTP POST s’arrête ou expire avec une erreur TCP avant la fin du flux, l’encodeur DOIT émettre une nouvelle requête POST à l’aide d’une nouvelle connexion et suivre les exigences précédentes. En outre, l’encodeur DOIT renvoyer les deux fragments MP4 précédents pour chaque piste dans le flux et reprendre sans introduire de discontinuité dans la chronologie du média. Le renvoi des deux derniers fragments MP4 pour chaque piste garantit l'absence de perte de données. En d’autres termes, si un flux contient à la fois une piste audio et vidéo, et si la requête POST en cours échoue, l’encodeur doit se reconnecter et renvoyer les deux derniers fragments de la piste audio, lesquels ont déjà été envoyés correctement, ainsi que les deux derniers fragments de la piste vidéo, lesquels ont déjà été envoyés correctement, pour garantir l’absence de perte de données. L’encodeur DOIT conserver un tampon « de transfert » des fragments multimédias, qu’il renvoie quand il se reconnecte.

## <a name="5-timescale"></a>5. Échelle de temps
[MS-SSTR](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251) décrit l’utilisation de l’échelle de temps pour **SmoothStreamingMedia** (section 2.2.2.1), **StreamElement** (section 2.2.2.3), **StreamFragmentElement** (section 2.2.2.6) et **LiveSMIL** (section 2.2.7.3.1). Si la valeur de l’échelle de temps n’est pas présente, la valeur par défaut utilisée est 10 000 000 (10 MHz). Bien que la spécification du format Smooth Streaming ne bloque pas l’utilisation d’autres valeurs d’échelle de temps, la plupart des implémentations de l’encodeur utilisent cette valeur par défaut (10 MHz) pour générer les données d’ingestion Smooth Streaming. En raison de la fonctionnalité [Azure Media Dynamic Packaging](./previous/media-services-dynamic-packaging-overview.md), nous vous recommandons d’utiliser une échelle de temps de 90 kHz pour les flux vidéos et de 44,1 kHz ou de 48,1 kHz pour les flux audio. Si des valeurs d’échelle de temps différentes sont utilisées pour des flux différents, l’échelle de temps au niveau du flux DOIT être envoyée. Pour plus d’informations, consultez [[MS-SSTR]](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251).     

## <a name="6-definition-of-stream"></a>6. Définition de « stream »
Stream est l’unité de base de l’opération dans l’ingestion en direct pour rédiger les présentations en direct, gérer le basculement du streaming et les scénarios de redondance. Stream se définit comme un seul flux binaire MP4 fragmenté pouvant contenir une piste unique ou plusieurs pistes. Une présentation en direct complète peut contenir un ou plusieurs flux, selon la configuration des encodeurs en direct. Les exemples suivants illustrent les différentes options d’utilisation des flux pour rédiger une présentation en direct.

**Exemple :** 

Un client veut créer une présentation de streaming en direct qui inclut les débits binaires audio/vidéo suivants :

Vidéo : 3000 Kbits/s, 1500 Kbits/s, 750 Kbits/s

Audio : 128 Kbits/s

### <a name="option-1-all-tracks-in-one-stream"></a>Option 1 : toutes les pistes dans un seul flux
Dans cette option, un encodeur unique génère toutes les pistes audio/vidéo et les regroupe dans un flux binaire MP4 fragmenté. Le flux binaire MP4 fragmenté est ensuite envoyé via une connexion HTTP POST. Dans cet exemple, il existe un seul flux pour cette présentation en direct.

![Flux : une piste][image2]

### <a name="option-2-each-track-in-a-separate-stream"></a>Option n°2 : chaque piste dans un flux distinct
Dans cette option, l’encodeur met une piste dans chaque flux binaire MP4 fragmenté, puis publie tous les flux sur des connexions HTTP distinctes. Cette option est réalisable avec un seul encodeur ou plusieurs. Du point de vue de l’ingestion en direct, cette présentation en direct se compose de quatre flux.

![Flux : pistes distinctes][image3]

### <a name="option-3-bundle-audio-track-with-the-lowest-bitrate-video-track-into-one-stream"></a>Option 3 : regrouper la piste audio et la piste vidéo au débit binaire le plus bas dans un seul flux
Dans cette option, le client choisit de regrouper la piste audio avec la piste vidéo au débit binaire le plus bas dans un seul flux binaire MP4 fragmenté et de laisser les deux autres pistes vidéo en tant que flux distincts. 

![Flux : pistes audio et vidéo][image4]

### <a name="summary"></a>Résumé
Cela ne constitue pas la liste exhaustive de toutes les options d’ingestion possibles pour cet exemple. En fait, tous les regroupements de pistes dans des flux sont pris en charge par l’ingestion en direct. Les clients et fournisseurs d'encodeurs peuvent choisir leurs propres implémentations selon la complexité d'ingénierie, la capacité de l'encodeur et les questions de redondance et de basculement. Toutefois, dans la plupart des cas, il n’y a qu’une piste audio pour l’ensemble de la présentation en direct. Il est donc important de s’assurer de l’intégrité du flux d’ingestion qui contient la piste audio. Cette remarque entraîne souvent le placement de la piste audio dans son propre flux (comme dans l’option 2) ou son regroupement avec la piste vidéo au débit binaire le plus bas (comme dans l’option 3). De plus, pour une meilleure redondance et une meilleure tolérance de panne, l’envoi de la même piste audio dans deux flux différents (option 2 avec pistes audio redondantes) ou le regroupement de la piste audio avec au moins deux des pistes vidéo au débit binaire le plus faible (option 3 avec audio regroupé avec au moins deux flux vidéo) sont fortement recommandés pour l’ingestion en direct dans Media Services.

## <a name="7-service-failover"></a>7. Basculement du service
Étant donné la nature de la diffusion en continu en direct, une bonne prise en charge du basculement s'avère essentielle pour garantir la disponibilité du service. Media Services est conçu pour gérer divers types de défaillances, notamment les erreurs de réseau, les erreurs de serveur et les problèmes de stockage. Utilisé conjointement avec la logique de basculement appropriée du côté de l’encodeur en direct, les clients peuvent obtenir un service de streaming en direct très fiable à partir du cloud.

Dans cette section, nous abordons les scénarios de basculement du service. Le cas échéant, la défaillance se produit quelque part au sein du service et elle se traduit par une erreur de réseau. Voici quelques recommandations sur l'implémentation de l'encodeur pour gérer le basculement du service :

1. Utilisez un délai d’expiration de 10 secondes pour établir la connexion TCP. Si une tentative d'établissement de la connexion prend plus de 10 secondes, abandonnez l'opération et réessayez. 
1. Utilisez un court délai d'expiration pour l'envoi des fragments de message de la requête HTTP. Si la durée du fragment MP4 cible s’élève à N secondes, utilisez un délai d’expiration d’envoi compris entre N et 2 N secondes. Par exemple, si la durée du fragment MP4 est de 6 secondes, utilisez un délai d’expiration compris entre 6 et 12 secondes. En cas d'expiration, réinitialisez la connexion, ouvrez une nouvelle connexion et reprenez l'ingestion du flux sur la nouvelle connexion. 
1. Conservez un tampon de substitution qui a les deux derniers fragments pour chaque piste qui ont été correctement et complètement envoyés au service.  Si la requête HTTP POST d'un flux est arrêtée ou arrive à expiration avant la fin du flux, ouvrez une nouvelle connexion et commencez une autre requête HTTP POST, renvoyez les en-têtes du flux, renvoyez les deux derniers fragments de chaque piste et reprenez le flux sans introduire de discontinuités dans la chronologie du média. Les risques de perte de données sont ainsi réduits.
1. Nous recommandons que l’encodeur ne limite PAS le nombre de nouvelles tentatives de connexion ou de reprise du streaming suite à une erreur TCP.
1. Après une erreur TCP :
  
    a. La connexion en cours DOIT être fermée et une nouvelle connexion DOIT être créée pour une nouvelle requête HTTP POST.

    b. La nouvelle URL HTTP POST DOIT être identique à l'URL POST initiale.
  
    c. La nouvelle requête HTTP POST DOIT inclure des en-têtes de flux (zone **ftyp**, **Live Server Manifest Box** et **moov**) qui sont identiques aux en-têtes de flux de la requête POST initiale.
  
    d. Les deux derniers fragments envoyés pour chaque piste doivent être renvoyés, et le streaming doit reprendre sans introduire de discontinuités dans la chronologie du média. Les horodatages des fragments MP4 doivent s'incrémenter en permanence, même dans les requêtes HTTP POST.
1. L'encodeur DOIT mettre fin à la requête HTTP POST si les données ne sont pas envoyées à un débit proportionné par rapport à la durée du fragment MP4.  Une requête HTTP POST qui n’envoie pas de données peut empêcher Media Services de se déconnecter rapidement de l’encodeur en cas de mise à jour du service. C’est pourquoi la requête HTTP POST des pistes partiellement allouées (signal d’annonce) DOIT avoir une durée de vie courte et se terminer dès que le fragment partiellement alloué est envoyé.

## <a name="8-encoder-failover"></a>8. Basculement de l’encodeur
Le basculement de l'encodeur est le deuxième type de scénario de basculement qui doit être traité pour la diffusion en continu en direct de bout en bout. Dans ce scénario, la condition d’erreur se produit côté encodeur. 

![Basculement de l’encodeur][image5]

Les attentes suivantes s’appliquent à partir du point de terminaison d’ingestion en direct en cas de basculement de l’encodeur :

1. Une instance de l’encodeur DOIT être créée pour continuer le streaming, comme illustré dans le diagramme (flux pour une vidéo de 3 000 k, avec une ligne en pointillés).
1. Le nouvel encodeur DOIT utiliser la même URL pour les requêtes HTTP POST que l'instance défaillante.
1. La requête POST du nouvel encodeur DOIT inclure les mêmes zones d'en-tête MP4 fragmenté que l'instance défaillante.
1. Le nouvel encodeur DOIT être correctement synchronisé avec tous les autres encodeurs en cours d’exécution pour la même présentation en direct afin de générer des échantillons audio/vidéo synchronisés avec des limites de fragments alignées.
1. Le nouveau flux DOIT être sémantiquement équivalent au flux précédent, et interchangeable aux niveaux de l’en-tête et du fragment.
1. Le nouvel encodeur DOIT tenter de minimiser les pertes de données. Les valeurs `fragment_absolute_time` et `fragment_index` des fragments multimédias DOIVENT augmenter à partir du point où l’encodeur s’est arrêté la dernière fois. Les valeurs `fragment_absolute_time` et `fragment_index` DOIVENT augmenter de manière continue, mais il est possible d’introduire une discontinuité, si besoin. Media Services ignore les fragments déjà reçus et traités. Une erreur du côté du renvoi des fragments est donc préférable à l’introduction de discontinuités dans la chronologie du média. 

## <a name="9-encoder-redundancy"></a>9. Redondance de l’encodeur
Pour certains événements en direct critiques qui exigent une disponibilité et une qualité d’expérience encore plus élevées, nous vous recommandons d’utiliser des encodeurs redondants en mode actif-actif pour obtenir un basculement transparent sans perte de données.

![Redondance de l’encodeur][image6]

Comme illustré dans ce diagramme, deux groupes d’encodeurs transmettent simultanément deux copies de chaque flux au service en direct. Cette configuration est prise en charge, car Media Services peut filtrer les fragments en double en fonction de l’ID du flux et de l’horodatage des fragments. Le flux en direct et l’archive obtenus consistent en une seule copie de tous les flux constituant l’agrégation la meilleure possible des deux sources. Par exemple, dans un cas extrême hypothétique, tant qu’il existe un encodeur (éventuellement différent) en cours d’exécution à un moment donné pour chaque flux, le flux en direct obtenu à partir du service est continu sans perte de données. 

La configuration requise pour ce scénario est presque identique à celle requise pour le cas « Basculement de l’encodeur », à l’exception près que le second ensemble d’encodeurs s’exécute en même temps que les encodeurs principaux.

## <a name="10-service-redundancy"></a>10. Redondance du service
Pour une distribution globale hautement redondante, vous devez parfois disposer d’une sauvegarde inter-régions pour traiter les urgences régionales. En développant la topologie « Redondance de l’encodeur », les clients peuvent choisir d’avoir un déploiement de services redondant dans une région différente qui est connectée au second ensemble d’encodeurs. Les clients peuvent également avoir recours à un fournisseur CDN pour déployer un gestionnaire de trafic global (GTM, Global Traffic Manager) devant les deux déploiements de services afin d’acheminer le trafic du client en toute transparence. La configuration requise pour les encodeurs est la même que pour le cas « Redondance de l’encodeur ». La seule exception est que le second ensemble d’encodeurs doit pointer vers un autre point de terminaison d’ingestion en direct. Le diagramme qui suit présente cette configuration :

![Redondance du service][image7]

## <a name="11-special-types-of-ingestion-formats"></a>11. Types spéciaux de formats d’ingestion
Cette section présente des types spéciaux de formats d’ingestion en direct, conçus pour gérer des scénarios spécifiques.

### <a name="sparse-track"></a>Piste partiellement allouée
Pendant une présentation de streaming en direct avec une expérience client riche, il est souvent nécessaire de transmettre des événements synchronisés ou des signaux intrabande avec les principales données multimédias. L’insertion de publicités dynamiques en direct en est un exemple. Ce type de signalisation d'événement diffère de la diffusion audio/vidéo en continu standard en raison de sa nature partiellement allouée. Autrement dit, les données de signalisation ne se produisent généralement pas de manière continue, et l’intervalle peut être difficile à prédire. Le concept de piste partiellement allouée a été conçu pour ingérer et diffuser des données de signalisation intrabande.

Les étapes suivantes constituent une implémentation recommandée pour l’ingestion d’une piste partiellement allouée :

1. Créez un flux binaire MP4 fragmenté distinct qui contient uniquement des pistes partiellement allouées, sans pistes audio/vidéo.
1. Dans la zone **Live Server Manifest Box** définie à la section 6 dans [1], utilisez le paramètre *parentTrackName* pour spécifier le nom de la piste parent. Pour plus d’informations, consultez la section 4.2.1.2.1.2 dans [1].
1. Dans la zone **Live Server Manifest Box**, **manifestOutput** DOIT avoir la valeur **true**.
1. Étant donné la nature partiellement allouée de l’événement de signalisation, voici ce que nous recommandons :
   
    a. Au début de l’événement en direct, l’encodeur envoie les zones d’en-tête initiales au service, ce qui permet au service d’enregistrer la piste partiellement allouée dans le manifeste du client.
   
    b. L'encodeur DOIT mettre fin à la requête HTTP POST quand les données ne sont pas envoyées. Une longue requête HTTP POST qui n’envoie pas de données peut empêcher Media Services de se déconnecter rapidement de l’encodeur en cas de mise à jour du service ou de redémarrage du serveur. Dans ces cas, le serveur multimédia est temporairement bloqué dans une opération de réception sur le socket.
   
    c. Pendant que les données de signalisation ne sont pas disponibles, l'encodeur DOIT fermer la requête HTTP POST. Pendant que la requête POST est active, l’encodeur DOIT envoyer des données.

    d. Pendant l’envoi de fragments partiellement alloués, l’encodeur peut définir un en-tête content-length explicite, s’il est disponible.

    e. Pendant l’envoi d’un fragment partiellement alloué avec une nouvelle connexion, l’encodeur DOIT commencer par envoyer les zones d’en-tête, puis les nouveaux fragments. Cette recommandation concerne les cas dans lesquels le basculement a eu lieu entre temps et où la nouvelle connexion est établie avec un nouveau serveur qui n’a pas vu la piste partiellement allouée auparavant.

    f. Le fragment de piste partiellement allouée devient disponible pour le client quand le fragment de piste parent correspondant dont la valeur d’horodatage est égale ou supérieure est mis à la disposition du client. Par exemple, si le fragment partiellement alloué a un horodatage de t=1000, après avoir visualisé l’horodatage de fragment " vidéo " 1000 ou au-delà (en supposant que le nom de la piste parent est " vidéo "), le client peut normalement télécharger le fragment partiellement alloué t=1000. Notez que le signal réel peut servir à un autre emplacement dans la chronologie de la présentation pour sa fonction désignée. Dans cet exemple, le fragment partiellement alloué de t=1000 peut avoir une charge utile XML, pour insérer une publicité à un emplacement situé quelques secondes plus tard.

    g. La charge utile des fragments de piste partiellement allouée peut se présenter sous différents formats (par exemple, XML, texte ou binaire), selon le scénario.

### <a name="redundant-audio-track"></a>Piste audio redondante
Dans un scénario de diffusion en continu adaptative HTTP classique (par exemple, Smooth Streaming ou DASH), la présentation dans son ensemble ne contient souvent qu’une seule piste audio. Contrairement aux pistes vidéo, qui ont plusieurs niveaux de qualité parmi lesquels le client peut choisir dans des conditions d’erreur, la piste audio peut correspondre à un seul point de défaillance si l’ingestion du flux qui contient la piste audio est rompue. 

Pour résoudre ce problème, Media Services prend en charge l’ingestion en direct des pistes audio redondantes. L'idée est que la même piste audio peut être envoyée plusieurs fois dans différents flux. Bien que le service enregistre une seule fois la piste audio dans le manifeste du client, il peut utiliser des pistes audio redondantes en tant que sauvegardes pour la récupération de fragments audio si la piste audio principale a des problèmes. Pour ingérer des pistes audio redondantes, l’encodeur doit :

1. Créer la même piste audio dans plusieurs flux binaires MP4 fragmentés. Les pistes audio redondantes DOIVENT être sémantiquement équivalentes, avec les mêmes horodatages de fragments, et être interchangeables aux niveaux de l’en-tête et du fragment.
1. Vérifier que l’entrée « audio » dans la zone Live Server Manifest (section 6 dans [1]) est la même pour toutes les pistes audio redondantes.

L’implémentation suivante est recommandée pour les pistes audio redondantes :

1. Envoyez chaque piste audio unique dans un flux toute seule. Envoyez également un flux redondant pour chacun de ces flux de piste audio, où le second flux diffère du premier uniquement par l’identificateur inclus dans l’URL HTTP POST : {protocole}://{adresse du serveur}/{chemin du point de publication}/Streams({identificateur}).
1. Utilisez des flux distincts pour envoyer les deux débits binaires vidéo les plus bas. Chacun de ces flux DOIT également contenir une copie de chaque piste audio unique. Par exemple, quand plusieurs langues sont prises en charge, ces flux DOIVENT contenir des pistes audio pour chaque langue.
1. Utilisez des instances de serveur (encodeur) distinctes pour encoder et envoyer les flux redondants mentionnés dans (1) et (2). 

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

[image1]: ./media/media-services-fmp4-live-ingest-overview/media-services-image1.png
[image2]: ./media/media-services-fmp4-live-ingest-overview/media-services-image2.png
[image3]: ./media/media-services-fmp4-live-ingest-overview/media-services-image3.png
[image4]: ./media/media-services-fmp4-live-ingest-overview/media-services-image4.png
[image5]: ./media/media-services-fmp4-live-ingest-overview/media-services-image5.png
[image6]: ./media/media-services-fmp4-live-ingest-overview/media-services-image6.png
[image7]: ./media/media-services-fmp4-live-ingest-overview/media-services-image7.png