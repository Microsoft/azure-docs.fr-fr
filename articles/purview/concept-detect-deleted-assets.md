---
title: Comment les analyses détectent les éléments supprimés
description: Cet article explique comment un compte Azure Purview détecte les ressources supprimées lors des analyses.
author: yaronyg
ms.author: yarong
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 10/16/2020
ms.openlocfilehash: 9b1515ef00355c831e161c01227678197792cc20
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550566"
---
# <a name="how-scans-detect-deleted-assets"></a>Comment les analyses détectent les éléments supprimés

Cet article explique comment Azure Purview utilise les résultats d’analyse pour détecter les ressources supprimées.

## <a name="background-info"></a>Informations générales

Un catalogue Azure Purview est conscient de l’état d’un magasin de données uniquement lorsqu’il l’analyse. Pour que le catalogue sache si un fichier, une table ou un conteneur a été supprimé(e), il compare la sortie de la dernière analyse à la sortie de l’analyse actuelle. Par exemple, supposons que la dernière fois que vous avez analysé un compte Azure Data Lake Storage Gen2, il incluait un dossier nommé *dossier1*. Lorsque le même compte est analysé à nouveau, *dossier1* est manquant. Par conséquent, le catalogue part du principe que le dossier a été supprimé.

## <a name="detecting-deleted-files"></a>Détection des fichiers supprimés

La logique de détection des fichiers manquants fonctionne pour plusieurs analyses effectuées par le même utilisateur et par différents utilisateurs. Supposons, par exemple, qu’un utilisateur exécute une analyse ponctuelle sur un magasin de données Data Lake Storage Gen2 sur les dossiers A, B et C. Plus tard, un autre utilisateur du même compte exécute une autre analyse ponctuelle sur les dossiers C, D et E du même magasin de données. Étant donné que le dossier C a été analysé deux fois, le catalogue le vérifie à la recherche de suppressions possibles. Toutefois, les dossiers A, B, D et E n’ont été analysés qu’une seule fois. Le catalogue ne vérifie donc pas les éventuelles ressources supprimées.

Pour conserver les fichiers supprimés hors de votre catalogue, il est important d’exécuter des analyses régulières. L’intervalle d’analyse est important, car le catalogue ne peut pas détecter les ressources supprimées tant qu’une autre analyse n’est pas exécutée. Par conséquent, si vous exécutez des analyses une fois par mois sur un magasin spécifique, le catalogue ne peut pas détecter les ressources de données supprimées dans ce magasin tant que vous n’avez pas exécuté la prochaine analyse un mois plus tard.

Lorsque vous énumérez des magasins de données volumineux comme Data Lake Storage Gen2, il existe plusieurs façons (notamment les erreurs d’énumération et les événements supprimés) de passer à côté des informations. Une analyse spécifique peut ne pas détecter lorsqu’un fichier a été créé ou supprimé. Par conséquent, sauf si le catalogue est certain qu’un fichier a été supprimé, il ne le supprime pas du catalogue. Cette stratégie signifie qu’il peut y avoir des erreurs lorsqu’un fichier qui n’existe pas dans le magasin de données analysé existe toujours dans le catalogue. Dans certains cas, il peut être nécessaire d’analyser un magasin de données deux ou trois fois avant de pouvoir détecter certaines ressources supprimées.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à utiliser les catalogues Azure Purview, consultez [Démarrage rapide : Créer un compte Purview](create-catalog-portal.md).
