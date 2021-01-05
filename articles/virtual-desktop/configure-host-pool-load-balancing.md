---
title: Configurer l’équilibrage de charge de Windows Virtual Desktop – Azure
description: Comment configurer la méthode d’équilibrage de charge pour un environnement Windows Virtual Desktop ?
author: Heidilohr
ms.topic: how-to
ms.date: 10/12/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 2c57ac10fbd318dd4bbb2dc86457e186dd824834
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91951653"
---
# <a name="configure-the-windows-virtual-desktop-load-balancing-method"></a>Configurer la méthode d’équilibrage de charge de Windows Virtual Desktop

La configuration de la méthode d’équilibrage de charge pour un pool d’hôtes vous permet d’ajuster l’environnement Windows Virtual Desktop afin de mieux répondre à vos besoins.

>[!NOTE]
> Cela ne s’applique pas à un pool d'hôtes de bureau persistant, car les utilisateurs disposent toujours d'un mappage 1:1 vers un hôte de session au sein du pool d’hôtes.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez suivi les instructions de l’article [Configurer le module PowerShell pour Windows Virtual Desktop](powershell-module.md) afin de télécharger et installer le module PowerShell et vous connecter à votre compte Azure.

## <a name="configure-breadth-first-load-balancing"></a>Configurer l’équilibrage de charge de largeur

L’équilibrage de charge de largeur correspond à la configuration par défaut des nouveaux pools d'hôtes non persistants. L’équilibrage de charge de largeur répartit les nouvelles sessions utilisateur entre tous les hôtes de session du pool. Lorsque vous configurez l'équilibrage de charge de largeur, vous pouvez définir une limite maximale de sessions par hôte de session du pool.

Pour configurer un pool d’hôtes à des fins d'équilibrage de charge de largeur sans ajuster la limite maximale de sessions, exécutez la cmdlet PowerShell suivante :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'BreadthFirst'
```

Ensuite, pour vérifier que vous avez défini la méthode d’équilibrage de charge en largeur d’abord, exécutez l’applet de commande suivante :

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, LoadBalancerType

Name             : hostpoolname
LoadBalancerType : BreadthFirst
```

Pour configurer un pool d’hôtes à des fins d'équilibrage de charge de largeur et utiliser une nouvelle limite maximale de sessions, exécutez la cmdlet PowerShell suivante :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'BreadthFirst' -MaxSessionLimit ###
```

## <a name="configure-depth-first-load-balancing"></a>Configurer l’équilibrage de charge de profondeur

L’équilibrage de charge de profondeur répartit les nouvelles sessions utilisateur vers l'hôte de session disponible doté du plus grand nombre de connexions, sans avoir atteint sa limite maximale de sessions.

>[!IMPORTANT]
>Quand vous configurez l’équilibrage de charge en profondeur d’abord, vous devez définir une limite maximale de sessions par hôte de session du pool.

Pour configurer un pool d’hôtes à des fins d'équilibrage de charge de profondeur, exécutez la cmdlet PowerShell suivante :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -LoadBalancerType 'DepthFirst' -MaxSessionLimit ###
```

>[!NOTE]
> L’algorithme d’équilibrage de charge en profondeur d’abord distribue les sessions aux hôtes de session en fonction de la limite maximale de ces derniers (`-MaxSessionLimit`). La valeur par défaut de ce paramètre est `999999`, ce qui correspond également au plus grand nombre qu’il soit possible d’affecter à cette variable. Ce paramètre est obligatoire avec l’algorithme d’équilibrage de charge en profondeur d’abord. Pour une expérience utilisateur optimale, veillez à définir le paramètre de limite maximale des hôtes de session sur le nombre le plus adapté à votre environnement.

Pour vérifier que le paramètre a été mis à jour, exécutez cette applet de commande :

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, LoadBalancerType, MaxSessionLimit

Name             : hostpoolname
LoadBalancerType : DepthFirst
MaxSessionLimit  : 6
```

## <a name="configure-load-balancing-with-the-azure-portal"></a>Configurer l’équilibrage de charge avec le portail Azure

Vous pouvez également configurer l’équilibrage de charge avec le portail Azure.

Pour configurer l’équilibrage de charge :

1. Connectez-vous au portail Azure à l’adresse https://portal.azure.com.
2. Recherchez et sélectionnez **Windows Virtual Desktop** sous Services.
3. Dans la page Windows Virtual Desktop, sélectionnez **Pools d’hôtes**.
4. Sélectionnez le nom du pool d’hôtes que vous souhaitez modifier.
5. Sélectionner **Propriétés**.
6. Entrez la **Limite de session maximale** dans le champ et sélectionnez l’**algorithme d’équilibrage de charge** de votre choix pour ce pool d’hôtes dans le menu déroulant.
7. Sélectionnez **Enregistrer**. Cette action applique les nouveaux paramètres d’équilibrage de charge.