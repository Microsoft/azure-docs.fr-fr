---
title: Fichier include
description: Fichier include
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 04/27/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: 1a51cd242eb26709464aa502938cd04807aea721
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97805666"
---
## <a name="common-scenarios"></a>Scénarios courants
Le mode rafale peut être très avantageux pour les scénarios suivants :
- **Amélioration des temps de démarrage** : avec le mode rafale, le démarrage de votre instance est sensiblement plus rapide. Par exemple, le disque de système d’exploitation par défaut pour les machines virtuelles Premium est le disque P4 dont les performances plafonnent à 120 IOPS et 25 Mo/s. En mode rafale, les performances du disque P4 peuvent atteindre 3 500 IOPS et 170 Mo/s, ce qui permet de diviser par 6 le temps de démarrage.
- **Gestion des traitements par lots** : certaines charges de travail de l’application qui sont de nature cyclique nécessitent la plupart du temps des performances de base, et pendant de brèves périodes des performances supérieures. Ce peut être le cas d’un programme de comptabilité qui traite au quotidien des transactions nécessitant un trafic de disque assez modéré. Ensuite, à la fin du mois, le programme rapproche des rapports nécessitant un trafic de disque beaucoup plus élevé.
- **Préparation aux pics de trafic** : les serveurs web et leurs applications peuvent être confrontés à des pics de trafic à tout moment. Si votre serveur web est adossé à des machines virtuelles ou à des disques utilisant le mode rafale, il est mieux équipé pour gérer les pics de trafic. 

## <a name="bursting-flow"></a>Flux de rafale
Le système de crédit de rafales s’applique de la même manière au niveau de la machine virtuelle et du disque. Votre ressource, qu’il s’agisse d’une machine virtuelle ou d’un disque, commence par utiliser des crédits entièrement stockés. Ces crédits autorisent des rafales de 30 minutes au débit de rafale maximal. Des crédits de rafale s’accumulent lorsque votre ressource s’exécute dans le cadre de ses limites de performances de stockage sur disque. Pour les IOPS et Mo/s que votre ressource utilise en deçà de la limite de performances, vous commencez à accumuler des crédits. Si votre ressource a accumulés des crédits à utiliser pour le mode rafale et que votre charge de travail a besoin de performances supplémentaires, votre ressource peut utiliser ces crédits pour dépasser votre limite de performances afin de bénéficier des performances d’e/s de disque dont elle a besoin pour répondre à la demande.



![Diagramme de compartiment de rafale](media/managed-disks-bursting/bucket-diagram.jpg)

C’est à vous de décider comment vous voulez utiliser les 30 minutes de rafale. Vous pouvez l’utiliser pendant 30 minutes consécutivement ou de manière sporadique tout au long de la journée. Lorsque le produit est déployé, il est prêt à recevoir des crédits complets et, lorsqu’il épuise les crédits, il faut moins d’une journée pour obtenir de nouveau des crédits entièrement mis à profit. Vous pouvez accumuler et dépenser leurs crédits de rafale à votre discrétion et il n’est pas nécessaire que le compartiment de 30 minutes soit à nouveau complet pour la rafale. À propos de l’accumulation de crédits de rafale, il faut noter qu’il diffère pour chaque ressource, car il est basé sur les IOPS et les Mo/s inutilisés sous les volumes de performances attribués. Cela signifie que des produits offrant des performances de base plus élevées peuvent accumuler des crédits de rafales plus rapidement que des produits offrant des performances de base inférieures. Par exemple, un disque P1 inactif accumule 120 IOPS, tandis qu’un disque P20 en accumule 2 300 quand il est inactif.

## <a name="bursting-states"></a>États de rafale
Lorsque le mode rafale est activé, votre ressource peut être dans trois états :
- **Accumulation** : le trafic d’e/s de la ressource utilise moins de capacités que la cible de performance. L’accumulation de crédits de rafale est effectuée séparément pour les IOPS et les Mo/s. Votre ressource peut accumuler des crédits d’IOPS et utiliser des crédits de Mo/s ou inversement.
- **Rafale** : le trafic de la ressource utilise plus de capacités que la cible de performance. Le trafic en rafale consomme indépendamment les crédits de l’IOPS ou de la bande passante.
- **Constant** : le trafic de la ressource est exactement au niveau de la cible de performance.

## <a name="examples-of-bursting"></a>Exemples de rafales
Les exemples suivants illustrent le fonctionnement du mode rafale avec différentes combinaisons de machines virtuelles et de disques. Pour que les exemples soient faciles à suivre, nous allons nous concentrer sur les Mo/s, mais la même logique s’applique aux IOPS.

### <a name="non-burstable-virtual-machine-with-burstable-disks"></a>Machine virtuelle ne pouvant pas utiliser le mode rafale avec des disques pouvant utiliser le mode rafale
**Combinaison de machines virtuelles et de disques :** 
- Standard_D8as_v4 
    - Mo/s non mis en cache : 192
- Disque de système d’exploitation P4
    - Mo/s approvisionnés : 25
    - Maximum de Mo/s en rafale : 170 
- 2 disques de données P10 
    - Mo/s approvisionnés : 100
    - Maximum de Mo/s en rafale : 170

 Lorsque la machine virtuelle démarre, elle récupère les données du disque du système d’exploitation. Étant donné que ce disque fait partie d’une machine virtuelle en cours de démarrage, il est saturé de crédits de rafale. Ces crédits permettent au disque du système d’exploitation de pousser sa vitesse de démarrage à 170 Mo/s, comme illustré ci-dessous :

![Démarrage avec MV sans rafale et disque en mode rafale](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-startup.jpg)

Une fois le démarrage terminé, une application est exécutée sur la machine virtuelle, dont la charge de travail n’est pas critique. Cette charge de travail nécessite 15 Mo/s qui sont répartis uniformément sur tous les disques :

![Inactivité avec MV sans rafale et disque en mode rafale](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-idling.jpg)

L’application doit ensuite effectuer un traitement par lots nécessitant 192 Mo/s. 2 Mo/s sont utilisés par le disque du système d’exploitation, le reste étant réparti uniformément entre les disques de données :

![Rafale avec MV sans rafale et disque en mode rafale](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-bursting.jpg)

### <a name="burstable-virtual-machine-with-non-burstable-disks"></a>Machine virtuelle pouvant utiliser le mode rafale avec des disques ne pouvant pas utiliser le mode rafale
**Combinaison de machines virtuelles et de disques :** 
- Standard_L8s_v2 
    - Mo/s non mis en cache : 160
    - Maximum de Mo/s en rafale : 1 280
- Disque de système d’exploitation P50
    - Mo/s approvisionnés : 250 
- 2 disques de données P10 
    - Mo/s approvisionnés : 250

 Après le démarrage initial, une application est exécutée sur la machine virtuelle, dont la charge de travail n’est pas critique. Cette charge de travail nécessite 30 Mo/s qui sont répartis uniformément sur tous les disques : ![Inactivité avec MV en mode rafale et disque sans rafale](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-normal.jpg)

L’application doit ensuite effectuer un traitement par lots nécessitant 600 Mo/s. La machine virtuelle Standard_L8s_v2 passe en mode rafale pour traiter cette demande, puis les requêtes adressées aux disques sont réparties uniformément sur les disques P50 :

![Rafale avec MV en mode rafale et disque sans rafale](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-bursting.jpg)
### <a name="burstable-virtual-machine-with-burstable-disks"></a>Machine virtuelle pouvant utiliser le mode rafale avec des disques ne pouvant pas utiliser le mode rafale
**Combinaison de machines virtuelles et de disques :** 
- Standard_L8s_v2 
    - Mo/s non mis en cache : 160
    - Maximum de Mo/s en rafale : 1 280
- Disque de système d’exploitation P4
    - Mo/s approvisionnés : 25
    - Maximum de Mo/s en rafale : 170 
- 2 disques de données P4 
    - Mo/s approvisionnés : 25
    - Maximum de Mo/s en rafale : 170 

Lorsque la VM démarre, elle passe en mode rafale pour demander sa limite de rafale de 1 280 Mo/s au disque du système d’exploitation qui répond avec des performances de rafale de 170 Mo/s :

![Démarrage avec MV en mode rafale et disque en mode rafale](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-startup.jpg)

Une fois le démarrage terminé, une application est exécutée sur la machine virtuelle. L’application a une charge de travail non critique nécessitant 15 Mo/s, qui est répartie uniformément sur tous les disques :

![Inactivité avec MV en mode rafale et disque en mode rafale](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-idling.jpg)

L’application doit ensuite effectuer un traitement par lots nécessitant 360 Mo/s. La machine virtuelle Standard_L8s_v2 passe en mode rafale pour traiter cette demande, puis les requêtes. Le disque du système d’exploitation ne nécessite uniquement 20 Mo/s. Les 340 Mo/s restants sont gérés par les disques de données P4 en mode rafale :  

![Rafale avec MV en mode rafale et disque en mode rafale](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-bursting.jpg)
