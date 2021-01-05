---
title: Exemples de types de classes dans Azure Lab Services | Microsoft Docs
description: Cite certains des types de classes pour lesquels il est possible de configurer des laboratoires à l’aide d’Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 7ebc89c040be732c230f0c3f54f68fb91466eb68
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436400"
---
# <a name="class-types-overview---azure-lab-services"></a>Vue d’ensemble des types de classes - Azure Lab Services

Azure Lab Services vous permet de configurer rapidement un environnement lab de salle de classe dans le cloud. Les articles de cette section fournissent des conseils sur la configuration de plusieurs types de labos avec Azure Lab Services.

## <a name="big-data-analytics"></a>Analytique du Big Data
Vous pouvez configurer un labo de GPU pour enseigner une classe Analytique du Big Data. Grâce à ce type de classe, les étudiants apprennent à gérer de gros volumes de données et à appliquer des algorithmes d’apprentissage automatique et statistique pour obtenir des informations sur les données. L’un des principaux objectifs pour les étudiants est d’apprendre à utiliser les outils d’Analytique données, tels que le package logiciel open source d’Apache Hadoop qui fournit des outils pour le stockage, la gestion et le traitement du Big Data. 

Pour plus d’informations sur la façon de configurer ce type de labo, consultez [Configurer un labo pour l’analytique du Big Data à l’aide du déploiement Docker de Hortonworks Data Platform](class-type-big-data-analytics.md).

## <a name="database-management"></a>Gestion de bases de données
Les concepts des bases de données constituent l’un des cours d’introduction abordés dans la plupart des départements informatiques à l’université. Vous pouvez configurer un labo pour un cours élémentaire de gestion de bases de données dans Azure Lab Services. Par exemple, vous pouvez configurer un modèle de machine virtuelle dans un labo avec un serveur de base de données [MySQL](https://www.mysql.com/) ou un serveur [SQL Server 2019](https://www.microsoft.com/sql-server/sql-server-2019).

Pour des informations détaillées sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner la gestion des bases de données pour les bases de données relationnelles](class-type-database-management.md).

## <a name="deep-learning-in-natural-language-processing"></a>Deep Learning pour le traitement en langage naturel
Vous pouvez configurer un laboratoire axé sur le Deep Learning dans le cadre du traitement en langage naturel (NLP) à l’aide d’Azure Lab Services. Le traitement en langage naturel (NLP) est une forme d’intelligence artificielle (IA) qui fournit aux ordinateurs des fonctionnalités de traduction, de reconnaissance vocale et d’autres fonctionnalités de compréhension de la langue. Les étudiants qui suivent un cours de traitement en langage naturel disposent d’une machine virtuelle Linux pour apprendre à appliquer des algorithmes de réseau neuronal afin de développer des modèles de deep learning utilisés pour l’analyse de la langue humaine écrite.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo axé sur le Deep Learning pour le traitement en langage naturel à l’aide d’Azure Lab Services](class-type-deep-learning-natural-language-processing.md).

## <a name="ethical-hacking"></a>Piratage éthique
Vous pouvez configurer un labo de classe qui se concentre sur l’analyse forensique du piratage éthique. Des tests d’intrusion, une pratique utilisée par la communauté de piratage éthique, sont effectués quand quelqu’un tente d’accéder au système ou au réseau pour détecter les vulnérabilités qu’un attaquant malveillant pourrait exploiter.

Dans une classe sur le piratage éthique, les étudiants apprennent les techniques modernes de défense face aux vulnérabilités. Chaque étudiant a accès à une machine virtuelle hôte Windows Server qui comporte deux machines virtuelles imbriquées : une avec une image [Metasploitable3](https://github.com/rapid7/metasploitable3) et une autre avec une image [Kali Linux](https://www.kali.org/). La machine virtuelle Metasploitable est utilisée à des fins d’exploitation.  La machine virtuelle Kali Linux permet d’accéder aux outils nécessaires pour exécuter des tâches d’investigation.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner le piratage éthique](class-type-ethical-hacking.md).

## <a name="matlab"></a>MATLAB
[MATLAB](https://www.mathworks.com/products/matlab.html), qui signifie Matrix Laboratory, est la plate-forme de programmation de [MathWorks](https://www.mathworks.com/).  Elle combine la puissance de calcul et la visualisation, ce qui en fait un outil populaire dans les domaines des mathématiques, de l’ingénierie, de la physique et de la chimie.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner MATLAB](class-type-matlab.md).

## <a name="project-lead-the-way-pltw"></a>Project Lead the Way (PLTW)
[Project Lead the Way (PLTW)](https://www.pltw.org/) est un organisme à but non lucratif qui propose des programmes éducatifs à l’intention des élèves du primaire à travers les États-Unis dans les domaines de l’informatique, de l’ingénierie et des sciences biomédicales.  Dans chaque cours PLTW, les élèves utilisent diverses applications logicielles dans le cadre de leur apprentissage pratique.

Pour plus d’informations sur la façon de configurer ces types de laboratoires, consultez [Configurer des labos pour des cours Project Lead the Way](class-type-pltw.md).

## <a name="python-and-jupyter-notebooks"></a>Python et Jupyter Notebooks
Vous pouvez configurer une machine modèle dans Azure Lab Services avec les outils nécessaires pour enseigner aux élèves comment utiliser [Jupyter Notebooks](http://jupyter-notebook.readthedocs.io). Jupyter Notebooks est un projet open source qui vous permet de combiner facilement du texte enrichi et du code source [Python](https://www.python.org/) exécutable sur un seul canevas appelé notebook. L’exécution d’un notebook produit un enregistrement linéaire des entrées et des sorties.  Ces sorties peuvent inclure du texte, des tables d’informations, des nuages de points, etc.

Pour des informations détaillées sur la configuration de ce type de labo, consultez [Configurer un labo pour enseigner la science des données avec Python et Jupyter Notebooks](class-type-jupyter-notebook.md).

## <a name="shell-scripting-on-linux"></a>Scripts shell sur Linux
Vous pouvez configurer un laboratoire pour enseigner la création de scripts shell sur Linux. Dans le cadre de l’administration système, l’écriture de scripts permet aux administrateurs d’éviter les tâches répétitives. Dans cet exemple de scénario, les scripts bash traditionnels et les scripts améliorés sont abordés. Les scripts améliorés sont des scripts qui associent des commandes bash et Ruby. Cette approche permet à Ruby de passer des données et fournit des commandes bash pour interagir avec le shell.

Les étudiants qui suivent ces cours d’écriture de scripts disposent d’une machine virtuelle Linux pour apprendre les bases de Linux et également se familiariser avec les scripts de shell bash. La machine virtuelle Linux est fournie avec un accès au Bureau à distance activé. En outre, les éditeurs de texte [gedit](https://help.gnome.org/users/gedit/stable/) et [Visual Studio Code](https://code.visualstudio.com/) y sont installés.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Scripts shell sur Linux](class-type-shell-scripting-linux.md).

## <a name="solidworks-computer-aided-design-cad"></a>Conception assistée par ordinateur (CAO) SolidWorks
Vous pouvez configurer un laboratoire GPU qui donne aux étudiants d’ingénierie l’accès à [SolidWorks](https://www.solidworks.com/).  SolidWorks fournit un environnement de CAO 3D pour la modélisation des objets solides.  Avec SolidWorks, les ingénieurs peuvent facilement créer, visualiser, simuler et documenter leurs conceptions.

Pour plus d’informations sur la configuration de ce type de labo, consultez [Configurer un labo pour les cours d’ingénierie avec SolidWorks](class-type-solidworks.md)

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Configurer un laboratoire axé sur le Deep Learning dans le cadre du traitement en langage naturel (NLP) à l’aide d’Azure Lab Services](class-type-deep-learning-natural-language-processing.md)
- [Scripts shell sur Linux](class-type-shell-scripting-linux.md)
- [Piratage éthique](class-type-ethical-hacking.md)