---
title: Configurer Oracle ASM sur une machine virtuelle Linux Azure | Microsoft Docs
description: Rendez Oracle ASM opérationnel rapidement dans votre environnement Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 6f0b58374ae3eb972993b544117d8cbb98371ce5
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602612"
---
# <a name="set-up-oracle-asm-on-an-azure-linux-virtual-machine"></a>Configurer Oracle ASM sur une machine virtuelle Linux Azure  

Les machines virtuelles fournissent un environnement informatique entièrement configurable et flexible. Ce didacticiel décrit le déploiement de base d’une machine virtuelle Azure associé à l’installation et à la configuration d’Oracle Automated Storage Management (ASM).  Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Oracle Database et s’y connecter
> * Installer et configurer Oracle Automated Storage Management
> * Installer et configurer Oracle Grid Infrastructure
> * Initialiser une installation Oracle ASM
> * Créer une base de données Oracle gérée par ASM


Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, ce didacticiel exige que vous exécutiez Azure CLI version 2.0.4 ou une version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli). 

## <a name="prepare-the-environment"></a>Préparer l’environnement

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Pour créer un groupe de ressources, utilisez la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Dans cet exemple, un groupe de ressources nommé *myResourceGroup* est créé dans la région *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

### <a name="create-a-vm"></a>Créer une machine virtuelle

Pour créer une machine virtuelle basée sur l’image d’Oracle Database et la configurer pour utiliser Oracle ASM, exécutez la commande [az vm create](/cli/azure/vm). 

L’exemple suivant crée une machine virtuelle nommée myVM qui a une taille Standard_DS2_v2 avec quatre disques de données associés de 50 Go. Il crée également des clés SSH si elles n’existent pas encore à un emplacement de clé par défaut.  Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.  

   ```azurecli-interactive
   az vm create --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --generate-ssh-keys \
    --data-disk-sizes-gb 50 50 50 50
   ```

Une fois que vous avez créé la machine virtuelle, Azure CLI affiche des informations similaires à l’exemple suivant. Notez la valeur pour `publicIpAddress`. Vous utilisez cette adresse pour accéder à la machine virtuelle.

   ```output
   {
     "fqdns": "",
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
     "location": "eastus",
     "macAddress": "00-0D-3A-36-2F-56",
     "powerState": "VM running",
     "privateIpAddress": "10.0.0.4",
     "publicIpAddress": "13.64.104.241",
     "resourceGroup": "myResourceGroup"
   }
   ```

### <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Pour créer une session SSH avec la machine virtuelle et configurer des paramètres supplémentaires, utilisez la commande suivante. Remplacez l’adresse IP par la valeur de `publicIpAddress` pour votre machine virtuelle.

```bash
ssh <publicIpAddress>
```

## <a name="install-oracle-asm"></a>Installer Oracle ASM

Pour installer Oracle ASM, procédez comme suit. 

Pour plus d’informations sur l’installation d’Oracle ASM, consultez [Oracle ASMLib Downloads for Oracle Linux 6](https://www.oracle.com/technetwork/server-storage/linux/asmlib/ol6-1709075.html).  

1. Vous devez ouvrir une session en tant qu’utilisateur ROOT afin de poursuivre l’installation d’ASM :

   ```bash
   sudo su -
   ```
   
2. Exécutez ces commandes supplémentaires pour installer les composants Oracle ASM :

   ```bash
    yum list | grep oracleasm 
    yum -y install kmod-oracleasm.x86_64 
    yum -y install oracleasm-support.x86_64 
    wget https://download.oracle.com/otn_software/asmlib/oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    yum -y install oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    rm -f oracleasmlib-2.0.12-1.el6.x86_64.rpm
   ```

3. Vérifiez qu’Oracle ASM est installé :

   ```bash
   rpm -qa |grep oracleasm
   ```

    Le résultat de cette commande doit énumérer les composants suivants :

    ```bash
   oracleasm-support-2.1.10-4.el6.x86_64
   kmod-oracleasm-2.0.8-15.el6_9.x86_64
   oracleasmlib-2.0.12-1.el6.x86_64
    ```

4. ASM requiert des rôles et des utilisateurs spécifiques afin de fonctionner correctement. Les commandes suivantes créent les comptes d’utilisateur et les groupes prérequis : 

   ```bash
    groupadd -g 54345 asmadmin 
    groupadd -g 54346 asmdba 
    groupadd -g 54347 asmoper 
    useradd -u 3000 -g oinstall -G dba,asmadmin,asmdba,asmoper grid 
    usermod -g oinstall -G dba,asmdba,asmadmin oracle
   ```

5. Vérifiez que les utilisateurs et les groupes ont été créés correctement :

   ```bash
   id grid
   ```

    Le résultat de cette commande doit énumérer les utilisateurs et les groupes suivants :

    ```bash
    uid=3000(grid) gid=54321(oinstall) groups=54321(oinstall),54322(dba),54345(asmadmin),54346(asmdba),54347(asmoper)
    ```
 
6. Créez un dossier pour l’utilisateur *grid* et changez son propriétaire :

   ```bash
   mkdir /u01/app/grid 
   chown grid:oinstall /u01/app/grid
   ```

## <a name="set-up-oracle-asm"></a>Configurer Oracle ASM

Pour ce didacticiel, l’utilisateur par défaut est *grid* et le groupe par défaut est *asmadmin*. Vérifiez que l’utilisateur *oracle* fait partie du groupe asmadmin. Pour configurer votre installation Oracle ASM, procédez comme suit :

1. La configuration du pilote de la bibliothèque Oracle ASM implique la définition de l’utilisateur par défaut (grid) et du groupe par défaut (asmadmin), ainsi que la configuration du lecteur pour un lancement au démarrage (choisissez y) et l’analyse des disques au démarrage (choisissez y). Vous devez répondre aux invites de la commande suivante :

   ```bash
   /usr/sbin/oracleasm configure -i
   ```

   Le résultat de cette commande doit ressembler à ce qui suit, jusqu’aux invites.

    ```output
   Configuring the Oracle ASM library driver.

   This will configure the on-boot properties of the Oracle ASM library
   driver. The following questions will determine whether the driver is
   loaded on boot and what permissions it will have. The current values
   will be shown in brackets ('[]'). Hitting <ENTER> without typing an
   answer will keep that current value. Ctrl-C will abort.

   Default user to own the driver interface []: grid
   Default group to own the driver interface []: asmadmin
   Start Oracle ASM library driver on boot (y/n) [n]: y
   Scan for Oracle ASM disks on boot (y/n) [y]: y
   Writing Oracle ASM library driver configuration: done
   ```

2. Affichez la configuration du disque :

   ```bash
   cat /proc/partitions
   ```

   Le résultat de cette commande doit ressembler à la liste suivante de disques disponibles.

   ```output
   8       16   14680064 sdb
   8       17   14678976 sdb1
   8        0   52428800 sda
   8        1     512000 sda1
   8        2   51915776 sda2
   8       48   52428800 sdd
   8       64   52428800 sde
   8       80   52428800 sdf
   8       32   52428800 sdc
   11       0       1152 sr0
   ```

3. Formatez le disque */dev/sdc* en exécutant la commande suivante et en répondant aux invites avec :
   - *n* pour la nouvelle partition
   - *p* pour la partition principale
   - *1* pour sélectionner la première partition
   - appuyez sur `enter` pour le premier cylindre de valeur par défaut
   - appuyez sur `enter` pour le dernier cylindre de valeur par défaut
   - appuyez sur *w* pour écrire les modifications dans la table de partition  

   ```bash
   fdisk /dev/sdc
   ```
   
   Avec les réponses fournies ci-dessus, la sortie de la commande `fdisk` doit se présenter comme suit :

   ```output
   Device contains not a valid DOS partition table, or Sun, SGI or OSF disklabel
   Building a new DOS disklabel with disk identifier 0xf865c6ca.
   Changes will remain in memory only, until you decide to write them.
   After that, of course, the previous content won't be recoverable.

   Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

   The device presents a logical sector size that is smaller than
   the physical sector size. Aligning to a physical sector (or optimal
   I/O) size boundary is recommended, or performance may be impacted.

   WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
           switch off the mode (command 'c') and change display units to
           sectors (command 'u').

   Command (m for help): n
   Command action
     e   extended
     p   primary partition (1-4)
   p
   Partition number (1-4): 1
   First cylinder (1-6527, default 1):
   Using default value 1
   Last cylinder, +cylinders or +size{K,M,G} (1-6527, default 6527):
   Using default value 6527

   Command (m for help): w
   The partition table has been altered!

   Calling ioctl() to re-read partition table.
   Syncing disks.
   ```

4. Répétez la commande `fdisk` précédente pour `/dev/sdd`, `/dev/sde` et `/dev/sdf`.

5. Vérifiez la configuration du disque :

   ```bash
   cat /proc/partitions
   ```

   La sortie de la commande doit se présenter comme suit :

   ```output
   major minor  #blocks  name

     8       16   14680064 sdb
     8       17   14678976 sdb1
     8       32   52428800 sdc
     8       33   52428096 sdc1
     8       48   52428800 sdd
     8       49   52428096 sdd1
     8       64   52428800 sde
     8       65   52428096 sde1
     8       80   52428800 sdf
     8       81   52428096 sdf1
     8        0   52428800 sda
     8        1     512000 sda1
     8        2   51915776 sda2
     11       0    1048575 sr0
   ```

6. Vérifiez l’état du service Oracle ASM et démarrez le service Oracle ASM :

   ```bash
   service oracleasm status 
   service oracleasm start
   ```

   La sortie de la commande doit se présenter comme suit :

   ```output
   Checking if ASM is loaded: no
   Checking if /dev/oracleasm is mounted: no
   Initializing the Oracle ASMLib driver:                     [  OK  ]
   Scanning the system for Oracle ASMLib disks:               [  OK  ]
   ```

7. Créez les disques Oracle ASM :

   ```bash
   service oracleasm createdisk ASMSP /dev/sdc1 
   service oracleasm createdisk DATA /dev/sdd1 
   service oracleasm createdisk DATA1 /dev/sde1 
   service oracleasm createdisk FRA /dev/sdf1
   ```

   La sortie de la commande doit se présenter comme suit :

   ```output
   Marking disk "ASMSP" as an ASM disk:                       [  OK  ]
   Marking disk "DATA" as an ASM disk:                        [  OK  ]
   Marking disk "DATA1" as an ASM disk:                       [  OK  ]
   Marking disk "FRA" as an ASM disk:                         [  OK  ]
   ```

8. Répertoriez les disques Oracle ASM :

   ```bash
   service oracleasm listdisks
   ```

   Le résultat de la commande doit énumérer les disques Oracle ASM suivants :

   ```output
    ASMSP
    DATA
    DATA1
    FRA
   ```

9. Modifiez les mots de passe pour les utilisateurs de la racine, d’Oracle et de Grid **Prenez note de ces nouveaux mots de passe**, car vous les utiliserez plus tard pendant l’installation.

   ```bash
   passwd oracle 
   passwd grid 
   passwd root
   ```

10. Changez l’autorisation du dossier :

    ```bash
    chmod -R 775 /opt 
    chown grid:oinstall /opt 
    chown oracle:oinstall /dev/sdc1 
    chown oracle:oinstall /dev/sdd1 
    chown oracle:oinstall /dev/sde1 
    chown oracle:oinstall /dev/sdf1 
    chmod 600 /dev/sdc1 
    chmod 600 /dev/sdd1 
    chmod 600 /dev/sde1 
    chmod 600 /dev/sdf1
    ```

## <a name="download-and-prepare-oracle-grid-infrastructure"></a>Télécharger et préparer Oracle Grid Infrastructure

Pour télécharger et préparer Oracle Grid Infrastructure, procédez comme suit :

1. Téléchargez Oracle Grid Infrastructure à partir de la [page de téléchargement d’Oracle ASM](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html). 

   Sous le téléchargement intitulé **Oracle Database 12c Release 1 Grid Infrastructure (12.1.0.2.0) for Linux x86-64**, téléchargez les deux fichiers .zip.

2. Une fois que vous avez téléchargé les fichiers .zip sur votre ordinateur client, vous pouvez utiliser SCP (Secure Copy Protocol) pour les copier sur votre machine virtuelle :

   ```bash
   scp *.zip <publicIpAddress>:.
   ```

3. Revenez via SSH sur votre machine virtuelle Oracle dans Azure afin de déplacer les fichiers .zip dans le dossier /opt. Ensuite, changez le propriétaire des fichiers :

   ```bash
   ssh <publicIPAddress>
   sudo mv ./*.zip /opt
   cd /opt
   sudo chown grid:oinstall linuxamd64_12102_grid_1of2.zip
   sudo chown grid:oinstall linuxamd64_12102_grid_2of2.zip
   ```

4. Décompressez les fichiers (installez l’outil de décompression Linux si ce n’est pas déjà fait).

   ```bash
   sudo yum install unzip
   sudo unzip linuxamd64_12102_grid_1of2.zip
   sudo unzip linuxamd64_12102_grid_2of2.zip
   ```

5. Changez l’autorisation :

   ```bash
   sudo chown -R grid:oinstall /opt/grid
   ```

6. Mettez à jour l’espace d’échange configuré. Les composants Oracle Grid ont besoin d’au moins 6,8 Go d’espace d’échange pour installer Grid. La taille du fichier d’échange par défaut pour les images Oracle Linux dans Azure est seulement de 2 048 Mo. Vous devez augmenter `ResourceDisk.SwapSizeMB` dans le fichier `/etc/waagent.conf` et redémarrer le service WALinuxAgent afin que les paramètres mis à jour soient appliqués. S’agissant d’un fichier en lecture seule, vous devez modifier les autorisations de fichier pour permettre un accès en écriture.

   ```bash
   sudo chmod 777 /etc/waagent.conf  
   vi /etc/waagent.conf
   ```

   Recherchez `ResourceDisk.SwapSizeMB` et remplacez la valeur par **8192**. Vous devez appuyer sur `insert` pour passer en mode d’insertion, saisir la valeur **8192**, puis appuyer sur `esc` pour revenir au mode de commande. Pour écrire les modifications et fermer le fichier, saisissez `:wq` et appuyez sur `enter`.
   
   > [!NOTE]
   > Nous vous recommandons vivement d’utiliser `WALinuxAgent` pour configurer l’espace d’échange afin qu’il soit toujours créé sur le disque éphémère local (temporaire). Vous bénéficierez ainsi de performances optimales. Pour plus d’informations, consultez [Comment faire pour ajouter un fichier d’échange dans des machines virtuelles Azure Linux](https://support.microsoft.com/en-us/help/4010058/how-to-add-a-swap-file-in-linux-azure-virtual-machines).

## <a name="prepare-your-local-client-and-vm-to-run-x11"></a>Préparer votre client local et la machine virtuelle pour exécuter x11
La configuration d’Oracle ASM nécessite une interface graphique pour mener à bien l’installation et la configuration. Nous utilisons le protocole x11 pour faciliter cette installation. Si vous utilisez un système client (Mac ou Linux) dont les fonctionnalités x11 sont déjà activées et configurées, vous pouvez ignorer cette installation et cette configuration propres aux ordinateurs Windows. 

1. Téléchargez [PuTTY](https://www.putty.org/) et [Xming](https://xming.en.softonic.com/) sur votre ordinateur Windows. Vous devez terminer l’installation de ces deux applications avec les valeurs par défaut avant de continuer.

2. Après avoir installé PuTTY, ouvrez une invite de commandes dans le dossier PuTTY (par exemple, C:\Program Files\PuTTY) et modifiez `puttygen.exe` afin de générer une clé.

3. Dans PuTTY Key Generator :
   
   1. Générez une clé en sélectionnant le bouton `Generate`.
   2. Copiez le contenu de la clé (Ctrl+C).
   3. Sélectionnez le bouton `Save private key`.
   4. Ignorez l’avertissement sur la sécurisation de la clé avec une phrase secrète, puis sélectionnez `OK`.

   ![Capture d’écran du générateur de clé PuTTY](./media/oracle-asm/puttykeygen.png)

4. Dans votre machine virtuelle, exécutez ces commandes :

   ```bash
   sudo su - grid
   mkdir .ssh 
   cd .ssh
   ```

5. Créez un fichier appelé `authorized_keys`. Collez le contenu de la clé dans ce fichier, puis enregistrez-le.

   > [!NOTE]
   > La clé doit contenir la chaîne `ssh-rsa`. En outre, le contenu de la clé doit être constitué d’une seule ligne de texte.
   >  

6. Sur votre système client, démarrez PuTTY. Dans le volet **Category** (Catégorie), accédez à **Connection** > **SSH** > **Auth** (Connexion > SSH > Authentification). Dans la zone **Private key file for authentication** (Fichier de clé privée pour l’authentification), accédez à la clé que vous avez générée précédemment.

   ![Capture d’écran des options d’authentification SSH](./media/oracle-asm/setprivatekey.png)

7. Dans le volet **Category** (Catégorie), accédez à **Connection** > **SSH** > **X11** (Connexion > SSH > X11). Activez la case à cocher **Enable X11 forwarding** (Activer le transfert X11).

   ![Capture d’écran des options de transfert SSH X11](./media/oracle-asm/enablex11.png)

8. Dans le volet **Category** (Catégorie), accédez à **Session**. Indiquez votre machine virtuelle Oracle ASM `<publicIPaddress>` dans la boîte de dialogue du nom hôte, renseignez un nouveau nom `Saved Session`, puis cliquez sur `Save`.  Une fois les modifications enregistrées, cliquez sur `open` pour vous connecter à votre machine virtuelle Oracle ASM.  Lors de la première connexion, vous êtes averti que le système distant n’est pas mis en cache dans le registre. Cliquez sur `yes` pour l’ajouter et continuer.

   ![Capture d’écran des options de session PuTTY](./media/oracle-asm/puttysession.png)

## <a name="install-oracle-grid-infrastructure"></a>Installer Oracle Grid Infrastructure

Pour installer Oracle Grid Infrastructure, procédez comme suit :

1. Connectez-vous en tant que **grid**. (Vous devez pouvoir vous connecter sans avoir à saisir un mot de passe.) 

   > [!NOTE]
   > Si vous exécutez Windows, vérifiez que vous avez démarré Xming avant de commencer l’installation.

   ```bash
   cd /opt/grid
   ./runInstaller
   ```

   Le programme d’installation d’Oracle Grid Infrastructure 12c Release 1 s’ouvre. (Plusieurs minutes peuvent être nécessaires au démarrage du programme d’installation.)

2. Dans la page **Select Installation Option** (Sélectionner l’option d’installation), sélectionnez **Install and Configure Oracle Grid Infrastructure for a Standalone Server** (Installer et configurer Oracle Grid Infrastructure pour un serveur autonome).

   ![Capture d’écran de la page Select Installation Option (Sélectionner l’option d’installation)](./media/oracle-asm/install01.png)

3. Sur la page **Select Product Languages** (Sélectionner les langues du produit), vérifiez que **English** (Anglais) ou la langue souhaitée est sélectionné.  Cliquez sur `next`.

4. Dans la page **Create ASM Disk Group** (Créer un groupe de disques ASM) :
   - Entrez un nom pour le groupe de disques.
   - Sous **Redundancy** (Redondance), sélectionnez **External** (Externe).
   - Sous **Allocation Unit Size** (Taille d’unité d’allocation), sélectionnez **4**.
   - Sous **Add Disks** (Ajouter des disques), sélectionnez **ORCLASMSP**.
   - Cliquez sur `next`.

5. Dans la page **Specify ASM Password** (Spécifier le mot de passe ASM), sélectionnez l’option **Use same passwords for these accounts** (Utiliser les mêmes mots de passe pour ces comptes) et entrez un mot de passe.

   ![Capture d’écran de la page Specify ASM Password (Spécifier le mot de passe ASM) du programme d’installation](./media/oracle-asm/install04.png)

6. Sur la page **Specify Management Options** (Spécifier les options de gestion), vous pouvez configurer EM Cloud Control. Cette option est ignorée ; cliquez sur `next` pour continuer. 

7. Dans la page **Privileged Operating System Groups** (Groupes privilégiés du système d’exploitation), utilisez les paramètres par défaut. Cliquez sur `next` pour continuer.

8. Dans la page **Specify Installation Location** (Spécifier l’emplacement d’installation), utilisez les paramètres par défaut. Cliquez sur `next` pour continuer.

9. Sur la page **Create Inventory** (Créer un inventaire), remplacez le répertoire de l’inventaire par `/u01/app/grid/oraInventory`. Cliquez sur `next` pour continuer.

   ![Capture d’écran de la page Create Inventory (Créer un inventaire) du programme d’installation](./media/oracle-asm/install08.png)

10. Dans la page **Root script execution configuration** (Configuration de l’exécution des scripts root), actualisez la case à cocher **Automatically run configuration scripts** (Exécuter automatiquement les scripts de configuration). Ensuite, sélectionnez l’option **Use "root" user credential** (Utiliser les informations d’identification de l’utilisateur « root »), puis entrez le mot de passe de l’utilisateur root.

    ![Capture d’écran de la page Use "root" user credential (Utiliser les informations d’identification de l’utilisateur « root ») du programme d’installation](./media/oracle-asm/install09.png)

11. Sur la page **Perform Prerequisite Checks** (Effectuer les tests de prérequis), le programme d’installation en cours échoue avec des erreurs. Ce comportement est normal. Sélectionnez `Fix & Check Again`.

12. Dans la boîte de dialogue **Fixup Script** (Corriger le script), cliquez sur `OK`.

13. Sur la page **Summary** (Récapitulatif), passez en revue les paramètres sélectionnés, puis cliquez sur `Install`.

    ![Capture d’écran de la page Summary (Récapitulatif) du programme d’installation](./media/oracle-asm/install12.png)

14. Une boîte de dialogue d’avertissement apparaît. Elle vous informe que les scripts de configuration doivent être exécutés en tant qu’utilisateur doté de privilèges. Cliquez sur `Yes` pour continuer.

15. Sur la page **Finish** (Terminer), cliquez sur `Close` pour terminer l’installation.

## <a name="set-up-your-oracle-asm-installation"></a>Configurer votre installation Oracle ASM

Pour configurer votre installation Oracle ASM, procédez comme suit :

1. Vérifiez que vous êtes toujours connecté en tant que **grid** à partir de votre session X11. Vous devrez peut-être appuyer sur `enter` pour réactiver le terminal. Ensuite, lancez l’Assistant de configuration d’Oracle Automated Storage Management :

   ```bash
   cd /u01/app/grid/product/12.1.0/grid/bin
   ./asmca
   ```

   L’Assistant de configuration d’Oracle ASM s’ouvre.

2. Dans la boîte de dialogue **Configure ASM: Disk Groups** (Configurer ASM : groupes de disques), cliquez sur le bouton `Create`, puis sur `Show Advanced Options`.

3. Dans la boîte de dialogue **Create Disk Group** (Créer un groupe de disques) :

   - Entrez le nom de groupe de disques **DATA**.
   - Sous **Select Member Disks** (Sélectionner les disques membres), sélectionnez **ORCL_DATA** et **ORCL_DATA1**.
   - Sous **Allocation Unit Size** (Taille d’unité d’allocation), sélectionnez **4**.
   - Cliquez sur `ok` pour créer le groupe de disques.
   - Cliquez sur `ok` pour fermer la fenêtre de confirmation.

   ![Capture d’écran de la boîte de dialogue Create Disk Group (Créer un groupe de disques)](./media/oracle-asm/asm02.png)

4. Dans la boîte de dialogue **Configure ASM: Disk Groups** (Configurer ASM : groupes de disques), cliquez sur le bouton `Create`, puis sur `Show Advanced Options`.

5. Dans la boîte de dialogue **Create Disk Group** (Créer un groupe de disques) :

   - Entrez le nom de groupe de disques **FRA**.
   - Sous **Redundancy** (Redondance), sélectionnez **External (none)** (Externe (aucun)).
   - Sous **Select Member Disks** (Sélectionner les disques membres), sélectionnez **ORCL_FRA**.
   - Sous **Allocation Unit Size** (Taille d’unité d’allocation), sélectionnez **4**.
   - Cliquez sur `ok` pour créer le groupe de disques.
   - Cliquez sur `ok` pour fermer la fenêtre de confirmation.

   ![Capture d’écran de la boîte de dialogue Create Disk Group (Créer un groupe de disques), avec l’option Externe (non) (Externe (aucune)) en évidence.](./media/oracle-asm/asm04.png)

6. Sélectionnez **Exit** (Quitter) pour fermer l’Assistant de configuration d’ASM.

   ![Capture d'écran de la boîte de dialogue Configure ASM: Disk Groups (Configurer ASM : Groupes de disques) avec le bouton Exit (Quitter)](./media/oracle-asm/asm05.png)

## <a name="create-the-database"></a>Création de la base de données

Le logiciel de base de données Oracle est déjà installé sur l’image de la Place de marché Azure. Pour créer une base de données, exécutez les étapes suivantes :

1. Changez l’utilisateur et passez au superutilisateur d’Oracle, puis initialisez l’écouteur pour la journalisation :

   ```bash
   su - oracle
   cd /u01/app/oracle/product/12.1.0/dbhome_1/bin
   ./dbca
   ```

   L’Assistant de configuration de base de données s’ouvre.

2. Sur la page **Database Operation** (Opération de base de données), cliquez sur `Create Database`.

3. Dans la page **Creation Mode** (Mode de création ) :

   - Entrez un nom pour la base de données.
   - Pour **Storage Type** (Type de stockage), assurez-vous que l’option **Automatic Storage Management (ASM)** est sélectionnée.
   - Pour l’**emplacement des fichiers de base de données**, utilisez l’emplacement par défaut ASM suggéré.
   - Pour la **zone de récupération rapide**, utilisez l’emplacement par défaut ASM suggéré.
   - Saisissez un **mot de passe administrateur** et **confirmez le mot de passe**.
   - Vérifiez que l’option `create as container database` est sélectionnée.
   - Saisissez une valeur pour `pluggable database name`.

4. Sur la page **Summary** (Récapitulatif), passez en revue les paramètres sélectionnés, puis cliquez sur `Finish` pour créer la base de données.

   ![Capture d’écran de la page Summary (Récapitulatif)](./media/oracle-asm/createdb03.png)

5. La base de données a été créée. Sur la page **Finish** (Terminer), vous pouvez déverrouiller des comptes supplémentaires pour utiliser cette base de données et modifier les mots de passe. Pour ce faire, sélectionnez **Password Management** (Gestion des mots de passe). Sinon, cliquez sur `close`.

## <a name="delete-the-vm"></a>Supprimer la machine virtuelle

Vous avez correctement configuré Oracle Automated Storage Management sur l’image de base de données Oracle à partir de la Place de marché Microsoft Azure.  Si vous n’avez plus besoin de cette machine virtuelle, vous pouvez utiliser la commande suivante pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées :

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Configurer Oracle DataGuard](configure-oracle-dataguard.md)

[Tutoriel : Configurer Oracle GoldenGate](Configure-oracle-golden-gate.md)

Revoir [Créer l’architecture d’une base de données Oracle](oracle-design.md)
