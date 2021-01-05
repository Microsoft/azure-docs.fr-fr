---
title: Vue d’ensemble de la création d’images Linux pour Azure
description: Comment apporter vos images de machine virtuelle Linux ou créer des images à utiliser dans Azure.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: overview
ms.workload: infrastructure
ms.date: 06/22/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: e364578cdec8696688cf19e14fd0529f1ca3fbb3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842615"
---
# <a name="bringing-and-creating-linux-images-in-azure"></a>Intégration et création d’images Linux dans Azure

Cette vue d’ensemble couvre les concepts de base relatifs à la création d’images ainsi qu’à la génération et à l’utilisation d’images Linux dans Azure. Avant d’intégrer une image personnalisée dans Azure, vous devez connaître les types et les options disponibles.

Cet article aborde les conditions et les points de décision relatifs aux images et explique les concepts clés afin que vous puissiez comprendre et être en mesure de créer vos propres images personnalisées selon votre spécification.

## <a name="difference-between-managed-disks-and-images"></a>Différence entre les disques managés et les images


Azure vous permet d’intégrer un disque dur virtuel à la plateforme, à utiliser en tant que [disque managé](../faq-for-disks.md#managed-disks) ou source pour une image. 

Les disques managés Azure sont des disques durs virtuels uniques. Vous pouvez soit utiliser un disque dur virtuel existant et créer un disque managé à partir de celui-ci, soit créer un disque managé vide à partir de zéro. Vous pouvez créer des machines virtuelles à partir de disques managés en associant le disque à la machine virtuelle, mais vous pouvez uniquement utiliser un disque dur virtuel avec une seule machine virtuelle. Vous ne pouvez pas modifier les propriétés du système d’exploitation. Azure tente simplement d’activer la machine virtuelle et de démarrer à l’aide de ce disque. 

Les images Azure peuvent être composées de plusieurs disques de système d’exploitation et disques de données. Lorsque vous utilisez une image managée pour créer une machine virtuelle, la plateforme effectue une copie de l’image et l’utilise pour créer la machine virtuelle, de sorte que l’image managée prend en charge la réutilisation de la même image pour plusieurs machines virtuelles. Azure fournit également des fonctionnalités de gestion avancées pour les images, telles que la réplication globale et la gestion de versions via [Shared Image Gallery](shared-image-galleries.md). 



## <a name="generalized-and-specialized"></a>Images généralisées et spécialisées

Azure offre deux principaux types d’images, généralisées et spécialisées. Les termes généralisées et spécialisées sont des termes Windows à l’origine, qui ont migré dans Azure. Ces types définissent la façon dont la plateforme gère la machine virtuelle quand elle l’active. Les deux types présentent des avantages et des inconvénients ainsi que des prérequis. Avant de commencer, vous devez connaître le type d’image dont vous aurez besoin. Vous trouverez ci-dessous un résumé des scénarios et du type que vous devez choisir :

| Scénario      | Type d’image  | Options de stockage |
| ------------- |:-------------:| :-------------:| 
| Créer une image qui peut être configurée pour une utilisation par plusieurs machines virtuelles, et je peux définir le nom d’hôte, ajouter un utilisateur administrateur et effectuer d’autres tâches lors du premier démarrage. | Généralisée | Shared Image Gallery ou images managées autonomes |
| Créer une image à partir d’un instantané de machine virtuelle ou d’une sauvegarde | Spécialisée |Shared Image Gallery ou disque managé |
| Créer rapidement une image qui n’a pas besoin de configuration pour la création de plusieurs machines virtuelles |Spécialisée |Galerie d’images partagées |


### <a name="generalized-images"></a>Images généralisées

Une image généralisée est une image qui requiert la finalisation de la configuration au premier démarrage. Par exemple, lors du premier démarrage, vous définissez le nom d’hôte, l’utilisateur administrateur et d’autres configurations spécifiques à la machine virtuelle. Cela est utile quand vous voulez que l’image soit réutilisée plusieurs fois et que vous souhaitez passer des paramètres lors de la création. Si l’image généralisée contient l’agent Azure, l’agent traite les paramètres et signale à la plateforme que la configuration initiale est terminée. Ce processus porte le nom de [provisionnement](./provisioning.md). 

Le provisionnement requiert qu’un provisionneur soit inclus dans l’image. Il existe deux provisionneurs :
- [Agent Linux Azure](../extensions/agent-linux.md)
- [cloud-init](./using-cloud-init.md)

Ce sont des [prérequis](./create-upload-generic.md) pour la création d’une image.


### <a name="specialized-images"></a>Images spécialisées
Il s’agit d’images entièrement configurées et ne nécessitant pas de paramètres de machine virtuelle ni spéciaux. La plateforme active simplement la machine virtuelle et vous devez gérer l’unicité au sein de la machine virtuelle, comme la définition d’un nom d’hôte, afin d’éviter les conflits DNS sur le même réseau virtuel. 

Les agents de provisionnement ne sont pas requis pour ces images. Toutefois, vous pouvez avoir besoin de fonctionnalités de gestion des extensions. Vous pouvez installer l’agent Linux, mais désactiver l’option de provisionnement. Même si vous n’avez pas besoin d’un agent de provisionnement, l’image doit répondre aux [prérequis](./create-upload-generic.md) pour les images Azure.


## <a name="image-storage-options"></a>Options de stockage des images
Lors de l’intégration de votre image Linux, vous disposez de deux options :

- Images managées pour la création simple de machines virtuelles dans un environnement de développement et de test.
- [Shared Image Gallery](shared-image-galleries.md) pour la création et le partage d’images à grande échelle.


### <a name="managed-images"></a>Images managées

Les images managées peuvent être utilisées pour créer plusieurs machines virtuelles, mais elles présentent de nombreuses limitations. Les images managées peuvent uniquement être créées à partir d’une source généralisée (machine virtuelle ou disque dur virtuel). Elles ne peuvent être utilisées que pour créer des machines virtuelles dans la même région et ne peuvent pas être partagées entre les abonnements et les locataires.

Les images managées peuvent être utilisées pour les environnements de développement et de test, où vous avez besoin de deux images généralisées simples à utiliser dans une seule région et un seul abonnement. 

### <a name="azure-shared-image-gallery-sig"></a>Azure Shared Image Gallery

Les galeries [Shared Image Gallery](shared-image-galleries.md) sont recommandées pour la création, la gestion et le partage d’images à grande échelle. Ces galeries vous permettent de structurer et d’organiser vos images.  

- Prise en charge des images généralisées et spécialisées.
- Prise en charge des images de génération 1 et 2.
- Réplication globale des images.
- la gestion de versions et le regroupement d’images pour faciliter la gestion ;
- Images hautement disponibles avec stockage redondant dans une zone (ZRS) dans les régions qui prennent en charge les zones de disponibilité. Le stockage redondant interzone (ZRS) offre une meilleure résilience en cas de défaillances de zones.
- Le partage entre différents abonnements, voire entre locataires Active Directory (AD), à l’aide du contrôle d’accès en fonction du rôle (RBAC) Azure.
- la mise à l’échelle de vos déploiements avec des réplicas d’image dans chaque région.

Globalement, vous créez une galerie Shared Image Gallery, qui est constituée des éléments suivants :
- Définitions d’images : il s’agit de conteneurs qui contiennent des groupes d’images.
- Versions d’images : il s’agit des images réelles



## <a name="hyper-v-generation"></a>Hyper-V Génération

Azure prend en charge Hyper-V Génération 1 (Gen1) et Génération 2 (Gen2). Gen2 est la dernière génération et offre des fonctionnalités supplémentaires par rapport à Gen1, par exemple la mémoire augmentée, Intel SGX (Intel Software Guard Extensions) et la mémoire persistante virtuelle (vPMEM). Les machines virtuelles locales de 2e génération présentent des fonctionnalités qui ne sont pas encore prises en charge dans Azure. Pour plus d’informations, consultez la section Fonctionnalités et capacités. Pour plus d’informations, consultez cet [article](../generation-2.md). Créez des images Gen2 si vous avez besoin des fonctionnalités supplémentaires.

Si vous avez encore besoin de créer votre propre image, vérifiez qu’elle répond aux [prérequis pour l’image](./create-upload-generic.md) et chargez-la sur Azure. Exigences spécifiques à la distribution :


- [Distributions CentOS](create-upload-centos.md)
- [Debian Linux](debian-create-upload-vhd.md)
- [Flatcar Container Linux](flatcar-create-upload-vhd.md)
- [Oracle Linux](oracle-create-upload-vhd.md)
- [Red Hat Enterprise Linux](redhat-create-upload-vhd.md)
- [SLES et openSUSE](suse-create-upload-vhd.md)
- [Ubuntu](create-upload-ubuntu.md)


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer une galerie [Shared Image Gallery](tutorial-custom-images.md).