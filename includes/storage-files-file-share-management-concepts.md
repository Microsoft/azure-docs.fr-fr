---
title: Fichier include
description: Fichier include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/26/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 8d26b6c92462abc13c42257d2c6e571156eacc0f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011754"
---
Les partages de fichiers Azure sont déployés dans des *comptes de stockage*, qui sont des objets de niveau supérieur représentant un pool de stockage partagé. Ce pool de stockage peut être utilisé pour déployer plusieurs partages de fichiers ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob, des files d’attente ou des tables. Toutes les ressources de stockage qui sont déployées dans un compte de stockage partagent les limites qui s’appliquent à ce compte de stockage. Pour voir les limites actuelles d’un compte de stockage, consultez [Objectifs de scalabilité et de performances d’Azure Files](../articles/storage/files/storage-files-scale-targets.md).

Il existe deux principaux types de comptes de stockage que vous allez utiliser pour les déploiements d’Azure Files : 
- **Comptes de stockage version 2 (GPv2) universels** : Les comptes de stockage GPv2 vous permettent de déployer des partages de fichiers Azure sur du matériel Standard/sur disque dur (HDD). En plus de stocker les partages de fichiers Azure, les comptes de stockage GPv2 peuvent stocker d’autres ressources de stockage, telles que des conteneurs d’objets blob, des files d’attente ou des tables. 
- **Comptes de stockage FileStorage** : Les comptes de stockage FileStorage vous permettent de déployer des partages de fichiers Azure sur du matériel Premium/sur disque SSD. Les comptes FileStorage peuvent uniquement être utilisés pour stocker des partages de fichiers Azure. Aucune autre ressource de stockage (conteneurs d’objets blob, files d’attente, tables, etc.) ne peut être déployée dans un compte FileStorage. Seuls les comptes FileStorage peuvent déployer des partages de fichiers SMB et NFS.

Il existe plusieurs autres types de comptes de stockage que vous pouvez rencontrer dans le portail Azure, PowerShell ou l’interface CLI. Deux types de comptes de stockage, BlockBlobStorage et BlobStorage, ne peuvent pas contenir de partages de fichiers Azure. Les deux autres types de comptes de stockage que vous pouvez voir sont les comptes version 1 (GPv1) universels et les comptes classiques, qui peuvent tous deux contenir des partages de fichiers Azure. Bien que les comptes de stockage GPv1 et classiques puissent contenir des partages de fichiers Azure, la plupart des nouvelles fonctionnalités d’Azure Files sont disponibles uniquement dans les comptes de stockage GPv2 et FileStorage. Nous vous recommandons donc d’utiliser uniquement les comptes de stockage GPv2 et FileStorage pour les nouveaux déploiements ainsi que pour mettre à niveau les comptes de stockage GPv1 et classiques s’ils existent déjà dans votre environnement.  