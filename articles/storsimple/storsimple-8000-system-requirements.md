---
title: Configuration requise pour StorSimple 8000 Series | Microsoft Docs
description: Décrit la configuration requise et les meilleures pratiques en matière de logiciel, de mise en réseau et de haute disponibilité pour une solution Microsoft Azure StorSimple.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 09/28/2017
ms.author: alkohli
ms.openlocfilehash: 6dcaa83980210a1f5449e8a2e0982cb8e39ff03d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966188"
---
# <a name="storsimple-8000-series-software-high-availability-and-networking-requirements"></a>Configurations logicielles, de haute disponibilité et réseau requises pour StorSimple 8000 Series

## <a name="overview"></a>Vue d’ensemble

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

Bienvenue dans Microsoft Azure StorSimple. Cet article décrit les configurations système requises et les meilleures pratiques pour votre appareil StorSimple et pour les clients de stockage accédant à l’appareil. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre système Azure StorSimple, puis d’y revenir si nécessaire pendant le déploiement et les opérations suivantes.

Les conditions requises sont les suivantes :

* **Configuration logicielle requise pour les clients de stockage** : décrit les systèmes d’exploitation pris en charge et les conditions supplémentaires requises pour ces systèmes d’exploitation.
* **Conditions requises de mise en réseau pour l’appareil StorSimple** : fournit des informations sur les ports qui doivent être ouverts dans votre pare-feu pour autoriser iSCSI, le cloud ou le trafic de gestion.
* **Conditions requises de haute disponibilité pour StorSimple** : décrit les exigences de haute disponibilité et les meilleures pratiques pour votre ordinateur hôte et votre appareil StorSimple.

## <a name="software-requirements-for-storage-clients"></a>Configuration logicielle requise pour les clients de stockage

La configuration logicielle suivante est requise pour les clients de stockage qui accèdent à votre appareil StorSimple.

| Systèmes d’exploitation pris en charge | Version requise | Conditions/remarques supplémentaires |
| --- | --- | --- |
| Windows Server |2008 R2 SP1, 2012, 2012 R2, 2016 |Les volumes iSCSI StorSimple sont pris en charge uniquement sur les types de disques Windows suivants :<ul><li>Volume simple sur disque de base</li><li>Volume simple et en miroir sur disque dynamique</li></ul>Seuls les initiateurs logiciels iSCSI présents en mode natif dans le système d’exploitation sont pris en charge. Les initiateurs matériels iSCSI ne sont pas pris en charge.<br></br>L’allocation dynamique Windows Server 2012 et 2016 les fonctionnalités ODX sont prises en charge si vous utilisez un volume iSCSI StorSimple.<br><br>StorSimple peut créer des volumes alloués dynamiquement et de façon complète. Il ne permet pas de créer des volumes entièrement ou partiellement alloués.<br><br>Le reformatage d’un volume alloué dynamiquement peut prendre beaucoup de temps. Nous vous recommandons de supprimer le volume, puis d’en créer un nouveau plutôt que de le reformater. Si vous préférez toutefois reformater un volume :<ul><li>Exécutez la commande suivante avant le reformatage pour éviter les retards de récupération d’espace : <br>`fsutil behavior set disabledeletenotify 1`</br></li><li>Une fois le formatage terminé, utilisez la commande suivante pour réactiver une récupération de l’espace :<br>`fsutil behavior set disabledeletenotify 0`</br></li><li>Appliquez le correctif Windows Server 2012, comme décrit dans l’article [KB 2878635](https://support.microsoft.com/kb/2870270) sur votre ordinateur Windows Server.</li></ul></li></ul></ul> Si vous configurez le Gestionnaire d’instantanés StorSimple ou l’adaptateur StorSimple pour SharePoint, consultez [Configuration logicielle requise pour les composants facultatifs](#software-requirements-for-optional-components). |
| VMWare ESX |5.5 et 6.0 |Pris en charge avec VMware vSphere en tant que client iSCSI. La fonctionnalité VAAI-block est prise en charge avec VMware vSphere sur les appareils StorSimple. |
| Linux RHEL/CentOS |5, 6 et 7 |Prise en charge des clients Linux iSCSI avec initiateur Open-iSCSI versions 5, 6 et 7. |
| Linux |SUSE Linux 11 | |

> [!NOTE]
> IBM AIX n’est actuellement pas pris en charge avec StorSimple.


## <a name="software-requirements-for-optional-components"></a>Configuration logicielle requise pour les composants facultatifs

La configuration logicielle requise suivante concerne les composants StorSimple  facultatifs (Gestionnaire d’instantanés StorSimple et adaptateur StorSimple pour SharePoint).

| Composant | Plateforme hôte | Conditions/remarques supplémentaires |
| --- | --- | --- |
| Gestionnaire d’instantanés StorSimple |Windows Server 2008 R2 SP1, 2012, 2012 R2 |L’utilisation du Gestionnaire d’instantanés StorSimple sur Windows Server est requise pour la sauvegarde/restauration des disques dynamiques en miroir et pour les sauvegardes cohérentes avec les applications.<br> StorSimple Snapshot Manager est pris en charge uniquement sous Windows Server 2008 R2 SP1 (64 bits), Windows Server 2012 R2 et Windows Server 2012.<ul><li>Si vous utilisez Windows Server 2012, vous devez installer .NET 3.5-4.5 avant d’installer le Gestionnaire d’instantanés StorSimple.</li><li>Si vous utilisez Windows Server 2008 R2 SP1, vous devez installer Windows Management Framework 3.0 avant d’installer le Gestionnaire d’instantanés StorSimple.</li></ul> |
| Adaptateur StorSimple pour SharePoint |Windows Server 2008 R2 SP1, 2012, 2012 R2 |<ul><li>L'adaptateur StorSimple pour SharePoint est uniquement pris en charge sur SharePoint 2010 et SharePoint 2013.</li><li>RBS requiert SQL Server Enterprise Edition, version 2008 R2 ou 2012.</li></ul> |

## <a name="networking-requirements-for-your-storsimple-device"></a>Configuration réseau requise pour votre appareil StorSimple

Votre appareil StorSimple est un appareil verrouillé. Toutefois, les ports doivent être ouverts dans votre pare-feu pour autoriser le trafic iSCSI, cloud et de gestion. Le tableau suivant répertorie les ports devant être ouverts dans votre pare-feu. Dans ce tableau, *entrée* ou *entrant* représente la direction à partir de laquelle les demandes client entrantes accèdent à votre appareil. *Sortant* ou *Sortie* représente la direction vers laquelle votre appareil StorSimple envoie des données de façon externe, au-delà du déploiement : par exemple, sortant vers Internet.

| Numéro de port <sup>1,2</sup> | Entrant ou sortant | Étendue de ports | Obligatoire | Notes |
| --- | --- | --- | --- | --- |
| TCP 80 (HTTP)<sup>3</sup> |Sortie |WAN |Non |<ul><li>Le port de sortie est utilisé pour accéder à Internet afin de récupérer les mises à jour.</li><li>Le proxy web sortant est configurable par l’utilisateur.</li><li>Pour autoriser les mises à jour du système, ce port doit également être ouvert pour les adresses IP fixes du contrôleur.</li></ul> |
| TCP 443 (HTTPS)<sup>3</sup> |Sortie |WAN |Oui |<ul><li>Le port de sortie est utilisé pour accéder aux données dans le cloud.</li><li>Le proxy web sortant est configurable par l’utilisateur.</li><li>Pour autoriser les mises à jour du système, ce port doit également être ouvert pour les adresses IP fixes du contrôleur.</li><li>Ce port est aussi utilisé sur les deux contrôleurs pour le garbage collection.</li></ul> |
| UDP 53 (DNS) |Sortie |WAN |Dans certains cas, consultez les notes. |Ce port est requis seulement si vous utilisez un serveur DNS Internet. |
| UDP 123 (NTP) |Sortie |WAN |Dans certains cas, consultez les notes. |Ce port est requis seulement si vous utilisez un serveur NTP Internet. |
| TCP 9354 |Sortie |WAN |Oui |Le port de sortie est utilisé par l’appareil StorSimple Manager pour communiquer avec le service StorSimple Device Manager. |
| 3260 (iSCSI) |Dans |LAN |Non |Ce port est utilisé pour accéder aux données via iSCSI. |
| 5985 |Dans |LAN |Non |Le port entrant est utilisé par le gestionnaire d’instantanés StorSimple pour communiquer avec l’appareil StorSimple.<br>Ce port est également utilisé lorsque vous vous connectez à distance à Windows PowerShell pour StorSimple via HTTP. |
| 5986 |Dans |LAN |Non |Ce port est utilisé lorsque vous vous connectez à distance à Windows PowerShell pour StorSimple via HTTPS. |

<sup>1</sup> Aucun port entrant ne doit être ouvert sur l’Internet public.

<sup>2</sup> Si plusieurs ports comportent une configuration de passerelle, l’ordre du trafic routé sortant est déterminé par l’ordre de routage des ports décrit dans la section [Routage de port](#routing-metric) ci-dessous.

<sup>3</sup> Les adresses IP fixes du contrôleur sur votre appareil StorSimple doivent être routables et doivent pouvoir se connecter à Internet directement ou via le proxy web configuré. Les adresses IP fixes servent à traiter les mises à jour de l’appareil et à nettoyer la mémoire. Si les contrôleurs de l’appareil ne peuvent pas se connecter à Internet via les adresses IP fixes, vous ne pouvez pas mettre à jour votre appareil StorSimple et l’opération de nettoyage de la mémoire ne se déroulera pas correctement.

> [!IMPORTANT]
> Vérifiez que le pare-feu ne modifie ou ne déchiffre pas le trafic TLS entre l’appareil StorSimple et Azure.


### <a name="url-patterns-for-firewall-rules"></a>Modèles d’URL pour règles de pare-feu

Les administrateurs réseau peuvent souvent configurer des règles de pare-feu avancées basées sur des modèles d’URL afin de filtrer le trafic entrant et sortant. Votre appareil StorSimple et le service StorSimple Device Manager dépendent d’autres applications Microsoft telles qu’Azure Service Bus, Azure Active Directory Access Control, des comptes de stockage et des serveurs Microsoft Update. Les modèles d’URL associés à ces applications peuvent être utilisés pour configurer des règles de pare-feu. Il est important de comprendre que les modèles d’URL associés à ces applications peuvent changer. L’administrateur réseau doit alors surveiller et mettre à jour les règles de pare-feu pour votre appareil StorSimple si nécessaire.

Dans la plupart des cas, nous vous recommandons de définir librement les règles de pare-feu pour le trafic sortant en fonction des ADresses IP fixes StorSimple. Toutefois, vous pouvez utiliser les informations ci-dessous pour définir les règles de pare-feu avancées qui sont nécessaires à la création d’environnements sécurisés.

> [!NOTE]
> Les adresses IP d’appareil (sources) doivent toujours être définies sur l’ensemble des interfaces réseau activées. Les adresses IP de destination doivent être définies sur les [plages d’adresses IP Azure Datacenter](https://www.microsoft.com/en-us/download/confirmation.aspx?id=41653).


#### <a name="url-patterns-for-azure-portal"></a>Modèles d’URL pour le portail Azure

| Modèle d’URL | Composant/Fonctionnalité | Adresses IP d’appareil |
| --- | --- | --- |
| `https://*.storsimple.windowsazure.com/*`<br>`https://*.accesscontrol.windows.net/*`<br>`https://*.servicebus.windows.net/*`<br>`https://login.windows.net` |Service StorSimple Device Manager<br>Service de contrôle d’accès<br>Azure Service Bus<br>Service d’authentification |Interfaces réseau activées pour le cloud |
| `https://*.backup.windowsazure.com` |Enregistrement de l’appareil |DATA 0 uniquement |
| `https://crl.microsoft.com/pki/*`<br>`https://www.microsoft.com/pki/*` |Révocation de certificat |Interfaces réseau activées pour le cloud |
| `https://*.core.windows.net/*` <br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Comptes de stockage Azure et surveillance |Interfaces réseau activées pour le cloud |
| `https://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`https://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`https://download.microsoft.com`<br>`http://wustat.windows.com`<br>`https://ntservicepack.microsoft.com` |Serveurs Microsoft Update<br> |Adresses IP fixes du contrôleur uniquement |
| `http://*.deploy.akamaitechnologies.com` |CDN Akamai |Adresses IP fixes du contrôleur uniquement |
| `https://*.partners.extranet.microsoft.com/*`<br>`https://dcupload.microsoft.com/`<br>`https://*.support.microsoft.com/` |Package de prise en charge |Interfaces réseau activées pour le cloud |

#### <a name="url-patterns-for-azure-government-portal"></a>Modèles d’URL pour le portail Azure Government

| Modèle d’URL | Composant/Fonctionnalité | Adresses IP d’appareil |
| --- | --- | --- |
| `https://*.storsimple.windowsazure.us/*`<br>`https://*.accesscontrol.usgovcloudapi.net/*`<br>`https://*.servicebus.usgovcloudapi.net/*`<br>`https://login.microsoftonline.us` |Service StorSimple Device Manager<br>Service de contrôle d’accès<br>Azure Service Bus<br>Service d’authentification |Interfaces réseau activées pour le cloud |
| `https://*.backup.windowsazure.us` |Enregistrement de l’appareil |DATA 0 uniquement |
| `https://crl.microsoft.com/pki/*`<br>`https://www.microsoft.com/pki/*` |Révocation de certificat |Interfaces réseau activées pour le cloud |
| `https://*.core.usgovcloudapi.net/*` <br>`https://*.data.microsoft.com`<br>`http://*.msftncsi.com` |Comptes de stockage Azure et surveillance |Interfaces réseau activées pour le cloud |
| `https://*.windowsupdate.microsoft.com`<br>`https://*.windowsupdate.microsoft.com`<br>`https://*.update.microsoft.com`<br> `https://*.update.microsoft.com`<br>`http://*.windowsupdate.com`<br>`https://download.microsoft.com`<br>`http://wustat.windows.com`<br>`https://ntservicepack.microsoft.com` |Serveurs Microsoft Update<br> |Adresses IP fixes du contrôleur uniquement |
| `http://*.deploy.akamaitechnologies.com` |CDN Akamai |Adresses IP fixes du contrôleur uniquement |
| `https://*.partners.extranet.microsoft.com/*`<br>`https://dcupload.microsoft.com/`<br>`https://*.support.microsoft.com/` |Package de prise en charge |Interfaces réseau activées pour le cloud |

### <a name="routing-metric"></a>Métrique de routage

Une métrique de routage est associée aux interfaces et à la passerelle qui acheminent les données vers les réseaux spécifiés. La métrique de routage est utilisée par le protocole de routage pour calculer le meilleur chemin vers une destination donnée, si plusieurs chemins existent pour la même destination. La métrique de routage est inversement proportionnelle à la préférence.

Dans le contexte de StorSimple, si plusieurs interfaces et passerelles de réseau sont configurées pour canaliser le trafic, les métriques de routage entrent en jeu afin de déterminer l’ordre relatif dans lequel les interfaces seront utilisées. Les métriques de routage ne peuvent pas être modifiées par l’utilisateur. Toutefois, vous pouvez utiliser l’applet de commande `Get-HcsRoutingTable` pour imprimer la table de routage (et les métriques de routage) sur votre appareil StorSimple. Pour plus d’informations sur l’applet de commande Get-HcsRoutingTable, consultez le [dépannage du déploiement StorSimple](./storsimple-8000-troubleshoot-deployment.md).

L’algorithme de métrique de routage utilisé pour la mise à jour 2 et versions ultérieures peut être décrit comme suit.

* Un ensemble de valeurs prédéterminées ont été attribuées aux interfaces réseau.
* Examinez la table d'exemple ci-dessous et les valeurs attribuées aux différentes interfaces réseau lorsqu'elles sont activées ou désactivées pour le cloud, mais avec une passerelle configurée. Notez que les valeurs attribuées ici ne sont fournies qu’à titre d’exemple.

    | interface réseau | Activée pour le cloud | Désactivée pour le cloud avec passerelle |
    |-----|---------------|---------------------------|
    | Data 0  | 1            | -                        |
    | Data 1  | 2            | 20                       |
    | Data 2  | 3            | 30                       |
    | Data 3  | 4            | 40                       |
    | Data 4  | 5            | 50                       |
    | Data 5  | 6            | 60                       |


* L'ordre dans lequel le trafic cloud sera acheminé sur l’ensemble des interfaces réseau est le suivant :
  
    *Data 0 &gt; Data 1 &gt; Date 2 &gt; Data 3 &gt; Data 4 &gt; Data 5*
  
    Cela peut s’expliquer par l’exemple suivant.
  
    Prenez l’exemple d’un appareil StorSimple avec deux interfaces réseau activées pour le cloud, Data 0 et Data 5. Data 1 à Data 4 sont désactivées pour le cloud mais disposent d’une passerelle configurée. L’ordre dans lequel le trafic sera acheminé pour cet appareil est le suivant :
  
    *Data 0 (1) &gt; Data 5 (6) &gt; Data 1 (20) &gt; Data 2 (30) &gt; Data 3 (40) &gt; Data 4 (50)*
  
    *Les nombres entre parenthèses indiquent les métriques de routage respectives.*
  
    En cas d’échec de Data 0, le trafic cloud sera acheminé via Data 5. Étant donné qu'une passerelle est configurée sur tous les autres réseaux, si Data 0 et Data 5 échouaient, le trafic cloud serait acheminé via Data 1.
* En cas d'échec d’une interface réseau activée pour le cloud, 3 tentatives de connexion à l’interface sont effectuées à une intervalle de 30 secondes. Si toutes les tentatives échouent, le trafic est acheminé vers l’interface activée pour le cloud disponible suivante selon la table de routage. Si toutes les interfaces activées pour le cloud échouent, l’appareil basculera vers l’autre contrôleur (pas de redémarrage dans ce cas).
* En cas d’échec de l’adresse IP virtuelle pour une interface réseau compatible iSCSI, 3 tentatives seront effectuées à une intervalle de 2 secondes. Ce comportement est le même que dans les versions précédentes. Si toutes les interfaces réseau iSCSI échouent, un basculement de contrôleur se produit (accompagné d'un redémarrage).
* Une alerte est également émise sur votre appareil StorSimple en cas d’échec de l’adresse IP virtuelle. Pour plus d'informations, consultez la page [alerte aide-mémoire](storsimple-8000-manage-alerts.md).
* En ce qui concerne les nouvelles tentatives, iSCSI a priorité sur le cloud.
  
    Prenons l’exemple suivant : un appareil StorSimple a deux interfaces réseau activées, Data 0 et Data 1. Data 0 est activée pour le cloud tandis que Data 1 est à la fois activée pour le cloud et compatible iSCSI. Aucune autre interface réseau sur cet appareil n’est activée pour le cloud ou compatible iSCSI.
  
    Si Data 1 échoue, étant donné qu'il s’agit de la dernière interface réseau iSCSI, cela entraîne un basculement de contrôleur vers Data 1 sur l'autre contrôleur.

### <a name="networking-best-practices"></a>Meilleures pratiques de mise en réseau

Outre les exigences de mise en réseau ci-dessus, pour obtenir des performances optimales pour votre solution StorSimple, veuillez respecter les meilleures pratiques suivantes :

* Assurez-vous que votre appareil StorSimple a une bande passante dédiée de 40 Mbits/s (ou plus) disponible à tout moment. Cette bande passante ne doit pas être partagée (ou l’allocation doit être garantie par le biais de l’utilisation de stratégies QoS) avec d’autres applications.
* Vérifiez que la connectivité réseau à Internet est disponible en permanence. Les connexions Internet sporadiques ou non fiables aux appareils, y compris aucune connectivité Internet quelle qu’elle soit, se traduisent par l’absence de prise en charge de la configuration.
* Isolez le trafic iSCSI et le trafic cloud en dédiant les interfaces réseau de votre appareil pour l’accès iSCSI et cloud. Pour plus d’informations, consultez la section relative à la [modification des interfaces réseau](storsimple-8000-modify-device-config.md#modify-network-interfaces) sur votre appareil StorSimple.
* N’utilisez pas une configuration Link Aggregation Control Protocol (LACP) pour vos interfaces réseau. Cette configuration n’est pas prise en charge.

## <a name="high-availability-requirements-for-storsimple"></a>Conditions requises de haute disponibilité pour StorSimple

La plateforme matérielle fournie avec la solution StorSimple offre des fonctionnalités de disponibilité et de fiabilité qui constituent la base d’une infrastructure de stockage à tolérance de panne et à haute disponibilité dans votre centre de données. Cependant, vous devez vous conformer aux meilleures pratiques et conditions requises pour garantir la disponibilité de votre solution StorSimple. Avant de déployer StorSimple, examinez attentivement les conditions requises et meilleures pratiques suivantes pour l’appareil StorSimple et les ordinateurs hôtes connectés.

Pour plus d’informations sur la surveillance et la maintenance des composants matériels de votre appareil StorSimple, voir [Utiliser le service StorSimple Device Manager pour surveiller les composants et l’état du matériel](storsimple-8000-monitor-hardware-status.md) et [Remplacement des composants matériels StorSimple](storsimple-8000-hardware-component-replacement.md).

### <a name="high-availability-requirements-and-procedures-for-your-storsimple-device"></a>Configuration requise pour la haute disponibilité et procédures pour votre appareil StorSimple

Lisez attentivement les informations suivantes pour garantir la haute disponibilité de votre appareil StorSimple.

#### <a name="pcms"></a>Modules PCM

Les appareils StorSimple incluent des modules d’alimentation et de refroidissement redondants et échangeables à chaud (PCM). Chaque module PCM est doté d’une capacité suffisante pour prendre en charge l’ensemble du châssis. Les deux modules PCM doivent être installés pour assurer un haut niveau de disponibilité.

* Connectez vos modules PCM à différentes sources d’alimentation pour assurer la disponibilité en cas de panne d’une source d’alimentation.
* Si un module PCM tombe en panne, demandez immédiatement son remplacement.
* Ne retirez un module PCM en panne que si vous disposez d’un module de rechange et êtes prêt à l’installer.
* Ne supprimez pas les deux modules PCM en même temps. Le module PCM inclut le module de batterie de secours. La suppression des deux modules PCM entraîne un arrêt sans protection de la batterie ; dans ce cas, l’état de l’appareil ne sera pas sauvegardé. Pour plus d’informations sur la batterie, consultez [Entretien du module de batterie de secours](storsimple-8000-battery-replacement.md#maintain-the-backup-battery-module).

#### <a name="controller-modules"></a>Modules de contrôleur

Les appareils StorSimple incluent des modules de contrôleur redondants et échangeables à chaud. Les modules de contrôleur fonctionnent en mode actif/passif. À tout moment, un module de contrôleur est actif et fournit un service, tandis que l’autre est passif. Le module de contrôleur passif est sous tension et devient opérationnel en cas de panne ou de retrait du module de contrôleur actif. Chaque module de contrôleur est doté d’une capacité suffisante pour prendre en charge l’ensemble du châssis. Les deux modules de contrôleur doivent être installés pour assurer un haut niveau de disponibilité.

* Assurez-vous que les deux modules de contrôleur sont installés en permanence.
* En cas de panne d’un module de contrôleur, demandez immédiatement son remplacement.
* Ne retirez un module de contrôleur défaillant que si vous disposez d’un module de rechange et êtes prêt à l’installer. La suppression d’un module pendant des périodes étendues affectera la ventilation et, par conséquent, le refroidissement du système.
* Assurez-vous que les connexions réseau aux deux modules de contrôleur sont identiques, et que les interfaces réseau connectées ont une configuration réseau identique.
* Si un module de contrôleur tombe en panne ou doit être remplacé, assurez-vous que l’autre module de contrôleur est dans un état actif avant de procéder au remplacement du module défaillant. Pour vérifier si un contrôleur est actif, consultez [Identification du contrôleur actif sur votre appareil](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device).
* Ne supprimez pas les deux modules de contrôleur en même temps. Si un contrôleur est en cours de basculement, n’arrêtez pas le module de contrôleur de secours, et ne le retirez pas du châssis.
* Après un basculement de contrôleur, attendez au moins cinq minutes avant de retirer l’un des deux modules de contrôleur.

#### <a name="network-interfaces"></a>Interfaces réseau

Les modules de contrôleur d’un appareil StorSimple possèdent chacun quatre interfaces réseau 1 Gigabit et deux interfaces réseau 10 Gigabit Ethernet.

* Assurez-vous que les connexions réseau aux deux modules de contrôleur sont identiques, et que les interfaces réseau auxquelles les interfaces des modules de contrôleur sont connectées ont une configuration réseau identique.
* Si possible, déployez les connexions réseau sur différents commutateurs pour assurer la disponibilité du service en cas de panne d’appareil réseau.
* Lorsque vous déconnectez la seule ou la dernière interface compatible iSCSI restante (avec affectation des adresses IP), désactivez tout d’abord l’interface, puis débranchez les câbles. Si l’interface est débranchée en premier, il s’ensuit un basculement du contrôleur actif vers le contrôleur passif. Si les interface correspondantes du contrôleur passif sont également débranchées, les deux contrôleurs redémarrent plusieurs fois avant qu’un seul contrôleur. ne soit choisi.
* Connectez au moins deux interfaces DATA au réseau à partir de chaque module de contrôleur.
* Si vous avez activé les deux interfaces de 10 GbE, déployez-les entre les différents commutateurs.
* Si possible, utilisez MPIO sur les serveurs pour vous assurer qu’ils peuvent tolérer une panne de liaison, de réseau ou d’interface.

Pour plus d’informations sur la mise en réseau de votre appareil pour la haute disponibilité et les performances, consultez [Installation de votre appareil StorSimple 8100](storsimple-8100-hardware-installation.md#cable-your-storsimple-8100-device) ou [Installation de votre appareil StorSimple 8600](storsimple-8600-hardware-installation.md#cable-your-storsimple-8600-device).

#### <a name="ssds-and-hdds"></a>Disques SSD et disques durs

Un appareil StorSimple comprend des disques SSD et des disques durs qui sont protégés au moyen d'espaces en miroir. Grâce aux espaces en miroir, un appareil est capable de tolérer une panne d’un ou plusieurs disques SSD ou disques durs.

* Assurez-vous que tous les disques SSD et disques durs sont en place.
* Si un disque SSD ou un disque dur tombe en panne, demandez immédiatement son remplacement.
* Si un disque SSD ou un disque dur doit être remplacé, veillez à retirer uniquement le disque défectueux.
* Ne retirez pas plusieurs disques SSD ou disques durs du système en même temps.
  Si au moins deux disques d’un même type (disque dur ou disque SSD) tombent en panne ou que plusieurs pannes consécutives surviennent en peu de temps, un dysfonctionnement du système et une perte potentielle de données risquent de se produire. Si cela se produit, [contactez le support Microsoft](storsimple-8000-contact-microsoft-support.md) .
* Pendant le remplacement, surveillez les **composants partagés** dans le panneau **Intégrité matérielle** pour les lecteurs de disques SSD et HDD. Une coche verte indique que les disques sont en bon état, tandis qu’un point d’exclamation rouge indique un disque SSD ou dur défectueux.
* Nous vous recommandons de configurer des instantanés cloud pour tous les volumes à protéger en cas de panne du système.

#### <a name="ebod-enclosure"></a>Boîtier EBOD

L’appareil StorSimple (modèle 8600) est équipé d’un boîtier EBOD (Extended Bunch of Disks) en plus du boîtier principal. Un boîtier EBOD se compose de contrôleurs EBOD et de disques durs qui sont protégés à l’aide d’espaces en miroir. Grâce aux espaces en miroir, un appareil est capable de tolérer une panne d’un ou de plusieurs disques durs. Le boîtier EBOD est connecté au boîtier principal avec de câbles SAS redondants.

* Assurez-vous que les deux modules de contrôleur du boîtier EBOD, les câbles SAS et tous les disques durs sont installés en permanence.
* Si un module de contrôleur de boîtier EBOD tombe en panne, demandez immédiatement son remplacement.
* Si un module de contrôleur de boîtier EBOD tombe en panne, avant de remplacer le module défectueux, assurez-vous que l’autre module de contrôleur est actif. Pour vérifier si un contrôleur est actif, consultez [Identification du contrôleur actif sur votre appareil](storsimple-8000-controller-replacement.md#identify-the-active-controller-on-your-device).
* Lors du remplacement d’un module de contrôleur EBOD, surveillez en permanence l’état du composant dans le service StorSimple Device Manager. Pour cela, accédez à **Surveiller** > **Intégrité matérielle**.
* Si un câble SAS est défectueux ou doit être remplacé (cette décision doit être prise après consultation du support technique Microsoft), veillez à retirer uniquement le câble à remplacer.
* Ne retirez pas les deux câbles SAS du système en même temps.

### <a name="high-availability-recommendations-for-your-host-computers"></a>Recommandations relatives à la haute disponibilité de vos ordinateurs hôtes

Lisez attentivement ces meilleures pratiques recommandées pour assurer la haute disponibilité des hôtes connectés à votre appareil StorSimple.

* Configurez StorSimple avec des [configurations de cluster de serveur de fichiers à deux nœuds][1]. En supprimant les points de défaillance uniques et en assurant la redondance côté hôte, l’ensemble de la solution devient hautement disponible.
* Utilisez des partages disponibles en continu avec Windows Server 2012 (SMB 3.0) pour bénéficier d’un haut niveau de disponibilité lors du basculement des contrôleurs de stockage. Pour plus d’informations sur la configuration de clusters de serveurs de fichiers et de partages disponibles en continu avec Windows Server 2012, consultez cette [vidéo de démonstration](https://channel9.msdn.com/Events/IT-Camps/IT-Camps-On-Demand-Windows-Server-2012/DEMO-Continuously-Available-File-Shares).

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les limites du système StorSimple](storsimple-8000-limits.md).
* [Découvrez comment déployer votre solution StorSimple](storsimple-8000-deployment-walkthrough-u2.md).

<!--Reference links-->
[1]: /previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731844(v=ws.10)