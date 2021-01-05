---
title: Superviser Azure FXT Edge Filer
description: Superviser l'état du matériel pour le cache de stockage hybride Azure FXT Edge Filer
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: how-to
ms.date: 06/20/2019
ms.author: rohogue
ms.openlocfilehash: 86e1d74d5e4ab9f6e799c73bcf0d807d0d874f21
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219715"
---
# <a name="monitor-azure-fxt-edge-filer-hardware-status"></a>Superviser l'état du matériel Azure FXT Edge Filer

Le système de cache de stockage hybride Azure FXT Edge Filer est équipé de plusieurs indicateurs d’état intégrés au châssis pour permettre aux administrateurs de mieux comprendre le fonctionnement du matériel.

## <a name="system-health-status"></a>État d’intégrité du système

Pour superviser les opérations de cache à un niveau supérieur, utilisez le logiciel de la page **Tableau de bord** du Panneau de configuration, comme décrit dans le [Guide du tableau de bord du Panneau de configuration](https://azure.github.io/Avere/legacy/dashboard/4_7/html/ops_dashboard_index.html)

## <a name="hardware-status-leds"></a>LED d'état du matériel

Cette section explique les différents indicateurs d’état intégrés au matériel Azure FXT Edge Filer.

### <a name="hard-drive-status-leds"></a>LED d'état du disque dur

![image de la partie avant du disque dur, horizontale, avec étiquettes de légende 2 (coin supérieur gauche), 1 (coin inférieur gauche) et 3 (côté droit)](media/fxt-monitor/fxt-drive-callouts.png)

Chaque support de disque est équipé de deux LED d'état : un indicateur d’activité (1) et un indicateur d’état (2).

* La LED d'activité (1) s'allume lorsque le disque est en cours d’utilisation.
* La LED d'état (2) indique l'état du disque en utilisant les codes du tableau ci-dessous.

| LED d’état du disque              | Signification  |
|-------------------------------------|----------------------------------------------------------|
| Clignote deux fois en vert par seconde      | Identification du disque *ou* <br> Préparation du disque pour le retrait  |
| Off (éteint)                         | Le système est toujours en cours de démarrage *ou* <br>Le disque peut être retiré |
| Clignote en vert, orange, puis s'arrête       | Une défaillance du disque est à prévoir   |
| Clignote quatre fois en orange par seconde | Échec du disque   |
| Vert continu                         | Le disque est en ligne |

Le côté droit du disque (3) est étiqueté avec sa capacité et autres informations.

Les numéros de disque figurent dans l’espace entre les disques. Dans Azure FXT Edge Filer, le disque 0 est le disque supérieur gauche, et le disque 1 est directement en dessous. La numérotation continue selon ce schéma.

![photo d’une baie de disque dur dans le châssis FXT, montrant les numéros de disque et les étiquettes de capacité](media/fxt-drives-photo.png)

## <a name="left-control-panel"></a>Panneau de configuration gauche

Le panneau de configuration avant gauche est équipé de plusieurs LED d'état (1) et d'un gros indicateur d’intégrité lumineux (2).

![panneau d’état gauche, avec 1, étiquetage des indicateurs d’état sur la gauche et 2, étiquetage du gros indicateur d'état lumineux sur la droite](media/fxt-monitor/fxt-control-panel-left.jpg)

### <a name="control-panel-status-indicators"></a>Indicateurs d’état du panneau de configuration

Les indicateurs d’état de gauche s'allument en orange en cas d'erreur du système. Le tableau ci-dessous décrit les causes possibles et les solutions à ces erreurs.

Si l’erreur persiste après avoir essayé ces solutions, [contactez le support](fxt-support-ticket.md) pour obtenir de l’aide.

| Icône | Description | État d’erreur | Solutions possibles |
|----------------|---------------|--------------------|----------------------|
| ![icône de disque](media/fxt-monitor/fxt-hd-icon.jpg) | État du disque | Erreur du disque | Consultez le journal des événements système pour savoir si le lecteur présente une erreur, ou <br>Exécutez le test de diagnostics en ligne approprié ; redémarrez le système et exécutez les diagnostics (ePSA), ou <br>Si les disques sont configurés dans un contrôleur RAID, redémarrez le système et accédez à l'utilitaire de configuration de l’adaptateur hôte |
|![icône de température](media/fxt-monitor/fxt-temp-icon.jpg) | État de la température | Erreur thermique - par exemple, panne d'un ventilateur ou température ambiante hors limites | Vérifiez les conditions adressables suivantes : <br>Absence ou échec d'un ventilateur de refroidissement <br>Capot du système, prise d’air, module de mémoire vide ou plaque de recouvrement arrière retiré(e) <br>Température ambiante trop élevée <br>Ventilation externe obstruée |
|![icône d’électricité](media/fxt-monitor/fxt-electric-icon.jpg) | État électrique | Erreur électrique - par exemple, tension hors plage, défaillance du bloc d’alimentation ou d'un régulateur de tension |  Vérifiez le journal des événements système ou les messages système pour en savoir plus sur le problème. En cas de problème du bloc d’alimentation, vérifiez la LED d'état correspondante et réinsérez-le, si besoin. |
|![icône de mémoire](media/fxt-monitor/fxt-memory-icon.jpg) | État de la mémoire | Erreur de mémoire | Vérifiez le journal des événements système ou les messages système pour localiser la défaillance de mémoire ; réinsérez le module de mémoire. |
|![icône de la carte PCIe](media/fxt-monitor/fxt-pcie-icon.jpg) | État de la carte PCIe | Erreur de carte PCIe | Redémarrez le système ; mettez à jour les pilotes de la carte PCIe ; réinstallez la carte |

### <a name="system-health-status-indicator"></a>Indicateur d’état d’intégrité du système

Le gros bouton lumineux à droite du panneau de configuration indique l’état général du système et sert aussi d'indicateur de localisation d'unité en mode ID système.

Appuyez sur le bouton ID et intégrité du système pour passer du mode ID système au mode intégrité du système.

|État d’intégrité du système | Condition |
|-------------------------------------------|-----------------------------------------------|
| Bleu continu | Fonctionnement normal : le système est activé, fonctionne normalement et le mode ID système n’est pas actif. <br/>Appuyez sur le bouton ID et intégrité du système pour passer en mode ID système. |
| Bleu clignotant | Le mode ID système est actif. Appuyez sur le bouton ID et intégrité du système pour passer en mode intégrité du système. |
| Orange continu | Le système est en mode de prévention de défaillance. Si le problème persiste, [contactez le Service clientèle et le Support Microsoft](fxt-support-ticket.md). |
| Orange clignotant | Erreur système. Consultez le journal des événements système afin d'y détecter d'éventuels messages d'erreur spécifiques. Pour plus d’informations sur les messages d'événement et d’erreur générés par le microprogramme système et les agents qui supervisent les composants du système, consultez la page de recherche de code d’erreur sur qrl.dell.com. |
