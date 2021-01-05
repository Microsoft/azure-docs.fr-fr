---
title: Déplacer un équilibreur de charge Azure interne vers une autre région Azure à l’aide du portail Azure
description: Utilisez un modèle Azure Resource Manager pour déplacer un équilibreur de charge Azure interne d’une région Azure vers une autre à l’aide du portail Azure.
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 09/18/2019
ms.author: allensu
ms.openlocfilehash: 68a2cb6926cb41956711a9e3c15d21c250d27f0b
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94698469"
---
# <a name="move-azure-internal-load-balancer-to-another-region-using-the-azure-portal"></a>Déplacer un équilibreur de charge Azure interne vers une autre région à l’aide du portail Azure

Il existe différents scénarios dans lesquels vous pourriez souhaiter déplacer votre équilibreur de charge interne existant d’une région à une autre. Par exemple, vous souhaiterez peut-être créer un équilibreur de charge interne avec la même configuration à des fins de test. Vous pouvez également déplacer un équilibreur de charge interne vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Les équilibreurs de charge internes Azure ne peuvent pas être déplacés d’une région à l’autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et le réseau virtuel d’un équilibreur de charge interne.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’équilibreur de charge et le réseau virtuel vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant les modèles dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’équilibreur de charge interne se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les équilibreurs de charge internes Azure ne peuvent pas être déplacés entre les régions.  Vous devez associer le nouvel équilibreur de charge à des ressources de la région cible.

- Pour exporter une configuration d’équilibreur de charge interne et déployer un modèle afin de créer un équilibreur de charge interne dans une autre région, vous devez disposer au moins du rôle Contributeur de réseaux.

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau, les machines virtuelles et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des équilibreurs de charge internes dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’équilibreurs de charge pour ce processus.  Voir [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits)


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer l’équilibreur de charge interne pour le déplacement à l’aide d’un modèle Resource Manager, et déplacer la configuration de l’équilibreur de charge interne vers la région cible à l’aide du portail Azure.  Dans le cadre de ce processus, la configuration du réseau virtuel de l’équilibreur de charge interne doit être incluse et doit être effectuée avant de déplacer l’équilibreur de charge interne.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="export-the-virtual-network-template-and-deploy-from-the-azure-portal"></a>Exporter le modèle de réseau virtuel et le déployer à partir du portail Azure

1. Connectez-vous au [Portail Azure](https://portal.azure.com) > **Groupes de ressources**.
2. Localisez le groupe de ressources qui contient le réseau virtuel source, puis cliquez dessus.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Déployer** dans le panneau **Exporter le modèle**.
5. Cliquez sur **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier **parameters.json** dans l’éditeur en ligne.
6. Pour modifier le paramètre du nom du réseau virtuel, modifiez la propriété **value** sous **parameters** :

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "virtualNetworks_myVNET1_name": {
                "value": "<target-virtual-network-name>"
            }
        }
    }
    ```
7. Remplacez la valeur nom du réseau virtuel source dans l’éditeur par le nom de votre choix pour le réseau virtuel cible. Veillez à placer le nom entre guillemets.

8. Cliquez sur **Enregistrer** dans l’éditeur.

9. Cliquez sur **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier **template.json** dans l’éditeur en ligne.

10. Pour modifier la région cible où le réseau virtuel sera déplacé, changez la propriété **location** sous Ressources :

    ```json
    "resources": [
                {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region>",
                    "properties": {
                        "provisioningState": "Succeeded",
                        "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                        "addressSpace": {
                            "addressPrefixes": [
                                "10.0.0.0/16"
                            ]
                        },

    ```

11. Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Le code d’une région est le nom de la région sans espace, **USA Centre** = **centralus**.

12. Vous pouvez également changer d’autres paramètres dans le fichier **template.json** si vous le souhaitez ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Espace d’adressage** : l’espace d’adressage du réseau virtuel peut être modifié avant l’enregistrement en modifiant la section **resources** > **addressSpace** et en modifiant la propriété **addressPrefixes** dans le fichier **template.json** :

        ```json
                "resources": [
                    {
                    "type": "Microsoft.Network/virtualNetworks",
                    "apiVersion": "2019-06-01",
                    "name": "[parameters('virtualNetworks_myVNET1_name')]",
                    "location": "<target-region",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "resourceGuid": "6e2652be-35ac-4e68-8c70-621b9ec87dcb",
                    "addressSpace": {
                        "addressPrefixes": [
                        "10.0.0.0/16"
                        ]
                    },

        ```

    * **Sous-réseau** : le nom du sous-réseau et l’espace d’adressage du sous-réseau peuvent être modifiés ou ajoutés en modifiant la section **subnets** du fichier **template.json**. Le nom du sous-réseau peut être changé en modifiant la propriété **name**. Vous pouvez modifier l’espace d’adressage du sous-réseau en modifiant la propriété **addressPrefix** dans le fichier **template.json** :

        ```json
                "subnets": [
                    {
                    "name": "subnet-1",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.0.0/24",
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    },
                    {
                    "name": "GatewaySubnet",
                    "etag": "W/\"d9f6e6d6-2c15-4f7c-b01f-bed40f748dea\"",
                    "properties": {
                    "provisioningState": "Succeeded",
                    "addressPrefix": "10.0.1.0/29",
                    "serviceEndpoints": [],
                    "delegations": [],
                    "privateEndpointNetworkPolicies": "Enabled",
                    "privateLinkServiceNetworkPolicies": "Enabled"
                    }
                    }

                ]
        ```

         Dans le fichier **template.json**, pour modifier le préfixe d’adresse, vous devez le modifier à deux emplacements, la section ci-dessus et la section **type** indiquée ci-dessous.  Modifiez la propriété **addressPrefix** pour qu’elle corresponde à celle ci-dessus :

        ```json
         "type": "Microsoft.Network/virtualNetworks/subnets",
           "apiVersion": "2019-06-01",
           "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/GatewaySubnet')]",
              "dependsOn": [
                 "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                   ],
              "properties": {
                 "provisioningState": "Succeeded",
                 "addressPrefix": "10.0.1.0/29",
                 "serviceEndpoints": [],
                 "delegations": [],
                 "privateEndpointNetworkPolicies": "Enabled",
                 "privateLinkServiceNetworkPolicies": "Enabled"
                  }
                 },
                  {
                  "type": "Microsoft.Network/virtualNetworks/subnets",
                  "apiVersion": "2019-06-01",
                  "name": "[concat(parameters('virtualNetworks_myVNET1_name'), '/subnet-1')]",
                     "dependsOn": [
                        "[resourceId('Microsoft.Network/virtualNetworks', parameters('virtualNetworks_myVNET1_name'))]"
                          ],
                     "properties": {
                        "provisioningState": "Succeeded",
                        "addressPrefix": "10.0.0.0/24",
                        "delegations": [],
                        "privateEndpointNetworkPolicies": "Enabled",
                        "privateLinkServiceNetworkPolicies": "Enabled"
                         }
                  }
         ]
        ```

13. Cliquez sur **Enregistrer** dans l’éditeur en ligne.

14. Cliquez **DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel le réseau virtuel cible sera déployé.

15. Cliquez **DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel le réseau virtuel cible sera déployé.  Vous pouvez cliquer sur **Créer** pour créer un nouveau groupe de ressources pour le réseau virtuel cible.  Vérifiez que le nom n’est pas identique à celui du groupe de ressources source du réseau virtuel existant.

16. Vérifiez que **DE BASE** > **Emplacement** est défini sur l’emplacement cible où vous souhaitez déployer le réseau virtuel.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré dans l’éditeur de paramètres ci-dessus.

18. Cochez la case sous **CONDITIONS GÉNÉRALES**.

19. Cliquez sur le bouton **Acheter** pour déployer le réseau virtuel cible.

### <a name="export-the-internal-load-balancer-template-and-deploy-from-azure-powershell"></a>Exporter le modèle d’équilibreur de charge interne et le déployer à partir d’Azure PowerShell

1. Connectez-vous au [Portail Azure](https://portal.azure.com) > **Groupes de ressources**.
2. Recherchez le groupe de ressources qui contient l’équilibreur de charge externe source, puis cliquez dessus.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Déployer** dans le panneau **Exporter le modèle**.
5. Cliquez sur **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier **parameters.json** dans l’éditeur en ligne.

6. Pour modifier le paramètre du nom de l’équilibreur de charge interne, remplacez la propriété **defaultValue** du nom de l’équilibreur de charge interne source par le nom de votre équilibreur de charge interne cible, en vous assurant que le nom est entre guillemets :

    ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-internal-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_internalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
    ```

6. Pour modifier la valeur du réseau virtuel cible qui a été déplacé ci-dessus, vous devez d’abord obtenir l’ID de ressource, puis le copier et le coller dans le fichier **parameters.json**. Pour obtenir l’ID :

    1. Connectez-vous au [portail Azure](https://portal.azure.com) > **Groupes de ressources** dans un autre onglet ou une autre fenêtre de navigateur.
    2. Recherchez le groupe de ressources cible qui contient le réseau virtuel déplacé lors des étapes ci-dessus, et cliquez dessus.
    3. Sélectionnez > **Paramètres** > **Propriétés**.
    4. Dans le panneau de droite, mettez en surbrillance l’**ID de ressource** et copiez-le dans le Presse-papiers.  Vous pouvez également cliquer sur le bouton **Copier dans le Presse-papiers** à droite du chemin de l’**ID de ressource**.
    5. Collez l’ID de ressource dans la propriété **defaultValue** dans l’éditeur **Modifier les paramètres** ouvert dans l’autre fenêtre ou onglet de navigateur :

        ```json
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
            "loadBalancers_myLoadBalancer_name": {
            "defaultValue": "<target-internal-lb-name>",
            "type": "String"
             },
            "virtualNetworks_myVNET2_internalid": {
             "defaultValue": "<target-vnet-resource-ID>",
             "type": "String"
             }
        ```
    6. Cliquez sur **Enregistrer** dans l’éditeur en ligne.

7. Cliquez sur **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier **template.json** dans l’éditeur en ligne.
8. Pour modifier la région cible où la configuration de l’équilibreur de charge interne sera déplacée, modifiez la propriété **location** sous **resources** dans le fichier **template.json** :

    ```json
        "resources": [
            {
                "type": "Microsoft.Network/loadBalancers",
                "apiVersion": "2019-06-01",
                "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
                "location": "<target-internal-lb-region>",
                "sku": {
                    "name": "Standard",
                    "tier": "Regional"
                },
    ```

9.  Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Le code d’une région est le nom de la région sans espace, **USA Centre** = **centralus**.

10. Vous pouvez également changer d’autres paramètres dans le modèle ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Référence SKU** : vous pouvez permuter la référence SKU de l’équilibreur de charge interne dans la configuration entre les valeurs basic et standard en modifiant la propriété **sku** > **name** dans le fichier **template.json** :

        ```json
        "resources": [
        {
            "type": "Microsoft.Network/loadBalancers",
            "apiVersion": "2019-06-01",
            "name": "[parameters('loadBalancers_myLoadBalancer_name')]",
            "location": "<target-internal-lb-region>",
            "sku": {
                "name": "Standard",
                "tier": "Regional"
            },
        ```
      Pour plus d’informations sur les différences entre les équilibreurs de charge des références SKU basic et standard, consultez [Présentation d’Azure Standard Load Balancer](./load-balancer-overview.md).

    * **Règles d’équilibrage de charge** : vous pouvez ajouter ou supprimer des règles d’équilibrage de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **loadBalancingRules** du fichier **template.json** :

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
       Pour plus d’informations sur les règles d’équilibrage de charge, consultez [Qu’est-ce qu’Azure Load Balancer ?](./load-balancer-overview.md).

    * **Sondes** : vous pouvez ajouter ou supprimer une sonde pour l’équilibreur de charge dans la configuration en ajoutant ou en supprimant des entrées dans la section **probes** du fichier **template.json** :

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
       Pour plus d’informations sur les sondes d’intégrité Azure Load Balancer, consultez [Sondes d’intégrité Load Balancer](./load-balancer-custom-probe-overview.md).

    * **Règle NAT de trafic entrant**  : vous pouvez ajouter ou supprimer des règles NAT de trafic entrant pour l’équilibreur de charge en ajoutant ou en supprimant des entrées dans la section **inboundNatRules** du fichier **template.json** :

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
        Pour terminer l’ajout ou la suppression d’une règle NAT de trafic entrant, il faut que la règle soit présente ou supprimée en tant que propriété **type** à la fin du fichier **template.json** :

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
        Pour plus d’informations sur les règles NAT de trafic entrant, consultez [Qu’est-ce qu’Azure Load Balancer ?](./load-balancer-overview.md)

12. Cliquez sur **Enregistrer** dans l’éditeur en ligne.

13. Cliquez sur **DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel l’équilibreur de charge interne cible sera déployé.

15. Cliquez sur **DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel l’équilibreur de charge cible sera déployé.  Vous pouvez cliquer sur **Créer** afin de créer un nouveau groupe de ressources pour l’équilibreur de charge interne cible, ou choisir le groupe de ressources existant créé ci-dessus pour le réseau virtuel.  Vérifiez que le nom n’est pas identique à celui du groupe de ressources source de l’équilibreur de charge interne source existant.

16. Vérifiez que **DE BASE** > **Emplacement** est défini sur l’emplacement cible où vous souhaitez déployer l’équilibreur de charge interne.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré dans l’éditeur de paramètres ci-dessus.  Vérifiez que les ID de ressources sont renseignés pour tous les réseaux virtuels de la configuration.

18. Cochez la case sous **CONDITIONS GÉNÉRALES**.

19. Cliquez sur le bouton **Acheter** pour déployer le réseau virtuel cible.

## <a name="discard"></a>Abandonner

Si vous souhaitez ignorer le réseau virtuel et l’équilibreur de charge interne cibles, supprimez le groupe de ressources qui contient le réseau virtuel et l’équilibreur de charge interne cibles.  Pour ce faire, sélectionnez le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de la page de vue d’ensemble.

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du réseau virtuel et de l’équilibreur de charge interne, supprimez le réseau virtuel source et le groupe de ressources ou l’équilibreur de charge interne source. Pour ce faire, sélectionnez le réseau virtuel et l’équilibreur de charge interne ou le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de chaque page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un équilibreur de charge interne Azure d’une région à une autre et vous avez nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Déplacer des machines virtuelles Azure vers une autre région](../site-recovery/azure-to-azure-tutorial-migrate.md)