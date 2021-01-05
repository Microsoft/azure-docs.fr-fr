---
title: Vue d’ensemble d’Azure Security Benchmark V2
description: Vue d’ensemble d’Azure Security Benchmark V2
author: msmbaldwin
manager: rkarlin
ms.service: security
ms.topic: conceptual
ms.date: 09/11/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 3bc97a932e622838315f717cc7b036a559787805
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97369155"
---
# <a name="overview-of-the-azure-security-benchmark-v2"></a>Vue d’ensemble d’Azure Security Benchmark (V2)

Azure Security Benchmark (ASB) prescrit les bonnes pratiques et des recommandations pour améliorer la sécurité des charges de travail, des données et des services dans Azure.

Ce benchmark fait partie d’un ensemble de principes de sécurité globaux qui incluent aussi :

- **Cloud Adoption Framework** – Conseils de sécurité sur notamment la [stratégie](/azure/cloud-adoption-framework/strategy/define-security-strategy), les [rôles et responsabilités](/azure/cloud-adoption-framework/organize/cloud-security), les [10 premières bonnes pratiques de sécurité Azure](/azure/cloud-adoption-framework/get-started/security#step-1-establish-essential-security-practices) et l’[implémentation de référence](/azure/cloud-adoption-framework/ready/enterprise-scale/).
- **Azure Well-Architected Framework** – Conseils sur la [sécurisation de vos charges de travail](/assessments/?mode=pre-assessment&session=local) dans Azure.
- **Bonnes pratiques de sécurité Microsoft** – [Recommandations](/security/compass/microsoft-security-compass-introduction) avec des exemples sur Azure.

 Azure Security Benchmark se concentre sur les zones de contrôle axées sur le cloud. Ces contrôles sont cohérents avec des points de benchmarks de sécurité bien connus, tels que ceux décrits dans la version 7.1 des contrôles CIS (Center for Internet Security) et dans la publication spéciale SP 800-53 du NIST (National Institute of Standards and Technology).
Les contrôles suivants sont inclus dans Azure Security Benchmark :

| Domaines de contrôle ASB | Description 
|--|--|
| [Sécurité&nbsp;réseau&nbsp;(NS)](security-controls-v2-network-security.md) | La sécurité réseau recouvre les contrôles destinés à sécuriser et protéger les réseaux Azure, ce qui inclut la sécurisation des réseaux virtuels, l’établissement de connexions privées, la prévention et l’atténuation des attaques externes et la sécurisation de DNS. |
| [Gestion&nbsp;des identités&nbsp;(IM)](security-controls-v2-identity-management.md) | La gestion des identités recouvre les contrôles destinés à établir une identité sécurisée et des contrôles d’accès avec Azure Active Directory, ce qui inclut l’utilisation de l’authentification unique, d’authentifications fortes, d’identités managées (et les principes de service) pour les applications, l’accès conditionnel et la supervision des anomalies de compte. |
| [Accès&nbsp;privilégié&nbsp;(PA)](security-controls-v2-privileged-access.md) | L’accès privilégié recouvre les contrôles destinés à protéger l’accès privilégié à votre locataire et vos ressources Azure, avec notamment toute une gamme de contrôles visant à protéger votre modèle d’administration, vos comptes d’administration et vos stations de travail à accès privilégié contre les risques délibérés et involontaires. |
| [Protection&nbsp;des données&nbsp;(DP)](security-controls-v2-data-protection.md) | La protection des données recouvre le contrôle de la protection des données au repos, en transit et par le biais de mécanismes d’accès autorisés, ce qui inclut la découverte, la classification, la protection et la supervision des ressources de données sensibles en utilisant le contrôle d’accès, le chiffrement et la journalisation dans Azure. |
| [Gestion&nbsp;des ressources&nbsp;(AM)](security-controls-v2-asset-management.md) | La gestion des ressources recouvre les contrôles destinés à assurer la visibilité et la gouvernance de la sécurité sur les ressources Azure, ce qui inclut des recommandations sur les autorisations pour le personnel de sécurité, l’accès sécurisé à l’inventaire des ressources et la gestion des approbations pour les services et les ressources (inventaire, suivi et correction). |
| [Journalisation&nbsp;et&nbsp;détection&nbsp;des menaces (LT)](security-controls-v2-logging-threat-detection.md) | La journalisation et la détection des menaces recouvrent les contrôles destinés à détecter les menaces dans Azure et à activer, collecter et stocker les journaux d’audit pour les services Azure, ce qui inclut l’activation des processus de détection, d’investigation et de correction avec des contrôles visant à générer des alertes de haute qualité avec la détection native des menaces dans les services Azure. Cela inclut également la collecte des journaux avec Azure Monitor, la centralisation de l’analyse de la sécurité avec Azure Sentinel, la synchronisation de l’heure et la conservation des journaux. |
| [Réponse&nbsp;aux incidents&nbsp;(IR)](security-controls-v2-incident-response.md) | La réponse aux incidents recouvre les contrôles du cycle de vie de la réponse aux incidents – préparation, détection et analyse, autonomie et activités post-incident, ce qui inclut l’utilisation de services Azure comme Azure Security Center et Sentinel pour automatiser le processus de réponse aux incidents. |
| [Gestion&nbsp;des&nbsp;postures&nbsp;et des vulnérabilités&nbsp;(PV)](security-controls-v2-posture-vulnerability-management.md) | La gestion des postures et des vulnérabilités porte essentiellement sur les contrôles destinés à évaluer et améliorer la posture de sécurité d’Azure, ce qui inclut l’analyse des vulnérabilités, les tests et la correction des intrusions, ainsi que le suivi de la configuration de la sécurité, la création de rapports et la correction dans les ressources Azure. |
| [Sécurité&nbsp;du point de terminaison&nbsp;(ES)](security-controls-v2-endpoint-security.md) | La sécurité des points de terminaison recouvre les contrôles de détection et d’arrêt des menaces, notamment l’utilisation de la technologie EDR (Endpoint Detection and Response) et du service anti-programme malveillant pour les points de terminaison dans les environnements Azure. |
| [Sauvegarde&nbsp;et&nbsp;récupération&nbsp;(BR)](security-controls-v2-backup-recovery.md) | La sauvegarde et la récupération recouvrent les contrôles destinés à s’assurer que les sauvegardes de données et de configurations aux différents niveaux de service sont effectuées, validées et protégées. |
| [Gouvernance&nbsp;et&nbsp;stratégie&nbsp;(GS)](security-controls-v2-governance-strategy.md) | La gouvernance et la stratégie fournissent des conseils pour garantir une stratégie de sécurité cohérente et une approche de gouvernance documentée pour guider et soutenir l’assurance sécurité, ce qui inclut l’établissement de rôles et de responsabilités pour les différentes fonctions de sécurité cloud, une stratégie technique unifiée ainsi que des stratégies et des standards connexes. |

## <a name="azure-security-benchmark-recommendations"></a>Recommandations relatives au benchmark de sécurité Azure

Chaque recommandation comprend les informations suivantes :

- **Identifiant Azure** : Identifiant de benchmark de sécurité Azure correspondant à la recommandation.
- **ID des contrôles CIS v7.1** : Contrôle(s) CIS v7.1 correspondant à cette recommandation.
- **ID NIST SP 800-53 r4** : Contrôle(s) NIST SP 800-53 r4 (modéré(s)) correspondant à cette recommandation.
- **Détails** : Raisons ayant motivé la recommandation et liens vers des conseils d’implémentation de celle-ci. Si la recommandation est prise en charge par Azure Security Center, l’information est également indiquée.
- **Responsabilité** : Client, fournisseur de service ou les deux en chargé de l’implémentation de cette recommandation. Les responsabilités en matière de sécurité sont partagées dans le cloud public. Certains contrôles de sécurité étant uniquement accessibles au fournisseur de services cloud, c’est lui qui en est responsable. Ces observations sont d’ordre général. Pour certains services individuels, la responsabilité sera différente de ce qui est indiqué dans le benchmark de sécurité Azure. Ces différences sont décrites dans les recommandations de référence pour les services individuels.
- **Parties prenantes de la sécurité des clients** : [Fonctions de sécurité](/azure/cloud-adoption-framework/organize/cloud-security#security-functions) dans l’organisation du client qui peuvent être comptables, responsables ou consultées pour le contrôle concerné. Cela peut varier d’une organisation à une autre en fonction de la structure d’organisation de la sécurité de l’entreprise ainsi que des rôles et des responsabilités configurés par rapport à la sécurité Azure.

> [!NOTE]
> La correspondance des contrôles entre ASB et les benchmarks du secteur (tels que NIST et CIS) indique uniquement qu’une fonctionnalité Azure spécifique peut être utilisée pour répondre en tout ou partie à une spécification de contrôle définie dans NIST ou CIS. Vous devez avoir conscience que cette implémentation ne se traduit pas nécessairement par une conformité totale du contrôle correspondant dans CIS ou NIST.

Vos commentaires détaillés et votre participation active à la réussite du benchmark de sécurité Azure sont les bienvenus. Si vous souhaitez adresser un commentaire directement à l’équipe Azure Security Benchmark, remplissez le formulaire disponible à l’adresse https://aka.ms/AzSecBenchmark

## <a name="download"></a>Télécharger

Vous pouvez télécharger le benchmark de sécurité Azure au [format de feuille de calcul](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Security%20Benchmark).

## <a name="next-steps"></a>Étapes suivantes 
- Reportez-vous au premier contrôle de sécurité : [Sécurité du réseau](security-control-network-security.md)
- Lire l’[introduction aux benchmarks de sécurité Azure](introduction.md)
- Découvrir les [Concepts de base de la sécurité Azure](../fundamentals/index.yml)