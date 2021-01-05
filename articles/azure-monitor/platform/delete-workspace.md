---
title: Supprimer et récupérer un espace de travail Azure Log Analytics | Microsoft Docs
description: Découvrez comment supprimer votre espace de travail Log Analytics si vous en avez créé un dans votre abonnement personnel ou restructurer votre modèle d’espace de travail.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020
ms.openlocfilehash: 0858d448cf768dbe6ea48f07247725fac30da860
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95758893"
---
# <a name="delete-and-recover-azure-log-analytics-workspace"></a>Supprimer et récupérer un espace de travail Azure Log Analytics

Cet article décrit le concept de suppression réversible d'un espace de travail Azure Log Analytics, et explique comment récupérer un espace de travail supprimé.

## <a name="considerations-when-deleting-a-workspace"></a>Considérations relatives à la suppression d'un espace de travail

Lorsque vous supprimez un espace de travail Log Analytics, une opération de suppression réversible est lancée pour permettre la récupération de l'espace de travail, y compris de ses données et des agents connectés, dans un délai de 14 jours, que la suppression ait été accidentelle ou intentionnelle. À l’issue de cette période de suppression réversible, la ressource d’espace de travail et ses données deviennent irrécupérables : ses données sont mises en file d’attente pour une suppression définitive et sont complètement supprimées dans un délai de 30 jours. Le nom de l’espace de travail est « libéré » et vous pouvez l’utiliser pour créer un espace de travail.

> [!NOTE]
> Si vous souhaitez modifier le comportement de suppression réversible et supprimer définitivement votre espace de travail, suivez les étapes décrites dans [Suppression définitive de l’espace de travail](#permanent-workspace-delete).

Soyez prudent lorsque vous supprimez un espace de travail, car celui-ci peut contenir une configuration et des données importantes dont la suppression peut avoir un impact négatif sur le fonctionnement de votre service. Examinez les agents, solutions et autres services Azure qui stockent leurs données dans Log Analytics, par exemple :

* Liste des solutions de gestion
* Azure Automation
* Agents s’exécutant sur des machines virtuelles Windows et Linux
* Agents s’exécutant sur des ordinateurs Windows et Linux dans votre environnement
* System Center Operations Manager

L’opération de suppression réversible supprime la ressource de l’espace de travail, et l’autorisation des utilisateurs associés est rompue. Les utilisateurs associés à d'autres espaces de travail peuvent continuer à utiliser Log Analytics avec ceux-ci.

## <a name="soft-delete-behavior"></a>Comportement de la suppression réversible

L'opération de suppression de l'espace de travail supprime la ressource Resource Manager de l'espace de travail, mais sa configuration et ses données sont conservées pendant 14 jours, tout en donnant l'impression que l'espace de travail a été supprimé. Pendant la période de suppression réversible, tous les agents et groupes d'administration System Center Operations Manager configurés pour se rapporter à l'espace de travail conservent l'état d'orphelins. Le service fournit en outre un mécanisme de récupération de l'espace de travail supprimé (données et ressources connectées comprises) qui a essentiellement pour effet d'annuler la suppression.

> [!NOTE] 
> Les solutions installées et les services liés comme votre compte Azure Automation sont définitivement supprimés de l’espace de travail au moment de la suppression et leur récupération est impossible. Ceux-ci doivent être reconfigurés après l’opération de récupération pour rétablir l’état précédemment configuré de l’espace de travail.

Vous pouvez supprimer un espace de travail à l'aide de [PowerShell](/powershell/module/azurerm.operationalinsights/remove-azurermoperationalinsightsworkspace?view=azurermps-6.13.0&preserve-view=true), de [l’API REST](/rest/api/loganalytics/workspaces/delete) ou à partir du [portail Azure](https://portal.azure.com).

### <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Espaces de travail Log Analytics**.
3. Dans la liste des espaces de travail Log Analytics, sélectionnez un espace de travail, puis cliquez sur **Supprimer** en haut du volet central.
4. Une page de confirmation s’affiche, qui illustre l’ingestion de données dans l’espace de travail au cours de la semaine dernière. 
5. Si vous souhaitez supprimer définitivement l’espace de travail en supprimant l’option pour la récupérer ultérieurement, cochez la case **Supprimer définitivement l’espace de travail**.
6. Tapez le nom de l’espace de travail pour confirmer, puis cliquez sur **Supprimer**.

   ![Confirmer la suppression d’un espace de travail](media/delete-workspace/workspace-delete.png)

### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name"
```

## <a name="permanent-workspace-delete"></a>Suppression d’espace de travail permanent
La méthode de suppression réversible peut ne pas convenir dans certains cas, par exemple, de développement et le test, où vous devez répéter un déploiement avec les mêmes paramètres et nom d’espace de travail. En pareils cas, vous pouvez supprimer définitivement votre espace de travail et annuler la période de suppression réversible. L’opération de suppression définitive de l’espace de travail libère le nom de ce dernier, si bien que vous pouvez créer un espace de travail en utilisant le même nom.

> [!IMPORTANT]
> Utilisez l’opération de suppression définitive d’espace de travail avec précaution, car elle est irréversible et vous ne pourrez récupérer ni votre espace de travail ni ses données.

Pour supprimer définitivement un espace de travail à l’aide du portail Azure, cochez la case **Supprimer définitivement l’espace de travail** avant de cliquer sur le bouton **Supprimer**.

Pour supprimer définitivement un espace de travail à l’aide de PowerShell, ajoutez la balise '-ForceDelete' pour supprimer définitivement votre espace de travail. L’option « -ForceDelete » est actuellement disponible avec Az.OperationalInsights 2.3.0 ou une version ultérieure. 

```powershell
PS C:\>Remove-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name" -Name "workspace-name" -ForceDelete
```

## <a name="recover-workspace"></a>Récupérer un espace de travail
Lorsque vous supprimez un espace de travail Log Analytics accidentellement ou intentionnellement, le service place l’espace de travail dans un état de suppression réversible, ce qui le rend inaccessible à toute opération. Le nom de l’espace de travail supprimé est conservé pendant la période de suppression réversible et ne peut pas être utilisé pour créer un espace de travail. Après la période de suppression réversible, l’espace de travail n’est plus récupérable. Il est planifié pour une suppression définitive, tandis que son nom est libéré et peut être utilisé pour créer un nouvel espace de travail.

Pendant la période de suppression réversible, vous pouvez récupérer votre espace de travail, notamment ses agents, sa configuration et ses agents connectés. Vous devez disposer d’autorisations de contributeur sur l’abonnement et le groupe de ressources où l’espace de travail se trouvait avant l’opération de suppression réversible. La récupération de l’espace de travail est effectuée en créant un espace de travail Log Analytics avec les détails de l’espace de travail supprimé, qui inclut les éléments suivant :

- Identifiant d’abonnement
- Nom du groupe ressources
- Nom de l’espace de travail
- Région

### <a name="azure-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste de ressources, saisissez **Log Analytics**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Espaces de travail Log Analytics**. Vous voyez la liste des espaces de travail dont vous disposez dans l’étendue sélectionnée.
3. Cliquez sur **Récupérer** dans le menu supérieur gauche pour ouvrir une page répertoriant les espaces de travail en état de suppression réversible qui sont récupérables.

   ![Capture d’écran de l’écran Espaces de travail Log Analytics dans le portail Azure avec Récupérer en surbrillance dans la barre de menus.](media/delete-workspace/recover-menu.png)

4. Sélectionnez l’espace de travail, puis cliquez sur **Récupérer** pour récupérer cet espace de travail.

   ![Capture d’écran de la boîte de dialogue Récupérer les espaces de travail Log Analytics supprimés dans le portail Azure avec un espace de travail en surbrillance et le bouton Récupérer sélectionné.](media/delete-workspace/recover-workspace.png)


### <a name="powershell"></a>PowerShell
```PowerShell
PS C:\>Select-AzSubscription "subscription-name-the-workspace-was-in"
PS C:\>New-AzOperationalInsightsWorkspace -ResourceGroupName "resource-group-name-the-workspace-was-in" -Name "deleted-workspace-name" -Location "region-name-the-workspace-was-in"
```

L’espace de travail et toutes ses données sont restaurés après l’opération de récupération. Les solutions et les services liés ont été définitivement supprimés de l’espace de travail lors de la suppression de ce dernier. Ils doivent être reconfigurés pour ramener l’espace de travail à son état précédemment configuré. Certaines des données peuvent ne pas être disponibles pour la requête après la récupération de l’espace de travail jusqu’à ce que les solutions associées soient réinstallées et que leurs schémas soient ajoutés à l’espace de travail.

> [!NOTE]
> * La recréation d’un espace de travail pendant la période de suppression réversible indique que le nom de cet espace de travail est déjà utilisé. 
 
## <a name="troubleshooting"></a>Dépannage

Vous devez disposer au moins des autorisations *Contributeur Log Analytics* pour supprimer un espace de travail.

* Si vous ignorez si l’espace de travail supprimé est ou non dans un état de suppression réversible et peut être récupéré, cliquez sur [Récupérer](#recover-workspace) dans la page *Espaces de travail Log Analytics* pour afficher la liste des espaces de travail supprimés de manière réversible par abonnement. Les espaces de travail définitivement supprimés ne figurent pas dans la liste.
* Si vous obtenez le message d’erreur *Ce nom d’espace de travail est déjà utilisé* ou un *conflit* s’est produit pendant la création d’un espace de travail ; en voici les raisons possibles :
  * Le nom de l’espace de travail n’est pas disponible et qu’il est utilisé par une personne de votre organisation ou par un autre client.
  * L’espace de travail a été supprimé au cours des 14 derniers jours et son nom est réservé pour la période de suppression réversible. Pour annuler la suppression réversible et supprimer définitivement votre espace de travail pour en créer un nouveau sous le même nom, suivez ces étapes afin de récupérer d’abord l’espace de travail et effectuer la suppression définitive :<br>
    1. [Récupérez](#recover-workspace) votre espace de travail.
    2. [Supprimez définitivement](#permanent-workspace-delete) votre espace de travail.
    3. Créez un espace de travail en reprenant le même nom d’espace de travail.
* Si vous voyez un code de réponse 204 qui indique *Ressource introuvable*, la cause peut être des tentatives répétées d’utiliser l’opération de suppression de l’espace de travail. 204 est une réponse vide, ce qui signifie généralement que la ressource n’existe pas. La suppression est donc terminée sans rien faire.
  Une fois l’appel de suppression terminé sur la back end, vous pouvez restaurer l’espace de travail et terminer l’opération de suppression permanente dans l’une des méthodes suggérées précédemment.

