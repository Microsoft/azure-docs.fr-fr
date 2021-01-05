---
title: Déplacer un réseau virtuel Azure vers une autre région Azure en utilisant le portail Azure
description: Déplacez un réseau virtuel Azure d’une région Azure vers une autre en utilisant un modèle Resource Manager et le portail Azure.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/26/2019
ms.author: allensu
ms.openlocfilehash: ed3da649ba65484a79b42ba5bb45431839e123d4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84711440"
---
# <a name="move-an-azure-virtual-network-to-another-region-by-using-the-azure-portal"></a>Déplacer un réseau virtuel Azure vers une autre région en utilisant le portail Azure

Il existe différents scénarios pour déplacer un réseau virtuel Azure existant d’une région à une autre. Par exemple, vous voulez créer un réseau virtuel avec la même configuration pour des tests et la même disponibilité que votre réseau virtuel existant. Vous pourriez aussi déplacer un réseau virtuel de production vers une autre région dans le cadre de la planification de la reprise d’activité.

Vous pouvez utiliser un modèle Azure Resource Manager pour terminer la migration du réseau virtuel vers une autre région. Pour ce faire, exportez le réseau virtuel vers un modèle, modifiez les paramètres pour qu’ils correspondent à la région de destination, puis déployez le modèle dans la nouvelle région. Pour plus d’informations sur les modèles Resource Manager, consultez [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que votre réseau virtuel Azure se trouve dans la région Azure à partir de laquelle vous voulez effectuer le déplacement.

- Pour exporter un réseau virtuel et déployer un modèle pour créer un réseau virtuel dans une autre région, vous avez besoin du rôle Contributeur de réseaux ou d’un rôle supérieur.

- Les appairages de réseaux virtuels ne sont pas recréés et échouent s’ils sont toujours présents dans le modèle. Avant d’exporter le modèle, vous devez supprimer les pairs du réseau virtuel. Vous pouvez ensuite les rétablir après le déplacement du réseau virtuel.

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les adresses IP publiques.

- Vérifiez que votre abonnement Azure vous permet de créer des réseaux virtuels dans la région cible. Pour activer le quota nécessaire, contactez le support technique.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout de réseaux virtuels pour ce processus. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-for-the-move"></a>Préparer le déplacement
Dans cette section, vous préparez le réseau virtuel pour le déplacement en utilisant un modèle Resource Manager. Vous déplacez ensuite le réseau virtuel vers la région cible en utilisant le portail Azure.

Pour exporter le réseau virtuel et déployer le réseau virtuel cible en utilisant le portail Azure, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources**.
1. Localisez le groupe de ressources qui contient le réseau virtuel source, puis sélectionnez-le.
1. Sélectionnez **Paramètres** > **Exporter le modèle**.
1. Dans le volet **Exporter le modèle**, sélectionnez **Déployer**.
1. Pour ouvrir le fichier *parameters.json* dans votre éditeur en ligne, sélectionnez **Modèle** > **Modifier les paramètres**.
1. Pour modifier le paramètre du nom du réseau virtuel, modifiez la propriété **value** sous **parameters** :

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

1. Dans l’éditeur, remplacez la valeur du nom du réseau virtuel source par le nom de votre choix pour le réseau virtuel cible. Veillez à placer le nom entre guillemets.

1. Sélectionnez **Enregistrer** dans l’éditeur.

1. Pour ouvrir le fichier *template.json* dans l’éditeur en ligne, sélectionnez **Modèle** > **Modifier le modèle**.

1. Dans l’éditeur en ligne, pour modifier la région cible où le réseau virtuel sera déplacé, modifiez la propriété **location** sous **resources** :

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

1. Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/). Le code d’une région est le nom de la région, sans espace (par exemple **USA Centre (Central US)**  = **centralus**).

1. (Facultatif) Vous pouvez aussi changer d’autres paramètres dans le modèle, en fonction de vos besoins :

    * **Espace d’adressage** : Avant d’enregistrer le fichier, vous pouvez modifier l’espace d’adressage du réseau virtuel en modifiant la section **resources** > **addressSpace** et en changeant la propriété **addressPrefixes** :

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

    * **Sous-réseau** : Vous pouvez changer ou ajouter le nom du sous-réseau et l’espace d’adressage du sous-réseau en modifiant la section **subnets** du modèle. Vous pouvez changer le nom du sous-réseau en modifiant la propriété **name**. Vous pouvez changer l’espace d’adressage du sous-réseau en modifiant la propriété **addressPrefix** :

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

        Pour changer le préfixe d’adresse dans le fichier *template.json*, modifiez-le à deux endroits : dans le code de la section précédente et dans la section **type** du code suivant. Modifiez la propriété **addressPrefix** dans le code suivant pour qu’elle corresponde à la propriété **addressPrefix** dans le code de la section précédente.

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

1. Dans l’éditeur en ligne, sélectionnez **Enregistrer**.

1. Pour choisir l’abonnement où le réseau virtuel cible sera déployé, sélectionnez **Général** > **Abonnement**.

1. Pour choisir le groupe de ressources où le réseau virtuel cible sera déployé, sélectionnez **Général** > **Groupe de ressources**. 

    Si vous devez créer un groupe de ressources pour le réseau virtuel cible, sélectionnez **Créer**. Vérifiez que le nom n’est pas identique à celui du groupe de ressources source dans le réseau virtuel existant.

1. Vérifiez que **Général** > **Emplacement** est défini sur l’emplacement cible où vous voulez déployer le réseau virtuel.

1. Sous **Paramètres**, vérifiez que le nom correspond à celui que vous avez entré précédemment dans l’éditeur de paramètres.

1. Cochez la case **Conditions générales**.

1. Pour déployer le réseau virtuel cible, sélectionnez **Acheter**.

## <a name="delete-the-target-virtual-network"></a>Supprimer le réseau virtuel cible

Pour supprimer le réseau virtuel cible, supprimez le groupe de ressources qui le contient. Pour ce faire :
1. Dans le tableau de bord du portail Azure, sélectionnez le groupe de ressources.
1. En haut du volet **Vue d’ensemble**, sélectionnez **Supprimer**.

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et effectuer le déplacement du réseau virtuel, vous supprimez le réseau virtuel ou le groupe de ressources source. Pour ce faire :
1. Dans le tableau de bord du portail Azure, sélectionnez le réseau virtuel ou le groupe de ressources.
1. En haut de chaque volet, sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un réseau virtuel Azure d’une région à une autre en utilisant le portail Azure, puis vous avez nettoyé les ressources sources non nécessaires. Pour plus d’informations sur le déplacement de ressources entre régions et la reprise d’activité dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
