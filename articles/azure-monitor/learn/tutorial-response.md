---
title: Répondre aux événements avec les alertes Azure Log Analytics | Microsoft Docs
description: Ce tutoriel vous aide à comprendre les alertes avec Log Analytics qui vous permettent d’identifier les informations importantes dans votre espace de travail, vous avertissent à un stade précoce de l’existence de problèmes ou appellent des actions pour tenter de les corriger.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 10/05/2018
ms.custom: mvc
ms.openlocfilehash: 8af1acb67961de105e5bf5900d7b10c437cb30d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83835867"
---
# <a name="respond-to-events-with-azure-monitor-alerts"></a>Répondre aux événements avec les alertes Azure Monitor
Les alertes dans Azure Monitor identifient des informations importantes dans votre référentiel Log Analytics. Elles sont créées par des règles d’alerte qui exécutent automatiquement des recherches dans les journaux à intervalles réguliers. Si les résultats de la recherche dans les journaux correspondent à des critères particuliers, un enregistrement d’alerte est créé et peut être configuré pour exécuter une réponse automatisée.  Ce didacticiel est la suite du didacticiel [Créer et partager des tableaux de bord de données Log Analytics](tutorial-logs-dashboards.md).   

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Création d'une règle d'alerte
> * Configurer un groupe d’action qui envoie une notification par e-mail

Pour effectuer l’exemple de ce didacticiel, vous devez disposer d’une machine virtuelle [connectée à l’espace de travail Log Analytics](quick-collect-azurevm.md).  

## <a name="sign-in-to-azure-portal"></a>Se connecter au portail Azure
Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com). 

## <a name="create-alerts"></a>Créez des alertes
Les alertes sont créées par des règles d’alerte dans Azure Monitor et peuvent exécuter automatiquement des requêtes enregistrées ou des recherches personnalisées dans les journaux à intervalles réguliers.  Vous pouvez créer des alertes sur la base de métriques de performances spécifiques ou quand certains événements sont créés, quand un événement fait défaut ou quand un certain nombre d’événements sont créés dans une fenêtre de temps donnée.  Par exemple, les alertes peuvent servir à vous avertir que l’utilisation moyenne du processeur dépasse un certain seuil, qu’une mise à jour manquante est détectée, ou qu’un événement est généré en raison de la détection de la non exécution d’un service Windows spécifique ou du démon Linux.  Si les résultats de la recherche répondent à des critères particuliers, une alerte est créée. La règle peut ensuite exécuter automatiquement une ou plusieurs actions, comme vous avertir de l’alerte ou appeler un autre processus de façon proactive. 

Dans l’exemple suivant, vous créez une règle d’alerte de mesure de métrique basée sur la requête *Machines virtuelles Azure - Utilisation des processeurs* enregistrée dans le [tutoriel Visualiser les données](tutorial-logs-dashboards.md).  Une alerte est créée pour chaque machine virtuelle qui dépasse un seuil de 90 %.  

1. Dans le portail Azure, cliquez sur **Tous les services**. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Log Analytics**.
2. Dans le volet gauche, sélectionnez **Alertes** puis cliquez sur **Nouvelle règle d’alerte** en haut de la page pour créer une nouvelle alerte.<br><br> ![Créer une nouvelle règle d’alerte](./media/tutorial-response/alert-rule-02.png)<br>
3. Pour la première étape, dans la section **Créer une alerte**, sélectionnez votre espace de travail Log Analytics en tant que ressource, car il s’agit d’un signal d’alerte basé sur des journaux.  Si vous avez plusieurs résultats, filtrez-les en choisissant l’**abonnement** spécifique dans la liste déroulante, qui contient la machine virtuelle et l’espace de travail Log Analytics créés précédemment.  Filtrez le **Type de ressource** en sélectionnant **Log Analytics** dans la liste déroulante.  Pour finir, sélectionnez la **ressource** **DefaultLAWorkspace** et cliquez sur **Terminé**.<br><br> ![Créer une tâche d’alerte étape 1](./media/tutorial-response/alert-rule-03.png)<br>
4. Dans la section **Critères d’alerte**, cliquez sur **Ajouter des critères** pour sélectionner notre requête enregistrée, puis la logique spécifiée que la règle d’alerte suit.  Dans le volet **Configurer la logique du signal**, sélectionnez *Machines virtuelles Azure - Utilisation du processeur* dans la liste.  Le volet se met à jour pour présenter les paramètres de configuration de l’alerte.  En haut, il affiche les résultats pour les 30 dernières minutes du signal sélectionné et pour la requête de recherche.  
5. Configurez l’alerte avec les informations suivantes :  
   a. Dans la liste déroulante **Basé sur**, sélectionnez **Mesure des métriques**.  Une mesure des métriques créera une alerte pour chaque objet de la requête dont la valeur dépasse notre seuil spécifié.  
   b. Pour la **Condition**, sélectionnez **Supérieur à** et entrez **90** pour le **Seuil**.  
   c. Dans la section Déclencher l’alerte selon, sélectionnez **Violations consécutives** et dans la liste déroulante, sélectionnez **Supérieur à** et entrez la valeur 3.  
   d. Dans la section Évaluation basée sur, modifiez la valeur **période** à **30** minutes. La règle s’exécute toutes les cinq minutes et renvoie les enregistrements qui ont été créés dans les trente minutes précédant l’heure actuelle.  Paramétrer la période de temps sur une durée plus longue compense la latence potentielle des données et garantit que la requête retourne des données pour éviter un faux positif où l’alerte ne se déclenche jamais.  
6. Cliquez sur **Terminé** pour terminer la règle d’alerte.<br><br> ![Configurer un signal d’alerte](./media/tutorial-response/alert-signal-logic-02.png)<br> 
7. Passons maintenant à la deuxième étape. Donnez un nom à votre alerte dans le champ **Nom de la règle d’alerte**, tel que **Pourcentage d’UC supérieur à 90 %** .  Spécifiez une **Description** détaillant les spécificités de l’alerte, puis sélectionnez **Critique (gravité 0)** pour la valeur de **Gravité** parmi les options fournies.<br><br> ![Configuration des détails de l’alerte](./media/tutorial-response/alert-signal-logic-04.png)<br>
8. Pour activer immédiatement la règle d’alerte lors de la création, acceptez la valeur par défaut pour l’option **Activer la règle lors de sa création**.
9. Pour la troisième et dernière étape, vous spécifiez un **Groupe d’actions**, ce qui garantit que les mêmes actions soient entreprises chaque fois qu’une alerte est déclenchée et peuvent être utilisées pour chaque règle que vous définissez.  Configurez un nouveau groupe d’actions avec les informations suivantes :  
   a. Sélectionnez **Nouveau groupe d’actions** et le volet **Ajouter un groupe action** s’affiche.  
   b. Pour le **Nom du groupe d’actions**, spécifiez un nom tel que **Opérations informatiques - Notifier** et un **Nom court** comme **itops-n**.  
   c. Vérifiez que les valeurs par défaut de l’**Abonnement** et du **Groupe de ressources** sont correctes. Si ce n’est pas le cas, sélectionnez la valeur correcte dans la liste déroulante.   
   d. Dans la section Actions, spécifiez un nom pour l’action, tel que **Envoyer un message électronique**, et pour le **Type d’action**, sélectionnez **E-mail/SMS/Push/Voix** dans la liste déroulante. Le volet Propriétés **E-mail/SMS/Push/Voix** s’ouvre sur la droite afin de fournir des informations supplémentaires.  
   e. Dans le volet **E-mail/SMS/Push/Voix**, autorisez les **e-mails** et fournissez une adresse de messagerie SMTP valide à laquelle envoyer le message.  
   f. Cliquez sur **OK** pour enregistrer vos modifications.<br><br> 

    ![Créer un nouveau groupe d’action](./media/tutorial-response/action-group-properties-01.png)

10. Cliquez sur **OK** pour créer le groupe d’actions. 
11. Cliquez sur **Créer une règle d’alerte** pour terminer la règle d’alerte. Son exécution démarre immédiatement.<br><br> ![Terminer la création d’une nouvelle règle d’alerte](./media/tutorial-response/alert-rule-01.png)<br> 

## <a name="view-your-alerts-in-azure-portal"></a>Visualiser les alertes dans le portail Azure
Maintenant que vous avez créé une alerte, vous pouvez afficher les alertes Azure dans un seul volet et gérer toutes les règles d’alerte de l’ensemble de vos abonnements Azure. Elle répertorie toutes les règles d’alerte (activées ou désactivées) et peut être triée en fonction des ressources cibles, des groupes de ressources, du nom de règle ou de l’état. Un résumé agrégé de toutes les alertes déclenchées et des règles d’alerte configurées/activées est inclus.<br><br> ![Page de l’état des alertes Azure](./media/tutorial-response/azure-alerts-02.png)  

Lorsque l’alerte se déclenche, le tableau affiche l’état et le nombre de fois que cela s’est produit dans l’intervalle de temps sélectionné (la valeur par défaut est les six dernières heures).  Vous devriez trouver dans votre boîte de réception un message électronique correspondant similaire à l’exemple suivant, indiquant la machine virtuelle qui pose problème et les meilleurs résultats correspondant à la requête de recherche dans ce cas.<br><br> ![Exemple d’action d’e-mail d’alerte](./media/tutorial-response/azure-alert-email-notification-01.png)

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez découvert en quoi les règles d’alerte permettaient d’identifier et de répondre à un stade précoce à un problème déterminé quand elles exécutent des recherches dans les journaux à intervalles réguliers et qu’elles correspondent à des critères particuliers.

Suivez ce lien pour examiner des exemples de scripts Log Analytics prédéfinis.  

> [!div class="nextstepaction"]
> [Exemples de scripts Log Analytics](../samples/powershell-samples.md)
