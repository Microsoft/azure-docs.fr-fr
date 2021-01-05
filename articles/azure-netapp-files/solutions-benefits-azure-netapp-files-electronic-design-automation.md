---
title: Avantages d’utiliser Azure NetApp Files pour automatiser la conception électronique | Microsoft Docs
description: Explique comment la solution Azure NetApp Files permet de répondre aux besoins du secteur de la conception de puces et de semi-conducteurs. Présente des scénarios de test qui exécutent un benchmark standard pour l’automatisation de la conception électronique (EDA, Electronic Design Automation) à l’aide d’Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/24/2020
ms.author: b-juche
ms.openlocfilehash: fcede16619e8488796adc6f4c60af30643c1aadf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82160151"
---
# <a name="benefits-of-using-azure-netapp-files-for-electronic-design-automation"></a>Avantages de l’utilisation d’Azure NetApp Files pour l’automatisation de la conception électronique

Le « time to market » est un facteur critique pour les acteurs du secteur de la conception de semi-conducteurs et de puces. La bande passante doit être élevée et la latence faible pour le stockage. Cet article explique la solution offerte par Azure NetApp Files pour répondre aux besoins du secteur. Il présente des scénarios de test qui exécutent un benchmark standard pour l’automatisation de la conception électronique (EDA, Electronic Design Automation) à l’aide d’Azure NetApp Files. 

## <a name="test-scenario-configurations"></a>Configurations des scénarios de test

Les tests impliquent trois scénarios avec les configurations suivantes. 

|    Scénario    |    Volumes    |    Clients<br> SLES15 D16s_v3  |
|----------------|---------------|--------------------------------|
|    Une         |    1          |    1                           |
|    Deux         |    6          |    24                          |
|    Trois       |    12         |    24                          |

Le premier scénario traite de l’étendue maximale potentielle d’un seul volume.  

Les deuxième et troisième scénarios évaluent les limites d’un point de terminaison Azure NetApp Files unique. Ils étudient les avantages potentiels des limites supérieures d’E/S et de la latence.

## <a name="test-scenario-results"></a>Résultats des scénarios de test

Le tableau suivant récapitule les résultats des scénarios de test.

|    Scénario       |    Taux d’E/S<br>  à 2 ms     |    Taux d’E/S<br>  en périphérie     |    Débit<br>  à 2 ms     |    Débit<br>  en périphérie     |
|-------------------|---------------------------|--------------------------------|-----------------------------|----------------------------------|
|    1 volume       |    39 601                 |    49 502                      |    692 Mio/s                 |    866 Mio/s                      |
|    6 volumes      |    255 613                |    317 000                     |    4 577 Mio/s               |    5 568 Mio/s                    |
|    12 volumes     |    256 612                |    319 196                     |    4 577 Mio/s               |    5 709 Mio/s                    |

Le scénario à volume unique représente la configuration d’application de base. Il s’agit du scénario de base pour les scénarios de test de suivi.  

Le scénario à six volumes illustre une augmentation linéaire (600 %) par rapport à la charge de travail à volume unique.  Tous les volumes au sein d’un même réseau virtuel sont accessibles par le biais d’une seule adresse IP.  

Le scénario à 12 volumes illustre une baisse générale de la latence par rapport au scénario à six volumes, mais il ne présente pas d’augmentation correspondante dans le débit réalisable.   

Le graphe suivant illustre la latence et le taux d’opérations pour la charge de travail EDA sur Azure NetApp Files.  

![Latence et taux d’opérations pour la charge de travail EDA sur Azure NetApp Files](../media/azure-netapp-files/solutions-electronic-design-automation-workload-latency-operation-rate.png)   

Le graphe suivant illustre la latence et le débit pour la charge de travail EDA sur Azure NetApp Files.  

![Latence et débit pour la charge de travail EDA sur Azure NetApp Files](../media/azure-netapp-files/solutions-electronic-design-automation-workload-latency-throughput.png) 

## <a name="layout-of-test-scenarios"></a>Disposition des scénarios de test 

Le tableau ci-dessous récapitule la disposition des scénarios de test.

|    Scénario de test     |    Nombre total de répertoires     |    Nombre total de fichiers     |
|----------------------|------------------------------------|------------------------------|
|    1 volume          |    88 000                          |    880 000                   |
|    6 volumes         |    568 000                         |    5 680 000                 |
|    12 volumes        |    568 000                         |    5 680 000                 |

La charge de travail complète est une combinaison de phases physiques et fonctionnelles s’exécutant simultanément. Elle représente un flux classique d’un ensemble d’outils EDA vers un autre.   

La phase fonctionnelle est constituée de spécifications initiales et d’une conception logique. La phase physique a lieu quand la conception logique est convertie en puce physique. Au cours des phases de validation et de tape-out, les contrôles finaux sont effectués et la conception est fournie à une usine de fabrication.  

La phase fonctionnelle comprend une combinaison d’E/S de lecture et d’écriture séquentielles et aléatoires. La phase fonctionnelle est gourmande en métadonnées, comme les opérations stat de fichiers et les appels d’accès. Bien que les opérations de métadonnées soient effectivement sans taille, les opérations de lecture et d’écriture sont comprises entre moins de 1 K et 16 Ko. La plupart des lectures sont comprises entre 4 K et 16 Ko. La plupart des écritures sont inférieures ou égales à 4 K. La phase physique est composée entièrement d’opérations de lecture et d’écriture séquentielles, avec une combinaison de tailles d’opérations de 32 K et 64 K.  

Dans les graphes ci-dessus, la majeure partie du débit provient de la phase physique séquentielle de la charge de travail. Les E/S proviennent de la petite phase fonctionnelle aléatoire et gourmande en métadonnées. Les deux phases se produisent en parallèle. 

En conclusion, vous pouvez associer Azure Compute à Azure NetApp Files pour l’EDA afin d’obtenir une bande passante scalable. 

## <a name="next-steps"></a>Étapes suivantes

- [Architectures de solution avec Azure NetApp Files](azure-netapp-files-solution-architectures.md)
