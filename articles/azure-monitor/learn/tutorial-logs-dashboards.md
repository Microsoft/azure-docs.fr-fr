---
title: Créer et partager des tableaux de bord de données Log Analytics | Microsoft Docs
description: Ce tutoriel vous aide à comprendre comment les tableaux de bord Log Analytics permettent de visualiser l’ensemble de vos requêtes de journal enregistrées en proposant une vue unique de votre environnement.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 05/28/2020
ms.custom: mvc
ms.openlocfilehash: 880d173c6eef9d267e8860ebad805e071706de42
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186148"
---
# <a name="create-and-share-dashboards-of-log-analytics-data"></a>Créer et partager des tableaux de bord de données Log Analytics

Les tableaux de bord Log Analytics vous permettent de visualiser l’ensemble de vos requêtes de journal enregistrées. Vous pouvez y trouver les données opérationnelles du service informatique, mettre ces données en corrélation et les partager dans l’organisation.  Ce tutoriel porte sur la création d’une requête de journal pour un tableau de bord partagé utilisé par votre équipe de support des opérations informatiques.  Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un tableau de bord partagé dans le portail Azure
> * Visualiser une requête dans le journal des performances 
> * Ajouter une requête de journal à un tableau de bord partagé 
> * Personnaliser une vignette dans un tableau de bord partagé

Pour effectuer l’exemple de ce didacticiel, vous devez disposer d’une machine virtuelle [connectée à l’espace de travail Log Analytics](quick-collect-azurevm.md).  
 
## <a name="sign-in-to-azure-portal"></a>Se connecter au portail Azure
Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-a-shared-dashboard"></a>Créer un tableau de bord partagé
Sélectionnez **Tableau de bord** pour ouvrir votre [tableau de bord](../../azure-portal/azure-portal-dashboards.md) par défaut. Votre tableau de bord a un aspect différent de l’exemple ci-dessous.

![Tableau de bord du portail Azure](media/tutorial-logs-dashboards/log-analytics-portal-dashboard.png)

Celui-ci vous permet de rassembler les données opérationnelles les plus importantes pour le service informatique dans toutes vos ressources Azure, notamment les données de télémétrie d’Azure Log Analytics.  Avant d’aborder la visualisation d’une requête de journal, commençons par créer et partager un tableau de bord.  Vous pouvez ensuite vous concentrer sur notre exemple de requête dans le journal de performances, qui sera restituée sous forme de graphique en courbes, et l’ajouter ensuite au tableau de bord.  

> [!NOTE]
> Les types de graphiques suivants sont pris en charge dans les tableaux de bord Azure à l’aide des requêtes de journal :
> - Graphique en aires
> - Histogramme
> - Graphique à secteurs (s’affiche dans le tableau de bord sous forme de bouée)
> - Nuage de points
> - Graphique temporel

Pour créer un tableau de bord, sélectionnez le bouton **Nouveau tableau de bord** en regard du nom du tableau de bord actuel.

![Créer un tableau de bord dans le portail Azure](media/tutorial-logs-dashboards/log-analytics-create-dashboard-01.png)

Cette action crée un nouveau tableau de bord privé vide, et vous fait passer en mode de personnalisation. Vous pouvez alors renommer votre tableau de bord et ajouter ou réorganiser les mosaïques. Modifiez le nom du tableau de bord, spécifiez *Sample Dashboard* pour ce tutoriel, puis sélectionnez **Personnalisation terminée**.<br><br> ![Enregistrer le tableau de bord Azure personnalisé](media/tutorial-logs-dashboards/log-analytics-create-dashboard-02.png)

Par défaut, lorsque vous créez un tableau de bord, celui-ci est privé, ce qui signifie que vous êtes la seule personne à pouvoir le consulter. Pour que les autres utilisateurs puissent le voir, utilisez le bouton **Partager** , disponible avec les autres commandes de tableau de bord.

![Partager un nouveau tableau de bord dans le portail Azure](media/tutorial-logs-dashboards/log-analytics-share-dashboard.png) 

Vous êtes invité à choisir un abonnement et un groupe de ressources dans lesquels votre tableau de bord doit être publié. Pour plus de commodité, l’expérience de publication du portail vous guide vers un modèle dans lequel vous placez les tableaux de bord dans un groupe de ressources appelé **tableaux de bord**.  Vérifiez l’abonnement sélectionné, puis cliquez sur **Publier**.  L’accès aux informations figurant dans le tableau de bord est contrôlé par un [Contrôle d’accès en fonction du rôle (RBAC) Azure](../../role-based-access-control/role-assignments-portal.md).   

## <a name="visualize-a-log-query"></a>Visualiser une requête de journal
[Log Analytics](../log-query/log-analytics-tutorial.md) est un portail dédié qui permet d’utiliser des requêtes de journal et leurs résultats. Ces fonctionnalités incluent la possibilité de modifier une requête sur plusieurs lignes et d’exécuter de façon sélective du code, des données Intellisense contextuelles et Smart Analytics. Dans ce tutoriel, vous utilisez Log Analytics pour créer une vue des performances sous forme graphique, l’enregistrer pour une requête ultérieure et l’épingler au tableau de bord partagé créé précédemment.

Ouvrez Log Analytics en sélectionnant **Journaux** dans le menu Azure Monitor. Une nouvelle requête vide s’affiche.

![page d'accueil](media/tutorial-logs-dashboards/homepage.png)

Entrez la requête suivante pour obtenir les enregistrements d’utilisation des processeurs pour les ordinateurs Windows et Linux, regroupés selon les critères Computer et TimeGenerated et affichés dans un graphique visuel. Cliquez sur **Exécuter** pour exécuter la requête et afficher le graphique qui en résulte.

```Kusto
Perf 
| where CounterName == "% Processor Time" and ObjectName == "Processor" and InstanceName == "_Total" 
| summarize AggregatedValue = avg(CounterValue) by bin(TimeGenerated, 1hr), Computer 
| render timechart
```

Enregistrez la requête en sélectionnant le bouton **Enregistrer** en haut de la page.

![Enregistrer la requête](media/tutorial-logs-dashboards/save-query.png)

Dans le panneau de configuration **Enregistrer la requête**, indiquez un nom, par exemple *Machines virtuelles Azure - Utilisation des processeurs* et une catégorie telle que *Tableaux de bord*, puis cliquez sur **Enregistrer**.  De cette façon, vous pouvez créer une bibliothèque de requêtes courantes que vous pouvez utiliser et modifier.  Enfin, épinglez ceci au tableau de bord partagé créé précédemment en sélectionnant le bouton **Épingler au tableau de bord** en haut à droite de la page, puis en sélectionnant le nom du tableau de bord.

Maintenant qu’une requête est épinglée au tableau de bord, vous remarquerez qu’elle a un titre générique et qu’elle est suivie d’un commentaire.

![Exemple de tableau de bord Azure](media/tutorial-logs-dashboards/log-analytics-modify-dashboard-01.png)

 Nous avons tout intérêt à la renommer en lui attribuant un nom explicite et compréhensible des utilisateurs.  Cliquez sur le bouton Modifier pour personnaliser le titre et le sous-titre de la vignette, puis cliquez sur **Mettre à jour**.  Une bannière s’affiche pour vous inviter à publier les modifications ou à les ignorer.  Cliquez sur **Enregistrer une copie**.  

![Configuration terminée de l’exemple de tableau de bord](media/tutorial-logs-dashboards/log-analytics-modify-dashboard-02.png)

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris à créer un tableau de bord dans le portail Azure et à lui ajouter une requête de journal.  Passez au tutoriel suivant pour découvrir les différentes réponses que vous pouvez implémenter en fonction des résultats des requêtes de journal.  

> [!div class="nextstepaction"]
> [Répondre aux événements avec des alertes Log Analytics](tutorial-response.md)