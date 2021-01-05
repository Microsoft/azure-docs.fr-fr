---
title: Exécuter des charges de travail Azure Machine Learning sur Apache Spark dans HDInsight
description: Découvrez comment exécuter des charges de travail Azure Machine Learning avec Machine learning automatisé (AutoML) sur Apache Spark dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/13/2019
ms.openlocfilehash: 6e472c65897fa57cdb1e0b09d94c62913e268040
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86087467"
---
# <a name="run-azure-machine-learning-workloads-with-automated-machine-learning-on-apache-spark-in-hdinsight"></a>Exécuter des charges de travail Azure Machine Learning avec Machine learning automatisé sur Apache Spark dans HDInsight

Azure Machine Learning simplifie et accélère la création, la formation et le déploiement de modèles Machine Learning. Dans Machine Learning automatisé (AutoML), vous commencez avec des données d’apprentissage comportant une fonctionnalité cible définie, puis vous itérez à travers des combinaisons d’algorithmes et de sélections de fonctionnalités pour choisir automatiquement le meilleur modèle pour vos données en fonction des scores d’apprentissage. HDInsight permet aux clients d’approvisionner des clusters avec des centaines de nœuds. AutoML exécuté sur Spark dans un cluster HDInsight permet aux utilisateurs d’utiliser la capacité de calcul sur ces nœuds afin d’exécuter des travaux d’apprentissage de manière montée en puissance et d’exécuter plusieurs travaux d’apprentissage en parallèle. Cela permet aux utilisateurs d’exécuter des expériences AutoML tout en partageant le calcul avec leurs autres charges de travail Big Data.

## <a name="install-azure-machine-learning-on-an-hdinsight-cluster"></a>Installer Azure Machine Learning sur un cluster HDInsight

Pour obtenir des didacticiels généraux sur Machine Learning automatisé, consultez le [Didacticiel : Utiliser le Machine Learning automatisé pour générer votre modèle de régression](../../machine-learning/tutorial-auto-train-models.md).
Tous les nouveaux clusters HDInsight-Spark sont préinstallés avec le Kit de développement logiciel (SDK) AzureML-AutoML.

> [!Note]
> Les packages Azure Machine Learning sont installés dans l’environnement conda Python3. Le bloc-notes Jupyter installé doit être exécuté à l’aide du noyau PySpark3.

Vous pouvez utiliser des notebooks Zeppelin pour utiliser AutoML.

> [!Note]
> Zeppelin comporte un [problème connu](https://community.hortonworks.com/content/supportkb/207822/the-livypyspark3-interpreter-uses-python-2-instead.html) tel que PySpark3 ne choisit la bonne version de Python. Utilisez la solution de contournement documentée.

## <a name="authentication-for-workspace"></a>Authentification pour l’espace de travail

La création d’espace de travail et la soumission d’expérience nécessitent un jeton d’authentification. Ce jeton peut être généré à l’aide d’une [application Azure AD](../../active-directory/develop/app-objects-and-service-principals.md). Un [utilisateur Azure AD](/azure/python/python-sdk-azure-authenticate) peut également générer le jeton d’authentification requis, si l’authentification multifacteur n’est pas activée sur le compte.  

L’extrait de code suivant crée un jeton d’authentification à l’aide d’une **application Azure AD**.

```python
from azureml.core.authentication import ServicePrincipalAuthentication
auth_sp = ServicePrincipalAuthentication(
    tenant_id='<Azure Tenant ID>',
    service_principal_id='<Azure AD Application ID>',
    service_principal_password='<Azure AD Application Key>'
)
```

L’extrait de code suivant crée un jeton d’authentification à l’aide d’un **utilisateur Azure AD**.

```python
from azure.common.credentials import UserPassCredentials
credentials = UserPassCredentials('user@domain.com', 'my_smart_password')
```

## <a name="loading-dataset"></a>Chargement de jeu de données

Machine learning automatisé sur Spark utilise des **dataflows**, qui sont des opérations immuables et évaluées de manière différée sur des données.  Un dataflow peut charger un jeu de données à partir d’un objet blob avec accès en lecture public, ou à partir d’une URL d’objet blob avec un jeton SAP.

```python
import azureml.dataprep as dprep

dataflow_public = dprep.read_csv(
    path='https://commonartifacts.blob.core.windows.net/automl/UCI_Adult_train.csv')

dataflow_with_token = dprep.read_csv(
    path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv?st=2018-06-15T23%3A01%3A42Z&se=2019-06-16T23%3A01%3A00Z&sp=r&sv=2017-04-17&sr=b&sig=ugQQCmeC2eBamm6ynM7wnI%2BI3TTDTM6z9RPKj4a%2FU6g%3D')
```

Vous pouvez également inscrire le magasin de données auprès de l’espace de travail à l’aide d’une inscription unique.

## <a name="experiment-submission"></a>Soumission d’expérimentations

Dans la [configuration de Machine Learning automatisé](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig), la propriété `spark_context` doit être définie pour que le package s’exécute en mode distribué. La propriété `concurrent_iterations`, qui est le nombre maximal d’itérations exécutées en parallèle, doit avoir pour valeur un nombre inférieur aux cœurs d’exécuteur de l’application Spark.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la motivation pour le Machine Learning automatisé, consultez [Release models at pace using Microsoft’s automated machine learning!](https://azure.microsoft.com/blog/release-models-at-pace-using-microsoft-s-automl/) (Mettre régulièrement en production des modèles à l’aide du Machine Learning automatisé de Microsoft)
* Pour plus d’informations sur les fonctionnalités de ML automatisé d’Azure ML, voir [Nouvelles fonctionnalités de Machine Learning automatisé dans Azure Machine Learning](https://azure.microsoft.com/blog/new-automated-machine-learning-capabilities-in-azure-machine-learning-service/)
* [Projet AutoML de Microsoft Research](https://www.microsoft.com/research/project/automl/)
