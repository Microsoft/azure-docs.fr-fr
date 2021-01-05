---
title: Développer des actions de script pour personnaliser des clusters Azure HDInsight
description: Découvrez comment utiliser des scripts Bash pour personnaliser des clusters HDInsight. Les actions de script vous permettent d’exécuter des scripts pendant ou après la création du cluster pour modifier ses paramètres de configuration ou pour installer des logiciels supplémentaires.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 11/28/2019
ms.openlocfilehash: f7959b639b75d912d44670c8b00a7327cb7857d6
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629440"
---
# <a name="script-action-development-with-hdinsight"></a>Développement d’actions de script avec HDInsight

Découvrez comment personnaliser votre cluster HDInsight à l’aide de scripts bash. Les actions de script sont un moyen de personnaliser HDInsight pendant ou après la création du cluster.

## <a name="what-are-script-actions"></a>Définition des actions de script

Les actions de script sont des scripts d'interpréteur de commandes qu’Azure exécute sur les nœuds de cluster pour apporter des modifications de configuration ou installer des logiciels. Une action de script est exécutée en tant qu’administrateur et offre des droits d’accès complets aux nœuds du cluster.

Les actions de script peuvent être appliquées selon les méthodes suivantes :

| Utilisez cette méthode pour appliquer un script... | Pendant la création du cluster... | Sur un cluster en cours d'exécution... |
| --- |:---:|:---:|
| Portail Azure |✓ |✓ |
| Azure PowerShell |✓ |✓ |
| Azure Classic CLI |&nbsp; |✓ |
| Kit HDInsight .NET SDK |✓ |✓ |
| Modèle Azure Resource Manager |✓ |&nbsp; |

Pour plus d’informations sur l’utilisation de ces méthodes pour appliquer des actions de script, consultez [Personnalisation de clusters HDInsight à l’aide d’actions de script](hdinsight-hadoop-customize-cluster-linux.md).

## <a name="best-practices-for-script-development"></a><a name="bestPracticeScripting"></a>Meilleures pratiques relatives au développement de scripts

Quand vous développez un script personnalisé pour un cluster HDInsight, tenez compte des meilleures pratiques suivantes :

* [Cibler la version Apache Hadoop](#bPS1)
* [Cibler la version du système d’exploitation](#bps10)
* [Fournir des liens stables vers les ressources de script](#bPS2)
* [Utiliser des ressources précompilées](#bPS4)
* [S’assurer que le script de personnalisation du cluster est idempotent](#bPS3)
* [Garantir la haute disponibilité de l’architecture du cluster](#bPS5)
* [Configurer les composants personnalisés pour utiliser le stockage d’objets blob Azure](#bPS6)
* [Écrire des informations sur STDOUT et STDERR](#bPS7)
* [Enregistrer des fichiers au format ASCII avec les fins de ligne LF](#bps8)
* [Utilisation de la logique de nouvelle tentative pour récupérer après une erreur temporaire](#bps9)

> [!IMPORTANT]  
> Les actions de script doivent se terminer dans les 60 minutes, ou le processus échoue. Lors de l’approvisionnement du nœud, le script s’exécute en même temps que les autres processus d'installation et de configuration. En raison de cette concurrence pour les ressources, par exemple au niveau du temps processeur ou de la bande passante, l’exécution du script risque de prendre plus de temps que dans votre environnement de développement.

### <a name="target-the-apache-hadoop-version"></a><a name="bPS1"></a>Cibler la version Apache Hadoop

Différentes versions de HDInsight sont équipées de différentes versions de services Hadoop et des composants installés. Si votre script attend une version de service ou de composant spécifique, vous devez utiliser uniquement le script avec la version de HDInsight qui inclut les composants requis. Vous trouverez des informations sur les versions de composant incluses dans HDInsight à l’aide du document [Contrôle des versions du composant HDInsight](hdinsight-component-versioning.md) .

### <a name="checking-the-operating-system-version"></a>Vérification de la version du système d’exploitation

Différentes versions de HDInsight s’appuient sur des versions spécifiques d’Ubuntu. Il peut exister des différences entre les versions de système d’exploitation que vous devez vérifier dans votre script. Par exemple, vous devrez peut-être installer un fichier binaire qui est lié à la version d’Ubuntu.

Pour vérifier la version du système d’exploitation, utilisez `lsb_release`. Par exemple, le script suivant montre comment référencer un fichier tar spécifique selon la version du système d’exploitation :

```bash
OS_VERSION=$(lsb_release -sr)
if [[ $OS_VERSION == 14* ]]; then
    echo "OS version is $OS_VERSION. Using hue-binaries-14-04."
    HUE_TARFILE=hue-binaries-14-04.tgz
elif [[ $OS_VERSION == 16* ]]; then
    echo "OS version is $OS_VERSION. Using hue-binaries-16-04."
    HUE_TARFILE=hue-binaries-16-04.tgz
fi
```

### <a name="target-the-operating-system-version"></a><a name="bps10"></a> Cibler la version du système d’exploitation

HDInsight est basé sur la distribution Ubuntu Linux. Les différentes versions de HDInsight s’appuient sur des versions différentes d’Ubuntu, ce qui peut avoir une incidence sur le comportement de votre script. Par exemple, HDInsight 3.4 et les versions antérieures sont basées sur des versions Ubuntu qui utilisent Upstart. Les versions 3.5 et supérieures sont basées sur Ubuntu 16.04, qui utilise Systemd. Systemd et Upstart reposent sur différentes commandes, donc votre script doit être écrit pour travailler avec les deux.

L’autre différence majeure entre HDInsight 3.4 et 3.5 résident dans le fait que `JAVA_HOME` pointe désormais vers Java 8. Le code suivant montre comment déterminer si le script est exécuté sur Ubuntu 14 ou 16 :

```bash
OS_VERSION=$(lsb_release -sr)
if [[ $OS_VERSION == 14* ]]; then
    echo "OS version is $OS_VERSION. Using hue-binaries-14-04."
    HUE_TARFILE=hue-binaries-14-04.tgz
elif [[ $OS_VERSION == 16* ]]; then
    echo "OS version is $OS_VERSION. Using hue-binaries-16-04."
    HUE_TARFILE=hue-binaries-16-04.tgz
fi
...
if [[ $OS_VERSION == 16* ]]; then
    echo "Using systemd configuration"
    systemctl daemon-reload
    systemctl stop webwasb.service    
    systemctl start webwasb.service
else
    echo "Using upstart configuration"
    initctl reload-configuration
    stop webwasb
    start webwasb
fi
...
if [[ $OS_VERSION == 14* ]]; then
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
elif [[ $OS_VERSION == 16* ]]; then
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
fi
```

Vous trouverez le script complet contenant ces extraits de code à l’adresse https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh.

Pour la version d’Ubuntu utilisée par HDInsight, consultez le document [Version du composant HDInsight](hdinsight-component-versioning.md).

Pour comprendre les différences entre Systemd et Upstart, consultez [Systemd pour les utilisateurs Upstart](https://wiki.ubuntu.com/SystemdForUpstartUsers).

### <a name="provide-stable-links-to-script-resources"></a><a name="bPS2"></a>Fournir des liens stables vers les ressources de script

Le script et les ressources associées doivent rester disponibles pendant toute la durée de vie du cluster. Ces ressources sont requises si de nouveaux nœuds sont ajoutés au cluster pendant les opérations de mise à l'échelle.

La meilleure pratique consiste à tout télécharger et à tout archiver dans un compte de stockage Azure de votre abonnement.

> [!IMPORTANT]  
> Le compte de stockage utilisé doit être le compte de stockage par défaut du cluster ou un conteneur public en lecture seule d’un autre compte de stockage.

Par exemple, les exemples fournis par Microsoft sont stockés dans le compte de stockage [https://hdiconfigactions.blob.core.windows.net/](https://hdiconfigactions.blob.core.windows.net/). Cet emplacement est un conteneur public en lecture seule géré par l’équipe de HDInsight.

### <a name="use-pre-compiled-resources"></a><a name="bPS4"></a>Utiliser des ressources précompilées

Pour réduire le temps nécessaire à l’exécution du script, évitez les opérations qui compilent des ressources depuis le code source. Par exemple, précompilez les ressources et stockez-les dans un compte de stockage de blobs Azure dans le même centre de données que HDInsight.

### <a name="ensure-that-the-cluster-customization-script-is-idempotent"></a><a name="bPS3"></a>S’assurer que le script de personnalisation du cluster est idempotent

Les scripts doivent être idempotents. Si le script est exécuté plusieurs fois, il doit retourner le cluster au même état à chaque fois.

Par exemple, un script qui modifie les fichiers de configuration ne doit pas ajouter d’entrées en double s’il est exécuté plusieurs fois.

### <a name="ensure-high-availability-of-the-cluster-architecture"></a><a name="bPS5"></a>Garantir la haute disponibilité de l’architecture du cluster

Les clusters HDInsight basés sur Linux proposent deux nœuds principaux actifs au sein du cluster, et les actions de script sont exécutées sur les deux nœuds. Si les composants que vous installez n’attendent qu’un seul nœud principal, n’installez pas les composants sur les deux nœuds principaux.

> [!IMPORTANT]  
> Les services fournis dans le cadre de HDInsight sont conçus pour basculer entre les deux nœuds principaux si nécessaire. Cette fonctionnalité n’est pas étendue aux composants personnalisés installés à l’aide d’actions de script. Si vous avez besoin que les composants personnalisés soient très disponibles, vous devez implémenter votre propre mécanisme de basculement.

### <a name="configure-the-custom-components-to-use-azure-blob-storage"></a><a name="bPS6"></a>Configurer les composants personnalisés pour utiliser le stockage d’objets blob Azure

Les composants que vous installez sur le cluster peuvent être configurés par défaut pour utiliser le stockage HDFS (Apache Hadoop Distributed File System). HDInsight utilise Stockage Azure ou Data Lake Storage comme stockage par défaut. Chacun d’eux fournit un système de fichiers compatible HDFS qui rend persistantes les données même en cas de suppression du cluster. Vous devrez peut-être configurer les composants que vous installez pour utiliser WASB ou ADL au lieu de HDFS.

Pour la plupart des opérations, il est inutile de spécifier le système de fichiers. Par exemple, ce qui suit copie le fichier hadoop-common.jar du système de fichiers local vers le stockage du cluster :

```bash
hdfs dfs -put /usr/hdp/current/hadoop-client/hadoop-common.jar /example/jars/
```

Dans cet exemple, la commande `hdfs` utilise en toute transparence l’emplacement de stockage de cluster par défaut. Pour certaines opérations, vous devrez peut-être spécifier l’URI. Par exemple, `adl:///example/jars` pour Azure Data Lake Storage Gen1, `abfs:///example/jars` pour Data Lake Storage Gen2 ou `wasb:///example/jars` pour Stockage Azure.

### <a name="write-information-to-stdout-and-stderr"></a><a name="bPS7"></a>Écrire des informations sur STDOUT et STDERR

HDInsight journalise la sortie de script qui est écrite dans STDOUT et STDERR. Vous pouvez afficher ces informations à l’aide de l’interface utilisateur web d’Ambari.

> [!NOTE]  
> Apache Ambari n’est disponible que si le cluster a été créé avec succès. Si vous utilisez une action de script lors de la création du cluster et que la création échoue, consultez la section de [Résoudre les problèmes liés aux actions de script](./troubleshoot-script-action.md) pour découvrir d’autres façons d’accéder aux informations de journalisation.

La plupart des utilitaires et des packages d’installation ont déjà écrit des informations dans STDOUT et STDERR. Toutefois, vous pouvez ajouter un enregistrement supplémentaire. Pour envoyer du texte à STDOUT, utilisez `echo`. Par exemple :

```bash
echo "Getting ready to install Foo"
```

Par défaut, `echo` envoie la chaîne vers STDOUT. Pour la diriger vers STDERR, ajoutez `>&2` avant `echo`. Par exemple :

```bash
>&2 echo "An error occurred installing Foo"
```

Cela redirige les informations écrites dans STDOUT vers STDERR (2) à la place. Pour plus d’informations sur la redirection des E/S, voir [https://www.tldp.org/LDP/abs/html/io-redirection.html](https://www.tldp.org/LDP/abs/html/io-redirection.html).

Pour plus d’informations sur l’affichage des informations consignées par les actions de script, consultez [Résoudre les problèmes liés aux actions de script](./troubleshoot-script-action.md).

### <a name="save-files-as-ascii-with-lf-line-endings"></a><a name="bps8"></a> Enregistrer des fichiers au format ASCII avec les fins de ligne LF

Les scripts d’interpréteur de commandes doivent être stockés au format ASCII, avec des lignes terminées se terminant par LF. Les fichiers qui sont stockés au format UTF-8 ou utilisent CRLF comme fin de ligne peuvent échouer avec l’erreur suivante :

```
$'\r': command not found
line 1: #!/usr/bin/env: No such file or directory
```

### <a name="use-retry-logic-to-recover-from-transient-errors"></a><a name="bps9"></a> Utilisation de la logique de nouvelle tentative pour récupérer après une erreur temporaire

Lors du téléchargement de fichiers, de l’installation de packages à l’aide d’apt-get ou de l’exécution d’autres actions qui transmettent des données sur Internet, l’action peut échouer à cause d’erreurs réseau temporaires. Par exemple, la ressource distante avec laquelle vous communiquez peut être en cours de basculement vers un nœud de sauvegarde.

Pour rendre votre script résistant aux erreurs temporaires, vous pouvez implémenter la logique de nouvelle tentative. La fonction suivante montre comment implémenter la logique de nouvelle tentative. Elle réessaie l’opération trois fois avant d’échouer.

```bash
#retry
MAXATTEMPTS=3

retry() {
    local -r CMD="$@"
    local -i ATTMEPTNUM=1
    local -i RETRYINTERVAL=2

    until $CMD
    do
        if (( ATTMEPTNUM == MAXATTEMPTS ))
        then
                echo "Attempt $ATTMEPTNUM failed. no more attempts left."
                return 1
        else
                echo "Attempt $ATTMEPTNUM failed! Retrying in $RETRYINTERVAL seconds..."
                sleep $(( RETRYINTERVAL ))
                ATTMEPTNUM=$ATTMEPTNUM+1
        fi
    done
}
```

Les exemples suivants montrent comment utiliser cette fonction.

```bash
retry ls -ltr foo

retry wget -O ./tmpfile.sh https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh
```

## <a name="helper-methods-for-custom-scripts"></a><a name="helpermethods"></a>Méthodes d'assistance pour les scripts personnalisés

Les méthodes d’assistance aux actions de script sont des utilitaires que vous pouvez utiliser lors de l’écriture de scripts personnalisés. Ces méthodes sont contenues dans le script [https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh). Pour les télécharger et les utiliser dans le cadre de votre script, utilisez les éléments suivants :

```bash
# Import the helper method module.
wget -O /tmp/HDInsightUtilities-v01.sh -q https://hdiconfigactions.blob.core.windows.net/linuxconfigactionmodulev01/HDInsightUtilities-v01.sh && source /tmp/HDInsightUtilities-v01.sh && rm -f /tmp/HDInsightUtilities-v01.sh
```

Les programmes d’assistance suivants disponibles pour une utilisation dans votre script :

| Utilisation de l’aide | Description |
| --- | --- |
| `download_file SOURCEURL DESTFILEPATH [OVERWRITE]` |Télécharge un fichier de l’URI source vers le chemin d’accès de fichier spécifié. Par défaut, il ne remplace pas un fichier existant. |
| `untar_file TARFILE DESTDIR` |Extrait un fichier tar (à l’aide de `-xf`,) dans le répertoire de destination. |
| `test_is_headnode` |Lorsque le script est exécuté sur un nœud principal de cluster, la valeur 1 est renvoyée ; dans le cas contraire, c’est la valeur 0. |
| `test_is_datanode` |Si le nœud actuel est un nœud de données (worker), la valeur 1 est renvoyée ; dans le cas contraire, c’est la valeur 0. |
| `test_is_first_datanode` |Si le nœud actuel est le premier nœud de données (worker) (nommé workernode0,) la valeur 1 est renvoyée ; dans le cas contraire, c’est la valeur 0. |
| `get_headnodes` |Renvoie le nom de domaine complet des nœuds principaux dans le cluster. Les noms sont séparés par des virgules. Une chaîne vide est renvoyée en cas d’erreur. |
| `get_primary_headnode` |Obtient le nom de domaine complet du nœud principal primaire. Une chaîne vide est renvoyée en cas d’erreur. |
| `get_secondary_headnode` |Obtient le nom de domaine complet du nœud principal secondaire. Une chaîne vide est renvoyée en cas d’erreur. |
| `get_primary_headnode_number` |Obtient le suffixe numérique du nœud principal primaire. Une chaîne vide est renvoyée en cas d’erreur. |
| `get_secondary_headnode_number` |Obtient le suffixe numérique du nœud principal secondaire. Une chaîne vide est renvoyée en cas d’erreur. |

## <a name="common-usage-patterns"></a><a name="commonusage"></a>Modes d'utilisation courants

Cette section fournit des conseils sur l'implémentation de certains des modèles d'utilisation courants que vous pouvez rencontrer lors de l'écriture de votre propre script personnalisé.

### <a name="passing-parameters-to-a-script"></a>Transmission de paramètres vers un script

Dans certains cas, votre script peut nécessiter des paramètres. Par exemple, vous aurez peut-être besoin du mot de passe d’administrateur pour le cluster lors de l’utilisation de l’API REST Ambari.

Les paramètres transmis au script sont appelés *paramètres positionnels* et sont affectés à `$1` pour ce qui concerne le premier paramètre, `$2` pour le deuxième et ainsi de suite. `$0` contient le nom du script lui-même.

Les valeurs transmises au script comme paramètres doivent être entourées de guillemets simples ('). Cela permet de s’assurer que la valeur transmise est traitée comme un littéral.

### <a name="setting-environment-variables"></a>Définition des variables d'environnement

La définition d’une variable d’environnement est effectuée de la façon suivante :

```bash
VARIABLENAME=value
```

Dans l’exemple précédent, `VARIABLENAME` est le nom de la variable. Pour accéder à la variable, utilisez `$VARIABLENAME`. Par exemple, pour affecter une valeur fournie par un paramètre de positionnement tel qu’une variable d’environnement nommée PASSWORD, utilisez l’instruction suivante :

```bash
PASSWORD=$1
```

Lors des accès ultérieurs aux informations, il est possible d’utiliser `$PASSWORD`.

Les variables d’environnement définies dans le script existent uniquement dans le cadre du script. Dans certains cas, vous devrez peut-être ajouter des variables d’environnement de niveau système qui persisteront une fois le script terminé. Pour ajouter des variables d’environnement de niveau système, ajoutez la variable à `/etc/environment`. Par exemple, l’instruction suivante ajoute `HADOOP_CONF_DIR` :

```bash
echo "HADOOP_CONF_DIR=/etc/hadoop/conf" | sudo tee -a /etc/environment
```

### <a name="access-to-locations-where-the-custom-scripts-are-stored"></a>Accès aux emplacements où sont stockés les scripts personnalisés

Les scripts utilisés pour personnaliser un cluster doivent être stockés dans un des emplacements suivants :

* Un __compte de stockage Azure__ qui est associé au cluster.

* Un __compte de stockage supplémentaire__ associé au cluster.

* Une __URI lisible publiquement__. Par exemple, une URL menant aux données stockées sur OneDrive, Dropbox ou un autre service d’hébergement de fichiers.

* Un compte __Azure Data Lake Storage__ associé à un groupe de ressources Azure. Pour plus d’informations sur l’utilisation d’Azure Data Lake Storage avec HDInsight, consultez [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

    > [!NOTE]  
    > Le principal de service utilisé par HDInsight pour accéder à Data Lake Storage doit avoir accès en lecture au script.

Les ressources utilisées par le script doivent également être disponibles publiquement.

Le stockage des fichiers dans un compte Stockage Azure ou Azure Data Lake Storage fournit un accès rapide, car tous deux se trouvent sur le réseau Azure.

> [!NOTE]  
> Le format d’URI utilisé pour référencer le script diffère selon le service utilisé. Pour les comptes de stockage associés au cluster HDInsight, utilisez `wasb://` ou `wasbs://`. Pour des URI lisibles publiquement, utilisez `http://` ou `https://`. Pour Data Lake Storage, utilisez `adl://`.

## <a name="checklist-for-deploying-a-script-action"></a><a name="deployScript"></a>Liste de vérification pour le déploiement d'une action de script

Voici les étapes à suivre avant de déployer un script :

* Placez les fichiers qui contiennent les scripts personnalisés dans un emplacement accessible aux nœuds du cluster lors du déploiement. Par exemple, l’emplacement de stockage par défaut du cluster. Les fichiers peuvent également être stockés dans les services d’hébergement lisibles publiquement.
* Vérifiez que le script est idempotent. Ainsi, le script peut être exécuté plusieurs fois sur le même nœud.
* Utilisez un dossier de fichiers temporaires /tmp pour conserver le fichier téléchargé utilisé par les scripts, puis nettoyez-les une fois que les scripts ont été exécutés.
* Si les paramètres au niveau du système d’exploitation ou les fichiers de configuration du service Hadoop sont modifiés, vous pouvez redémarrer les services HDInsight.

## <a name="how-to-run-a-script-action"></a><a name="runScriptAction"></a>Comment exécuter une action de script

Vous pouvez utiliser des actions de script pour personnaliser des clusters HDInsight à l’aide des méthodes suivantes :

* Portail Azure
* Azure PowerShell
* Modèles Microsoft Azure Resource Manager
* Le kit de développement logiciel (SDK) HDInsight .NET.

Pour plus d’informations sur l’utilisation de chaque méthode, consultez [Comment utiliser une action de script](hdinsight-hadoop-customize-cluster-linux.md).

## <a name="custom-script-samples"></a><a name="sampleScripts"></a>Exemples de scripts personnalisés

Microsoft fournit des exemples de scripts pour installer des composants sur un cluster HDInsight. Consultez [Installer et utiliser Hue sur les clusters HDInsight](hdinsight-hadoop-hue-linux.md) pour obtenir un exemple d’action de script.

## <a name="troubleshooting"></a>Dépannage

Voici des erreurs susceptibles de se produire quand vous utilisez les scripts que vous avez développés :

**Erreur**  : `$'\r': command not found`. Parfois suivi par `syntax error: unexpected end of file`.

*Cause*  : Cette erreur se produit lorsque les lignes d’un script se terminent par CRLF. Les systèmes UNIX attendent seulement LF comme fin de ligne.

Ce problème se produit souvent lorsque le script est créé dans un environnement Windows, car CRLF est une fin de ligne commune à de nombreux éditeurs de texte sous Windows.

*Résolution*  : Si c’est une option dans votre éditeur de texte, sélectionnez le format Unix ou LF comme fin de ligne. Vous pouvez également utiliser les commandes suivantes sur un système Unix pour changer la séquence CRLF en LF :

> [!NOTE]  
> Les commandes suivantes sont à peu près équivalentes dans la mesure où elles doivent changer les fins de ligne CRLF en LF. Sélectionnez-en une basée sur les utilitaires disponibles sur votre système.

| Commande | Notes |
| --- | --- |
| `unix2dos -b INFILE` |Le fichier d’origine est sauvegardé avec une extension .BAK |
| `tr -d '\r' < INFILE > OUTFILE` |OUTFILE contient une version avec des terminaisons LF uniquement |
| `perl -pi -e 's/\r\n/\n/g' INFILE` | Modifie directement le fichier |
| ```sed 's/$'"/`echo \\\r`/" INFILE > OUTFILE``` |OUTFILE contient une version avec des terminaisons LF uniquement. |

**Erreur**  : `line 1: #!/usr/bin/env: No such file or directory`.

*Cause*  : Cette erreur se produit lorsque le script a été enregistré au format UTF-8 avec une marque d’ordre d’octet (BOM).

*Résolution*  : Enregistrez le fichier au format ASCII ou UTF-8 sans marque d’ordre d’octet. Vous pouvez également utiliser la commande suivante sur un système Linux ou Unix pour créer un fichier sans marque d’ordre d’octet :

```bash
awk 'NR==1{sub(/^\xef\xbb\xbf/,"")}{print}' INFILE > OUTFILE
```

Remplacez `INFILE` par le fichier contenant la marque d’ordre d’octet. `OUTFILE` doit être un nouveau nom de fichier contenant le script sans la marque d’ordre d’octet.

## <a name="next-steps"></a><a name="seeAlso"></a>Étapes suivantes

* Découvrez comment [Personnaliser des clusters HDInsight à l’aide d’une action de script](hdinsight-hadoop-customize-cluster-linux.md)
* Utilisez la [Référence du Kit de développement logiciel (SDK) .NET HDInsight](/dotnet/api/overview/azure/hdinsight) pour en savoir plus sur la création d’applications .NET qui gèrent HDInsight
* Utilisez l’ [API REST HDInsight](/rest/api/hdinsight/) pour savoir comment utiliser REST pour effectuer des actions de gestion sur des clusters HDInsight.