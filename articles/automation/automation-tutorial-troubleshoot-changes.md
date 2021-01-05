---
title: Résoudre les problèmes liés aux modifications apportées à une machine virtuelle Azure dans Azure Automation | Microsoft Docs
description: Cet article explique comment résoudre les problèmes liés aux modifications apportées à une machine virtuelle Azure.
services: automation
ms.subservice: change-inventory-management
keywords: modification, suivi, suivi des modifications, change tracking, inventory, inventaire, automatisation, automation
ms.date: 12/05/2018
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: 28c440f27dcbd4ac509adea83d5c3085488cb488
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92204246"
---
# <a name="troubleshoot-changes-on-an-azure-vm"></a>Résoudre les problèmes liés aux modifications apportées à une machine virtuelle Azure

Dans ce didacticiel, vous allez apprendre à dépanner les modifications apportées à une machine virtuelle Azure. En activant Change Tracking and Inventory, vous pouvez suivre les modifications apportées aux logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs.
L’identification de ces modifications de configuration peut vous aider à mettre à jour les problèmes opérationnels constatés dans votre environnement.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Activer Change Tracking and Inventory pour une machine virtuelle
> * Rechercher les journaux d’activité des modifications des services arrêtés
> * Configurer le suivi des modifications
> * Activer la connexion du journal d’activité
> * Déclencher un événement
> * Afficher les modifications
> * Configurer des alertes

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un [compte Automation](./index.yml) qui contiendra les runbooks Watcher et d’action, ainsi que la tâche Watcher.
* Une [machine virtuelle](../virtual-machines/windows/quick-create-portal.md) à activer pour la fonctionnalité.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

Pour ce tutoriel, vous devez d’abord activer Change Tracking and Inventory. Si vous avez déjà activé la fonctionnalité, cette étape n’est pas nécessaire.

>[!NOTE]
>Si les champs sont grisés, une autre solution Automation est activée pour la machine virtuelle et vous devez utiliser les mêmes espace de travail et compte Automation.

1. Sélectionnez **Machines virtuelles**, puis sélectionnez une machine virtuelle dans la liste.
2. Dans le menu de gauche, sélectionnez **Inventory** sous **Opérations**. La page Inventory s’ouvre.

    ![Activer la modification](./media/automation-tutorial-troubleshoot-changes/enableinventory.png)

3. Choisissez l’espace de travail [Log Analytics](../azure-monitor/log-query/log-query-overview.md). Cet espace de travail collecte les données qui sont générées par des fonctionnalités, telles que Change Tracking and Inventory. L’espace de travail fournit un emplacement unique permettant de consulter et d’analyser les données provenant de plusieurs sources.

    [!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

4. Sélectionnez le compte Automation à utiliser.

5. Configurez l’emplacement pour le déploiement.

5. Cliquez sur **Activer** pour déployer la fonctionnalité destinée à votre machine virtuelle. 

Pendant la configuration, la machine virtuelle est provisionnée avec l’agent Log Analytics pour Windows et un [Runbook Worker hybride](automation-hybrid-runbook-worker.md). L’activation de Change Tracking and Inventory peut prendre 15 minutes. Pendant ce temps, vous ne devez pas fermer la fenêtre du navigateur.

Dès la fonctionnalité activée, des informations sur les logiciels installés et les modifications apportées à la machine virtuelle sont envoyées aux journaux Azure Monitor.
Entre 30 minutes et 6 heures peuvent être nécessaires pour que les données soient disponibles pour l’analyse.

## <a name="use-change-tracking-and-inventory-in-azure-monitor-logs"></a>Utiliser Change Tracking and Inventory dans les journaux Azure Monitor

Change Tracking and Inventory génère des données de journal qui sont envoyées aux journaux Azure Monitor. Pour rechercher dans les journaux en exécutant des requêtes, sélectionnez **Log Analytics** en haut de la page Change Tracking. Les données de Change Tracking sont stockées sous le type `ConfigurationChange`.

L’exemple de requête Log Analytics ci-après retourne tous les services Windows arrêtés.

```loganalytics
ConfigurationChange
| where ConfigChangeType == "WindowsServices" and SvcState == "Stopped"
```

Pour en savoir plus sur l’exécution et la recherche de fichiers journaux dans les journaux d’activité Azure Monitor, consultez [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="configure-change-tracking"></a>Configurer le suivi des modifications

Avec le suivi des modifications, vous choisissez les fichiers et clés de Registre à collecter et à suivre par l’intermédiaire de **Modifier les paramètres** en haut de la page Change Tracking sur votre machine virtuelle. Vous pouvez ajouter des clés de Registre Windows, des fichiers Windows ou Linux à suivre sur la page de configuration de l’espace de travail.

> [!NOTE]
> Le suivi des modifications comme l’inventaire utilisent les mêmes paramètres de collection ; ces paramètres sont configurés au niveau de l’espace de travail.

### <a name="add-a-windows-registry-key"></a>Ajouter une clé de Registre Windows

1. Dans l’onglet **Registre Windows**, sélectionnez **Ajouter**. 

1. Dans la page Ajouter le Registre Windows pour le suivi des modifications, entrez les informations correspondant à la clé à suivre et cliquez sur **Enregistrer**

    |Propriété  |Description  |
    |---------|---------|
    |activé     | Détermine si le paramètre est appliqué.        |
    |Item Name     | Nom convivial du fichier à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
    |Clé de Registre Windows   | Chemin d’accès pour rechercher le fichier. Exemple : « HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders\Common Startup »      |

### <a name="add-a-windows-file"></a>Ajouter un fichier Windows

1. Dans l’onglet **Fichiers Windows**, sélectionnez **Ajouter**. 

1. Dans la page Ajouter le fichier Windows pour le suivi des modifications, entrez les informations correspondant au fichier ou répertoire à suivre et cliquez sur **Enregistrer**

    |Propriété  |Description  |
    |---------|---------|
    |activé     | Détermine si le paramètre est appliqué.        |
    |Item Name     | Nom convivial du fichier à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
    |Entrer le chemin     | Chemin d’accès pour rechercher le fichier. Exemple : « c:\temp\\\*.txt »<br>Vous pouvez également utiliser des variables d’environnement telles que « %winDir%\System32\\\*.* »         |
    |Récursivité     | Détermine si la récursivité est utilisée lorsque vous recherchez l’élément à suivre.        |
    |Télécharger le contenu du fichier pour tous les paramètres| Active ou désactive le chargement du contenu du fichier pour le suivi des modifications. Options disponibles : **True** ou **False**.|

### <a name="add-a-linux-file"></a>Ajouter un fichier Linux

1. Dans l’onglet **Fichiers Linux**, sélectionnez **Ajouter**. 

1. Dans la page Ajouter le fichier Linux pour le suivi des modifications, entrez les informations correspondant au fichier ou répertoire à suivre et cliquez sur **Enregistrer**.

    |Propriété  |Description  |
    |---------|---------|
    |activé     | Détermine si le paramètre est appliqué.        |
    |Item Name     | Nom convivial du fichier à suivre.        |
    |Groupe     | Nom de groupe pour le regroupement logique des fichiers.        |
    |Entrer le chemin     | Chemin d’accès pour rechercher le fichier. Exemple : « /etc/*.conf ».       |
    |Type de chemin     | Type d’élément à suivre. Valeurs possibles : fichier et répertoire.        |
    |Récursivité     | Détermine si la récursivité est utilisée lorsque vous recherchez l’élément à suivre.        |
    |Utiliser sudo     | Ce paramètre détermine si sudo est utilisé lorsque vous vérifiez l’élément.         |
    |Liens     | Ce paramètre détermine le traitement des liens symboliques lorsque vous parcourez les répertoires.<br> **Ignorer** : ignore les liens symboliques et n’inclut pas les fichiers/répertoires référencés.<br>**Suivre** : suit les liens symboliques pendant les opérations de récursivité et inclut aussi les fichiers/répertoires référencés.<br>**Gérer** : suit les liens symboliques et autorise la modification du traitement du contenu retourné.      |
    |Télécharger le contenu du fichier pour tous les paramètres| Active ou désactive le chargement du contenu du fichier pour le suivi des modifications. Options disponibles : True ou False.|

   > [!NOTE]
   > La valeur **Gérer** pour la propriété **Liens** n’est pas recommandée. L’extraction du contenu du fichier n’est pas prise en charge.

## <a name="enable-activity-log-connection"></a>Activer la connexion du journal d’activité

1. À partir de la page Suivi des modifications sur votre machine virtuelle, sélectionnez **Gérer la connexion du journal d’activité**. 

2. Dans la page Journal d’activité Azure, cliquez sur **Connexion** pour connecter Change Tracking and Inventory au journal d’activité Azure de votre machine virtuelle.

3. Accédez à la page Vue d’ensemble de votre machine virtuelle, puis sélectionnez **Arrêter** pour arrêter la machine. 

4. Lorsque vous y êtes invité, sélectionnez **Oui** pour arrêter la machine virtuelle. 

5. Lorsque la machine virtuelle est libérée, sélectionnez **Démarrer** pour la redémarrer. Le fait d’arrêter et de démarrer une machine virtuelle enregistre un événement dans son journal d’activité. 

## <a name="view-changes"></a>Afficher les modifications

1. Revenez à la page du suivi des modifications, puis sélectionnez l’onglet **Événements** au bas de la page. 

2. Après un certain temps, les événements du suivi des modifications apparaissent dans le graphique et le tableau. Le graphique affiche les modifications qui se sont produites au fil du temps. Le graphique linéaire, en haut, affiche les événements du Journal d’activité Azure. Chacune des lignes des graphiques à barres représente un type de modification différent dont il est possible d’effectuer le suivi. Ces types sont les démons Linux, les fichiers, les clés de Registre Windows, les logiciels et les services Windows. L’onglet Modifications donne des détails sur les changements qui sont affichés, la modification la plus récente apparaissant en premier.

    ![Afficher les événements dans le portail](./media/automation-tutorial-troubleshoot-changes/viewevents.png)

3. Remarquez que plusieurs modifications ont été apportées au système, y compris aux logiciels et aux services. Vous pouvez utiliser les filtres en haut de la page pour filtrer les résultats par **type de modification** ou par plage de temps.

    ![Liste des modifications apportées à la machine virtuelle](./media/automation-tutorial-troubleshoot-changes/change-tracking-list.png)

4. En sélectionnant une modification **WindowsServices**, vous ouvrez la page Détails de la modification qui présente des détails sur la modification ainsi que les valeurs avant et après. Dans ce cas, le service Protection logicielle a été arrêté.

    ![Comment afficher le détail des modifications dans le portail](./media/automation-tutorial-troubleshoot-changes/change-details.png)

## <a name="configure-alerts"></a>Configurer des alertes

L’affichage des modifications dans le Portail Azure est pratique, mais pouvoir être averti d’une modification (l’arrêt d’un service, par exemple) peut être bien plus avantageux. Ajoutons une alerte pour un service arrêté. 

1. Dans le portail Azure, accédez à **Surveillance**. 

2. Sous **Services partagés**, sélectionnez **Alertes**, puis cliquez sur **+ Nouvelle règle d’alerte**.

3. Cliquez sur **Sélectionner** pour choisir une ressource. 

4. Dans la page de sélection d’une ressource, choisissez **Log Analytics** dans le menu déroulant **Filtrer par type de ressource**. 

5. Sélectionnez votre espace de travail Log Analytics, puis cliquez sur **Terminé**.

    ![Sélectionner une ressource](./media/automation-tutorial-troubleshoot-changes/select-a-resource.png)

6. Cliquez sur **Ajouter une condition**.

7. Dans le tableau de la page de configuration de la logique du signal, sélectionnez **Recherche personnalisée dans les journaux**. 

8. Entrez la requête suivante dans la zone de texte de la requête de recherche :

    ```loganalytics
    ConfigurationChange | where ConfigChangeType == "WindowsServices" and SvcName == "W3SVC" and SvcState == "Stopped" | summarize by Computer
    ```

    Cette requête renvoie les ordinateurs dont le service W3SVC s’est arrêté au cours de l’intervalle spécifié.

9. Pour **Seuil**, sous **Logique d’alerte**, entrez **0**. Lorsque vous avez fini, cliquez sur **Terminé**.

    ![Configurer la logique du signal](./media/automation-tutorial-troubleshoot-changes/configure-signal-logic.png)

10. Sélectionnez **Créer** sous **Groupes d’action**. Un groupe d’actions est un groupe que vous pouvez utiliser dans plusieurs alertes. Les actions peuvent inclure, sans s’y limiter, les notifications par e-mail, les runbooks, les webhooks et bien plus encore. Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions](../azure-monitor/platform/action-groups.md).

11. Sous **Détails de l’alerte**, entrez un nom et une description pour l’alerte. 

12. Définissez le niveau de **Gravité** sur **Information (gravité 2)** , **Avertissement (gravité 1)** ou **Critique (gravité 0)** .

13. Dans la zone **Nom du groupe d’actions** , entrez un nom pour l’alerte et un nom court. Le nom court est utilisé à la place du nom complet du groupe d’actions lorsque les notifications sont envoyées à l’aide de ce groupe.

14. Pour **Actions**, entrez le nom de l’action, par exemple **Administrateurs de la messagerie**. 

15. Pour **TYPE D’ACTION**, sélectionnez **E-mail/SMS/Push/Voix**. 

16. Pour **DÉTAILS**, sélectionnez **Modifier les détails**.

    ![Ajouter un groupe d'actions](./media/automation-tutorial-troubleshoot-changes/add-action-group.png)

17. Dans le volet E-mail/SMS/Push/Voix, entrez un nom, activez la case à cocher **E-mail**, puis entrez une adresse e-mail valide. Lorsque cela est fait, cliquez sur **OK** dans le volet, puis sur **OK** dans la page d’ajout d’un groupe d’actions.

18. Pour personnaliser l’objet de l’e-mail d’alerte, sélectionnez **Personnaliser les actions**. 

19. Pour **Créer une règle**, sélectionnez **Objet de l’e-mail**, puis choisissez **Créer une règle d’alerte**. L’alerte vous avertit quand un déploiement de mises à jour s’est correctement déroulé et précise quelles machines en ont bénéficié. L’image suivante est un exemple d’e-mail reçu quand le service W3SVC s’arrête.

    ![La capture d’écran montre une notification par e-mail reçue quand les services W3SVC s’arrêtent.](./media/automation-tutorial-troubleshoot-changes/email.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Activer Change Tracking and Inventory pour une machine virtuelle
> * Rechercher les journaux d’activité des modifications des services arrêtés
> * Configurer le suivi des modifications
> * Activer la connexion du journal d’activité
> * Déclencher un événement
> * Afficher les modifications
> * Configurer des alertes

Accédez ensuite à la présentation de la fonctionnalité Change Tracking and Inventory pour en savoir plus.

> [!div class="nextstepaction"]
> [Vue d’ensemble de Change Tracking and Inventory](change-tracking/overview.md)
