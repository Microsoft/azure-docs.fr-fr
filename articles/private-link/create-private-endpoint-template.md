---
title: Créer un point de terminaison privé dans Azure Private Link
description: Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer un point de terminaison privé.
services: private-link
author: mblanco77
ms.service: private-link
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: 553d36a86671617417b6b9b1ea47966c3ba3fdf6
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88705807"
---
# <a name="quickstart-create-a-private-endpoint-by-using-an-arm-template"></a>Démarrage rapide : Créer un point de terminaison privé à l’aide d’un modèle Resource Manager

Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer un point de terminaison privé.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Vous pouvez également suivre ce guide de démarrage rapide en utilisant le [portail Azure](create-private-endpoint-portal.md), [Azure PowerShell](create-private-endpoint-powershell.md) ou l’interface [Azure CLI](create-private-endpoint-cli.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Vous devez avoir un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="review-the-template"></a>Vérifier le modèle

Ce modèle crée un point de terminaison privé pour une instance d’Azure SQL Database.

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-private-endpoint-sql/).

:::code language="json" source="~/quickstart-templates/101-private-endpoint-sql/azuredeploy.json":::

Plusieurs ressources Azure sont définies dans le modèle :

- [**Microsoft.Sql/servers**](/azure/templates/microsoft.sql/servers) : l’instance de SQL Database contenant l’exemple de base de données.
- [**Microsoft.Sql/servers/databases**](/azure/templates/microsoft.sql/servers/databases) : l’exemple de base de données.
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks) : le réseau virtuel sur lequel le point de terminaison privé est déployé.
- [**Microsoft.Network/privateEndpoints**](/azure/templates/microsoft.network/privateendpoints) : le point de terminaison privé pour accéder à l’instance de SQL Database.
- [**Microsoft.Network/privateDnsZones**](/azure/templates/microsoft.network/privatednszones) : la zone utilisée pour résoudre l’adresse IP du point de terminaison privé.
- [**Microsoft.Network/privateDnsZones/virtualNetworkLinks**](/azure/templates/microsoft.network/privatednszones/virtualnetworklinks)
- [**Microsoft.Network/privateEndpoints/privateDnsZoneGroups**](/azure/templates/microsoft.network/privateendpoints/privateDnsZoneGroups) : le groupe de zones utilisé pour associer le point de terminaison privé à une zone DNS privée.
- [**Microsoft.Network/publicIpAddresses**](/azure/templates/microsoft.network/publicIpAddresses) : l’adresse IP publique utilisée pour accéder à la machine virtuelle.
- [**Microsoft.Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces) : l’interface réseau de la machine virtuelle.
- [**Microsoft.Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines) : la machine virtuelle utilisée pour tester la connexion privée avec le point de terminaison privé sur l’instance de SQL Database.

## <a name="deploy-the-template"></a>Déployer le modèle

Voici comment déployer le modèle ARM sur Azure :

1. Pour vous connecter à Azure et ouvrir le modèle, sélectionnez **Déployer sur Azure**. Le modèle crée le point de terminaison privé, l’instance de SQL Database, l’infrastructure réseau et la machine virtuelle à valider.

   [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-sql%2Fazuredeploy.json)

2. Sélectionnez ou créez votre groupe de ressources.
3. Entrez le nom de connexion de l’administrateur SQL et le mot de passe associé.
4. Saisissez le nom d’utilisateur et le mot de passe administrateur de la machine virtuelle.
5. Lisez la déclaration des conditions générales. Si vous acceptez, sélectionnez **J’accepte les conditions générales mentionnées ci-dessus** > **Acheter**. Le déploiement peut prendre 20 minutes ou plus.

## <a name="validate-the-deployment"></a>Valider le déploiement

> [!NOTE]
> Le modèle Resource Manager génère un nom unique pour la ressource de la machine virtuelle myVm<b>{uniqueid}</b> et pour celle de SQL Database sqlserver<b>{uniqueid}</b>. Remplacez la valeur générée pour **{uniqueid}** .

### <a name="connect-to-a-vm-from-the-internet"></a>Se connecter à une machine virtuelle à partir d’Internet

Connectez-vous à la machine virtuelle _myVm{uniqueid}_ via Internet comme suit :

1. Dans la barre de recherche du portail, saisissez _myVm{uniqueid}_ .

2. Sélectionnez **Connecter**. **Se connecter à une machine virtuelle** s’ouvre.

3. Sélectionnez **Télécharger le fichier RDP**. Azure crée un fichier de protocole RDP (Remote Desktop Protocol) ( _.rdp_) et le télécharge sur votre ordinateur.

4. Ouvrez le fichier .rdp téléchargé.

   a. Si vous y êtes invité, sélectionnez **Connexion**.

   b. Entrez le nom d’utilisateur et le mot de passe spécifiés quand vous avez créé la machine virtuelle.

      > [!NOTE]
      > Vous devrez peut-être sélectionner **Plus de choix** > **Utiliser un autre compte** pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle.

5. Sélectionnez **OK**.

6. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez un avertissement de certificat, sélectionnez **Oui** ou **Continuer**.

7. Une fois le bureau de la machine virtuelle affiché, réduisez-le pour revenir à votre poste de travail local.

### <a name="access-the-sql-database-server-privately-from-the-vm"></a>Accéder au serveur SQL Database en privé à partir de la machine virtuelle

Voici comment vous allez vous connecter au serveur SQL Database depuis la machine virtuelle en utilisant le point de terminaison privé.

1.  Dans le Bureau à distance de _myVM{uniqueid}_ , ouvrez PowerShell.
2.  Entrez les informations suivantes : nslookup sqlserver{uniqueid}.database.windows.net. 
    Vous recevez un message similaire à celui ci :

    ```
      Server:  UnKnown
      Address:  168.63.129.16
      Non-authoritative answer:
      Name:    sqlserver.privatelink.database.windows.net
      Address:  10.0.0.5
      Aliases:  sqlserver.database.windows.net
    ```

3.  Installez SQL Server Management Studio.
4.  Dans  **Se connecter au serveur**, entrez ou sélectionnez les informations suivantes :
    - **Type de serveur** : Sélectionnez **Moteur de base de données**.
    - **Nom du serveur** : Sélectionnez **sqlserver{uniqueid}.database.windows.net**.
    - **Nom d’utilisateur** : Entrez le nom d’utilisateur fourni lors de la création.
    - **Mot de passe** : Entrez le mot de passe fourni lors de la création.
    - **Mémoriser le mot de passe** : Sélectionnez  **Oui**.

5.  Sélectionnez **Connecter**.
6.  Dans le menu de gauche, accédez à **Bases de données**.
7.  Si vous le souhaitez, vous pouvez créer ou interroger des informations à partir de _sample-db_.
8.  Fermez la connexion du Bureau à distance à _myVm{uniqueid}_ .

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le point de terminaison privé, supprimez le groupe de ressources. Cette action supprime le point de terminaison privé et toutes les ressources qui y sont associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Azure Private Link](private-link-overview.md).
