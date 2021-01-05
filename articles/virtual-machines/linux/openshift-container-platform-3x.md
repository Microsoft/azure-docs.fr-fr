---
title: Déployer OpenShift Container Platform 3.11 dans Azure
description: Déployez OpenShift Container Platform 3.11 dans Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: how-to
ms.workload: infrastructure
ms.date: 04/05/2020
ms.author: haroldw
ms.openlocfilehash: fab8f88a39730411503af273902a53f169e3fe57
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97703733"
---
# <a name="deploy-openshift-container-platform-311-in-azure"></a>Déployer OpenShift Container Platform 3.11 dans Azure

Vous pouvez utiliser plusieurs méthodes pour déployer OpenShift Container Platform 3.11 dans Azure :

- Vous pouvez déployer manuellement les composants d’infrastructure Azure nécessaires, puis suivre la [documentation OpenShift Container Platform](https://docs.openshift.com/container-platform).
- Vous pouvez également utiliser un [modèle Resource Manager](https://github.com/Microsoft/openshift-container-platform/) existant qui simplifie le déploiement du cluster OpenShift Container Platform.
- Une autre option consiste à utiliser l’[offre de la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/osatesting.open-shift-azure-proxy).

Pour toutes les options, vous devez avoir un abonnement Red Hat. Pendant le déploiement, l’instance Red Hat Enterprise Linux est inscrite dans l’abonnement Red Hat et associée à l’ID du pool contenant les droits pour OpenShift Container Platform.
Vérifiez que vous avez un nom d’utilisateur, un mot de passe et un ID de pool RHSM (Red Hat Subscription Manager) valides. Vous pouvez utiliser une clé d’activation, un ID d’organisation et un ID de pool. Vous pouvez vérifier ces informations en vous connectant au site https://access.redhat.com.


## <a name="deploy-using-the-openshift-container-platform-resource-manager-311-template"></a>Déployer à l’aide du modèle Resource Manager d’OpenShift Container Platform 3.11

### <a name="private-clusters"></a>Clusters privés

Le déploiement de clusters OpenShift privés nécessite plus que le simple fait de ne pas avoir d’adresse IP publique associée à l’équilibreur de charge principal (console web) ou à l’équilibreur de charge secondaire (routeur).  En règle générale, un cluster privé utilise un serveur DNS personnalisé (et pas le serveur Azure DNS par défaut), un nom de domaine personnalisé (par exemple, contoso.com) et un ou plusieurs réseaux virtuels prédéfinis.  Pour les clusters privés, vous devez configurer votre réseau virtuel avec tous les sous-réseaux appropriés et les paramètres du serveur DNS à l’avance.  Utilisez ensuite **existingMasterSubnetReference**, **existingInfraSubnetReference**, **existingCnsSubnetReference** et  **existingNodeSubnetReference** pour spécifier le sous-réseau existant dans l’optique d’une utilisation par le cluster.

Si un cluster principal privé est sélectionné (**masterClusterType**=private), une adresse IP privée statique doit être spécifiée pour **masterPrivateClusterIp**.  Cette adresse IP est attribuée au composant frontal de l’équilibreur de charge principal.  L’adresse IP doit se trouver dans la plage CIDR du sous-réseau principal et ne doit pas être utilisée.  **masterClusterDnsType** doit être défini sur « custom » et le nom DNS principal doit être fourni pour **masterClusterDns**.  Le nom DNS doit correspondre à l’adresse IP privée statique et permet d’accéder à la console sur les nœuds principaux.

Si un routeur privé est sélectionné (**routerClusterType**=private), une adresse IP privée statique doit être spécifiée pour **routerPrivateClusterIp**.  Cette adresse IP est attribuée au composant frontal de l’équilibreur de charge secondaire.  L’adresse IP doit se trouver dans la plage CIDR du sous-réseau secondaire et ne doit pas être utilisée.  **routingSubDomainType** doit avoir la valeur « custom » et le nom DNS générique de routage doit être fourni pour **routingSubDomain**.  

Si des clusters principaux privés et un routeur privé sont sélectionnés, le nom de domaine personnalisé doit également être entré pour **domainName**

Après un déploiement réussi, le nœud Bastion est le seul nœud avec une adresse IP publique, auquel vous pouvez appliquer un protocole SSH.  Même si les nœuds principaux sont configurés pour l’accès public, ils ne sont pas exposés à un accès SSH.

Pour effectuer le déploiement à l’aide du modèle Resource Manager, vous utilisez un fichier de paramètres qui fournit les paramètres d’entrée. Pour personnaliser davantage le déploiement, dupliquez (fork) le dépôt GitHub et changez les éléments appropriés.

Voici une liste non exhaustive d’options de personnalisation courantes :

- Taille de machine virtuelle bastion (variable dans azuredeploy.json)
- Conventions de nommage (variables dans azuredeploy.json)
- Caractéristiques de cluster OpenShift, modifiées dans le fichier hosts (deployOpenShift.sh)

### <a name="configure-the-parameters-file"></a>Configurer le fichier de paramètres

Le [modèle OpenShift Container Platform](https://github.com/Microsoft/openshift-container-platform) a plusieurs branches disponibles pour différentes versions d’OpenShift Container Platform.  Selon vos besoins, vous pouvez effectuer le déploiement directement à partir du dépôt, ou dupliquer (fork) le dépôt et personnaliser les modèles ou les scripts avant le déploiement.

Utilisez la valeur de `appId` du principal de service créé précédemment pour le paramètre `aadClientId`.

L’exemple suivant montre un fichier de paramètres nommé azuredeploy.parameters.json avec toutes les entrées obligatoires.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "_artifactsLocation": {
            "value": "https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master"
        },
        "location": {
            "value": "eastus"
        },
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_D4s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_D4s_v3"
        },
        "cnsVmSize": {
            "value": "Standard_E4s_v3"
        },
        "osImageType": {
            "value": "defaultgallery"
        },
        "marketplaceOsImage": {
            "value": {
                "publisher": "RedHat",
                "offer": "RHEL",
                "sku": "7-RAW",
                "version": "latest"
            }
        },
        "storageKind": {
            "value": "changeme"
        },
        "openshiftClusterPrefix": {
            "value": "changeme"
        },
        "minorVersion": {
            "value": "69"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 3
        },
        "nodeInstanceCount": {
            "value": 3
        },
        "cnsInstanceCount": {
            "value": 3
        },
        "osDiskSize": {
            "value": 64
        },
        "dataDiskSize": {
            "value": 64
        },
        "cnsGlusterDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "changeme"
        },
        "enableMetrics": {
            "value": "false"
        },
        "enableLogging": {
            "value": "false"
        },
        "enableCNS": {
            "value": "false"
        },
        "rhsmUsernameOrOrgId": {
            "value": "changeme"
        },
        "rhsmPoolId": {
            "value": "changeme"
        },
        "rhsmBrokerPoolId": {
            "value": "changeme"
        },
        "sshPublicKey": {
            "value": "GEN-SSH-PUB-KEY"
        },
        "keyVaultSubscriptionId": {
            "value": "255a325e-8276-4ada-af8f-33af5658eb34"
        },
        "keyVaultResourceGroup": {
            "value": "changeme"
        },
        "keyVaultName": {
            "value": "changeme"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "changeme"
        },
        "domainName": {
            "value": "contoso.com"
        },
        "masterClusterDnsType": {
            "value": "default"
        },
        "masterClusterDns": {
            "value": "console.contoso.com"
        },
        "routingSubDomainType": {
            "value": "nipio"
        },
        "routingSubDomain": {
            "value": "apps.contoso.com"
        },
        "virtualNetworkNewOrExisting": {
            "value": "new"
        },
        "virtualNetworkName": {
            "value": "changeme"
        },
        "addressPrefixes": {
            "value": "10.0.0.0/14"
        },
        "masterSubnetName": {
            "value": "changeme"
        },
        "masterSubnetPrefix": {
            "value": "10.1.0.0/16"
        },
        "infraSubnetName": {
            "value": "changeme"
        },
        "infraSubnetPrefix": {
            "value": "10.2.0.0/16"
        },
        "nodeSubnetName": {
            "value": "changeme"
        },
        "nodeSubnetPrefix": {
            "value": "10.3.0.0/16"
        },
        "existingMasterSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/mastersubnet"
        },
        "existingInfraSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/infrasubnet"
        },
        "existingCnsSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/cnssubnet"
        },
        "existingNodeSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/nodesubnet"
        },
        "masterClusterType": {
            "value": "public"
        },
        "masterPrivateClusterIp": {
            "value": "10.1.0.200"
        },
        "routerClusterType": {
            "value": "public"
        },
        "routerPrivateClusterIp": {
            "value": "10.2.0.200"
        },
        "routingCertType": {
            "value": "selfsigned"
        },
        "masterCertType": {
            "value": "selfsigned"
        }
    }
}
```

Remplacez les paramètres par vos informations spécifiques.

Les paramètres peuvent différer selon les versions. Vérifiez quels sont les paramètres nécessaires pour la branche que vous utilisez.

### <a name="azuredeployparametersjson-file-explained"></a>Explication du fichier azuredeploy.Parameters.json

| Propriété | Description | Options valides | Valeur par défaut |
|----------|-------------|---------------|---------------|
| `_artifactsLocation`  | URL pour les artefacts (json, scripts, etc.) |  |  https:\//raw.githubusercontent.com/Microsoft/openshift-container-platform/master  |
| `location` | Région Azure dans laquelle déployer les ressources |  |  |
| `masterVmSize` | Taille de la machine virtuelle principale. Sélectionnez l’une des tailles de machine virtuelle autorisées, qui sont répertoriées dans le fichier azuredeploy.json |  | Standard_E2s_v3 |
| `infraVmSize` | Taille de la machine virtuelle secondaire. Sélectionnez l’une des tailles de machine virtuelle autorisées, qui sont répertoriées dans le fichier azuredeploy.json |  | Standard_D4s_v3 |
| `nodeVmSize` | Taille de la machine virtuelle du nœud d’application. Sélectionnez l’une des tailles de machine virtuelle autorisées, qui sont répertoriées dans le fichier azuredeploy.json |  | Standard_D4s_v3 |
| `cnsVmSize` | Taille de la machine virtuelle du nœud CNS (stockage natif de conteneurs). Sélectionnez l’une des tailles de machine virtuelle autorisées, qui sont répertoriées dans le fichier azuredeploy.json |  | Standard_E4s_v3 |
| `osImageType` | Image RHEL à utiliser. defaultgallery: On-Demand; marketplace: third-party image | defaultgallery <br> place de marché | defaultgallery |
| `marketplaceOsImage` | Si `osImageType` est marketplace, entrez les valeurs appropriées pour « publisher » « offer », « sku » et « version » de l’offre marketplace. Ce paramètre est un type d’objet |  |  |
| `storageKind` | Type de stockage à utiliser  | géré<br> non managé | géré |
| `openshiftClusterPrefix` | Préfixe de cluster utilisé pour configurer les noms d’hôtes de tous les nœuds.  Il doit contenir entre 1 et 20 caractères |  | mycluster |
| `minoVersion` | Version mineure d’OpenShift Container Platform 3.11 à déployer |  | 69 |
| `masterInstanceCount` | Nombre de nœuds principaux à déployer | 1, 3, 5 | 3 |
| `infraInstanceCount` | Nombre de nœuds secondaires à déployer | 1, 2, 3 | 3 |
| `nodeInstanceCount` | Nombre de nœuds à déployer | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 | 2 |
| `cnsInstanceCount` | Nombre de nœuds CNS à déployer | 3, 4 | 3 |
| `osDiskSize` | Taille du disque du système d’exploitation pour la machine virtuelle (en Go) | 64, 128, 256, 512, 1024, 2048 | 64 |
| `dataDiskSize` | Taille du disque de données à joindre aux nœuds pour le volume Docker (en Go) | 32, 64, 128, 256, 512, 1024, 2048 | 64 |
| `cnsGlusterDiskSize` | Taille du disque de données à joindre aux nœuds CNS pour glusterfs (en Go) | 32, 64, 128, 256, 512, 1024, 2048 | 128 |
| `adminUsername` | Nom d’utilisateur administrateur pour la connexion au système d’exploitation (machine virtuelle) et l’utilisateur initial OpenShift |  | ocpadmin |
| `enableMetrics` | Activez les métriques. Les métriques nécessitent davantage de ressources : vous devez donc sélectionner une taille appropriée pour la machine virtuelle secondaire | true <br> false | false |
| `enableLogging` | Activez la journalisation. Le pod elasticsearch nécessite 8 Go de RAM : vous devez donc sélectionner une taille appropriée pour la machine virtuelle secondaire | true <br> false | false |
| `enableCNS` | Activer le CNS | true <br> false | false |
| `rhsmUsernameOrOrgId` | ID d’organisation ou nom d’utilisateur du gestionnaire d’abonnements Red Hat |  |  |
| `rhsmPoolId` | ID du pool du gestionnaire d’abonnements Red Hat qui contient vos droits OpenShift pour les nœuds de calcul |  |  |
| `rhsmBrokerPoolId` | ID du pool du gestionnaire d’abonnements Red Hat qui contient vos droits OpenShift pour les nœuds principaux et secondaires. Si vous ne disposez pas de plusieurs ID de pool, entrez le même ID de pool « rhsmPoolId » |  |
| `sshPublicKey` | Copiez votre clé publique SSH ici |  |  |
| `keyVaultSubscriptionId` | ID de l’abonnement contenant le Key Vault |  |  |
| `keyVaultResourceGroup` | Nom du groupe de ressources qui contient le Key Vault |  |  |
| `keyVaultName` | Nom du Key Vault que vous avez créé |  |  |
| `enableAzure` | Activer le fournisseur de cloud Azure | true <br> false | true |
| `aadClientId` | ID de client Azure Active Directory également connu en tant qu’ID d’application pour le principal de service |  |  |
| `domainName` | Nom de domaine personnalisé à utiliser (le cas échéant). Défini sur « none » en l’absence de déploiement complet du cluster privé |  | Aucun |
| `masterClusterDnsType` | Type de domaine pour la console web OpenShift. « default » utilise l’étiquette DNS des adresses IP publiques principales et secondaires. « custom » permet de définir votre propre nom | default <br> custom | default |
| `masterClusterDns` | Nom DNS personnalisé à utiliser pour accéder à la console web OpenShift si vous avez sélectionné « custom » pour `masterClusterDnsType` |  | console.contoso.com |
| `routingSubDomainType` | Si la valeur est « nipio », `routingSubDomain` utilise nip.io.  Utilisez « custom » si vous souhaitez vous servir de votre propre domaine pour le routage | nipio <br> custom | nipio |
| `routingSubDomain` | Nom DNS générique à utiliser pour le routage si vous avez sélectionné « custom » pour `routingSubDomainType` |  | apps.contoso.com |
| `virtualNetworkNewOrExisting` | Vous pouvez choisir entre créer ou sélectionner un réseau virtuel | existing <br> new | new |
| `virtualNetworkResourceGroupName` | Nom du groupe de ressources pour le nouveau réseau virtuel si vous avez sélectionné « new » pour `virtualNetworkNewOrExisting` |  | resourceGroup().name |
| `virtualNetworkName` | Nom du réseau virtuel à créer si vous avez sélectionné « new » pour `virtualNetworkNewOrExisting` |  | openshiftvnet |
| `addressPrefixes` | Préfixe d’adresse du nouveau réseau virtuel |  | 10.0.0.0/14 |
| `masterSubnetName` | Nom du sous-réseau principal |  | mastersubnet |
| `masterSubnetPrefix` | Plage CIDR utilisée pour le sous-réseau principal : il doit s’agir d’un sous-ensemble d’addressPrefix |  | 10.1.0.0/16 |
| `infraSubnetName` | Nom du sous-réseau secondaire |  | infrasubnet |
| `infraSubnetPrefix` | Plage CIDR utilisée pour le sous-réseau secondaire : il doit s’agir d’un sous-ensemble d’addressPrefix |  | 10.2.0.0/16 |
| `nodeSubnetName` | Nom du sous-réseau du nœud |  | nodesubnet |
| `nodeSubnetPrefix` | Plage CIDR utilisée pour le sous-réseau du nœud : il doit s’agir d’un sous-ensemble d’addressPrefix |  | 10.3.0.0/16 |
| `existingMasterSubnetReference` | Référence complète au sous-réseau existant pour les nœuds principaux. Inutile si vous créez un réseau virtuel/sous-réseau |  |  |
| `existingInfraSubnetReference` | Référence complète au sous-réseau existant pour les nœuds secondaires. Inutile si vous créez un réseau virtuel/sous-réseau |  |  |
| `existingCnsSubnetReference` | Référence complète au sous-réseau existant pour les nœuds CNS. Inutile si vous créez un réseau virtuel/sous-réseau |  |  |
| `existingNodeSubnetReference` | Référence complète au sous-réseau existant pour les nœuds de calcul. Inutile si vous créez un réseau virtuel/sous-réseau |  |  |
| `masterClusterType` | Indiquez si le cluster utilise des nœuds principaux privés ou publics. Si vous choisissez les nœuds privés, les nœuds principaux ne sont pas exposés sur Internet au travers d’une adresse IP publique. Au lieu de cela, une adresse IP privée, spécifiée dans `masterPrivateClusterIp`, est utilisée | public <br> private | public |
| `masterPrivateClusterIp` | Si des nœuds principaux privés sont sélectionnés, une adresse IP privée doit être spécifiée pour l’équilibreur de charge interne dans le cas des nœuds principaux. L’adresse IP statique doit se trouver dans le bloc CIDR pour le sous-réseau principal et ne doit pas être déjà utilisée. Si des nœuds principaux publics sont sélectionnés, cette valeur ne sera pas utilisée mais elle doit toujours être spécifiée |  | 10.1.0.200 |
| `routerClusterType` | Indiquez si le cluster utilise des nœuds secondaires privés ou publics. Si vous choisissez des nœuds privés, les nœuds secondaires ne sont pas exposés sur Internet au travers d’une adresse IP publique. Au lieu de cela, une adresse IP privée, spécifiée dans `routerPrivateClusterIp`, est utilisée | public <br> private | public |
| `routerPrivateClusterIp` | Si des nœuds secondaires privés sont sélectionnés, une adresse IP privée doit être spécifiée pour l’équilibreur de charge interne dans le cas des nœuds secondaires. L’adresse IP statique doit se trouver dans le bloc CIDR pour le sous-réseau secondaire et ne doit pas être déjà utilisée. Si des nœuds secondaires publics sont sélectionnés, cette valeur ne sera pas utilisée mais elle doit toujours être spécifiée |  | 10.2.0.200 |
| `routingCertType` | Utilisez un certificat personnalisé pour le domaine de routage ou le certificat autosigné par défaut : suivez les instructions dans la section **Certificats personnalisés** | selfsigned <br> custom | selfsigned |
| `masterCertType` | Utilisez un certificat personnalisé pour le domaine principal ou le certificat autosigné par défaut : suivez les instructions dans la section **Certificats personnalisés** | selfsigned <br> custom | selfsigned |

<br>

### <a name="deploy-using-azure-cli"></a>Déployer à l’aide d’Azure CLI

> [!NOTE] 
> La commande suivante requiert Azure CLI 2.0.8 ou version ultérieure. Pour vérifier la version d’Azure CLI, exécutez la commande `az --version`. Pour mettre à jour la version de l’interface CLI, consultez [Installer Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latesti).

L’exemple suivant déploie le cluster OpenShift et toutes les ressources associées dans un groupe de ressources nommé openshiftrg, avec le nom de déploiement myOpenShiftCluster. Le modèle est référencé directement à partir du dépôt GitHub, et un fichier de paramètres locaux nommé azuredeploy.parameters.json est utilisé.

```azurecli 
az deployment group create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

La durée du déploiement varie en fonction du nombre total de nœuds déployés et des options configurées, avec un minimum de 60 minutes. Le nom de domaine complet du DNS Bastion et l’URL de la console OpenShift s’affichent dans le terminal à la fin du déploiement.

```json
{
  "Bastion DNS FQDN": "bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com",
  "OpenShift Console URL": "http://openshiftlb.eastus.cloudapp.azure.com/console"
}
```

Si vous ne souhaitez pas lier la ligne de commande d’attente de la fin du déploiement, ajoutez `--no-wait` aux options pour le déploiement du groupe. La sortie du déploiement peut être récupérée à partir du portail Azure, dans la section du déploiement du groupe de ressources.

## <a name="connect-to-the-openshift-cluster"></a>Se connecter au cluster OpenShift

Une fois le déploiement terminé, récupérez la connexion à partir de la section de sortie du déploiement. Connectez-vous à la console OpenShift dans un navigateur à l’aide de l’**URL de la console OpenShift**. Vous pouvez également établir une connexion SSH à l’hôte Bastion. Dans l’exemple suivant, le nom d’utilisateur administrateur est clusteradmin et le nom de domaine complet du DNS d’adresses IP publiques Bastion est bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com :

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez la commande [az group delete](/cli/azure/group) pour supprimer le groupe de ressources, le cluster OpenShift et toutes les ressources associées quand vous n’en avez plus besoin.

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Étapes suivantes

- [Tâches de post-déploiement](./openshift-container-platform-3x-post-deployment.md)
- [Résoudre les problèmes de déploiement d’OpenShift dans Azure](./openshift-container-platform-3x-troubleshooting.md)
- [Prise en main d’OpenShift Container Platform](https://docs.openshift.com)
