---
title: Voir l’état des tâches Azure Import/Export | Microsoft Docs
description: Découvrez comment voir l’état des tâches d’importation/exportation Azure et des lecteurs utilisés. Comprendre les facteurs qui affectent le temps nécessaire au traitement d’un travail.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 05/17/2018
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: dc8c944e1eb665f3f0bc83e28e1e5469d2da501a
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92781989"
---
# <a name="view-the-status-of-azure-importexport-jobs"></a>Voir l’état des tâches Azure Import/Export

Cet article fournit des informations sur la vérification de l’état des lecteurs et des tâches Azure Import/Export. Le service Azure Import/Export est utilisé pour transférer de manière sécurisée de grandes quantités de données dans des objets blob Azure et Azure Files. Le service est également utilisé pour exporter des données du Stockage Blob Azure.  

## <a name="view-job-and-drive-status"></a>Voir l’état de la tâche et des lecteurs
Vous pouvez suivre l’état de vos travaux d’importation ou d’exportation à partir du portail Azure en sélectionnant l’onglet **Importer/Exporter** . La liste de vos tâches s’affiche sur la page.

![Afficher l’état des tâches](./media/storage-import-export-service/jobstate.png)

## <a name="view-job-status"></a>Afficher le statut de la tâche

Selon la phase de traitement de votre lecteur, vous obtiendrez l’un des statuts suivants :

| Statut de tâche | Description |
|:--- |:--- |
| Creating | Dès qu’une tâche est créée, son état est défini sur **Création** . Quand le travail est en état **Création** , le service Import/Export suppose que les disques n’ont pas été expédiés au centre de données. Un travail peut rester dans cet état pendant deux semaines maximum. Passé ce délai, le service le supprime automatiquement. |
| Expédition | Après avoir expédié votre colis, vous devez mettre à jour les informations de suivi dans le portail Azure.  Cela a pour effet de modifier l’état du travail en **Expédition** . La tâche reste dans l’état **Expédition** pendant deux semaines maximum. 
| Reçu | Quand tous les lecteurs ont été reçus au centre de données, l’état de la tâche est défini sur **Reçu** . |
| Transferring | Une fois que le traitement a commencé pour au moins un lecteur, l’état de la tâche est défini sur **Transfert** . Pour plus d’informations, accédez à [États de lecteur](#view-drive-status). |
| Packaging | Une fois que tous les lecteurs ont été traités, la tâche est définie sur l’état **Emballage** jusqu’à ce que les lecteurs vous soient réexpédiés. |
| Completed | Une fois que tous les lecteurs vous ont été réexpédiés, si la tâche s’est effectuée sans erreur, son état est défini sur **Effectué** . La tâche est automatiquement supprimée au bout de 90 jours dans l’état **Effectué** . |
| Fermés | Une fois que tous les lecteurs vous sont réexpédiés, si des erreurs se sont produites pendant le traitement du travail, celui-ci est défini sur **Fermé** . La tâche est automatiquement supprimée au bout de 90 jours dans l’état **Fermé** . |

## <a name="view-drive-status"></a>Afficher l’état des disques

Le tableau ci-dessous décrit le cycle de vie d’un disque individuel tout au long d’une tâche d’importation ou d’exportation. L’état actuel de chaque lecteur d’une tâche est visible dans le portail Azure.

Le tableau suivant décrit chacun des états par lesquels le disque d’une tâche peut passer.

| État du disque | Description |
|:--- |:--- |
| Spécifié | Pour une tâche d’importation, quand elle est créée dans le portail Azure, l’état initial d’un lecteur est **Spécifié** . Pour une tâche d’exportation, comme aucun lecteur n’est spécifié à la création de la tâche, l’état initial du lecteur est **Reçu** . |
| Reçu | Le lecteur passe à l’état **Reçu** une fois que le service Import/Export a traité les lecteurs reçus de l’entreprise de transport pour une tâche d’importation. Pour une tâche d’exportation, l’état initial du lecteur est l’état **Reçu** . |
| NeverReceived (Jamais reçu) | Le lecteur passe à l’état **NeverReceived** quand le colis associé à une tâche est arrivé, mais ne contient pas le lecteur. Un lecteur passe également à cet état si le centre de données n’a pas encore reçu le package, et si le service a reçu les informations d’expédition au moins deux semaines auparavant. |
| Transferring | Un lecteur passe à l’état **Transfert** quand le service commence à transférer les données du lecteur dans le Stockage Azure. |
| Completed | Un lecteur passe à l’état **Effectué** quand le service a transféré toutes les données sans erreurs.
| CompletedMoreInfo (Terminé avec des informations) | Un lecteur passe à l’état **CompletedMoreInfo** après que le service a rencontré des erreurs pendant la copie des données à destination ou à partir du disque. Les informations peuvent inclure des erreurs, des avertissements ou des messages d’informations sur le remplacement des objets blob.
| ShippedBack (Renvoyé) | Un lecteur passe à l’état **ShippedBack** quand il a été réexpédié par le centre de données à l’adresse de retour. |

Cette image à partir du portail Azure affiche l’état du lecteur d’un exemple de travail :

![Afficher l’état des disques](./media/storage-import-export-service/drivestate.png)

Le tableau suivant décrit les états associés aux défaillances de disque et les mesures mises en œuvre pour chacun de ces états.

| État du disque | Événement | Résolution / Étape suivante |
|:--- |:--- |:--- |
| NeverReceived (Jamais reçu) | Un lecteur marqué comme **NeverReceived** (parce qu’il n’a pas été reçu dans le cadre de l’expédition du travail) arrive dans une autre expédition. | L’équipe des opérations passe le lecteur à l’état **Reçu** . |
| N/A | Un lecteur qui n’est associé à aucune tâche arrive au centre de données dans le cadre d’un autre travail. | Le lecteur est marqué comme lecteur supplémentaire. Il vous est renvoyé quand le travail associé au package d’origine est terminé. |

## <a name="time-to-process-job"></a>Temps de traitement du travail
La durée de traitement d’une tâche d’importation/exportation varie en fonction de plusieurs facteurs comme les suivants :

-  Délai d’expédition
-  Chargement au centre de données
-  Type de tâche et taille des données copiées
-  Nombre de disques d’une tâche. 

Le service Import/Export n’a pas de contrat de niveau de service mais s’efforce d’effectuer la copie en 7 à 10 jours après réception des disques. En plus de l’état publié sur le portail Azure, vous pouvez utiliser des API REST pour suivre la progression du travail. Utilisez le paramètre de pourcentage effectué dans l’appel d’API de l’opération [Répertorier les travaux](/previous-versions/azure/dn529083(v=azure.100)) pour fournir le pourcentage de progression de la copie.


## <a name="next-steps"></a>Étapes suivantes

* [Transférer des données avec l’utilitaire de ligne de commande AzCopy](./storage-use-azcopy-v10.md)
* [Exemple d’API REST Azure Import Export](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/)