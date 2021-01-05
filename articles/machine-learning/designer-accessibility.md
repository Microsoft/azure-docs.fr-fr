---
title: Utiliser les fonctionnalités d’accessibilité dans le concepteur
titleSuffix: Azure Machine Learning
description: Découvrez les raccourcis clavier et les fonctionnalités d’accessibilité du lecteur d’écran disponibles dans le concepteur.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.author: peterlu
author: peterclu
ms.date: 01/09/2020
ms.custom: designer
ms.openlocfilehash: 86cb5260a59f864658fbb7ac1c1da2d943c6253e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90893432"
---
# <a name="use-a-keyboard-to-use-azure-machine-learning-designer"></a>Employer un clavier pour utiliser le concepteur Azure Machine Learning

Découvrez comment employer un clavier et un lecteur d’écran pour utiliser le concepteur Azure Machine Learning. Pour obtenir une liste des raccourcis clavier qui fonctionnent partout dans le portail Azure, consultez [Raccourcis clavier du portail Azure](../azure-portal/azure-portal-keyboard-shortcuts.md)

Ce workflow a été testé avec le [Narrateur](https://support.microsoft.com/help/22798/windows-10-complete-guide-to-narrator) et [JAWS](https://www.freedomscientific.com/products/software/jaws/), mais il doit fonctionner avec d’autres lecteurs d’écran standard.

## <a name="navigate-the-pipeline-graph"></a>Parcourir le graphe de pipeline

Le graphe de pipeline est organisé sous la forme d’une liste imbriquée. La liste externe est une liste de modules, qui décrit tous les modules dans le graphe de pipeline. La liste interne est une liste de connexions, qui décrit toutes les connexions d’un module spécifique.  

1. Dans la liste de modules, utilisez la touche de direction pour basculer entre les modules.
1. Utilisez la touche Tab pour ouvrir la liste de connexions du module cible.
1. Utilisez la touche de direction pour basculer entre les ports de connexion du module.
1. Utilisez « G » pour accéder au module cible.

## <a name="edit-the-pipeline-graph"></a>Modifier le graphe de pipeline

### <a name="add-a-module-to-the-graph"></a>Ajouter un module au graphe

1. Utilisez Ctrl+F6 pour basculer le focus du canevas vers l’arborescence du module.
1. Recherchez le module souhaité dans l’arborescence du module à l’aide du contrôle TreeView standard.

### <a name="edit-a-module"></a>Modifier un module

Pour connecter un module à un autre module :

1. Utilisez Ctrl+Maj+H quand vous ciblez un module dans la liste de modules pour ouvrir l’application d’assistance à la connexion.
1. Modifiez les ports de connexion du module.

Pour ajuster les propriétés d’un module :

1. Utilisez Ctrl+Maj+E quand vous ciblez un module pour ouvrir les propriétés du module.
1. Modifiez les propriétés du module.

## <a name="navigation-shortcuts"></a>Raccourcis de navigation

| Séquence de touches | Description |
|-|-|
| Ctrl+F6 | Bascule le focus entre le canevas et l’arborescence du module |
| Ctrl+F1   | Ouvre la carte d’informations quand le focus se situe sur un nœud dans l’arborescence du module |
| Ctrl+Maj+H | Ouvre l’application d’assistance à la connexion quand le focus se trouve sur un nœud |
| Ctrl+Maj+E | Ouvre les propriétés du module quand le focus se trouve sur un nœud |
| Ctrl+G | Déplace le focus sur le premier nœud défaillant en cas d’échec de l’exécution du pipeline |

## <a name="action-shortcuts"></a>Raccourcis d’action

Utilisez les raccourcis suivants avec la touche d’accès rapide. Pour plus d’informations sur les touches d’accès rapide, consultez https://en.wikipedia.org/wiki/Access_key.

| Séquence de touches | Action |
|-|-|
| Touche d’accès rapide+R | Exécuter |
| Touche d’accès rapide+P | Publish |
| Touche d’accès rapide+C | Clone |
| Touche d’accès rapide+D | Déployer |
| Touche d’accès rapide+I | Créer/mettre à jour un pipeline d’inférence |
| Touche d’accès rapide+B | Créer/mettre à jour un pipeline d’inférence par lots |
| Touche d’accès rapide+K | Ouvrir la liste déroulante Créer un pipeline d’inférence |
| Touche d’accès rapide+U | Ouvrir la liste déroulante Mettre à jour le pipeline d’inférence |
| Touche d’accès rapide+M | Ouvrir la liste déroulante Plus (...) |

## <a name="next-steps"></a>Étapes suivantes

- [Activer le contraste élevé ou modifier le thème](../azure-portal/set-preferences.md#choose-a-theme-or-enable-high-contrast)
- [Outils liés à l’accessibilité chez Microsoft](https://www.microsoft.com/accessibility)
