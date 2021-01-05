---
title: Intégrité de l’infrastructure Azure
description: Découvrez l’intégrité de l’infrastructure Azure et les mesures prises par Microsoft pour la sécuriser, telles que les analyses antivirus sur les builds de composants logiciels.
services: security
documentationcenter: na
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
ms.date: 07/06/2018
ms.author: terrylan
ms.openlocfilehash: b0df14dfcf66f53e01989a27a95afa5e19fcb2fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543637"
---
# <a name="azure-infrastructure-integrity"></a>Intégrité de l’infrastructure Azure

## <a name="software-installation"></a>Installation de logiciels
Tous les composants de la pile de logiciels installés dans l’environnement Azure sont conçus de manière personnalisée conformément au processus Microsoft Security Development Lifecycle (SDL). Tous les composants logiciels, notamment les images de système d’exploitation et SQL Database, sont déployés dans le cadre du processus de gestion des modifications et des versions. Le système d’exploitation qui s’exécute sur tous les nœuds est une version personnalisée de Windows Server 2008 ou Windows Server 2012. La version exacte est choisie par le contrôleur de structure en fonction du rôle qu’il a l’intention d’attribuer au système d’exploitation. En outre, le système d’exploitation hôte n’autorise pas l’installation de composants logiciels non autorisés.

Certains composants Azure sont déployés en tant que clients Azure s’exécutant sur une machine virtuelle invitée sur un système d’exploitation invité.

## <a name="virus-scans-on-builds"></a>Analyses antivirus sur les builds
Les builds de composants logiciels Azure (notamment le système d’exploitation) doivent subir une analyse antivirus à l’aide de l’outil de protection antivirus Endpoint Protection. Chaque analyse antivirus crée un journal dans le répertoire de build associé, détaillant ce qui a été analysé et les résultats de l’analyse. L’analyse antivirus fait partie du code source de build pour chaque composant dans Azure. Le code ne passe pas en production sans avoir subi une analyse antivirus n’ayant révélé aucun problème. Si des problèmes sont signalés, le build est figé et transmis aux équipes de sécurité au sein de Microsoft Security afin d’identifier où le code « non autorisé » a pénétré dans le build.

## <a name="closed-and-locked-environment"></a>Environnement fermé et verrouillé
Par défaut, aucun compte d’utilisateur n’est créé sur les nœuds d’infrastructure Azure et les machines virtuelles invitées. Les comptes d’administrateur Windows par défaut sont également désactivés. Les administrateurs d’Azure Live Support peuvent, avec une authentification appropriée, se connecter à ces machines et administrer le réseau de production Azure à des fins de réparation d’urgence.

## <a name="azure-sql-database-authentication"></a>Authentification Azure SQL Database
Comme avec toute implémentation de SQL Server, la gestion des comptes utilisateur doit être contrôlée étroitement. Azure SQL Database prend uniquement en charge l’authentification SQL Server. Des mots de passe à la sécurité élevée et configurés avec des droits spécifiques doivent aussi être utilisés afin de compléter le modèle de sécurité des données du client.

## <a name="acls-and-firewalls-between-the-microsoft-corporate-network-and-an-azure-cluster"></a>Listes ACL et pare-feux entre le réseau d’entreprise Microsoft et un cluster Azure
Les listes de contrôle d'accès (ACL) et les pare-feux entre la plateforme de service et le réseau d’entreprise Microsoft protègent les instances SQL Database contre tout accès interne non autorisé. En outre, seuls les utilisateurs compris dans les plages d’adresses IP du réseau d’entreprise Microsoft peuvent accéder au point de terminaison de gestion de plateforme Windows Fabric.

## <a name="acls-and-firewalls-between-nodes-in-a-sql-database-cluster"></a>Listes ACL et pare-feux entre les nœuds dans un cluster SQL Database
En guise de protection supplémentaire, dans le cadre de la stratégie de défense en profondeur, des listes ACL et un pare-feu ont été implémentés entre les nœuds d’un cluster SQL Database. Toutes les communications au sein du cluster de plateforme Windows Fabric, ainsi que tout le code en cours d’exécution, sont approuvés.

## <a name="custom-monitoring-agents"></a>Exemple de surveillance personnalisée
SQL Database utilise des agents de gestion (MA) personnalisés appelés « agents de surveillance » afin de surveiller l’intégrité du cluster SQL Database.

## <a name="web-protocols"></a>Protocoles web

### <a name="role-instance-monitoring-and-restart"></a>Surveillance et redémarrage des instances de rôle
Azure garantit que tous les rôles en cours d’exécution (rôles de travail de traitement backend ou rôles web orientés vers Internet) déployés sont soumis à une surveillance de l’intégrité soutenue afin de garantir qu’ils délivrent efficacement les services dans lesquels ils ont été provisionnés. Dans le cas où un rôle perdrait son intégrité, suite à une défaillance critique de l’application hébergée ou à un problème de configuration sous-jacent au sein de l’instance de rôle proprement dite, le contrôleur de structure détectera le problème au sein de l’instance de rôle et lancera un état correctif.

### <a name="compute-connectivity"></a>Connectivité de calcul
Azure garantit que l’application/service déployé(e) est accessible par le biais de protocoles web standards. Les instances virtuelles de rôles web orientés vers Internet disposeront d’une connectivité Internet externe et seront accessibles directement par les utilisateurs web. Afin de protéger la sensibilité et l’intégrité des opérations effectuées par les rôles de travail pour le compte des instances virtuelles de rôles web accessibles publiquement, les instances virtuelles de rôles de travail de traitement backend disposent d’une connectivité Internet externe mais ne sont pas accessibles directement par un utilisateur web externe.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Protection des données client Azure](protection-customer-data.md)
