---
title: Ajouter ou supprimer des nœuds dans un cluster Service Fabric autonome
description: Apprenez à ajouter ou supprimer des nœuds d’un cluster Azure Service Fabric sur une machine physique ou virtuelle sous Windows Server, qu’elle soit locale ou dans un cloud.
ms.topic: conceptual
ms.date: 11/02/2017
ms.openlocfilehash: 3e5f32274d2263bc5bf1bbec8f1626d519f8ca3f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842918"
---
# <a name="add-or-remove-nodes-to-a-standalone-service-fabric-cluster-running-on-windows-server"></a>Ajouter ou supprimer des nœuds d’un cluster Service Fabric autonome sous Windows Server
Une fois que vous avez [créé votre cluster Service Fabric autonome sur des ordinateurs Windows Server](service-fabric-cluster-creation-for-windows-server.md), les besoins de votre entreprise peuvent évoluer et vous obliger à ajouter ou supprimer des nœuds dans votre cluster, comme décrit dans cet article.

> [!NOTE]
> La fonctionnalité d’ajout et de suppression de nœud n’est pas prise en charge dans les clusters de développement locaux.

## <a name="add-nodes-to-your-cluster"></a>Ajouter des nœuds à votre cluster

1. Préparez l’ordinateur ou la machine virtuelle que vous souhaitez ajouter à votre cluster en suivant les étapes décrites dans [Planifier et préparer votre déploiement de cluster Service Fabric](service-fabric-cluster-standalone-deployment-preparation.md).

2. Identifiez le domaine d’erreur et le domaine de mise à niveau auxquels vous allez ajouter cette machine virtuelle ou cet ordinateur.

   Si vous utilisez des certificats pour sécuriser le cluster, ils sont censés être installés dans les magasins de certificats locaux pour préparer la jonction du nœud au cluster. La même chose s’applique quand vous utilisez d’autres formes de sécurité.

3. Avec Bureau à distance (RDP), accédez à la machine virtuelle ou à l’ordinateur que vous souhaitez ajouter au cluster.

4. Copiez ou [téléchargez le package autonome Service Fabric pour Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690) sur cette machine virtuelle ou cet ordinateur et décompressez le package.

5. Exécutez PowerShell avec des privilèges élevés, puis accédez à l’emplacement du package décompressé.

6. Exécutez le script *AddNode.ps1* avec les paramètres qui décrivent le nouveau nœud à ajouter. L’exemple suivant ajoute un nouveau nœud nommé VM5, de type NodeType0, avec l’adresse IP 182.17.34.52, à UD1 et fd:/dc1/r0. `ExistingClusterConnectionEndPoint` est un point de terminaison de connexion pour un nœud déjà présent dans le cluster existant. Il peut s’agir de l’adresse IP de *n’importe quel* nœud du cluster. 

   Non sécurisé (prototypage) :

   ```
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -AcceptEULA
   ```

   Sécurisé (basé sur un certificat) :

   ```  
   $CertThumbprint= "***********************"
    
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -X509Credential -ServerCertThumbprint $CertThumbprint  -AcceptEULA

   ```

   Une fois le script exécuté, vous pouvez vérifier si le nouveau nœud a été ajouté en exécutant la cmdlet [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps).

7. Pour garantir la cohérence entre les différents nœuds du cluster, vous devez lancer une mise à niveau de la configuration. Exécutez [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) pour obtenir le dernier fichier de configuration et ajoutez le nœud nouvellement ajouté à la section « Nodes ». Il est également recommandé de disposer en permanence de la dernière configuration de cluster disponible au cas où vous devriez redéployer un cluster de même configuration.

   ```
    {
        "nodeName": "vm5",
        "iPAddress": "182.17.34.52",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD1"
    }
   ```

8. Exécutez [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) pour commencer la mise à niveau.

   ```
   Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
   ```

   Vous pouvez surveiller la progression de la mise à niveau avec Service Fabric Explorer. Vous pouvez également exécuter [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps).

### <a name="add-nodes-to-clusters-configured-with-windows-security-using-gmsa"></a>Ajouter des nœuds aux clusters configurés avec la sécurité Windows à l’aide de gMSA
Pour les clusters configurés avec un compte de service géré de groupe (gMSA) (https://technet.microsoft.com/library/hh831782.aspx), un nouveau nœud peut être ajouté à l’aide d’une mise à niveau de la configuration :
1. Exécutez [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) sur l’un des nœuds existants pour obtenir le dernier fichier de configuration, et ajoutez des détails sur le nouveau nœud à ajouter dans la section « Nodes ». Assurez-vous que le nouveau nœud fait partie du même compte géré de groupe. Ce compte doit être un compte Administrateur sur tous les ordinateurs.

    ```
        {
            "nodeName": "vm5",
            "iPAddress": "182.17.34.52",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD1"
        }
    ```
2. Exécutez [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) pour commencer la mise à niveau.

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
    ```
    Vous pouvez surveiller la progression de la mise à niveau avec Service Fabric Explorer. Vous pouvez également exécuter [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps) pour cela.

### <a name="add-node-types-to-your-cluster"></a>Ajouter des types de nœuds à votre cluster
Pour ajouter un nouveau type de nœud, modifiez votre configuration afin de l’inclure dans la section « NodeTypes », sous « Properties », puis commencez une mise à niveau de la configuration à l’aide de [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps). Une fois la mise à niveau effectuée, vous pouvez ajouter de nouveaux nœuds à votre cluster avec ce type de nœud.

## <a name="remove-nodes-from-your-cluster"></a>Supprimer des nœuds de votre cluster
Pour supprimer un nœud d’un cluster à l’aide d’une mise à niveau de la configuration, procédez comme suit :

1. Exécutez [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) pour obtenir le dernier fichier de configuration et *supprimez* le nœud de la section « Nodes ».
Ajoutez le paramètre « NodesToBeRemoved » à la section « Setup », dans la section « FabricSettings ». La valeur indiquée sous « value » doit être une liste des noms des nœuds à supprimer, séparés par des virgules.

    ```
         "fabricSettings": [
            {
            "name": "Setup",
            "parameters": [
                {
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
                },
                {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
                },
                {
                "name": "NodesToBeRemoved",
                "value": "vm0, vm1"
                }
            ]
            }
        ]
    ```
2. Exécutez [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) pour commencer la mise à niveau.

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

    ```
    Vous pouvez surveiller la progression de la mise à niveau avec Service Fabric Explorer. Vous pouvez également exécuter [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps).

> [!NOTE]
> La suppression de nœuds peut entraîner plusieurs mises à niveau. Certains nœuds sont marqués avec la balise `IsSeedNode=”true”` et peuvent être identifiés en interrogeant le manifeste de cluster à l’aide de `Get-ServiceFabricClusterManifest`. La suppression de ces nœuds peut prendre plus de temps car, dans ce cas, les nœuds initiaux devront être déplacés. Le cluster doit conserver au moins 3 nœuds de type nœud principal.
> 
> 

### <a name="remove-node-types-from-your-cluster"></a>Supprimer des types de nœuds de votre cluster
Avant de supprimer un type de nœud, vérifiez s’il existe des nœuds qui référencent le type de nœud concerné. Supprimez ces nœuds avant de supprimer le type de nœud correspondant. Une fois que tous les nœuds correspondants sont supprimés, vous pouvez supprimer le NodeType de la configuration du cluster et commencer une mise à niveau de la configuration à l’aide de [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps).


### <a name="replace-primary-nodes-of-your-cluster"></a>Remplacer les nœuds principaux de votre cluster
Le remplacement des nœuds principaux doit être effectué un nœud à la fois, au lieu de supprimer, puis d’ajouter des nœuds par lots.


## <a name="next-steps"></a>Étapes suivantes
* [Paramètres de configuration pour un cluster Windows autonome](service-fabric-cluster-manifest.md)
* [Sécuriser un cluster autonome sur Windows à l’aide de certificats X509](service-fabric-windows-cluster-x509-security.md)
* [Créer un cluster Service Fabric autonome avec des machines virtuelles Azure sous Windows](./service-fabric-cluster-creation-via-arm.md)
