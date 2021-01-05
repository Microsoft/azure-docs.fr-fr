---
title: Gérer des clusters du Pack Sécurité Entreprise – Azure HDInsight
description: Découvrez comment gérer des clusters Azure HDInsight avec le Pack Sécurité Entreprise.
author: omidm1
ms.author: omidm
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/04/2019
ms.openlocfilehash: 3dcb5d7ed75bda8422ba3bd461b08d3bfb2d974f
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92541007"
---
# <a name="manage-hdinsight-clusters-with-enterprise-security-package"></a>Gérer des clusters HDInsight avec le pack Sécurité Entreprise

Découvrez les utilisateurs et les rôles du pack Sécurité Entreprise (ESP) HDInsight et apprenez à gérer des clusters ESP.

## <a name="use-vscode-to-link-to-domain-joined-cluster"></a>Utiliser VSCode pour établir un lien avec un cluster joint à un domaine

Vous pouvez lier un cluster normal à l’aide du nom d’utilisateur Apache Ambari managé, mais également lier un cluster de sécurité Apache Hadoop à l’aide du nom d’utilisateur de domaine (par exemple : `user1@contoso.com`).

1. Ouvrez [Visual Studio Code](https://code.visualstudio.com/). Assurez-vous que l’extension [Spark & Hive Tools](../hdinsight-for-vscode.md) est installée.

1. Suivez les étapes de [liaison d’un cluster](../hdinsight-for-vscode.md#link-a-cluster) pour Visual Studio Code.

## <a name="use-intellij-to-link-to-domain-joined-cluster"></a>Utiliser IntelliJ pour établir un lien avec un cluster joint à un domaine

Vous pouvez lier un cluster normal à l’aide du nom d’utilisateur Ambari managé, mais également lier un cluster de sécurité Hadoop à l’aide du nom d’utilisateur de domaine (par exemple : `user1@contoso.com`).

1. Ouvrez IntelliJ IDEA. Assurez-vous que tous les [prérequis](../spark/apache-spark-intellij-tool-plugin.md#prerequisites) sont respectés.

1. Suivez les étapes de [liaison d’un cluster](../spark/apache-spark-intellij-tool-plugin.md#link-a-cluster) pour IntelliJ.

## <a name="use-eclipse-to-link-to-domain-joined-cluster"></a>Utiliser Eclipse pour établir un lien avec un cluster joint à un domaine

Vous pouvez lier un cluster normal à l’aide du nom d’utilisateur Ambari managé, mais également lier un cluster de sécurité Hadoop à l’aide du nom d’utilisateur de domaine (par exemple : `user1@contoso.com`).

1. Ouvrez Eclipse. Assurez-vous que tous les [prérequis](../spark/apache-spark-eclipse-tool-plugin.md#prerequisites) sont respectés.

1. Suivez les étapes de [liaison d’un cluster](../spark/apache-spark-eclipse-tool-plugin.md#link-a-cluster) pour Eclipse.

## <a name="access-the-clusters-with-enterprise-security-package"></a>Accéder aux clusters à l’aide du Pack Sécurité Entreprise

Enterprise Security Package (anciennement HDInsight Premium) permet à plusieurs utilisateurs d’accéder au cluster avec une authentification gérée par Active Directory et une autorisation exécutée par Apache Ranger et les ACL Storage (ACL ADLS). L’autorisation définit des limites sécurisées entre plusieurs utilisateurs et permet uniquement aux utilisateurs disposant de privilèges d’accéder aux données en fonction des stratégies d’autorisation.

La sécurité et l’isolement des utilisateurs sont des aspects importants pour un cluster HDInsight qui utilise Enterprise Security Package. Pour répondre à ces exigences, l’accès SSH au cluster avec le Pack Sécurité Entreprise est pris en charge pour l’utilisateur local sélectionné au moment de la création du cluster, ainsi que pour les utilisateurs disponibles dans AAD-DS (c.-à-d. Kerberos). Le tableau suivant présente les méthodes d’accès recommandées pour chaque type de cluster :

|Charge de travail|Scénario|Méthode d'accès|
|--------|--------|-------------|
|Apache Hadoop|Hive – Travaux/requêtes interactifs  |<ul><li>[Beeline](#beeline)</li><li>[Affichage Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Apache Spark|Travaux/requêtes interactifs, PySpark interactif|<ul><li>[Beeline](#beeline)</li><li>[Zeppelin avec Livy](../spark/apache-spark-zeppelin-notebook.md)</li><li>[Affichage Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Apache Spark|Scénarios de traitement par lots – Spark Submit, PySpark|<ul><li>[Livy](../spark/apache-spark-livy-rest-interface.md)</li></ul>|
|Requête interactive (LLAP)|Interactive|<ul><li>[Beeline](#beeline)</li><li>[Affichage Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Power BI](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Quelconque|Installer une application personnalisée|<ul><li>[Actions de script](../hdinsight-hadoop-customize-cluster-linux.md)</li></ul>|

   > [!NOTE]  
   > Jupyter n’est pas installé/pris en charge dans Pack Sécurité Entreprise.

L’utilisation d’API standard est utile en matière de sécurité. Vous bénéficiez également des avantages suivants :

- **Gestion** : vous pouvez gérer votre code et automatiser vos travaux à l’aide des API standard, comme Livy, HS2, etc.
- **Audit** – Avec SSH, il n’y a aucun moyen de contrôler qui utilise SSH pour accéder au cluster. La situation est différente lorsque les travaux sont construits via des points de terminaison standard puisqu’ils sont exécutés dans le contexte de l’utilisateur.

### <a name="use-beeline"></a><a name="beeline"></a>Utiliser BeeLine

Installez Beeline sur votre ordinateur et connectez-vous via l’Internet public, puis utilisez les paramètres suivants :

```
- Connection string: -u 'jdbc:hive2://<clustername>.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2'
- Cluster login name: -n admin
- Cluster login password -p 'password'
```

Si vous avez Beeline installé localement et que vous vous connectez sur un réseau virtuel Azure, utilisez les paramètres suivants :

```
Connection string: -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Pour rechercher le nom de domaine complet d’un nœud principal, utilisez les informations contenues dans le document Gérer des clusters HDInsight à l’aide de l’API REST d’Ambari.

## <a name="users-of-hdinsight-clusters-with-esp"></a>Utilisateurs de clusters HDInsight avec ESP

Pendant la création d’un cluster HDInsight non ESP, deux comptes d’utilisateurs sont créés :

- **Administrateur Ambari** : ce compte est également appelé *utilisateur Hadoop* ou *utilisateur HTTP* . Ce compte peut être utilisé pour se connecter à Ambari à l’adresse `https://CLUSTERNAME.azurehdinsight.net`. Il peut également être utilisé pour exécuter des requêtes sur des vues Ambari, exécuter des travaux à l’aide d’outils externes (par exemple, PowerShell, Templeton, Visual Studio) et s’authentifier avec le pilote ODBC Hive et les outils décisionnels (par exemple, Excel, Power BI ou Tableau).

Un cluster HDInsight avec ESP compte trois nouveaux utilisateurs, en plus de l’administrateur Ambari.

- **Administrateur Ranger** :  ce compte est le compte administrateur Apache Ranger local. Il ne s’agit pas d’un utilisateur de domaine Active Directory. Ce compte peut être utilisé pour configurer des stratégies et d’autres administrateurs utilisateurs ou administrateurs délégués (pour que ces utilisateurs puissent gérer les stratégies). Par défaut, le nom d’utilisateur est *admin* et le mot de passe est le même que le mot de passe administrateur Ambari. Le mot de passe peut être mis à jour à partir de la page Paramètres dans Ranger.

- **Utilisateur de domaine administrateur de cluster** : ce compte est un utilisateur de domaine Active Directory désigné comme administrateur du cluster Hadoop, y compris Ambari et Ranger. Vous devez fournir les informations d’identification de cet utilisateur lors de la création du cluster. Cet utilisateur possède les privilèges suivants :
    - Joindre des machines au domaine et les placer dans l’unité d’organisation que vous spécifiez lors de la création du cluster.
    - Créer des principaux de service au sein de l’unité d’organisation que vous spécifiez lors de la création du cluster.
    - Créer des entrées DNS inversées.

    Notez que les autres utilisateurs d’Active Directory possèdent également ces privilèges.

    Il existe des points de terminaison au sein du cluster (par exemple, Templeton) qui ne sont pas gérés par Ranger et ne sont donc pas sécurisés. Ces points de terminaison sont verrouillés pour tous les utilisateurs, à l’exception de l’utilisateur de domaine administrateur du cluster.

- **Standard** : pendant la création du cluster, vous pouvez fournir plusieurs groupes Active Directory. Les utilisateurs de ces groupes sont synchronisés avec Ranger et Ambari. Ce sont des utilisateurs de domaine qui ont accès uniquement aux points de terminaison gérés par Ranger (par exemple, Hiveserver2). Les stratégies RBAC et les audits sont tous applicables à ces utilisateurs.

## <a name="roles-of-hdinsight-clusters-with-esp"></a>Rôles des clusters HDInsight avec ESP

Le pack Sécurité Entreprise HDInsight contient les rôles suivants :

- Administrateur de cluster
- Opérateur de cluster
- Administrateur de services
- Opérateur de service
- Utilisateur de cluster

**Pour voir les autorisations de ces rôles.**

1. Ouvrez l’interface utilisateur de gestion Ambari.  Reportez-vous à [Ouverture de l’interface utilisateur de gestion Ambari](#open-the-ambari-management-ui).
2. Dans le menu de gauche, sélectionnez **Rôles** .
3. Sélectionnez le point d’interrogation bleu pour afficher les autorisations :

    ![Autorisations des rôles HDInsight ESP](./media/apache-domain-joined-manage/hdinsight-domain-joined-roles-permissions.png)

## <a name="open-the-ambari-management-ui"></a>Ouverture de l’interface utilisateur de gestion Ambari

1. Accédez à `https://CLUSTERNAME.azurehdinsight.net/`, où CLUSTERNAME est le nom de votre cluster.
1. Connectez-vous à Ambari avec le nom d’utilisateur et le mot de passe du domaine de l’administrateur du cluster.
1. Sélectionnez le menu déroulant **admin** dans l’angle supérieur droit, puis sélectionnez **Manage Ambari** (Gérer Ambari).

    ![HDInsight ESP - Gérer Apache Ambari](./media/apache-domain-joined-manage/hdinsight-domain-joined-manage-ambari.png)

    L’interface utilisateur ressemble à ce qui suit :

    ![HDInsight ESP - Interface utilisateur de gestion d’Apache Ambari](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui.png)

## <a name="list-the-domain-users-synchronized-from-your-active-directory"></a>Énumération des utilisateurs du domaine synchronisés à partir d’Active Directory

1. Ouvrez l’interface utilisateur de gestion Ambari.  Reportez-vous à [Ouverture de l’interface utilisateur de gestion Ambari](#open-the-ambari-management-ui).
2. Dans le menu de gauche, sélectionnez **Users** (Utilisateurs). Vous devriez voir tous les utilisateurs synchronisés à partir d’Active Directory sur le cluster HDInsight.

    ![Énumération des utilisateurs de l’interface utilisateur de gestion Ambari pour HDInsight ESP](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-users.png)

## <a name="list-the-domain-groups-synchronized-from-your-active-directory"></a>Énumération des groupes du domaine synchronisés à partir d’Active Directory

1. Ouvrez l’interface utilisateur de gestion Ambari.  Reportez-vous à [Ouverture de l’interface utilisateur de gestion Ambari](#open-the-ambari-management-ui).
2. Dans le menu de gauche, sélectionnez **Groups** (Groupes). Vous devriez voir tous les groupes synchronisés à partir d’Active Directory sur le cluster HDInsight.

    ![Énumération des groupes de l’interface utilisateur de gestion Ambari pour HDInsight ESP](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-groups.png)

## <a name="configure-hive-views-permissions"></a>Configuration des autorisations des affichages Hive

1. Ouvrez l’interface utilisateur de gestion Ambari.  Reportez-vous à [Ouverture de l’interface utilisateur de gestion Ambari](#open-the-ambari-management-ui).
2. Dans le menu de gauche, sélectionnez **Views** (Vues).
3. Sélectionnez **HIVE** pour afficher les détails.

    ![Affichages Hive de l’interface utilisateur de gestion Ambari pour HDInsight ESP](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views.png)

4. Sélectionnez le lien **Hive View** (Affichage Hive) pour configurer les affichages Hive.
5. Faites défiler jusqu’à la section **Autorisations** .

    ![Autorisations de configuration des affichages Hive de l’interface utilisateur de gestion Ambari pour HDInsight ESP](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views-permissions.png)

6. Sélectionnez **Add User** (Ajouter un utilisateur) ou **Add Group** (Ajouter un groupe), puis spécifiez les utilisateurs ou les groupes qui peuvent utiliser les affichages Hive.

## <a name="configure-users-for-the-roles"></a>Configuration des utilisateurs pour les rôles

 Pour afficher la liste des rôles et de leurs autorisations, consultez Rôles des clusters HDInsight avec ESP.

1. Ouvrez l’interface utilisateur de gestion Ambari.  Reportez-vous à [Ouverture de l’interface utilisateur de gestion Ambari](#open-the-ambari-management-ui).
2. Dans le menu de gauche, sélectionnez **Rôles** .
3. Sélectionnez **Add User** (Ajouter un utilisateur) ou **Add Group** (Ajouter un groupe) pour affecter des utilisateurs et des groupes aux différents rôles.

## <a name="next-steps"></a>Étapes suivantes

- Pour configurer un cluster HDInsight avec le Pack Sécurité Entreprise, consultez [Configurer des clusters HDInsight avec ESP](./apache-domain-joined-configure-using-azure-adds.md).
- Pour configurer des stratégies Hive et exécuter des requêtes Hive, consultez [Configurer des stratégies Apache Hive pour des clusters HDInsight avec ESP](apache-domain-joined-run-hive.md).