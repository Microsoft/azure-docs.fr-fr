---
title: Conseils sur la récupération d’urgence pour Azure Data Lake Storage Gen1 | Microsoft Docs
description: Découvrez comment protéger vos données contre les pannes à l’échelle régionale ou les suppressions accidentelles au-delà du stockage localement redondant d’Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 02/21/2018
ms.author: twooley
ms.openlocfilehash: 48136f8d9172c3674e849e24efca4ae5070f83ab
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109117"
---
# <a name="high-availability-and-disaster-recovery-guidance-for-data-lake-storage-gen1"></a>Conseils sur la haute disponibilité et la reprise d’activité pour Data Lake Storage Gen1

Data Lake Storage Gen1 fournit un stockage localement redondant (LRS). Par conséquent, les données de votre compte Data Lake Storage Gen1 sont résilientes aux défaillances matérielles temporaires au sein d’un centre de données grâce à des réplicas automatisés. Cela garantit durabilité et haute disponibilité et permet de respecter le contrat de niveau de service de Data Lake Storage Gen1. Cet article donne des conseils sur la protection des données contre les rares pannes ou suppressions accidentelles qui peuvent se produire à l’échelle d’une région.

## <a name="disaster-recovery-guidance"></a>Guide de récupération d’urgence

Il est essentiel pour vous de préparer un plan de reprise d’activité. Passez en revue les informations contenues dans cet article et ces ressources supplémentaires pour mieux créer votre propre plan.

* [Récupération d’urgence et haute disponibilité pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery)
* [Guide technique de la résilience Azure](/azure/architecture/framework/resiliency/overview)

### <a name="best-practice-recommendations"></a>Recommandations de bonnes pratiques

Nous vous recommandons de copier vos données critiques vers un autre compte Data Lake Storage Gen1 dans une autre région et à une fréquence adaptée aux besoins de votre plan de récupération d’urgence. Il existe différents moyens de copier des données, notamment [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md), [Azure PowerShell](data-lake-store-get-started-powershell.md) ou [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md). Azure Data Factory est un service utile pour créer et déployer des pipelines de déplacement des données à intervalles réguliers.

En cas de panne régionale, vous pouvez alors accéder à vos données dans la région où les données ont été copiées. Vous pouvez surveiller le [tableau de bord Azure Service Health](https://azure.microsoft.com/status/) pour déterminer l’état du service Azure dans le monde entier.

## <a name="data-corruption-or-accidental-deletion-recovery-guidance"></a>Aide à la récupération suite à une corruption de données ou à une suppression accidentelle

Si Data Lake Storage Gen1 assure la résilience des données via des réplicas automatisés, cela ne protège aucunement votre application (ou les développeurs/utilisateurs) de la corruption des données ou d’une suppression accidentelle.

Pour éviter toute suppression accidentelle, nous vous recommandons de définir tout d’abord les stratégies d’accès adaptées à votre compte Data Lake Storage Gen1. Cela inclut la mise en œuvre de [verrous de ressources Azure](../azure-resource-manager/management/lock-resources.md) pour verrouiller des ressources importantes ainsi que l’application d’un contrôle d’accès au niveau du compte et des fichiers à l’aide des [fonctionnalités de sécurité Data Lake Storage Gen1](data-lake-store-security-overview.md) disponibles. Nous vous recommandons également de créer régulièrement des copies de vos données critiques avec [ADLCopy](data-lake-store-copy-data-azure-storage-blob.md), [Azure PowerShell](data-lake-store-get-started-powershell.md) ou [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md) dans un autre compte Data Lake Storage Gen1, dossier ou abonnement Azure. Cela peut servir à la récupération après un incident de corruption ou de suppression de données. Azure Data Factory est un service utile pour créer et déployer des pipelines de déplacement des données à intervalles réguliers.

Vous pouvez également activer la [journalisation des diagnostics](data-lake-store-diagnostic-logs.md) pour un compte Data Lake Storage Gen1 afin de collecter des pistes d’audit d’accès aux données. Les pistes d’audit fournissent des informations sur les personnes qui ont peut-être supprimé ou mis à jour un fichier.

## <a name="next-steps"></a>Étapes suivantes

* [Prise en main de Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)