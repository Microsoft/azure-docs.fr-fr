---
title: Protocole de diffusion en continu lisse (MS-SSTR) Amendement pour HEVC-Azure
description: Cette spécification décrit le protocole et le format requis pour l’ingestion de streaming en direct basée sur le format MP4 avec HEVC pour Azure Media Services. Seules les modifications nécessaires à la distribution HEVC sont spécifiées dans cet article, à l’exception de la mention « Pas de modification » qui indique que le texte est copié pour clarification uniquement.
services: media-services
documentationcenter: ''
author: johndeu
manager: femila
editor: ''
ms.assetid: f27d85de-2cb8-4269-8eed-2efb566ca2c6
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/19/2019
ms.author: johndeu
ms.openlocfilehash: 6454bc863cb5fd628d581fff380c5ab61354f762
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87053041"
---
# <a name="smooth-streaming-protocol-ms-sstr-amendment-for-hevc"></a>Avenant relatif au protocole Smooth Streaming (MS-SSTR) pour HEVC 

## <a name="1-introduction"></a>1 Introduction 

Cet article fournit des modifications détaillées à appliquer à la spécification du protocole Smooth Streaming [MS-SSTR] afin d’activer Smooth Streaming de vidéo encodée HEVC. Dans cette spécification, nous ne présentons que les modifications nécessaires pour distribuer le codec vidéo HEVC. L’article suit le même schéma de numérotation que la spécification [MS-SSTR]. Les titres vides présentés dans l’article sont fournis afin d’orienter le lecteur vers leur position dans la spécification [MS-SSTR].  « Pas de modification » indique que le texte est copié à des fins de clarification uniquement.

L’article indique les exigences d’implémentation technique pour le signalement de codec vidéo HEVC (à l'aide de pistes de format « hev1 » ou « hvc1 ») dans un manifeste Smooth Streaming. Les références normatives sont mises à jour pour référencer les normes MPEG standards actuelles qui incluent HEVC, Chiffrement commun de HEVC, et les noms de zones du format de fichier multimédia de base ISO ont été mis à jour pour être cohérents avec les dernières spécifications. 

La spécification du protocole Smooth Streaming [MS-SSTR] référencée décrit le format de câble utilisé pour distribuer les médias numériques en direct et à la demande, tels qu’audio et vidéo, des manières suivantes : d’un encodeur à un serveur web, d’un serveur à un autre serveur et d’un serveur à un client HTTP.
L’utilisation d’une structure de distribution de données basée sur MPEG-4 ([[MPEG4-RA])](https://go.microsoft.com/fwlink/?LinkId=327787) sur HTTP permet une commutation transparente quasiment en temps réel entre différents niveaux de qualité de contenu multimédia compressé. Le résultat est une expérience de lecture constante pour l’utilisateur final du client HTTP, même si le réseau et les conditions de rendu vidéo changent pour l’ordinateur ou l’appareil client.

## <a name="11-glossary"></a>1.1 Glossaire 

Les termes suivants sont définis dans *[MS-GLOS]* :

>   **identificateur global unique (GUID) identificateur unique universel (UUID)**

Les termes suivants sont spécifiques à ce document :

>  **temps de composition :** durée de présentation d’un exemple au client, telle que définie dans [[ISO/CEI-14496-12].](https://go.microsoft.com/fwlink/?LinkId=183695)
> 
>   **CENC** : chiffrement commun, tel que défini dans [ISO/CEI 23001-7] Deuxième édition.
> 
>   **temps de décodage :** le temps nécessaire pour qu’un échantillon soit décodé sur le client, tel que défini dans [[ISO/IEC 14496-12:2008]](https://go.microsoft.com/fwlink/?LinkId=183695).

**fragment :** unité téléchargeable indépendamment de **média** qui comprend un ou plusieurs **exemples**.

>   **HEVC :** codage vidéo haute efficacité, tel que défini dans [ISO/CEI 23008-2]
> 
>   **manifeste :** métadonnées relatives à la **présentation** qui permet à un client d’effectuer des requêtes de **média**. **média :** données audio, vidéo et texte compressées utilisées par le client pour lire une **présentation**. **format de média :** format bien défini de représentation audio ou vidéo en tant qu’**exemple** compressé.
> 
>   **présentation :** ensemble de tous les **flux** et des métadonnées associées nécessaires pour lire un film. **requête :** Un message  HTTP envoyé du client au serveur, tel que défini dans [[RFC2616]](https://go.microsoft.com/fwlink/?LinkId=90372) **réponse :** message HTTP envoyé par le serveur au client, tel que défini dans [[RFC2616]](https://go.microsoft.com/fwlink/?LinkId=90372)
> 
>   **exemple :** plus petite unité fondamentale (par exemple, une image) dans laquelle des **média** sont stockés et traités.
> 
>   **PEUT, DOIT, NE DOIT PAS :** Ces termes (en majuscules) sont utilisés comme décrit dans [[RFC2119]](https://go.microsoft.com/fwlink/?LinkId=90317) Toutes les instructions de comportement facultatif utilisent PEUT, DOIT ou NE DOIT PAS.

## <a name="12-references"></a>1.2 Références

>   Les références à la documentation Microsoft Open Specifications n’incluent pas d’année de publication car les liens dirigent vers la dernière version des documents, qui sont mis à jour fréquemment. Les références à d’autres documents incluent l’année de publication (si disponible).

### <a name="121-normative-references"></a>1.2.1 Références normatives 

>  [MS-SSTR] Protocole de diffusion en continu lisse *v20140502*[https://msdn.microsoft.com/library/ff469518.aspx](/openspecs/windows_protocols/ms-sstr/8383f27f-7efe-4c60-832a-387274457251)
> 
>   [ISO/CEI 14496-12] Organisation internationale de normalisation, « Technologies de l’information -- Codage des objets audiovisuels -- Partie 12: Format ISO de base pour les fichiers médias », ISO/CEI 14496-12:2014, Édition 4, Plus Corrigendum 1, Avenants 1 & 2.
>   <https://standards.iso.org/ittf/PubliclyAvailableStandards/c061988_ISO_IEC_14496-12_2012.zip>
> 
>   [ISO/CEI 14496-15] Organisation internationale de normalisation, « Technologies de l’information -- Codage des objets audiovisuels -- Partie 15: Transport de vidéo structuré en unités NAL au format ISO de base pour les fichiers médias », ISO 14496-15:2015, Édition 3.
>   <https://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=65216>
> 
>   [ISO/CEI 23008-2] Technologies de l’information -- High efficiency coding and media   delivery in heterogeneous environments -- Part 2: High efficiency video   coding: 2013 ou édition plus récente   <https://standards.iso.org/ittf/PubliclyAvailableStandards/c035424_ISO_IEC_23008-2_2013.zip>
> 
>   [ISO/IEC 23001-7] Technologies de l’information -- Technologies des systèmes MPEG -- Partie 7: Cryptage commun des fichiers au format de fichier de médias de la base ISO, édition CENC 2:2015 <https://www.iso.org/iso/catalogue_detail.htm?csnumber=65271>
> 
>   [RFC-6381] IETF RFC-6381, “The 'Codecs' and 'Profiles' Parameters for   "Bucket" Media Types” <https://tools.ietf.org/html/rfc6381>
> 
>   [MPEG4-RA] L’autorité d’inscription MP4, « MP4REG », [http://www.mp4ra.org](https://go.microsoft.com/fwlink/?LinkId=327787)
> 
>   [RFC2119] Bradner, S., « Key words for use in RFCs to Indicate Requirement   Levels », BCP 14, RFC 2119, mars 1997,   [https://www.rfc-editor.org/rfc/rfc2119.txt](https://go.microsoft.com/fwlink/?LinkId=90317)

### <a name="122-informative-references"></a>1.2.2 Références informatives 

>   [MS-GLOS] Microsoft Corporation, « *Glossaire principal des protocoles Windows* ».
> 
>   [RFC3548] Josefsson, S., Ed., « The Base16, Base32, and Base64 Data   Encodings », RFC 3548, juillet 2003, [https://www.ietf.org/rfc/rfc3548.txt](https://go.microsoft.com/fwlink/?LinkId=90432)
> 
>   [RFC5234] Crocker, D., Ed. et Overell, P., « Augmented BNF for Syntax   Specifications: ABNF », STD 68, RFC 5234, janvier 2008,   [https://www.rfc-editor.org/rfc/rfc5234.txt](https://go.microsoft.com/fwlink/?LinkId=123096)


## <a name="13-overview"></a>1.3 Vue d’ensemble 

>   Seules les modifications apportées à la spécification Smooth Streaming nécessaires pour la distribution de HEVC sont spécifiées ci-dessous. Les en-têtes de sections inchangées sont répertoriés pour conserver l’emplacement dans la spécification Smooth Streaming [MS-SSTR] référencée.

## <a name="14-relationship-to-other-protocols"></a>1.4 Relation aux autres protocoles 

## <a name="15-prerequisitespreconditions"></a>1.5 Configuration requise/Conditions préalables 

## <a name="16-applicability-statement"></a>1.6 Instruction d’applicabilité 

## <a name="17-versioning-and-capability-negotiation"></a>1.7 Contrôle de version et négociation de capacité 

## <a name="18-vendor-extensible-fields"></a>1.8 Champs extensibles de fournisseur 

>   La méthode suivante DOIT être utilisée pour identifier les flux de données utilisant le format vidéo HEVC :
> 
>   * **Codes descriptifs personnalisés des formats de média :** cette fonctionnalité est fournie par le champ **FourCC**, comme spécifié dans la section *2.2.2.5*.
>   Les implémenteurs peuvent vérifier l’absence de conflit des extensions en inscrivant les codes d’extension à MPEG4-RA, comme spécifié dans [[ISO/CEI-14496-12]](https://go.microsoft.com/fwlink/?LinkId=183695)

## <a name="19-standards-assignments"></a>1.9 Affectations de normes 

## <a name="2-messages"></a>2 Messages 

## <a name="21-transport"></a>2.1 Transport

## <a name="22-message-syntax"></a>2.2 Syntaxe de message 

### <a name="221-manifest-request"></a>2.2.1 Requête du manifeste 

### <a name="222-manifest-response"></a>2.2.2 Réponse du manifeste 

#### <a name="2221-smoothstreamingmedia"></a>2.2.2.1 SmoothStreamingMedia 

>   **MinorVersion (variable) :** version mineure du message de réponse du manifeste. DOIT être défini sur 2. (Pas de modification)
> 
>   **TimeScale (variable) :** échelle de temps de l’attribut de durée, spécifié en nombre d’incréments d’une seconde. La valeur par défaut est
> 1. (Pas de modification)
> 
>    La valeur recommandée est de 90 000 pour représenter la durée exacte des trames vidéo et fragments contenant des fractions de trame vidéo (par exemple, 30/1,001 Hz).

#### <a name="2222-protectionelement"></a>2.2.2.2 ProtectionElement 

ProtectionElement DOIT être présent lorsque le chiffrement commun (CENC) est appliqué aux flux vidéo ou audio. Les flux chiffrés HEVC DOIVENT être conformes au chiffrement commun 2ème édition [ISO/CEI 23001-7]. Seules les tranches de données en unités VCL NAL DOIVENT être chiffrées.

#### <a name="2223-streamelement"></a>2.2.2.3 StreamElement 

>   **StreamTimeScale (variable) :** échelle de temps de durée et valeurs de temps dans ce flux, spécifiés en nombre d’incréments d’une seconde. La valeur 90000 est recommandée pour les flux HEVC. Une valeur correspondant à la fréquence d’échantillonnage de forme d’onde (par exemple, 48000 ou 44100) est recommandée pour les flux audio.

##### <a name="22231-streamprotectionelement"></a>2.2.2.3.1 StreamProtectionElement

#### <a name="2224-urlpattern"></a>2.2.2.4 UrlPattern 

#### <a name="225-trackelement"></a>2.2.5 TrackElement 

>   **FourCC (variable) :** code de quatre caractères qui identifie le format de média utilisé pour chaque exemple. La gamme de valeurs suivantes est réservée avec les significations sémantiques suivantes :
> 
> * « hev1 » : les exemples vidéo de cette piste utilisent la vidéo HEVC, à l’aide du format de description d’exemple « hev1 » spécifié dans [ISO/CEI-14496-15].
>
> * « hvc1 »: Les exemples vidéo de cette piste utilisent la vidéo HEVC, à l’aide du format de description d’exemple « hvc1 » indiqué dans [ISO/CEI-14496-15].
> 
>   **CodecPrivateData (variable):** données indiquant des paramètres spécifiques au format de média et communes à tous les exemples de la piste, représentées sous forme de chaîne d’octets codés en hexadécimal. Le format et la signification sémantique de la séquence d’octets varie en fonction de la valeur du champ **FourCC** comme suit :
> 
>   * Lorsqu’un TrackElement décrit une vidéo HEVC, le champ **FourCC** DOIT être défini sur **« hev1 »** ou **« hvc1 »**
> 
>   le champ **CodecPrivateData** DOIT contenir une représentation de chaîne codée en hexadécimal de la séquence d’octets suivante, spécifiée dans ABNF [[RFC5234] :](https://go.microsoft.com/fwlink/?LinkId=123096) (pas de modification à partir de MS-SSTR)
> 
>   * %x00 %x00 %x00 %x01 SPSField %x00 %x00 %x00 %x01 PPSField
> 
>   * SPSField contient le jeu de paramètres de séquence (SPS).
> 
>   * PPSField contient le jeu de paramètres de tranche (SPS).
> 
>   Remarque : le jeu de paramètres vidéo (VPS) n’est pas contenu dans CodecPrivateData, mais doit être contenu dans l’en-tête des fichiers stockés dans la zone « hvcC ». Les systèmes utilisant le protocole Smooth Streaming doivent signaler les paramètres de décodage supplémentaires (par exemple, HEVC Tier) à l’aide de l’attribut personnalisé « codecs ».

##### <a name="22251-customattributeselement"></a>2.2.2.5.1 CustomAttributesElement 

#### <a name="226-streamfragmentelement"></a>2.2.6 StreamFragmentElement 

>   Le champ **SmoothStreamingMedia’s MajorVersion** DOIT être défini sur 2, et le champ **MinorVersion** DOIT être défini sur 2. (Pas de modification)

##### <a name="22261-trackfragmentelement"></a>2.2.2.6.1 TrackFragmentElement 

### <a name="223-fragment-request"></a>2.2.3 Requête du fragment 

>   **Remarque** : Le format de média par défaut requis pour **MinorVersion** 2 et «  hev1 » ou « hvc1 »   est le format de fichier multimédia de base ISO « iso8 » , indiqué dans [ISO/CEI 14496-12]    Format de fichier multimédia de base ISO quatrième édition et [ISO/CEI 23001-7] Chiffrement    commun deuxième édition.

### <a name="224-fragment-response"></a>2.2.4 Réponse du fragment 

#### <a name="2241-moofbox"></a>2.2.4.1 MoofBox 

#### <a name="2242-mfhdbox"></a>2.2.4.2 MfhdBox 

#### <a name="2243-trafbox"></a>2.2.4.3 TrafBox 

#### <a name="2244-tfxdbox"></a>2.2.4.4 TfxdBox 

>   **TfxdBox** est déconseillé et sa fonction est remplacée par la zone Track Fragment Decode Time (« tfdt ») spécifiée dans [ISO/CEI 14496-12] section 8.8.12.
> 
>   **Remarque** : un client peut calculer la durée d’un fragment en additionnant les durées d’exemples répertoriées dans la zone Track Run (« trun ») ou en multipliant le nombre de durées d’exemples par la durée d’exemple par défaut. baseMediaDecodeTime dans « tfdt » plus la durée de fragment est égal au paramètre de temps d’URL du fragment suivant.
> 
>   Une zone Producer Reference Time (« prft ») doit être insérée avant une zone Movie   Fragment (« moof ») en fonction des besoins, pour indiquer l’heure UTC correspondant à la zone Track Fragment Decode Time du premier exemple référencé par la zone Movie Fragment, comme spécifié dans [ISO/CEI 14496-12] section 8.16.5.

#### <a name="2245-tfrfbox"></a>2.2.4.5 TfrfBox 

>   **TfrfBox** est déconseillé et sa fonction est remplacée par la zone Track   Fragment Decode Time (« tfdt ») spécifiée dans [ISO/CEI 14496-12] section 8.8.12.
> 
>   **Remarque** : un client peut calculer la durée d’un fragment en additionnant les durées d’exemples répertoriées dans la zone Track Run (« trun ») ou en multipliant le nombre de durées d’exemples par la durée d’exemple par défaut. baseMediaDecodeTime dans « tfdt » plus la durée de fragment est égal au paramètre de temps d’URL du fragment suivant. Anticipez les adresses déconseillées du fait qu’elles retardent le streaming en direct.

#### <a name="2246-tfhdbox"></a>2.2.4.6 TfhdBox 

>   **TfhdBox** et les champs associés encapsulent les valeurs par défaut par exemples de métadonnées dans le fragment. La syntaxe du champ **TfhdBox** est un sous-ensemble strict de la syntaxe de la zone Track Fragment Header défini dans [[ISO/CEI-14496-12]](https://go.microsoft.com/fwlink/?LinkId=183695) section 8.8.7.
> 
>   **BaseDataOffset (8 octets) :** décalage, en octets, entre le début du champ **MdatBox** et le champ d’exemple dans le champ **MdatBox**. Pour signaler cette restriction, vous devez définir l’indicateur default-base-is-moof (0x020000).

#### <a name="2247-trunbox"></a>2.2.4.7 TrunBox 

>   **TrunBox** et les champs associés encapsulent par exemples de métadonnées pour le fragment demandé. La syntaxe de **TrunBox** est un sous-ensemble strict de la zone Version 1 Track Fragment Run défini dans [[ISO/CEI-14496-](https://go.microsoft.com/fwlink/?LinkId=183695)*12]* section 8.8.8.
> 
>   **SampleCompositionTimeOffset (4 octets) :** décalage de temps de composition d’exemple de chaque exemple ajusté afin que le temps de présentation du premier exemple présenté dans le fragment soit égal au temps de décodage du premier exemple décodé. Des décalages de composition d’exemple vidéo négatifs DOIVENT être utilisés,
> 
>   comme défini dans [[ISO/CEI-14496-12].](https://go.microsoft.com/fwlink/?LinkId=183695)
> 
>   Remarque : cela permet d’éviter une erreur de synchronisation vidéo due à un retard vidéo audio égal au délai de suppression de mémoire tampon d’image décodée le plus long, et préserve le minutage de la présentation entre d’autres fragments pouvant avoir des délais de suppression différents.
> 
>   La syntaxe des champs définis dans cette section, spécifiée dans ABNF [[RFC5234]](https://go.microsoft.com/fwlink/?LinkId=123096) est la même, à l’exception de :
> 
>   SampleCompositionTimeOffset = SIGNED_INT32

#### <a name="2248-mdatbox"></a>2.2.4.8 MdatBox 

#### <a name="2249-fragment-response-common-fields"></a>2.2.4.9 Champs communs de réponse du fragment 

### <a name="225-sparse-stream-pointer"></a>2.2.5 Pointeur de flux partiellement alloué 

### <a name="226-fragment-not-yet-available"></a>2.2.6 Fragment pas encore disponible 

### <a name="227-live-ingest"></a>2.2.7 Ingestion en direct 

#### <a name="2271-filetype"></a>2.2.7.1 FileType 

>   **FileType (variable)** : spécifie le sous-type et l’utilisation prévue du fichier MPEG-4 ([[MPEG4-RA])](https://go.microsoft.com/fwlink/?LinkId=327787), et des attributs de niveau supérieur.
> 
>   **MajorBrand (variable) :** marque principale du fichier multimédia. DOIT être défini sur « isml ».
> 
>   **MinorVersion (variable) :** version mineure du fichier multimédia. DOIT être défini sur 1.
> 
>   **CompatibleBrands (variable) :** spécifie les marques de MPEG-4 prises en charge.
>   DOIT inclure « ccff » et « iso8 ».
> 
>   La syntaxe des champs définis dans cette section, spécifiée dans ABNF [[RFC5234]](https://go.microsoft.com/fwlink/?LinkId=123096) est la suivante :

```properties
FileType = MajorBrand MinorVersion CompatibleBrands
MajorBrand = STRING_UINT32
MinorVersion = STRING_UINT32
CompatibleBrands = "ccff" "iso8" 0\*(STRING_UINT32)
```

**Remarque** : les marques de compatibilité « ccff » et « iso8 » indiquent que les fragments sont conformes au « Format de fichier conteneur commun » et au chiffrement commun [ISO/CEI 23001-7] et au format de fichier multimédia de base ISO 4 [ISO/CEI 14496-12].

#### <a name="2272-streammanifestbox"></a>2.2.7.2 StreamManifestBox 

##### <a name="22721-streamsmil"></a>2.2.7.2.1 StreamSMIL 

#### <a name="2273-liveservermanifestbox"></a>2.2.7.3 LiveServerManifestBox 

##### <a name="22731-livesmil"></a>2.2.7.3.1 LiveSMIL 

#### <a name="2274-moovbox"></a>2.2.7.4 MoovBox 

#### <a name="2275-fragment"></a>2.2.7.5 Fragment 

##### <a name="22751-track-fragment-extended-header"></a>2.2.7.5.1 En-tête étendu du fragment de piste 

### <a name="228-server-to-server-ingest"></a>2.2.8 Ingestion serveur à serveur 

## <a name="3-protocol-details"></a>3 Détails du protocole 


## <a name="31-client-details"></a>3.1 Détails du client 

### <a name="311-abstract-data-model"></a>3.1.1 Modèle de données abstraites 

#### <a name="3111-presentation-description"></a>3.1.1.1 Description de la présentation 

>   L’élément de données Description de la présentation encapsule toutes les métadonnées de la présentation.
> 
>   Métadonnées de la présentation : jeu de métadonnées communes à tous les flux de la présentation. Les métadonnées de la présentation comprennent les champs suivants, spécifiés dans la section *2.2.2.1* :
> 
> * **MajorVersion**
> * **MinorVersion**
> * **TimeScale**
> * **Durée**
> * **IsLive**
> * **LookaheadCount**
> * **DVRWindowLength**
> 
>   Les présentations contenant le jeu de flux HEVC DOIVENT définir :

```properties
MajorVersion = 2
MinorVersion = 2
```

>   LookaheadCount = 0 (Note: outils déconseillés)
> 
>   Les présentations DOIVENT également définir :

```properties
TimeScale = 90000
```

>   Collection de flux : collection d’éléments de données de description de flux, comme spécifié dans la section *3.1.1.1.2*.
> 
>   Description de la protection : collection d’éléments de données de description des métadonnées du système de protection, comme spécifié dans la section *3.1.1.1.1*.

##### <a name="31111-protection-system-metadata-description"></a>3.1.1.1.1 Description des métadonnées du système de protection 

>   L’élément de données de description des métadonnées du système de protection encapsule les métadonnées spécifiques à un système de protection du contenu. (Pas de modification)
> 
>   Description de l’en-tête de protection : métadonnées de protection du contenu appartenant à un système de protection du contenu. La description de l’en-tête de protection comprend les champs suivants, spécifiés dans la section *2.2.2.2* :
> 
>   * **SystemID**
>   * **ProtectionHeaderContent**

##### <a name="31112-stream-description"></a>3.1.1.1.2 Description du flux 

###### <a name="311121-track-description"></a>3.1.1.1.2.1 Description de la piste 

###### <a name="3111211-custom-attribute-description"></a>3.1.1.1.2.1.1 Description de l’attribut personnalisé 

##### <a name="3113-fragment-reference-description"></a>3.1.1.3 Description de la référence de fragment 

###### <a name="31131-track-specific-fragment-reference-description"></a>3.1.1.3.1 Description de la référence de fragment spécifique à la piste 

#### <a name="3112-fragment-description"></a>3.1.1.2 Description du fragment 

##### <a name="31121-sample-description"></a>3.1.1.2.1 Description de l’exemple 

### <a name="312-timers"></a>3.1.2 Minuteurs 

### <a name="313-initialization"></a>3.1.3 Initialisation 

### <a name="314-higher-layer-triggered-events"></a>3.1.4 Événements déclenchés de niveau supérieur 

#### <a name="3141-open-presentation"></a>3.1.4.1 Ouvrir la présentation 

#### <a name="3142-get-fragment"></a>3.1.4.2 Obtenir un fragment 

#### <a name="3143-close-presentation"></a>3.1.4.3 Fermer la présentation 

### <a name="315-processing-events-and-sequencing-rules"></a>3.1.5 Traitement des événements et règles de séquencement 

#### <a name="3151-manifest-request-and-manifest-response"></a>3.1.5.1 Requête du manifeste et réponse du manifeste 

#### <a name="3152-fragment-request-and-fragment-response"></a>3.1.5.2 Requête du fragment et réponse du fragment

## <a name="32-server-details"></a>3.2 Détails du serveur

## <a name="33-live-encoder-details"></a>3.3 Détails de l’encodeur en direct 

## <a name="4-protocol-examples"></a>4 Exemples de protocoles 

## <a name="5-security"></a>5 Sécurité 

## <a name="51-security-considerations-for-implementers"></a>5.1 Considérations relatives à la sécurité pour les implémenteurs

>   Si le contenu transporté à l’aide de ce protocole a une valeur commerciale élevée, un système de protection du contenu doit être utilisé pour empêcher toute utilisation non autorisée du contenu. **ProtectionElement** peut être utilisé pour transporter les métadonnées liées à l’utilisation d’un système de protection du contenu. Le contenu audio et vidéo protégé DOIT être chiffré tel que spécifié par Chiffrement commun MPEG Deuxième édition : 2015 [ISO/IEC 23001-7].
> 
>   **Remarque** : pour la vidéo HEVC, seules les données de tranche de VCL NAL sont chiffrées. Les en-têtes de tranche et autres NAL sont accessibles aux applications de présentation avant le déchiffrement. dans un chemin d’accès vidéo sécurisé, les informations chiffrées ne sont pas disponibles pour les applications de présentation.

## <a name="52-index-of-security-parameters"></a>5.2 Index de paramètres de sécurité 


| **Paramètre de sécurité**  | **Section**         |
|-------------------------|---------------------|
| ProtectionElement       | *2.2.2.2*           |
| Zones de chiffrement commun | *[ISO/CEI 23001-7]* |

## <a name="53-common-encryption-boxes"></a>5.3 Zones de chiffrement commun

Les zones suivantes peuvent être présentes dans les réponses du fragment lorsque le chiffrement commun est appliqué, et sont spécifiées dans [ISO/CEI 23001-7] ou [ISO/CEI 14496-12] :

1.  Zone d’en-tête spécifique au système de protection (« pssh »)

2.  Zone de chiffrement d’exemple (« senc »)

3.  Zone de décalage d’informations auxiliaires d’exemple (« saio »)

4.  Zone de taille d’informations auxiliaires d’exemple (« saiz »)

5.  Zone de description du groupe d’exemples (« sgpd »)

6.  Zone d’exemple à groupe (« sbgp »)

---

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
