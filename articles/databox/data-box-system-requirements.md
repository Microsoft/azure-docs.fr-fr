---
title: Configuration système requise de Microsoft Azure Data Box | Microsoft Docs
description: Découvrez la configuration système importante qui est demandée pour votre solution Azure Data Box et pour les clients accédant au Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 10/02/2020
ms.author: alkohli
ms.openlocfilehash: 5f1623ef4dde59e816e3afe5a5f5894c49469580
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91767857"
---
# <a name="azure-data-box-system-requirements"></a>Conditions requises pour le système Data Box

Cet article décrit les exigences système les plus importantes concernant Microsoft Azure Data Box et les clients qui se connectent à Data Box. Nous vous recommandons de lire attentivement les informations suivantes avant de déployer votre Data Box, puis de vous y référer si nécessaire pendant le déploiement, et une fois qu’il est en cours d’exécution.

Les conditions requises sont les suivantes :

* **Exigences logicielles :** Pour les hôtes qui se connectent à Data Box, indique les systèmes d’exploitation pris en charge, les protocoles de transfert de fichiers, les comptes de stockage, les types de stockage, ainsi que les navigateurs pour l’interface utilisateur web locale.
* **Configuration réseau requise :** Pour Data Box, indique les exigences de connexion réseau et de port pour une utilisation optimale de Data Box.


## <a name="software-requirements"></a>Configuration logicielle requise

La configuration logicielle requise inclut les systèmes d’exploitation pris en charge, les protocoles de transfert de fichiers, les comptes de stockage, les types de stockage, ainsi que les navigateurs pour l’interface utilisateur web locale.

### <a name="supported-operating-systems-for-clients"></a>Systèmes d’exploitation pris en charge pour les clients

[!INCLUDE [data-box-supported-os-clients](../../includes/data-box-supported-os-clients.md)]


### <a name="supported-file-transfer-protocols-for-clients"></a>Protocoles de transfert de fichiers pris en charge pour les clients

[!INCLUDE [data-box-supported-file-systems-clients](../../includes/data-box-supported-file-systems-clients.md)]

> [!IMPORTANT] 
> La connexion à des partages de Data Box n’est pas prise en charge via REST pour les ordres d’exportation. 

### <a name="supported-storage-accounts"></a>Comptes de stockage pris en charge

[!INCLUDE [data-box-supported-storage-accounts](../../includes/data-box-supported-storage-accounts.md)]

### <a name="supported-storage-types"></a>Types de stockage pris en charge

[!INCLUDE [data-box-supported-storage-types](../../includes/data-box-supported-storage-types.md)]

### <a name="supported-web-browsers"></a>Navigateurs web pris en charge

[!INCLUDE [data-box-supported-web-browsers](../../includes/data-box-supported-web-browsers.md)]

## <a name="networking-requirements"></a>Configuration requise du réseau

Votre centre de données doit avoir un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si une connexion 10 GbE n’est pas disponible, une liaison de données 1 GbE peut être utilisée pour copier des données. Toutefois, cela aura un impact sur la vitesse de copie.

### <a name="port-requirements"></a>Exigences de port

Le tableau ci-dessous répertorie les ports qui doivent être ouverts dans votre pare-feu pour autoriser le trafic SMB ou NFS. Dans ce tableau, *Entrée* (ou *entrant*) indique que les requêtes clientes qui accèdent à votre appareil sont des requêtes entrantes. *Sortie* (ou *sortant*) indique que votre appareil Data Box envoie des données vers l’extérieur, hors déploiement, par exemple, vers Internet.

[!INCLUDE [data-box-port-requirements](../../includes/data-box-port-requirements.md)]


## <a name="next-steps"></a>Étapes suivantes

* [Déployer votre Azure Data Box](data-box-deploy-ordered.md)
