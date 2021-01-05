---
title: 'Tutoriel : Créer une passerelle NAT - Portail Azure'
titlesuffix: Azure Virtual Network NAT
description: Ce guide de démarrage rapide montre comment créer une passerelle NAT à l’aide du portail Azure
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a NAT gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/24/2020
ms.author: allensu
ms.openlocfilehash: 5010dc08b695a0376ace5dde935c63caf0c39633
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012071"
---
# <a name="tutorial-create-a-nat-gateway-using-the-azure-portal"></a>Tutoriel : Créer une passerelle NAT avec le portail Azure

Ce tutoriel vous montre comment utiliser le service NAT de réseau virtuel Azure. Vous allez créer une passerelle NAT pour fournir une connectivité sortante à une machine virtuelle dans Azure. 

Si vous préférez, au lieu d’utiliser le portail, vous pouvez effectuer ces étapes dans [Azure CLI](quickstart-create-nat-gateway-cli.md), [Azure PowerShell](quickstart-create-nat-gateway-powershell.md) ou déployer un [modèle ARM](quickstart-create-nat-gateway-powershell.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Avant de déployer une machine virtuelle et de pouvoir utiliser votre passerelle NAT, il nous faut créer le groupe de ressources et le réseau virtuel.

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupNAT |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | USA Est 2      |
| **\<IPv4-address-space>**   | 192.168.0.0\16          |
| **\<subnet-name>**          | mySubnet        |
| **\<subnet-address-range>** | 192.168.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-a-vm-to-use-the-nat-gateway"></a>Créer une machine virtuelle pour utiliser la passerelle NAT

Nous allons à présent créer une machine virtuelle pour utiliser le service NAT. Cette machine virtuelle dispose d’une adresse IP publique à utiliser comme adresse IP publique au niveau de l’instance pour vous permettre d’accéder à la machine virtuelle. Le service NAT reconnaît la direction du flux et remplace la destination Internet par défaut dans votre sous-réseau. L’adresse IP publique de la machine virtuelle n’est pas utilisée pour les connexions sortantes.

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Compute** > **Ubuntu Server 18.04 LTS**, ou recherchez **Ubuntu Server 18.04 LTS** dans la recherche de la Place de marché.

2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : Tapez **myVM**.
   - **Détails de l’instance** > **Région** > sélectionnez **USA Est 2**.
   - **Compte d’administrateur** > **Authentification, tapez** : Sélectionnez **Mot de passe**.
   - **Compte d’administrateur** > Entrez les informations **Nom d’utilisateur**, **Mot de passe** et **Confirmer le mot de passe**.
   - **Règles des ports d’entrée** > **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Règles des ports d’entrée** > **Sélectionner des ports d’entrée** : Sélectionnez **SSH (22)** .
   - Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.

3. Dans l’onglet **Mise en réseau**, vérifiez que les points suivants sont sélectionnés :
   - **Réseau virtuel** : **MyVNet**
   - **Sous-réseau** : **mySubnet**
   - **Adresse IP publique** > Sélectionnez **Créer**.  Dans la fenêtre **Créer une adresse IP publique**, tapez **myPublicIPVM** dans le champ **Nom**, puis choisissez **Standard** comme référence **SKU**.  Cliquez sur **OK**.
   - **Groupe de sécurité réseau de la carte réseau** : Sélectionnez **De base**.
   - **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Sélectionner des ports d’entrée** : Vérifiez que l’option **SSH** est sélectionnée.

4. Dans l’onglet **Gestion**, sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.

5. Sélectionnez **Revoir + créer**. 

6. Vérifiez les paramètres, puis cliquez sur **Créer**.

## <a name="create-the-nat-gateway"></a>Créer la passerelle NAT

Vous pouvez utiliser une ou plusieurs ressources d’adresse IP publique, des préfixes d’adresse IP publique, ou ces deux options à la fois. Nous allons ajouter une ressource d’adresse IP publique, un préfixe d’adresse IP publique et une ressource de passerelle NAT.

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants du service NAT à l’aide de la ressource de passerelle NAT :
  - Un pool d’adresses IP publiques et un préfixe d’adresse IP publique à utiliser pour les flux sortants qui sont traduits par la ressource de passerelle NAT.
  - Passer la valeur par défaut du délai d’inactivité de 4 minutes à 10 minutes.

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Adresse IP publique**, ou recherchez **Adresse IP publique** dans la recherche de la Place de marché.

2. Dans **Créer une adresse IP publique**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Version de l’adresse IP | Sélectionnez **IPv4**.
    | SKU | Sélectionnez **Standard**.
    | Nom | Entrez **MyPublicIP**. |
    | Abonnement | Sélectionnez votre abonnement.|
    | Resource group | Sélectionnez **myResourceGroupNAT**. |
    | Emplacement | Sélectionnez **USA Est**.|

3. Conservez les autres valeurs par défaut et sélectionnez **Créer**.

### <a name="create-a-public-ip-prefix"></a>Créer un préfixe d’adresse IP publique

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Préfixe d’adresse IP publique**, ou recherchez **Préfixe d’adresse IP publique** dans la recherche de la Place de marché. 

2. Dans **Créer un préfixe d’adresse IP publique**, entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions de base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**>
   - **Détails de l’instance** > **Nom** : Tapez **myPublicIPprefix**.
   - **Détails de l’instance** > **Région** : Sélectionnez **USA Est**.
   - **Détails de l’instance** > **Taille du préfixe** : Sélectionnez **/31 (2 adresses)**

3. Conservez les autres valeurs par défaut et sélectionnez **Vérifier + créer**.

4. Passez en revue les paramètres, puis sélectionnez **Créer**.
   

### <a name="create-a-nat-gateway-resource"></a>Créer une ressource de passerelle NAT

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Passerelle NAT**, ou recherchez **Passerelle NAT** dans la recherche de la Place de marché.

2. Dans **Créer une passerelle NAT (traduction d’adresses réseau)** , entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions de base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**.
   - **Détails de l’instance** > **Nom de la passerelle NAT** : Tapez **myNATgateway**.
   - **Détails de l’instance** > **Région** : Sélectionnez **USA Est**.
   - **Détails de l’instance** > **Délai d’inactivité (minutes)**  : Tapez **10**.
   - Sélectionnez l’onglet **IP publique**, ou sélectionnez **Suivant : IP publique**.

3. Sous l’onglet **IP publique**, tapez ou sélectionnez les valeurs suivantes :
   - **Adresses IP publiques** : Sélectionnez **myPublicIP**.
   - **Préfixes d’adresses IP publiques** : Sélectionnez **myPublicIPprefix**.
   - Sélectionnez l’onglet **Sous-réseau**, ou sélectionnez **Suivant : Sous-réseau**.

4. Sous l’onglet **Sous-réseau**, tapez ou sélectionnez les valeurs suivantes :
   - **Réseau virtuel** : Sélectionnez **myResourceGroupNAT** > **myVnet**.
   - **Nom du sous-réseau** : Activez la case à cocher en regard de **mySubnet**.

5. Sélectionnez **Revoir + créer**.

6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="discover-the-ip-address-of-the-vm"></a>Découvrir l’adresse IP de la machine virtuelle

1. Dans la partie gauche du portail, sélectionnez **Groupes de ressources**.
2. Sélectionnez **myResourceGroupNAT**.
3. Sélectionnez **myVM**.
4. Dans **Vue d’ensemble** copiez la valeur **Adresse IP publique**, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser pour accéder à la machine virtuelle.

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser pour accéder à la machine virtuelle.

## <a name="sign-in-to-vm"></a>Se connecter à la machine virtuelle

Ouvrez une session [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur. Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle.

```azurecli-interactive
ssh <username>@<ip-address-destination>
```

Vous êtes désormais prêt à utiliser le service NAT.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la passerelle NAT et toutes les ressources associées. Sélectionnez le groupe de ressources **myResourceGroupNAT** qui contient la passerelle NAT, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une passerelle NAT et une machine virtuelle pour l’utiliser. 

Consultez les métriques dans Azure Monitor pour découvrir le fonctionnement de votre service NAT. Diagnostiquez des problèmes, tels que l’épuisement des ressources des ports SNAT disponibles.  L’épuisement des ressources de ports SNAT est résolu en ajoutant des ressources supplémentaires, d’adresse IP publique ou de préfixe d’adresse IP publique, ou des deux à la fois.


- Apprenez-en davantage sur le service [NAT de Réseau virtuel Azure](./nat-overview.md).
- Apprenez-en davantage sur la [ressource de passerelle NAT](./nat-gateway-resource.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure CLI](./quickstart-create-nat-gateway-cli.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec le portail Azure](./quickstart-create-nat-gateway-portal.md).
> [!div class="nextstepaction"]

