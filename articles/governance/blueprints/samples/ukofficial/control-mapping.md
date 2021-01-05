---
title: Exemples de contrôle de blueprint UK OFFICIAL et UK NHS
description: Mappage des contrôles des exemples de blueprint UK OFFICIAL et UK NHS. Chaque contrôle est mis en correspondance avec une ou plusieurs définitions Azure Policy qui simplifient l’évaluation.
ms.date: 11/05/2020
ms.topic: sample
ms.openlocfilehash: 352ba30a21c638c68401e2f8e471096a777fbde9
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009379"
---
# <a name="control-mapping-of-the-uk-official-and-uk-nhs-blueprint-samples"></a>Mappage des contrôles des exemples de blueprint UK OFFICIAL et UK NHS

L’article suivant décrit en détail comment les exemples de blueprint UK OFFICIAL et UK NHS mappent aux contrôles UK OFFICIAL et UK NHS. Pour plus d’informations sur les contrôles, voir [UK OFFICIAL](https://www.gov.uk/government/publications/government-security-classifications).

Les mappages suivants sont effectués aux contrôles **UK OFFICIAL** et **UK NHS**. Utilisez le volet de navigation de droite pour accéder directement à la correspondance d’un contrôle spécifique. De nombreux contrôles mappés sont implémentés avec une initiative [Azure Policy](../../../policy/overview.md). Pour examiner l’initiative complète, ouvrez **Stratégie** dans le portail Azure et sélectionnez la page **Définitions**. Ensuite, recherchez et sélectionnez l’initiative de stratégie intégrée **\[Préversion\] Auditer les contrôles UK OFFICIAL et UK NHS et déployer des extensions de machine virtuelle spécifiques pour prendre en charge les exigences d’audit**.

> [!IMPORTANT]
> Chaque contrôle ci-dessous est associé à une ou plusieurs définitions [Azure Policy](../../../policy/overview.md). Ces stratégies peuvent vous aider à [évaluer la conformité](../../../policy/how-to/get-compliance-data.md) avec le contrôle ; toutefois, il n’existe pas souvent de correspondance un-à-un ou parfaite entre un contrôle et une ou plusieurs stratégies. Ainsi, la **conformité** dans Azure Policy fait uniquement référence aux stratégies elles-mêmes ; cela ne garantit pas que vous êtes entièrement conforme à toutes les exigences d’un contrôle. En outre, la norme de conformité comprend des contrôles qui ne sont traités par aucune définition Azure Policy pour l’instant. Par conséquent, la conformité dans Azure Policy n’est qu’une vue partielle de l’état de conformité global. Les associations entre les contrôles et les définitions Azure Policy pour cet exemple de blueprint de conformité peuvent changer au fil du temps. Pour afficher l’historique des changements, consultez l’[historique des validations GitHub](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/ukofficial/control-mapping.md).

## <a name="1-data-in-transit-protection"></a>1 Protection des données en transit

Le blueprint vous aide à vérifier que le transfert d’informations avec les services Azure est sécurisé en affectant des définitions [Azure Policy](../../../policy/overview.md) qui auditent les connexions non sécurisées aux comptes de stockage et au cache Redis.

- Seules les connexions sécurisées à votre cache Redis doivent être activées
- La sécurisation du transfert vers des comptes de stockage doit être activée
- Afficher les résultats d’audit des serveurs web Windows qui n’utilisent pas de protocole de communication sécurisé
- L'application web ne doit pas être accessible via HTTPS
- Function App ne doit pas être accessible via HTTPS
- L'application API doit uniquement être accessible via HTTPS

## <a name="23-data-at-rest-protection"></a>2.3 Protection des données au repos

Ce blueprint vous aide à appliquer votre stratégie sur l’utilisation des contrôles de chiffrement en affectant des définitions [Azure Policy](../../../policy/overview.md) qui appliquent des contrôles de chiffrement spécifiques et détectent l’utilisation de paramètres de chiffrement faibles. Le fait de savoir où vos ressources Azure peuvent avoir des configurations de chiffrement non optimales peut vous aider à prendre des mesures correctives visant à vérifier que les ressources sont configurées conformément à votre stratégie de sécurité des informations. Plus précisément, les stratégies assignées par ce blueprint exigent le chiffrement des comptes Data Lake Storage ; exigent le chiffrement transparent des données dans les bases de données SQL ; vérifient le chiffrement sur les comptes de stockage, bases de données SQL, disques de machine virtuelle et variables de compte Automation ; vérifient les connexions non sécurisées aux comptes de stockage et au Cache Redis ; vérifient si le chiffrement par mot de passe des machines virtuelles est faible ; et vérifient les communication Service Fabric non chiffrées.

- Le chiffrement de disque doit être appliqué sur les machines virtuelles
- Les variables de compte Automation doivent être chiffrées
- La propriété ClusterProtectionLevel doit être définie sur EncryptAndSign pour les clusters Service Fabric
- La technologie Transparent Data Encryption doit être activée sur les bases de données SQL
- Déployer le chiffrement transparent des données sur les bases de données SQL
- Exiger un chiffrement sur les comptes Data Lake Store
- Emplacements autorisés (« UK SOUTH » et « UK WEST »sont codés en dur)
- Emplacements autorisés pour les groupes de ressources (« UK SOUTH » et « UK WEST » sont codés en dur)

## <a name="52-vulnerability-management"></a>5.2 Gestion des vulnérabilités

Ce blueprint vous permet de gérer les vulnérabilités du système d’informations en affectant des définitions [Azure Policy](../../../policy/overview.md) qui surveillent l’absence de protection de point de terminaison, les mises à jour système manquantes, ainsi que les vulnérabilités en rapport avec le système d’exploitation, SQL et les machines virtuelles. Ces insights fournissent des informations en temps réel sur l’état de la sécurité de vos ressources déployées et peuvent vous aider à classer par ordre de priorité les actions de correction.

- Superviser les agents Endpoint Protection manquants dans Azure Security Center
- Des mises à jour système doivent être installées sur vos machines
- Les mises à jour système doivent être installées sur les groupes de machines virtuelles identiques
- Les vulnérabilités de la configuration de sécurité sur vos machines doivent être corrigées
- Les vulnérabilités de vos bases de données SQL doivent être éliminées
- Les vulnérabilités doivent être corrigées avec une solution d’évaluation des vulnérabilités
- L’évaluation des vulnérabilités doit être activée sur vos serveurs SQL
- L’évaluation des vulnérabilités doit être activée sur votre instance managée SQL
- Les vulnérabilités détectées dans la configuration de la sécurité de vos groupes de machines virtuelles identiques doivent être corrigées
- Advanced Data Security doit être activée sur votre instance managée SQL
- Advanced Data Security doit être activé sur vos serveurs SQL

## <a name="53-protective-monitoring"></a>5.3 Surveillance à des fins de protection

Ce blueprint vous aide à protéger les ressources du système d’informations en affectant des définitions [Azure Policy](../../../policy/overview.md) qui fournissent une supervision de protection sur l’accès illimité, l’activité de liste verte et les menaces.

- Les comptes de stockage doivent limiter l’accès réseau
- Les contrôles d’application adaptatifs permettant de définir les applications sécurisées doivent être activés sur vos machines
- Auditer des machines virtuelles pour lesquelles la reprise d’activité après sinistre n’est pas configurée
- Azure DDoS Protection Standard doit être activé
- Les types Advanced Threat Protection doivent être définis sur « Tous » dans les paramètres Advanced Data Security de l’instance gérée SQL.
- Les types Advanced Threat Protection doivent être définis sur « Tous » dans les paramètres Advanced Data Security du serveur SQL.
- Déployer la détection de menaces sur les serveurs SQL
- Déployer l’extension Microsoft IaaSAntimalware par défaut pour Windows Server

## <a name="9-secure-user-management"></a>9 Gestion sécurisée des utilisateurs 

Le contrôle d’accès en fonction du rôle Azure (Azure RBAC) vous permet de gérer les utilisateurs qui ont accès aux ressources dans Azure.
À l’aide du portail Azure, vous pouvez passer en revue les utilisateurs ayant accès aux ressources Azure et leurs autorisations. Ce blueprint vous permet de limiter et de contrôler les droits d’accès en affectant des définitions [Azure Policy](../../../policy/overview.md) pour auditer les comptes externes avec des autorisations de propriétaire ou de lecture/écriture, et les comptes avec des autorisations de propriétaire, de lecture ou d’écriture pour lesquels l’authentification multifacteur n’est pas activée.

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement

## <a name="10-identity-and-authentication"></a>10 Identité et authentification

Ce blueprint vous permet de limiter et de contrôler les droits d’accès en affectant des définitions [Azure Policy](../../../policy/overview.md) pour auditer les comptes externes avec des autorisations de propriétaire ou de lecture/écriture, et les comptes avec des autorisations de propriétaire, de lecture ou d’écriture pour lesquels l’authentification multifacteur n’est pas activée.

- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations de propriétaire sur votre abonnement
- L’authentification multifacteur doit être activée sur les comptes disposant d’autorisations d’écriture sur votre abonnement
- L'authentification multifacteur doit être activée sur les comptes disposant d’autorisations de lecture de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de lecture doivent être supprimés de votre abonnement

Ce blueprint affecte des définitions Azure Policy afin de vérifier l’utilisation de l’authentification Azure Active Directory pour les serveurs SQL et Service Fabric. L’utilisation de l’authentification Azure Active Directory permet une gestion simplifiée des autorisations et une gestion centralisée des identités des utilisateurs de bases de données et d’autres services Microsoft.

- Un administrateur Azure Active Directory doit être approvisionné pour les serveurs SQL
- Les clusters Service Fabric ne doivent utiliser Azure Active Directory que pour l’authentification client

Ce blueprint affecte également des définitions Azure Policy qui permettent de déterminer les comptes devant en priorité faire l’objet d’une révision, notamment les comptes déconseillés et les comptes externes. Si nécessaire, vous pouvez empêcher des comptes de se connecter (ou les supprimer). Dans ce cas, les droits d’accès aux ressources Azure sont immédiatement supprimés.
Ce blueprint affecte deux définitions Azure Policy pour auditer un compte déconseillé dont la suppression doit être envisagée.

- Les comptes déconseillés doivent être supprimés de votre abonnement
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations d’écriture doivent être supprimés de votre abonnement

Ce blueprint affecte également une définition Azure Policy qui vérifie les autorisations associées aux fichiers de mot de passe des machines virtuelles Linux qui émettent une alerte si elles sont définies incorrectement. Cette conception vous d’appliquer une action corrective pour vérifier que les authentificateurs ne sont pas compromis.

- Afficher les résultats d’audit des machines virtuelles Linux qui n’ont pas les autorisations de fichier passwd définies sur 0644

Ce blueprint vous aide à appliquer des mots de passe forts en affectant des définitions Azure Policy qui détectent les machines virtuelles Windows ne mettant pas en œuvre une force minimale ou d’autres exigences relatives aux mots de passe. Le fait d’avoir connaissance des machines virtuelles qui enfreignent la stratégie de force des mots de passe peut vous aider à prendre des actions correctives visant à rendre les mots de passe de tous les comptes d’utilisateurs de machine virtuelle conformes à la stratégie.

- Afficher les résultats d’audit des machines virtuelles Windows qui n’ont pas le paramètre de complexité de mot de passe activé
- Afficher les résultats d’audit des machines virtuelles Windows qui n’ont pas l’antériorité maximale du mot de passe définie sur 70 jours
- Afficher les résultats d’audit des machines virtuelles Windows qui n’ont pas l’antériorité minimale du mot de passe définie sur 1 jour
- Afficher les résultats d’audit des machines virtuelles Windows qui ne limitent pas la longueur minimale du mot de passe à 14 caractères
- Afficher les résultats d’audit des machines virtuelles Windows qui autorisent la réutilisation des 24 mots de passe précédents

Ce blueprint vous permet également de contrôler l’accès aux ressources Azure en affectant des définitions Azure Policy. Ces stratégies vérifient l’utilisation des types de ressources et des configurations susceptibles d’accorder un accès plus permissif aux ressources. Le fait de savoir quelles ressources enfreignent ces stratégies peut vous aider à prendre des actions correctives visant à limiter l’accès aux ressources Azure aux utilisateurs autorisés.

- Afficher les résultats d’audit des machines virtuelles Linux qui ont des comptes sans mot de passe
- Afficher les résultats d’audit des machines virtuelles Linux qui autorisent les connexions à distance à partir des comptes sans mot de passe
- Les comptes de stockage doivent être migrés vers de nouvelles ressources Azure Resource Manager
- Les machines virtuelles doivent être migrées vers de nouvelles ressources Azure Resource Manager
- Faire l’audit des machines virtuelles n’utilisant aucun disque managé

## <a name="11-external-interface-protection"></a>11 Protection des interfaces externes

Outre l’utilisation de plus de 25 stratégies pour une gestion sécurisée appropriée des utilisateurs, ce blueprint vous aide à protéger les interfaces de service contre les accès non autorisés en attribuant une définition [ Stratégie Azure](../../../policy/overview.md) qui surveille les comptes de stockage non restreints.
Des comptes de stockage avec un accès illimité, peuvent permettre un accès involontaire à des informations contenues dans le système d’information. Cet blueprint attribue également une stratégie qui active des contrôles d’application adaptatifs sur des machines virtuelles.

- Les comptes de stockage doivent limiter l’accès réseau
- Les contrôles d’application adaptatifs permettant de définir les applications sécurisées doivent être activés sur vos machines
- L'accès via un point de terminaison accessible sur Internet doit être limité
- Les recommandations de renforcement de réseau adaptatif doivent être appliquées sur les machines virtuelles accessibles à partir d’Internet
- La solution de protection des points de terminaison doit être installée sur les groupes de machines virtuelles identiques
- Les ports de gestion des machines virtuelles doivent être protégés par un contrôle d’accès réseau juste-à-temps
- Le débogage à distance doit être désactivé pour les applications de fonction
- Le débogage à distance doit être désactivé pour les applications web
- Le débogage à distance doit être désactivé pour les applications API

## <a name="13-audit-information-for-users"></a>13 Informations d’audit pour les utilisateurs

Ce blueprint vous permet de garantir que les événements système sont journalisés en affectant des définitions [Azure Policy](../../../policy/overview.md) qui vérifient les paramètres de journalisation sur les ressources Azure.
Une stratégie affectée vérifie également si les machines virtuelles n’envoient pas de journaux à un espace de travail Log Analytics spécifié.

- Advanced Data Security doit être activé sur vos serveurs SQL
- Auditer le paramètre de diagnostic
- \[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Linux
- \[Préversion\] : Déployer Log Analytics Agent pour les machines virtuelles Windows
- Déployer Network Watcher lors de la création de réseaux virtuels


## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez passé en revue le mappage des contrôles des blueprints UK OFFICIAL et UK NHS, consultez les articles suivants pour découvrir une vue d’ensemble et comment déployer cet exemple :

> [!div class="nextstepaction"]
> [Blueprints UK OFFICIAL et UK NHS – Vue d’ensemble](./index.md)
> [Blueprints UK OFFICIAL et UK NHS – Étapes de déploiement](./deploy.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
