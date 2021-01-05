---
title: Qu’est-ce que la machine virtuelle Science des données Azure ?
titleSuffix: Azure Data Science Virtual Machine
description: Vue d’ensemble d’Azure Data Science Virtual Machine - Une machine virtuelle facile à utiliser sur la plateforme cloud Azure avec des bibliothèques et des outils préinstallés et configurés pour la science des données.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: overview
ms.date: 04/02/2020
ms.openlocfilehash: 7bcee0f571fe42e4c00499f188e35d0e0ff2ebe7
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322495"
---
# <a name="what-is-the-azure-data-science-virtual-machine-for-linux-and-windows"></a>Qu’est-ce qu’Azure Data Science Virtual Machine pour Linux et Windows ?

La Data Science Virtual Machine (DSVM) est une image de machine virtuelle personnalisée sur la plateforme cloud Azure spécialement conçue pour la science des données. De nombreux outils de science des données répandus y sont préinstallés et préconfigurés, qui permettent de commencer à créer des applications intelligentes pour de l’analytique avancée.

La Data Science Virtual Machine est disponible sur :

+ Windows Server 2019
+ Ubuntu 18.04 LTS

## <a name="comparison-with-azure-machine-learning"></a>Comparaison avec Azure Machine Learning

DSVM est une image de machine virtuelle personnalisée pour la science des données, mais [Azure Machine Learning](../overview-what-is-azure-ml.md) (AzureML) est une plateforme de bout en bout qui comprend :

+ Capacité de calcul managée
  + Compute Instances (Instances de calcul)
  + Clusters de calcul pour les tâches ML distribuées
  + Clusters d’inférence pour la notation en temps réel
+ Magasins de données (par exemple Blob, ADLS Gen2, SQL DB)
+ Suivi des expériences
+ La gestion des modèles
+ Notebooks
+ Environnements (gérer les dépendances Conda et R)
+ L’étiquetage
+ Pipelines (automatiser les workflows de science des données de bout en bout)

### <a name="comparison-with-azureml-compute-instances"></a>Comparaison avec les instances de calcul AzureML

Les [instances de calcul Azure Machine Learning](../concept-compute-instance.md) sont des images de machine virtuelle entièrement configurées et __managées__ , tandis que DSVM est une machine virtuelle __non managée__.

Les principales différences entre ces deux offres de produit sont détaillées ci-dessous :


|Fonctionnalité |Science des données<br>Machine virtuelle |AzureML<br>Instance de calcul  | 
|---------|---------|---------|
| Entièrement managée | Non        | Oui        |
|Prise en charge de la langue     |  Python, R, Julia, SQL, C#,<br> Java, Node.js, F#       | Python et R        |
|Système d'exploitation     | Ubuntu<br>Windows         |    Ubuntu     |
|Option GPU préconfigurée     |  Oui       |    Oui     |
|Option de scale-up | Oui | Oui |
|Accès SSH    | Oui        |    Oui     |
|Accès RDP    | Oui        |     Non    |
|Intégré<br>Notebooks hébergés     |   Non<br>(nécessite une configuration supplémentaire)      |      Oui   |
|Authentification unique intégrée     | Non <br>(nécessite une configuration supplémentaire)         |    Oui     |
|Collaboration intégrée     | Non         | Oui        |
|Outils préinstallés     |  Jupyter(lab), RStudio Server, VSCode,<br> Visual Studio, PyCharm, Juno,<br>Power BI Desktop, SSMS, <br>Microsoft Office 365, Apache Drill       |     Jupyter(lab)<br> RStudio Server   |

## <a name="sample-use-cases"></a>Exemples de cas d’utilisation

Nous illustrons ci-dessous quelques cas d’utilisation courants pour les clients DSVM.

### <a name="short-term-experimentation-and-evaluation"></a>Expérimentation et évaluation à court terme

Vous pouvez utiliser une DSVM pour évaluer et apprendre de nouveaux [outils](./tools-included.md) de science des données, notamment en consultant nos [exemples et procédures pas à pas](./dsvm-samples-and-walkthroughs.md).

### <a name="deep-learning-with-gpus"></a>Deep Learning avec les GPU

Dans la DSVM, vos modèles d’entraînement peuvent utiliser des algorithmes de deep learning sur du matériel basé sur des unités de traitement graphique (GPU). En tirant parti des fonctionnalités de mise à l’échelle des machines virtuelles de la plateforme Azure, la DSVM vous aide à utiliser le matériel basé sur GPU dans le cloud en fonction de vos besoins. Vous pouvez basculer vers une machine virtuelle basée sur GPU quand vous entraînez de grands modèles ou que vous avez besoin d’une grande rapidité de calcul, mais souhaitez garder le même disque de système d’exploitation. Vous pouvez choisir une des références SKU de machines virtuelles de série N activées pour les GPU avec DSVM. Notez que les références SKU de machines virtuelles activées pour les GPU ne sont pas prises en charge sur les comptes Azure gratuits.

Les pilotes GPU, les frameworks et les versions GPU des frameworks de deep learning sont préinstallés sur les éditions Windows de la DSVM. Sur les éditions Linux, le deep learning sur les GPU est activé sur les DSVM Ubuntu. 

Vous pouvez également déployer l’édition Ubuntu ou Windows de la DSVM sur une machine virtuelle Azure qui n’est pas basée sur des GPU. Dans ce cas, toutes les infrastructures d’apprentissage profond repasseront en mode UC.

[Découvrez-en plus sur les frameworks de deep learning et d’intelligence artificielle disponibles](dsvm-tools-deep-learning-frameworks.md).

### <a name="data-science-training-and-education"></a>Formation et éducation de la science des données

Les formateurs et instructeurs en entreprise qui enseignent la science des données fournissent généralement une image de machine virtuelle. Avec l’image, les étudiants disposent d’une installation cohérente et d’exemples qui fonctionnent de façon prévisible.

La DSVM crée un environnement à la demande avec une configuration cohérente qui simplifie le support et les soucis de compatibilité. Ceci est d’autant plus bénéfique dans les cas où ces environnements doivent être créés fréquemment, en particulier lors de formations plus courtes.


## <a name="whats-included-on-the-dsvm"></a>Ce qui est inclut dans la DSVM

Pour voir la liste complète des outils des DSVM Windows et Linux, [cliquez ici](tools-included.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez les articles suivants :

+ Windows :
  + [Configurer une DSVM Windows](provision-vm.md)
  + [Science des données sur une DSVM Windows](vm-do-ten-things.md)

+ Linux :
  + [Configurer une DSVM Linux (Ubuntu)](dsvm-ubuntu-intro.md)
  + [Science des données sur une DSVM Linux](linux-dsvm-walkthrough.md)