---
title: Présentation de ML Services sur Azure HDInsight
description: Découvrez comment utiliser ML Services sur HDInsight pour créer des applications pour l’analyse Big Data.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/20/2020
ms.openlocfilehash: b3a6fc2ff45d61d63118edc23f40f69f16db9131
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536145"
---
# <a name="what-is-ml-services-in-azure-hdinsight"></a>Qu’est-ce que ML Services dans Azure HDInsight

Microsoft Machine Learning Server vous est proposé comme option de déploiement au moment de créer des clusters HDInsight dans Azure. Le type de cluster qui fournit cette option est appelé **ML Services** . Cette fonctionnalité fournit un accès à la demande à des méthodes d’analytique adaptables et distribuées sur HDInsight.

ML Services sur HDInsight fournit les dernières fonctionnalités analytiques R pour des jeux de données de toute taille ou presque. Les jeux de données peuvent être chargés dans un stockage Data Lake ou Blob Azure. Vos applications R peuvent utiliser les packages R open source (plus de 8 000 disponibles). Les routines de ScaleR, package analytique Big Data de Microsoft, sont également disponibles.

Le nœud de périphérie offre un emplacement pratique pour la connexion au cluster et l’exécution de vos scripts R. Le nœud de périphérie permet d’exécuter des fonctions distribuées parallélisées de ScaleR sur les différents cœurs du serveur. Vous pouvez également les exécuter sur les différents nœuds du cluster en utilisant Hadoop Map Reduce avec ScaleR. Vous pouvez aussi utiliser des contextes de calcul Apache Spark.

Les modèles ou prévisions résultant des analyses peuvent être téléchargés pour une utilisation locale. Ils peuvent également être `operationalized` ailleurs dans Azure. notamment par le biais d’[Azure Machine Learning Studio (classique)](https://studio.azureml.net) et d’un [service web](../../machine-learning/classic/deploy-a-machine-learning-web-service.md).

## <a name="get-started-with-ml-services-on-hdinsight"></a>Démarrer avec ML Services sur HDInsight

Pour créer un cluster ML Services dans HDInsight, sélectionnez le type de cluster **ML Services** . Le type de cluster ML Services inclut ML Server sur les nœuds de données et le nœud de périphérie. Le nœud de périphérie sert de zone d’atterrissage pour l’analytique basée sur ML Services. Consultez [Créer des clusters Apache Hadoop à l’aide du portail Azure](../hdinsight-hadoop-create-linux-clusters-portal.md) pour une procédure pas à pas permettant de créer le cluster.

## <a name="why-choose-ml-services-in-hdinsight"></a>Pourquoi choisir ML Services dans HDInsight ?

ML Services dans HDInsight offre les avantages suivants :

### <a name="ai-innovation-from-microsoft-and-open-source"></a>Innovation Microsoft et open source en matière d’IA

  ML Services comprend un ensemble distribué et très adaptable d’algorithmes tels que [RevoscaleR](/machine-learning-server/r-reference/revoscaler/revoscaler), [revoscalepy](/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) et [microsoftML](/machine-learning-server/python-reference/microsoftml/microsoftml-package). Ces algorithmes peuvent fonctionner avec des tailles de données supérieures à la taille de la mémoire physique. Ils s’exécutent également sur une large gamme de plateformes de façon distribuée. En savoir plus sur la collection Microsoft de [packages R](/machine-learning-server/r-reference/introducing-r-server-r-package-reference) et de [packages Python](/machine-learning-server/python-reference/introducing-python-package-reference) inclus avec le produit.
  
  ML Services fait le lien entre ces innovations Microsoft et les contributions de la communauté open source (kits d’outils d’IA, R, Python), le tout sur une plateforme unique de niveau entreprise. Quel que soit le package R ou Python de Machine Learning open source, il peut fonctionner aux côtés de n’importe quelle innovation propriétaire de Microsoft.

### <a name="simple-secure-and-high-scale-operationalization-and-administration"></a>Opérationnalisation et administration simples, sécurisées et à grande échelle

  Les entreprises qui reposent sur des environnements et des modèles classiques consentent trop de temps et d’efforts à l’opérationnalisation. Cela se traduit par une augmentation des coûts et un allongement des délais, ce qui inclut le temps nécessaire à la traduction des modèles, les itérations pour maintenir leur validité et les tenir à jour, les approbations réglementaires et la gestion des autorisations.

  ML Services offre une [opérationnalisation](/machine-learning-server/what-is-operationalization) de niveau entreprise. Quand un modèle Machine Learning est terminé, il suffit de quelques clics pour générer des API de services web. Ces [services web](/machine-learning-server/operationalize/concept-what-are-web-services) sont hébergés sur une grille de serveurs dans le cloud et peuvent être intégrés à des applications métier. La possibilité d’effectuer le déploiement sur une grille élastique vous permet d’évoluer en toute transparence en fonction des besoins de votre entreprise, pour la notation par lot et en temps réel. Pour obtenir des instructions, consultez [Rendre opérationnel ML Services sur HDInsight](r-server-operationalize.md).

<!---
* **Deep ecosystem engagements to deliver customer success with optimal total cost of ownership**

  Individuals embarking on the journey of making their applications intelligent or simply wanting to learn the new world of AI and machine learning, need the right resources to help them get started. In addition to this documentation, Microsoft provides several learning resources and has engaged several training partners to help you ramp up and become productive quickly.
--->

> [!NOTE]  
> Le type de cluster ML Services sur HDInsight est pris en charge uniquement sur HDInsight 3.6. La mise hors service de HDInsight 3.6 est prévue pour le 31 décembre 2020.

## <a name="key-features-of-ml-services-on-hdinsight"></a>Fonctionnalités clés de ML Services sur HDInsight

ML Services sur HDInsight inclut les fonctionnalités suivantes.

| Catégorie de fonctionnalité | Description |
|------------------|-------------|
| Pour R | [Packages R](/machine-learning-server/r-reference/introducing-r-server-r-package-reference) pour les solutions écrites en R, avec une distribution open source de R et une infrastructure d’exécution pour l’exécution de script. |
| Pour Python | [Modules Python](/machine-learning-server/python-reference/introducing-python-package-reference) pour les solutions écrites en Python, avec une distribution open source de Python et une infrastructure d’exécution pour l’exécution de script.
| [Modèles préentraînés](/machine-learning-server/install/microsoftml-install-pretrained-models) | Pour une analyse visuelle et une analyse des sentiments sur le texte, prêts pour noter les données que vous fournissez. |
| [Déploiement et consommation](r-server-operationalize.md) | `Operationalize` votre serveur et déployez des solutions en tant que service web. |
| [Exécution à distance](r-server-hdinsight-manage.md#connect-remotely-to-microsoft-ml-services) | Démarrez des sessions à distance sur un cluster ML Services de votre réseau à partir de votre station de travail cliente. |

## <a name="data-storage-options-for-ml-services-on-hdinsight"></a>Options de stockage de données pour ML Services sur HDInsight

Le stockage par défaut du système de fichiers HDFS peut être un compte de stockage Azure ou un stockage Azure Data Lake Storage. Les données chargées dans le stockage en cluster durant l’analyse deviennent persistantes. Les données restent disponibles même après la suppression du cluster. Différents outils permettent de gérer le transfert de données vers le stockage, notamment la fonction de chargement basée sur le portail du compte de stockage et l’utilitaire AzCopy.

Vous pouvez autoriser l’accès à des magasins Blob et Data Lake supplémentaires au moment de la création du cluster. Vous n’êtes pas limité par l’option de stockage principal utilisée.  Consultez l’article [Options de stockage Azure pour ML Services sur HDInsight](./r-server-storage.md) pour en savoir plus sur l’utilisation de plusieurs comptes de stockage.

Vous pouvez également utiliser Azure Files comme option de stockage pour une utilisation sur le nœud de périphérie. Le service Azure Files vous permet d’utiliser des partages de fichiers créés dans un stockage Azure avec le système de fichiers Linux. Pour plus d’informations, consultez [Options de stockage Azure pour ML Services sur HDInsight](r-server-storage.md).

## <a name="access-ml-services-edge-node"></a>Accéder à un nœud de périphérie ML Services

Vous pouvez vous connecter à Microsoft ML Server sur le nœud de périphérie à l’aide d’un navigateur ou de SSH/PuTTY. La console R est installée par défaut au moment de la création du cluster.  

## <a name="develop-and-run-r-scripts"></a>Développer et exécuter des scripts R

Vos scripts R peuvent utiliser n’importe quel package R open source parmi les plus de 8 000 disponibles. Vous pouvez également utiliser les routines parallélisées et distribuées de la bibliothèque ScaleR. Les scripts exécutés sur le nœud de périphérie s’exécutent dans l’interpréteur R sur ce nœud, sauf dans le cadre des étapes qui appellent les fonctions ScaleR avec un contexte de calcul Map Reduce (RxHadoopMR) ou Spark (RxSpark). Les fonctions s’exécutent de manière distribuée sur les nœuds de données associés aux données. Pour plus d’informations sur les options de contexte, consultez [Options de contexte de calcul pour ML Services sur HDInsight](r-server-compute-contexts.md).

## <a name="operationalize-a-model"></a>`Operationalize` un modèle

Une fois la modélisation de vos données terminée, `operationalize` le modèle de façon à effectuer des prédictions pour de nouvelles données, que ce soit à partir d’Azure ou localement. Ce processus est appelé notation. La notation est possible dans HDInsight, dans Azure Machine Learning ou en local.

### <a name="score-in-hdinsight"></a>Noter dans HDInsight

Pour noter dans HDInsight, écrivez une fonction R. La fonction appelle votre modèle pour effectuer des prévisions pour un nouveau fichier de données que vous avez chargé sur votre compte de stockage. Ensuite, enregistrez les prévisions dans le compte de stockage. Vous pouvez exécuter cette routine à la demande sur le nœud de périphérie de votre cluster ou à l’aide d’une tâche planifiée.

### <a name="score-in-azure-machine-learning-aml"></a>Noter dans Azure Machine Learning (AML)

Pour noter à l’aide d’Azure Machine Learning, utilisez le package R Azure Machine Learning open source nommé [AzureML](https://cran.r-project.org/src/contrib/Archive/AzureML/) pour publier votre modèle en tant que service web Azure. Pour plus de commodité, ce package est déjà installé sur le nœud de périphérie. Ensuite, utilisez les fonctionnalités d’Azure Machine Learning pour créer une interface utilisateur pour le service web, puis appelez le service web en fonction des besoins de notation. Convertissez ensuite les objets de modèle ScaleR en objets de modèle open source équivalents pour les utiliser avec le service web. Utilisez les fonctions de forçage de ScaleR, notamment `as.randomForest()` dans le cas des modèles basés sur un ensemble, pour cette conversion.

### <a name="score-on-premises"></a>Noter localement

Pour noter localement après la création de votre modèle, sérialisez le modèle dans R, téléchargez-le, désérialisez-le, puis utilisez-le pour noter de nouvelles données. Vous pouvez noter les nouvelles données en adoptant l’approche décrite plus haut dans Noter dans HDInsight ou en utilisant des [services web](/machine-learning-server/operationalize/concept-what-are-web-services).

## <a name="maintain-the-cluster"></a>Maintenance du cluster

### <a name="install-and-maintain-r-packages"></a>Installation et maintenance des packages R

La plupart des packages R utilisés seront requis sur le nœud périphérique, car la plupart des étapes des scripts R y sont exécutées. Pour installer des packages R supplémentaires sur le nœud de périphérie, vous pouvez utiliser la méthode `install.packages()` de R.

Si vous utilisez simplement des routines de bibliothèque ScaleR, vous n’avez généralement pas besoin de packages R supplémentaires. Vous aurez peut-être besoin de packages supplémentaires pour l’exécution de **rxExec** ou **RxDataStep** sur les nœuds de données.

Les packages supplémentaires peuvent être installés à l’aide d’une action de script après la création du cluster. Pour plus d’informations, consultez [Gérer un cluster ML Services dans HDInsight](r-server-hdinsight-manage.md).

### <a name="change-apache-hadoop-mapreduce-memory-settings"></a>Modifier les paramètres de mémoire d’Apache Hadoop MapReduce

La mémoire disponible pour ML Services peut être modifiée quand un travail MapReduce est exécuté. Pour modifier un cluster, utilisez l’interface utilisateur d’Apache Ambari pour votre cluster. Pour obtenir des instructions sur l’interface utilisateur d’Ambari, consultez [Gérer des clusters HDInsight à l’aide de l’interface utilisateur web d’Apache Ambari](../hdinsight-hadoop-manage-ambari.md).

La mémoire disponible pour ML Services peut être modifiée à l’aide de commutateurs Hadoop dans l’appel à **RxHadoopMR**  :

```r
hadoopSwitches = "-libjars /etc/hadoop/conf -Dmapred.job.map.memory.mb=6656"
```

### <a name="scale-your-cluster"></a>Mettre à l’échelle votre cluster

Vous pouvez faire monter ou descendre en puissance un cluster ML Services sur HDInsight existant via le portail. Quand vous effectuez un scale-up, vous bénéficiez d’une capacité supplémentaire pour les tâches de traitement plus importantes. Vous pouvez rétablir l’échelle d’origine d’un cluster quand il est inactif. Pour obtenir des instructions concernant la mise à l’échelle d’un cluster, consultez la section [Gérer des clusters HDInsight](../hdinsight-administer-use-portal-linux.md).

### <a name="maintain-the-system"></a>Maintenance du système

La maintenance du système d’exploitation est effectuée sur les machines virtuelles Linux sous-jacentes dans un cluster HDInsight pendant les heures creuses. En règle générale, la maintenance s’effectue à 3 h 30 (heure locale de la machine virtuelle) tous les lundis et jeudis. Les mises à jour n’impactent pas plus d’un quart du cluster à la fois.

Les travaux en cours d’exécution peuvent être ralentis durant la maintenance. Elles doivent cependant être exécutées jusqu’à la fin. L’ensemble des logiciels personnalisés ou des données locales que vous détenez sont conservés durant ces événements de maintenance, sauf si une défaillance irrémédiable se produit, nécessitant une reconstruction du cluster.

## <a name="ide-options-for-ml-services-on-hdinsight"></a>Options IDE pour ML Services sur HDInsight

Le nœud de périmètre Linux d’un cluster HDInsight est la zone d’accueil pour l’analyse basée sur R. Les versions récentes de HDInsight offrent l’IDE basé sur navigateur de RStudio Server sur le nœud de périphérie. RStudio Server est plus productif que la console R pour le développement et l’exécution.

Un IDE de bureau peut accéder au cluster par le biais d’un contexte de calcul Spark ou MapReduce distant. Options disponibles : [Outils R pour Visual Studio](https://marketplace.visualstudio.com/items?itemName=MikhailArkhipov007.RTVS2019) (RTVS) de Microsoft, RStudio et StatET sur Eclipse de Walware.

Accédez à la console R sur le nœud de périphérie en tapant **R** à l’invite de commandes. Quand vous utilisez l’interface de la console, il est pratique de développer un script R dans un éditeur de texte. Ensuite, coupez des sections de votre script et collez-les dans la console R en fonction de vos besoins.

## <a name="pricing"></a>Tarifs

Les prix associés à un cluster ML Services HDInsight sont structurés de la même manière que ceux d’autres types de cluster HDInsight. Ils sont basés sur le dimensionnement des machines virtuelles sous-jacentes pour le nom, les données et les nœuds de périphérie. Les ajouts par cœur-heure sont également pris en compte. Pour plus d’informations, voir [Tarification de HDInsight](https://azure.microsoft.com/pricing/details/hdinsight/).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation des clusters ML Services sur HDInsight, consultez les articles suivants :

* [Exécuter un script R sur un cluster ML Services dans HDInsight Azure à l’aide de RStudio Server](machine-learning-services-quickstart-job-rstudio.md)
* [Options de contexte de calcul pour un cluster ML Services sur HDInsight](r-server-compute-contexts.md)
* [Options de stockage pour un cluster ML Services sur HDInsight](r-server-storage.md)