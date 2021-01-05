---
title: Outils de développement
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez les outils et environnements de développement intégrés disponibles sur Data Science Virtual Machine.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 42a595246f45b2c7423aaf77c95f17243cf8813c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87012381"
---
# <a name="development-tools-on-the-azure-data-science-virtual-machine"></a>Outils de développement sur Azure Data Science Virtual Machine

Data Science Virtual Machine (DSVM) regroupe plusieurs outils populaires dans un environnement de développement intégré (IDE) très productif. Voici quelques-uns des outils disponibles sur la machine virtuelle DSVM.

## <a name="visual-studio-community-edition"></a>Visual Studio Community Edition

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE à usage général      |
| Versions DSVM prises en charge      | Windows : Visual Studio 2017, Windows 2019 : Visual Studio 2019      |
| Utilisations classiques      | Développement de logiciels    |
| Comment est-il configuré et installé sur la machine virtuelle DSVM ?      | Charge de travail Science des données (outils Python et R), charge de travail Azure (Hadoop, Data Lake), Node.js, outils SQL Server, [Azure Machine Learning pour Visual Studio Code](https://github.com/Microsoft/vs-tools-for-ai)    |
| Comment l’utiliser et l’exécuter ?      | Raccourci sur le Bureau (`C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\devenv.exe`). Visuellement, ouvrez Visual Studio à l’aide de l’icône du Bureau ou du menu **Démarrer**. Recherchez des programmes (en tapant touche Windows + S), suivi de **Visual Studio**. À partir d’ici, vous pouvez créer des projets dans des langages comme C#, Python, R et Node.js.   |
| Outils connexes sur la machine virtuelle DSVM      |     Visual Studio Code, RStudio, Juno  |

> [!NOTE]
> Il est possible que vous receviez un message indiquant que votre période d’évaluation a expiré. Entrez les informations d’identification de votre compte Microsoft. Vous pouvez aussi créer un compte gratuit pour accéder à Visual Studio Community.

## <a name="visual-studio-code"></a>Visual Studio Code 

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE à usage général      |
| Versions DSVM prises en charge      | Windows, Linux     |
| Utilisations classiques      | Éditeur de code et Intégration de Git   |
| Comment l’utiliser et l’exécuter ?      | Raccourci sur le Bureau (`C:\Program Files (x86)\Microsoft VS Code\Code.exe`) dans Windows, raccourci sur le Bureau ou terminal (`code`) dans Linux    |
| Outils connexes sur la machine virtuelle DSVM      |     Visual Studio, RStudio, Juno  |

## <a name="rstudio-desktop"></a>RStudio Desktop

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE client pour le langage R   |
| Versions DSVM prises en charge      | Windows, Linux      |
| Utilisations classiques      |  Développement R     |
| Comment l’utiliser et l’exécuter ?      | Raccourci sur le Bureau (`C:\Program Files\RStudio\bin\rstudio.exe`) sur Windows, raccourci sur le Bureau (`/usr/bin/rstudio`) sur Linux      |
| Outils connexes sur la machine virtuelle DSVM      |   Visual Studio, Visual Studio Code, Juno      |

## <a name="rstudio-server"></a>RStudio Server

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE client pour le langage R   |
| Qu’est-ce que c’est ?   | IDE basé sur le web pour R    |
| Versions DSVM prises en charge      | Linux      |
| Utilisations classiques      |  Développement R     |
| Comment l’utiliser et l’exécuter ?      | Activez le service avec _systemctl enable rstudio-server_, puis démarrez-le avec _systemctl start rstudio-server_. Connectez-vous ensuite à RStudio Server sur http:\//adresse_IP_de_votre_machine_virtuelle:8787.       |
| Outils connexes sur la machine virtuelle DSVM      |   Visual Studio, Visual Studio Code, RStudio Desktop      |

## <a name="juno"></a>Juno 

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE client pour le langage Julia   |
| Versions DSVM prises en charge      | Windows, Linux      |
| Utilisations classiques      |  Développement Julia     |
| Comment l’utiliser et l’exécuter ?      | Raccourci sur le Bureau (`C:\JuliaPro-0.5.1.1\Juno.bat`) sur Windows, raccourci sur le Bureau (`/opt/JuliaPro-VERSION/Juno`) sur Linux      |
| Outils connexes sur la machine virtuelle DSVM      |   Visual Studio, Visual Studio Code, RStudio      |

## <a name="pycharm"></a>Pycharm

| Category | Valeur |
| ------------- | ------------- |
| Qu’est-ce que c’est ?   | IDE client pour le langage Python    |
| Versions DSVM prises en charge      | Windows 2019, Linux      |
| Utilisations classiques      |  Développement Python     |
| Comment l’utiliser et l’exécuter ?      | Raccourci sur le Bureau (`C:\Program Files\tk`) sur Windows. Raccourci sur le Bureau (`/usr/bin/pycharm`) sur Linux      |
| Outils connexes sur la machine virtuelle DSVM      |   Visual Studio, Visual Studio Code, RStudio      |
