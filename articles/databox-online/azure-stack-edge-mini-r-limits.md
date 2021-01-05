---
title: Limites d’Azure Stack Edge Mini R | Microsoft Docs
description: Décrit les limites système et les tailles recommandées pour Azure Stack Edge Mini R.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: alkohli
ms.openlocfilehash: 6f01d62d1d11f5ff90661482ffd2db112657eee5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465506"
---
# <a name="azure-stack-edge-mini-r-limits"></a>Limites d’Azure Stack Edge Mini R


Tenez compte de ces limites quand vous déployez et utilisez votre solution Azure Stack Edge Mini R.

## <a name="azure-stack-edge-service-limits"></a>Limites du service Azure Stack Edge

[!INCLUDE [azure-stack-edge-gateway-service-limits](../../includes/azure-stack-edge-gateway-service-limits.md)]

## <a name="azure-stack-edge-mini-r-device-limits"></a>Limites de l’appareil Azure Stack Edge Mini R

Le tableau suivant décrit les limites de l’appareil Azure Stack Edge Mini R.

| Description | Limite|
|---|---:|
|Non. de fichiers par appareil | 100 millions <!--check with devs-->|
|Non. de partages par conteneur | 1|
|Nombre maximal de points de terminaison de partage et de points de terminaison REST par appareil| 24 |
|Nombre maximal de comptes de stockage hiérarchisé par appareil| 24|
|Taille de fichier maximale écrite sur un partage| 500 Go|
|Nombre maximal de groupes de ressources par appareil| 800|

## <a name="azure-storage-limits"></a>Limites du stockage Azure

[!INCLUDE [azure-stack-edge-gateway-storage-limits](../../includes/azure-stack-edge-gateway-storage-limits.md)]

## <a name="data-upload-caveats"></a>Mises en garde relatives au chargement de données

[!INCLUDE [azure-stack-edge-gateway-storage-data-upload-caveats](../../includes/azure-stack-edge-gateway-storage-data-upload-caveats.md)]

## <a name="azure-storage-account-size-and-object-size-limits"></a>Limites de taille d’objet et de compte de stockage Azure

[!INCLUDE [azure-stack-edge-gateway-storage-acct-limits](../../includes/azure-stack-edge-gateway-storage-acct-limits.md)]

## <a name="azure-object-size-limits"></a>Limites de taille des objets Azure

[!INCLUDE [azure-stack-edge-gateway-storage-object-limits](../../includes/azure-stack-edge-gateway-storage-object-limits.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Stack Edge](azure-stack-edge-gpu-deploy-prep.md)
