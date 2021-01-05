---
title: Créer un cluster Service Fabric Linux dans Azure
description: Découvrez comment déployer un cluster Service Fabric Linux dans un réseau virtuel Azure existant à l’aide de l’interface Azure CLI.
ms.topic: conceptual
ms.date: 02/14/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 52eba2e5780b1a66f3884a764631908335372273
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92738963"
---
# <a name="deploy-a-linux-service-fabric-cluster-into-an-azure-virtual-network"></a>Déployer un cluster Service Fabric Linux dans un réseau virtuel Azure

Dans cet article, vous découvrirez comment déployer un cluster Service Fabric Linux dans un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) à l’aide de l’interface Azure CLI et d’un modèle. Lorsque vous avez terminé, vous disposez d’un cluster en cours d’exécution dans le cloud sur lequel vous pouvez déployer des applications. Pour créer un cluster Windows à l’aide de PowerShell, consultez la section relative à la [création d’un cluster Windows sécurisé sur Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Installez l’[interface de ligne de commande (CLI) de Service Fabric](service-fabric-cli.md).
* Installer [Azure CLI](/cli/azure/install-azure-cli)
* Pour découvrir les concepts clés des clusters, consultez [Vue d’ensemble des clusters Azure](service-fabric-azure-clusters-overview.md)
* [Planifiez et préparez](service-fabric-cluster-azure-deployment-preparation.md) un déploiement de cluster de production.

Les procédures suivantes créent un cluster Service Fabric à sept nœuds. Pour calculer le coût lié à l’exécution d’un cluster Service Fabric dans Azure, utilisez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/).

## <a name="download-and-explore-the-template"></a>Télécharger et explorer le modèle

Téléchargez les fichiers de modèle Resource Manager suivants :

Pour Ubuntu 16.04 LTS :

* [AzureDeploy.json][template]
* [AzureDeploy.Parameters.json][parameters]

Pour Ubuntu 18.04 LTS :

* [AzureDeploy.json][template2]
* [AzureDeploy.Parameters.json][parameters2]

Pour Ubuntu 18.04 LTS, les différences entre les deux modèles sont les suivantes : 
* l’attribut **vmImageSku** défini sur « 18.04-LTS »
* la valeur **typeHandlerVersion** de chaque nœud définie sur 1.1
* pour la ressource Microsoft.ServiceFabric/clusters :
   - **apiVersion** défini sur « 2019-03-01 » ou une version ultérieure
   - propriété **vmImage** définie sur « Ubuntu18_04 »

Ce modèle déploie un cluster sécurisé de sept machines virtuelles et trois types de nœuds dans un réseau virtuel.  D’autres exemples de modèles sont disponibles sur [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Le modèle [AzureDeploy.json][template] déploie un certain nombre de ressources, notamment celles ci-dessous.

### <a name="service-fabric-cluster"></a>Cluster Service Fabric

Dans la ressource **Microsoft.ServiceFabric/clusters** , un cluster Linux est déployé avec les caractéristiques suivantes :

* Trois types de nœuds
* Cinq nœuds dans le type de nœud principal (configurable dans les paramètres du modèle) et un nœud dans chacun des autres types
* Système d’exploitation : Ubuntu 16.04 LTS/Ubuntu 18.04 LTS (configurable dans les paramètres du modèle)
* certificat sécurisé (configurable dans les paramètres du modèle)
* [service DNS](service-fabric-dnsservice.md) activé
* [niveau de durabilité](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) Bronze (configurable dans les paramètres du modèle)
* [niveau de fiabilité](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) Silver (configurable dans les paramètres du modèle)
* point de terminaison de connexion client : 19000 (configurable dans les paramètres du modèle)
* point de terminaison de passerelle HTTP : 19080 (configurable dans les paramètres du modèle)

### <a name="azure-load-balancer"></a>Équilibrage de charge Azure

Dans la ressource **Microsoft.Network/loadBalancers** , un équilibreur de charge est configuré et des sondes et règles sont configurées pour les ports suivants :

* point de terminaison de connexion client : 19000
* point de terminaison de passerelle HTTP : 19080
* port de l’application : 80
* port de l’application : 443

### <a name="virtual-network-and-subnet"></a>Réseau virtuel et sous-réseau

Les noms du réseau virtuel et du sous-réseau sont déclarés dans les paramètres du modèle.  Les espaces d’adressage du réseau virtuel et du sous-réseau sont également déclarés dans les paramètres de modèle et configurés dans la ressource **Microsoft.Network/virtualNetworks** :

* Espace d’adressage du réseau virtuel : 10.0.0.0/16
* Espace d’adressage du sous-réseau Service Fabric : 10.0.2.0/24

Si d’autres ports de l’application sont nécessaires, vous devez ajuster les ressources Microsoft.Network/loadBalancers pour autoriser le trafic entrant.

## <a name="set-template-parameters"></a>Définir les paramètres de modèle

Le fichier **AzureDeploy.Parameters** déclare de nombreuses valeurs servant à déployer le cluster et les ressources associées. Voici certains des paramètres que vous devrez peut-être modifier pour votre déploiement :

|Paramètre|Valeur d'exemple|Notes|
|---|---||
|adminUsername|vmadmin| Nom d’utilisateur administrateur pour les machines virtuelles de cluster. |
|adminPassword|Password#1234| Mot de passe d’administrateur pour les machines virtuelles de cluster.|
|clusterName|mysfcluster123| Nom du cluster. |
|location|southcentralus| Emplacement du cluster. |
|certificateThumbprint|| <p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat.</p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez la valeur d’empreinte du certificat SHA-1. Par exemple, « 6190390162C988701DB5676EB81083EA608DCCF3 ». </p>|
|certificateUrlValue|| <p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat.</p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez l’URL du certificat. Par exemple, « https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346 ».</p>|
|sourceVaultValue||<p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat.</p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez la valeur de coffre source. Par exemple, « /subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT ».</p>|

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Déployer le réseau virtuel et le cluster

Puis, configurez la topologie de réseau et déployez le cluster Service Fabric. Le modèle Resource Manager **AzureDeploy.json** crée un réseau virtuel et un sous-réseau pour Service Fabric. Le modèle déploie également un cluster avec la sécurité de certificat activée.  Pour les clusters de production, utilisez un certificat délivré par une autorité de certification (AC) en tant que certificat de cluster. Un certificat auto-signé peut être utilisé pour garantir la sécurité des clusters de test.

Dans cet article, le modèle déploie un cluster qui utilise l’empreinte numérique du certificat pour identifier le certificat de cluster.  Deux certificats ne peuvent pas avoir la même empreinte numérique, ce qui complique leur gestion. Basculer un cluster déployé de l’utilisation des empreintes de certificat à l’utilisation des noms communs de certificat simplifie considérablement la gestion des certificats.  Pour savoir comment mettre à jour le cluster pour que celui-ci utilise des noms communs de certificat pour la gestion des certificats, consultez [Modifier un cluster pour qu’il passe de l’utilisation d’une empreinte de certificat à l’utilisation d’un nom commun](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-using-an-existing-certificate"></a>Créer un cluster à l’aide d’un certificat existant

Le script suivant utilise la commande [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest) et le modèle pour déployer un nouveau cluster sécurisé à l’aide d’un certificat existant. La commande crée aussi un coffre de clés dans Azure et charge votre certificat.

```azurecli
ResourceGroupName="sflinuxclustergroup"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates\MyCertificate.pem"

# sign in to your Azure account and select your subscription
az login
az account set --subscription <guid>

# Create a new resource group for your deployment and give it a name and a location.
az group create --name $ResourceGroupName --location $Location

# Create the Service Fabric cluster.
az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --certificate-password $Password --certificate-file $CertPath \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName  \
   --template-file AzureDeploy.json --parameter-file AzureDeploy.Parameters.json
```

### <a name="create-a-cluster-using-a-new-self-signed-certificate"></a>Créer un cluster à l’aide d’un nouveau certificat auto-signé

Le script suivant utilise la commande [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest) et un modèle pour déployer un nouveau cluster dans Azure. La commande crée aussi un coffre de clés dans Azure, ajoute un nouveau certificat autosigné dans le coffre de clés, puis télécharge le fichier de certificat localement.

```azurecli
ResourceGroupName="sflinuxclustergroup"
ClusterName="sflinuxcluster"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates"

az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --cluster-name $ClusterName --template-file C:\temp\cluster\AzureDeploy.json \
   --parameter-file C:\temp\cluster\AzureDeploy.Parameters.json --certificate-password $Password \
   --certificate-output-folder $CertPath --certificate-subject-name $ClusterName.$Location.cloudapp.azure.com \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName
```

## <a name="connect-to-the-secure-cluster"></a>Se connecter à un cluster sécurisé

Connectez-vous au cluster avec votre clé par le biais de la commande `sfctl cluster select` de l’interface de ligne de commande de Service Fabric.  Notez que vous ne pouvez utiliser que l’option **--no-verify** pour un certificat auto-signé.

```console
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

Vérifiez que vous êtes connecté et que le cluster est sain à l’aide de la commande `sfctl cluster health`.

```console
sfctl cluster health
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne passez pas immédiatement à l’article suivant, vous souhaiterez peut-être [supprimer le cluster](./service-fabric-tutorial-delete-cluster.md) pour éviter de subir des frais.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [mettre à l'échelle un cluster.](service-fabric-tutorial-scale-cluster.md)

Dans cet article, le modèle déploie un cluster qui utilise l’empreinte numérique du certificat pour identifier le certificat de cluster.  Deux certificats ne peuvent pas avoir la même empreinte numérique, ce qui complique leur gestion. Basculer un cluster déployé de l’utilisation des empreintes de certificat à l’utilisation des noms communs de certificat simplifie considérablement la gestion des certificats.  Pour savoir comment mettre à jour le cluster pour que celui-ci utilise des noms communs de certificat pour la gestion des certificats, consultez [Modifier un cluster pour qu’il passe de l’utilisation d’une empreinte de certificat à l’utilisation d’un nom commun](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.Parameters.json
[template2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.json
[parameters2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.Parameters.json
