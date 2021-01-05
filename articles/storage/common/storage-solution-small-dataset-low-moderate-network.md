---
title: Options de transfert de données Azure pour les petits jeux de données avec une bande passante réseau faible ou modérée | Microsoft Docs
description: Votre environnement dispose d'une bande passante réseau faible à modérée et vous souhaitez transférer des petits jeux de données ? Apprenez à choisir la solution de transfert de données Azure qui convient.
services: storage
author: alkohli
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.date: 12/05/2018
ms.author: alkohli
ms.openlocfilehash: f59d1e297ba4d7607d7abd07a78da4784f55d20f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023225"
---
# <a name="data-transfer-for-small-datasets-with-low-to-moderate-network-bandwidth"></a>Transférer de petits jeux de données avec une bande passante réseau faible ou modérée
 
Cet article fournit une vue d'ensemble des solutions de transfert de données adaptées aux environnements disposant d'une bande passante réseau faible à modérée et au transfert de petits jeux de données. Il décrit également les options de transfert de données recommandées et la matrice de fonctionnalités clés correspondant à ce scénario.

Pour une vue d’ensemble de toutes les options de transfert de données disponibles, voir [Choisir une solution de transfert de données Azure](storage-choose-data-transfer-solution.md).

## <a name="scenario-description"></a>Description du scénario

Les petits jeux de données font référence aux données qui se mesurent en Go ou en quelques To. Une bande passante réseau faible ou modérée correspond à un débit entre 45 Mbits/s (connexion T3 dans le centre de données) et 1 Gbit/s.

- Si vous ne transférez que quelques fichiers et n’avez pas besoin d’automatiser le transfert de données, vous pouvez utiliser les outils avec une interface graphique.
- Si vous êtes familiarisé avec l’administration système, vous pouvez utiliser des outils de script/programmatique ou de ligne de commande.

## <a name="recommended-options"></a>Options recommandées

Les options recommandées dans ce scénario sont :

- **Outils avec interface graphique** comme l’Explorateur Stockage Azure et le stockage Azure dans le portail Azure. Ils vous permettent d’afficher facilement vos données et de transférer rapidement quelques fichiers.

    - **Explorateur Stockage Azure** : il s’agit d’un outil multiplateforme qui vous permet de gérer le contenu de vos comptes de stockage Azure. Avec lui, vous pouvez charger, télécharger et gérer des objets blob, des fichiers, des files d’attente, des tables et des entités Azure Cosmos DB. Utilisez-le avec le Stockage Blob pour gérer des objets blob et des dossiers, ainsi que charger et télécharger des objets blob entre votre système de fichiers local et le Stockage Blob, ou entre deux comptes de stockage.
    - **Portail Azure** : le stockage Azure dans le portail Azure fournit une interface basée sur le web pour explorer les fichiers et télécharger de nouveaux fichiers (un à la fois). C’est une bonne solution si vous ne souhaitez pas installer d’outils ni lancer des commandes pour explorer rapidement vos fichiers, ou simplement pour en charger de nouveaux en petite quantité.

- **Outils de script/programmatique** comme les API REST AzCopy/PowerShell/Azure CLI et du stockage Azure.

    - **AzCopy** – utilisez cet outil en ligne de commande pour copier facilement des données vers et à partir du Stockage Blob, Fichier et Table Azure avec des performances optimales. Il prend en charge la concurrence et le parallélisme, ainsi que la possibilité de reprendre les opérations de copie après une interruption.
    - **Azure PowerShell** : pour les utilisateurs qui connaissent l’administration système, il est possible de se servir du module de stockage Azure dans Azure PowerShell pour transférer des données.
    - **Azure CLI** : utilisez cet outil multiplateforme pour gérer les services Azure et charger des données sur le Stockage Azure.
    - **API/kits SDK Stockage Azure** : lorsque vous créez une application, vous pouvez la développer à l'aide des API REST/SDK Stockage Azure et utiliser les bibliothèques clientes disponibles en plusieurs langues.


## <a name="comparison-of-key-capabilities"></a>Comparaison des fonctionnalités clés

Le tableau suivant résume les différences entre les principales fonctionnalités.

| Fonctionnalité | Explorateur de stockage Azure | Portail Azure | AzCopy<br>Azure PowerShell<br>Azure CLI | API REST ou SDK Stockage Azure |
|---------|------------------------|--------------|-----------------------------------------|---------------------------------|
| Disponibilité | Télécharger et installer <br>Outils autonomes | Outils d’exploration basés sur le web dans le portail Azure | Outil de ligne de commande |Interfaces programmables dans .NET, Java, Python, JavaScript, C++, Go, Ruby et PHP |
| Interface graphique | Oui | Oui | Non | Non |
| Plateformes prises en charge | Windows, Mac, Linux | Basé sur le Web |Windows, Mac, Linux |Toutes les plateformes |
| Opérations de stockage Blob autorisées<br>pour les blobs et les dossiers | Télécharger<br>Téléchargement<br>Gérer | Télécharger<br>Téléchargement<br>Gérer |Télécharger<br>Téléchargement<br>Gérer | Oui, personnalisable |
| Stockage Data Lake Gen1 autorisé<br>opérations pour les fichiers et les dossiers | Télécharger<br>Téléchargement<br>Gérer | Non |Télécharger<br>Téléchargement<br>Gérer                   | Non |
| Opérations de stockage de fichiers autorisées<br>pour les fichiers et les répertoires | Télécharger<br>Téléchargement<br>Gérer | Télécharger<br>Téléchargement<br>Gérer   |Télécharger<br>Téléchargement<br>Gérer | Oui, personnalisable |
| Opérations de stockage de tables autorisées<br>pour les tables |Gérer | Non |Prise en charge des tables dans AzCopy v7 |Oui, personnalisable|
| Stockage File d'attente autorisé | Gérer | Non  |Non | Oui, personnalisable|


## <a name="next-steps"></a>Étapes suivantes

- Apprenez à [transférer des données avec l’Explorateur Stockage Azure](../../machine-learning/team-data-science-process/move-data-to-azure-blob-using-azure-storage-explorer.md).
- [Transférer des données avec AzCopy](./storage-use-azcopy-v10.md)