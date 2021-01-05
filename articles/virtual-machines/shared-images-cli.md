---
title: Créer des galeries d’images partagées avec l’interface de ligne de commande Azure
description: Dans cet article, vous allez apprendre à utiliser l’interface de ligne de commande Azure pour créer une image partagée d’une machine virtuelle dans Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: ff50f61fe159c518d488da9f9911e40f1ddb8ca3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87501455"
---
# <a name="create-a-shared-image-gallery-with-the-azure-cli"></a>Créer une galerie d’images partagées avec Azure CLI

Une [galerie d’images partagées](./linux/shared-image-galleries.md) simplifie considérablement le partage d’images personnalisées dans votre organisation. Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des configurations comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. 

Shared Image Gallery vous permet de partager vos images de machines virtuelles personnalisées avec d’autres utilisateurs. Choisissez les images à partager, les régions dans lesquelles vous souhaitez les rendre disponibles et les personnes avec lesquelles vous voulez les partager. 

[!INCLUDE [virtual-machines-common-shared-images-cli](../../includes/virtual-machines-common-shared-images-cli.md)]


## <a name="next-steps"></a>Étapes suivantes

Créez une version de l’image à partir d’une [machine virtuelle](image-version-vm-cli.md) ou d’une [image managée](image-version-managed-image-cli.md) à l’aide d’Azure CLI.

Pour plus d’informations sur les galeries d’images partagées, consultez la [vue d’ensemble](./linux/shared-image-galleries.md). Si vous rencontrez des problèmes, consultez [Résoudre les problèmes des galeries d’images partagées](troubleshooting-shared-images.md).
