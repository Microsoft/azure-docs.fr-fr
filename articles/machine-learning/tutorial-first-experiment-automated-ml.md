---
title: Créer des modèles de classification de ML automatisé
titleSuffix: Azure Machine Learning
description: Découvrez comment entraîner et déployer des modèles de classification avec l’interface de ML (machine learning) automatisé d’Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
author: cartacioS
ms.author: sacartac
ms.reviewer: nibaccam
ms.date: 07/10/2020
ms.custom: automl
ms.openlocfilehash: 4b2769139e74289c4760b5c398c80380afea351f
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921888"
---
# <a name="tutorial-create-a-classification-model-with-automated-ml-in-azure-machine-learning"></a>Tutoriel : Créer un modèle de classification avec le ML automatisé dans Azure Machine Learning


Dans ce tutoriel, vous allez découvrir comment créer un modèle de classification simple sans écrire une seule ligne de code, en utilisant le machine learning automatisé d’Azure Machine Learning Studio. Ce modèle de classification prédit si un client va souscrire à un compte à terme auprès d’une institution financière.

Avec le machine learning automatisé, vous pouvez automatiser des tâches fastidieuses. Le machine learning automatisé itère rapidement sur de nombreuses combinaisons d’algorithmes et d’hyperparamètres pour vous aider à trouver le meilleur modèle basé sur une métrique de réussite de votre choix.

Pour obtenir un exemple de prévision de série chronologique, consultez [Tutoriel : Prévision de la demande et AutoML](tutorial-automated-ml-forecast.md).

Dans ce tutoriel, vous allez apprendre à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créez un espace de travail Machine Learning.
> * Exécuter une expérience de machine learning automatisé.
> * Voir les détails de l'expérience.
> * Déployer le modèle.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://aka.ms/AMLFree).

* Téléchargez le fichier de données [**bankmarketing_train.csv**](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv). La colonne **y** indique si un client a souscrit à un compte à terme, qui est ensuite identifié comme colonne cible pour les prédictions de ce tutoriel. 

## <a name="create-a-workspace"></a>Créer un espace de travail

Un espace de travail Azure Machine Learning est une ressource fondamentale du cloud que vous utilisez pour expérimenter, entraîner et déployer des modèles Machine Learning. Il lie votre abonnement Azure et votre groupe de ressources à un objet facile à consommer dans le service. 

Il existe de nombreuses [façons de créer un espace de travail](how-to-manage-workspace.md). Dans ce tutoriel, vous allez créer un espace de travail par le biais du portail Azure, une console web pour la gestion de vos ressources Azure.

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

>[!IMPORTANT] 
> Prenez note de votre **espace de travail** et de votre **abonnement**. Vous en aurez besoin pour être sûr de créer votre expérience au bon endroit. 

## <a name="get-started-in-azure-machine-learning-studio"></a>Bien démarrer dans Azure Machine Learning Studio

Vous effectuez les étapes de configuration et d’exécution d’expérience suivantes via Azure Machine Learning Studio, accessible à l’adresse https://ml.azure.com. Cette interface web centralisée comprend des outils de machine learning permettant de mettre en œuvre des scénarios de science des données pour les utilisateurs de science des données de tous niveaux de compétence. Studio n’est pas prise en charge par les navigateurs Internet Explorer.

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com).

1. Sélectionnez votre abonnement et l’espace de travail que vous avez créé.

1. Sélectionnez **Prise en main**.

1. Dans le volet gauche, sélectionnez **Automated ML** (ML automatisé) sous la section **Author** (Créer).

   Puisqu’il s’agit de votre première expérience de ML automatisé, vous verrez une liste vide et des liens vers la documentation.

   ![Page de prise en main](./media/tutorial-first-experiment-automated-ml/get-started.png)

1. Sélectionnez **+Nouvelle exécution de ML automatisé**. 

## <a name="create-and-load-dataset"></a>Créer et charger un jeu de données

Avant de configurer votre expérience, chargez votre fichier de données dans votre espace de travail sous la forme d’un jeu de données Azure Machine Learning. De cette façon, vous pouvez vérifier que la mise en forme de vos données convient à votre expérience.

1. Pour créer un jeu de données, sélectionnez **À partir de fichiers locaux** dans la liste déroulante **+ Créer un jeu de données**. 

    1. Dans le formulaire **Informations de base**, donnez un nom unique à votre jeu de données et indiquez éventuellement une description. L’interface de ML automatisé ne prend actuellement en charge que les jeux de données tabulaires. Le type de jeu de données par défaut doit donc être *Tabulaire*.

    1. Sélectionnez **Suivant** en bas à gauche.

    1. Dans le formulaire **Sélection d’un magasin de données et de fichiers**, sélectionnez le magasin de données par défaut qui a été automatiquement configuré durant la création de votre espace de travail : **workspaceblobstore (Stockage Blob Azure)** . C’est là que vous chargerez votre fichier de données pour le rendre accessible à votre espace de travail.

    1. Sélectionnez **Parcourir**.
    
    1. Choisissez le fichier **bankmarketing_train.csv** sur votre ordinateur local. Il s’agit du fichier que vous avez téléchargé en tant que [prérequis](https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_train.csv).

    1. Donnez un nom unique à votre jeu de données et indiquez éventuellement une description. 

    1. En bas à gauche, sélectionnez **Suivant** pour le charger dans le conteneur par défaut qui a été configuré automatiquement lors de la création de votre espace de travail.  
    
       Une fois le chargement terminé, le formulaire Settings and preview (Paramètres et aperçu) est prérenseigné en fonction du type de fichier. 
       
    1. Vérifiez que le formulaire **Settings and preview** est renseigné comme ci-dessous, puis sélectionnez **Next** (Suivant).
        
        Champ|Description| Valeur pour le tutoriel
        ---|---|---
        Format de fichier|Définit la disposition et le type des données stockées dans un fichier.| Delimited
        Délimiteur|Un ou plusieurs caractères utilisés pour spécifier la limite entre&nbsp; des régions distinctes et indépendantes dans du texte brut ou d’autres flux de données. |Comma
        Encodage|Identifie la table de schéma bits/caractères à utiliser pour lire votre jeu de données.| UTF-8
        En-têtes de colonne| Indique la façon dont les éventuels en-têtes du jeu de données sont traités.| Tous les fichiers ont les mêmes en-têtes
        Ignorer les lignes | Indique le nombre éventuel de lignes ignorées dans le jeu de données.| None

    1. Le formulaire **Schema** permet de configurer davantage vos données pour cette expérience. Pour cet exemple, sélectionnez le bouton bascule correspondant à la caractéristique **day_of_week**, afin de ne pas l’inclure pour cette expérience. Sélectionnez **Suivant**.

        ![Configuration de l’onglet Aperçu](./media/tutorial-first-experiment-automated-ml/schema-tab-config.gif)

    1. Dans le formulaire **Confirmer les détails**, vérifiez que les informations correspondent à celles qui ont été précédemment renseignées sur les formulaires **Informations de base, magasin de données et sélection de fichiers** et **Paramètres et aperçu**.
    
    1. Sélectionnez **Créer** pour terminer la création de votre jeu de données.
    
    1. Sélectionnez votre jeu de données une fois qu’il apparaît dans la liste.
    
    1. Passez en revue l’**Aperçu des données** pour vous assurer que vous n’avez pas inclus **day_of_week**, puis sélectionnez **OK**.

    1. Sélectionnez **Suivant**.

## <a name="configure-experiment-run"></a>Configurer l’exécution de l’expérience

Une fois que vous avez chargé et configuré vos données, vous pouvez configurer votre expérience. Cette configuration comprend des tâches de conception d’expérience, comme la sélection de la taille de votre environnement de calcul et la spécification de la colonne que vous voulez prédire. 

1. Remplissez le formulaire **Configurer l’exécution** comme suit :
    1. Entrez le nom suivant pour l’expérience : `my-1st-automl-experiment`

    1. Sélectionnez **y** comme colonne cible, ce que vous souhaitez prédire. Cette colonne indique si le client a souscrit à un compte de dépôt à terme.
    
    1. Sélectionnez **Create a new compute** (Créer un nouveau calcul), puis configurez la cible de calcul. Une cible de calcul est un environnement de ressources local ou informatique utilisé pour exécuter votre script d’entraînement ou pour héberger votre déploiement de service. Pour cette expérience, nous utilisons un calcul informatique. 

        Champ | Description | Valeur pour le tutoriel
        ----|---|---
        Nom du calcul |Nom unique qui identifie votre contexte de calcul.|automl-compute
        Type de&nbsp;machine&nbsp;virtuelle| Sélectionnez le type de machine virtuelle pour votre calcul.|Processeur (CPU)
        Taille de la&nbsp;machine&nbsp;virtuelle| Sélectionnez la taille de la machine virtuelle pour votre calcul.|Standard_DS12_V2
        Nombre minimal/maximal de nœuds| Pour profiler des données, vous devez spécifier un ou plusieurs nœuds.|Nœuds min. : 1<br>Nœuds max. : 6
        Secondes d’inactivité avant le scale-down | Durée d’inactivité avant que le cluster ne fasse l’objet d’un scale-down au nombre de nœuds minimal.|120 (par défaut)
        Paramètres avancés | Paramètres pour configurer et autoriser un réseau virtuel pour votre expérience.| None
        1. Sélectionnez **Créer** pour accéder à la cible de calcul. 

            **Quelques minutes sont nécessaires pour achever l’opération**. 

        1. Une fois la création terminée, sélectionnez votre nouvelle cible de calcul dans la liste déroulante.

    1. Sélectionnez **Suivant**.

1. Dans le formulaire **Type et paramètre de la tâche**, configurez votre expérience de machine learning automatisé en spécifiant le type et les paramètres de configuration de la tâche de machine learning.
    
    1.  Sélectionnez **Classification** comme type de tâche de machine learning.

    1. Sélectionnez **Afficher des paramètres de configuration supplémentaires** et renseignez les champs comme suit. Ces paramètres permettent de mieux contrôler le travail d’entraînement. Sinon, les valeurs par défaut sont appliquées en fonction de la sélection de l’expérience et des données.

        Configurations&nbsp;supplémentaires|Description|Valeur&nbsp;pour&nbsp;le tutoriel
        ------|---------|---
        Métrique principale| Métrique d’évaluation selon laquelle l’algorithme de Machine Learning sera mesuré.|AUC_weighted
        Expliquer le meilleur modèle| Montre automatiquement l’explicabilité sur le meilleur modèle créé par le ML automatisé.| Activer
        Algorithmes bloqués | Algorithmes que vous souhaitez exclure du travail de formation| None
        Critère de sortie| Lorsqu’une condition est remplie, la tâche d’entraînement est arrêtée. |Durée&nbsp;de la tâche&nbsp;de formation : 1 <br> Seuil&nbsp;de score&nbsp;de métrique : None
        Validation | Choisissez un type de validation croisée et un nombre de tests.|Type de validation :<br>&nbsp;validation croisée&nbsp;k-fold <br> <br> Nombre de validations : 2
        Accès concurrentiel| Nombre maximal d’itérations parallèles exécutées par itération| Nombre maximal&nbsp;d’itérations&nbsp;simultanées : 5
        
        Sélectionnez **Enregistrer**.

1. Sélectionnez **Terminer** pour exécuter l’expérience. L’écran **Détails de l’exécution** s’ouvre et affiche l’**État de l’exécution** dans la partie supérieure au début de la préparation de l’expérience.

>[!IMPORTANT]
> La préparation nécessaire à l’exécution de l’expérience prend **10 à 15** minutes.
> Une fois que l’exécution a commencé, **2-3 minutes supplémentaires sont nécessaires pour chaque itération**.  
> Sélectionnez **Actualiser** périodiquement pour voir l’état de l’exécution à mesure que l’expérience progresse.
>
> Dans un environnement de production, cette durée est un peu plus longue. Toutefois, dans le cadre de ce tutoriel, nous vous suggérons de commencer à explorer les algorithmes testés dans l’onglet **Modèles** à mesure qu’ils se terminent, pendant que les autres sont encore en cours d’exécution. 

##  <a name="explore-models"></a>Explorer les modèles

Accédez à l’onglet **Modèles** pour voir les algorithmes (modèles) testés. Par défaut, les modèles sont classés par score de métrique à mesure qu’ils se terminent. Pour ce tutoriel, le modèle qui obtient le score le plus élevé d’après la métrique **AUC_weighted** choisie figure en haut de la liste.

En attendant que toutes les modèles d’expérience se terminent, sélectionnez le **Nom de l’algorithme** d’un modèle terminé pour explorer ses performances en détail. 

L’exemple suivant parcourt les onglets **Détails** et **Métriques** pour montrer les propriétés, les métriques et les graphiques de performances du modèle sélectionné. 

![Détails sur l’exécution de l’itération](./media/tutorial-first-experiment-automated-ml/run-detail.gif)

## <a name="deploy-the-best-model"></a>Déployer le meilleur modèle

L’interface de machine learning automatisé vous permet de déployer le meilleur modèle comme service web en quelques étapes. Le déploiement consiste à intégrer le modèle pour qu’il puisse prédire de nouvelles données et identifier les domaines potentiels d’opportunités. 

Dans le cadre de cette expérience, le déploiement sur un service web signifie que l’établissement financier dispose désormais d’une solution web itérative et scalable pour identifier les clients potentiels d’un compte à terme. 

Vérifiez si l’exécution de votre expérience est terminée. Pour cela, revenez à la page d’exécution du parent en sélectionnant **Exécution 1** en haut de votre écran. Un état **Terminé** est affiché en haut à gauche de l’écran. 

Une fois l’expérience terminée, la page **Détails** est renseignée avec une section **Récapitulatif du meilleur modèle**. Dans ce contexte d’expérience, **VotingEnsemble** est considéré comme le meilleur modèle d’après la métrique **AUC_weighted**.  

Nous déployons ce modèle, mais nous vous informons que le déploiement prend 20 minutes environ. Le processus de déploiement comporte plusieurs étapes, notamment l’inscription du modèle, la génération de ressources et leur configuration pour le service web.

1. Sélectionnez **VotingEnsemble** pour ouvrir la page spécifique au modèle.

1. Sélectionnez le bouton **Déployer** en haut à gauche.

1. Renseignez le volet **Déployer un modèle** de la façon suivante :

    Champ| Valeur
    ----|----
    Nom du déploiement| my-automl-deploy
    Description du déploiement| Déploiement de ma première expérience de Machine Learning automatisé
    Type de capacité de calcul | Sélectionnez une instance de calcul Azure (ACI)
    Activer l’authentification| Désactivez. 
    Utiliser des déploiements personnalisés| Désactivez. Permet de générer automatiquement le fichier de pilote par défaut (script de score) et le fichier d’environnement. 
    
    Pour cet exemple, nous utilisons les valeurs par défaut fournies dans le menu *Avancé*. 

1. Sélectionnez **Déployer**.  

    Un message vert de réussite apparaît en haut de l’écran **Exécuter**, et dans le volet **Récapitulatif du modèle**, un message d’état s’affiche sous **État du déploiement**. Cliquez régulièrement sur **Actualiser** pour vérifier l’état du déploiement.
    
Vous disposez maintenant d’un service web opérationnel pour générer des prédictions. 

Passez aux [**Étapes suivantes**](#next-steps) pour en savoir plus sur l’utilisation de votre nouveau service web et tester vos prédictions à l’aide de la prise en charge d’Azure Machine Learning intégrée dans Power BI.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les fichiers de déploiement sont plus volumineux que les fichiers de données et d’expérimentation. Le coût de leur stockage est donc plus élevé. Supprimez uniquement les fichiers de déploiement pour réduire les coûts associés à votre compte, ou si vous souhaitez conserver vos fichiers d’expérience et d’espace de travail. Dans le cas contraire, supprimez l’intégralité du groupe de ressources, si vous n’envisagez pas d’utiliser les fichiers.  

### <a name="delete-the-deployment-instance"></a>Supprimer l’instance de déploiement

Supprimez uniquement l’instance de déploiement d’Azure Machine Learning (accessible sur https:\//ml.azure.com/) si vous souhaitez conserver le groupe de ressources et l’espace de travail pour d’autres tutoriels et à des fins d’exploration. 

1. Accéder à [Azure Machine Learning](https://ml.azure.com/). Accédez à votre espace de travail et, à gauche dans le volet **Ressources**, sélectionnez **Points de terminaison**. 

1. Sélectionnez le déploiement à supprimer et sélectionnez **Supprimer**. 

1. Sélectionnez **Continuer**.

### <a name="delete-the-resource-group"></a>Supprimer le groupe de ressources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel sur le machine learning automatisé, vous avez utilisé l’interface de ML automatisé d’Azure Machine Learning pour créer et déployer un modèle de classification. Pour plus d’informations et pour connaître les étapes suivantes, consultez ces articles :

> [!div class="nextstepaction"]
> [Utiliser un service web](how-to-consume-web-service.md#consume-the-service-from-power-bi)

+ En savoir plus sur le [Machine Learning automatisé](concept-automated-ml.md).
+ Pour plus d’informations sur les métriques et les graphiques de classification, consultez l’article [Comprendre les résultats du Machine Learning automatisé](how-to-understand-automated-ml.md).
+ Découvrez-en plus sur la [caractérisation](how-to-configure-auto-features.md#featurization).
+ En savoir plus sur le [profilage des données](how-to-connect-data-ui.md#profile).


>[!NOTE]
> Ce jeu de données Bank Marketing est disponible sous licence [Creative Commons (CCO : domaine public)](https://creativecommons.org/publicdomain/zero/1.0/). Tous les droits du contenu individuel de la base de données sont concédés sous [licence du contenu de base de données](https://creativecommons.org/publicdomain/zero/1.0/) et sont disponibles sur [Kaggle](https://www.kaggle.com/janiobachmann/bank-marketing-dataset). Ce jeu de données était initialement disponible dans la [base de données Machine Learning UCI](https://archive.ics.uci.edu/ml/datasets/bank+marketing).<br><br>
> [Moro et al., 2014] S. Moro, P. Cortez et P. Rita. A Data-Driven Approach to Predict the Success of Bank Telemarketing. Decision Support Systems, Elsevier, 62:22-31, Juin 2014.
