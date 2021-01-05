---
title: Synchronisation LDAP dans Ranger et Apache Ambari dans Azure HDInsight
description: Cet article aborde la synchronisation LDAP dans Ranger et Ambari, et fournit des indications générales.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: 0779ac261fbb4ee91bf63021bb0cc685a371c2b2
ms.sourcegitcommit: bbd66b477d0c8cb9adf967606a2df97176f6460b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93234067"
---
# <a name="ldap-sync-in-ranger-and-apache-ambari-in-azure-hdinsight"></a>Synchronisation LDAP dans Ranger et Apache Ambari dans Azure HDInsight

Les clusters du Pack Sécurité Entreprise (ESP) HDInsight utilisent Ranger pour obtenir une autorisation. Apache Ambari et Ranger synchronisent tous les deux les utilisateurs et les groupes de manière indépendante et fonctionnent un peu différemment. Cet article est destiné à traiter la synchronisation LDAP dans Ranger et Ambari.

## <a name="general-guidelines"></a>Recommandations générales

* Déployez toujours les clusters avec un ou plusieurs groupes.
* Si vous souhaitez utiliser plus de groupes dans le cluster, vérifiez s’il est judicieux de mettre à jour les appartenances aux groupes dans Azure Active Directory (Azure AD).
* Si vous souhaitez modifier les groupes de clusters, vous pouvez modifier les filtres de synchronisation à l’aide d’Ambari.
* Toutes les modifications apportées à l’appartenance au groupe dans Azure AD sont répercutées dans le cluster lors des synchronisations suivantes. Vous devez d’abord synchroniser les modifications sur Azure AD Domain Services (Azure AD DS), puis sur les clusters.
* Les clusters HDInsight utilisent Samba/Winbind pour projeter les appartenances aux groupes sur les nœuds de cluster.
* Les membres du groupe sont synchronisés de manière transitive (tous les sous-groupes et leurs membres) à la fois avec Ambari et Ranger. 

## <a name="users-are-synced-separately"></a>Les utilisateurs sont synchronisés séparément

 * Ambari et Ranger ne partagent pas la base de données utilisateur car ils ont deux utilités différentes. 
   * Si un utilisateur doit utiliser l’interface utilisateur Ambari, il doit être synchronisé avec Ambari. 
   * Si l’utilisateur n’est pas synchronisé avec Ambari, l’interface utilisateur/l’API Ambari le rejettera, mais d’autres parties du système fonctionneront (celles-ci sont protégées par Ranger ou Resource Manager, et non pas par Ambari).
   * Pour inclure des utilisateurs ou des groupes dans les stratégies Ranger, les principaux doivent être synchronisés de manière explicite dans Ranger.

## <a name="ambari-user-sync-and-configuration"></a>Configuration et synchronisation d’utilisateurs Ambari

À partir des nœuds principaux, un travail Cron, `/opt/startup_scripts/start_ambari_ldap_sync.py`, est exécuté toutes les heures pour planifier la synchronisation des utilisateurs. Le travail Cron appelle les API REST Ambari pour effectuer la synchronisation. Le script soumet une liste d’utilisateurs et de groupes à synchroniser. (Étant donné que les utilisateurs peuvent ne pas appartenir aux groupes spécifiés, les deux sont spécifiés individuellement.) Ambari synchronise sAMAccountName en tant que nom d’utilisateur et tous les membres du groupe, de manière transitive.

Les journaux doivent se trouver dans `/var/log/ambari-server/ambari-server.log`. Pour plus d’informations, consultez [Configurer le niveau de journalisation d’Ambari](https://docs.cloudera.com/HDPDocuments/Ambari-latest/administering-ambari/content/amb_configure_ambari_logging_level.html).

Dans les clusters Data Lake, le hook de publication de créations d’utilisateur est utilisé pour créer les dossiers de base des utilisateurs synchronisés et ils sont définis comme propriétaires de ces dossiers. Si l’utilisateur n’est pas synchronisé correctement avec Ambari, il peut rencontrer des échecs dans l’exécution des tâches, car le dossier d’accueil pourrait ne pas être configuré correctement.

## <a name="ranger-user-sync-and-configuration"></a>Configuration et synchronisation d’utilisateurs Ranger

Ranger dispose d’un moteur de synchronisation intégré qui s’exécute toutes les heures pour synchroniser les utilisateurs. Il ne partage pas la base de données utilisateur avec Ambari. HDInsight configure le filtre de recherche pour synchroniser l’utilisateur administrateur, l’utilisateur de surveillance et les membres du groupe spécifié lors de la création du cluster. Les membres du groupe seront synchronisés de manière transitive :

1. Désactivez la synchronisation incrémentielle.
1. Activez le mappage de synchronisation de groupes d’utilisateurs.
1. Spécifiez le filtre de recherche permettant d’inclure les membres des groupes transitifs.
1. Synchronisez l’attribut sAMAccountName pour les utilisateurs et l’attribut de nom des groupes.

### <a name="group-or-incremental-sync"></a>Synchronisation incrémentielle ou de groupes

Ranger prend en charge une option de synchronisation des groupes, mais celle-ci fonctionne comme une intersection avec le filtre utilisateur, et non comme une union entre les appartenances aux groupes et le filtre utilisateur. Un cas d’utilisation typique pour le filtre de synchronisation de groupes dans Ranger est : - filtre de groupe : (dn=clusteradmingroup), filtre d’utilisateur : (city=seattle).

La synchronisation incrémentielle fonctionne uniquement pour les utilisateurs qui sont déjà synchronisés (la première fois). Elle ne synchronisera pas les nouveaux utilisateurs ajoutés aux groupes après la synchronisation initiale.

### <a name="update-ranger-sync-filter"></a>Mettre à jour le filtre de synchronisation de Ranger

Le filtre LDAP se trouve dans l’interface utilisateur Ambari, sous la section de configuration de la synchronisation des utilisateurs Ranger. Le filtre existant se présente sous la forme `(|(userPrincipalName=bob@contoso.com)(userPrincipalName=hdiwatchdog-core01@CONTOSO.ONMICROSOFT.COM)(memberOf:1.2.840.113556.1.4.1941:=CN=hadoopgroup,OU=AADDC Users,DC=contoso,DC=onmicrosoft,DC=com))`. Veillez à ajouter un prédicat à la fin et à tester le filtre à l’aide de la commande de recherche `net ads`, de ldp.exe ou de quelque chose de semblable.

## <a name="ranger-user-sync-logs"></a>Journaux de synchronisation des utilisateurs Ranger

La synchronisation des utilisateurs Ranger peut se produire en dehors de l’un ou l’autre des nœuds principaux. Les journaux se trouvent dans `/var/log/ranger/usersync/usersync.log`. Pour augmenter le niveau de détail des journaux, effectuez les étapes suivantes :

1. Connectez-vous à Ambari.
1. Accédez à la section de configuration de Ranger.
1. Accédez à la section **usersync-log4j** avancée.
1. Modifiez `log4j.rootLogger` en niveau `DEBUG`. (Après l’avoir modifié, il doit ressembler à `log4j.rootLogger = DEBUG,logFile,FilterLog`.)
1. Enregistrez la configuration, puis redémarrez Ranger.

## <a name="known-issues-with-ranger-user-sync"></a>Problèmes connus avec la synchronisation des utilisateurs de Ranger
* Si le nom du groupe contient des caractères Unicode, la synchronisation Ranger ne parvient pas à synchroniser cet objet. Si un utilisateur appartient à un groupe qui possède des caractères internationaux, Ranger synchronise l’appartenance à un groupe partiel.
* Le nom d’utilisateur (sAMAccountName) et le nom de groupe (nom) doivent comporter 20 caractères au maximum. Si le nom du groupe est plus long, l’utilisateur sera traité comme s’il n’appartenait pas au groupe lors du calcul des autorisations.

## <a name="next-steps"></a>Étapes suivantes

* [Problèmes d’authentification dans Azure HDInsight](./domain-joined-authentication-issues.md)
* [Synchroniser les utilisateurs Azure AD vers un cluster HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
