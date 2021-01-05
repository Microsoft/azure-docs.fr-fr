---
title: Introduction au benchmark de sécurité Azure
description: Introduction au benchmark de sécurité
author: msmbaldwin
manager: rkarlin
ms.service: security
ms.topic: conceptual
ms.date: 12/16/2019
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: f3b492534bedde50d2413b88fa38cd6325ed5df7
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91328256"
---
# <a name="azure-security-benchmark-introduction"></a>Introduction aux benchmarks de sécurité Azure

De nouvelles fonctionnalités et de nouveaux services sont publiés quotidiennement dans Azure, les développeurs publient rapidement de nouvelles applications cloud reposant sur ces services, et les attaquants cherchent toujours de nouvelles façons d’exploiter des ressources mal configurées. Le cloud évolue rapidement, les développeurs évoluent rapidement et les attaquants sont toujours sur le qui-vive. Comment vous assurer que vos déploiements cloud sont sécurisés ? En quoi les pratiques de sécurité pour les systèmes cloud diffèrent-elles des systèmes locaux ? Comment surveiller la cohérence entre de nombreuses équipes de développement indépendantes ?

Microsoft a découvert que l’utilisation de *benchmarks de sécurité* peut vous aider à sécuriser rapidement les déploiements dans le cloud. Les recommandations de référence de votre fournisseur de services cloud vous offrent un point de départ pour sélectionner des paramètres de configuration de sécurité spécifiques dans votre environnement et vous permettent de réduire rapidement les risques pour votre organisation.

Le benchmark de sécurité Azure comprend un ensemble de recommandations de sécurité à fort impact qui peuvent vous permettre de sécuriser les services que vous utilisez dans Azure :

- **Contrôles de sécurité** : Ces recommandations s’appliquent généralement à votre locataire Azure et à vos services Azure. Chaque recommandation identifie une liste des parties prenantes qui sont généralement impliquées dans la planification, l’approbation ou l’implémentation du benchmark. 
- **Lignes de base de service** : Elles appliquent les contrôles à des services Azure individuels pour fournir des recommandations sur la configuration de la sécurité de ces services.

## <a name="implement-the-azure-security-benchmark"></a>Implémenter le benchmark de sécurité Azure
- **Planifiez** votre implémentation de test de sécurité Azure en consultant la [documentation](overview.md) pour les contrôles d’entreprise et les lignes de base spécifiques aux services pour planifier votre infrastructure de contrôle et la façon dont elle est s’applique à des contrôles tels que CIS (Controls v7.1) et le framework NIST (SP800-53).
- **Analysez** votre conformité avec l’état du benchmark de sécurité Azure (et d’autres jeux de contrôles) à l’aide du [tableau de bord de conformité](../../security-center/security-center-compliance-dashboard.md) d’Azure Security Center.
- **Établissez des garde-fous** pour automatiser les configurations sécurisées et appliquer la conformité avec le benchmark de sécurité Azure (et les autres exigences de votre organisation) avec Azure Blueprints et Azure Policy.
 
Notez que le benchmark de sécurité Azure v2 est aligné avec les [Bonnes pratiques de sécurité Microsoft](/security/compass/microsoft-security-compass-introduction) (anciennement Azure Security Compass) afin que le benchmark de sécurité Azure offre une vue consolidée unique des recommandations de sécurité de Microsoft Azure.

## <a name="common-use-cases"></a>Cas d’usage courants

Le benchmark de sécurité Azure est fréquemment utilisé pour répondre à ces défis courants pour les clients ou les partenaires de service qui :
- Découvrent Azure et recherchent les meilleures pratiques de sécurité pour garantir un déploiement sécurisé.
- Améliorent la sécurité de déploiements Azure existants pour prioriser les risques et atténuations les plus importants.
- Approuvent des services Azure pour une utilisation technologique et professionnelle pour répondre à des instructions de sécurité spécifiques.
- Répondent aux exigences réglementaires pour les clients qui proviennent de secteurs gouvernementaux ou hautement réglementés, tels que les fournisseurs de services financiers et de santé (ou les fournisseurs de services qui ont besoin de créer des systèmes pour ces clients). Ces clients doivent s’assurer que leur configuration Azure répond aux fonctionnalités de sécurité spécifiées dans un cadre sectoriel, comme CIS, NIST ou PCI. Le benchmark de sécurité Azure offre une approche efficace avec les contrôles déjà prémappés à ces tests d’évaluation du secteur.

## <a name="terminology"></a>Terminologie

Les termes « contrôle », « benchmark » et « base de référence » sont souvent utilisés dans la documentation du benchmark de sécurité Azure ; il est important de comprendre comment Azure les utilise.


| Terme | Description | Exemple |
|--|--|--|
| Control | Un contrôle est une description de haut niveau d’une fonctionnalité ou d’une activité à traiter, qui n’est pas propre à une technologie ou à une implémentation. | La protection des données constitue l’un des contrôles de sécurité. Ce contrôle contient des actions spécifiques qui doivent être réglées pour garantir la protection des données. |
| Référence | Un benchmark contient des recommandations de sécurité pour une technologie spécifique, comme Azure. Les recommandations sont classées en fonction du contrôle auquel elles appartiennent. | Le benchmark de sécurité Azure comprend les recommandations de sécurité propres à la plateforme Azure |
| Ligne de base | Une ligne de base est l’implémentation du benchmark d’évaluation sur le service Azure individuel. Chaque organisation choisit la recommandation de benchmark et les configurations correspondantes qui sont nécessaires dans l’étendue de l’implémentation Azure. | La société Contoso cherche à activer les fonctionnalités de sécurité d’Azure SQL en suivant la configuration recommandée dans la ligne de base de sécurité Azure SQL.

Nous attendons vos commentaires sur le benchmark de sécurité Azure ! Nous vous invitons à les formuler dans la zone de commentaires ci-dessous. Si vous préférez partager votre avis de manière plus privée avec l’équipe de benchmark de sécurité Azure, vous pouvez remplir le formulaire à l’adresse https://aka.ms/AzSecBenchmark.
