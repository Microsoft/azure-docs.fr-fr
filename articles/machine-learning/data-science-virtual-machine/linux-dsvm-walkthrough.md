---
title: Explorer Linux
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment effectuer plusieurs tâches courantes de science des données à l’aide de la Data Science Virtual Machine Linux.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 09/17/2020
ms.openlocfilehash: b0ee83cdf3ea710974ce6985db3fc8ce8782351c
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93315791"
---
# <a name="data-science-with-an-ubuntu-data-science-virtual-machine-in-azure"></a>Science des données avec une image Data Science Virtual Machine Ubuntu sur Azure

Cette procédure pas à pas vous montre comment effectuer plusieurs tâches courantes de science des données à l’aide de la Data Science Virtual Machine (DSVM) Ubuntu. La Data Science VM Ubuntu est une image de machine virtuelle disponible sur Azure qui est préinstallée avec plusieurs outils couramment utilisés dans le cadre de l’analyse de données et du Machine Learning. Les composants logiciels clés sont détaillés dans [Approvisionnement d’une machine virtuelle de science des données Ubuntu](./dsvm-ubuntu-intro.md). L’image de la Data Science Virtual Machine facilite la prise en main de la science des données en quelques minutes, sans avoir à installer et à configurer individuellement chacun des outils individuellement. Vous pouvez facilement mettre à l’échelle la DSVM si nécessaire, et vous pouvez l’arrêter lorsqu’elle n’est pas utilisée. Cette DSVM est donc flexible et économique.

Au cours de cette procédure pas à pas, nous analysons le jeu de données [spambase](https://archive.ics.uci.edu/ml/datasets/spambase). Spambase est un ensemble d’e-mails marqués comme courrier indésirable ou normal (n’est pas considéré comme courrier indésirable). Spambase contient également des statistiques sur le contenu des e-mails. Nous parlerons des statistiques plus loin dans la procédure pas à pas.

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir utiliser une DSVM Linux, vous devez posséder les composants requis suivants :

* **Abonnement Azure**. pour obtenir un abonnement Azure, consultez [Créer votre compte Azure gratuit aujourd'hui](https://azure.microsoft.com/free/).

* [**Instance Ubuntu de Data Science Virtual Machine**](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804). Pour plus d’informations sur l’approvisionnement de cette machine virtuelle, consultez [Approvisionnement d’une machine virtuelle de science des données Ubuntu](./release-notes.md).
* [**X2Go**](https://wiki.x2go.org/doku.php) installé sur votre ordinateur et une session XFCE ouverte. Pour plus d’informations, consultez [Installer et configurer le client X2Go](dsvm-ubuntu-intro.md#x2go).
* Pour une expérience de défilement plus fluide, dans le navigateur web Firefox de la DSVM, basculez l'indicateur `gfx.xrender.enabled`dans `about:config`. [Plus d’informations](https://www.reddit.com/r/firefox/comments/4nfmvp/ff_47_unbearable_slow_over_remote_x11/) Envisagez également de régler `mousewheel.enable_pixel_scrolling` sur `False`. [Plus d’informations](https://support.mozilla.org/questions/981140)

## <a name="download-the-spambase-dataset"></a>Télécharger le jeu de données spambase

Le jeu de données [spambase](https://archive.ics.uci.edu/ml/datasets/spambase) est un ensemble de données relativement réduit qui ne contient que 4 601 exemples. Le jeu de données est une taille pratique pour illustrer certaines des fonctionnalités clés de la DSVM, car il a des besoins en ressources modestes.

> [!NOTE]
> Cette procédure pas à pas a été créée à l’aide d’une DSVM Linux de taille D2 v2 (édition Ubuntu 18.04). Vous pouvez utiliser une DSVM cette taille pour effectuer les procédures décrites dans cette procédure pas à pas.

Si vous avez besoin de plus d’espace de stockage, vous pouvez créer des disques supplémentaires et les joindre à votre DSVM. Les disques utilisent le stockage Azure persistant ; ainsi, leurs données sont conservées même si le serveur est réapprovisionné en raison d’un redimensionnement ou d’un arrêt. Pour ajouter un disque et l’attacher à votre DSVM, suivez les étapes de la section [Ajouter un disque à une machine virtuelle Linux](../../virtual-machines/linux/add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Les étapes d’ajout d’un disque utilisent Azure CLI, qui est déjà installée sur la DSVM. Vous pouvez effectuer l’intégralité des étapes à partir de la DSVM elle-même. Une autre option pour augmenter le stockage consiste à utiliser [Azure Files](../../storage/files/storage-how-to-use-files-linux.md).

Pour télécharger les données, ouvrez une fenêtre de terminal et exécutez cette commande :

```bash
wget --no-check-certificate https://archive.ics.uci.edu/ml/machine-learning-databases/spambase/spambase.data
```

Le fichier téléchargé n’a pas de ligne d’en-tête. Nous allons créer un autre fichier qui possède un en-tête. Exécutez cette commande pour créer un fichier avec les en-têtes appropriés :

```bash
echo 'word_freq_make, word_freq_address, word_freq_all, word_freq_3d,word_freq_our, word_freq_over, word_freq_remove, word_freq_internet,word_freq_order, word_freq_mail, word_freq_receive, word_freq_will,word_freq_people, word_freq_report, word_freq_addresses, word_freq_free,word_freq_business, word_freq_email, word_freq_you, word_freq_credit,word_freq_your, word_freq_font, word_freq_000, word_freq_money,word_freq_hp, word_freq_hpl, word_freq_george, word_freq_650, word_freq_lab,word_freq_labs, word_freq_telnet, word_freq_857, word_freq_data,word_freq_415, word_freq_85, word_freq_technology, word_freq_1999,word_freq_parts, word_freq_pm, word_freq_direct, word_freq_cs, word_freq_meeting,word_freq_original, word_freq_project, word_freq_re, word_freq_edu,word_freq_table, word_freq_conference, char_freq_semicolon, char_freq_leftParen,char_freq_leftBracket, char_freq_exclamation, char_freq_dollar, char_freq_pound, capital_run_length_average,capital_run_length_longest, capital_run_length_total, spam' > headers
```

Ensuite, concaténez les deux fichiers :

```bash
cat spambase.data >> headers
mv headers spambaseHeaders.data
```

Le jeu de données possède plusieurs types de statistiques pour chaque e-mail :

* Les colonnes comme **word\_freq\__MOT_** indiquent le pourcentage de mots dans l’e-mail correspondant à *MOT*. Par exemple, si la valeur **word\_freq\_make** correspond à **1** , 1 % de tous les mots dans l’e-mail était *make*.
* Les colonnes comme **char\_freq\__CAR_** indiquent le pourcentage de caractères dans l’e-mail correspondant à *CAR*.
* **capital\_run\_length\_longest** est la longueur la plus longue d’une séquence de lettres majuscules.
* **capital\_run\_length\_average** est la longueur moyenne de toutes les séquences de lettres majuscules.
* **capital\_run\_length\_total** est la longueur totale de toutes les séquences de lettres majuscules.
* **spam** indique si l’e-mail a été considéré comme du courrier indésirable ou non (1 = courrier indésirable, 0 = courrier non indésirable).

## <a name="explore-the-dataset-by-using-r-open"></a>Explorer le jeu de données à l’aide de R Open

Nous allons examiner les données et découvrir certaines fonctionnalités de base du Machine Learning avec R. La Data Science VM est fournie avec [Microsoft R Open](https://mran.revolutionanalytics.com/open/) préinstallé. Les bibliothèques mathématiques multithread dans cette version préinstallée de R offrent de meilleures performances que les versions monothread. R Open fournit également la reproductibilité à l’aide d’une capture instantanée du référentiel du package CRAN.

Pour récupérer des copies des exemples de code utilisés dans cette procédure pas à pas, utilisez git pour cloner le référentiel Azure-Machine-Learning-Data-Science. Git est préinstallé sur la DSVM. Depuis la ligne de commande git, exécutez :

```bash
git clone https://github.com/Azure/Azure-MachineLearning-DataScience.git
```

Ouvrez une fenêtre de terminal et démarrez une nouvelle session R dans la console interactive R. Vous pouvez également utiliser RStudio, qui est préinstallé sur la DSVM.

Pour importer les données et configurer l’environnement :

```R
data <- read.csv("spambaseHeaders.data")
set.seed(123)
```

Pour afficher un résumé des statistiques de chaque colonne :

```R
summary(data)
```

Pour une vue différente des données :

```R
str(data)
```

Dans cette vue, vous voyez le type de chaque variable et les premières valeurs dans le jeu de données.

La colonne **spam** a été lue comme un entier, mais il s’agit en fait d’une variable par catégorie (ou facteur). Pour définir son type :

```R
data$spam <- as.factor(data$spam)
```

Pour effectuer une analyse exploratoire, utilisez le package [ggplot2](https://ggplot2.tidyverse.org/) , une bibliothèque de graphiques populaire pour R qui est préinstallée sur la Data Science VM. À partir des données récapitulatives affichées précédemment, nous disposons de statistiques résumées sur la fréquence du point d’exclamation. Nous allons tracer ces fréquences ici en exécutant les commandes suivantes :

```R
library(ggplot2)
ggplot(data) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Étant donné que la barre de zéro décale le tracé, nous allons l’éliminer :

```R
email_with_exclamation = data[data$char_freq_exclamation > 0, ]
ggplot(email_with_exclamation) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Il existe une densité non triviale au-dessus de 1 qui semble intéressante. Examinons uniquement ces données :

```R
ggplot(data[data$char_freq_exclamation > 1, ]) + geom_histogram(aes(x=char_freq_exclamation), binwidth=0.25)
```

Ensuite, répartissez-les en courrier indésirable et courrier légitime :

```R
ggplot(data[data$char_freq_exclamation > 1, ], aes(x=char_freq_exclamation)) +
geom_density(lty=3) +
geom_density(aes(fill=spam, colour=spam), alpha=0.55) +
xlab("spam") +
ggtitle("Distribution of spam \nby frequency of !") +
labs(fill="spam", y="Density")
```

Ces exemples devraient vous aider à effectuer des tracés similaires et à explorer les données dans les autres colonnes.

## <a name="train-and-test-a-machine-learning-model"></a>Apprentissage et test d’un modèle de Machine Learning

Nous allons effectuer l’apprentissage de quelques modèles de Machine Learning pour classer les e-mails dans le jeu de données comme courrier indésirable ou courrier légitime. Dans cette section, nous allons former un modèle d’arbre de décision et un modèle de forêt aléatoire. Ensuite, nous allons tester la précision des prédictions.

> [!NOTE]
> Le package *rpart* (partition récursive et arbres de régression) utilisé dans le code suivant est déjà installé sur la Data Science VM.

Nous allons commencer par diviser le jeu de données en jeu d’apprentissage et jeu de test :

```R
rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)
```

Ensuite, créez un arbre de décision pour classer les e-mails :

```R
require(rpart)
model.rpart <- rpart(spam ~ ., method = "class", data = trainSet)
plot(model.rpart)
text(model.rpart)
```

Voici le résultat :

![Un diagramme de l’arbre de décision créé](./media/linux-dsvm-walkthrough/decision-tree.png)

Pour déterminer la qualité d’exécution sur l’ensemble d’apprentissage, utilisez le code suivant :

```R
trainSetPred <- predict(model.rpart, newdata = trainSet, type = "class")
t <- table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Pour déterminer la qualité d’exécution sur le jeu de test :

```R
testSetPred <- predict(model.rpart, newdata = testSet, type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

Essayons également un modèle de forêts aléatoires. Les forêts aléatoires effectuent l’apprentissage d’une multitude d’arbres de décision et génèrent une classe qui constitue le mode de classification de tous les arbres de décision individuels. Elles fournissent une approche de Machine Learning plus puissante, car elles corrigent la tendance d’un modèle d’arbre de décision à dépasser un jeu de données d’apprentissage.

```R
require(randomForest)
trainVars <- setdiff(colnames(data), 'spam')
model.rf <- randomForest(x=trainSet[, trainVars], y=trainSet$spam)

trainSetPred <- predict(model.rf, newdata = trainSet[, trainVars], type = "class")
table(`Actual Class` = trainSet$spam, `Predicted Class` = trainSetPred)

testSetPred <- predict(model.rf, newdata = testSet[, trainVars], type = "class")
t <- table(`Actual Class` = testSet$spam, `Predicted Class` = testSetPred)
accuracy <- sum(diag(t))/sum(t)
accuracy
```

<a name="deep-learning"></a>

## <a name="deep-learning-tutorials-and-walkthroughs"></a>Didacticiels et procédures pas à pas de l’apprentissage profond

Outre les exemples basés sur les frameworks, un ensemble de procédures détaillées complètes est également fourni. Ces procédures vous permettent de vous lancer dans le développement d’applications d’apprentissage profond dans des domaines tels que la reconnaissance vocale ou bien la compréhension d’image et de texte.

- [Exécution de réseaux neuronaux sur différents frameworks](https://github.com/ilkarman/DeepLearningFrameworks) : Une procédure pas à pas complète qui vous montre comment migrer du code depuis un framework vers un autre. Elle montre également comment comparer les performances des modèles et du runtime entre les frameworks. 

- [Guide pratique pour créer une solution de bout en bout afin de détecter des produits dans des images](https://github.com/Azure/cortana-intelligence-product-detection-from-images) : la détection d’image est une technique qui peut localiser des objets dans les images et les classer. Les retombées concrètes de cette technologie semblent très prometteuses dans de nombreux domaines professionnels. Par exemple, un détaillant peut utiliser cette technique pour déterminer quel produit un client a choisi dans un rayon. Il peut ensuite utiliser ces informations pour gérer le stock des produits. 

- [Deep learning pour l’audio](/archive/blogs/machinelearning/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure) : ce tutoriel montre comment entraîner un modèle de deep learning pour la détection d’événements audio sur le [jeu de données de sons urbains](https://urbansounddataset.weebly.com/). Ce didacticiel fournit également une vue d’ensemble de l’utilisation des données audio.

- [Classification de documents texte](https://github.com/anargyri/lstm_han) : cette procédure pas à pas montre comment créer et effectuer l'apprentissage de deux architectures de réseau neuronal différentes : Réseau d’attention hiérarchisée à mémoire à long ou court terme. Ces réseaux neuronaux utilisent l’API Keras de l’apprentissage profond pour classer les documents texte. Keras facilite l’utilisation de trois des frameworks d’apprentissage profond les plus populaires, à savoir Microsoft Cognitive Toolkit, TensorFlow et Theano.

## <a name="other-tools"></a>Autres outils

Les sections restantes vous montrent comment utiliser certains des outils installés sur la DSVM Linux. Nous abordons ces outils :

* XGBoost
* Python
* JupyterHub
* Rattle
* PostgreSQL et SQuirreL SQL
* Azure Synapse Analytics (anciennement SQL DW)

### <a name="xgboost"></a>XGBoost

[XGBoost](https://xgboost.readthedocs.org/en/latest/) offre une implémentation rapide et précise des arborescences optimisées.

```R
require(xgboost)
data <- read.csv("spambaseHeaders.data")
set.seed(123)

rnd <- runif(dim(data)[1])
trainSet = subset(data, rnd <= 0.7)
testSet = subset(data, rnd > 0.7)

bst <- xgboost(data = data.matrix(trainSet[,0:57]), label = trainSet$spam, nthread = 2, nrounds = 2, objective = "binary:logistic")

pred <- predict(bst, data.matrix(testSet[, 0:57]))
accuracy <- 1.0 - mean(as.numeric(pred > 0.5) != testSet$spam)
print(paste("test accuracy = ", accuracy))
```

XGBoost peut également appeler à partir de Python ou d’une ligne de commande.

### <a name="python"></a>Python

Pour le développement Python, les distributions Python Anaconda 3.5 et 2.7 sont installées sur la DSVM.

> [!NOTE]
> La distribution Anaconda comprend [Conda](https://conda.pydata.org/docs/index.html). Vous pouvez utiliser Conda pour créer des environnements Python personnalisés sur lesquels des versions ou des packages différents sont installés.

Nous allons lire le jeu de données spambase et classer les e-mails avec des machines à vecteurs de support dans Scikit-learn :

```Python
import pandas
from sklearn import svm
data = pandas.read_csv("spambaseHeaders.data", sep = ',\s*')
X = data.ix[:, 0:57]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

Pour effectuer des prédictions :

```Python
clf.predict(X.ix[0:20, :])
```

Pour montrer comment publier un point de terminaison Azure Machine Learning, nous allons créer un modèle plus basique. Nous utiliserons les trois variables que nous avons utilisées quand nous avons publié le modèle R plus tôt :

```Python
X = data[["char_freq_dollar", "word_freq_remove", "word_freq_hp"]]
y = data.ix[:, 57]
clf = svm.SVC()
clf.fit(X, y)
```

### <a name="jupyterhub"></a>JupyterHub

La distribution Anaconda dans la machine virtuelle de science des données est fournie avec un Jupyter Notebook, un environnement multiplateforme pour partager Python, R, ou le code et l’analyse Julia. Le serveur Jupyter Notebook est accessible via JupyterHub. Vous vous connectez à https://\<DSVM DNS name or IP address\>:8000/ en utilisant votre nom d’utilisateur et votre mot de passe Linux locaux. Tous les fichiers de configuration pour JupyterHub se trouvent dans /etc/jupyterhub.

> [!NOTE]
> Pour utiliser le gestionnaire de package Python (via la commande `pip`) à partir d’un Jupyter Notebook dans le noyau actif, utilisez cette commande dans la cellule de code :
>
>   ```Python
>    import sys
>    ! {sys.executable} -m pip install numpy -y
>   ```
> 
> Pour utiliser le programme d'installation Conda (via la commande `conda`) à partir d’un Jupyter Notebook dans le noyau actif, utilisez cette commande dans une cellule de code :
>
>   ```Python
>    import sys
>    ! {sys.prefix}/bin/conda install --yes --prefix {sys.prefix} numpy
>   ```

Plusieurs exemples de notebooks sont déjà installés sur la DSVM :

* Exemples de notebooks Python :
  * [IntroToJupyterPython.ipynb](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroToJupyterPython.ipynb)
* Exemple de notebook R :
  * [IntroTutorialinR](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Data-Science-Virtual-Machine/Samples/Notebooks/IntroTutorialinR.ipynb) 

> [!NOTE]
> Le langage Julia est également disponible à partir de la ligne de commande sur la DSVM Linux.

### <a name="rattle"></a>Rattle

[Rattle](https://cran.r-project.org/web/packages/rattle/index.html) (l’ *o* util d’ *analyse* *R* *p* our *a* pprendre plus *f* acilement) est un outil R graphique pour l’exploration de données. Rattle possède une interface intuitive qui facilite la charge, l’exploration et la transformation des données, ainsi que la création et l’évaluation des modèles. [Rattle : une interface utilisateur graphique pour l’exploration de données pour R](https://journal.r-project.org/archive/2009-2/RJournal_2009-2_Williams.pdf) fournit une procédure pas à pas présentant les fonctionnalités de Rattle.

Installez et démarrez Rattle en exécutant les commandes suivantes :

```R
if(!require("rattle")) install.packages("rattle")
require(rattle)
rattle()
```

> [!NOTE]
> Vous n’avez pas besoin d’installer Rattle sur la DSVM. Toutefois, vous pouvez être invité à installer des packages supplémentaires quand Rattle s’ouvre.

Rattle utilise une interface basée sur des onglets. La plupart des onglets correspondent aux étapes du [Processus de science des données de l’équipe](../team-data-science-process/index.yml), telles que le chargement de données ou leur exploration. Le processus de science des données se déroule de gauche à droite à travers les onglets. Le dernier onglet contient un journal des commandes R exécutées par Rattle.

Pour charger et configurer le jeu de données :

1. Pour charger le fichier, sélectionnez l’onglet **Données**.
1. Choisissez le sélecteur en regard de **Nom de fichier** , puis sélectionnez **spambaseHeaders.data**.
1. Pour charger le fichier, sélectionnez **Exécuter**. Vous devez voir un résumé de chaque colonne, notamment son type de données identifié, qu’il s’agisse d’une entrée, d’une cible ou d’un autre type de variable, ainsi que le nombre de valeurs uniques.
1. Rattle a correctement identifié la colonne **spam** comme étant la cible. Sélectionnez la colonne **Spam** , puis définissez le **Type de données cible** sur **Par catégorie**.

Pour explorer les données :

1. Sélectionnez l’onglet **Explorer** .
1. Pour afficher des informations sur les types de variables et certaines statistiques de résumé, sélectionnez **Résumé** > **Exécuter**.
1. Pour afficher d’autres types de statistiques relatives à chaque variable, sélectionnez d’autres options, comme **Décrire** ou **Concepts de base**.

Vous pouvez également utiliser l'onglet **Explorer** pour générer des tracés pertinents. Pour tracer un histogramme des données :

1. Sélectionnez **Distributions**.
1. Pour **word_freq_remove** et **word_freq_you** , sélectionnez **Histogramme**.
1. Sélectionnez **Exécuter**. Vous devez voir les deux graphiques de densité dans une seule fenêtre de graphique, où il est clair que le mot _you_ (vous) apparaît beaucoup plus fréquemment dans les e-mails que le mot _remove_ (supprimer).

Les tracés de **Corrélation** sont également intéressants. Pour créer un tracé :

1. Pour **Type** , sélectionnez **Corrélation**.
1. Sélectionnez **Exécuter**.
1. Rattle vous avertit qu’il recommande un maximum de 40 variables. Sélectionnez **Oui** pour afficher le tracé.

Des corrélations intéressantes existent : le terme _technologie_ est étroitement corrélé avec les termes _HP_ et _laboratoires_ , par exemple. Il est également étroitement corrélé avec _650_ , car le code de région des donneurs du dataset est 650.

Les valeurs numériques des corrélations entre les mots sont disponibles dans la fenêtre **Explorer**. Il est intéressant de noter, par exemple, que le terme _technologie_ est négativement corrélé avec les termes _votre_ et _argent_.

Rattle peut transformer le jeu de données pour gérer certains problèmes courants. Par exemple, il vous permet de remettre à l’échelle les fonctionnalités, d’imputer les valeurs manquantes, de gérer les valeurs hors-norme et de supprimer des variables ou des observations avec des données manquantes. Rattle peut également identifier des règles d’association entre des observations et des variables. Ces onglets ne sont pas couverts dans cette procédure pas à pas d’introduction.

Rattle peut également exécuter l’analyse du cluster. Nous allons exclure certaines fonctionnalités pour simplifier la lecture de la sortie. Sous l’onglet **Données** , sélectionnez **Ignorer** en regard de chacune des variables à l’exception de ces 10 éléments :

* word_freq_hp
* word_freq_technology
* word_freq_george
* word_freq_remove
* word_freq_your
* word_freq_dollar
* word_freq_money
* capital_run_length_longest
* word_freq_business
* spam

Revenez à l'onglet **Cluster**. Sélectionnez **KMeans** , puis définissez le **Nombre de clusters** sur **4**. Sélectionnez **Exécuter**. Les résultats s’affichent dans la fenêtre de sortie. Un cluster possède une fréquence élevée de _george_ et de _hp_ et est probablement un e-mail professionnel légitime.

Pour créer un modèle Machine Learning d’arbre de décision de base :

1. Sélectionnez l’onglet **Modèle** ,
1. Pour le **Type** , sélectionnez **Arborescence**.
1. Sélectionnez **Exécuter** pour afficher l’arbre sous forme de texte dans la fenêtre de sortie.
1. Sélectionnez le bouton **Dessin** pour afficher une version graphique. L’arbre de décision ressemble à l’arbre obtenu précédemment à l’aide de rpart.

Une des fonctionnalités utiles de Rattle est sa capacité à exécuter plusieurs méthodes Machine Learning et à les évaluer rapidement. Voici la procédure à suivre :

1. Pour **Type** , sélectionnez **Tout**.
1. Sélectionnez **Exécuter**.
1. Une fois l’exécution de Rattle terminée, vous pouvez sélectionner n’importe quelle valeur de **Type** , par exemple **SVM** , et afficher les résultats.
1. Vous pouvez également comparer les performances des modèles sur le jeu de validation à l’aide de l’onglet **Évaluer** . Par exemple, la sélection de l’option **Matrice d’erreur** affiche la matrice de confusion, l’erreur globale et l’erreur de classe moyennée pour chaque modèle sur le jeu de validation. Vous pouvez également tracer des courbes ROC, exécuter des analyses de sensibilité et d’autres types d’évaluations de modèle.

Une fois que vous avez terminé la création de modèles, sélectionnez l’onglet **Journal** pour afficher le code R exécuté par Rattle pendant votre session. Vous pouvez sélectionner le bouton **Exporter** pour l’enregistrer.

> [!NOTE]
> La version actuelle de Rattle contient un bogue. Pour modifier le script ou l’utiliser afin de répéter les étapes ultérieurement, vous devez insérer un symbole dièse **#** devant l’option *Exporter ce journal…* dans le texte du journal.

### <a name="postgresql-and-squirrel-sql"></a>PostgreSQL et SQuirreL SQL

La machine virtuelle de science des données est fournie avec PostgreSQL installé. PostgreSQL est une base de données relationnelle, open source et sophistiquée. Cette section montre comment charger notre jeu de données spam dans PostgreSQL, puis l’interroger.

Avant de pouvoir charger les données, vous devez autoriser l’authentification par mot de passe à partir de l’hôte local. À l’invite de commandes, exécutez :

```Bash
sudo gedit /var/lib/pgsql/data/pg_hba.conf
```

Au bas du fichier de configuration, plusieurs lignes détaillent les connexions autorisées :

```
# "local" is only for Unix domain socket connections:
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
```

Modifiez la ligne **connexions locales IPv4** pour utiliser **md5** au lieu d’ **ident** , afin de permettre la connexion avec un nom d’utilisateur et un mot de passe :

```
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
```

Ensuite, redémarrez le service PostgreSQL :

```Bash
sudo systemctl restart postgresql
```

Pour lancer *psql* , un terminal interactif pour PostgreSQL, en tant qu’utilisateur postgres intégré, exécutez la commande suivante :

```Bash
sudo -u postgres psql
```

Créez un nouveau compte d’utilisateur en utilisant le nom d’utilisateur du compte Linux que vous avez utilisé pour vous connecter. Créez un mot de passe :

```Bash
CREATE USER <username> WITH CREATEDB;
CREATE DATABASE <username>;
ALTER USER <username> password '<password>';
\quit
```

Connectez-vous à psql :

```Bash
psql
```

Importez les données dans une base de données :

```SQL
CREATE DATABASE spam;
\c spam
CREATE TABLE data (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer);
\copy data FROM /home/<username>/spambase.data DELIMITER ',' CSV;
\quit
```

À présent, nous allons explorer les données et exécuter des requêtes à l’aide de SQuirreL SQL, un outil graphique que vous pouvez utiliser pour interagir avec les bases de données via un pilote JDBC.

Pour commencer, dans le menu **Applications** , ouvrez SQuirreL SQL. Pour configurer le pilote :

1. Sélectionnez **Windows** > **Afficher les pilotes**.
1. Cliquez avec le bouton droit sur **PostgreSQL** et sélectionnez **Modifier le pilote**.
1. Sélectionnez **Chemin d’accès de la classe supplémentaire** > **Ajouter**.
1. Pour le **Nom de fichier** , entrez **/usr/share/java/jdbcdrivers/postgresql-9.4.1208.jre6.jar**.
1. Sélectionnez **Ouvrir**.
1. Sélectionnez **Répertorier les pilotes**. Pour **Nom de la classe** , sélectionnez **org.postgresql.Driver** , puis sélectionnez **OK**.

Pour configurer la connexion au serveur local :

1. Sélectionnez **Windows** > **Afficher les alias.**
1. Sélectionnez le bouton **+** pour créer un nouvel alias. Pour le nouveau nom d’alias, entrez **Base de données de courrier indésirable**. 
1. Pour **Pilote** , sélectionnez **PostgreSQL**.
1. Définissez l’URL sur **jdbc:postgresql://localhost/spam**.
1. Entrez votre nom d'utilisateur et votre mot de passe.
1. Sélectionnez **OK**.
1. Pour ouvrir la fenêtre **Connexion** , double-cliquez sur l’alias de la **Base de données de courrier indésirable**.
1. Sélectionnez **Connecter**.

Pour exécuter des requêtes :

1. Sélectionnez l’onglet **SQL** .
1. Dans la zone de requête située en haut de l'onglet **SQL** , entrez une requête de base, par exemple `SELECT * from data;`.
1. Appuyez sur Ctrl+Entrée pour exécuter la requête. Par défaut, SQuirreL SQL renvoie les 100 premières lignes de votre requête.

Il existe de nombreuses requêtes supplémentaires, que vous pouvez exécuter pour explorer ces données. Par exemple, en quoi la fréquence du mot *make* diffère-t-elle entre le courrier indésirable et le courrier légitime ?

```SQL
SELECT avg(word_freq_make), spam from data group by spam;
```

Ou quelles sont les caractéristiques des e-mails qui contiennent souvent le terme *3d* ?

```SQL
SELECT * from data order by word_freq_3d desc;
```

La plupart des e-mails qui ont une occurrence élevée de *3d* sont apparemment du spam. Ces informations peuvent être utiles pour la création d’un modèle prédictif afin de classer les e-mails.

Si vous souhaitiez effectuer du Machine Learning avec des données stockées dans une base de données PostgreSQL, envisagez d’utiliser [MADlib](https://madlib.incubator.apache.org/).

### <a name="azure-synapse-analytics-formerly-sql-dw"></a>Azure Synapse Analytics (anciennement SQL DW)

Azure Synapse Analytics est une base de données de mise à l’échelle basée sur le cloud qui prend en charge le traitement de grands volumes de données relationnelles et non relationnelles. Pour plus d’informations, consultez l’article [Qu’est-ce qu’Azure Synapse Analytics ?](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md).

Pour vous connecter à l’entrepôt de données et créer la table, exécutez la commande suivante depuis une invite de commandes :

```Bash
sqlcmd -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -I
```

À l’invite sqlcmd, exécutez la commande suivante :

```SQL
CREATE TABLE spam (word_freq_make real, word_freq_address real, word_freq_all real, word_freq_3d real,word_freq_our real, word_freq_over real, word_freq_remove real, word_freq_internet real,word_freq_order real, word_freq_mail real, word_freq_receive real, word_freq_will real,word_freq_people real, word_freq_report real, word_freq_addresses real, word_freq_free real,word_freq_business real, word_freq_email real, word_freq_you real, word_freq_credit real,word_freq_your real, word_freq_font real, word_freq_000 real, word_freq_money real,word_freq_hp real, word_freq_hpl real, word_freq_george real, word_freq_650 real, word_freq_lab real,word_freq_labs real, word_freq_telnet real, word_freq_857 real, word_freq_data real,word_freq_415 real, word_freq_85 real, word_freq_technology real, word_freq_1999 real,word_freq_parts real, word_freq_pm real, word_freq_direct real, word_freq_cs real, word_freq_meeting real,word_freq_original real, word_freq_project real, word_freq_re real, word_freq_edu real,word_freq_table real, word_freq_conference real, char_freq_semicolon real, char_freq_leftParen real,char_freq_leftBracket real, char_freq_exclamation real, char_freq_dollar real, char_freq_pound real, capital_run_length_average real, capital_run_length_longest real, capital_run_length_total real, spam integer) WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
GO
```

Copiez les données à l’aide de bcp :

```bash
bcp spam in spambaseHeaders.data -q -c -t  ',' -S <server-name>.database.windows.net -d <database-name> -U <username> -P <password> -F 1 -r "\r\n"
```

> [!NOTE]
> Le fichier téléchargé contient des fins de ligne de style Windows. L’outil bcp attend des fins de ligne de style Unix. Utilisez l’indicateur -r pour l’indiquer à bcp.

Ensuite, interrogez à l’aide de sqlcmd :

```sql
select top 10 spam, char_freq_dollar from spam;
GO
```

Vous pouvez également interroger à l’aide de SQuirreL SQL. Suivez des étapes similaires à PostgreSQL avec le pilote JDBC SQL Server. Le pilote JDBC se trouve dans le dossier /usr/share/java/jdbcdrivers/sqljdbc42.jar.