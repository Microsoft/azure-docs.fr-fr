---
title: 'Tutoriel : Récupérer des éléments sur Windows Server'
description: Dans ce didacticiel, vous allez apprendre comment utiliser l’agent Microsoft Azure Recovery Services (MARS) pour récupérer des éléments à partir d’Azure sur un serveur Windows.
ms.topic: tutorial
ms.date: 02/14/2018
ms.custom: mvc
ms.openlocfilehash: 746c901747cf1c0b87612a31fbabcb657d5c4a0c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88263110"
---
# <a name="recover-files-from-azure-to-a-windows-server"></a>Récupérer des fichiers à partir d’Azure vers un serveur Windows Server

La sauvegarde Azure permet de récupérer des éléments individuels à partir de sauvegardes de votre serveur Windows Server. La récupération de fichiers individuels est utile si vous devez restaurer rapidement des fichiers accidentellement supprimés. Ce didacticiel décrit l’utilisation de l’agent Microsoft Azure Recovery Services pour récupérer des éléments à partir de sauvegardes que vous avez déjà effectuées dans Azure. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Lancer la récupération d’éléments individuels
> * Sélectionner un point de récupération
> * Restaurer des éléments à partir d’un point de récupération

Ce tutoriel part du principe que vous avez déjà effectué les étapes permettant de [Sauvegarder un serveur Windows Server vers Azure](backup-windows-with-mars-agent.md) et que vous avez au moins une sauvegarde de vos fichiers Windows Server dans Azure.

## <a name="initiate-recovery-of-individual-items"></a>Lancer la récupération d’éléments individuels

Un Assistant d’interface utilisateur utile nommé Sauvegarde Microsoft Azure est installé avec l’agent Microsoft Azure Recovery Services. L’Assistant Sauvegarde Microsoft Azure fonctionne avec l’agent Microsoft Azure Recovery Services pour récupérer des données de sauvegarde à partir de points de récupération stockés dans Azure. Utilisez l’Assistant Sauvegarde Microsoft Azure pour identifier les fichiers ou dossiers que vous voulez restaurer sur Windows Server.

1. Ouvrez le composant logiciel enfichable **Sauvegarde Microsoft Azure**. Vous pouvez le trouver en recherchant **Sauvegarde Microsoft Azure** sur votre ordinateur.

    ![Composant logiciel enfichable Sauvegarde Microsoft Azure](./media/tutorial-backup-restore-files-windows-server/mars.png)

2. Dans l’Assistant, sélectionnez **Récupérer des données** dans le **volet Actions** de la console de l’agent pour démarrer l’Assistant **Récupérer des données**.

    ![Sélectionnez Récupérer les données](./media/tutorial-backup-restore-files-windows-server/mars-recover-data.png)

3. Dans la page **Prise en main**, sélectionnez **Ce serveur (nom du serveur)** puis **Suivant**.

4. Dans la page **Sélectionner le mode de récupération**, sélectionnez **Fichiers et dossiers individuels**, puis **Suivant** pour commencer le processus de sélection de point de récupération.

5. Dans la page **Sélectionner le volume et la date**, sélectionnez le volume qui contient les fichiers ou les dossiers à restaurer, puis **Monter**. Sélectionnez une date et une heure dans le menu déroulant qui correspondent à un point de récupération. Les dates en **gras** indiquent la disponibilité d’au moins un point de récupération ce jour-là.

    ![Sélectionnez un volume et une date](./media/tutorial-backup-restore-files-windows-server/mars-select-date.png)

    Quand vous sélectionnez **Monter**, la sauvegarde Azure rend le point de récupération disponible sous la forme d’un disque. Parcourez et restaurez les fichiers à partir du disque.

## <a name="restore-items-from-a-recovery-point"></a>Restaurer des éléments à partir d’un point de récupération

1. Une fois le volume de récupération monté, sélectionnez **Parcourir** pour ouvrir l’Explorateur Windows et rechercher les fichiers et dossiers à récupérer.

    ![Sélectionnez Parcourir](./media/tutorial-backup-restore-files-windows-server/mars-browse-recover.png)

    Vous pouvez ouvrir les fichiers directement à partir du volume de récupération pour vérifier les fichiers.

2. Dans l’Explorateur Windows, copiez les fichiers et les dossiers que vous voulez restaurer et collez-les à l’emplacement souhaité sur le serveur.

    ![Copiez les fichiers et les dossiers](./media/tutorial-backup-restore-files-windows-server/mars-final.png)

3. Quand vous avez terminé la restauration des fichiers et des dossiers, dans la page **Rechercher et récupérer des fichiers** de l’Assistant **Récupérer des données**, sélectionnez **Démonter**.

    ![Sélectionner l’option de démontage](./media/tutorial-backup-restore-files-windows-server/unmount-and-confirm.png)

4. Sélectionnez **Oui** pour confirmer que vous souhaitez démonter le volume.

    Une fois la capture instantanée démontée, le message **Tâche terminée** s’affiche dans le volet **Tâches** de la console de l’agent.

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel marque la fin des didacticiels sur la sauvegarde et la restauration de données Windows Server vers Azure. Pour en savoir plus sur la sauvegarde Azure, consultez l’exemple PowerShell sur la sauvegarde des machines virtuelles chiffrées.

> [!div class="nextstepaction"]
> [Sauvegarder une machine virtuelle chiffrée](./scripts/backup-powershell-sample-backup-encrypted-vm.md)
