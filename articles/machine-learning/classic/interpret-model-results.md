---
title: 'ML Studio (classique) : Interprétation des résultats du modèle – Azure'
description: Sélection du paramètre optimal défini pour un algorithme en utilisant et visualisant des sorties de modèle de notation.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.date: 11/29/2017
ms.openlocfilehash: 74210475cc8c0efd274d6e80c3f1c89b5206a97d
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95509507"
---
# <a name="interpret-model-results-in-azure-machine-learning-studio-classic"></a>Interpréter les résultats de modèle dans Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Cette croix (X) signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Cette rubrique vous explique comment visualiser et interpréter les résultats de prédiction dans Azure Machine Learning Studio (classique). Après avoir formé un modèle et effectué des prédictions sur celui-ci (c’est-à-dire « noté le modèle »), vous avez besoin de comprendre et d’interpréter les résultats de prédiction.

Il existe quatre principaux types de modèles Machine Learning dans Azure Machine Learning Studio (classique) :

* classification ;
* Clustering
* régression ;
* systèmes de recommandation.

Les modules utilisés pour la prédiction en plus de ces modèles sont :

* [Noter le modèle][score-model] pour la classification et la régression
* [Attribuer aux clusters][assign-to-clusters] pour le clustering
* [Noter la recommandation Matchbox][score-matchbox-recommender] pour les systèmes de recommandation

Découvrez comment [choisir des paramètres pour optimiser vos algorithmes dans ML Studio (classique)](algorithm-parameters-optimize.md).

Pour savoir comment évaluer vos modèles, consultez [Comment évaluer les performances d’un modèle](evaluate-model-performance.md).

Si vous débutez avec ML Studio (classique), [découvrez comment créer une expérience simple](create-experiment.md).

## <a name="classification"></a>classification ;
Il existe deux sous-catégories de problèmes de classification :

* Problèmes avec uniquement deux classes (classification double classe ou binaire)
* Problèmes avec plus de deux classes (classification multiclasse)

Azure Machine Learning Studio (classique) propose différents modules pour traiter chacun de ces types de classification, mais les méthodes permettant d’interpréter les résultats des prédictions sont similaires.

### <a name="two-class-classification"></a>Classification double classe.
**Exemple d'expérience**

Un exemple de problème de classification double classe est la classification de fleurs d’iris. La tâche consiste à classer les fleurs d’iris en fonction de leurs caractéristiques. Le jeu de données Iris fourni dans Azure Machine Learning Studio (classique) constitue un sous-ensemble de ce fameux [Jeu de données Iris](https://en.wikipedia.org/wiki/Iris_flower_data_set) contenant des instances d’uniquement deux espèces de fleur (classes 0 et 1). Chaque fleur comporte quatre caractéristiques (la longueur des sépales, la largeur des sépales, la longueur des pétales et la largeur des pétales).

![Capture d’écran de l’expérience iris](./media/interpret-model-results/1.png)

Figure 1. Expérience d’un problème de classification double classe iris

Une expérience a été réalisée pour résoudre ce problème, comme mentionnée dans la Figure 1. Un modèle d'arbre de décision augmentée, double classe a été formé et noté. Vous pouvez dès à présent visualiser les résultats de prédiction du module [Noter le modèle][score-model] en cliquant sur le port de sortie du module [Noter le modèle][score-model], puis sur **Visualiser** dans le menu qui s’affiche.

![Module Noter le modèle](./media/interpret-model-results/1_1.png)

Ceci affiche les résultats de notation, comme mentionné dans la Figure 2.

![Résultats de l’expérience de classification double classe iris](./media/interpret-model-results/2.png)

Figure 2 : Visualisation du résultat du modèle de notation dans la classification double classe

**Interprétation du résultat**

La table des résultats est constituée de six colonnes. Les quatre colonnes de gauche représentent les quatre fonctionnalités. Les deux colonnes de droite, Étiquettes notées et Probabilités notées, représentent les résultats de prédiction. La colonne Probabilités notées indique la probabilité qu’une fleur appartienne à la classe positive (Classe 1). Par exemple, le premier numéro indiqué dans la colonne (0,028571) signifie qu’il y a une probabilité de 0,028571 que la première fleur appartienne à la Classe 1. La colonne Étiquettes notées indique la classe prédite pour chaque fleur. Elle est déterminée en fonction de la colonne Probabilités notées. Si la probabilité notée d’une fleur est supérieure à 0,5, elle est prédite en tant que Classe 1. Sinon, elle est prédite en tant que classe 0.

**Publication du service web**

Une fois que les résultats de prédiction ont été compris et considérés comme pertinents, l’expérience peut être publiée en tant que service web pour vous permettre de la déployer sur différentes applications et être sollicitée pour obtenir des prédictions de classe sur n’importe quelle nouvelle fleur d’iris. Pour découvrir comment convertir une expérience de formation en expérience de notation et la publier sous forme de service web, consultez [Tutoriel 3 : Déployer un modèle de risque de crédit](tutorial-part3-credit-risk-deploy.md). Cette procédure vous permet de bénéficier d’une expérience de notation, comme indiqué dans la Figure 3.

![Capture d’écran de l’expérience de notation](./media/interpret-model-results/3.png)

Figure 3. Notation de l’expérience d’un problème de classification double classe Iris

Vous devez à présent définir l’entrée et la sortie du service web. L’entrée est caractérisée par le port d’entrée droit de [Noter le modèle][score-model], qui représente l’entrée des fonctionnalités de la fleur d’iris. La sortie est, quant à elle, définie en fonction de l’intérêt que vous portez à la classe prédite (étiquette notée), à la probabilité notée ou bien aux deux. Dans cet exemple, on considère ici que vous êtes intéressé par les deux. Pour sélectionner les colonnes de sortie souhaitées, nous devons utiliser un module [Sélectionner des colonnes dans le jeu de données][select-columns]. Cliquez sur le module [Sélectionner des colonnes dans le jeu de données][select-columns], sur **Lancer le sélecteur de colonne** situé dans le volet droit, puis sélectionnez **Étiquettes notées** et **Probabilités notées**. Une fois le port de sortie du module [Sélectionner des colonnes dans le jeu de données][select-columns] configuré et exécuté à nouveau, nous devons être en mesure de publier l’expérience de notation en tant que service web en cliquant sur le bouton du bas, intitulé **PUBLIER LE SERVICE WEB**. L'expérience finale est semblable à celle mentionnée dans la Figure 4.

![Expérience de classification double classe iris](./media/interpret-model-results/4.png)

Figure 4. Expérience de notation finale d’un problème de classification double classe Iris

Une fois le service web exécuté et certaines valeurs caractéristiques d’une instance de test saisies, le résultat fournit deux nombres. Le premier nombre représente l’étiquette notée, le deuxième la probabilité notée. Cette fleur est prédite en tant que Classe 1 avec une probabilité de 0,9655.

![Test de l’interprétation Noter le modèle](./media/interpret-model-results/4_1.png)

![Résultats du test d’évaluation](./media/interpret-model-results/5.png)

Figure 5. Résultat du service web de classification double classe Iris

### <a name="multi-class-classification"></a>Classification multiclasse.
**Exemple d'expérience**

Lors de cette expérience, vous réalisez une tâche de reconnaissance de lettres en tant qu’exemple de classification multiclasse. Le classificateur essaye de prédire une lettre %28classe%29, compte tenu de certaines valeurs d’attribut manuscrites, extraites d’images manuscrites.

![Exemple de reconnaissance de lettres](./media/interpret-model-results/5_1.png)

Dans les données de formation, 16 caractéristiques sont extraites d’images de lettres écrites. Les 26 lettres forment nos 26 classes. La figure 6 montre une expérience qui va former un modèle de classification multiclasse de reconnaissance de lettres et établir des prédictions sur le même ensemble de fonctionnalités d’un jeu de données de test.

![Expérience de classification multiclasse de reconnaissance de lettre](./media/interpret-model-results/6.png)

Figure 6. Expérience de problème de classification multiclasse de reconnaissance de lettre

Lorsque vous visualisez les résultats du module [Noter le modèle][score-model] en cliquant sur le port de sortie du module [Noter le modèle][score-model], puis sur **Visualiser**, vous devez voir apparaître une fenêtre, comme illustré dans la Figure 7.

![Résultats Noter le modèle](./media/interpret-model-results/7.png)

Figure 7. Visualisation du résultat de modèle de notation dans la classification multiclasse

**Interprétation du résultat**

Les 16 colonnes de gauche représentent les valeurs caractéristiques de l’ensemble du test. Les colonnes intitulées Probabilités notées de classe « XX » sont similaires à la colonne Probabilités notées dans le cas d’utilisation d’une classification double classe. Elles indiquent la probabilité que l'entrée correspondante est comprise dans une certaine classe. Par exemple, pour la première entrée, il y a une probabilité de 0,003571 que ce soit un « A », une probabilité de 0,000451 que ce soit un « B », et ainsi de suite. La dernière colonne (Étiquettes notées) est identique aux Étiquettes notées dans le cas d’utilisation d’une classification double classe. Elle sélectionne la classe présentant la probabilité notée la plus élevée en tant que classe prédite de l'entrée correspondante. Par exemple, pour la première entrée, l’étiquette notée est un « F », car elle présente la probabilité la plus élevée que ce soit un « F » (0,916995).

**Publication du service web**

Vous pouvez également obtenir l’étiquette notée pour chaque entrée et la probabilité de l’étiquette notée. La logique de base consiste à trouver la probabilité la plus élevée parmi toutes les probabilités notées. Pour ce faire, vous devez utiliser le module [Exécuter le script R][execute-r-script]. Le code R est mentionné dans la Figure 8 et le résultat de l’expérience dans la Figure 9.

![Exemple de code R](./media/interpret-model-results/8.png)

Figure 8. Code R permettant d’extraire les étiquettes notées et les probabilités associées des étiquettes

![Résultat de l’expérience](./media/interpret-model-results/9.png)

Figure 9. Expérience de notation finale d’un problème de classification multiclasse de reconnaissance de lettres

Une fois le service web publié et exécuté, et certaines valeurs caractéristiques d’une instance de test saisies, le résultat renvoyé est semblable à celui mentionné dans la Figure 10. Cette lettre écrite, accompagnée de ses 16 caractéristiques extraites, est prédite pour être un « T » avec une probabilité de 0,9715.

![Test de l’interprétation du module de notation](./media/interpret-model-results/9_1.png)

![Résultat de test](./media/interpret-model-results/10.png)

Figure 10. Résultat du service web de classification multiclasse

## <a name="regression"></a>régression ;
Les problèmes de régression sont différents des problèmes de classification. Dans un problème de classification, vous essayez de prédire les classes discrètes, telles que la classe à laquelle appartient une fleur d’iris. Mais dans un problème de régression, comme indiqué dans l’exemple suivant, nous essayons de prédire une variable continue, comme le prix d’une voiture.

**Exemple d'expérience**

Utilisez la prédiction du prix d’une voiture comme exemple de régression. Vous essayez de prédire le prix d’une voiture en fonction de ses fonctionnalités comme la marque, le type de carburant, le type de voiture, le type de traction, etc. L'expérience est indiquée dans la Figure 11.

![Expérience de régression de prix des véhicules automobiles](./media/interpret-model-results/11.png)

Figure 11. Expérience de problème de régression de prix des véhicules automobiles

Visualisation du module [Noter le modèle][score-model], le résultat est semblable à celui mentionné dans la Figure 12.

![Résultats d’évaluation pour le problème de prédiction du prix des véhicules automobiles](./media/interpret-model-results/12.png)

Figure 12 : Résultats d’évaluation du problème de prédiction du prix des véhicules automobiles

**Interprétation du résultat**

Les étiquettes notées représentent la colonne de résultat dans ce résultat de notation. Les nombres constituent le prix prédit pour chaque voiture.

**Publication du service web**

Vous pouvez publier l’expérience de régression dans un service web et la solliciter pour prédire le prix d’une voiture de la même manière que dans le cas d’utilisation d’une classification double classe.

![Expérience de notation pour le problème de prédiction du prix des véhicules automobiles](./media/interpret-model-results/13.png)

Figure 13 : Expérience de notation pour un problème de prédiction du prix des véhicules automobiles

Exécution du service web, le résultat renvoyé est semblable à celui indiqué dans la Figure 14. Le prix prédit de cette voiture est de 15 085,52 $.

![Module de notation d’interprétation de test](./media/interpret-model-results/13_1.png)

![Résultats du module de notation](./media/interpret-model-results/14.png)

Figure 14. Résultat du service web d’un problème de régression du prix d’une voiture

## <a name="clustering"></a>Clustering
**Exemple d'expérience**

Utilisons le jeu de données Iris pour réaliser une expérience de clustering. Nous filtrons ici les étiquettes de classe dans le jeu de données de manière à ce qu’il ne contienne que des caractéristiques et puisse être utilisé uniquement pour le clustering. Dans ce cas d’utilisation Iris, définissez deux clusters lors du processus de formation, pour regrouper les fleurs en deux classes. L'expérience est indiquée dans la Figure 15.

![Expérience d’un problème de clustering Iris](./media/interpret-model-results/15.png)

Figure 15. Expérience d’un problème de clustering Iris

Le clustering diffère de la classification, car le jeu de données d’apprentissage ne dispose pas lui-même des étiquettes réelles. Le clustering regroupe les instances du jeu de données de formation dans des groupes distincts. Lors du processus de formation, le modèle étiquette les entrées en prenant compte des différences existantes entre leurs fonctionnalités. Le modèle formé peut être ensuite utilisé pour classifier les entrées futures. Deux parties du résultat nous intéressent au sein d'un problème de clustering. La première partie est l’étiquetage du jeu de données d’apprentissage et le second est le classement du nouveau jeu de données avec le modèle formé.

La première partie du résultat peut être visualisée en cliquant sur le port de sortie gauche de [Former le modèle de clustering][train-clustering-model], puis sur **Visualiser**. La visualisation est présentée dans la Figure 16.

![Résultat du clustering](./media/interpret-model-results/16.png)

Figure 16. Visualisation du résultat de clustering pour le jeu de données de formation

Le résultat de la deuxième partie, regroupant de nouvelles entrées avec le modèle de clustering formé, est indiqué dans la Figure 17.

![Visualiser le résultat de clustering](./media/interpret-model-results/17.png)

Figure 17. Visualisation du résultat de clustering sur un nouveau jeu de données

**Interprétation du résultat**

Bien que provenant d’étapes d’expérience différentes, les résultats des deux parties sont similaires et interprétés de la même manière. Les quatre premières colonnes représentent les caractéristiques. La dernière colonne, Affectations, représente les résultats de prédiction. Les entrées affectées du même numéro sont prédites dans le même groupe, c’est-à-dire qu’elles partagent d’une certaine façon des similitudes (cet te expérience utilise la mesure de la distance euclidienne par défaut). Étant donné que vous avez spécifié 2 comme nombre de clusters, les entrées dans Affectations sont étiquetées 0 ou 1.

**Publication du service web**

Vous pouvez publier l’expérience de clustering dans un service web et la solliciter pour des prédictions de clustering de la même manière que dans le cas d’utilisation d’une classification double classe.

![Expérience de notation d’un problème de clustering Iris](./media/interpret-model-results/18.png)

Figure 18. Expérience de notation d’un problème de clustering Iris

Une fois le service web exécuté, le résultat renvoyé est semblable à celui indiqué dans la Figure 19. Cette fleur est prédite dans le cluster 0.

![Module de notation d’interprétation de test](./media/interpret-model-results/18_1.png)

![Résultat du module de notation](./media/interpret-model-results/19.png)

Figure 19. Résultat du service web de classification double classe Iris

## <a name="recommender-system"></a>Système de recommandation
**Exemple d'expérience**

Pour les systèmes de recommandation, vous pouvez utiliser le problème de recommandation de restaurants à titre d’exemple : recommander des restaurants aux clients en fonction de leur historique de notation. Les données d'entrée sont composées de trois parties :

* Notations de restaurants attribuées par les clients
* Données caractéristiques des clients
* Données sur les caractéristiques de restaurants

Plusieurs tâches peuvent être effectuées via le module [Former la recommandation Matchbox][train-matchbox-recommender] d’Azure Machine Learning Studio (classique), à savoir :

* Prédire des notations pour un utilisateur et un élément donné
* Recommander des éléments à un utilisateur donné
* Trouver des utilisateurs associés à un utilisateur donné
* Trouver des éléments associés à un utilisateur donné

Vous pouvez choisir ce que vous voulez en sélectionnant parmi les quatre options du menu **Type de prédiction de recommandation**. Vous pouvez parcourir quatre scénarios.

![Recommandation matchbox](./media/interpret-model-results/19_1.png)

Une expérience Azure Machine Learning Studio (classique) type pour un système de recommandation est semblable à celui de la Figure 20. Pour plus d’informations sur l’utilisation de ces modules de système de recommandation, consultez les rubriques [Former la recommandation Matchbox][train-matchbox-recommender] et [Noter la recommandation Matchbox][score-matchbox-recommender].

![Expérience du système de recommandation](./media/interpret-model-results/20.png)

Figure 20. Expérience du système de recommandation

**Interprétation du résultat**

**Prédire des notations pour un utilisateur et un élément donné**

En sélectionnant **Prédiction de notation** dans le menu **Type de prédiction de recommandation**, vous demandez au système de recommandation de prédire la notation pour un utilisateur et un élément donné. La visualisation de la sortie [Noter la recommandation Matchbox][score-matchbox-recommender] est semblable à celle mentionnée dans la Figure 21.

![Visualisation du résultat de notation du système de recommandation - Prédiction de notation](./media/interpret-model-results/21.png)

Figure 21. Visualisation du résultat de notation du système de recommandation - Prédiction de notation

Les deux premières colonnes représentent les paires utilisateur-élément fournies par les données d'entrée. La troisième colonne représente la notation prédite d'un utilisateur pour un élément donné. Par exemple, dans la première ligne, le client U1048 est censé attribuer au restaurant 135 026 la note de 2.

**Recommander des éléments à un utilisateur donné**

En sélectionnant **Recommandation d’éléments** dans le menu **Type de prédiction de recommandation**, vous demandez au système de recommandation de recommander des éléments à un utilisateur donné. Le dernier paramètre à choisir dans ce scénario est la *sélection d’éléments recommandés*. L'option **À partir d'éléments notés (pour l'évaluation de modèle)** est principalement utilisée pour l'évaluation de modèle lors du processus de formation. Pour cette étape de prédiction, nous choisissons **À partir de tous les éléments**. La visualisation de la sortie [Noter la recommandation Matchbox][score-matchbox-recommender] est semblable à celle mentionnée dans la Figure 22.

![Visualisation du résultat de notation du système de recommandation - Recommandation d’éléments](./media/interpret-model-results/22.png)

Figure 22. Visualisation du résultat de notation du système de recommandation - Recommandation d’éléments

La première des six colonnes représente le nom de l’utilisateur donné pour lequel nous recommandons des éléments, fournis par les données d’entrée. Les cinq colonnes restantes représentent les éléments recommandés à l’utilisateur, classés par ordre décroissant en fonction de leur pertinence. Par exemple, dans la première ligne, le restaurant le plus recommandé au client U1048 est le restaurant 134986, suivi des restaurants 135018, 134975, 135021 et 132862.

**Trouver des utilisateurs associés à un utilisateur donné**

En sélectionnant **Utilisateurs associés** dans **Type de prédiction de recommandation**, nous demandons au système de recommandation de trouver des utilisateurs associés à un utilisateur donné. Les utilisateurs associés sont les utilisateurs qui ont des préférences similaires. Le dernier paramètre à choisir dans ce scénario est la *sélection de l’utilisateur associé*. L’option **À partir d’utilisateurs qui ont noté des éléments (pour l’évaluation de modèle)** est principalement utilisée pour l’évaluation de modèle lors du processus de formation. Pour cette étape de prédiction, choisissez **À partir de tous les utilisateurs**. La visualisation de la sortie [Noter la recommandation Matchbox][score-matchbox-recommender] est semblable à celle mentionnée dans la Figure 23.

![Visualisation du résultat de notation du système de recommandation - Utilisateurs associés](./media/interpret-model-results/23.png)

Figure 23. Visualisation du résultat de notation du système de recommandation - Utilisateurs associés

La première des six colonnes contient les ID utilisateurs donnés nécessaires pour rechercher des utilisateurs liés, comme fournis par les données d’entrée. Les cinq colonnes restantes répertorient les utilisateurs associés prédits, classés par ordre décroissant en fonction de leur pertinence. Par exemple, dans la première ligne, le client le plus pertinent pour le client U1048 est le client U1051, suivi des clients U1066, U1044, U1017 et U1072.

**Trouver des éléments associés à un élément donné**

En sélectionnant **Éléments associés** dans le menu **Type de prédiction de recommandation**, vous demandez au système de recommandation de trouver des éléments associés à un élément donné. Les éléments associés sont les éléments les plus susceptibles d'être appréciés par le même utilisateur. Le dernier paramètre à choisir dans ce scénario est la *sélection de l’article associé*. L'option **À partir d'éléments notés (pour l'évaluation de modèle)** est principalement utilisée pour l'évaluation de modèle lors du processus de formation. Pour cette étape de prédiction, nous choisissons **À partir de tous les éléments** . La visualisation de la sortie [Noter la recommandation Matchbox][score-matchbox-recommender] est semblable à celle mentionnée dans la Figure 24.

![Visualisation du résultat de notation du système de recommandation - articles associés](./media/interpret-model-results/24.png)

Figure 24. Visualisation du résultat de notation du système de recommandation - articles associés

La première des six colonnes représente le nom de l’élément donné pour les éléments associés, fournis par les données d’entrée. Les cinq colonnes restantes répertorient les éléments associés prédits, classés par ordre décroissant en fonction de leur pertinence. Par exemple, dans la première ligne, l’élément le plus pertinent pour l’élément 135026 est l’élément 135074, suivi des éléments 135035, 132875, 135055 et 134992.

**Publication du service web**

Le processus de publication de ces expériences en tant que services web permettant d'obtenir des prédictions est similaire pour chacun des quatre scénarios. Nous nous attardons ici sur le second scénario, la recommandation d’éléments à un utilisateur donné, à titre d’exemple. Vous pouvez suivre la même procédure pour les trois autres.

Enregistrement du système de recommandation formé en tant que modèle formé et filtrage des données d’entrée dans une colonne d’ID utilisateur unique (tel que cela a été demandé), vous pouvez raccorder l’expérience, comme mentionné dans la Figure 25, et la publier en tant que service web.

![Expérience de notation d’un problème de recommandation de restaurants](./media/interpret-model-results/25.png)

Figure 25. Expérience de notation d’un problème de recommandation de restaurants

Exécution du service web, le résultat renvoyé est semblable à celui indiqué dans la Figure 26. Les cinq restaurants recommandés pour l’utilisateur U1048 sont les restaurants 134986, 135018, 134975, 135021 et 132862.

![Exemple de service de système de recommandation](./media/interpret-model-results/25_1.png)

![Résultat de l’exemple d’expérience](./media/interpret-model-results/26.png)

Figure 26. Résultat du service web d’un problème de recommandation de restaurants

<!-- Module References -->
[assign-to-clusters]: /azure/machine-learning/studio-module-reference/assign-data-to-clusters
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[score-matchbox-recommender]: /azure/machine-learning/studio-module-reference/score-matchbox-recommender
[score-model]: /azure/machine-learning/studio-module-reference/score-model
[train-clustering-model]: /azure/machine-learning/studio-module-reference/train-clustering-model
[train-matchbox-recommender]: /azure/machine-learning/studio-module-reference/train-matchbox-recommender