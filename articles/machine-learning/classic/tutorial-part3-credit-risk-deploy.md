---
title: 'Tutoriel ML Studio (classique) 3 : Déployer des modèles de risque de crédit – Azure'
description: Tutoriel détaillé indiquant comment créer une solution d’analyse prédictive pour l’évaluation des risques de crédit dans Azure Machine Learning Studio (classique). Ce tutoriel est la troisième partie d’une série de tutoriels qui en compte trois. Il montre comment déployez un modèle en tant que service web.
keywords: risque de crédit, solution d’analyse prédictive, évaluation des risques, déployer, service web
author: likebupt
ms.author: keli19
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: tutorial
ms.date: 07/27/2020
ms.openlocfilehash: 3afcf87c360651ac314450910fbf5ab72afd289a
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95503904"
---
# <a name="tutorial-3-deploy-credit-risk-model---azure-machine-learning-studio-classic"></a>Tutoriel 3 : Déployer un modèle de risque de crédit - Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Cette croix (X) signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Dans ce tutoriel, vous étudiez de manière approfondie le processus de développement d’une solution d’analyse prédictive. Vous développez un modèle simple dans Machine Learning Studio (classique).  Vous déployez ensuite le modèle en tant que service web Azure Machine Learning.  Ce modèle déployé peut effectuer des prédictions à l’aide de nouvelles données. Ce tutoriel est le **troisième tutoriel d’une série qui en compte trois**.

Supposons que vous deviez prédire le risque lié à l'octroi d'un crédit à un individu sur la base des informations fournies lors d'une demande de crédit.  

L’évaluation du risque de crédit est un problème complexe, mais ce tutoriel va le simplifier un peu. Vous allez l’utiliser comme exemple de création d’une solution d’analyse prédictive à l’aide de Microsoft Azure Machine Learning Studio (classique). Vous allez utiliser Machine Learning Studio (classique) et un service web Machine Learning pour cette solution. 

Dans ce tutoriel en trois parties, vous commencez avec des données de risque crédit disponibles publiquement.  Ensuite, vous développez et entraînez un modèle prédictif.  Enfin, vous déployez le modèle en tant que service web.

Dans la [première partie du tutoriel](tutorial-part1-credit-risk.md), vous avez créé un espace de travail Machine Learning Studio (classique), chargé des données et créé une expérience.

Dans la [deuxième partie du tutoriel](tutorial-part2-credit-risk-train.md), vous avez entraîné et évalué des modèles.

Dans cette partie du tutoriel, vous allez effectuer les opérations suivantes :

> [!div class="checklist"]
> * Préparer le déploiement
> * Déploiement du service web
> * Test du service web
> * Gérer le service web
> * Accès au service web

## <a name="prerequisites"></a>Prérequis

Effectuez la [deuxième partie du tutoriel](tutorial-part2-credit-risk-train.md).

## <a name="prepare-for-deployment"></a>Préparer le déploiement
Pour que d’autres personnes puissent utiliser le modèle prédictif que vous avez développé dans ce tutoriel, vous pouvez le déployer en tant que service web sur Azure.

Jusqu’à présent, vous avez testé l’expérience de l’entraînement de votre modèle. Mais le service déployé n’effectue plus l’apprentissage ; il va produire de nouvelles prédictions en évaluant l’entrée de l’utilisateur en fonction de notre modèle. Nous allons donc effectuer quelques préparatifs pour convertir cette expérience ***d’entraînement** _ en expérience _*_prédictive_*_ 

La préparation pour le déploiement est un processus en trois étapes :  

1. Supprimer l’un des modèles
1. Convertir l’_expérience d’entraînement* que vous avez créée en *expérience prédictive*
1. Déploiement de l’expérience prédictive sous la forme d’un service web

### <a name="remove-one-of-the-models"></a>Supprimer l’un des modèles

Tout d’abord, vous devez réduire un peu cette expérience. Vous disposez actuellement de deux modèles différents dans l’expérience, mais vous ne voulez utiliser qu’un seul modèle au moment de déployer cette expérience en tant que service web.  

Supposons que vous ayez décidé que le modèle d’arbre optimisé est plus adapté que le modèle SVM. La première chose à faire est de supprimer le module [Machine à vecteurs de support à deux classes][two-class-support-vector-machine], ainsi que les modules qui ont été utilisés pour sa formation. Vous pouvez d'abord copier l'expérience en cliquant sur **Enregistrer sous** dans la partie inférieure de la zone de dessin.

Vous devez supprimer les modules suivants :  

* [Machine à vecteurs de support à deux classes][two-class-support-vector-machine]
* Modules [Former le modèle][train-model] et [Noter le modèle][score-model] qui lui étaient connectés
* [Normaliser les données][normalize-data] (les deux)
* [Évaluer le modèle][evaluate-model] (puisque nous avons terminé d’évaluer les modèles)

Sélectionnez chaque module et appuyez sur la touche Suppr, ou cliquez avec le bouton droit sur le module puis sélectionnez **Supprimer**. 

![Met en évidence les modules à supprimer pour retirer le modèle de Machine à vecteurs de support](./media/tutorial-part3-credit-risk-deploy/publish3a.png)

Notre modèle doit alors ressembler à ceci :

![Interface obtenue une fois le modèle de Machine à vecteurs de support supprimé](./media/tutorial-part3-credit-risk-deploy/publish3.png)

À présent, nous sommes prêts à déployer ce modèle avec le module [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree].

### <a name="convert-the-training-experiment-to-a-predictive-experiment"></a>Convertir l'expérience de formation en expérience prédictive

Pour préparer ce modèle pour le déploiement, vous devez convertir cette expérience d’entraînement en expérience prédictive. Cela implique trois étapes :

1. Enregistrer le modèle que vous avez entraîné, puis remplacer vos modules d’entraînement
1. Réduire l’expérience en supprimant les modules uniquement nécessaires à l’apprentissage
1. Définir où le service web doit accepter l’entrée et où il génère la sortie

Vous pourriez effectuer cette opération manuellement, mais heureusement, il est possible d’accomplir ces trois étapes en cliquant sur **Configurer le service web** en bas du canevas de l’expérience (puis en sélectionnant l’option **Service web prédictif**).

> [!TIP]
> Pour en savoir plus sur ce qui se passe quand vous convertissez une expérience de formation en une expérience de prévision, consultez [Guide pratique pour préparer votre modèle au déploiement dans Azure Machine Learning Studio (classique)](deploy-a-machine-learning-web-service.md).

Lorsque vous cliquez sur **Configurer le service web**, plusieurs choses se produisent :

* Le modèle formé est converti en un module **Modèle formé** et stocké dans la palette de modules située à gauche de la zone de dessin de l’expérience (sous **Modèles formés**).
* Les modules qui ont été utilisés pour l’apprentissage sont supprimés :
  * [Arbre de décision optimisé à deux classes][two-class-boosted-decision-tree]
  * [Former le modèle][train-model]
  * [Fractionner les données][split]
  * Deuxième module [Exécuter le script R][execute-r-script] utilisé pour les données de test
* Le modèle formé enregistré est rajouté à l’expérience.
* Les modules **Entrée du service web** et **Sortie du service web** sont ajoutés (ils identifient où les données de l’utilisateur entrent dans le modèle, ainsi que les données renvoyées lors de l’accès au service web)

> [!NOTE]
> Vous constatez que l’expérience est enregistrée en deux parties, sous les onglets ajoutés en haut de la zone de dessin. L’expérience de formation d’origine se trouve sous l’onglet **Expérience de formation**, tandis que l’expérience prédictive tout juste créée est sous **Expérience prédictive**. L’expérience prédictive est celle que vous déployez sous la forme d’un service web.

Vous devez effectuer une étape supplémentaire avec cette expérience particulière.
Vous avez ajouté deux modules [Exécuter un script R][execute-r-script] pour pondérer les données. Cette opération n’étant nécessaire que pour l’entraînement et le test, vous pouvez retirer ces modules du modèle final.
Machine Learning Studio (classique) a supprimé un module [Exécuter le script R][execute-r-script] lors de la suppression du module [Fractionner][split]. Vous pouvez à présent supprimer l’autre et connecter [Éditeur de métadonnées][metadata-editor] directement à [Noter le modèle][score-model].    

Notre expérience doit alors ressembler à cela :  

![Scoring the trained model](./media/tutorial-part3-credit-risk-deploy/publish4.png)


> [!NOTE]
> Vous vous demandez peut-être pourquoi vous avez laissé le jeu de données Données de carte de crédit allemande UCI dans l’expérience prédictive. Comme ce service va utiliser les données de l’utilisateur et non le jeu de données d’origine, pourquoi conserver ce dernier dans le modèle ?
> 
> Il est vrai que ce service n'a pas besoin des données de la carte de crédit d'origine. Mais il a besoin du schéma pour ces données, incluant des informations telles que le nombre de colonnes et lesquelles sont numériques. Ces informations sur le schéma sont indispensables pour interpréter les données de l’utilisateur. Vous laissez ces composants connectés de façon à ce que le module de notation dispose du schéma du jeu de données quand le service est en cours d’exécution. Les données ne sont pas utilisées, uniquement le schéma.  
> 
>Il est important de noter que si votre jeu de données d’origine contenait l’étiquette, alors le schéma attendu à partir de l’entrée web s’attend aussi à une colonne portant l’étiquette ! Un moyen de contourner ce problème consiste à supprimer l’étiquette et toutes les autres données dans le jeu de données d’apprentissage, mais pas dans les entrées web, avant de connecter l’entrée web et le jeu de données d’apprentissage à un module commun. 
> 

Exécutez une dernière fois l’expérience (cliquez sur **Exécuter**). Si vous voulez vérifier que le modèle fonctionne toujours, cliquez sur la sortie du module [Noter le modèle][score-model] et sélectionnez **Afficher les résultats**. Vous constatez que les données d’origine sont affichées, ainsi que la valeur du risque sur le crédit (« Étiquettes notées ») et la probabilité de la notation (« Probabilités notées »). 

## <a name="deploy-the-web-service"></a>Déploiement du service web
Vous pouvez déployer l’expérience en tant que service web classique ou nouveau service web basé sur Azure Resource Manager.

### <a name="deploy-as-a-classic-web-service"></a>Déployer comme un service web classique
Pour déployer un service web classique dérivé de notre expérience, cliquez sur **Déployer le service web** sous la zone de dessin, puis sélectionnez **Déployer le service web [Classique]** . Machine Learning Studio (classique) déploie l’expérience en tant que service web et vous amène au tableau de bord associé à ce service web. Depuis cette page, vous pouvez revenir à l’expérience (**Afficher l’instantané** ou **Afficher les données les plus récentes**) et exécuter un test simple du service web (voir **Tester le service web**, ci-dessous). Vous y trouverez également des informations sur la création d’applications pouvant accéder au service web (l’étape suivante de ce tutoriel aborde ce point plus en détail).

![Tableau de bord du service web](./media/tutorial-part3-credit-risk-deploy/publish6.png)


Vous pouvez configurer le service en cliquant sur l'onglet **CONFIGURATION** . Vous pouvez modifier le nom du service (il s'agit par défaut du nom de l'expérience) et lui attribuer une description. Vous pouvez également attribuer des étiquettes plus significatives aux données d’entrée et de sortie.  

![Configuration du service web](./media/tutorial-part3-credit-risk-deploy/publish5.png)


### <a name="deploy-as-a-new-web-service"></a>Déployer comme un nouveau service web

> [!NOTE] 
> Pour déployer un nouveau service web, vous devez disposer d’autorisations suffisantes dans l’abonnement dans lequel vous déployez le service web. Pour en savoir plus, consultez la rubrique [Gérer un service web à l’aide du portail des services web Azure Machine Learning](manage-new-webservice.md). 

Pour déployer un nouveau service web dérivé de notre expérience :

1. Cliquez sur **Déployer le service web** sous la zone de dessin, puis sélectionnez **Déployer le service web [nouveau]** . Machine Learning Studio (classique) vous redirige vers la page **Deploy Experiment (Déployer l’expérience)** des services web Azure Machine Learning.

1. Entrez le nom du service web. 

1. Dans **Price Plan (Plan de tarification)** , choisissez un plan de tarification ou sélectionnez « Créer », nommez le nouveau plan et sélectionnez l’option de plan mensuel. Les niveaux de plan s’appliquent par défaut aux plans de votre région par défaut et votre service web est déployé dans cette région.

1. Cliquez sur **Déployer**.

Après quelques minutes, la page **Démarrage rapide** de votre service web s’ouvre.

Vous pouvez configurer le service en cliquant sur l’onglet **Configurer**. Dans cette page, vous pouvez modifier le nom du service et indiquer une description. 

Pour tester le service web, cliquez sur l’onglet **Tester** (consultez **Tester le service web** ci-dessous). Pour plus d’informations sur la création d’applications pouvant accéder au service web, cliquez sur l’onglet **Consommer** (l’étape suivante de ce tutoriel aborde ce point plus en détail).

> [!TIP]
> Vous pouvez mettre à jour le service web après l’avoir déployé. Par exemple, si vous souhaitez modifier votre modèle, modifiez l’expérience de formation, ajustez les paramètres du modèle, cliquez sur **Déployer le service web**, puis sélectionnez **Déployer le service web [Classic]** ou **Déployer le service web [Nouveau]** . Lorsque vous redéployez l’expérience, le service web est remplacé par votre modèle mis à jour.  
> 
> 

## <a name="test-the-web-service"></a>Test du service web

Dans le service web, les données de l’utilisateur transitent par le module **Entrée du service web** puis par le module [Noter le modèle][score-model] où elles sont notées. Selon votre configuration de l’expérience prédictive, le modèle attend des données du même format que le jeu de données de risque de crédit d’origine.
Les résultats sont renvoyés à l’utilisateur par le service web via le module **Sortie du service web**.

> [!TIP]
> Conformément à votre configuration de l’expérience prédictive, tous les résultats du module [Noter le modèle][score-model] sont retournés. Ces résultats incluent toutes les données d’entrée, ainsi que la valeur du risque de crédit et la probabilité de la notation. Mais vous pouvez renvoyer quelque chose de différent si vous le souhaitez. Par exemple, vous pouvez retourner simplement la valeur du risque de crédit. Pour cela, insérez un module [Colonnes de projets][select-columns] entre [Calculer le score du modèle][score-model] et **Sortie du service web** pour éliminer les colonnes que le service web ne doit pas retourner. 
> 
> 

Vous pouvez tester le service web Classique dans **Machine Learning Studio (classique)** ou dans le portail des **services web Azure Machine Learning**.
Vous pouvez tester un nouveau service web uniquement dans le portail **des services web Machine Learning**.

> [!TIP]
> Lors du test dans le portail des services web Azure Machine Learning, vous pouvez demander au portail de créer un échantillon de données pour tester le service Demande-Réponse. Dans la page **Configurer**, sélectionnez « Oui » pour **Échantillon de données activé ?** . Lorsque vous ouvrez l’onglet Demande-Réponse dans la page **Tester**, le portail intègre l’échantillon provenant du jeu de données Risque de crédit.

### <a name="test-a-classic-web-service"></a>Tester un service web classique

Vous pouvez tester le service web Classique dans Machine Learning Studio (classique) ou dans le portail des services web Azure Machine Learning. 

#### <a name="test-in-machine-learning-studio-classic"></a>Test dans Machine Learning Studio (classique)

1. Sur la page **TABLEAU DE BORD** du service web, cliquez sur le bouton **Test** sous **Point de terminaison par défaut**. La boîte de dialogue qui s’affiche vous demande les données d’entrée du service. Les colonnes sont identiques à celles du jeu de données d’origine (Risque de crédit).  

1. Entrez un jeu de données, puis cliquez sur **OK**. 

#### <a name="test-in-the-machine-learning-web-services-portal"></a>Tester dans le portail des services web Machine Learning

1. Sur la page **TABLEAU DE BORD** du service web, cliquez sur le lien d’aperçu **Test** sous **Point de terminaison par défaut**. La page de test dans le portail des services web Azure Machine Learning pour le point de terminaison de service web s’ouvre en vous invitant à entrer les données du service. Les colonnes sont identiques à celles du jeu de données d’origine (Risque de crédit).

2. Cliquez sur **Tester Demande-réponse**. 

### <a name="test-a-new-web-service"></a>Tester un nouveau service web

Vous pouvez tester un nouveau service web uniquement dans le portail des services web Machine Learning.

1. Dans le portail des [services web Azure Machine Learning](https://services.azureml.net/quickstart), cliquez sur **Tester** en haut de la page. La page **Test** s’ouvre pour vous permettre d’entrer les données du service. Les champs d’entrée affichés correspondent à ceux du jeu de données d’origine (Risque de crédit). 

1. Entrez un jeu de données, puis cliquez sur **Test Request-Response**(Tester la requête-réponse).

Les résultats du test apparaissent sur le côté droit de la page, dans la colonne de sortie. 


## <a name="manage-the-web-service"></a>Gérer le service web

Une fois votre service web déployé, qu’il soit classique ou nouveau, vous pouvez le gérer dans le portail [des services web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart).

Pour surveiller les performances de votre service web :

1. Se connecter au [portail des services web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart)
1. Cliquez sur **Services web**.
1. Cliquer sur votre service web
1. Cliquez sur **Tableau de bord**.

## <a name="access-the-web-service"></a>Accès au service web

Dans l’étape précédente de ce tutoriel, vous avez déployé un service web qui utilise votre modèle prédictif de risque de crédit. Désormais, les utilisateurs peuvent lui envoyer des données et recevoir des résultats. 

Il s’agit d’un service web Azure qui peut recevoir et renvoyer des données de deux manières à l’aide d’API REST :  

* **Demande/Réponse** : l'utilisateur envoie une ou plusieurs lignes de crédit au service en utilisant le protocole HTTP et le service répond avec un ou plusieurs jeux de résultats.
* **Exécution en lots** : l'utilisateur stocke une ou plusieurs lignes de données de crédit dans le stockage d'objets blob Azure, puis envoie l'emplacement du stockage d'objets blob au service. Le service évalue toutes les lignes de données dans l'objet blob d'entrée, enregistre les résultats dans un autre objet blob et renvoie l'URL de ce conteneur. 

> [!NOTE]
> Les noms de colonnes de fonctionnalités dans Studio (classique) **respectent la casse**. Assurez-vous que les noms de colonnes dans vos données d’entrée pour l’appel du service web sont les mêmes que dans le jeu de données d’apprentissage.

Pour plus d’informations sur l’accès au service web et sur son utilisation, consultez [Utilisation d’un service web Microsoft Azure Machine Learning à l’aide d’un modèle d’application web](./consume-web-services.md).



## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [machine-learning-studio-clean-up](../../../includes/machine-learning-studio-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez effectué les étapes suivantes :

> [!div class="checklist"]
> * Préparer le déploiement
> * Déploiement du service web
> * Test du service web
> * Gérer le service web
> * Accès au service web

Vous pouvez également développer une application personnalisée pour accéder au service web à l'aide du code de démarrage fourni dans R, C# et les langages de programmation Python.

> [!div class="nextstepaction"]
> [Consommer un service web Azure Machine Learning](consume-web-services.md)

<!-- Module References -->
[evaluate-model]: /azure/machine-learning/studio-module-reference/evaluate-model
[execute-r-script]: /azure/machine-learning/studio-module-reference/execute-r-script
[metadata-editor]: /azure/machine-learning/studio-module-reference/edit-metadata
[normalize-data]: /azure/machine-learning/studio-module-reference/normalize-data
[score-model]: /azure/machine-learning/studio-module-reference/score-model
[split]: /azure/machine-learning/studio-module-reference/split-data
[train-model]: /azure/machine-learning/studio-module-reference/train-model
[two-class-boosted-decision-tree]: /azure/machine-learning/studio-module-reference/two-class-boosted-decision-tree
[two-class-support-vector-machine]: /azure/machine-learning/studio-module-reference/two-class-support-vector-machine
[select-columns]: /azure/machine-learning/studio-module-reference/select-columns-in-dataset