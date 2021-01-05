---
title: Notes de publication d’Azure Data Box Gateway et Azure Data Box Edge 1906 | Microsoft Docs
description: Décrit les problèmes majeurs existants et les solutions pour Azure Data Box Gateway et Azure Data Box Edge, version 1906.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: 51f4995ffd6a86022d95df15ae0eb7694d878c60
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580935"
---
# <a name="azure-data-box-edge-and-azure-data-box-gateway-1906-release-notes"></a>Notes de publication d’Azure Data Box Edge et Azure Data Box Gateway 1906

Les notes de publication suivantes identifient les problèmes majeurs existants et les problèmes résolus dans la version 1906 d’Azure Data Box Edge et Azure Data Box Gateway. 

Les notes de publication sont régulièrement mises à jour ; les problèmes critiques nécessitant une solution de contournement sont ajoutés au fur et à mesure de leur découverte. Lisez attentivement les informations contenues dans les notes de publication avant de déployer votre Data Box Edge/Data Box Gateway.

Cette version correspond aux versions logicielles :

- **Data Box Gateway 1906 (1.6.978.743)**
- **Data Box Edge 1906 (1.6.978.743)**

> [!NOTE]
> La mise à jour 1906 peut être appliquée uniquement aux appareils Data Box Edge qui exécutent la version en disponibilité générale ou 1905 du logiciel.

## <a name="whats-new"></a>Nouveautés

- **Résolution de bogue dans le workflow de gestion de clé de récupération** - Dans la version antérieure, un bogue empêchait l’application de la clé de récupération. Ce bogue est résolu dans cette version. Nous vous recommandons vivement d’appliquer cette mise à jour, car la clé de récupération vous permet de récupérer les données de l’appareil, au cas où celui-ci ne pourrait pas démarrer. Pour plus d’informations, consultez les instructions indiquant comment [enregistrer la clé de récupération durant le déploiement de Data Box Edge ou Data Box Gateway](../databox-online/azure-stack-edge-deploy-connect-setup-activate.md#set-up-and-activate-the-physical-device).
- **Améliorations de la journalisation du FPGA (Field Programmable Gate Array)** - À partir de la version 1905, des améliorations ont été apportées à la journalisation et aux alertes relatives au FPGA. Il s’agit toujours d’une mise à jour obligatoire pour Data Box Edge, si vous utilisez la fonctionnalité de computing en périphérie avec le FPGA. Pour plus d’informations, voyez comment [transformer des données avec le computing en périphérie sur votre Data Box Edge](../databox-online/azure-stack-edge-deploy-configure-compute-advanced.md).

## <a name="known-issues-in-ga-release"></a>Problèmes connus dans la version mise à la disposition générale

Aucun nouveau problème n’est indiqué dans les notes de publication de cette version. Tous les problèmes mentionnés dans les notes de publication proviennent des versions précédentes. Pour afficher la liste des problèmes connus, accédez à [Problèmes connus dans la version mise à la disposition générale](data-box-gateway-release-notes.md#known-issues-in-ga-release).


## <a name="next-steps"></a>Étapes suivantes

- [Préparer le déploiement d’Azure Data Box Gateway](data-box-gateway-deploy-prep.md)
- [Préparer le déploiement d’Azure Data Box Edge](../databox-online/azure-stack-edge-deploy-prep.md)
