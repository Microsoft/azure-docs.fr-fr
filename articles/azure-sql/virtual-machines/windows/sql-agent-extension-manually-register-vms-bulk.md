---
title: Inscrire plusieurs machines virtuelles SQL dans Azure avec l’extension SQL IaaS Agent
description: Inscrivez en bloc des machines virtuelles SQL Server avec l’extension SQL IaaS Agent pour améliorer la facilité de gestion.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/07/2020
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 558daede55f6563155d3f54e97d77c0a3ca4de59
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357220"
---
# <a name="register-multiple-sql-vms-in-azure-with-the-sql-iaas-agent-extension"></a>Inscrire plusieurs machines virtuelles SQL dans Azure avec l’extension SQL IaaS Agent
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment inscrire en bloc vos machines virtuelles SQL Server dans Azure auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md) à l’aide de la cmdlet Azure PowerShell `Register-SqlVMs`. 


Cet article vous apprend à inscrire des machines virtuelles SQL Server manuellement en bloc. Vous pouvez également inscrire [toutes les machines virtuelles SQL Server automatiquement](sql-agent-extension-automatic-registration-all-vms.md) ou [des machines virtuelles SQL Server individuelles manuellement](sql-agent-extension-manually-register-single-vm.md). 

## <a name="overview"></a>Vue d’ensemble

L’applet de commande `Register-SqlVMs` peut être utilisée pour inscrire toutes les machines virtuelles d’une liste donnée d’abonnements, de groupes de ressources ou d’une liste de machines virtuelles spécifiques. La cmdlet inscrira les machines virtuelles en [mode d’administration léger](sql-server-iaas-agent-extension-automate-management.md#management-modes), puis générera [un rapport et un fichier journal](#output-description). 

Le processus d’inscription ne présente aucun risque, n’a pas de temps d’arrêt et ne redémarrera pas le service SQL Server ou la machine virtuelle. 

## <a name="prerequisites"></a>Prérequis

Pour inscrire votre machine virtuelle SQL Server auprès de l’extension, vous aurez besoin des éléments suivants : 

- Un [abonnement Azure](https://azure.microsoft.com/free/) qui a été [inscrit auprès du fournisseur **Microsoft.SqlVirtualMachine**](sql-agent-extension-manually-register-single-vm.md#register-subscription-with-rp) et qui contient des machines virtuelles SQL Server non inscrites. 
- Les informations d’identification du client utilisées pour inscrire les machines virtuelles existent dans chacun des rôles Azure suivants : **Contributeur de machine virtuelle**, **Contributeur** ou **Propriétaire**. 
- La version la plus récente d’[Az PowerShell (5.0 au minimum)](/powershell/azure/new-azureps-module-az). 


## <a name="get-started"></a>Bien démarrer

Avant de continuer, vous devez d’abord créer une copie locale du script, l’importer en tant que module PowerShell et vous connecter à Azure. 

### <a name="create-the-script"></a>Créez le script.

Pour créer le script, copiez le [script complet](#full-script) à la fin de cet article et enregistrez-le localement en tant que `RegisterSqlVMs.psm1`. 

### <a name="import-the-script"></a>Importer le script

Une fois le script créé, vous pouvez l’importer en tant que module dans le terminal PowerShell. 

Ouvrez un terminal PowerShell d’administration et accédez à l’emplacement où vous avez enregistré le fichier `RegisterSqlVMs.psm1`. Ensuite, exécutez l’applet de commande PowerShell suivante pour importer le script sous la forme d’un module : 

```powershell-interactive
Import-Module .\RegisterSqlVMs.psm1
```

### <a name="connect-to-azure"></a>Connexion à Azure

Utilisez l’applet de commande PowerShell suivante pour vous connecter à Azure :

```powershell-interactive
Connect-AzAccount
```


## <a name="all-vms-in-a-list-of-subscriptions"></a>Toutes les machines virtuelles dans une liste d’abonnements 

Utilisez l’applet de commande suivante pour inscrire toutes les machines virtuelles SQL Server dans une liste d’abonnements :

```powershell-interactive
Register-SqlVMs -SubscriptionList SubscriptionId1,SubscriptionId2
```

Exemple de sortie : 

```
Number of subscriptions registration failed for 
because you do not have access or credentials are wrong: 1
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 4
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 3
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-a-single-subscription"></a>Toutes les machines virtuelles dans un seul abonnement

Utilisez l’applet de commande suivante pour inscrire toutes les machines virtuelles SQL Server dans un seul abonnement : 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1
```

Exemple de sortie :

```
Total VMs Found: 10
VMs Already registered: 1
Number of VMs registered successfully: 5
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the  guest agent on VM is not running: 2
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-multiple-resource-groups"></a>Toutes les machines virtuelles dans plusieurs groupes de ressources

Utilisez l’applet de commande suivante pour inscrire toutes les machines virtuelles SQL Server dans plusieurs groupes de ressources au sein d’un seul abonnement :

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupList ResourceGroup1,ResourceGroup2
```

Exemple de sortie :

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="all-vms-in-a-resource-group"></a>Toutes les machines virtuelles au sein d’un groupe de ressources

Utilisez l’applet de commande suivante pour inscrire toutes les machines virtuelles SQL Server dans un seul groupe de ressources : 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1
```

Exemple de sortie :

```
Total VMs Found: 4
VMs Already registered: 1
Number of VMs registered successfully: 1
Number of VMs failed to register due to error: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="specific-vms-in-a-single-resource-group"></a>Machines virtuelles spécifiques dans un seul groupe de ressources

Utilisez l’applet de commande suivante pour inscrire des machines virtuelles SQL Server spécifiques dans un seul groupe de ressources :

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -VmList VM1,VM2,VM3
```

Exemple de sortie :

```
Total VMs Found: 3
VMs Already registered: 0
Number of VMs registered successfully: 1
Number of VMs skipped as VM or the guest agent on VM is not running: 1
Number of VMs skipped as they are not running SQL Server On Windows: 1

Please find the detailed report in file RegisterSqlVMScriptReport1571314821.txt
Please find the error details in file VMsNotRegisteredDueToError1571314821.log
```

## <a name="a-specific-vm"></a>Machine virtuelle spécifique

Utilisez l’applet de commande suivante pour inscrire une machine virtuelle SQL Server spécifique : 

```powershell-interactive
Register-SqlVMs -Subscription SubscriptionId1 -ResourceGroupName ResourceGroup1 -Name VM1
```

Exemple de sortie :

```
Total VMs Found: 1
VMs Already registered: 0
Number of VMs registered successfully: 1

Please find the detailed report in  file RegisterSqlVMScriptReport1571314821.txt
```


## <a name="output-description"></a>Description de la sortie

Un rapport et un fichier journal sont générés chaque fois que l’applet de commande `Register-SqlVMs` est utilisée. 

### <a name="report"></a>Rapport

Le rapport est généré sous la forme d’un fichier `.txt` nommé `RegisterSqlVMScriptReport<Timestamp>.txt` où le timestamp est l’heure à laquelle l’applet de commande a été démarrée. Le rapport indique les informations suivantes :

| **Valeur de sortie** | **Description** |
| :--------------  | :-------------- | 
| Nombre d’abonnements dont l’inscription a échoué, car vous n’avez pas d’accès ou les informations d’identification sont incorrectes | Cela fournit le nombre et la liste des abonnements qui ont rencontré des problèmes avec l’authentification fournie. L’erreur détaillée est disponible dans le journal en recherchant l’ID d’abonnement. | 
| Nombre d’abonnements qui n’ont pas pu être essayés, car ils ne sont pas inscrits auprès du fournisseur de ressources | Cette section contient le nombre et la liste des abonnements qui n’ont pas été inscrits auprès de l’extension SQL IaaS Agent. |
| Nombre total de machines virtuelles trouvées | Le nombre de machines virtuelles qui ont été trouvées dans l’étendue des paramètres communiqués à l’applet de commande. | 
| Machines virtuelles déjà inscrites | Le nombre de machines virtuelles qui ont été ignorées, car elles ont déjà été inscrites auprès de l’extension. |
| Nombre de machines virtuelles inscrites avec succès | Le nombre de machines virtuelles qui ont été correctement inscrites après l’exécution de l’applet de commande. Répertorie les machines virtuelles inscrites au format `SubscriptionID, Resource Group, Virtual Machine`. | 
| Nombre d’échecs d’inscription de machines virtuelles en raison d’une erreur | Le nombre de machines virtuelles dont l’inscription a échoué en raison d’une erreur. Les détails de l’erreur se trouvent dans le fichier journal. | 
| Nombre de machines virtuelles ignorées car la machine virtuelle ou l’agent invité sur la machine virtuelle n’est pas en cours d’exécution | Le nombre et la liste des machines virtuelles qui n’ont pas pu être inscrites car la machine virtuelle ou l’agent invité sur la machine virtuelle n’étaient pas en cours d’exécution. Vous pouvez procéder à une nouvelle tentative une fois que la machine virtuelle ou l’agent invité a été démarré. De plus amples informations sont disponibles dans le fichier journal. |
| Nombre de machines virtuelles ignorées car elles n’exécutent pas SQL Server sur Windows | Nombre de machines virtuelles qui ont été ignorées, car elles n’exécutent pas SQL Server ou ne sont pas des machines virtuelles Windows. Les machines virtuelles sont répertoriées au format `SubscriptionID, Resource Group, Virtual Machine`. | 
| &nbsp; | &nbsp; |

### <a name="log"></a>Journal 

Les erreurs sont consignées dans le fichier journal nommé `VMsNotRegisteredDueToError<Timestamp>.log`, où timestamp représente l’heure à laquelle le script a démarré. Si l’erreur se trouve au niveau de l’abonnement, le journal contient l’ID d’abonnement séparé par des virgules et le message d’erreur. Si l’erreur est liée à l’inscription de la machine virtuelle, le journal contient l’ID d’abonnement, le nom du groupe de ressources, le nom de la machine virtuelle, le code d’erreur et le message, séparés par des virgules. 

## <a name="remarks"></a>Notes

Lorsque vous inscrivez des machines virtuelles SQL Server auprès de l’extension à l’aide du script fourni, prenez en compte les éléments suivants :

- L’inscription auprès de l’extension requiert un agent invité en cours d’exécution sur la machine virtuelle SQL Server. Les images de Windows Server 2008 ne disposent pas d’un agent invité. Par conséquent, ces machines virtuelles échouent et doivent être inscrites manuellement à l’aide du [mode d’administration NoAgent](sql-server-iaas-agent-extension-automate-management.md#management-modes).
- Une logique de nouvelle tentative est intégrée pour surmonter les erreurs transparentes. Si la machine virtuelle est correctement inscrite, il s’agit d’une opération rapide. Toutefois, si l’inscription échoue, chaque machine virtuelle sera retentée.  Par conséquent, vous devez prévoir un temps considérable pour terminer le processus d’inscription, bien que l’exigence de temps réel dépende du type et du nombre d’erreurs. 

## <a name="full-script"></a>Script complet

Pour obtenir le script complet sur GitHub, consultez [Bulk register SQL Server VMs with Az PowerShell (Inscrire en bloc des machines virtuelles SQL Server à l’aide d’Az PowerShell)](https://github.com/Azure/azure-docs-powershell-samples/blob/master/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1). 

Copiez le script entier et enregistrez-le sous le nom `RegisterSqLVMs.psm1`.

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-virtual-machine/register-sql-vms/RegisterSqlVMs.psm1 "Bulk register SQL Server virtual machines")]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants : 

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Questions fréquentes (FAQ) pour SQL Server sur une machine virtuelle Windows](frequently-asked-questions-faq.md)
* [Guide des tarifs pour SQL Server sur une machine virtuelle Windows](pricing-guidance.md)
* [Notes de publication pour SQL Server sur une machine virtuelle Windows](../../database/doc-changes-updates-release-notes.md)
