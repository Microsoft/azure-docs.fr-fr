---
title: Broker d’ID Azure HDInsight
description: Découvrez le broker d’ID Azure HDInsight pour simplifier l’authentification pour les clusters Apache Hadoop joints à un domaine.
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 11/03/2020
ms.openlocfilehash: c6bc5ca748a35b17c61d314e96f7284d30e7fc3b
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338127"
---
# <a name="azure-hdinsight-id-broker-hib"></a>Broker d’ID Azure HDInsight

Cet article explique comment configurer et utiliser la fonctionnalité de broker d’ID Azure HDInsight. Vous pouvez utiliser cette fonctionnalité pour bénéficier de l’authentification OAuth moderne sur Apache Ambari tout en disposant de la mise en œuvre de l’authentification multifacteur sans nécessiter de hachages de mots de passe hérités dans Azure Active Directory Domain Services (Azure AD DS).

## <a name="overview"></a>Vue d’ensemble

Le broker d’ID HDInsight simplifie les configurations d’authentification complexes dans les scénarios suivants :

* Votre organisation s’appuie sur une fédération pour authentifier les utilisateurs afin d’accéder aux ressources du cloud. Auparavant, pour utiliser des clusters Pack Sécurité Entreprise HDInsight, vous deviez activer la synchronisation de hachage de mot de passe à partir de votre environnement local pour Azure Active Directory (Azure AD). Cette exigence peut être difficile ou indésirable pour certaines organisations.
* Votre organisation souhaite mettre en œuvre l’authentification multifacteur pour l’accès web ou l’accès HTTP à Apache Ambari et d’autres ressources de cluster.

Le broker d’ID HDInsight fournit l’infrastructure d’authentification qui permet la transition de protocole d’OAuth (moderne) vers Kerberos (hérité) sans avoir à synchroniser les hachages de mots de passe sur Azure AD DS. Cette infrastructure est constituée de composants s’exécutant sur une machine virtuelle Windows Server sur laquelle le nœud de broker d’ID HDInsight est activé, ainsi que les nœuds de passerelle de cluster.

Utilisez le tableau suivant pour déterminer la meilleure option d’authentification en fonction des besoins de votre organisation.

|Options d’authentification |Configuration de HDInsight | Facteurs à prendre en considération |
|---|---|---|
| Entièrement OAuth | Pack Sécurité Entreprise + broker d’ID HDInsight | Option la plus sécurisée. (L’authentification multifacteur est prise en charge.) La synchronisation de hachage de mot de passe n’est *pas* obligatoire. Pas d’accès ssh/kinit/keytab pour les comptes locaux, qui n’ont pas le hachage de mot de passe dans Azure AD DS. Les comptes cloud seulement peuvent toujours utiliser ssh/kinit/keytab. Accès web à Ambari via OAuth. Nécessite la mise à jour des applications héritées (par exemple, JDBC/ODBC) pour prendre en charge OAuth.|
| OAuth + authentification de base | Pack Sécurité Entreprise + broker d’ID HDInsight | Accès web à Ambari via OAuth. Les applications héritées continuent d’utiliser l’authentification de base. L’authentification multifacteur doit être désactivée pour l’accès à l’authentification de base. La synchronisation de hachage de mot de passe n’est *pas* obligatoire. Pas d’accès ssh/kinit/keytab pour les comptes locaux, qui n’ont pas le hachage de mot de passe dans Azure AD DS. Les comptes cloud seulement peuvent toujours utiliser ssh/kinit. |
| Authentification de base complète | Pack Sécurité Entreprise | Similaire aux configurations locales. La synchronisation de hachage de mot de passe sur Azure AD DS est obligatoire. Les comptes locaux peuvent utiliser ssh/kinit ou keytab. L’authentification multifacteur doit être désactivée si le stockage de sauvegarde est Azure Data Lake Storage Gen2. |

Le diagramme suivant illustre le flux d’authentification OAuth moderne pour tous les utilisateurs, y compris les utilisateurs fédérés, après l’activation du broker d’ID HDInsight :

:::image type="content" source="media/identity-broker/identity-broker-architecture.png" alt-text="Diagramme illustrant le flux d’authentification avec le broker d’ID HDInsight.":::

Dans ce diagramme, le client (autrement dit, un navigateur ou une application) doit d’abord acquérir le jeton OAuth. Il présente ensuite le jeton à la passerelle dans une requête HTTP. Si vous êtes déjà connecté à d’autres services Azure, tels que le portail Azure, vous pouvez vous connecter à votre cluster HDInsight avec une expérience d’authentification unique.

Il existe toujours de nombreuses applications héritées qui prennent uniquement en charge l’authentification de base (autrement dit, le nom d’utilisateur et le mot de passe). Pour ces scénarios, vous pouvez toujours utiliser l’authentification de base HTTP pour vous connecter aux passerelles de cluster. Dans cette configuration, vous devez garantir la connectivité réseau entre les nœuds de passerelle et le point de terminaison Active Directory Federation Services (AD FS) pour garantir une ligne directe à partir des nœuds de passerelle.

Le diagramme suivant illustre le flux d’authentification de base pour les utilisateurs fédérés. Tout d’abord, la passerelle tente d’effectuer l’authentification à l’aide du [flux ROPC](../../active-directory/develop/v2-oauth-ropc.md). Si aucun hachage de mot de passe n’est synchronisé avec Azure AD, la passerelle retourne à la découverte du point de terminaison AD FS et termine l’authentification en accédant au point de terminaison AD FS.

:::image type="content" source="media/identity-broker/basic-authentication.png" alt-text="Diagramme illustrant l’architecture avec l’authentification de base.":::


## <a name="enable-hdinsight-id-broker"></a>Activer le broker d’ID HDInsight

Pour créer un cluster Pack Sécurité Entreprise avec le broker d’ID HDInsight activé :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Suivez les étapes de création de base pour un cluster Pack Sécurité Entreprise. Pour plus d’informations, consultez [Créer un cluster HDInsight avec le Pack Sécurité Entreprise](apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp).
1. Sélectionnez **Activer le broker d’ID HDInsight**.

La fonctionnalité de broker d’ID HDInsight ajoute une machine virtuelle supplémentaire au cluster. Cette machine virtuelle est le nœud du broker d’ID HDInsight et comprend des composants serveur pour prendre en charge l’authentification. Le nœud du broker d’ID HDInsight est joint au domaine Azure AD DS.

![Diagramme illustrant l’option permettant d’activer le broker d’ID HDInsight.](./media/identity-broker/identity-broker-enable.png)

### <a name="use-azure-resource-manager-templates"></a>Utiliser les modèles Azure Resource Manager

Si vous ajoutez un nouveau rôle appelé `idbrokernode` avec les attributs suivants au profil de calcul de votre modèle, le cluster est créé avec le nœud de broker d’ID HDInsight activé :

```json
.
.
.
"computeProfile": {
    "roles": [
        {
            "autoscale": null,
            "name": "headnode",
           ....
        },
        {
            "autoscale": null,
            "name": "workernode",
            ....
        },
        {
            "autoscale": null,
            "name": "idbrokernode",
            "targetInstanceCount": 2,
            "hardwareProfile": {
                "vmSize": "Standard_A2_V2"
            },
            "virtualNetworkProfile": {
                "id": "string",
                "subnet": "string"
            },
            "scriptActions": [],
            "dataDisksGroups": null
        }
    ]
}
.
.
.
```

Pour un exemple complet de modèle ARM, consultez celui qui est publié [ici](https://github.com/Azure-Samples/hdinsight-enterprise-security/tree/main/ESP-HIB-PL-Template).


## <a name="tool-integration"></a>Intégration d’outils

Les outils HDInsight sont mis à jour pour prendre en charge OAuth en mode natif. Utilisez ces outils pour un accès basé sur OAuth moderne aux clusters. Le [plug-in IntelliJ](../spark/apache-spark-intellij-tool-plugin.md#integrate-with-hdinsight-identity-broker-hib) HDInsight peut être utilisé pour les applications basées sur Java, comme Scala. [Les outils Spark et Hive pour Visual Studio Code](../hdinsight-for-vscode.md) peuvent être utilisés pour les tâches PySpark et Hive. Les outils prennent en charge les tâches par lots et interactives.

## <a name="ssh-access-without-a-password-hash-in-azure-ad-ds"></a>Accès SSH sans hachage de Mot de passe dans Azure AD DS

|Options SSH |Facteurs à prendre en considération |
|---|---|
| Compte de machine virtuelle local (par exemple, sshuser) | Vous avez fourni ce compte lors de la création du cluster. Il n’existe aucune authentification Kerberos pour ce compte. |
| Compte cloud seulement (par exemple, alice@contoso.onmicrosoft.com) | Le hachage de mot de passe est disponible dans Azure AD DS. L’authentification Kerberos est possible via SSH Kerberos. |
| Compte local (par exemple, alice@contoso.com) | L’authentification SSH Kerberos n’est possible que si un hachage de mot de passe est disponible dans Azure AD DS. Dans le cas contraire, cet utilisateur ne peut pas utiliser SSH pour accéder au cluster. |

Pour utiliser SSH avec une machine virtuelle jointe à un domaine ou pour exécuter la commande `kinit`, vous devez fournir un mot de passe. L’authentification SSH Kerberos requiert que le hachage soit disponible dans Azure AD DS. Si vous souhaitez utiliser SSH uniquement pour les scénarios d’administration, vous pouvez créer un compte cloud seulement et l’utiliser pour accéder au cluster via SSH. Les autres utilisateurs locaux peuvent toujours utiliser les outils Ambari ou HDInsight ou l’authentification de base HTTP sans que le hachage de mot de passe ne soit disponible dans Azure AD DS.

Si votre organisation ne synchronise pas les hachages de mot de passe avec Azure AD DS, il est recommandé de créer un utilisateur cloud seulement dans Azure AD. Ensuite, affectez-le en tant qu’administrateur du cluster lorsque vous créez le cluster et utilisez-le à des fins d’administration. Vous pouvez l’utiliser pour accéder à la racine des machines virtuelles via SSH.

Pour résoudre les problèmes d’authentification, consultez [ce guide](./domain-joined-authentication-issues.md).

## <a name="clients-using-oauth-to-connect-to-an-hdinsight-gateway-with-hdinsight-id-broker"></a>Clients utilisant OAuth pour se connecter à une passerelle HDInsight avec le broker d’ID HDInsight

Dans la configuration du broker d’ID HDInsight, les applications personnalisées et les clients qui se connectent à la passerelle peuvent être mis à jour pour acquérir, dans un premier temps, le jeton OAuth requis. Suivez les étapes décrites dans [ce document](../../storage/common/storage-auth-aad-app.md) pour obtenir le jeton avec les informations suivantes :

*   URI de ressource OAuth : `https://hib.azurehdinsight.net` 
*   AppId : 7865c1d2-f040-46cc-875f-831a1ef6a28a
*   Autorisation : (nom : Cluster.ReadWrite, id: 8f89faa0-ffef-4007-974d-4989b39ad77d)

Après l’acquisition du jeton OAuth, utilisez-le dans l’en-tête d’autorisation de la requête HTTP adressée à la passerelle de cluster (par exemple, https://<clustername>-int.azurehdinsight.net). Un exemple de commande curl pour l’API Apache Livy peut se présenter comme suit :
    
```bash
curl -k -v -H "Authorization: Bearer Access_TOKEN" -H "Content-Type: application/json" -X POST -d '{ "file":"wasbs://mycontainer@mystorageaccount.blob.core.windows.net/data/SparkSimpleTest.jar", "className":"com.microsoft.spark.test.SimpleFile" }' "https://<clustername>-int.azurehdinsight.net/livy/batches" -H "X-Requested-By:<username@domain.com>"
``` 

Pour utiliser Beeline et Livy, vous pouvez également suivre les exemples de code fournis [ici](https://github.com/Azure-Samples/hdinsight-enterprise-security/tree/main/HIB/HIBSamples) afin de configurer votre client de façon à utiliser OAuth et à se connecter au cluster.

## <a name="faq"></a>Questions fréquentes (FAQ)
### <a name="what-app-is-created-by-hdinsight-in-aad"></a>Quelle application est créée par HDInsight dans AAD ?
Pour chaque cluster, une application tierce est inscrite dans AAD avec l’URI du cluster comme identifierUri (par exemple `https://clustername.azurehdinsight.net`).

### <a name="why-are-users-prompted-for-consent-before-using-hib-enabled-clusters"></a>Pourquoi les utilisateurs sont-ils invités à donner leur consentement avant d’utiliser des clusters HIB ?
Dans AAD, le consentement est requis pour permettre à toutes les applications tierces d’authentifier les utilisateurs ou d’accéder aux données.

### <a name="can-the-consent-be-approved-programatically"></a>Le consentement peut-il être approuvé par programme ?
L’API Microsoft Graph vous permet d’automatiser le consentement. Consultez la [documentation de l’API](/graph/api/resources/oauth2permissiongrant?view=graph-rest-1.0). La séquence permettant d’automatiser le consentement est la suivante :

* Inscrire une application et lui accorder des autorisations Application.ReadWrite.All pour accéder à Microsoft Graph
* Après la création d’un cluster, interroger l’application de cluster en fonction de l’URI de l’identificateur
* Inscrire le consentement pour l’application

Lorsque le cluster est supprimé, HDInsight supprime l’application et il n’est pas nécessaire de nettoyer les consentements.

 


## <a name="next-steps"></a>Étapes suivantes

* [Configurer un cluster HDInsight avec le pack Sécurité Entreprise en utilisant Azure Active Directory Domain Services](apache-domain-joined-configure-using-azure-adds.md)
* [Synchroniser des utilisateurs Azure Active Directory vers un cluster HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
* [Surveiller les performances du cluster](../hdinsight-key-scenarios-to-monitor.md)