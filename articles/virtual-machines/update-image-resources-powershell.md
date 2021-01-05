---
title: Répertorier, mettre à jour et supprimer des ressources d’image à l’aide de PowerShell
description: Répertoriez, mettez à jour et supprimez des ressources d’image dans votre galerie d’images partagées à l’aide d’Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/27/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: c6bc4fea05f79c446f318eb4cbc86beb160ea108
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82793965"
---
# <a name="list-update-and-delete-image-resources-using-powershell"></a>Répertorier, mettre à jour et supprimer des ressources d’image à l’aide de PowerShell 

Vous pouvez gérer vos ressources de galerie d’images partagées à l’aide d’Azure PowerShell.

[!INCLUDE [virtual-machines-common-gallery-list-ps.md](../../includes/virtual-machines-common-gallery-list-ps.md)]

[!INCLUDE [virtual-machines-common-shared-images-update-delete-ps](../../includes/virtual-machines-common-shared-images-update-delete-ps.md)]

## <a name="next-steps"></a>Étapes suivantes

[Générateur d’image Azure (préversion)](./linux/image-builder-overview.md) permet d’automatiser la création d’une version de l’image. Vous pouvez même l’utiliser pour mettre à jour et [créer une nouvelle version de l’image à partir d’une image existante](./linux/image-builder-gallery-update-image-version.md).