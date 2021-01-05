---
title: Démarrage du système d’exploitation – L’ordinateur a redémarré de manière inattendue ou a rencontré une erreur inattendue
description: Cet article décrit les étapes de résolution des problèmes lorsque la machine virtuelle redémarre inopinément ou rencontre une erreur inattendue lors de l’installation de Windows.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 1d249a4e-71ba-475d-8461-31ff13e57811
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 06/22/2020
ms.author: v-mibufo
ms.openlocfilehash: cfeb040893ae2be5842959ed8458bd713bebe6ee
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96512135"
---
# <a name="os-start-up--computer-restarted-unexpectedly-or-encountered-an-unexpected-error"></a>Démarrage du système d’exploitation – L’ordinateur a redémarré de manière inattendue ou a rencontré une erreur inattendue

Cet article décrit les étapes de résolution des problèmes lorsque la machine virtuelle redémarre inopinément ou rencontre une erreur lors de l’installation de Windows.

## <a name="symptom"></a>Symptôme

Quand vous utilisez les [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, vous pouvez voir la capture d’écran montrant l’échec de l’installation de Windows avec l’erreur suivante :

**L’ordinateur a redémarré de manière inattendue ou a rencontré une erreur inattendue. L’installation de Windows ne peut pas continuer. Pour installer Windows, cliquez sur « OK » pour redémarrer l’ordinateur, puis redémarrez l’installation.**

![Erreur lors de l’installation de Windows : L’ordinateur a redémarré de manière inattendue ou a rencontré une erreur inattendue. L’installation de Windows ne peut pas continuer. Pour installer Windows, cliquez sur « OK » pour redémarrer l’ordinateur, puis redémarrez l’installation.](./media/unexpected-restart-error-during-vm-boot/1.png)
 
![Erreur lors du démarrage des services par le programme d’installation de Windows : L’ordinateur a redémarré de manière inattendue ou a rencontré une erreur inattendue. L’installation de Windows ne peut pas continuer. Pour installer Windows, cliquez sur « OK » pour redémarrer l’ordinateur, puis redémarrez l’installation.](./media/unexpected-restart-error-during-vm-boot/2.png)

## <a name="cause"></a>Cause

L’ordinateur tente d’effectuer un démarrage initial d’une [image généralisée](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation), mais rencontre des problèmes dus au traitement d’un fichier de réponses personnalisé (Unattend.Xml). **Les fichiers de réponses personnalisés ne sont pas pris en charge dans Azure**. 

Le fichier de réponses est un fichier XML spécial contenant des définitions et des valeurs pour les paramètres de configuration que vous souhaitez automatiser lors de l’installation du système d’exploitation Windows Server. Les options de configuration incluent des instructions sur la façon de partitionner des disques, l’emplacement où trouver l’image Windows à installer, les clés de produit à appliquer et d’autres commandes à exécuter.

Ici aussi, les fichiers de réponses personnalisés ne sont pas pris en charge dans Azure. Par conséquent, cette situation se produit quand une image a été préparée pour une utilisation dans Azure, mais que vous avez spécifié un fichier Unattend.xml personnalisé en utilisant **SYSPREP** avec un indicateur similaire à la commande suivante :

`sysprep /oobe /generalize /unattend:<your file’s name> /shutdown`

Dans Azure, utilisez l’option **Entrer en mode OOBE (Out-of-Box Experience)** dans l’**interface graphique utilisateur de l’outil de préparation du système**, ou utilisez `sysprep /oobe` au lieu du fichier Unattend.xml.

Ce problème se produit le plus souvent quand vous utilisez sysprep avec une machine virtuelle locale pour charger une machine virtuelle généralisée sur Azure. Dans ce cas, vous pouvez également vous intéresser à la manière de charger correctement une machine virtuelle généralisée.

## <a name="solution"></a>Solution

### <a name="do-not-use-unattendxml"></a>Ne pas utiliser Unattend.xml

Pour résoudre ce problème, suivez les [conseils d’Azure sur la préparation/capture d’image](../windows/upload-generalized-managed.md) et préparez une nouvelle image généralisée. Pendant l’exécution de sysprep, **n’utilisez pas l’indicateur `/unattend:<your file’s name>`** . À la place, utilisez uniquement les indicateurs ci-dessous :

`sysprep /oobe /generalize /shutdown`

- Out-of-box-experience (OOBE) est le paramètre pris en charge pour les machines virtuelles Azure.

Vous pouvez également utiliser l’**interface graphique utilisateur de l’outil de préparation du système** pour accomplir la même tâche que la commande ci-dessus en sélectionnant les options ci-dessous :

- Ouvrir Out-of-Box-Experience
- Généraliser
- Arrêter
 
![Fenêtre de l’outil de préparation du système avec les options OOBE, Généraliser et Arrêter sélectionnées.](./media/unexpected-restart-error-during-vm-boot/3.png)
