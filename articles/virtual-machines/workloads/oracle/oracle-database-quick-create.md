---
title: Créer une base de données Oracle dans une machine virtuelle Azure | Microsoft Docs
description: Configurez et exécutez rapidement une base de données Oracle Database 12c dans votre environnement Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: quickstart
ms.date: 10/05/2020
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 6468acb598cee26c46b62d64c748f0e393f27271
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967956"
---
# <a name="create-an-oracle-database-in-an-azure-vm"></a>Créer une base de données Oracle dans une machine virtuelle Azure

Ce guide explique le fonctionnement d’Azure CLI dans le but de déployer une machine virtuelle Azure depuis la [galerie d’images Oracle Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Oracle.OracleDatabase12102EnterpriseEdition?tab=Overview) pour créer une base de données Oracle 12c. Une fois que le serveur est déployé, vous vous connectez par le biais d’une connexion SSH pour configurer la base de données Oracle. 

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande localement, vous devez exécuter Azure CLI version 2.0.4 ou une version ultérieure pour poursuivre la procédure décrite dans ce guide de démarrage rapide. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. 

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>Créer une machine virtuelle

Pour créer une machine virtuelle, utilisez la commande [az vm create](/cli/azure/vm). 

L’exemple suivant permet de créer une machine virtuelle nommée `myVM`. Il crée également des clés SSH si elles n’existent pas déjà à un emplacement de clé par défaut. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.  

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --admin-username azureuser \
    --generate-ssh-keys
```

Une fois que vous avez créé la machine virtuelle, Azure CLI affiche des informations similaires à l’exemple suivant. Notez la valeur pour `publicIpAddress`. Vous utilisez cette adresse pour accéder à la machine virtuelle.

```output
{
  "fqdns": "",
  "id": "/subscriptions/{snip}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Pour créer une session SSH avec la machine virtuelle, utilisez la commande suivante. Remplacez l’adresse IP par la valeur de `publicIpAddress` pour votre machine virtuelle.

```bash
ssh azureuser@<publicIpAddress>
```

## <a name="create-the-database"></a>Création de la base de données

Le logiciel Oracle est déjà installé sur l’image Place de marché. Créer une base de données comme suit. 

1.  Basculez vers l’utilisateur *oracle*, puis démarrez l’écouteur Oracle :

    ```bash
    $ sudo -su oracle
    $ lsnrctl start
    ```

    Le résultat ressemble à ce qui suit :

    ```output
    Copyright (c) 1991, 2014, Oracle.  All rights reserved.

    Starting /u01/app/oracle/product/12.1.0/dbhome_1/bin/tnslsnr: please wait...

    TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Log messages written to /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))

    Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.1.0.2.0 - Production
    Start Date                23-MAR-2017 15:32:08
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Log File         /u01/app/oracle/diag/tnslsnr/myVM/listener/alert/log.xml
    Listening Endpoints Summary...
    (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=myVM.twltkue3xvsujaz1bvlrhfuiwf.dx.internal.cloudapp.net)(PORT=1521)))
    The listener supports no services
    The command completed successfully
    ```
2. Créer un répertoire de données pour les fichiers de données Oracle

    ```bash
        mkdir /u01/app/oracle/oradata
    ```

3.  Créez la base de données :

    ```bash
    dbca -silent \
           -createDatabase \
           -templateName General_Purpose.dbc \
           -gdbname cdb1 \
           -sid cdb1 \
           -responseFile NO_VALUE \
           -characterSet AL32UTF8 \
           -sysPassword OraPasswd1 \
           -systemPassword OraPasswd1 \
           -createAsContainerDatabase true \
           -numberOfPDBs 1 \
           -pdbName pdb1 \
           -pdbAdminPassword OraPasswd1 \
           -databaseType MULTIPURPOSE \
           -automaticMemoryManagement false \
           -storageType FS \
           -datafileDestination "/u01/app/oracle/oradata/" \
           -ignorePreReqs
    ```

    La création de la base de données ne nécessite que quelques minutes.

    La sortie ressemble à ce qui suit :

    ```output
        Copying database files
        1% complete
        2% complete
        8% complete
        13% complete
        19% complete
        27% complete
        Creating and starting Oracle instance
        29% complete
        32% complete
        33% complete
        34% complete
        38% complete
        42% complete
        43% complete
        45% complete
        Completing Database Creation
        48% complete
        51% complete
        53% complete
        62% complete
        70% complete
        72% complete
        Creating Pluggable Databases
        78% complete
        100% complete
        Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for further details.
    ```

4. Fixer les variables oracle

    Avant de vous connecter, vous devez fixer deux variables d’environnement : *ORACLE_HOME* et *ORACLE_SID*.

    ```bash
        ORACLE_SID=cdb1; export ORACLE_SID
    ```

    Vous pouvez également ajouter les variables ORACLE_HOME et ORACLE_SID au fichier .bashrc. Cela aura pour effet de sauvegarder les variables d’environnement lors des connexions futures. Confirmez le fait que les instructions suivantes ont été ajoutées au fichier `~/.bashrc` à l’aide de l’éditeur de votre choix.

    ```bash
    # Add ORACLE_SID. 
    export ORACLE_SID=cdb1 
    ```

## <a name="oracle-em-express-connectivity"></a>Connectivité à Oracle EM Express

Pour un outil de gestion GUI que vous pouvez utiliser pour explorer la base de données, configurez Oracle EM Express. Pour vous connecter à Oracle EM Express, vous devez avoir préalablement configuré le port dans Oracle. 

1. Connectez-vous à votre base de données en utilisant sqlplus :

    ```bash
    sqlplus / as sysdba
    ```

2. Une fois connecté, définissez le port 5502 pour EM Express.

    ```bash
    exec DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
    ```

3. Ouvrez le conteneur PDB1, le cas échéant, mais commencez par vérifier le statut :

    ```bash
    select con_id, name, open_mode from v$pdbs;
    ```

    Le résultat ressemble à ce qui suit :

    ```output
      CON_ID NAME                           OPEN_MODE 
      ----------- ------------------------- ---------- 
      2           PDB$SEED                  READ ONLY 
      3           PDB1                      MOUNT
    ```

4. Si le paramètre OPEN_MODE pour `PDB1` n’est pas défini sur READ WRITE, exécutez les commandes suivantes pour ouvrir PDB1 :

   ```bash
    alter session set container=pdb1;
    alter database open;
   ```

Vous devez taper `quit` à la fin de la session sqlplus, puis `exit` pour vous déconnecter de l’utilisateur Oracle.

## <a name="automate-database-startup-and-shutdown"></a>Automatisation du démarrage et de l’arrêt de la base de données

La base de données Oracle par défaut ne s’enclenche pas automatiquement lorsque vous redémarrez la machine virtuelle. Pour configurer la base de données Oracle afin qu’elle démarre automatiquement, connectez-vous d’abord en tant qu’utilisateur racine. Puis, créez et mettez à jour des fichiers système.

1. Connexion en tant qu’utilisateur racine

    ```bash
    sudo su -
    ```

2.  À l’aide de l’éditeur de votre choix, modifiez le fichier `/etc/oratab` et remplacez la valeur par défaut `N` par `Y` :

    ```bash
    cdb1:/u01/app/oracle/product/12.1.0/dbhome_1:Y
    ```

3.  Créez un fichier nommé `/etc/init.d/dbora` et collez le contenu suivant :

    ```bash
    #!/bin/sh
    # chkconfig: 345 99 10
    # Description: Oracle auto start-stop script.
    #
    # Set ORA_HOME to be equivalent to $ORACLE_HOME.
    ORA_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
    ORA_OWNER=oracle

    case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        # Remove "&" if you don't want startup as a background process.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart $ORA_HOME" &
        touch /var/lock/subsys/dbora
        ;;

    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut $ORA_HOME" &
        rm -f /var/lock/subsys/dbora
        ;;
    esac
    ```

4.  Modifiez les autorisations des fichiers *chmod* comme suit :

    ```bash
    chgrp dba /etc/init.d/dbora
    chmod 750 /etc/init.d/dbora
    ```

5.  Créez des liens symboliques pour le démarrage et l’arrêt, comme suit :

    ```bash
    ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
    ```

6.  Pour tester vos modifications, redémarrez la machine virtuelle :

    ```bash
    reboot
    ```

## <a name="open-ports-for-connectivity"></a>Ouverture des ports pour la connectivité

La dernière étape consiste à configurer quelques points de terminaison externes. Pour configurer le groupe de sécurité réseau qui protège la machine virtuelle, commencez par fermer votre session SSH dans la machine virtuelle (elle doit avoir été exclue de SSH au cours du redémarrage lors de l’étape précédente). 

1.  Pour ouvrir le point de terminaison que vous utilisez afin d’accéder à la base de données Oracle à distance, créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule), comme suit : 

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup\
        --nsg-name myVmNSG \
        --name allow-oracle \
        --protocol tcp \
        --priority 1001 \
        --destination-port-range 1521
    ```

2.  Pour ouvrir le point de terminaison que vous utilisez afin d’accéder à Oracle EM Express à distance, créez une règle de groupe de sécurité réseau avec la commande [az network nsg rule create](/cli/azure/network/nsg/rule), comme suit :

    ```azurecli-interactive
    az network nsg rule create \
        --resource-group myResourceGroup \
        --nsg-name myVmNSG \
        --name allow-oracle-EM \
        --protocol tcp \
        --priority 1002 \
        --destination-port-range 5502
    ```

3. En cas de besoin, obtenez à nouveau l’adresse IP publique de la machine virtuelle avec la commande [az network public-ip show](/cli/azure/network/public-ip), comme suit :

    ```azurecli-interactive
    az network public-ip show \
        --resource-group myResourceGroup \
        --name myVMPublicIP \
        --query [ipAddress] \
        --output tsv
    ```

4.  Connectez-vous à EM Express à partir de votre navigateur. Assurez-vous que votre navigateur est compatible avec EM Express (l’installation de Flash est requise) : 

    ```https
    https://<VM ip address or hostname>:5502/em
    ```

Vous pouvez vous connecter à l’aide du compte **SYS**, puis activer la case à cocher **as sysdba**. Utilisez le mot de passe **OraPasswd1** que vous avez défini lors de l’installation. 

![Capture d’écran de la page de connexion Oracle OEM Express](./media/oracle-quick-start/oracle_oem_express_login.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé d’explorer votre première base de données Oracle sur Azure et que la machine virtuelle n’est plus nécessaire, vous pouvez utiliser la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les autres [solutions Oracle sur Azure](./oracle-overview.md). 

Effectuez le didacticiel relatif à [l’installation et à la configuration d’Oracle Automated Storage Management](configure-oracle-asm.md).
