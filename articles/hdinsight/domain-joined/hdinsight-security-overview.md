---
title: Vue d’ensemble de la sécurité d’entreprise dans Azure HDInsight
description: Découvrez les différentes méthodes pour garantir la sécurité de l’entreprise dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: seoapr2020
ms.date: 08/24/2020
ms.openlocfilehash: 7f450d54a0039f591178ae839fbb404f31d80671
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537267"
---
# <a name="overview-of-enterprise-security-in-azure-hdinsight"></a>Vue d’ensemble de la sécurité d’entreprise dans Azure HDInsight

Azure HDInsight propose un certain nombre de méthodes pour répondre aux besoins de votre entreprise en matière de sécurité. La plupart de ces solutions ne sont pas activées par défaut. Cette flexibilité vous permet de choisir les fonctionnalités de sécurité les plus importantes pour vous, et vous aide à éviter de payer pour des fonctionnalités dont vous n’avez pas besoin. Cette flexibilité signifie également qu’il est de votre responsabilité de vérifier que les solutions appropriées sont activées pour votre installation et votre environnement.

Cet article aborde les solutions de sécurité du point de vue des quatre piliers de sécurité classiques : la sécurité du périmètre, l’authentification, l’autorisation et le chiffrement.

Il présente également le **Pack Sécurité Entreprise (ESP) Azure HDInsight** , qui fournit l’authentification basée sur Active Directory, la prise en charge multi-utilisateur et le contrôle d’accès en fonction du rôle pour les clusters HDInsight.

## <a name="enterprise-security-pillars"></a>Piliers de la sécurité d’entreprise

L’une des manières d’aborder la sécurité de l’entreprise divise les solutions de sécurité en quatre groupes principaux en fonction du type de contrôle. Ces groupes sont également appelés piliers de sécurité. Il s’agit de la sécurité du périmètre, de l’authentification, de l’autorisation et du chiffrement.

### <a name="perimeter-security"></a>Sécurité du périmètre

La sécurité du périmètre dans HDInsight est obtenue par le biais de [réseaux virtuels](../hdinsight-plan-virtual-network-deployment.md). Un administrateur d’entreprise peut créer un cluster à l’intérieur d’un réseau virtuel (VNET) et utiliser des groupes de sécurité réseau (NSG) pour restreindre l’accès au réseau virtuel. Seules les adresses IP autorisées dans les règles de groupes de sécurité réseau entrantes sont en mesure de communiquer avec le cluster HDInsight. Cette configuration assure la sécurité du périmètre.

Tous les clusters déployés sur un réseau virtuel auront également un point de terminaison privé. Le point de terminaison correspond à une adresse IP privée sur le réseau virtuel pour un accès HTTP privé aux passerelles de cluster.

### <a name="authentication"></a>Authentification

Le [Pack Sécurité Entreprise](apache-domain-joined-architecture.md) d’HDInsight fournit l’authentification basée sur Active Directory, la prise en charge multi-utilisateur et le contrôle d’accès en fonction du rôle. L’intégration d’Active Directory est obtenue à l’aide d’[Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md). Grâce à ces fonctionnalités, vous pouvez créer un cluster HDInsight joint à un domaine Active Directory, puis configurer une liste d’employés de l’entreprise qui peuvent s’authentifier auprès du cluster.

Avec cette configuration, des employés de l’entreprise peuvent se connecter aux nœuds du cluster à l’aide de leurs informations d’identification de domaine. Ils peuvent aussi utiliser leurs informations d’identification de domaine pour s’authentifier auprès d’autres points de terminaison approuvés, comme les vues Apache Ambari, ODBC, JDBC, PowerShell et les API REST, pour interagir avec le cluster.

### <a name="authorization"></a>Autorisation

L’une des bonnes pratiques que la plupart des entreprises appliquent consiste à s’assurer que chaque employé ne dispose pas d’un accès complet à toutes les ressources de l’entreprise. Parallèlement, l’administrateur peut définir des stratégies de contrôle d’accès en fonction du rôle pour les ressources du cluster. Cette action est uniquement disponible dans les clusters ESP.

L’administrateur Hadoop peut configurer le contrôle d’accès en fonction du rôle (RBAC). Les configurations sécurisent Apache [Hive](apache-domain-joined-run-hive.md), [HBase](apache-domain-joined-run-hbase.md) et [Kafka](apache-domain-joined-run-kafka.md) avec des plug-ins Apache Ranger. La configuration de stratégies RBAC vous permet d’associer des autorisations à un rôle dans l’organisation. Cette couche d’abstraction permet de s’assurer plus facilement que les utilisateurs disposent uniquement des autorisations nécessaires pour effectuer leur travail. Ranger vous permet également d’auditer l’accès aux données par les employés et toute modification apportée aux stratégies de contrôle d’accès.

Par exemple, l’administrateur peut configurer [Apache Ranger](https://ranger.apache.org/) pour définir des stratégies de contrôle d’accès pour Hive. Cette fonctionnalité garantit le filtrage au niveau des lignes et des colonnes (masquage des données), et filtre les données sensibles contre les utilisateurs non autorisés.

### <a name="auditing"></a>Audit

L’audit de l’accès aux ressources de cluster est nécessaire pour effectuer le suivi des accès non autorisés ou non intentionnels des ressources. Il est tout aussi important que la protection des ressources du cluster contre tout accès non autorisé.

L’administrateur peut afficher et signaler tout accès aux données et aux ressources du cluster HDInsight. L’administrateur peut afficher et signaler les modifications des stratégies de contrôle d’accès.

Pour accéder aux journaux d’audit Apache Ranger et Ambari, et aux journaux d’accès SSH, [activez Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md#cluster-auditing), puis affichez les tables qui fournissent des enregistrements d’audit.

### <a name="encryption"></a>Chiffrement

La protection des données est essentielle pour se conformer aux exigences de l’organisation en matière de conformité et de sécurité. Parallèlement à la restriction de l’accès aux données des employés non autorisés, vous devez procéder à leur chiffrement.

HDInsight prend en charge le chiffrement des données au repos avec les clés gérées par la plateforme et les [clés gérées par le client](../disk-encryption.md). Le chiffrement des données en transit est géré avec TLS et IPSec. Pour plus d’informations, consultez [Chiffrement en transit pour Azure HDInsight](encryption-in-transit.md).

### <a name="compliance"></a>Conformité

Les offres de conformité Azure reposent sur différents types de garanties, notamment des certifications formelles, des attestations, des validations et des autorisations, des évaluations produites par des sociétés d’audit tierces indépendantes, des amendements contractuels, des auto-évaluations et des documents d’aide à la clientèle produits par Microsoft. Pour plus d'informations sur la conformité de HDInsight, reportez-vous au [Centre de gestion de la confidentialité Microsoft](https://www.microsoft.com/trust-center) et à la [Présentation de la conformité Microsoft Azure](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942).

## <a name="shared-responsibility-model"></a>Modèle de responsabilité partagée

L’image suivante résume les principales zones de sécurité du système et les solutions de sécurité disponibles dans chacune d’entre elles. Elle met également en évidence les zones de sécurité qui vous incombent en tant que client, et celles qui sont de la responsabilité d’HDInsight en tant que fournisseur de services.

![Diagramme des responsabilités partagées HDInsight](./media/hdinsight-security-overview/hdinsight-shared-responsibility.png)

Le tableau suivant fournit des liens vers des ressources pour chaque type de solution de sécurité.

| Zone de sécurité | Solutions disponibles | Partie responsable |
|---|---|---|
| Sécurité de l’accès aux données | Configurer des [listes de contrôle d’accès (ACL)](../../storage/blobs/data-lake-storage-access-control.md) pour Azure Data Lake Storage Gen1 et Gen2  | Customer |
|  | Activer la propriété [« Transfert sécurisé requis »](../../storage/common/storage-require-secure-transfer.md) sur le compte de stockage | Customer |
|  | Configurer des [pare-feu Stockage Azure](../../storage/common/storage-network-security.md) et des réseaux virtuels | Customer |
|  | Configurer des [points de terminaison de service de réseau virtuel Azure](../../virtual-network/virtual-network-service-endpoints-overview.md) pour Cosmos DB et [Azure SQL DB](../../azure-sql/database/vnet-service-endpoint-rule-overview.md) | Customer |
|  | Assurez-vous que la fonctionnalité [Chiffrement en transit](./encryption-in-transit.md) est activée afin d’utiliser TLS et IPSec pour les communications à l’intérieur du cluster. | Customer |
|  | Configurer des [clés gérées par le client](../../storage/common/customer-managed-keys-configure-key-vault.md) pour le chiffrement du Stockage Azure | Customer |
|  | Contrôler l’accès à vos données par le support Azure à l’aide de [Customer Lockbox](../../security/fundamentals/customer-lockbox-overview.md) | Customer |
| Sécurité des applications et des middlewares (intergiciels) | Intégrer à AAD-DS et [configurer ESP](apache-domain-joined-configure-using-azure-adds.md) ou utiliser [HIB pour l’authentification OAuth](identity-broker.md)| Customer |
|  | Configurer des stratégies d’[autorisation Apache Ranger](apache-domain-joined-run-hive.md) | Customer |
|  | Utiliser les [journaux Azure Monitor](../hdinsight-hadoop-oms-log-analytics-tutorial.md) | Customer |
| Sécurité du système d’exploitation | Créer des clusters avec l’image de base sécurisée la plus récente | Customer |
|  | Garantir la [mise à jour corrective du système d’exploitation](../hdinsight-os-patching.md) à intervalles réguliers | Customer |
|  | Garantir le [chiffrement de disque CMK pour les machines virtuelles](../disk-encryption.md) | Customer |
| Sécurité du réseau | Configurer un [réseau virtuel](../hdinsight-plan-virtual-network-deployment.md) |
|  | Configurer des [règles de groupe de sécurité réseau entrantes](../control-network-traffic.md) ou une [liaison privée](../hdinsight-private-link.md) | Customer |
|  | Configurer la [restriction du trafic sortant](../hdinsight-restrict-outbound-traffic.md) avec le pare-feu | Customer |
|  | Configurer le [chiffrement IPSec en transit](encryption-in-transit.md) entre les nœuds de cluster | Customer |
| Infrastructure virtualisée | N/A | HDInsight (fournisseur de cloud) |
| Sécurité de l’infrastructure physique | N/A | HDInsight (fournisseur de cloud) |

## <a name="next-steps"></a>Étapes suivantes

* [Planifier des clusters HDInsight avec ESP](apache-domain-joined-architecture.md)
* [Configurer des clusters HDInsight avec ESP](./apache-domain-joined-configure-using-azure-adds.md)
* [Gérer des clusters HDInsight avec ESP](apache-domain-joined-manage.md)