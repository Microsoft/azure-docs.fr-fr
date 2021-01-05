---
title: Licence du kit de portage du client Microsoft&reg; Smooth Streaming
description: En savoir plus sur la licence du kit de portage du client Microsoft&reg; Smooth Streaming.
services: media-services
documentationcenter: ''
author: xpouyat
manager: femila
editor: ''
ms.assetid: e3b488e7-8428-4c10-a072-eb3af46c82ad
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/21/2019
ms.author: xpouyat
ms.openlocfilehash: c82fb9764b29119092bf313b2c10f00c983e8c03
ms.sourcegitcommit: 19dce034650c654b656f44aab44de0c7a8bd7efe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/04/2020
ms.locfileid: "91710768"
---
# <a name="licensing-microsoftreg-smooth-streaming-client-porting-kit"></a>Licence du kit de portage du client Microsoft&reg; Smooth Streaming

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
## <a name="overview"></a>Vue d’ensemble
Le kit de portage client Smooth Streaming Microsoft (**SSPK** en abrégé) est une implémentation du client Smooth Streaming optimisée pour aider les fabricants de périphériques intégrés, les opérateurs mobiles et du câble, les fournisseurs de services de contenu, les fabricants de combinés, les éditeurs de logiciels indépendants (ISV) et les fournisseurs de solutions à créer des produits et des services pour diffuser du contenu adaptatif dans un format Smooth Streaming. SSPK est une implémentation Smooth Streaming cliente indépendante de l’appareil et de la plateforme pouvant être transférée par le titulaire de la licence vers n’importe quel appareil ou plateforme. 

Vous trouverez ci-après une architecture de haut niveau, et le package du kit de portage Smooth Streaming IIS est l’implémentation de client Smooth Streaming fournie par Microsoft. Il inclut la logique de base pour une lecture de contenu Smooth Streaming. Ce contenu est ensuite transféré par des partenaires pour un appareil ou une plateforme spécifique, grâce à l’implémentation des interfaces appropriées. 

![SSPK](./media/media-services-sspk/sspk-arch.png)

## <a name="description"></a>Description
SSPK est concédé sous licence selon des conditions représentant une excellente valeur pour l’entreprise. La licence SSPK offre au secteur :

* Source de kit de portage Smooth Streaming en C++ 
  * implémente la fonctionnalité Client Smooth Streaming
  * ajoute une analyse de format, heuristique, une logique de mise en mémoire tampon, etc.
* API d’application joueur 
  * interfaces de programmation pour une interaction avec une application de lecteur multimédia
* Interface de couche d’abstraction de plateforme (PAL) 
  * interfaces de programmation pour l‘interaction avec le système d‘exploitation (threads, sockets)
* Interface de couche d’abstraction matérielle 
  * interfaces de programmation pour l‘interaction avec les décodeurs A/V matériels (décodage, rendu)
* Interface de gestion de droits numériques (Digital Rights Management - DRM) 
  * interfaces de programmation pour la gestion des DRM via la couche Abstraction DRM (DAL)
  * Le kit de portage PlayReady Microsoft est fourni séparément, mais s’intègre via cette interface. Pour plus d‘informations sur les licences pour appareil Microsoft PlayReady cliquez [ici](https://www.microsoft.com/playready/licensing/device_technology.mspx#pddipdl).
* Échantillons de l’implémentation 
  * exemple d‘implémentation PAL pour Linux
  * exemple d‘implémentation HAL pour GStreamer

## <a name="licensing-options"></a>Options de licence
Le kit de portage Smooth Streaming client est accessible aux détenteurs de licences sous forme de deux contrats de licence distincts : l’un correspondant au développement des produits intermédiaires Smooth Streaming client, et l’autre pour la distribution de produits Smooth Streaming finaux pour les clients finaux.

* Pour les fabricants de puces, les intégrateurs système ou les fournisseurs de logiciels indépendants (ISV) qui ont besoin d‘un kit de portage de code source pour développer des produits intermédiaires, un kit de portage client Smooth Streaming Microsoft, **Licence produit intermédiaire** doit être exécutée.
* Pour les fabricants de périphériques ou les éditeurs de logiciels qui exigent des droits de distribution pour les produits de Smooth Streaming pour client final, c’est le kit de portage client Smooth Streaming **licence du produit Final** qui doit être exécuté.

### <a name="microsoft-smooth-streaming-client-porting-kit-interim-product-license"></a>Licence de produit intermédiaire pour le kit Microsoft Smooth Streaming Client Porting
Sous cette licence, Microsoft propose un kit de portage Smooth Streaming client et les droits de propriété intellectuelle nécessaires pour développer et distribuer les produits intermédiaires de client de Smooth Streaming pour les autres titulaires de licences de kit de portage Smooth Streaming client les produits finaux client de Smooth Streaming.

#### <a name="fee-structure"></a>Structure des frais
Des frais de licence de 50 000 dollars US payables en une fois offrent un accès au kit de portage Smooth Streaming client. 

### <a name="microsoft-smooth-streaming-client-porting-kit-final-product-license"></a>Licence de produit final pour le kit de portage Smooth Streaming client
Avec cette licence, Microsoft offre les droits de propriété intellectuelle nécessaires pour recevoir les produits intermédiaires de client de Smooth Streaming de la part d’autres titulaires de licences de kit de portage Smooth Streaming client et distribuer des produits Smooth Streaming à la marque de la société en continu lisse pour les clients finaux.

#### <a name="fee-structure"></a>Structure des frais
Le produit client final de Smooth Streaming est proposé selon un modèle soumis à redevance :

* 0,10 $ par implémentation de périphérique expédiés
* La redevance est limitée à 50 000 dollars chaque année
* Pas de redevance pour les 10 000 premières implémentations de périphérique chaque année 

## <a name="licensing-procedure-and-sspk-access"></a>Procédure d’achat de licence et accès à SSPK
Veuillez envoyer un e-mail à [sspkinfo@microsoft.com](mailto:sspkinfo@microsoft.com) pour toute requête relative aux licences.

Le portail de distribution SSPK Distribution est accessible aux détenteurs de licences intermédiaires enregistrés.

Les titulaires de licence SSPK intermédiaire ou finale peuvent soumettre des questions techniques à [smoothpk@microsoft.com](mailto:smoothpk@microsoft.com).

## <a name="microsoft-smooth-streaming-client-interim-product-agreement-licensees"></a>Les titulaires de licence d’accord de produit intermédiaire de client de diffusion lisse Microsoft

* Adroit Business Solutions, Inc
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* Albis Technologies Ltd.
* Alticast Corporation
* Amazon Digital Services, Inc.
* Arion Technology, Inc.
* AVC Multimedia Software Co., Ltd.
* Cavium, Inc.
* EchoStar Purchasing Corporation
* Enseo, Inc.
* Fluendo S.A.
* Guangzhou Shikun Electronics., Ltd.
* HANDAN BroadInfoCom Co., Ltd.
* Infomir GMBH
* Irdeto USA Inc.
* Liberty Global Services BV
* MediaTek Inc.
* MStar Co, Ltd
* Nintendo Co., Ltd.
* OpenTV, Inc.
* Saffron Digital Limited
* Sichuan Changhong Electric Co., Ltd
* SoftAtHome
* Sony Corporation
* Tatung Technology Inc.
* Top Victory Investments, Ltd.
* Vestel Elektronik Sanayi ve Ticaret A.S.
* VisualOn, Inc.
* ZTE Corporation

## <a name="microsoft-smooth-streaming-client-final-product-agreement-licensees"></a>Titulaires de licence d’accord de produit final de client de diffusion lisse Microsoft
* Advanced Digital Broadcast SA
* AirTies Kablosuz Iletism Sanayive Dis Ticaret A.S.
* AmTRAN Technology Co., Ltd 
* Arcadyan Technology Corporation
* Arcelik A.S
* Compal Electronics, Inc.
* EXPRESS LUCK TECHNOLOGY LIMITED
* Fluendo S.A.
* FUNAI ELECTRIC CO., LTD
* Hisense International Co., Ltd. 
* HKC Corporation Limited
* Hong Kong Konka Ltd
* Innolux Corporation
* Innopia Technologies, Inc
* K-Tronics (Suzhou) Technology Co., Ltd. 
* Kaonmedia Co., Ltd.
* KDDI Corporation
* Mega Fame Electronics Co. Limited
* MIRC Electronics Limited
* Nintendo Co., Ltd.
* ONEPLUS ELECTRONICS (SHENZHEN) CO., LTD.
* Panasonic Corporation
* Qingdao Haier Optronics Co., Ltd.
* Shenzhen ATEKO PHOTO Electricity Co.,Ltd.
* Shenzhen Chuangwei-RGB Electronics Co.,Ltd.
* Shenzhen Jiuzhou Electric Co., Ltd
* Shenzhen KTC Technology Co., Ltd. 
* Shenzhen Maxmade Technology Co., Ltd
* Shenzhen MTC Co., Ltd
* Shenzhen Skyworth Digital Technology Co., Ltd
* Sichuan Changhong Electric Co., Ltd.
* Skardin Industrial Corp.
* Sky Deutschland Fernsehen GmbH & Co. KG
* SMARDTV GLOBAL SAS
* SoftAtHome
* Sony Corporation
* Technicolor Delivery Technologies, SAS
* Top Victory Investments, Ltd.
* UMC Poland sp. z .o.o.
* ZTE Corporation

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

