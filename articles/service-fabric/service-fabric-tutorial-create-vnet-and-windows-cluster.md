---
title: Créer un cluster Service Fabric exécutant Windows dans Azure
description: Dans ce tutoriel, vous allez découvrir comment déployer un cluster Windows Service Fabric dans un réseau virtuel Azure et un groupe de sécurité réseau à l’aide de PowerShell.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: a7390858e55a456ec5fb2f851be1a7443be97082
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86245039"
---
# <a name="tutorial-deploy-a-service-fabric-cluster-running-windows-into-an-azure-virtual-network"></a>Tutoriel : Déployer un cluster Service Fabric exécutant Windows sur un réseau virtuel Azure

Ce tutoriel est la première partie d’une série d’étapes. Vous allez découvrir comment déployer un cluster Azure Service Fabric exécutant Windows dans un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) et un [groupe de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md) à l’aide de PowerShell et d’un modèle. Quand vous avez terminé, vous disposez d’un cluster en cours d’exécution dans le cloud sur lequel vous pouvez déployer des applications. Pour créer un cluster Linux qui utilise Azure CLI, consultez [Créer un cluster Linux sécurisé sur Azure](service-fabric-tutorial-create-vnet-and-linux-cluster.md).

Ce didacticiel décrit un scénario de production. Si vous souhaitez créer un petit cluster à des fins de test, consultez [Créer un cluster de test](./scripts/service-fabric-powershell-create-secure-cluster-cert.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel dans Azure à l’aide de PowerShell
> * Créer un coffre de clés et charger un certificat
> * Configurer l’authentification Azure Active Directory
> * Configurer la collecte de diagnostics
> * Configurer le service EventStore
> * Configurer les journaux Azure Monitor
> * Créer un cluster Service Fabric sécurisé dans Azure PowerShell
> * Sécuriser le cluster avec un certificat X.509
> * Se connecter à un cluster à l’aide de PowerShell
> * Supprimer un cluster

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer un cluster sécurisé sur Azure
> * [Superviser un cluster](service-fabric-tutorial-monitor-cluster.md)
> * [Mettre à l’échelle un cluster](service-fabric-tutorial-scale-cluster.md)
> * [Mettre à niveau le runtime d’un cluster](service-fabric-tutorial-upgrade-cluster.md)
> * [Supprimer un cluster](service-fabric-tutorial-delete-cluster.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Installez le [Kit de développement logiciel (SDK) Service Fabric et le module PowerShell](service-fabric-get-started.md).
* Installez [Azure PowerShell](/powershell/azure/install-az-ps).
* Passez en revue les concepts clés des [clusters Azure](service-fabric-azure-clusters-overview.md).
* [Planifiez et préparez](service-fabric-cluster-azure-deployment-preparation.md) un déploiement de cluster de production.

Les procédures suivantes créent un cluster Service Fabric à sept nœuds. Utilisez la [calculatrice de prix Azure](https://azure.microsoft.com/pricing/calculator/) pour calculer le coût lié à l’exécution d’un cluster Service Fabric dans Azure.

## <a name="download-and-explore-the-template"></a>Télécharger et explorer le modèle

Téléchargez les fichiers de modèle Azure Resource Manager suivants :

* [azuredeploy.json][template]
* [azuredeploy.parameters.json][parameters]

Ce modèle déploie un cluster sécurisé de sept machines virtuelles et trois types de nœuds dans un réseau virtuel et un groupe de sécurité réseau.  D’autres exemples de modèles sont disponibles sur [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). Le modèle [azuredeploy.json][template] déploie un certain nombre de ressources, notamment celles ci-dessous.

### <a name="service-fabric-cluster"></a>Cluster Service Fabric

Dans la ressource **Microsoft.servicefabric/clusters**, un cluster Windows est configuré avec les caractéristiques suivantes :

* Trois types de nœuds
* Cinq nœuds dans le type de nœud principal (configurables dans les paramètres du modèle) et un nœud dans chacun des deux autres types
* Système d’exploitation : Windows Server 2016 Datacenter avec Containers (configurables dans les paramètres du modèle)
* Sécurisation par certificat (configurable dans les paramètres du modèle)
* [Proxy inverse](service-fabric-reverseproxy.md) activé
* [Service DNS](service-fabric-dnsservice.md) activé
* [Niveau de durabilité](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) Bronze (configurable dans les paramètres du modèle)
* [Niveau de fiabilité](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) Silver (configurable dans les paramètres du modèle)
* Point de terminaison de connexion client : 19000 (configurable dans les paramètres du modèle)
* point de terminaison de passerelle HTTP : 19080 (configurable dans les paramètres du modèle)

### <a name="azure-load-balancer"></a>Azure Load Balancer

Dans la ressource **Microsoft.Network/loadBalancers**, un équilibreur de charge est configuré. Des sondes et règles sont configurées pour les ports suivants :

* Point de terminaison de connexion client : 19000
* point de terminaison de passerelle HTTP : 19080
* Port de l’application : 80
* Port de l’application : 443
* proxy inverse de Service Fabric : 19081

Si d’autres ports de l’application sont nécessaires, vous devez ajuster les ressources **Microsoft.Network/loadBalancers** et **Microsoft.Network/networkSecurityGroups** pour autoriser le trafic entrant.

### <a name="virtual-network-subnet-and-network-security-group"></a>Réseau virtuel, sous-réseau et groupe de sécurité réseau

Les noms du réseau virtuel, du sous-réseau et du groupe de sécurité réseau sont déclarés dans les paramètres du modèle. Les espaces d’adressage du réseau virtuel et du sous-réseau sont également déclarés dans les paramètres de modèle et configurés dans la ressource **Microsoft.Network/virtualNetworks** :

* Espace d’adressage du réseau virtuel : 172.16.0.0/20
* Espace d’adressage du sous-réseau Service Fabric : 172.16.2.0/23

Les règles de trafic entrant suivantes sont activées dans la ressource **Microsoft.Network/networkSecurityGroups**. Vous pouvez modifier les valeurs de port en modifiant les variables de modèle.

* ClientConnectionEndpoint (TCP) : 19000
* HttpGatewayEndpoint (HTTP/TCP) : 19080
* SMB : 445
* Internodecommunication : 1025, 1026, 1027
* Plage de ports éphémères : 49152 à 65534 (besoin d’un minimum de 256 ports)
* Ports pour l’utilisation de l’application : 80 et 443
* Plage de ports de l’application : 49152 à 65534 (utilisée pour la communication entre les services). Les autres ports ne sont pas ouverts sur l’équilibreur de charge)
* Bloquer tous les autres ports

Si d’autres ports de l’application sont nécessaires, vous devez ajuster les ressources **Microsoft.Network/loadBalancers** et **Microsoft.Network/networkSecurityGroups** pour autoriser le trafic entrant.

### <a name="windows-defender"></a>Windows Defender
Par défaut, le [programme antivirus Windows Defender](/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-on-windows-server-2016) est installé sur Windows Server 2016 et fonctionne correctement. L’interface utilisateur est installée par défaut sur certaines références SKU, mais elle n’est pas obligatoire. Pour tous les types de nœuds ou groupes de machines virtuelles identiques déclarés dans le modèle, l’[extension Azure VM Antimalware](../virtual-machines/extensions/iaas-antimalware-windows.md) est utilisée pour exclure les répertoires et les processus Service Fabric :

```json
{
"name": "[concat('VMIaaSAntimalware','_vmNodeType0Name')]",
"properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "IaaSAntimalware",
        "typeHandlerVersion": "1.5",
        "settings": {
        "AntimalwareEnabled": "true",
        "Exclusions": {
                "Paths": "D:\\SvcFab;D:\\SvcFab\\Log;C:\\Program Files\\Microsoft Service Fabric",
                "Processes": "Fabric.exe;FabricHost.exe;FabricInstallerService.exe;FabricSetup.exe;FabricDeployer.exe;ImageBuilder.exe;FabricGateway.exe;FabricDCA.exe;FabricFAS.exe;FabricUOS.exe;FabricRM.exe;FileStoreService.exe"
        },
        "RealtimeProtectionEnabled": "true",
        "ScheduledScanSettings": {
                "isEnabled": "true",
                "scanType": "Quick",
                "day": "7",
                "time": "120"
        }
        },
        "protectedSettings": null
}
}
```

## <a name="set-template-parameters"></a>Définir les paramètres de modèle

Le fichier de paramètres [azuredeploy.parameters.json][parameters] déclare de nombreuses valeurs servant à déployer le cluster et les ressources associées. Voici les paramètres à modifier pour votre déploiement :

**Paramètre** | **Exemple de valeur** | **Remarques** 
|---|---|---|
|adminUsername|vmadmin| Nom d’utilisateur administrateur pour les machines virtuelles de cluster. [Conditions requises pour les noms d’utilisateur de la machine virtuelle](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm). |
|adminPassword|Password#1234| Mot de passe d’administrateur pour les machines virtuelles de cluster. [Conditions requises pour les mots de passe de la machine virtuelle](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).|
|clusterName|mysfcluster123| Nom du cluster. Peut contenir seulement des lettres et des chiffres. Le nom peut contenir entre 3 et 23 caractères.|
|location|southcentralus| Emplacement du cluster. |
|certificateThumbprint|| <p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat.</p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez la valeur d’empreinte du certificat SHA-1. Par exemple, « 6190390162C988701DB5676EB81083EA608DCCF3 ».</p> |
|certificateUrlValue|| <p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat. </p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez l’URL du certificat. Par exemple, « https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346 ».</p>|
|sourceVaultValue||<p>La valeur doit être vide si vous créez un certificat auto-signé ou si vous fournissez un fichier de certificat.</p><p>Pour utiliser un certificat existant déjà chargé dans un coffre de clés, renseignez la valeur de coffre source. Par exemple, « /subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT ».</p>|

## <a name="set-up-azure-active-directory-client-authentication"></a>Configurer l’authentification client Azure Active Directory
Pour les clusters Service Fabric déployés dans un réseau public hébergé dans Azure, les recommandations concernant l’authentification mutuelle client à nœud sont les suivantes :
* Utiliser Azure Active Directory pour l’identité client
* Utiliser un certificat pour l’identité du serveur et le chiffrement TLS de la communication HTTP

La configuration d’Azure Active Directory (Azure AD) pour authentifier les clients d’un cluster Service Fabric doit être effectuée avant de [créer le cluster](#createvaultandcert). Azure AD permet aux organisation (appelées locataires) de gérer l’accès utilisateur aux applications. 

Un cluster Service Fabric offre plusieurs points d’entrée pour ses fonctionnalités de gestion, notamment les outils web [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)et [Visual Studio](service-fabric-manage-application-in-visual-studio.md). Par conséquent, vous allez créer deux applications Azure AD pour contrôler l’accès au cluster : une application web et une application native.  Après avoir créé les applications, vous devez affecter les utilisateurs aux rôles en lecture seule et administrateur.

> [!NOTE]
> Vous devez exécuter les étapes suivantes avant de créer le cluster. Étant donné que les scripts attendent des noms de cluster et des points de terminaison, ces valeurs doivent être des valeurs planifiées et non celles que vous avez déjà créées.

Dans cet article, nous partons du principe que vous avez déjà créé un locataire. Si ce n’est pas le cas, commencez par lire [Guide pratique pour obtenir un locataire Azure Active Directory](../active-directory/develop/quickstart-create-new-tenant.md).

Pour simplifier les étapes de configuration d’Azure AD avec un cluster Service Fabric, nous avons créé un ensemble de scripts Windows PowerShell. [Téléchargez les scripts](https://github.com/Azure-Samples/service-fabric-aad-helpers) sur votre ordinateur.

### <a name="create-azure-ad-applications-and-assign-users-to-roles"></a>Créer des applications Azure AD et attribuer des rôles aux utilisateurs
Créez deux applications Azure AD pour contrôler l'accès au cluster : une application web et une application native. Une fois que vous avez créé les applications pour représenter votre cluster, attribuez à vos utilisateurs les [rôles pris en charge par Service Fabric](service-fabric-cluster-security-roles.md) : lecture seule et administrateur.

Exécutez `SetupApplications.ps1`, puis entrez l'ID du locataire, le nom du cluster et l'URL de réponse de l'application web en tant que paramètres. Spécifiez les noms d’utilisateur et les mots de passe des utilisateurs. Par exemple :

```powershell
$Configobj = .\SetupApplications.ps1 -TenantId '<MyTenantID>' -ClusterName 'mysfcluster123' -WebApplicationReplyUrl 'https://mysfcluster123.eastus.cloudapp.azure.com:19080/Explorer/index.html' -AddResourceAccess
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestUser' -Password 'P@ssword!123'
.\SetupUser.ps1 -ConfigObj $Configobj -UserName 'TestAdmin' -Password 'P@ssword!123' -IsAdmin
```

> [!NOTE]
> Pour les clouds nationaux (par exemple, Azure Government, Azure Chine, Azure Allemagne), spécifiez le paramètre `-Location`.

Vous trouverez votre *ID de locataire*, ou ID de répertoire, dans le [portail Azure](https://portal.azure.com). Sélectionnez **Azure Active Directory** > **Propriétés**, puis copiez la valeur **ID de répertoire**.

Le paramètre *ClusterName* est utilisé pour préfixer les applications Azure AD créées par le script. Il ne doit pas forcément correspondre précisément au nom du cluster. Il facilite seulement la mise en correspondance des artefacts Azure AD avec le cluster Service Fabric en cours d’utilisation.

Le paramètre *WebApplicationReplyUrl* est le point de terminaison par défaut qu'Azure AD renvoie à vos utilisateurs après leur connexion. Définissez ce point de terminaison en tant que point de terminaison Service Fabric Explorer pour votre cluster, qui est par défaut :

https://&lt;cluster_domain&gt;:19080/Explorer

Vous êtes invité à vous connecter à un compte qui dispose de privilèges d’administration pour le locataire Azure AD. Une fois que vous vous êtes connecté, le script crée les applications web et native pour représenter votre cluster Service Fabric. Dans les applications du locataire dans le [portail Azure](https://portal.azure.com), vous devez voir deux nouvelles entrées :

   * *ClusterName*\_Cluster
   * *ClusterName*\_Client

Comme le script imprime le code JSON exigé par le modèle Resource Manager quand vous créez le cluster, il est préférable de garder la fenêtre PowerShell ouverte.

```json
"azureActiveDirectory": {
  "tenantId":"<guid>",
  "clusterApplication":"<guid>",
  "clientApplication":"<guid>"
},
```

### <a name="add-azure-ad-configuration-to-use-azure-ad-for-client-access"></a>Ajouter la configuration Azure AD afin d’utiliser Azure AD pour l’accès client
Dans [azuredeploy.json][template], configurez Azure AD dans la section **Microsoft.ServiceFabric/clusters**. Ajoutez des paramètres pour l’ID du locataire, l’ID de l’application de cluster et l’ID de l’application cliente.  

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...

    "aadTenantId": {
      "type": "string",
      "defaultValue": "0e3d2646-78b3-4711-b8be-74a381d9890c"
    },
    "aadClusterApplicationId": {
      "type": "string",
      "defaultValue": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
    },
    "aadClientApplicationId": {
      "type": "string",
      "defaultValue": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
    }
  },

...

{
  "apiVersion": "2018-02-01",
  "type": "Microsoft.ServiceFabric/clusters",
  "name": "[parameters('clusterName')]",
  ...
  "properties": {
    ...
    "azureActiveDirectory": {
      "tenantId": "[parameters('aadTenantId')]",
      "clusterApplication": "[parameters('aadClusterApplicationId')]",
      "clientApplication": "[parameters('aadClientApplicationId')]"
    },
    ...
  }
}
```

Ajoutez les valeurs des paramètres dans le fichier de paramètres [azuredeploy.parameters.json][parameters]. Par exemple :

```json
"aadTenantId": {
"value": "0e3d2646-78b3-4711-b8be-74a381d9890c"
},
"aadClusterApplicationId": {
"value": "cb147d34-b0b9-4e77-81d6-420fef0c4180"
},
"aadClientApplicationId": {
"value": "7a8f3b37-cc40-45cc-9b8f-57b8919ea461"
}
```
<a id="configurediagnostics" name="configurediagnostics_anchor"></a>

## <a name="configure-diagnostics-collection-on-the-cluster"></a>Configurer la collecte de diagnostics sur le cluster
Quand vous exécutez un cluster Service Fabric, il peut être intéressant de recueillir les journaux de tous les nœuds pour les regrouper dans un emplacement central. La centralisation des journaux d’activité vous permet d’analyser et résoudre les problèmes que vous pourriez rencontrer dans votre cluster ou dans les applications et services exécutés dans ce cluster.

Pour charger et recueillir des journaux, vous pouvez utiliser l’extension Diagnostics Azure pour Windows (WAD), qui charge les journaux dans Stockage Azure, ou envoyer les journaux à Azure Application Insights ou Event Hubs. Vous pouvez également utiliser un processus externe pour lire les événements à partir du stockage et les placer dans une plateforme d’analyse comme les journaux Azure Monitor ou une autre solution d’analyse des journaux.

Si vous suivez ce tutoriel, la collecte des diagnostics est déjà configurée dans le [modèle][template].

Si l’extension Diagnostics n’est pas déployée sur l’un de vos clusters existants, vous pouvez l’ajouter ou la mettre à jour par le biais du modèle de cluster. Modifiez le modèle Resource Manager utilisé pour créer le cluster existant ou téléchargez le modèle à partir du portail. Modifiez le fichier template.json en effectuant les opérations suivantes :

Ajoutez une nouvelle ressource de stockage à la section resources dans le modèle :
```json
"resources": [
...
{
  "apiVersion": "2015-05-01-preview",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
  "location": "[parameters('computeLocation')]",
  "sku": {
    "accountType": "[parameters('applicationDiagnosticsStorageAccountType')]"
  },
  "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
...
]
```

Ensuite, ajoutez des paramètres pour le type et le nom du compte de stockage à la section parameters du modèle. Remplacez le texte de l’espace réservé storage account name goes here par le nom du compte de stockage de votre choix.

```json
"parameters": {
...
"applicationDiagnosticsStorageAccountType": {
    "type": "string",
    "allowedValues": [
    "Standard_LRS",
    "Standard_GRS"
    ],
    "defaultValue": "Standard_LRS",
    "metadata": {
    "description": "Replication option for the application diagnostics storage account"
    }
},
"applicationDiagnosticsStorageAccountName": {
    "type": "string",
    "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
    "metadata": {
    "description": "Name for the storage account that contains application diagnostics data from the cluster"
    }
},
...
}
```

Ensuite, ajoutez l’extension **IaaSDiagnostics** au tableau d’extensions de la propriété **VirtualMachineProfile** de chaque ressource **Microsoft.Compute/virtualMachineScaleSets** dans le cluster.  Si vous utilisez l’[exemple de modèle][template], il existe trois groupes de machines virtuelles identiques (un pour chaque type de nœud dans le cluster).

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
                    "properties": {
                        "type": "IaaSDiagnostics",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
                        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
                        "storageAccountEndPoint": "https://core.windows.net/"
                        },
                        "publisher": "Microsoft.Azure.Diagnostics",
                        "settings": {
                        "WadCfg": {
                            "DiagnosticMonitorConfiguration": {
                            "overallQuotaInMB": "50000",
                            "EtwProviders": {
                                "EtwEventSourceProviderConfiguration": [
                                {
                                    "provider": "Microsoft-ServiceFabric-Actors",
                                    "scheduledTransferKeywordFilter": "1",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricReliableActorEventTable"
                                    }
                                },
                                {
                                    "provider": "Microsoft-ServiceFabric-Services",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricReliableServiceEventTable"
                                    }
                                }
                                ],
                                "EtwManifestProviderConfiguration": [
                                {
                                    "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                                    "scheduledTransferLogLevelFilter": "Information",
                                    "scheduledTransferKeywordFilter": "4611686018427387904",
                                    "scheduledTransferPeriod": "PT5M",
                                    "DefaultEvents": {
                                    "eventDestination": "ServiceFabricSystemEventTable"
                                    }
                                }
                                ]
                            }
                            }
                        },
                        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
                        },
                        "typeHandlerVersion": "1.5"
                    }
                }
            ...
            ]
        }
    }
}
```
<a id="configureeventstore" name="configureeventstore_anchor"></a>

## <a name="configure-the-eventstore-service"></a>Configurer le service EventStore
Le service EventStore est une option de supervision de Service Fabric. EventStore offre un moyen de comprendre l’état de votre cluster ou de vos charges de travail à un moment donné dans le temps. EventStore est un service Service Fabric avec état qui conserve les événements du cluster. Les événements sont exposés via Service Fabric Explorer, REST et les API. EventStore interroge le cluster directement pour obtenir des données de diagnostic sur une entité de votre cluster et doit être utilisé pour aider à :

* Diagnostiquer les problèmes de développement ou de test, ou lorsque vous utilisez peut-être un pipeline de surveillance.
* Vérifier que les actions de gestion que vous entreprenez sur votre cluster sont traitées correctement
* Obtenir un « instantané » de la façon dont interagit Service Fabric avec une entité en particulier



Pour activer le service EventStore sur votre cluster, ajoutez le code suivant à la propriété **fabricSettings** de la ressource **Microsoft.ServiceFabric/clusters** :

```json
"apiVersion": "2018-02-01",
"type": "Microsoft.ServiceFabric/clusters",
"name": "[parameters('clusterName')]",
"properties": {
    ...
    "fabricSettings": [
        ...
        {
            "name": "EventStoreService",
            "parameters": [
                {
                "name": "TargetReplicaSetSize",
                "value": "3"
                },
                {
                "name": "MinReplicaSetSize",
                "value": "1"
                }
            ]
        }
    ]
}
```
<a id="configureloganalytics" name="configureloganalytics_anchor"></a>

## <a name="set-up-azure-monitor-logs-for-the-cluster"></a>Configurer les journaux Azure Monitor pour le cluster

Nous vous recommandons d’utiliser les journaux Azure Monitor pour superviser les événements au niveau du cluster. Pour configurer les journaux Azure Monitor afin de superviser votre cluster, vous devez [activer les diagnostics pour voir les événements au niveau du cluster](#configure-diagnostics-collection-on-the-cluster).  

L’espace de travail a besoin d’être connecté aux données de diagnostic provenant de votre cluster.  Ces données de journaux sont stockées dans le compte de stockage *applicationDiagnosticsStorageAccountName*, dans les tables WADServiceFabric*EventTable, WADWindowsEventLogsTable et WADETWEventTable.

Ajoutez l’espace de travail Azure Log Analytics et ajoutez la solution à l’espace de travail :

```json
"resources": [
    ...
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[parameters('omsWorkspacename')]",
        "type": "Microsoft.OperationalInsights/workspaces",
        "properties": {
            "sku": {
                "name": "Free"
            }
        },
        "resources": [
            {
                "apiVersion": "2015-11-01-preview",
                "name": "[concat(variables('applicationDiagnosticsStorageAccountName'),parameters('omsWorkspacename'))]",
                "type": "storageinsightconfigs",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]",
                    "[concat('Microsoft.Storage/storageAccounts/', variables('applicationDiagnosticsStorageAccountName'))]"
                ],
                "properties": {
                    "containers": [],
                    "tables": [
                        "WADServiceFabric*EventTable",
                        "WADWindowsEventLogsTable",
                        "WADETWEventTable"
                    ],
                    "storageAccount": {
                        "id": "[resourceId('Microsoft.Storage/storageaccounts/', variables('applicationDiagnosticsStorageAccountName'))]",
                        "key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('applicationDiagnosticsStorageAccountName')),'2015-06-15').key1]"
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Memory",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Available MBytes"
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "type": "datasources",
                "name": "sampleWindowsPerfCounter2",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
                ],
                "kind": "WindowsPerformanceCounter",
                "properties": {
                    "objectName": "Service Fabric Service",
                    "instanceName": "*",
                    "intervalSeconds": 10,
                    "counterName": "Average milliseconds per request"
                }
            }
        ]
    },
    {
        "apiVersion": "2015-11-01-preview",
        "location": "[parameters('omsRegion')]",
        "name": "[variables('solution')]",
        "type": "Microsoft.OperationsManagement/solutions",
        "dependsOn": [
            "[concat('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        ],
        "properties": {
            "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename'))]"
        },
        "plan": {
            "name": "[variables('solution')]",
            "publisher": "Microsoft",
            "product": "[Concat('OMSGallery/', variables('solutionName'))]",
            "promotionCode": ""
        }
    }
]
```

Ensuite, ajoutez des paramètres :
```json
"parameters": {
    ...
    "omsWorkspacename": {
        "type": "string",
        "defaultValue": "mysfomsworkspace",
        "metadata": {
            "description": "Name of your OMS Log Analytics Workspace"
        }
    },
    "omsRegion": {
        "type": "string",
        "defaultValue": "West Europe",
        "allowedValues": [
            "West Europe",
            "East US",
            "Southeast Asia"
        ],
        "metadata": {
            "description": "Specify the Azure Region for your OMS workspace"
        }
    }
}
```

Ensuite, ajoutez des variables :
```json
"variables": {
    ...
    "solution": "[Concat('ServiceFabric', '(', parameters('omsWorkspacename'), ')')]",
    "solutionName": "ServiceFabric"
}
```

Ajoutez l’extension de l’agent Log Analytics à chaque groupe de machines virtuelles identiques dans le cluster, et connectez l’agent à l’espace de travail Log Analytics. Cela permet de collecter des données de diagnostic sur les conteneurs, les applications et les performances. Si vous l’ajoutez en tant qu’extension à la ressource de groupe de machines virtuelles identiques, Azure Resource Manager veille à l’installer sur tous les nœuds, même lors de la mise à l’échelle du cluster.

```json
"apiVersion": "2018-10-01",
"type": "Microsoft.Compute/virtualMachineScaleSets",
"name": "[variables('vmNodeType1Name')]",
"properties": {
    ...
    "virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat(variables('vmNodeType0Name'),'OMS')]",
                    "properties": {
                        "publisher": "Microsoft.EnterpriseCloud.Monitoring",
                        "type": "MicrosoftMonitoringAgent",
                        "typeHandlerVersion": "1.0",
                        "autoUpgradeMinorVersion": true,
                        "settings": {
                            "workspaceId": "[reference(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')), '2015-11-01-preview').customerId]"
                        },
                        "protectedSettings": {
                            "workspaceKey": "[listKeys(resourceId('Microsoft.OperationalInsights/workspaces/', parameters('omsWorkspacename')),'2015-11-01-preview').primarySharedKey]"
                        }
                    }
                }
            ...
            ]
        }
    }
}
```

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Déployer le réseau virtuel et le cluster

Puis, configurez la topologie de réseau et déployez le cluster Service Fabric. Le modèle Resource Manager [azuredeploy.json][template] crée un réseau virtuel, un sous-réseau et un groupe de sécurité réseau pour Service Fabric. Le modèle déploie également un cluster avec la sécurité de certificat activée. Pour les clusters de production, utilisez un certificat délivré par une autorité de certification en tant que certificat de cluster. Un certificat auto-signé peut être utilisé pour garantir la sécurité des clusters de test.

Dans cet article, le modèle déploie un cluster qui utilise l’empreinte numérique du certificat pour identifier le certificat de cluster. Deux certificats ne peuvent pas avoir la même empreinte numérique, ce qui complique leur gestion. Le fait de basculer un cluster déployé des empreintes de certificat aux noms communs de certificat simplifie la gestion des certificats. Pour découvrir comment mettre à jour le cluster pour que celui-ci utilise des noms communs de certificat pour la gestion des certificats, consultez [Modifier un cluster pour qu’il passe de l’utilisation d’une empreinte de certificat à l’utilisation d’un nom commun](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-by-using-an-existing-certificate"></a>Créer un cluster à l’aide d’un certificat existant

Le script suivant utilise l’applet de commande [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) et un modèle pour déployer un nouveau cluster dans Azure. L’applet de commande crée un coffre de clés dans Azure et charge votre certificat.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$clustername = "mysfcluster123"  # Must match the clustername parameter in the template
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateFile $certpath
```

### <a name="create-a-cluster-by-using-a-new-self-signed-certificate"></a>Créer un cluster à l’aide d’un nouveau certificat auto-signé

Le script suivant utilise l’applet de commande [New-AzServiceFabricCluster](/powershell/module/az.servicefabric/New-azServiceFabricCluster) et un modèle pour déployer un nouveau cluster dans Azure. L’applet de commande crée un coffre de clés dans Azure, ajoute un nouveau certificat auto-signé dans le coffre de clés, puis télécharge le fichier de certificat localement.

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # Must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$certfolder="c:\mycertificates\"
$clustername = "mysfcluster123"
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# Sign in to your Azure account and select your subscription
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>

# Create a new resource group for your deployment, and give it a name and a location.
New-AzResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-CertificateOutputFolder $certfolder -KeyVaultName $vaultname -KeyVaultResourceGroupName $vaultgroupname -CertificateSubjectName $subname

```

## <a name="connect-to-the-secure-cluster"></a>Se connecter à un cluster sécurisé

Connectez-vous au cluster à l’aide du module Service Fabric PowerShell installé avec le SDK Service Fabric.  Tout d’abord, installez le certificat dans le magasin personnel de l’utilisateur actuel sur votre ordinateur. Exécutez la commande PowerShell suivante :

```powershell
$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\mycertificates\mysfcluster20170531104310.pfx `
        -Password $certpwd
```

Vous êtes maintenant prêt à vous connecter à votre cluster sécurisé.

Le module **Service Fabric** PowerShell fournit de nombreuses cmdlets pour la gestion des services, applications et clusters Service Fabric. Pour vous connecter au cluster sécurisé, utilisez la cmdlet [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster). Les détails du point de terminaison de connexion et de l’empreinte de certificat SHA-1 se trouvent dans la sortie de l’étape précédente.

Si vous avez précédemment configuré l’authentification du client Azure AD, exécutez la commande suivante : 
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
        -KeepAliveIntervalInSec 10 `
        -AzureActiveDirectory `
        -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10
```

Si vous n’avez pas configuré l’authentification du client Azure AD, exécutez la commande suivante :
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -FindType FindByThumbprint -FindValue C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -StoreLocation CurrentUser -StoreName My
```

Vérifiez que vous êtes connecté et que le cluster est sain à l’aide de l’applet de commande [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth).

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les autres articles de cette série de tutoriels utilisent le cluster que vous avez créé. Si vous ne passez pas immédiatement à l’article suivant, vous souhaiterez peut-être [supprimer le cluster](./service-fabric-tutorial-delete-cluster.md) pour éviter de subir des frais.

## <a name="next-steps"></a>Étapes suivantes

Passez au tutoriel suivant pour découvrir comment mettre à l’échelle votre cluster.

> [!div class="checklist"]
> * Créer un réseau virtuel dans Azure à l’aide de PowerShell
> * Créer un coffre de clés et charger un certificat
> * Configurer l’authentification Azure Active Directory
> * Configurer la collecte de diagnostics
> * Configurer le service EventStore
> * Configurer les journaux Azure Monitor
> * Créer un cluster Service Fabric sécurisé dans Azure PowerShell
> * Sécuriser le cluster avec un certificat X.509
> * Se connecter à un cluster à l’aide de PowerShell
> * Supprimer un cluster

Maintenant, passez au tutoriel suivant pour découvrir comment superviser votre cluster.
> [!div class="nextstepaction"]
> [Superviser un cluster](service-fabric-tutorial-monitor-cluster.md)

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Windows-3-NodeTypes-Secure-NSG/AzureDeploy.Parameters.json
