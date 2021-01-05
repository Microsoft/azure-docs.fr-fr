---
title: Partenaires Azure Virtual WAN et emplacements | Microsoft Docs
description: Cet article contient une liste des localisations des partenaires et hubs Azure Virtual WAN.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to find a Virtual WAN partner
ms.openlocfilehash: 188bc14de502640ae05f1d1eb55330d86cee50b4
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621477"
---
# <a name="virtual-wan-partners-and-virtual-hub-locations"></a>Localisation des partenaires et hub virtuel Virtual WAN

Cet article fournit des informations sur les partenaires et les régions Virtual WAN prises en charge pour la connectivité dans le Hub virtuel.

Un WAN virtuel Azure est un service de mise en réseau qui offre une connectivité de branche à branche optimisée et automatisée via Azure. Le WAN virtuel vous permet de vous connecter et de configurer des appareils de branche pour communiquer avec Azure. Cela peut se faire manuellement ou à l’aide d’appareils d’un fournisseur via un WAN virtuel partenaire. L’utilisation des appareils de partenaires permet une utilisation facile, une simplification de la connectivité et une gestion de la configuration.

La connectivité est établie de manière automatisée de l’appareil en local vers le Hub virtuel. Un hub virtuel est un réseau virtuel géré par Microsoft. Le hub contient différents points de terminaison de service pour activer la connectivité à partir de votre réseau local (vpnsite). Vous ne pouvez disposer que d’un seul hub par région.

## <a name="branch-ipsec-connectivity-automation-from-partners"></a><a name="automation"></a>Automatisation de la connectivité IPSec de filiale à partir de partenaires

Les appareils qui se connectent à Azure Virtual WAN disposent, pour ce faire, d’une automation intégrée. Celle-ci est généralement configurée à distance dans l’interface utilisateur (ou équivalent) de gestion des appareils, qui configure la connectivité et la gestion de la configuration du périphérique VPN de branche vers un point de terminaison VPN Azure Virtual Hub (passerelle VPN).

L’automation générale suivante est configurée dans la console de l’appareil/le centre de gestion :

* Autorisations appropriées pour que l’appareil accède au groupe de ressources Azure Virtual WAN
* Chargement de l’appareil de branche dans Azure Virtual WAN
* Téléchargement automatique des informations de connectivité Azure
* Configuration de l’appareil de branche sur site 

Certains partenaires de connectivité peuvent étendre l’automation pour inclure la création du réseau virtuel Azure Virtual Hub et la passerelle VPN. Pour en savoir plus sur l’automatisation, consultez [Conseils d’automatisation pour les partenaires Virtual WAN](virtual-wan-configure-automation-providers.md).

## <a name="branch-ipsec-connectivity-partners"></a><a name="partners"></a>Partenaires de connectivité IPSec de filiale

[!INCLUDE [partners](../../includes/virtual-wan-partners-include.md)]

Les partenaires suivants sont inscrits sur notre feuille de route selon une feuille de conditions signée entre les entreprises indiquant l’étendue des travaux d’automatisation de la connectivité IPsec entre le dispositif partenaire et les passerelles VPN Azure Virtual WAN : 128 Technologies, Arista, F5 Networks, Oracle SD-WAN (Talari) et SharpLink.

## <a name="partners-with-integrated-virtual-hub-offerings"></a>Partenaires disposant d’offres Hub virtuel intégrées
En plus de disposer d’une connectivité IPSec de filiale automatisée, certains partenaires proposent des **appliances virtuelle réseau** (Network Virtual Appliances, NVA) qui peuvent être intégrése directement dans le hub Azure Virtual WAN.  Cela permet aux clients d’arrêter leurs connexions de filiale dans une appliance tierce compatible dans le Hub virtuel.  

Les partenaires qui proposent la NVA dans le hub Virtual WAN doivent :

* Avoir implémenté l’automatisation de la connectivité IPSec à partir de leur appareil de filiale et avoir intégré leur offre de NVA au hub Azure Virtual WAN.
* Disposer d’une offre de NVA existante disponible dans la Place de marché Azure.

Si vous êtes partenaire et que vous avez des questions sur la NVA managée dans l’offre Hub virtuel, contactez-nous à l’adresse vwannvaonboarding@microsoft.com

## <a name="integrated-virtual-hub-nva-partners"></a>Partenaires NVA Virtual Hub intégrés
Ces partenaires disposent d’offres **Application managée** pouvant à présent être déployées dans le hub Virtual WAN.

|Partenaires|Configuration/Procédure/Guide de déploiement|
|---|---|
|[Barracuda Networks](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/barracudanetworks.barracuda_cloudgenwan_gateway?tab=Overviewus/marketplace/apps/barracudanetworks.barracuda_cloudgenwan_gateway?tab=Overview)| [Guide de déploiement de réseau WAN Barracuda CloudGen](https://campus.barracuda.com/product/cloudgenwan/doc/91980640/deployment/)|
|[Cisco Cloud Service Router(CSR) VWAN](https://aka.ms/ciscoMarketPlaceOffer)| Au cours de la préversion publique du réseau étendu Cisco Cloud Services (CSR) dans VWAN Hub, Cisco exige que le client final s’inscrive en tant que client Cisco EFT (Early Field Trial) en envoyant un e-mail à vwan_public_preview@external.cisco.com et en demandant le Guide de déploiement vManage. |

Les partenaires suivants sont prêts à intégrer prochainement la NVA dans les offres de hub virtuel : Aviatrix, Citrix, VeloCloud et Versa Networks.

## <a name="locations"></a><a name="locations"></a>Emplacements

[!INCLUDE [regions](../../includes/virtual-wan-regions-include.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ sur Virtual WAN](virtual-wan-faq.md).

* Pour plus d’informations sur l’automatisation de la connectivité à Azure Virtual WAN, consultez [Conseils d’automatisation pour les partenaires Virtual WAN](virtual-wan-configure-automation-providers.md).
