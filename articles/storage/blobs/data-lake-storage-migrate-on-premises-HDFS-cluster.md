---
title: Migrer un magasin HDFS local vers Stockage Azure à l’aide d’Azure Data Box
description: Migrez les données d’un magasin HDFS local vers Stockage Azure (Stockage Blob Azure ou Data Lake Storage Gen2) en utilisant un appareil Data Box.
author: normesta
ms.service: storage
ms.date: 02/14/2019
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: jamesbak
ms.openlocfilehash: e58137dd680ff9a2be2bd657f0969304b526873f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913111"
---
# <a name="migrate-from-on-prem-hdfs-store-to-azure-storage-with-azure-data-box"></a>Migrer un magasin HDFS local vers Stockage Azure à l’aide d’Azure Data Box

Vous pouvez migrer les données d'un magasin HDFS en local de votre cluster Hadoop vers Stockage Azure (Stockage Blob Azure ou Data Lake Storage Gen2) en utilisant un appareil Data Box. Vous pouvez choisir entre les appareils Data Box Disk, Data Box de 80 To ou Data Box Heavy de 770 To.

Cet article vous aide à effectuer les étapes suivantes :

> [!div class="checklist"]
> * Se préparer à la migration de vos données.
> * Copier vos données sur un appareil Data Box Disk, Data Box ou Data Box Heavy.
> * Retourner l'appareil à Microsoft
> * Appliquer des autorisations d’accès aux fichiers et aux répertoires (Data Lake Storage Gen2 uniquement)

## <a name="prerequisites"></a>Prérequis

Vous avez besoin des éléments suivants pour finaliser la migration.

* Un compte de stockage Azure.

* Un cluster Hadoop en local contenant vos données source.

* Une [Azure Data Box](https://azure.microsoft.com/services/storage/databox/).

  * [Commandez votre Data Box](../../databox/data-box-deploy-ordered.md) ou [Data Box Heavy](../../databox/data-box-heavy-deploy-ordered.md). 

  * Branchez et connectez votre [Data Box](../../databox/data-box-deploy-set-up.md) ou [Data Box Heavy](../../databox/data-box-heavy-deploy-set-up.md) à un réseau local.

Si vous êtes prêt, commençons.

## <a name="copy-your-data-to-a-data-box-device"></a>Copier vos données vers une Azure Data Box

Si vos données tiennent dans une seule Data Box, copiez les données vers la Data Box. 

Si la taille de vos données dépasse la capacité de la Data Box, utilisez la procédure optionnelle [ pour répartir les données entre plusieurs Data Box](#appendix-split-data-across-multiple-data-box-devices), puis effectuez cette étape. 

Pour copier les données de votre magasin HDFS en local vers une Data Box, vous devez configurer quelques paramètres, puis lancer l'outil [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html).

Suivez ces étapes pour copier des données via les API REST du stockage Blob/Object vers votre Data Box. L'interface de l'API REST fera apparaître l’appareil comme un magasin HDFS sur votre cluster.

1. Avant de copier les données via REST, identifiez les primitives de sécurité et de connexion pour accéder à l'interface REST sur la Data Box ou la Data Box Heavy. Connectez-vous à l'interface web locale de la Data Box, puis accédez à page **Connexion et copie**. Dans le compte de stockage Azure de votre appareil, sous **Paramètres d’accès**, localisez puis sélectionnez **REST**.

    ![Page « Connexion et copie ».](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connect-rest.png)

2. Dans la boîte de dialogue Accéder au compte de stockage et charger des données, copiez le **point de terminaison du service BLOB** et la **clé du compte de stockage**. Dans le point de terminaison du service blob, supprimez la partie `https://` et la barre oblique finale.

    Dans ce cas, le point de terminaison est : `https://mystorageaccount.blob.mydataboxno.microsoftdatabox.com/`. La partie hôte de l'URI que vous utiliserez est : `mystorageaccount.blob.mydataboxno.microsoftdatabox.com`. Pour obtenir un exemple, consultez la section [Se connecter à REST via http](../../databox/data-box-deploy-copy-data-via-rest.md). 

     ![Boîte de dialogue « Accéder au compte de stockage et charger des données »](media/data-lake-storage-migrate-on-premises-HDFS-cluster/data-box-connection-string-http.png)

3. Ajoutez le point de terminaison et l'adresse IP du nœud Data Box ou Data Box Heavy à `/etc/hosts` sur chaque nœud.

    ```    
    10.128.5.42  mystorageaccount.blob.mydataboxno.microsoftdatabox.com
    ```

    Si vous utilisez un autre mécanisme pour le DNS, vous devez vous assurer que le point de terminaison Data Box peut être résolu.

4. Définissez la variable d’environnement `azjars` sur l’emplacement des fichiers jar `hadoop-azure` et `azure-storage`. Vous trouverez ces fichiers dans le répertoire d'installation de Hadoop.

    Pour déterminer si ces fichiers existent, utilisez la commande suivante : `ls -l $<hadoop_install_dir>/share/hadoop/tools/lib/ | grep azure`. Remplacez l’espace réservé `<hadoop_install_dir>` par le chemin du répertoire où vous avez installé Hadoop. Veillez à utiliser des chemins d'accès complets.

    Exemples :

    `azjars=$hadoop_install_dir/share/hadoop/tools/lib/hadoop-azure-2.6.0-cdh5.14.0.jar` `azjars=$azjars,$hadoop_install_dir/share/hadoop/tools/lib/microsoft-windowsazure-storage-sdk-0.6.0.jar`

5. Créez le conteneur de stockage à utiliser pour la copie des données. Vous devez également spécifier un répertoire de destination dans le cadre de cette commande. À ce stade, il peut s’agir d’un répertoire de destination fictif.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -mkdir -p  wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Remplacez l’espace réservé `<blob_service_endpoint>` par le nom de votre point de terminaison de service blob.

    * Remplacez l’espace réservé `<account_key>` par la clé d’accès de votre compte.

    * Remplacez l’espace réservé `<container-name>` par le nom de votre conteneur.

    * Remplacez l’espace réservé `<destination_directory>` par le nom du répertoire où vous voulez copier vos données.

6. Exécutez une commande list pour vérifier que votre conteneur et votre répertoire ont été créés.

    ```
    hadoop fs -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint>=<account_key> \
    -ls -R  wasb://<container_name>@<blob_service_endpoint>/
    ```

   * Remplacez l’espace réservé `<blob_service_endpoint>` par le nom de votre point de terminaison de service blob.

   * Remplacez l’espace réservé `<account_key>` par la clé d’accès de votre compte.

   * Remplacez l’espace réservé `<container-name>` par le nom de votre conteneur.

7. Copiez les données du magasin HDFS Hadoop vers le stockage d’objets blob Data Box, dans le conteneur que vous avez créé précédemment. Si le répertoire de copie est introuvable, la commande le crée automatiquement.

    ```
    hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.<blob_service_endpoint<>=<account_key> \
    -filters <exclusion_filelist_file> \
    [-f filelist_file | /<source_directory> \
           wasb://<container_name>@<blob_service_endpoint>/<destination_directory>
    ```

    * Remplacez l’espace réservé `<blob_service_endpoint>` par le nom de votre point de terminaison de service blob.

    * Remplacez l’espace réservé `<account_key>` par la clé d’accès de votre compte.

    * Remplacez l’espace réservé `<container-name>` par le nom de votre conteneur.

    * Remplacez l’espace réservé `<exlusion_filelist_file>` par le nom du fichier contenant votre liste d'exclusions de fichiers.

    * Remplacez l’espace réservé `<source_directory>` par le nom du répertoire contenant les données à copier.

    * Remplacez l’espace réservé `<destination_directory>` par le nom du répertoire où vous voulez copier vos données.

    L’option `-libjars` sert à rendre les fichiers `hadoop-azure*.jar` et les fichiers `azure-storage*.jar` dépendants disponibles pour `distcp`. Cela peut déjà être le cas pour certains clusters.

    L'exemple suivant montre comment la commande `distcp` est utilisée pour copier des données.

    ```
     hadoop distcp \
    -libjars $azjars \
    -D fs.AbstractFileSystem.wasb.Impl=org.apache.hadoop.fs.azure.Wasb \
    -D fs.azure.account.key.mystorageaccount.blob.mydataboxno.microsoftdatabox.com=myaccountkey \
    -filter ./exclusions.lst -f /tmp/copylist1 -m 4 \
    /data/testfiles \
    wasb://hdfscontainer@mystorageaccount.blob.mydataboxno.microsoftdatabox.com/data
    ```
  
    Pour accélérer la copie :

    * Essayez d’augmenter le nombre de mappeurs. (L’exemple ci-dessus utilise `m` = 4 mappeurs.)

    * Essayez d’exécuter plusieurs éléments `distcp` en parallèle.

    * Rappelez-vous que les gros fichiers sont plus performants que les petits fichiers.

## <a name="ship-the-data-box-to-microsoft"></a>Expédier la Data Box à Microsoft

Suivez ces étapes pour préparer et expédier la Data Box à Microsoft.

1. Tout d’abord, [préparez l’expédition de votre Data Box ou Data Box Heavy](../../databox/data-box-deploy-copy-data-via-rest.md).

2. Une fois la préparation de l’appareil terminée, téléchargez les fichiers de nomenclature. Vous utiliserez ces fichiers de nomenclature ou de manifeste plus tard pour vérifier les données chargées vers Azure.

3. Mettez l’appareil hors tension et débranchez les câbles.

4. Planifiez un enlèvement avec UPS.

    * Pour les Data Box, consultez [Expédier votre Data Box](../../databox/data-box-deploy-picked-up.md).

    * Pour les Data Box Heavy, consultez [Expédier votre Data Box Heavy](../../databox/data-box-heavy-deploy-picked-up.md).

5. Une fois que Microsoft a reçu votre appareil, ce dernier est connecté au réseau du centre de données et les données sont chargées vers le compte de stockage que vous avez spécifié au moment où vous avez commandé l’appareil. À l’aide des fichiers de nomenclature, vérifiez que toutes vos données sont chargées vers Azure. 

## <a name="apply-access-permissions-to-files-and-directories-data-lake-storage-gen2-only"></a>Appliquer des autorisations d’accès aux fichiers et aux répertoires (Data Lake Storage Gen2 uniquement)

Les données figurent déjà dans votre compte de stockage Azure. Vous allez à présent appliquer des autorisations d’accès aux fichiers et aux répertoires.

> [!NOTE]
> Cette étape est nécessaire uniquement si vous utilisez Azure Data Lake Storage Gen2 comme banque de données. Si vous n'utilisez qu'un compte de stockage d’objets blob sans espace de noms hiérarchique comme banque de données, vous pouvez ignorer cette section.

### <a name="create-a-service-principal-for-your-azure-data-lake-storage-gen2-account"></a>Créer un principal de service pour votre compte Azure Data Lake Storage Gen2

Pour créer un principal de service, consultez [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

* Au cours des étapes décrites dans la section [Attribuer un rôle à l’application](../../active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application) de l’article, veillez à affecter le rôle **Contributeur aux données Blob du stockage** au principal de service.

* Au cours des étapes indiquées dans la section [Obtenir les valeurs de connexion](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) de l’article, enregistrez l’ID d’application et le secret client dans un fichier texte. Vous en aurez besoin bientôt.

### <a name="generate-a-list-of-copied-files-with-their-permissions"></a>Générer une liste des fichiers copiés avec leurs autorisations

À partir du cluster Hadoop en local, exécutez cette commande :

```bash

sudo -u hdfs ./copy-acls.sh -s /{hdfs_path} > ./filelist.json
```

Cette commande génère une liste de fichiers copiés avec leurs autorisations.

> [!NOTE]
> Selon le nombre de fichiers dans HDFS, cette commande peut prendre beaucoup de temps à exécuter.

### <a name="generate-a-list-of-identities-and-map-them-to-azure-active-directory-add-identities"></a>Générer une liste d'identités et les mapper aux identités Azure Active Directory (ADD)

1. Téléchargez le script `copy-acls.py`. Consultez la section [Télécharger des scripts d'aide et configurer votre nœud de périphérie pour qu'il les exécute](#download-helper-scripts) de cet article.

2. Exécutez cette commande pour générer une liste d’identités uniques.

   ```bash
   
   ./copy-acls.py -s ./filelist.json -i ./id_map.json -g
   ```

   Ce script génère un fichier nommé `id_map.json` qui contient les identités nécessaires pour mapper des identités basées sur ADD.

3. Ouvrez le fichier `id_map.json` dans un éditeur de texte.

4. Pour chaque objet JSON qui apparaît dans le fichier, mettez à jour l’attribut `target` d’un nom d’utilisateur principal (UPN) ADD ou d’un ID d’objet (OID), avec l’identité mappée correspondante. Quand vous avez terminé, enregistrez le fichier. Vous en aurez besoin dans la prochaine étape.

### <a name="apply-permissions-to-copied-files-and-apply-identity-mappings"></a>Appliquer des autorisations à des fichiers copiés et appliquer des mappages d’identités

Exécutez cette commande pour appliquer des autorisations aux données que vous avez copiées dans le compte Data Lake Storage Gen2 :

```bash
./copy-acls.py -s ./filelist.json -i ./id_map.json  -A <storage-account-name> -C <container-name> --dest-spn-id <application-id>  --dest-spn-secret <client-secret>
```

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

* Remplacez l’espace réservé `<container-name>` par le nom de votre conteneur.

* Remplacez les espaces réservés `<application-id>` et `<client-secret>` par l’ID d’application et la clé secrète client que vous avez collectés lorsque vous avez créé le principal du service.

## <a name="appendix-split-data-across-multiple-data-box-devices"></a>Annexe : Répartir les données sur plusieurs Data Box

Avant de déplacer vos données vers une Data Box, vous devrez télécharger des scripts d'aide, vérifier que vos données sont organisées pour tenir sur une Data Box, puis exclure tout fichier inutile.

<a id="download-helper-scripts"></a>

### <a name="download-helper-scripts-and-set-up-your-edge-node-to-run-them"></a>Télécharger des scripts d'aide et configurer votre nœud de périphérie pour qu'il les exécute

1. À partir du nœud de périphérie ou du nœud principal votre cluster Hadoop en local, exécutez cette commande :

   ```bash
   
   git clone https://github.com/jamesbak/databox-adls-loader.git
   cd databox-adls-loader
   ```

   Cette commande clone le référentiel GitHub contenant contient les scripts d'aide.

2. Assurez-vous que le package [jq](https://stedolan.github.io/jq/) est installé sur votre ordinateur local.

   ```bash
   
   sudo apt-get install jq
   ```

3. Installez le package Python [requests](https://2.python-requests.org/en/master/).

   ```bash
   
   pip install requests
   ```

4. Définissez les autorisations d’exécution sur les scripts requis.

   ```bash
   
   chmod +x *.py *.sh

   ```

### <a name="ensure-that-your-data-is-organized-to-fit-onto-a-data-box-device"></a>Assurez-vous que vos données sont organisées de manière à tenir dans un service Data Box.

Si la taille de vos données dépasse celle d'une seule Data Box, vous pouvez diviser les fichiers en groupes afin de les stocker sur plusieurs Data Box.

Si vos données ne dépassent pas la taille d'une seule Data Box, vous pouvez passer à la section suivante.

1. Avec des autorisations élevées, exécutez le script `generate-file-list` que vous avez téléchargé en suivant les instructions de la section précédente.

   Voici une description des paramètres de la commande :

   ```
   sudo -u hdfs ./generate-file-list.py [-h] [-s DATABOX_SIZE] [-b FILELIST_BASENAME]
                    [-f LOG_CONFIG] [-l LOG_FILE]
                    [-v {DEBUG,INFO,WARNING,ERROR}]
                    path

   where:
   positional arguments:
   path                  The base HDFS path to process.

   optional arguments:
   -h, --help            show this help message and exit
   -s DATABOX_SIZE, --databox-size DATABOX_SIZE
                        The size of each Data Box in bytes.
   -b FILELIST_BASENAME, --filelist-basename FILELIST_BASENAME
                        The base name for the output filelists. Lists will be
                        named basename1, basename2, ... .
   -f LOG_CONFIG, --log-config LOG_CONFIG
                        The name of a configuration file for logging.
   -l LOG_FILE, --log-file LOG_FILE
                        Name of file to have log output written to (default is
                        stdout/stderr)
   -v {DEBUG,INFO,WARNING,ERROR}, --log-level {DEBUG,INFO,WARNING,ERROR}
                        Level of log information to output. Default is 'INFO'.
   ```

2. Copiez les listes de fichiers générées vers HDFS afin qu’elles soient accessibles à la tâche [DistCp](https://hadoop.apache.org/docs/stable/hadoop-distcp/DistCp.html).

   ```
   hadoop fs -copyFromLocal {filelist_pattern} /[hdfs directory]
   ```

### <a name="exclude-unnecessary-files"></a>Exclure les fichiers inutiles

Vous devrez exclure certains répertoires depuis la tâche DisCp. Par exemple, excluez les répertoires qui contiennent des informations d’intégrité empêchant l’exécution du cluster.

Sur le cluster Hadoop local dans lequel vous envisagez de lancer la tâche DistCp, créez un fichier qui spécifie la liste des répertoires à exclure.

Voici un exemple :

```
.*ranger/audit.*
.*/hbase/data/WALs.*
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez le fonctionnement de Data Lake Storage Gen2 avec des clusters HDInsight. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md).