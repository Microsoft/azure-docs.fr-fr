---
title: Attacher ou détacher un disque de données sur une machine virtuelle dans Azure DevTest Labs
description: Découvrez comment attacher ou détacher un disque de données sur une machine virtuelle dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 8eebfbda421233bcec780d441a4020acce740618
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91328511"
---
# <a name="attach-or-detach-a-data-disk-to-a-virtual-machine-in-azure-devtest-labs"></a>Attacher ou détacher un disque de données sur une machine virtuelle dans Azure DevTest Labs
[Azure Disques managés](../virtual-machines/managed-disks-overview.md) gère les comptes de stockage associés aux disques de données des machines virtuelles. Un utilisateur attache un nouveau disque de données à une machine virtuelle et spécifie le type et la taille du disque voulu, puis Azure crée et gère le disque automatiquement. Le disque de données peut ensuite être détaché de la machine virtuelle, soit pour être rattaché ultérieurement à la même machine virtuelle, soit pour être attaché à une autre machine virtuelle qui appartient au même utilisateur.

Cette fonctionnalité est pratique pour la gestion de stockage ou de logiciels en dehors de chaque machine virtuelle individuelle. Si les logiciels ou le stockage existent déjà à l’intérieur d’un disque de données, celui-ci peut être facilement attaché, détaché et rattaché à n’importe quelle machine virtuelle appartenant à l’utilisateur qui a ce disque de données.

## <a name="attach-a-data-disk"></a>Association d’un disque de données
Avant d’attacher un disque de données, lisez les conseils suivants :

- La taille de la machine virtuelle détermine le nombre de disques de données que vous pouvez attacher. Pour en savoir plus, voir la rubrique [Tailles de machines virtuelles](../virtual-machines/sizes.md).
- Vous pouvez attacher un disque de données à une seule machine virtuelle en cours d’exécution. Assurez-vous que la machine virtuelle s’exécute avant d’essayer d’y attacher un disque de données.

### <a name="attach-a-new-disk"></a>Attacher un nouveau disque
Suivez ces étapes pour créer et attacher un disque de données managé à une machine virtuelle dans Azure DevTest Labs.

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires. 
1. Dans la liste de **Mes machines virtuelles**, sélectionnez-en une en cours d’exécution.
1. Dans le menu de gauche, sélectionnez **Disques**.
1. Choisissez **Attacher un nouveau disque** pour créer un disque de données et l’attacher à la machine virtuelle.

    ![Attacher un nouveau disque de données à une machine virtuelle](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-new.png)
1. Renseignez le volet **Attacher un nouveau disque** en indiquant un nom de disque de données, un type et une taille.

    ![Compléter le formulaire « attacher un nouveau disque »](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-new-form.png)
1. Sélectionnez **OK**.

Au bout de quelques instants, le nouveau disque de données est créé et attaché à la machine virtuelle, et apparaît dans la liste des **Disques de données** pour cette machine.

### <a name="attach-an-existing-disk"></a>Association d'un disque existant
Suivez ces étapes pour rattacher un disque de données disponible à une machine virtuelle en cours d’exécution. 

1. Sélectionnez une machine virtuelle en cours d’exécution à laquelle vous souhaitez rattacher un disque de données.
1. Dans le menu de gauche, sélectionnez **Disques**.
1. Sélectionnez **Attacher un disque existant** pour attacher à la machine virtuelle un disque de données disponible.

    ![Capture d’écran montrant le paramètre « Disques » sélectionné et l’option « Attacher un disque existant ».](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-existing-button.png)

1. À partir du volet **Attacher un disque existant**, sélectionnez OK.

    ![Attacher un disque de données existant à une machine virtuelle](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-existing.png)

Au bout de quelques instants, le disque de données est attaché à la machine virtuelle et apparaît dans la liste des **Disques de données** pour cette celle-ci.

## <a name="detach-a-data-disk"></a>Détachement d'un disque de données
Lorsque vous n’avez plus besoin d’un disque de données attaché à une machine virtuelle, détachez-le tout simplement. Le fait de détacher le disque le supprime de la machine virtuelle, mais il est conservé dans le stockage pour une utilisation ultérieure.

Si vous souhaitez réutiliser les données du disque, vous pouvez le rattacher à la même machine virtuelle ou à une autre machine.

### <a name="detach-from-the-vms-management-pane"></a>Détacher un disque à partir du volet de gestion de la machine virtuelle
1. À partir de votre liste de machines virtuelles, sélectionnez une machine possédant un disque de données attaché.
1. Dans le menu de gauche, sélectionnez **Disques**.
1. Dans la liste des **Disques de données**, sélectionnez le disque de données que vous souhaitez détacher.

    ![Sélectionner des disques de données pour une machine virtuelle](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-button.png) 
1. Sélectionnez **Détacher** dans le haut du volet d’informations du disque.

    ![Capture d’écran montrant le volet d’informations d’un disque avec l’action « Détacher » mise en évidence.](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-data-disk2.png)
1. Sélectionnez **Oui** pour confirmer que vous souhaitez détacher le disque de données.

Le disque est détaché et disponible pour être attaché à une autre machine virtuelle. 
### <a name="detach-from-the-labs-main-pane"></a>Détacher un disque à partir du volet principal du laboratoire
1. Dans de volet principal de votre laboratoire, sélectionnez **Mes disques de données**.
1. Cliquez avec le bouton droit sur le disque de données à détacher, ou sélectionnez les points de suspension ( **...** ) correspondants, puis choisissez **Détacher**.

    ![Détachement d'un disque de données](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-data-disk.png)
1. Sélectionnez **Oui** pour confirmer que vous souhaitez le détacher.

   > [!NOTE]
   > Si un disque de données est déjà détaché, vous pouvez choisir de le supprimer de votre liste de disques de données disponibles en sélectionnant **Supprimer**.
   >
   >

## <a name="upgrade-an-unmanaged-data-disk"></a>Mettre à niveau un disque de données non managé
Si une machine virtuelle existante utilise des disques de données non managés, vous pouvez facilement convertir cette machine virtuelle pour qu’elle utilise des disques managés. Ce processus convertit le disque du système d’exploitation ainsi que tous les autres disques de données attachés.

Pour mettre à niveau un disque de données non managé, suivez les étapes décrites dans cet article et [détachez le disque de données](#detach-a-data-disk) d’une machine virtuelle non managée. Ensuite, [rattachez le disque](#attach-an-existing-disk) à une machine virtuelle managée pour le mettre automatiquement à niveau en le faisant passer de disque non managé à disque managé.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la gestion des disques de données des [machines virtuelles revendicables](devtest-lab-add-claimable-vm.md#unclaim-a-vm).
