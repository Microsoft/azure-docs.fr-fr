---
title: 'ML Studio (classique) : Exécution de scripts Python – Azure'
description: Découvrez comment utiliser le module Exécuter le script Python pour utiliser du code Python dans des expériences Machine Learning Studio (classique) et des services web.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: devx-track-python, previous-author=heatherbshapiro, previous-ms.author=hshapiro
ms.date: 03/12/2019
ms.openlocfilehash: 822bb43aaeb3cc1cf5914bb1600ef754441c4206
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93310201"
---
# <a name="execute-python-machine-learning-scripts-in-azure-machine-learning-studio-classic"></a>Exécution des scripts d’apprentissage automatique Python dans Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Python est un précieux outil dans le coffre à outils de nombreux scientifiques de données. Il est utilisé à chaque étape des flux de travail d’apprentissage automatique typiques, y compris l'exploration des données, l'extraction des fonctionnalités, la formation, la validation et le déploiement de modèles.

Cet article explique comment utiliser le module Exécuter le script Python pour utiliser du code Python dans vos expériences Azure Machine Learning Studio (classique) et vos services web.

## <a name="using-the-execute-python-script-module"></a>Utilisation du module Exécuter le script Python

L’interface principale de Python dans Studio (classique) est accessible via le module [Exécuter le script Python][execute-python-script]. Il accepte jusqu’à trois entrées et génère jusqu’à deux sorties, comme le module [Exécuter le script R][execute-r-script]. Le code Python est saisi dans la boîte de paramètres à l’aide d’une fonction de point d’entrée spécialement nommée, appelée `azureml_main`.

![Module Exécuter un script Python](./media/execute-python-scripts/execute-machine-learning-python-scripts-module.png)

![Exemple de code Python dans la zone des paramètres du module](./media/execute-python-scripts/embedded-machine-learning-python-script.png)

### <a name="input-parameters"></a>Paramètres d'entrée

Des entrées vers le module Python sont exposées sous forme de trames de données Pandas. La fonction `azureml_main` accepte jusqu'à deux trames de données Pandas facultatives en tant que paramètres.

Le mappage entre les ports d’entrée et les paramètres de fonction est positionnel :

- Le premier port d’entrée connecté est mappé au premier paramètre de la fonction.
- La seconde entrée (si elle est connectée) est mappée au second paramètre de la fonction.
- La troisième entrée est utilisée pour [importer d’autres modules Python](#import-modules).

Les sémantiques plus détaillées qui décrivent la procédure à adopter pour mapper les ports d'entrée vers les paramètres de la fonction `azureml_main` sont mentionnées ci-dessous.

![Tableau des configurations de ports d'entrée et signature Python résultante](./media/execute-python-scripts/python-script-inputs-mapped-to-parameters.png)

### <a name="output-return-values"></a>Valeurs renvoyées pour la sortie

La fonction `azureml_main` doit renvoyer une seule image de données Pandas empaquetée dans une [séquence](https://docs.python.org/2/c-api/sequence.html) Python telle qu'un tuple, une liste ou un tableau NumPy. Le premier élément de cette séquence est renvoyé dans le premier port de sortie du module. Le second port de sortie du module est utilisé pour les [visualisations](#visualizations) et ne nécessite aucune valeur de retour. Ce schéma est illustré ci-dessous.

![Mappage des ports d'entrée vers des paramètres et valeur de renvoi vers le port de sortie](./media/execute-python-scripts/map-of-python-script-inputs-outputs.png)

## <a name="translation-of-input-and-output-data-types"></a>Conversion des types de données d’entrée et de sortie

Les jeux de données Studio sont différents des trames de données Pandas. Par conséquent, les jeux de données d’entrée dans Studio (classique) sont convertis en trames de données Pandas, et les trames de données en sortie sont reconverties en jeux de données Studio (classique). Pendant ce processus de conversion, les conversions suivantes sont également effectuées :

 **Type de données Python** | **Procédure de traduction Studio** |
| --- | --- |
| Chaînes et valeurs numériques| Traduit tel quel |
| Pandas 'NA' | Traduit par 'Missing value' (valeur manquante) |
| Vecteurs d’index | Non pris en charge* |
| Noms de colonnes non-chaîne | Appeler `str` sur les noms de colonnes |
| Dupliquer les noms de colonnes | Ajoutez un suffixe numérique : (1), (2), (3), et ainsi de suite.

**Les tableaux de données d’entrée dans la fonction Python ont toujours un index numérique de 64 bits compris entre 0 et le nombre de lignes moins 1*

## <a name="importing-existing-python-script-modules"></a><a id="import-modules"></a>Importation des modules de script Python existants

Le serveur principal utilisé pour exécuter Python est basé sur [Anaconda](https://www.anaconda.com/distribution/), une distribution scientifique Python largement utilisée. Il est fourni avec près de 200 des packages Python les plus couramment utilisés dans des charges de travail centrées sur les données. Actuellement, Studio (classique) ne prend pas en charge l’utilisation de systèmes de gestion de paquets comme Pip ou Conda pour installer et gérer des bibliothèques externes.  Si vous jugez nécessaire d'incorporer d’autres bibliothèques, utilisez le scénario suivant comme guide.

Un cas typique consiste à incorporer des scripts Python existants dans des expériences Studio (classique). Le module [Exécuter le script Python][execute-python-script] accepte un fichier zip contenant des modules Python au troisième port d’entrée. Le fichier est décompressé par l’infrastructure d’exécution lors de l’exécution, et le contenu est ajouté au chemin de bibliothèque de l’interpréteur Python. La fonction du point d'entrée `azureml_main` peut ensuite importer ces modules directement. 

Prenez l’exemple du fichier Hello.py contenant une fonction simple « Hello, World ».

![Fonction définie par l’utilisateur dans le fichier Hello.py](./media/execute-python-scripts/figure4.png)

Ensuite, nous créons un fichier Hello.zip contenant Hello.py :

![Fichier zip contenant le code Python défini par l'utilisateur](./media/execute-python-scripts/figure5.png)

Chargez le fichier zip en tant que jeu de données dans Studio (classique). Créez et exécutez ensuite une expérience qui utilise le code Python dans le fichier Hello.zip en le joignant au troisième port d’entrée du module **Exécuter le script Python** , comme illustré dans l’image suivante.

![Exemple d’expérience avec Hello.zip comme entrée d’un module Exécuter le script Python](./media/execute-python-scripts/figure6a.png)

![Code Python défini par l'utilisateur et chargé en tant que fichier zip](./media/execute-python-scripts/figure6b.png)

La sortie du module montre que le fichier zip a été décompressé et que la fonction `print_hello` a été exécutée.

![Sortie du module indiquant une fonction définie par l'utilisateur](./media/execute-python-scripts/figure7.png)

## <a name="accessing-azure-storage-blobs"></a>Accès à Stockage Blob Azure

Vous pouvez accéder aux données stockées dans un compte Stockage Blob Azure en suivant ces étapes :

1. Téléchargez localement le [package Stockage Blob Azure pour Python](https://azuremlpackagesupport.blob.core.windows.net/python/azure.zip).
1. Chargez le fichier zip sur votre espace de travail Studio (classique) en tant que jeu de données.
1. Créer votre objet BlobService avec `protocol='http'`

```
from azure.storage.blob import BlockBlobService

# Create the BlockBlockService that is used to call the Blob service for the storage account
block_blob_service = BlockBlobService(account_name='account_name', account_key='account_key', protocol='http')
```

1. Désactiver **Transfert sécurisé requis** dans votre onglet de paramètres **Configuration** de stockage

![Désactiver l’option Transfert sécurisé requis dans le portail Azure](./media/execute-python-scripts/disable-secure-transfer-required.png)

## <a name="operationalizing-python-scripts"></a>Mise en place des scripts Python

Les modules [Exécuter le script Python][execute-python-script] utilisés dans une expérience de notation sont appelés lorsqu’ils sont publiés en tant que service web. Par exemple, l’image ci-dessous illustre une expérience de notation qui détient le code permettant d’évaluer une seule expression Python.

![Espace de travail Studio pour un service web](./media/execute-python-scripts/figure3a.png)

![Expression Pandas Python](./media/execute-python-scripts/python-script-with-python-pandas.png)

Un service web créé à partir de cette expérience effectue les actions suivantes :

1. Utilise une expression Python comme entrée (sous forme de chaîne)
1. Envoie une expression Python à l’interpréteur Python
1. Renvoie une table contenant l’expression et le résultat évalué.

## <a name="working-with-visualizations"></a><a id="visualizations"></a>Utilisation des visualisations

Les graphiques créés à l'aide de MatplotLib peuvent être renvoyés par le module [Exécuter le script Python][execute-python-script]. Mais les graphiques ne sont pas redirigés automatiquement vers des images, comme lorsque vous utilisez R. De ce fait, l'utilisateur doit enregistrer explicitement les graphiques en fichiers PNG.

Pour générer des images à partir de MatplotLib, vous devez vous effectuer les étapes suivantes :

1. Basculez le back-end sur « AGG » à partir du convertisseur par défaut basé sur Qt.
1. Créez un objet figure.
1. Obtenez l'axe et générez sur celui-ci tous les graphiques.
1. Enregistrez la figure dans un fichier PNG.

Ce processus est mentionné dans les images ci-dessous, qui créent une matrice de nuages de points à l’aide de la fonction scatter_matrix dans Pandas.

![Code pour enregistrer les figures MatplotLib dans des images](./media/execute-python-scripts/figure-v1-8.png)

![Clic sur Visualiser dans un module Exécuter le script Python pour afficher les figures](./media/execute-python-scripts/figure-v2-9a.png)

![Visualisation des graphiques pour un exemple d’expérience utilisant du code Python](./media/execute-python-scripts/figure-v2-9b.png)

Il est possible de renvoyer plusieurs figures en les enregistrant dans différentes images. Le runtime Studio (classique) récupère toutes les images et les concatène pour la visualisation.

## <a name="advanced-examples"></a>Exemples avancés

L’environnement Anaconda installé dans Studio (classique) contient les packages courants tels que NumPy, SciPy et Scikits-Learn. Ces packages peuvent être utilisés efficacement pour le traitement des données dans un pipeline d’apprentissage automatique.

Par exemple, les expériences et le script suivants illustrent l’utilisation des apprenants d’ensemble dans Scikits-Learn pour calculer les notations d’importance des fonctionnalités d’un jeu de données. Les notations peuvent servir à réaliser une sélection de fonctionnalités supervisée avant d’être alimentées dans un autre modèle.

Voici la fonction Python permettant de calculer les notations d’importance et de classer les fonctionnalités conformément à ces dernières :

![Fonction permettant de classer les fonctionnalités par notations](./media/execute-python-scripts/figure8.png)

L’expérience suivante calcule, puis retourne les notations d’importance des fonctionnalités dans le jeu de données « Diabète chez les indiens Pima » dans Azure Machine Learning Studio (classique) :

![Expérience permettant de classer des fonctionnalités dans l'ensemble de données du diabète chez les indiens Pima, en utilisant Python](./media/execute-python-scripts/figure9a.png)

![Visualisation de la sortie du module Exécuter le script Python](./media/execute-python-scripts/figure9b.png)

## <a name="limitations"></a>Limites

Le module [Exécuter le script Python][execute-python-script] présente actuellement les limites suivantes :

### <a name="sandboxed-execution"></a>Exécution dans un bac à sable (sandbox)

Le runtime Python est actuellement exécuté dans un bac à sable (sandbox) et n'autorise pas l'accès au réseau ou au système de fichiers local de façon permanente. Tous les fichiers enregistrés localement sont isolés et supprimés une fois que le module se termine. Le code Python ne peut pas accéder à la plupart des répertoires sur la machine qui les exécute, excepté le répertoire actuel et ses sous-répertoires.

### <a name="lack-of-sophisticated-development-and-debugging-support"></a>Manque de support de développement et de débogage sophistiqué

Le module Python ne prend pas en charge les fonctionnalités IDE telles que Intellisense et Débogage. En outre, si le module échoue lors de l’exécution, l’arborescence complète des appels de procédure Python est disponible. Mais elle doit être affichée dans le journal de sortie du module. Nous vous recommandons de développer et de déboguer vos scripts Python dans un environnement tel qu’IPython, puis d’importer le code dans le module.

### <a name="single-data-frame-output"></a>Sortie de trame de données unique

Le point d'entrée Python n'est autorisé qu'à renvoyer une trame de données unique en tant que sortie. Il est pour l’instant impossible de renvoyer des objets arbitraires Python tels que les modèles formés directement lors de l’exécution du runtime Studio (classique). Comme avec [Exécuter le script R][execute-r-script], qui présente les mêmes restrictions, il est souvent possible de stocker des objets dans un tableau d’octets et de les renvoyer ensuite dans une trame de données.

### <a name="inability-to-customize-python-installation"></a>Personnalisation de l'installation de Python impossible

L'ajout de modules Python personnalisés n'est, à l'heure actuelle, possible qu'à l'aide du mécanisme de fichiers zip décrit précédemment. Bien que ceci soit faisable pour les petits modules, la procédure est assez lourde pour les gros modules (notamment les modules constitués de DLL natives) ou un grand nombre de modules.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez le [Centre pour développeurs Python](https://azure.microsoft.com/develop/python/).

<!-- Module References -->
[execute-python-script]: /azure/machine-learning/studio-module-reference/execute-python-script
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script