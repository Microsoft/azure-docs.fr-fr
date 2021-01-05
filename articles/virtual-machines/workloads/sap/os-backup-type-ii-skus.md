---
title: Sauvegarde et restauration du système d’exploitation pour les références (SKU) de type II SAP HANA (grandes instances) sur Azure | Microsoft Docs
description: Effectuer la sauvegarde et la restauration du système d’exploitation pour les références (SKU) de type II SAP HANA (grandes instances) sur Azure
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e690f6abc9c597cc1facb7b3e59604bb711cf274
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94955665"
---
# <a name="os-backup-and-restore-for-type-ii-skus-of-revision-3-stamps"></a>Sauvegarde et restauration du système d’exploitation pour les références (SKU) de type II avec révision 3

Ce document décrit les étapes permettant de sauvegarder et de restaurer le système de fichiers du système d’exploitation pour les **références (SKU) de type II** des grandes instances HANA de révision 3. 

>[!Important]
> **Cet article ne s’applique pas aux déploiements de références SKU de type II dans les tampons de grande instance HANA de révision 4.** Les numéros d’unité logique de démarrage des unités de grande instance HANA de type II déployées dans les tampons de grande instance HANA de révision 4 peuvent être sauvegardés avec des captures instantanées de stockage, comme c’est le cas avec les références SKU de type I déjà dans les tampons de révision 3.


>[!NOTE]
>Les scripts de sauvegarde du système d’exploitation utilisent le logiciel ReaR, qui est préinstallé sur le serveur.  

Une fois l’approvisionnement effectué par l’équipe de gestion `Service Management` Microsoft, le serveur est par défaut configuré avec une planification de deux sauvegardes pour préserver la sauvegarde du système de fichiers du système d’exploitation. Vous pouvez vérifier la planification des tâches de sauvegarde avec la commande suivante :
```
#crontab –l
```
Vous pouvez modifier la planification de la sauvegarde à tout moment avec la commande suivante :
```
#crontab -e
```
## <a name="how-to-take-a-manual-backup"></a>Comment effectuer une sauvegarde manuelle ?

La sauvegarde du système d’exploitation est d’ores et déjà planifiée avec une **tâche Cron**. Toutefois, vous pouvez aussi effectuer la sauvegarde du système de fichiers du système d’exploitation manuellement. Pour effectuer une sauvegarde manuelle, exécutez la commande suivante :

```
#rear -v mkbackup
```
La capture d’écran suivante montre l’exemple d’une sauvegarde manuelle :

![procédure](media/HowToHLI/OSBackupTypeIISKUs/HowtoTakeManualBackup.PNG)


## <a name="how-to-restore-a-backup"></a>Comment restaurer une sauvegarde ?

Vous pouvez restaurer une sauvegarde complète ou un fichier individuel à partir de la sauvegarde. Pour effectuer la restauration, utilisez la commande suivante :

```
#tar  -xvf  <backup file>  [Optional <file to restore>]
```
Après la restauration, le fichier est récupéré dans le répertoire de travail actuel.

La commande suivante illustre la restauration d’un fichier */etc/fstab* à partir du fichier de sauvegarde *backup.tar.gz*.
```
#tar  -xvf  /osbackups/hostname/backup.tar.gz  etc/fstab 
```
>[!NOTE] 
>Vous devez copier le fichier dans l’emplacement souhaité après sa restauration à partir de la sauvegarde.

La capture d’écran suivante illustre la restauration d’une sauvegarde complète :

![Capture d'écran représentant une fenêtre d'invite de commandes avec la restauration.](media/HowToHLI/OSBackupTypeIISKUs/HowtoRestoreaBackup.PNG)

## <a name="how-to-install-the-rear-tool-and-change-the-configuration"></a>Comment installer l’outil ReaR et modifier la configuration ? 

Les packages ReaR (Relax-and-Recover) sont **préinstallés** dans les **références (SKU) de type II** des grandes instances HANA, et aucune action de votre part n’est nécessaire. Vous pouvez commencer directement à utiliser ReaR pour la sauvegarde du système d’exploitation.
Toutefois, dans le cas où vous devez installer vous-même les packages, vous pouvez suivre les étapes répertoriées pour installer et configurer l’outil ReaR.

Pour installer les packages de sauvegarde **ReaR**, utilisez les commandes suivantes :

Pour le système d’exploitation **SLES**, utilisez la commande suivante :
```
#zypper install <rear rpm package>
```
Pour le système d’exploitation **RHEL**, utilisez la commande suivante : 
```
#yum install rear -y
```
Pour configurer l’outil ReaR, vous devez mettre à jour les paramètres **OUTPUT_URL** et **BACKUP_URL** dans le *fichier /etc/rear/local.conf*.
```
OUTPUT=ISO
ISO_MKISOFS_BIN=/usr/bin/ebiso
BACKUP=NETFS
OUTPUT_URL="nfs://nfsip/nfspath/"
BACKUP_URL="nfs://nfsip/nfspath/"
BACKUP_OPTIONS="nfsvers=4,nolock"
NETFS_KEEP_OLD_BACKUP_COPY=
EXCLUDE_VG=( vgHANA-data-HC2 vgHANA-data-HC3 vgHANA-log-HC2 vgHANA-log-HC3 vgHANA-shared-HC2 vgHANA-shared-HC3 )
BACKUP_PROG_EXCLUDE=("${BACKUP_PROG_EXCLUDE[@]}" '/media' '/var/tmp/*' '/var/crash' '/hana' '/usr/sap'  ‘/proc’)
```

La capture d’écran suivante illustre la restauration d’une sauvegarde complète : ![Capture d'écran représentant une fenêtre d'invite de commandes avec la restauration à l'aide de l'outil ReaR.](media/HowToHLI/OSBackupTypeIISKUs/RearToolConfiguration.PNG)
