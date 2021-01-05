---
title: Présentation de la capture de paquets dans Azure Network Watcher | Microsoft Docs
description: Cette page fournit une vue d’ensemble de la fonctionnalité de capture de paquets de Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 1c458508dbf8d98349ec8549af32e3dd48bbd09b
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966426"
---
# <a name="introduction-to-variable-packet-capture-in-azure-network-watcher"></a>Présentation de la capture de paquets variable dans Azure Network Watcher

La fonctionnalité de capture de paquets variable de Network Watcher vous permet de créer des sessions de capture de paquets afin d’effectuer le suivi du trafic en direction et en provenance d’une machine virtuelle. La capture des paquets permet de diagnostiquer les anomalies réseau de manière proactive et réactive. Elle permet aussi de collecter des statistiques réseau, d’obtenir des informations sur les intrusions, de déboguer des communications client-serveur, etc.

La capture de paquets est une extension de machine virtuelle qui est démarrée à distance par le biais de Network Watcher. Cette fonctionnalité allège la tâche d’exécution manuelle d’une capture de paquets sur la machine virtuelle souhaitée et permet ainsi d’économiser un temps précieux. La capture de paquets peut être déclenchée par l’intermédiaire du portail, de PowerShell, de l’interface de ligne de commande ou de l’API REST. Les alertes de machine virtuelle constituent un exemple de mode de déclenchement de la capture de paquets. Des filtres sont fournis pour la session de capture afin de vous permettre de capturer uniquement le trafic que vous souhaitez analyser. Ces filtres reposent sur des informations à 5 tuples (protocole, adresse IP locale, adresse IP distante, port local et port distant). Les données capturées sont stockées dans le disque local ou dans un objet blob de stockage. Il existe une limite de 10 sessions de capture de paquets par région par abonnement. Cette limite s’applique uniquement aux sessions, mais pas aux fichiers de capture de paquets enregistrés localement sur la machine virtuelle ou dans un compte de stockage.

> [!IMPORTANT]
> La capture de paquets requiert une extension de machine virtuelle `AzureNetworkWatcherExtension`. Pour installer l’extension sur une machine virtuelle Windows, consultez la page [Azure Network Watcher Agent virtual machine extension for Windows](../virtual-machines/extensions/network-watcher-windows.md) (Extension de machine virtuelle d’agent Azure Network Watcher pour Windows). Pour une machine virtuelle Linux, consultez la page [Azure Network Watcher Agent virtual machine extension for Linux](../virtual-machines/extensions/network-watcher-linux.md) (Extension de machine virtuelle d’agent Azure Network Watcher pour Linux).

Pour capturer uniquement les informations qui vous intéressent, vous disposez des options ci-après dans le cadre d’une session de capture de paquets :

**Configuration de la capture**

|Propriété|Description|
|---|---|
|**Nombre maximal d’octets par paquet (octets)** | Nombre d’octets capturés dans chaque paquet. Si ce champ est vide, tous les octets sont capturés. Nombre d’octets capturés dans chaque paquet. Si ce champ est vide, tous les octets sont capturés. Si seul l’en-tête IPv4 vous intéresse, indiquez 34 dans ce champ. |
|**Nombre maximal d’octets par session (octets)** | Nombre total d’octets capturés. Une fois que cette valeur est atteinte, la session prend fin.|
|**Délai imparti (secondes)** | Contrainte de temps définie pour la session de capture de paquets. La valeur par défaut est de 18 000 secondes (5 heures).|

**Filtrage (facultatif)**

|Propriété|Description|
|---|---|
|**Protocole** | Protocole permettant de filtrer la capture de paquets. Valeurs possibles : TCP, UDP et Tous.|
|**Adresse IP locale** | Cette valeur filtre la capture de paquets sur ceux dont l’adresse IP locale correspond à cette valeur de filtre.|
|**Port local** | Cette valeur filtre la capture de paquets sur ceux dont le port local correspond à cette valeur de filtre.|
|**Adresse IP distante** | Cette valeur filtre la capture de paquets sur ceux dont l’adresse IP distante correspond à cette valeur de filtre.|
|**Port distant** | Cette valeur filtre la capture de paquets sur ceux dont le port distant correspond à cette valeur de filtre.|

### <a name="next-steps"></a>Étapes suivantes

Découvrez comment gérer les captures de paquets par le biais du portail en consultant l’article [Manage packet capture in the Azure portal](network-watcher-packet-capture-manage-portal.md) (Gérer les captures de paquets dans le Portail Azure), ou par l’intermédiaire de PowerShell en consultant l’article [Manage Packet Capture with PowerShell](network-watcher-packet-capture-manage-powershell.md) (Gérer les captures de paquets à l’aide de PowerShell).

Apprenez à créer des captures de paquets proactives reposant sur des alertes de machine virtuelle en consultant l’article [Create an alert triggered packet capture (Créer une capture de paquets déclenchée par alerte)](network-watcher-alert-triggered-packet-capture.md).

<!--Image references-->
[1]: ./media/network-watcher-packet-capture-overview/figure1.png