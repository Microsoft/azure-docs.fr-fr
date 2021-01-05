---
title: 'Tutoriel : Créer et tester une passerelle NAT - Portail Azure'
titlesuffix: Azure Virtual Network NAT
description: Ce tutoriel montre comment créer une passerelle NAT avec le portail Azure, et tester le service NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumundD
Customer intent: I want to test a NAT Gateway for outbound connectivity for my virtual network.
ms.service: virtual-network
ms.subservice: nat
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/24/2020
ms.author: allensu
ms.openlocfilehash: d798725892a9586c17cd7023863fe5cf7df05cb6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84417835"
---
# <a name="tutorial-create-a-nat-gateway-using-the-azure-portal-and-test-the-nat-service"></a>Tutoriel : Créer une passerelle NAT avec le portail Azure et tester le service NAT

Dans ce tutoriel, vous allez créer une passerelle NAT pour fournir une connectivité sortante à des machines virtuelles dans Azure. Pour tester la passerelle NAT, vous déployez une machine virtuelle source et une machine virtuelle de destination. Vous allez tester la passerelle NAT en établissant des connexions sortantes à une adresse IP publique, de la machine virtuelle source vers la machine virtuelle de destination.  Ce tutoriel déploie la source et la destination sur deux réseaux virtuels différents dans le même groupe de ressources dans l’unique but de rester simple.

Si vous préférez, vous pouvez effectuer ces étapes dans [Azure CLI](tutorial-create-validate-nat-gateway-cli.md) ou [Azure PowerShell](tutorial-create-validate-nat-gateway-powershell.md) au lieu du portail.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="prepare-the-source-for-outbound-traffic"></a>Préparer la source du trafic sortant

Nous allons vous guider tout au long de la configuration d’un environnement de test complet et de l’exécution même des tests dans les étapes suivantes. Nous allons commencer par la source, qui utilisera la ressource de passerelle NAT que nous créerons dans les étapes suivantes.

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Avant de déployer une machine virtuelle et de pouvoir utiliser votre passerelle NAT, il nous faut créer le groupe de ressources et le réseau virtuel.

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupNAT |
| **\<virtual-network-name>** | myVNetsource          |
| **\<region-name>**          | USA Est 2      |
| **\<IPv4-address-space>**   | 192.168.0.0/16          |
| **\<subnet-name>**          | mySubnetsource        |
| **\<subnet-address-range>** | 192.168.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-source-virtual-machine"></a>Créer la machine virtuelle source

Nous allons à présent créer une machine virtuelle pour utiliser le service NAT. Cette machine virtuelle dispose d’une adresse IP publique à utiliser comme adresse IP publique au niveau de l’instance pour vous permettre d’accéder à la machine virtuelle. Le service NAT reconnaît la direction du flux et remplace la destination Internet par défaut dans votre sous-réseau. L’adresse IP publique de la machine virtuelle n’est pas utilisée pour les connexions sortantes.

Afin de tester la passerelle NAT, nous allons attribuer une ressource d’adresse IP publique en tant qu’adresse IP publique au niveau de l’instance pour accéder à cette machine virtuelle de l’extérieur. Cette adresse est uniquement utilisée pour y accéder dans le cadre du test.  Nous allons vous montrer comment le service NAT est prioritaire sur les autres options sortantes.

Vous pouvez également créer cette machine virtuelle sans adresse IP publique, et créer une autre machine virtuelle à utiliser comme jumpbox sans adresse IP publique dans le cadre d’un exercice.

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Compute** > **Ubuntu Server 18.04 LTS**, ou recherchez **Ubuntu Server 18.04 LTS** dans la recherche de la Place de marché.

2. Dans **Créer une machine virtuelle**, entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : entrez **MyVMsource**.
   - **Détails de l’instance** > **Région** > sélectionnez **USA Est 2**.
   - **Compte d’administrateur** > **Authentification, entrez** : Sélectionnez **Mot de passe**.
   - **Compte d’administrateur** > Entrez les informations **Nom d’utilisateur**, **Mot de passe** et **Confirmer le mot de passe**.
   - **Règles des ports d’entrée** > **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Règles des ports d’entrée** > **Sélectionner des ports d’entrée** : Sélectionnez **SSH (22)** .
   - Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.

3. Dans l’onglet **Mise en réseau**, vérifiez que les points suivants sont sélectionnés :
   - **Réseau virtuel** : **myVnetsource**
   - **Sous-réseau** : **mySubnetsource**
   - **Adresse IP publique** > Sélectionnez **Créer**.  Dans la fenêtre **Créer une adresse IP publique**, entrez **myPublicIPsourceVM** dans le champ **Nom**. Sélectionnez **Standard** pour la référence **SKU**. Laissez les autres valeurs par défaut et cliquez sur **OK**.
   - **Groupe de sécurité réseau de la carte réseau** : Sélectionnez **De base**.
   - **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Sélectionner des ports d’entrée** : Vérifiez que l’option **SSH** est sélectionnée.

4. Dans l’onglet **Gestion**, sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.

5. Sélectionnez **Revoir + créer**.

6. Vérifiez les paramètres, puis cliquez sur **Créer**.

## <a name="create-the-nat-gateway"></a>Créer la passerelle NAT

Vous pouvez utiliser une ou plusieurs ressources d’adresse IP publique, des préfixes d’adresse IP publique, ou ces deux options à la fois avec la passerelle NAT. Nous allons ajouter une ressource d’adresse IP publique, un préfixe d’adresse IP publique et une ressource de passerelle NAT.

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
    | Nom | Entrez **myPublicIPsource**. |
    | Abonnement | Sélectionnez votre abonnement.|
    | Resource group | Sélectionnez **myResourceGroupNAT**. |
    | Emplacement | Sélectionnez **USA Est**.|

3. Conservez les autres valeurs par défaut et sélectionnez **Créer**.

### <a name="create-a-public-ip-prefix"></a>Créer un préfixe d’adresse IP publique

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Préfixe d’adresse IP publique**, ou recherchez **Préfixe d’adresse IP publique** dans la recherche de la Place de marché.

2. Dans **Créer un préfixe d’adresse IP publique**, entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions de base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**>
   - **Détails de l’instance** > **Nom** : entrez **myPublicIPprefixsource**.
   - **Détails de l’instance** > **Région** : Sélectionnez **USA Est**.
   - **Détails de l’instance** > **Taille du préfixe** : Sélectionnez **/31 (2 adresses)**

3. Conservez les autres valeurs par défaut et sélectionnez **Vérifier + créer**.

4. Passez en revue les paramètres, puis sélectionnez **Créer**.


### <a name="create-a-nat-gateway-resource"></a>Créer une ressource de passerelle NAT

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Passerelle NAT**, ou recherchez **Passerelle NAT** dans la recherche de la Place de marché.

2. Dans **Créer une passerelle NAT (traduction d’adresses réseau)** , entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions de base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**.
   - **Détails de l’instance** > **Nom de la passerelle NAT** : entrez **myNATgateway**.
   - **Détails de l’instance** > **Région** : Sélectionnez **USA Est**.
   - **Détails de l’instance** > **Délai d’inactivité (minutes)**  : entrez **10**.
   - Sélectionnez l’onglet **IP publique**, ou sélectionnez **Suivant : IP publique**.

3. Sous l’onglet **IP publique**, entrez ou sélectionnez les valeurs suivantes :
   - **Adresses IP publiques** : Sélectionnez **myPublicIPsource**.
   - **Préfixes d’adresses IP publiques** : Sélectionnez **myPublicIPprefixsource**.
   - Sélectionnez l’onglet **Sous-réseau**, ou sélectionnez **Suivant : Sous-réseau**.

4. Sous l’onglet **Sous-réseau**, entrez ou sélectionnez les valeurs suivantes :
   - **Réseau virtuel** : Sélectionnez **myResourceGroupNAT** > **myVnetsource**.
   - **Nom du sous-réseau** : Activez la case à cocher en regard de **mySubnetsource**.

5. Sélectionnez **Revoir + créer**.

6. Passez en revue les paramètres, puis sélectionnez **Créer**.

Tout le trafic sortant vers les destinations Internet utilise désormais le service NAT.  Il n’est pas nécessaire de configurer un UDR.


## <a name="prepare-destination-for-outbound-traffic"></a>Préparer la destination pour le trafic sortant

Nous allons maintenant créer une destination pour le trafic sortant qui est traduit par le service NAT, et ainsi vous permettre de le tester.


## <a name="virtual-network-and-parameters-for-destination"></a>Réseau virtuel et paramètres pour la destination

Avant de déployer une machine virtuelle pour la destination, nous devons créer un réseau virtuel sur lequel la machine virtuelle de destination peut résider. Les étapes suivantes sont les mêmes que celles de la machine virtuelle source, avec quelques modifications mineures pour exposer le point de terminaison de destination.

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupNAT |
| **\<virtual-network-name>** | myVNetdestination          |
| **\<region-name>**          | USA Est 2      |
| **\<IPv4-address-space>**   | 10.1.0.0/16          |
| **\<subnet-name>**          | mySubnetdestination        |
| **\<subnet-address-range>** | 10.1.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-destination-virtual-machine"></a>Créer la machine virtuelle de destination

1. En haut à gauche dans le portail, sélectionnez **Créer une ressource** > **Compute** > **Ubuntu Server 18.04 LTS**, ou recherchez **Ubuntu Server 18.04 LTS** dans la recherche de la Place de marché.

2. Dans **Créer une machine virtuelle**, entrez ou sélectionnez les valeurs suivantes sous l’onglet **Fonctions base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupNAT**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : entrez **myVMdestination**.
   - **Détails de l’instance** > **Région** > sélectionnez **USA Est 2**.
   - **Compte d’administrateur** > **Authentification, entrez** : Sélectionnez **Mot de passe**.
   - **Compte d’administrateur** > Entrez les informations **Nom d’utilisateur**, **Mot de passe** et **Confirmer le mot de passe**.
   - **Règles des ports d’entrée** > **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Règles des ports d’entrée** > **Sélectionner des ports d’entrée** : Sélectionnez **SSH (22)** et **HTTP (80)** .
   - Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.

3. Dans l’onglet **Mise en réseau**, vérifiez que les points suivants sont sélectionnés :
   - **Réseau virtuel** : **myVnetdestination**
   - **Sous-réseau** : **mySubnetdestination**
   - **Adresse IP publique** > Sélectionnez **Créer**.  Dans la fenêtre **Créer une adresse IP publique**, entrez **myPublicIPdestinationVM** dans le champ **Nom**. Sélectionnez **Standard** pour **SKU**. Laissez les autres valeurs par défaut et cliquez sur **OK**.
   - **Groupe de sécurité réseau de la carte réseau** : Sélectionnez **De base**.
   - **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Sélectionner des ports d’entrée** : Vérifiez que les options **SSH** et **HTTP** sont sélectionnées.

4. Dans l’onglet **Gestion**, sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.

5. Sélectionnez **Revoir + créer**.

6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="prepare-a-web-server-and-test-payload-on-destination-vm"></a>Préparer un serveur web et une charge utile de test sur la machine virtuelle de destination

Tout d’abord, nous avons besoin de découvrir l’adresse IP de la machine virtuelle de destination. 

1. Dans la partie gauche du portail, sélectionnez **Groupes de ressources**.
2. Sélectionnez **myResourceGroupNAT**.
3. Sélectionnez **myVMdestination**.
4. Dans **Vue d’ensemble** copiez la valeur **Adresse IP publique**, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser pour accéder à la machine virtuelle.

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser dans les étapes suivantes. Indiquez qu’il s’agit de la machine virtuelle de destination.

### <a name="sign-in-to-destination-vm"></a>Se connecter à la machine virtuelle de destination

Ouvrez une session [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur. Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle.

```azurecli-interactive
ssh <username>@<ip-address-destination>
```

Copiez et collez les commandes suivantes une fois que vous êtes connecté.  

```bash
sudo apt-get -y update && \
sudo apt-get -y upgrade && \
sudo apt-get -y dist-upgrade && \
sudo apt-get -y autoremove && \
sudo apt-get -y autoclean && \
sudo apt-get -y install nginx && \
sudo ln -sf /dev/null /var/log/nginx/access.log && \
sudo touch /var/www/html/index.html && \
sudo rm /var/www/html/index.nginx-debian.html && \
sudo dd if=/dev/zero of=/var/www/html/100k bs=1024 count=100
```

Ces commandes mettent à jour votre machine virtuelle, installent Nginx et créent un fichier de 100 Ko. Ce fichier sera récupéré à partir de la machine virtuelle source par le biais du service NAT.

Fermez la session SSH avec la machine virtuelle de destination.

## <a name="prepare-test-on-source-vm"></a>Préparer le test sur la machine virtuelle source

Tout d’abord, nous avons besoin de découvrir l’adresse IP de la machine virtuelle source.

1. Dans la partie gauche du portail, sélectionnez **Groupes de ressources**.
2. Sélectionnez **myResourceGroupNAT**.
3. Sélectionnez **myVMsource**.
4. Dans **Vue d’ensemble** copiez la valeur **Adresse IP publique**, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser pour accéder à la machine virtuelle.

>[!IMPORTANT]
>Copiez l’adresse IP publique, puis collez-la dans un bloc-notes afin de pouvoir l’utiliser dans les étapes suivantes. Indiquez qu’il s’agit de la machine virtuelle source.

### <a name="log-into-source-vm"></a>Se connecter à la machine virtuelle source

Ouvrez un nouvel onglet pour [Azure Cloud Shell](https://shell.azure.com) dans votre navigateur.  Utilisez l’adresse IP récupérée à l’étape précédente pour établir une connexion SSH à la machine virtuelle. 

```azurecli-interactive
ssh <username>@<ip-address-source>
```

Copiez et collez les commandes suivantes pour préparer le test du service NAT.

```bash
sudo apt-get -y update && \
sudo apt-get -y upgrade && \
sudo apt-get -y dist-upgrade && \
sudo apt-get -y autoremove && \
sudo apt-get -y autoclean && \
sudo apt-get install -y nload golang && \
echo 'export GOPATH=${HOME}/go' >> .bashrc && \
echo 'export PATH=${PATH}:${GOPATH}/bin' >> .bashrc && \
. ~/.bashrc &&
go get -u github.com/rakyll/hey

```

Cette commande met à jour votre machine virtuelle, installe Go, installe [hey](https://github.com/rakyll/hey) à partir de GitHub, et met à jour votre environnement d’interpréteur de commandes.

Vous êtes désormais prêt à tester le service NAT.

## <a name="validate-nat-service"></a>Valider le service NAT

Quand vous êtes connecté à la machine virtuelle source, vous pouvez utiliser **curl** et **hey** pour générer des requêtes vers l’adresse IP de destination.

Utilisez curl pour récupérer le fichier de 100 Ko.  Dans l’exemple ci-dessous, remplacez **\<ip-address-destination>** par l’adresse IP de destination que vous avez copiée précédemment.  Le paramètre **--output** indique que le fichier récupéré sera ignoré.

```bash
curl http://<ip-address-destination>/100k --output /dev/null
```

Vous pouvez également générer une série de requêtes à l’aide de **hey**. Là encore, remplacez **\<ip-address-destination>** par l’adresse IP de destination que vous avez copiée précédemment.

```bash
hey -n 100 -c 10 -t 30 --disable-keepalive http://<ip-address-destination>/100k
```

Cette commande génère 100 requêtes, 10 simultanément, avec un délai d’expiration de 30 secondes, et sans réutiliser la connexion TCP.  Chaque demande récupère 100 Ko.  À la fin de l’exécution, **hey** fournit quelques statistiques sur le fonctionnement du service NAT.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, la passerelle NAT et toutes les ressources associées. Sélectionnez le groupe de ressources **myResourceGroupNAT** qui contient la passerelle NAT, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez créé une passerelle NAT, créé une machine virtuelle source et une machine virtuelle de destination, puis testé la passerelle NAT.

Consultez les métriques dans Azure Monitor pour découvrir le fonctionnement de votre service NAT. Diagnostiquez des problèmes, tels que l’épuisement des ressources des ports SNAT disponibles.  L’épuisement des ressources de ports SNAT est résolu facilement en ajoutant des ressources supplémentaires, d’adresse IP publique ou de préfixe d’adresse IP publique, ou des deux à la fois.

- Apprenez-en davantage sur le service [Nat de Réseau virtuel](./nat-overview.md).
- Apprenez-en davantage sur la [ressource de passerelle NAT](./nat-gateway-resource.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure CLI](./quickstart-create-nat-gateway-cli.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec Azure PowerShell](./quickstart-create-nat-gateway-powershell.md).
- Guide de démarrage rapide du déploiement d’une [ressource de passerelle NAT avec le portail Azure](./quickstart-create-nat-gateway-portal.md).

> [!div class="nextstepaction"]

