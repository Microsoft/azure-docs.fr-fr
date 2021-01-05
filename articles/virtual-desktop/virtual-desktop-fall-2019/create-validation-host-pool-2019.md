---
title: Mises à jour du service de pool d’hôtes Windows Virtual Desktop (classique) - Azure
description: Apprenez à créer un pool d’hôtes de validation dans Windows Virtual Desktop (classique) pour superviser les mises à jour de service avant de déployer les mises à jour en production.
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 291f1e8b8870257c233dc32894ff49b26c0a3501
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91323527"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates-in-windows-virtual-desktop-classic"></a>Tutoriel : Créer un pool d’hôtes pour valider les mises à jour du service dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer les objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../create-validation-host-pool.md).

Les pools d'hôtes sont des ensembles d'une ou de plusieurs machines virtuelles identiques dans des environnements de locataires Windows Virtual Desktop. Nous vous recommandons de créer un pool d’hôtes de validation dans lequel les mises à jour de service seront appliquées en premier. Cela vous permet de superviser les mises à jour de service avant que le service ne les applique à votre environnement standard ou non validé. Sans pool d’hôtes de validation, vous risquez de ne pas détecter les modifications qui introduisent des erreurs, ce qui peut entraîner des temps d’arrêt pour les utilisateurs de votre environnement de production.

Pour veiller à ce que vos applications fonctionnent avec les dernières mises à jour, le pool d’hôtes de validation doit également être aussi semblable que possible aux pools d’hôtes de votre environnement non validé. Les utilisateurs sont invités à se connecter aussi fréquemment au pool d’hôtes de validation qu’au pool d’hôtes standard. Si vous disposez de tests automatisés sur votre pool d’hôtes, vous devez inclure ces tests au pool d’hôtes de validation.

Vous pouvez déboguer les problèmes dans le pool d’hôtes de validation à l'aide de [la fonctionnalité de diagnostic](diagnostics-role-service-2019.md) ou des [articles de résolution des problèmes Windows Virtual Desktop](troubleshoot-set-up-overview-2019.md).

>[!NOTE]
> Nous vous recommandons d'utiliser le pool d’hôtes de validation pour tester toutes les mises à jour à venir.

Avant de commencer, si vous ne l’avez pas déjà fait, [téléchargez et importez le module PowerShell Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/). Exécutez ensuite l’applet de commande suivante pour vous connecter à votre compte :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="create-your-host-pool"></a>Créer votre pool d'hôtes

Vous pouvez créer un pool d'hôtes en suivant les instructions de ces articles :
- [Tutoriel : Créer un pool d’hôtes avec la Place de marché Azure](create-host-pools-azure-marketplace-2019.md)
- [Créer un pool d’hôtes avec le modèle Azure Resource Manager](create-host-pools-arm-template.md)
- [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell-2019.md)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Définir votre pool d’hôtes en tant que pool d’hôtes de validation

Exécutez les cmdlets PowerShell suivantes pour définir le nouveau pool d’hôtes en tant que pool d’hôtes de validation. Remplacez les valeurs entre guillemets par les valeurs correspondant à votre session :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
Set-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool" -ValidationEnv $true
```

Exécutez la cmdlet PowerShell suivante pour vérifier que la propriété de validation a été définie. Remplacez les valeurs entre guillemets par les valeurs correspondant à votre session.

```powershell
Get-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool"
```

Les résultats de la cmdlet doivent se présenter comme suit :

```
    TenantName          : contoso
    TenantGroupName     : Default Tenant Group
    HostPoolName        : contosoHostPool
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty    : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnv       : True
    Ring                :
```

## <a name="update-schedule"></a>Mettre à jour la planification

Les mises à jour de service sont mensuelles. S’il existe des problèmes majeurs, les mises à jour critiques sont fournies à un rythme plus fréquent.

## <a name="next-steps"></a>Étapes suivantes

Un pool d’hôtes de validation étant créé, vous pouvez apprendre à utiliser Azure Service Health pour superviser votre déploiement de Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Configurer des alertes de service](set-up-service-alerts-2019.md)
