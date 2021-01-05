---
title: Protection de vos ressources réseau dans Azure Security Center
description: Ce document traite des suggestions dans Azure Security Center qui peuvent vous aider à protéger vos ressources réseau Azure et à rester en conformité avec les stratégies de sécurité.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 96c55a02-afd6-478b-9c1f-039528f3dea0
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/05/2019
ms.author: memildin
ms.openlocfilehash: 28d0d7b14c18a1d98be1872b1555930b2e1650c3
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342448"
---
# <a name="protect-your-network-resources"></a>Protéger vos ressources réseau
Azure Security Center analyse continuellement l’état de sécurité de vos ressources Azure pour les meilleures pratiques de sécurité réseau. Lorsque Security Center identifie des failles de sécurité potentielles, il crée des suggestions qui vous guident tout au long du processus de configuration des contrôles nécessaires afin de renforcer et protéger vos ressources.

Pour obtenir la liste complète des recommandations relatives à la mise en réseau, consultez [Recommandations relatives à la mise en réseau](recommendations-reference.md#recs-network).

Cet article traite des suggestions relatives à la sécurité réseau qui s’appliquent à vos ressources Azure. Les suggestions relatives aux réseaux concernent les pare-feux nouvelle génération, les groupes de sécurité réseau, les règles de trafic entrant trop permissives concernant l’accès juste-à-temps aux machines virtuelles et plus encore. Pour obtenir la liste des suggestions relatives à la mise en réseau et des actions de correction, consultez [Gestion des suggestions de sécurité dans Azure Security Center.](security-center-recommendations.md)

Les fonctionnalités de **mise en réseau** de Security Center sont les suivantes : 

- Carte réseau (nécessite Azure Defender)
- [Renforcement réseau adaptatif](security-center-adaptive-network-hardening.md) (nécessite Azure Defender)
- Suggestions en matière de sécurité réseau
 
## <a name="view-your-networking-resources-and-their-recommendations"></a>Afficher vos ressources de mise en réseau et leurs recommandations

Dans la page [Inventaire des ressources](asset-inventory.md), utilisez le filtre de type de ressource pour sélectionner les ressources de mise en réseau que vous souhaitez examiner :

:::image type="content" source="./media/security-center-network-recommendations/network-filters-inventory.png" alt-text="Types de ressources du réseau dans l’inventaire des ressources" lightbox="./media/security-center-network-recommendations/network-filters-inventory.png":::


## <a name="network-map"></a>Mappage réseau

Le mappage réseau interactif fournit un affichage graphique avec des superpositions de sécurité qui vous proposent des suggestions et des aperçus pour renforcer vos ressources réseau. À l’aide du mappage, vous pouvez voir la topologie du réseau de vos charges de travail Azure et des connexions entre vos machines virtuelles et sous-réseaux. Vous pouvez voir la capacité à examiner au niveau du détail certaines ressources ainsi que leurs suggestions à partir du mappage.

Pour ouvrir le mappage réseau :

1. Dans le menu de Security Center, ouvrez le tableau de bord Azure Defender et sélectionnez **Carte réseau**.

    :::image type="content" source="./media/security-center-network-recommendations/opening-network-map.png" alt-text="Types de ressources du réseau dans l’inventaire des ressources" lightbox="./media/security-center-network-recommendations/opening-network-map.png":::

1. Sélectionnez le menu **Couches**, puis **Topologie**.
 
La valeur par défaut du mappage de la topologie affiche :

- Les abonnements que vous avez sélectionné dans Azure. Le mappage prend en charge plusieurs abonnements.
- Machines virtuelles, sous-réseaux et réseaux virtuels du type de ressource Resource Manager (les ressources Azure Classic ne sont pas prises en charge)
- Réseaux virtuels appairés
- Uniquement les ressources disposant de [suggestions de réseau](security-center-recommendations.md) avec un niveau de gravité élevé ou moyen  
- Ressources accessibles sur Internet
- Le mappage est optimisé pour les abonnements que vous avez sélectionnés dans Azure. Si vous modifiez votre sélection, le mappage est recalculé et ré-optimisé en fonction de vos nouveaux paramètres.  

[![Mappage de la topologie de mise en réseau](./media/security-center-network-recommendations/network-map-info.png)](./media/security-center-network-recommendations/network-map-info.png#lightbox)

## <a name="understanding-the-network-map"></a>Compréhension du mappage réseau

Le mappage réseau peut afficher vos ressources Azure dans un affichage **Topologie** et un affichage **Trafic**. 

### <a name="the-topology-view"></a>Affichage Topologie

Dans l’affichage **Topologie** du mappage réseau, vous pouvez afficher les aperçus suivants concernant vos ressources de mise en réseau :

- Dans le cercle intérieur, vous pouvez voir tous les réseaux virtuels au sein de vos abonnements sélectionnés. Le cercle suivant contient tous les sous-réseaux. Le cercle extérieur contient toutes les machines virtuelles.
- Les lignes qui connectent vos ressources dans le mappage vous permettent de savoir quelles ressources sont associées entre elles et de connaître la structure de votre réseau Azure. 
- Utilisez les indicateurs de niveau de gravité pour obtenir rapidement une vue d’ensemble des ressources disposant de suggestions ouvertes de Security Center.
- Vous pouvez cliquer sur les ressources pour les explorer au niveau du détail et afficher les détails et suggestions directement ainsi que dans le contexte du mappage réseau.  
- Si trop de ressources sont affichées sur le mappage, Azure Security Center utilise son algorithme propriétaire pour mettre vos ressources en cluster de manière intelligente. Il met en surbrillance les ressources qui sont dans l’état le plus critique et disposent des suggestions de niveau de gravité le plus élevé. 

Étant donné que le mappage est interactif et dynamique, chaque nœud est interactif, et l’affichage peut changer en fonction des filtres :

1. Vous pouvez modifier ce que vous voyez sur le mappage réseau à l’aide de filtres en haut. Vous pouvez centrer le mappage sur :

   -  **Intégrité de la sécurité** : vous pouvez filtrer le mappage selon la gravité (haute, moyenne, faible) de vos ressources Azure.
   - **Suggestions** : vous pouvez sélectionner les ressources qui sont affichées selon les suggestions actives sur ces ressources. Par exemple, vous pouvez afficher uniquement les ressources pour lesquels Security Center vous recommande d’activer des groupes de sécurité réseau.
   - **Zones de réseau** : par défaut, le mappage affiche uniquement les ressources accessibles sur Internet. Vous pouvez également sélectionner les machines virtuelles internes.
 
2. Vous pouvez cliquer sur **Réinitialiser** dans le coin supérieur gauche à tout moment pour retourner le mappage à son état par défaut.

Pour explorer une ressource au niveau du détail :

1. Lorsque vous sélectionnez une ressource spécifique sur le mappage, le volet droit s’ouvre et vous donne des informations générales sur la ressource, les solutions de sécurité connectées, s’il y en a, et les suggestions relatives à la ressource. Vous aurez le même type de comportement pour chaque type de ressource que vous sélectionnez. 
2. Lorsque vous pointez un nœud du mappage avec la souris, vous pouvez afficher des informations générales sur la ressource, y compris l’abonnement, le type de ressource et le groupe de ressources.
3. Utilisez le lien pour zoomer sur l’info-bulle et recentrer le mappage sur ce nœud spécifique. 
4. Pour recentrer le mappage sur un autre élément que ce nœud spécifique, effectuez un zoom arrière.

### <a name="the-traffic-view"></a>Affichage Trafic

L’affichage **Trafic** vous offre un mappage de tout les trafics possibles entre vos ressources. Il vous un mappage visuel de toutes les règles que vous avez configurées et qui définissent quelles ressources peuvent communiquer avec qui. Cela permet de voir la configuration existante des groupes de sécurité réseau ainsi que d’identifier rapidement les configurations à risque possibles au sein de vos charges de travail.

### <a name="uncover-unwanted-connections"></a>Découvrir les connexions indésirables

Le point fort de cet affichage se situe dans sa capacité à vous montrer ces connexions autorisées avec les vulnérabilités qui peuvent exister. Vous pouvez alors utiliser cette combinaison de données pour effectuer le renforcement nécessaire sur vos ressources. 

Par exemple, vous pouvez détecter deux machines pour lesquelles vous ignoriez qu’elles pouvaient communiquer. Cela vous permet de mieux isoler les charges de travail et les sous-réseaux.

### <a name="investigate-resources"></a>Examiner les ressources

Pour explorer une ressource au niveau du détail :

1. Lorsque vous sélectionnez une ressource spécifique sur le mappage, le volet droit s’ouvre et vous donne des informations générales sur la ressource, les solutions de sécurité connectées, s’il y en a, et les suggestions relatives à la ressource. Vous aurez le même type de comportement pour chaque type de ressource que vous sélectionnez. 
2. Cliquez sur **Trafic** pour afficher la liste des trafics entrant et sortant possibles sur la ressource. Il s’agit d’une liste complète de ceux qui peuvent communiquer avec la ressource, et inversement, ainsi que des protocoles et ports utilisés pour communiquer. Par exemple, quand vous sélectionnez une machine virtuelle, toutes les machines virtuelles avec lesquelles elle peut communiquer sont montrées, et quand vous sélectionnez un sous-réseau, tous les sous-réseaux avec lesquels elle peut communiquer sont montrés.

**Ces données sont basées sur les analyses de groupes de sécurité réseau, ainsi que sur des algorithmes de Machine Learning avancés qui analysent plusieurs règles pour en comprendre les croisements et les interactions.** 

[![Mappage de trafic réseau](./media/security-center-network-recommendations/network-map-traffic.png)](./media/security-center-network-recommendations/network-map-traffic.png#lightbox)


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les recommandations qui s’appliquent à d’autres types de ressources Azure, consultez les rubriques suivantes :

- [Protection de vos machines et de vos applications dans Azure Security Center](./asset-inventory.md)