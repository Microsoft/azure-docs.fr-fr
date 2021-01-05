---
title: 'Tutoriel : Sauvegarder des machines virtuelles Linux dans le portail Azure'
description: Avec ce didacticiel, vous allez apprendre à utiliser le portail Azure pour protéger vos machines virtuelles Linux avec le service Sauvegarde Azure.
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 08e6491e1d8d94e8e6e9112e5a19682018103325
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91400347"
---
# <a name="tutorial-back-up-and-restore-files-for-linux-virtual-machines-in-azure"></a>Tutoriel : Sauvegarder et restaurer des fichiers pour des machines virtuelles Linux dans Azure

Vous pouvez protéger vos données en effectuant des sauvegardes à intervalles réguliers. Azure Backup crée des points de récupération stockés dans des coffres de récupération géoredondants. Quand vous effectuez une restauration à partir d’un point de récupération, vous pouvez restaurer la machine virtuelle entière ou des fichiers spécifiques. Cet article explique comment restaurer un fichier unique sur une machine virtuelle Linux exécutant nginx. Si vous ne disposez d’aucune machine virtuelle, vous pouvez en créer une à l’aide du [démarrage rapide Linux](quick-create-cli.md). Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une sauvegarde de machine virtuelle
> * Planifier une sauvegarde quotidienne
> * Restaurer un fichier à partir d’une sauvegarde

## <a name="backup-overview"></a>Présentation de la sauvegarde

Lorsque le service Azure Backup lance une sauvegarde, il déclenche l’extension de sauvegarde pour prendre un instantané à un moment donné. Le service Azure Backup utilise l’extension _VMSnapshotLinux_ dans Linux. L’extension est installée lors de la première sauvegarde de machine virtuelle si cette machine virtuelle est exécutée. Si la machine virtuelle n’est pas en cours d’exécution, le service Sauvegarde prend un instantané du stockage sous-jacent (car aucune écriture de l’application n’a lieu pendant l’arrêt de la machine virtuelle).

Par défaut, le service Azure Backup effectue une sauvegarde cohérente avec le système de fichiers pour la machine virtuelle Linux, bien qu’il puisse être configuré pour effectuer une [sauvegarde cohérente avec les applications en utilisant des infrastructures pré-script et post-script](../../backup/backup-azure-linux-app-consistent.md). Après que le service Sauvegarde Azure a pris l’instantané, les données sont transférées vers le coffre de sauvegarde. Pour optimiser l’efficacité, le service identifie et transfère uniquement les blocs de données qui ont été modifiés depuis la sauvegarde précédente.

Une fois le transfert de données terminé, l’instantané est supprimé et un point de récupération est créé.


## <a name="create-a-backup"></a>Création d'une sauvegarde
Créez une sauvegarde quotidienne planifiée dans un coffre Recovery Services :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche, sélectionnez **Machines virtuelles**. 
3. Dans la liste, sélectionnez la machine virtuelle que vous souhaitez sauvegarder.
4. Dans le panneau de la machine virtuelle, dans la section **Opérations**, cliquez sur **Sauvegarde**. Le panneau **Activer la sauvegarde** s’ouvre.
5. Dans **Coffre Recovery Services**, cliquez sur **Créer** et fournissez le nom du nouveau coffre. Un coffre est créé dans le même groupe de ressources et au même emplacement que la machine virtuelle.
6. Cliquez sur **Stratégie de sauvegarde**. Pour cet exemple, conservez les valeurs par défaut et cliquez sur **OK**.
7. Dans le panneau **Activer la sauvegarde**, cliquez sur **Activer la sauvegarde**. Cette opération crée une sauvegarde quotidienne selon la planification par défaut.
10. Pour créer un point de récupération initial, dans le panneau **Sauvegarde**, cliquez sur **Sauvegarder maintenant**.
11. Dans le panneau **Sauvegarder maintenant**, cliquez sur l’icône de calendrier. Utilisez le contrôle de calendrier pour sélectionner le dernier jour de conservation de ce point de récupération, puis cliquez sur **Sauvegarder**.
12. Le panneau **Sauvegarde** de votre machine virtuelle affiche le nombre de points de récupération terminés.

    ![Points de récupération](./media/tutorial-backup-vms/backup-complete.png)

La première sauvegarde prend environ 20 minutes. Une fois la sauvegarde terminée, passez à l’étape suivante de ce didacticiel.

## <a name="restore-a-file"></a>Restaurer un fichier

Si vous supprimez un fichier ou y apportez des modifications accidentellement, vous pouvez utiliser la récupération de fichier pour restaurer le fichier à partir de votre coffre de sauvegarde. La récupération de fichier utilise un script qui s’exécute sur la machine virtuelle pour monter le point de récupération en tant que lecteur local. Ces lecteurs restent montés pendant 12 heures pour vous permettre de copier des fichiers à partir du point de récupération et de les restaurer sur la machine virtuelle.  

Dans cet exemple, nous allons vous montrer comment récupérer la page web nginx par défaut /var/www/html/index.nginx-debian.html. Ici, l’adresse IP publique de notre machine virtuelle est *13.69.75.209*. Vous pouvez trouver l’adresse IP de votre machine virtuelle en procédant comme suit :

 ```azurecli
 az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
 ```

 
1. Sur votre ordinateur local, ouvrez un navigateur et tapez l’adresse IP publique de votre machine virtuelle pour afficher la page web nginx par défaut.

    ![Capture d’écran montrant la page web nginx par défaut.](./media/tutorial-backup-vms/nginx-working.png)

1. Connectez-vous avec SSH à votre machine virtuelle.

    ```bash
    ssh 13.69.75.209
    ```

2. Supprimez /var/www/html/index.nginx-debian.html.

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```
    
4. Sur votre ordinateur local, actualisez le navigateur en appuyant sur CTRL + F5 pour confirmer que cette page nginx par défaut a disparu.

    ![Capture d’écran montrant que la page nginx par défaut a disparu.](./media/tutorial-backup-vms/nginx-broken.png)
    
1. Sur votre ordinateur local, connectez-vous au [portail Azure](https://portal.azure.com/).
6. Dans le menu de gauche, sélectionnez **Machines virtuelles**. 
7. Sélectionnez la machine virtuelle dans la liste.
8. Dans le panneau de la machine virtuelle, au niveau de la section **Paramètres**, cliquez sur **Sauvegarde**. Le panneau **Sauvegarde** s’ouvre. 
9. Dans le menu en haut du panneau, sélectionnez **Récupération de fichier**. Le panneau **Récupération de fichier** s’affiche.
10. À l’**Étape 1 : Sélectionner un point de récupération**, sélectionnez un point de récupération dans la liste déroulante.
11. À l’**Étape 2 : Télécharger le script pour parcourir et restaurer des fichiers**, cliquez sur le bouton **Télécharger le fichier exécutable**. Enregistrez le fichier téléchargé sur votre ordinateur local.
7. Cliquez sur **Télécharger le script** pour télécharger le fichier de script localement.
8. Ouvrez une invite bash et tapez la commande suivante, en remplaçant *Linux_myVM_05-05-2017.sh* par le chemin et le nom de fichier du script que vous avez téléchargé, *azureuser* par le nom d’utilisateur de la machine virtuelle, et *13.69.75.209* par l’adresse IP publique de votre machine virtuelle.
    
    ```bash
    scp Linux_myVM_05-05-2017.sh azureuser@13.69.75.209:
    ```
    
9. Sur votre ordinateur local, ouvrez une connexion SSH vers la machine virtuelle.

    ```bash
    ssh 13.69.75.209
    ```
    
10. Sur votre machine virtuelle, ajoutez des autorisations d’exécution au fichier de script.

    ```bash
    chmod +x Linux_myVM_05-05-2017.sh
    ```
    
11. Sur votre machine virtuelle, exécutez le script pour monter le point de récupération comme un système de fichiers.

    ```bash
    ./Linux_myVM_05-05-2017.sh
    ```
    
12. La sortie du script vous donne le chemin d’accès pour le point de montage. Le résultat ressemble à ce qui suit :

    ```output
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
                          
    Connecting to recovery point using ISCSI service...
    
    Connection succeeded!
    
    Please wait while we attach volumes of the recovery point to this machine...
                         
    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath 

    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170505191055/Volume1

    ************ Open File Explorer to browse for files. ************

    After recovery, to remove the disks and close the connection to the recovery point, please click 'Unmount Disks' in step 3 of the portal.

    Please enter 'q/Q' to exit...
    ```

12. Sur votre machine virtuelle, copiez la page web nginx par défaut du point de montage vers l’emplacement où vous avez supprimé le fichier.

    ```bash
    sudo cp ~/myVM-20170505191055/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```
    
17. Sur votre ordinateur local, ouvrez l’onglet du navigateur dans lequel vous êtes connecté à l’adresse IP de la machine virtuelle affichant la page nginx par défaut. Appuyez sur CTRL + F5 pour actualiser la page du navigateur. Vous devriez maintenant voir que la page par défaut fonctionne à nouveau.

    ![Page web nginx par défaut](./media/tutorial-backup-vms/nginx-working.png)

18. Sur votre ordinateur local, revenez à l’onglet du navigateur du portail Azure et à l’**Étape 3 : Démonter les disques après la récupération** cliquez sur le bouton **Démonter les disques**. Si vous avez omis cette étape, la connexion au point de montage est automatiquement fermée après 12 heures. Une fois ces 12 heures écoulées, vous devez télécharger un nouveau script pour créer un nouveau point de montage.


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer une sauvegarde de machine virtuelle
> * Planifier une sauvegarde quotidienne
> * Restaurer un fichier à partir d’une sauvegarde

Passez au didacticiel suivant pour en savoir plus sur la surveillance des machines virtuelles.

> [!div class="nextstepaction"]
> [Gouverner les machines virtuelles](tutorial-govern-resources.md)
