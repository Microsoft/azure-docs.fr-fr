---
title: Configuration d’environnements de science des données dans Azure - Processus TDSP
description: Configurer des environnements de science des données dans Azure à utiliser dans le processus TDSP (Team Data Science Process).
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 557550627fe2c39571a848723e5d716324fee240
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93321173"
---
# <a name="set-up-data-science-environments-for-use-in-the-team-data-science-process"></a>Configuration d’environnements de science des données à utiliser dans le processus TDSP (Team Data Science Process)
Le processus TDSP (Team Data Science Process) utilise différents environnements de science de données pour le stockage, le traitement et l’analyse de données. Il peut s’agir par exemple de Stockage Blob Azure, de plusieurs types de machines virtuelles Azure, de clusters HDInsight (Hadoop) et d’espaces de travail Azure Machine Learning. Le choix de l’environnement à utiliser dépend du type et de la quantité des données à modéliser, ainsi que de la destination cible de ces données dans le cloud. 

* Pour plus d’informations sur les questions à prendre en compte lors de cette décision, consultez la page [Planifier votre environnement de science des données Azure Machine Learning](plan-your-environment.md). 
* Pour accéder à un catalogue de scénarios que vous pouvez rencontrer lorsque vous effectuez des analyses avancées, consultez la page [Scénarios du processus TDSP (Team Data Science Process)](plan-sample-scenarios.md)

Les articles suivants expliquent comment configurer les différents environnements de science de données utilisés par le processus TDSP (Team Data Science Process).

* [Compte de stockage Azure](../../storage/common/storage-account-create.md)
* [Cluster HDInsight (Hadoop)](../../hdinsight/spark/apache-spark-jupyter-spark-sql.md)
* [Espace de travail Azure Machine Learning Studio (classique)](../classic/create-workspace.md)

La **machine virtuelle pour la science des données de Microsoft (DSVM)** est également disponible sous forme d’image de machine virtuelle Azure. Cette machine virtuelle est préinstallée et configurée avec plusieurs outils couramment utilisés dans le cadre de l’analyse de données et de l’apprentissage automatique. La machine virtuelle DSVM est disponible sur Windows et Linux. Pour plus d’informations, consultez [Présentation de la machine virtuelle Science des données basée sur le cloud pour Linux et Windows](../data-science-virtual-machine/overview.md).

Découvrez comment créer :

- [Machine virtuelle DSVM Windows](../data-science-virtual-machine/provision-vm.md)
- [Machine virtuelle DSVM Ubuntu](../data-science-virtual-machine/dsvm-ubuntu-intro.md)
- [Machine virtuelle DSVM CentOS](../data-science-virtual-machine/release-notes.md)