---
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: include
ms.date: 06/24/2020
ms.author: alkohli
ms.openlocfilehash: 2868e5a53686cfa94dc206c1aab65fe866d19b6d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028571"
---
Voici les limites de la taille des données qui sont copiées dans le compte de stockage. Assurez-vous que les données que vous chargez sont conformes à ces limites. Pour obtenir les informations les plus récentes sur ces limites, consultez [Objectifs de performance et d’extensibilité du stockage Blob](../articles/storage/blobs/scalability-targets.md) et [Objectifs de performance et d’extensibilité pour Azure Files](../articles/storage/files/storage-files-scale-targets.md).

| Taille des données copiées dans le compte de stockage Azure                      | Limite par défaut          |
|---------------------------------------------------------------------|------------------------|
| Objet blob de blocs et objet blob de pages                                            | La limite maximale est identique à la [limite de stockage définie pour l’abonnement Azure](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits), et elle inclut les données de toutes les sources, y compris Data Box. |
| Azure Files                                                          | Data Box prend en charge les partages de fichiers volumineux (100 Tio) s’ils sont activés avant la création de la commande Data Box. <br> Dans le cas contraire, la taille maximale des partages de fichiers est de 5 Tio. <br> Les partages de fichiers Premium ne sont pas encore pris en charge.<br> Tous les dossiers sous *StorageAccount_AzureFiles* doivent respecter cette limite. <br> Pour plus d’informations, consultez [Activation et création de partages de fichiers volumineux](../articles/storage/files/storage-files-how-to-create-large-file-share.md).      |