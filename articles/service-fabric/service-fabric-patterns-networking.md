---
title: Modèles de mise en réseau pour Azure Service Fabric
description: Décrit les modèles de mise en réseau courants de Service Fabric et explique comment créer un cluster avec les fonctionnalités de mise en réseau d’Azure.
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: 20bd5e931307725016c3e2ad69dae91214b2caab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87421465"
---
# <a name="service-fabric-networking-patterns"></a>Modèles de mise en réseau de Service Fabric
Vous pouvez intégrer votre cluster Azure Service Fabric avec d’autres fonctionnalités de mise en réseau Azure. Dans cet article, nous vous expliquons comment créer des clusters qui utilisent les fonctionnalités suivantes :

- [Réseau virtuel ou sous-réseau existant](#existingvnet)
- [Adresse IP publique statique](#staticpublicip)
- [Équilibreur de charge interne uniquement](#internallb)
- [Équilibreur de charge interne et externe](#internalexternallb)

Service Fabric s’exécute dans un groupe de machines virtuelles identiques standard. Toutes les fonctionnalités que vous pouvez utiliser dans un groupe de machines virtuelles identiques sont utilisables avec un cluster Service Fabric. Les sections de mise en réseau des modèles Azure Resource Manager pour Service Fabric et les groupes de machines virtuelles identiques sont strictement les mêmes. Après avoir effectué le déploiement sur un réseau virtuel existant, il est facile d’intégrer d’autres fonctionnalités de mise en réseau, comme Azure ExpressRoute, la passerelle VPN Azure, un groupe de sécurité réseau et le peering de réseau virtuel.

### <a name="allowing-the-service-fabric-resource-provider-to-query-your-cluster"></a>Autoriser le fournisseur de ressources Service Fabric à interroger le cluster

Service Fabric se distingue d’autres fonctionnalités de mise en réseau sur un aspect. Le [Portail Azure](https://portal.azure.com) utilise en interne le fournisseur de ressources Service Fabric pour effectuer des appels dans un cluster afin d’obtenir des informations sur les nœuds et les applications. Le fournisseur de ressources Service Fabric requiert un accès entrant accessible publiquement au port de la passerelle HTTP (port 19080, par défaut) sur le point de terminaison de gestion. [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) utilise le point de terminaison de gestion pour gérer votre cluster. Le fournisseur de ressources Service Fabric utilise également ce port pour demander des informations sur votre cluster, à afficher sur le Portail Azure. 

Si le port 19080 n’est pas accessible à partir du fournisseur de ressources Service Fabric, un message du type *Nœuds introuvables* apparaît sur le portail et votre liste de nœuds et d’applications apparaît vide. Si vous souhaitez voir votre cluster sur le Portail Azure, votre équilibreur de charge doit exposer une adresse IP publique et votre groupe de sécurité réseau doit autoriser le trafic entrant du port 19080. Si votre configuration ne respecte pas ces conditions, le Portail Azure n’affiche pas l’état de votre cluster.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="templates"></a>Modèles

Tous les modèles Service Fabric se trouvent dans [GitHub](https://github.com/Azure/service-fabric-scripts-and-templates/tree/master/templates/networking). Vous devriez pouvoir déployer les modèles tels quels avec les commandes PowerShell suivantes. Si vous déployez le modèle de réseau virtuel Azure existant ou le modèle d’adresse IP publique statique, commencez par lire la section [Configuration initiale](#initialsetup) de cet article.

<a id="initialsetup"></a>
## <a name="initial-setup"></a>Configuration initiale

### <a name="existing-virtual-network"></a>Réseau virtuel existant

Dans l’exemple suivant, nous commençons avec un réseau virtuel existant nommé ExistingRG-vnet, dans le groupe de ressources **ExistingRG**. Le sous-réseau est nommé default. Ces ressources par défaut sont créées lors de l’élaboration d’une machine virtuelle standard avec le Portail Azure. Vous pouvez créer le réseau virtuel et le sous-réseau sans créer la machine virtuelle, mais l’objectif principal de l’ajout d’un cluster à un réseau virtuel existant est de fournir une connectivité réseau à d’autres machines virtuelles. La création de la machine virtuelle offre un bon exemple d’utilisation courante d’un réseau virtuel existant. Si votre cluster Service Fabric utilise seulement un équilibreur de charge interne sans adresse IP publique, vous pouvez utiliser la machine virtuelle et son adresse IP publique comme *jump box* sécurisée.

### <a name="static-public-ip-address"></a>Adresse IP publique statique

Une adresse IP publique statique est généralement une ressource dédiée gérée de façon distincte de la ou des machines virtuelles auxquelles elle est affectée. Elle est approvisionnée dans un groupe de ressources réseau dédié (et non dans le groupe de ressources du cluster Service Fabric proprement dit). Créez une adresse IP publique statique nommée staticIP1 dans le même groupe de ressources ExistingRG, soit sur le Portail Azure soit avec PowerShell :

```powershell
PS C:\Users\user> New-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG -Location westus -AllocationMethod Static -DomainNameLabel sfnetworking

Name                     : staticIP1
ResourceGroupName        : ExistingRG
Location                 : westus
Id                       : /subscriptions/1237f4d2-3dce-1236-ad95-123f764e7123/resourceGroups/ExistingRG/providers/Microsoft.Network/publicIPAddresses/staticIP1
Etag                     : W/"fc8b0c77-1f84-455d-9930-0404ebba1b64"
ResourceGuid             : 77c26c06-c0ae-496c-9231-b1a114e08824
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Static
IpAddress                : 40.83.182.110
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : null
DnsSettings              : {
                             "DomainNameLabel": "sfnetworking",
                             "Fqdn": "sfnetworking.westus.cloudapp.azure.com"
                           }
```

### <a name="service-fabric-template"></a>Modèle Service Fabric

Dans les exemples de cet article, nous utilisons le modèle Service Fabric template.json. Vous pouvez utiliser l’Assistant du portail standard pour télécharger le modèle sur le portail avant de créer un cluster. Vous pouvez également utiliser l’un des [exemples de modèles](https://github.com/Azure-Samples/service-fabric-cluster-templates), comme le [cluster Service Fabric à cinq nœuds sécurisé](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure).

<a id="existingvnet"></a>
## <a name="existing-virtual-network-or-subnet"></a>Réseau virtuel ou sous-réseau existant

1. Modifiez le paramètre du sous-réseau et donnez-lui le nom du sous-réseau existant, puis ajoutez deux nouveaux paramètres pour faire référence au réseau virtuel existant :

    ```json
        "subnet0Name": {
                "type": "string",
                "defaultValue": "default"
            },
            "existingVNetRGName": {
                "type": "string",
                "defaultValue": "ExistingRG"
            },

            "existingVNetName": {
                "type": "string",
                "defaultValue": "ExistingRG-vnet"
            },
            /*
            "subnet0Name": {
                "type": "string",
                "defaultValue": "Subnet-0"
            },
            "subnet0Prefix": {
                "type": "string",
                "defaultValue": "10.0.0.0/24"
            },*/
    ```

   Vous pouvez également commenter le paramètre avec le nom « virtualNetworkName », afin qu’il ne vous invite pas à entrer à deux reprises le nom du réseau virtuel dans le panneau de déploiement de cluster du portail Azure.

2. Mettez en commentaires l’attribut `nicPrefixOverride` de `Microsoft.Compute/virtualMachineScaleSets`, car vous utilisez un sous-réseau existant et que vous avez désactivé cette variable à l’étape 1.

    ```json
            /*"nicPrefixOverride": "[parameters('subnet0Prefix')]",*/
    ```

3. Modifiez la variable `vnetID` pour qu’elle pointe vers le réseau virtuel existant :

    ```json
            /*old "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",*/
            "vnetID": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingVNetRGName'), '/providers/Microsoft.Network/virtualNetworks/', parameters('existingVNetName'))]",
    ```

4. Supprimez `Microsoft.Network/virtualNetworks` de vos ressources, afin qu’Azure ne crée pas de nouveau réseau virtuel :

    ```json
    /*{
    "apiVersion": "[variables('vNetApiVersion')]",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('virtualNetworkName')]",
    "location": "[parameters('computeLocation')]",
    "properties": {
        "addressSpace": {
            "addressPrefixes": [
                "[parameters('addressPrefix')]"
            ]
        },
        "subnets": [
            {
                "name": "[parameters('subnet0Name')]",
                "properties": {
                    "addressPrefix": "[parameters('subnet0Prefix')]"
                }
            }
        ]
    },
    "tags": {
        "resourceType": "Service Fabric",
        "clusterName": "[parameters('clusterName')]"
    }
    },*/
    ```

5. Commentez le réseau virtuel à partir de l’attribut `dependsOn` de `Microsoft.Compute/virtualMachineScaleSets`, de façon à ne pas dépendre de la création d’un réseau virtuel :

    ```json
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Computer/virtualMachineScaleSets",
    "name": "[parameters('vmNodeType0Name')]",
    "location": "[parameters('computeLocation')]",
    "dependsOn": [
        /*"[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]",
        */
        "[Concat('Microsoft.Storage/storageAccounts/', variables('uniqueStringArray0')[0])]",

    ```

6. Déployez le modèle :

    ```powershell
    New-AzResourceGroup -Name sfnetworkingexistingvnet -Location westus
    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingexistingvnet -TemplateFile C:\SFSamples\Final\template\_existingvnet.json
    ```

    Après le déploiement, votre réseau virtuel doit normalement inclure les nouvelles machines virtuelles du groupe identique. Le type de nœud du groupe de machines virtuelles identiques doit afficher le réseau virtuel et le sous-réseau existants. Vous pouvez également utiliser le protocole RDP (Remote Desktop) pour accéder à la machine virtuelle qui était déjà dans le réseau virtuel, et exécuter une commande ping sur les nouvelles machines virtuelles du groupe identique :

    ```
    C:>\Users\users>ping 10.0.0.5 -n 1
    C:>\Users\users>ping NOde1000000 -n 1
    ```

Vous pouvez également consulter un [exemple qui n’est pas propre à Service Fabric](https://github.com/gbowerman/azure-myriad/tree/main/existing-vnet).


<a id="staticpublicip"></a>
## <a name="static-public-ip-address"></a>Adresse IP publique statique

1. Ajoutez les paramètres du groupe de ressources d’adresses IP statiques existant, du nom et du nom de domaine complet (FQDN) :

    ```json
    "existingStaticIPResourceGroup": {
                "type": "string"
            },
            "existingStaticIPName": {
                "type": "string"
            },
            "existingStaticIPDnsFQDN": {
                "type": "string"
    }
    ```

2. Supprimez le paramètre `dnsName`. (L’adresse IP statique en comporte déjà un.)

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

3. Ajoutez une variable pour faire référence à l’adresse IP statique existante :

    ```json
    "existingStaticIP": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', parameters('existingStaticIPResourceGroup'), '/providers/Microsoft.Network/publicIPAddresses/', parameters('existingStaticIPName'))]",
    ```

4. Supprimez `Microsoft.Network/publicIPAddresses` de vos ressources, afin qu’Azure ne crée pas de nouvelle adresse IP :

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

5. Commentez l’adresse IP à partir de l’attribut `dependsOn` de `Microsoft.Network/loadBalancers`, de façon à ne pas dépendre de la création d’une adresse IP :

    ```json
    "apiVersion": "[variables('lbIPApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    "name": "[concat('LB', '-', parameters('clusterName'), '-', parameters('vmNodeType0Name'))]",
    "location": "[parameters('computeLocation')]",
    /*
    "dependsOn": [
        "[concat('Microsoft.Network/publicIPAddresses/', concat(parameters('lbIPName'), '-', '0'))]"
    ], */
    "properties": {
    ```

6. Dans la ressource `Microsoft.Network/loadBalancers`, modifiez l’élément `publicIPAddress` de `frontendIPConfigurations` pour faire référence à l’adresse IP statique existante au lieu d’une adresse nouvellement créée :

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                "publicIPAddress": {
                                    /*"id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"*/
                                    "id": "[variables('existingStaticIP')]"
                                }
                            }
                        }
                    ],
    ```

7. Dans la ressource `Microsoft.ServiceFabric/clusters`, remplacez `managementEndpoint` par le nom de domaine complet du DNS de l’adresse IP statique. Si vous utilisez un cluster sécurisé, veillez à remplacer *http://* par *https://* . (Notez que cette étape s’applique uniquement aux clusters Service Fabric. Si vous utilisez un groupe de machines virtuelles identiques, ignorez cette étape.)

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',parameters('existingStaticIPDnsFQDN'),':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

8. Déployez le modèle :

    ```powershell
    New-AzResourceGroup -Name sfnetworkingstaticip -Location westus

    $staticip = Get-AzPublicIpAddress -Name staticIP1 -ResourceGroupName ExistingRG

    $staticip

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkingstaticip -TemplateFile C:\SFSamples\Final\template\_staticip.json -existingStaticIPResourceGroup $staticip.ResourceGroupName -existingStaticIPName $staticip.Name -existingStaticIPDnsFQDN $staticip.DnsSettings.Fqdn
    ```

Après le déploiement, votre équilibreur de charge est lié à l’adresse IP statique publique de l’autre groupe de ressources. Le point de terminaison de connexion du client Service Fabric et le point de terminaison [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) pointent vers le nom de domaine complet du DNS de l’adresse IP statique.

<a id="internallb"></a>
## <a name="internal-only-load-balancer"></a>Équilibreur de charge interne uniquement

Ce scénario remplace l’équilibreur de charge externe dans le modèle Service Fabric par défaut par un équilibreur de charge interne uniquement. Pour connaître les implications pour le Portail Azure et le fournisseur de ressources Service Fabric, voir [cette section de l’article](#allowing-the-service-fabric-resource-provider-to-query-your-cluster).

1. Supprimez le paramètre `dnsName`. (Il n’est pas nécessaire.)

    ```json
    /*
    "dnsName": {
        "type": "string"
    },
    */
    ```

2. Si vous utilisez une méthode d’allocation statique, vous pouvez éventuellement ajouter un paramètre d’adresse IP statique. Si vous utilisez une méthode d’allocation dynamique, il n’est pas nécessaire d’effectuer cette étape.

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

3. Supprimez `Microsoft.Network/publicIPAddresses` de vos ressources, afin qu’Azure ne crée pas de nouvelle adresse IP :

    ```json
    /*
    {
        "apiVersion": "[variables('publicIPApiVersion')]",
        "type": "Microsoft.Network/publicIPAddresses",
        "name": "[concat(parameters('lbIPName'),)'-', '0')]",
        "location": "[parameters('computeLocation')]",
        "properties": {
            "dnsSettings": {
                "domainNameLabel": "[parameters('dnsName')]"
            },
            "publicIPAllocationMethod": "Dynamic"        
        },
        "tags": {
            "resourceType": "Service Fabric",
            "clusterName": "[parameters('clusterName')]"
        }
    }, */
    ```

4. Supprimez l’attribut `dependsOn` de l’adresse IP de `Microsoft.Network/loadBalancers`, de façon à ne pas dépendre de la création d’une adresse IP. Ajoutez l’attribut `dependsOn` du réseau virtuel, car l’équilibreur de charge dépend à présent du sous-réseau du réseau virtuel :

    ```json
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'))]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /*"[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"*/
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
    ```

5. Modifiez le paramètre `frontendIPConfigurations` de l’équilibreur de charge : remplacez l’utilisation d’une `publicIPAddress` par l’utilisation d’un sous-réseau et d’une `privateIPAddress`. `privateIPAddress` utilise une adresse IP interne statique prédéfinie. Pour utiliser une adresse IP dynamique, supprimez l’élément `privateIPAddress`, puis donnez la valeur **Dynamic** à `privateIPAllocationMethod`.

    ```json
                "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /*
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                } */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
    ```

6. Dans la ressource `Microsoft.ServiceFabric/clusters`, modifiez `managementEndpoint` pour qu’il pointe vers l’adresse de l’équilibreur de charge interne. Si vous utilisez un cluster sécurisé, veillez à remplacer *http://* par *https://* . (Notez que cette étape s’applique uniquement aux clusters Service Fabric. Si vous utilisez un groupe de machines virtuelles identiques, ignorez cette étape.)

    ```json
                    "fabricSettings": [],
                    /*"managementEndpoint": "[concat('http://',reference(concat(parameters('lbIPName'),'-','0')).dnsSettings.fqdn,':',parameters('nt0fabricHttpGatewayPort'))]",*/
                    "managementEndpoint": "[concat('http://',reference(variables('lbID0')).frontEndIPConfigurations[0].properties.privateIPAddress,':',parameters('nt0fabricHttpGatewayPort'))]",
    ```

7. Déployez le modèle :

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternallb -TemplateFile C:\SFSamples\Final\template\_internalonlyLB.json
    ```

Après le déploiement, votre équilibreur de charge utilise l’adresse IP statique privée 10.0.0.250. Si vous avez un autre ordinateur dans le même réseau virtuel, vous pouvez accéder au point de terminaison [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) interne. Notez qu’il se connecte à l’un des nœuds derrière l’équilibreur de charge.

<a id="internalexternallb"></a>
## <a name="internal-and-external-load-balancer"></a>Équilibreur de charge interne et externe

Dans ce scénario, vous commencez avec l’équilibreur de charge externe à type de nœud unique existant et vous ajoutez un équilibreur de charge interne pour le même type de nœud. Un port back-end associé à un pool d’adresses back-end ne peut être affecté qu’à un seul équilibreur de charge. Choisissez quel équilibreur de charge doit avoir vos ports d’application et lequel doit avoir vos points de terminaison de gestion (ports 19000 et 19080). Si vous placez les points de terminaison de gestion sur l’équilibreur de charge interne, gardez à l’esprit les restrictions du fournisseur de ressources Service Fabric décrites dans [cette section de l’article](#allowing-the-service-fabric-resource-provider-to-query-your-cluster). Dans l’exemple que nous utilisons, les points de terminaison de gestion restent sur l’équilibreur de charge externe. Vous ajoutez également un port d’application 80 et le placez sur l’équilibreur de charge interne.

Dans un cluster à deux types de nœuds, l’un des types de nœuds est sur l’équilibreur de charge externe. L’autre type de nœud est sur l’équilibreur de charge interne. Pour utiliser un cluster à deux types de nœuds, modifiez le deuxième équilibreur de charge en équilibreur de charge interne dans le modèle à deux types de nœuds créé sur le portail (fourni avec deux équilibreurs de charge). Pour plus d’informations, consultez la section [Équilibreur de charge interne uniquement](#internallb).

1. Ajoutez le paramètre d’adresse IP statique de l’équilibreur de charge interne. (Pour lire les notes portant sur l’utilisation d’une adresse IP dynamique, consultez les sections précédentes de cet article.)

    ```json
            "internalLBAddress": {
                "type": "string",
                "defaultValue": "10.0.0.250"
            }
    ```

2. Ajoutez un paramètre port d’application 80.

3. Pour ajouter des versions internes des variables de mise en réseau existantes, copiez et collez-les et ajoutez « -Int » au nom :

    ```json
    /* Add internal load balancer networking variables */
            "lbID0-Int": "[resourceId('Microsoft.Network/loadBalancers', concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal'))]",
            "lbIPConfig0-Int": "[concat(variables('lbID0-Int'),'/frontendIPConfigurations/LoadBalancerIPConfig')]",
            "lbPoolID0-Int": "[concat(variables('lbID0-Int'),'/backendAddressPools/LoadBalancerBEAddressPool')]",
            "lbProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricGatewayProbe')]",
            "lbHttpProbeID0-Int": "[concat(variables('lbID0-Int'),'/probes/FabricHttpGatewayProbe')]",
            "lbNatPoolID0-Int": "[concat(variables('lbID0-Int'),'/inboundNatPools/LoadBalancerBEAddressNatPool')]",
            /* Internal load balancer networking variables end */
    ```

4. Si vous commencez avec le modèle généré par le portail qui utilise le port d’application 80, le modèle du portail par défaut ajoute AppPort1 (port 80) sur l’équilibreur de charge externe. Dans ce cas, supprimez AppPort1 des sondes et des règles `loadBalancingRules` de l’équilibreur de charge externe pour pouvoir l’ajouter à l’équilibreur de charge interne :

    ```json
    "loadBalancingRules": [
        {
            "name": "LBHttpRule",
            "properties":{
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('nt0fabricHttpGatewayPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[variables('lbHttpProbeID0')]"
                },
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortLBRule1",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('loadBalancedAppPort1')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"            
                },
                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concate(variables('lbID0'), '/probes/AppPortProbe1')]"
                },
                "protocol": "tcp"
            }
        }*/

    ],
    "probes": [
        {
            "name": "FabricGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricTcpGatewayPort')]",
                "protocol": "tcp"
            }
        },
        {
            "name": "FabricHttpGatewayProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('nt0fabricHttpGatewayPort')]",
                "protocol": "tcp"
            }
        } /* Remove AppPort1 from the external load balancer.
        {
            "name": "AppPortProbe1",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port": "[parameters('loadBalancedAppPort1')]",
                "protocol": "tcp"
            }
        } */

    ],
    "inboundNatPools": [
    ```

5. Ajoutez une deuxième ressource `Microsoft.Network/loadBalancers`. Elle ressemble à l’équilibreur de charge interne créé dans la section [Équilibreur de charge interne uniquement](#internallb), mais utilise les variables d’équilibreur de charge « -Int » et implémente uniquement le port d’application 80. Cela supprime également `inboundNatPools`, de façon à maintenir les points de terminaison RDP sur l’équilibreur de charge public. Pour qu’ils soient sur l’équilibreur de charge interne, déplacez `inboundNatPools` de l’équilibreur de charge externe vers cet équilibreur de charge interne :

    ```json
            /* Add a second load balancer, configured with a static privateIPAddress and the "-Int" load balancer variables. */
            {
                "apiVersion": "[variables('lbApiVersion')]",
                "type": "Microsoft.Network/loadBalancers",
                /* Add "-Internal" to the name. */
                "name": "[concat('LB','-', parameters('clusterName'),'-',parameters('vmNodeType0Name'), '-Internal')]",
                "location": "[parameters('computeLocation')]",
                "dependsOn": [
                    /* Remove public IP dependsOn, add vnet dependsOn
                    "[concat('Microsoft.Network/publicIPAddresses/',concat(parameters('lbIPName'),'-','0'))]"
                    */
                    "[concat('Microsoft.Network/virtualNetworks/',parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "frontendIPConfigurations": [
                        {
                            "name": "LoadBalancerIPConfig",
                            "properties": {
                                /* Switch from Public to Private IP address
                                */
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',concat(parameters('lbIPName'),'-','0'))]"
                                }
                                */
                                "subnet" :{
                                    "id": "[variables('subnet0Ref')]"
                                },
                                "privateIPAddress": "[parameters('internalLBAddress')]",
                                "privateIPAllocationMethod": "Static"
                            }
                        }
                    ],
                    "backendAddressPools": [
                        {
                            "name": "LoadBalancerBEAddressPool",
                            "properties": {}
                        }
                    ],
                    "loadBalancingRules": [
                        /* Add the AppPort rule. Be sure to reference the "-Int" versions of backendAddressPool, frontendIPConfiguration, and the probe variables. */
                        {
                            "name": "AppPortLBRule1",
                            "properties": {
                                "backendAddressPool": {
                                    "id": "[variables('lbPoolID0-Int')]"
                                },
                                "backendPort": "[parameters('loadBalancedAppPort1')]",
                                "enableFloatingIP": "false",
                                "frontendIPConfiguration": {
                                    "id": "[variables('lbIPConfig0-Int')]"
                                },
                                "frontendPort": "[parameters('loadBalancedAppPort1')]",
                                "idleTimeoutInMinutes": "5",
                                "probe": {
                                    "id": "[concat(variables('lbID0-Int'),'/probes/AppPortProbe1')]"
                                },
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "probes": [
                    /* Add the probe for the app port. */
                    {
                            "name": "AppPortProbe1",
                            "properties": {
                                "intervalInSeconds": 5,
                                "numberOfProbes": 2,
                                "port": "[parameters('loadBalancedAppPort1')]",
                                "protocol": "tcp"
                            }
                        }
                    ],
                    "inboundNatPools": [
                    ]
                },
                "tags": {
                    "resourceType": "Service Fabric",
                    "clusterName": "[parameters('clusterName')]"
                }
            },
    ```

6. Dans le `networkProfile` de la ressource `Microsoft.Compute/virtualMachineScaleSets`, ajoutez le pool d’adresses back-end interne :

    ```json
    "loadBalancerBackendAddressPools": [
                                                        {
                                                            "id": "[variables('lbPoolID0')]"
                                                        },
                                                        {
                                                            /* Add internal BE pool */
                                                            "id": "[variables('lbPoolID0-Int')]"
                                                        }
    ],
    ```

7. Déployez le modèle :

    ```powershell
    New-AzResourceGroup -Name sfnetworkinginternalexternallb -Location westus

    New-AzResourceGroupDeployment -Name deployment -ResourceGroupName sfnetworkinginternalexternallb -TemplateFile C:\SFSamples\Final\template\_internalexternalLB.json
    ```

Après le déploiement, deux équilibreurs de charge apparaissent dans le groupe de ressources. Si vous parcourez les équilibreurs de charge, vous voyez l’adresse IP publique et les points de terminaison de gestion (ports 19000 et 19080) affectés à l’adresse IP publique. L’adresse IP interne statique et le point de terminaison d’application (port 80) affectés à l’équilibreur de charge interne apparaissent également. Les deux équilibreurs de charge utilisent le même pool back-end de groupe de machines virtuelles identiques.

## <a name="notes-for-production-workloads"></a>Remarques pour les charges de travail de production

Les modèles GitHub ci-dessus sont conçus pour fonctionner avec la référence SKU par défaut pour Azure Standard Load Balancer (SLB), la référence SKU De base. Ce SLB n’a pas de contrat SLA. Pour les charges de travail de production, la référence SKU Standard doit donc être utilisée. Pour plus d’informations, consultez la [vue d’ensemble d’Azure Standard Load Balancer](../load-balancer/load-balancer-overview.md). Tout cluster Service Fabric utilisant la référence SKU Standard pour SLB doit vérifier que chaque type de nœud a une règle autorisant le trafic sortant sur le port 443. Cette opération est nécessaire pour terminer l’installation du cluster, et tout déploiement sans règle de ce type échouera. Dans l’exemple ci-dessus d’un équilibreur de charge « interne uniquement », un équilibreur de charge externe supplémentaire doit être ajouté au modèle avec une règle autorisant le trafic sortant pour le port 443.

## <a name="next-steps"></a>Étapes suivantes
[Créer un cluster](service-fabric-cluster-creation-via-arm.md)

Après le déploiement, deux équilibreurs de charge apparaissent dans le groupe de ressources. Si vous parcourez les équilibreurs de charge, vous voyez l’adresse IP publique et les points de terminaison de gestion (ports 19000 et 19080) affectés à l’adresse IP publique. L’adresse IP interne statique et le point de terminaison d’application (port 80) affectés à l’équilibreur de charge interne apparaissent également. Les deux équilibreurs de charge utilisent le même pool back-end de groupe de machines virtuelles identiques.
