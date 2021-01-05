---
title: Exemples de contrôles de blueprint PCI-DSS v3.2.1
description: Mappage des contrôles de l'exemple de blueprint Payment Card Industry Data Security Standard v3.2.1 vers Azure Policy et Azure RBAC.
ms.date: 08/19/2020
ms.topic: sample
ms.openlocfilehash: c634e8cc64f6376251e2cd07725c5c5a643b27c7
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91931425"
---
# <a name="control-mapping-of-the-pci-dss-v321-blueprint-sample"></a>Mappage des contrôles de l’exemple de blueprint PCI-DSS v3.2.1

L’article suivant décrit en détail le mappage de l’exemple de blueprint PCI-DSS v3.2.1 Azure Blueprints aux contrôles PCI-DSS v3.2.1. Pour plus d’informations sur les contrôles, consultez [PCI-DSS v3.2.1](https://www.pcisecuritystandards.org/documents/PCI_DSS_v3-2-1.pdf).

Les mappages suivants concernent les contrôles **PCI-DSS v3.2.1:2018**. Utilisez le volet de navigation de droite pour accéder directement à la correspondance d’un contrôle spécifique. De nombreux contrôles mappés sont implémentés avec une initiative [Azure Policy](../../../policy/overview.md). Pour examiner l’initiative complète, ouvrez **Stratégie** dans le portail Azure et sélectionnez la page **Définitions**. Ensuite, recherchez et sélectionnez l’initiative de stratégie intégrée **\[Préversion\] Auditer les contrôles PCI v3.2.1:2018 et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit**.

> [!IMPORTANT]
> Chaque contrôle ci-dessous est associé à une ou plusieurs définitions [Azure Policy](../../../policy/overview.md). Ces stratégies peuvent vous aider à [évaluer la conformité](../../../policy/how-to/get-compliance-data.md) avec le contrôle ; toutefois, il n’existe pas souvent de correspondance un-à-un ou parfaite entre un contrôle et une ou plusieurs stratégies. Ainsi, la **conformité** dans Azure Policy fait uniquement référence aux stratégies elles-mêmes ; cela ne garantit pas que vous êtes entièrement conforme à toutes les exigences d’un contrôle. En outre, la norme de conformité comprend des contrôles qui ne sont traités par aucune définition Azure Policy pour l’instant. Par conséquent, la conformité dans Azure Policy n’est qu’une vue partielle de l’état de conformité global. Les associations entre les contrôles et les définitions Azure Policy pour cet exemple de blueprint de conformité peuvent changer au fil du temps. Pour afficher l’historique des changements, consultez l’[historique des validations GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/pci-dss-3.2.1/control-mapping.md).

## <a name="132-and-134-boundary-protection"></a>1.3.2 et 1.3.4 Protection de la limite

Ce blueprint vous aide à gérer et à contrôler les réseaux en affectant une définition [Azure Policy](../../../policy/overview.md) qui supervise les groupes de sécurité réseau associés à des règles permissives. Les règles trop permissives sont susceptibles d’autoriser des accès réseau involontaires et doivent être réexaminées. Ce blueprint attribue une définition Azure Policy qui supervisent les points de terminaison, applications et comptes de stockage non protégés. Les points de terminaison et les applications qui ne sont pas protégés par un pare-feu, de même que les comptes de stockage avec un accès illimité, peuvent permettre un accès involontaire aux informations contenues dans le système d’information.

- Auditer l'accès réseau non restreint aux comptes de stockage
- L'accès via un point de terminaison accessible sur Internet doit être limité

## <a name="34a-41-41g-41h-and-653-cryptographic-protection"></a>3.4.a, 4.1, 4.1.g, 4.1.h et 6.5.3 Protection par chiffrement

Ce blueprint vous aide à appliquer votre stratégie avec utilisation des contrôles de chiffrement en affectant des définitions [Azure Policy](../../../policy/overview.md) qui mettent en œuvre des contrôles de chiffrement spécifiques et détectent l’utilisation de paramètres de chiffrement faibles. Le fait de savoir où vos ressources Azure peuvent avoir des configurations de chiffrement non optimales peut vous aider à prendre des mesures correctives visant à vérifier que les ressources sont configurées conformément à votre stratégie de sécurité des informations. Plus précisément, les stratégies affectées par ce blueprint exigent chiffrement transparent des données sur les bases de données SQL ; vérifient l’absence de chiffrement sur les comptes de stockage et les variables de compte Automation. Il existe également des stratégies qui auditent les connexions non sécurisées aux comptes de stockage, applications de fonctions, applications web, applications API et cache Redis, et auditent les communications Service Fabric non chiffrées.

- Function App ne doit pas être accessible via HTTPS
- L'application web ne doit pas être accessible via HTTPS
- L'application API doit uniquement être accessible via HTTPS
- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL
- Le chiffrement de disque doit être appliqué sur les machines virtuelles
- Les variables de compte Automation doivent être chiffrées
- Seules les connexions sécurisées à votre cache Redis doivent être activées
- La sécurisation du transfert vers des comptes de stockage doit être activée
- La propriété ClusterProtectionLevel doit être définie sur EncryptAndSign pour les clusters Service Fabric
- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL
- Déployer le chiffrement transparent des données sur les bases de données SQL

## <a name="51-62-66-and-1121-vulnerability-scanning-and-system-updates"></a>5.1, 6.2, 6.6 and 11.2.1 Analyse des vulnérabilités et mises à jour système

Ce blueprint vous permet de gérer les vulnérabilités du système d’informations en attribuant des définitions [Azure Policy](../../../policy/overview.md) qui supervisent les mises à jour système manquantes ainsi que les vulnérabilités en rapport avec le système d’exploitation, SQL et les machines virtuelles dans Azure Security Center. Azure Security Center fournit des fonctionnalités de création de rapports qui vous permettent d’obtenir des insights en temps réel sur l’état de la sécurité des ressources Azure déployées.

- Superviser les agents Endpoint Protection manquants dans Azure Security Center
- Déployer l’extension Microsoft IaaSAntimalware par défaut pour Windows Server
- Déployer la détection de menaces sur les serveurs SQL
- Des mises à jour système doivent être installées sur vos machines
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités

## <a name="711-712-and-713-separation-of-duties"></a>7.1.1. 7.1.2 et 7.1.3 Séparation des tâches

Le fait d’avoir un seul propriétaire d’abonnement Azure ne permet pas d’assurer la redondance administrative. À l’inverse, un nombre trop élevé de propriétaires d’abonnement Azure peut augmenter le risque d’une violation par le biais d’un compte de propriétaire compromis. Ce blueprint vous permet de maintenir un nombre approprié de propriétaires d’abonnement Azure en affectant des définitions [Azure Policy](../../../policy/overview.md) qui auditent le nombre de propriétaires pour les abonnements Azure. La gestion des autorisations des propriétaires d’abonnement peut vous aider à implémenter une séparation appropriée des tâches.

- Plusieurs propriétaires doivent être affectés à votre abonnement
- Trois propriétaires au plus doivent être désignés pour votre abonnement 

## <a name="32-721-831a-and-831b-management-of-privileged-access-rights"></a>3.2, 7.2.1, 8.3.1.a et 8.3.1.b Gestion des droits d’accès privilégié

Ce blueprint vous permet de limiter et de contrôler les droits d’accès privilégié en affectant des définitions [Azure Policy](../../../policy/overview.md) pour auditer les comptes externes avec des autorisations de propriétaire et/ou en écriture et les comptes avec des autorisations de propriétaire et/ou en écriture pour lesquels l’authentification multifacteur n’est pas activée. Le contrôle d’accès en fonction du rôle (Azure RBAC) pour gérer qui a accès aux ressources Azure. Ces dernières étant non exemptes d’erreurs, le fait de savoir où elles sont implémentées peut vous aider à déterminer les besoins réels et l’implémentation appropriée. Ce blueprint affecte également des définitions [Azure Policy](../../../policy/overview.md) afin d’auditer l’utilisation de l’authentification Azure Active Directory pour les serveurs SQL. L’utilisation de l’authentification Azure Active Directory simplifie la gestion des autorisations et centralise la gestion centralisée des identités des utilisateurs de bases de données et d’autres services  
Microsoft.
 
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement
- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL
- Auditer l’utilisation de règles personnalisées RBAC

## <a name="812-and-815-least-privilege-and-review-of-user-access-rights"></a>8.1.2 et 8.1.5 Privilège minimum et Revue des droits d’accès utilisateurs

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer les utilisateurs qui ont accès aux ressources dans Azure. À l’aide du portail Azure, vous pouvez passer en revue les utilisateurs ayant accès aux ressources Azure et leurs autorisations. Ce blueprint affecte des définitions [Azure Policy](../../../policy/overview.md) qui permettent de déterminer les comptes devant en priorité faire l’objet d’une revue, notamment les comptes dépréciés et les comptes externes avec des autorisations élevées.

- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement

## <a name="813-removal-or-adjustment-of-access-rights"></a>8.1.3 Suppression ou adaptation des droits d’accès

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer les utilisateurs qui ont accès aux ressources dans Azure. Grâce à Azure Active Directory et Azure RBAC, vous pouvez mettre à jour les rôles d’utilisateur pour refléter les changements organisationnels. Si nécessaire, vous pouvez empêcher des comptes de se connecter (ou les supprimer). Dans ce cas, les droits d’accès aux ressources Azure sont immédiatement supprimés. Ce blueprint affecte des définitions [Azure Policy](../../../policy/overview.md) pour auditer le compte déprécié dont la suppression doit être envisagée.

- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

## <a name="823ab-824ab-and-825-password-based-authentication"></a>8.2.3.a,b, 8.2.4.a,b et 8.2.5 Authentification basée sur un mot de passe

Ce blueprint vous aide à appliquer des mots de passe forts en affectant des définitions [Azure Policy](../../../policy/overview.md) qui détectent les machines virtuelles Windows ne mettant pas en œuvre une force minimale ni d’autres exigences relatives aux mots de passe. Le fait d’avoir connaissance des machines virtuelles qui enfreignent la stratégie de force des mots de passe peut vous aider à prendre des actions correctives visant à rendre les mots de passe de tous les comptes d’utilisateurs de machine virtuelle conformes à la stratégie.

- \[Préversion\] : Auditer les machines virtuelles Windows qui n’ont pas l’antériorité maximale du mot de passe définie sur 70 jours
- \[Préversion\] : Déployer des exigences pour auditer les machines virtuelles Windows qui n’ont pas l’antériorité maximale du mot de passe définie sur 70 jours
- \[Préversion\] : Auditer les machines virtuelles Windows qui ne limitent pas la longueur minimale du mot de passe à 14 caractères
- \[Préversion\] : Déployer des exigences pour auditer les machines virtuelles Windows qui ne limitent pas la longueur minimale du mot de passe à 14 caractères
- \[Préversion\] : Auditer les machines virtuelles Windows qui autorisent la réutilisation des 24 mots de passe précédents
- \[Préversion\] : Déployer des exigences pour auditer les machines virtuelles Windows qui autorisent la réutilisation des 24 mots de passe précédents

## <a name="103-and-1054-audit-generation"></a>10.3 et 10.5.4 Génération de l’audit

Ce blueprint vous permet de garantir que les événements système sont journalisés en affectant des définitions [Azure Policy](../../../policy/overview.md) qui vérifient les paramètres de journalisation sur les ressources Azure.
Les journaux de diagnostic fournissent des insights sur les opérations effectuées dans vos ressources Azure. Les journaux Azure s’appuient sur les horloges internes synchronisées pour créer un enregistrement corrélé dans le temps des événements entre les ressources.

- L’audit doit être activé sur les paramètres de sécurité des données avancés sur SQL Server
- Auditer le paramètre de diagnostic
- Auditer les paramètres d'audit au niveau du serveur SQL
- Déployer l’audit sur des serveurs SQL
- Les comptes de stockage doivent être migrés vers de nouvelles ressources Azure Resource Manager
- Les machines virtuelles doivent être migrées vers de nouvelles ressources Azure Resource Manager

## <a name="1236-and-1237-information-security"></a>12.3.6 et 12.3.7 Sécurité des informations

Ce blueprint vous permet de gérer et de contrôler votre réseau en attribuant des définitions [Azure Policy](../../../policy/overview.md) qui auditent les emplacements réseau acceptables et les produits d’entreprise approuvés autorisés pour l’environnement. Chaque entreprise peut personnaliser ceux-ci à l'aide des paramètres de chaque stratégie.

- Emplacements autorisés
- Emplacements autorisés pour les groupes de ressources

## <a name="next-steps"></a>Étapes suivantes

Vous venez de passer en revue le mappage des contrôles du blueprint PCI-DSS v3.2.1. Consultez à présent les articles suivants pour obtenir une vue d'ensemble et découvrir comment déployer cet exemple :

> [!div class="nextstepaction"]
> [Blueprint PCI-DSS v3.2.1 - Vue d'ensemble](./index.md)
> [Blueprint PCI-DSS v3.2.1 - Déployer des étapes](./deploy.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).