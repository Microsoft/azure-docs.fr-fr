---
title: 'ML Studio (classique) : Démarrage rapide : Créer une expérience de science des données – Azure'
description: Ce guide de démarrage rapide sur l’apprentissage automatique vous montre comment créer une expérience de science des données simple. Nous allons prédire le prix d’une voiture à l’aide d’un algorithme de régression.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: quickstart
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/06/2019
ms.openlocfilehash: 81842fb3a9a68e818a3e8c94aa6db690cb1d94f5
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95486277"
---
# <a name="quickstart-create-your-first-data-science-experiment-in-machine-learning-studio-classic"></a>Démarrage rapide : Créer votre première expérience de science des données dans Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Cette croix (X) signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


[!INCLUDE [Designer notice](../../../includes/designer-notice.md)]

Dans ce guide de démarrage rapide, vous utilisez [Azure Machine Learning Studio (classique)](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio) pour créer une expérience d’apprentissage automatique qui prédit le prix d’une voiture d’après plusieurs variables comme la marque et les caractéristiques techniques.

Si vous ne connaissez pas bien l’apprentissage automatique, la série de vidéos [Science des données pour les débutants](data-science-for-beginners-the-5-questions-data-science-answers.md) offre une excellente introduction à l’apprentissage automatique, en présentant les termes et les concepts courants.

Ce guide de démarrage rapide suit le workflow par défaut d’une expérience :

1. **Création d’un modèle**
    - [Obtention des données]
    - [Préparation des données]
    - [Définition des fonctionnalités]
1. **Formation du modèle**
    - [Sélection et application d’un algorithme]
1. **Notation et test du modèle**
    - [Prédiction des nouveaux prix des voitures]

[Obtention des données]: #get-the-data
[Préparation des données]: #prepare-the-data
[Définition des fonctionnalités]: #define-features
[Sélection et application d’un algorithme]: #choose-and-apply-an-algorithm
[Prédiction des nouveaux prix des voitures]: #predict-new-automobile-prices

## <a name="get-the-data"></a>Obtenir les données

Les données sont la première chose dont vous avez besoin pour l’apprentissage automatique.
Vous pouvez utiliser plusieurs exemples de jeux de données inclus dans Studio (classique) ou importer d’autres données à partir de sources diverses. Pour les besoins de cet exemple, nous allons utiliser le jeu de données **Données sur le prix des véhicules automobiles (brutes)** inclus dans votre espace de travail.
Ce jeu de données comprend des entrées relatives à plusieurs véhicules, notamment des informations sur la marque, le modèle, les caractéristiques techniques et le prix.

> [!TIP]
> Vous pouvez obtenir une copie de travail de l’expérience suivante dans la [galerie Azure AI](https://gallery.azure.ai). Accédez à **[Votre première expérience de science des données : prédiction sur les prix automobiles](https://gallery.azure.ai/Experiment/Your-first-data-science-experiment-Automobile-price-prediction-1)** et cliquez sur **Ouvrir dans Studio** pour télécharger une copie de l’expérience dans votre espace de travail Machine Learning Studio (classique).

Voici comment obtenir ce jeu de données dans le cadre de votre expérience.

1. Créez une expérience en cliquant sur **+ NOUVEAU** en bas de la fenêtre Machine Learning Studio (classique). Sélectionnez **EXPÉRIENCE** >  **Expérience vide**.

1. Un nom par défaut est attribué à l’expérience : il apparaît en haut du canevas. Sélectionnez le texte et remplacez-le par un nom plus significatif, par exemple **Prédiction sur les prix automobiles**. Le nom n’a pas besoin d’être unique.

    ![Renommer l’expérience](./media/create-experiment/rename-experiment.png)

1. Sur la gauche de la zone de dessin de l’expérience se trouve une palette de jeux de données et de modules. Tapez la valeur **automobile** dans la zone de recherche se trouvant en haut de cette palette, afin de rechercher le jeu de données **Données sur le prix des véhicules automobiles (brutes)** . Faites glisser ce jeu de données vers le canevas de l’expérience.

    ![Recherchez le jeu de données d’automobile et faites-le glisser vers le canevas de l’expérience](./media/create-experiment/type-automobile.png)

Pour voir à quoi ressemblent ces données, cliquez sur le port de sortie situé en bas du jeu de données d’automobile, puis sélectionnez **Visualiser**.

![Cliquez sur le port de sortie et sélectionnez « Visualiser »](./media/create-experiment/select-visualize.png)

> [!TIP]
> Les jeux de données et les modules disposent de ports d’entrée et de sortie représentés par de petits cercles : les ports d’entrée se situent en haut, tandis que les ports de sortie se situent en bas.
Pour créer un flux de données dans votre expérience, connectez le port de sortie d’un module au port d’entrée d’un autre module.
À tout moment, vous pouvez cliquer sur le port de sortie d’un jeu de données ou d’un module afin de voir à quoi les données ressemblent à ce stade dans le flux de données.

Dans ce jeu de données, chaque ligne représente un véhicule automobile et chaque colonne représente une variable associée au véhicule automobile. Nous allons prédire le prix dans la colonne la plus à droite (colonne 26, intitulée « price ») en fonction des autres variables associées à chaque véhicule automobile.

![Affichez les données automobiles dans la fenêtre de visualisation des données](./media/create-experiment/visualize-auto-data.png)

Fermez la fenêtre de visualisation en cliquant sur le symbole «**x**» dans le coin supérieur droit.

## <a name="prepare-the-data"></a>Préparer les données

Pour pouvoir être analysé, un jeu de données nécessite généralement un traitement préalable. Vous avez peut-être remarqué l'absence de certaines valeurs dans les colonnes des différentes lignes. Pour que vous puissiez analyser les données correctement, ces valeurs manquantes doivent être nettoyées. Nous supprimerons toutes les lignes dans lesquelles des valeurs sont manquantes. De plus, la colonne **normalized-losses** contient une grande quantité de valeurs manquantes. Nous allons donc l’exclure du modèle.

> [!TIP]
> Le nettoyage des valeurs manquantes des données d’entrée est un prérequis pour l’utilisation de la plupart des modules.

Nous commençons par ajouter un module qui supprime toute la colonne **normalized-losses**. Nous ajoutons ensuite un module qui supprime toutes les lignes où il manque des données.

1. Dans la zone de recherche située en haut de la palette de modules, entrez la chaîne **sélectionner des colonnes** afin de rechercher le module [Sélectionner des colonnes dans le jeu de données][select-columns]. Faites ensuite glisser ce module vers le canevas de l’expérience. Ce module permet de sélectionner les colonnes de données à inclure ou exclure du modèle.

1. Connectez le port de sortie du jeu de données **Données sur le prix des véhicules automobiles (brutes)** au port d’entrée de Sélectionner des colonnes dans le jeu de données.

    ![Ajoutez le module « Sélectionner des colonnes dans le jeu de données » dans le canevas de l’expérience et connectez-le](./media/create-experiment/type-select-columns.png)

1. Cliquez sur le module [Sélectionner des colonnes dans le jeu de données][select-columns], puis cliquez sur **Lancer le sélecteur de colonne** dans le volet **Propriétés**.

   - Sur la gauche, cliquez sur **With rules**
   - Sous **Commencer par**, cliquez sur **Toutes les colonnes**. Ces règles indiquent au module [Sélectionner des colonnes dans le jeu de données][select-columns] de transmettre toutes les colonnes, sauf celles que nous nous apprêtons à exclure.
   - Dans les listes déroulantes, sélectionnez **Exclure** et **Noms des colonnes**, puis cliquez dans la zone de texte. Une liste de colonnes s’affiche. Sélectionnez la colonne **normalized-losses**, qui est alors ajoutée à la zone de texte.
   - Cliquez sur le bouton en forme de coche (OK) pour fermer le sélecteur de colonne (en bas à droite).

     ![Lancez le sélecteur de colonne et excluez la colonne « normalized-losses »](./media/create-experiment/launch-column-selector.png)

     À présent, le volet de propriétés du module **Sélectionner des colonnes dans le jeu de données** indique qu’il transmettra toutes les colonnes du jeu de données, à l’exception de **normalized-losses**.

     ![Le volet Propriétés indique que la colonne « normalized-losses » est exclue](./media/create-experiment/showing-excluded-column.png)

     > [!TIP] 
     > Vous pouvez ajouter un commentaire dans un module en double-cliquant sur ce module, puis en saisissant du texte. Ceci peut vous aider à voir d'un seul coup d'œil ce que fait chaque module dans votre expérience. Dans ce cas, double-cliquez sur le module [Sélectionner des colonnes dans le jeu de données][select-columns], puis entrez le commentaire suivant : « Exclure les pertes normalisées ».

     ![Double-cliquez sur un module pour ajouter un commentaire](./media/create-experiment/add-comment.png)

1. Faites glisser le module [Nettoyer les données manquantes][clean-missing-data] jusqu’à la zone de dessin de l’expérience et connectez-le au module [Sélectionner des colonnes dans le jeu de données][select-columns]. Dans le volet **Propriétés**, sélectionnez **Supprimer toute la ligne** sous **Mode de nettoyage**. Ces options indiquent au module [Clean Missing Data][clean-missing-data] de nettoyer les données en supprimant les lignes dans lesquelles il manque des valeurs. Double-cliquez sur le module et saisissez le commentaire suivant : « Supprimer les lignes de valeur manquantes ».

    ![Définissez le mode de nettoyage du module « Nettoyage des données manquantes » sur « Supprimer toute la ligne »](./media/create-experiment/set-remove-entire-row.png)

1. Exécutez l’expérience en cliquant sur **EXÉCUTER** au bas de la page.

    Une fois l’expérience terminée, une coche verte s’affiche en regard de chaque module pour indiquer la réussite de leurs opérations. Notez que le statut **Exécution terminée** s’affiche dans le coin supérieur droit de la fenêtre.

    ![Une fois l’exécution terminée, l’expérience doit ressembler à ceci](./media/create-experiment/early-experiment-run.png)

> [!TIP]
> Pourquoi exécutons-nous l’expérience maintenant ? Lors de l’exécution de l’expérience, les définitions de colonne de nos données sont transmises à partir du jeu de données, par le biais des modules [Sélectionner des colonnes dans le jeu de données][select-columns] et [Nettoyage des données manquantes][clean-missing-data]. Autrement dit, tous les modules que nous connectons à [Nettoyage des données manquantes][clean-missing-data] disposent de ces mêmes informations.

Nous disposons maintenant de données nettoyées. Si vous souhaitez afficher le jeu de données nettoyé, cliquez sur le port de sortie gauche du module [Nettoyage des données manquantes][clean-missing-data] et sélectionnez **Visualiser**. Vous pouvez constater que la colonne **normalized-losses** n’est plus là et qu’il ne manque plus de données.

Maintenant que les données sont nettoyées, nous pouvons indiquer les fonctionnalités que nous allons utiliser dans le modèle de prévision.

## <a name="define-features"></a>Définition des fonctionnalités

Dans le domaine du Machine Learning, les *caractéristiques* sont les propriétés individuelles mesurables d’un élément qui vous intéresse. Dans notre jeu de données, chaque ligne représente un véhicule et chaque colonne une fonctionnalité de ce véhicule.

La recherche du jeu de fonctionnalités adéquat pour la création d’un modèle de prévision requiert certaines expériences et des connaissances sur le problème qui se pose. Certaines fonctionnalités sont mieux adaptées à la prévision que d’autres. Certaines fonctionnalités ont une forte corrélation avec d’autres fonctionnalités et peuvent être supprimées. Par exemple, city-mpg et highway-mpg sont étroitement liées : nous pouvons donc conserver l’une et supprimer l’autre sans trop affecter la prédiction.

Nous allons développer un modèle utilisant un sous-ensemble de ces fonctionnalités pour notre jeu de données. Vous pouvez revenir en arrière et sélectionner d’autres fonctionnalités, relancer l’expérience et voir si vous obtenez de meilleurs résultats. Mais pour commencer, nous allons essayer les fonctionnalités suivantes :

> make, body-style, wheel-base, engine-size, horsepower, peak-rpm, highway-mpg, price

1. Faites glisser un autre module [Sélectionner des colonnes dans le jeu de données][select-columns] vers le canevas de l’expérience. Connectez le port de sortie de gauche du module [Nettoyage des données manquantes][clean-missing-data] à l’entrée du module [Sélectionner des colonnes dans le jeu de données][select-columns].

    ![Connectez le module « Sélectionner des colonnes dans le jeu de données » au module « Nettoyage des données manquantes »](./media/create-experiment/connect-clean-to-select.png)

1. Double-cliquez sur le module et saisissez le commentaire suivant : « Sélection des fonctionnalités pour la prévision ».

1. Cliquez sur l’option **Lancer le sélecteur de colonne** figurant dans le volet **Propriétés**.

1. Cliquez sur **With rules**(à l’aide de règles).

1. Sous **Commencer par**, cliquez sur **Aucune colonne**. Dans la ligne de filtre, sélectionnez **Inclure** et **Noms des colonnes**, puis sélectionnez notre liste de noms de colonnes dans la zone de texte. Ce filtre indique au module de transmettre uniquement les colonnes (fonctionnalités) sélectionnées.

1. Cliquez sur le bouton en forme de coche (OK) pour continuer.

    ![Sélectionnez les colonnes (fonctionnalités) à inclure dans la prédiction](./media/create-experiment/select-columns-to-include.png)

Ce module produit un jeu de données filtré qui contient uniquement les fonctionnalités que nous souhaitons transmettre à l’algorithme d’apprentissage utilisé à l’étape suivante. Plus tard, vous pouvez reprendre la procédure en utilisant une autre sélection de fonctionnalités.

## <a name="choose-and-apply-an-algorithm"></a>Sélection et application d’un algorithme

À présent que les données sont prêtes, la construction d'un modèle de prévision passe par la formation et le test. Nous allons utiliser nos données pour former le modèle, puis tester le modèle pour voir dans quelle mesure il peut prédire les prix.
<!-- For now, don't worry about *why* we need to train and then test a model.-->

La *classification* et la *régression* sont deux types d’algorithmes de machine learning supervisé. La classification permet de prédire une réponse à partir d'un jeu de catégories défini, comme une couleur (rouge, bleu ou vert). La régression est utilisée pour prédire un nombre.

Étant donné que nous voulons prédire un prix, correspondant à un nombre, nous allons utiliser un algorithme de régression. Dans cet exemple, nous utilisons un modèle de *régression linéaire*.


Nous formons le modèle en lui fournissant un jeu de données qui inclut le prix. Le modèle analyse les données et recherche les corrélations entre les fonctionnalités d’un véhicule automobile et son prix. Puis nous testons le modèle : nous lui affectons un ensemble de fonctionnalités pour véhicules automobiles que nous connaissons et nous étudions la précision du modèle concernant la prédiction des prix.

Nous allons utiliser nos données pour la formation et le test en les divisant en jeux de données distincts de formation et de test.

1. Sélectionnez et faites glisser le module [Fractionner les données][split] sur le canevas d’expérience et connectez-le au dernier module [Sélectionner des colonnes dans le jeu de données][select-columns].

1. Cliquez sur le module [Fractionner les données][split] pour le sélectionner. Rechercher **Fraction de lignes dans le premier jeu de données de sortie** (dans le volet **Propriétés** à droite du canevas) et attribuez-lui la valeur 0,75. Ainsi, nous allons utiliser 75 % des données pour former le modèle, et 25 % pour le tester.

    ![Définissez la part de fractionnement du module « Fractionner les données » sur 0,75](./media/create-experiment/set-split-data-percentage.png)

    > [!TIP]
    > En modifiant le paramètre **Valeur de départ aléatoire** , vous pouvez produire différents échantillons aléatoires pour la formation et le test. Ce paramètre contrôle la valeur de départ du générateur de nombres pseudo-aléatoire.

1. Exécutez l’expérience. Lors de l’expérience, les modules [Sélectionner des colonnes dans le jeu de données][select-columns] et [Fractionner les données][split] transmettent des définitions de colonne aux modules que nous allons ajouter par la suite.  

1. Pour sélectionner l’algorithme d’apprentissage, développez la catégorie **Machine Learning** dans la palette des modules, à gauche de la zone de dessin, puis développez **Initialiser le modèle**. Différentes catégories de modules s'affichent, permettant d'initialiser des algorithmes d'apprentissage automatique. Pour les besoins de cet exemple, sélectionnez le module [Régression linéaire][linear-regression] sous la catégorie **Régression**, puis faites-le glisser vers le canevas de l’expérience. Vous pouvez également rechercher le module en tapant « régression linéaire » dans la zone de recherche de la palette.

1. Recherchez et faites glisser le module [Entraîner le modèle][train-model] jusqu’à la zone de dessin. Connectez la sortie du module [Régression linéaire][linear-regression] à l’entrée de gauche du module [Entraîner votre modèle][train-model], puis connectez la sortie des données d’entraînement (port gauche) du module [Fractionner les données][split] à l’entrée de droite du module [Entraîner votre modèle][train-model].

    ![Connectez le module « Former le modèle » aux modules « Régression linéaire » et « Fractionner les données »](./media/create-experiment/connect-train-model.png)

1. Cliquez sur le module [Entraîner votre modèle][train-model], cliquez sur l’option **Lancer le sélecteur de colonne** dans le volet **Propriétés**, puis sélectionnez la colonne **Price**. **Price** est la valeur à prédire par notre modèle.

    Vous pouvez sélectionner la colonne **price** dans le sélecteur de colonne en la faisant passer de la liste **Colonnes disponibles** à la liste **Colonnes sélectionnées**.

    ![Sélectionnez la colonne price pour le module « Former le modèle »](./media/create-experiment/select-price-column.png)

1. Exécutez l’expérience.

Nous disposons à présent d’un modèle de régression formé qui permet de noter de nouvelles données automobiles pour effectuer des prédictions de prix.

![Une fois l’exécution terminée, l’expérience doit ressembler à ceci](./media/create-experiment/second-experiment-run.png)

## <a name="predict-new-automobile-prices"></a>Prédiction des nouveaux prix des voitures

À présent que nous avons formé le modèle à l'aide de 75 % de nos données, nous pouvons l'utiliser pour la notation du reste de nos données (25 %), afin de voir s'il fonctionne correctement.

1. Recherchez et faites glisser le module [Noter le modèle][score-model] vers le canevas de l’expérience. Connectez la sortie du module [Entraîner le modèle][train-model] au port d’entrée de gauche du module [Scorer le modèle][score-model]. Connectez la sortie des données de test (port de droite) du module [Fractionner les données][split] au port d’entrée de droite du module [Scorer le modèle][score-model].

    ![Connectez le module « Noter le modèle » aux modules « Former le modèle » et « Fractionner les données »](./media/create-experiment/connect-score-model.png)

1. Exécutez l’expérience et affichez la sortie du module [Noter le modèle][score-model] en cliquant sur le port de sortie de [Noter le modèle][score-model], puis sélectionnez **Visualiser**. La sortie affiche les valeurs de prévision associées au prix, ainsi que les valeurs connues des données de test.  

    ![Sortie du module « Noter le modèle »](./media/create-experiment/score-model-output.png)

1. Enfin, nous testons la qualité des résultats. Sélectionnez et faites glisser le module [Évaluer le modèle][evaluate-model] vers le canevas de l’expérience, puis connectez la sortie du module [Noter le modèle][score-model] à l’entrée de gauche du module [Évaluer le modèle][evaluate-model]. L’expérience finale doit ressembler à ceci :

    ![Expérience finale](./media/create-experiment/complete-linear-regression-experiment.png)

1. Exécutez l’expérience.

Pour afficher la sortie du module [Évaluer le modèle][evaluate-model], cliquez sur le port de sortie, puis sélectionnez **Visualiser**.

![Résultats de l’évaluation de l’expérience](./media/create-experiment/evaluation-results.png)

Les statistiques suivantes s’affichent pour notre modèle :

- **Erreur Absolue Moyenne** (EAM) la moyenne des erreurs absolues (une *erreur* correspond à la différence entre la valeur prévue et la valeur réelle).
- **Racine de l’erreur quadratique moyenne** (RMSE) : la racine carrée de la moyenne des erreurs carrées des prévisions effectuées sur le jeu de données de test.
- **Erreur absolue relative**: la moyenne des erreurs absolues relative à la différence absolue entre les valeurs réelles et la moyenne de toutes les valeurs réelles.
- **Erreur carrée relative** : la moyenne des erreurs carrées relative à la différence carrée entre les valeurs réelles et la moyenne de toutes les valeurs réelles.
- **Coefficient de détermination** : aussi nommé **valeur R au carré**, il s’agit d’une mesure statistique indiquant à quel point un modèle correspond aux données.

Pour chacune des statistiques liées aux erreurs, les valeurs les plus petites sont privilégiées. En effet, une valeur plus petite indique un degré de correspondance plus étroit avec la valeur réelle. Plus la valeur du **Coefficient de détermination**, est proche de un (1.0), plus la prévision est correcte.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [machine-learning-studio-clean-up](../../../includes/machine-learning-studio-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une expérience simple à partir d’un exemple de jeu de données. Pour approfondir le processus de création et de déploiement d’un modèle, passez au tutoriel sur les solutions prédictives.

> [!div class="nextstepaction"]
> [Tutoriel : Développer une solution prédictive dans Studio (classique)](tutorial-part1-credit-risk.md)

<!-- Module References -->
[evaluate-model]: /azure/machine-learning/studio-module-reference/evaluate-model
[linear-regression]: /azure/machine-learning/studio-module-reference/linear-regression
[clean-missing-data]: /azure/machine-learning/studio-module-reference/clean-missing-data
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset
[score-model]: /azure/machine-learning/studio-module-reference/score-model
[split]: /azure/machine-learning/studio-module-reference/split-data
[train-model]: /azure/machine-learning/studio-module-reference/train-model