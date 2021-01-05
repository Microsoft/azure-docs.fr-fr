---
title: Azure Security Benchmark V2 – Protection des données
description: Protection des données Azure Security Benchmark V2
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 687c344aefc70729c85fb37d615ec0a272ff4fde
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368866"
---
# <a name="security-control-v2-data-protection"></a>Contrôle de sécurité V2 : Protection des données

La protection des données recouvre le contrôle de protection des données au repos, en transit et via des mécanismes d’accès autorisés. Cela comprend la découverte, la classification, la protection et la supervision des ressources de données sensibles via le contrôle d’accès, le chiffrement et la journalisation dans Azure.

## <a name="dp-1-discovery-classify-and-label-sensitive-data"></a>DP-1 : Découvrir, classifier et étiqueter des données sensibles

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| DP-1 | 13.1, 14.5, 14.7 | SC-28 |

Découvrez, classifiez et étiquetez vos données sensibles de façon à concevoir les contrôles appropriés pour faire en sorte que le stockage, le traitement et la transmission sécurisées des informations sensibles soient assurés par les systèmes technologiques de l’organisation. 

Utilisez Azure Information Protection (et son outil d’analyse associé) pour les informations sensibles présentes dans les documents Office au niveau d’Azure, de l’environnement local, d’Office 365 et ailleurs. 

Vous pouvez utiliser Azure SQL Information Protection pour faciliter la classification et l’étiquetage des informations stockées dans les bases de données Azure SQL Database.

- [Étiqueter des informations sensibles à l’aide d’Azure Information Protection](/azure/information-protection/what-is-information-protection) 

- [Guide pratique pour implémenter la recherche de données Azure SQL](../../azure-sql/database/data-discovery-and-classification-overview.md)

**Responsabilité** : Partagé

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)  

- [Sécurité des données](/azure/cloud-adoption-framework/organize/cloud-security-data-security) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-2-protect-sensitive-data"></a>DP-2 : Protection des données sensibles

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| DP-2 | 13.2, 2.10 | SC-7, AC-4 |

Protégez les données sensibles en limitant l’accès avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC), des contrôles d’accès basés sur le réseau et des contrôles spécifiques dans les services Azure (comme le chiffrement dans les bases de données SQL et autres). 

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise. La stratégie de segmentation de l’entreprise doit aussi être informée de l’emplacement des systèmes et données sensibles ou vitaux de l’entreprise.

Pour la plateforme sous-jacente gérée par Microsoft, Microsoft traite tout le contenu client en tant que contenu sensible, et assure une protection contre la perte et l’exposition des données client. Pour assurer la sécurité des données des clients dans Azure, Microsoft a implémenté certains contrôles et fonctionnalités de protection des données par défaut.

- [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md)

- [Présentation de la protection des données client dans Azure](../fundamentals/protection-customer-data.md)

**Responsabilité** : Partagé

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Sécurité des données](/azure/cloud-adoption-framework/organize/cloud-security-data-security)

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-3-monitor-for-unauthorized-transfer-of-sensitive-data"></a>DP-3 : Détection des transferts non autorisés de données sensibles

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| DP-3 | 13.3 | AC-4, SI-4 |

Détectez les transferts non autorisés de données vers des emplacements situés hors du champ de visibilité ou de contrôle de l’entreprise. Cela implique généralement de surveiller des activités anormales (transferts volumineux ou inhabituels) qui pourraient être le signe d’une exfiltration de données non autorisée. 

Dans Stockage Azure et Azure SQL, la protection avancée contre les menaces (ou Advanced Threat Protection, ATP) peut générer une alerte en cas de transfert d’informations anormal qui pourrait être le signe d’un transfert non autorisé d’informations sensibles. 

Azure Information Protection (AIP) offre des fonctionnalités de supervision pour les informations qui ont été classifiées et étiquetées. 

Si cela est nécessaire pour la conformité de la protection contre la perte de données (DLP), vous pouvez utiliser une solution DLP basée sur l’hôte pour appliquer des contrôles de détection et/ou de prévention de façon à empêcher l’exfiltration de données.

- [Activer ATP pour Azure SQL](../../azure-sql/database/threat-detection-overview.md)

- [Activer ATP pour Stockage Azure](../../storage/common/azure-defender-storage-configure.md?tabs=azure-security-center)

**Responsabilité** : Partagé

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Opérations de sécurité](/azure/cloud-adoption-framework/organize/cloud-security) 

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

## <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4 : Chiffrement des informations sensibles en transit

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| DP-4 | 14.4 | SC-8 |

En complément des contrôles d’accès, les données en transit doivent être protégées contre les attaques « hors bande » (par exemple, la capture de trafic) à l’aide du chiffrement pour empêcher les attaquants de lire ou modifier facilement les données. 

C’est certes facultatif pour le trafic sur les réseaux privés, mais essentiel pour le trafic sur les réseaux externes et publics. Pour le trafic HTTP, vérifiez que les clients se connectant à vos ressources Azure peuvent négocier TLS v1.2 ou une version ultérieure. Pour la gestion à distance, utilisez SSH (pour Linux) ou RDP/TLS (pour Windows) plutôt qu’un protocole non chiffré. Les versions et protocoles SSL, TLS et SSH rendus obsolètes et les chiffrements faibles doivent être désactivés.  

Par défaut, Azure assure le chiffrement des données en transit entre les centres de données Azure. 

- [Présentation du chiffrement en transit avec Azure](../fundamentals/encryption-overview.md#encryption-of-data-in-transit)

- [Informations sur la sécurité TLS](/security/engineering/solving-tls1-problem)

- [Double chiffrement pour les données Azure en transit](../fundamentals/double-encryption.md#data-in-transit)

**Responsabilité** : Partagé

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops) 

- [Sécurité des données](/azure/cloud-adoption-framework/organize/cloud-security-data-security)

## <a name="dp-5-encrypt-sensitive-data-at-rest"></a>DP-5 : Chiffrement des données sensibles au repos

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| DP-5 | 14.8 | SC-28, SC-12 |

En complément des contrôles d’accès, les données au repos doivent être protégées contre les attaques « hors bande » (telles que l’accès au stockage sous-jacent) à l’aide du chiffrement. Cela vise à empêcher que les attaquants puissent facilement lire ou modifier les données. 

Par défaut, Azure assure un chiffrement des données au repos. Pour les données hautement sensibles, vous avez le choix entre plusieurs options pour implémenter un chiffrement supplémentaire au repos sur toutes les ressources Azure, le cas échéant. Azure gère vos clés de chiffrement par défaut, mais Azure met à votre disposition des options pour gérer vos propres clés (clés gérées par le client) pour certains services Azure.

- [Présentation du chiffrement au repos dans Azure](../fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services)

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](../../storage/common/customer-managed-keys-configure-key-vault.md)

- [Modèle de chiffrement et table de gestion des clés](../fundamentals/encryption-models.md)

- [Double chiffrement des données au repos dans Azure](../fundamentals/double-encryption.md#data-at-rest)

**Responsabilité** : Partagé

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-infrastructure-endpoint)

- [Sécurité des applications et DevOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Sécurité des données](/azure/cloud-adoption-framework/organize/cloud-security-data-security)