---
title: 'Sécurité : Effectuer la migration d’Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les bonnes pratiques concernant la sécurité et DevOps pour la migration des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/19/2019
ms.openlocfilehash: 6fd59a346a36a304f699894c493d8ba9800489b2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545257"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---security-and-devops-best-practices"></a>Migrer des clusters Apache Hadoop locaux vers Azure HDInsight - bonnes pratiques de sécurité et DevOps

Cet article fournit des recommandations pour la sécurité et DevOps dans les systèmes Azure HDInsight. Il fait partie d’une série qui propose des bonnes pratiques pour aider à la migration des systèmes Apache Hadoop locaux vers Azure HDInsight.

## <a name="secure-and-govern-cluster-with-enterprise-security-package"></a>Sécuriser et régir un cluster avec le Pack Sécurité Entreprise

Le Pack Sécurité Entreprise (ESP) accepte l’authentification basée sur Active Directory, la prise en charge multi-utilisateur et le contrôle d’accès en fonction du rôle. Quand l’option ESP est choisie, le cluster HDInsight est joint au domaine Active Directory et l’administrateur d’entreprise peut configurer le contrôle d’accès en fonction du rôle (RBAC) pour assurer la sécurité Apache Hive avec Apache Ranger. L’administrateur peut également auditer l’accès des données par les employés et les modifications apportées aux stratégies de contrôle d’accès.

Les fonctionnalités ESP sont disponibles sur les types de clusters suivants : Apache Hadoop, Apache Spark, Apache HBase, Apache Kafka et Interactive Query (Hive LLAP).

Utilisez les étapes suivantes pour déployer le cluster HDInsight joint au domaine :

- Déployer Azure Active Directory (AAD) en transmettant le nom de domaine.
- Déployer Azure Active Directory Domain Services (AAD DS).
- Créer le réseau virtuel et le sous-réseau requis.
- Déployer une machine virtuelle dans le réseau virtuel pour gérer AAD DS.
- Joindre la machine virtuelle au domaine.
- Installer AD et les outils DNS.
- Demander à l’administrateur AAD DS de créer une unité d’organisation.
- Activer LDAPS pour AAD DS.
- Créer un compte de service dans Azure Active Directory avec l’autorisation d’administrateur en lecture et écriture déléguée sur l’unité d’organisation. Ce compte de service peut ensuite joindre des machines au domaine et placer les principaux de machine dans l’unité d’organisation. Il peut aussi créer des principaux de service au sein de l’unité d’organisation que vous spécifiez lors de la création du cluster.

    > [!Note]
    > Le compte de service n’a pas besoin d’être un compte administrateur de domaine AD.

- Déployer le cluster ESP HDInsight en définissant les paramètres suivants :

    |Paramètre |Description |
    |---|---|
    |Nom de domaine|nom de domaine associé à l’instance d’Azure AD DS.|
    |Nom d’utilisateur de domaine|compte de service dans le domaine managé du contrôleur de domaine Azure AD DS que vous avez créé à la section précédente, par exemple `hdiadmin@contoso.onmicrosoft.com`. Cet utilisateur de domaine devient l’administrateur de ce cluster HDInsight.|
    |Mot de passe du domaine|mot de passe du compte de service.|
    |Unité d’organisation|nom unique de l’unité d’organisation à utiliser avec le cluster HDInsight, par exemple `OU=HDInsightOU,DC=contoso,DC=onmicrosoft,DC=com`. Si cette unité d’organisation n’existe pas, le cluster HDInsight tente de créer l’unité d’organisation en utilisant les privilèges du compte de service.|
    |URL LDAPS|par exemple, `ldaps://contoso.onmicrosoft.com:636`.|
    |Accéder au groupe d’utilisateurs|groupes de sécurité dont vous souhaitez synchroniser les utilisateurs avec le cluster, par exemple `HiveUsers`. Si vous souhaitez spécifier plusieurs groupes d’utilisateurs, séparez-les par des points-virgules (« ; »). Le ou les groupes doivent exister dans le répertoire avant de créer le cluster ESP.|

Pour plus d’informations, consultez les articles suivants :

- [Introduction aux fonctions de sécurité Apache Hadoop sur la base de clusters HDInsight joints à un domaine](../domain-joined/hdinsight-security-overview.md)
- [Planifier des clusters Apache Hadoop Azure joints à un domaine dans HDInsight](../domain-joined/apache-domain-joined-architecture.md)
- [Configurer un cluster HDInsight joint à un domaine avec Azure Active Directory Domain Services](../domain-joined/apache-domain-joined-configure-using-azure-adds.md)
- [Synchroniser des utilisateurs Azure Active Directory vers un cluster HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
- [Configurer les stratégies Apache Hive dans HDInsight joint à un domaine](../domain-joined/apache-domain-joined-run-hive.md)
- [Exécuter Apache Oozie dans des clusters HDInsight Hadoop joints à un domaine](../domain-joined/hdinsight-use-oozie-domain-joined-clusters.md)

## <a name="implement-end-to-end-enterprise-security"></a>Implémenter la sécurité d’entreprise de bout en bout

La sécurité d’entreprise de bout en bout peut être obtenue à l’aide des contrôles suivants :

**Pipeline de données privées et protégées (sécurité au niveau du périmètre)**
    - La sécurité au niveau du périmètre peut être obtenue par le biais des réseaux virtuels Azure, des groupes de sécurité réseau et du service de passerelle.

**Authentification et autorisation pour l’accès aux données**
    - Créez un cluster HDInsight joint à un domaine avec Azure Active Directory Domain Services. (Pack Sécurité Entreprise).
    - Utilisez Ambari pour fournir l’accès en fonction du rôle aux ressources de cluster pour les utilisateurs AD.
    - Utilisez Apache Ranger pour définir des stratégies de contrôle d’accès pour Hive au niveau de la table/colonne/ligne.
    - L’accès SSH au cluster peut être limité uniquement à l’administrateur.

**Audit**
    - Affichez et signalez tout accès aux données et aux ressources du cluster HDInsight.
    - Affichez et signalez toutes les modifications des stratégies de contrôle d’accès.

**Chiffrement**
    - Chiffrement côté serveur transparent à l’aide de clés gérées par Microsoft ou de clés gérées par le client.
    - Dans le chiffrement des données en transit avec chiffrement côté client, https et TLS.

Pour plus d’informations, consultez les articles suivants :

- [Vue d’ensemble des réseaux virtuels Azure](../../virtual-network/virtual-networks-overview.md)
- [Vue d’ensemble des groupes de sécurité réseau Azure](../../virtual-network/network-security-groups-overview.md)
- [Peering de réseaux virtuels Azure](../../virtual-network/virtual-network-peering-overview.md)
- [Guide de sécurité du Stockage Azure](../../storage/blobs/security-recommendations.md)
- [Chiffrement du service de stockage Azure au repos](../../storage/common/storage-service-encryption.md)

## <a name="use-monitoring--alerting"></a>Utiliser la supervision et les alertes

Pour plus d’informations, consultez l’article :

[Vue d’ensemble d’Azure Monitor](../../azure-monitor/overview.md)

## <a name="upgrade-clusters"></a>Mettre à niveau des clusters

Effectuez régulièrement une mise à niveau vers la dernière version de HDInsight pour tirer parti des fonctionnalités les plus récentes. Les étapes suivantes peuvent servir à mettre à niveau le cluster vers la dernière version :

1. Créez un cluster HDInsight TEST à l’aide de la dernière version de HDInsight disponible.
1. Testez le nouveau cluster pour vous assurer que les travaux et les charges de travail fonctionnent comme prévu.
1. Modifiez les travaux, applications ou charges de travail en fonction des besoins.
1. Sauvegardez toutes les données temporaires stockées localement sur les nœuds du cluster,
1. Supprimez le cluster existant.
1. Créez un cluster avec la dernière version de HDInsight dans le même sous-réseau de réseau virtuel en utilisant les mêmes données et magasin de données par défaut que le cluster précédent.
1. Importez toutes les données temporaires qui ont été sauvegardées.
1. Démarrez des tâches ou poursuivez le traitement avec le nouveau cluster.

Pour plus d’informations, consultez l’article : [Mettre à niveau le cluster HDInsight](../hdinsight-upgrade-cluster.md).

## <a name="patch-cluster-operating-systems"></a>Appliquer un correctif aux systèmes d’exploitation de clusters

Pour plus d’informations, consultez l’article : [Mise à jour corrective du système d’exploitation pour HDInsight](../hdinsight-os-patching.md).

## <a name="post-migration"></a>Après la migration

1. **Corriger les applications**  : apportez de façon itérative les modifications nécessaires aux travaux, processus et scripts.
2. **Effectuer des tests**  : exécutez de façon itérative des tests fonctionnels et de performance.
3. **Optimiser**  : résolvez tous les problèmes de performances basés sur les résultats des tests ci-dessus, puis relancez les tests pour confirmer les améliorations des performances.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [HDInsight 4.0](./apache-hadoop-introduction.md).