---
title: Déployer Avere vFXT pour Azure
description: Découvrez comment utiliser l’assistant de déploiement disponible sur la place de marché Azure pour déployer un cluster avec Avere vFXT pour Azure.
author: ekpgh
ms.service: avere-vfxt
ms.topic: how-to
ms.date: 01/13/2020
ms.author: rohogue
ms.openlocfilehash: 110fc658ffc3b72de91bbf1c9562f3efeecaf55b
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342414"
---
# <a name="deploy-the-vfxt-cluster"></a>Déployer le cluster vFXT

Cette procédure explique comment utiliser l'Assistant de déploiement disponible sur la Place de marché Microsoft Azure. L'Assistant déploie automatiquement le cluster à l'aide d'un modèle Azure Resource Manager. Une fois que vous avez entré les paramètres dans le formulaire et que vous avez cliqué sur **Créer**, Azure effectue automatiquement les tâches suivantes :

* Crée le contrôleur de cluster ; il s'agit d'une machine virtuelle de base contenant le logiciel nécessaire au déploiement et à la gestion du cluster.
* Configurez l'infrastructure du réseau virtuel et du groupe de ressources, notamment en créant de nouveaux éléments.
* Crée les machines virtuelles du nœud de cluster et les configurez en tant que cluster Avere.
* Si nécessaire, crée un conteneur blob Azure et le configure en tant que système de stockage principal du cluster.

Après avoir suivi les instructions de ce document, vous disposerez d’un réseau virtuel, d’un sous-réseau, d’un contrôleur de cluster et d’un cluster vFXT, comme le montre le diagramme suivant. Ce diagramme montre le système de stockage principal Azure Blob facultatif, qui inclut un nouveau conteneur de stockage blob (dans un nouveau compte de stockage, qui n’est pas affiché ici) et un point de terminaison de service pour le stockage Microsoft dans le sous-réseau.

![Diagramme constitué de trois rectangle concentriques englobant les composants du cluster Avere. Le rectangle externe est étiqueté « Groupe de ressources » et contient un hexagone étiqueté « Stockage blob (facultatif) ». Le rectangle suivant est étiqueté « réseau virtuel : 10.0.0.0/16 » et ne contient pas tous les composants uniques. Le rectangle intérieur est étiqueté « Sous-réseau : 10.0.0.0/24 » et contient une machine virtuelle étiquetée « Cluster contrôleur », une pile de trois machines virtuelles étiquetée « Nœuds vFXT (cluster vFXT) » et un hexagone étiqueté « Point de terminaison de service ». Le diagramme contient aussi une flèche reliant le point de terminaison de service (qui est à l’intérieur du sous-réseau) et le stockage d’objets blob (situé à l’extérieur du sous-réseau et du réseau virtuel, dans le groupe de ressources). La flèche traverse le sous-réseau et les limites du réseau virtuel.](media/avere-vfxt-deployment.png)

Avant d'utiliser le modèle de création, vérifiez que les conditions préalables suivantes sont réunies :  

* [Nouvel abonnement](avere-vfxt-prereqs.md#create-a-new-subscription)
* [Autorisations du propriétaire de l’abonnement](avere-vfxt-prereqs.md#configure-subscription-owner-permissions)
* [Quota pour le cluster vFXT](avere-vfxt-prereqs.md#quota-for-the-vfxt-cluster)
* [Point de terminaison de service de stockage (si nécessaire)](avere-vfxt-prereqs.md#create-a-storage-service-endpoint-in-your-virtual-network-if-needed) : nécessaire pour les déploiements qui utilisent un réseau virtuel existant et créent un stockage d’objets blob

Pour plus d’informations sur la planification du déploiement du cluster et les étapes à suivre, consultez [Planifier votre système Avere vFXT](avere-vfxt-deploy-plan.md) et [Vue d’ensemble du déploiement](avere-vfxt-deploy-overview.md).

## <a name="create-the-avere-vfxt-for-azure"></a>Créer le déploiement Avere vFXT pour Azure

Sur le portail Azure, accédez au modèle de création en recherchant Avere et en sélectionnant « Modèle Avere vFXT pour Azure ARM ».

![Fenêtre du navigateur affichant le portail Azure avec « Nouveau > Place de marché > Tout ». Sur la page Tout, le champ de recherche contient le terme « avere » et le deuxième résultat, « Modèle Avere vFXT pour Azure ARM », est surligné en rouge pour le mettre en évidence.](media/avere-vfxt-template-choose.png)

Après avoir lu les détails de la page Modèle Avere vFXT pour Azure ARM, cliquez sur son bouton **Créer** pour commencer.

![Place de marché Azure sur laquelle est affichée la première page de du modèle de déploiement](media/avere-vfxt-deploy-first.png)

Le modèle est divisé en quatre étapes : deux pages de collecte d'informations, ainsi que des étapes de validation et de confirmation.

* La première page rassemble les paramètres de la machine virtuelle du contrôleur de cluster.
* La deuxième page collecte les paramètres de création du cluster et des ressources supplémentaires comme les sous-réseaux et le stockage.
* La troisième page récapitule vos choix et valide la configuration.
* La quatrième page contient les conditions générales du logiciel et vous permet de lancer le processus de création du cluster.

## <a name="page-one-parameters---cluster-controller-information"></a>Paramètres de la première page - Informations sur le contrôleur de cluster

La première page du modèle de déploiement se concentre sur le contrôleur de cluster.

![Première page du modèle de déploiement](media/avere-vfxt-deploy-1.png)

Renseignez les informations suivantes :

* **Nom du contrôleur de cluster** : définissez le nom de la machine virtuelle du contrôleur de cluster.

* **Nom d'utilisateur du contrôleur** : définissez le nom d'utilisateur racine de la machine virtuelle du contrôleur de cluster.

* **Type d'authentification** : choisissez un mot de passe ou une authentification par clé publique SSH pour vous connecter au contrôleur. La méthode d'authentification par clé publique SSH est recommandée. Si vous avez besoin d'aide, consultez [Comment créer et utiliser des clés SSH](../virtual-machines/linux/ssh-from-windows.md).

* **Mot de passe** ou **Clé publique SSH** : selon le type d'authentification que vous avez sélectionné, vous devez fournir une clé publique RSA ou un mot de passe dans les champs suivants. Ces informations d'identification sont utilisées avec le nom d'utilisateur fourni précédemment.

* **Abonnement** : sélectionnez l'abonnement correspondant au déploiement Avere vFXT.

* **Groupe de ressources** : sélectionnez un groupe de ressources vide existant pour le déploiement Avere vFXT, ou cliquez sur « Créer nouveau » et entrez un nouveau nom de groupe de ressources.

* **Emplacement** : sélectionnez l'emplacement Azure de votre cluster et de vos ressources.

Cliquez sur **OK** lorsque vous avez terminé.

> [!NOTE]
> Si vous souhaitez que le contrôleur de cluster dispose d'une adresse IP publique, créez un nouveau réseau virtuel pour le cluster au lieu de sélectionner un réseau existant. Ce paramètre se trouve sur la deuxième page.

## <a name="page-two-parameters---vfxt-cluster-information"></a>Paramètres de la deuxième page - Informations sur le cluster vFXT

La deuxième page du modèle de déploiement vous permet de définir la taille du cluster, le type de nœud, la taille du cache et les paramètres de stockage, entre autres paramètres.

![Deuxième page du modèle de déploiement](media/avere-vfxt-deploy-2.png)

* **Nombre de nœuds du cluster Avere vFXT** : choisissez le nombre de nœuds dans le cluster. Le minimum est de trois nœuds et le maximum de douze.

* **Mot de passe d'administration du cluster** : créez le mot de passe pour l'administration du cluster. Ce mot de passe est utilisé avec le nom d'utilisateur ```admin``` pour vous connecter au panneau de configuration du cluster à partir duquel vous pouvez surveiller le cluster et configurer ses paramètres.

* **Nom du cluster Avere vFXT** : attribuez un nom unique au cluster.

* **Taille** : cette section affiche le type de machine virtuelle qui sera utilisé pour les nœuds de cluster. Bien qu’une seule option est recommandée, le lien **Modifier la taille** ouvre un tableau contenant des informations détaillées sur ce type d’instance et un lien vers un calculateur de coût.

* **Taille du cache par nœud** : le cache du cluster est réparti sur les nœuds du cluster. La taille totale du cache de votre cluster Avere vFXT correspondra donc à la taille multipliée par le nombre de nœuds.

  Configuration recommandée : Utilisez 4 To par nœud pour les nœuds Standard_E32s_v3.

* **Réseau virtuel** : définissez un nouveau réseau virtuel pour héberger le cluster, ou sélectionnez un réseau virtuel existant qui remplit les conditions préalables décrites dans [Planifier votre système Avere vFXT ](avere-vfxt-deploy-plan.md#subscription-resource-group-and-network-infrastructure).

  > [!NOTE]
  > Si vous créez un réseau virtuel, le contrôleur de cluster dispose d'une adresse IP publique pour vous permettre d'accéder au nouveau réseau privé. Si vous choisissez un réseau virtuel existant, le contrôleur de cluster est configuré sans adresse IP publique.
  >
  > Une adresse IP publiquement visible sur le contrôleur de cluster facilite l'accès au cluster vFXT, mais pose un léger risque pour la sécurité.
  >* Une adresse IP publique vous permet d’utiliser le contrôleur de cluster en tant qu’hôte de saut (« jump host »). Vous pouvez ainsi vous connecter au cluster Avere vFXT à partir d’un emplacement externe au réseau virtuel.
  >* Si vous n’avez aucune adresse IP publique sur le contrôleur, vous devez utiliser un autre hôte de saut, une connexion VPN ou ExpressRoute pour accéder au cluster. Par exemple, utilisez un réseau virtuel existant sur lequel une connexion VPN a déjà été configurée.
  >* Si vous créez un contrôleur avec une adresse IP publique, vous devez protéger la machine virtuelle du contrôleur avec un groupe de sécurité réseau. Par défaut, le déploiement Avere vFXT pour Azure crée un groupe de sécurité réseau qui limite l'accès entrant au seul port 22 pour les contrôleurs dotés d'adresses IP publiques. Vous pouvez renforcer la protection du système en verrouillant l'accès à votre plage d'adresses IP sources, c'est-à-dire autoriser uniquement les connexions des machines que vous avez l'intention d'utiliser pour l'accès au cluster.

  Un nouveau réseau virtuel est également configuré avec un point de terminaison de service de stockage pour Stockage Blob Azure et un contrôle d’accès réseau verrouillé autorisant uniquement les adresses IP provenant du sous-réseau de cluster.

* **Sous-réseau** : choisissez un sous-réseau ou créez-en un nouveau.

* **Créer et utiliser le stockage d'objets blob** : choisissez **true** afin de créer un nouveau conteneur d'objets Blob Azure et configurez-le en tant que stockage principal pour le nouveau cluster Avere vFXT. Cette option crée également un compte de stockage dans le même groupe de ressources que le cluster, et crée un point de terminaison du service de stockage de Microsoft dans le sous-réseau du cluster.
  
  Si vous fournissez un réseau virtuel, il doit avoir un point de terminaison de service de stockage avant de créer le cluster. (Pour plus d’informations, consultez [Planifier votre système Avere vFXT](avere-vfxt-deploy-plan.md).)

  Définissez ce champ sur **false** si vous ne souhaitez pas créer de nouveau conteneur. Dans ce cas, vous devez joindre et configurer le stockage après la création du cluster. Pour obtenir des instructions, consultez [Configurer le stockage](avere-vfxt-add-storage.md).

* **Compte de stockage** : si vous créez un conteneur blob Azure, entrez un nom pour le nouveau compte de stockage.

## <a name="validation-and-purchase"></a>Validation et achat

La troisième page récapitule la configuration et valide les paramètres. Une fois la validation réussie, vérifiez le résumé et cliquez sur le bouton **OK**.

> [!TIP]
> Vous pouvez enregistrer les paramètres de création de ce cluster en cliquant sur le lien **Télécharger le modèle et les paramètres** en regard du bouton **OK**. Ces informations peuvent être utiles si vous devez créer un cluster similaire ultérieurement, par exemple pour créer un cluster de remplacement dans un scénario de récupération d’urgence. (Pour en savoir plus, lisez [Guide de récupération d’urgence](disaster-recovery.md).)

![Troisième page du modèle de déploiement - Validation](media/avere-vfxt-deploy-3.png)

La page quatre indique les conditions d’utilisation et donne des liens vers les informations de confidentialité et de tarification.

Entrez les informations de contact manquantes, puis cliquez sur le bouton **Créer** pour accepter les termes du contrat et créer l’instance Avere vFXT pour un cluster Azure.

![Quatrième page du modèle de déploiement - Conditions générales, bouton Créer](media/avere-vfxt-deploy-4.png)

Le déploiement du cluster prend 15 à 20 minutes.

## <a name="gather-template-output"></a>Collecter la sortie du modèle

Après avoir créé le cluster, le modèle Avere vFXT fournit des informations importantes sur le nouveau cluster.

> [!TIP]
> Veillez à copier **l'adresse IP de gestion** à partir de la sortie du modèle. Vous aurez besoin de cette adresse pour administrer le cluster.

Pour trouver ces informations :

1. Accédez au groupe de ressources de votre contrôleur de cluster.

1. À gauche, cliquez sur **Déploiements**, puis sur **microsoft-avere.vfxt-template**.

   ![Page du portail Groupe de ressources comprenant les déploiements sélectionnés à gauche et microsoft-avere.vfxt-template dans une table sous Nom du déploiement](media/avere-vfxt-outputs-deployments.png)

1. À gauche, cliquez sur **Sorties**. Copiez les valeurs dans chacun des champs.

   ![page affichant les valeurs SSHSTRING, RESOURCE_GROUP, LOCATION, NETWORK_RESOURCE_GROUP, NETWORK, SUBNET, SUBNET_ID, VSERVER_IPs et MGMT_IP dans des champs, à droite des étiquettes](media/avere-vfxt-outputs-values.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que le cluster est en cours d’exécution et que vous connaissez son adresse IP de gestion, [connectez-vous à l’outil de configuration du cluster](avere-vfxt-cluster-gui.md).

Utilisez l’interface de configuration pour personnaliser votre cluster, y compris les tâches de configuration suivantes :

* [Activer la prise en charge](avere-vfxt-enable-support.md)
* [Ajouter un stockage](avere-vfxt-add-storage.md) (si nécessaire)