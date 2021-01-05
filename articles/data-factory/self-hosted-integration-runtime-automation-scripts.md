---
title: Automatisation de l’installation du runtime d’intégration auto-hébergé à l’aide de scripts PowerShell locaux
description: Pour automatiser l’installation du runtime d’intégration auto-hébergé sur des ordinateurs locaux.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.custom: seo-lt-2019
ms.date: 05/09/2020
ms.openlocfilehash: 36414c975e97dbaa7d8747da98c31eeb12fbc206
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636967"
---
# <a name="automating-self-hosted-integration-runtime-installation-using-local-powershell-scripts"></a>Automatisation de l’installation du runtime d’intégration auto-hébergé à l’aide de scripts PowerShell locaux
Pour automatiser l’installation du runtime d’intégration auto-hébergé sur des ordinateurs locaux (autres que des machines virtuelles Azure pour lesquelles nous pouvons tirer parti du modèle Resource Manager), vous pouvez utiliser des scripts PowerShell locaux. Cet article présente deux scripts que vous pouvez utiliser.

## <a name="prerequisites"></a>Prérequis

* Lancez PowerShell sur votre ordinateur local. Pour exécuter les scripts, vous devez choisir **Exécuter en tant qu’administrateur** .
* [Téléchargez](https://www.microsoft.com/download/details.aspx?id=39717) le logiciel du runtime d'intégration auto-hébergé. Copiez le chemin d’accès où se trouve le fichier téléchargé. 
* Vous avez également besoin d’une **clé d’authentification** pour inscrire le runtime d’intégration auto-hébergé.
* Pour automatiser les mises à jour manuelles, vous devez disposer d’un runtime d’intégration auto-hébergé préconfiguré.

## <a name="scripts-introduction"></a>Introduction aux scripts 

> [!NOTE]
> Ces scripts sont créés à l’aide de [l’utilitaire de ligne de commande documenté](./create-self-hosted-integration-runtime.md#set-up-an-existing-self-hosted-ir-via-local-powershell) dans le runtime d’intégration auto-hébergé. Si nécessaire, vous pouvez personnaliser ces scripts en conséquence pour répondre à leurs besoins d’automatisation.
> Les scripts doivent être appliqués par nœud. Veillez donc à les exécuter sur tous les nœuds en cas de configuration à haute disponibilité (deux nœuds ou plus).

* Pour l’automatisation de l’installation : Installez et inscrivez un nouveau nœud de runtime d’intégration auto-hébergé à l’aide de **[InstallGatewayOnLocalMachine.ps1](https://github.com/nabhishek/SelfHosted-IntegrationRuntime_AutomationScripts/blob/master/InstallGatewayOnLocalMachine.ps1)** - Le script peut être utilisé pour installer le nœud du runtime d’intégration auto-hébergé et l’inscrire avec une clé d’authentification. Le script accepte deux arguments, **le premier** spécifiant l’emplacement du [runtime d’intégration auto-hébergé](https://www.microsoft.com/download/details.aspx?id=39717) sur un disque local, **le deuxième** spécifiant la **clé d’authentification** (pour l’inscription du nœud du runtime d’intégration auto-hébergé).

* Pour automatiser les mises à jour manuelles : Mettez à jour le nœud du runtime d’intégration auto-hébergé avec une version spécifique ou la version la plus récente **[script-update-gateway.ps1](https://github.com/nabhishek/SelfHosted-IntegrationRuntime_AutomationScripts/blob/master/script-update-gateway.ps1)** - Cette option est également prise en charge si vous avez désactivé la mise à jour automatique ou si vous souhaitez avoir davantage de contrôle sur les mises à jour. Le script peut être utilisé pour mettre à jour le nœud du runtime d’intégration auto-hébergé vers la version la plus récente ou vers une version supérieure spécifiée (il n’est pas possible de passer à une version antérieure). Il accepte un argument pour spécifier le numéro de version (exemple : version 3.13.6942.1). Si aucune version n’est spécifiée, il met toujours à jour le runtime d’intégration auto-hébergé avec la dernière version trouvée dans les [téléchargements](https://www.microsoft.com/download/details.aspx?id=39717).
    > [!NOTE]
    > Seules les 3 dernières versions peuvent être spécifiées. Dans l’idéal, il est utilisé pour mettre à jour un nœud existant vers la dernière version. **IL PART DU PRINCIPE QUE VOUS DISPOSEZ D’UN RUNTIME D'INTÉGRATION AUTO-HÉBERGÉ ENREGISTRÉ** . 

## <a name="usage-examples"></a>Exemples d'utilisation

### <a name="for-automating-setup"></a>Pour l’automatisation de l’installation
1. Téléchargez le runtime d’intégration auto-hébergé à partir d’[ici](https://www.microsoft.com/download/details.aspx?id=39717). 
1. Spécifiez le chemin d’accès vers le fichier SHIR MSI (fichier d’installation) téléchargé précédemment. Par exemple, si le chemin d’accès est *C:\Users\username\Downloads\IntegrationRuntime_4.7.7368.1.msi* , vous pouvez utiliser l’exemple de ligne de commande PowerShell ci-dessous pour cette tâche :

   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\InstallGatewayOnLocalMachine.ps1 -path "C:\Users\username\Downloads\IntegrationRuntime_4.7.7368.1.msi" -authKey "[key]"
   ```

    > [!NOTE]
    > Remplacez [key] par la clé d’authentification pour inscrire votre runtime d'intégration.
    > Remplacez « username » par votre nom d’utilisateur.
    > Spécifiez l’emplacement du fichier « InstallGatewayOnLocalMachine.ps1 » lors de l’exécution du script. Dans cet exemple, nous l’avons stocké sur le bureau.

1. Si un runtime d’intégration auto-hébergé est préinstallé sur votre ordinateur, le script le désinstalle automatiquement, puis en configure un nouveau. Vous verrez la fenêtre suivante apparaître : ![configurer le runtime d’intégration](media/self-hosted-integration-runtime-automation-scripts/integration-runtime-configure.png)

1. Une fois l’installation et l’inscription de la clé terminées, vous verrez les résultats *Réussite de l’installation de la passerelle* et *Réussite de l’inscription la passerelle* dans votre PowerShell local.
        [![résultat de l’exécution du script 1](media/self-hosted-integration-runtime-automation-scripts/script-1-run-result.png)](media/self-hosted-integration-runtime-automation-scripts/script-1-run-result.png#lightbox)

### <a name="for-automating-manual-updates"></a>Pour automatiser les mises à jour manuelles
Ce script est utilisé pour mettre à jour/installer + inscrire le dernier runtime d’intégration auto-hébergé. L’exécution du script effectue les étapes suivantes :
1. Vérifier la version actuelle du runtime d’intégration auto-hébergé
2. Obtenir la version la plus récente ou la version spécifiée à partir de l’argument
3. S’il existe une version plus récente que la version actuelle :
    * télécharger le fichier msi du runtime d'intégration auto-hébergé
    * le mettre à niveau

Vous pouvez suivre l’exemple de ligne de commande ci-dessous pour utiliser ce script :
* Télécharger et installer la passerelle la plus récente :

   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\script-update-gateway.ps1
   ```    
* Télécharger et installer la passerelle de la version spécifiée :
   ```powershell
   PS C:\windows\system32> C:\Users\username\Desktop\script-update-gateway.ps1 -version 3.13.6942.1
   ``` 
   Si votre version actuelle est déjà la plus récente, vous verrez le résultat suivant, car aucune mise à jour n’est requise.   
    [![résultat de l’exécution du script 2](media/self-hosted-integration-runtime-automation-scripts/script-2-run-result.png)](media/self-hosted-integration-runtime-automation-scripts/script-2-run-result.png#lightbox)