---
title: Langues prises en charge
titleSuffix: Azure Data Science Virtual Machine
description: Langages de programmation pris en charge et outils associés, préinstallés sur Data Science Virtual Machine.
keywords: outils de science des données, machine virtuelle science des données, outils pour la science des données, science des données linux
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 8a73336ba0a324c65ee0de764d81a8e69ab6079c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87843264"
---
# <a name="languages-supported-on-the-data-science-virtual-machine"></a>Langages pris en charge sur la machine virtuelle DSVM 

L’environnement DSVM (Data Science Virtual Machine) est fourni avec plusieurs langages et outils de développement prédéfinis qui permettent de créer vos applications d’intelligence artificielle (IA). Voici les plus connus.

## <a name="python-windows-server-2016-edition"></a>Python (Édition Windows Server 2016)

| Category | Valeur |
| ------------- | ------------- |
| Versions de langage prises en charge | Python 2.7 et 3.7 |
| Éditions DSVM prises en charge      | Windows Server 2016     |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Deux environnements `conda` généraux sont créés : <br /> * L’environnement `root` situé dans `/anaconda/` est Python 3.7. <br/> * L’environnement `python2` situé dans `/anaconda/envs/python2` est Python 2.7.       |
| Liens vers des exemples      | Des exemples de notebooks Jupyter pour Python sont inclus.     |
| Outils connexes sur la machine virtuelle DSVM      | PySpark, R, Julia.      |

> [!NOTE]
> Les builds de Windows Server 2016 créées avant mars 2018 contiennent Python 3.5 et Python 2.7. Python 2.7 est l’environnement conda **root** et **py37** est l’environnement Python 3.7.

### <a name="how-to-use-and-run-it"></a>Comment l’utiliser et l’exécuter ?    

* Exécutez à l’invite de commandes :

  Ouvrez une invite de commandes et utilisez l’une des méthodes suivantes, en fonction de la version de Python que vous voulez exécuter :

    ```
    # To run Python 2.7
    activate python2
    python --version
    
    # To run Python 3.7
    activate 
    python --version 
    ```
    
* Utiliser dans un IDE :

  Utilisez Python Tools pour Visual Studio (PTVS), installé avec Visual Studio Community Edition. Par défaut, le seul environnement configuré automatiquement dans PTVS est Python 3.6. 

    > [!NOTE]
    > Pour pointer PTVS vers Python 2.7, vous devez créer un environnement personnalisé dans PTVS. Pour définir le chemin de cet environnement dans Visual Studio Community Edition, accédez à **Outils** -> **Outils Python** -> **Environnements Python**, puis sélectionnez **Personnalisé**. Ensuite, définissez **c:\anaconda\envs\python2** comme emplacement et sélectionnez **Détection automatique**.

* Utiliser dans Jupyter :

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez choisir le type de noyau _Python [Conda Root]_ pour Python 3.7 et _Python [Conda env:python2]_ pour Python 2.7.

* Installer les packages Python :

  Les environnements Python par défaut sur la machine virtuelle DSVM sont des environnements généraux lisibles par tous les utilisateurs. Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer des packages dans l’environnement général, activez l’environnement racine ou python2 en exécutant la commande `activate` en tant qu’administrateur. Vous pouvez ensuite utiliser un gestionnaire de package comme `conda` ou `pip` pour installer ou mettre à jour les packages.

## <a name="python-linux-edition"></a>Python (édition Linux)

| Category | Valeur |
| ------------- | ------------- |
| Versions de langage prises en charge | Python 2.7 et 3.5 |
| Éditions DSVM prises en charge      | Linux   |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Deux environnements `conda` généraux sont créés : <br /> * L’environnement `root` à l’emplacement `/anaconda/` est Python 2.7. <br/> * L’environnement `py35` à l’emplacement `/anaconda/envs/py35` est Python 3.5.       |
| Liens vers des exemples      | Des exemples de notebooks Jupyter pour Python sont inclus.     |
| Outils connexes sur la machine virtuelle DSVM      | PySpark, R, Julia      |

### <a name="how-to-use-and-run-it"></a>Comment l’utiliser et l’exécuter ?    

* Exécuter dans un terminal :

  Ouvrez le terminal et effectuez l’une des étapes suivantes en fonction de la version de Python à exécuter :

    ```
    # To run Python 2.7
    source activate 
    python --version
    
    # To run Python 3.5
    source activate py35
    python --version
    
    ```
* Utiliser dans un IDE :

  Utilisez PyCharm, installé dans l’édition Visual Studio Community. 

* Utiliser dans Jupyter :

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez choisir le type de noyau **Python [Conda Root]** pour Python 2.7 et **Python [Conda env:py35]** pour l’environnement Python 3.5. 

* Installer les packages Python :

  Les environnements Python par défaut sur la machine virtuelle DSVM sont des environnements généraux lisibles par tous les utilisateurs, Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer les packages dans l’environnement général, activez l’environnement root ou py35 en exécutant la commande `source activate` en tant qu’administrateur ou utilisateur disposant d’autorisations sudo. Vous pouvez ensuite utiliser un gestionnaire de package comme `conda` ou `pip` pour installer ou mettre à jour les packages.


## <a name="r"></a>R

| Category | Valeur |
| ------------- | ------------- |
| Versions de langage prises en charge | Microsoft R Open 3.x (100 % compatible avec CRAN-R)<br /> Microsoft R Server 9.x Developer Edition (plateforme R d’entreprise scalable)|
| Éditions DSVM prises en charge      | Linux, Windows     |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Windows : `C:\Program Files\Microsoft\ML Server\R_SERVER` <br />Linux : `/usr/lib64/microsoft-r/3.3/lib64/R`    |
| Liens vers des exemples      | Des exemples de notebooks Jupyter pour R sont inclus.     |
| Outils connexes sur la machine virtuelle DSVM      | SparkR, Python, Julia      |

### <a name="how-to-use-and-run-it"></a>Comment l’utiliser et l’exécuter ?    

**Windows** :

* Exécutez à l’invite de commandes :

  Ouvrez une invite de commandes et tapez `R`.

* Utiliser dans un IDE :

  Utilisez RTools for Visual Studio (RTVS) installé avec Visual Studio Community Edition ou RStudio. Ces outils sont disponibles dans le menu Démarrer ou en tant qu’icône sur le Bureau. 

* Utiliser dans Jupyter

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez définir **R** comme type de noyau pour utiliser le noyau R Jupyter (IRKernel).

* Installer des packages R :

  R est installé sur la machine virtuelle DSVM dans un environnement général lisible par tous les utilisateurs. Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer les packages dans l’environnement général, exécutez R en appliquant l’une des méthodes précédentes. Vous pouvez ensuite utiliser le gestionnaire de package R `install.packages()` pour installer ou mettre à jour les packages.

**Linux** :

* Exécuter dans un terminal :

  Ouvrez un terminal et exécutez `R`.  

* Utiliser dans un IDE :

  Utilisez RStudio, installé sur la machine virtuelle DSVM Linux.  

* Utiliser dans Jupyter :

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez définir **R** comme type de noyau pour utiliser le noyau R Jupyter (IRKernel). 

* Installer des packages R :

  R est installé sur la machine virtuelle DSVM dans un environnement général lisible par tous les utilisateurs. Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer les packages dans l’environnement général, exécutez R en appliquant l’une des méthodes précédentes. Vous pouvez ensuite utiliser le gestionnaire de package R `install.packages()` pour installer ou mettre à jour les packages.


## <a name="julia"></a>Julia

| Category | Valeur |
| ------------- | ------------- |
| Versions de langage prises en charge | 0,6 |
| Éditions DSVM prises en charge      | Linux, Windows     |
| Comment est-il configuré / installé sur la machine virtuelle DSVM ?  | Windows : Installé dans `C:\JuliaPro-VERSION`<br /> Linux : Installé dans `/opt/JuliaPro-VERSION`    |
| Liens vers des exemples      | Des exemples de notebooks Jupyter pour Julia sont inclus.     |
| Outils connexes sur la machine virtuelle DSVM      | Python, R      |

### <a name="how-to-use-and-run-it"></a>Comment l’utiliser et l’exécuter ?    

**Windows** :

* Exécuter à l’invite de commandes

  Ouvrez une invite de commandes et exécutez `julia`.
* Utiliser dans un IDE :

  Utilisez `Juno` avec l’IDE Julia installé sur la machine virtuelle DSVM et disponible en tant que raccourci sur le Bureau.

* Utiliser dans Jupyter :

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez définir **Julia VERSION** comme type de noyau.

* Installer les packages Julia :

  L’emplacement de Julia par défaut est un environnement général lisible par tous les utilisateurs. Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer les packages dans l’environnement général, exécutez Julia en appliquant l’une des méthodes précédentes. Vous pouvez ensuite exécuter des commandes du gestionnaire de package Julia comme `Pkg.add()` pour installer ou mettre à jour les packages.


**Linux** :
* Exécuter dans un terminal :

  Ouvrez un terminal et exécutez `julia`.
* Utiliser dans un IDE :

  Utilisez `Juno` avec l’IDE Julia installé sur la machine virtuelle DSVM et disponible en tant que raccourci dans le menu **Application**.

* Utiliser dans Jupyter :

  Ouvrez Jupyter et sélectionnez **Nouveau** pour créer un notebook. Vous pouvez définir **Julia VERSION** comme type de noyau.

* Installer les packages Julia :

  L’emplacement de Julia par défaut est un environnement général lisible par tous les utilisateurs. Toutefois, seuls les administrateurs peuvent écrire et installer des packages généraux. Pour installer les packages dans l’environnement général, exécutez Julia en appliquant l’une des méthodes précédentes. Vous pouvez ensuite exécuter des commandes du gestionnaire de package Julia comme `Pkg.add()` pour installer ou mettre à jour les packages.

## <a name="other-languages"></a>Autres langages

**C#**  : disponible sur Windows et accessible par le biais de l’édition Visual Studio Community ou à une `Developer Command Prompt for Visual Studio` où vous pouvez exécuter la commande `csc`.

**Java** : OpenJDK est disponible sur les éditions Linux et Windows de la machine virtuelle DSVM, et défini sur le chemin. Pour utiliser Java, vous pouvez taper la commande `javac` ou `java` à une invite de commandes dans Windows ou dans l’interpréteur de commandes Bash dans Linux.

**Node.JS** : Node.js est disponible sur les éditions Linux et Windows de la machine virtuelle DSVM, et défini sur le chemin. Pour accéder à Node.js, tapez la commande `node` ou `npm` à une invite de commandes dans Windows ou dans l’interpréteur de commandes Bash dans Linux. Sur Windows, l’extension Visual Studio pour les outils Node.js est installée afin de fournir un IDE graphique pour le développement de votre application Node.js.

**F#**  : disponible sur Windows et accessible par le biais de l’édition Visual Studio Community ou à une `Developer Command Prompt for Visual Studio` où vous pouvez exécuter la commande `fsc`.
