---
title: Guide pratique pour répondre aux alertes Azure Defender pour DNS
description: Découvrez la procédure à suivre pour répondre aux alertes d’Azure Defender pour DNS.
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 375a81127a000741ca5e0397642800794610bcda
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754510"
---
# <a name="respond-to-azure-defender-for-dns-alerts"></a>Répondre aux alertes Azure Defender pour DNS

Lorsque vous recevez une alerte d’Azure Defender pour DNS, nous vous recommandons de l’examiner et d’y répondre de la façon suivante. Azure Defender pour DNS protège toutes les ressources connectées. Ainsi, même si vous connaissez l’application ou l’utilisateur qui a déclenché l’alerte, il est important de vérifier la situation dans laquelle s’inscrit cette dernière.  


## <a name="step-1-contact"></a>Étape 1. Contact

1. Contactez le propriétaire de la ressource pour déterminer si le comportement était attendu ou intentionnel.
1. Si l’activité est conforme à ce qui était prévu, ignorez l’alerte.
1. Si l’activité est inattendue, traitez la ressource comme potentiellement compromise et atténuez les risques comme décrit à l’étape suivante.

## <a name="step-2-immediate-mitigation"></a>Étape 2. Atténuation immédiate 

1. Isolez la ressource du réseau pour empêcher tout mouvement latéral.
1. Exécutez une analyse complète du logiciel anti-programme malveillant sur la ressource en suivant les conseils de correction qui en résultent.
1. Examinez les logiciels installés et en cours d’exécution sur la ressource en supprimant les packages inconnus ou indésirables.
1. Rétablissez la machine à un bon état de fonctionnement connu en réinstallant le système d’exploitation, le cas échéant, et en restaurant les logiciels à partir d’une source vérifiée et exempte de programmes malveillants.
1. Résolvez les recommandations d’Azure Security Center pour la machine en corrigeant les problèmes de sécurité mis en évidence afin d’éviter les futures violations.


## <a name="next-steps"></a>Étapes suivantes

Sur cette page a été expliqué le processus de réponse à une alerte d’Azure Defender pour DNS. Pour toute information connexe, consultez les pages suivantes :

- [Présentation d’Azure Defender pour DNS](defender-for-dns-introduction.md)
- [Supprimer les alertes d’Azure Defender](alerts-suppression-rules.md)
- [Exporter en continu les données Security Center](continuous-export.md)