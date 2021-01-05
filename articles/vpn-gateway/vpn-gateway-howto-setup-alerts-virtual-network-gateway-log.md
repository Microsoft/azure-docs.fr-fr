---
title: 'Passerelle VPN Azure : Configurer des alertes sur des événements de journalisation de ressource de diagnostic'
description: Apprenez comment configurer des alertes basées sur des événements de journalisation de ressource de la passerelle VPN Azure avec Azure Monitor Log Analytics.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/18/2020
ms.author: alzam
ms.openlocfilehash: 70cac6ef566ef52409cd9667708c2fc297e046f7
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656612"
---
# <a name="set-up-alerts-on-resource-log-events-from-vpn-gateway"></a>Configurer des alertes sur des événements de journalisation de ressource de la passerelle VPN

Cet article vous permet de configurer des alertes basées sur des événements de journalisation de ressource de la passerelle VPN Azure avec Azure Monitor Log Analytics. 

Les journaux de ressources suivants sont disponibles dans Azure :

|***Nom** _ | _*_Description_*_ |
|--- | --- |
|GatewayDiagnosticLog | Contient des journaux de ressources pour des événements de configuration de passerelle, des modifications principales et des événements de maintenance. |
|TunnelDiagnosticLog | Contient des événements de changement d’état de tunnel. Les événements de connexion/déconnexion de tunnel affichent une brève explication du changement d’état, le cas échéant |
|RouteDiagnosticLog | Consigne les modifications apportées aux routes statiques et aux événements BGP qui se produisent sur la passerelle |
|IKEDiagnosticLog | Consigne les événements et messages de contrôle IKE sur la passerelle |
|P2SDiagnosticLog | Consigne les événements et messages de contrôle point à site sur la passerelle. Les informations relatives à la source de connexion sont fournies pour les connexions IKEv2 et OpenVPN uniquement |

## <a name="set-up-alerts-in-the-azure-portal"></a><a name="setup"></a>Configurer des alertes dans le portail Azure

L’exemple de procédure suivant crée une alerte pour un événement de déconnexion qui implique un tunnel VPN de site à site :


1. Dans le portail Azure, recherchez _ *Log Analytics** sous **Tous les services** et sélectionnez **Espaces de travail Log Analytics**.

   ![Sélections permettant d’accéder aux espaces de travail Log Analytics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert0.png "Créer")

2. Sélectionnez **Créer** dans la page **Log Analytics**.

   ![Page Log Analytics avec le bouton Créer](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert1.png  "Sélectionnez")

3. Sélectionnez **Créer** et renseignez les détails.

   ![Détails permettant la création d’un espace de travail Log Analytics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert2.png  "Sélectionnez")

4. Recherchez votre passerelle VPN dans le panneau **Monitor** > **Paramètres de diagnostic**.

   ![Sélections permettant de rechercher la passerelle VPN dans Paramètres de diagnostic](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert3.png  "Sélectionnez")

5. Pour activer les diagnostics, double-cliquez sur la passerelle, puis sélectionnez **Activer les diagnostics**.

   ![Sélections permettant l’activation des diagnostics](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert4.png  "Sélectionnez")

6. Renseignez les détails, puis vérifiez que les options **Envoyer à Log Analytics** et **TunnelDiagnosticLog** sont sélectionnées. Choisissez l’espace de travail Log Analytics que vous avez créé à l’étape 3.

   ![Cases à cocher sélectionnées](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert5.png  "Sélectionnez")

   > [!NOTE]
   > L’affichage initial des données peut prendre quelques heures.

7. Accédez à la vue d’ensemble de la ressource de passerelle réseau virtuel et sélectionnez **Alertes** à partir de l’onglet **Supervision**. Ensuite, créez une règle d’alerte ou modifiez une règle d’alerte existante.

   ![Sélections permettant la création d’une règle d’alerte](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Sélectionnez")

   ![point à site](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert6.png  "Sélectionnez")
8. Sélectionnez l’espace de travail Log Analytics et la ressource.

   ![Sélections pour l’espace de travail et la ressource](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert7.png  "Sélectionnez")

9. Sélectionnez **Recherche personnalisée dans les journaux** comme logique de signal sous **Ajouter une condition**.

   ![Sélections permettant une recherche personnalisée dans les journaux](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert8.png  "Sélectionnez")

10. Entrez la requête suivante dans la zone de texte **Requête de recherche**. Remplacez les valeurs dans <> et TimeGenerated par les valeurs appropriées.

    ```
    AzureDiagnostics
    | where Category == "TunnelDiagnosticLog"
    | where _ResourceId == tolower("<RESOURCEID OF GATEWAY>")
    | where TimeGenerated > ago(5m) 
    | where remoteIP_s == "<REMOTE IP OF TUNNEL>"
    | where status_s == "Disconnected"
    | project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId 
    | sort by TimeGenerated asc
    ```

    Définissez la valeur de seuil sur 0 et sélectionnez **Terminé**.

    ![Entrée d’une requête et sélection d’un seuil](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert9.png  "Sélectionnez")

11. Dans la page **Créer une règle**, sélectionnez **Créer** sous la section **GROUPES D’ACTIONS**. Renseignez les détails, puis sélectionnez **OK**.

    ![Détails d’un nouveau groupe d’actions](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert10.png  "Sélectionnez")

12. Dans la page **Créer une règle**, renseignez les détails pour **Personnaliser les actions** et assurez-vous que le nom correct apparaît dans la section **NOM DU GROUPE D’ACTIONS**. Sélectionnez **Créer une règle d’alerte** pour créer la règle.

    ![Sélections pour la création d’une règle](./media/vpn-gateway-howto-setup-alerts-virtual-network-gateway-log/log-alert11.png  "Sélectionnez")

## <a name="set-up-alerts-by-using-powershell"></a><a name="setuppowershell"></a>Définition d’alertes à l’aide de PowerShell

Les étapes de l’exemple suivant créent une alerte pour un événement de déconnexion qui implique un tunnel VPN de site à site.

1. Créez un espace de travail Log Analytics :

   ```powershell
   $Location           = 'westus2'
   $ResourceGroupName  = 'TestRG1'
   $Sku                = 'pergb2018'
   $WorkspaceName      = 'LogAnalyticsWS123'

   New-AzOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku $Sku -ResourceGroupName $ResourceGroupName
   ```

2. Activez les diagnostics pour la passerelle VPN :

   ```powershell
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   Set-AzDiagnosticSetting `
       -Name 'VPN tunnel' `
       -ResourceId $VpnGateway.Id `
       -WorkspaceId $Workspace.ResourceId `
       -Enabled $true `
       -Category 'TunnelDiagnosticLog'
   ```

3. Créez un groupe d’actions.

   Ce code crée un groupe d’actions qui envoie une notification par courrier électronique lorsqu’une alerte est déclenchée :

   ```powershell
   $ActionGroupName            = 'EmailAdmins'   # Max. 60 characters long
   $ActionGroupShortName       = 'EmailAdmins'   # Max. 12 characters long
   $ActionGroupReceiverName    = 'My receiver Name'
   $EmailAddress               = 'xyz@contoso.com'
   $ResourceGroupName          = 'TestRG1'

   $ActionGroupReceiver = New-AzActionGroupReceiver -Name $ActionGroupReceiverName -UseCommonAlertSchema -EmailReceiver -EmailAddress $EmailAddress

   Set-AzActionGroup `
      -ResourceGroupName $ResourceGroupName `
      -Name $ActionGroupName `
      -ShortName $ActionGroupShortName `
      -Receiver @($ActionGroupReceiver)
   ```

4. Créer une règle d’alerte basée sur une recherche de journal personnalisée :

   ```powershell
   $ActionGroupName    = 'EmailAdmins'
   $EmailSubject       = 'Redmond VPN tunnel is disconnected'
   $Location           = 'westus2'
   $RemoteIp           = '104.42.209.46'
   $ResourceGroupName  = 'TestRG1'
   $VpnGatewayName     = 'VNet1GW'
   $WorkspaceName      = 'LogAnalyticsWS123'

   $VpnGateway         = Get-AzVirtualNetworkGateway -Name $VpnGatewayName -ResourceGroupName $ResourceGroupName
   $Workspace          = Get-AzOperationalInsightsWorkspace -Name $WorkspaceName -ResourceGroupName $ResourceGroupName

   $Query = @"
   AzureDiagnostics |
   where Category == "TunnelDiagnosticLog" |
   where TimeGenerated > ago(5m) |
   where _ResourceId == tolower("$($VpnGateway.id)") |
   where remoteIP_s == "$($RemoteIp)" |
   where status_s == "Disconnected" |
   project TimeGenerated, OperationName, instance_s, Resource, ResourceGroup, _ResourceId |
   sort by TimeGenerated asc
   "@

   $Source             = New-AzScheduledQueryRuleSource -Query $Query -DataSourceId $Workspace.ResourceId
   $Schedule           = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 5 -TimeWindowInMinutes 5
   $TriggerCondition   = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator 'GreaterThan' -Threshold 0

   $ActionGroup        = Get-AzActionGroup -ResourceGroupName $ResourceGroupName -Name $ActionGroupName
   $AznsActionGroup    = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup $ActionGroup.Id -EmailSubject $EmailSubject
   $AlertingAction     = New-AzScheduledQueryRuleAlertingAction -AznsAction $AznsActionGroup -Severity '1' -Trigger $TriggerCondition

   New-AzScheduledQueryRule `
       -ResourceGroupName $ResourceGroupName `
       -Location $Location `
       -Action $AlertingAction `
       -Enabled $true `
       -Description 'The tunnel between Azure and Redmond with IP address 104.42.209.46 is disconnected' `
       -Schedule $Schedule `
       -Source $Source `
       -Name 'The Azure to Redmond tunnel is disconnected'
   ```

## <a name="next-steps"></a>Étapes suivantes

Pour configurer des alertes sur les métriques de tunnel, consultez [Configurer des alertes sur les métriques de passerelle VPN](vpn-gateway-howto-setup-alerts-virtual-network-gateway-metric.md).
