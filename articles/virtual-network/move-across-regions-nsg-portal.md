---
title: Déplacer un groupe de sécurité réseau Azure vers une autre région Azure à l’aide du Portail Azure
description: Utilisez un modèle Azure Resource Manager pour déplacer un groupe de sécurité réseau Azure d’une région Azure vers une autre à l’aide du Portail Azure.
author: asudbring
ms.service: virtual-network
ms.topic: how-to
ms.date: 08/31/2019
ms.author: allensu
ms.openlocfilehash: a22dc6dc0c4fc199d3f262b18aeeae5090a06dce
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84689314"
---
# <a name="move-azure-network-security-group-nsg-to-another-region-using-the-azure-portal"></a>Déplacer un groupe de sécurité réseau Azure vers une autre région à l’aide du Portail Azure

Il existe différents scénarios dans lesquels vous pouvez être amené à déplacer vos groupe de sécurité réseau existants d’une région à une autre. Par exemple, vous pouvez avoir besoin de créer un groupe de sécurité réseau avec la même configuration et les mêmes règles de sécurité à des fins de test. Vous pouvez également déplacer un groupe de sécurité réseau vers une autre région dans le cadre de la planification de la reprise d’activité après sinistre.

Les groupes de sécurité Azure ne peuvent pas être déplacés d’une région vers une autre. Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante et les règles de sécurité d’un groupe de sécurité réseau.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant le groupe de sécurité réseau vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant le modèle dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que le groupe de sécurité réseau Azure se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Les groupes de sécurité réseau Azure ne peuvent pas être déplacés entre les régions.  Vous devez associer le nouveau groupe de sécurité réseau à des ressources dans la région cible.

- Pour exporter une configuration de groupe de sécurité réseau et déployer un modèle afin de créer un groupe de sécurité réseau dans une autre région, vous devez disposer au minimum du rôle Contributeur de réseaux.

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les adresses IP publiques et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des groupes de sécurité réseau dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout de groupes de sécurité réseau pour ce processus.  Consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer le groupe de sécurité réseau pour le déplacement de la configuration et de la règle de sécurité à l’aide d’un modèle Resource Manager et comment déplacer la configuration et les règles de sécurité du groupe de sécurité réseau vers la région cible à l’aide du portail.


### <a name="export-the-template-and-deploy-from-the-portal"></a>Exporter le modèle et le déployer à partir du portail

1. Connectez-vous au [Portail Azure](https://portal.azure.com) > **Groupes de ressources**.
2. Recherchez le groupe de ressources contenant le groupe de sécurité réseau source et cliquez dessus.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Déployer** dans le panneau **Exporter le modèle**.
5. Cliquez sur **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier **parameters.json** dans l’éditeur en ligne.
6. Pour modifier le paramètre du nom du groupe de sécurité réseau, modifiez la propriété **value** sous **parameters** :

    ```json
            {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
            "contentVersion": "1.0.0.0",
            "parameters": {
            "networkSecurityGroups_myVM1_nsg_name": {
               "value": "<target-nsg-name>"
                }
               }
            }
    ```

7. Remplacez la valeur du groupe de sécurité réseau source dans l’éditeur par le nom de votre choix pour le groupe de sécurité réseau cible. Veillez à placer le nom entre guillemets.

8.  Cliquez sur **Enregistrer** dans l’éditeur.

9.  Cliquez sur **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier **template.json** dans l’éditeur en ligne.

10. Pour modifier la région cible où la configuration et les règles de sécurité du groupe de sécurité réseau seront déplacées, modifiez la propriété **location** sous **resources** dans l’éditeur en ligne :

    ```json
            "resources": [
            {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-06-01",
            "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
            "location": "<target-region>",
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78",
             }
            }
           ]

    ```

11. Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Le code d’une région est le nom de la région sans espace, **USA Centre** = **centralus**.

12. Vous pouvez également changer d’autres paramètres dans le modèle ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Règles de sécurité** : vous pouvez modifier les règles déployées dans le groupe de sécurité réseau cible en ajoutant ou en supprimant des règles dans la section **securityRules** du fichier **template.json** :

        ```json
           "resources": [
            {
            "type": "Microsoft.Network/networkSecurityGroups",
            "apiVersion": "2019-06-01",
            "name": "[parameters('networkSecurityGroups_myVM1_nsg_name')]",
            "location": "<target-region>",
            "properties": {
                "provisioningState": "Succeeded",
                "resourceGuid": "2c846acf-58c8-416d-be97-ccd00a4ccd78",
                "securityRules": [
                    {
                        "name": "RDP",
                        "etag": "W/\"c630c458-6b52-4202-8fd7-172b7ab49cf5\"",
                        "properties": {
                            "provisioningState": "Succeeded",
                            "protocol": "TCP",
                            "sourcePortRange": "*",
                            "destinationPortRange": "3389",
                            "sourceAddressPrefix": "*",
                            "destinationAddressPrefix": "*",
                            "access": "Allow",
                            "priority": 300,
                            "direction": "Inbound",
                            "sourcePortRanges": [],
                            "destinationPortRanges": [],
                            "sourceAddressPrefixes": [],
                            "destinationAddressPrefixes": []
                             }
                    },
                ]
            }
        ```

      Pour terminer l’ajout ou la suppression des règles dans le groupe de sécurité réseau cible, vous devez également modifier les types de règles personnalisées à la fin du fichier **template.json** selon le format de l’exemple ci-dessous :

      ```json
           {
            "type": "Microsoft.Network/networkSecurityGroups/securityRules",
            "apiVersion": "2019-06-01",
            "name": "[concat(parameters('networkSecurityGroups_myVM1_nsg_name'), '/Port_80')]",
            "dependsOn": [
                "[resourceId('Microsoft.Network/networkSecurityGroups', parameters('networkSecurityGroups_myVM1_nsg_name'))]"
            ],
            "properties": {
                "provisioningState": "Succeeded",
                "protocol": "*",
                "sourcePortRange": "*",
                "destinationPortRange": "80",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 310,
                "direction": "Inbound",
                "sourcePortRanges": [],
                "destinationPortRanges": [],
                "sourceAddressPrefixes": [],
                "destinationAddressPrefixes": []
            }
      ```

13. Cliquez sur **Enregistrer** dans l’éditeur en ligne.

14. Cliquez sur **DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel le groupe de sécurité réseau sera déployé.

15. Cliquez sur **DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel le groupe de sécurité réseau sera déployé.  Vous pouvez cliquer sur **Créer** pour créer un groupe de ressources pour le groupe de sécurité réseau cible.  Vérifiez que le nom n’est pas identique à celui du groupe de ressources source du groupe de sécurité réseau existant.

16. Vérifiez que **DE BASE** > **Emplacement** est défini avec l’emplacement cible où vous souhaitez déployer le groupe de sécurité réseau.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré dans l’éditeur de paramètres ci-dessus.

18. Cochez la case sous **CONDITIONS GÉNÉRALES**.

19. Cliquez sur le bouton **Acheter** pour déployer le groupe de sécurité réseau cible.

## <a name="discard"></a>Abandonner

Si vous souhaitez abandonner le groupe de sécurité réseau cible, supprimez le groupe de ressources contenant le groupe de sécurité réseau cible.  Pour ce faire, sélectionnez le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de la page de vue d’ensemble.

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement du groupe de sécurité réseau, supprimez le groupe de ressources ou le groupe de sécurité réseau source. Pour ce faire, sélectionnez le groupe de sécurité réseau ou le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de chaque page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé un groupe de sécurité réseau Azure d’une région à une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
