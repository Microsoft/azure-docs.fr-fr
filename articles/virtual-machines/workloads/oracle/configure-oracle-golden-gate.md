---
title: Implémenter Oracle Golden Gate sur une machine virtuelle Linux Azure | Microsoft Docs
description: Configurez et exécutez rapidement une base de données Oracle Golden Gate dans votre environnement Azure.
author: dbakevlar
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.reviewer: cynthn
ms.openlocfilehash: 3fff58c240341776a3bb99c059c179cc4f9d96e9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999142"
---
# <a name="implement-oracle-golden-gate-on-an-azure-linux-vm"></a>Implémenter Oracle Golden Gate sur une machine virtuelle Linux Azure 

L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce guide explique comment utiliser l’interface CLI Azure pour déployer une base de données Oracle 12c depuis une image de la galerie de la Place de marché Azure. 

Ce document vous explique pas à pas comment créer, installer et configurer Oracle Golden Gate sur une machine virtuelle Azure. Dans ce didacticiel, deux machines virtuelles sont configurées dans un groupe à haute disponibilité dans une seule région. Le même didacticiel peut être utilisé pour configurer OracleGolden Gate pour des machines virtuelles dans différentes zones de disponibilité dans une même région Azure ou pour la configuration de machines virtuelles dans deux régions différentes.

Avant de commencer, assurez-vous que l’interface de ligne de commande Azure est installée. Pour plus d’informations, consultez le [Guide d’installation de l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

## <a name="prepare-the-environment"></a>Préparer l’environnement

Pour installer Oracle Golden Gate, vous devez créer deux machines virtuelles Azure sur le même groupe à haute disponibilité. L’image Place de marché que vous utilisez pour créer la machine virtuelle est **Oracle:Oracle-Database-Ee:12.1.0.2:latest**.

Vous devez également savoir utiliser Unix Editor vi et avoir une connaissance élémentaire de x11 (X Windows).

Voici un résumé de la configuration de l’environnement :
> 
> |  | **Site principal** | **Site de réplication** |
> | --- | --- | --- |
> | **Version d’Oracle** |Oracle 12c Version 2 – (12.1.0.2) |Oracle 12c Version 2 – (12.1.0.2)|
> | **Nom de la machine** |myVM1 |myVM2 |
> | **Système d’exploitation** |Oracle Linux 6.x |Oracle Linux 6.x |
> | **SID Oracle** |CDB1 |CDB1 |
> | **Schéma de réplication** |TEST|TEST |
> | **Propriétaire Golden Gate/réplication** |C##GGADMIN |REPUSER |
> | **Processus Golden Gate** |EXTORA |REPORA|


### <a name="sign-in-to-azure"></a>Connexion à Azure 

Connectez-vous à votre abonnement Azure avec la commande [az login](/cli/azure/reference-index). Ensuite, suivez les instructions à l’écran.

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et à partir duquel elles peuvent être gérées. 

L’exemple suivant crée un groupe de ressources nommé `myResourceGroup` à l’emplacement `westus`.

```azurecli
az group create --name myResourceGroup --location westus
```

### <a name="create-an-availability-set"></a>Créer un groupe à haute disponibilité

L’étape suivante est facultative mais recommandée. Pour plus d’informations, consultez [Instructions pour les groupes à haute disponibilité Azure](/previous-versions/azure/virtual-machines/windows/infrastructure-example).

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). 

L’exemple qui suit permet de créer deux machines virtuelles, nommées `myVM1` et `myVM2`. Créez les clés SSH si elles n’existent pas déjà dans un emplacement de clé par défaut. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.

#### <a name="create-myvm1-primary"></a>Créez myVM1 (machine virtuelle principale) :

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM1 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

Lorsque la machine virtuelle a été créée, l’interface CLI Azure affiche des informations similaires à l’exemple suivant. (Notez la valeur de `publicIpAddress`. Cette adresse est utilisée pour accéder à la machine virtuelle.)

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

#### <a name="create-myvm2-replicate"></a>Créez myVM2 (machine virtuelle de réplication) :

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM2 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

Prenez note également de la valeur de `publicIpAddress` après sa création.

### <a name="open-the-tcp-port-for-connectivity"></a>Ouvrir le port TCP pour la connectivité

L’étape suivante consiste à configurer des points de terminaison externes, qui vous permettent d’accéder à distance à la base de données Oracle. Pour configurer les points de terminaison externes, exécutez les commandes suivantes.

#### <a name="open-the-port-for-myvm1"></a>Ouvrez le port pour myVM1 :

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm1NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

Les résultats doivent ressembler à la réponse suivante :

```output
{
  "access": "Allow",
  "description": null,
  "destinationAddressPrefix": "*",
  "destinationPortRange": "1521",
  "direction": "Inbound",
  "etag": "W/\"bd77dcae-e5fd-4bd6-a632-26045b646414\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myVmNSG/securityRules/allow-oracle",
  "name": "allow-oracle",
  "priority": 999,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sourceAddressPrefix": "*",
  "sourcePortRange": "*"
}
```

#### <a name="open-the-port-for-myvm2"></a>Ouvrez le port pour myVM2 :

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm2NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

### <a name="connect-to-the-virtual-machine"></a>Connectez-vous à la machine virtuelle.

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle. Remplacez l’adresse IP par la valeur `publicIpAddress` de votre machine virtuelle.

```bash
ssh <publicIpAddress>
```

### <a name="create-the-database-on-myvm1-primary"></a>Créer la base de données sur myVM1 (machine virtuelle principale)

Le logiciel Oracle est déjà installé sur l’image Place de marché, l’étape suivante consiste donc à installer la base de données. 

Exécutez le logiciel en tant que superutilisateur « oracle » :

```bash
sudo su - oracle
```

Créez la base de données :

```bash
$ dbca -silent \
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
   -ignorePreReqs
```

Les résultats doivent ressembler à la réponse suivante :

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
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for more details.
```

Définissez les variables ORACLE_SID et ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

Si vous le souhaitez, vous pouvez ajouter ORACLE_HOME et ORACLE_SID au fichier .bashrc, de façon à enregistrer ces paramètres pour des connexions futures :

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Démarrer l’écouteur d’Oracle

```bash
$ lsnrctl start
```

### <a name="create-the-database-on-myvm2-replicate"></a>Créer la base de données sur myVM2 (machine virtuelle de réplication)

```bash
sudo su - oracle
```

Créez la base de données :

```bash
$ dbca -silent \
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
   -ignorePreReqs
```

Définissez les variables ORACLE_SID et ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

Si vous le souhaitez, vous pouvez ajouter ORACLE_HOME et ORACLE_SID au fichier .bashrc, de façon à enregistrer ces paramètres pour des connexions futures.

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Démarrer l’écouteur d’Oracle

```bash
$ sudo su - oracle
$ lsnrctl start
```

## <a name="configure-golden-gate"></a>Configurer Golden Gate 
Pour configurer Golden Gate, suivez les étapes de cette section.

### <a name="enable-archive-log-mode-on-myvm1-primary"></a>Activer le mode de journalisation d’archive sur myVM1 (machine virtuelle principale)

```bash
$ sqlplus / as sysdba
SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
NOARCHIVELOG

SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
```
Activez la journalisation forcée et assurez-vous qu’au moins un fichier journal est présent.

```bash
SQL> ALTER DATABASE FORCE LOGGING;
SQL> ALTER SYSTEM SWITCH LOGFILE;
SQL> ALTER SYSTEM set enable_goldengate_replication=true;
SQL> ALTER PLUGGABLE DATABASE PDB1 OPEN;
SQL> ALTER SESSION SET CONTAINER=PDB1;
SQL> ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
SQL> EXIT;
```

### <a name="download-golden-gate-software"></a>Télécharger le logiciel Golden Gate
Pour télécharger et préparer le logiciel Oracle Golden Gate, procédez comme suit :

1. Téléchargez le fichier **fbo_ggs_Linux_x64_shiphome.zip** à partir de la [page de téléchargement Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html). Un ensemble de fichiers .zip à télécharger doit se trouver sous le titre de téléchargement **Oracle GoldenGate 12.x.x.x pour Oracle Linux x86-64**.

2. Après avoir téléchargé les fichiers .zip sur votre ordinateur client, utilisez SCP (Secure Copy Protocol) pour copier les fichiers sur votre machine virtuelle :

   ```bash
   $ scp fbo_ggs_Linux_x64_shiphome.zip <publicIpAddress>:<folder>
   ```

3. Déplacez les fichiers .zip dans le dossier **/opt**. Ensuite, changez le propriétaire des fichiers comme suit :

   ```bash
   $ sudo su -
   # mv <folder>/*.zip /opt
   ```

4. Décompressez les fichiers (installez l’utilitaire unzip de Linux s’il n’est pas déjà installé) :

   ```bash
   # yum install unzip
   # cd /opt
   # unzip fbo_ggs_Linux_x64_shiphome.zip
   ```

5. Changez l’autorisation :

   ```bash
   # chown -R oracle:oinstall /opt/fbo_ggs_Linux_x64_shiphome
   ```

### <a name="prepare-the-client-and-vm-to-run-x11-for-windows-clients-only"></a>Préparer le client et la machine virtuelle à exécuter x11 (pour les clients Windows uniquement)
Il s’agit d’une étape facultative. Vous pouvez l’ignorer si vous utilisez un client Linux ou si x11 est déjà configuré.

1. Téléchargez PuTTY et Xming sur votre ordinateur Windows :

   * [Télécharger PuTTY](https://www.putty.org/)
   * [Télécharger Xming](https://xming.en.softonic.com/)

2. Après avoir installé PuTTY, dans le dossier de PuTTY (par exemple C:\Program Files\PuTTY), exécutez puttygen.exe (PuTTY Key Generator).

3. Dans PuTTY Key Generator :

   - Pour générer une clé, sélectionnez le bouton **Generate** (Générer).
   - Copiez le contenu de la clé (**Ctrl+C**).
   - Sélectionnez le bouton **Save private key** (Enregistrer la clé privée).
   - Ignorer l’avertissement qui s’affiche, puis sélectionnez **OK**.

   ![Capture d’écran de la page du générateur de clé PuTTY](./media/oracle-golden-gate/puttykeygen.png)

4. Dans votre machine virtuelle, exécutez ces commandes :

   ```bash
   # sudo su - oracle
   $ mkdir .ssh (if not already created)
   $ cd .ssh
   ```

5. Créez un fichier nommé **authorized_keys**. Collez le contenu de la clé dans ce fichier, puis enregistrez-le.

   > [!NOTE]
   > La clé doit contenir la chaîne `ssh-rsa`. En outre, le contenu de la clé doit être constitué d’une seule ligne de texte.
   >  

6. Démarrez PuTTY. Dans le panneau **Category** (Catégorie), sélectionnez **Connection** > **SSH** > **Auth** (Connexion > SSH > Authentification). Dans la zone **Private key file for authentication** (Fichier de clé privée pour l’authentification), accédez à la clé que vous avez générée précédemment.

   ![Capture d’écran de la page Set Private Key (Définir la clé privée)](./media/oracle-golden-gate/setprivatekey.png)

7. Dans le panneau **Category** (Catégorie), sélectionnez **Connection** > **SSH** > **X11** (Connexion > SSH > X11). Puis sélectionnez **Enable X11 forwarding** (Activer le transfert X11).

   ![Capture d’écran de la page Enable X11 (Activer X11)](./media/oracle-golden-gate/enablex11.png)

8. Dans le volet **Category** (Catégorie), accédez à **Session**. Entrez les informations de l’hôte, puis sélectionnez **Open** (Ouvrir).

   ![Capture d’écran de la page de session](./media/oracle-golden-gate/puttysession.png)

### <a name="install-golden-gate-software"></a>Installer le logiciel Golden Gate

Pour installer Oracle Golden Gate, procédez comme suit :

1. Connectez-vous en tant qu’oracle. (Vous devez pouvoir vous connecter sans avoir à saisir un mot de passe.) Vérifiez que Xming est en cours d’exécution avant de commencer l’installation.

   ```bash
   $ cd /opt/fbo_ggs_Linux_x64_shiphome/Disk1
   $ ./runInstaller
   ```

2. Sélectionnez « Oracle GoldenGate for Oracle Database 12c ». Puis sélectionnez **Suivant** pour continuer.

   ![Capture d’écran de la page Select Installation (Sélectionner l’installation)](./media/oracle-golden-gate/golden_gate_install_01.png)

3. Modifiez l’emplacement du logiciel. Puis sélectionnez la zone **Start Manager** (Gestionnaire de démarrage) et entrez l’emplacement de la base de données. Sélectionnez **Suivant** pour continuer.

   ![Capture d’écran de la page Select Installation (Sélectionner l’installation)](./media/oracle-golden-gate/golden_gate_install_02.png)

4. Modifiez le répertoire de l’inventaire, puis sélectionnez **Suivant** pour continuer.

   ![Capture d'écran de la page Sélectionner l'installation qui affiche le répertoire d'installation.](./media/oracle-golden-gate/golden_gate_install_03.png)

5. Dans l’écran **Summary** (Récapitulatif), sélectionnez **Install** (Installer) pour continuer.

   ![Capture d'écran représentant la page Sélectionner l'installation et le bouton Installer.](./media/oracle-golden-gate/golden_gate_install_04.png)

6. Vous pourriez être invité à exécuter un script en tant que « root ». Dans ce cas, ouvrez une session distincte, ssh vers la machine virtuelle, sudo vers la racine, puis exécutez le script. Sélectionnez **OK** pour continuer.

   ![Capture d'écran représentant l'emplacement du script et montrant comment exécuter le script de configuration.](./media/oracle-golden-gate/golden_gate_install_05.png)

7. Une fois l’installation terminée, sélectionnez **Close** (Fermer) pour terminer le processus.

   ![Capture d'écran de la page Sélectionner l'installation qui affiche le bouton Fermer.](./media/oracle-golden-gate/golden_gate_install_06.png)

### <a name="set-up-service-on-myvm1-primary"></a>Configurer le service sur myVM1 (machine virtuelle principale)

1. Créez ou mettez à jour le fichier tnsnames.ora :

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Créez les comptes propriétaire et utilisateur de Golden Gate.

   > [!NOTE]
   > Le compte propriétaire doit avoir le préfixe C##.
   >

    ```bash
    $ sqlplus / as sysdba
    SQL> CREATE USER C##GGADMIN identified by ggadmin;
    SQL> EXEC dbms_goldengate_auth.grant_admin_privilege('C##GGADMIN',container=>'ALL');
    SQL> GRANT DBA to C##GGADMIN container=all;
    SQL> connect C##GGADMIN/ggadmin
    SQL> ALTER SESSION SET CONTAINER=PDB1;
    SQL> EXIT;
    ```

3. Créez le compte d’utilisateur de test de Golden Gate :

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> @demo_ora_insert
   SQL> EXIT;
   ```

4. Configurez le fichier de paramètres d’extraction.

   Démarrez l’interface de ligne de commande Golden Gate (ggsci) :

   ```bash
   $ sudo su - oracle
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> DBLOGIN USERID test@pdb1, PASSWORD test
   Successfully logged into database  pdb1
   GGSCI>  ADD SCHEMATRANDATA pdb1.test
   2017-05-23 15:44:25  INFO    OGG-01788  SCHEMATRANDATA has been added on schema test.
   2017-05-23 15:44:25  INFO    OGG-01976  SCHEMATRANDATA for scheduling columns has been added on schema test.

   GGSCI> EDIT PARAMS EXTORA
   ```

5. Ajoutez ce qui suit au fichier de paramètres d’extraction (en utilisant les commandes vi). Appuyez sur la touche ÉCHAP, ':wq!' pour enregistrer le fichier. 

   ```bash
   EXTRACT EXTORA
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTRAIL ./dirdat/rt  
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT 
   LOGALLSUPCOLS
   UPDATERECORDFORMAT COMPACT
   TABLE pdb1.test.TCUSTMER;
   TABLE pdb1.test.TCUSTORD;
   ```

6. Extrait de registre--extrait intégré :

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci

   GGSCI> dblogin userid C##GGADMIN, password ggadmin
   Successfully logged into database CDB$ROOT.

   GGSCI> REGISTER EXTRACT EXTORA DATABASE CONTAINER(pdb1)

   2017-05-23 15:58:34  INFO    OGG-02003  Extract EXTORA successfully registered with database at SCN 1821260.

   GGSCI> exit
   ```

7. Configurer des points de contrôle d’extraction et démarrer l’extraction en temps réel :

   ```bash
   $ ./ggsci
   GGSCI>  ADD EXTRACT EXTORA, INTEGRATED TRANLOG, BEGIN NOW
   EXTRACT (Integrated) added.

   GGSCI>  ADD RMTTRAIL ./dirdat/rt, EXTRACT EXTORA, MEGABYTES 10
   RMTTRAIL added.

   GGSCI>  START EXTRACT EXTORA

   Sending START request to MANAGER ...
   EXTRACT EXTORA starting

   GGSCI > info all

   Program     Status      Group       Lag at Chkpt  Time Since Chkpt

   MANAGER     RUNNING
   EXTRACT     RUNNING     EXTORA      00:00:11      00:00:04
   ```

   Dans cette étape, vous recherchez le SCN de départ, qui sera utilisé ultérieurement, dans une autre section :

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> SELECT current_scn from v$database;
   CURRENT_SCN
   -----------
      1857887
   SQL> EXIT;
   ```

   ```bash
   $ ./ggsci
   GGSCI> EDIT PARAMS INITEXT
   ```

   ```bash
   EXTRACT INITEXT
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTASK REPLICAT, GROUP INITREP
   TABLE pdb1.test.*, SQLPREDICATE 'AS OF SCN 1857887'; 
   ```

   ```bash
   GGSCI> ADD EXTRACT INITEXT, SOURCEISTABLE
   ```

### <a name="set-up-service-on-myvm2-replicate"></a>Configurer le service sur myVM2 (machine virtuelle de réplication)


1. Créez ou mettez à jour le fichier tnsnames.ora :

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Créez un compte de réplication :

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> create user repuser identified by rep_pass container=current;
   SQL> grant dba to repuser;
   SQL> exec dbms_goldengate_auth.grant_admin_privilege('REPUSER',container=>'PDB1');
   SQL> connect repuser/rep_pass@pdb1 
   SQL> EXIT;
   ```

3. Créez un compte d’utilisateur de test de Golden Gate :

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> EXIT;
   ```

4. Fichier de paramètres REPLICAT pour répliquer les modifications : 

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> EDIT PARAMS REPORA  
   ```

   Contenu du fichier de paramètres REPORA :

   ```bash
   REPLICAT REPORA
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/repora.dsc, PURGE, MEGABYTES 100
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT
   DBOPTIONS INTEGRATEDPARAMS(parallelism 6)
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;
   ```

5. Configurez un point de contrôle de réplication :

   ```bash
   GGSCI> ADD REPLICAT REPORA, INTEGRATED, EXTTRAIL ./dirdat/rt
   GGSCI> EDIT PARAMS INITREP

   ```

   ```bash
   REPLICAT INITREP
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/tcustmer.dsc, APPEND
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;   
   ```

   ```bash
   GGSCI> ADD REPLICAT INITREP, SPECIALRUN
   ```

### <a name="set-up-the-replication-myvm1-and-myvm2"></a>Configurer la réplication (myVM1 et myVM2)

#### <a name="1-set-up-the-replication-on-myvm2-replicate"></a>1. Configurer la réplication sur myVM2 (machine virtuelle de réplication)

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> EDIT PARAMS MGR
  ```

Mettez le fichier à jour avec le contenu suivant :

  ```bash
  PORT 7809
  ACCESSRULE, PROG *, IPADDR *, ALLOW
  ```

Puis, redémarrez le service Manager :

  ```bash
  GGSCI> STOP MGR
  GGSCI> START MGR
  GGSCI> EXIT
  ```

#### <a name="2-set-up-the-replication-on-myvm1-primary"></a>2. Configurer la réplication sur myVM1 (machine virtuelle principale)

Démarrez la charge initiale et recherchez les erreurs :

```bash
$ cd /u01/app/oracle/product/12.1.0/oggcore_1
$ ./ggsci
GGSCI> START EXTRACT INITEXT
GGSCI> VIEW REPORT INITEXT
```

#### <a name="3-set-up-the-replication-on-myvm2-replicate"></a>3. Configurer la réplication sur myVM2 (machine virtuelle de réplication)

Remplacez le numéro SCN par le numéro que vous avez obtenu précédemment :

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  START REPLICAT REPORA, AFTERCSN 1857887
  ```

La réplication a commencé et vous pouvez la tester en insérant de nouveaux enregistrements dans les tables TEST.


### <a name="view-job-status-and-troubleshooting"></a>Afficher l’état du travail et la résolution des problèmes

#### <a name="view-reports"></a>Afficher les rapports
Pour afficher des rapports sur myVM1, exécutez les commandes suivantes :

  ```bash
  GGSCI> VIEW REPORT EXTORA 
  ```
 
Pour afficher des rapports sur myVM2, exécutez les commandes suivantes :

  ```bash
  GGSCI> VIEW REPORT REPORA
  ```

#### <a name="view-status-and-history"></a>Afficher l’état et l’historique
Pour afficher l’état et l’historique sur myVM1, exécutez les commandes suivantes :

  ```bash
  GGSCI> dblogin userid c##ggadmin, password ggadmin 
  GGSCI> INFO EXTRACT EXTORA, DETAIL
  ```

Pour afficher l’état et l’historique sur myVM2, exécutez les commandes suivantes :

  ```bash
  GGSCI> dblogin userid repuser@pdb1 password rep_pass 
  GGSCI> INFO REP REPORA, DETAIL
  ```
Cela achève l’installation et la configuration de Golden Gate sur Oracle Linux.


## <a name="delete-the-virtual-machine"></a>Supprimer la machine virtuelle

La commande suivante permet de supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées lorsque vous n’en avez plus besoin.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

[Didacticiel de création de machines virtuelles hautement disponibles](../../linux/create-cli-complete.md)

[Découvrir des exemples de commande de déploiement de machine virtuelle](../../linux/cli-samples.md)
