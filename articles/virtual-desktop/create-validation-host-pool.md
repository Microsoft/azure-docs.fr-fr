---
title: Mises à jour du service de pool d'hôtes Windows Virtual Desktop - Azure
description: Comment créer un pool d’hôtes de validation pour surveiller les mises à jour de service avant de déployer les mises à jour en production.
author: Heidilohr
ms.topic: tutorial
ms.date: 12/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: ead4c0aa7d8d71642fd8a4635edbabcafee5b6c2
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563243"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates"></a>Tutoriel : Créer un pool d’hôtes pour valider les mises à jour de service

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/create-validation-host-pool-2019.md).

Les pools d’hôtes comprennent une ou plusieurs machines virtuelles identiques situées dans un environnement Windows Virtual Desktop. Nous vous recommandons vivement de créer un pool d’hôtes de validation dans lequel les mises à jour de service seront appliquées en premier. Cela vous permet de superviser les mises à jour de service avant que le service ne les applique à votre environnement standard ou non validé. Sans pool d’hôtes de validation, vous risquez de ne pas détecter les modifications qui génèrent des erreurs, ce qui peut entraîner des temps d’arrêt pour les utilisateurs de votre environnement standard.

Pour veiller à ce que vos applications fonctionnent avec les dernières mises à jour, le pool d’hôtes de validation doit également être aussi semblable que possible aux pools d’hôtes de votre environnement non validé. Les utilisateurs sont invités à se connecter aussi fréquemment au pool d’hôtes de validation qu’au pool d’hôtes standard. Si vous disposez de tests automatisés sur votre pool d’hôtes, vous devez inclure ces tests au pool d’hôtes de validation.

Vous pouvez déboguer les problèmes dans le pool d’hôtes de validation à l'aide de [la fonctionnalité de diagnostic](diagnostics-role-service.md) ou des [articles de résolution des problèmes Windows Virtual Desktop](troubleshoot-set-up-overview.md).

>[!NOTE]
> Nous vous recommandons d'utiliser le pool d’hôtes de validation pour tester toutes les mises à jour à venir.

>[!IMPORTANT]
>Windows Virtual Desktop avec l’intégration Azure Resource Management a actuellement des difficultés à activer et désactiver les environnements de validation. Nous mettrons à jour cet article quand nous aurons résolu le problème.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, suivez les instructions fournies dans [Configurer le module PowerShell Windows Virtual Desktop](powershell-module.md) pour configurer votre module PowerShell et vous connecter à Azure.

## <a name="create-your-host-pool"></a>Créer votre pool d'hôtes

Vous pouvez créer un pool d'hôtes en suivant les instructions de ces articles :
- [Tutoriel : Créer un pool d’hôtes avec la Place de marché Azure](create-host-pools-azure-marketplace.md)
- [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell.md)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Définir votre pool d’hôtes en tant que pool d’hôtes de validation

Exécutez les cmdlets PowerShell suivantes pour définir le nouveau pool d’hôtes en tant que pool d’hôtes de validation. Remplacez les valeurs entre crochets par les valeurs correspondant à votre session :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -ValidationEnvironment:$true
```

Exécutez la cmdlet PowerShell suivante pour vérifier que la propriété de validation a été définie. Remplacez les valeurs entre crochets par les valeurs correspondant à votre session.

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | Format-List
```

Les résultats de la cmdlet doivent se présenter comme suit :

```powershell
    HostPoolName        : hostpoolname
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty   : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnvironment : True
```

## <a name="enable-your-validation-environment-with-the-azure-portal"></a>Activez votre environnement de validation avec le portail Azure

Vous pouvez également utiliser le portail Azure pour activer votre environnement de validation.

Pour utiliser le portail Azure pour configurer votre pool d’hôtes de validation :

1. Connectez-vous au portail Azure sur <https://portal.azure.com>.
2. Recherchez et sélectionnez **Windows Virtual Desktop**.
3. Dans la page Windows Virtual Desktop, sélectionnez **Pools d’hôtes**.
4. Sélectionnez le nom du pool d’hôtes que vous souhaitez modifier.
5. Sélectionner **Propriétés**.
6. Dans le champ d’environnement de validation, sélectionnez **Oui** pour activer l’environnement de validation.
7. Sélectionnez **Enregistrer**. Les nouveaux paramètres seront appliqués.

## <a name="update-schedule"></a>Mettre à jour la planification

Les mises à jour de service sont mensuelles. S’il existe des problèmes majeurs, les mises à jour critiques sont fournies à un rythme plus fréquent.

S’il existe des mises à jour de service, vérifiez que vous avez au moins un petit groupe d’utilisateurs qui se connectent chaque jour pour valider l’environnement. Nous vous recommandons de consulter régulièrement notre site [TechCommunity](https://techcommunity.microsoft.com/t5/forums/searchpage/tab/message?filter=location&q=wvdupdate&location=forum-board:WindowsVirtualDesktop&sort_by=-topicPostDate&collapse_discussion=true) et de suivre les publications avec WVDUPdate pour rester informé des mises à jour de service.

## <a name="next-steps"></a>Étapes suivantes

Un pool d’hôtes de validation étant créé, vous pouvez apprendre à utiliser Azure Service Health pour superviser votre déploiement de Windows Virtual Desktop.

> [!div class="nextstepaction"]
> [Configurer des alertes de service](./set-up-service-alerts.md)
