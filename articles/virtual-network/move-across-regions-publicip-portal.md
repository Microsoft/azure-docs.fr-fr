---
title: Déplacer la configuration des adresses IP publiques Azure vers une autre région Azure via Portail Azure
description: Utilisez un modèle pour déplacer la configuration des adresses IP publiques d’une région Azure vers une autre à l’aide du portail Azure.
author: asudbring
ms.service: virtual-network
ms.subservice: ip-services
ms.topic: how-to
ms.date: 08/29/2019
ms.author: allensu
ms.openlocfilehash: 23fe515ddfdecb9ef168dd662e3fa2d91ece688f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84711474"
---
# <a name="move-azure-public-ip-configuration-to-another-region-using-the-azure-portal"></a>Déplacer la configuration des adresses IP publiques Azure vers une autre région à l’aide du portail Azure

Il existe différents scénarios dans lesquels vous pouvez être amené à déplacer les configurations de vos adresses IP publiques Azure existantes d’une région à une autre. Par exemple, vous pouvez avoir besoin de créer une adresse IP publique avec la même configuration et la même référence SKU à des fins de test. Vous pouvez également déplacer une configuration d’adresse IP publique vers une autre région dans le cadre de la planification de la récupération d’urgence.

**Les adresses IP publiques Azure sont spécifiques à une région et ne peuvent pas être déplacées d’une région à une autre.** Toutefois, vous pouvez utiliser un modèle Azure Resource Manager pour exporter la configuration existante d’une adresse IP publique.  Vous pouvez ensuite déplacer la ressource dans une autre région en exportant l’adresse IP publique vers un modèle, en modifiant les paramètres pour qu’ils correspondent à la région de destination, puis en déployant le modèle dans la nouvelle région.  Pour plus d’informations sur Resource Manager et les modèles, consultez [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal).


## <a name="prerequisites"></a>Prérequis

- Vérifiez que l’adresse IP publique Azure se trouve dans la région Azure à partir de laquelle vous souhaitez effectuer le déplacement.

- Vous ne pouvez pas déplacer les adresses IP publiques Azure entre les régions.  Vous devez associer la nouvelle adresse IP publique à des ressources dans la région cible.

- Pour exporter une configuration d’adresse IP publique et déployer un modèle pour créer une adresse IP publique dans une autre région, vous devez disposer au minimum du rôle Contributeur de réseaux.

- Identifiez la topologie du réseau source et toutes les ressources que vous utilisez actuellement. Cette disposition comprend notamment les équilibreurs de charge, les groupes de sécurité réseau et les réseaux virtuels.

- Vérifiez que votre abonnement Azure vous permet de créer des adresses IP publiques dans la région cible utilisée. Contactez le support pour activer le quota requis.

- Vérifiez que votre abonnement dispose de suffisamment de ressources pour prendre en charge l’ajout d’adresses IP publiques pour ce processus.  Consultez [Abonnement Azure et limites, quotas et contraintes de service](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#networking-limits).


## <a name="prepare-and-move"></a>Préparer et déplacer
Les étapes suivantes montrent comment préparer l’adresse IP publique pour le déplacement à l’aide d’un modèle Resource Manager et comment déplacer la configuration de l’adresse IP publique vers la région cible à l’aide du Portail Azure.

### <a name="export-the-template-and-deploy-from-a-script"></a>Exporter le modèle et le déployer à partir d’un script

1. Connectez-vous au [Portail Azure](https://portal.azure.com) > **Groupes de ressources**.
2. Recherchez le groupe de ressources qui contient l’adresse IP publique source et cliquez dessus.
3. Sélectionnez **Paramètres** > **Exporter le modèle**.
4. Choisissez **Déployer** dans le panneau **Exporter le modèle**.
5. Cliquez sur **MODÈLE** > **Modifier les paramètres** pour ouvrir le fichier **parameters.json** dans l’éditeur en ligne.
8. Pour modifier le paramètre du nom d’adresse IP publique, remplacez la propriété sous **parameters** > **value** par le nom de votre adresse IP publique cible, en veillant à placer le nom entre guillemets :

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
8.  Cliquez sur **Enregistrer** dans l’éditeur.

9.  Cliquez sur **MODÈLE** > **Modifier le modèle** pour ouvrir le fichier **template.json** dans l’éditeur en ligne.

10. Pour modifier la région cible où l’adresse IP publique sera déplacée, changez la propriété **location** sous **resources** :

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

11. Pour obtenir les codes d’emplacement des régions, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Le code d’une région est le nom de la région sans espace, **USA Centre** = **centralus**.

12. Vous pouvez également changer d’autres paramètres dans le modèle ; ces paramètres sont facultatifs en fonction de vos besoins :

    * **Référence SKU** : vous pouvez permuter la référence SKU de l’adresse IP publique dans la configuration entre les valeurs basic et standard en modifiant la propriété **sku** > **name** dans le fichier **template.json** :

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

        Pour plus d’informations sur les différences entre les adresses IP publiques des références SKU basic et standard, consultez [Créer, modifier ou supprimer une adresse IP publique](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address) :

    * **Méthode d’allocation d’adresse IP publique** et **Délai d’inactivité** : vous pouvez changer ces deux options dans le modèle en permutant la propriété **publicIPAllocationMethod** entre les valeurs **Dynamic** et **Static**. Vous pouvez changer le délai d’inactivité en affectant la valeur souhaitée à la propriété **idleTimeoutInMinutes**.  La valeur par défaut est **4** :

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

        Pour plus d’informations sur les méthodes d’allocation et les valeurs de délai d’inactivité, consultez [Créer, modifier ou supprimer une adresse IP publique](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address).


13. Cliquez sur **Enregistrer** dans l’éditeur en ligne.

14. Cliquez sur **DE BASE** > **Abonnement** pour choisir l’abonnement dans lequel l’adresse IP publique cible sera déployée.

15. Cliquez sur **DE BASE** > **Groupe de ressources** pour choisir le groupe de ressources dans lequel l’adresse IP publique cible sera déployée.  Vous pouvez cliquer sur **Créer** afin de créer un groupe de ressources pour l’adresse IP publique cible.  Vérifiez que le nom n’est pas identique à celui du groupe de ressources source de l’adresse IP publique cible existante.

16. Vérifiez que **DE BASE** > **Emplacement** est défini avec l’emplacement cible où vous souhaitez déployer l’adresse IP publique.

17. Sous **PARAMÈTRES**, vérifiez que le nom correspond à celui que vous avez entré dans l’éditeur de paramètres ci-dessus.

18. Cochez la case sous **CONDITIONS GÉNÉRALES**.

19. Cliquez sur le bouton **Acheter** pour déployer l’adresse IP publique cible.

## <a name="discard"></a>Abandonner

Si vous souhaitez abandonner l’adresse IP publique cible, supprimez le groupe de ressources contenant l’adresse IP publique cible.  Pour ce faire, sélectionnez le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de la page de vue d’ensemble.

## <a name="clean-up"></a>Nettoyer

Pour valider les modifications et terminer le déplacement de l’adresse IP publique cible, supprimez le groupe de ressources ou l’adresse IP publique source. Pour ce faire, sélectionnez l’adresse IP publique ou le groupe de ressources à partir de votre tableau de bord dans le portail, puis sélectionnez **Supprimer** en haut de chaque page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déplacé une adresse IP publique Azure d’une région à une autre et nettoyé les ressources sources.  Pour en savoir plus sur le déplacement de ressources entre régions et la reprise d’activité après sinistre dans Azure, consultez :


- [Déplacer des ressources vers un nouveau groupe de ressource ou un nouvel abonnement](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-move-resources)
- [Déplacer des machines virtuelles Azure vers une autre région](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-tutorial-migrate)
