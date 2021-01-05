---
title: Déployer un cluster managé Service Fabric (version préliminaire) à l’aide d’Azure Resource Manager
description: Découvrez comment créer un cluster managé Service Fabric avec un modèle Azure Resource Manager
ms.topic: quickstart
ms.date: 09/28/2020
ms.custom: references_regions
ms.openlocfilehash: 079e8e3e39bd306332abb43fa2ae1f8cc04e305b
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91410393"
---
# <a name="quickstart-deploy-a-service-fabric-managed-cluster-preview-with-an-azure-resource-manager-template"></a>Démarrage rapide : Déployer un cluster managé Service Fabric (version préliminaire) avec un modèle Azure Resource Manager

Les clusters managés Service Fabric sont une évolution du modèle de ressource de cluster Azure Service Fabric qui simplifie votre expérience de déploiement et de gestion de cluster. Les clusters managés Service Fabric sont une ressource entièrement encapsulée qui vous permet de déployer une seule ressource de cluster Service Fabric au lieu de déployer toutes les ressources sous-jacentes qui composent un cluster Service Fabric. Cet article explique comment déployer un cluster managé Service Fabric à des fins de test dans Azure à l’aide d’un modèle Azure Resource Manager (modèle ARM).

Le cluster SKU de base à trois nœuds déployé dans ce didacticiel est conçu uniquement pour être utilisé à des fins pédagogiques (plutôt que des charges de travail de production). Pour plus d’informations, consultez [Références SKU de cluster managé Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus).

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce guide de démarrage rapide :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce guide de démarrage rapide provient de [Exemples Azure - Modèles de cluster Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Basic-SKU-1-NT).

## <a name="create-a-client-certificate"></a>Créer un certificat client

Les clusters managés Service Fabric utilisent un certificat client comme clé pour le contrôle d’accès. Si vous disposez déjà d’un certificat client que vous souhaitez utiliser pour le contrôle d’accès à votre cluster, vous pouvez ignorer cette étape.

Si vous avez besoin de créer un certificat client, suivez les étapes décrites dans [Définir et récupérer un certificat dans Azure Key Vault](../key-vault/certificates/quick-create-portal.md).

Notez l’empreinte numérique du certificat, car cela sera nécessaire pour déployer le modèle à l’étape suivante.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle.

      [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fservice-fabric-cluster-templates%2Fmaster%2FSF-Managed-Basic-SKU-1-NT%2Fazuredeploy.json)

2. Sélectionner ou entrer les valeurs suivantes

    Pour ce guide de démarrage rapide, fournissez vos propres valeurs pour les paramètres de modèle suivants :

    * **Abonnement**: Sélectionnez un abonnement Azure.
    * **Groupe de ressources** : Sélectionnez **Créer nouveau**. Entrez un nom unique pour le groupe de ressources, tel que *myResourceGroup*, puis choisissez **OK**.
    * **Emplacement** : Sélectionnez un emplacement, comme **eastus2**. Les régions prises en charge pour les clusters managés Service Fabric (préversion) incluent `centraluseuap`, `eastus2euap`, `eastasia`, `northeurope`, `westcentralus` et `eastus2`.
    * **Nom du cluster** : Entrez un nom unique pour votre cluster, par exemple *mysfcluster*.
    * **Nom d’utilisateur administrateur** : Entrez un nom pour l’administrateur à utiliser pour le protocole RDP sur les machines virtuelles sous-jacentes dans le cluster.
    * **Mot de passe administrateur** : Entrez un mot de passe pour l’administrateur à utiliser pour le protocole RDP sur les machines virtuelles sous-jacentes dans le cluster.
    * **Empreinte de certificat du client** : Indiquez l’empreinte du certificat client que vous souhaitez utiliser pour accéder à votre cluster. Si vous n’avez pas de certificat, suivez [Définir et récupérer un certificat](../key-vault/certificates/quick-create-portal.md) pour créer un certificat auto-signé.
    * **Nom du type de nœud** : Entrez un nom unique pour votre type de nœud, par exemple *nt1*.
    * **J’accepte les termes et conditions mentionnés ci-dessus** : Cochez cette case pour accepter. 

3. Sélectionnez **Achat**.

4. Le déploiement de votre cluster Service Fabric managé peut prendre quelques minutes. Attendez que le déploiement se termine correctement avant de passer aux étapes suivantes.

## <a name="validate-the-deployment"></a>Valider le déploiement

### <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Une fois le déploiement terminé, recherchez la valeur Service Fabric Explorer dans la sortie, puis ouvrez l’adresse dans un navigateur web pour voir votre cluster dans Service Fabric Explorer. Lorsque vous êtes invité à entrer un certificat, utilisez le certificat pour lequel l’empreinte numérique du client a été fournie dans le modèle.

> [!NOTE]
> Vous pouvez trouver la sortie du déploiement dans le Portail Azure sous l’onglet Déploiements de groupes de ressources.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources de votre cluster managé Service Fabric. Pour supprimer le groupe de ressources à l’aide du portail :

1. Entrez le nom de votre groupe de ressources dans la zone *Recherche* en haut du portail. Lorsque vous voyez le groupe de ressources utilisé dans ce démarrage rapide dans les résultats de recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Dans le champ **TYPE THE RESOURCE GROUP NAME: (TAPER LE NOM DU GROUPE DE RESSOURCES :)** , tapez le nom du groupe de ressources et sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un cluster Service Fabric managé. Pour en savoir plus sur la mise à l’échelle d’un cluster, consultez :

> [!div class="nextstepaction"]
> [Monter en charge un cluster managé Service Fabric](tutorial-managed-cluster-scale.md)
