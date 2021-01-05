---
title: Ajouter/supprimer un point de terminaison de serveur Azure File Sync | Microsoft Docs
description: Apprenez à ajouter ou supprimer un point de terminaison de serveur à l’aide d’Azure File Sync. Un point de terminaison de serveur est un emplacement spécifique sur un serveur inscrit, comme un dossier sur un volume de serveur.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: f75f0d1ae12db11590f8ce62f3c7b4c0f3e12817
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013936"
---
# <a name="addremove-an-azure-file-sync-server-endpoint"></a>Ajouter/supprimer un point de terminaison de serveur Azure File Sync
Azure File Sync vous permet de centraliser les partages de fichiers de votre organisation dans Azure Files sans perdre la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Pour ce faire, Azure File Sync transforme vos serveurs Windows en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible sur Windows Server pour accéder à vos données localement (y compris SMB, NFS et FTPS) et vous pouvez avoir autant de caches que nécessaire dans le monde entier.

Un *point de terminaison de serveur* représente un emplacement spécifique sur un *serveur inscrit*, comme un dossier sur un volume de serveur ou la racine du volume. Plusieurs points de terminaison de serveur peuvent se trouver sur le même volume si leurs espaces de noms ne se chevauchent pas (par exemple, F:\sync1 et F:\sync2) et si chaque point de terminaison est synchronisé avec un groupe de synchronisation unique. Vous pouvez configurer des stratégies de hiérarchisation cloud individuellement pour chaque point de terminaison de serveur. Si vous ajoutez un emplacement de serveur avec un ensemble de fichiers existants comme point de terminaison de serveur à un groupe de synchronisation, ces fichiers sont fusionnés avec tout autre fichier déjà présent sur les autres points de terminaison dans le groupe de synchronisation.

Consultez [Comment déployer Azure File Sync](storage-sync-files-deployment-guide.md) pour plus d’informations sur comment déployer Azure File Sync de bout en bout.

## <a name="prerequisites"></a>Conditions préalables requises
Pour créer un point de terminaison de serveur, vous devez d’abord vérifier que les critères suivants sont remplis : 
- L’agent de synchronisation de fichiers Azure est installé le serveur et ce dernier a été inscrit. Vous trouverez des instructions pour l’installation de l’agent Azure File Sync dans l’article [Inscrire/désinscrire un serveur auprès d’Azure File Sync](storage-sync-files-server-registration.md). 
- Vérifiez qu’un service de synchronisation de stockage a été déployé. Consultez le [Guide pratique pour déployer Azure File Sync](storage-sync-files-deployment-guide.md) pour en savoir plus sur le déploiement d’un service de synchronisation de stockage. 
- Vérifiez qu’un groupe de synchronisation a été déployé. Découvrez comment [Créer un groupe de synchronisation](storage-sync-files-deployment-guide.md#create-a sync-group-and-a-cloud-endpoint).
- Vérifiez que le serveur est connecté à Internet et qu’Azure est accessible. Nous utilisons le port 443 pour toutes les communications entre le serveur et notre service.

## <a name="add-a-server-endpoint"></a>Ajouter un point de terminaison de serveur
Pour ajouter un point de terminaison de serveur, accédez au groupe de synchronisation souhaité, puis sélectionnez « Ajouter un point de terminaison de serveur ».

![Ajouter un nouveau point de terminaison de serveur dans le volet Groupe de synchronisation](media/storage-sync-files-server-endpoint/add-server-endpoint-1.png)

Les informations suivantes sont requises sous l’option **Ajouter un point de terminaison de serveur** :

- **Serveur inscrit** : nom du serveur ou du cluster sur lequel créer le point de terminaison de serveur.
- **Chemin d’accès** : chemin sur le serveur Windows Server à synchroniser en tant qu’élément du groupe de synchronisation.
- **Hiérarchisation cloud** : commutateur pour activer ou désactiver la hiérarchisation cloud. Quand elle est activée, la hiérarchisation cloud *hiérarchise* les fichiers dans vos partages de fichiers Azure. Les partages de fichiers locaux sont alors convertis en cache, plutôt qu’en copie complète du jeu de données, pour vous aider à optimiser l’espace sur votre serveur.
- **Espace libre du volume** : quantité d’espace libre à réserver sur le volume sur lequel se trouve le point de terminaison de serveur. Par exemple, si l’espace libre du volume est définie sur 50 % sur un volume avec un point de terminaison de serveur unique, environ la moitié de la quantité de données sera hiérarchisée dans Azure Files. Que la hiérarchisation cloud soit activée ou non, le partage de fichiers Azure dispose toujours d’une copie complète des données dans le groupe de synchronisation.

Sélectionnez **Créer** pour ajouter le point de terminaison de serveur. Les fichiers situés dans un espace de noms d’un groupe de synchronisation seront désormais synchronisés. 

## <a name="remove-a-server-endpoint"></a>Supprimer un point de terminaison de serveur
Si vous ne voulez plus utiliser Azure File Sync pour un point de terminaison de serveur donné, vous pouvez supprimer le point de terminaison de serveur. 

> [!Warning]  
> Ne tentez pas de résoudre les problèmes de synchronisation, de hiérarchisation cloud ou tout autre aspect d’Azure File Sync en supprimant et recréant le point de terminaison de serveur, sauf si un ingénieur Microsoft vous le demande explicitement. La suppression d’un point de terminaison de serveur est une opération destructrice, et les fichiers hiérarchisés sur le point de terminaison de serveur ne sont pas « reconnectés » à leur emplacement sur le partage de fichiers Azure après la recréation du point de terminaison de serveur, ce qui entraîne des erreurs de synchronisation. Notez également que les fichiers hiérarchisés qui existent en dehors de l’espace de noms de point de terminaison de serveur risquent d’être définitivement perdus. Les fichiers hiérarchisés peuvent exister dans votre point de terminaison de serveur même si la hiérarchisation cloud n’a jamais été activée.

Pour garantir que tous les fichiers hiérarchisés sont rappelés avant de supprimer le point de terminaison de serveur, désactivez la hiérarchisation cloud sur le point de terminaison de serveur, puis exécutez l’applet de commande PowerShell suivante pour rappeler tous les fichiers hiérarchisés au sein de l’espace de noms de votre point de terminaison de serveur :

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -Order CloudTieringPolicy
```
Spécifier `-Order CloudTieringPolicy` rappelle, dans un premier temps, les fichiers récemment modifiés.
Les autres paramètres facultatifs mais utiles à prendre en compte sont les suivants :
* `-ThreadCount` détermine le nombre de fichiers rappelés en parallèle.
* `-PerFileRetryCount` détermine la fréquence à laquelle un rappel portant sur une fichier actuellement bloqué est tenté.
* `-PerFileRetryDelaySeconds` détermine le délai, en secondes, entre les tentatives de rappel, et doit systématiquement être utilisé avec le paramètre précédent.

> [!Note]  
> Si le volume local qui héberge le serveur n’a pas assez d’espace libre pour rappeler toutes les données hiérarchisées, l’applet de commande `Invoke-StorageSyncFileRecall` échoue.  

Pour supprimer le point de terminaison de serveur

1. Accédez au service de synchronisation de stockage auquel votre serveur est inscrit.
2. Accédez au groupe de synchronisation souhaité.
3. Supprimez le point de terminaison de serveur souhaité du groupe de synchronisation dans le service de synchronisation de stockage. Pour ce faire, vous pouvez cliquer avec le bouton droit sur le point de terminaison de serveur concerné dans le volet Groupe de synchronisation.

    ![Suppression d’un point de terminaison de serveur d’un groupe de synchronisation](media/storage-sync-files-server-endpoint/remove-server-endpoint-1.png)

## <a name="next-steps"></a>Étapes suivantes
- [Inscrire ou désinscrire un serveur auprès d’Azure File Sync](storage-sync-files-server-registration.md)
- [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Superviser Azure File Sync](storage-sync-files-monitoring.md)
