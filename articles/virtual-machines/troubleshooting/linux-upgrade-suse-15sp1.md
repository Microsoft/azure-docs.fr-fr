---
title: Mettre à niveau une machine virtuelle Azure avec SUSE Linux Enterprise Server vers SUSE 15 SP1 | Microsoft Docs
description: Cet article fournit des étapes générales sur la façon d’utiliser le système de migration de la distribution SUSE afin de mettre à niveau SUSE Linux Enterprise Server vers SUSE 15 SP1 pour une machine virtuelle Azure.
services: virtual-machines-linux
documentationcenter: ''
author: amkarmak
manager: dcscontentpm
editor: ''
tags: virtual-machines
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.date: 05/21/2020
ms.author: arremana
ms.openlocfilehash: 7ed355f82b88f460ff4b372484a690f166a15550
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91359509"
---
# <a name="upgrade-azure-vm-with-sles-12-to-sles-15-sp1"></a>Mettre à niveau une machine virtuelle Azure avec SLES 12 vers SLES 15 SP1

Cet article fournit des étapes générales sur la façon de mettre à niveau SUSE Linux Enterprise server (SLES) 12 vers SLES 15 SP1 pour une machine virtuelle Azure. Pour plus d’informations, consultez [Using the SUSE Distribution Migration System](https://documentation.suse.com/suse-distribution-migration-system/1.0/single-html/distribution-migration-system/index.html) (Utilisation du système de migration de la distribution SUSE) et [SUSE Linux Enterprise Server 15 SP1 Upgrade Guide](https://documentation.suse.com/sles/15-SP1/single-html/SLES-upgrade/index.html#sec-update-preparation-update) (Guide de mise à niveau de SUSE Linux Enterprise Server 15 SP1).

## <a name="supported-upgrade-paths"></a>Chemins de mise à niveau pris en charge
La version SLES actuelle doit être SLES 12 SP4 ou 12 SP5 pour que vous puissiez passer à SLES 15 SP1.

![Capture d’écran du chemin de mise à niveau pris en charge](./media/linux-upgrade-suse-15sp1/upgrade-path.png)

## <a name="prerequisites"></a>Prérequis

- Planifiez l’activité de migration en fonction de la fenêtre de temps mort approuvée, car la machine virtuelle redémarre au cours de la migration.
- Avant l’activité de migration, effectuez une sauvegarde complète de la machine virtuelle.
- Si la sauvegarde n’est pas configurée, effectuez une sauvegarde de capture instantanée du disque du système d’exploitation.
- [Vérifiez si la machine virtuelle est de génération v1 ou de génération v2](#check-the-generation-version-for-a-vm).

## <a name="upgrade-from-suse-12-sp4-or-sp5-to-suse-15-sp1"></a>Mise à niveau de SUSE 12 SP4 ou SP5 vers SUSE 15 SP1

1. Installez la dernière version du package pour la machine virtuelle :

    ```
    zypper clean --all
    zypper refresh
    zypper update
    ```

2. Une fois l’installation terminée, redémarrez la machine virtuelle.

3. Vérifiez la version du noyau et du système d’exploitation. Assurez-vous que la version est SUSE 12 SP4 ou SUSE 12 SP5.

    ```
    uname -a
    cat /etc/os-release
    ```

4. Installez le package **suse-migration-sle15-activation**. Quand ce dernier est installé, un autre package **SLES15-Migration** est automatiquement installé en tant que package de dépendance. 

   ```
   zypper install suse-migration-sle15-activation
   ```

5. Une fois l’installation terminée, exécutez la commande `reboot` pour redémarrer la machine virtuelle.

6. Accédez au [portail Azure](https://portal.azure.com), sélectionnez la machine virtuelle, puis sélectionnez **Console série**. Vous verrez que le système s’arrête sur « reboot: Restarting system ». Ce processus prend environ 15 à 45 minutes. Pour les machines virtuelles de 2e génération, il est possible qu’il s’arrête sur l’écran « reboot: Restarting system ». Dans ce cas, attendez 45 minutes. Si le processus ne reprend toujours pas, accédez à la page **Vue d’ensemble** de la machine virtuelle dans le portail Azure, arrêtez la machine virtuelle, puis redémarrez-la.

     ![Capture d’écran des messages de la console série](./media/linux-upgrade-suse-15sp1/reboot-message.png)

8. Une fois le système redémarré avec le nouveau noyau, le message suivant s’affiche.

     ![Capture d’écran des messages de la console série après le redémarrage du système avec le nouveau noyau.](./media/linux-upgrade-suse-15sp1/output-message.png)
9. Vérifiez la version du noyau et du système d’exploitation afin de déterminer si le système est correctement mis à niveau.

    ```
    uname -a
    cat /etc/os-release
    ```

## <a name="check-the-generation-version-for-a-vm"></a>Vérifier la version de génération d’une machine virtuelle

Vous pouvez utiliser l’une des méthodes suivantes pour vérifier la version de la génération :

- Dans le terminal SLES, exécutez la commande `dmidecode | grep -i hyper`. S’il s’agit d’une machine virtuelle de génération v1, aucune sortie n’est retournée. Pour les machines virtuelles de générations v2, la sortie suivante s’affiche :

     ![Capture d’écran de la sortie pour une machine virtuelle de génération 2](./media/linux-upgrade-suse-15sp1/output-gen2.png)
- Dans le [portail Azure](https://portal.azure.com), accédez aux **Propriétés** de la machine virtuelle, puis cochez le champ **Génération de machine virtuelle**.
