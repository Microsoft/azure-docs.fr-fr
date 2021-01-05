---
title: Se connecter à Kafka à l’aide de réseaux virtuels - Azure HDInsight
description: Découvrez comment vous connecter directement à Kafka sur HDInsight via un réseau virtuel Azure. Découvrez comment se connecter à Kafka à partir de clients de développement à l’aide d’une passerelle VPN, ou à partir de clients de votre réseau local à l’aide d’un périphérique de passerelle VPN.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-python
ms.date: 03/04/2020
ms.openlocfilehash: ca5bff7f8a14efc0916210296048e3731aec5a59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87873464"
---
# <a name="connect-to-apache-kafka-on-hdinsight-through-an-azure-virtual-network"></a>Se connecter à Apache Kafka sur HDInsight via un réseau virtuel Azure

Découvrez comment vous connecter directement à Apache Kafka sur HDInsight via un réseau virtuel Azure. Ce document fournit des informations sur la connexion à Kafka en utilisant les configurations suivantes :

* À partir des ressources d’un réseau local. Cette connexion est établie à l’aide d’un périphérique VPN (logiciel ou matériel) sur votre réseau local.
* À partir d’un environnement de développement à l’aide d’un client de logiciel VPN.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="architecture-and-planning"></a>Architecture et planification

HDInsight n’autorise pas la connexion directe à Kafka via l’internet public. Au lieu de cela, les clients de Kafka (producteurs et consommateurs) doivent utiliser une des méthodes de connexion suivantes :

* Exécutez le client dans le même réseau virtuel que Kafka sur HDInsight. Cette configuration est utilisée dans le document [Démarrer avec Apache Kafka sur HDInsight](apache-kafka-get-started.md). Le client s’exécute directement sur les nœuds du cluster HDInsight ou sur une autre machine virtuelle dans le même réseau.

* Connectez un réseau privé, notamment votre réseau local, au réseau virtuel. Cette configuration permet aux clients de votre réseau local de travailler directement avec Kafka. Pour activer cette configuration, effectuez les tâches suivantes :

  1. Créez un réseau virtuel.
  2. Créez une passerelle VPN qui utilise une configuration de site à site. La configuration utilisée dans ce document se connecte à un périphérique de passerelle VPN de votre réseau local.
  3. Créez un serveur DNS dans le réseau virtuel.
  4. Configurez le transfert entre les serveurs DNS de chaque réseau.
  5. Créez un cluster Kafka dans HDInsight sur le réseau virtuel.

     Pour plus d’informations, consultez la section [Se connecter à Apache Kafka à partir d’un réseau local](#on-premises).

* Connectez des machines individuelles au réseau virtuel à l’aide d’une passerelle VPN et d’un client VPN. Pour activer cette configuration, effectuez les tâches suivantes :

  1. Créez un réseau virtuel.
  2. Créez une passerelle VPN qui utilise une configuration de point à site. Cette configuration peut être utilisée avec les clients Windows et MacOS.
  3. Créez un cluster Kafka dans HDInsight sur le réseau virtuel.
  4. Configurez Kafka pour la publication d’adresses IP. Cette configuration permet au client de se connecter à l’aide du répartiteur d’adresse IP au lieu des noms de domaine.
  5. Téléchargez et utilisez le client VPN sur le système de développement.

     Pour plus d’informations, consultez la section [Se connecter à Apache Kafka avec un client VPN](#vpnclient).

     > [!WARNING]  
     > Cette configuration est recommandée uniquement à des fins de développement en raison des limitations suivantes :
     >
     > * Chaque client doit se connecter à l’aide d’un client de logiciel VPN.
     > * Le client VPN ne transmet pas de demandes de résolution de noms au réseau virtuel. Vous devez donc utiliser l’adressage IP pour communiquer avec Kafka. La communication IP nécessite une configuration supplémentaire sur le cluster Kafka.

Pour plus d’informations sur l’utilisation de HDInsight dans un réseau virtuel, voir [Plan a virtual network for Azure HDInsight clusters (Planifier un réseau virtuel pour des clusters Azure HDInsight)](../hdinsight-plan-virtual-network-deployment.md).

## <a name="connect-to-apache-kafka-from-an-on-premises-network"></a><a id="on-premises"></a> Se connecter à Apache Kafka à partir d’un réseau local

Pour créer un cluster Kafka qui communique avec votre réseau local, suivez les étapes du document [Connecter HDInsight à votre réseau local](./../connect-on-premises-network.md).

> [!IMPORTANT]  
> Lorsque vous créez le cluster HDInsight, sélectionnez __Kafka__ comme type de cluster.

Les étapes ci-après créent la configuration suivante :

* Réseau virtuel Azure
* Passerelle VPN de site à site
* Compte de stockage Azure (utilisé par HDInsight)
* Kafka sur HDInsight

Pour vérifier qu’un client Kafka peut se connecter localement au cluster, utilisez les étapes décrites dans la section [Exemple : client Python](#python-client).

## <a name="connect-to-apache-kafka-with-a-vpn-client"></a><a id="vpnclient"></a> Se connecter à Apache Kafka avec un client VPN

En suivant les étapes de cette section, vous pouvez créer la configuration ci-dessous :

* Réseau virtuel Azure
* Passerelle VPN de point à site
* Compte Stockage Azure (utilisé par HDInsight)
* Kafka sur HDInsight

1. Suivez les étapes du document [Utilisation des certificats auto-signés pour les connexions de point à site](../../vpn-gateway/vpn-gateway-certificates-point-to-site.md). Ce document crée les certificats nécessaires pour la passerelle.

2. Ouvrez une invite de commande PowerShell et utilisez le code suivant pour vous connecter à votre abonnement Azure :

    ```powershell
    Connect-AzAccount
    # If you have multiple subscriptions, uncomment to set the subscription
    #Select-AzSubscription -SubscriptionName "name of your subscription"
    ```

3. Utilisez le code suivant pour créer des variables qui contiennent des informations de configuration :

    ```powershell
    # Prompt for generic information
    $resourceGroupName = Read-Host "What is the resource group name?"
    $baseName = Read-Host "What is the base name? It is used to create names for resources, such as 'net-basename' and 'kafka-basename':"
    $location = Read-Host "What Azure Region do you want to create the resources in?"
    $rootCert = Read-Host "What is the file path to the root certificate? It is used to secure the VPN gateway."

    # Prompt for HDInsight credentials
    $adminCreds = Get-Credential -Message "Enter the HTTPS user name and password for the HDInsight cluster" -UserName "admin"
    $sshCreds = Get-Credential -Message "Enter the SSH user name and password for the HDInsight cluster" -UserName "sshuser"

    # Names for Azure resources
    $networkName = "net-$baseName"
    $clusterName = "kafka-$baseName"
    $storageName = "store$baseName" # Can't use dashes in storage names
    $defaultContainerName = $clusterName
    $defaultSubnetName = "default"
    $gatewaySubnetName = "GatewaySubnet"
    $gatewayPublicIpName = "GatewayIp"
    $gatewayIpConfigName = "GatewayConfig"
    $vpnRootCertName = "rootcert"
    $vpnName = "VPNGateway"

    # Network settings
    $networkAddressPrefix = "10.0.0.0/16"
    $defaultSubnetPrefix = "10.0.0.0/24"
    $gatewaySubnetPrefix = "10.0.1.0/24"
    $vpnClientAddressPool = "172.16.201.0/24"

    # HDInsight settings
    $HdiWorkerNodes = 4
    $hdiVersion = "3.6"
    $hdiType = "Kafka"
    ```

4. Utilisez le code suivant pour créer le réseau virtuel et le groupe de ressources Azure :

    ```powershell
    # Create the resource group that contains everything
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create the subnet configuration
    $defaultSubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -AddressPrefix $defaultSubnetPrefix
    $gatewaySubnetConfig = New-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -AddressPrefix $gatewaySubnetPrefix

    # Create the subnet
    New-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AddressPrefix $networkAddressPrefix `
        -Subnet $defaultSubnetConfig, $gatewaySubnetConfig

    # Get the network & subnet that were created
    $network = Get-AzVirtualNetwork -Name $networkName `
        -ResourceGroupName $resourceGroupName
    $gatewaySubnet = Get-AzVirtualNetworkSubnetConfig -Name $gatewaySubnetName `
        -VirtualNetwork $network
    $defaultSubnet = Get-AzVirtualNetworkSubnetConfig -Name $defaultSubnetName `
        -VirtualNetwork $network

    # Set a dynamic public IP address for the gateway subnet
    $gatewayPublicIp = New-AzPublicIpAddress -Name $gatewayPublicIpName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -AllocationMethod Dynamic
    $gatewayIpConfig = New-AzVirtualNetworkGatewayIpConfig -Name $gatewayIpConfigName `
        -Subnet $gatewaySubnet `
        -PublicIpAddress $gatewayPublicIp

    # Get the certificate info
    # Get the full path in case a relative path was passed
    $rootCertFile = Get-ChildItem $rootCert
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($rootCertFile)
    $certBase64 = [System.Convert]::ToBase64String($cert.RawData)
    $p2sRootCert = New-AzVpnClientRootCertificate -Name $vpnRootCertName `
        -PublicCertData $certBase64

    # Create the VPN gateway
    New-AzVirtualNetworkGateway -Name $vpnName `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        -IpConfigurations $gatewayIpConfig `
        -GatewayType Vpn `
        -VpnType RouteBased `
        -EnableBgp $false `
        -GatewaySku Standard `
        -VpnClientAddressPool $vpnClientAddressPool `
        -VpnClientRootCertificates $p2sRootCert
    ```

    > [!WARNING]  
    > Ce processus peut prendre plusieurs minutes.

5. Utilisez le code suivant pour créer le compte Stockage Azure et le conteneur de blobs :

    ```powershell
    # Create the storage account
    New-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $storageName `
        -SkuName Standard_GRS `
        -Location $location `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly 1

    # Get the storage account keys and create a context
    $defaultStorageKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName `
        -Name $storageName)[0].Value
    $storageContext = New-AzStorageContext -StorageAccountName $storageName `
        -StorageAccountKey $defaultStorageKey

    # Create the default storage container
    New-AzStorageContainer -Name $defaultContainerName `
        -Context $storageContext
    ```

6. Utilisez le code suivant pour créer le cluster HDInsight :

    ```powershell
    # Create the HDInsight cluster
    New-AzHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $clusterName `
        -Location $location `
        -ClusterSizeInNodes $hdiWorkerNodes `
        -ClusterType $hdiType `
        -OSType Linux `
        -Version $hdiVersion `
        -HttpCredential $adminCreds `
        -SshCredential $sshCreds `
        -DefaultStorageAccountName "$storageName.blob.core.windows.net" `
        -DefaultStorageAccountKey $defaultStorageKey `
        -DefaultStorageContainer $defaultContainerName `
        -DisksPerWorkerNode 2 `
        -VirtualNetworkId $network.Id `
        -SubnetName $defaultSubnet.Id
    ```

   > [!WARNING]  
   > Ce processus prend environ 15 minutes.

### <a name="configure-kafka-for-ip-advertising"></a>Configuration de Kafka pour la publication d’adresses IP

Par défaut, Apache Zookeeper retourne le nom de domaine des répartiteurs Kafka aux clients. Cette configuration ne fonctionne pas avec le client logiciel VPN, car il ne peut pas utiliser la résolution de noms pour les entités du réseau virtuel. Suivez les étapes ci-dessous pour configurer Kafka afin qu’il publie des adresses IP au lieu des noms de domaine :

1. À l’aide d’un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`. Remplacez `CLUSTERNAME` par le nom du cluster Kafka sur HDInsight.

    Lorsque vous y êtes invité, utilisez le nom et le mot de passe utilisateur HTTPS correspondant au cluster. L’interface utilisateur web d’Ambari pour le cluster s’affiche.

2. Pour afficher plus d’informations sur Kafka, sélectionnez __Kafka__ dans la liste sur la gauche.

    ![Liste des services avec l’option Kafka en surbrillance](./media/apache-kafka-connect-vpn-gateway/select-kafka-service.png)

3. Pour afficher la configuration Kafka, sélectionnez __Configurations__ en haut et au centre de la page.

    ![Apache Ambari - Configuration des services](./media/apache-kafka-connect-vpn-gateway/select-kafka-config1.png)

4. Pour trouver la configuration __kafka-env__, entrez `kafka-env` dans le champ __Filtre__ situé en haut à droite.

    ![Configuration Kafka, pour kafka-env](./media/apache-kafka-connect-vpn-gateway/search-for-kafka-env.png)

5. Pour configurer Kafka afin qu’il publie des adresses IP, ajoutez le texte suivant au bas du champ __kafka-env-template__ :

    ```
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. Pour configurer l’interface écoutée par Kafka, entrez `listeners` dans le champ __Filtre__ situé en haut à droite.

7. Pour configurer Kafka afin qu’il écoute toutes les interfaces réseau, remplacez la valeur du champ __écouteurs__ par `PLAINTEXT://0.0.0.0:9092`.

8. Utilisez le bouton __Enregistrer__ pour enregistrer les modifications apportées à la configuration. Entrez un message texte décrivant les modifications. Sélectionnez __OK__ une fois les modifications apportées.

    ![Apache Ambari - Enregistrement de la configuration](./media/apache-kafka-connect-vpn-gateway/save-configuration-button.png)

9. Pour éviter des erreurs lors du redémarrage de Kafka, utilisez le bouton __Service Actions__ (Actions du service) et sélectionnez __Activer le mode de maintenance__. Sélectionnez OK pour terminer cette opération.

    ![Actions de service, avec l’option Activer le mode de maintenance en surbrillance](./media/apache-kafka-connect-vpn-gateway/turn-on-maintenance-mode.png)

10. Pour redémarrer Kafka, utilisez le bouton __Redémarrer__ et sélectionnez __Restart All Affected__ (Redémarrer tous les éléments affectés). Confirmez le redémarrage, puis utilisez le bouton __OK__ une fois l’opération terminée.

    ![Bouton Redémarrer avec l’option Restart All Affected (Redémarrer tous les éléments affectés) en surbrillance](./media/apache-kafka-connect-vpn-gateway/restart-required-button.png)

11. Pour désactiver le mode de maintenance, utilisez le bouton __Service Actions__ (Actions du service) et sélectionnez __Désactiver le mode de maintenance__. Sélectionnez **OK** pour terminer cette opération.

### <a name="connect-to-the-vpn-gateway"></a>Connexion à la passerelle VPN

Pour vous connecter à la passerelle VPN, suivez la section __Connexion à Azure__ du document [Configuration d’une connexion de point à site à un réseau virtuel à l’aide de PowerShell](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md#connect).

## <a name="example-python-client"></a><a id="python-client"></a> Exemple : client Python

Pour valider la connectivité à Kafka, procédez comme suit pour créer et exécuter un producteur et un consommateur Python :

1. Pour récupérer le nom de domaine complet (FQDN) et les adresses IP des nœuds du cluster Kafka, utilisez l’une des méthodes suivantes :

    ```powershell
    $resourceGroupName = "The resource group that contains the virtual network used with HDInsight"

    $clusterNICs = Get-AzNetworkInterface -ResourceGroupName $resourceGroupName | where-object {$_.Name -like "*node*"}

    $nodes = @()
    foreach($nic in $clusterNICs) {
        $node = new-object System.Object
        $node | add-member -MemberType NoteProperty -name "Type" -value $nic.Name.Split('-')[1]
        $node | add-member -MemberType NoteProperty -name "InternalIP" -value $nic.IpConfigurations.PrivateIpAddress
        $node | add-member -MemberType NoteProperty -name "InternalFQDN" -value $nic.DnsSettings.InternalFqdn
        $nodes += $node
    }
    $nodes | sort-object Type
    ```

    ```azurecli
    az network nic list --resource-group <resourcegroupname> --output table --query "[?contains(name,'node')].{NICname:name,InternalIP:ipConfigurations[0].privateIpAddress,InternalFQDN:dnsSettings.internalFqdn}"
    ```

    Ce script suppose que `$resourceGroupName` est le nom du groupe de ressources Azure qui contient le réseau virtuel Azure.

    Enregistrez les informations retournées afin de les utiliser dans les étapes suivantes.

2. Utilisez ce qui suit pour installer le client [kafka-python](https://kafka-python.readthedocs.io/) :

    ```bash
    pip install kafka-python
    ```

3. Pour envoyer des données à Kafka, utilisez le code Python suivant :

   ```python
   from kafka import KafkaProducer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # NOTE: you don't need the full list of worker nodes, just one or two.
   producer = KafkaProducer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'])
   for _ in range(50):
      producer.send('testtopic', b'test message')
   ```

    Remplacez les entrées `'kafka_broker'` par les adresses renvoyées à partir de l’étape 1 de cette section :

   * Si vous utilisez un __client logiciel VPN__, remplacez les entrées `kafka_broker` avec l’adresse IP de vos nœuds de travail.

   * Si vous avez __activé la résolution de noms via un serveur DNS personnalisé__, remplacez les entrées `kafka_broker` avec le nom de domaine complet des nœuds de travail.

     > [!NOTE]
     > Ce code envoie la chaîne `test message` à la rubrique `testtopic`. La configuration par défaut de Kafka sur HDInsight consiste à créer la rubrique si elle n’existe pas.

4. Pour récupérer les messages à partir de Kafka, utilisez le code Python suivant :

   ```python
   from kafka import KafkaConsumer
   # Replace the `ip_address` entries with the IP address of your worker nodes
   # Again, you only need one or two, not the full list.
   # Note: auto_offset_reset='earliest' resets the starting offset to the beginning
   #       of the topic
   consumer = KafkaConsumer(bootstrap_servers=['kafka_broker_1','kafka_broker_2'],auto_offset_reset='earliest')
   consumer.subscribe(['testtopic'])
   for msg in consumer:
     print (msg)
   ```

    Remplacez les entrées `'kafka_broker'` par les adresses renvoyées à partir de l’étape 1 de cette section :

    * Si vous utilisez un __client logiciel VPN__, remplacez les entrées `kafka_broker` avec l’adresse IP de vos nœuds de travail.

    * Si vous avez __activé la résolution de noms via un serveur DNS personnalisé__, remplacez les entrées `kafka_broker` avec le nom de domaine complet des nœuds de travail.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’utilisation de HDInsight dans un réseau virtuel, consultez le document [Plan a virtual network deployment for Azure HDInsight clusters (Planifier un déploiement de réseau virtuel pour des clusters Azure HDInsight)](../hdinsight-plan-virtual-network-deployment.md).

Pour plus d’informations sur la création d’un réseau virtuel Azure avec une passerelle VPN de point à site, consultez les documents suivants :

* [Configuration d’une connexion point à site à un réseau virtuel à l’aide du portail Azure](../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)

* [Configuration d’une connexion de point à site à un réseau virtuel à l’aide de PowerShell](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

Pour plus d’informations sur l’utilisation d’Apache Kafka sur HDInsight, consultez les documents suivants :

* [Prise en main d’Apache Kafka sur HDInsight](apache-kafka-get-started.md)
* [Utiliser la mise en miroir avec Apache Kafka sur HDInsight](apache-kafka-mirroring.md)
