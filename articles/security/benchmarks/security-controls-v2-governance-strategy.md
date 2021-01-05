---
title: Benchmark de sécurité Azure v2 - Gouvernance et stratégie
description: Benchmark de sécurité Azure v2, gouvernance et stratégie
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: af9a86613b3aa29cc01b5ef0daa1f3854940deaf
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96534061"
---
# <a name="security-control-v2-governance-and-strategy"></a>Contrôle de sécurité V2 : Gouvernance et stratégie

La gouvernance et la stratégie fournissent des conseils pour garantir une stratégie de sécurité cohérente et une approche de gouvernance documentée pour guider et soutenir l’assurance sécurité, ce qui inclut l’établissement de rôles et de responsabilités pour les différentes fonctions de sécurité cloud, une stratégie technique unifiée ainsi que des stratégies et des standards connexes.

## <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1 : Définir la stratégie de gestion des ressources et de protection des données

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-1 | 2, 13 | SC, AC |

Veillez à documenter et à communiquer une stratégie claire pour la surveillance et la protection continues des systèmes et des données. Hiérarchisez la découverte, l’évaluation, la protection et la surveillance des données et systèmes critiques de l’entreprise. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

- Norme de classification des données en fonction des risques pour l’entreprise

- Visibilité de l’organisation de sécurité sur les risques et l’inventaire des actifs 

- Approbation de l’organisation de sécurité pour les services Azure en vue de leur utilisation 

- Sécurité des ressources tout au long de leur cycle de vie

- Stratégie de contrôle d’accès requise conformément à la classification des données organisationnelles

- Utilisation des fonctionnalités de protection des données natives et tierces d’Azure

- Exigences de chiffrement des données pour les cas d’utilisation en transit et au repos

- Normes de chiffrement appropriées

Pour plus d’informations, consultez les références suivantes :
- [Recommandation d’architecture de sécurité Azure - Stockage, données et chiffrement](/azure/architecture/framework/security/storage-data-encryption?bc=%2fsecurity%2fcompass%2fbreadcrumb%2ftoc.json&toc=%2fsecurity%2fcompass%2ftoc.json)

- [Notions de base de la sécurité Azure - Sécurité, chiffrement et stockage des données Azure](../fundamentals/encryption-overview.md)

- [Cloud Adoption Framework - Meilleures pratiques en matière de chiffrement et de sécurité des données Azure](../fundamentals/data-encryption-best-practices.md?bc=%2fazure%2fcloud-adoption-framework%2f_bread%2ftoc.json&toc=%2fazure%2fcloud-adoption-framework%2ftoc.json)

- [Benchmark de sécurité Azure - Gestion des ressources](security-controls-v2-asset-management.md)

- [Benchmark de sécurité Azure - Protection des données](security-controls-v2-data-protection.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-2-define-enterprise-segmentation-strategy"></a>GS-2 : Définir la stratégie de segmentation d'entreprise

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-2 | 4, 9, 16 | AC, CA, SC |

Établissez une stratégie à l'échelle de l'entreprise pour segmenter l'accès aux ressources à l'aide d'une combinaison de contrôles d'identité, de réseau, d'application, d'abonnement, de groupe de gestion et autres.

Trouvez le bon équilibre entre la nécessité de séparation sur le plan de la sécurité et la nécessité d'exécuter quotidiennement les systèmes qui doivent communiquer entre eux et accéder aux données.

Veillez à ce que la stratégie de segmentation soit implémentée de manière cohérente pour tous les types de contrôle, y compris pour les modèles d'identité, d'accès et de sécurité du réseau, les modèles d'autorisation/d'accès aux applications et les contrôles des processus humains.

- [Aide relative à la stratégie de segmentation dans Azure (vidéo)](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Aide relative à la stratégie de segmentation dans Azure (document)](/security/compass/governance#enterprise-segmentation-strategy)

- [Aligner la segmentation du réseau avec la stratégie de segmentation d’entreprise](/security/compass/network-security-containment#align-network-segmentation-with-enterprise-segmentation-strategy)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-3-define-security-posture-management-strategy"></a>GS-3 : Définir la stratégie de gestion de la posture de la sécurité

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-3 | 20, 3, 5 | RA, CM, SC |

Mesurez et atténuez en permanence les risques liés à vos ressources individuelles et à l’environnement dans lequel elles sont hébergées. Priorisez les ressources à valeur élevée et les surfaces d’attaque hautement exposées, comme les applications publiées, les points d’entrée et de sortie du réseau, les points de terminaison utilisateur et administrateur, etc.

- [Benchmark de sécurité Azure - Gestion de la posture et des vulnérabilités](security-controls-v2-posture-vulnerability-management.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-4-align-organization-roles-responsibilities-and-accountabilities"></a>GS-4 : Aligner les rôles et les responsabilités de l’organisation

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-4 | N/A | PL, PM |

Veillez à documenter et à communiquer une stratégie claire pour les rôles et les responsabilités de votre organisation de sécurité. Veillez à définir clairement les responsabilités pour les décisions relatives à la sécurité, à former tout le monde au modèle de responsabilité partagée et à former les équipes techniques à la technologie permettant de sécuriser le cloud.

- [Meilleures pratiques pour la sécurité Azure 1 – Personnes : Former les équipes pour le parcours vers la sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#1-people-educate-teams-about-the-cloud-security-journey)

- [Meilleures pratiques pour la sécurité Azure 2 – Personnes : Former les équipes pour les technologies de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#2-people-educate-teams-on-cloud-security-technology)

- [Meilleures pratiques pour la sécurité Azure 3 – Processus : Affecter les responsabilités pour les décisions de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-5-define-network-security-strategy"></a>GS-5 : Définir la stratégie de sécurité réseau

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-5 | 9 | CA, SC |

Établissez une approche de sécurité réseau Azure dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

- Responsabilité centralisée pour la gestion et la sécurité du réseau

- Modèle de segmentation de réseau virtuel aligné avec la stratégie de segmentation de l’entreprise

- Stratégie de correction dans différents scénarios de menaces et d’attaques

- Stratégie de périphérie d’Internet et d’entrée et de sortie

- Stratégie de cloud hybride et d’interconnexion locale

- Artefacts de sécurité réseau à jour (par exemple diagrammes réseau, architecture de réseau de référence)

Pour plus d’informations, consultez les références suivantes :

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Benchmark de sécurité Azure – Sécurité réseau](security-controls-v2-network-security.md)

- [Vue d’ensemble de la sécurité réseau d’Azure](../fundamentals/network-overview.md)

- [Stratégie d’architecture de réseau d’entreprise](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-6-define-identity-and-privileged-access-strategy"></a>GS-6 : Définir une stratégie d’accès privilégié et d’identité

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-6 | 16, 4 | AC, AU, SC |

Établissez une approche d’identité Azure et d’accès privilégié dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

- Un système centralisé d’identité et d’authentification et son interconnexion avec d’autres systèmes d’identité internes et externes

- Méthodes d’authentification fortes dans différents cas d’usage et différentes conditions

- Protection des utilisateurs disposant de privilèges élevés

- Surveillance et gestion des activités anormales des utilisateurs  

- Vérification de l’identité et de l’accès des utilisateurs et processus de rapprochement

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Gestion des identités](security-controls-v2-identity-management.md)

- [Benchmark de sécurité Azure - Accès privilégié](security-controls-v2-privileged-access.md)

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Vue d’ensemble de la sécurité et de la gestion des identités Azure](../fundamentals/identity-management-overview.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7 : Définir la stratégie de journalisation et de réponse aux menaces

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-7 | 19 | IR, AU, RA, SC |

Établissez une stratégie de journalisation et de réponse aux menaces pour détecter et corriger rapidement les menaces tout en répondant aux exigences de conformité. Veillez à ce que les analystes reçoivent des alertes de haute qualité et des expériences homogènes afin qu’ils puissent se concentrer sur les menaces plutôt que sur l’intégration et les étapes manuelles. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

- Rôle et responsabilités de l’organisation d’opérations de sécurité (SecOP) 

- Un processus de réponse aux incidents bien défini, aligné avec NIST ou autre cadre réglementaire du secteur 

- Capture et rétention des journaux pour prendre en charge la détection des menaces, la réponse aux incidents et les besoins de conformité

- Visibilité centralisée des informations de corrélation sur les menaces, avec SIEM, les fonctionnalités Azure natives et d’autres sources 

- Plan de communication et de notification avec vos clients, fournisseurs et les parties publiques pertinentes

- Utilisation de plateformes Azure natives et tierces pour la gestion des incidents, comme la journalisation et la détection des menaces, les investigations et la correction et l’éradication des attaques

- Processus de gestion des incidents et des activités postérieures aux incidents, comme les leçons apprises et la rétention des preuves

Pour plus d’informations, consultez les références suivantes :
- [Benchmark de sécurité Azure - Journalisation et détection des menaces](security-controls-v2-logging-threat-detection.md)

- [Benchmark de sécurité Azure - Réponse aux incidents](security-controls-v2-incident-response.md)

- [Meilleures pratiques pour la sécurité Azure 4 - Processus. Mise à jour des processus de réponse aux incidents pour le cloud](/azure/cloud-adoption-framework/security/security-top-10#3-process-assign-accountability-for-cloud-security-decisions)

- [Guide pour le cadre d’adoption d’Azure, la journalisation et la prise de décision pour les rapports](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Mise à l’échelle, gestion et surveillance d’entreprise Azure](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)

## <a name="gs-8-define-backup-and-recovery-strategy"></a>GS-8 : Définir la stratégie de sauvegarde et de récupération

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP800-53 r4 |
|--|--|--|--|
| GS-8 | 10 | CP |

Établissez une stratégie de sauvegarde et de récupération Azure pour votre organisation. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

- Définitions des objectifs de temps de récupération (RTO) et de point de récupération (RPO) conformément aux objectifs de résilience de votre entreprise

- Conception de la redondance dans vos applications et configuration de l’infrastructure

- Protection de la sauvegarde à l’aide du contrôle d’accès et du chiffrement des données

Pour plus d’informations, consultez les références suivantes :
- [Benchmark de sécurité Azure - Sauvegarde et récupération](security-controls-v2-backup-recovery.md)

- [Azure Well-Architecture Framework - Sauvegarde et récupération d’urgence pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery)

- [Cadre d’adoption d’Azure - Continuité d'activité et reprise d'activité](/azure/cloud-adoption-framework/ready/enterprise-scale/business-continuity-and-disaster-recovery)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Toutes les parties prenantes](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)