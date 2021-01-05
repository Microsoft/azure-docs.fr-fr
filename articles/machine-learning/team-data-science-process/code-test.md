---
title: Tester du code de science des données avec Azure DevOps Services - Team Data Science Process
description: Test de code de science des données dans Azure avec le jeu de données de prédiction des revenus des adultes UCI, avec Team Data Science Process et Azure DevOps Services
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=weig, previous-ms.author=weig
ms.openlocfilehash: ce37ee51e549a2bd6f2747f28af8c038c39d8f1f
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656833"
---
# <a name="data-science-code-testing-on-azure-with-the-team-data-science-process-and-azure-devops-services"></a>Test de code de science des données dans Azure avec Team Data Science Process et Azure DevOps Services
Cet article donne des recommandations préliminaires pour tester les codes dans un flux de travail de science des données. Ce type de test permet aux chercheurs de données de vérifier leur code systématiquement et efficacement pour en garantir la qualité et le résultat. Nous nous basons sur un projet TDSP (Team Data Science Process) [qui utilise le jeu de données UCI Adult Income](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome) que nous avons publié précédemment afin de montrer comment tester les codes. 

## <a name="introduction-on-code-testing"></a>Présentation des tests de code
L’« unité de test » constitue une pratique de longue date dans le domaine du développement logiciel. Toutefois, dans le domaine de la science des données, il est souvent difficile de savoir ce que signifie « unité de test » et comment tester le code au cours des différentes étapes d’un cycle de vie de la science des données, par exemple :

* Préparation des données
* Examen de la qualité des données
* Modélisation
* Déploiement de modèle 

Cet article remplace le terme « test unitaire » par « test de code ». Il fait référence aux tests comme étant les fonctions qui aident à évaluer si le code, à une certaine étape d’un cycle de vie de science des données, produit les résultats « attendus ». La personne qui écrit le test définit ce qu’elle attend du résultat de la fonction, par exemple, la vérification ou la modélisation de la qualité des données.

Cet article contient des ressources utiles qui serviront de référence.

## <a name="azure-devops-for-the-testing-framework"></a>Azure DevOps pour l’infrastructure de test
Cet article décrit comment exécuter et automatiser les tests à l’aide d’Azure DevOps. Vous pouvez tout à fait utiliser d’autres outils. Nous montrons également comment configurer une build automatique à l’aide d’Azure DevOps et d’agents de build. Pour les agents de build, nous utilisons des machines virtuelles Azure Data Science (DSVM).

## <a name="flow-of-code-testing"></a>Flux du test de code
Le flux de travail global d’un test de code dans un projet de science des données prend la forme suivante : 

![Diagramme du flux de test de code](./media/code-test/test-flow-chart.PNG)

    
## <a name="detailed-steps"></a>Procédure détaillée

Utilisez les étapes suivantes pour configurer et exécuter un test de code et une build automatique à l’aide d’un agent de build et d’Azure DevOps :

1. Créez un projet dans l’application de bureau Visual Studio :

    ![Écran de création d’un projet dans Visual Studio](./media/code-test/create_project.PNG)

   Une fois créé, le projet apparaît dans l’Explorateur de solutions, dans le volet de droite :
    
    ![Étapes de création d’un projet](./media/code-test/create_python_project_in_vs.PNG)

    ![Explorateur de solutions](./media/code-test/solution_explorer_in_vs.PNG)

1. Insérez le code de votre projet dans le référentiel de code de projet Azure DevOps : 

    ![Référentiel de code de projet](./media/code-test/create_repo.PNG)

1. Imaginons que vous avez effectué un travail de préparation des données : ingestion des données, ingénierie des fonctionnalités et création des colonnes d’étiquettes. Vous souhaitez vous assurer que votre code va générer les résultats escomptés. Voici un code que vous pouvez utiliser pour tester si le code de traitement des données fonctionne correctement :

    * Vérifiez que les noms de colonne sont exacts :
    
      ![Code pour la correspondance des noms de colonnes](./media/code-test/check_column_names.PNG)

    * Vérifiez que les niveaux de réponse sont exacts :

      ![Code pour la correspondance des niveaux](./media/code-test/check_response_levels.PNG)

    * Vérifiez que le pourcentage de réponses est raisonnable :

      ![Code pour le pourcentage de réponses](./media/code-test/check_response_percentage.PNG)

    * Vérifiez le taux de réponses manquantes de chaque colonne dans les données :
    
      ![Code pour le taux de réponses manquantes](./media/code-test/check_missing_rate.PNG)


1. Une fois que vous avez terminé le traitement des données et le travail d’ingénierie des fonctionnalités et que vous avez conçu un modèle adapté, assurez-vous que celui-ci peut évaluer correctement les nouveaux jeux de données. Vous pouvez utiliser les deux tests suivants pour vérifier les niveaux de prédiction et la distribution des valeurs d’étiquette :

    * Vérifiez les niveaux de prédiction :
    
      ![Code pour la vérification des niveaux de prédiction](./media/code-test/check_prediction_levels.PNG)

    * Vérifiez la distribution des valeurs de prédiction :

      ![Code pour la vérification des valeurs de prédiction](./media/code-test/check_prediction_values.PNG)

1. Placez toutes les fonctions de test dans un script Python nommé **test_funcs.py** :

    ![Script Python pour les fonctions de test](./media/code-test/create_file_test_func.PNG)


1. Dès que les codes de test sont prêts, vous pouvez configurer l’environnement de test dans Visual Studio.

   Créez un fichier Python appelé **test1.py**. Dans ce fichier, créez une classe qui inclut tous les tests à effectuer. L’exemple suivant montre la préparation de six tests :
    
    ![Fichier Python contenant une liste de tests dans une classe](./media/code-test/create_file_test1_class.PNG)

1. Ces tests peuvent être détectés automatiquement si vous placez **codetest.testCase** après le nom de la classe. Ouvrez l’Explorateur de tests dans le volet de droite, puis sélectionnez **Run All** (Tout exécuter). L’exécution des tests s’effectue de manière séquentielle et un message vous indique s’ils ont ou non abouti.

    ![Exécution des tests](./media/code-test/run_tests.PNG)

1. Archivez votre code dans le référentiel de projet à l’aide des commandes Git. Votre travail le plus récent apparaîtra sous peu dans Azure DevOps.

    ![Commandes Git pour archiver le code](./media/code-test/git_check_in.PNG)

    ![Travail le plus récent dans Azure DevOps](./media/code-test/git_check_in_most_recent_work.PNG)

1. Configurez la build automatique et le test dans Azure DevOps :

    a. Dans le référentiel du projet, sélectionnez **Build and Release** (Générer et libérer), puis **+New** (+Nouveau) pour créer un nouveau processus de build.

    ![Sélections pour démarrer un nouveau processus de build](./media/code-test/create_new_build.PNG)

    b. Suivez les invites pour sélectionner l’emplacement du code source, le nom du projet, le référentiel et les informations de branche.
    
    ![Source, nom, référentiel et informations de branche](./media/code-test/fill_in_build_info.PNG)

    c. Sélectionnez un modèle. Étant donné qu’il n’existe aucun modèle de projet Python, commencez par sélectionner **Processus vide**. 

    ![Liste des modèles et bouton Processus vide](./media/code-test/start_empty_process_template.PNG)

    d. Donnez un nom à la build et sélectionnez l’agent. Vous pouvez choisir ici la valeur par défaut si vous souhaitez utiliser une machine DSVM et terminer le processus de génération. Pour plus d’informations sur la configuration des agents, voir [Créer et libérer des agents](/azure/devops/pipelines/agents/agents?view=vsts).
    
    ![Sélections de la build et de l’agent](./media/code-test/select_agent.PNG)

    e. Sélectionnez **+** dans le volet de gauche afin d’ajouter une tâche pour cette phase de génération. Étant donné que nous allons exécuter le script Python **test1.py** pour terminer toutes les vérifications, cette tâche utilise une commande PowerShell pour exécuter le code Python.
    
    ![Volet permettant d’ajouter des tâches avec PowerShell sélectionné](./media/code-test/add_task_powershell.PNG)

    f. Dans les détails de PowerShell, renseignez les informations requises, notamment le nom et la version de PowerShell. Choisissez **Script Inline** comme type. 
    
    Dans la zone sous **Script Inline**, saisissez **python test1.py**. Assurez-vous que la variable d’environnement est correctement configurée pour Python. Si vous avez besoin d’un noyau ou d’une version différente de Python, vous pouvez spécifier explicitement le chemin d’accès, comme indiqué dans l’illustration : 
    
    ![Détails de PowerShell](./media/code-test/powershell_scripts.PNG)

    g. Sélectionnez **Save & Queue** (Enregistrer et mettre en file d’attente) pour terminer le processus de pipeline de build.

    ![Bouton Save & queue (Enregistrer et mettre en file d’attente)](./media/code-test/save_and_queue_build_definition.PNG)

Désormais, à chaque fois qu’une nouvelle validation sera transmise au référentiel du code, le processus de génération démarrera automatiquement. Vous pouvez définir n’importe quelle branche. Le processus exécute le fichier **test1.py** sur l’ordinateur agent, l’objectif étant de s’assurer que tous les éléments définis dans le code s’exécutent correctement. 

Si les alertes sont correctement définies, vous serez averti par courrier électronique une fois la build terminée. Vous pouvez également vérifier l’état de la build dans Azure DevOps. En cas d’échec, vous pouvez vérifier les détails de la build et déterminer quelle partie est défaillante.

![Notification par courrier électronique de la réussite de la build](./media/code-test/email_build_succeed.PNG)

![Notification dans Azure DevOps de la réussite de la build](./media/code-test/vs_online_build_succeed.PNG)

## <a name="next-steps"></a>Étapes suivantes
* Consultez le [référentiel UCI Income Prediction](https://github.com/Azure/MachineLearningSamples-TDSPUCIAdultIncome) pour obtenir des exemples concrets de tests unitaires pour les scénarios de science des données.
* Appuyez-vous sur le plan et les exemples fournis précédemment dans le scénario UCI Income Prediction pour vos propres projets de science des données.

## <a name="references"></a>References
* [Processus Team Data Science Process](./index.yml)
* [Outils de test Visual Studio](https://www.visualstudio.com/vs/features/testing-tools/)
* [Ressources de test Azure DevOps](https://www.visualstudio.com/team-services/)
* [Machines virtuelles Data Science](https://azure.microsoft.com/services/virtual-machines/data-science-virtual-machines/)