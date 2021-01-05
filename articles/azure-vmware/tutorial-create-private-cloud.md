---
title: 'Tutoriel : Déployer un cluster vSphere dans Azure'
description: Découvrez comment déployer un cluster vSphere dans Azure en utilisant Azure VMware Solution
ms.topic: tutorial
ms.date: 11/19/2020
ms.openlocfilehash: 93937f8ca0918494810885f5cb45de571a6e1529
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966307"
---
# <a name="tutorial-deploy-an-azure-vmware-solution-private-cloud-in-azure"></a>Tutoriel : Déployer un cloud privé Azure VMware Solution dans Azure

Azure VMware Solution vous donne la possibilité de déployer un cluster vSphere dans Azure. Le déploiement initial minimal est de trois hôtes. Des hôtes supplémentaires peuvent être ajoutés un à la fois, jusqu’à un maximum de 16 hôtes par cluster. 

Comme Azure VMware Solution ne vous permet pas de gérer votre cloud privé avec votre vCenter local au lancement, vous devez effectuer une configuration supplémentaire. Ces procédures et les prérequis associés sont traités dans ce tutoriel.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un cloud privé Azure VMware Solution
> * Vérifier le cloud privé déployé

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Droits d’administration appropriés et autorisation de créer un cloud privé.
- Vérifiez que vous disposez d’un réseau approprié configuré comme décrit dans [Tutoriel : Check-list du réseau](tutorial-network-checklist.md).

## <a name="register-the-resource-provider"></a>Inscrire le fournisseur de ressources

[!INCLUDE [register-resource-provider-steps](includes/register-resource-provider-steps.md)]


## <a name="create-a-private-cloud"></a>Créer un cloud privé

Vous pouvez créer un cloud privé Azure VMware Solution en utilisant le [portail Azure](#azure-portal) ou [Azure CLI](#azure-cli).

### <a name="azure-portal"></a>Portail Azure

[!INCLUDE [create-avs-private-cloud-azure-portal](includes/create-private-cloud-azure-portal-steps.md)]

### <a name="azure-cli"></a>Azure CLI

Pour créer un cloud privé Azure VMware Solution, vous pouvez utiliser l’interface Azure CLI à l’aide d’Azure Cloud Shell au lieu du portail Azure.  Pour obtenir la liste des commandes que vous pouvez utiliser avec Azure VMware Solution, consultez [Commandes Azure VMware](/cli/azure/ext/vmware/vmware).

#### <a name="open-azure-cloud-shell"></a>Ouvrir Azure Cloud Shell

Sélectionnez **Essayer** dans le coin supérieur droit d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur **Entrée** pour les exécuter.

#### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande `[az group create](/cli/azure/group)`. Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive

az group create --name myResourceGroup --location eastus
```

#### <a name="create-a-private-cloud"></a>Créer un cloud privé

Spécifiez un nom pour le groupe de ressources et le cloud privé, un emplacement et la taille du cluster.

| Propriété  | Description  |
| --------- | ------------ |
| **-g** (nom de groupe de ressources)     | Nom du groupe de ressources pour vos ressources de cloud privé.        |
| **-n** (nom du cloud privé)     | Nom de votre cloud privé Azure VMware Solution.        |
| **--location**     | Emplacement utilisé pour votre cloud privé.         |
| **--cluster-size**     | Taille du cluster La valeur minimale est 3.         |
| **--network-block**     | Bloc réseau d’adresses IP CIDR à utiliser pour votre cloud privé. Le bloc d’adresses ne doit pas chevaucher ceux utilisés dans d’autres réseaux virtuels se trouvant dans votre abonnement et des réseaux locaux.        |
| **--sku** | Valeur de référence SKU : AV36 |

```azurecli-interactive
az vmware private-cloud create -g myResourceGroup -n myPrivateCloudName --location eastus --cluster-size 3 --network-block xx.xx.xx.xx/22 --sku AV36
```

## <a name="delete-an-azure-vmware-solution-private-cloud"></a>Supprimer un cloud privé Azure VMware Solution

Si vous avez un cloud privé Azure VMware Solution dont vous n’avez plus besoin, vous pouvez le supprimer. Un cloud privé Azure VMware Solution comprend un domaine de réseau isolé, un ou plusieurs clusters vSphere provisionnés sur des hôtes serveur dédiés et plusieurs machines virtuelles. Quand un cloud privé est supprimé, toutes les machines virtuelles, leurs données et les clusters sont supprimés. Les hôtes nus dédiés sont réinitialisés sans problème et retournés au pool libre. Le domaine de réseau provisionné pour le client est supprimé.  

> [!CAUTION]
> La suppression du cloud privé est une opération irréversible. Une fois le cloud privé supprimé, les données ne peuvent pas être récupérées, car cela met fin à l’ensemble des charges de travail et composants en cours d’exécution, et détruit toutes les données et tous les paramètres de configuration du cloud privé, notamment les adresses IP publiques.

### <a name="prerequisites"></a>Prérequis

Une fois qu’un cloud privé est supprimé, il n’existe aucun moyen de récupérer les machines virtuelles et leurs données. Les données de machines virtuelles seront nécessaires ultérieurement, mais l’administrateur doit d’abord sauvegarder toutes les données avant de supprimer le cloud privé.

### <a name="steps-to-delete-an-azure-vmware-solution-private-cloud"></a>Étapes à suivre pour supprimer un cloud privé Azure VMware Solution

1. Accédez à la page Azure VMware Solutions à partir du portail Azure.

2. Sélectionnez le cloud privé à supprimer.
 
3. Entrez le nom du cloud privé, puis sélectionnez **Oui**. Le processus de suppression se termine en quelques heures.  

## <a name="azure-vmware-commands"></a>Commandes Azure VMware

Pour obtenir la liste des commandes que vous pouvez utiliser avec Azure VMware Solution, consultez [Commandes Azure VMware](/cli/azure/ext/vmware/vmware).

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un cloud privé Azure VMware Solution
> * Vérifier le cloud privé déployé
> * Supprimer un cloud privé Azure VMware Solution

Passez au didacticiel suivant pour découvrir comment créer un serveur de rebond. Vous utilisez le serveur de rebond pour vous connecter à votre environnement afin de pouvoir gérer votre cloud privé en local.


> [!div class="nextstepaction"]
> [Accéder à un cloud privé Azure VMware Solution](tutorial-access-private-cloud.md)