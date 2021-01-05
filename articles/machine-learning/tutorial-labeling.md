---
title: 'Tutoriel : Créer un projet d’étiquetage pour la classification d’images'
titleSuffix: Azure Machine Learning
description: Apprenez à gérer le processus d’étiquetage des images afin qu’elles puissent être utilisées dans les modèles de classification d’images multiclasse.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.author: sgilley
author: sdgilley
ms.reviewer: ranku
ms.date: 04/09/2020
ms.custom: data4ml
ms.openlocfilehash: 5aec3d6bb2afeadda64ac19563f8334a6aef3b77
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347504"
---
# <a name="tutorial-create-a-labeling-project-for-multi-class-image-classification"></a>Tutoriel : Créer un projet d’étiquetage pour la classification d’images multiclasse 


Ce tutoriel vous montre comment gérer le processus d’étiquetage (également appelé balisage) des images à utiliser comme données pour générer des modèles Machine Learning. L’étiquetage des données dans Azure Machine Learning est en préversion publique.

Si vous souhaitez entraîner un modèle Machine Learning pour classifier des images, vous avez besoin de centaines voire de milliers d’images correctement étiquetées.  Azure Machine Learning vous aide à gérer la progression de votre équipe privée d’experts du domaine à mesure qu’ils étiquettent vos données.
 
Dans ce tutoriel, vous allez utiliser des images de chats et de chiens.  Comme chaque image représente un chat ou un chien, il s’agit d’un projet d’étiquetage *multiclasse*. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
>
> * Créer un compte de stockage Azure et charger des images sur le compte.
> * Créez un espace de travail Machine Learning.
> * Créer un projet d’étiquetage d’images multiclasse.
> * Étiqueter vos données.  Vous ou vos étiqueteurs pouvez effectuer cette tâche.
> * Mener à bien le projet en examinant et en exportant les données.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://aka.ms/AMLFree).

## <a name="create-a-workspace"></a>Créer un espace de travail

Un espace de travail Azure Machine Learning est une ressource fondamentale du cloud que vous utilisez pour expérimenter, entraîner et déployer des modèles Machine Learning. Il lie votre abonnement Azure et votre groupe de ressources à un objet facile à consommer dans le service.

Il existe de nombreuses [façons de créer un espace de travail](how-to-manage-workspace.md). Dans ce tutoriel, vous allez créer un espace de travail par le biais du portail Azure, une console web pour la gestion de vos ressources Azure.

[!INCLUDE [aml-create-portal](../../includes/aml-create-in-portal.md)]

## <a name="start-a-labeling-project"></a>Démarrer un projet d’étiquetage

Vous allez ensuite gérer le projet d’étiquetage de données dans Azure Machine Learning Studio, une interface consolidée qui comprend des outils Machine Learning permettant de mettre en œuvre des scénarios de science des données pour les utilisateurs de science des données de tous niveaux de compétences. Studio n’est pas prise en charge par les navigateurs Internet Explorer.

1. Connectez-vous à [Azure Machine Learning Studio](https://ml.azure.com).

1. Sélectionnez votre abonnement et l’espace de travail que vous avez créé.

### <a name="create-a-datastore"></a><a name="create-datastore"></a>Créer un magasin de données

Les magasins de données Azure Machine Learning permettent de stocker les informations de connexion, comme votre ID d’abonnement et votre autorisation de jeton. Vous utilisez ici un magasin de données pour vous connecter au compte de stockage qui contient les images de ce tutoriel.

1. Sur le côté gauche de votre espace de travail, sélectionnez **Magasins de données**.

1. Sélectionnez **+ Nouveau magasin de données**.

1. Remplissez le formulaire avec les paramètres suivants :

    Champ|Description 
    ---|---
    Nom du magasin de données | Donnez un nom au magasin de données.  Ici, nous utilisons **labeling_tutorial**.
    Type de magasin de données | Sélectionnez le type de stockage.  Ici, nous utilisons **Stockage Blob Azure**, le stockage préféré pour les images.
    Méthode de sélection du compte | Sélectionnez **Entrer manuellement**.
    URL | `https://azureopendatastorage.blob.core.windows.net/openimagescontainer`
    Type d'authentification | Sélectionnez **Jeton SAS**.
    Clé de compte | `?sv=2019-02-02&ss=bfqt&srt=sco&sp=rl&se=2025-03-25T04:51:17Z&st=2020-03-24T20:51:17Z&spr=https&sig=7D7SdkQidGT6pURQ9R4SUzWGxZ%2BHlNPCstoSRRVg8OY%3D`

1. Sélectionnez **Créer** pour créer le magasin de données.

### <a name="create-a-labeling-project"></a>Créer un projet d’étiquetage

Maintenant que vous disposez de l’accès aux données à étiqueter, créez votre projet d’étiquetage.

1. En haut de la page, sélectionnez **Projets**.

1. Sélectionnez **+ Ajouter un projet**.

    :::image type="content" source="media/tutorial-labeling/create-project.png" alt-text="Créer un projet":::

### <a name="project-details"></a>Détails du projet

1. Utilisez l’entrée suivante pour le formulaire **Détails du projet** :

    Champ|Description 
    ---|---
    Nom du projet | Donnez un nom à votre projet.  Ici, nous allons utiliser **tutorial-cats-n-dogs**.
    Type de tâche d’étiquetage | Sélectionnez **Classification d’images (plusieurs classes)** .  
    
    Sélectionnez **Suivant** pour poursuivre la création du projet.

### <a name="select-or-create-a-dataset"></a>Sélectionner ou créer un jeu de données

1.   Dans le formulaire **Sélectionner ou créer un jeu de données**, sélectionnez le deuxième choix, **Créer un jeu de données**, puis le lien **Du magasin de données**.

1. Utilisez l’entrée suivante pour le formulaire **Créer un jeu de données à partir d’un magasin de données** :

    1. Dans le formulaire **Informations de base**, ajoutez un nom ; ici, nous allons utiliser **images-for-tutorial**.  Ajoutez la description de votre choix.  Sélectionnez ensuite **Suivant**.
    1. Dans le formulaire **Sélection d’un magasin de données**, utilisez la liste déroulante pour sélectionner votre **Magasin de données créé précédemment**, par exemple **tutorial_images (Stockage Blob Azure)**
    1. Ensuite, toujours dans le formulaire **Sélection d’un magasin de données**, sélectionnez **Parcourir**, puis **MultiClass - DogsCats**.  Sélectionnez **Enregistrer** pour utiliser **/MultiClass - DogsCats** comme chemin.
    1. Sélectionnez **Suivant** pour confirmer les détails, puis **Créer** pour créer le jeu de données.
    1. Sélectionnez le cercle en regard du nom du jeu de données dans la liste, par exemple **images-for-tutorial**.

1. Sélectionnez **Suivant** pour poursuivre la création du projet.

### <a name="incremental-refresh"></a>Actualisation incrémentielle

Si vous envisagez d’ajouter de nouvelles images à votre jeu de données, l’actualisation incrémentielle recherche ces nouvelles images et les ajoute à votre projet.  Lorsque vous activez cette fonctionnalité, le projet recherche régulièrement de nouvelles images.  Comme vous n’allez pas ajouter de nouvelles images au magasin de données pour ce tutoriel, laissez cette fonctionnalité désactivée.

Sélectionnez **Suivant** pour continuer.

### <a name="label-classes"></a>Classes d’étiquettes

1. Dans le formulaire **Classes d’étiquettes**, tapez un nom d’étiquette, puis sélectionnez **+ Ajouter une étiquette** pour taper l’étiquette suivante.  Pour ce projet, les étiquettes sont **Cat**, **Dog** et **Uncertain**.

1. Sélectionnez **Suivant** quand vous avez ajouté toutes les étiquettes.

### <a name="labeling-instructions"></a>Instructions d’étiquetage

1. Dans le formulaire **Instructions d’étiquetage**, vous pouvez indiquer un lien vers un site web qui fournit des instructions détaillées pour vos étiqueteurs.  Nous allons laisser cette valeur vide pour ce tutoriel.

1. Vous pouvez également ajouter une brève description de la tâche directement dans le formulaire.  Tapez **Tutoriel d’étiquetage - Chats & Chiens**.

1. Sélectionnez **Suivant**.

1. Dans la section **Étiquetage assisté par ML**, laissez la case décochée. L’étiquetage assisté par ML requiert plus de données que ce que vous allez utiliser dans ce tutoriel.

1. Sélectionnez **Créer un projet**.

Cette page ne s’actualise pas automatiquement. Après une pause, actualisez manuellement la page jusqu’à ce que l’état du projet indique **Créé**.

## <a name="start-labeling"></a>Commencer l’étiquetage

Vous avez maintenant configuré vos ressources Azure et un projet d’étiquetage de données. Il est temps d’ajouter des étiquettes à vos données.

### <a name="tag-the-images"></a>Baliser les images

Dans cette partie du tutoriel, vous allez passer du rôle d’*administrateur de projet* à celui d’*étiqueteur*.  Toute personne disposant d’un accès Contributeur à votre espace de travail peut devenir un étiqueteur.

1. Dans [Machine Learning Studio](https://ml.azure.com), sélectionnez **Étiquetage des données** sur le côté gauche pour rechercher votre projet.  

1. Sélectionnez **Lien d’étiquette** pour le projet.

1. Lisez les instructions, puis sélectionnez **Tâches**.

1. Sélectionnez une image miniature sur la droite pour afficher le nombre d’images que vous souhaitez étiqueter en une seule fois. Vous devez étiqueter toutes ces images avant de pouvoir continuer. Changez de disposition uniquement quand vous avez une nouvelle page de données non étiquetées. Tout changement de disposition a pour effet d’effacer le travail de balisage en cours de la page.

1. Sélectionnez une ou plusieurs images, puis une balise à appliquer à la sélection. La balise s’affiche sous l’image.  Continuez de sélectionner et de baliser toutes les images de la page.  Pour sélectionner toutes les images affichées simultanément, choisissez **Tout sélectionner**. Pour appliquer une balise, sélectionnez au moins une image.


    > [!TIP]
    > Vous pouvez sélectionner les neuf premières étiquettes à l’aide des touches numériques du clavier.

1. Une fois que toutes les images de la page sont balisées, sélectionnez **Soumettre** pour envoyer ces étiquettes.

    ![Étiquetage des images](media/tutorial-labeling/catsndogs.gif)

1. Une fois que vous avez envoyé les étiquettes relatives aux données disponibles, Azure actualise la page en affichant un nouvel ensemble d’images provenant de la file d’attente de travail.

## <a name="complete-the-project"></a>Terminer le projet

À présent, vous allez reprendre le rôle d’*administrateur de projet* pour le projet d’étiquetage.

En tant que responsable, vous souhaiterez peut-être examiner le travail de votre étiqueteur.  

### <a name="review-labeled-data"></a>Examiner les données étiquetées

1. Dans [Machine Learning Studio](https://ml.azure.com), sélectionnez **Étiquetage des données** sur le côté gauche pour rechercher votre projet.  

1. Sélectionnez le lien du nom du projet.

1. Le tableau de bord affiche la progression de votre projet.

1. En haut de la page, sélectionnez **Données**.

1. Sur le côté gauche, sélectionnez **Données étiquetées** pour afficher vos images balisées.  

1. Lorsque vous n’êtes pas d’accord avec une étiquette, sélectionnez l’image, puis **Rejeter** en bas de la page.  Les balises sont supprimées et l’image est replacée dans la file d’attente des images sans étiquette.

### <a name="export-labeled-data"></a>Exporter les données étiquetées

Vous pouvez exporter les données d’étiquette pour Machine Learning - Expérimentation à tout moment. Il arrive souvent que les utilisateurs exportent plusieurs fois et entraînent des modèles différents, au lieu d’attendre que toutes les images soient étiquetées.

Vous pouvez exporter les étiquettes d’image au [format COCO](http://cocodataset.org/#format-data) ou en tant que jeu de données Azure Machine Learning. Le format du jeu de données permet de l’utiliser facilement pour l’entraînement dans Azure Machine Learning.  

1. Dans [Machine Learning Studio](https://ml.azure.com), sélectionnez **Étiquetage des données** sur le côté gauche pour rechercher votre projet.  

1. Sélectionnez le lien du nom du projet.

1. Sélectionnez **Exporter** et choisissez **Exporter comme jeu de données Azure ML**. 

    L’état de l’exportation s’affiche juste en dessous du bouton **Exporter**. 

1. Une fois que les étiquettes sont correctement exportées, sélectionnez **Jeux de données** sur le côté gauche pour afficher les résultats.

## <a name="clean-up-resources"></a>Nettoyer les ressources


[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez étiqueté des images.  À présent, utilisez vos données étiquetées :

> [!div class="nextstepaction"]
> [Entraîner un modèle de reconnaissance d’images Machine Learning](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/work-with-data/datasets-tutorial/labeled-datasets/labeled-datasets.ipynb).
