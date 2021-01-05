---
title: Limites d’Azure Stack Edge Pro | Microsoft Docs
description: Découvrez les limites et les tailles recommandées lorsque vous déployez et exploitez Azure Stack Edge Pro, notamment les limites de service, les limites d’appareil et les limites de stockage.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/12/2020
ms.author: alkohli
ms.openlocfilehash: f1bb5662b48765c08ec58d1f7fa9d341de4937bc
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91992765"
---
# <a name="azure-stack-edge-pro-limits"></a>Limites d’Azure Stack Edge Pro

Tenez compte de ces limites quand vous déployez et utilisez votre solution Microsoft Azure Stack Edge Pro. 

## <a name="azure-stack-edge-service-limits"></a>Limites du service Azure Stack Edge

[!INCLUDE [data-box-edge-gateway-service-limits](../../includes/data-box-edge-gateway-service-limits.md)]

## <a name="azure-stack-edge-device-limits"></a>Limites de l’appareil Azure Stack Edge

Le tableau suivant décrit les limites de l’appareil Azure Stack Edge Pro. 

Le tableau suivant décrit les limites de l’appareil Azure Stack Edge.

| Description | Valeur |
|---|---|
|Non. de fichiers par appareil |100 millions |
|Non. de partages par conteneur |1 |
|Nombre maximal de points de terminaison de partage et de points de terminaison REST par appareil| 24 |
|Nombre maximal de comptes de stockage hiérarchisé par appareil| 24|
|Taille de fichier maximale écrite sur un partage| 5 To |
|Nombre maximal de groupes de ressources par appareil| 800 |

## <a name="azure-storage-limits"></a>Limites du stockage Azure

[!INCLUDE [data-box-edge-gateway-storage-limits](../../includes/data-box-edge-gateway-storage-limits.md)]

## <a name="data-upload-caveats"></a>Mises en garde relatives au chargement de données

[!INCLUDE [data-box-edge-gateway-storage-data-upload-caveats](../../includes/data-box-edge-gateway-storage-data-upload-caveats.md)]

## <a name="azure-storage-account-size-and-object-size-limits"></a>Limites de taille d’objet et de compte de stockage Azure

[!INCLUDE [data-box-edge-gateway-storage-acct-limits](../../includes/data-box-edge-gateway-storage-acct-limits.md)]


## <a name="azure-object-size-limits"></a>Limites de taille des objets Azure

[!INCLUDE [data-box-edge-gateway-storage-object-limits](../../includes/data-box-edge-gateway-storage-object-limits.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)
