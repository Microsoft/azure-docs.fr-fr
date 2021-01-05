---
title: Gérer un cluster ML Services sur HDInsight - Azure
description: Découvrez comment gérer différentes tâches sur le cluster ML Services dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/19/2019
ms.openlocfilehash: 898a02796d578d76f9b45d167f4e92a4bf9831ba
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536281"
---
# <a name="manage-ml-services-cluster-on-azure-hdinsight"></a>Gérer un cluster ML Services dans Azure HDInsight

Dans cet article, vous découvrez comment gérer un cluster ML Services existant sur Azure HDInsight pour effectuer certaines tâches comme ajouter plusieurs utilisateurs simultanés, se connecter à distance à un cluster ML Services, changer le contexte de calcul, etc.

## <a name="prerequisites"></a>Prérequis

* Un cluster ML Services sur HDInsight. Consultez [Création clusters Apache Hadoop à l’aide du Portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) et sélectionnez **ML Services** pour le **Type de cluster** .

* Un client Secure Shell (SSH) : un client SSH est utilisé pour se connecter à distance au cluster HDInsight et exécuter des commandes directement sur celui-ci. Pour en savoir plus, consultez [Utiliser SSH avec HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a name="enable-multiple-concurrent-users"></a>Autoriser plusieurs utilisateurs simultanés

Vous pouvez autoriser plusieurs utilisateurs simultanés pour le cluster ML Server sur HDInsight en les ajoutant au nœud de périphérie sur lequel la version de RStudio Community s’exécute. Lorsque vous créez un cluster HDInsight, vous devez fournir deux utilisateurs, un utilisateur HTTP et un utilisateur SSH :

![HDI - Portail Azure - Paramètres de connexion](./media/r-server-hdinsight-manage/hdi-concurrent-users1.png)

- **Nom d’utilisateur de connexion du cluster**  : un utilisateur HTTP pour l’authentification via la passerelle HDInsight qui est utilisée pour protéger les clusters HDInsight que vous avez créés. Cet utilisateur HTTP est utilisé pour accéder à l’IU Apache Ambari, à l’IU Apache Hadoop YARN ainsi qu’à d’autres composants d’interface utilisateur.
- **Nom d’utilisateur Secure Shell (SSH)**  : un utilisateur SSH pour l’accès au cluster via Secure Shell. Ce dernier correspond à un utilisateur dans le système Linux pour tous les nœuds principaux, les nœuds Worker et les nœuds de périmètre. Par conséquent, vous pouvez utiliser Secure Shell pour accéder à n’importe quel nœud dans un cluster distant.

La version de R Studio Server Community utilisée dans le cluster ML Services sur HDInsight n’accepte comme mécanisme de connexion que les noms d’utilisateur et mots de passe Linux. Elle ne prend pas en charge le passage de jetons. Par conséquent, quand vous essayez d’accéder à R Studio pour la première fois sur un cluster ML Services, vous devez vous connecter deux fois.

- Connectez-vous d’abord à l’aide des informations d’identification de l’utilisateur HTTP via la passerelle HDInsight.

- Utilisez ensuite les informations d’identification d’utilisateur SSH pour vous connecter à RStudio.
  
Actuellement, on ne peut créer qu’un seul compte d’utilisateur SSH lors de l’approvisionnement d’un cluster HDInsight. Ainsi, pour permettre à plusieurs utilisateurs d’accéder au cluster ML Services sur HDInsight, vous devez créer des utilisateurs supplémentaires dans le système Linux.

Comme RStudio s’exécute sur le nœud de périphérie du cluster, il y a plusieurs étapes à suivre ici :

1. Utiliser l’utilisateur SSH existant pour se connecter au nœud de périphérie
2. Ajouter d’autres utilisateurs Linux dans le nœud de périmètre
3. Utiliser la version RStudio Community avec l’utilisateur créé

### <a name="step-1-use-the-created-ssh-user-to-sign-in-to-the-edge-node"></a>Étape 1 : Utiliser l’utilisateur SSH créé pour se connecter au nœud de périphérie

Suivez les instructions de l’article [Se connecter à HDInsight (Apache Hadoop) à l’aide de SSH](../hdinsight-hadoop-linux-use-ssh-unix.md) pour accéder au nœud de périphérie. L’adresse du nœud de périphérie pour le cluster ML Services sur HDInsight est `CLUSTERNAME-ed-ssh.azurehdinsight.net`.

### <a name="step-2-add-more-linux-users-in-edge-node"></a>Étape 2 : Ajouter d’autres utilisateurs Linux dans le nœud de périmètre

Pour ajouter un utilisateur au nœud de périmètre, exécutez les commandes suivantes :

```bash
# Add a user 
sudo useradd <yournewusername> -m

# Set password for the new user
sudo passwd <yournewusername>
```

La capture d’écran qui suit présente les résultats.

![Capture d’écran montrant les utilisateurs simultanés](./media/r-server-hdinsight-manage/hdi-concurrent-users2.png)

Ignorez le message vous invitant à entrer le « mot de passe Kerberos actuel » en appuyant simplement sur la touche **Entrée** . L’option `-m` de la commande `useradd` indique que le système va créer un dossier de base pour l’utilisateur, requis pour RStudio Community.

### <a name="step-3-use-rstudio-community-version-with-the-user-created"></a>Étape 3 : Utiliser la version RStudio Community avec l’utilisateur créé

Accédez à RStudio à partir de `https://CLUSTERNAME.azurehdinsight.net/rstudio/`. Si vous vous connectez pour la première fois après avoir créé le cluster, entrez les informations d’identification de l’administrateur du cluster, suivies des informations d’identification de l’utilisateur SSH que vous avez créées. S’il ne s’agit pas de votre première connexion, entrez uniquement les informations d’identification de l’utilisateur SSH que vous avez créé.

Vous pouvez aussi vous connecter simultanément en utilisant les informations d’identification d’origine (par défaut, *sshuser* ) à partir d’une autre fenêtre de navigateur.

Notez également que les utilisateurs récemment ajoutés ne possèdent pas les privilèges racine dans le système Linux. Cependant, ils bénéficient du même accès à l’ensemble des fichiers dans les stockages distants HDFS et WASB.

## <a name="connect-remotely-to-microsoft-ml-services"></a>Se connecter à distance à Microsoft ML Services

Vous pouvez configurer l’accès au contexte de calcul HDInsight Spark à partir d’une instance distante de ML Client s’exécutant sur votre bureau. Pour ce faire, vous devez spécifier les options hdfsShareDir, shareDir, sshUsername, sshHostname, sshSwitches et sshProfileScript quand vous définissez le contexte de calcul RxSpark sur votre ordinateur de bureau. Par exemple :

```r
myNameNode <- "default"
myPort <- 0

mySshHostname  <- '<clustername>-ed-ssh.azurehdinsight.net'  # HDI secure shell hostname
mySshUsername  <- '<sshuser>'# HDI SSH username
mySshSwitches  <- '-i /cygdrive/c/Data/R/davec'   # HDI SSH private key

myhdfsShareDir <- paste("/user/RevoShare", mySshUsername, sep="/")
myShareDir <- paste("/var/RevoShare" , mySshUsername, sep="/")

mySparkCluster <- RxSpark(
    hdfsShareDir = myhdfsShareDir,
    shareDir     = myShareDir,
    sshUsername  = mySshUsername,
    sshHostname  = mySshHostname,
    sshSwitches  = mySshSwitches,
    sshProfileScript = '/etc/profile',
    nameNode     = myNameNode,
    port         = myPort,
    consoleOutput= TRUE
)
```

Pour plus d’informations, consultez la section « Utilisation de Microsoft Machine Learning Server comme client Apache Hadoop » dans [Guide pratique pour utiliser RevoScaleR dans un contexte de calcul Apache Spark](/machine-learning-server/r/how-to-revoscaler-spark#more-spark-scenarios)

## <a name="use-a-compute-context"></a>Utiliser un contexte de calcul

Un contexte de calcul vous permet de contrôler si le calcul doit être effectué localement sur le nœud de périmètre, ou s’il doit être distribué entre les nœuds du cluster HDInsight.  Pour un exemple de définition d’un contexte de calcul avec RStudio Server, consultez [Exécuter un script R sur un cluster de ML Services dans Azure HDInsight avec RStudio Server](machine-learning-services-quickstart-job-rstudio.md).

## <a name="distribute-r-code-to-multiple-nodes"></a>Distribuer le code R à plusieurs nœuds

Avec ML Services sur HDInsight, vous pouvez utiliser du code R existant et l’exécuter sur plusieurs nœuds du cluster à l’aide de la commande `rxExec`. Cette fonction est utile lors d’un balayage paramétrique ou lorsque vous effectuez des simulations. Le code suivant montre comment utiliser `rxExec` :

```r
rxExec( function() {Sys.info()["nodename"]}, timesToRun = 4 )
```

Si vous utilisez toujours le contexte Spark, cette commande renvoie à la valeur nodename (nom de nœud) des nœuds Worker sur lesquels le code `(Sys.info()["nodename"])` s’exécute. Par exemple, sur un cluster à quatre nœuds, vous vous attendez à recevoir une sortie similaire à l’extrait de code suivant :

```r
$rxElem1
    nodename
"wn3-mymlser"

$rxElem2
    nodename
"wn0-mymlser"

$rxElem3
    nodename
"wn3-mymlser"

$rxElem4
    nodename
"wn3-mymlser"
```

## <a name="access-data-in-apache-hive-and-parquet"></a>Accès aux données dans Apache Hive et Parquet

HDInsight ML Services permet un accès direct aux données de Hive et Parquet pour une utilisation par les fonctions de ScaleR dans le contexte de calcul Spark. Ces fonctionnalités sont disponibles via les nouvelles fonctions de source de données ScaleR appelées RxHiveData et RxParquetData qui fonctionnent à l’aide de Spark SQL pour charger les données directement dans un tableau de données Spark pour analyse par ScaleR.

Le code suivant offre quelques exemples de code relatifs à l’utilisation des nouvelles fonctions :

```r
#Create a Spark compute context:
myHadoopCluster <- rxSparkConnect(reset = TRUE)

#Retrieve some sample data from Hive and run a model:
hiveData <- RxHiveData("select * from hivesampletable",
                       colInfo = list(devicemake = list(type = "factor")))
rxGetInfo(hiveData, getVarInfo = TRUE)

rxLinMod(querydwelltime ~ devicemake, data=hiveData)

#Retrieve some sample data from Parquet and run a model:
rxHadoopMakeDir('/share')
rxHadoopCopyFromLocal(file.path(rxGetOption('sampleDataDir'), 'claimsParquet/'), '/share/')
pqData <- RxParquetData('/share/claimsParquet',
                        colInfo = list(
                            age    = list(type = "factor"),
                            car.age = list(type = "factor"),
                            type = list(type = "factor")
                        ) )
rxGetInfo(pqData, getVarInfo = TRUE)

rxNaiveBayes(type ~ age + cost, data = pqData)

#Check on Spark data objects, cleanup, and close the Spark session:
lsObj <- rxSparkListData() # two data objs are cached
lsObj
rxSparkRemoveData(lsObj)
rxSparkListData() # it should show empty list
rxSparkDisconnect(myHadoopCluster)
```

Pour plus d’informations sur l’utilisation de ces nouvelles fonctions, consultez l’aide en ligne de ML Services via les commandes `?RxHivedata` et `?RxParquetData`.  

## <a name="install-additional-r-packages-on-the-cluster"></a>Installer des packages R supplémentaires sur le cluster

### <a name="to-install-r-packages-on-the-edge-node"></a>Pour installer des packages R sur le nœud de périphérie

Si vous souhaitez installer des packages R supplémentaires sur le nœud de périphérie, vous pouvez utiliser `install.packages()` directement à partir de la console R une fois que vous vous êtes connecté au nœud de périphérie par le biais de SSH. 

### <a name="to-install-r-packages-on-the-worker-node"></a>Pour installer des packages R sur le nœud Worker

Pour installer des packages R sur les nœuds Worker du cluster, vous devez utiliser une action de script. Les actions de script sont des scripts de commandes par lot permettant de modifier la configuration du cluster HDInsight ou d’installer d’autres logiciels, comme des packages R supplémentaires. 

> [!IMPORTANT]  
> L’utilisation d’actions de script pour installer des packages R supplémentaires n’est possible qu’après la création du cluster. N’utilisez pas cette procédure au moment de créer le cluster, car le script a besoin que ML Services soit entièrement configuré.

1. Suivez les étapes fournies dans [Personnaliser des clusters à l’aide d’une action de script](../hdinsight-hadoop-customize-cluster-linux.md).

3. Pour **Envoyer une action de script** , entrez les informations suivantes :

   * Pour **Type de script** , sélectionnez **Personnalisé** .

   * Dans le champ **Nom** , renseignez un nom pour l’action de script.

     * Pour **URI de script bash** , entrez `https://mrsactionscripts.blob.core.windows.net/rpackages-v01/InstallRPackages.sh`. Il s’agit du script permettant d’installer les packages R supplémentaires sur le nœud Worker.

   * Sélectionnez uniquement la case à cocher en regard de **Worker** .

   * **Paramètres**  : les packages R à installer. Par exemple : `bitops stringr arules`

   * Sélectionnez la case à cocher en regard de **Conservez cette action de script** .  

   > [!NOTE]
   > 1. Par défaut, tous les packages R sont installés à partir d’un instantané du référentiel Microsoft MRAN compatible avec la version de ML Server qui a été installée. Si vous souhaitez installer des versions plus récentes des packages, vous risquez de rencontrer des problèmes de compatibilité. Toutefois, vous pouvez effectuer ce type d’installation en spécifiant `useCRAN` comme premier élément de la liste des packages, par exemple `useCRAN bitops, stringr, arules`.  
   > 2. Certains packages R nécessitent des bibliothèques de système Linux supplémentaires. Pour plus de commodité, HDInsight ML Services est préinstallé avec les dépendances nécessaires aux 100 packages R les plus demandés. Toutefois, si les packages R que vous installez nécessitent des bibliothèques supplémentaires, vous devez télécharger le script de base utilisé ici et ajouter des étapes pour installer les bibliothèques système. Vous devez ensuite charger le script modifié dans un conteneur d’objets blob publics dans Azure Storage, puis utiliser le script modifié pour installer les packages.
   >    Pour plus d’informations sur le développement d’actions de script, consultez la section [Développer des actions de script](../hdinsight-hadoop-script-actions-linux.md).

   ![Portail Azure - Envoyer une action de script](./media/r-server-hdinsight-manage/submit-script-action.png)

4. Sélectionnez **Créer** pour exécuter le script. Une fois le script exécuté, les packages R sont disponibles sur tous les nœuds de travail.

## <a name="next-steps"></a>Étapes suivantes

* [Rendre opérationnel un cluster ML Services sur HDInsight](r-server-operationalize.md)
* [Options de contexte de calcul pour un cluster ML Services sur HDInsight](r-server-compute-contexts.md)
* [Options de stockage Azure pour un cluster ML Services sur HDInsight](r-server-storage.md)