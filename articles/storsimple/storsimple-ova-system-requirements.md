---
title: Configuration système requise pour Microsoft Azure StorSimple Virtual Array
description: En savoir plus sur la configuration logicielle et du réseau requise pour votre StorSimple Virtual Array
author: alkohli
ms.assetid: ea1d3bca-e71b-453d-aa82-440d2638f5e3
ms.service: storsimple
ms.topic: conceptual
ms.date: 07/25/2019
ms.author: alkohli
ms.openlocfilehash: 5970e82619667a47ba160c84df2cdeb145b0dab8
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966171"
---
# <a name="storsimple-virtual-array-system-requirements"></a>Configuration système requise pour StorSimple Virtual Array

[!INCLUDE [storsimple-virtual-array-eol-banner](../../includes/storsimple-virtual-array-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

Cet article décrit la configuration système requise importante pour Microsoft Azure StorSimple Virtual Array et pour les clients de stockage accédant au tableau. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre système Azure StorSimple, puis d’y revenir si nécessaire pendant le déploiement et les opérations suivantes.

Les conditions requises sont les suivantes :

* **Configuration logicielle requise pour les clients de stockage** : décrit les plateformes de virtualisation, les navigateurs web, les initiateurs iSCSI, les clients SMB pris en charge, la configuration minimale requise pour les appareils virtuels et toutes les exigences requises pour ces systèmes d'exploitation.
* **Conditions requises de mise en réseau pour l’appareil StorSimple** : fournit des informations sur les ports qui doivent être ouverts dans votre pare-feu pour autoriser iSCSI, le cloud ou le trafic de gestion.

Les informations de configuration système requise StorSimple publiées dans cet article s'appliquent uniquement aux tableaux virtuels StorSimple.

* Pour les appareils de la gamme 8000, accédez à [Configuration système requise pour votre appareil de la gamme StorSimple 8000](./storsimple-8000-system-requirements.md).
* Pour les appareils de la gamme 7000, accédez à [Configuration système requise pour votre appareil de la gamme StorSimple 5000-7000](http://onlinehelp.storsimple.com/1_StorSimple_System_Requirements).

## <a name="software-requirements"></a>Configuration logicielle requise
La configuration logicielle requise inclut les informations sur les navigateurs web pris en charge, les versions SMB, les plateformes de virtualisation et la configuration minimale requise pour l'appareil virtuel.

### <a name="supported-virtualization-platforms"></a>Plateformes de virtualisation prises en charge
| **Hyperviseur** | **Version** |
| --- | --- |
| Hyper-V |Windows Server 2008 R2 SP1 et versions ultérieures |
| VMware ESXi |5.0, 5.5, 6.0 et 6.5. |

> [!IMPORTANT]
> N’installez pas les outils VMware sur votre StorSimple Virtual Array, car cela engendrerait une configuration non prise en charge.

### <a name="virtual-device-requirements"></a>Configuration requise de l'appareil virtuel
| **Composant** | **Prérequis** |
| --- | --- |
| Nombre minimal de processeurs virtuels (cœurs) |4 |
| Quantité minimale de mémoire (RAM) |8 Go <br> Pour un serveur de fichiers, 8 Go pour moins de 2 millions de fichiers et 16 Go pour 2 à 4 millions de fichiers|
| Espace disque<sup>1</sup> |Disque de système d'exploitation - 80 Go <br></br>Disque de données - 500 Go à 8 To |
| Nombre minimal d'interfaces réseau |1 |
| Bande passante Internet<sup>2</sup> |Bande passante minimale requise : 5 Mbits/s <br> Bande passante recommandée : 100 Mbits/s <br> La vitesse de transfert des données varie selon la bande passante Internet. Par exemple, le transfert de 100 Go de données prend deux jours avec une bande passante de 5 Mbits/s. Cela peut entraîner des échecs de sauvegarde, car les sauvegardes quotidiennes ne peuvent pas dans ce cas s’effectuer en un seul jour. Avec une bande passante de 100 Mbits/s, le transfert de 100 Go de données s’effectue en deux heures et demie.   |

<sup>1</sup> - Allocation dynamique

<sup>2</sup> - La configuration requise du réseau peut varier selon le taux de modification quotidien des données. Par exemple, si un appareil doit sauvegarder 10 Go de modifications ou plus pendant une journée, alors la sauvegarde quotidienne via une connexion 5 Mbits/s peut prendre jusqu'à 4,25 heures (si les données ne sont pas compressées ou dédupliquées).

### <a name="supported-web-browsers"></a>Navigateurs web pris en charge
| **Composant** | **Version** | **Conditions/remarques supplémentaires** |
| --- | --- | --- |
| Microsoft Edge |Version la plus récente | |
| Internet Explorer |Version la plus récente |Testé avec Internet Explorer 11 |
| Google Chrome |Version la plus récente |Testé avec Chrome 46 |

### <a name="supported-storage-clients"></a>Clients de stockage pris en charge
La configuration logicielle requise suivante concerne les initiateurs iSCSI qui accèdent à votre StorSimple Virtual Array (configuré comme un serveur iSCSI).

| **Systèmes d’exploitation pris en charge** | **Version requise** | **Conditions/remarques supplémentaires** |
| --- | --- | --- |
| Windows Server |2008 R2 SP1, 2012, 2012 R2 |StorSimple peut créer des volumes alloués dynamiquement et de façon complète. Il ne permet pas de créer des volumes entièrement ou partiellement alloués. Les volumes iSCSI StorSimple sont pris en charge uniquement pour : <ul><li>Volumes simples sur des disques de base Windows.</li><li>Windows NTFS pour formater un volume.</li> |

La configuration logicielle requise suivante concerne les clients SMB qui accèdent à votre StorSimple Virtual Array (configuré comme un serveur de fichiers).

| **Version SMB** |
| --- |
| SMB 2.x |
| SMB 3.0 |
| SMB 3.02 |

> [!IMPORTANT]
> Ne copiez pas et ne stockez pas de fichiers protégés par le système de fichiers EFS Windows pour le serveur de fichiers du groupe virtuel StorSimple. Cela donne lieu à une configuration non prise en charge.


### <a name="supported-storage-format"></a>Format de stockage pris en charge
Seul le stockage d’objets blob de blocs Azure est pris en charge. Les objets blob de pages ne sont pas pris en charge. En savoir plus [sur les objets blob de blocs et de pages](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs).

## <a name="networking-requirements"></a>Configuration requise du réseau
Le tableau ci-dessous répertorie les ports qui doivent être ouverts dans votre pare-feu pour autoriser le trafic iSCSI, SMB, cloud ou de gestion. Dans ce tableau, *entrée* ou *entrant* représente la direction à partir de laquelle les demandes client entrantes accèdent à votre appareil. *Sortant* ou *Sortie* représente la direction vers laquelle votre appareil StorSimple envoie des données de façon externe, au-delà du déploiement : par exemple, sortant vers Internet.

| **Numéro de port <sup>1</sup>** | **Entrant ou sortant** | **Étendue de ports** | **Obligatoire** | **Remarques** |
| --- | --- | --- | --- | --- |
| TCP 80 (HTTP) |Sortie |WAN |Non |Le port de sortie est utilisé pour accéder à Internet afin de récupérer les mises à jour. <br></br>Le proxy web sortant est configurable par l’utilisateur. |
| TCP 443 (HTTPS) |Sortie |WAN |Oui |Le port de sortie est utilisé pour accéder aux données dans le cloud. <br></br>Le proxy web sortant est configurable par l’utilisateur. |
| UDP 53 (DNS) |Sortie |WAN |Dans certains cas, consultez les notes. |Ce port est requis seulement si vous utilisez un serveur DNS Internet. <br></br> Notez que si vous déployez un serveur de fichiers, nous recommandons l’utilisation d’un serveur DNS local. |
| UDP 123 (NTP) |Sortie |WAN |Dans certains cas, consultez les notes. |Ce port est requis seulement si vous utilisez un serveur NTP Internet.<br></br> Notez que si vous déployez un serveur de fichiers, nous vous recommandons de synchroniser l’heure avec vos contrôleurs de domaine Active Directory. |
| TCP 80 (HTTP) |Dans |LAN |Oui |Il s'agit du port d'entrée pour l'interface utilisateur locale de l'appareil StorSimple pour la gestion locale. <br></br> Notez que l’accès à l’interface utilisateur locale par le biais du protocole HTTP est automatiquement redirigé vers HTTPS. |
| TCP 443 (HTTPS) |Dans |LAN |Oui |Il s'agit du port d'entrée pour l'interface utilisateur locale de l'appareil StorSimple pour la gestion locale. |
| TCP 3260 (iSCSI) |Dans |LAN |Non |Ce port est utilisé pour accéder aux données via iSCSI. |

<sup>1</sup> Aucun port entrant ne doit être ouvert sur l’Internet public.

> [!IMPORTANT]
> Vérifiez que le pare-feu ne modifie ou ne déchiffre pas le trafic TLS entre l’appareil StorSimple et Azure.
> 
> 

### <a name="url-patterns-for-firewall-rules"></a>Modèles d’URL pour règles de pare-feu
Les administrateurs réseau peuvent souvent configurer des règles de pare-feu avancées basées sur des modèles d’URL afin de filtrer le trafic entrant et sortant. Votre tableau virtuel et le service StorSimple Device Manager dépendent d’autres applications Microsoft telles qu’Azure Service Bus, Azure Active Directory Access Control, ou que des comptes de stockage et des serveurs Microsoft Update. Les modèles d’URL associés à ces applications peuvent être utilisés pour configurer des règles de pare-feu. Il est important de comprendre que les modèles d’URL associés à ces applications peuvent changer. L’administrateur réseau doit alors surveiller et mettre à jour les règles de pare-feu pour votre appareil StorSimple si nécessaire. 

Dans la plupart des cas, nous vous recommandons de définir librement les règles de pare-feu pour le trafic sortant en fonction des ADresses IP fixes StorSimple. Toutefois, vous pouvez utiliser les informations ci-dessous pour définir les règles de pare-feu avancées qui sont nécessaires à la création d’environnements sécurisés.

> [!NOTE]
> 
> * Les adresses IP d’appareil (sources) doivent toujours être définies sur l’ensemble des interfaces réseau activées pour le cloud. 
> * Les adresses IP de destination doivent être définies sur les [plages d’adresses IP Azure Datacenter](https://www.microsoft.com/download/confirmation.aspx?id=41653).
> 
> 

| Modèle d’URL | Composant/Fonctionnalité |
| --- | --- |
| `https://*.storsimple.windowsazure.com/*`<br>`https://*.accesscontrol.windows.net/*`<br>`https://*.servicebus.windows.net/*` <br>`https://login.windows.net`|Service StorSimple Device Manager<br>Service de contrôle d’accès<br>Azure Service Bus<br>Service d’authentification|
| `http://*.backup.windowsazure.com` |Enregistrement de l’appareil |
| `https://crl.microsoft.com/pki/*`<br>`https://www.microsoft.com/pki/*` |Révocation de certificat |
| `https://*.core.windows.net/*`<br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Comptes de stockage Azure et surveillance |
| `https://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`https://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`https://download.microsoft.com`<br>`http://wustat.windows.com`<br>`https://ntservicepack.microsoft.com` |Serveurs Microsoft Update<br> |
| `http://*.deploy.akamaitechnologies.com` |CDN Akamai |
| `https://*.partners.extranet.microsoft.com/*` |Package de prise en charge |
| `https://*.data.microsoft.com` |Service de télémétrie dans Windows, consultez l’article [Mise à jour de l’expérience client et du diagnostic de la télémétrie](https://support.microsoft.com/en-us/kb/3068708) |

## <a name="next-steps"></a>Étapes suivantes
* [Préparation du portail pour déployer StorSimple Virtual Array](storsimple-virtual-array-deploy1-portal-prep.md)