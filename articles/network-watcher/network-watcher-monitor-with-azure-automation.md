---
title: Détecter un problème sur les passerelles VPN et les surveiller – Azure Automation
titleSuffix: Azure Network Watcher
description: Cet article explique comment diagnostiquer la connectivité locale avec Azure Automation et Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: af671996722524de9af1a90ae8dfde27f814c8c2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011810"
---
# <a name="monitor-vpn-gateways-with-network-watcher-troubleshooting"></a>Surveiller les passerelles VPN à l’aide de la résolution des problèmes Network Watcher

Pour fournir des services fiables aux clients, il est essentiel d’obtenir des informations détaillées sur les performances de votre réseau. Il est donc essentiel de détecter rapidement les conditions d’indisponibilité du réseau et d’effectuer des actions correctives pour atténuer ces conditions. Azure Automation vous permet d’implémenter et d’exécuter une tâche dans un environnement de programmation via des runbooks. L’utilisation d’Azure Automation crée un environnement parfait pour l’exécution d’une surveillance réseau proactive et continue et d’alertes.

## <a name="scenario"></a>Scénario

Le scénario illustré dans l’image suivante est une application multicouche, avec une connectivité locale établie à l’aide d’un tunnel et d’une passerelle VPN. S’assurer que la passerelle VPN est opérationnelle est essentiel pour le bon fonctionnement des applications.

Un runbook est créé avec un script pour vérifier l’état de la connexion du tunnel VPN, avec l’API de résolution des problèmes des ressources . Si l’état n’est pas intègre, un déclencheur d’e-mail est envoyé aux administrateurs.

![Exemple de scénario][scenario]

Ce scénario va :

- Créer un runbook appelant l’applet de commande `Start-AzureRmNetworkWatcherResourceTroubleshooting` pour résoudre les problèmes liés à l’état de la connexion
- Associer une planification au runbook

## <a name="before-you-begin"></a>Avant de commencer

Avant de réaliser ce scénario, vous devez disposer des éléments suivants :

- Un compte Azure Automation dans Azure. Assurez-vous que le compte automation a les derniers modules et qu’il a également le module AzureRM.Network. Le module AzureRM.Network est disponible dans la galerie des modules pour l’ajouter à votre compte automation.
- Vous devez avoir configuré des informations d’identification dans Azure Automation. Pour en savoir plus, consultez l’article [Sécurité dans Azure Automation](../automation/automation-security-overview.md).
- Un serveur SMTP valide (Microsoft 365, votre messagerie locale ou autre) et les informations d'identification définies dans Azure Automation.
- Une passerelle de réseau virtuel configurée dans Azure.
- Un compte de stockage existant avec un conteneur existant pour y stocker les journaux d’activité.

> [!NOTE]
> L’infrastructure décrite dans l’image précédente est présentée à titre d’illustration. Elle n’a pas été créée avec la procédure indiquée dans cet article.

### <a name="create-the-runbook"></a>Créer le runbook

La première étape de configuration de l’exemple consiste à créer le runbook. Cet exemple utilise un compte d’identification. Pour en savoir plus sur les comptes d’identification, consultez l’article [Authentifier des Runbooks avec un compte d’identification Azure](../automation/manage-runas-account.md).

### <a name="step-1"></a>Étape 1

Accédez à Azure Automation dans le [portail Azure](https://portal.azure.com) et cliquez sur **Runbooks**.

![présentation du compte Automation][1]

### <a name="step-2"></a>Étape 2

Cliquez sur **Ajouter un runbook** pour démarrer le processus de création du runbook.

![panneau de runbooks][2]

### <a name="step-3"></a>Étape 3 :

Sous **Création rapide**, cliquez sur **Créer un runbook** pour créer le runbook.

![panneau ajouter un runbook][3]

### <a name="step-4"></a>Étape 4

Dans cette étape, nous nommons le runbook ; dans cet exemple, il est appelé **Get-VPNGatewayStatus**. Il est important de donner un nom descriptif au runbook et il est recommandé de lui attribuer un nom qui suive les normes d’affectation de noms PowerShell standard. Le type de runbook correspondant à cet exemple est **PowerShell**. Les autres options sont Graphique, Flux de travail PowerShell et Flux de travail PowerShell graphique.

![panneau runbook][4]

### <a name="step-5"></a>Étape 5

Dans cette étape, le runbook est créé. L’exemple de code suivant fournit tout le code nécessaire pour l’exemple. Les éléments du code qui contiennent \<value\> doivent être remplacés par les valeurs de votre abonnement.

Utilisez le code suivant et cliquez sur **Enregistrer**.

```powershell
# Set these variables to the proper values for your environment
$automationCredential = "<work or school account>"
$fromEmail = "<from email address>"
$toEmail = "<to email address>"
$smtpServer = "<smtp.office365.com>"
$smtpPort = 587
$runAsConnectionName = "<AzureRunAsConnection>"
$subscriptionId = "<subscription id>"
$region = "<Azure region>"
$vpnConnectionName = "<vpn connection name>"
$vpnConnectionResourceGroup = "<resource group name>"
$storageAccountName = "<storage account name>"
$storageAccountResourceGroup = "<resource group name>"
$storageAccountContainer = "<container name>"

# Get credentials for work or school account
$cred = Get-AutomationPSCredential -Name $automationCredential

# Get the connection "AzureRunAsConnection "
$servicePrincipalConnection=Get-AutomationConnection -Name $runAsConnectionName

"Logging in to Azure..."
Connect-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $servicePrincipalConnection.TenantId `
    -ApplicationId $servicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint
"Setting context to a specific subscription"
Set-AzureRmContext -SubscriptionId $subscriptionId

$nw = Get-AzurermResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $region }
$networkWatcher = Get-AzureRmNetworkWatcher -Name $nw.Name -ResourceGroupName $nw.ResourceGroupName
$connection = Get-AzureRmVirtualNetworkGatewayConnection -Name $vpnConnectionName -ResourceGroupName $vpnConnectionResourceGroup
$sa = Get-AzureRmStorageAccount -Name $storageAccountName -ResourceGroupName $storageAccountResourceGroup 
$storagePath = "$($sa.PrimaryEndpoints.Blob)$($storageAccountContainer)"
$result = Start-AzureRmNetworkWatcherResourceTroubleshooting -NetworkWatcher $networkWatcher -TargetResourceId $connection.Id -StorageId $sa.Id -StoragePath $storagePath

if($result.code -ne "Healthy")
    {
        $body = "Connection for $($connection.name) is: $($result.code) `n$($result.results[0].summary) `nView the logs at $($storagePath) to learn more."
        Write-Output $body
        $subject = "$($connection.name) Status"
        Send-MailMessage `
        -To $toEmail `
        -Subject $subject `
        -Body $body `
        -UseSsl `
        -Port $smtpPort `
        -SmtpServer $smtpServer `
        -From $fromEmail `
        -BodyAsHtml `
        -Credential $cred
    }
else
    {
    Write-Output ("Connection Status is: $($result.code)")
    }
```

### <a name="step-6"></a>Étape 6

Une fois le runbook enregistré, vous devez lui associer une planification pour automatiser son démarrage. Pour démarrer le processus, cliquez sur **Planification**.

![Étape 6][6]

## <a name="link-a-schedule-to-the-runbook"></a>Associer une planification au runbook

Vous devez créer une planification. Cliquez sur **Associer une planification à votre runbook**.

![Étape 7][7]

### <a name="step-1"></a>Étape 1

Dans le panneau **Planification**, cliquez sur **Créer une planification**.

![Étape 8][8]

### <a name="step-2"></a>Étape 2

Dans le panneau **Nouvelle planification**, renseignez les informations de planification. Les valeurs qui peuvent être définies figurent dans la liste suivante :

- **Nom** : nom convivial de la planification.
- **Description** : description de la planification.
- **Démarre** : cette valeur est une combinaison de la date, de l’heure et du fuseau horaire qui constituent le moment auquel la planification est déclenchée.
- **Périodicité** : cette valeur détermine la répétition des planifications.  Les valeurs valides sont **Une fois** ou **Périodique**.
- **Tous les** : intervalle de périodicité de la planification exprimé en heures, jours, semaines ou mois.
- **Définir l’expiration** : la valeur détermine si la planification doit arriver à expiration ou non. Peut être définie sur **Oui** ou **Non**. Une date et une heure valides doivent être fournies si la valeur Oui est choisie.

> [!NOTE]
> Si vous devez exécuter un runbook plus souvent que toutes les heures, vous devez créer plusieurs planifications à différents intervalles (autrement dit 15, 30 et 45 minutes après l’heure).

![Étape 9][9]

### <a name="step-3"></a>Étape 3 :

Cliquez sur Enregistrer pour enregistrer la planification dans le runbook.

![Étape 10][10]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous comprenez comment intégrer la résolution des problèmes Network Watcher à Azure Automation, découvrez comment déclencher des captures de paquets sur des alertes de machine virtuelle en consultant l’article [Create an alert triggered packet capture with Azure Network Watcher (Créer une capture de paquets déclenchée par alerte avec Azure Network Watcher)](network-watcher-alert-triggered-packet-capture.md).

<!-- images -->
[scenario]: ./media/network-watcher-monitor-with-azure-automation/scenario.png
[1]: ./media/network-watcher-monitor-with-azure-automation/figure1.png
[2]: ./media/network-watcher-monitor-with-azure-automation/figure2.png
[3]: ./media/network-watcher-monitor-with-azure-automation/figure3.png
[4]: ./media/network-watcher-monitor-with-azure-automation/figure4.png
[5]: ./media/network-watcher-monitor-with-azure-automation/figure5.png
[6]: ./media/network-watcher-monitor-with-azure-automation/figure6.png
[7]: ./media/network-watcher-monitor-with-azure-automation/figure7.png
[8]: ./media/network-watcher-monitor-with-azure-automation/figure8.png
[9]: ./media/network-watcher-monitor-with-azure-automation/figure9.png
[10]: ./media/network-watcher-monitor-with-azure-automation/figure10.png