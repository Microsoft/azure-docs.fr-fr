---
title: Proxy inverse Azure Service Fabric
description: Comprenez comment installer et configurer le service de proxy inverse pour une application Azure Service Fabric.
ms.topic: conceptual
ms.date: 11/13/2018
ms.author: pepogors
ms.openlocfilehash: f8a9025a50b2815f0e6030e7baf317b261c8c462
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86256349"
---
# <a name="set-up-and-configure-reverse-proxy-in-azure-service-fabric"></a>Installer et configurer un proxy inverse dans Azure Service Fabric
Un proxy inverse est un service Azure Service Fabric facultatif qui aide des microservices s’exécutant dans un cluster Service Fabric à découvrir d’autres services ayant des points de terminaison HTTP, et à communiquer avec ces services. Pour en savoir plus, voir [Proxy inverse dans Azure Service Fabric](service-fabric-reverseproxy.md). Cet article vous montre comment installer et configurer un proxy inverse dans votre cluster. 

## <a name="enable-reverse-proxy-using-azure-portal"></a>Activer un proxy inverse à l’aide du portail Azure

Le portail Azure fournit une option permettant d’activer un proxy inverse lors de la création d’un cluster Service Fabric. Vous ne pouvez pas mettre à niveau un cluster existant pour utiliser un proxy inverse via le portail. 

Pour configurer un proxy inverse quand vous [créez un cluster à l’aide du portail Azure](./service-fabric-cluster-creation-via-portal.md), procédez comme suit :

1. À l’**Étape 2 : configuration de cluster**, sous **Configuration du type de nœud**, sélectionnez **Activer un proxy inverse**.

   ![Activer un proxy inverse sur le portail](./media/service-fabric-reverseproxy-setup/enable-rp-portal.png)
2. (Facultatif) Pour configurer un proxy inverse sécurisé, vous devez configurer un certificat TLS/SSL. À l’**Étape 3 : sécurité**, dans **Configurer les paramètres de sécurité du cluster**, sous **Type de configuration**, sélectionnez **Personnalisé**. Ensuite, sous **Certificat SSL de proxy inverse**, sélectionnez **Inclure un certificat SSL pour proxy inverse**, puis entrez les détails de votre certificat.

   ![Configurer un proxy inverse sécurisé sur le portail](./media/service-fabric-reverseproxy-setup/configure-rp-certificate-portal.png)

   Si vous choisissez de ne pas configurer le proxy inverse avec un certificat lorsque vous créez le cluster, vous pouvez le faire ultérieurement via le modèle Resource Manager pour le groupe de ressources du cluster. Pour en savoir plus, [Activer le proxy inverse via des modèles Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates).

## <a name="enable-reverse-proxy-via-azure-resource-manager-templates"></a>Activer le proxy inverse via les modèles Azure Resource Manager

Pour des clusters sur Azure, vous pouvez utiliser le modèle Azure Resource Manager pour activer le proxy inverse dans Service Fabric. Vous pouvez activer le proxy inverse lorsque vous créez le cluster, ou en mettant à jour le cluster par la suite. 

Pour un nouveau cluster, vous pouvez [créer un modèle Resource Manager personnalisé](service-fabric-cluster-creation-via-arm.md) ou utiliser un exemple de modèle. 

Pour trouver des exemples de modèles Resource Manager qui pourront vous aider à configurer un proxy inverse sécurisé pour un cluster Azure, voir la rubrique [Secure Reverse Proxy Sample Templates](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample) (exemples de modèles de proxy inverse sécurisé) sur GitHub. La section [Configure HTTPS Reverse Proxy in a secure cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample/README.md#configure-https-reverse-proxy-in-a-secure-cluster) (Configurer un proxy inverse HTTPS dans un cluster sécurisé) du fichier README contient des instructions et les modèles à utiliser pour configurer un proxy inverse sécurisé avec un certificat ainsi que pour procéder à la substitution de certificat.

Pour un cluster existant, vous pouvez exporter le modèle Resource Manager pour le groupe de ressources du cluster à l’aide du [portail Azure](../azure-resource-manager/templates/export-template-portal.md), de [PowerShell](../azure-resource-manager/management/manage-resources-powershell.md) ou d’[Azure CLI](../azure-resource-manager/management/manage-resources-cli.md).

Une fois que vous disposez d’un modèle Resource Manager, vous pouvez activer le proxy inverse en procédant comme suit :

1. Définissez un port pour le proxy inverse, dans la [section des paramètres](../azure-resource-manager/templates/template-syntax.md) du modèle.

    ```json
    "SFReverseProxyPort": {
        "type": "int",
        "defaultValue": 19081,
        "metadata": {
            "description": "Endpoint for Service Fabric Reverse proxy"
        }
    },
    ```
2. Spécifiez le port pour chaque objet nodetype dans la [ **[section de type de ressource](../azure-resource-manager/templates/template-syntax.md) Microsoft.servicefabric/clusters**](/azure/templates/microsoft.servicefabric/clusters).

    Le port est identifié par le nom de paramètre, reverseProxyEndpointPort.

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
       "nodeTypes": [
          {
           ...
           "reverseProxyEndpointPort": "[parameters('SFReverseProxyPort')]",
           ...
          },
        ...
        ],
        ...
    }
    ```
3. Pour configurer des certificats TLS/SSL sur le port pour le proxy inverse, ajoutez le certificat à la propriété ***reverseProxyCertificate*** dans la [section de type de ressource](../azure-resource-manager/templates/template-syntax.md) **Microsoft.ServiceFabric/clusters**.

    ```json
    {
        "apiVersion": "2016-09-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            "[concat('Microsoft.Storage/storageAccounts/', parameters('supportLogStorageAccountName'))]"
        ],
        "properties": {
            ...
            "reverseProxyCertificate": {
                "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                "x509StoreName": "[parameters('sfReverseProxyCertificateStoreName')]"
            },
            ...
            "clusterState": "Default",
        }
    }
    ```

### <a name="supporting-a-reverse-proxy-certificate-thats-different-from-the-cluster-certificate"></a>Prise en charge d’un certificat de proxy inverse différent du certificat de cluster
 Si le certificat du proxy inverse est différent du certificat qui sécurise le cluster, le certificat spécifié précédemment doit être installé sur la machine virtuelle et ajouté à la liste de contrôle d’accès (ACL) afin que Service Fabric puisse y accéder. Vous pouvez faire cela dans la [section de type de ressource](../azure-resource-manager/templates/template-syntax.md) [**Microsoft.Compute/virtualMachineScaleSets**](/azure/templates/microsoft.compute/virtualmachinescalesets). Pour l’installation, ajoutez ce certificat à l’osProfile. La section d’extension du modèle peut mettre à jour le certificat dans l’ACL.

  ```json
  {
    "apiVersion": "[variables('vmssApiVersion')]",
    "type": "Microsoft.Compute/virtualMachineScaleSets",
    ....
      "osProfile": {
          "adminPassword": "[parameters('adminPassword')]",
          "adminUsername": "[parameters('adminUsername')]",
          "computernamePrefix": "[parameters('vmNodeType0Name')]",
          "secrets": [
            {
              "sourceVault": {
                "id": "[parameters('sfReverseProxySourceVaultValue')]"
              },
              "vaultCertificates": [
                {
                  "certificateStore": "[parameters('sfReverseProxyCertificateStoreValue')]",
                  "certificateUrl": "[parameters('sfReverseProxyCertificateUrlValue')]"
                }
              ]
            }
          ]
        }
   ....
   "extensions": [
          {
              "name": "[concat(parameters('vmNodeType0Name'),'_ServiceFabricNode')]",
              "properties": {
                      "type": "ServiceFabricNode",
                      "autoUpgradeMinorVersion": false,
                      ...
                      "publisher": "Microsoft.Azure.ServiceFabric",
                      "settings": {
                        "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                        "nodeTypeRef": "[parameters('vmNodeType0Name')]",
                        "dataPath": "D:\\\\SvcFab",
                        "durabilityLevel": "Bronze",
                        "testExtension": true,
                        "reverseProxyCertificate": {
                          "thumbprint": "[parameters('sfReverseProxyCertificateThumbprint')]",
                          "x509StoreName": "[parameters('sfReverseProxyCertificateStoreValue')]"
                        },
                  },
                  "typeHandlerVersion": "1.0"
              }
          },
      ]
    }
  ```
> [!NOTE]
> Lorsque vous utilisez des certificats différents du certificat de cluster pour activer le certificat du proxy inverse sur un cluster existant, installez le proxy inverse et mettez à jour l’ACL sur le cluster avant d’activer le proxy inverse. Effectuez le déploiement du [modèle Azure Resource Manager](service-fabric-cluster-creation-via-arm.md) en utilisant les paramètres mentionnés précédemment avant de commencer un déploiement pour activer le proxy inverse aux étapes 1 à 3.

## <a name="enable-reverse-proxy-on-standalone-clusters"></a>Activer un proxy inverse sur des clusters autonomes

Pour des clusters autonomes, vous activez un proxy inverse dans le fichier ClusterConfig.json. Vous pouvez activer un proxy inverse lors de la création d’un cluster ou en mettant à niveau la configuration d’un cluster existant. Pour en savoir plus sur les paramètres disponibles dans les fichiers ClusterConfig.json, voir [Paramètres de cluster autonome](./service-fabric-cluster-manifest.md).

Les étapes suivantes montrent les paramètres à utiliser pour activer un proxy inverse et, éventuellement, sécuriser celui-ci avec un certificat X.509. 

1. Pour activer un proxy inverse, définissez la **reverseProxyEndpointPort** valeur pour le type de nœud sous **Propriétés** dans la configuration du cluster. Le JSON suivant montre la définition du port du point de terminaison du proxy inverse sur 19081 pour des nœuds de type « NodeType0 » :

   ```json
       "properties": {
          ... 
           "nodeTypes": [
               {
                   "name": "NodeType0",
                   ...
                   "reverseProxyEndpointPort": "19081",
                   ...
               }
           ],
          ...
       }
   ```
2. (Facultatif) Pour un proxy inverse sécurisé, configurez un certificat dans la section **Sécurité** sous **Propriétés**. 
   - Pour un environnement de développement ou de test, vous pouvez utiliser le paramètre **ReverseProxyCertificate** :

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificate": {
                          "Thumbprint": "[Thumbprint]",
                          "ThumbprintSecondary": "[Thumbprint]",
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```
   - Pour un environnement de production, le paramètre **ReverseProxyCertificateCommonNames** est recommandé :

      ```json
          "properties": {
              ...
              "security": {
                  ...
                  "CertificateInformation": {
                      ...
                      "ReverseProxyCertificateCommonNames": {
                        "CommonNames": [
                            {
                              "CertificateCommonName": "[CertificateCommonName]"
                            }
                          ],
                          "X509StoreName": "My"
                      },
                      ...
                  }
              },
              ...
          }
      ```

   Pour en savoir plus sur la configuration et la gestion des certificats pour un cluster autonome, ainsi que sur la configuration des certificats utilisés pour sécuriser un proxy inverse, voir [Sécurité basée sur un certificat X509](./service-fabric-windows-cluster-x509-security.md).

Après avoir modifié votre fichier ClusterConfig.json pour activer un proxy inverse, suivez les instructions de la section [Mettre à niveau la configuration du cluster](service-fabric-cluster-config-upgrade-windows-server.md) pour envoyer les modifications à votre cluster.


## <a name="expose-reverse-proxy-on-a-public-port-through-azure-load-balancer"></a>Exposer un proxy inverse sur un port public via Azure Load Balancer

Pour adresser le proxy inverse à partir de l’extérieur d’un cluster Azure, définissez des règles Azure Load Balancer et une sonde d’intégrité Azure pour le port du proxy inverse. Vous pouvez effectuer ces étapes à l’aide du portail Azure ou du modèle Resource Manager à tout moment après avoir créé le cluster. 

> [!WARNING]
> Lorsque vous configurez le port du proxy inverse dans l’équilibreur de charge, les systèmes se trouvant à l’extérieur du cluster peuvent atteindre tous les microservices du cluster exposant un point de terminaison HTTP. Cela signifie que des microservices destinés à être internes pourraient se révéler détectables par un utilisateur malveillant déterminé. Il peut en résulter de graves vulnérabilités qui peuvent être exploitées, notamment dans les cas de figure suivants :
>
> * Un utilisateur malveillant lance une attaque par déni de service en appelant à de nombreuses reprises un service interne qui ne dispose pas d’une surface d’attaque suffisamment renforcée.
> * Un utilisateur malveillant remet des paquets incorrects à un service interne, ce qui entraîne un comportement inattendu.
> * Un service destiné à être interne retourne des informations privées ou sensibles censées être hors de portée des services extérieurs au cluster, ce qui les expose à un utilisateur malveillant. 
>
> Veillez à bien comprendre et à limiter les conséquences potentielles sur la sécurité de votre cluster et des applications associées avant de rendre le port du proxy inverse public. 
>

Si vous souhaitez exposer un proxy inverse publiquement pour un cluster autonome, la manière de procéder dépend du système hébergeant le cluster, mais cela dépasse le cadre de cet article. Toutefois, l’avertissement précédent concernant l’exposition publique d’un proxy inverse reste d’application.

### <a name="expose-the-reverse-proxy-using-azure-portal"></a>Exposer le proxy inverse à l’aide du portail Azure 

1. Dans le portail Azure, cliquez sur le groupe de ressources de votre cluster, puis sur l’équilibreur de charge du cluster.
2. Si vous voulez ajouter une sonde d’intégrité pour le port du proxy inverse, dans le volet gauche de la fenêtre de l’équilibreur de charge, sous **PARAMÈTRES**, cliquez sur **Sondes d’intégrité**. Cliquez ensuite sur **Ajouter** en haut de la fenêtre Sondes d’intégrité, entrez les détails relatifs au port du proxy inverse, puis cliquez sur **OK**. Par défaut, le port du proxy inverse est 19081, sauf si vous l’avez modifié lors de la création du cluster.

   ![Configurer un sonde d’intégrité de proxy inverse](./media/service-fabric-reverseproxy-setup/lb-rp-probe.png)
3. Pour ajouter une règle d’équilibreur de charge afin d’exposer le port du proxy inverse, dans le volet gauche de la fenêtre de l’équilibreur de charge, sous **PARAMÈTRES**, cliquez sur **Règles d’équilibrage de charge**. Cliquez ensuite sur **Ajouter** en haut de la fenêtre des règles d’équilibrage de charge, puis entrez des détails concernant le port du proxy inverse. Veillez à définir la valeur **Port** sur le port sur lequel vous voulez que le proxy inverse soit exposé, la valeur **Port principal** sur le port que vous avez défini quand vous avez activé le proxy inverse, et la valeur **Sonde d’intégrité**  pour la sonde d’intégrité que vous avez configurée à l’étape précédente. Définissez les autres champs de façon appropriée, puis cliquez sur **OK**.

   ![Configurer une règle d’équilibrage de charge pour le proxy inverse](./media/service-fabric-reverseproxy-setup/lb-rp-rule.png)

### <a name="expose-the-reverse-proxy-via-resource-manager-templates"></a>Exposer le proxy inverse via des modèles Resource Manager

Le JSON suivant fait référence au modèle utilisé dans [Activer le proxy inverse via les modèles Azure Resource Manager](#enable-reverse-proxy-via-azure-resource-manager-templates). Reportez-vous à cette section du document pour plus d’informations sur la façon de créer un modèle Resource Manager ou d’exporter un modèle pour un cluster existant.  Les modifications sont apportées à la [section de type de ressource](../azure-resource-manager/templates/template-syntax.md) [**Microsoft.Network/loadbalancers**](/azure/templates/microsoft.network/loadbalancers).

```json
{
    "apiVersion": "[variables('lbApiVersion')]",
    "type": "Microsoft.Network/loadBalancers",
    ...
    ...
    "loadBalancingRules": [
        ...
        {
            "name": "LBSFReverseProxyRule",
            "properties": {
                "backendAddressPool": {
                    "id": "[variables('lbPoolID0')]"
                },
                "backendPort": "[parameters('SFReverseProxyPort')]",
                "enableFloatingIP": "false",
                "frontendIPConfiguration": {
                    "id": "[variables('lbIPConfig0')]"
                },
                "frontendPort": "[parameters('SFReverseProxyPort')]",
                "idleTimeoutInMinutes": "5",
                "probe": {
                    "id": "[concat(variables('lbID0'),'/probes/SFReverseProxyProbe')]"
                },
                "protocol": "tcp"
            }
        }
    ],
    "probes": [
        ...
        {
            "name": "SFReverseProxyProbe",
            "properties": {
                "intervalInSeconds": 5,
                "numberOfProbes": 2,
                "port":     "[parameters('SFReverseProxyPort')]",
                "protocol": "tcp"
            }
        }  
    ]
}
```


## <a name="customize-reverse-proxy-behavior-using-fabric-settings"></a>Personnaliser le comportement d’un proxy inverse à l’aide de paramètres de structure

Vous pouvez personnaliser le comportement d’un proxy inverse via des paramètres de structure dans le modèle Resource Manager pour des clusters hébergés dans Azure ou dans le fichier ClusterConfig.json pour des clusters autonomes. Les paramètres qui contrôlent le comportement de proxy inverse figurent dans la section [**ApplicationGateway/Http**](./service-fabric-cluster-fabric-settings.md#applicationgatewayhttp) de la section **fabricSettings** sous la section **Propriétés** du cluster. 

Par exemple, vous pouvez spécifier la valeur de **DefaultHttpRequestTimeout** pour définir le délai d’expiration des demandes adressées au proxy inverse sur 180 secondes comme dans le JSON suivant :

   ```json
   {
   "fabricSettings": [
             ...
             {
               "name": "ApplicationGateway/Http",
               "parameters": [
                 {
                   "name": "DefaultHttpRequestTimeout",
                   "value": "180"
                 }
               ]
             }
           ],
           ...
   }
   ``` 

Pour plus d’informations sur la mise à jour des paramètres de structure pour des clusters Azure, voir [Personnaliser les paramètres de cluster à l’aide de modèles Resource Manager](service-fabric-cluster-config-upgrade-azure.md). Pour des clusters autonomes, voir [Personnaliser les paramètres de cluster pour les clusters autonomes](service-fabric-cluster-config-upgrade-windows-server.md). 

Plusieurs paramètres de structure sont utilisés pour aider à établir une communication sécurisée entre un proxy inverse et des services. Pour obtenir des informations détaillées sur ces paramètres, consultez [Se connecter à un service sécurisé avec le proxy inverse](service-fabric-reverseproxy-configure-secure-communication.md).

## <a name="next-steps"></a>Étapes suivantes
* [Configurer le transfert vers un service HTTP sécurisé avec le proxy inverse](service-fabric-reverseproxy-configure-secure-communication.md)
* Pour les options de configuration du proxy inverse, voir la [section ApplicationGateway/Http dans Personnaliser les paramètres de cluster Service Fabric](service-fabric-cluster-fabric-settings.md#applicationgatewayhttp).
