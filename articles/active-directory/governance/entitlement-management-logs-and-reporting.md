---
title: Archivage et rapport avec Azure Monitor – Gestion des droits d’utilisation Azure AD
description: Découvrez comment archiver des journaux et créer des rapports avec Azure Monitor dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: af9921510ba11ab1bd7d0664b2c4c19e042864d3
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746641"
---
# <a name="archive-logs-and-reporting-on-azure-ad-entitlement-management-in-azure-monitor"></a>Archiver les journaux et créer des rapports sur la gestion des droits d’utilisation Azure AD dans Azure Monitor

Azure AD stocke les événements d’audit jusqu’à un maximum de 30 jours dans le journal d’audit. Toutefois, vous pouvez conserver les données d’audit au-delà de la période de conservation par défaut fournie dans [Pendant combien de temps les données de rapport sont-elles conservées par Azure AD ?](../reports-monitoring/reference-reports-data-retention.md) en les routant vers un compte de stockage Azure ou en utilisant Azure Monitor. Vous pouvez ensuite utiliser des classeurs, ainsi que des rapports et des requêtes personnalisés sur ces données.


## <a name="configure-azure-ad-to-use-azure-monitor"></a>Configurer Azure AD pour utiliser Azure Monitor
Avant d’utiliser les classeurs Azure Monitor, vous devez configurer Azure AD pour envoyer une copie de ses journaux d’audit à Azure Monitor.

L’archivage des journaux d’audit Azure AD vous demande d’avoir Azure Monitor dans un abonnement Azure. Vous pouvez en savoir plus sur les prérequis et les coûts estimés de l’utilisation d’Azure Monitor dans [Journaux d’activité Azure AD dans Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md).

**Rôle prérequis** : Administrateur général

1. Connectez-vous au portail Azure en tant qu’utilisateur doté du rôle d’administrateur général. Vérifiez que vous avez accès au groupe de ressources contenant l’espace de travail Azure Monitor.
 
1. Sélectionnez **Azure Active Directory**, puis cliquez sur **Paramètres de diagnostic** sous Surveillance dans le menu de navigation de gauche. Vérifiez s’il existe déjà un paramètre pour envoyer les journaux d’audit à cet espace de travail.

1. Si aucun paramètre n’est encore défini, cliquez sur **Ajouter un paramètre de diagnostic**. Utilisez les instructions fournies dans l’article [Intégrer des journaux d’activité Azure AD aux journaux d’activité Azure Monitor](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md#send-logs-to-azure-monitor) pour envoyer le journal d’audit Azure AD à l’espace de travail Azure Monitor.

    ![Volet Paramètres de diagnostic](./media/entitlement-management-logs-and-reporting/audit-log-diagnostics-settings.png)


1. Une fois le journal envoyé à Azure Monitor, sélectionnez **Espaces de travail Log Analytics**, puis sélectionnez l’espace de travail qui contient les journaux d’audit Azure AD.

1. Sélectionnez **Utilisation et estimation des coûts**, puis cliquez sur **Conservation des données**. Réglez le curseur sur le nombre de jours où vous souhaitez conserver les données pour satisfaire à vos exigences d’audit.

    ![Volet Espaces de travail Log Analytics](./media/entitlement-management-logs-and-reporting/log-analytics-workspaces.png)

1. Plus tard, pour voir la plage de dates conservées dans votre espace de travail, vous pouvez utiliser le classeur *Plage de dates du journal archivé* :  
    
    1. Sélectionnez **Azure Active Directory**, puis cliquez sur **Classeurs**. 
    
    1. Développez la section **Résolution des problèmes d’Azure Active Directory**, puis cliquez sur **Plage de dates du journal archivé**. 


## <a name="view-events-for-an-access-package"></a>Visualiser les événements pour un package d’accès  

Pour voir les événements d’un package d’accès, vous devez avoir accès à l’espace de travail Azure Monitor sous-jacent (pour plus d’informations, consultez [Gérer l’accès aux données du journal et les espaces de travail dans Azure Monitor](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions)) et dans un des rôles suivants : 

- Administrateur général  
- Administrateur de sécurité  
- Lecteur de sécurité  
- Lecteur de rapport  
- Administrateur d’application  

Utilisez la procédure suivante pour voir les événements : 

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, puis cliquez sur **Classeurs**. Si vous n’avez qu’un seul abonnement, passez à l’étape 3. 

1. Si vous avez plusieurs abonnements, sélectionnez l’abonnement qui contient l’espace de travail.  

1. Sélectionnez le classeur nommé *Activité du package d’accès*. 

1. Dans ce classeur, sélectionnez une plage de temps (changez-la en **Tout** si vous n’êtes pas sûr), puis sélectionnez un ID de package d’accès dans la liste déroulante de tous les packages d’accès ayant eu de l’activité pendant cette période. Les événements liés au package d’accès qui se sont produits pendant l’intervalle de temps sélectionné s’affichent.  

    ![Visualiser les événements du package d’accès](./media/entitlement-management-logs-and-reporting/view-events-access-package.png) 

    Chaque ligne comprend l’heure, l’ID de package d’accès, le nom de l’opération, l’ID d’objet, l’UPN et le nom d’affichage de l’utilisateur qui a démarré l’opération.  Des détails supplémentaires sont inclus dans le JSON.   


## <a name="create-custom-azure-monitor-queries-using-the-azure-portal"></a>Créer des requêtes Azure Monitor personnalisées à l’aide du portail Azure
Vous pouvez créer vos propres requêtes sur les événements d’audit Azure AD, y compris les événements de gestion des droits d’utilisation.  

1. Dans la page Azure Active Directory du portail Azure, cliquez sur **Journaux** sous la section Surveillance du menu de navigation gauche pour créer une page de requête.

1. Votre espace de travail doit être affiché dans l’angle supérieur gauche de la page de requête. Si vous avez plusieurs espaces de travail Azure Monitor et que l’espace de travail que vous utilisez pour stocker les événements d’audit Azure AD ne s’affiche pas, cliquez sur **Sélectionner la portée**. Sélectionnez alors l’abonnement et l’espace de travail appropriés.

1. Ensuite, dans la zone de texte de requête, supprimez la chaîne « search * » et remplacez-la par la requête suivante :

    ```
    AuditLogs | where Category == "EntitlementManagement"
    ```

1. Cliquez ensuite sur **Exécuter**. 

    ![Cliquer sur Exécuter pour démarrer la requête](./media/entitlement-management-logs-and-reporting/run-query.png)

Le tableau affiche les événements du journal d’audit pour la gestion des droits d’utilisation au cours de la dernière heure par défaut. Vous pouvez changer le paramètre « Plage horaire » pour voir des événements plus anciens. Toutefois, le changement de ce paramètre entraîne uniquement l’affichage des événements qui se sont produits après la configuration d’Azure AD pour l’envoi des événements à Azure Monitor.

Si vous souhaitez connaître les événements d’audit les plus anciens et les plus récents conservés dans Azure Monitor, utilisez la requête suivante :

```
AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type
```

Pour plus d’informations sur les colonnes stockées pour les événements d’audit dans Azure Monitor, consultez [Interpréter le schéma des journaux d’audit Azure Active Directory dans Azure Monitor](../reports-monitoring/reference-azure-monitor-audit-log-schema.md).

## <a name="create-custom-azure-monitor-queries-using-azure-powershell"></a>Créer des requêtes Azure Monitor personnalisées à l’aide d’Azure PowerShell

Vous pouvez accéder aux journaux via PowerShell après avoir configuré Azure AD pour envoyer les journaux à Azure Monitor. Ensuite, envoyez des requêtes à partir de scripts ou de la ligne de commande PowerShell, sans avoir besoin d’être un administrateur général dans le locataire. 

### <a name="ensure-the-user-or-service-principal-has-the-correct-role-assignment"></a>S’assurer que l’utilisateur ou le principal de service dispose de l’attribution de rôle appropriée

Assurez-vous que vous, l’utilisateur ou le principal de service, qui vous authentifierez auprès d’Azure AD, détenez le rôle Azure approprié dans l’espace de travail Log Analytics. Les options de rôle sont Lecteur Log Analytics et Contributeur Log Analytics. Si vous détenez déjà l’un de ces rôles, passez à la section [Récupérer l’ID Log Analytics avec un abonnement Azure](#retrieve-log-analytics-id-with-one-azure-subscription).

Pour définir l’attribution de rôle et créer une requête, effectuez les étapes suivantes :

1. Dans le portail Azure, recherchez l’[espace de travail Log Analytics](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.OperationalInsights%2Fworkspaces
).

1. Sélectionnez **Contrôle d’accès (IAM)** .

1. Ensuite, cliquez sur **Ajouter** pour ajouter une attribution de rôle.

    ![Ajouter une attribution de rôle](./media/entitlement-management-logs-and-reporting/workspace-set-role-assignment.png)

### <a name="install-azure-powershell-module"></a>Installer le module Azure PowerShell

Une fois que vous disposez de l’attribution de rôle appropriée, lancez PowerShell et [installez le module Azure PowerShell](/powershell/azure/install-az-ps?view=azps-3.3.0) (si vous ne l’avez pas encore fait) en tapant :

```azurepowershell
install-module -Name az -allowClobber -Scope CurrentUser
```
    
Vous êtes maintenant prêt à vous authentifier sur Azure AD et à récupérer l’ID de l’espace de travail Log Analytics que vous interrogez.

### <a name="retrieve-log-analytics-id-with-one-azure-subscription"></a>Récupérer l’ID Log Analytics avec un abonnement Azure
Si vous avez un seul abonnement Azure et un seul espace de travail Log Analytics, tapez la commande suivante pour vous authentifier auprès d’Azure AD, vous connecter à cet abonnement et récupérer cet espace de travail :
 
```azurepowershell
Connect-AzAccount
$wks = Get-AzOperationalInsightsWorkspace
```
 
### <a name="retrieve-log-analytics-id-with-multiple-azure-subscriptions"></a>Récupérer l’ID Log Analytics avec plusieurs abonnements Azure

 [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) fonctionne dans un abonnement à la fois. Par conséquent, si vous avez plusieurs abonnements Azure, vous devez vous assurer de vous connecter à celui qui dispose de l’espace de travail Log Analytics avec les journaux Azure AD. 
 
 Les applets de commande suivantes affichent la liste des abonnements et recherchent l’ID de l’abonnement qui dispose de l’espace de travail Log Analytics :
 
```azurepowershell
Connect-AzAccount
$subs = Get-AzSubscription
$subs | ft
```
 
Vous pouvez réauthentifier votre session PowerShell et l’associer à cet abonnement à l’aide d’une commande telle que `Connect-AzAccount –Subscription $subs[0].id`. Pour en savoir plus sur l’authentification sur Azure à partir de PowerShell, y compris de manière non interactive, consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps?view=azps-3.3.0&viewFallbackFrom=azps-2.5.0
).

Si vous avez plusieurs espaces de travail Log Analytics dans cet abonnement, l’applet de commande [Get-AzOperationalInsightsWorkspace](/powershell/module/Az.OperationalInsights/Get-AzOperationalInsightsWorkspace) retourne la liste des espaces de travail. Vous pouvez alors trouver celui qui contient les journaux Azure AD. Le champ `CustomerId` retourné par cette applet de commande est identique à la valeur de l’« ID d’espace de travail » affichée dans le portail Azure, dans la vue d’ensemble de l’espace de travail Log Analytics.
 
```powershell
$wks = Get-AzOperationalInsightsWorkspace
$wks | ft CustomerId, Name
```

### <a name="send-the-query-to-the-log-analytics-workspace"></a>Envoyer la requête à l’espace de travail Log Analytics
Enfin, une fois que vous disposez d’un espace de travail identifié, vous pouvez utiliser [Invoke-AzOperationalInsightsQuery](/powershell/module/az.operationalinsights/Invoke-AzOperationalInsightsQuery?view=azps-3.3.0
) pour envoyer une requête Kusto à cet espace de travail. Ces requêtes sont rédigées dans le [langage de requête Kusto](/azure/kusto/query/).
 
Par exemple, vous pouvez récupérer la plage de dates des enregistrements d’événements d’audit à partir de l’espace de travail Log Analytics, avec les applets de commande PowerShell pour envoyer une requête comme suit :
 
```powershell
$aQuery = "AuditLogs | where TimeGenerated > ago(3653d) | summarize OldestAuditEvent=min(TimeGenerated), NewestAuditEvent=max(TimeGenerated) by Type"
$aResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $aQuery
$aResponse.Results |ft
```

Vous pouvez également récupérer des événements de gestion des droits d’utilisation à l’aide d’une requête telle que :

```azurepowershell
$bQuery = 'AuditLogs | where Category == "EntitlementManagement"'
$bResponse = Invoke-AzOperationalInsightsQuery -WorkspaceId $wks[0].CustomerId -Query $Query
$bResponse.Results |ft 
```

## <a name="next-steps"></a>Étapes suivantes :
- [Créer des rapports interactifs avec les classeurs Azure Monitor](../../azure-monitor/platform/workbooks-overview.md)