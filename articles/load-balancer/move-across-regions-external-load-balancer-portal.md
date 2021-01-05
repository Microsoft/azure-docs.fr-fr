---
title: Déplacer un équilibreur de charge externe Azure vers une autre région Azure à l’aide du portail Azure
description: Utilisez un modèle Azure Resource Manager pour déplacer un équilibreur de charge externe Azure d’une région Azure vers une autre à l’aide du portail Azure.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/17/2019
ms.author: allensu
ms.openlocfilehash: f83ff3d1d03354daef3466c1f48eaa505e378634
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94693747"
---
# <a name="move-an-external-load-balancer-to-another-region-by-using-the-azure-portal"></a>Déplacer un équilibreur de charge externe vers une autre région à l’aide du portail Azure

Il existe différents scénarios dans lesquels vous pouvez avoir à déplacer un équilibreur de charge externe d’une région à une autre. Par exemple, vous souhaiterez peut-être créer un autre équilibreur de charge externe avec la même configuration à des fins de test. Vous pouvez également déplacer un équilibreur de charge externe vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Vous ne pouvez pas à proprement parler déplacer un équilibreur de charge externe Azure d’une région vers une autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et l’adresse IP publique d’un équilibreur de charge externe. Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’équilibreur de charge et l’adresse IP publique vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant les modèles dans la nouvelle région. Pour plus d’informations sur Resource Manager et les modèles, consultez [Exporter des groupes de ressources vers des modèles](../azure-resource-manager/management/manage-resource-groups-powershell.md#export-resource-groups-to-templates).


## <a name="prerequisites"></a>Conditions préalables requises

- Vérifiez que l’équilibreur de charge externe se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les équilibreurs de charge externes Azure ne peuvent pas être déplacés entre les régions. Vous devez associer le nouvel équilibreur de charge à des ressources de la région cible.

- Pour exporter une configuration d’équilibreur de charge externe et déployer un modèle afin de créer un équilibreur de charge externe dans une autre région, vous devez disposer au moins du rôle Contributeur de réseaux.

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau, les adresses IP publiques et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des équilibreurs de charge externes dans la région cible. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’équilibreurs de charge. Consultez [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

## <a name="prepare-and-move"></a>Préparer et déplacer
Les procédures suivantes montrent comment préparer l’équilibreur de charge externe pour le déplacement à l’aide d’un modèle Resource Manager, et déplacer la configuration de l’équilibreur de charge externe vers la région cible à l’aide du portail Azure. Vous devez commencer par exporter la configuration d’adresse IP publique de l’équilibreur de charge externe.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-public-ip-template-and-deploy-the-public-ip-from-the-portal"></a>Exporter le modèle d’adresse IP publique et déployer l’adresse IP publique à partir du portail

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources**.
2. Recherchez le groupe de ressources qui contient l’adresse IP publique source et sélectionnez-le.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Sélectionnez **Déployer** sous **Exporter le modèle**.
5. Sélectionnez **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier parameters.json dans l’éditeur en ligne.
8. Pour modifier le paramètre du nom d’adresse IP publique, remplacez la propriété **value** sous **parameters** par le nom de votre adresse IP publique cible. Veillez à placer le nom entre guillemets.

    ```json
            {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "publicIPAddresses_myVM1pubIP_name": {
            "value": "<target-publicip-name>"
              }
             }
            }

    ```

    Sélectionnez **Enregistrer** dans l’éditeur.

9.  Sélectionnez **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier template.json dans l’éditeur en ligne.

10. Pour modifier la région cible vers laquelle l’adresse IP publique sera déplacée, changez la propriété **location** sous **resources** :

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []
               }
               }
             ]
    ```
  
    Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région est le nom de la région sans espace. Par exemple, le code pour Central US (USA Centre) est **centralus**.
    
12. Vous pouvez également changer d’autres paramètres dans le modèle, en fonction de vos besoins :

    * **Référence (SKU)**  : vous pouvez permuter la référence SKU de l’adresse IP publique dans la configuration entre les valeurs basic et standard en modifiant la propriété **name** sous **sku** dans le fichier template.json :

        ```json
          "resources": [
         {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
        ```

        Pour plus d’informations sur les différences entre les adresses IP publiques des références SKU basic et standard, consultez [Créer, modifier ou supprimer une adresse IP publique](../virtual-network/virtual-network-public-ip-address.md).

    * **Méthode d’allocation des adresses IP publiques** et **Délai d’inactivité**. Vous pouvez modifier la méthode d’allocation des adresses IP publiques en changeant la valeur de la propriété **publicIPAllocationMethod** de **Dynamic** à **Static** ou de **Static** à **Dynamic**. Vous pouvez modifier le délai d’inactivité en indiquant la valeur souhaitée dans la propriété **idleTimeoutInMinutes**. La valeur par défaut est **4**.

        ```json
          "resources": [
         {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2019-06-01",
            "name": "[parameters('publicIPAddresses_myPubIP_name')]",
            "location": "<target-region>",
            "sku": {
                "name": "Basic",
                "tier": "Regional"
            },
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "7549a8f1-80c2-481a-a073-018f5b0b69be",
                "ipAddress": "52.177.6.204",
                "publicIPAddressVersion": "IPv4",
                "publicIPAllocationMethod": "Dynamic",
                "idleTimeoutInMinutes": 4,
                "ipTags": []

        ```

        Pour plus d’informations sur les méthodes d’allocation et les valeurs de délai d’inactivité, consultez [Créer, modifier ou supprimer une adresse IP publique](../virtual-network/virtual-network-public-ip-address.md).

 
13. Sélectionnez **Enregistrer** dans l’éditeur en ligne.

14. Sélectionnez **OPTIONS DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel l’adresse IP publique cible sera déployée.

15. Sélectionnez **OPTIONS DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel l’adresse IP publique cible sera déployée. Vous pouvez sélectionner **Créer** afin de créer un groupe de ressources pour l’adresse IP publique cible. Vérifiez que le nom n’est pas identique à celui du groupe de ressources source de l’adresse IP publique cible existante.

16. Vérifiez que **OPTIONS DE BASE** > **Emplacement** est défini sur l’emplacement cible où vous souhaitez déployer l’adresse IP publique.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré précédemment dans l’éditeur de paramètres.

18. Cochez la case **CONDITIONS GÉNÉRALES**.

19. Sélectionnez **Acheter** pour déployer l’adresse IP publique cible.

20. Si vous avez une autre adresse IP publique utilisée pour la traduction d’adresses réseau pour le trafic sortant pour l’équilibreur de charge en cours de déplacement, répétez les étapes ci-dessus pour exporter et déployer la deuxième adresse IP publique sortante vers la région cible.

### <a name="export-the-external-load-balancer-template-and-deploy-the-load-balancer-from-the-azure-portal"></a>Exporter le modèle d’équilibreur de charge externe et déployer l’équilibreur de charge à partir du portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources**.
2. Recherchez le groupe de ressources qui contient l’équilibreur de charge externe source, puis sélectionnez-le.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Sélectionnez **Déployer** sous **Exporter le modèle**.
5. Sélectionnez **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier parameters.json dans l’éditeur en ligne.

5. Pour modifier le paramètre du nom de l’équilibreur de charge externe, remplacez la propriété **value** du nom de l’équilibreur de charge externe source par le nom de votre équilibreur de charge externe cible. Veillez à placer le nom entre guillemets.

    ```json
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {
          "loadBalancers_myLoadbalancer_ext_name": {
          "value": "<target-external-lb-name>"
    },
          "publicIPAddresses_myPubIP_in_externalid": {
          "value": "<target-publicIP-resource-ID>"
    },

    ```

6.  Pour modifier la valeur de l’adresse IP publique cible que vous avez déplacée au cours des étapes précédentes, vous devez d’abord obtenir l’ID de ressource, puis le coller dans le fichier parameters.json. Pour obtenir l’ID :

    1. Dans un autre onglet ou une autre fenêtre de navigateur, connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Groupes de ressources** .
    2. Recherchez le groupe de ressources cible contenant l’adresse IP publique déplacée au cours des étapes précédentes. Sélectionnez-le.
    3. Sélectionnez **Paramètres** > **Propriétés**.
    4. Dans le panneau de droite, mettez en surbrillance l’**ID de ressource** et copiez-le dans le Presse-papiers. Vous pouvez également sélectionner **Copier dans le Presse-papiers** à droite du chemin de l’**ID de ressource**.
    5. Collez l’ID de ressource dans la propriété **value** dans l’éditeur **Modifier les paramètres** ouvert dans l’autre fenêtre ou onglet de navigateur :

        ```json
           ```json
           "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
           "contentVersion": "1.0.0.0",
           "parameters": {
              "loadBalancers_myLoadbalancer_ext_name": {
              "value": "<target-external-lb-name>"
        },
              "publicIPAddresses_myPubIP_in_externalid": {
              "value": "<target-publicIP-resource-ID>"
        },

        ```
    6. Sélectionnez **Enregistrer** dans l’éditeur en ligne.


7.  Si vous avez configuré la traduction d’adresses réseau pour le trafic sortant et des règles de trafic sortant pour l’équilibreur de charge, une troisième entrée sera présente dans ce fichier pour l’ID externe de l’adresse IP publique sortante. Répétez les étapes précédentes dans la **région cible** pour obtenir l’ID de l’adresse IP publique sortante. Collez cet ID dans le fichier parameters.json :

    ```json
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
                "loadBalancers_myLoadbalancer_ext_name": {
                "value": "<target-external-lb-name>",

            },
                "publicIPAddresses_myPubIP_in_externalid": {
                "value": "<target-publicIP-resource-ID>",

            },
                "publicIPAddresses_myPubIP_out_externalid": {
                "defaultValue": "<target-publicIP-outbound-resource-ID>",

            }
        },
    ```

8.  Sélectionnez **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier template.json dans l’éditeur en ligne.
9.  Pour modifier la région cible dans laquelle la configuration de l’équilibreur de charge externe sera déplacée, modifiez la propriété **location** sous **resources** dans le fichier template.json :

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-external-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

10. Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région est le nom de la région sans espace. Par exemple, le code pour Central US (USA Centre) est **centralus**.

11. Vous pouvez également changer d’autres paramètres dans le modèle, en fonction de vos besoins :

    * **Référence (SKU)**  : vous pouvez permuter la référence SKU de l’équilibreur de charge externe dans la configuration entre les valeurs basic et standard en modifiant la propriété **name** sous **sku** dans le fichier template.json :

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-external-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Pour plus d’informations sur les différences entre les équilibreurs de charge des références SKU basic et standard, consultez [Présentation d’Azure Standard Load Balancer](./load-balancer-overview.md).

    * **Règles d’équilibrage de charge** : vous pouvez ajouter ou supprimer des règles d’équilibrage de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **loadBalancingRules** du fichier template.json :

        ```json
        "loadBalancingRules": [
                    {
                        "name": "myInboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 80,
                            "backendPort": 80,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false,
                            "loadDistribution": "Default",
                            "disableOutboundSnat": true,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolInbound')]"
                            },
                            "probe": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/probes/myHTTPProbe')]"
                            }
                        }
                    }
                ]
        ```
       Pour plus d’informations sur les règles d’équilibrage de charge, consultez [Qu’est-ce qu’Azure Load Balancer ?](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview).

    * **Sondes** : vous pouvez ajouter ou supprimer une sonde pour l’équilibreur de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **probes** du fichier template.json :

        ```json
        "probes": [
                    {
                        "name": "myHTTPProbe",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "Http",
                            "port": 80,
                            "requestPath": "/",
                            "intervalInSeconds": 15,
                            "numberOfProbes": 2
                        }
                    }
                ],
        ```
       Pour plus d’informations, consultez [Sondes d’intégrité Load Balancer](https://docs.microsoft.com/azure/load-balancer/load-balancer-custom-probe-overview).

    * **Règles NAT de trafic entrant** : vous pouvez ajouter ou supprimer des règles NAT de trafic entrant pour l’équilibreur de charge en ajoutant ou en supprimant des entrées dans la section **inboundNatRules** du fichier template.json :

        ```json
        "inboundNatRules": [
                    {
                        "name": "myInboundNATRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "frontendIPConfiguration": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                            },
                            "frontendPort": 4422,
                            "backendPort": 3389,
                            "enableFloatingIP": false,
                            "idleTimeoutInMinutes": 4,
                            "protocol": "Tcp",
                            "enableTcpReset": false
                        }
                    }
                ]
        ```
        Pour terminer l’ajout ou la suppression d’une règle NAT de trafic entrant, il faut que la règle soit présente ou supprimée en tant que propriété **type** à la fin du fichier template.json :

        ```json
        {
            "type": "Microsoft.Network/loadBalancers/inboundNatRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('loadBalancers_myLoadBalancer_name'), '/myInboundNATRule')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "frontendIPConfiguration": {
                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPinbound')]"
                },
                "frontendPort": 4422,
                "backendPort": 3389,
                "enableFloatingIP": false,
                "idleTimeoutInMinutes": 4,
                "protocol": "Tcp",
                "enableTcpReset": false
            }
        }
        ```
        Pour plus d’informations sur les règles NAT de trafic entrant, consultez [Qu’est-ce qu’Azure Load Balancer ?](https://docs.microsoft.com/azure/load-balancer/load-balancer-overview).

    * **Règles de trafic sortant** : vous pouvez ajouter ou supprimer des règles de trafic sortant dans la configuration en modifiant la propriété **outboundRules** dans le fichier template.json :

        ```json
        "outboundRules": [
                    {
                        "name": "myOutboundRule",
                        "etag": "W/\"39e5e9cd-2d6d-491f-83cf-b37a259d86b6\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "allocatedOutboundPorts": 10000,
                            "protocol": "All",
                            "enableTcpReset": false,
                            "idleTimeoutInMinutes": 15,
                            "backendAddressPool": {
                                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/backendAddressPools/myBEPoolOutbound')]"
                            },
                            "frontendIPConfigurations": [
                                {
                                    "id": "[concat(resourceId('Microsoft.Network/loadBalancers', parameters('loadBalancers_myLoadBalancer_name')), '/frontendIPConfigurations/myfrontendIPoutbound')]"
                                }
                            ]
                        }
                    }
                ]
        ```

         Pour plus d’informations, consultez [Règles de trafic sortant dans Load Balancer](./load-balancer-outbound-connections.md#outboundrules).

12. Sélectionnez **Enregistrer** dans l’éditeur en ligne.

13. Sélectionnez **OPTIONS DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel l’équilibreur de charge externe cible sera déployé.

15. Sélectionnez **OPTIONS DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel l’équilibreur de charge cible sera déployé. Vous pouvez sélectionner **Créer** afin de créer un groupe de ressources pour l’équilibreur de charge externe cible. Vous pouvez également choisir le groupe de ressources existant que vous avez créé précédemment pour l’adresse IP publique. Vérifiez que le nom n’est pas identique à celui du groupe de ressources source de l’équilibreur de charge externe source existant.

16. Vérifiez que **OPTIONS DE BASE** > **Emplacement** est défini sur l’emplacement cible où vous souhaitez déployer l’équilibreur de charge externe.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré précédemment dans l’éditeur de paramètres. Vérifiez que les ID de ressources sont renseignés pour toutes les adresses IP publiques de la configuration.

18. Cochez la case **CONDITIONS GÉNÉRALES**.

19. Sélectionnez **Acheter** pour déployer l’adresse IP publique cible.

## <a name="discard"></a>Abandonner

Si vous souhaitez ignorer l’adresse IP publique cible et l’équilibreur de charge externe, supprimez le groupe de ressources qui les contient. Pour ce faire, sélectionnez le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de la page de vue d’ensemble.

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement de l’adresse IP publique et de l’équilibreur de charge externe, supprimez l’adresse IP publique source et le groupe de ressources ou l’équilibreur de charge externe source. Pour ce faire, sélectionnez le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de chaque page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un équilibreur de charge externe Azure d’une région à une autre et vous avez nettoyé les ressources sources. Pour plus d’informations sur le déplacement de ressources entre régions et la reprise d’activité dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)