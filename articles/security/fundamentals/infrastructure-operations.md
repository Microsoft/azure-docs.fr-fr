---
title: Gestion du réseau de production Azure - Microsoft Azure
description: Cet article décrit la façon dont Microsoft gère et exploite le réseau de production Azure pour sécuriser les centres de données Azure.
services: security
documentationcenter: n
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/30/2019
ms.author: terrylan
ms.openlocfilehash: d41fe409b4a44a4c2af3670d76dd3a83a300feae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68727127"
---
# <a name="management-and-operation-of-the-azure-production-network"></a>Gestion et fonctionnement du réseau de production Azure    
Cet article décrit la façon dont Microsoft gère et exploite le réseau de production Azure pour sécuriser les centres de données Azure.

## <a name="monitor-log-and-report"></a>Superviser, journaliser et créer des rapports

Pour administrer et faire fonctionner le réseau de production Azure, les équipes des opérations Azure et Azure SQL Database doivent coordonner leurs efforts. Ces dernières utilisent plusieurs outils de monitoring des performances système et d’application dans l’environnement. Elles utilisent également les outils appropriés pour superviser les appareils réseau, les serveurs, les services et les processus d’application.

Pour garantir une exécution sécurisée des services qui s’exécutent dans l’environnement Microsoft Azure, les équipes d’opérations implémentent plusieurs niveaux de monitoring, de journalisation et de création de rapports, notamment les actions suivantes :

- Avant tout, Microsoft Monitoring Agent (MMA) collecte des informations de monitoring et de journal de diagnostic à de nombreux emplacements, y compris le contrôleur de structure (FC) et le système d’exploitation (OS) racine, et les écrit dans des fichiers journaux. Il pousse ensuite un sous-ensemble synthétisé des informations sur un compte de stockage Azure préconfiguré. Par ailleurs, le service indépendant de supervision et de diagnostic lit différentes données de journal de diagnostic et de supervision et synthétise les informations. Il écrit ensuite les informations dans un journal intégré. Azure utilise l’extension Azure Security Monitoring (ASM) intégrée, qui est une extension du système de monitoring Azure. Elle présente des composants qui observent, analysent et reportent des événements de sécurité pertinents de différents points de la plateforme.

- La plateforme Azure SQL Database Windows Fabric offre des services d’administration, de déploiement, de développement et de supervision d’opérations pour Azure SQL Database. Elle offre des services distribués de déploiement à plusieurs étapes, une supervision de l’intégrité, des réparations automatiques et une conformité aux versions des services. Elle propose les services suivants :

   - Fonctionnalités de modélisation de service avec un environnement de développement haute fidélité (les clusters de centre de données sont chers et rares).
   - Déploiement en un clic et workflows de mise à niveau pour l’amorçage et la maintenance des services.
   - Rapports sur l’intégrité avec des workflows de réparation automatique pour permettre une réparation spontanée.
   - Supervision, alertes et débogage en temps réel des installations sur les nœuds d’un système distribué.
   - Un ensemble centralisé de données et de métriques de fonctionnement de la racine distribuée permet d’obtenir des insights d’analyse et de service.
   - Outils de fonctionnement du déploiement, de la gestion des changements et du monitoring.
   - La plateforme Azure SQL Database Windows Fabric et des scripts de surveillance s’exécutent en continu et supervisent en temps réel.

Si une anomalie se produit, le processus de réponse aux incidents suivi par l’équipe de triage des incidents Azure est activé. Le personnel compétent du support Azure est notifié pour répondre à l’incident. Le suivi et la résolution des problèmes sont documentés et gérés dans un système centralisé de gestion des tickets. Les métriques de la disponibilité système sont disponibles sous l’accord de confidentialité et sur demande.

## <a name="corporate-network-and-multi-factor-access-to-production"></a>Réseau d’entreprise et accès multifacteur en production
La base d’utilisateurs du réseau d’entreprise inclut le personnel du support d’Azure. Le réseau d’entreprise prend en charge les fonctions internes de l’entreprise et comprend l’accès aux applications internes qui sont utilisées pour le support Azure. Le réseau d’entreprise est à la fois logiquement et physiquement séparé du réseau de production Azure. Le personnel Azure a accès au réseau d’entreprise sur ses ordinateurs portables et ses stations de travail Azure. Tous les utilisateurs doivent avoir un compte Azure Active Directory (Azure AD), avec un nom d’utilisateur et un mot de passe, pour accéder aux ressources du réseau d’entreprise. L’accès au réseau d’entreprise utilise les comptes Azure AD, qui sont émis pour l’ensemble du personnel Microsoft, les sous-traitants et les fournisseurs, et sont gérés par Microsoft Information Technology. Les identificateurs d’utilisateur uniques distinguent le personnel en fonction de leur poste chez Microsoft.

L’accès aux applications Azure internes est contrôlé par une authentification avec AD FS (Active Directory Federation Services). AD FS est un service hébergé par Microsoft Information Technology qui permet d’authentifier les utilisateurs du réseau d’entreprise en appliquant un jeton sécurisé et des revendications utilisateur. AD FS permet aux applications internes d’Azure d’authentifier les utilisateurs sur le domaine Active Directory d’entreprise Microsoft. Pour accéder au réseau de production à partir de l’environnement du réseau d’entreprise, l’utilisateur doit s’authentifier à l’aide de l’authentification multifacteur.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
