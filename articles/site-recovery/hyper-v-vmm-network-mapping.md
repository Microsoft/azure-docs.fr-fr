---
title: À propos du mappage réseau Hyper-V (avec VMM) avec Site Recovery
description: Décrit la façon de configurer le mappage réseau pour la récupération d’urgence des machines virtuelles Hyper-V (managées dans des clouds VMM) sur Azure avec Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: 6b68b4c943ec96620427978c2309f27e1fb1f217
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74082554"
---
# <a name="prepare-network-mapping-for-hyper-v-vm-disaster-recovery-to-azure"></a>Préparer le mappage réseau pour la récupération d’urgence de machines virtuelles Hyper-V sur Azure


Cet article vous aide à comprendre et à préparer un mappage réseau pour la réplication de machines virtuelles Hyper-V de clouds System Center Virtual Machine Manager (VMM) vers Azure ou vers un site secondaire, à l’aide du service [Azure Site Recovery](site-recovery-overview.md).


## <a name="prepare-network-mapping-for-replication-to-azure"></a>Préparer le mappage réseau pour la réplication vers Azure

Lorsque vous effectuez une réplication vers Azure, le mappage réseau opère entre des réseaux de machines virtuelles sur un serveur VMM source, et des réseaux virtuels Azure cibles. Il effectue les opérations suivantes :
-  **Connexion réseau** : s’assure que les machines virtuelles Azure répliquées sont connectées au réseau mappé. Toutes les machines qui basculent sur le même réseau peuvent se connecter entre elles, même si elles ont basculé dans des plans de récupération différents.
- **Passerelle réseau** : si une passerelle réseau est configurée sur le réseau Azure cible, les machines virtuelles peuvent se connecter à d’autres machines virtuelles locales.

Le mappage réseau fonctionne de la façon détaillée ici.

- Vous mappez un réseau source de machines virtuelles VMM à un réseau virtuel Azure.
- Après basculement, les machines virtuelles Azure du réseau source sont connectées au réseau virtuel cible mappé.
- Les nouvelles machines virtuelles connectées au réseau de machines virtuelles source sont connectées au réseau Azure mappé lors de la réplication.
- Si le réseau cible est associé à plusieurs sous-réseaux et que l’un d’eux présente le même nom que le sous-réseau dans lequel se trouve la machine virtuelle source, la machine virtuelle de réplication se connecte à ce sous-réseau cible après le basculement.
- S’il n’existe aucun sous-réseau cible avec un nom correspondant, la machine virtuelle se connecte au premier sous-réseau du réseau.

## <a name="prepare-network-mapping-for-replication-to-a-secondary-site"></a>Préparer le mappage réseau pour la réplication vers un site secondaire

Lorsque vous effectuez une réplication vers un site secondaire, le mappage réseau opère entre des réseaux de machines virtuelles sur un serveur VMM source, et des réseaux virtuels sur un serveur VMM cible. Il effectue les opérations suivantes :

- **Connexion réseau** : connecte les machines virtuelles aux réseaux appropriés après basculement. La machine virtuelle de réplication sera connectée au réseau cible mappé au réseau source.
- **Placement optimal des machines virtuelles** : place de manière optimale les machines virtuelles de réplication sur des serveurs hôtes Hyper-V. Les machines virtuelles de réplication sont placés sur des hôtes qui peuvent accéder aux réseaux de machines virtuelles mappés.
- **Aucun mappage réseau** : si vous ne configurez pas de mappage réseau, des machines virtuelles de réplication ne sont pas connectées à des réseaux de machines virtuelles après basculement.

Le mappage réseau fonctionne de la façon détaillée ici.

- Le mappage réseau peut être configuré entre des réseaux de machines virtuelles sur deux serveurs VMM, ou sur un seul serveur VMM, lorsque deux sites sont gérés par le même serveur.
- Lorsque le mappage réseau est correctement configuré et que la réplication est activée, une machine virtuelle située à l’emplacement principal est connectée à un réseau, et son réplica (à l’emplacement cible) est connecté à son réseau mappé.
- Lorsque vous sélectionnez un réseau de machines virtuelles cible dans le cadre du mappage réseau dans Site Recovery, les clouds VMM sources qui utilisent le réseau de machines virtuelles source sont affichés, ainsi que les réseaux de machines virtuelles cibles disponibles sur les clouds cibles qui sont utilisés pour la protection.
- Si le réseau cible est associé à plusieurs sous-réseaux et que l’un d’eux présente le même nom que le sous-réseau dans lequel se trouve la machine virtuelle source, la machine virtuelle de réplication est connectée à ce sous-réseau cible après le basculement. S’il n’existe aucun sous-réseau cible avec un nom correspondant, la machine virtuelle sera connectée au premier sous-réseau du réseau.

## <a name="example"></a>Exemple

Voici un exemple permettant d’illustrer ce processus. Prenons l’exemple d’une entreprise ayant ouvert deux bureaux, l’un à New York et l’autre à Chicago.

**Lieu** | **Serveur VMM** | **Réseaux de machines virtuelles** | **Mappé à**
---|---|---|---
New York | VMM-NewYork| VMNetwork1-NewYork | Mappé au réseau VMNetwork1-Chicago
 |  | VMNetwork2-NewYork | Non mappé
Chicago | VMM-Chicago| VMNetwork1-Chicago | Mappé au réseau VMNetwork1-NewYork
 | | VMNetwork2-Chicago | Non mappé

Dans cet exemple :

- Lorsqu’une machine virtuelle de réplication est créée pour une machine virtuelle connectée au réseau VMNetwork1-NewYork, elle est connectée au réseau VMNetwork1-Chicago.
- Lorsqu’une machine virtuelle de réplication est créée pour le réseau VMNetwork2-NewYork ou VMNetwork2-Chicago, elle n’est pas connecté à un réseau.

Voici comment les clouds VMM sont configurés dans notre exemple d’organisation, ainsi que les réseaux logiques associés aux clouds.

### <a name="cloud-protection-settings"></a>Paramètres de protection des clouds

**Cloud protégé** | **Cloud de protection** | **Réseau logique (New York)**  
---|---|---
GoldCloud1 | GoldCloud2 |
SilverCloud1| SilverCloud2 |
GoldCloud2 | <p>N/D</p><p></p> | <p>LogicalNetwork1-NewYork</p><p>LogicalNetwork1-Chicago</p>
SilverCloud2 | <p>N/D</p><p></p> | <p>LogicalNetwork1-NewYork</p><p>LogicalNetwork1-Chicago</p>

### <a name="logical-and-vm-network-settings"></a>Paramètres des réseaux de machines virtuelles et des réseaux logiques

**Lieu** | **Réseau logique** | **Réseau de machines virtuelles associé**
---|---|---
New York | LogicalNetwork1-NewYork | VMNetwork1-NewYork
Chicago | LogicalNetwork1-Chicago | VMNetwork1-Chicago
 | LogicalNetwork2Chicago | VMNetwork2-Chicago

### <a name="target-network-settings"></a>Paramètres de réseau cible

Selon ces paramètres, lorsque vous sélectionnez le réseau de machines virtuelles cible, le tableau suivant répertorie les choix disponibles.

**Select** | **Cloud protégé** | **Cloud de protection** | **Réseau cible disponible**
---|---|---|---
VMNetwork1-Chicago | SilverCloud1 | SilverCloud2 | Disponible
 | GoldCloud1 | GoldCloud2 | Disponible
VMNetwork2-Chicago | SilverCloud1 | SilverCloud2 | Non disponible
 | GoldCloud1 | GoldCloud2 | Disponible


Si le réseau cible est associé à plusieurs sous-réseaux et que l’un d’eux présente le même nom que le sous-réseau dans lequel se trouve la machine virtuelle source, l’ordinateur virtuel de réplication est connecté à ce sous-réseau cible après le basculement. S’il n’existe aucun sous-réseau cible avec un nom correspondant, la machine virtuelle sera connectée au premier sous-réseau du réseau.


### <a name="failback-behavior"></a>Comportement de restauration automatique

Pour voir ce qui se produit en cas de restauration automatique (réplication inverse), supposons que le réseau VMNetwork1-NewYork est mappé au réseau VMNetwork1-Chicago, avec les paramètres suivants.


**Machine virtuelle** | **Connectée au réseau de machines virtuelles**
---|---
MV1 | VMNetwork1-Network
VM2 (réplica de VM1) | VMNetwork1-Chicago

Examinons ce qui se passe dans différents scénarios possibles avec ces paramètres.

**Scénario** | **Résultat**
---|---
Aucune modification n’est apportée aux propriétés du réseau de la machine VM2 après le basculement. | La machine VM1 reste connectée au réseau source.
Les propriétés du réseau de la machine VM2 sont modifiées après le basculement ; la machine est déconnectée. | La machine VM1 est déconnectée.
Les propriétés du réseau de la machine VM2 sont modifiées après le basculement ; la machine est connectée au réseau VMNetwork2-Chicago. | Si le réseau VMNetwork2-Chicago n’est pas mappé, la machine VM1 est déconnectée.
Le mappage réseau de VMNetwork1-Chicago est modifié. | La machine VM1 est connectée au réseau désormais mappé à VMNetwork1-Chicago.



## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur](hyper-v-vmm-networking.md) l’adressage IP après le basculement sur un site VMM secondaire.
- [En savoir plus](concepts-on-premises-to-azure-networking.md) sur l’adressage IP après le basculement vers Azure
