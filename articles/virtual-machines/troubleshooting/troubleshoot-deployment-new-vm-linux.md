---
title: Résoudre les problèmes de déploiement de machines virtuelles Linux | Microsoft Docs
description: Résoudre les problèmes de déploiement Resource Manager liés à la création d’une machine virtuelle Linux dans Azure
services: virtual-machines-linux, azure-resource-manager
documentationcenter: ''
author: DavidCBerry13
manager: gwallace
editor: ''
tags: top-support-issue, azure-resource-manager
ms.assetid: 906a9c89-6866-496b-b4a4-f07fb39f990c
ms.service: virtual-machines-linux
ms.workload: na
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 09/09/2016
ms.author: daberry
ms.openlocfilehash: f48963a4d18e80cb67bfbbdc532d34f89b8b5d8a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87028445"
---
# <a name="troubleshoot-resource-manager-deployment-issues-with-creating-a-new-linux-virtual-machine-in-azure"></a>Résoudre les problèmes de déploiement Resource Manager liés à la création d’une machine virtuelle Linux dans Azure
[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

[!INCLUDE [support-disclaimer](../../../includes/support-disclaimer.md)]

## <a name="top-issues"></a>Problèmes principaux
[!INCLUDE [support-disclaimer](../../../includes/virtual-machines-linux-troubleshoot-deploy-vm-top.md)]

Pour toute autre question ou problèmes concernant le déploiement de machine virtuelle, consultez [Troubleshoot deploying Linux virtual machine issues in Azure](troubleshoot-deploy-vm-linux.md) (Résolution des problèmes de déploiement de la machine virtuelle Linux dans Azure).

## <a name="collect-activity-logs"></a>Collecte des journaux d’activité
Pour résoudre les problèmes, commencez par collecter les journaux d’activité afin d’identifier l’erreur associée au problème. Les liens suivants contiennent des informations détaillées sur la marche à suivre.

[Voir les opérations de déploiement](../../azure-resource-manager/templates/deployment-history.md)

[Afficher les journaux d’activité pour gérer les ressources Azure](../../azure-resource-manager/management/view-activity-logs.md)

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[!INCLUDE [virtual-machines-linux-troubleshoot-deployment-new-vm-table](../../../includes/virtual-machines-linux-troubleshoot-deployment-new-vm-table.md)]

**O :** si le système d’exploitation est de type Linux généralisé et s’il est chargé et/ou capturé avec le paramètre généralisé, il n’y aura aucune erreur. De même, si le système d’exploitation est de type Linux spécialisé et qu’il est chargé et/ou capturé avec le paramètre spécialisé, il n’y aura aucune erreur.

**Erreurs de chargement :**

**N<sup>1</sup> :** si le système d’exploitation est de type Linux généralisé et qu’il est chargé comme spécialisé, vous obtiendrez une erreur de délai d’attente d’approvisionnement, car la machine virtuelle est bloquée à l’étape d’approvisionnement.

**N<sup>2</sup> :** si le système d’exploitation est de type Linux spécialisé et qu’il est chargé comme généralisé, vous obtiendrez une erreur d’échec d’approvisionnement, car la nouvelle machine virtuelle s’exécute avec le nom d’ordinateur, le nom d’utilisateur et le mot de passe d’origine.

**Résolution :**

pour corriger ces deux erreurs, chargez le disque dur virtuel d’origine, disponible en mode local, avec le même paramétrage que pour le système d’exploitation (généralisé/spécialisé). Pour effectuer un chargement de type généralisé, n’oubliez pas de commencer par exécuter -deprovision.

**Erreurs de capture :**

**N<sup>3</sup> :** si le système d’exploitation est de type Linux généralisé et qu’il est capturé comme spécialisé, vous obtiendrez une erreur de délai d’attente d’approvisionnement, car la machine virtuelle d’origine ne peut pas être utilisée tant qu’elle est identifiée comme généralisée.

**N<sup>4</sup> :** si le système d’exploitation est de type Linux spécialisé et qu’il est capturé comme généralisé, vous obtiendrez une erreur d’échec d’approvisionnement, car la nouvelle machine virtuelle s’exécute avec le nom d’ordinateur, le nom d’utilisateur et le mot de passe d’origine. En outre, la machine virtuelle d’origine n’est pas utilisable tant qu’elle est marquée comme spécialisée.

**Résolution :**

Pour corriger ces deux erreurs, supprimez l’image actuelle du portail, et [effectuez une nouvelle capture à partir des disques durs virtuels en cours](../linux/capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json), avec le même paramétrage que celui du système d’exploitation (généralisé/spécialisé).

## <a name="issue-custom-gallery-marketplace-image-allocation-failure"></a>Problème : Image personnalisée/de la galerie/de la Place de marché ; échec d’allocation
Cette erreur se produit lorsque la nouvelle demande de la machine virtuelle est épinglée à un cluster qui ne prend pas en charge la taille de machine virtuelle requise ou qui n’a pas d’espace libre suffisant pour prendre en charge la demande.

**Cause 1 :** Le cluster ne peut pas prendre en charge la taille de machine virtuelle demandée.

**Résolution 1 :**

* Relancez la requête en utilisant une taille inférieure pour la machine virtuelle.
* Si la taille de la machine virtuelle requise ne peut pas être modifiée :
  * Arrêtez toutes les machines virtuelles dans le groupe à haute disponibilité.
    Cliquez sur **Groupes de ressources** > *votre groupe de ressources* > **Ressources** > *votre groupe à haute disponibilité* > **Machines virtuelles** > *votre machine virtuelle* > **Arrêter**.
  * Une fois que toutes les machines virtuelles sont arrêtées, créez une machine virtuelle à la taille souhaitée.
  * Démarrez la nouvelle machine virtuelle en premier, puis sélectionnez chacune des machines virtuelles arrêtées et cliquez sur **Démarrer**.

**Cause 2 :** Le cluster n’a pas de ressources libres.

**Résolution 2 :**

* Relancez la demande ultérieurement.
* Si la nouvelle machine virtuelle peut faire partie d’un autre groupe à haute disponibilité
  * Créez une machine virtuelle dans un autre groupe à haute disponibilité (dans la même région).
  * Ajoutez la nouvelle machine virtuelle au même réseau virtuel.

## <a name="next-steps"></a>Étapes suivantes
Si vous rencontrez des problèmes lorsque vous démarrez une machine virtuelle Linux arrêtée ou que vous redimensionnez une machine virtuelle Linux existante dans Azure, consultez [Résoudre les problèmes de déploiement Resource Manager liés au redémarrage ou au redimensionnement d’une machine virtuelle Linux existante dans Azure](./troubleshoot-deploy-vm-linux.md?toc=/azure/virtual-machines/linux/toc.json).
