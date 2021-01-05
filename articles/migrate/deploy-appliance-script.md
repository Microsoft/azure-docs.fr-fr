---
title: Configurer une appliance Azure Migrate avec un script
description: Découvrez comment configurer une appliance Azure Migrate avec un script
ms.topic: how-to
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.date: 04/16/2020
ms.openlocfilehash: 38db1e68adde21d0e566256e9393f6fa4fc58824
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753788"
---
# <a name="set-up-an-appliance-with-a-script"></a>Configurer une appliance avec un script

Suivez cet article pour créer une [appliance Azure Migrate](./migrate-appliance-architecture.md) à des fins d'évaluation/de migration des machines virtuelles VMware et Hyper-V. Vous devez exécuter un script pour créer une appliance et vérifier qu'elle peut se connecter à Azure. 

Vous pouvez déployer l’appliance pour des machines virtuelles VMware et Hyper-V à l’aide d’un script ou d'un modèle que vous téléchargez à partir du portail Azure. Un script s'avère particulièrement utile si vous ne parvenez pas à créer de machine virtuelle à l’aide du modèle téléchargé.

- Pour utiliser un modèle, suivez les tutoriels pour [VMware](./tutorial-discover-vmware.md) ou [Hyper-V](./tutorial-discover-hyper-v.md).
- Pour configurer une appliance pour les serveurs physiques, seul un script peut être utilisé. Suivez [cet article](how-to-set-up-appliance-physical.md).
- Pour configurer une appliance dans le cloud Azure Government, consultez [cet article](deploy-appliance-script-government.md).

## <a name="prerequisites"></a>Prérequis

Le script configure l’appliance Azure Migrate sur une machine physique ou virtuelle existante.

- L’ordinateur qui jouera le rôle d’appliance doit remplir les conditions suivantes en matière de matériel et de système d’exploitation :

Scénario | Spécifications
--- | ---
VMware | Windows Server 2016 avec 32 Go de mémoire, huit processeurs virtuels, environ 80 Go de stockage sur disque
Hyper-V | Windows Server 2016 avec 16 Go de mémoire, huit processeurs virtuels, environ 80 Go de stockage sur disque
- L’ordinateur a également besoin d’un commutateur virtuel externe. Elle a besoin d’une adresse IP statique ou dynamique et d’un accès à Internet.
- Avant de déployer l’appliance, examinez en détail la configuration requise pour les [machines virtuelles VMware](migrate-appliance.md#appliance---vmware) et les [machines virtuelles Hyper-V](migrate-appliance.md#appliance---hyper-v).
- N’exécutez pas le script sur une appliance de Azure Migrate existante.

## <a name="set-up-the-appliance-for-vmware"></a>Configurer l'appliance pour VMware

Pour configurer l'appliance pour VMware, téléchargez le fichier compressé AzureMigrateInstaller-Server-Public.zip à partir du portail ou de [cette page](https://go.microsoft.com/fwlink/?linkid=2140334), puis extrayez son contenu. Exécutez le script PowerShell pour lancer l'application web de l'appliance. Installez l'appliance et configurez-la pour la première fois. Inscrivez ensuite l'appliance auprès du projet Azure Migrate.


### <a name="verify-file-security"></a>Vérifier la sécurité du fichier

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-VMware-Public.zip SHA256```
3. Vérifiez la dernière version de l’appliance et le script du cloud public Azure :

    **Algorithme** | **Télécharger** | **SHA256**
    --- | --- | ---
    VMware (85,8 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2116601) | 85b74d93dfcee43412386141808d82147916330e6669df94c7969fe1b3d0fe72



### <a name="run-the-script"></a>Exécuter le script

Le script effectue les opérations suivantes :

- Installe les agents et une application web.
- Installe des rôles Windows, à savoir le service d’activation Windows (WAS), IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec des paramètres persistants pour Azure Migrate.
- Crée les fichiers journaux et de configuration comme suit :
    - **Fichiers config** : %ProgramData%\Microsoft Azure\Config
    - **Fichiers journaux** : %ProgramData%\Microsoft Azure\Logs

Pour exécuter le script :

1. Extrayez le fichier zip dans un dossier sur la machine qui doit héberger l’appliance. Veillez à ne pas exécuter le script sur une machine d'une appliance Azure Migrate existante.
2. Lancez PowerShell sur la machine avec un privilège administrateur (élevé).
3. Modifiez le répertoire PowerShell en sélectionnant le dossier contenant l’extraction du fichier zip téléchargé.
4. Exécutez le script **AzureMigrateInstaller.ps1** comme suit :

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario VMware ```
   
5. Une fois correctement exécuté, le script lance l’application web de l’appliance afin que vous puissiez configurer celle-ci. En cas de problèmes, consultez les journaux du script sous C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Vérifier l’accès

Vérifiez que l’appliance peut se connecter aux URL Azure pour le cloud [public](migrate-appliance.md#public-cloud-urls).

## <a name="set-up-the-appliance-for-hyper-v"></a>Configurer l'appliance pour Hyper-V

Pour configurer l'appliance pour Hyper-V, téléchargez le fichier compressé AzureMigrateInstaller-Server-Public.zip à partir du portail ou de [cette page](https://go.microsoft.com/fwlink/?linkid=2105112), puis extrayez son contenu. Exécutez le script PowerShell pour lancer l'application web de l'appliance. Installez l'appliance et configurez-la pour la première fois. Inscrivez ensuite l'appliance auprès du projet Azure Migrate.


### <a name="verify-file-security"></a>Vérifier la sécurité du fichier

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-HyperV.zip SHA256```

3. Vérifiez la dernière version de l’appliance et le script du cloud public Azure :

    **Scénario** | **Télécharger** | **SHA256**
    --- | --- | ---
    Hyper-V (85,8 Mo) | [Version la plus récente](https://go.microsoft.com/fwlink/?linkid=2116657) |  9bbef62e2e22481eda4b77c7fdf05db98c3767c20f0a873114fb0dcfa6ed682a

### <a name="run-the-script"></a>Exécuter le script

Le script effectue les opérations suivantes :

- Installe les agents et une application web.
- Installe des rôles Windows, à savoir le service d’activation Windows (WAS), IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec des paramètres persistants pour Azure Migrate.
- Crée les fichiers journaux et de configuration comme suit :
    - **Fichiers config** : %ProgramData%\Microsoft Azure\Config
    - **Fichiers journaux** : %ProgramData%\Microsoft Azure\Logs

Pour exécuter le script :

1. Extrayez le fichier zip dans un dossier sur la machine qui doit héberger l’appliance. Veillez à ne pas exécuter le script sur une machine d'une appliance Azure Migrate existante.
2. Lancez PowerShell sur la machine avec un privilège administrateur (élevé).
3. Modifiez le répertoire PowerShell en sélectionnant le dossier contenant l’extraction du fichier zip téléchargé.
4. Exécutez le script **AzureMigrateInstaller.ps1** comme suit : 

    ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 -scenario Hyperv ```
   
5. Une fois correctement exécuté, le script lance l’application web de l’appliance afin que vous puissiez configurer celle-ci. En cas de problèmes, consultez les journaux du script sous C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>Timestamp</em>.log.

### <a name="verify-access"></a>Vérifier l’accès

Vérifiez que l’appliance peut se connecter aux URL Azure pour le cloud [public](migrate-appliance.md#public-cloud-urls).

## <a name="next-steps"></a>Étapes suivantes

Après avoir déployé l'appliance, vous devez la configurer pour la première fois et l'inscrire auprès du projet Azure Migrate.

- Configurez l’appliance pour [VMware](how-to-set-up-appliance-vmware.md#configure-the-appliance).
- Configurez l’appliance pour [Hyper-V](how-to-set-up-appliance-hyper-v.md#configure-the-appliance).