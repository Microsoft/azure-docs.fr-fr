---
title: Notes de publication d’Azure Data Box Gateway 1905 | Microsoft Docs
description: Décrit les problèmes majeurs existants et les résolutions pour Azure Data Box Gateway 1905 exécutant la version en disponibilité générale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: 7337e940ff67f0c0cfd38e5e58a2ac81bf593970
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580938"
---
# <a name="azure-data-box-edge-and-azure-data-box-gateway-1905-release-notes"></a>Notes de publication d’Azure Data Box Edge et d’Azure Data Box Gateway 1905

## <a name="overview"></a>Vue d’ensemble

Les notes de publication suivantes identifient les problèmes majeurs existants et les problèmes résolus dans la version 1905 d’Azure Data Box Edge et Azure Data Box Gateway.

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez attentivement les informations contenues dans les notes de publication avant de déployer votre Data Box Edge/Data Box Gateway. 

Cette version correspond aux versions logicielles :

- **Data Box Gateway 1905 (1.6.887.626)**
- **Data Box Edge 1905 (1.6.887.626)**

> [!NOTE]
> La mise à jour 1905 peut être appliquée uniquement aux appareils Data Box Edge qui exécutent la version en disponibilité générale du logiciel.

## <a name="whats-new"></a>Nouveautés

- **Améliorations de journalisation du Field Programmable Gate Array (FPGA)**  : dans cette version, nous avons apporté des améliorations de journalisation et d’alerte au FPGA. Il s’agit d’une mise à jour requise pour Data Box Edge si vous utilisez la fonctionnalité de computing en périphérie avec le FPGA. Pour plus d’informations, voyez comment [transformer des données avec le computing en périphérie sur votre Data Box Edge](../databox-online/azure-stack-edge-deploy-configure-compute-advanced.md).

## <a name="known-issues-in-ga-release"></a>Problèmes connus dans la version mise à la disposition générale

Aucun nouveau problème n’est indiqué dans les notes de publication de cette version. Tous les problèmes mentionnés dans les notes de publication proviennent des versions précédentes. Pour afficher la liste des problèmes connus, accédez à [Problèmes connus dans la version mise à la disposition générale](data-box-gateway-release-notes.md#known-issues-in-ga-release).


## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Data Box Gateway](data-box-gateway-deploy-prep.md)
- [Préparer le déploiement d’Azure Data Box Edge](../databox-online/azure-stack-edge-deploy-prep.md)
