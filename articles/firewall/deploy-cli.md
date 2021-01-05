---
title: Déployer et configurer un pare-feu Azure à l’aide d’Azure CLI
description: Dans cet article, vous découvrez comment déployer et configurer un pare-feu Azure avec Azure CLI.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 08/29/2019
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: 6993bd10caf2f7e489de8074e311f75710b83d82
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659435"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-cli"></a>Déployer et configurer un pare-feu Azure à l’aide d’Azure CLI

Le contrôle de l’accès réseau sortant est une partie importante d’un plan de sécurité réseau global. Par exemple, vous pouvez souhaiter limiter l’accès aux sites web. Vous pouvez aussi vouloir limiter l’accès à certaines adresses IP et à certains ports sortants.

Vous pouvez contrôler l’accès réseau sortant à partir d’un sous-réseau Azure à l’aide du Pare-feu Azure. Avec le Pare-feu Azure, vous pouvez configurer les éléments suivants :

* Règles d’application qui définissent des noms de domaine complets (FQDN) accessibles depuis un sous-réseau. Le nom de domaine complet peut également [ inclure les instances SQL](sql-fqdn-filtering.md).
* Règles réseau qui définissent l’adresse source, le protocole, le port de destination et l’adresse de destination.

Le trafic réseau est soumis aux règles de pare-feu configurées lorsque vous routez votre trafic réseau vers le pare-feu en tant que sous-réseau de passerelle par défaut.

Dans le cadre de cet article, vous créez un seul réseau virtuel simplifié avec trois sous-réseaux pour un déploiement facile. Pour les déploiements de production, l’utilisation d’un [modèle Hub and Spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) est recommandée. Le pare-feu se trouve dans son propre réseau virtuel. Les serveurs de la charge de travail se trouvent dans des réseaux virtuels appairés dans la même région avec un ou plusieurs sous-réseaux.

* **AzureFirewallSubnet** : le pare-feu est dans ce sous-réseau.
* **Workload-SN** : le serveur de la charge de travail est dans ce sous-réseau. Le trafic réseau de ce sous-réseau traverse le pare-feu.
* **Jump-SN** : le serveur « jump » est dans ce sous-réseau. Le serveur de rebond possède une adresse IP publique à laquelle vous pouvez vous connecter à l’aide du Bureau à distance. De là, vous pouvez alors vous connecter (à l’aide d’un autre Bureau à distance) au serveur de la charge de travail.

![Tutoriel relatif à l’infrastructure réseau](media/tutorial-firewall-rules-portal/Tutorial_network.png)

Dans cet article, vous apprendrez comment :

* Configurer un environnement réseau de test
* Déployer un pare-feu
* Créer un itinéraire par défaut
* Configurer une règle d’application pour autoriser l’accès à www.google.com
* Configurer une règle de réseau pour autoriser l’accès aux serveurs DNS externes
* Tester le pare-feu

Si vous préférez, vous pouvez effectuer cette procédure en utilisant le [portail Azure](tutorial-firewall-deploy-portal.md) ou [Azure PowerShell](deploy-ps.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.4 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="set-up-the-network"></a>Configurer le réseau

Tout d’abord, créez un groupe de ressources qui contiendra les ressources nécessaires pour déployer le pare-feu. Ensuite, créez un réseau virtuel, des sous-réseaux et des serveurs de test.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Le groupe de ressources contient toutes les ressources nécessaires à ce déploiement.

```azurecli-interactive
az group create --name Test-FW-RG --location eastus
```

### <a name="create-a-vnet"></a>Créer un réseau virtuel

Ce réseau virtuel comporte trois sous-réseaux.

> [!NOTE]
> La taille du sous-réseau AzureFirewallSubnet est /26. Pour plus d’informations sur la taille du sous-réseau, consultez le [FAQ Pare-feu Azure](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

```azurecli-interactive
az network vnet create \
  --name Test-FW-VN \
  --resource-group Test-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.0.1.0/26
az network vnet subnet create \
  --name Workload-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.2.0/24
az network vnet subnet create \
  --name Jump-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.3.0/24
```

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Maintenant créez les machines virtuelles de rebond et de charge de travail, et placez-les dans les sous-réseaux appropriés.
Quand vous y êtes invité, tapez un mot de passe pour la machine virtuelle.

Créez la machine virtuelle Srv-Jump.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Jump \
    --location eastus \
    --image win2016datacenter \
    --vnet-name Test-FW-VN \
    --subnet Jump-SN \
    --admin-username azureadmin
az vm open-port --port 3389 --resource-group Test-FW-RG --name Srv-Jump
```



Pour les besoins du test, créez une carte réseau pour Srv-Work avec les adresses IP de serveur DNS spécifiques et sans adresse IP publique.

```azurecli-interactive
az network nic create \
    -g Test-FW-RG \
    -n Srv-Work-NIC \
   --vnet-name Test-FW-VN \
   --subnet Workload-SN \
   --public-ip-address "" \
   --dns-servers 209.244.0.3 209.244.0.4
```

Créez maintenant la machine virtuelle de charge de travail.
Quand vous y êtes invité, tapez un mot de passe pour la machine virtuelle.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Work \
    --location eastus \
    --image win2016datacenter \
    --nics Srv-Work-NIC \
    --admin-username azureadmin
```

## <a name="deploy-the-firewall"></a>Déployer le pare-feu

Déployez maintenant le pare-feu dans le réseau virtuel.

```azurecli-interactive
az network firewall create \
    --name Test-FW01 \
    --resource-group Test-FW-RG \
    --location eastus
az network public-ip create \
    --name fw-pip \
    --resource-group Test-FW-RG \
    --location eastus \
    --allocation-method static \
    --sku standard
az network firewall ip-config create \
    --firewall-name Test-FW01 \
    --name FW-config \
    --public-ip-address fw-pip \
    --resource-group Test-FW-RG \
    --vnet-name Test-FW-VN
az network firewall update \
    --name Test-FW01 \
    --resource-group Test-FW-RG 
az network public-ip show \
    --name fw-pip \
    --resource-group Test-FW-RG
fwprivaddr="$(az network firewall ip-config list -g Test-FW-RG -f Test-FW01 --query "[?name=='FW-config'].privateIpAddress" --output tsv)"
```

Notez l’adresse IP privée. Vous l’utiliserez plus tard lors de la création de l’itinéraire par défaut.

## <a name="create-a-default-route"></a>Créer un itinéraire par défaut

Créer une table, avec la propagation de routes BGP désactivée

```azurecli-interactive
az network route-table create \
    --name Firewall-rt-table \
    --resource-group Test-FW-RG \
    --location eastus \
    --disable-bgp-route-propagation true
```

Créez la route.

```azurecli-interactive
az network route-table route create \
  --resource-group Test-FW-RG \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $fwprivaddr
```

Associez la table de routage au sous-réseau

```azurecli-interactive
az network vnet subnet update \
    -n Workload-SN \
    -g Test-FW-RG \
    --vnet-name Test-FW-VN \
    --address-prefixes 10.0.2.0/24 \
    --route-table Firewall-rt-table
```

## <a name="configure-an-application-rule"></a>Configurer une règle d’application

La règle d’application autorise un accès sortant vers www.google.com.

```azurecli-interactive
az network firewall application-rule create \
   --collection-name App-Coll01 \
   --firewall-name Test-FW01 \
   --name Allow-Google \
   --protocols Http=80 Https=443 \
   --resource-group Test-FW-RG \
   --target-fqdns www.google.com \
   --source-addresses 10.0.2.0/24 \
   --priority 200 \
   --action Allow
```

Le Pare-feu Azure comprend un regroupement de règles intégré pour les noms de domaine complets d’infrastructure qui sont autorisés par défaut. Ces noms de domaine complets sont spécifiques à la plateforme et ne peuvent pas être utilisés à d’autres fins. Pour plus d’informations, consultez [Noms de domaine complets d’infrastructure](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Configurer une règle de réseau

La règle de réseau autorise un accès sortant vers deux adresses IP sur le port 53 (DNS).

```azurecli-interactive
az network firewall network-rule create \
   --collection-name Net-Coll01 \
   --destination-addresses 209.244.0.3 209.244.0.4 \
   --destination-ports 53 \
   --firewall-name Test-FW01 \
   --name Allow-DNS \
   --protocols UDP \
   --resource-group Test-FW-RG \
   --priority 200 \
   --source-addresses 10.0.2.0/24 \
   --action Allow
```

## <a name="test-the-firewall"></a>Tester le pare-feu

Testez maintenant le pare-feu pour vérifier qu’il fonctionne comme prévu.

1. Notez l’adresse IP privée de la machine virtuelle **Srv-Work** :

   ```azurecli-interactive
   az vm list-ip-addresses \
   -g Test-FW-RG \
   -n Srv-Work
   ```

1. Connectez un bureau à distance à la machine virtuelle **Srv-Jump**, puis connectez-vous. De là, ouvrez une connexion Bureau à distance à l’adresse IP privée **Srv-Work** et connectez-vous.

3. Sur **SRV-Work**, ouvrez une fenêtre PowerShell et exécutez les commandes suivantes :

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Les deux commandes doivent retourner des réponses, montrant que vos requêtes DNS passent au travers du pare-feu.

1. Exécutez les commandes suivantes :

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   Les requêtes `www.google.com` doivent réussir, et les requêtes `www.microsoft.com` échouer. Cet exemple montre que vos règles de pare-feu fonctionnent comme prévu.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

* Vous pouvez résoudre les noms DNS à l’aide du serveur DNS externe configuré.
* Vous pouvez accéder au nom de domaine complet autorisé, mais pas à d’autres.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Conservez vos ressources de pare-feu pour le prochain tutoriel ou, si vous n’en avez plus besoin, supprimez le groupe de ressources **Test-FW-RG** pour supprimer toutes les ressources associées au pare-feu :

```azurecli-interactive
az group delete \
  -n Test-FW-RG
```

## <a name="next-steps"></a>Étapes suivantes

* [Tutoriel : Superviser les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)