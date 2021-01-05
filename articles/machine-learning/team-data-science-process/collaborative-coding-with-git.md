---
title: Codage collaboratif avec Git - Team Data Science Process
description: Comment développer le code de projets de science des données en collaboration avec Git selon une planification agile.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: ca24a781f4f3ad5c210813dabbb896de35056ed6
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588707"
---
# <a name="collaborative-coding-with-git"></a>Développement collaboratif avec Git

Cet article explique comment utiliser Git comme framework de développement de code collaborative pour les projets de science des données. L’article explique comment lier du code dans Azure Repos à des éléments de travail de [développement agile](agile-development.md) dans Azure Boards, comment effectuer des revues de code, et comment créer et fusionner des demandes de tirage (pull requests) pour les changements.

## <a name="link-a-work-item-to-an-azure-repos-branch"></a><a name='Linkaworkitemwithagitbranch-1'></a>Lier un élément de travail à une branche Azure Repos 

Azure DevOps offre un moyen pratique de connecter un élément de travail Récit utilisateur ou Tâche Azure Boards avec une branche d’un dépôt Git d’Azure Repos. Vous pouvez lier votre récit utilisateur ou votre tâche directement au code qui y est associé. 

Pour connecter un élément de travail à une nouvelle branche, sélectionnez les points de suspension de **Actions** ( **...** ) en regard de l’élément de travail puis, dans le menu contextuel, faites défiler jusqu’à **Nouvelle branche** et sélectionnez cette option.  

![1](./media/collaborative-coding-with-git/1-sprint-board-view.png)

Dans la boîte de dialogue **Créer une branche**, spécifiez le nom de la nouvelle branche, et le dépôt et la branche Git d’Azure Repos. Le dépôt de base doit se trouver dans le même projet Azure DevOps que l’élément de travail. La branche de base peut être n’importe quelle branche existante. Sélectionnez **Créer une branche**. 

![2](./media/collaborative-coding-with-git/2-create-a-branch.png)

Vous pouvez également créer une branche en utilisant la commande bash Git suivante dans Windows ou Linux :

```bash
git checkout -b <new branch name> <base branch name>

```
Si vous ne spécifiez pas de \<base branch name>, la nouvelle branche est basée sur `main`. 

Pour passer à votre branche de travail, exécutez la commande suivante : 

```bash
git checkout <working branch name>
```

Une fois que vous avez basculé vers la branche de travail, vous pouvez commencer à développer des artefacts de code ou de documentation pour l’élément de travail. L’exécution de `git checkout main` vous ramène à la branche `main`.

Une bonne pratique est de créer une branche Git pour chaque élément de travail Récit utilisateur. Ensuite, pour chaque élément de travail Tâche, vous pouvez créer une branche basée sur la branche Récit utilisateur. Organisez les branches dans une hiérarchie qui correspond à la relation Récit utilisateur-Tâche quand plusieurs personnes travaillent sur des récits utilisateur différents pour le même projet, ou sur des tâches différentes pour le même récit utilisateur. Vous pouvez réduire les conflits en faisant en sorte que chaque membre de l’équipe travaille sur une branche différente, sur du code différent ou sur d’autres artefacts lors du partage d’une branche. 

Le diagramme suivant montre la stratégie de création de branches recommandée pour TDSP. Vous n’aurez peut-être pas besoin d’autant de branches, en particulier si seulement une ou deux personnes travaillent sur un projet, ou si une seule personne est chargée de l’ensemble des tâches d’un récit utilisateur. La séparation de la branche de développement de la branche principale est néanmoins toujours une bonne pratique, qui peut aider à empêcher que la branche de mise en production soit interrompue par des activités de développement. Pour obtenir une description plus complète du modèle des branches Git, consultez [A Successful Git Branching Model](https://nvie.com/posts/a-successful-git-branching-model/).

![3](./media/collaborative-coding-with-git/3-git-branches.png)

Vous pouvez également lier un élément de travail à une branche existante. Dans la page **Détails** d’un élément de travail, sélectionnez **Ajouter un lien**. Sélectionnez ensuite une branche existante à laquelle lier l’élément de travail, puis sélectionnez **OK**. 

![4](./media/collaborative-coding-with-git/4-link-to-an-existing-branch.png)

## <a name="work-on-the-branch-and-commit-changes"></a><a name='WorkonaBranchandCommittheChanges-2'></a>Travailler sur la branche et valider les changements 

Après avoir apporté un changement à votre élément de travail, comme l’ajout d’un fichier de script R à la branche `script` de votre machine locale, vous pouvez valider le changement de votre branche locale dans la branche de travail en amont en utilisant les commandes bash Git suivantes :

```bash
git status
git add .
git commit -m "added an R script file"
git push origin script
```

![5](./media/collaborative-coding-with-git/5-sprint-push-to-branch.png)

## <a name="create-a-pull-request"></a><a name='CreateapullrequestonVSTS-3'></a>Créer une demande de tirage

Après une ou plusieurs validations et notifications push, quand vous êtes prêt à fusionner votre branche de travail actuelle dans sa branche de base, vous pouvez créer et envoyer une *demande de tirage (pull request)* dans Azure Repos. 

Dans la page principale de votre projet Azure DevOps, pointez sur **Dépôts** > **Demandes de tirage (Pull requests)** dans le volet de navigation gauche. Sélectionnez ensuite un des boutons **Nouvelle demande de tirage (pull request)** ou le lien **Créer une demande de tirage**.

![6](./media/collaborative-coding-with-git/6-spring-create-pull-request.png)

Dans l’écran **Nouvelle demande de tirage**, accédez si nécessaire au dépôt Git et à la branche où vous voulez fusionner vos changements. Ajoutez ou modifiez d’éventuelles autres informations de votre choix. Sous **Réviseurs**, ajoutez les noms des réviseurs, puis sélectionnez **Créer**. 

![7](./media/collaborative-coding-with-git/7-spring-send-pull-request.png)

## <a name="review-and-merge"></a><a name='ReviewandMerge-4'></a>Réviser et fusionner

Une fois que vous avez créé la demande de tirage, vos réviseurs reçoivent une notification par e-mail leur demandant de réviser la demande de tirage. Les réviseurs testent si les changements fonctionnent, et vérifient si possible avec le demandeur les changements apportés. Les réviseurs peuvent formuler des commentaires, demander des changements, et approuver ou rejeter la demande de tirage en fonction de leur évaluation. 

![8](./media/collaborative-coding-with-git/8-add_comments.png)

Une fois que les réviseurs ont approuvé les changements, vous-même ou quelqu’un d’autre avec des autorisations de fusion pouvez fusionner la branche de travail avec sa branche de base. Sélectionnez **Terminé**, puis sélectionnez **Terminer la fusion** dans la boîte de dialogue **Terminer la demande de tirage**. Vous pouvez choisir de supprimer la branche de travail une fois celle-ci fusionnée. 

![10](./media/collaborative-coding-with-git/10-spring-complete-pullrequest.png)

Vérifiez que la demande est marquée comme **TERMINÉE**. 

![11](./media/collaborative-coding-with-git/11-spring-merge-pullrequest.png)

Quand vous revenez aux **Dépôts** dans le volet de navigation de gauche, vous pouvez voir que vous êtes passé à la branche principale, car la branche `script` a été supprimée.

![12](./media/collaborative-coding-with-git/12-spring-branch-deleted.png)

Vous pouvez également utiliser les commandes bash Git suivantes pour fusionner la branche de travail `script` avec la branche de base et pour supprimer la branche de travail après la fusion :

```bash
git checkout main
git merge script
git branch -d script
```

![13](./media/collaborative-coding-with-git/13-spring-branch-deleted-commandline.png)

## <a name="next-steps"></a>Étapes suivantes

[Exécuter des tâches de science des données](execute-data-science-tasks.md) montre comment utiliser des outils pour effectuer plusieurs tâches de science des données courantes, comme l’exploration interactive des données, l’analyse des données, la création de rapports et la création de modèles.

[Exemples de procédures pas à pas](walkthroughs.md) contient une liste de procédures pas à pas pour des scénarios spécifiques, avec des liens et des descriptions miniatures. Les scénarios liés montrent également comment combiner des outils et des services cloud et locaux dans des workflows ou des pipelines pour créer des applications intelligentes. 

