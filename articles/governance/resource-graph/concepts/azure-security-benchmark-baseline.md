---
title: Base de référence de sécurité Azure Resource Graph pour le benchmark de sécurité Azure
description: La base de référence de sécurité Azure Resource Graph fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: resource-graph
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: a85fdf1e1e27b1ce5abb0c7890b717d48ba3bd3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89230447"
---
# <a name="azure-resource-graph-security-baseline-for-azure-security-benchmark"></a>Base de référence de sécurité Azure Resource Graph pour le benchmark de sécurité Azure

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure](../../../security/benchmarks/overview.md) à Azure Resource Graph. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Azure Resource Graph. Les **contrôles** non applicables à Azure Resource Graph ont été exclus. Pour voir comment Azure Resource Graph est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité du réseau virtuel Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).



## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Resource Graph permet d’accéder aux types et aux propriétés des ressources sur la base du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Auditez et passez en revue régulièrement l’accès accordé aux principaux de sécurité (utilisateurs, groupes et comptes de service) pour vous assurer que les requêtes retournent les résultats des ressources appropriées.

* [Autorisations dans Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

* [Comment utiliser les révisions d’accès des identités Azure](../../../active-directory/governance/access-reviews-overview.md)


**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès aux données et ressources. Pour utiliser Azure Resource Graph, vous devez également disposer d’un accès approprié aux ressources que vous souhaitez interroger. Cet accès doit être limité en lecture seule et accordé uniquement au personnel requis.

* [Autorisations dans Azure Resource Graph](../overview.md#permissions-in-azure-resource-graph)

* [Comment configurer Azure RBAC](../../../role-based-access-control/role-assignments-rest.md)


**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir toutes les ressources prises en charge au sein de vos abonnements, groupes d’administration et locataires. Vérifiez que vous disposez des autorisations appropriées dans votre locataire et pouvez répertorier tous les abonnements Azure ainsi que les ressources qu’ils contiennent.

* [Guide pratique pour créer des requêtes avec Azure Resource Graph](../first-query-portal.md)

* [Présentation d’Azure RBAC](../../../role-based-access-control/overview.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation. Utilisez Azure Resource Graph pour interroger les ressources Azure approuvées et l’historique des modifications (préversion) pour passer en revue les instantanés et voir ce qui a changé.

* [Explorer les ressources Azure avec Azure Resource Graph](../first-query-portal.md)

* [Obtenir les changements des ressources](../how-to/get-resource-changes.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Resource Graph pour interroger et découvrir les ressources au sein de vos abonnements, groupes d’administration et locataires. Vérifiez que toutes les ressources Azure figurant dans l’environnement sont approuvées.

* [Explorer les ressources Azure avec Azure Resource Graph](../first-query-portal.md)

* [Exemples : Requêtes de démarrage pour Azure Resource Graph](../samples/starter.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../../../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../../../security/benchmarks/security-baselines-overview.md)
