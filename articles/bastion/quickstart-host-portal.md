---
title: 'Démarrage rapide : Configurer Azure Bastion et se connecter à une machine virtuelle via une adresse IP privée et un navigateur'
titleSuffix: Azure Bastion
description: Dans cet article de démarrage rapide, découvrez comment créer un hôte Azure Bastion à partir d’une machine virtuelle et comment s’y connecter de manière sécurisée par le biais de votre navigateur à l’aide d’une adresse IP privée.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: quickstart
ms.date: 10/15/2020
ms.author: cherylmc
ms.openlocfilehash: 325f39b695d80c14ed7097d071380b937458546c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021484"
---
# <a name="quickstart-connect-to-a-vm-securely-through-a-browser-via-private-ip-address"></a>Démarrage rapide : Se connecter de manière sécurisée à une machine virtuelle par le biais d’un navigateur via une adresse IP privée

Vous pouvez vous connecter à une machine virtuelle par le biais de votre navigateur en utilisant le portail Azure et Azure Bastion. Cet article de démarrage rapide vous explique comment configurer Azure Bastion en fonction des paramètres de votre machine virtuelle, puis vous connecter à celle-ci par le biais du portail. La machine virtuelle n’a pas besoin d’une adresse IP publique, d’un logiciel client, d’un agent ou d’une configuration spéciale. Une fois le service approvisionné, l’expérience RDP/SSH est disponible pour toutes les machines virtuelles du même réseau virtuel. Pour plus d’informations sur Azure Bastion, consultez [Présentation d’Azure Bastion](bastion-overview.md).

## <a name="prerequisites"></a><a name="prereq"></a>Configuration requise

* Compte Azure avec un abonnement actif. Si vous n’en avez pas, [créez-en une gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Pour pouvoir vous connecter à une machine virtuelle par le biais de votre navigateur avec Bastion, vous devez être en mesure de vous connecter au portail Azure.

* Une machine virtuelle Windows dans un réseau virtuel. Si vous n’avez pas de machine virtuelle, créez-en une à l’aide du guide de [Démarrage rapide : Créer une machine virtuelle](../virtual-machines/windows/quick-create-portal.md).

  * Si vous avez besoin d’exemples de valeurs, consultez les [exemples de valeurs](#values) fournis.
  * Si vous disposez déjà d’un réseau virtuel, veillez à le sélectionner sous l’onglet Réseau lors de la création de votre machine virtuelle.
  * Si vous ne disposez pas encore d’un réseau virtuel, vous pouvez en créer un en même temps que votre machine virtuelle.
  * Vous n’avez pas besoin d’avoir une adresse IP publique pour cette machine virtuelle pour pouvoir vous connecter par le biais d’Azure Bastion.

* Rôles de machine virtuelle nécessaires :
  * Rôle de lecteur sur la machine virtuelle.
  * Rôle de lecteur sur la carte réseau avec adresse IP privée de la machine virtuelle.
  
* Ports de machine virtuelle nécessaires :
  * Ports d’entrée : RDP (3389)

### <a name="example-values"></a><a name="values"></a>Exemples de valeurs

Lors de la création de cette configuration, vous pouvez utiliser les exemples de valeurs suivants ou les remplacer par vos propres valeurs.

**Valeurs de machine virtuelle et de réseau virtuel de base :**

|**Nom** | **Valeur** |
| --- | --- |
| Machine virtuelle| TestVM |
| Resource group | TestRG |
| Région | USA Est |
| Réseau virtuel | TestVNet1 |
| Espace d’adressage | 10.0.0.0/16 |
| Sous-réseaux | Front-end : 10.0.0.0/24 |

**Valeurs Azure Bastion :**

|**Nom** | **Valeur** |
| --- | --- |
| Nom | TestVNet1-bastion |
| + Nom du sous-réseau | AzureBastionSubnet |
| Adresses AzureBastionSubnet | Un sous-réseau dans votre espace d’adressage de réseau virtuel avec un masque de sous-réseau /27. Par exemple, 10.0.1.0/27.  |
| Adresse IP publique |  Création |
| Nom de l’adresse IP publique | VNet1BastionPIP  |
| Référence SKU d’adresse IP publique |  standard  |
| Affectation  | statique |

## <a name="create-a-bastion-host"></a><a name="createvmset"></a>Créer un hôte Bastion

Il existe différentes manières de configurer un hôte bastion. Dans les étapes suivantes, vous allez créer un hôte bastion dans le portail Azure directement à partir de votre machine virtuelle. Quand vous créez un hôte à partir d’une machine virtuelle, différents paramètres sont automatiquement renseignés conformément à votre machine virtuelle et/ou votre réseau virtuel.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à la machine virtuelle à laquelle vous voulez vous connecter, puis sélectionnez **Se connecter**.

   :::image type="content" source="./media/quickstart-host-portal/vm-settings.png" alt-text="Paramètres de machine virtuelle" lightbox="./media/quickstart-host-portal/vm-settings.png":::
1. Dans la liste déroulante, sélectionnez **Bastion**.
1. Dans la page **TestVM | Se connecter**, sélectionnez **Utiliser Bastion**.

   :::image type="content" source="./media/quickstart-host-portal/select-bastion.png" alt-text="Sélectionner Bastion" border="false":::

1. Dans la page **Bastion**, renseignez les champs suivants :

   * **Nom** : nommez l’hôte bastion.
   * **Sous-réseau** : Il s’agit de l’espace d’adressage de réseau virtuel sur lequel la ressource Bastion sera déployée. Le sous-réseau doit être créé avec le nom **AzureBastionSubnet**. Utilisez un sous-réseau de /27 ou plus (/27, /26, /25, etc.).
   * Sélectionnez **Gérer la configuration du sous-réseau**.
1. Dans la page **Sous-réseaux**, sélectionnez **+ Sous-réseau**.

   :::image type="content" source="./media/quickstart-host-portal/subnet.png" alt-text="+ Sous-réseau":::
    
1. Dans la page **Ajouter un sous-réseau**, pour **Nom**, tapez **AzureBastionSubnet**.
   * Pour la plage d’adresses de sous-réseau, choisissez une adresse de sous-réseau qui se trouve dans votre espace d’adressage de réseau virtuel.
   * N’ajustez pas les autres paramètres. Sélectionnez **OK** pour accepter et enregistrer les changements de sous-réseau.

   :::image type="content" source="./media/quickstart-host-portal/add-subnet.png" alt-text="Ajouter un sous-réseau":::
1. Cliquez sur le bouton Précédent de votre navigateur pour revenir à la page **Bastion**, puis continuez à spécifier des valeurs.
   * **Adresse IP publique** : Laissez l’option **Créer** définie.
   * **Nom de l’adresse IP publique** : Nom de la ressource de l’adresse IP publique.
   * **Attribution** : Statique par défaut. Vous ne pouvez pas utiliser une attribution Dynamique pour Azure Bastion.
   * **Groupe de ressources** : Le même groupe de ressources que la machine virtuelle.

   :::image type="content" source="./media/quickstart-host-portal/validate.png" alt-text="Créer l’hôte bastion":::
1. Sélectionnez **Créer** pour créer l’hôte bastion. Azure valide vos paramètres, puis crée l’hôte. La création et le déploiement de l’hôte et de ses ressources prennent environ 5 minutes.

## <a name="connect"></a><a name="connect"></a>Connexion

Une fois Bastion déployé sur le réseau virtuel, l’écran devient la page de connexion.

1. Entrez le nom d’utilisateur et le mot de passe pour votre machine virtuelle. puis **Connecter**.

   :::image type="content" source="./media/quickstart-host-portal/connect-vm.png" alt-text="Capture d’écran montrant la boîte de dialogue Se connecter à l’aide d’Azure Bastion":::
1. La connexion RDP à cette machine virtuelle s’ouvre directement dans le portail Azure (via le protocole HTML5) à l’aide du port 443 et le service Bastion.

   :::image type="content" source="./media/quickstart-host-portal/connected.png" alt-text="Connexion RDP":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez fini d’utiliser le réseau virtuel et les machines virtuelles, supprimez le groupe de ressources et toutes les ressources qu’il contient :

1. Entrez le nom de votre groupe de ressources dans la zone **Rechercher** en haut du portail et sélectionnez-le dans les résultats de la recherche.

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Entrez votre groupe de ressources dans **TAPER LE NOM DU GROUPE DE RESSOURCES**, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un hôte bastion pour votre réseau virtuel, puis vous vous êtes connecté à une machine virtuelle de manière sécurisée avec Bastion. Vous pouvez à présent passer à l’étape suivante si vous souhaitez vous connecter à un groupe de machines virtuelles identiques.

> [!div class="nextstepaction"]
> [Se connecter à un groupe de machines virtuelles identiques à l’aide d’Azure Bastion](bastion-connect-vm-scale-set.md)
