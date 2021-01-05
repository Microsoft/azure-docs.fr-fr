---
title: Créer un projet d’étiquetage des données
titleSuffix: Azure Machine Learning
description: Découvrez comment créer et exécuter des projets d’étiquetage de façon à marquer des données pour le machine learning.  Pour faciliter cette tâche, utilisez l’étiquetage assisté par ML ou l’étiquetage par opérateur humain dans la boucle.
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.date: 07/27/2020
ms.custom: data4ml
ms.openlocfilehash: 5abb222a6211e2f3f92c002a49987ac54edc78e2
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346654"
---
# <a name="create-a-data-labeling-project-and-export-labels"></a>Créer un projet d’étiquetage des données et exporter des étiquettes 



L’étiquetage de grandes quantités de données est souvent un casse-tête dans les projets de Machine Learning. Les projets qui ont un composant de vision par ordinateur, par exemple la classification d’images ou la détection d’objet, nécessitent généralement des étiquettes pour des milliers d’images.
 
L’étiquetage des données [Azure Machine Learning](https://ml.azure.com/) vous fournit un emplacement central pour créer, gérer et superviser les projets d’étiquetage. Il vous permet de coordonner les données, les étiquettes et les membres de l’équipe pour gérer efficacement les tâches d’étiquetage. Machine Learning prend en charge la classification d’images (multiétiquette ou multiclasse) ainsi que l’identification d’objets avec des cadres englobants.

L’étiquetage des données effectue un suivi de la progression et gère la file d’attente des tâches d’étiquetage incomplètes.

Vous pouvez démarrer et arrêter le projet, ainsi que contrôler la progression de l’étiquetage. Vous pouvez passer en revue les données étiquetées et les exporter au format COCO ou en tant que jeu de données Azure Machine Learning.

> [!Important]
> Seuls les projets d’étiquetage de classification des images et d’identification des objets sont actuellement pris en charge. De plus, les images de données doivent être disponibles dans un magasin de données blob Azure. (Si vous ne disposez pas d’un magasin de données, vous pouvez charger des images pendant la création du projet.)

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
> * Création d’un projet
> * Spécifier les données et la structure du projet
> * Exécuter et surveiller le projet
> * Exporter les étiquettes


## <a name="prerequisites"></a>Prérequis

* Les données à étiqueter, dans des fichiers locaux ou un stockage Blog Azure.
* L’ensemble d’étiquettes à appliquer.
* Des instructions pour l’étiquetage.
* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://aka.ms/AMLFree) avant de commencer.
* Un espace de travail Machine Learning. Consultez [Créer un espace de travail Microsoft Azure Machine Learning](how-to-manage-workspace.md).

## <a name="create-a-labeling-project"></a>Créer un projet d’étiquetage

Les projets d’étiquetage sont administrés à partir d’Azure Machine Learning. La page **Projets d’étiquetage** permet de gérer les projets.

Si vos données se trouvent déjà dans le Stockage Blob Azure, vous devez les rendre disponibles sous la forme d’un magasin de données avant de créer le projet d’étiquetage. Pour obtenir un exemple d’utilisation d’un magasin de données, consultez [Tutoriel : Créer votre premier projet d’étiquetage de classification d’images](tutorial-labeling.md).

Pour créer un projet, sélectionnez **Ajouter un projet**. Donnez-lui un nom approprié, puis sélectionnez **Type de tâche d’étiquetage**.

:::image type="content" source="media/how-to-create-labeling-projects/labeling-creation-wizard.png" alt-text="Assistant Création de projet d’étiquetage":::

* Choisissez **Classification d'images (plusieurs classes)** pour les projets où vous souhaitez appliquer uniquement une *seule étiquette* d’un ensemble d’étiquettes à une image.
* Choisissez **Classification d’images (plusieurs étiquettes)** pour les projets où vous souhaitez appliquer *une ou plusieurs* étiquettes d’un ensemble d’étiquettes à une image. Par exemple, vous pouvez étiqueter la photo d’un chien avec *dog* (chien) et *daytime* (jour).
* Choisissez **Identification d’objet (cadre englobant)** pour les projets où vous souhaitez affecter une étiquette et un cadre englobant à chaque objet d’une image.

Quand vous êtes prêt à continuer, sélectionnez **Suivant**.

## <a name="specify-the-data-to-label"></a>Spécifier les données à étiqueter

Si vous avez déjà créé un jeu de données qui contient vos données, sélectionnez-le dans la liste déroulante **Sélectionner un jeu de données existant**. Sinon, sélectionnez **Créer un jeu de données** pour utiliser un magasin de données Azure existant ou pour charger des fichiers locaux.

> [!NOTE]
> Un projet ne peut pas contenir plus de 500 000 images.  Si votre jeu de données en contient davantage, seules les 500 000 premières images sont chargées.  

### <a name="create-a-dataset-from-an-azure-datastore"></a>Créer un jeu de données à partir d’un magasin de données Azure

Dans de nombreux cas, il suffit de charger des fichiers locaux. Toutefois, l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) constitue un moyen plus rapide et plus robuste de transférer une grande quantité de données. Nous recommandons l’Explorateur Stockage en tant que méthode par défaut pour déplacer des fichiers.

Pour créer un jeu de données à partir de données que vous avez déjà stockées dans le Stockage Blob Azure :

1. Sélectionnez **Créer un jeu de données** > **À partir du magasin de données**.
1. Affectez un **Nom** à votre jeu de données.
1. Choisissez **Fichier** en tant que **Type de jeu de données**.  Seuls les types de jeux de données de fichier sont pris en charge.
1. Sélectionnez le magasin de données.
1. Si vos données se trouvent dans un sous-dossier de votre Stockage Blob, choisissez **Parcourir** pour sélectionner le chemin.
    * Ajoutez « /** » au chemin pour inclure tous les fichiers des sous-dossiers du chemin sélectionné.
    * Ajoutez « * */* .* » pour inclure toutes les données figurant dans le conteneur actuel et ses sous-dossiers.
1. Fournissez une description pour votre jeu de données.
1. Sélectionnez **Suivant**.
1. Vérifiez les détails. Sélectionnez **Précédent** pour modifier les paramètres, ou **Créer** pour créer le jeu de données.


### <a name="create-a-dataset-from-uploaded-data"></a>Créer un jeu de données à partir des données chargées

Pour charger directement vos données :

1. Sélectionnez **Créer un jeu de données** > **À partir de fichiers locaux**.
1. Affectez un **Nom** à votre jeu de données.
1. Choisissez « Fichier » en tant que **Type de jeu de données**.
1. *Facultatif :* Sélectionnez **Paramètres avancés** pour personnaliser le magasin de données, le conteneur et le chemin de vos données.
1. Sélectionnez **Parcourir** pour choisir les fichiers locaux à charger.
1. Fournissez une description de votre jeu de données.
1. Sélectionnez **Suivant**.
1. Vérifiez les détails. Sélectionnez **Précédent** pour modifier les paramètres, ou **Créer** pour créer le jeu de données.

Les données sont chargées dans le magasin d’objets blob par défaut (« workspaceblobstore ») de votre espace de travail Machine Learning.

## <a name="configure-incremental-refresh"></a><a name="incremental-refresh"> </a> Configurer l’actualisation incrémentielle

Si vous envisagez d’ajouter de nouvelles images à votre jeu de données, utilisez l’actualisation incrémentielle pour ajouter ces nouvelles images à votre projet.   Quand l’**actualisation incrémentielle** est activée, des recherches sont régulièrement effectuées dans le jeu de données pour que de nouvelles images soient ajoutées à un projet, en fonction du taux d’achèvement de l’étiquetage.   La recherche de nouvelles données s’arrête quand le projet contient le nombre maximal de 500 000 images.

Pour ajouter d’autres images à votre projet, utilisez l’[Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) à charger dans le dossier approprié du stockage Blob. 

Cochez la case **Activer l’actualisation incrémentielle** quand vous souhaitez que votre projet supervise continuellement les nouvelles données dans le magasin de données. Ces données sont tirées (pull) dans votre projet une fois par jour quand l’option est activée.

Décochez cette case si vous ne souhaitez pas que les nouvelles images qui apparaissent dans le magasin de données soient ajoutées à votre projet.

Vous trouverez l’horodatage de la dernière actualisation dans la section **Actualisation incrémentielle** de l’onglet **Détails** pour votre projet.


## <a name="specify-label-classes"></a>Spécifier des classes d’étiquettes

Dans la page **Classes d’étiquettes**, spécifiez l’ensemble des classes permettant de catégoriser vos données. Faites-le avec précaution, car la justesse et la rapidité de vos étiqueteurs sont affectées par leur capacité à opérer des choix corrects parmi les classes. Par exemple, au lieu d’indiquer le genre et l’espèce complets de plantes ou d’animaux, utilisez un code de champ ou abrégez le genre.

Entrez une étiquette par ligne. Utilisez le bouton **+** pour ajouter une nouvelle ligne. Si vous avez plus de 3 ou 4 étiquettes mais moins de 10, vous pouvez ajouter des chiffres devant les noms (« 1 : », « 2 : »), ce qui permet aux étiqueteurs d’utiliser les touches numériques pour accélérer le travail.

## <a name="describe-the-labeling-task"></a>Décrire la tâche d’étiquetage

Il est important d’expliquer clairement la tâche d’étiquetage. Dans la page **Instructions d’étiquetage**, vous pouvez soit ajouter un lien vers un site externe où trouver des instructions relatives à l’étiquetage, soit fournir des instructions dans la zone d’édition de la page. Veillez à ce que les instructions soient axées sur la tâche et appropriées pour le public. Prenez en compte les questions suivantes :

* Quelles sont les étiquettes qu’ils vont voir, et comment vont-ils les choisir ? Existe-t-il un texte de référence à consulter ?
* Que doivent-ils faire si aucune étiquette ne semble appropriée ?
* Que doivent-ils faire si plusieurs étiquettes semblent appropriées ?
* Quel seuil de confiance doivent-ils appliquer à une étiquette ? Voulez-vous qu’ils exercent leur « meilleur jugement » s’ils hésitent ?
* Que doivent-ils faire avec des objets partiellement masqués ou qui se chevauchent ?
* Que doivent-ils faire si un objet est coupé par le bord de l’image ?
* Que doivent-ils faire après avoir soumis une étiquette, s’ils pensent avoir commis une erreur ?

Pour les cadres englobants, les questions importantes sont les suivantes :

* Comment le rectangle englobant est-il défini pour cette tâche ? Doivent-ils être entièrement à l’intérieur des objets, ou doivent-il être à l’extérieur ? Doivent-ils être rognés le plus fidèlement possible, ou une certaine approximation est-elle acceptable ?
* Quel niveau de soin et de cohérence attendez-vous des étiqueteurs dans la définition des cadres englobants ?
* Comment étiqueter l’objet qui est partiellement affiché dans l’image ? 
* Comment étiqueter l’objet qui est partiellement recouvert par un autre objet ?

>[!NOTE]
> Notez bien que les étiqueteurs peuvent sélectionner les 9 premières étiquettes à l’aide des touches numériques allant de 1 à 9.

## <a name="use-ml-assisted-labeling"></a>Utiliser l’étiquetage assisté par ML

La page **ML assisted labeling** (Étiquetage assisté par ML) vous permet de déclencher des modèles Machine Learning automatiques pour accélérer la tâche d’étiquetage. Au début de votre projet d’étiquetage, les images sont mélangées dans un ordre aléatoire pour réduire le biais potentiel. Cependant, le biais éventuellement présent dans le jeu de données se reflète dans le modèle entraîné. Par exemple, si 80 % de vos images appartiennent à une même classe, environ 80 % des données utilisées pour l’entraînement du modèle feront partie de cette classe. Cet entraînement n’inclut pas l’apprentissage actif.

Sélectionnez *Enable ML assisted labeling* (Activer l’étiquetage assisté par ML) et spécifiez un GPU pour activer l’étiquetage assisté, qui se compose de deux phases :
* Clustering
* Préétiquetage

Le nombre exact d’images étiquetées nécessaires au démarrage de l’étiquetage assisté n’est pas un nombre fixe.  Il peut varier considérablement d’un projet d’étiquetage à un autre. Pour certains projets, il est parfois possible de voir des tâches de préétiquetage ou de clustering après l’étiquetage manuel de 300 images. L’étiquetage assisté par ML s’appuie sur une technique appelée *apprentissage par transfert* (ou « Transfer Learning »), qui utilise un modèle préentraîné pour accélérer le processus d’entraînement. Si les classes de votre jeu de données sont similaires à celles du modèle préentraîné, les préétiquettes peuvent être disponibles après seulement quelques centaines d’images étiquetées manuellement. Si votre jeu de données est très différent des données utilisées pour préentraîner le modèle, cela peut prendre beaucoup plus de temps.

Sachant que les étiquettes finales dépendent encore de l’entrée de l’étiqueteur, cette technologie d’étiquetage est parfois appelée *Opérateur humain dans la boucle*.

> [!NOTE]
> L’étiquetage des données assisté par ML ne prend pas en charge les comptes de stockage par défaut sécurisés derrière un [réseau virtuel](how-to-network-security-overview.md). Vous devez utiliser un compte de stockage autre que celui par défaut pour l’étiquetage des données assisté par ML. Le compte de stockage autre que celui par défaut peut être sécurisé derrière le réseau virtuel. 

### <a name="clustering"></a>Clustering

Après l’envoi d’un certain nombre d’étiquettes, le modèle Machine Learning pour la classification d’images commence à regrouper les images similaires.  Ces images similaires sont présentées aux étiqueteurs dans le même écran pour accélérer le balisage manuel. Le clustering est particulièrement utile quand l’étiqueteur affiche une grille de 4, 6 ou 9 images. 

Une fois qu’un modèle Machine Learning a été entraîné sur vos données étiquetées manuellement, le modèle est tronqué à sa dernière couche entièrement connectée. Les images non étiquetées transitent ensuite par le modèle tronqué dans un processus appelé « incorporation » ou « featurization ». Chaque image est ainsi incorporée dans un espace hautement dimensionnel défini par cette couche de modèle. Les images qui correspondent aux voisins les plus proches dans l’espace sont utilisées pour les tâches de clustering. 

La phase de clustering n’apparaît pas pour les modèles de détection d’objets.

### <a name="prelabeling"></a>Préétiquetage

Une fois qu’un nombre suffisant d’étiquettes d’images a été envoyé, un modèle de classification est utilisé pour prédire les balises d’images. Ou un modèle de détection d’objets est utilisé pour prédire les cadres englobants. L’étiqueteur voit dès lors les pages qui contiennent les étiquettes prédites déjà présentes dans chaque image. Pour la détection d’objets, des zones prédites sont également affichées. La tâche doit ensuite examiner ces prédictions et corriger les images mal étiquetées avant d’envoyer la page.  

Une fois qu’un modèle Machine Learning a été entraîné sur vos données étiquetées manuellement, le modèle est évalué sur un ensemble d’images de test étiquetées manuellement pour déterminer sa précision à plusieurs seuils de confiance. Ce processus d’évaluation sert à déterminer le seuil de confiance au-dessus duquel le modèle est suffisamment précis pour afficher des préétiquettes. Le modèle est ensuite évalué par rapport aux données non étiquetées. Les images dont les prédictions ont un niveau de confiance supérieur à ce seuil sont utilisées pour le préétiquetage.

## <a name="initialize-the-labeling-project"></a>Initialiser le projet d’étiquetage

Une fois le projet d’étiquetage initialisé, certains de ses aspects sont non modifiables. Vous ne pouvez pas changer le type de tâche ou le jeu de données. Vous *pouvez* modifier les étiquettes et l’URL de la description de la tâche. Passez en revue attentivement les paramètres avant de créer le projet. Une fois le projet envoyé, vous êtes redirigé vers la page d’accueil **Étiquetage des données**, qui indique **Initialisation** comme état du projet.

> [!NOTE]
> Cette page peut ne pas s’actualiser automatiquement. Après une pause, actualisez donc manuellement la page pour que l’état du projet indique **Créé**.

## <a name="run-and-monitor-the-project"></a>Exécuter et surveiller le projet
Une fois le projet initialisé, Azure commence à l’exécuter. Sélectionnez le projet dans la page **Étiquetage des données** principale pour voir les détails du projet.

Pour mettre en pause ou redémarrer le projet, changez l’état **En cours d’exécution** en haut à droite. Vous pouvez uniquement étiqueter des données quand le projet est en cours d’exécution.

### <a name="dashboard"></a>tableau de bord

L’onglet **Tableau de bord** affiche la progression de la tâche d’étiquetage.

:::image type="content" source="media/how-to-create-labeling-projects/labeling-dashboard.png" alt-text="Tableau de bord d’étiquetage des données":::

Le graphique de progression indique le nombre d’éléments qui ont été étiquetés et le nombre d’éléments en attente d’étiquetage.  Il se peut que les éléments en attente :

* N’aient pas encore été ajoutés à une tâche
* Aient été inclus dans une tâche qui a été assignée à un étiqueteur mais qui n’est pas encore terminée 
* Soient dans la file d’attente des tâches à affecter

La section centrale affiche la file d’attente des tâches à affecter. Lorsque l’étiquetage assisté par ML est désactivé, cette section indique le nombre de tâches manuelles à affecter. Lorsque l’étiquetage assisté par ML est activé, les éléments suivants s’affichent également :

* Tâches contenant des éléments en cluster dans la file d’attente
* Tâches contenant des éléments pré-étiquetés dans la file d’attente

En outre, lorsque l’étiquetage assisté par ML est activé, une petite barre de progression s’affiche lors de la prochaine exécution de l’apprentissage.  Les sections Expériences fournissent des liens pour chaque exécution Machine Learning.

* Apprentissage : effectue l’apprentissage d’un modèle pour prédire les étiquettes
* Validation : détermine si la prédiction de ce modèle est utilisée pour l’étiquetage préalable des éléments 
* Inférence : exécution de prédiction pour les nouveaux éléments
* Caractérisation : éléments de clusters (uniquement pour les projets de classification d’images)

Dans la partie droite se trouve une distribution des étiquettes pour les tâches qui sont terminées.  Notez que, dans certains types de projets, un élément peut avoir plusieurs étiquettes, auquel cas le nombre total d’étiquettes peut être supérieur au nombre total d’éléments.

### <a name="data-tab"></a>Onglet Données

Sous l’onglet **Données**, vous pouvez voir votre jeu de données et passer en revue les données étiquetées. Si vous voyez des données étiquetées de manière incorrecte, sélectionnez-les, puis choisissez **Rejeter**. Les étiquettes sont alors supprimées et les données replacées en file d’attente des données sans étiquette.

### <a name="details-tab"></a>Onglet Détails

Affichez les détails de votre projet.  Dans cet onglet, vous pouvez :

* Afficher les détails du projet et les jeux de données d’entrée
* Activer l’actualisation incrémentielle
* Afficher les détails du conteneur de stockage utilisé pour stocker les sorties étiquetées dans votre projet
* Ajouter des étiquettes à votre projet
* Modifier les instructions que vous donnez à vos étiquettes
* Modifier les détails de l’étiquetage assisté par ML, notamment l’activation/la désactivation

### <a name="access-for-labelers"></a>Accès pour les étiqueteurs

Toute personne ayant accès à votre espace de travail peut étiqueter les données de votre projet.  Vous pouvez également personnaliser les autorisations associées à vos étiquettes afin que l’utilisateur puisse accéder aux étiquettes, mais pas à d’autres parties de l’espace de travail ou à votre projet d’étiquetage.  Pour plus d’informations, consultez [Gérer l’accès à un espace de travail Azure Machine Learning](how-to-assign-roles.md) et découvrez comment créer le [rôle personnalisé Étiqueteur](how-to-assign-roles.md#labeler).

## <a name="add-new-label-class-to-a-project"></a>Ajouter une nouvelle classe d’étiquette à un projet

Durant le processus d’étiquetage, vous pouvez constater que des étiquettes supplémentaires sont nécessaires pour classifier vos images.  Par exemple, vous pouvez ajouter une étiquette « inconnu » ou « autre » pour indiquer des images pouvant prêter à confusion.

Pour ajouter une ou plusieurs étiquettes à un projet, effectuez les étapes suivantes :

1. Sélectionnez le projet dans la page **Étiquetage des données** principale.
1. En haut à droite de la page, basculez **En cours d’exécution** sur **Mis en pause** pour arrêter les étiqueteurs dans leur activité.
1. Sélectionnez l’onglet **Détails**.
1. Dans la liste à gauche, sélectionnez **Classes d’étiquette**.
1. En haut de la liste, sélectionnez **+ Ajouter des étiquettes** ![Ajouter une étiquette](media/how-to-create-labeling-projects/add-label.png).
1. Dans le formulaire, ajoutez votre nouvelle étiquette et choisissez comment procéder.  Étant donné que vous avez modifié les étiquettes disponibles pour une image, choisissez comment traiter les données déjà étiquetées :
    * Recommencer en supprimant toutes les étiquettes existantes.  Choisissez cette option si vous souhaitez commencer l’étiquetage depuis le début avec le nouvel ensemble complet d’étiquettes. 
    * Recommencer en conservant toutes les étiquettes existantes.  Choisissez cette option pour marquer toutes les données comme non étiquetées, mais en conservant les étiquettes existantes comme étiquettes par défaut pour les images précédemment étiquetées.
    * Continuer en conservant toutes les étiquettes existantes. Choisissez cette option pour laisser toutes les données déjà étiquetées telles quelles et commencer à utiliser la nouvelle étiquette pour les données qui ne sont pas encore étiquetées.
1. Modifiez votre page d’instructions au besoin pour la ou les nouvelles étiquettes.
1. Une fois que vous avez ajouté toutes les nouvelles étiquettes, en haut à droite de la page, basculez **Mis en pause** sur **En cours d’exécution** pour redémarrer le projet.  

## <a name="export-the-labels"></a>Exporter les étiquettes

Vous pouvez exporter les données d’étiquette pour Machine Learning - Expérimentation à tout moment. Vous pouvez exporter les étiquettes d’image au [format COCO](http://cocodataset.org/#format-data) ou en tant que [jeu de données Azure Machine Learning avec des étiquettes](how-to-use-labeled-dataset.md). Utilisez le bouton **Exporter** de la page **Détails du projet** de votre projet d’étiquetage.

Le fichier COCO est créé dans le magasin d’objets blob par défaut de l’espace de travail Azure Machine Learning, dans un dossier situé au sein de *export/coco*. Vous pouvez accéder au jeu de données Azure Machine Learning exporté dans la section **Jeux de données** de Machine Learning. La page de détails du jeu de données fournit également un exemple de code pour accéder aux étiquettes à partir de Python.

![Jeu de données exporté](./media/how-to-create-labeling-projects/exported-dataset.png)

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Créer votre premier projet d’étiquetage de classification d’images](tutorial-labeling.md).
* Étiqueter des images pour [la classification d’images ou la détection d’objets](how-to-label-images.md)
* En savoir plus sur [Azure Machine Learning et Machine Learning Studio (classique)](./overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)
