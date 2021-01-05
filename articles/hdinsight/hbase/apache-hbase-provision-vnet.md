---
title: Créer des clusters HBase dans un réseau virtuel - Azure
description: Prise en main de HBase dans Azure HDInsight Découvrez comment créer des clusters HDInsight HBase sur Azure Virtual Network.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/23/2019
ms.openlocfilehash: 82e3374491aa119d9985ea7ef31e180c920511d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86087739"
---
# <a name="create-apache-hbase-clusters-on-hdinsight-in-azure-virtual-network"></a>Créer des clusters Apache HBase sur HDInsight dans un réseau virtuel Azure

Découvrez comment créer des clusters Apache HBase Azure HDInsight dans un [réseau virtuel Azure](https://azure.microsoft.com/services/virtual-network/).

Avec l’intégration du réseau virtuel, les clusters Apache HBase peuvent être déployés sur le même réseau virtuel que vos applications pour permettre à celles-ci de communiquer directement avec HBase. Voici les avantages :

* Connectivité directe de l’application web aux nœuds du cluster HBase, ce qui permet les communications au moyen d’API d’appel de procédure distante (RPC) Java HBase.
* Amélioration des performances en évitant à votre trafic de transiter par plusieurs passerelles et équilibrages de charge.
* Capacité de traitement des informations critiques de façon plus sécurisée sans exposer de points de terminaison publics.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-apache-hbase-cluster-into-virtual-network"></a>Créer des clusters Apache HBase sur un réseau virtuel

Dans cette section, vous allez créer un cluster Apache HBase basé sur Linux avec un compte de stockage Azure dépendant dans un réseau virtuel Azure à l’aide d’un [modèle Azure Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md). Pour d'autres méthodes de création de cluster et comprendre les paramètres, consultez [Création de clusters HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Pour plus d’informations sur l’utilisation d’un modèle pour créer des clusters Apache Hadoop dans HDInsight, consultez [Création de clusters Apache Hadoop dans HDInsight à l’aide de modèles Azure Resource Manager](../hdinsight-hadoop-create-linux-clusters-arm-templates.md)

> [!NOTE]  
> Certaines propriétés sont codées en dur dans le modèle. Par exemple :
>
> * **Emplacement** : USA Est 2
> * **Version de cluster** : 3.6
> * **Nombre de nœuds worker du cluster** : 2
> * **Compte de stockage par défaut** : une chaîne unique
> * **Nom du réseau virtuel** : CLUSTERNAME-vnet
> * **Espace d’adressage du réseau virtuel** : 10.0.0.0/16
> * **Nom du sous-réseau** : subnet1
> * **Plage d’adresses de sous-réseau** : 10.0.0.0/24
>
> `CLUSTERNAME` est remplacé par le nom de cluster que vous fournissez lors de l’utilisation du modèle.

1. Sélectionnez l’image suivante pour ouvrir le modèle dans le portail Azure. Ce modèle se trouve dans les [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-linux-vnet/).

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux-vnet%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-provision-vnet/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

1. Dans la boîte de dialogue **Déploiement personnalisé**, sélectionnez **Modifier le modèle**.

1. À la ligne 165, remplacez la valeur `Standard_A3` par `Standard_A4_V2`. Ensuite, sélectionnez **Enregistrer**.

1. Remplissez le modèle restant avec les informations suivantes :

    |Propriété |Valeur |
    |---|---|
    |Abonnement|Sélectionnez un abonnement Azure utilisé pour créer le cluster HDInsight, le compte de stockage dépendant et le réseau virtuel Azure.|
    Resource group|Sélectionnez **Créer nouveau** et spécifiez un nouveau nom de groupe de ressources.|
    |Emplacement|Sélectionnez l’emplacement du groupe de ressources.|
    |Nom du cluster|Entrez un nom pour le cluster Hadoop à créer.|
    |Nom d’utilisateur et mot de passe de la connexion de cluster|Le nom d’utilisateur par défaut est **admin**. Fournissez un mot de passe.|
    |Nom d’utilisateur et mot de passe SSH|Le nom d’utilisateur par défaut est **sshuser**.  Fournissez un mot de passe.|

    Sélectionnez **J’accepte les termes et conditions mentionnés ci-dessus**.

1. Sélectionnez **Achat**. La création d’un cluster prend environ 20 minutes. Une fois le cluster créé, vous pouvez sélectionner le cluster dans le portail pour l’ouvrir.

Après avoir terminé ce tutoriel, vous devriez peut-être supprimer le cluster. Avec HDInsight, vos données sont stockées Azure Storage, pour que vous puissiez supprimer un cluster en toute sécurité s’il n’est pas en cours d’utilisation. Vous devez également payer pour un cluster HDInsight, même lorsque vous ne l’utilisez pas. Étant donné que les frais pour le cluster sont bien plus élevés que les frais de stockage, économique, mieux vaut supprimer les clusters lorsqu’ils ne sont pas utilisés. Pour obtenir des instructions sur la suppression d’un cluster, consultez [Gestion des clusters Apache Hadoop dans HDInsight au moyen du portail Azure](../hdinsight-administer-use-portal-linux.md#delete-clusters).

Pour commencer à utiliser votre nouveau cluster HBase, vous pouvez utiliser les procédures figurant dans la rubrique [Bien démarrer avec l’utilisation d’Apache HBase et d’Apache Hadoop dans HDInsight](./apache-hbase-tutorial-get-started-linux.md).

## <a name="connect-to-the-apache-hbase-cluster-using-apache-hbase-java-rpc-apis"></a>Se connecter au cluster Apache HBase avec les API RPC Java Apache HBase

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle IaaS dans le même réseau virtuel Azure et le même sous-réseau. Pour plus d’informations sur la création d’une machine virtuelle IaaS, consultez [Création d’une machine virtuelle exécutant Windows Server](../../virtual-machines/windows/quick-create-portal.md). Lors des étapes décrites dans ce document, vous devez utiliser les valeurs suivantes pour la configuration du réseau :

* **Réseau virtuel** : CLUSTERNAME-vnet
* **Sous-réseau** : subnet1

> [!IMPORTANT]  
> Remplacez `CLUSTERNAME` par le nom que vous avez utilisé lors de la création du cluster HDInsight dans les étapes précédentes.

À l’aide de ces valeurs, la machine virtuelle est placée dans le même réseau virtuel et le même sous-réseau que ceux du cluster HDInsight. Cette configuration leur permet de communiquer directement entre eux. Il existe un moyen de créer un cluster HDInsight avec un nœud de périphérie vide. Le nœud de périphérie peut servir à gérer le cluster.  Pour plus d’informations, consultez [Utiliser des nœuds de périmètre vides dans HDInsight](../hdinsight-apps-use-edge-node.md).

### <a name="obtain-fully-qualified-domain-name"></a>Obtenir le nom de domaine complet

Lorsque vous utilisez une application Java pour vous connecter à distance à HBase, vous devez utiliser le nom de domaine complet. Pour déterminer cela, vous devez obtenir le suffixe DNS propre à la connexion du cluster HBase. Pour ce faire, vous pouvez utiliser l’une des méthodes suivantes :

* Utilisez un navigateur web pour effectuer un appel [Apache Ambari](https://ambari.apache.org/) :

    Accédez à `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts?minimal_response=true`. Un fichier JSON avec les suffixes DNS est renvoyé.

* Utiliser le site Web Ambari :

    1. Accédez à `https://CLUSTERNAME.azurehdinsight.net`.
    2. Sélectionnez **Hôtes** dans le menu supérieur.

* Utiliser Curl pour effectuer des appels REST :

    ```bash
    curl -u <username>:<password> -k https://CLUSTERNAME.azurehdinsight.net/ambari/api/v1/clusters/CLUSTERNAME.azurehdinsight.net/services/hbase/components/hbrest
    ```

Dans les données JavaScript Object Notation (JSON) renvoyées, recherchez l’entrée « host_name ». Elle contient le nom de domaine complet des nœuds du cluster. Par exemple :

```
"host_name" : "hn0-hbaseg.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net"
```

La partie du nom de domaine commençant par le nom de cluster est le suffixe DNS. Par exemple : `hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

<!--
3.    Change the primary DNS suffix configuration of the virtual machine. This enables the virtual machine to automatically resolve the host name of the HBase cluster without explicit specification of the suffix. For example, the *workernode0* host name will be correctly resolved to workernode0 of the HBase cluster.

    To make the configuration change:

    1. RDP into the virtual machine.
    2. Open **Local Group Policy Editor**. The executable is gpedit.msc.
    3. Expand **Computer Configuration**, expand **Administrative Templates**, expand **Network**, and then click **DNS Client**.
    - Set **Primary DNS Suffix** to the value obtained in step 2:

        ![hdinsight.hbase.primary.dns.suffix](./media/apache-hbase-provision-vnet/hdi-primary-dns-suffix.png)
    4. Click **OK**.
    5. Reboot the virtual machine.
-->

### <a name="verify-communication-inside-virtual-network"></a>Vérifier la communication au sein du réseau virtuel

Pour vérifier que la machine virtuelle peut communiquer avec le cluster HBase, utilisez la commande `ping headnode0.<dns suffix>` à partir de la machine virtuelle. Par exemple : `ping hn0-hbaseg.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

Pour utiliser ces informations dans une application Java, vous pouvez suivre la procédure décrite dans la rubrique [Utilisation d’Apache Maven pour créer des applications Java utilisant Apache HBase avec HDInsight (Hadoop)](./apache-hbase-build-java-maven-linux.md) pour créer une application. Pour que l’application soit connectée à un serveur HBase distant, modifiez le fichier **hbase-site.xml** dans cet exemple afin d’utiliser le nom de domaine complet pour Zookeeper. Par exemple :

```xml
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper0.<dns suffix>,zookeeper1.<dns suffix>,zookeeper2.<dns suffix></value>
</property>
```

> [!NOTE]  
> Pour plus d’informations sur la résolution de noms dans des réseaux virtuels Azure, y compris sur la façon d’utiliser votre propre serveur DNS, consultez [Résolution de noms pour les machines virtuelles et les instances de rôle](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer un cluster Apache HBase. Pour plus d'informations, consultez les rubriques suivantes :

* [Prise en main de HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Utiliser des nœuds de périphérie vides dans HDInsight](../hdinsight-apps-use-edge-node.md)
* [Configurer la réplication Apache HBase dans HDInsight](apache-hbase-replication.md)
* [Créer des clusters Apache Hadoop dans HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
* [Bien démarrer avec l’utilisation d’Apache HBase et d’Apache Hadoop dans HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Présentation du réseau virtuel.](../../virtual-network/virtual-networks-overview.md)
