---
title: Fichier Include
description: Fichier Include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 09/25/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: f8959904482bda8dc26e4df626377027e1ab4bd1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005540"
---
Pour créer un compte de stockage à usage général v2 dans le portail Azure, procédez comme suit :

1. Dans le menu du portail Azure, sélectionnez **Tous les services**. Dans la liste des ressources, tapez **Comptes de stockage**. Au fur et à mesure de la saisie, la liste est filtrée. Sélectionnez **Comptes de stockage**.
1. Sur la fenêtre **Comptes de stockage**, sélectionnez **Ajouter**.
1. Sur l’onglet **Informations de base**, sélectionnez l’abonnement dans lequel créer le compte de stockage.
1. Sous le champ **Groupe de ressources** , sélectionnez le groupe de ressources désiré ou créez-en un.  Pour plus d’informations sur les groupes de ressources Azure, consultez l’article [Vue d’ensemble d’Azure Resource Manager](../articles/azure-resource-manager/management/overview.md).
1. Ensuite, entrez un nom pour votre compte de stockage. Le nom choisi doit être unique dans tout Azure. Le nom doit aussi contenir entre 3 et 24 caractères, et uniquement des lettres minuscules et des chiffres.
1. Sélectionnez l’emplacement de votre compte de stockage ou utilisez l’emplacement par défaut.
1. Sélectionnez un niveau de performances. Le niveau par défaut est *Standard*.
1. Définissez le champ **Type de compte** sur *Stockage V2 (usage général v2)* .
1. Spécifiez la méthode de réplication du compte de stockage. L’option de réplication par défaut est *Stockage géo-redondant avec accès en lecture (RA-GRS)* . Pour plus d’informations sur les options de réplication disponibles, consultez [Redondance de Stockage Azure](../articles/storage/common/storage-redundancy.md).
1. Spécifiez le niveau d’accès pour les objets blob dans le compte de stockage. Le niveau par défaut est *chaud*. Pour plus d’informations sur les niveaux d’accès des objets blob, consultez [Niveaux d’accès chaud, froid et archive pour les objets blob](../articles/storage/blobs/storage-blob-storage-tiers.md).
1. Pour utiliser Azure Data Lake Storage, choisissez l’onglet **Avancé**, puis définissez **Espace de noms hiérarchiques** sur **Activé**. Pour plus d’informations, consultez [Introduction à d’Azure Data Lake Storage Gen2](../articles/storage/blobs/data-lake-storage-introduction.md)
1. Cliquez sur **Vérifier + créer** pour passer en revue vos paramètres de compte de stockage et créer le compte.
1. Sélectionnez **Create** (Créer).

L’illustration suivante montre les paramètres de l’onglet **Informations de base** d’un nouveau compte de stockage :

:::image type="content" source="media/storage-create-account-portal-include/account-create-portal.png" alt-text="Capture d’écran montrant comment créer un compte de stockage dans le portail Azure":::