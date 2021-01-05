---
title: Autoriser les utilisateurs pour Ambari Views - Azure HDInsight
description: Guide pratique pour gérer les autorisations des utilisateurs et des groupes pour les clusters HDInsight avec le Pack Sécurité Entreprise activé.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: b1cd6d93ff45d7fb40ae7ca1874343486bd0b8cb
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92547926"
---
# <a name="authorize-users-for-apache-ambari-views"></a>Autoriser les utilisateurs à accéder à Apache Ambari Views

Les [clusters HDInsight avec le Pack Sécurité Entreprise (PSE) activé](./domain-joined/hdinsight-security-overview.md) offrent des fonctionnalités d’entreprise, notamment l’authentification par le biais d’Azure Active Directory. Vous pouvez synchroniser [de nouveaux utilisateurs](hdinsight-sync-aad-users-to-cluster.md) ajoutés aux groupes Azure AD qui ont accès au cluster, permettant ainsi à ces utilisateurs d’effectuer certaines actions. L’utilisation d’utilisateurs, de groupes et d’autorisations dans [Apache Ambari](https://ambari.apache.org/) est prise en charge à la fois pour les clusters HDInsight Pack Sécurité Entreprise et pour les clusters HDInsight standard.

Les utilisateurs Active Directory peuvent se connecter aux nœuds de cluster à l’aide de leurs informations d’identification de domaine. Ils ont également la possibilité d’utiliser ces identifiants pour authentifier les interactions du cluster auprès d’autres points de terminaison approuvés, comme [Hue](https://gethue.com/), Ambari Views, ODBC, JDBC, PowerShell et les API REST.

> [!WARNING]  
> Ne modifiez pas le mot de passe pour l’agent de surveillance Ambari (hdinsightwatchdog) sur votre cluster HDInsight basé sur Linux. La modification du mot de passe élimine la possibilité d’utiliser les actions de script ou d’effectuer des opérations de mise à l’échelle sur votre cluster.

Si ce n’est déjà fait, suivez [ces instructions](./domain-joined/apache-domain-joined-configure-using-azure-adds.md) pour provisionner un nouveau cluster PSE.

## <a name="access-the-ambari-management-page"></a>Accéder à la page de gestion Ambari

Pour ouvrir la **page de gestion Ambari** dans l’ [interface utilisateur web d’Apache Ambari](hdinsight-hadoop-manage-ambari.md), accédez à `https://CLUSTERNAME.azurehdinsight.net`. Entrez le nom d’utilisateur de l’administrateur du cluster et le mot de passe que vous avez défini lors de la création du cluster. Ensuite, sur le tableau de bord Ambari, sélectionnez **Gérer Ambari** sous le menu **admin** :

![Gestion du tableau de bord Apache ambari](./media/hdinsight-authorize-users-to-ambari/manage-apache-ambari.png)

## <a name="add-users"></a>Ajouter des utilisateurs

### <a name="add-users-through-the-portal"></a>Ajouter des utilisateurs par le biais du portail

1. Dans la page de gestion, sélectionnez **Utilisateurs** .

    ![Utilisateurs de la page de gestion Apache Ambari](./media/hdinsight-authorize-users-to-ambari/apache-ambari-management-page-users.png)

1. Sélectionnez **+ Créer un utilisateur local** .

1. Fournissez un **Nom d’utilisateur** et un **Mot de passe** . Sélectionnez **Enregistrer** .

### <a name="add-users-through-powershell"></a>Ajouter des utilisateurs par le biais de PowerShell

Modifiez les variables ci-dessous en remplaçant `CLUSTERNAME`, `NEWUSER` et `PASSWORD` par les valeurs appropriées.

```powershell
# Set-ExecutionPolicy Unrestricted

# Begin user input; update values
$clusterName="CLUSTERNAME"
$user="NEWUSER"
$userpass='PASSWORD'
# End user input

$adminCredentials = Get-Credential -UserName "admin" -Message "Enter admin password"

$clusterName = $clusterName.ToLower()
$createUserUrl="https://$($clusterName).azurehdinsight.net/api/v1/users"

$createUserBody=@{
    "Users/user_name" = "$user"
    "Users/password" = "$userpass"
    "Users/active" = "$true"
    "Users/admin" = "$false"
} | ConvertTo-Json

# Create user
$statusCode =
Invoke-WebRequest `
    -Uri $createUserUrl `
    -Credential $adminCredentials `
    -Method POST `
    -Headers @{"X-Requested-By" = "ambari"} `
    -Body $createUserBody | Select-Object -Expand StatusCode

if ($statusCode -eq 201) {
    Write-Output "User is created: $user"
}
else
{
    Write-Output 'User is not created'
    Exit
}

$grantPrivilegeUrl="https://$($clusterName).azurehdinsight.net/api/v1/clusters/$($clusterName)/privileges"

$grantPrivilegeBody=@{
    "PrivilegeInfo" = @{
        "permission_name" = "CLUSTER.USER"
        "principal_name" = "$user"
        "principal_type" = "USER"
    }
} | ConvertTo-Json

# Grant privileges
$statusCode =
Invoke-WebRequest `
    -Uri $grantPrivilegeUrl `
    -Credential $adminCredentials `
    -Method POST `
    -Headers @{"X-Requested-By" = "ambari"} `
    -Body $grantPrivilegeBody | Select-Object -Expand StatusCode

if ($statusCode -eq 201) {
    Write-Output 'Privilege is granted'
}
else
{
    Write-Output 'Privilege is not granted'
    Exit
}

Write-Host "Pausing for 100 seconds"
Start-Sleep -s 100

$userCredentials = "$($user):$($userpass)"
$encodedUserCredentials = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($userCredentials))
$zookeeperUrlHeaders = @{ Authorization = "Basic $encodedUserCredentials" }
$getZookeeperurl="https://$($clusterName).azurehdinsight.net/api/v1/clusters/$($clusterName)/services/ZOOKEEPER/components/ZOOKEEPER_SERVER"

# Perform query with new user
$zookeeperHosts =
Invoke-WebRequest `
    -Uri $getZookeeperurl `
    -Method Get `
    -Headers $zookeeperUrlHeaders

Write-Output $zookeeperHosts
```

### <a name="add-users-through-curl"></a>Ajouter des utilisateurs par le biais de Curl

Modifiez les variables ci-dessous en remplaçant `CLUSTERNAME`, `ADMINPASSWORD`, `NEWUSER` et `USERPASSWORD` par les valeurs appropriées. Le script est conçu pour être exécuté avec bash. De légères modifications seraient nécessaires pour une invite de commandes Windows.

```bash
export clusterName="CLUSTERNAME"
export adminPassword='ADMINPASSWORD'
export user="NEWUSER"
export userPassword='USERPASSWORD'

# create user
curl -k -u admin:$adminPassword -H "X-Requested-By: ambari" -X POST \
-d "{\"Users/user_name\":\"$user\",\"Users/password\":\"$userPassword\",\"Users/active\":\"true\",\"Users/admin\":\"false\"}" \
https://$clusterName.azurehdinsight.net/api/v1/users

echo "user created: $user"

# grant permissions
curl -k -u admin:$adminPassword -H "X-Requested-By: ambari" -X POST \
-d '[{"PrivilegeInfo":{"permission_name":"CLUSTER.USER","principal_name":"'$user'","principal_type":"USER"}}]' \
https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/privileges

echo "Privilege is granted"

echo "Pausing for 100 seconds"
sleep 10s

# perform query using new user account
curl -k -u $user:$userPassword -H "X-Requested-By: ambari" \
-X GET "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER"
```

## <a name="grant-permissions-to-apache-hive-views"></a>Accorder des autorisations sur les affichages Apache Hive

Ambari s’accompagne d’instances d’affichages pour [Apache Hive](https://hive.apache.org/) et [Apache Tez](https://tez.apache.org/), entre autres. Pour accorder l’accès à une ou plusieurs instances d’affichages Hive, accédez à la **page de gestion Ambari** .

1. Sur la page de gestion, sélectionnez le lien **Affichages** sous le titre de menu **Affichages** , à gauche.

    ![Liens de l’affichage des vues Apache Ambari](./media/hdinsight-authorize-users-to-ambari/apache-ambari-views-link.png)

2. Sur la page Affichages, développez la ligne **HIVE** . Par défaut, un affichage Hive est créé lorsque lors de l’ajout au cluster du service Hive. Vous pouvez également créer d’autres instances d’affichages Hive selon vos besoins. Sélectionnez un affichage Hive :

    ![Vues HDInsight – Vue Apache Hive](./media/hdinsight-authorize-users-to-ambari/views-apache-hive-view.png)

3. Allez en bas de la page Affichage. Sous la section *Autorisations* , vous avez deux possibilités pour accorder des autorisations sur l’affichage à des utilisateurs de domaine :

**Accorder une autorisation à ces utilisateurs** ![Accorder une autorisation à ces utilisateurs](./media/hdinsight-authorize-users-to-ambari/hdi-add-user-to-view.png)

**Accorder une autorisation à ces groupes** ![Accorder une autorisation à ces groupes](./media/hdinsight-authorize-users-to-ambari/add-group-to-view-permission.png)

1. Pour ajouter un utilisateur, appuyez sur le bouton **Ajouter un utilisateur** .

   * Commencez à taper le nom de l’utilisateur : une liste déroulante de noms déjà définis apparaît.

     ![La saisie semi-automatique des utilisateurs Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-user-autocomplete.png)

   * Sélectionnez ou terminez de taper le nom d’utilisateur. Pour ajouter ce nom d’utilisateur comme nouvel utilisateur, appuyez sur le bouton **Nouveau** .

   * Pour enregistrer vos modifications, cochez la **case bleue** .

     ![Octroi d’autorisations d’utilisateur Apache Ambari](./media/hdinsight-authorize-users-to-ambari/user-entered-permissions.png)

1. Pour ajouter un groupe, appuyez sur le bouton **Ajouter un groupe** .

   * Commencez à taper le nom du groupe. Le processus de sélection d’un nom de groupe existant ou d’ajout d’un nouveau groupe est le même que pour les utilisateurs.
   * Pour enregistrer vos modifications, cochez la **case bleue** .

     ![Octroi d’autorisations Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-group-entered.png)

Il est utile d’ajouter directement des utilisateurs à un affichage afin de pouvoir affecter à un utilisateur l’autorisation de l’utiliser sans qu’il soit un membre d’un groupe disposant d’autorisations supplémentaires. Pour réduire la quantité de traitement administratif, il est parfois plus simple d’affecter des autorisations à des groupes.

## <a name="grant-permissions-to-apache-tez-views"></a>Accorder des autorisations sur les affichages Apache Tez

Les instances d’affichages [Apache Tez](https://tez.apache.org/) autorisent les utilisateurs à surveiller et à déboguer toutes les tâches Tez, soumises par des requêtes [Apache Hive](https://hive.apache.org/) et des scripts [Apache Pig](https://pig.apache.org/). Par défaut, une instance d’affichage Tez est créée lors de la configuration du cluster.

Pour affecter des utilisateurs et des groupes à une instance d’affichage Tez, développez la ligne **TEZ** sur la page Affichages, comme nous l’avons décrit précédemment.

![Vues HDInsight – Vue Apache Tez](./media/hdinsight-authorize-users-to-ambari/views-apache-tez-view.png)

Pour ajouter des utilisateurs ou des groupes, répétez les étapes 3, 4 et 5 de la section précédente.

## <a name="assign-users-to-roles"></a>Affecter des utilisateurs aux rôles

Il existe cinq rôles de sécurité pour les utilisateurs et les groupes, listés par ordre décroissant d’autorisations d’accès :

* Administrateur de cluster
* Opérateur de cluster
* Administrateur de services
* Opérateur de service
* Utilisateur de cluster

Pour gérer les rôles, accédez à la **page de gestion Ambari** , puis sélectionnez le lien **Rôles** dans le groupe de menus *Clusters* , à gauche.

![Liens du menu Rôles Apache Ambari](./media/hdinsight-authorize-users-to-ambari/cluster-roles-menu-link.png)

Pour afficher la liste des autorisations accordées à chaque rôle, cliquez sur le point d’interrogation bleu près de l’en-tête du tableau **Rôles** , sur la page Rôles.

![Autorisations - Liens du menu Rôles Apache Ambari](./media/hdinsight-authorize-users-to-ambari/roles-menu-permissions.png "Autorisations - Liens du menu Rôles Apache Ambari")

Cette page présente deux affichages différents qui permettent de gérer les rôles des utilisateurs et des groupes : le mode Bloc et le mode Liste.

### <a name="block-view"></a>Mode Bloc

Le mode Bloc affiche chaque rôle dans sa propre ligne, et propose les options **Affecter des rôles à ces utilisateurs** et **Affecter des rôles à ces groupes** décrites précédemment.

![Vue de bloc de rôles Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-roles-block-view.png)

### <a name="list-view"></a>Mode Liste

Le mode Liste propose des fonctionnalités de modification rapide dans deux catégories : les utilisateurs et les groupes.

* La catégorie Utilisateurs du mode Liste affiche la liste de tous les utilisateurs, ce qui permet de sélectionner un rôle pour chaque utilisateur dans la liste déroulante.

    ![Vue de liste de rôles Apache Ambari – Utilisateurs](./media/hdinsight-authorize-users-to-ambari/roles-list-view-users.png)

* La catégorie Groupes du mode Liste affiche tous les groupes, ainsi que le rôle affecté à chacun. Dans notre exemple, la liste des groupes est synchronisée à partir des groupes Azure Active Directory spécifiés dans la propriété **Accéder au groupe d’utilisateurs** des paramètres de domaine du cluster. Consultez [Créer un cluster HDInsight avec le Pack Sécurité Entreprise activé](./domain-joined/apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp).

    ![Vue de liste de rôles Apache Ambari – Groupes](./media/hdinsight-authorize-users-to-ambari/roles-list-view-groups.png)

    Dans l’image ci-dessus, le groupe « hiveusers » a le rôle *Utilisateur du cluster* . Il s’agit d’un rôle en lecture seule qui permet aux utilisateurs de ce groupe d’afficher, mais non de modifier, les configurations de service et les mesures du cluster.

## <a name="log-in-to-ambari-as-a-view-only-user"></a>Ouvrir une session Ambari en tant qu’utilisateur avec accès en lecture seule

Nous avons affecté à notre utilisateur de domaine Azure AD « hiveuser1 » des autorisations sur les affichages Hive et Tez. Si l’on lance l’interface utilisateur Web d’Ambari et que l’on entre les informations d’identification de domaine de cet utilisateur (nom d’utilisateur Azure AD au format e-mail et mot de passe), il est redirigé vers la page Ambari Views. À ce stade, l’utilisateur peut sélectionner l’affichage de son choix, à condition qu’il soit accessible. Il ne peut pas visiter les autres parties du site, notamment le tableau de bord, les services, les hôtes, les alertes les pages d’administration.

![Utilisateur Apache Ambari avec des vues uniquement](./media/hdinsight-authorize-users-to-ambari/ambari-user-views-only.png)

## <a name="log-in-to-ambari-as-a-cluster-user"></a>Ouvrir une session Ambari en tant qu’utilisateur du cluster

Nous avons affecté le rôle *Utilisateur du cluster* à notre utilisateur de domaine Azure AD « hiveuser2 ». Ce rôle peut accéder au tableau de bord et à tous les éléments de menu. Un utilisateur du cluster a moins d’options autorisées qu’un administrateur. Par exemple, hiveuser2 peut afficher les configurations de chacun des services, mais il ne peut pas les modifier.

![Affichage du tableau de bord Apache Ambari](./media/hdinsight-authorize-users-to-ambari/user-cluster-user-role.png)

## <a name="next-steps"></a>Étapes suivantes

* [Configurer des stratégies Apache Hive dans HDInsight avec le Pack Sécurité Entreprise](./domain-joined/apache-domain-joined-run-hive.md)
* [Gérer des clusters HDInsight avec le Pack Sécurité Entreprise activé](./domain-joined/apache-domain-joined-manage.md)
* [Utiliser l’affichage Apache Hive avec Apache Hadoop dans HDInsight](hadoop/apache-hadoop-use-hive-ambari-view.md)
* [Synchroniser les utilisateurs Azure AD sur le cluster](hdinsight-sync-aad-users-to-cluster.md)
* [Gérer des clusters HDInsight à l’aide de l’API REST d’Apache Ambari](./hdinsight-hadoop-manage-ambari-rest-api.md)