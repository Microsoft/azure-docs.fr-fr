---
title: Créer un laboratoire dans Azure DevTest Labs | Microsoft Docs
description: Cet article vous guide dans le processus de création d’un laboratoire à l’aide du portail Azure et d'Azure DevTest Labs.
ms.topic: article
ms.date: 10/12/2020
ms.openlocfilehash: 9fa33a59dd35bfe3469f30f2349f8a08c45bd5e3
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92058339"
---
# <a name="create-a-lab-in-azure-devtest-labs"></a>Créer un laboratoire dans Azure DevTest Labs

Un laboratoire dans Azure DevTest Labs est l’infrastructure qui comprend un groupe de ressources, telles que des machines virtuelles, qui vous permet de mieux gérer ces ressources en spécifiant des limites et des quotas. Cet article vous guide dans le processus de création d’un laboratoire à l’aide du portail Azure.

## <a name="prerequisites"></a>Conditions préalables requises

Pour créer un laboratoire, vous devez avoir :

* Un abonnement Azure. Pour en savoir plus sur les options d’achat d’Azure, consultez [Comment acheter Azure](https://azure.microsoft.com/pricing/purchase-options/) ou [Évaluation d’un mois gratuite](https://azure.microsoft.com/pricing/free-trial/). Pour créer le laboratoire, vous devez être le propriétaire de l’abonnement.

## <a name="get-started-with-azure-devtest-labs-in-minutes"></a>Prenez en main Azure DevTest Labs en quelques minutes.

En cliquant sur le lien suivant, vous serez transféré vers la page du portail Azure qui vous permet de commencer à créer un laboratoire dans Azure DevTest Labs.

[Prenez en main Azure DevTest Labs en quelques minutes.](https://go.microsoft.com/fwlink/?LinkID=627034&clcid=0x409)

## <a name="fill-out-settings-for-your-new-account"></a>Renseigner les paramètres de votre nouveau compte

Dans la page **Créer un compte DevTest Labs**, renseignez les paramètres suivants.

> [!TIP]
> En bas de chaque page, vous trouverez un lien qui vous permet de **télécharger un modèle d’automatisation**.

### <a name="basic-settings"></a>Paramètres de base

Par défaut, l’onglet **Paramètres de base** s’affiche. 

![paramètres de base](./media/devtest-lab-create-lab/basic-settings.png)

Renseignez les valeurs suivantes :

|Nom|Description|
|---|---|
|**Abonnement** | Obligatoire. Sélectionnez **l’abonnement** à associer au laboratoire.|
|**Groupe de ressources**| Obligatoire. Entrez un **nom pour le groupe de ressources** pour le laboratoire. Créez-en un s’il n’en existe pas.|
|**Nom du laboratoire**| Obligatoire. Entrez un **nom** pour le laboratoire.|
|**Lieu**|Obligatoire. Sélectionnez un emplacement dans lequel stocker le laboratoire.|
|**Environnements publics**| Voir [Configurer et utiliser des environnements publics](devtest-lab-configure-use-public-environments.md).

### <a name="auto-shutdown-settings"></a>Paramètres d’arrêt automatique

Basculez sur la page **Arrêt automatique** pour en voir les paramètres. L’arrêt automatique vous permet d’arrêter automatiquement toutes les machines d’un laboratoire à une heure planifiée chaque jour.

![Onglet Arrêt automatique](./media/devtest-lab-create-lab/auto-shutdown.png)

Sur la page, vous pouvez activer la fonctionnalité **Arrêt automatique** et définir les paramètres d’arrêt automatique de toutes les machines virtuelles du laboratoire. La fonction d’arrêt automatique est essentiellement une fonctionnalité économique dans laquelle vous pouvez spécifier les moments auxquels arrêter automatiquement la machine virtuelle. Vous pouvez modifier les paramètres d’arrêt automatique après avoir créé le laboratoire en suivant les étapes décrites dans l’article [Gérer toutes les stratégies d’un laboratoire dans Azure DevTest Labs](./devtest-lab-set-lab-policy.md#set-auto-shutdown).

### <a name="networking"></a>Mise en réseau

Lors de la création d’un laboratoire, un réseau par défaut est créé pour vous. Basculez sur l’onglet**Réseau** pour modifier ou configurer le paramètre comme vous le souhaitez. Par exemple, sélectionnez un réseau virtuel existant.

![Onglet Réseau ](./media/devtest-lab-create-lab/networking.png)

### <a name="tags"></a>Étiquettes

Saisissez les informations **NOM** et **VALEUR** pour les **balises** si vous souhaitez créer un balisage personnalisé à ajouter à toutes les ressources que vous créerez dans le laboratoire. Les balises sont utiles pour vous aider à gérer et à organiser les ressources du laboratoire par catégorie. Pour plus d’informations sur les balises, y compris comment ajouter des balises après avoir créé le laboratoire, consultez [Ajouter des balises à un laboratoire](devtest-lab-add-tag.md).

![Onglet Balises ](./media/devtest-lab-create-lab/tags.png)

### <a name="review-and-create"></a>Examiner et créer

Une fois terminé, sélectionnez **Créer**. Vous pouvez surveiller l’état du processus de création du laboratoire en regardant la zone **Notifications** en haut à droite de la page du portail. 

![Onglet Créer](./media/devtest-lab-create-lab/create-1.png)

## <a name="completed-the-creation"></a>Terminer la création

Une fois l’opération terminée, le bouton **Accéder à la ressource** s’affiche en bas de la page et dans la fenêtre de notification. Vous pouvez également actualiser la page **DevTest Labs** pour afficher le laboratoire nouvellement créé dans la liste des laboratoires.  

![Création du service](./media/devtest-lab-create-lab/create-2.png)

Appuyez sur le bouton **Accéder à la ressource** et vous serez redirigé vers la page d’accueil de votre nouveau compte DevTest Labs.

![Ressource](./media/devtest-lab-create-lab/go-to-resource.png)

Vous pouvez également rechercher **DevTest Labs** dans le portail Azure. Sélectionnez votre nouveau compte dans la liste et accédez à la page d’accueil. 

![Le service a été créé](./media/devtest-lab-create-lab/created.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez créé votre laboratoire, voici quelques étapes à prendre en compte :

* [Sécuriser l’accès à un laboratoire](devtest-lab-add-devtest-user.md)
* [Définir des stratégies de laboratoire](devtest-lab-set-lab-policy.md)
* [Créer un modèle de laboratoire](devtest-lab-create-template.md)
* [Créer des artefacts personnalisés pour vos machines virtuelles](devtest-lab-artifact-author.md)
* [Ajouter une machine virtuelle à un laboratoire](devtest-lab-add-vm.md)

