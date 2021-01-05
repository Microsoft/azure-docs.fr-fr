---
title: Index des exemples de blueprint
description: Index d’exemples de conformité et standard pour le déploiement d’environnements, de stratégies et de bases du Framework d’adoption du cloud avec Azure Blueprints.
ms.date: 09/14/2020
ms.topic: sample
ms.openlocfilehash: def5e51a3db7e0733e0d7e1ef5c6895aedb7128d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90530979"
---
# <a name="azure-blueprints-samples"></a>Exemples Azure Blueprints

Le tableau suivant inclut des liens vers des exemples pour Azure Blueprints. Chaque exemple est de qualité production et prêt à être déployé aujourd’hui pour vous aider à répondre à vos divers besoins de conformité.

## <a name="standards-based-blueprint-samples"></a>Exemples de blueprints basés sur des normes

| Exemple | Description |
|---------|---------|
| [Australian Government ISM PROTECTED](./ism-protected/index.md) | Fournit des garde-fous pour la conformité à la norme Australian Government ISM PROTECTED. |
| [Benchmark de sécurité Azure](./azure-security-benchmark.md) | Propose des garde-fous pour la conformité au [benchmark de sécurité Azure](../../../security/benchmarks/overview.md). |
| [PBMM fédéral du Canada](./canada-federal-pbmm/index.md) | Propose des garde-fous pour la conformité au profil PBMN (Protected B, Medium Integrity, Medium Availability) fédéral du Canada. |
| [CIS Microsoft Azure Foundations Benchmark](./cis-azure-1-1-0.md)| Fournit un ensemble de stratégies pour vous aider à vous conformer aux recommandations de CIS Microsoft Azure Foundations Benchmark. |
| [DoD Impact Level 4](./dod-impact-level-4/index.md) | Fournit un ensemble de stratégies vous permettant de vous conformer à DoD Impact Level 4. |
| [DoD Impact Level 5](./dod-impact-level-5/index.md) | Fournit un ensemble de stratégies vous permettant de vous conformer à DoD Impact Level 5. |
| [FedRAMP Moderate](./fedramp-m/index.md) | Fournit un ensemble de stratégies pour vous aider à vous conformer à FedRAMP Moderate. |
| [FedRAMP High](./fedramp-h/index.md) | Fournit un ensemble de stratégies vous permettant de vous conformer à FedRAMP High. |
| [HIPAA HITRUST 9.2](./hipaa-hitrust-9-2.md) | Fournit un ensemble de stratégies permettant de se conformer à HIPAA HITRUST. |
| [IRS 1075](./irs-1075/index.md) | Propose des garde-fous pour la conformité à la norme IRS 1075.|
| [ISO 27001](./iso27001/index.md) | Propose des garde-fous pour la conformité à la norme ISO 27001. |
| [Services partagés ISO 27001](./iso27001-shared/index.md) | Fournit un ensemble de modèles d’infrastructure conformes et de garde-fou stratégiques qui facilitent l’attestation ISO 27001. |
| [Charge de travail App Service Environment/SQL Database ISO 27001](./iso27001-ase-sql-workload/index.md) | Fournit une infrastructure supplémentaire à l’exemple de blueprint [Services partagés ISO 27001](./iso27001-shared/index.md). |
| [Média](./media/index.md) | Fournit un ensemble de stratégies vous permettant de vous conformer à Media MPAA. |
| [NIST SP 800-53 R4](./nist-sp-800-53-r4.md) | Propose des rails de sécurité pour la conformité à NIST SP 800-53 R4. |
| [NIST SP 800-171 R2](./nist-sp-800-171-r2.md) | Propose des rails de sécurité pour la conformité à NIST SP 800-171 R2. |
| [PCI-DSS v3.2.1](./pci-dss-3.2.1/index.md) | Fournit un ensemble de stratégies pour la mise en conformité à PCI-DSS v3.2.1. |
| [SWIFT CSP-CSCF v2020](./swift-2020/index.md) | Facilite la mise en conformité à SWIFT CSP-CSCF v2020. |
| [Gouvernance de UK OFFICIAL et de UK NHS](./ukofficial/index.md) | Fournit un ensemble de modèles d’infrastructure conformes et de garde-fous stratégiques qui facilitent l’attestation UK OFFICIAL et UK NHS. |
| [Base du framework d’adoption du cloud](./caf-foundation/index.md) | Fournit un ensemble de contrôles destinés à vous aider à gérer vos ressources cloud en adéquation avec le [Framework d’adoption de Microsoft Cloud pour Azure](/azure/architecture/cloud-adoption/governance/journeys/index). |
| [Zone d’accueil de migration du framework d’adoption du cloud](./caf-migrate-landing-zone/index.md) | Fournit un ensemble de contrôles destinés à vous aider à configurer la migration de votre première charge de travail et à gérer vos ressources cloud en adéquation avec le [Framework d’adoption de Microsoft Cloud pour Azure](/azure/architecture/cloud-adoption/migrate/index). |

## <a name="samples-strategy"></a>Stratégie d’exemples

:::image type="complex" source="../media/blueprint-samples-strategy.png" alt-text="Diagramme montrant où des exemples de blueprint conviennent pour la complexité architecturale ou les exigences de conformité." border="false":::
   Décrit un système de coordonnées dans lequel la complexité architecturale est sur l’axe X et les exigences de conformité sur l’axe Y.  À mesure que la complexité architecturale et les exigences de conformité augmentent, adoptez les exemples de blueprint standard du portail désigné dans la région E. Pour les clients qui commencent à utiliser Azure, tirez parti des blueprints Base et Zone d’atterrissage du Cloud Adoption Framework (C A F) désignés par les régions A et B. L’espace restant est attribué aux blueprints personnalisés créés par les clients et les partenaires des régions C, D et F. :::image-end:::

Les blueprints de base et de zone d’accueil de migration du framework d’adoption du cloud partent du principe que le client prépare un abonnement propre existant pour la migration de ressources et de charges de travail locales vers Azure.
(Région A et B dans la figure).  

Vous avez la possibilité d’effectuer une itération sur les exemples de blueprint et de rechercher des modèles de personnalisations qu’un client applique. Vous avez également la possibilité d’aborder de manière proactive des blueprints spécifiques d’un secteur, par exemple, ceux des services financiers et du commerce électronique (extrémité supérieure de la région B). De même, nous envisageons de créer des blueprints répondant à des considérations architecturales complexes, telles que des abonnements multiples, la haute disponibilité, des ressources interrégionales et des clients qui appliquent des contrôles à des abonnements et ressources existants (régions C et D).

Il existe des exemples de blueprints qui répondent à un scénario de client où les exigences de conformité et les complexités architecturales sont élevées (région E dans la figure). La région F de la figure sera utilisée par les clients et partenaires qui tirent parti des exemples de blueprints et les personnalisent pour leurs besoins spécifiques.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [cycle de vie des blueprints](../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md).
- Résoudre les problèmes durant l’affectation d’un blueprint en suivant les étapes de [dépannage général](../troubleshoot/general.md).
