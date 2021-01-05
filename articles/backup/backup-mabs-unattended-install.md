---
title: Installation sans assistance du serveur de sauvegarde Azure V2
description: Utilisez un script PowerShell pour installer sans assistance le Serveur de sauvegarde Azure V2. Ce type d’installation est également appelé « installation silencieuse ».
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 1539089e713bcf8e959707c6ff4a608f062a7c00
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74172243"
---
# <a name="run-an-unattended-installation-of-azure-backup-server"></a>Effectuer une installation sans assistance du Serveur de sauvegarde Azure

Découvrez comment effectuer une installation sans assistance du Serveur de sauvegarde Azure.

Ces étapes ne s’appliquent pas si vous installez le serveur de sauvegarde Azure v1.

## <a name="install-backup-server"></a>Installer le Serveur de sauvegarde

1. Sur le serveur hébergeant le Serveur de sauvegarde Azure V2 ou versions ultérieures, créez un fichier texte. (vous pouvez créer le fichier dans Bloc-notes ou dans un autre éditeur de texte). Enregistrez le fichier sous MABSSetup.ini.

2. Collez le code suivant dans le fichier MABSSetup.ini. Remplacez le texte entre crochets (\< \>) par des valeurs de votre environnement. Voici un exemple de texte :

   ```text
   [OPTIONS]
   UserName=administrator
   CompanyName=<Microsoft Corporation>
   SQLMachineName=localhost
   SQLInstanceName=<SQL instance name>
   SQLMachineUserName=administrator
   SQLMachinePassword=<admin password>
   SQLMachineDomainName=<machine domain>
   ReportingMachineName=localhost
   ReportingInstanceName=<reporting instance name>
   SqlAccountPassword=<admin password>
   ReportingMachineUserName=<username>
   ReportingMachinePassword=<reporting admin password>
   ReportingMachineDomainName=<domain>
   VaultCredentialFilePath=<vault credential full path and complete name>
   SecurityPassphrase=<passphrase>
   PassphraseSaveLocation=<passphrase save location>
   UseExistingSQL=<1/0 use or do not use existing SQL>
   ```

3. Enregistrez le fichier . Ensuite, à une invite de commandes avec élévation de privilèges, entrez la commande suivante :

   ```cmd
   start /wait <cdlayout path>/Setup.exe /i  /f <.ini file path>/setup.ini /L <log path>/setup.log
   ```

Vous pouvez utiliser les indicateurs suivants pour l’installation :</br>
**/f** : chemin du fichier .ini</br>
**/l** : chemin du journal</br>
**/i** : chemin d’installation</br>
**/x** : chemin de désinstallation</br>

## <a name="next-steps"></a>Étapes suivantes

Après avoir installé le Serveur de sauvegarde, apprenez à préparer votre serveur ou commencez à protéger une charge de travail.

- [Préparer les charges de travail du serveur de sauvegarde](backup-azure-microsoft-azure-backup.md)
- [Utiliser le Serveur de sauvegarde pour sauvegarder un serveur VMware](backup-azure-backup-server-vmware.md)
- [Utiliser le serveur de sauvegarde pour sauvegarder SQL Server](backup-azure-sql-mabs.md)
- [Ajouter un stockage de sauvegarde moderne au Serveur de sauvegarde](backup-mabs-add-storage.md)
