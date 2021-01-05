---
title: 'Tutoriel : Prévision de la demande et AutoML'
titleSuffix: Azure Machine Learning
description: Découvrez comment entraîner et déployer un modèle de prévision de la demande avec le Machine Learning automatisé dans Azure Machine Learning Studio.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: sacartac
ms.reviewer: nibaccam
author: cartacioS
ms.date: 07/10/2020
ms.custom: automl
ms.openlocfilehash: 8b354abb98c56a572badf2421b0d7dbbd25f7a63
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921856"
---
# <a name="tutorial-forecast-demand-with-automated-machine-learning"></a>Tutoriel : Prévoir la demande avec le Machine Learning automatisé


Dans ce tutoriel, vous allez utiliser le Machine Learning automatisé, ou ML automatisé, dans Azure Machine Learning Studio afin de créer un modèle de prévision de série chronologique qui prédit la demande de location pour un service de vélos en libre-service.

Pour obtenir un exemple de modèle de classification, consultez [Tutoriel : Créer un modèle de classification avec le ML automatisé dans Azure Machine Learning](tutorial-first-experiment-automated-ml.md).

Dans ce tutoriel, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer et charger un jeu de données
> * Configurer et exécuter une expérience de ML automatisé
> * Indiquez les paramètres de prévision.
> * Explorer les résultats de l’expérience
> * Déployer le meilleur modèle

## <a name="prerequisites"></a>Prérequis

* Un espace de travail Azure Machine Learning. Consultez [Créer un espace de travail Microsoft Azure Machine Learning](how-to-manage-workspace.md). 

* Télécharger le fichier de données [bike-no.csv](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/bike-no.csv)

## <a name="get-started-in-azure-machine-learning-studio"></a>Bien démarrer dans Azure Machine Learning Studio

Pour ce tutoriel, vous allez créer l’exécution de votre expérience de ML automatisé dans Azure Machine Learning Studio, une interface web centralisée qui comprend des outils de machine learning permettant de mettre en œuvre des scénarios de science des données pour des utilisateurs de science des données de tous niveaux de compétence. Studio n’est pas prise en charge par les navigateurs Internet Explorer.

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com).

1. Sélectionnez votre abonnement et l’espace de travail que vous avez créé.

1. Sélectionnez **Prise en main**.

1. Dans le volet gauche, sélectionnez **Automated ML** (ML automatisé) sous la section **Author** (Créer).

1. Sélectionnez **+Nouvelle exécution de ML automatisé**. 

## <a name="create-and-load-dataset"></a>Créer et charger un jeu de données

Avant de configurer votre expérience, chargez votre fichier de données dans votre espace de travail sous la forme d’un jeu de données Azure Machine Learning. De cette façon, vous pouvez vérifier que la mise en forme de vos données convient à votre expérience.

1. Dans le formulaire **Sélectionner un jeu de données**, sélectionnez **Des fichiers locaux** dans la liste déroulante **+Créer un jeu de données**. 

    1. Dans le formulaire **Informations de base**, donnez un nom unique à votre jeu de données et indiquez éventuellement une description. Le type de jeu de données par défaut est **Tabulaire** dans la mesure où le ML automatisé dans Azure Machine Learning Studio ne prend actuellement en charge que les jeux de données tabulaires.
    
    1. Sélectionnez **Suivant** en bas à gauche.

    1. Dans le formulaire **Sélection d’un magasin de données et de fichiers**, sélectionnez le magasin de données par défaut qui a été automatiquement configuré durant la création de votre espace de travail : **workspaceblobstore (Stockage Blob Azure)** . Il s’agit de l’emplacement de stockage où vous chargerez votre fichier de données. 

    1. Sélectionnez **Parcourir**. 
    
    1. Choisissez le fichier **bike-no.csv** sur votre ordinateur local. Il s’agit du fichier que vous avez téléchargé en tant que [prérequis](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-bike-share/bike-no.csv).

    1. Sélectionnez **Suivant**.

       Une fois le chargement terminé, le formulaire Settings and preview (Paramètres et aperçu) est prérenseigné en fonction du type de fichier. 
       
    1. Vérifiez que le formulaire **Settings and preview** est renseigné comme ci-dessous, puis sélectionnez **Next** (Suivant).
        
        Champ|Description| Valeur pour le tutoriel
        ---|---|---
        Format de fichier|Définit la disposition et le type des données stockées dans un fichier.| Delimited
        Délimiteur|Un ou plusieurs caractères utilisés pour spécifier la limite entre&nbsp; des régions distinctes et indépendantes dans du texte brut ou d’autres flux de données. |Comma
        Encodage|Identifie la table de schéma bits/caractères à utiliser pour lire votre jeu de données.| UTF-8
        En-têtes de colonne| Indique la façon dont les éventuels en-têtes du jeu de données sont traités.| Utiliser les en-têtes du premier fichier
        Ignorer les lignes | Indique le nombre éventuel de lignes ignorées dans le jeu de données.| None

    1. Le formulaire **Schema** permet de configurer davantage vos données pour cette expérience. 
    
        1. Pour cet exemple, choisissez d’ignorer les colonnes **casual** et **registered**. Ces colonnes étant une décomposition de la colonne **cnt**, nous ne les incluons pas.

        1. Dans cet exemple, conservez également les valeurs par défaut pour **Propriétés** et **Type**. 
        
        1. Sélectionnez **Suivant**.

    1. Dans le formulaire **Confirmer les détails**, vérifiez que les informations correspondent à celles qui ont été précédemment renseignées sur les formulaires **Informations de base** et **Paramètres et aperçu**.

    1. Sélectionnez **Créer** pour terminer la création de votre jeu de données.

    1. Sélectionnez votre jeu de données une fois qu’il apparaît dans la liste.

    1. Sélectionnez **Suivant**.

## <a name="configure-experiment-run"></a>Configurer l’exécution de l’expérience

Une fois vos données chargées et configurées, configurez votre cible de calcul à distance et sélectionnez dans vos données la colonne que vous souhaitez prédire.

1. Remplissez le formulaire **Configurer l’exécution** comme suit :
    1. Entrez un nom d’expérience : `automl-bikeshare`

    1. Sélectionnez **cnt** comme colonne cible, ce que vous souhaitez prédire. Cette colonne indique le nombre total de locations de vélos en libre-service.

    1. Sélectionnez **Create a new compute** (Créer un nouveau calcul), puis configurez la cible de calcul. Le ML automatisé prend uniquement en charge la capacité de calcul Azure Machine Learning. 

        Champ | Description | Valeur pour le tutoriel
        ----|---|---
        Nom du calcul |Nom unique qui identifie votre contexte de calcul.|bike-compute
        Type de&nbsp;machine&nbsp;virtuelle|Sélectionnez le type de machine virtuelle pour votre calcul.|Processeur (CPU)
        Taille de la&nbsp;machine&nbsp;virtuelle| Sélectionnez la taille de la machine virtuelle pour votre calcul.|Standard_DS12_V2
        Nombre minimal/maximal de nœuds| Pour profiler des données, vous devez spécifier un ou plusieurs nœuds.|Nœuds min. : 1<br>Nœuds max. : 6
        Secondes d’inactivité avant le scale-down | Durée d’inactivité avant que le cluster ne fasse l’objet d’un scale-down au nombre de nœuds minimal.|120 (par défaut)
        Paramètres avancés | Paramètres pour configurer et autoriser un réseau virtuel pour votre expérience.| None
  
        1. Sélectionnez **Créer** pour accéder à la cible de calcul. 

            **Quelques minutes sont nécessaires pour achever l’opération**. 

        1. Une fois la création terminée, sélectionnez votre nouvelle cible de calcul dans la liste déroulante.

    1. Sélectionnez **Suivant**.

## <a name="select-forecast-settings"></a>Sélectionner les paramètres de prévision

Terminez la configuration de votre expérience de ML automatisé en spécifiant le type de tâche de Machine Learning et les paramètres de configuration.

1. Dans le formulaire **Type de tâche et paramètres**, sélectionnez **Prévisions de série chronologique** comme type de tâche de Machine Learning.

1. Sélectionnez **date** comme **Colonne Heure** et laissez **Identificateurs de séries chronologiques** vide. 

1. L’**horizon de prévision** est la période dans le futur pour laquelle vous voulez faire des prédictions.  Décochez Détection automatique et tapez 14 dans le champ. 

1. Sélectionnez **Afficher des paramètres de configuration supplémentaires** et renseignez les champs comme suit. Ces paramètres permettent de mieux contrôler le travail d’entraînement, et de spécifier les paramètres de votre prévision. Sinon, les valeurs par défaut sont appliquées en fonction de la sélection de l’expérience et des données.

    Configurations&nbsp;supplémentaires|Description|Valeur&nbsp;pour&nbsp;le tutoriel
    ------|---------|---
    Métrique principale| Métrique d’évaluation selon laquelle l’algorithme de Machine Learning sera mesuré.|Erreur quadratique moyenne normalisée
    Expliquer le meilleur modèle| Montre automatiquement l’explicabilité sur le meilleur modèle créé par le ML automatisé.| Activer
    Algorithmes bloqués | Algorithmes que vous souhaitez exclure du travail de formation| Arbres aléatoires extrêmes
    Paramètres de prévision supplémentaires| Ces paramètres contribuent à améliorer la justesse de votre modèle. <br><br> _**Décalages de cibles de prévision**_  : antériorité avec laquelle vous voulez construire les décalages d’une variable cible <br> _**Fenêtre dynamique cible**_  : spécifie la taille de la fenêtre dynamique sur laquelle des caractéristiques telles que *max, min* et *somme* seront générées. | <br><br>Décalages&nbsp;de cibles de&nbsp;prévision : None <br> Taille&nbsp;de la&nbsp;fenêtre dynamique&nbsp;cible : None
    Critère de sortie| Lorsqu’une condition est remplie, la tâche d’entraînement est arrêtée. |Durée&nbsp;de la tâche&nbsp;de formation : 3 <br> Seuil&nbsp;de score&nbsp;de métrique : None
    Validation | Choisissez un type de validation croisée et un nombre de tests.|Type de validation :<br>&nbsp;validation croisée&nbsp;k-fold <br> <br> Nombre de validations : 5
    Accès concurrentiel| Nombre maximal d’itérations parallèles exécutées par itération| Nombre maximal&nbsp;d’itérations&nbsp;simultanées : 6
    
    Sélectionnez **Enregistrer**.

## <a name="run-experiment"></a>Exécuter une expérience

Pour exécuter votre expérience, sélectionnez **Terminer**. L’écran **Détails de l’exécution** s’ouvre avec **État de l’exécution**  en haut à côté du numéro de l’exécution. Cet état est mis à jour à mesure que l’expérience progresse.

>[!IMPORTANT]
> La préparation nécessaire à l’exécution de l’expérience prend **10 à 15** minutes.
> Une fois que l’exécution a commencé, **2-3 minutes supplémentaires sont nécessaires pour chaque itération**.  <br> <br>
> Dans un environnement de production, ce processus prend du temps. Pendant que vous attendez, nous vous suggérons de commencer à explorer les algorithmes testés sous l’onglet **Modèles** à mesure qu’ils se terminent. 

##  <a name="explore-models"></a>Explorer les modèles

Accédez à l’onglet **Modèles** pour voir les algorithmes (modèles) testés. Par défaut, les modèles sont classés par score de métrique à mesure qu’ils se terminent. Pour ce tutoriel, le modèle qui obtient le score le plus élevé d’après la métrique **Erreur quadratique moyenne normalisée** choisie figure en haut de la liste.

En attendant que toutes les modèles d’expérience se terminent, sélectionnez le **Nom de l’algorithme** d’un modèle terminé pour explorer ses performances en détail. 

L’exemple suivant parcourt les onglets **Détails** et **Métriques** pour montrer les propriétés, les métriques et les graphiques de performances du modèle sélectionné. 

![Détails de l’exécution](./media/tutorial-automated-ml-forecast/explore-models-ui.gif)

## <a name="deploy-the-model"></a>Déployer le modèle

Dans Azure Machine Learning Studio, le Machine Learning automatisé vous permet de déployer le meilleur modèle en tant que service web en quelques étapes. Le déploiement consiste à intégrer le modèle pour qu’il puisse prédire de nouvelles données et identifier les domaines potentiels d’opportunités. 

Pour cette expérience, le déploiement sur un service web signifie que la société de vélos en libre-service dispose désormais d’une solution web itérative et scalable pour prévoir la demande de location. 

Une fois l’exécution terminée, revenez à la page d’exécution du parent en sélectionnant **Exécution 1** en haut de votre écran.

Dans la section **Récapitulatif du meilleur modèle**, **StackEnsemble** est considéré comme étant le meilleur modèle dans le contexte de cette expérience selon la métrique **Erreur quadratique moyenne normalisée**.  

Nous déployons ce modèle, mais nous vous informons que le déploiement prend 20 minutes environ. Le processus de déploiement comporte plusieurs étapes, notamment l’inscription du modèle, la génération de ressources et leur configuration pour le service web.

1. Sélectionnez **StackEnsemble** pour ouvrir la page spécifique au modèle.

1. Sélectionnez le bouton **Déployer** situé dans la partie supérieure gauche de l’écran.

1. Renseignez le volet **Déployer un modèle** de la façon suivante :

    Champ| Valeur
    ----|----
    Nom du déploiement| bikeshare-deploy
    Description du déploiement| déploiement de la demande de vélos en libre-service
    Type de capacité de calcul | Sélectionnez une instance de calcul Azure (ACI)
    Activer l’authentification| Désactivez. 
    Utiliser les ressources d’un déploiement personnalisé| Désactivez. La désactivation permet de générer automatiquement le fichier de pilote par défaut (script de scoring) et le fichier d’environnement. 
    
    Pour cet exemple, nous utilisons les valeurs par défaut fournies dans le menu *Avancé*. 

1. Sélectionnez **Déployer**.  

    Un message vert de réussite apparaît en haut de l’écran **Exécuter**, indiquant que le déploiement a été correctement démarré. Vous pouvez voir la progression du déploiement dans le volet **Récapitulatif du modèle** sous **État du déploiement**.
    
Une fois le déploiement terminé, vous disposez d’un service web opérationnel pour générer des prédictions. 

Passez aux [**Étapes suivantes**](#next-steps) pour en savoir plus sur la consommation de votre nouveau service web et tester vos prédictions à l’aide de la prise en charge d’Azure Machine Learning intégrée à Power BI.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les fichiers de déploiement sont plus volumineux que les fichiers de données et d’expérimentation. Le coût de leur stockage est donc plus élevé. Supprimez uniquement les fichiers de déploiement pour réduire les coûts associés à votre compte, ou si vous souhaitez conserver vos fichiers d’expérience et d’espace de travail. Dans le cas contraire, supprimez l’intégralité du groupe de ressources, si vous n’envisagez pas d’utiliser les fichiers.  

### <a name="delete-the-deployment-instance"></a>Supprimer l’instance de déploiement

Supprimez uniquement l’instance de déploiement d’Azure Machine Learning Studio si vous souhaitez conserver le groupe de ressources et l’espace de travail pour d’autres tutoriels et à des fins d’exploration. 

1. Accédez à [Azure Machine Learning Studio](https://ml.azure.com/). Accédez à votre espace de travail et, à gauche dans le volet **Ressources**, sélectionnez **Points de terminaison**. 

1. Sélectionnez le déploiement à supprimer et sélectionnez **Supprimer**. 

1. Sélectionnez **Continuer**.

### <a name="delete-the-resource-group"></a>Supprimer le groupe de ressources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez utilisé le ML automatisé dans Azure Machine Learning Studio pour créer et déployer un modèle de prévision de série chronologique qui prédit la demande de location de vélos en libre-service. 

Pour savoir comment créer un schéma pris en charge par Power BI pour faciliter la consommation de votre service web récemment déployé, lisez cet article :

> [!div class="nextstepaction"]
> [Utiliser un service web](how-to-consume-web-service.md#consume-the-service-from-power-bi)

+ En savoir plus sur le [Machine Learning automatisé](concept-automated-ml.md).
+ Pour plus d’informations sur les métriques et les graphiques de classification, consultez l’article [Comprendre les résultats du Machine Learning automatisé](how-to-understand-automated-ml.md).
+ Découvrez-en plus sur la [caractérisation](how-to-configure-auto-features.md#featurization).
+ En savoir plus sur le [profilage des données](how-to-connect-data-ui.md#profile).

>[!NOTE]
> Ce jeu de données de vélos en libre-service a été modifié pour les besoins de ce tutoriel. Ce jeu de données, mis à disposition dans le cadre d’un [concours Kaggle](https://www.kaggle.com/c/bike-sharing-demand/data), a été initialement proposé par [Capital Bikeshare](https://www.capitalbikeshare.com/system-data). Il est également accessible dans la [base de données de Machine Learning de l’UCI](http://archive.ics.uci.edu/ml/datasets/Bike+Sharing+Dataset).<br><br>
> Source : Fanaee-T, Hadi et Gama, Joao, Event labeling combining ensemble detectors and background knowledge, Progress in Artificial Intelligence (2013) : p. 1-15, Springer Berlin Heidelberg.
