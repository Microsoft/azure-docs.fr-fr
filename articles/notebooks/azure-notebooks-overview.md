---
title: Vue d’ensemble d’Azure Notebooks (préversion)
description: Exécutez des Notebooks Jupyter dans le cloud à l’aide du service gratuit Azure Notebooks (préversion), pour lequel aucune installation ou configuration n’est nécessaire.
ms.topic: overview
ms.date: 04/05/2019
ms.openlocfilehash: 7e622bdef785085384395bb0c8ac3efba2b2053a
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400257"
---
# <a name="overview-of-azure-notebooks-preview"></a>Vue d’ensemble d’Azure Notebooks (préversion)

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Azure Notebooks est un service hébergé gratuit à l’aide duquel vous pouvez développer et exécuter des Jupyter Notebooks dans le cloud sans aucune installation. [Jupyter](https://jupyter.org/) (anciennement IPython) est un projet open source qui vous permet de combiner facilement du texte Markdown, du code exécutable, des données persistantes, des graphiques et des visualisations dans un canevas partageable appelé *notebook* (image tirée de jupyter.org) :

[![Exemples de Notebooks Jupyter](https://jupyter.org/assets/jupyterpreview.png)](https://jupyter.org/assets/jupyterpreview.png#lightbox)

Fort de cette solide combinaison de code, de graphiques et de texte explicatif, Jupyter est devenu populaire dans de nombreuses utilisations, notamment les instructions de science des données, le nettoyage et la transformation des données, la simulation numérique, la modélisation statistique et le développement de modèles Machine Learning.

## <a name="hassle-free-experience"></a>Expérience simplifiée

Azure Notebooks vous permet d’aborder rapidement le prototypage, la science des données, la recherche universitaire ou la programmation en Python :

- Un scientifique des données dispose d’un accès instantané à un environnement Anaconda complet sans aucune installation.
- Un enseignant peut fournir un environnement Python simple aux étudiants.
- Un présentateur peut donner une conférence ou un webinaire en direct sans demander aux participants de consacrer 45 minutes à installer un logiciel.
- Un développeur ou un amateur peut utiliser des notebooks en guise de bloc-notes de code.

Les notebooks deviennent encore plus puissants quand les personnes peuvent collaborer dessus par le biais d’un service cloud accessible via le navigateur comme Azure Notebooks (en préversion). Dans le cloud, les utilisateurs n’ont pas besoin d’installer Jupyter localement ni de se soucier de la gestion d’un environnement. Le cloud facilite également le partage des notebooks (et des fichiers de données associés) avec d’autres utilisateurs autorisés, évitant ainsi les complications liées au partage de notebooks par des moyens externes tels que les dépôts de contrôle de code source. Avec Azure Notebooks, les utilisateurs peuvent également copier (ou « cloner ») des notebooks dans leur propre compte à des fins de modification ou d’expérimentation, ce qui est particulièrement utile pour la formation.

Azure Notebooks étant une plateforme générale d’écriture, d’exécution et de partage de code, vous pouvez l’utiliser pour de nombreux scénarios différents :

- Apprendre un nouveau langage de programmation : essayez l’un de [tutoriels de présentation](https://notebooks.azure.com/Microsoft/projects/samples/html/Introduction%20to%20Python.ipynb)
- Découvrir la science des données : consultez l’[ouvrage de Jake VanderPlas](https://notebooks.azure.com/jakevdp/projects/PythonDataScienceHandbook)
- [Dispenser un cours](https://notebooks.azure.com/garth-wells/projects/CUED-IA-Computing-Michaelmas) pour des centaines d’étudiants
- Animer un webinaire en ligne ou lors d’une conférence sans consacrer de temps à l’installation 
- Permettre aux utilisateurs de GitHub de charger et d’exécuter directement des notebooks en [créant un badge de lancement GitHub](https://notebooks.azure.com/help/projects/sharing/create-a-github-badge)
- Diffuser des [diaporamas semblables à ceux de PowerPoint](https://notebooks.azure.com/help/jupyter-notebooks/slides) où le code dans les diapositives est exécutable !

En bref, Azure Notebooks vous aide à accomplir votre travail plus efficacement.

> [!Note]
> Vous trouverez plus d’informations sur Jupyter proprement dit sur [jupyter.org](https://jupyter.org/) et dans la [documentation Jupyter](https://jupyter-notebook.readthedocs.io/en/latest/).

## <a name="pricing-and-quotas"></a>Tarifs et quotas

Azure Notebooks est un service gratuit, mais chaque projet est limité à 4 Go de mémoire et 1 Go de données afin d’éviter tout abus. Les utilisateurs légitimes qui dépassent ces limites doivent passer un test Captcha pour continuer à exécuter des notebooks.

Pour lever toutes les limites, connectez-vous à Azure Notebooks avec un compte Azure Active Directory (par exemple un compte d’entreprise). Si ce compte est associé à un abonnement Azure, vous pouvez vous connecter à n’importe quelle instance d’Azure Data Science Virtual Machine au sein de cet abonnement. 

La durée d’existence garantie des serveurs de notebooks est de 8 heures maximum. Dans la plupart des cas, votre conteneur n’est pas soumis à cette limite et continue de s’exécuter après cette date. Toutefois, les sessions de longue durée peuvent parfois être arrêtées pour maintenir la stabilité du système.

## <a name="available-kernels-and-environments"></a>Environnements et noyaux disponibles

Pour chaque notebook, vous sélectionnez le noyau (autrement dit, l’environnement d’exécution) utilisé pour exécuter les cellules de code. Azure Notebooks prend en charge les noyaux suivants :

- Python 2.7 + Anaconda2-5.3.0
- Python 3.6 + Anaconda3-5.3.0
- Python 3.5 + Anaconda3-4.2.0 (va être déprécié)
- R 3.4.1 + Microsoft R Open 3.4.1
- F# 4.1.9

Azure Notebooks contient également des packages supplémentaires en plus des distributions de base. Les noyaux Python, par exemple, comprennent les bibliothèques numpy, pandas, scikit-learn, matplotlib et bokeh.

Vous pouvez également personnaliser un projet afin de créer un environnement pour tous les notebooks de ce projet.

En plus des distributions de base, Azure Notebooks est préinstallé avec de nombreux packages supplémentaires utiles aux scientifiques des données. Vous pouvez aussi installer vos propres packages à l’aide de la procédure standard pour chaque langage.

## <a name="pre-configured-jupyter-extensions"></a>Extensions Jupyter préconfigurées

Azure Notebooks est préconfiguré avec les extensions Jupyter suivantes :

- [RISE](https://github.com/damianavila/RISE) : extension de diaporama Jupyter (également appelée live_reveal).
- [JupyterLab](https://github.com/jupyterlab/jupyterlab) : environnement de calcul complet pour travailler avec des Notebooks Jupyter.
- [Altair](https://github.com/ellisonbg/altair) : bibliothèque de visualisation de statistiques déclarative pour Python.
- [BQPlot](https://github.com/bloomberg/bqplot) : framework de traçage interactif pour notebooks Jupyter.
- [IpyWidgets](https://github.com/jupyter-widgets/ipywidgets) : widgets HTML interactifs pour notebooks Jupyter.

## <a name="issues-and-getting-help"></a>Problèmes et assistance

Azure Notebooks étant toujours en préversion, le service peut subir des pannes temporaires qui peuvent être plus fréquentes ou plus longues que celles d’autres services Azure. Certaines fonctionnalités peuvent être incomplètes ou contenir des bogues.

À l’heure actuelle, nous vous déconseillons d’utiliser la préversion d’Azure Notebooks pour les applications critiques pour l’entreprise ou pour les notebooks et les données sensibles.

Pour discuter de vos questions sur Azure Notebooks, signalez un problème dans le [dépôt GitHub](https://github.com/Microsoft/AzureNotebooks/issues).

## <a name="next-steps"></a>Étapes suivantes  

- Guides de démarrage rapide :

  - [Se connecter et définir un identifiant utilisateur](quickstart-sign-in-azure-notebooks.md)
  - [Exporter un Notebook Jupyter](quickstart-export-jupyter-notebook-project.md)
