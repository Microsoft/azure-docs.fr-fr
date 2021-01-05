---
title: Tutoriel - Restaurer des fichiers sur une machine virtuelle avec Sauvegarde Azure
description: Découvrez comment effectuer des restaurations au niveau du fichier sur une machine virtuelle avec Sauvegarde Azure et Recovery Services.
ms.topic: tutorial
ms.date: 01/31/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d977919b806be32b84001a9b91dc9e396fbd63ce
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96557907"
---
# <a name="restore-files-to-a-virtual-machine-in-azure"></a>Restaurer des fichiers vers une machine virtuelle dans Azure

Azure Backup crée des points de récupération stockés dans des coffres de récupération géoredondants. Quand vous effectuez une restauration à partir d’un point de récupération, vous pouvez restaurer la machine virtuelle entière ou des fichiers individuels. Cet article explique comment restaurer des fichiers individuels. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Répertorier et sélectionner des points de récupération
> * Connecter un point de récupération vers une machine virtuelle
> * Restaurer des fichiers à partir d’un point de récupération

## <a name="prerequisites"></a>Prérequis

Ce didacticiel nécessite une machine virtuelle Linux qui a été protégée avec Sauvegarde Azure. Pour simuler une suppression accidentelle de fichiers et le processus de récupération, vous supprimez une page d’un serveur web. Si vous avez besoin d’une machine virtuelle Linux qui exécute un serveur Web et qui a été protégée avec Sauvegarde Azure, consultez [Sauvegarder une machine virtuelle dans Azure avec l’interface CLI](quick-backup-vm-cli.md).

Préparez votre environnement :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0.18 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="backup-overview"></a>Présentation de la sauvegarde

Quand Azure lance une sauvegarde, l’extension de sauvegarde sur la machine virtuelle capture un instantané à un point dans le temps. L’extension de sauvegarde est installée sur la machine virtuelle quand la première sauvegarde est demandée. Sauvegarde Azure peut également capturer un instantané du stockage sous-jacent si la machine virtuelle n’est pas en cours d’exécution durant la sauvegarde.

Par défaut, Sauvegarde Azure effectue une sauvegarde cohérente du système de fichiers. Une fois que Sauvegarde Azure a pris l’instantané, les données sont transférées vers le coffre Recovery Services. Pour optimiser l’efficacité, Sauvegarde Azure identifie et transfère uniquement les blocs de données qui ont été modifiés depuis la sauvegarde précédente.

Une fois le transfert de données terminé, l’instantané est supprimé et un point de récupération est créé.

## <a name="delete-a-file-from-a-vm"></a>Supprimer un fichier d’une machine virtuelle

Si vous supprimez ou modifiez accidentellement un fichier, vous pouvez restaurer des fichiers individuels à partir d’un point de récupération. Ce processus vous permet de parcourir les fichiers sauvegardés dans un point de récupération et de restaurer uniquement les fichiers dont vous avez besoin. Dans cet exemple, nous supprimons un fichier d’un serveur web pour illustrer le processus de récupération au niveau du fichier.

1. Pour vous connecter à votre machine virtuelle, obtenez l’adresse IP de votre machine virtuelle à l’aide de la commande [az vm show](/cli/azure/vm#az-vm-show) :

     ```azurecli-interactive
     az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
     ```

2. Pour confirmer que votre site web fonctionne, ouvrez un navigateur web à l’adresse IP publique de votre machine virtuelle. Laissez ouverte la fenêtre du navigateur web.

    ![Page web NGINX par défaut](./media/tutorial-restore-files/nginx-working.png)

3. Connectez-vous à votre machine virtuelle avec SSH. Remplacez *publicIpAddress* par l’adresse IP publique que vous avez obtenue dans une commande précédente :

    ```bash
    ssh publicIpAddress
    ```

4. Supprimez la page par défaut du serveur web à l’adresse */var/www/html/index.nginx-debian.html* comme suit :

    ```bash
    sudo rm /var/www/html/index.nginx-debian.html
    ```

5. Dans votre navigateur web, actualisez la page web. Le site web ne charge plus la page, comme indiqué dans l’exemple suivant :

    ![Le site web NGINX ne charge plus la page par défaut](./media/tutorial-restore-files/nginx-broken.png)

6. Fermez la session SSH sur votre machine virtuelle comme suit :

    ```bash
    exit
    ```

## <a name="generate-file-recovery-script"></a>Générer un script de récupération de fichier

Pour restaurer vos fichiers, Sauvegarde Azure fournit un script à exécuter sur votre machine virtuelle, qui se connecte à votre point de récupération comme un lecteur local. Vous pouvez parcourir ce lecteur local, restaurer des fichiers sur la machine virtuelle, puis déconnecter le point de récupération. Sauvegarde Azure continue de sauvegarder vos données en fonction de la stratégie affectée à la planification et à la rétention.

1. Pour répertorier les points de récupération de votre machine virtuelle, utilisez [az backup recoverypoint list](/cli/azure/backup/recoverypoint#az-backup-recoverypoint-list). Dans cet exemple, nous sélectionnons le point de récupération le plus récent pour la machine virtuelle nommée *myVM* et qui est protégée dans *myRecoveryServicesVault* :

    ```azurecli-interactive
    az backup recoverypoint list \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --query [0].name \
        --output tsv
    ```

2. Pour obtenir le script qui connecte ou monte le point de récupération sur votre machine virtuelle, utilisez [az backup restore files mount-rp](/cli/azure/backup/restore/files#az-backup-restore-files-mount-rp). L’exemple suivant obtient le script pour la machine virtuelle nommée *myVM* et qui est protégée dans *myRecoveryServicesVault*.

    Remplacez *myRecoveryPointName* par le nom du point de récupération que vous avez obtenu dans la commande précédente :

    ```azurecli-interactive
    az backup restore files mount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

    Le script est téléchargé et un mot de passe s’affiche, comme dans l’exemple suivant :

    ```output
    File downloaded: myVM_we_1571974050985163527.sh. Use password c068a041ce12465
    ```

3. Pour transférer le script à votre machine virtuelle, utilisez Secure copie (SCP). Indiquez le nom de votre script téléchargé, puis remplacez *publicIpAddress* par l’adresse IP publique de votre machine virtuelle. Veillez à inclure l’élément `:` à la fin de la commande SCP, comme suit :

    ```bash
    scp myVM_we_1571974050985163527.sh 52.174.241.110:
    ```

## <a name="restore-file-to-your-vm"></a>Restaurer un fichier sur votre machine virtuelle

Une fois le script de récupération copié sur votre machine virtuelle, vous pouvez maintenant connecter au point de récupération et restaurer des fichiers.

>[!NOTE]
> Consultez les informations [ici](backup-azure-restore-files-from-vm.md#step-2-ensure-the-machine-meets-the-requirements-before-executing-the-script) pour savoir si vous pouvez exécuter le script sur votre machine virtuelle avant de continuer.

1. Connectez-vous à votre machine virtuelle avec SSH. Remplacez *publicIpAddress* par l’adresse IP publique de votre machine virtuelle, comme suit :

    ```bash
    ssh publicIpAddress
    ```

2. Pour permettre à votre script de s’exécuter correctement, ajoutez des autorisations d’exécution à l’aide de la commande **chmod**. Entrez le nom de votre propre script :

    ```bash
    chmod +x myVM_we_1571974050985163527.sh
    ```

3. Pour monter le point de récupération, exécutez le script. Entrez le nom de votre propre script :

    ```bash
    ./myVM_we_1571974050985163527.sh
    ```

    Lorsque le script s’exécute, vous êtes invité à entrer un mot de passe pour accéder au point de récupération. Entrez le mot de passe indiqué dans la sortie de la commande [az backup restore files mount-rp](/cli/azure/backup/restore/files#az-backup-restore-files-mount-rp) précédente qui a généré le script de récupération.

    La sortie du script vous donne le chemin d’accès au point de récupération. L’exemple de sortie suivant montre que le point de récupération est monté sur */home/azureuser/myVM-20170919213536/Volume1* :

    ```output
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
    Please enter the password as shown on the portal to securely connect to the recovery point. : c068a041ce12465

    Connecting to recovery point using ISCSI service...

    Connection succeeded!

    Please wait while we attach volumes of the recovery point to this machine...

    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath

    1)  | /dev/sdc  |  /dev/sdc1  |  /home/azureuser/myVM-20170919213536/Volume1

    ************ Open File Explorer to browse for files. ************
    ```

4. Utilisez **cp** pour copier la page web NGINX par défaut du point de récupération monté vers l’emplacement du fichier d’origine. Remplacez le point de montage */home/azureuser/myVM-20170919213536/Volume1* par votre propre emplacement :

    ```bash
    sudo cp /home/azureuser/myVM-20170919213536/Volume1/var/www/html/index.nginx-debian.html /var/www/html/
    ```

5. Dans votre navigateur web, actualisez la page web. Le site web charge désormais la page correctement, comme indiqué dans l’exemple suivant :

    ![Le site web NGINX se charge maintenant correctement](./media/tutorial-restore-files/nginx-restored.png)

6. Fermez la session SSH sur votre machine virtuelle comme suit :

    ```bash
    exit
    ```

7. Démontez le point de récupération de votre machine virtuelle à l’aide de la commande [az backup restore files unmount-rp](/cli/azure/backup/restore/files#az-backup-restore-files-unmount-rp). L’exemple suivant démonte le point de récupération de la machine virtuelle nommée *myVM* dans *myRecoveryServicesVault*.

    Remplacez *myRecoveryPointName* par le nom de votre point de récupération que vous avez obtenu dans les commandes précédentes :

    ```azurecli-interactive
    az backup restore files unmount-rp \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --rp-name myRecoveryPointName
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez connecté un point de récupération à une machine virtuelle et restauré des fichiers pour un serveur web. Vous avez appris à :

> [!div class="checklist"]
>
> * Répertorier et sélectionner des points de récupération
> * Connecter un point de récupération vers une machine virtuelle
> * Restaurer des fichiers à partir d’un point de récupération

Passez à l’étape suivante du didacticiel pour en savoir plus sur la sauvegarde de Windows Server dans Azure.

> [!div class="nextstepaction"]
> [Sauvegarder Windows Server dans Azure](tutorial-backup-windows-server-to-azure.md)
