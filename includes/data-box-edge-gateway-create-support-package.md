---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/05/2019
ms.author: alkohli
ms.openlocfilehash: b657ee32e76dd90671f7e91337ced01b925889a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177522"
---
Si vous rencontrez des problèmes avec l’appareil, vous pouvez créer un package de support dans les journaux système. Le support Microsoft utilise ce package pour résoudre les problèmes. Pour créer un package de support, effectuez les étapes suivantes :

1. [Connectez-vous à l’interface PowerShell de votre appareil](#connect-to-the-powershell-interface).
2. Utilisez la commande `Get-HcsNodeSupportPackage` pour créer un package de support. Utilisez la cmdlet comme suit :

    ```powershell
    Get-HcsNodeSupportPackage [-Path] <string> [-Zip] [-ZipFileName <string>] [-Include {None | RegistryKeys | EtwLogs
            | PeriodicEtwLogs | LogFiles | DumpLog | Platform | FullDumps | MiniDumps | ClusterManagementLog | ClusterLog |
            UpdateLogs | CbsLogs | StorageCmdlets | ClusterCmdlets | ConfigurationCmdlets | KernelDump | RollbackLogs |
            Symbols | NetworkCmdlets | NetworkCmds | Fltmc | ClusterStorageLogs | UTElement | UTFlag | SmbWmiProvider |
            TimeCmds | LocalUILogs | ClusterHealthLogs | BcdeditCommand | BitLockerCommand | DirStats | ComputeRolesLogs |
            ComputeCmdlets | DeviceGuard | Manifests | MeasuredBootLogs | Stats | PeriodicStatLogs | MigrationLogs |
            RollbackSupportPackage | ArchivedLogs | Default}] [-MinimumTimestamp <datetime>] [-MaximumTimestamp <datetime>]
            [-IncludeArchived] [-IncludePeriodicStats] [-Credential <pscredential>]  [<CommonParameters>]
    ```

    La cmdlet collecte les journaux à partir de votre appareil et les copie vers un réseau ou un partage local spécifié.

    Les paramètres utilisés sont les suivants :

    - `-Path` : permet de spécifier le réseau ou le chemin d’accès local vers lequel y copier le package de support. (obligatoire)
    - `-Credential` : permet de spécifier les informations d’identification pour accéder au chemin protégé.
    - `-Zip` : permet de générer un fichier zip.
    - `-Include` : permet d’inclure les composants à inclure dans le package de support. S’il n’est pas renseigné, la valeur `Default` est supposée.
    - `-IncludeArchived` : permet d’inclure des journaux archivés dans le package de support.
    - `-IncludePeriodicStats` : permet d’inclure des journaux de statistiques périodiques dans le package de support.

    
