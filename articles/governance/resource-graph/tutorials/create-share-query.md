---
title: 'Tutoriel : Gérer les requêtes dans le portail Azure'
description: Dans ce tutoriel, vous créez une requête Resource Graph et vous la partagez avec d’autres personnes dans le portail Azure.
ms.date: 10/14/2020
ms.topic: tutorial
ms.openlocfilehash: 98d4c7d0493686a45bdfef9d85823432cb637a66
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057924"
---
# <a name="tutorial-create-and-share-an-azure-resource-graph-query-in-the-azure-portal"></a>Tutoriel : Créer et partager une requête Azure Resource Graph sur le portail Azure

L'Explorateur Azure Resource Graph vous permet d'enregistrer vos requêtes Resource Graph directement sur le portail Azure. Il existe deux types de requêtes : les requêtes _privées_ et les requêtes _partagées_. Une requête privée est enregistrée dans les paramètres de votre portail Azure. Une requête partagée, quant à elle, est une ressource Azure Resource Manager qui peut être gérée avec un contrôle d’accès en fonction du rôle Azure (Azure RBAC) et protégée à l’aide de verrous de ressources. Les deux types de requêtes sont chiffrées au repos.

L'enregistrement de requêtes sur le portail Azure vous fait gagner du temps puisqu'il vous évite de devoir rechercher vos requêtes favorites ou couramment utilisées. Lorsque vous partagez des requêtes, vous aidez votre équipe à atteindre des objectifs de cohérence et d'efficacité par la répétition.

Dans ce didacticiel, vous allez apprendre à effectuer les tâches suivantes :

> [!div class="checklist"]
> - Créer et supprimer une requête privée
> - Créer une requête partagée
> - Découvrir des requêtes partagées
> - Supprimer une requête partagée

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer d’un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-and-delete-a-private-query"></a>Créer et supprimer une requête privée

Les requêtes privées ne sont accessibles et visibles que par le compte qui les crée. Comme elles sont enregistrées dans les paramètres du portail Azure d'un compte, elles ne peuvent être créées, utilisées et supprimées que depuis le portail Azure. Une requête privée n'est pas une ressource Resource Manager. Pour créer une requête privée, procédez comme suit :

1. Dans le menu du portail, sélectionnez **Tous les services** ou utilisez la zone de recherche Azure située en haut de toutes les pages. Recherchez l'**Explorateur Resource Graph**, puis sélectionnez-le.

1. Dans l'onglet **Requête 1** de la page Explorateur Azure Resource Graph, entrez la requête suivante :

   ```kusto
   Resources
   | where type =~ 'Microsoft.Compute/virtualMachines'
   | summarize count() by tostring(properties.storageProfile.osDisk.osType)
   ```

   Sélectionnez **Exécuter la requête** pour afficher les résultats de la requête dans le volet inférieur.

   Pour plus d'informations sur cette requête, consultez [Exemples - Nombre de machines virtuelles par type de système d'exploitation](../samples/starter.md#count-os).

1. Sélectionnez **Enregistrer** ou **Enregistrer sous**, entrez **Nombre de machines virtuelles par système d'exploitation**, conservez le type **Requête privée**, puis sélectionnez **Enregistrer** en bas du volet **Enregistrer la requête**. Le titre de l'onglet change : **Requête 1** est remplacé par **Nombre de machines virtuelles par système d'exploitation**.

1. Sur le portail Azure, quittez l'Explorateur Azure Resource Graph, puis rouvrez-le. Notez que la requête enregistrée n'apparaît plus et que l'onglet **Requête 1** est réapparu.

1. Sélectionnez **Ouvrir une requête**. Assurez-vous que le type est défini sur **Requête privée**. Le nom enregistré, **Nombre de machines virtuelles par système d'exploitation**, apparaît maintenant dans la liste **Nom de la requête**. Lorsque vous sélectionnez le lien titre de la requête enregistrée, il est chargé dans un nouvel onglet portant le nom de cette requête.

   > [!NOTE] 
   > Lorsqu'une requête enregistrée est ouverte et que l'onglet affiche son nom, il suffit de sélectionner le bouton **Enregistrer** pour la mettre à jour avec les modifications apportées. Pour créer une nouvelle requête enregistrée à partir de cette requête ouverte, sélectionnez **Enregistrer sous** et procédez comme si vous enregistriez une nouvelle requête.

1. Pour supprimer la requête enregistrée, sélectionnez à nouveau **Ouvrir une requête** et vérifiez que le champ **Type** est défini sur **Requête privée**. Sur la ligne de la requête `Count VMs by OS` enregistrée, sélectionnez **Supprimer** (icône Corbeille). Dans la boîte de dialogue de confirmation, sélectionnez **Oui** pour valider la suppression de la requête.
   Puis fermez le volet **Ouvrir une requête**.

## <a name="create-a-shared-query"></a>Créer une requête partagée

Contrairement à une requête privée, une requête partagée est une ressource Resource Manager. Cela signifie que la requête est enregistrée dans un groupe de ressources, qu'elle peut être gérée et contrôlée à l'aide du contrôle Azure RBAC, et qu'elle peut même être protégée à l'aide de verrous de ressources. Comme il s'agit d'une ressource, toute personne disposant des autorisations appropriées peut la voir et l'utiliser. Pour créer une requête partagée, procédez comme suit :

1. Dans le menu du portail, sélectionnez **Tous les services** ou utilisez la zone de recherche Azure située en haut de toutes les pages pour rechercher et sélectionner l'**Explorateur Resource Graph**.

1. Dans l'onglet **Requête 1** de la page Explorateur Azure Resource Graph, entrez la requête suivante :

   ```kusto
   Resources
   | where type =~ 'Microsoft.Compute/virtualMachines'
   | summarize count() by tostring(properties.storageProfile.osDisk.osType)
   ```
    
   Sélectionnez **Exécuter la requête** pour afficher les résultats de la requête dans le volet inférieur.

   Pour plus d'informations sur cette requête, consultez [Exemples - Nombre de machines virtuelles par type de système d'exploitation](../samples/starter.md#count-os).

1. Sélectionnez **Enregistrer** ou **Enregistrer sous**.

   :::image type="content" source="../media/create-share-query/save-shared-query-buttons.png" alt-text="Enregistrer la nouvelle requête à l’aide du bouton Enregistrer" border="false":::

1. Dans le volet **Enregistrer la requête**, entrez le nom **Nombre de machines virtuelles par système d'exploitation**.

1. Remplacez le type par **Requête partagée**, définissez la description sur **Nombre de machines virtuelles par type de système d'exploitation**, puis définissez **Abonnement** pour spécifier l'emplacement de création de la ressource de requête.

1. Cochez la case **Publier dans le groupe de ressources resource-graph-queries** et définissez l'**Emplacement du groupe de ressources** sur **(USA) USA Centre-Ouest**.

1. Sélectionnez **Enregistrer** en bas du volet **Enregistrer la requête**. Le titre de l'onglet change : **Requête 1** est remplacé par **Nombre de machines virtuelles par système d'exploitation**. La première fois que le groupe de ressources **resource-graph-queries** est utilisé, l'enregistrement peut prendre un certain temps car le groupe de ressources est en cours de création.
   
   :::image type="content" source="../media/create-share-query/save-shared-query-window.png" alt-text="Enregistrer la nouvelle requête à l’aide du bouton Enregistrer" border="false":::

   > [!NOTE] 
   > Vous pouvez décocher la case **Publier dans le groupe de ressources resource-graph-queries** si vous souhaitez fournir le nom d'un groupe de ressources existant pour l'enregistrement de la requête partagée. L'utilisation du groupe de ressources nommé par défaut pour les requêtes facilite la découverte des requêtes partagées et permet de mieux comprendre la raison d'être de ce groupe de ressources. Vous pouvez toutefois choisir de sélectionner un groupe de ressources existant pour des raisons de sécurité, en fonction des autorisations existantes.

1. Sur le portail Azure, quittez l'Explorateur Azure Resource Graph, puis rouvrez-le. Notez que la requête enregistrée n'apparaît plus et que l'onglet **Requête 1** est réapparu.

1. Sélectionnez **Ouvrir une requête**. Vérifiez que le type est défini sur **Requête partagée** et que la combinaison **Abonnement**/**Groupe de ressources** correspond à l'emplacement où vous avez enregistré la requête. L'élément **Nombre de machines virtuelles par système d'exploitation** enregistré apparaît désormais dans la liste **Nom de la requête**. Sélectionnez le lien titre de la requête enregistrée pour le charger dans un nouvel onglet portant le nom de cette requête. Comme il s'agit d'une requête partagée, une icône apparaît dans l'onglet situé en regard du titre pour indiquer qu'elle est partagée.

   :::image type="content" source="../media/create-share-query/show-saved-shared-query.png" alt-text="Enregistrer la nouvelle requête à l’aide du bouton Enregistrer" border="false":::

   > [!NOTE] 
   > Lorsqu'une requête enregistrée est ouverte et que l'onglet affiche son nom, le bouton **Enregistrer** la met à jour avec les modifications apportées. Pour créer une nouvelle requête enregistrée, sélectionnez **Enregistrer sous** et procédez comme si vous enregistriez une nouvelle requête.

## <a name="discover-shared-queries"></a>Découvrir des requêtes partagées

Dans la mesure où une requête partagée est une ressource Resource Manager, différentes méthodes sont disponibles pour y accéder :

- À partir de l’Explorateur Resource Graph. Sélectionnez **Ouvrir une requête** et définissez le type sur **Requête partagée**.
- À partir de la page Requêtes Resource Graph du portail.
- À partir du groupe de ressources dans lequel la requête partagée a été enregistrée.
- Par le biais d'une requête adressée à Resource Graph.

### <a name="view-resource-graph-queries"></a>Afficher les requêtes Resource Graph

Dans le portail Azure, la page Requêtes Resource Graph affiche les requêtes partagées auxquelles le compte connecté a accès. Cette page permet un filtrage par nom, abonnement, groupe de ressources et autres propriétés de la requête Resource Graph. Vous pouvez également marquer, exporter et supprimer des requêtes Resource Graph à l'aide de cette interface.

La sélection d'une requête ouvre la page de cette requête Resource Graph. Comme pour les autres ressources Resource Manager, cette page offre une vue d'ensemble interactive, avec le journal d'activité, le contrôle d'accès et les balises. Vous pouvez également appliquer un verrou de ressource à partir de cette page.

Accédez à la page Requêtes Resource Graph à partir du menu du portail en sélectionnant **Tous les services** ou en utilisant la zone de recherche Azure située en haut de toutes les pages. Recherchez et sélectionnez l’**Explorateur Resource Graph**.

### <a name="list-resource-groups-resources"></a>Répertorier les ressources des groupes de ressources

La requête Resource Graph est répertoriée avec d'autres ressources qui font partie d'un groupe de ressources.
La sélection de la requête Resource Graph ouvre la page de cette requête. Les points de suspension et les options du menu contextuel (accessible via un clic droit) fonctionnent de la même manière que sur la page Requête Resource Graph.

### <a name="query-resource-graph"></a>Interroger Resource Graph

Vous pouvez rechercher des requêtes Resource Graph en adressant une requête à Resource Graph. La requête Resource Graph suivante se limite au type `Microsoft.ResourceGraph/queries`, puis utilise `project` pour ne répertorier que le nom, l'heure de modification et la requête elle-même :

```kusto
Resources
| where type == "microsoft.resourcegraph/queries"
| project name, properties.timeModified, properties.query
```

## <a name="run-a-shared-query"></a>Exécuter une requête partagée

Une requête partagée de graphe de ressource peut être exécutée avec la syntaxe `{{shared-query-uri}}` (préversion). Pour plus d’informations, consultez [Syntaxe de requête partagée](../concepts/query-language.md#shared-query-syntax).

## <a name="delete-a-shared-query"></a>Supprimer une requête partagée

Si une requête partagée n'a plus aucune utilité, n'hésitez pas à la supprimer. En supprimant une requête partagée, vous supprimez la ressource Resource Manager correspondante. Tous les tableaux de bord sur lesquels le graphique de résultats était épinglé affichent désormais un message d'erreur. Quand ce message d’erreur s’affiche, utilisez le bouton **Supprimer du tableau de bord** pour effacer le contenu de votre tableau de bord.

Vous pouvez supprimer une requête partagée via les interfaces suivantes :
- Page Requêtes Resource Graph
- Page Requête Resource Graph
- Page **Ouvrir une requête** de l'Explorateur Resource Graph
- Page Groupes de ressources

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au terme de ce didacticiel, supprimez les requêtes privées et partagées que vous avez créées si vous ne souhaitez pas les conserver.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé des requêtes privées et partagées. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](../concepts/query-language.md)