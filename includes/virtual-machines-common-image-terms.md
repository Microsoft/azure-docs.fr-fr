---
title: Fichier include
description: Fichier include
services: virtual-machines-windows, virtual-machines-linux
author: cynthn
ms.service: multiple
ms.topic: include
ms.date: 10/09/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 98d765e2f6909f00f8dfe76d06aef017aad67adf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "71174949"
---
## <a name="terminology"></a>Terminologie

Une image de Place de Marché dans Azure a les attributs suivants :

* **Éditeur** : L’organisation qui a créé l’image. Exemples : Canonical, MicrosoftWindowsServer
* **Offre** : Nom du groupe d’images associées, créé par un éditeur. Exemples : UbuntuServer, WindowsServer
* **SKU** : L’instance d’une offre, par exemple une version majeure d’un logiciel. Exemples : 18.04-LTS, 2019-Datacenter
* **Version** : Numéro de version d’une référence SKU d’image. 

Pour identifier une image de Place de Marché lorsque vous déployez une machine virtuelle par programmation, indiquez ces valeurs individuellement en tant que paramètres. Certains outils acceptent l’*URN* de l’image, qui combine ces valeurs en les séparant par un deux-points (:) : *Éditeur*:*Offre*:*Référence SKU*:*Version*. Dans l’URN, vous pouvez remplacer le numéro de version par la valeur « latest », qui permet de sélectionner la dernière version de l’image. 

Si l’éditeur d’images fournit des conditions de licences et d’achat supplémentaires, vous devez accepter les termes du contrat et activer le déploiement par programmation. Vous devez également indiquer les paramètres du *forfait* lors du déploiement d’une machine virtuelle par programmation. Consultez [Déployer une image avec les termes du contrat de la Place de Marché](#deploy-an-image-with-marketplace-terms).
