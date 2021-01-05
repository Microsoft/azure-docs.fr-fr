---
title: Intégration à Azure Security Center avec PowerShell
description: Ce document vous guide tout au long du processus d’intégration d’Azure Security Center à l’aide de cmdlets PowerShell.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: e400fcbf-f0a8-4e10-b571-5a0d0c3d0c67
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/02/2018
ms.author: memildin
ms.openlocfilehash: 20fce643defd12897eceb1accc7ade8b10548568
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341163"
---
# <a name="automate-onboarding-of-azure-security-center-using-powershell"></a>Automatiser l’intégration d’Azure Security Center à l’aide de PowerShell

Vous pouvez sécuriser vos charges de travail Azure par programme à l’aide du module PowerShell d’Azure Security Center.
PowerShell vous permet d’automatiser des tâches et d’éviter toute erreur humaine inhérente aux tâches manuelles. Cela est particulièrement utile dans le cadre de déploiements à grande échelle impliquant des dizaines d’abonnements avec des centaines de milliers de ressources qui doivent toutes être sécurisées dès le début.

L’intégration d’Azure Security Center à l’aide de PowerShell vous permet d’automatiser par programme l’intégration et la gestion de vos ressources Azure, ainsi que d’ajouter les contrôles de sécurité nécessaires.

Cet article fournit un exemple de script PowerShell que vous pouvez modifier et utiliser dans votre environnement pour déployer Azure Security Center vers vos abonnements. 

Dans cet exemple, nous allons activer Security Center sur un abonnement dont l’ID est d07c0080-170c-4c24-861d-9c817742786c, puis appliquer les paramètres recommandés qui fournissent un niveau élevé de protection, en activant Azure Defender qui fournit des fonctionnalités avancées de détection des menaces et de protection contre celles-ci :

1. Activer [Azure Defender](azure-defender.md). 
 
2. Définissez l’espace de travail Log Analytics auquel l’agent Log Analytics enverra les données collectées sur les machines virtuelles associées à l’abonnement. Dans cet exemple, il s’agit d’un espace de travail défini par l’utilisateur (myWorkspace).

3. Activez la fonctionnalité d’approvisionnement automatique d’agent d’Azure Security Center qui [déploie l’agent Log Analytics](security-center-enable-data-collection.md#auto-provision-mma).

5. Définissez le [CISO de l’organisation en tant que contact de sécurité pour les événements notables et alertes Security Center](security-center-provide-security-contact-details.md).

6. Assignez les [stratégies de sécurité par défaut](tutorial-security-policy.md) d’Azure Security Center.

## <a name="prerequisites"></a>Prérequis

Avant d’exécuter les cmdlets Azure Security Center, vous devez effectuer les étapes suivantes :

1. Exécutez PowerShell en tant qu’administrateur.

1. Exécutez les commandes suivantes dans PowerShell :
      
    ```Set-ExecutionPolicy -ExecutionPolicy AllSigned```

    ```Install-Module -Name Az.Security -Force```

## <a name="onboard-security-center-using-powershell"></a>Intégrer Azure Security Center à l’aide de PowerShell

1. Inscrivez vos abonnements auprès du fournisseur de ressources d’Azure Security Center :

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.Security'```

1. Facultatif : Définissez le niveau de couverture (Azure Defender activé/désactivé) des abonnements. Si la valeur n’est pas définie, Defender est désactivé :

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Set-AzSecurityPricing -Name "default" -PricingTier "Standard"```

1. Configurez un espace de travail Log Analytics auquel les agents rendront compte. Vous devez disposer d’un espace de travail Log Analytics que vous avez créé, auquel les machines virtuelles de l’abonnement rendront compte. Vous pouvez définir plusieurs abonnements rendant compte au même espace de travail. À défaut de définition, l’espace de travail par défaut est utilisé.

    ```Set-AzSecurityWorkspaceSetting -Name "default" -Scope "/subscriptions/d07c0080-170c-4c24-861d-9c817742786c" -WorkspaceId"/subscriptions/d07c0080-170c-4c24-861d-9c817742786c/resourceGroups/myRg/providers/Microsoft.OperationalInsights/workspaces/myWorkspace"```

1. Approvisionnez automatiquement l’installation de l’agent Log Analytics sur vos machines virtuelles Azure :
    
    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```
    
    ```Set-AzSecurityAutoProvisioningSetting -Name "default" -EnableAutoProvision```

    > [!NOTE]
    > Nous vous recommandons d’activer l’approvisionnement automatique pour vous assurer que vos machines virtuelles Azure sont automatiquement protégées par Azure Security Center.
    >

1. Facultatif : il est fortement recommandé de définir les contacts de sécurité pour les abonnements que vous intégrez, qui seront utilisés comme destinataires des alertes et notifications générées par Security Center :

    ```Set-AzSecurityContact -Name "default1" -Email "CISO@my-org.com" -Phone "2142754038" -AlertAdmin -NotifyOnAlert```

1. Assignez l’initiative de stratégie Azure Security Center par défaut :

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'```

    ```$Policy = Get-AzPolicySetDefinition | where {$_.Properties.displayName -EQ 'Enable Monitoring in Azure Security Center'} New-AzPolicyAssignment -Name 'ASC Default <d07c0080-170c-4c24-861d-9c817742786c>' -DisplayName 'Security Center Default <subscription ID>' -PolicySetDefinition $Policy -Scope '/subscriptions/d07c0080-170c-4c24-861d-9c817742786c'```

Vous avez correctement intégré Azure Security Center avec PowerShell.

Vous pouvez désormais utiliser ces cmdlets PowerShell avec des scripts d’automatisation pour itérer par programme dans les abonnements et ressources. Cela permet de gagner du temps et réduit la probabilité d’erreur humaine. Vous pouvez utiliser cette [exemple de script](https://github.com/Microsoft/Azure-Security-Center/blob/master/quickstarts/ASC-Samples.ps1) en tant que référence.




## <a name="see-also"></a>Voir aussi
Pour en savoir plus sur la façon d’utiliser PowerShell pour automatiser l’intégration à Azure Security Center, voir l’article suivant :

* [Az.Security](/powershell/module/az.security)

Pour en savoir plus sur Azure Security Center, voir l’article suivant :

* [Définition des stratégies de sécurité dans Azure Security Center](tutorial-security-policy.md) : découvrez comment configurer des stratégies de sécurité pour vos groupes de ressources et abonnements Azure.
* [Gestion et résolution des alertes de sécurité dans Azure Security Center](security-center-managing-and-responding-alerts.md) : découvrez comment gérer et résoudre les alertes de sécurité.