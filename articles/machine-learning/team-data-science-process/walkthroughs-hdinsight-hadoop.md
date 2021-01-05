---
title: Analytique sur Azure HDInsight Hadoop avec Hive – Team Data Science Process
description: Exemples du processus TDSP (Team Data Science Process) qui montrent comment utiliser Hive sur Azure HDInsight Hadoop pour effectuer des analyses prédictives.
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
ms.openlocfilehash: f72ea6ed5f0eec076d181ef56c99c4f1308a7741
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75864160"
---
# <a name="hdinsight-hadoop-data-science-walkthroughs-using-hive-on-azure"></a>Guides de la science des données HDInsight Hadoop avec Hive sur Azure 

Ces guides utilisent Hive avec un cluster HDInsight Hadoop pour effectuer des analyses prédictives. Ils suivent les étapes décrites dans le processus TDSP (Team Data Science Process). Vous trouverez une vue d’ensemble du processus TDSP sur la page [Processus de science des données](overview.md) Vous trouverez une présentation d’Azure HDInsight sur la page [Présentation d’Azure HDInsight, la pile de technologies Hadoop, et des clusters Hadoop](../../hdinsight/hadoop/apache-hadoop-introduction.md).

D’autres guides de la science des données qui appliquent le processus TDSP sont regroupés en fonction de la **plateforme** qu’ils utilisent. Consultez [Procédures pas à pas du processus TDSP (Team Data Science Process)](walkthroughs.md) pour connaître les détails de ces exemples.


## <a name="predict-taxi-tips-using-hive-with-hdinsight-hadoop"></a>Prédire les pourboires laissés aux taxis avec Hive et HDInsight Hadoop

Le guide [Utiliser les clusters HDInsight Hadoop](hive-walkthrough.md) utilise des données sur les taxis de New York pour prédire : 

- si un pourboire sera laissé ; 
- la distribution du montant des pourboires.

Le scénario est implémenté avec Hive et un [cluster Azure HDInsight Hadoop](https://azure.microsoft.com/services/hdinsight/). Vous découvrirez comment stocker et explorer des données, et effectuer du feature engineering à partir d’un jeu de données public de courses et de tarifs des taxis new-yorkais. Vous utiliserez également Azure Machine Learning pour générer et déployer les modèles.

## <a name="predict-advertisement-clicks-using-hive-with-hdinsight-hadoop"></a>Prédire les clics publicitaires avec Hive et HDInsight Hadoop

Le guide [Utiliser des clusters Azure HDInsight Hadoop sur un jeu de données de 1 To](hive-criteo-walkthrough.md) utilise un jeu de données de clics [Criteo](https://labs.criteo.com/downloads/download-terabyte-click-logs/) public pour prédire si un pourboire sera laissé, ainsi que les montants attendus. Le scénario est implémenté avec Hive et un [cluster Azure HDInsight Hadoop](https://azure.microsoft.com/services/hdinsight/) pour stocker, explorer et sous-échantillonner des données, ainsi qu’effectuer du feature engineering. Il utilise Azure Machine Learning pour générer, entraîner et évaluer un modèle de classification binaire qui prédit si un utilisateur cliquera sur une publicité. Le guide se conclut en expliquant comment publier un de ces modèles en tant que service web.


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les composants clés qui constituent le processus TDSP (Team Data Science Process), consultez la page [Vue d’ensemble du processus TDSP (Team Data Science Process)](overview.md).

Pour en savoir plus sur le cycle de vie du processus TDSP (Team Data Science Process), qui permet de structurer les projets de science des données, consultez la page [Cycle de vie du processus TDSP (Team Data Science Process)](lifecycle.md). Le cycle de vie présente les étapes, du début à la fin, que les projets suivent généralement quand ils sont exécutés. 

