---
title: Configuration système requise de Microsoft Azure Data Box Heavy | Microsoft Docs
description: En savoir plus sur les conditions logicielles et réseau requises pour votre solution Azure Data Box Heavy
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: article
ms.date: 07/03/2019
ms.author: alkohli
ms.openlocfilehash: 6097dba9193fce4c8158137b488080107881994a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84707750"
---
# <a name="azure-data-box-heavy-system-requirements"></a>Configuration requise pour Azure Data Box Heavy

Cet article décrit les principaux éléments de configuration système qui sont requis pour votre appareil Azure Data Box Heavy et pour les clients qui se connectent à cet appareil. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre Data Box Heavy, puis d’y revenir si nécessaire pendant le déploiement, et après pour son fonctionnement.

Les conditions requises sont les suivantes :

* **Configuration requise logicielle pour les hôtes se connectant à Data Box Heavy**: décrit les plateformes prises en charge, les navigateurs pour l’interface utilisateur web locale, les clients SMB et les exigences supplémentaires pour les hôtes qui peuvent se connecter à l’appareil Data Box.
* **Configuration requise réseau pour Data Box Heavy** : fournit des informations sur la configuration réseau nécessaire au fonctionnement optimal de l’appareil Data Box Heavy.

## <a name="software-requirements"></a>Configuration logicielle requise

La configuration logicielle requise inclut les informations sur les systèmes d’exploitation pris en charge, les navigateurs pris en charge pour l’interface utilisateur web locale et les clients SMB.

### <a name="supported-operating-systems-for-clients"></a>Systèmes d’exploitation pris en charge pour les clients

[!INCLUDE [data-box-supported-os-clients](../../includes/data-box-supported-os-clients.md)]

### <a name="supported-file-systems-for-linux-clients"></a>Systèmes de fichiers pris en charge pour les clients Linux

[!INCLUDE [data-box-supported-file-systems-clients](../../includes/data-box-supported-file-systems-clients.md)]

### <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

[!INCLUDE [data-box-supported-storage-accounts](../../includes/data-box-supported-storage-accounts.md)]

### <a name="supported-storage-types"></a>Types de stockage pris en charge

[!INCLUDE [data-box-supported-storage-types](../../includes/data-box-supported-storage-types.md)]

### <a name="supported-web-browsers"></a>Navigateurs web pris en charge

[!INCLUDE [data-box-supported-web-browsers](../../includes/data-box-supported-web-browsers.md)]

## <a name="networking-requirements"></a>Configuration requise du réseau

Votre centre de données doit avoir un réseau haut débit. Pour bénéficier de vitesses de copie supérieures, il est possible d’utiliser deux connexions 40 GbE (une par nœud) en parallèle. Si aucune connexion 40 GbE n’est disponible, nous vous recommandons d’avoir au moins deux connexions 10 GbE (une par nœud).

### <a name="port-requirements"></a>Exigences de port

Le tableau ci-dessous répertorie les ports qui doivent être ouverts dans votre pare-feu pour autoriser le trafic SMB ou NFS. Dans ce tableau, *entrée* ou *entrant* représente la direction à partir de laquelle les requêtes clientes entrantes accèdent à votre appareil. *Sortie* ou *sortant* désigne la direction dans laquelle votre appareil Data Box Heavy envoie des données de façon externe, au-delà du déploiement : par exemple, sortant vers Internet.

[!INCLUDE [data-box-port-requirements](../../includes/data-box-port-requirements.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Déployer votre Azure Data Box](data-box-deploy-ordered.md)
