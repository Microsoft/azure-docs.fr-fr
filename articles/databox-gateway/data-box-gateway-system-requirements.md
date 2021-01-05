---
title: Configuration système Microsoft Azure Data Box Gateway | Microsoft Docs
description: En savoir plus sur la configuration logicielle et réseau requise pour Azure Data Box Gateway
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 05/08/2019
ms.author: alkohli
ms.openlocfilehash: aadaedfd2c9ecf544d142e42a0fbeb410324b7d8
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96581109"
---
# <a name="azure-data-box-gateway-system-requirements"></a>Configuration système Azure Data Box Gateway

Cet article décrit la configuration système importante pour votre solution Microsoft Azure Data Box Gateway et pour les clients se connectant à Azure Data Box Gateway. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre solution Data Box Gateway, puis d’y revenir si nécessaire pendant le déploiement et les opérations suivantes. 

La configuration système requise pour l’appareil virtuel Data Box Gateway inclut les aspects suivants :

- **Configuration logicielle pour les hôtes** : décrit les plateformes prises en charge, les navigateurs pour l’interface utilisateur de configuration locale, les clients SMB et des exigences supplémentaires pour les hôtes qui se connectent à l’appareil.
- **Configuration réseau pour l’appareil** : fournit des informations sur la configuration réseau nécessaire au fonctionnement de l’appareil virtuel.


## <a name="specifications-for-the-virtual-device"></a>Spécifications pour l’appareil virtuel

Le système hôte sous-jacent pour le Data Box Gateway est en mesure de dédier les ressources suivantes pour approvisionner votre appareil virtuel :

| Spécifications                                          | Description              |
|---------------------------------------------------------|--------------------------|
| Processeurs virtuels (cœurs)   | 4 minimum |
| Mémoire  | 8 Go minimum|
| Disponibilité|Nœud unique|
| Disques| Disque du système d'exploitation : 250 Go <br> Disque de données : 2 To minimum, allocation dynamique et sauvegarde sur SSD|
| Interfaces réseau|1 ou plusieurs interfaces réseau virtuelles|


## <a name="supported-os-for-clients-connected-to-device"></a>Systèmes d’exploitation pris en charge pour les clients connectés à l’appareil

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>Protocoles pris en charge pour l’accès des clients à l’appareil

[!INCLUDE [Supported protocols for clients accessing device](../../includes/data-box-edge-gateway-supported-client-protocols.md)]

## <a name="supported-virtualization-platforms-for-device"></a>Plateformes de virtualisation prises en charge pour l’appareil

| **Système d’exploitation/plateforme**  |**Versions**   |**Remarques**  |
|---------|---------|---------|
|Hyper-V  |  2012 R2 <br> 2016 <br> 2019 |         |
|VMware ESXi     | 6.0 <br> 6.5 <br> 6.7       |Les outils VMware ne sont pas pris en charge.         |


## <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

[!INCLUDE [Supported storage accounts](../../includes/data-box-edge-gateway-supported-storage-accounts.md)]


## <a name="supported-storage-types"></a>Types de stockage pris en charge

[!INCLUDE [Supported storage types](../../includes/data-box-edge-gateway-supported-storage-types.md)]

## <a name="supported-browsers-for-local-web-ui"></a>Navigateurs pris en charge pour l’interface utilisateur web locale

[!INCLUDE [Supported browsers for local web UI](../../includes/data-box-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>Configuration requise du port réseau

Le tableau ci-dessous répertorie les ports qui doivent être ouverts dans votre pare-feu pour autoriser le trafic SMB, cloud ou de gestion. Dans ce tableau, *entrée* ou *entrant* représente la direction à partir de laquelle les requêtes clientes entrantes accèdent à votre appareil. *Sortie* ou *sortant* représente la direction vers laquelle votre appareil Data Box Gateway envoie des données de façon externe, au-delà du déploiement : par exemple, sortant vers Internet.

[!INCLUDE [Port configuration for device](../../includes/data-box-edge-gateway-port-config.md)]

## <a name="url-patterns-for-firewall-rules"></a>Modèles d’URL pour règles de pare-feu

Les administrateurs réseau peuvent souvent configurer des règles de pare-feu avancées basées sur des modèles d’URL afin de filtrer le trafic entrant et sortant. Votre appareil Data Box Gateway et le service Data Box Gateway dépendent d’autres applications Microsoft comme Azure Service Bus, Azure Active Directory Access Control, des comptes de stockage et des serveurs Microsoft Update. Les modèles d’URL associés à ces applications peuvent être utilisés pour configurer des règles de pare-feu. Il est important de comprendre que les modèles d’URL associés à ces applications peuvent changer. L’administrateur réseau doit alors surveiller et mettre à jour les règles de pare-feu pour votre appareil Data Box Gateway si nécessaire.

Dans la plupart des cas, nous vous recommandons de définir librement les règles de pare-feu pour le trafic sortant en fonction des adresses IP fixes Data Box Gateway. Toutefois, vous pouvez utiliser les informations ci-dessous pour définir les règles de pare-feu avancées qui sont nécessaires à la création d’environnements sécurisés.

> [!NOTE]
> - Les adresses IP d’appareil (sources) doivent toujours être définies sur l’ensemble des interfaces réseau activées pour le cloud.
> - Les adresses IP de destination doivent être définies sur les [plages d’adresses IP Azure Datacenter](https://www.microsoft.com/download/confirmation.aspx?id=41653).

[!INCLUDE [URL patterns for firewall](../../includes/data-box-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-azure-government"></a>Modèles d’URL pour Azure Government

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/data-box-edge-gateway-gov-url-patterns-firewall.md)]

## <a name="internet-bandwidth"></a>Bande passante Internet

[!INCLUDE [Internet bandwidth](../../includes/data-box-edge-gateway-internet-bandwidth.md)]

## <a name="next-step"></a>Étape suivante

* [Déployer votre instance Azure Data Box Gateway](data-box-gateway-deploy-prep.md)

