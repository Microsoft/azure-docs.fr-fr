---
title: Publication d’applications Azure HDInsight
description: Découvrez comment créer une application HDInsight, puis la publier sur la Place de marché Microsoft Azure.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 05/14/2018
ms.author: hrasheed
ms.openlocfilehash: ca84cb6cdd6b47976eadbc5298701a46fe677426
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96007135"
---
# <a name="publish-an-hdinsight-application-in-the-azure-marketplace"></a>Publier une application HDInsight sur la Place de marché Microsoft Azure
Vous pouvez installer une application Azure HDInsight sur un cluster HDInsight basé sur Linux. Cet article explique comment publier une application HDInsight sur la Place de marché Microsoft Azure. Pour obtenir des informations générales sur la publication sur la Place de marché Microsoft Azure, consultez [Publier une offre sur la Place de marché Microsoft Azure](../marketplace/overview.md).

Les applications HDInsight utilisent le modèle *BYOL (apportez votre propre licence)* . Dans un scénario BYOL, un fournisseur d’applications est responsable de concéder la licence des applications aux utilisateurs. Les utilisateurs paient uniquement pour les ressources Azure qu’ils créent, telles que le cluster HDInsight, ainsi que les nœuds et machines virtuelles du cluster. À l’heure actuelle, la facturation de l’application n’est pas effectuée dans Azure.

Pour plus d’informations, consultez les articles suivants relatifs à l’application HDInsight :

* [Installer des applications HDInsight](hdinsight-apps-install-applications.md) : découvrez comment installer une application HDInsight sur vos clusters.
* [Installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md) : découvrez comment installer et tester des applications HDInsight personnalisées.

## <a name="prerequisites"></a>Conditions préalables requises
Pour envoyer votre application personnalisée à la Place de marché, vous devez d’abord [créer et tester votre application personnalisée](hdinsight-apps-install-custom-applications.md).

Vous devez également enregistrer votre compte de développeur. Pour plus d’informations, consultez [Publier une offre sur la Place de marché Microsoft Azure](../marketplace/overview.md) et [Créer un compte de développeur Microsoft](../marketplace/overview.md).

## <a name="define-the-application"></a>Définition de l’application
La publication d’applications sur la Place de marché comprend deux étapes. Définissez d’abord un fichier *createUiDef.json*. Le fichier createUiDef.json indique les clusters avec lesquels votre application est compatible. Publiez ensuite le modèle depuis le portail Azure. Voici un exemple de fichier createUiDef.json :

```json
{
    "handler": "Microsoft.HDInsight",
    "version": "0.0.1-preview",
    "clusterFilters": {
        "types": ["Hadoop", "HBase", "Storm", "Spark"],
        "versions": ["3.6"]
    }
}
```

| Champ | Description | Valeurs possibles |
| --- | --- | --- |
| types |Les types de cluster compatibles avec l’application. |Hadoop, HBase, Storm, Spark (ou toute combinaison de ceux-ci) |
| versions |Les types de cluster HDInsight compatibles avec l’application. |3.4 |

## <a name="application-installation-script"></a>Script d’installation de l’application
Lorsqu’une application est installée sur un cluster (existant ou nouveau), un nœud de périphérie est créé. Le script d’installation de l’application s’exécute sur le nœud de périphérie.

  > [!IMPORTANT]  
  > Le nom du script d’installation de l’application sur un cluster spécifique doit être unique et avoir le format suivant :
  > 
  > "name": "[concat('hue-install-v0','-' ,uniquestring(‘applicationName’)]"
  > 
  > Le nom du script est composé de trois parties :
  > 
  > * Un préfixe, qui doit inclure soit le nom de l’application, soit un nom en lien avec cette dernière.
  > * Un trait d’union, pour une meilleure lisibilité.
  > * Une fonction de chaîne unique utilisant le nom de l’application comme paramètre.
  > 
  > Dans la liste d’actions de script persistantes, l’exemple précédent apparaît sous le nom : **hue-install-v0-4wkahss55hlas**. Consultez cet [exemple de charge utile JSON](https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/Hue/azuredeploy.json).
  > 

Le script d’installation doit avoir les caractéristiques suivantes :
* Le script est idempotent. Plusieurs appels au script produisent le même résultat.
* La version du script est correctement gérée. Utilisez un autre emplacement pour le script lorsque vous effectuez une mise à niveau ou testez des modifications. Cela permet d’éviter aux clients qui installent l’application d’être affectés par les mises à jour ou les tests. 
* Le script a une journalisation adéquate à chaque étape. Les journaux d’activité de script sont généralement la seule façon de déboguer les problèmes d’installation des applications.
* Les appels passés à des services ou ressources externes font l’objet de nouvelles tentatives afin que l’installation ne soit pas affectée par des problèmes réseau temporaires.
* Si votre script démarre des services sur les nœuds, les services sont analysés et configurés pour démarrer automatiquement en cas de redémarrage d’un nœud.

## <a name="package-the-application"></a>Empaquetage de l’application
Créez un fichier zip contenant tous les fichiers requis pour l’installation de votre application HDInsight. Vous utilisez le fichier .zip pour publier l’application. Ce fichier .zip contient les fichiers suivants :

* createUiDefinition.json
* mainTemplate.json (voir l’exemple de la page [Installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md))
* Tous les scripts nécessaires

> [!NOTE]  
> Vous pouvez héberger les fichiers d’application (y compris des fichiers d’application web) sur n’importe quel point de terminaison accessible au public.

## <a name="publish-the-application"></a>Publication de l'application
Pour publier une application HDInsight :

1. Connectez-vous à la [Publication Azure](https://publish.windowsazure.com/).
2. Dans le menu de gauche, sélectionnez **Modèles de solution**.
3. Entrez un titre, puis sélectionnez **Créer un modèle de solution**.
4. Si vous n’avez pas encore enregistré votre organisation, sélectionnez **Create Dev Center account and join the Azure program** (Créer un compte du Centre de développement et participer au programme Azure).  Pour plus d’informations, consultez [Créer un compte de développeur Microsoft](../marketplace/overview.md).
5. Sélectionnez **Définir des Topologies pour démarrer**. Un modèle de solution est « parent » de toutes ses topologies. Vous pouvez définir plusieurs topologies dans une offre ou un modèle de solution. Quand une offre est envoyée dans l’environnement intermédiaire, toutes ses topologies l’accompagnent. 
6. Entrez un nom de topologie, puis sélectionnez **+** .
7. Entrez une nouvelle version, puis sélectionnez **+** .
8. Téléchargez le fichier .zip que vous avez créé lorsque vous avez empaqueté l’application.  
9. Sélectionnez **Request Certification**(Demander la certification). L’équipe de certification Microsoft examine les fichiers et certifie la topologie.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [installer des applications HDInsight](hdinsight-apps-install-applications.md) sur vos clusters.
* Découvrez comment [installer des applications HDInsight personnalisées](hdinsight-apps-install-custom-applications.md) et déployer une application HDInsight non publiée dans HDInsight.
* Découvrez comment [utiliser une action de script pour personnaliser des clusters HDInsight basés sur Linux](hdinsight-hadoop-customize-cluster-linux.md) et ajouter d’autres applications. 
* Découvrez comment [créer des clusters Apache Hadoop basés sur Linux dans HDInsight à l’aide de modèles Azure Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).
* Découvrez comment [utiliser un nœud de périphérie vide dans HDInsight](hdinsight-apps-use-edge-node.md) pour accéder aux clusters HDInsight, tester des applications HDInsight et héberger des applications HDInsight.