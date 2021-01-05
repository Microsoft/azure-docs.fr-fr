---
title: Azure Media Services - Signalisation de métadonnées chronométrées dans le streaming en direct
description: Cette spécification décrit les méthodes de signalisation des métadonnées chronométrées lors de la réception et de la diffusion en continu vers Azure Media Services. Cela inclut la prise en charge de signaux génériques de métadonnées chronométrées (ID3), ainsi que la signalisation SCTE-35 pour l'insertion de jointures publicitaires et la signalisation des conditions de la jointure.
services: media-services
documentationcenter: ''
author: johndeu
manager: femila
editor: johndeu
ms.assetid: 265b94b1-0fb8-493a-90ec-a4244f51ce85
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2019
ms.author: johndeu
ms.openlocfilehash: f826ee9ef3c9fff0b721a9c79d3c12e0adbd5f7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91336392"
---
# <a name="signaling-timed-metadata-in-live-streaming"></a>Signalisation de métadonnées chronométrées dans une vidéo en flux continu 

Dernière mise à jour : 2019-08-22

### <a name="conformance-notation"></a>Notation de conformité

« DOIT », « NE DOIT PAS », « OBLIGATOIRE », « DEVRA », « NE DEVRA PAS », « DEVRAIT », « NE DEVRAIT PAS », « RECOMMANDÉ », « NON RECOMMANDÉE », « PEUT » et « OPTIONNELLE » figurant dans ce document doivent être interprétés de la manière décrite dans le document RFC 2119.

## <a name="1-introduction"></a>1. Introduction 

Afin de signaler l'insertion d’annonces ou d'événements de métadonnées personnalisés sur un lecteur de client, les diffuseurs utilisent souvent des métadonnées chronométrées incorporées dans la vidéo. Pour permettre ces scénarios, Media Services prend en charge le transport de métadonnées chronométrées, du point de réception du canal de diffusion en continu en direct à l'application cliente.
Cette spécification présente plusieurs modes pris en charge par Media Services pour les métadonnées chronométrées à l’intérieur des signaux de diffusion en continu en direct.

1. Signalement [SCTE-35] conforme aux normes indiquées par [SCTE-35], [SCTE-214-1], [SCTE-214-3] et [RFC8216]

2. Signalement [SCTE-35] conforme à la spécification héritée [Adobe-Primetime] pour la signalisation d’annuaire RTMP.
   
3. Un mode de signalisation de métadonnées générique chronométré, pour les messages qui ne sont **PAS** [SCTE-35] et qui peuvent comporter [ID3v2] ou d’autres schémas personnalisés définis par le développeur d’applications.

## <a name="11-terms-used"></a>1.1 Termes utilisés

| Terme                | Définition                                                                                                                                                                                                                                    |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Interruption publicitaire            | Emplacement ou moment où une ou plusieurs annonces peuvent être planifiées pour la remise; identique à l’opportunité de disponibilité et de placement.                                                                                                                     |
| Service de décision publicitaire | Service externe qui détermine la ou les annonces et les durées à afficher à l’utilisateur. Les services sont généralement fournis par un partenaire et ne relèvent pas de ce document.                                                                    |
| Signal                 | Indication du temps et des paramètres de la prochaine pause publicitaire. Notez que les signaux peuvent indiquer un basculement en attente vers une pause publicitaire, un basculement en attente vers la publicité suivante au sein d’une pause publicitaire et un basculement en attente d’une pause publicitaire vers le contenu principal.           |
| Packager            | Le « point de terminaison de streaming » Azure Media Services fournit des fonctionnalités d’empaquetage dynamique pour DASH et TLS et est appelé « Packager » dans le secteur des médias.                                                                              |
| Heure de présentation   | Heure à laquelle un événement est présenté à un utilisateur. Il s’agit de l’heure dans la chronologie du média à laquelle un utilisateur voit l’événement. Par exemple, l’heure de présentation d’un message de commande splice_info() SCTE-35 est splice_time(). |
| Heure d’arrivée        | Heure à laquelle un message d’événement arrive. L’heure d’arrivée est généralement distincte de l’heure de présentation de l’événement, dans la mesure où les événements sont envoyés avant l’heure de leur présentation.                                                    |
| Piste partiellement allouée        | Piste de média qui n’est pas continue et qui est synchronisée avec une piste parente ou de contrôle.                                                                                                                                                  |
| Origine              | Service de diffusion en continu de média Azure.                                                                                                                                                                                                             |
| Récepteur de canal        | Service de diffusion en continu en direct Azure.                                                                                                                                                                                                        |
| HLS                 | Protocole de diffusion en continu HTTP Apple                                                                                                                                                                                                            |
| DASH                | Diffusion en continu adaptative dynamique sur HTTP.                                                                                                                                                                                                          |
| Lisse              | Protocole de diffusion en continu lisse.                                                                                                                                                                                                                     |
| MPEG2-TS            | Flux de transport MPEG-2.                                                                                                                                                                                                                      |
| RTMP                | Protocole multimédia en temps réel.                                                                                                                                                                                                                 |
| uimsbf              | Entier non signé, avec le bit le plus significatif en tête.                                                                                                                                                                                                 |

---

## <a name="12-normative-references"></a>1.2 Références normatives

Les documents suivants contiennent des dispositions qui, par référence à ce texte, constituent des dispositions du présent document. Tous les documents sont soumis à des révisions par les organismes de normalisation et les lecteurs sont encouragés à examiner la possibilité d’appliquer les éditions les plus récentes des documents répertoriés ci-dessous. Il est également rappelé aux lecteurs que les éditions plus récentes des documents référencés peuvent ne pas être compatibles avec cette version des spécifications de métadonnées chronométrées pour Azure Media Services.


| Standard          | Définition                                                                                                                                                                                                     |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Adobe-Primetime] | [Spécification de signalement d’insertion de programme numérique Primetime 1.2](https://www.adobe.com/content/dam/acom/en/devnet/primetime/PrimetimeDigitalProgramInsertionSignalingSpecification.pdf)                       |
| [Adobe-Flash-AS]  | [Informations de référence sur le langage Flash ActionScript](https://help.adobe.com/archive/en_US/as2/flashlite_2.x_3.x_aslr.pdf)                                                                                                   |
| [AMF0]            | [« Action Message Format AMF0 »](https://download.macromedia.com/pub/labs/amf/amf0_spec_121207.pdf)                                                                                                              |
| [DASH-IF-IOP]     | Guide d’interopérabilité du Forum du secteur DASH v4.2 [https://dashif-documents.azurewebsites.net/DASH-IF-IOP/master/DASH-IF-IOP.html](https://dashif-documents.azurewebsites.net/DASH-IF-IOP/master/DASH-IF-IOP.html)    |
| [HLS-TMD]         | Métadonnées chronométrées pour HTTP Live Streaming - [https://developer.apple.com/streaming](https://developer.apple.com/streaming)                                                                                        |
| [CMAF-ID3]        | [Métadonnées chronométrées au format CMAF (Common Media Application Format)](https://github.com/AOMediaCodec/id3-emsg)                                                                                                        |
| [ID3v2]           | Version de balise ID3 2.4.0  [http://id3.org/id3v2.4.0-structure](http://id3.org/id3v2.4.0-structure)                                                                                                                |
| [ISO-14496-12]    | ISO/IEC 14496-12 : Partie 12: Format ISO de base pour les fichiers médias, quatrième édition 15/07/2012                                                                                                                                 |
| [MPEGDASH]        | Information technology – Dynamic adaptive streaming over HTTP (DASH) – Part 1: Media presentation description and segment formats. Mai 2014. Publié. URL : https://www.iso.org/standard/65274.html         |
| [MPEGCMAF]        | Information technology -- Multimedia application format (MPEG-A) -- Part 19 : Common media application format (CMAF) for segmented media. Janvier 2018. Publié. URL : https://www.iso.org/standard/71975.html |
| [MPEGCENC]        | [ISO/IEC 7] Technologies de l'information -- Technologies des systèmes MPEG -- Partie 7 : Cryptage commun des fichiers au format de fichier de médias de la base ISO. Février 2016. Publié. URL : https://www.iso.org/standard/68042.html                   |
| [MS-SSTR]         | [« Microsoft Smooth Streaming Protocol », 15 mai 2014](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251)                                                     |
| [MS-SSTR-Ingest]  | [Spécification d’ingestion en direct au format MP4 fragmenté Azure Media Services](./media-services-fmp4-live-ingest-overview.md)                                                      |
| [RFC8216]         | R. Pantos, Ed.; W. May. HTTP Live Streaming. Août 2017. Informatif. [https://tools.ietf.org/html/rfc8216](https://tools.ietf.org/html/rfc8216)                                                            |
| [RFC4648]         | The Base16, Base32, and Base64 Data Encodings - [https://tools.ietf.org/html/rfc4648](https://tools.ietf.org/html/rfc4648)                                                                                     |
| [RTMP]            | [« Adobe’s Real-Time Messaging Protocol », 21 décembre 2012](https://www.adobe.com/devnet/rtmp.html)                                                                                                            |
| [SCTE-35-2019]    | SCTE 35 : 2019 - Digital Program Insertion Cueing Message for Cable - https://www.scte.org/SCTEDocs/Standards/ANSI_SCTE%2035%202019r1.pdf                                                                       |
| [SCTE-214-1]      | SCTE 214-1 2016 – MPEG DASH for IP-Based Cable Services Part 1: MPD Constraints and Extensions                                                                                                                 |
| [SCTE-214-3]      | SCTE 214-3 2015 MPEG DASH for IP-Based Cable Services Part 3: DASH/FF Profile                                                                                                                                  |
| [SCTE-224]        | SCTE 224 2018r1 – Event Scheduling and Notification Interface                                                                                                                                                  |
| [SCTE-250]        | Event and Signaling Management API (ESAM)                                                                                                                                                                      |

---


## <a name="2-timed-metadata-ingest"></a>2. Réception de métadonnées chronométrées

Azure Media Services prend en charge les métadonnées intrabande en temps réel pour les protocoles [RTMP] et Smooth Streaming [MS-SSTR-Ingest]. Les métadonnées en temps réel peuvent être utilisées pour définir des événements personnalisés, avec vos propres schémas personnalisés (JSON, binaires, XML), ainsi que des formats définis par le secteur comme ID3 ou SCTE-35 pour la signalisation des annonces dans un flux de diffusion. 

Cet article fournit des détails sur la façon d’envoyer des signaux de métadonnées chronométrées personnalisés à l’aide des protocoles de réception pris en charge d'Azure Media Services. Cet article explique également comment les manifestes pour TLS, DASH et Smooth Streaming sont agrémentés de signaux de métadonnées chronométrées et comment ils sont transportés en intrabande lorsque le contenu est remis à l’aide de CMAF (fragments MP4) ou de segments Transport Stream (TS) pour HLS. 

Les scénarios de cas d’usage courants pour les métadonnées chronométrées sont les suivants :

 - Signaux publicitaires SCTE-35 pour déclencher des coupures publicitaires dans un événement en direct ou une diffusion linéaire
 - Métadonnées ID3 personnalisées pouvant déclencher des événements dans une application cliente (navigateur, iOS ou Android)
 - Métadonnées JSON, binaires ou XML personnalisées définies pour déclencher des événements dans une application cliente
 - Télémétrie à partir d’un encodeur live, d’une caméra IP ou d’un drone
 - Événements à partir d’une caméra IP comme la détection des mouvements, des visages, etc.
 - Informations de position géographique à partir d’une caméra d’action, d’un drone ou d’un appareil mobile
 - Paroles de chanson
 - Limites du programme sur un flux en temps réel linéaire
 - Images ou métadonnées augmentées à afficher sur un flux en temps réel
 - Scores sportifs ou informations sur les jeux chronométrés
 - Packages de publicité interactive à afficher avec la vidéo dans le navigateur
 - Questionnaires ou sondages
  
Les événements en direct et le packager Azure Media Services sont capables de recevoir ces signaux de métadonnées chronométrées et de les convertir en flux de métadonnées pouvant atteindre des applications clientes à l’aide de protocoles basés sur des normes, tels que HLS et DASH.


## <a name="21-rtmp-timed-metadata"></a>2.1 Métadonnées chronométrées RTMP

Le protocole [RTMP] permet d’envoyer des signaux de métadonnées chronométrées pour différents scénarios, y compris des métadonnées personnalisées et des signaux publicitaires SCTE-35. 

Les signaux publicitaires (messages de signal) sont envoyés sous forme de messages de signal [AMF0] incorporés dans le flux [RTMP]. Les messages de signal peuvent être envoyés avant que l’événement réel ou d’un signal de jointure publicitaire [SCTE35] doivent se produire. Pour prendre en charge ce scénario, le timestamp réel de présentation de l’événement est envoyé dans le message de signal. Pour plus d'informations, voir [AMF0].

Les commandes [AMF0] suivantes sont prises en charge par Azure Media Services à des fins de réception RTMP :

- **onUserDataEvent** : utilisée pour les métadonnées personnalisées ou les métadonnées chronométrées [ID3v2]
- **onAdCue** : utilisée principalement pour signaler une opportunité de placement publicitaire dans le stream en direct. Deux formes de signal sont prises en charge, un mode simple et un mode « SCTE-35 ». 
- **onCuePoint** - pris en charge par certains encodeurs matériels locaux, tels que l’encodeur Elemental Live, pour signaler des messages [SCTE35]. 
  

Le tableau suivant décrit le format de la charge utile du message AMF que Media Services ingère pour les modes de message [SCTE35] et « simple ».

Le nom du message [AMF0] peut être utilisé pour différencier plusieurs flux d’événements du même type.  Pour les messages [SCTE-35] et le mode « simple », le nom du message AMF DOIT être « onAdCue », comme requis dans la spécification [Adobe-Primetime].  Tous les champs non listés ci-dessous sont ignorés par Azure Media Services à la réception.

## <a name="211-rtmp-with-custom-metadata-using-onuserdataevent"></a>2.1.1 RTMP avec métadonnées personnalisées utilisant « onUserDataEvent »

Si vous souhaitez fournir des flux de métadonnées personnalisés à partir de votre encodeur en amont, caméra IP, drone ou appareil à l’aide du protocole RTMP, utilisez le type de commande de message de données « onUserDataEvent » [AMF0].

La commande de message de données **« onUserDataEvent »** DOIT contenir une charge utile de message avec la définition suivante pour être capturée par Media Services et empaquetée dans le format de fichier intrabande, ainsi que les manifestes pour HLS, Dash et Smooth Streaming.
Il est recommandé d’envoyer des messages de métadonnées chronométrées au plus une fois toutes les 0,5 seconde (500 ms) pour éviter que des problèmes de stabilité liés au stream en direct ne surviennent. Chaque message peut agréger des métadonnées à partir de plusieurs images si vous devez fournir des métadonnées au niveau de l'image. Si vous envoyez des flux à débit binaire multiple, il est recommandé de fournir également les métadonnées sur un débit binaire unique pour réduire la bande passante et éviter les interférences avec le traitement vidéo/audio. 

La charge utile pour **« onUserDataEvent »** doit correspondre à un message au format XML EventStream [MPEGDASH]. Cela facilite la transmission des schémas définis personnalisés qui peuvent être acheminés dans des charges utiles « emsg » pour le contenu CMAF [MPEGCMAF] fourni via les protocoles HLS ou DASH. Chaque message d'événement en direct DASH contient un élément schemeIdUri qui fonctionne comme un identificateur de schéma de message URN et définit la charge utile du message. Certains schémas tels que « https://aomedia.org/emsg/ID3  » pour [ID3v2] ou **urn: SCTE: scte35:2013: bin** pour [SCTE-35] sont normalisés par les consortiums du secteur à des fins d'interopérabilité. Tout fournisseur d’applications peut définir son propre schéma personnalisé à l’aide d’une URL qu’il contrôle (domaine détenu) et fournir une spécification à cette URL, le cas échéant. Si un lecteur possède un descripteur pour le schéma défini, il s’agit du seul composant qui doit comprendre la charge utile et le protocole.

Le schéma correspondant à la charge utile XML EventStream [MPEG-DASH] est défini comme (extrait d'ISO-IEC-23009-1 DASH 3e édition). Notez qu’un seul « EventType » par « EventStream » est pris en charge pour l’instant. Seul le premier élément **Événement** est traité si plusieurs événements sont fournis dans **EventStream**.

```xml
  <!-- Event Stream -->
  <xs:complexType name="EventStreamType">
    <xs:sequence>
      <xs:element name="Event" type="EventType" minOccurs="0" maxOccurs="unbounded"/>
      <xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute ref="xlink:href"/>
    <xs:attribute ref="xlink:actuate" default="onRequest"/>
    <xs:attribute name="schemeIdUri" type="xs:anyURI" use="required"/>
    <xs:attribute name="value" type="xs:string"/>
    <xs:attribute name="timescale" type="xs:unsignedInt"/>
  </xs:complexType>
  <!-- Event  -->
  <xs:complexType name="EventType">
    <xs:sequence>
      <xs:any namespace="##other" processContents="lax" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="presentationTime" type="xs:unsignedLong" default="0"/>
    <xs:attribute name="duration" type="xs:unsignedLong"/>
    <xs:attribute name="id" type="xs:unsignedInt"/>
    <xs:attribute name="contentEncoding" type="ContentEncodingType"/>
    <xs:attribute name="messageData" type="xs:string"/>
    <xs:anyAttribute namespace="##other" processContents="lax"/>
  </xs:complexType>
```


### <a name="example-xml-event-stream-with-id3-schema-id-and-base64-encoded-data-payload"></a>Exemple de flux d’événements XML avec ID de schéma ID3 et charge utile de données encodées en base64  
```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <EventStream schemeIdUri="https://aomedia.org/emsg/ID3">
         <Event contentEncoding="Base64">
          -- base64 encoded ID3v2 full payload here per [CMAF-TMD] --
         </Event>
   <EventStream>
```

### <a name="example-event-stream-with-custom-schema-id-and-base64-encoded-binary-data"></a>Exemple de flux d’événements avec ID de schéma personnalisé et données binaires encodées en base64  
```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <EventStream schemeIdUri="urn:example.org:custom:binary">
         <Event contentEncoding="Base64">
          -- base64 encoded custom binary data message --
         </Event>
   <EventStream>
```

### <a name="example-event-stream-with-custom-schema-id-and-custom-json"></a>Exemple de flux d’événements avec ID de schéma personnalisé et JSON personnalisé  
```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <EventStream schemeIdUri="urn:example.org:custom:JSON">
         <Event>
          [
            {"key1" : "value1"},
            {"key2" : "value2"}
          ]
         </Event>
   <EventStream>
```

### <a name="built-in-supported-scheme-id-uris"></a>URI d’ID de schéma pris en charge intégrés
| URI d’ID de schéma                 | Description                                                                                                                                                                                                                                          |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| https:\//aomedia.org/emsg/ID3 | Décrit la façon dont les métadonnées [ID3v2] peuvent être transmises en tant que métadonnées chronométrées dans un MP4 fragmenté [MPEGCMAF] compatible CMAF. Pour plus d'informations, consultez [Métadonnées chronométrées au format CMAF (Common Media Application Format)](https://github.com/AOMediaCodec/id3-emsg) |

### <a name="event-processing-and-manifest-signaling"></a>Traitement des événements et signalisation des manifestes

À réception d’un événement **« onUserDataEvent »** valide, Azure Media Services recherche une charge utile XML valide qui correspond à EventStreamType (défini dans [MPEGDASH]), analyse la charge utile XML et la convertit en zone « emsg » version 1 de fragment MP4 [MPEGCMAF] à des fins de stockage dans l'archive en temps réel et de transmission au packager Media Services.   Le packager détecte la zone « emsg » dans le stream en direct et :

- (a) l'empaquète dynamiquement dans des segments TS à des fins de remise aux clients HLS en conformité avec la spécification de métadonnées chronométrées HLS [HLS-TMD], ou
- (b) la transmet pour une remise en fragments CMAF via HLS ou DASH, ou 
- (c) la convertit en signal de piste partiellement allouée pour une remise via Smooth Streaming [MS-SSTR].

En plus des paquets CMAF ou TS PES au format « emsg » pour HLS, les manifestes pour DASH (MPD) et Smooth Streaming contiennent une référence aux flux d’événements intrabande (également appelée piste partiellement allouée dans Smooth Streaming).

Les événements individuels ou leurs charges utiles de données ne sont PAS directement générés dans les manifestes HLS, DASH ou Smooth. 

### <a name="additional-informational-constraints-and-defaults-for-onuserdataevent-events"></a>Contraintes et valeurs par défaut d’informations supplémentaires pour les événements onUserDataEvent

- Si l’échelle de temps n’est pas définie dans l’élément EventStream, l’échelle de temps RTMP 1 kHz est utilisée par défaut
- La remise d’un message onUserDataEvent est limitée à une fois toutes les 500 ms max. Si vous envoyez des événements de manière plus fréquente, cela peut avoir un impact sur la bande passante et la stabilité du flux en temps réel

## <a name="212-rtmp-ad-cue-signaling-with-onadcue"></a>2.1.2 Signalisation de signal publicitaire RTMP avec « onAdCue »

Azure Media Services peut écouter et répondre à plusieurs types de messages [AMF0] utilisables pour signaler différentes métadonnées synchronisées en temps réel dans le flux en direct.  La spécification [Adobe-Primetime] définit deux types de signal appelés modes « simple » et « SCTE-35 ». Pour le mode « simple », Media Services prend en charge un seul message de signal AMF appelé « onAdCue » à l’aide d’une charge utile qui correspond à la table ci-dessous définie pour le signal « mode simple ».  

La section suivante présente la charge utile du « mode simple » RTMP, utilisable pour signaler un signal publicitaire « spliceOut » de base qui sera transmis au manifeste du client pour TLS, DASH et Microsoft Smooth Streaming. Cela s’avère très utile dans les scénarios où le client ne dispose pas d’un système complexe d’insertion ou de déploiement de signalement d’annonces basées sur SCTE-35, et où il utilise un encodeur local de base pour envoyer le message de signal via une API. En règle générale, l’encodeur local prend en charge une API REST pour déclencher ce signal, ce qui permet également « d’épisser » le flux vidéo en insérant une image IDR dans la vidéo et en démarrant un nouveau groupe d’images.

## <a name="213--rtmp-ad-cue-signaling-with-onadcue---simple-mode"></a>2.1.3 Signalisation de signal publicitaire RTMP avec « onAdCue » - Mode simple

| Nom du champ | Type de champ | Requis ? | Descriptions                                                                                                                                                                                                                                                                        |
| ---------- | ---------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type       | String     | Obligatoire  | Message d'événement.  Doit être « SpliceOut » pour désigner une jointure en mode simple.                                                                                                                                                                                                         |
| id         | String     | Obligatoire  | Identificateur unique décrivant la jointure ou le segment. Identifie cette instance du message.                                                                                                                                                                                       |
| duration   | Number     | Obligatoire  | Durée de la jointure. Les unités sont des fractions de seconde.                                                                                                                                                                                                                           |
| elapsed    | Number     | Facultatif  | Lorsque le signal est répété pour prendre en charge le réglage, ce champ est la quantité de temps de présentation qui s’est écoulée depuis le début de la jointure. Les unités sont des fractions de seconde. Lorsque vous utilisez le mode simple, cette valeur ne doit pas dépasser la durée originale de la jointure. |
| time       | Number     | Obligatoire  | Doit être l’heure de la jointure, dans le temps de présentation. Les unités sont des fractions de seconde.                                                                                                                                                                                                |

---
 
#### <a name="example-mpeg-dash-manifest-output-when-using-adobe-rtmp-simple-mode"></a>Exemple de sortie de manifeste MPEG DASH lors de l’utilisation du mode simple Adobe RTMP

Consultez l’exemple [3.3.2.1 MPEG DASH .mpd EventStream utilisant le mode simple Adobe](#3321-example-mpeg-dash-mpd-manifest-signaling-of-rtmp-streaming-using-adobe-simple-mode)

Consultez l’exemple [3.3.3.1 manifeste DASH avec une période unique et le mode simple Adobe ](#3331-example-mpeg-dash-manifest-mpd-with-single-period-eventstream-using-adobe-simple-mode-signals)

#### <a name="example-hls-manifest-output-when-using-adobe-rtmp-simple-mode"></a>Exemple de sortie de manifeste HLS lors de l’utilisation du mode simple Adobe RTMP

Consultez l’exemple [3.2.2 manifeste HLS utilisant le mode simple Adobe et la balise EXT-X-CUE](#322-apple-hls-with-adobe-primetime-ext-x-cue)

## <a name="214-rtmp-ad-cue-signaling-with-onadcue---scte-35-mode"></a>2.1.4 Signalisation de signal publicitaire RTMP avec « onAdCue » - Mode SCTE-35

Lorsque vous travaillez avec un flux de travail de production de diffusion plus avancé qui requiert la transmission du message de charge utile SCTE-35 complet au manifeste TLS ou DASH, il est préférable d’utiliser le « mode SCTE-35 » de la spécification [Adobe-Primetime].  Ce mode prend en charge les signaux SCTE-35 intrabande envoyés directement dans un encodeur live local, qui encode ensuite les signaux dans le flux RTMP à l’aide du « mode SCTE-35 » indiqué dans la spécification [Adobe-Primetime]. 

En général, les messages SCTE-35 peuvent apparaître uniquement dans des entrées de flux de transport MPEG-2 (TS) sur un encodeur local. Contactez le fabricant de votre encodeur pour plus d’informations sur la configuration d’une réception de flux de transport contenant SCTE-35 et son activation pour le transfert vers RTMP en mode Adobe SCTE-35.

Dans ce scénario, la charge utile suivante DOIT être envoyée à partir de l’encodeur local à l’aide du type de message **« onAdCue »** [AMF0].

| Nom du champ | Type de champ | Requis ? | Descriptions                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---------- | ---------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| cue        | String     | Obligatoire  | Message d'événement.  Pour les messages [SCTE-35], il DOIT s’agir de [RFC4648] binary splice_info_section() codé en Base64 de façon à ce que les messages soient envoyés aux clients HLS, Smooth Streaming et Dash.                                                                                                                                                                                                                               |
| type       | String     | Obligatoire  | URN ou URL identifiant le schéma du message. Pour les messages [SCTE-35], cela **DOIT** être **« scte35 »** de façon à ce que les messages soient envoyés aux clients HLS, Smooth Streaming et Dash conformément à la norme [Adobe-Primetime]. Éventuellement, l’URN « urn: SCTE:scte35:2013:bin » peut également être utilisée pour signaler un message [SCTE-35].                                                                                                        |
| id         | String     | Obligatoire  | Identificateur unique décrivant la jointure ou le segment. Identifie cette instance du message.  Les messages dont la sémantique est identique auront la même valeur.                                                                                                                                                                                                                                                       |
| duration   | Number     | Obligatoire  | Durée de l’événement ou du segment de jointure publicitaire, si elle est connue. Si elle est inconnue, la valeur **DOIT** être 0.                                                                                                                                                                                                                                                                                                                    |
| elapsed    | Number     | Facultatif  | Lorsque le signal publicitaire [SCTE-35] est répété pour le réglage, ce champ est la quantité de temps de présentation qui s’est écoulée depuis le début de la jointure. Les unités sont des fractions de seconde. En mode de [SCTE-35], cette valeur peut dépasser la durée originale spécifiée de la jointure ou du segment.                                                                                                                   |
| time       | Number     | Obligatoire  | Heure de présentation de l’événement ou de la jointure publicitaire.  L’heure et la durée de présentation **DOIVENT** être en phase avec les points d’accès du flux (SAP) de type 1 ou 2, comme défini dans la norme [ISO-14496-12] Annexe I. Pour les sorties HLS, l’heure et la durée **DOIVENT** correspondre aux limites de segment. L’heure et la durée de présentation de messages d’événements différents au sein du même flux d’événements NE DOIVENT PAS se chevaucher. Les unités sont des fractions de seconde. |

---

<!---
#### Example MPEG DASH .mpd manifest with SCTE-35 mode
See [Section 3.3.3.2 example DASH manifest with SCTE-35](#3332-example-mpeg-dash-manifest-mpd-with-multi-period-eventstream-using-adobe-scte35-mode-signaling)
--->

#### <a name="example-hls-manifest-m3u8-with-scte-35-mode-signal"></a>Exemple de manifeste HLS .m3u8 avec signal en mode SCTE-35
Consultez [Section 3.2.1.1 exemple de manifeste HLS avec SCTE-35](#3211-example-hls-manifest-m3u8-showing-ext-x-cue-signaling-of-scte-35)



## <a name="215-rtmp-ad-signaling-with-oncuepoint-for-elemental-live"></a>2.1.5 Signalisation de signal publicitaire RTMP avec « onCuePoint » pour Elemental Live

L’encodeur local Elemental Live prend en charge les marqueurs publicitaires dans le signal RTMP. Azure Media Services prend actuellement en charge uniquement le type de marqueur publicitaire « onCuePoint » pour RTMP.  Vous pouvez activer cette option dans les paramètres de groupe Adobe RTMP au sein des paramètres ou de l’API de l’encodeur live Elemental Media en définissant « **ad_markers** » sur « onCuePoint ».  Pour plus de détails, veuillez vous référer à la documentation d’Elemental Live. L’activation de cette fonctionnalité dans le groupe RTMP transmettra les signaux SCTE-35 aux sorties Adobe RTMP à traiter par Azure Media Services.

Le type de message « onCuePoint » est défini dans [Adobe-Flash-AS] et possède la structure de charge utile suivante une fois envoyé à partir de la sortie RTMP Elemental Live.


| Propriété   | Description                                                                                                                                                                                                                     |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name       | Le nom DOIT être «**scte35**» par Elemental Live.                                                                                                                                                                              |
| time       | La durée en secondes au cours de laquelle le point de repère s’est produit dans le fichier vidéo pendant la chronologie                                                                                                                                           |
| type       | Le type de point de repère DOIT être défini sur« **event** ».                                                                                                                                                                             |
| parameters | Un tableau associatif de chaînes de paires nom/valeur contenant les informations du message SCTE-35, y compris l’ID et la durée. Ces valeurs sont analysées par Azure Media Services et incluses dans la balise de décoration du manifeste. |


Quand ce mode de marqueur publicitaire est utilisé, la sortie du manifeste TLS est similaire au mode « simple » d’Adobe.


#### <a name="example-mpeg-dash-mpd-single-period-adobe-simple-mode-signals"></a>Exemple de MPEG DASH MPD, monophasé, signaux en mode simple Adobe

```xml
<?xml version="1.0" encoding="utf-8"?>
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" profiles="urn:mpeg:dash:profile:isoff-live:2011" type="dynamic" publishTime="2020-01-07T18:58:03Z" minimumUpdatePeriod="PT0S" timeShiftBufferDepth="PT58M56S" availabilityStartTime="2020-01-07T17:44:47Z" minBufferTime="PT7S">
    <Period start="PT0S">
        <EventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="scte35" timescale="10000000">
            <Event presentationTime="1583497601000000" duration="300000000" id="1085900"/>
            <Event presentationTime="1583500901666666" duration="300000000" id="1415966"/>
            <Event presentationTime="1583504202333333" duration="300000000" id="1746033"/>
            <Event presentationTime="1583507502666666" duration="300000000" id="2076066"/>
            <Event presentationTime="1583510803333333" duration="300000000" id="2406133"/>
            <Event presentationTime="1583514104000000" duration="300000000" id="2736200"/>
            <Event presentationTime="1583517404666666" duration="300000000" id="3066266"/>
            <Event presentationTime="1583520705333333" duration="300000000" id="3396333"/>
            <Event presentationTime="1583524006000000" duration="300000000" id="3726400"/>
            <Event presentationTime="1583527306666666" duration="300000000" id="4056466"/>
            <Event presentationTime="1583530607333333" duration="300000000" id="4386533"/>
        </EventStream>
        <AdaptationSet id="1" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.4D400C" maxWidth="256" maxHeight="144" startWithSAP="1">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="scte35"/>
            <SegmentTemplate timescale="10000000" presentationTimeOffset="1583486678426666" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="1583495318000000" d="64000000" r="34"/>
                    <S d="43000000"/>
                    <S d="21000000"/>
                    <!-- ... Truncated for brevity of sample-->

                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="1583495318000000" type="0" wallClockTime="2020-01-07T17:59:10.957Z" presentationTime="1583495318000000"/>
            <Representation id="1_V_video_3750956353252827751" bandwidth="149952" width="256" height="144"/>
        </AdaptationSet>
        <AdaptationSet id="2" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.2" lang="en">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="scte35"/>
            <Label>ambient</Label>
            <SegmentTemplate timescale="10000000" presentationTimeOffset="1583486678426666" media="QualityLevels($Bandwidth$)/Fragments(ambient=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(ambient=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="1583495254426666" d="64000000" r="35"/>
                    <S d="43093334"/>
                    <S d="20906666"/>
                    <!-- ... Truncated for brevity of sample-->

                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="1583495254426666" type="0" wallClockTime="2020-01-07T17:59:04.600Z" presentationTime="1583495254426666"/>
            <Representation id="5_A_ambient_9125670592623055209" bandwidth="96000" audioSamplingRate="48000"/>
        </AdaptationSet>
    </Period>
</MPD>
```

#### <a name="example-hls-playlist-adobe-simple-mode-signals-using-ext-x-cue-tag-truncated--for-brevity"></a>Exemple de sélection HLS, signaux en mode simple Adobe utilisant la balise EXT-X-CUE (tronquée « ... » par souci de concision)

L’exemple suivant illustre la sortie du packager dynamique Media Services pour un flux de réception RTMP utilisant les signaux en mode « simple » Adobe et la balise héritée [Adobe-Primetime] EXT-X-CUE.  

```
#EXTM3U
#EXT-X-VERSION:8
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:7
#EXT-X-INDEPENDENT-SEGMENTS
#EXT-X-PROGRAM-DATE-TIME:2020-01-07T17:44:47Z
#EXTINF:6.400000,no-desc
Fragments(video=1583486742000000,format=m3u8-aapl-v8)
#EXTINF:6.400000,no-desc
Fragments(video=1583486806000000,format=m3u8-aapl-v8)
...
#EXTINF:6.166667,no-desc
Fragments(video=1583487638000000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667
#EXTINF:0.233333,no-desc
Fragments(video=1583487699666666,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667,ELAPSED=0.233333
#EXTINF:6.400000,no-desc
Fragments(video=1583487702000000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667,ELAPSED=6.633333
#EXTINF:6.400000,no-desc
Fragments(video=1583487766000000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667,ELAPSED=13.033333
#EXTINF:6.400000,no-desc
Fragments(video=1583487830000000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667,ELAPSED=19.433333
#EXTINF:6.400000,no-desc
Fragments(video=1583487894000000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID=95766,TYPE="SpliceOut",DURATION=30.000000,TIME=158348769.966667,ELAPSED=25.833333
#EXTINF:4.166667,no-desc
Fragments(video=1583487958000000,format=m3u8-aapl-v8)
#EXTINF:2.233333,no-desc
Fragments(video=1583487999666666,format=m3u8-aapl-v8)
#EXTINF:6.400000,no-desc
Fragments(video=1583488022000000,format=m3u8-aapl-v8)
...
```

### <a name="216-cancellation-and-updates"></a>2.1.6 Annulation et mises à jour

Des messages peuvent être annulés ou mis à jour par l’envoi de plusieurs messages avec les mêmes heure et ID de présentation. L’heure et l’ID de présentation identifient de façon unique l’événement, et le dernier message reçu pour une heure de présentation spécifique qui correspond aux contraintes de décompte est le message traité. L’événement mis à jour remplace tout message reçu précédemment. La contrainte de décompte est de quatre secondes. Les messages reçus au moins quatre secondes avant l’heure de présentation seront traités.

## <a name="22-fragmented-mp4-ingest-smooth-streaming"></a>2.2 Réception de MP4 fragmentée (diffusion en continu lisse)

Pour les exigences relatives à la réception de flux en direct, voir [MS-SSTR-Ingest]. Les sections suivantes fournissent des détails sur la réception de métadonnées de présentation chronométrées.  Les métadonnées de présentation chronométrées sont reçues en tant que piste partiellement allouée, telle que définie dans la zone de manifeste du serveur en direct (voir MS-SSTR) et la zone de vidéo (« moov »).  

Chaque fragment se compose d’une zone de fragment vidéo (« moof ») et d’une zone de données multimédias (« mdat »), où la zone « mdat » est le message binaire.

Pour permettre l’insertion précise de publicités, l’encodeur DOIT diviser le fragment à l’heure de la présentation, où l’insertion du signal est requise.  Un nouveau fragment DOIT être créé, commençant par une nouvelle image IDR ou des points d’accès de flux (SAP) de type 1 ou 2, comme défini dans l’Annexe I de la norme [ISO-14496-12]
<!--- This allows the Azure Media Packager to properly generate an HLS manifest and a DASH multi-period manifest where the new Period begins at the frame-accurate splice conditioned presentation time. --->

### <a name="221-live-server-manifest-box"></a>2.2.1 Zone de manifeste du serveur en direct

La piste partiellement allouée **DOIT** être déclarée dans la zone de manifeste du serveur en direct avec une entrée **\<textstream\>** , et **DOIT** avoir l’ensemble d’attributs suivants :

| **Nom de l’attribut** | **Type de champ** | **Obligatoire ?** | **Description**                                                                                                                                                                                                              |
| ------------------ | -------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| systemBitrate      | Number         | Obligatoire      | **DOIT** être « 0 », indiquant une piste avec un débit binaire variable inconnu.                                                                                                                                                          |
| parentTrackName    | String         | Obligatoire      | **DOIT** être le nom de la piste parent sur laquelle les codes temporels de la piste partiellement allouée sont alignés à l’échelle de temps. La piste parent ne peut pas être une piste partiellement allouée.                                                                             |
| manifestOutput     | Boolean        | Obligatoire      | **DOIT** être « true » pour indiquer que la piste partiellement allouée sera incorporée dans le manifeste du client lisse.                                                                                                                        |
| Subtype            | String         | Obligatoire      | **DOIT** être le code « DATA » de quatre caractères.                                                                                                                                                                                  |
| Schéma             | String         | Obligatoire      | **DOIT** être un URN ou une URL identifiant le schéma de message. Pour les messages [SCTE-35], il **DOIT** s’agir de « urn : scte:scte35:2013:bin » de façon à ce que les messages soient envoyés aux clients HLS, Smooth Streaming et Dash conformément à la norme [SCTE-35]. |
| trackName          | String         | Obligatoire      | **DOIT** être le nom de la piste partiellement allouée. Le trackName peut servir à différencier plusieurs flux d’événements dont le schéma est identique. Chaque flux d’événements **DOIT** avoir un nom de piste unique.                                |
| timescale          | Number         | Facultatif      | **DOIT** être l’échelle de temps de la piste parent.                                                                                                                                                                               |

---

### <a name="222-movie-box"></a>2.2.2 Zone de vidéo

La zone de vidéo (« moov ») suit la zone de manifeste du serveur en direct en tant que partie de l’en-tête de flux pour une piste partiellement allouée.

La zone « moov » **DOIT** contenir une zone **TrackHeaderBox (« tkhd »)** , telle que définie dans la norme [ISO-14496-12] avec les contraintes suivantes :

| **Nom du champ** | **Type de champ**          | **Obligatoire ?** | **Description**                                                                                                    |
| -------------- | ----------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------ |
| duration       | Entier non signé 64 bits | Obligatoire      | **DOIT** être 0, étant donné que la zone de piste contient zéro exemple et que la durée totale des exemples dans la zone de piste est 0. |

---

La zone « moov » **DOIT** contenir un **HandlerBox (« hdlr »)** , tel que défini dans la norme [ISO-14496-12] avec les contraintes suivantes :

| **Nom du champ** | **Type de champ**          | **Obligatoire ?** | **Description**       |
| -------------- | ----------------------- | ------------- | --------------------- |
| handler_type   | Entier non signé 32 bits | Obligatoire      | **DOIT** être « meta ». |

---

La zone « stsd » **DOIT** contenir une zone de MetaDataSampleEntry avec un nom de codage tel que défini dans la norme [ISO-14496-12].  Par exemple, pour les messages SCTE-35, le nom de codage **DOIT** être « scte ».

### <a name="223-movie-fragment-box-and-media-data-box"></a>2.2.3 Zone de fragment vidéo et zone de données multimédias

Les fragments de piste partiellement allouée se composent d’une zone de fragment vidéo (« moof ») et d’une zone de données multimédias (« mdat »).

> [!NOTE]
> Pour permettre l’insertion précise de publicités, l’encodeur DOIT diviser le fragment à l’heure de la présentation, où l’insertion du signal est requise.  Un nouveau fragment DOIT être créé, commençant par une nouvelle image IDR ou des points d’accès de flux (SAP) de type 1 ou 2, comme défini dans l’Annexe I de la norme [ISO-14496-12]
> 

La zone MovieFragmentBox (« moof ») **DOIT** contenir une zone **TrackFragmentExtendedHeaderBox (« uuid »)** telle que définie dans [MS-SSTR] avec les champs suivants :

| **Nom du champ**         | **Type de champ**          | **Obligatoire ?** | **Description**                                                                                           |
| ---------------------- | ----------------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| fragment_absolute_time | Entier non signé 64 bits | Obligatoire      | **DOIT** être l’heure d’arrivée de l’événement. Cette valeur aligne le message sur la piste parent.           |
| fragment_duration      | Entier non signé 64 bits | Obligatoire      | **DOIT** être la durée de l’événement. La durée peut être zéro pour indiquer que la durée est inconnue. |

---


La zone de MediaDataBox (« mdat ») **DOIT** avoir le format suivant :

| **Nom du champ**          | **Type de champ**                   | **Obligatoire ?** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----------------------- | -------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| version                 | Entier non signé 32 bits (uimsbf) | Obligatoire      | Détermine le format du contenu de la zone « mdat ». Les versions non reconnues seront ignorées. Actuellement, la seule version prise en charge est 1.                                                                                                                                                                                                                                                                                                                                                                      |
| id                      | Entier non signé 32 bits (uimsbf) | Obligatoire      | Identifie cette instance du message. Des messages dont la sémantique est identique ont la même valeur ; le traitement de toute zone de message d’événement ayant le même ID est suffisant.                                                                                                                                                                                                                                                                                                                            |
| presentation_time_delta | Entier non signé 32 bits (uimsbf) | Obligatoire      | La somme de fragment_absolute_time, spécifié dans TrackFragmentExtendedHeaderBox et le presentation_time_delta, **DOIT** être l’heure de présentation de l’événement. L’heure et la durée de présentation **DOIVENT** être en phase avec les points d’accès du flux (SAP) de type 1 ou 2, comme défini dans la norme [ISO-14496-12] Annexe I. Pour les sorties HLS, l’heure et la durée **DOIVENT** correspondre aux limites de segment. L’heure et la durée de présentation de messages d’événements différents au sein du même flux d’événements **NE DOIVENT PAS** se chevaucher. |
| message                 | tableau d’octets                       | Obligatoire      | Message d'événement. Pour les messages [SCTE-35], le message est le binaire splice_info_section(). Pour les messages [SCTE-35], il **DOIT** s’agir de splice_info_section() de façon à ce que les messages soient envoyés aux clients HLS, Smooth Streaming et Dash conformément à la norme [SCTE-35]. Pour les messages [SCTE-35], le binaire splice_info_section() est la charge utile de la zone « mdat », et il n’**EST PAS** codé en Base64.                                                                                                                     |

---


### <a name="224-cancellation-and-updates"></a>2.2.4 Annulation et mises à jour

Des messages peuvent être annulés ou mis à jour par l’envoi de plusieurs messages avec les mêmes heure et ID de présentation.  L’heure et l’ID de présentation identifient l’événement. Le dernier message reçu pour une heure de présentation spécifique qui répond aux contraintes de décompte est le message traité. Le message mis à jour remplace tous les messages reçus précédemment.  La contrainte de décompte est de quatre secondes. Les messages reçus au moins quatre secondes avant l’heure de présentation seront traités. 


## <a name="3-timed-metadata-delivery"></a>3 Remise de métadonnées chronométrées

Les données de flux de données d’événements sont opaques pour Media Services. Media Services transmet simplement trois éléments d’informations entre le point de terminaison de réception et le point de terminaison client. Les propriétés suivantes sont fournies au client conformément à la norme [SCTE-35] et/ou au protocole de diffusion en continu du client :

1.  Schéma : URN ou URL identifiant le schéma du message.
2.  Heure de présentation : heure de présentation de l’événement dans la chronologie du média.
3.  Durée : durée de l’événement.
4.  ID : identificateur unique facultatif pour l’événement.
5.  Message : données de l’événement.

## <a name="31-microsoft-smooth-streaming-manifest"></a>3.1 Manifeste Microsoft Smooth Streaming  

Pour plus d’informations sur la mise en forme d’une piste de message partiellement allouée, consultez Gestion d’une piste partiellement allouée [MS-SSTR]. Pour les messages [SCTE35], Smooth Streaming génère le splice_info_section () encodé en Base64 dans un fragment partiellement alloué.
StreamIndex **DOIT** avoir un sous-type « Data » et CustomAttributes **DOIT** contenir un attribut avec Nom = « Schema » et Valeur = « urn: scte: scte35:2013:bin ».

#### <a name="smooth-client-manifest-example-showing-base64-encoded-scte35-splice_info_section"></a>Exemple de manifeste de client lissé avec un codage Base64 [SCTE35] splice_info_section ()
```xml
<?xml version="1.0" encoding="utf-8"?>
<SmoothStreamingMedia MajorVersion="2" MinorVersion="0" TimeScale="10000000" IsLive="true" Duration="0"
  LookAheadFragmentCount="2" DVRWindowLength="6000000000">

  <StreamIndex Type="video" Name="video" Subtype="" Chunks="0" TimeScale="10000000"
    Url="QualityLevels({bitrate})/Fragments(video={start time})">
    <QualityLevel Index="0" Bitrate="230000"
      CodecPrivateData="250000010FC3460B50878A0B5821FF878780490800704704DC0000010E5A67F840" FourCC="WVC1"
      MaxWidth="364" MaxHeight="272"/>
    <QualityLevel Index="1" Bitrate="305000"
      CodecPrivateData="250000010FC3480B50878A0B5821FF87878049080894E4A7640000010E5A67F840" FourCC="WVC1"
      MaxWidth="364" MaxHeight="272"/>
    <c t="0" d="20000000" r="300" />
  </StreamIndex>
  <StreamIndex Type="audio" Name="audio" Subtype="" Chunks="0" TimeScale="10000000"
    Url="QualityLevels({bitrate})/Fragments(audio={start time})">
    <QualityLevel Index="0" Bitrate="96000" CodecPrivateData="1000030000000000000000000000E00042C0"
      FourCC="WMAP" AudioTag="354" Channels="2" SamplingRate="44100" BitsPerSample="16" PacketSize="4459"/>
    <c t="0" d="20000000" r="300" />
  </StreamIndex>
  <StreamIndex Type="text" Name="scte35-sparse-stream" Subtype="DATA" Chunks="0" TimeScale="10000000"
    ParentStreamIndex="video" ManifestOutput="true" 
    Url="QualityLevels({bitrate})/Fragments(captions={start time})">
    <QualityLevel Index="0" Bitrate="0" CodecPrivateData="" FourCC="">
      <CustomAttributes>
        <Attribute Name="Scheme" Value="urn:scte:scte35:2013:bin"/>
      </CustomAttributes>
    </QualityLevel>
    <!-- The following <c> and <f> fragments contains the base64-encoded [SCTE35] splice_info_section() message -->
    <c t="600000000" d="300000000">    <f>PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTUwMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</f>
    </c>
    <c t="1200000000" d="400000000">      <f>PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48QWNxdWlyZWRTaWduYWwgeG1sbnM9InVybjpjYWJsZWxhYnM6bWQ6eHNkOnNpZ25hbGluZzozLjAiIGFjcXVpc2l0aW9uUG9pbnRJZGVudGl0eT0iRVNQTl9FYXN0X0FjcXVpc2l0aW9uX1BvaW50XzEiIGFjcXVpc2l0aW9uU2lnbmFsSUQ9IjRBNkE5NEVFLTYyRkExMUUxQjFDQTg4MkY0ODI0MDE5QiIgYWNxdWlzaXRpb25UaW1lPSIyMDEyLTA5LTE4VDEwOjE0OjI2WiI+PFVUQ1BvaW50IHV0Y1BvaW50PSIyMDEyLTA5LTE4VDEwOjE0OjM0WiIvPjxTQ1RFMzVQb2ludERlc2NyaXB0b3Igc3BsaWNlQ29tbWFuZFR5cGU9IjUiPjxTcGxpY2VJbnNlcnQgc3BsaWNlRXZlbnRJRD0iMzQ0NTY4NjkxIiBvdXRPZk5ldHdvcmtJbmRpY2F0b3I9InRydWUiIHVuaXF1ZVByb2dyYW1JRD0iNTUzNTUiIGR1cmF0aW9uPSJQVDFNMFMiIGF2YWlsTnVtPSIxIiBhdmFpbHNFeHBlY3RlZD0iMTAiLz48L1NDVEUzNVBvaW50RGVzY3JpcHRvcj48U3RyZWFtVGltZXM+PFN0cmVhbVRpbWUgdGltZVR5cGU9IkhTUyIgdGltZVZhbHVlPSI1MTYyMDAwMDAwMDAiLz48L1N0cmVhbVRpbWVzPjwvQWNxdWlyZWRTaWduYWw+</f>
    </c>
  </StreamIndex>
</SmoothStreamingMedia>
```

## <a name="32-apple-hls-manifest-decoration"></a>3.2 Décoration de manifeste Apple HLS

Azure Media Services prend en charge les balises de manifeste TLS suivantes pour signaler les informations d’usage publicitaire lors d’un événement en direct ou à la demande. 

<!--- EXT-X-DATERANGE as defined in Apple HLS [RFC8216] --->
- EXT-X-CUE tel que défini dans [Adobe-Primetime]
<!--- this mode is considered "legacy".  Customers should adopt the EXT-X-DATERANGE tag when possible. --->

La sortie des données pour chaque balise varie selon le mode de signal de réception utilisé. Par exemple, la réception RTMP avec le mode simple de Adobe ne contient pas la charge utile encodée en base64 SCTE-35.

<!---
## 3.2.1 Apple HLS with EXT-X-DATERANGE (recommended)

The Apple HTTP Live Streaming [RFC8216] specification allows for signaling of [SCTE-35] messages. The messages are inserted into the segment playlist in an EXT-X-DATERANGE tag per [RFC8216] section titled "Mapping SCTE-35 into EXT-X-DATERANGE".  The client application layer can parse the M3U playlist and process M3U tags, or receive the events through the Apple player framework.  

The **RECOMMENDED** approach in Azure Media Services (version 3 API) is to follow [RFC8216] and use the EXT-X_DATERANGE tag for [SCTE35] ad avail decoration in the manifest.
--->


## <a name="3211-example-hls-manifest-m3u8-showing-ext-x-cue-signaling-of-scte-35"></a>3.2.1.1 Exemple de manifeste HLS .m3u8 présentant la signalisation EXT-X-CUE de SCTE-35

L'exemple suivant de sortie du manifeste HLS du packager dynamique Media Services affiche la balise EXT-X-CUE pour [Adobe-Primetime] en mode SCTE35. 

```
#EXTM3U
#EXT-X-VERSION:8
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:2
#EXT-X-INDEPENDENT-SEGMENTS
#EXT-X-PROGRAM-DATE-TIME:2020-01-07T19:40:50Z
#EXTINF:1.501500,no-desc
Fragments(video=22567545,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22702680,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22837815,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22972950,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=23108085,format=m3u8-aapl-v8)
#EXTINF:1.234567,no-desc
Fragments(video=23243220,format=m3u8-aapl-v8)
#EXTINF:0.016689,no-desc
Fragments(video=23354331,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=0.000022
#EXTINF:0.250244,no-desc
Fragments(video=23355833,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=0.250267
#EXTINF:0.850856,no-desc
Fragments(video=23378355,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.101122
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=0.000000,TIME=260.610344,CUE="/DAgAAAAAAXdAP/wDwUAAAPqf0/+AWXk0wABAQEAAGB86Fo="
#EXTINF:0.650644,no-desc
Fragments(video=23454932,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.751767
#EXTINF:0.050044,no-desc
Fragments(video=23513490,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.801811
#EXTINF:1.451456,no-desc
Fragments(video=23517994,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=3.253267
#EXTINF:1.501500,no-desc
Fragments(video=23648625,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=4.754767
#EXTINF:1.501500,no-desc
Fragments(video=23783760,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=6.256267
#EXTINF:1.501500,no-desc
Fragments(video=23918895,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=7.757767
#EXTINF:1.501500,no-desc
Fragments(video=24054030,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=9.259267
#EXTINF:1.501500,no-desc
Fragments(video=24189165,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=10.760767
#EXTINF:1.501500,no-desc
Fragments(video=24324300,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=12.262267
#EXTINF:1.501500,no-desc
Fragments(video=24459435,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=13.763767
#EXTINF:1.501500,no-desc
Fragments(video=24594570,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=15.265267
#EXTINF:1.501500,no-desc
Fragments(video=24729705,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=16.766767
#EXTINF:1.501500,no-desc
Fragments(video=24864840,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=18.268267
#EXTINF:1.501500,no-desc
Fragments(video=24999975,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=19.769767
#EXTINF:1.501500,no-desc
Fragments(video=25135110,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=21.271267
#EXTINF:1.501500,no-desc
Fragments(video=25270245,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=22.772767
#EXTINF:1.501500,no-desc
Fragments(video=25405380,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=24.274267
#EXTINF:1.501500,no-desc
Fragments(video=25540515,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=25.775767
#EXTINF:1.501500,no-desc
Fragments(video=25675650,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=27.277267
#EXTINF:1.501500,no-desc
Fragments(video=25810785,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=28.778767
#EXTINF:1.501500,no-desc
Fragments(video=25945920,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=30.280267
#EXTINF:1.501500,no-desc
Fragments(video=26081055,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=31.781767
#EXTINF:1.501500,no-desc
Fragments(video=26216190,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=33.283267
#EXTINF:1.501500,no-desc
Fragments(video=26351325,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=34.784767
#EXTINF:1.501500,no-desc
Fragments(video=26486460,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=36.286267
#EXTINF:1.501500,no-desc
Fragments(video=26621595,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=37.787767
#EXTINF:1.501500,no-desc
Fragments(video=26756730,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=39.289267
#EXTINF:1.501500,no-desc
Fragments(video=26891865,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=40.790767
#EXTINF:1.501500,no-desc
Fragments(video=27027000,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=42.292267
#EXTINF:1.501500,no-desc
Fragments(video=27162135,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=43.793767
#EXTINF:1.501500,no-desc
Fragments(video=27297270,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=45.295267
#EXTINF:1.501500,no-desc
Fragments(video=27432405,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=46.796767
#EXTINF:1.501500,no-desc
Fragments(video=27567540,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=48.298267
#EXTINF:1.501500,no-desc
Fragments(video=27702675,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=49.799767
#EXTINF:1.501500,no-desc
Fragments(video=27837810,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=51.301267
#EXTINF:1.501500,no-desc
Fragments(video=27972945,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=52.802767
#EXTINF:1.501500,no-desc
Fragments(video=28108080,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=54.304267
#EXTINF:1.501500,no-desc
Fragments(video=28243215,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=55.805767
#EXTINF:1.501500,no-desc
Fragments(video=28378350,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=57.307267
#EXTINF:1.501500,no-desc
Fragments(video=28513485,format=m3u8-aapl-v8)
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=58.808767
#EXTINF:1.501500,no-desc
Fragments(video=28648620,format=m3u8-aapl-v8)

```


<!---
THIS VERSION HAS THE HLSv8 DATERANGE Tags in it
~~~
#EXTM3U
#EXT-X-VERSION:8
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:2
#EXT-X-INDEPENDENT-SEGMENTS
#EXT-X-PROGRAM-DATE-TIME:2020-01-07T19:40:50Z
#EXTINF:1.501500,no-desc
Fragments(video=22567545,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22702680,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22837815,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=22972950,format=m3u8-aapl-v8)
#EXTINF:1.501500,no-desc
Fragments(video=23108085,format=m3u8-aapl-v8)
#EXTINF:1.234567,no-desc
Fragments(video=23243220,format=m3u8-aapl-v8)
#EXTINF:0.016689,no-desc
Fragments(video=23354331,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=0.000022
#EXTINF:0.250244,no-desc
Fragments(video=23355833,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=0.250267
#EXTINF:0.850856,no-desc
Fragments(video=23378355,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.101122
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:10.610Z",SCTE35-IN=0xFC30200000000005DD00FFF00F05000003EA7F4FFE0165E4D3000101010000607CE85A
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=0.000000,TIME=260.610344,CUE="/DAgAAAAAAXdAP/wDwUAAAPqf0/+AWXk0wABAQEAAGB86Fo="
#EXTINF:0.650644,no-desc
Fragments(video=23454932,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.751767
#EXTINF:0.050044,no-desc
Fragments(video=23513490,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=1.801811
#EXTINF:1.451456,no-desc
Fragments(video=23517994,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=3.253267
#EXTINF:1.501500,no-desc
Fragments(video=23648625,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=4.754767
#EXTINF:1.501500,no-desc
Fragments(video=23783760,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=6.256267
#EXTINF:1.501500,no-desc
Fragments(video=23918895,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=7.757767
#EXTINF:1.501500,no-desc
Fragments(video=24054030,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=9.259267
#EXTINF:1.501500,no-desc
Fragments(video=24189165,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=10.760767
#EXTINF:1.501500,no-desc
Fragments(video=24324300,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=12.262267
#EXTINF:1.501500,no-desc
Fragments(video=24459435,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=13.763767
#EXTINF:1.501500,no-desc
Fragments(video=24594570,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=15.265267
#EXTINF:1.501500,no-desc
Fragments(video=24729705,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=16.766767
#EXTINF:1.501500,no-desc
Fragments(video=24864840,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=18.268267
#EXTINF:1.501500,no-desc
Fragments(video=24999975,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=19.769767
#EXTINF:1.501500,no-desc
Fragments(video=25135110,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=21.271267
#EXTINF:1.501500,no-desc
Fragments(video=25270245,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=22.772767
#EXTINF:1.501500,no-desc
Fragments(video=25405380,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=24.274267
#EXTINF:1.501500,no-desc
Fragments(video=25540515,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=25.775767
#EXTINF:1.501500,no-desc
Fragments(video=25675650,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=27.277267
#EXTINF:1.501500,no-desc
Fragments(video=25810785,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=28.778767
#EXTINF:1.501500,no-desc
Fragments(video=25945920,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=30.280267
#EXTINF:1.501500,no-desc
Fragments(video=26081055,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=31.781767
#EXTINF:1.501500,no-desc
Fragments(video=26216190,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=33.283267
#EXTINF:1.501500,no-desc
Fragments(video=26351325,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=34.784767
#EXTINF:1.501500,no-desc
Fragments(video=26486460,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=36.286267
#EXTINF:1.501500,no-desc
Fragments(video=26621595,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=37.787767
#EXTINF:1.501500,no-desc
Fragments(video=26756730,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=39.289267
#EXTINF:1.501500,no-desc
Fragments(video=26891865,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=40.790767
#EXTINF:1.501500,no-desc
Fragments(video=27027000,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=42.292267
#EXTINF:1.501500,no-desc
Fragments(video=27162135,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=43.793767
#EXTINF:1.501500,no-desc
Fragments(video=27297270,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=45.295267
#EXTINF:1.501500,no-desc
Fragments(video=27432405,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=46.796767
#EXTINF:1.501500,no-desc
Fragments(video=27567540,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=48.298267
#EXTINF:1.501500,no-desc
Fragments(video=27702675,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=49.799767
#EXTINF:1.501500,no-desc
Fragments(video=27837810,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=51.301267
#EXTINF:1.501500,no-desc
Fragments(video=27972945,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=52.802767
#EXTINF:1.501500,no-desc
Fragments(video=28108080,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=54.304267
#EXTINF:1.501500,no-desc
Fragments(video=28243215,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=55.805767
#EXTINF:1.501500,no-desc
Fragments(video=28378350,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=57.307267
#EXTINF:1.501500,no-desc
Fragments(video=28513485,format=m3u8-aapl-v8)
#EXT-X-DATERANGE:ID="1002",START-DATE="2020-01-07T19:45:09.509Z",SCTE35-OUT=0xFC30250000000005DD00FFF01405000003EA7FEFFE016461B8FE00526363000101010000F20D5E37
#EXT-X-CUE:ID="1002",TYPE="scte35",DURATION=59.993278,TIME=259.509244,CUE="/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==",ELAPSED=58.808767
#EXTINF:1.501500,no-desc
Fragments(video=28648620,format=m3u8-aapl-v8)

~~~

--->

## <a name="322-apple-hls-with-adobe-primetime-ext-x-cue"></a>3.2.2 Apple HLS avec Adobe Primetime EXT-X-CUE

Media Services (API version 2 et 3) prend en charge la sortie de la balise EXT-X-CUE telle que définie dans [Adobe-Primetime] « Mode SCTE-35 ». Dans ce mode, Azure Media Services incorpore le code Base64 [SCTE-35] splice_info_section () dans la balise EXT-X-CUE.  

La balise EXT-X-CUE « héritée » est définie comme ci-dessous et peut également faire référence dans la spécification [Adobe-Primetime]. Cette option ne doit être utilisée que pour les signaux SCTE35 hérités, si nécessaire. Sinon, la balise recommandée est définie comme EXT-X-DATERANGE dans [RFC8216]. 

| **Nom de l’attribut** | **Type**                      | **Obligatoire ?**                             | **Description**                                                                                                                                                                                                                                                                          |
| ------------------ | ----------------------------- | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CUE                | Chaîne entre guillemets.                 | Obligatoire                                  | Message encodé sous forme de chaîne en Base64, comme décrit dans [RFC4648]. Pour les messages [SCTE-35], il s’agit de la zone splice_info_section() en Base64.                                                                                                                                      |
| TYPE               | Chaîne entre guillemets.                 | Obligatoire                                  | URN ou URL identifiant le schéma du message. Pour les messages [SCTE-35], le type prend la valeur spéciale « scte35 ».                                                                                                                                                                          |
| id                 | Chaîne entre guillemets.                 | Obligatoire                                  | Identificateur unique de l’événement. Si l’ID n’est pas spécifié lors de la réception du message, Azure Media Services génère un id unique.                                                                                                                                              |
| DURATION           | Nombre décimal à virgule flottante. | Obligatoire                                  | Durée de l’événement. Si elle est inconnue, la valeur **DOIT** être 0. Les unités sont des fractions de seconde.                                                                                                                                                                                           |
| ELAPSED            | Nombre décimal à virgule flottante. | Facultatif, mais requis pour la fenêtre glissante. | Lorsque le signal est répété pour prendre en charge une fenêtre glissante de présentation, ce champ **DOIT** être le temps de présentation qui s’est écoulé depuis le début de l’événement. Les unités sont des fractions de seconde. Cette valeur peut dépasser la durée originale spécifiée de la jointure ou du segment. |
| TEMPS               | Nombre décimal à virgule flottante. | Obligatoire                                  | Heure de présentation de l’événement. Les unités sont des fractions de seconde.                                                                                                                                                                                                                        |

La couche d’application du lecteur HLS utilisera le TYPE pour identifier le format du message, décoder celui-ci, appliquer les conversions de temps nécessaires, et traiter l’événement.  Les événements sont synchronisé dans la sélection de segments de la piste parent, en fonction de leur horodatage.  Ils sont insérés devant le segment le plus proche (balise #EXTINF).


### <a name="323-hls-m3u8-manifest-example-using-adobe-primetime-ext-x-cue"></a>3.2.3 Exemple de manifeste HLS .m3u8 utilisant EXT-X-CUE Adobe Primetime

L’exemple suivant montre l’ornement de manifeste HLS à l’aide de la balise EXT-X-CUE Adobe Primetime.  Le paramètre « CUE » contient uniquement les propriétés TYPE et Duration, ce qui signifie qu’il s’agissait d’une source RTMP utilisant la signalisation en mode simple Adobe.  
<!---If this was a SCTE-35 mode signal, the tag would include the base64 encoded binary SCTE-35 payload as seen in the [3.2.1.1 example](#3211-example-hls-manifest-m3u8-showing-ext-x-daterange-signaling-of-scte-35).
--->


```
#EXTM3U
#EXT-X-VERSION:4
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-ALLOW-CACHE:NO
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-TARGETDURATION:11
#EXT-X-PROGRAM-DATE-TIME:2019-12-10T09:18:14Z
#EXTINF:10.010000,no-desc
Fragments(video=4011540820,format=m3u8-aapl)
#EXTINF:10.010000,no-desc
Fragments(video=4011550830,format=m3u8-aapl)
#EXTINF:10.010000,no-desc
Fragments(video=4011560840,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000
#EXTINF:8.008000,no-desc
Fragments(video=4011570850,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=0.593000
#EXTINF:4.170000,no-desc
Fragments(video=4011578858,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=4.763000
#EXTINF:9.844000,no-desc
Fragments(video=4011583028,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=14.607000
#EXTINF:10.010000,no-desc
Fragments(video=4011592872,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=24.617000
#EXTINF:10.010000,no-desc
Fragments(video=4011602882,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=34.627000
#EXTINF:10.010000,no-desc
Fragments(video=4011612892,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=44.637000
#EXTINF:10.010000,no-desc
Fragments(video=4011622902,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=54.647000
#EXTINF:10.010000,no-desc
Fragments(video=4011632912,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=64.657000
#EXTINF:10.010000,no-desc
Fragments(video=4011642922,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=74.667000
#EXTINF:10.010000,no-desc
Fragments(video=4011652932,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=84.677000
#EXTINF:10.010000,no-desc
Fragments(video=4011662942,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=94.687000
#EXTINF:10.010000,no-desc
Fragments(video=4011672952,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=104.697000
#EXTINF:10.010000,no-desc
Fragments(video=4011682962,format=m3u8-aapl)
#EXT-X-CUE:ID=4011578265,TYPE="SpliceOut",DURATION=119.987000,TIME=4011578.265000,ELAPSED=114.707000
#EXTINF:10.010000,no-desc
Fragments(video=4011692972,format=m3u8-aapl)
#EXTINF:8.008000,no-desc
Fragments(video=4011702982,format=m3u8-aapl)

```

### <a name="324-hls-message-handling-for-adobe-primetime-ext-x-cue"></a>3.2.4 Gestion des messages HLS pour EXT-X-CUE Adobe Primetime

Les événements sont signalés dans la sélection de segments de chaque piste audio et vidéo. La position de la balise EXT-X-CUE **DOIT** toujours être immédiatement devant le premier segment HLS (pour jointure externe ou début de segment) ou immédiatement derrière les derniers segments HLS (pour jointure interne ou fin du segment) auxquels ses attributs TIME et DURATION se rapportent, comme requis par [Adobe-Primetime].

Quand une fenêtre glissante de présentation est activée, la balise EXT-X-CUE **DOIT** être répétée assez souvent pour que la jointure ou le segment soient toujours entièrement décrits dans la sélection de segments, et l’attribut ELAPSED **DOIT** être utilisé pour indiquer la durée d’activité de la jointure ou du segment, comme requis par [Adobe-Primetime].

Quand une fenêtre glissante de présentation est activée, les balises EXT-X-CUE sont supprimées de la sélection de segments lorsque le temps de multimédia auquel elles font référence est en dehors de la fenêtre glissante de présentation.

## <a name="33-dash-manifest-decoration-mpd"></a>3.3 Décoration de manifeste DASH (MPD)

[MPEGDASH] permet de signaler les événements de trois façons :

1.  Événements signalés dans le MPD EventStream
2.  Événements signalés dans la bande à l’aide de la boîte de message d’événement (« EMSG »)
3.  Une combinaison de 1 et 2

Les événements signalés dans le MPD EventStream sont utiles pour une diffusion en continu VOD, car les clients ont accès à tous les événements, dès que le MPD est téléchargé. Ils sont également utiles pour la signalisation SSAI, où le fournisseur SSAI en aval doit analyser les signaux à partir d’un manifeste MPD de plusieurs périodes, et insérer du contenu publicitaire de manière dynamique.  La solution intrabande (« EMSG ») est utile pour le streaming en direct où les clients n’ont pas besoin de télécharger à nouveau le MPD, ou il n’y a aucune manipulation de manifeste SSAI entre le client et l’origine. 

Le comportement par défaut Azure Media Services pour DASH est de signaler dans MPD EventStreams MPD et intrabande à l’aide de la boîte de message d’événement (« EMSG »).

Les messages de signal ingérés via [RTMP] ou [MS-SSTR-Ingest] sont mappés dans des événements DASH, à l’aide de boîtes « EMSG » intrabandes et/ou MPD EventStreams. 

La signalisation SCTE-35 intrabande pour DASH suit la définition et les exigences définies dans [SCTE-214-3] et également dans la section 13.12.2 de la norme [DASH-IF-IOP] (« SCTE35 Events »). 

Pour le transport intrabande [SCTE-35], la boîte de message d’événement (« EMSG ») utilise le schemeId = « urn:scte:scte35:2013:bin ». Pour la décoration du manifeste MPD, le schemeId EventStream utilise « urn:scte:scte35:2014:xml+bin ».  Ce format est une représentation XML de l’événement, qui comprend une sortie binaire codée en base64 du message SCTE-35 complet qui est arrivé à la réception. 

Les définitions de référence normative de transport des messages de signal [SCTE-35] en DASH sont disponibles dans la section 6.7.4 de la norme [SCTE-214-1] (MPD) et la section 7.3.2 de la norme [SCTE-214-3] (transport des messages de signal SCTE 35).

### <a name="331-mpeg-dash-mpd-eventstream-signaling"></a>3.3.1 Signalisation EventStream MPEG DASH (MPD)

La décoration de manifeste (MPD) sera signalée dans le MPD à l’aide de l’élément EventStream qui apparaît dans l’élément Period. Le schemeId utilisé est « urn:scte:scte35:2014:xml+bin ».


> [!NOTE]
> À des fins de concision, [SCTE-35] autorise l’utilisation de la section codée en Base64 dans l’élément Signal.Binary (plutôt que l’élément Signal.SpliceInfoSection) comme alternative au transport d’un message de signal entièrement analysé.
> Azure Media Services utilise cette approche « xml+bin » pour signaler dans le manifeste MPD.
> Il s’agit également de la méthode recommandée utilisée dans la norme [DASH-IF-IOP] - Voir la section intitulée [« flux d’événements d’insertion publicitaire » du guide DASH-IF-IOP](https://dashif-documents.azurewebsites.net/DASH-IF-IOP/master/DASH-IF-IOP.html#ads-insertion-event-streams)
> 


L’élément EventStream a les attributs suivants :

| **Nom de l’attribut** | **Type**                | **Obligatoire ?** | **Description**                                                                                                                                                                                                                                                                                                                                                                         |
| ------------------ | ----------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| scheme_id_uri      | string                  | Obligatoire      | Identifie le schéma du message. Le schéma est défini sur la valeur de l’attribut Scheme dans la zone de manifeste du serveur en direct. La valeur **DOIT**  être un URN ou une URL identifiant le schéma de message. Le schemeId de sortie pris en charge doit être « urn:scte:scte35:2014: xml+bin » par la section 6.7.4 de la norme [SCTE-214-1] (MPD), car le service prend en charge uniquement « xml+bin » à ce stade pour la concision dans le MPD. |
| value              | string                  | Facultatif      | Valeur de chaîne supplémentaire utilisée par les propriétaires du schéma pour personnaliser la sémantique du message. Afin de différencier plusieurs flux d’événements dont le schéma est identique, la valeur **DOIT** être définie sur le nom du flux d’événements (trackName pour [MS-SSTR-Ingest] ou nom du message AMF pour une réception [RTMP]).                                                                         |
| Échelle de temps          | Entier non signé 32 bits | Obligatoire      | L’échelle de temps, en cycles par seconde.                                                                                                                                                                                                                                                                                                                                                     |


### <a name="332-example-event-streams-for-mpeg-dash"></a>3.3.2 Exemples de flux d’événements pour MPEG DASH

#### <a name="3321-example-mpeg-dash-mpd-manifest-signaling-of-rtmp-streaming-using-adobe-simple-mode"></a>3.3.2.1 Exemple de signalement de manifeste MPEG DASH .mpd de la diffusion RTMP en continu utilisant le mode simple Adobe

L’exemple suivant montre un extrait EventStream du packager dynamique Media Services pour un flux RTMP utilisant la signalisation en mode « simple » Adobe.

```xml
<!-- Example EventStream element using "urn:com:adobe:dpi:simple:2015" Adobe simple signaling per [Adobe-Primetime] -->
    <EventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="simplesignal" timescale="10000000">
        <Event presentationTime="1583497601000000" duration="300000000" id="1085900"/>
        <Event presentationTime="1583500901666666" duration="300000000" id="1415966"/>
        <Event presentationTime="1583504202333333" duration="300000000" id="1746033"/>
        <Event presentationTime="1583507502666666" duration="300000000" id="2076066"/>
        <Event presentationTime="1583510803333333" duration="300000000" id="2406133"/>
        <Event presentationTime="1583514104000000" duration="300000000" id="2736200"/>
        <Event presentationTime="1583517404666666" duration="300000000" id="3066266"/>
        <Event presentationTime="1583520705333333" duration="300000000" id="3396333"/>
        <Event presentationTime="1583524006000000" duration="300000000" id="3726400"/>
        <Event presentationTime="1583527306666666" duration="300000000" id="4056466"/>
        <Event presentationTime="1583530607333333" duration="300000000" id="4386533"/>
    </EventStream>
```


#### <a name="3322-example-mpeg-dash-mpd-manifest-signaling-of-an-rtmp-stream-using-adobe-scte-35-mode"></a>3.3.2.2 Exemple de signalement de manifeste MPEG DASH .mpd d’un flux RTMP utilisant le mode SCTE-35 Adobe

L’exemple suivant montre un extrait EventStream du packager dynamique Media Services pour un flux RTMP utilisant la signalisation en mode SCTE-35 Adobe.

Exemple d’élément EventStream utilisant la signalisation de style xml + bin par [SCTE-214-1]

```xml

      <EventStream schemeIdUri="urn:scte:scte35:2014:xml+bin" value="scte35" timescale="10000000">
        <Event presentationTime="2595092444" duration="11011000" id="1002">
            <Signal xmlns="http://www.scte.org/schemas/35/2016">
                <Binary>/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==</Binary>
            </Signal>
        </Event>
        <Event presentationTime="2606103444" id="1002">
            <Signal xmlns="http://www.scte.org/schemas/35/2016">
                <Binary>/DAgAAAAAAXdAP/wDwUAAAPqf0/+AWXk0wABAQEAAGB86Fo=</Binary>
            </Signal>
        </Event>
      </EventStream>
```



> [!IMPORTANT]
> Notez que presentationTime est l’heure de présentation de l’événement [SCTE-35] traduite pour être relative à l’heure de début de la période, et non à l’heure d’arrivée du message.
> [MPEGDASH] définit le Event@presentationTime comme « Spécifie le temps de présentation multimédia de l’événement par rapport au début de la période.
> La valeur de la durée de présentation en secondes est la division de la valeur de cet attribut et la valeur de l’attribut EventStream@timescale.
> Si elle n’est pas présente, la valeur de l’heure de présentation est 0.


#### <a name="3331-example-mpeg-dash-manifest-mpd-with-single-period-eventstream-using-adobe-simple-mode-signals"></a>3.3.3.1 Exemple de manifeste MPEG DASH (MPD) avec une période unique, EventStream, utilisant des signaux en mode simple Adobe

L’exemple suivant montre la sortie du packager dynamique Media Services pour un flux RTMP source utilisant la méthode de signal publicitaire en mode « simple » Adobe. La sortie est un manifeste de période unique qui affiche un EventStream à l’aide de l’URI schemeId défini sur « urn:com:adobe:dpi:simple:2015 » et la propriété de valeur définie sur « simplesignal ».
Chaque signal simple est fourni dans un élément d’événement avec les propriétés @presentationTime, @durationet @id remplies en fonction des signaux simples entrants.

```xml
<?xml version="1.0" encoding="utf-8"?>
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" profiles="urn:mpeg:dash:profile:isoff-live:2011" type="static" mediaPresentationDuration="PT28M1.680S" minBufferTime="PT3S">
    <Period>
        <EventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="simplesignal" timescale="1000">
            <Event presentationTime="4011578265" duration="119987" id="4011578265"/>
        </EventStream>
        <AdaptationSet id="1" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.4D4028" maxWidth="1920" maxHeight="1080" startWithSAP="1">
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="simplesignal"/>
            <ProducerReferenceTime id="4011460740" type="0" wallClockTime="2020-01-25T19:35:54.740Z" presentationTime="4011460740"/>
            <SegmentTemplate timescale="1000" presentationTimeOffset="4011460740" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="4011460740" d="2002" r="57"/>
                    <S d="1401"/>
                    <S d="601"/>
                    <S d="2002"/>

                     <!--> ... video segments truncated for sample brevity </-->

                </SegmentTimeline>
            </SegmentTemplate>
            <Representation id="1_V_video_14759481473095519504" bandwidth="6000000" width="1920" height="1080"/>
            <Representation id="1_V_video_1516803357996956148" bandwidth="4000000" codecs="avc1.4D401F" width="1280" height="720"/>
            <Representation id="1_V_video_5430608182379669372" bandwidth="2600000" codecs="avc1.4D401F" width="960" height="540"/>
            <Representation id="1_V_video_3780180650986497347" bandwidth="1000000" codecs="avc1.4D401E" width="640" height="360"/>
            <Representation id="1_V_video_13759117363700265707" bandwidth="699000" codecs="avc1.4D4015" width="480" height="270"/>
            <Representation id="1_V_video_6140004908920393176" bandwidth="400000" codecs="avc1.4D4015" width="480" height="270"/>
            <Representation id="1_V_video_10673801877453424365" bandwidth="200000" codecs="avc1.4D400D" width="320" height="180"/>
        </AdaptationSet>
        <AdaptationSet id="2" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.2">
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="simplesignal"/>
            <ProducerReferenceTime id="4011460761" type="0" wallClockTime="2020-01-25T19:35:54.761Z" presentationTime="4011460761"/>
            <Label>audio</Label>
            <SegmentTemplate timescale="1000" presentationTimeOffset="4011460740" media="QualityLevels($Bandwidth$)/Fragments(audio=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(audio=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="4011460761" d="1984"/>
                    <S d="2005" r="1"/>
                    <S d="2006"/>

                    <!--> ... audio segments truncated for example brevity </-->

                </SegmentTimeline>
            </SegmentTemplate>
            <Representation id="5_A_audio_17504386117102112482" bandwidth="128000" audioSamplingRate="48000"/>
        </AdaptationSet>
    </Period>
</MPD>

```

<!---
#### 3.3.3.2 Example MPEG DASH manifest (MPD) with multi-period, EventStream, using Adobe SCTE35 mode signaling

The following example shows the output from the Media Services dynamic packager for a source RTMP stream using the Adobe SCTE35 mode signaling.
In this case, the output manifest is a multi-period DASH .mpd with an EventStream element, and @schemeIdUri property set to "urn:scte:scte35:2014:xml+bin" and a @value property set to "scte35". Each Event element in the EventStream contains the full base64 encoded binary SCTE35 signal 

~~~ xml
<?xml version="1.0" encoding="utf-8"?>
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" profiles="urn:mpeg:dash:profile:isoff-live:2011" type="dynamic" publishTime="2020-01-07T19:42:44Z" minimumUpdatePeriod="PT0S" timeShiftBufferDepth="PT58M56S" availabilityStartTime="2020-01-07T19:40:50Z" minBufferTime="PT4S">
    <Period start="PT2M48.168S" id="main-content_0">
        <AdaptationSet id="1" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.640020" maxWidth="1280" maxHeight="720" startWithSAP="1">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <SegmentTemplate timescale="90000" presentationTimeOffset="15135120" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="15135120" d="135135" r="59"/>
                    <S d="111111"/>
                    <S d="1502"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="15135120" type="0" wallClockTime="2020-01-07T19:40:50Z" presentationTime="15135120"/>
            <Representation id="1_V_video_5322324134428436312" bandwidth="3500000" width="1280" height="720"/>
            <Representation id="1_V_video_16981495139092747609" bandwidth="2200000" width="960" height="540"/>
            <Representation id="1_V_video_1384718563016940751" bandwidth="1350000" codecs="avc1.64001F" width="704" height="396"/>
            <Representation id="1_V_video_4425970933904124207" bandwidth="850000" codecs="avc1.64001E" width="512" height="288"/>
            <Representation id="1_V_video_11952982975776937431" bandwidth="550000" codecs="avc1.640016" width="384" height="216"/>
            <Representation id="1_V_video_10673801877453424365" bandwidth="200000" codecs="avc1.640015" width="340" height="192"/>
        </AdaptationSet>
        <AdaptationSet id="2" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.5" lang="en">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <Label>audio</Label>
            <SegmentTemplate timescale="44100" presentationTimeOffset="7416208" media="QualityLevels($Bandwidth$)/Fragments(audio=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(audio=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="7417856" d="133120"/>
                    <S d="132096" r="1"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="7417856" type="0" wallClockTime="2020-01-07T19:40:50.037Z" presentationTime="7417856"/>
            <Representation id="5_A_audio_17504386117102112482" bandwidth="128000" audioSamplingRate="44100"/>
        </AdaptationSet>
    </Period>
    <Period start="PT4M19.509S" id="scte-35_0">
        <EventStream schemeIdUri="urn:scte:scte35:2014:xml+bin" value="scte35" timescale="10000000">
            <Event presentationTime="2595092444" duration="11011000" id="1002">
                <Signal xmlns="http://www.scte.org/schemas/35/2016">
                    <Binary>/DAlAAAAAAXdAP/wFAUAAAPqf+/+AWRhuP4AUmNjAAEBAQAA8g1eNw==</Binary>
                </Signal>
            </Event>
            <Event presentationTime="2606103444" id="1002">
                <Signal xmlns="http://www.scte.org/schemas/35/2016">
                    <Binary>/DAgAAAAAAXdAP/wDwUAAAPqf0/+AWXk0wABAQEAAGB86Fo=</Binary>
                </Signal>
            </Event>
        </EventStream>
        <AdaptationSet id="1" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.640020" maxWidth="1280" maxHeight="720" startWithSAP="1">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <SegmentTemplate timescale="90000" presentationTimeOffset="23355832" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="23355833" d="22522"/>
                    <S d="76577"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="23355833" type="0" wallClockTime="2020-01-07T19:42:21.341Z" presentationTime="23355833"/>
            <Representation id="1_V_video_5322324134428436312" bandwidth="3500000" width="1280" height="720"/>
            <Representation id="1_V_video_16981495139092747609" bandwidth="2200000" width="960" height="540"/>
            <Representation id="1_V_video_1384718563016940751" bandwidth="1350000" codecs="avc1.64001F" width="704" height="396"/>
            <Representation id="1_V_video_4425970933904124207" bandwidth="850000" codecs="avc1.64001E" width="512" height="288"/>
            <Representation id="1_V_video_11952982975776937431" bandwidth="550000" codecs="avc1.640016" width="384" height="216"/>
            <Representation id="1_V_video_10673801877453424365" bandwidth="200000" codecs="avc1.640015" width="340" height="192"/>
        </AdaptationSet>
        <AdaptationSet id="2" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.5" lang="en">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <Label>audio</Label>
            <SegmentTemplate timescale="44100" presentationTimeOffset="11444358" media="QualityLevels($Bandwidth$)/Fragments(audio=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(audio=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="11446272" d="49152"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="11446272" type="0" wallClockTime="2020-01-07T19:42:21.384Z" presentationTime="11446272"/>
            <Representation id="5_A_audio_17504386117102112482" bandwidth="128000" audioSamplingRate="44100"/>
        </AdaptationSet>
    </Period>
    <Period start="PT4M20.610S" id="main-content_1">
        <AdaptationSet id="1" group="1" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="video" mimeType="video/mp4" codecs="avc1.640020" maxWidth="1280" maxHeight="720" startWithSAP="1">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <SegmentTemplate timescale="90000" presentationTimeOffset="23454931" media="QualityLevels($Bandwidth$)/Fragments(video=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(video=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="23454932" d="58558"/>
                    <S d="4504"/>
                    <S d="130631"/>
                    <S d="135135" r="12"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="23454932" type="0" wallClockTime="2020-01-07T19:42:22.442Z" presentationTime="23454932"/>
            <Representation id="1_V_video_5322324134428436312" bandwidth="3500000" width="1280" height="720"/>
            <Representation id="1_V_video_16981495139092747609" bandwidth="2200000" width="960" height="540"/>
            <Representation id="1_V_video_1384718563016940751" bandwidth="1350000" codecs="avc1.64001F" width="704" height="396"/>
            <Representation id="1_V_video_4425970933904124207" bandwidth="850000" codecs="avc1.64001E" width="512" height="288"/>
            <Representation id="1_V_video_11952982975776937431" bandwidth="550000" codecs="avc1.640016" width="384" height="216"/>
            <Representation id="1_V_video_10673801877453424365" bandwidth="200000" codecs="avc1.640015" width="340" height="192"/>
        </AdaptationSet>
        <AdaptationSet id="2" group="5" profiles="ccff" bitstreamSwitching="false" segmentAlignment="true" contentType="audio" mimeType="audio/mp4" codecs="mp4a.40.5" lang="en">
            <InbandEventStream schemeIdUri="urn:mpeg:dash:event:2012" value="1"/>
            <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
            <Label>audio</Label>
            <SegmentTemplate timescale="44100" presentationTimeOffset="11492916" media="QualityLevels($Bandwidth$)/Fragments(audio=$Time$,format=mpd-time-csf)" initialization="QualityLevels($Bandwidth$)/Fragments(audio=i,format=mpd-time-csf)">
                <SegmentTimeline>
                    <S t="11495424" d="28672"/>
                    <S d="1024"/>
                    <S d="131072"/>
                    <S d="132096"/>
                    <S d="133120"/>
                    <S d="132096" r="1"/>
                    <S d="133120"/>
                </SegmentTimeline>
            </SegmentTemplate>
            <ProducerReferenceTime id="11495424" type="0" wallClockTime="2020-01-07T19:42:22.499Z" presentationTime="11495424"/>
            <Representation id="5_A_audio_17504386117102112482" bandwidth="128000" audioSamplingRate="44100"/>
        </AdaptationSet>
    </Period>
</MPD>

~~~

--->

### <a name="334-mpeg-dash-in-band-event-message-box-signaling"></a>3.3.4 Signalisation de zone de message d’événement dans la bande MPEG DASH

Un flux d’événement dans la bande nécessite que le MPD ait un élément InbandEventStream au niveau défini de l’adaptation.  Cet élément a un attribut schemeIdUri obligatoire et un attribut d’échelle de temps facultatif, qui s’affichent également dans la zone de message d’événement (« emsg »).  Les zones de message d’événement avec des identificateurs de schéma qui ne sont pas définis dans le MPD ne **DOIVENT PAS** être présentes.

Pour le transport intrabande [SCTE-35], les signaux **DOIVENT** utiliser le schemeId=« urn:scte:scte35:2013:bin ».
Les définitions normatives du transport des messages intrabandes [SCTE-35] sont définies dans la section 7.3.2 de la norme [SCTE-214-3] (transport des messages de signal SCTE 35).

Les détails suivants décrivent les valeurs spécifiques que le client doit attendre dans le « EMSG » en conformité avec la norme [SCTE-214-3] :

| **Nom du champ**          | **Type de champ**          | **Obligatoire ?** | **Description**                                                                                                                                                                                                                                                                                        |
| ----------------------- | ----------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| scheme_id_uri           | string                  | Obligatoire      | Identifie le schéma du message. Le schéma est défini sur la valeur de l’attribut Scheme dans la zone de manifeste du serveur en direct. La valeur **DOIT** être un URN identifiant le schéma de message. Pour les messages [SCTE-35], il **DOIT** s’agir de « urn:scte:scte35:2013:bin » en conformité avec la norme [SCTE-214-3]          |
| Valeur                   | string                  | Obligatoire      | Valeur de chaîne supplémentaire utilisée par les propriétaires du schéma pour personnaliser la sémantique du message. Afin de différencier plusieurs flux d’événements dont le schéma est identique, la valeur sera définie sur le nom du flux d’événements (trackName pour une réception lisse ou nom du message AMF pour une réception RTMP). |
| Échelle de temps               | Entier non signé 32 bits | Obligatoire      | Échelle de temps, en battements par seconde, des champs d’heures et de durée dans la zone « emsg ».                                                                                                                                                                                                            |
| Presentation_time_delta | Entier non signé 32 bits | Obligatoire      | Delta de temps de présentation multimédia entre l’heure de la présentation de l’événement et l’heure de présentation la plus précoce dans ce segment. L’heure et la durée de présentation **DOIVENT** être en phase avec les points d’accès de flux (SAP) de type 1 ou 2, comme définis dans [ISO-14496-12] l’annexe I.                                  |
| event_duration          | Entier non signé 32 bits | Obligatoire      | Durée de l’événement ou 0xFFFFFFFF pour indiquer une durée inconnue.                                                                                                                                                                                                                              |
| Id                      | Entier non signé 32 bits | Obligatoire      | Identifie cette instance du message. Les messages dont la sémantique est identique auront la même valeur. Si l’ID n’est pas spécifié lors de la réception du message, Azure Media Services génère un id unique.                                                                                        |
| Message_data            | tableau d’octets              | Obligatoire      | Message d'événement. Pour les messages [SCTE-35], leurs données sont le binaire splice_info_section() en conformité avec la norme [SCTE-214-3]                                                                                                                                                                        |


#### <a name="example-inbandevenstream-entity-for-adobe-simple-mode"></a>Exemple d’entité InBandEvenStream pour le mode simple Adobe
```xml

      <InbandEventStream schemeIdUri="urn:com:adobe:dpi:simple:2015" value="amssignal"/>
```

### <a name="335-dash-message-handling"></a>3.3.5 Gestion des messages DASH

Les événements sont signalés dans la bande, dans la zone « emsg », pour les pistes tant audio que vidéo.  La signalisation se produit pour toutes les demandes de segments pour lesquelles la valeur presentation_time_delta est inférieure ou égale à 15 secondes. 

Quand une fenêtre de présentation glissante est activée, les messages d’événement sont supprimés du MPD lorsque la somme du temps et la durée du message d’événement sont inférieures au temps des données multimédias dans le manifeste.  En d’autres termes, les messages d’événements sont supprimés du manifeste lorsque le temps multimédia auquel ils font référence sort de la fenêtre glissante de présentation.

## <a name="4-scte-35-ingest-implementation-guidance-for-encoder-vendors"></a>4. Guide d’implémentation de l’ingestion SCTE-35 pour les fournisseurs d’encodeurs

Les instructions suivantes sont des problèmes courants qui peuvent avoir un impact sur l’implémentation d’un fournisseur d’encodeurs de cette spécification.  Les instructions ci-dessous ont été collectées en fonction des commentaires de partenaires réels pour faciliter l’implémentation de cette spécification pour d’autres. 

Les messages [SCTE-35] sont reçus dans un format binaire à l’aide du schéma **« urn:scte:scte35:2013:bin »** pour la [réception MS-SSTR] et le type **« scte35 »** pour la réception RTMP. Pour faciliter la conversion d’une synchronisation [SCTE-35], qui est basée sur des horodatages de présentation de flux de transport MPEG-2 (PTS), un mappage entre PTS (pts_time + pts_adjustment de splice_time()) et la chronologie du média sont fournis par l’heure de présentation de l’événement (champ fragment_absolute_time pour la réception lisse et champ d’heure pour la réception RTMP). Le mappage est nécessaire, car la valeur PTS 33 bits change environ toutes les 26,5 heures.

La réception de diffusion en continu lisse [MS-SSTR-Ingest] **nécessite** que la zone de données multimédias (« mdat ») contienne la valeur **splice_info_section()** définie dans la norme [SCTE-35]. 

Pour la réception RTMP, l’attribut cue du message AMF est défini sur la valeur **splice_info_section()** codée en base64 définie dans la norme [SCTE-35].  


Lorsque les messages ont le format décrit ci-dessus, ils sont envoyés aux clients HLS, Lisse et DASH comme définis ci-dessus.  

Lors du test de votre implémentation avec la plateforme Azure Media Services, commencez par effectuer des tests avec un LiveEvent « pass-through » avant de passer au test sur un LiveEvent d’encodage.

---

## <a name="change-history"></a>Historique des modifications

| Date     | Modifications                                                                                                             |
| -------- | ------------------------------------------------------------------------------------------------------------------- |
| 02/07/19  | Révision de la réception RTMP, ajout de RTMP « onCuePoint » pour Elemental Live                                            |
| 22/08/19 | Mise à jour pour ajouter OnUserDataEvent à RTMP pour les métadonnées personnalisées                                                          |
| 08/01/20  | Correction de l’erreur de mode RTMP simple et RTMP SCTE35. La valeur « onCuePoint » a été remplacée par « onAdCue ». Tableau en mode simple mis à jour. |
| 4/08/20  | Suppression de la prise en charge de la balise DATERANGE pour correspondre à l’implémentation dans le service de production.    |

## <a name="next-steps"></a>Étapes suivantes
Afficher les parcours d’apprentissage de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]
