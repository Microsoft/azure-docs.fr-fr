---
title: Fichier include
description: Fichier include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: b7c097306ba46d4f0024aecc55994508e2d8a090
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011718"
---
Pour protéger les données de vos partages de fichiers Azure contre la perte ou l’altération des données, tous les partages de fichiers Azure stockent plusieurs copies de chaque fichier au fur et à mesure de leur écriture. En fonction des exigences de votre charge de travail, vous pouvez sélectionner des degrés de redondance supplémentaires. Azure Files prend actuellement en charge les options de redondance des données suivantes :

- **Localement redondant** : Le stockage localement redondant, souvent appelé LRS, signifie que chaque fichier est stocké trois fois au sein d’un cluster de stockage Azure. Cela le protège de la perte de données due à des défaillances matérielles, telles qu’un lecteur de disque défectueux.
- **Redondant interzone** : Le stockage redondant interzone, souvent appelé ZRS, signifie que chaque fichier est stocké trois fois sur trois clusters de stockage Azure distincts. À l’instar du stockage localement redondant, la redondance de zone offre trois copies de chaque fichier. Celles-ci sont toutefois isolées physiquement dans trois clusters de stockage distincts situés au sein de différentes *zones de disponibilité* Azure. Les Zones de disponibilité sont des emplacements physiques uniques au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Une écriture sur le stockage n’est pas acceptée tant qu’elle n’est pas écrite dans les clusters de stockage dans les trois zones de disponibilité. 
- **Géoredondant** : Le stockage géoredondant, souvent appelé GRS, est comparable au stockage localement redondant, car un fichier est stocké trois fois au sein d’un cluster de stockage Azure dans la région primaire. Toutes les écritures sont ensuite répliquées de façon asynchrone dans une région secondaire définie par Microsoft. Le stockage géoredondant fournit six copies de vos données réparties entre deux régions Azure. En cas de sinistre majeur, tel que la perte permanente d’une région Azure en raison d’une catastrophe naturelle ou d’un autre événement similaire, Microsoft effectue un basculement afin que la région secondaire en vigueur devienne la région primaire, assurant toutes les opérations. Étant donné que la réplication entre les régions primaire et secondaire est asynchrone, en cas de sinistre majeur, les données qui ne sont pas encore répliquées dans la région secondaire seront perdues. Vous pouvez également effectuer le basculement manuel d’un compte de stockage géoredondant.
- **Géoredondant interzone** : Le stockage géoredondant interzone, souvent appelé GZRS, est comparable au stockage redondant interzone, car un fichier est stocké trois fois sur trois clusters de stockage distincts dans la région primaire. Toutes les écritures sont ensuite répliquées de façon asynchrone dans une région secondaire définie par Microsoft. Le processus de basculement pour le stockage géoredondant interzone fonctionne de la même façon que pour le stockage géoredondant.

Les partages de fichiers Azure Standard prennent en charge les quatre types de redondance, tandis que les partages de fichiers Azure Premium prennent uniquement en charge les stockages redondant localement et redondant interzone.

Les comptes de stockage v2 universels (GPv2) fournissent deux options de redondance supplémentaires qui ne sont pas prises en charge par Azure Files : le stockage géographiquement redondant avec accès en lecture, souvent appelé RA-GRS, et le stockage géographiquement redondant interzone avec accès en lecture, souvent appelé RA-GZRS. Vous pouvez approvisionner des partages de fichiers Azure dans des comptes de stockage avec ces options définies, mais Azure Files ne prend pas en charge la lecture de la région secondaire. Les partages de fichiers Azure déployés dans des comptes de stockage géographiquement redondant ou géographiquement redondant interzone avec accès en lecture sont facturés en tant que stockage géoredondant ou géoredondant interzone, respectivement.