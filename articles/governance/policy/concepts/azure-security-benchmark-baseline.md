---
title: Base de référence de sécurité Azure Policy pour le benchmark de sécurité Azure
description: La base de référence de sécurité Azure Policy fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: azure-policy
ms.topic: conceptual
ms.date: 07/02/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 8b18da5e41e235b560918ad26117a0162078862c
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91540980"
---
# <a name="azure-policy-security-baseline-for-azure-security-benchmark"></a>Base de référence de sécurité Azure Policy pour le benchmark de sécurité Azure

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure](../../../security/benchmarks/overview.md) à Azure Policy. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par **domaines de conformité** et par **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Azure Policy. **Les contrôles** non applicables à Azure Policy ont été exclus. Pour voir comment Azure Policy est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Azure Policy](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Pour un mappage des contrôles du benchmark de sécurité Azure aux définitions de stratégie intégrées via l’initiative intégrée, consultez [Conformité réglementaire : benchmark de sécurité Azure](../samples/azure-security-benchmark.md).

Azure Policy utilise le terme de _Propriété_ à la place de _Responsabilité_. Pour des détails sur la _propriété_, consultez [Définition de stratégie Azure Policy](./definition-structure.md#type) et [Responsabilité partagée dans le cloud](../../../security/fundamentals/shared-responsibility.md).


## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Contrôle de sécurité : Journalisation et supervision](../../../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide** : Azure Policy utilise les journaux d’activité, qui sont automatiquement activés, pour inclure la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

* [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../../../azure-monitor/platform/diagnostic-settings.md)

* [Présentation de la journalisation et des différents types de journaux dans Azure](../../../azure-monitor/platform/platform-logs-overview.md)


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../../../security/benchmarks/security-control-identity-access-control.md).*

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration. 

Vous pouvez également activer une solution JIT (Juste-à-temps)/JEA (Just-Enough-Access) à l’aide de rôles [Azure AD Privileged Identity Management](../../../active-directory/privileged-identity-management/pim-configure.md) ou d’[Azure Resource Manager](../../../azure-resource-manager/management/overview.md).


**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Aide** : Utilisez des stations de travail avec accès privilégié avec l’authentification multifacteur (MFA) configurée pour la connexion aux ressources Azure et leur configuration.

* [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

* [Guide pratique pour activer l’authentification MFA dans Azure](../../../active-directory/authentication/howto-mfa-getstarted.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d’informations, consultez [Contrôle de sécurité : Protection des données](../../../security/benchmarks/security-control-data-protection.md).*

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide** : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès à Azure Policy.

* [Autorisations Azure RBAC dans Azure Policy](../overview.md#azure-rbac-permissions-in-azure-policy)

* [Comment configurer Azure RBAC](../../../role-based-access-control/role-assignments-portal.md)


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide** : Utilisez Azure Monitor avec les journaux d’activités pour créer des alertes lorsque des modifications sont apportées à Azure Policy.

* [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../../../azure-monitor/platform/alerts-activity-log.md)


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../../../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories. Utilisez l’effet _modifier_ d’Azure Policy pour signaler et appliquer la conformité et la gouvernance cohérente des balises.

* [Tutoriel : Créer et gérer les stratégies](../tutorials/create-and-manage.md)

* [Tutoriel : Gérer la gouvernance des balises](../tutorials/govern-tags.md)


**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Aide** : créer un inventaire des définitions de stratégie approuvées et des affectations de stratégie conformément aux besoins de votre organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

* [Guide pratique pour configurer et gérer Azure Policy](../tutorials/create-and-manage.md)


**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../../../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../../../security/benchmarks/security-baselines-overview.md)
