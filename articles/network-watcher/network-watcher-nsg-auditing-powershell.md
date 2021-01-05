---
title: Automatiser l’audit NSG - affichage du groupe de sécurité
titleSuffix: Azure Network Watcher
description: Cette page fournit des instructions sur configuration de l’audit d’un groupe de sécurité réseau
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
ms.openlocfilehash: 177215775c9e83286aa98872eed0ab211a8f36ff
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94948746"
---
# <a name="automate-nsg-auditing-with-azure-network-watcher-security-group-view"></a>Automatiser l’audit NSG avec la vue de groupe de sécurité réseau Network Watcher

Les clients sont souvent confrontés au défi de la vérification des mesures de sécurité de leur infrastructure. Ce défi n’est pas différent pour leurs machines virtuelles dans Azure. Il est important de disposer d’un profil de sécurité similaire en fonction des règles de groupe de sécurité réseau (NSG) appliquées. Grâce à la vue Groupe de sécurité, vous pouvez désormais obtenir la liste des règles appliquées à une machine virtuelle au sein d’un NSG. Vous pouvez définir un profil de sécurité NSG final, ouvrir la vue Groupe de sécurité selon une fréquence hebdomadaire, puis comparer la sortie au profil final et créer un rapport. Vous pouvez ainsi identifier facilement toutes les machines virtuelles qui ne sont pas conformes au profil de sécurité établi.

Si vous n’êtes pas familiarisé avec les groupes de sécurité réseau, consultez la [vue d’ensemble de la sécurité réseau](../virtual-network/network-security-groups-overview.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Avant de commencer

Dans ce scénario, vous comparez une base correcte connue avec les résultats de la vue de groupe de sécurité renvoyés pour une machine virtuelle.

Ce scénario suppose que vous ayez déjà suivi la procédure décrite dans [Create a Network Watcher (Créer une instance Network Watcher)](network-watcher-create.md) pour créer une instance Network Watcher. Ce scénario suppose également qu’un groupe de ressources avec une machine virtuelle valide existe et peut être utilisé.

## <a name="scenario"></a>Scénario

Le scénario décrit dans cet article obtient la vue de groupe de sécurité pour une machine virtuelle.

Dans ce scénario, vous allez :

- Récupérer un ensemble de règles correct connu
- Récupérer une machine virtuelle avec l’API REST
- Obtenir la vue de groupe de sécurité pour la machine virtuelle
- Évaluer la réponse

## <a name="retrieve-rule-set"></a>Récupérer l’ensemble de règles

La première étape dans cet exemple consiste à utiliser une base existante. L’exemple suivant est un json extrait d’un groupe de sécurité réseau existant à l’aide de l’applet de commande `Get-AzNetworkSecurityGroup` qui est utilisée comme base pour cet exemple.

```json
[
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "3389",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1000,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "default-allow-rdp",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/default-allow-rdp"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "111",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1010,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "MyRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/MyRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "*",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "112",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Allow",
        "Priority":  1020,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "My2ndRuleDoNotDelete",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/My2ndRuleDoNotDelete"
    },
    {
        "Description":  null,
        "Protocol":  "TCP",
        "SourcePortRange":  "*",
        "DestinationPortRange":  "5672",
        "SourceAddressPrefix":  "*",
        "DestinationAddressPrefix":  "*",
        "Access":  "Deny",
        "Priority":  1030,
        "Direction":  "Inbound",
        "ProvisioningState":  "Succeeded",
        "Name":  "ThisRuleNeedsToStay",
        "Etag":  "W/\"d8859256-1c4c-4b93-ba7d-73d9bf67c4f1\"",
        "Id":  "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/Microsoft.Network/networkSecurityGroups/testvm1-nsg/securityRules/ThisRuleNeedsToStay"
    }
]
```

## <a name="convert-rule-set-to-powershell-objects"></a>Convertir l’ensemble de règles en objets PowerShell

Dans cette étape, nous lisons un fichier json qui a été créé précédemment avec les règles qui sont censées se trouver sur le groupe de sécurité réseau pour cet exemple.

```powershell
$nsgbaserules = Get-Content -Path C:\temp\testvm1-nsg.json | ConvertFrom-Json
```

## <a name="retrieve-network-watcher"></a>Récupérer Network Watcher

L’étape suivante consiste à récupérer l’instance Network Watcher. La variable `$networkWatcher` est transmise à l’applet de commande `AzNetworkWatcherSecurityGroupView`.

```powershell
$networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq "WestCentralUS" } 
```

## <a name="get-a-vm"></a>Obtenir une machine virtuelle

Une machine virtuelle est requise sur laquelle exécuter l’applet de commande `Get-AzNetworkWatcherSecurityGroupView`. L’exemple suivant obtient un objet machine virtuelle.

```powershell
$VM = Get-AzVM -ResourceGroupName "testrg" -Name "testvm1"
```

## <a name="retrieve-security-group-view"></a>Récupérer la vue Groupe de sécurité

L’étape suivante consiste à récupérer le résultat de la vue Groupe de sécurité. Ce résultat est comparé au json « de base » qui a été présenté précédemment.

```powershell
$secgroup = Get-AzNetworkWatcherSecurityGroupView -NetworkWatcher $networkWatcher -TargetVirtualMachineId $VM.Id
```

## <a name="analyzing-the-results"></a>Analyser les résultats

La réponse est regroupée par interfaces réseau. Les différents types de règles renvoyées sont efficaces et constituent les règles de sécurité par défaut. Le résultat est subdivisé en fonction de l’application, sur un sous-réseau ou une carte réseau virtuelle.

Le script PowerShell suivant compare les résultats de la vue Groupe de sécurité avec une sortie existante d’un NSG. L’exemple suivant est un exemple simple de la façon dont les résultats peuvent être comparés avec l’applet de commande `Compare-Object`.

```powershell
Compare-Object -ReferenceObject $nsgbaserules `
-DifferenceObject $secgroup.NetworkInterfaces[0].NetworkInterfaceSecurityRules `
-Property Name,Description,Protocol,SourcePortRange,DestinationPortRange,SourceAddressPrefix,DestinationAddressPrefix,Access,Priority,Direction
```

L’exemple suivant est le résultat. Vous pouvez constater que deux règles qui se trouvaient dans le premier ensemble de règles n’étaient pas présentes dans la comparaison.

```
Name                     : My2ndRuleDoNotDelete
Description              : 
Protocol                 : *
SourcePortRange          : *
DestinationPortRange     : 112
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Allow
Priority                 : 1020
Direction                : Inbound
SideIndicator            : <=

Name                     : ThisRuleNeedsToStay
Description              : 
Protocol                 : TCP
SourcePortRange          : *
DestinationPortRange     : 5672
SourceAddressPrefix      : *
DestinationAddressPrefix : *
Access                   : Deny
Priority                 : 1030
Direction                : Inbound
SideIndicator            : <=
```

## <a name="next-steps"></a>Étapes suivantes

Si les paramètres ont été modifiés, consultez la page [Gérer les groupes de sécurité réseau](../virtual-network/manage-network-security-group.md) afin d’effectuer le suivi du groupe de sécurité réseau et des règles de sécurité concernés.