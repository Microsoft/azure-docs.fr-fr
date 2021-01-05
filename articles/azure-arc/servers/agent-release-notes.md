---
title: Nouveautés de l’agent des serveurs activés par Azure Arc
description: Cet article contient les notes de publication de l’agent des serveurs activés par Azure Arc. Pour la plupart des problèmes résumés ici, il existe des liens menant à des informations supplémentaires.
ms.topic: conceptual
ms.date: 12/14/2020
ms.openlocfilehash: 3a01113d0d6416fb050f5c66191d5c420b7ac137
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505645"
---
# <a name="whats-new-with-azure-arc-enabled-servers-agent"></a>Nouveautés de l’agent des serveurs activés par Azure Arc

L’agent Connected Machine des serveurs activés par Azure Arc reçoit des améliorations de manière continue. Pour vous informer des développements les plus récents, cet article détaille les thèmes suivants :

- Versions les plus récentes
- Problèmes connus
- Résolution des bogues

## <a name="december-2020"></a>Décembre 2020

### <a name="new-feature"></a>Nouvelle fonctionnalité

Ajout de la prise en charge de Windows Server 2008 R2

### <a name="fixed"></a>Fixe

Résolution d’un problème empêchant l’installation de l’extension de script personnalisé sur Linux.

## <a name="november-2020"></a>Novembre 2020

### <a name="fixed"></a>Fixe

Résolution d’un problème où la configuration du proxy pouvait être perdue après la mise à niveau des distributions RPM.

## <a name="october-2020"></a>Octobre 2020

### <a name="fixed"></a>Fixe

- Correction d’un script proxy pour gérer l’emplacement alternatif du fichier d’unité de démon GC.
- Modification de la fiabilité de l’agent GuestConfig.
- Prise en charge de l’agent GuestConfig pour la région US Gov Virginie.
- Les messages de rapport de l’extension de l’agent GuestConfig sont plus détaillés en cas d’échec.

## <a name="september-2020"></a>Septembre 2020

Version : 1.0 (Disponibilité générale)

### <a name="plan-for-change"></a>Modification planifiée

- La prise en charge des agents en préversion (toutes les versions antérieures à 1.0) sera supprimée dans une prochaine mise à jour de service.
- Suppression de la prise en charge du point de terminaison de secours `.azure-automation.net`. Si vous avez un proxy, vous devez autoriser le point de terminaison `*.his.arc.azure.com`.
- Si Azure Connected Machine Agent est installé sur une machine virtuelle hébergée dans Azure, il n’est pas possible d’installer ou de modifier des extensions de machine virtuelle à partir de la ressource Serveurs activés par Arc. Cela permet d’éviter les opérations d’extension conflictuelles effectuées à partir des ressources **Microsoft.Compute** et **Microsoft.HybridCompute** de l’ordinateur virtuel. Utilisez la ressource **Microsoft.Compute** pour l’ordinateur pour toutes les opérations d’extension.
- Le nom du processus Configuration des invité a changé, passant de *gcd* à *gcad* sur Linux, et de *gcservice* à *gcarcservice* sur Windows.

### <a name="new-feature"></a>Nouvelle fonctionnalité

- Ajout de l’option `azcmagent logs` pour collecter des informations à des fins de support.
- Ajout de l’option `azcmagent license` pour afficher le CLUF.
- Ajout de l’option `azcmagent show --json` pour sortir l’état de l’agent dans un format facile à analyser.
- Ajout de l’indicateur dans la sortie `azcmagent show` pour indiquer si le serveur se trouve sur une machine virtuelle hébergée dans Azure.
- Ajout de l’option `azcmagent disconnect --force-local-only` pour autoriser la réinitialisation de l’état de l’agent local lorsque le service Azure est inaccessible.
- Ajout de l’option `azcmagent connect --cloud` pour prendre en charge des clouds supplémentaires. Dans cette version, seul Azure est pris en charge par le service au moment de la mise en production de l’agent.
- L’agent a été localisé dans les langues prises en charge par Azure.

### <a name="fixed"></a>Fixe

- Améliorations de la vérification de la connectivité.
- Correction du problème lié à la perte des paramètres du serveur proxy lors de la mise à niveau de l’agent sur Linux.
- Problèmes résolus lors de la tentative d’installation de l’agent sur un serveur exécutant Windows Server 2012 R2.
- Améliorations de la fiabilité d’installation des extensions

## <a name="august-2020"></a>Août 2020

Version : 0.11

- Cette mise en production a annoncé précédemment la prise en charge d’Ubuntu 20.04. Étant donné que certaines extensions de machine virtuelle Azure ne prennent pas en charge Ubuntu 20.04, la prise en charge de cette version d’Ubuntu est en cours de suppression.

- Améliorations de la fiabilité des déploiements d’extension.

### <a name="known-issues"></a>Problèmes connus

Si vous utilisez une version antérieure de l’agent Linux et qu’elle est configurée pour utiliser un serveur proxy, vous devez reconfigurer le paramètre de serveur proxy après la mise à niveau. Pour cela, exécutez `sudo azcmagent_proxy add http://proxyserver.local:83`.

## <a name="next-steps"></a>Étapes suivantes

Avant d'évaluer ou d'activer des serveurs avec Azure Arc sur plusieurs machines hybrides, consultez [Vue d'ensemble d'Azure Connected Machine Agent](agent-overview.md) pour en savoir plus sur les exigences et les détails techniques relatifs à l'agent, ainsi que sur les méthodes de déploiement.