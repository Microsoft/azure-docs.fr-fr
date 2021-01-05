---
title: Guide pratique pour répondre aux alertes Azure Defender pour Resource Manager
description: Découvrez la procédure à suivre pour répondre aux alertes d’Azure Defender pour Resource Manager.
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 54790795aab8aac247e17198159130d7139dd38c
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754493"
---
# <a name="respond-to-azure-defender-for-resource-manager-alerts"></a>Répondre aux alertes Azure Defender pour Resource Manager

Lorsque vous recevez une alerte d’Azure Defender pour Resource Manager, nous vous recommandons de l’examiner et d’y répondre de la façon suivante. Azure Defender pour Resource Manager protège toutes les ressources connectées. Ainsi, même si vous connaissez l’application ou l’utilisateur qui a déclenché l’alerte, il est important de vérifier la situation dans laquelle s’inscrit cette dernière.  


## <a name="step-1-contact"></a>Étape 1. Contact

1. Contactez le propriétaire de la ressource pour déterminer si le comportement était attendu ou intentionnel.
1. Si l’activité est conforme à ce qui était prévu, ignorez l’alerte.
1. Si l’activité est inattendue, traitez les comptes d’utilisateurs, les abonnements et les machines virtuelles concernés comme étant compromis et atténuez les risques comme décrit à l’étape suivante.

## <a name="step-2-immediate-mitigation"></a>Étape 2. Atténuation immédiate 

1. Corrigez les comptes d’utilisateur compromis :
    - S’ils ne vous sont pas familiers, supprimez-les, car ils ont peut-être été créés par un intervenant représentant une menace.
    - S’ils vous sont familiers, modifiez leurs informations d’authentification.
    - Utilisez les journaux d’activité Azure pour passer en revue toutes les activités effectuées par l’utilisateur et identifier celles qui sont suspectes.

1. Corrigez les abonnements compromis :
    - Supprimez les runbooks inconnus du compte Automation compromis.
    - Révisez les autorisations IAM pour l’abonnement et supprimez les autorisations pour tout compte d’utilisateur inconnu.
    - Passez en revue toutes les ressources Azure de l’abonnement et supprimez celles qui ne vous sont pas familières.
    - Révisez et examinez les alertes de sécurité pour l’abonnement dans Azure Security Center.
    - Utilisez les journaux d’activité Azure pour passer en revue toutes les activités effectuées par l’abonnement et identifier celles qui sont suspectes.

1. Corrigez les machines virtuelles compromises :
    - Modifiez les mots de passe de tous les utilisateurs.
    - Exécutez une analyse complète du logiciel anti-programme malveillant sur l’ordinateur.
    - Réinitialisez les ordinateurs à partir d’une source exempte de programme malveillant.


## <a name="next-steps"></a>Étapes suivantes

Sur cette page a été expliqué le processus de réponse à une alerte d’Azure Defender pour Resource Manager. Pour toute information connexe, consultez les pages suivantes :

- [Présentation d’Azure Defender pour Resource Manager](defender-for-resource-manager-introduction.md)
- [Supprimer les alertes d’Azure Defender](alerts-suppression-rules.md)
- [Exporter en continu les données Security Center](continuous-export.md)