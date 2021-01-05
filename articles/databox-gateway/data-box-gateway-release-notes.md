---
title: Notes de mise à disponibilité générale d’Azure Data Box Gateway | Microsoft Docs
description: Décrit les problèmes majeurs existants et les résolutions pour Azure Data Box Gateway exécutant la version en disponibilité générale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: 2984f7990b9570c5ec57633de7f7e50162fb6f46
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580965"
---
# <a name="azure-data-box-edgeazure-data-box-gateway-general-availability-release-notes"></a>Notes de mise à disponibilité générale d’Azure Data Box Edge/Azure Data Box Gateway

## <a name="overview"></a>Vue d’ensemble

Les notes de publication suivantes identifient les problèmes majeurs existants et les problèmes résolus dans la version en disponibilité générale (GA) d’Azure Data Box Edge et Azure Data Box Gateway. 

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez attentivement les informations contenues dans les notes de publication avant de déployer votre Data Box Edge/Data Box Gateway.

La version en disponibilité générale correspond aux versions logicielles :

- **Data Box Gateway 1903 (1.5.814.447)**
- **Data Box Edge 1903 (1.5.814.447)**


## <a name="whats-new"></a>Nouveautés

- **Nouvelles images de disque virtuel** : de nouveaux VHD et VMDK sont désormais disponibles dans le portail Azure. Téléchargez ces images pour provisionner, configurer et déployer de nouveaux appareils Data Box Gateway GA. Les appareils Data Box Gateway créés dans les préversions antérieures ne peuvent pas être mis à jour vers cette version. Pour plus d’informations, consultez [Tutorial: Prepare to deploy Azure Data Box Gateway (Preview)](data-box-gateway-deploy-prep.md) (Didacticiel : Préparation du déploiement d’Azure Data Box Gateway (préversion)).
- **Prise en charge de NFS** : la prise en charge de NFS est actuellement en préversion et disponible pour les clients v3.0 et v4.1 qui accèdent aux appareils Data Box Edge et Data Box Gateway.
- **Résilience du stockage** : votre appareil Data Box Edge peut résister à la défaillance d’un disque de données grâce à la fonctionnalité de résilience du stockage. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Vous pouvez activer la résilience du stockage en sélectionnant l’option **Résilient** dans les **Paramètres de stockage** dans l’interface utilisateur web locale.


## <a name="known-issues-in-ga-release"></a>Problèmes connus dans la version mise à la disposition générale

Le tableau suivant résume les problèmes connus pour la version actuelle de Data Box Gateway.

| Non. | Fonctionnalité | Problème | Solution de contournement/commentaires |
| --- | --- | --- | --- |
| **1.** |Types de fichier | Les types de fichiers suivants ne sont pas pris en charge : fichiers de caractères, fichiers de blocs, sockets, pipes et liens symboliques.  |La copie de ces fichiers entraîne la création de fichiers de longueur nulle sur le partage NFS. Ces fichiers restent dans un état d’erreur et sont également signalés dans le fichier *error.xml*. <br> Si des liens symboliques existent vers des répertoires, ces derniers ne sont jamais marqués hors connexion. Par conséquent, vous pouvez ne pas voir la croix grise sur les répertoires, qui indique qu’ils sont hors connexion et que tout le contenu associé a été complètement chargé vers Azure. |
| **2.** |Suppression | En raison d’un bogue dans cette version, si un partage NFS est supprimé, le partage peut ne pas être supprimé. L’état du partage indique *Suppression*.  |Cela ne se produit que lorsque le partage utilise un nom de fichier non pris en charge. |
| **3.** |Copier | Échec de la copie des données avec l’erreur :  Impossible de terminer l’opération demandée du fait d’une limitation du système de fichiers.  |Les autres flux de données associés à une taille de fichier supérieure à 128 Ko ne sont pas pris en charge.   |


## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Data Box Gateway](data-box-gateway-deploy-prep.md).
- [Préparer le déploiement d’Azure Data Box Edge](../databox-online/azure-stack-edge-deploy-prep.md).
