---
title: Disponibilité générale de Microsoft Threat Modeling Tool version du 22/03/2020 – Azure
description: Compilation des notes de publication pour l’outil de modélisation des menaces version 7.3.00316.1.
author: jegeib
ms.author: jegeib
ms.service: security
ms.topic: article
ms.date: 03/22/2020
ms.openlocfilehash: e182d40d20529b5743fa9105c68108a8ae55d943
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94516898"
---
# <a name="threat-modeling-tool-update-release-73003161---03222020"></a>Version mise à jour de Threat Modeling Tool 7.3.00316.1 – 22/03/2020

La version 7.3.00316.1 de Microsoft Threat Modeling Tool (TMT) a été publiée le 22 mars 2020 et contient les modifications suivantes :

- Améliorations de l’accessibilité
- Résolution des bogues
- Nouvelle fonctionnalité DiagramReader

## <a name="notable-bug-fixes"></a>Correctifs de bogues notables

### <a name="exporting-the-threat-list-to-csv"></a>Exportation de la liste des menaces au format CSV

La fonction d’exportation en CSV sélectionnait de manière incohérente les champs de la liste des menaces qui devaient être exportés. Désormais, tous les champs de la liste des menaces sont exportés dans le fichier CSV. 

### <a name="ux-bugs"></a>Bogues UX

- Les menus d’aide du workflow principal (créer/ouvrir/analyser) et l’éditeur de modèle ont désormais des options de menu cohérentes.
- La barre de recherche du volet Gabarits comporte désormais un curseur standard et des étiquettes appropriées ont été ajoutées.

## <a name="new-features"></a>Nouvelles fonctionnalités

### <a name="diagramreader-feature-has-been-added"></a>La fonctionnalité DiagramReader a été ajoutée

Une nouvelle fonctionnalité DiagramReader a été ajoutée dans le menu principal lorsqu’un modèle est ouvert. Cette fonctionnalité permet de convertir la représentation graphique du modèle en une narration textuelle. 

## <a name="system-requirements"></a>Configuration système requise

- Systèmes d’exploitation pris en charge :
  - [Mise à jour anniversaire Windows 10](https://blogs.windows.com/windowsexperience/2016/08/02/how-to-get-the-windows-10-anniversary-update/#HTkoK5Zdv0g2F2Zq.97) ou version ultérieure
- Version .NET requise :
  - [.NET 4.7.1](https://go.microsoft.com/fwlink/?LinkId=863262) ou version ultérieure
- Autres conditions requises :
  - Une connexion Internet pour recevoir des mises à jour de l’outil ainsi que des modèles

## <a name="documentation-and-feedback"></a>Documentation et commentaires

- La documentation de Threat Modeling Tool se trouve sur [docs.microsoft.com](./threat-modeling-tool.md) et contient des informations [sur l’utilisation de l’outil](./threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Étapes suivantes

Télécharger la version la plus récente de [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).