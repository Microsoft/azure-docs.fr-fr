---
title: Collecter des journaux de ressources à partir d’une ressource Azure et les analyser avec Azure Monitor
description: Tutoriel permettant de configurer les paramètres de diagnostic afin de collecter des journaux de ressources à partir d’une ressource Azure dans un espace de travail Log Analytics où ils peuvent être analysés avec une requête de journal.
ms.subservice: ''
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 12/15/2019
ms.openlocfilehash: b68d4779d33d2f8a1f1c2bae35f438ad2ccccabb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90090072"
---
# <a name="tutorial-collect-and-analyze-resource-logs-from-an-azure-resource"></a>Tutoriel : Collecter et analyser des journaux de ressources à partir d’une ressource Azure

Les journaux de ressources fournissent des insights sur le fonctionnement détaillé des ressources Azure, et sont utiles pour superviser leur intégrité et leur disponibilité. Les ressources Azure génèrent automatiquement des journaux de ressources, mais vous devez configurer l’emplacement où ils doivent être collectés. Ce tutoriel vous guide tout au long du processus de création d’un paramètre de diagnostic afin de collecter des journaux de ressources pour une ressource dans votre abonnement Azure et l’analyser avec une requête de journal.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un espace de travail Log Analytics dans Azure Monitor.
> * Créer un paramètre de diagnostic pour collecter des journaux de ressources. 
> * Créer une requête de journal simple pour analyser des journaux.


## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous avez besoin d’une ressource Azure à superviser. Vous pouvez utiliser n’importe quelle ressource de votre abonnement Azure qui prend en charge les paramètres de diagnostic. Pour déterminer si une ressource prend en charge les paramètres de diagnostic, accédez à son menu dans le portail Azure et vérifiez qu’une option **Paramètres de diagnostic** figure dans la section **Supervision** du menu.


## <a name="log-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).


## <a name="create-a-workspace"></a>Créer un espace de travail
Un espace de travail Log Analytics dans Azure Monitor collecte et indexe les données des journaux à partir de diverses sources, et permet d’effectuer une analyse avancée à l’aide d’un puissant langage de requête. L’espace de travail Log Analytics doit exister avant que vous ne créiez un paramètre de diagnostic pour y envoyer des données. Vous pouvez utiliser un espace de travail existant dans votre abonnement Azure ou en créer un avec la procédure suivante. 

> [!NOTE]
> Bien que vous puissiez travailler avec les données dans les espaces de travail Log Analytics du menu **Azure Monitor**, la création et la gestion des espaces de travail s’effectuent dans le menu **Espaces de travail Log Analytics**.

1. Dans **Tous les services**, sélectionnez **Espaces de travail Log Analytics**.
2. Cliquez sur **Ajouter** en haut de l’écran et fournissez les détails suivants pour l’espace de travail :
   - **Espace de travail Log Analytics** : nom du nouvel espace de travail. Ce nom doit être globalement unique sur l’ensemble des abonnements Azure Monitor.
   - **Abonnement**: sélectionnez l’abonnement où stocker l’espace de travail. Il n’est pas obligatoire qu’il s’agisse du même abonnement que celui de la ressource supervisée.
   - **Groupe de ressources** : sélectionnez un groupe de ressources existant ou cliquez sur **Créer** pour en créer un. Il n’est pas obligatoire qu’il s’agisse du même groupe de ressources que celui de la ressource supervisée.
   - **Emplacement** : sélectionnez une région Azure ou créez-en une nouvelle. Il n’est pas obligatoire qu’il s’agisse du même emplacement que celui de la ressource supervisée.
   - **Niveau tarifaire** : Sélectionnez le niveau tarifaire *Paiement à l’utilisation*. Vous pourrez modifier ce niveau tarifaire ultérieurement. Cliquez sur le lien **Tarifs Log Analytics** pour en savoir plus sur les différents niveaux tarifaires.

    ![Nouvel espace de travail](media/tutorial-resource-logs/new-workspace.png)

3. Cliquez sur **OK** pour créer l’espace de travail.

## <a name="create-a-diagnostic-setting"></a>Créer un paramètre de diagnostic
Les [paramètres de diagnostic](../platform/diagnostic-settings.md) définissent où les journaux de ressources doivent être envoyés pour une ressource particulière. Un même paramètre de diagnostic peut avoir plusieurs [destinations](../platform/diagnostic-settings.md#destinations), mais nous n’utiliserons qu’un espace de travail Log Analytics dans ce tutoriel.

1. Dans la section **Supervision** du menu de votre ressource, sélectionnez **Paramètres de diagnostic**.
2. Un message « Aucun paramètre de diagnostic défini » doit s’afficher. Cliquez sur **Ajouter le paramètre de diagnostic**.

    ![Paramètres de diagnostic](media/tutorial-resource-logs/diagnostic-settings.png)

3. Chaque paramètre de diagnostic comporte trois parties :
 
   - **Name** : il n’a aucun effet significatif, et n’a pour but que de décrire le paramètre.
   - **Destinations** : Une ou plusieurs destinations auxquelles envoyer les journaux. Tous les services Azure partagent le même ensemble de trois destinations possibles. Chaque paramètre de diagnostic peut définir une ou plusieurs destinations, mais pas plus d’une destination d’un type spécifique. 
   - **Categories** : catégories de journaux à envoyer à chacune des destinations. L’ensemble des catégories varie en fonction de chaque service Azure.

4. Sélectionnez **Envoyer vers un espace de travail Log Analytics**, puis sélectionnez l’espace de travail que vous avez créé.
5. Sélectionnez les catégories que vous souhaitez collecter. Consultez la documentation de chaque service pour obtenir la définition de ses catégories disponibles.

    ![Paramètre de diagnostic](media/tutorial-resource-logs/diagnostic-setting.png)

6. Cliquez sur **Enregistrer** pour enregistrer les paramètres de diagnostic.

    
 
 ## <a name="use-a-log-query-to-retrieve-logs"></a>Utiliser une requête de journal pour récupérer des journaux
Les données sont récupérées à partir d’un espace de travail Log Analytics à l’aide d’une requête de journal écrite en langage KQL (Kusto Query Language). Les insights et solutions dans Azure Monitor fournissent des requêtes de journal afin de récupérer des données pour un service particulier, mais vous pouvez travailler directement avec des requêtes de journal et leurs résultats dans le portail Azure avec Log Analytics. 

1. Dans la section **Supervision** du menu de votre ressource, sélectionnez **Journaux**.
2. Log Analytics s’ouvre avec une fenêtre de requête vide et l’étendue définie sur votre ressource. Les requêtes n’incluront que les enregistrements de cette ressource.

    > [!NOTE]
    > Si vous avez ouvert Log Analytics à partir du menu Azure Monitor, l’étendue sera définie sur l’espace de travail Log Analytics. Dans ce cas, les requêtes incluront tous les enregistrements dans l’espace de travail.
   
    ![La capture d’écran montre les journaux d’une application logique affichant une nouvelle requête avec le nom de l’application logique mis en évidence.](media/tutorial-resource-logs/logs.png)

4. Le service présenté dans l’exemple écrit les journaux de ressources dans la table **AzureDiagnostics**, mais les autres services peuvent écrire dans d’autres tables. Consultez [Services, schémas et catégories pris en charge pour les journaux de ressources Azure](../platform/resource-logs-schema.md) pour plus de détails sur les tables utilisées par les différents services Azure.

    > [!NOTE]
    > Plusieurs services écrivent des journaux de ressources dans la table AzureDiagnostics. Si vous démarrez Log Analytics à partir du menu Azure Monitor, vous devez ajouter une instruction `where` avec la colonne `ResourceProvider` pour spécifier votre service particulier. Quand vous démarrez Log Analytics à partir du menu d’une ressource, l’étendue est définie uniquement sur les enregistrements de cette ressource ; cette colonne n’est donc pas obligatoire. Pour obtenir des exemples de requêtes, consultez la documentation du service.


5. Tapez une requête, puis cliquez sur **Exécuter** pour inspecter les résultats. 
6. Pour obtenir un tutoriel sur l’écriture de requêtes de journal, consultez [Bien démarrer avec les requêtes de journal dans Azure Monitor](../log-query/get-started-queries.md).

    ![Requête de journal](media/tutorial-resource-logs/log-query-1.png)




## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris à collecter des journaux de ressources dans un espace de travail Log Analytics, suivez un tutoriel sur l’écriture de requêtes de journal pour analyser ces données.

> [!div class="nextstepaction"]
> [Bien démarrer avec les requêtes de journal Azure Monitor](../log-query/get-started-queries.md).
