---
title: Fichier Include
description: Fichier Include
services: virtual-machines-windows, virtual-machines-linux
author: cynthn
ms.service: multiple
ms.topic: include
ms.date: 10/09/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 8e0e549f88caf4a541642bab77faf54b5e536b29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "71174948"
---
## <a name="deploy-an-image-with-marketplace-terms"></a>Déployer une image avec les termes du contrat de la Place de Marché

Certaines images de machine virtuelle dans la Place de marché Microsoft Azure ont des conditions de licence et d’achat supplémentaires que vous devez accepter pour pouvoir les déployer par programme.  

Pour déployer une machine virtuelle à partir d’une de ces images, vous devez accepter les conditions de l’image et effectuer un déploiement par programmation. Cette opération n’est à effectuer qu’une seule fois par abonnement. Ensuite, chaque fois que vous déployez une machine virtuelle par programmation à partir de l’image, vous devez également spécifier les paramètres du *forfait*.

Les sections suivantes décrivent diverses opérations :

* Déterminer si une image de la Place de marché a des conditions de licence supplémentaires 
* Accepter les termes du contrat par programmation
* Fournir des paramètres de forfait lorsque vous déployez une machine virtuelle par programmation

