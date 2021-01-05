---
title: Fichier include
description: Fichier include
services: batch
author: JnHs
ms.service: batch
ms.topic: include
ms.date: 04/06/2018
ms.author: jenhayes
ms.custom: include file
ms.openlocfilehash: 019e8db54c1cfd9f436f880b8ddbb9bfa31c50bc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96026901"
---
Une tâche qui s’exécute dans Azure Batch peut produire des données de sortie pendant son exécution. Le plus souvent, les données de sortie des tâches doivent être stockées pour être récupérées par d’autres tâches du travail et/ou par l’application cliente qui a exécuté le travail. Les tâches écrivent les données de sortie dans le système de fichiers d’un nœud de calcul Batch, mais toutes les données du nœud sont perdues lorsque le nœud est réinitialisé ou qu’il quitte le pool. En outre, les tâches sont parfois associées à une période de rétention de fichier, au terme de laquelle les fichiers créés par la tâche sont supprimés. Pour toutes ces raisons, il est important de conserver la sortie des tâches dont vous aurez besoin par la suite dans un magasin de données tel que [Stockage Azure](../articles/storage/index.yml).

Pour plus d’options de compte de stockage dans Batch, consultez [Comptes de Stockage Azure et Batch](../articles/batch/accounts.md#azure-storage-accounts).