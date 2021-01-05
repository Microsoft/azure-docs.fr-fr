---
title: Guide de configuration accélérée de compte de laboratoire pour Azure Lab Services
description: Ce guide aide les administrateurs à configurer rapidement un compte de laboratoire pour une utilisation au sein de leur école.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: aa3e7b586546b3d87f5c6029b284eeb1402ed171
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659860"
---
# <a name="lab-account-setup-guide"></a>Guide de configuration de compte de laboratoire
Pour configurer votre environnement Azure Lab Services, les administrateurs doivent d’abord configurer un **compte de laboratoire** au sein de votre abonnement Azure. Un compte de laboratoire est un conteneur pour vos laboratoires. Cette opération ne prend que quelques minutes.

Ce guide comprend trois sections :
-  La première section se concentre sur les prérequis qui doivent être remplis *avant* de configurer votre compte de laboratoire.
-  La deuxième section fournit des conseils sur la planification des paramètres de votre compte de laboratoire.
-  La troisième section fournit des instructions pas à pas pour la configuration d’un compte de laboratoire.

## <a name="prerequisites-for-setting-up-your-lab-account"></a>Prérequis pour la configuration de votre compte de laboratoire
Cette section décrit les prérequis que vous devez suivre pour pouvoir configurer un compte de laboratoire.

### <a name="obtain-an-azure-subscription"></a>Obtenir un abonnement Azure
Pour créer un compte de laboratoire, vous devez avoir accès à un abonnement Azure configuré pour votre établissement scolaire. Votre établissement scolaire peut avoir un ou plusieurs abonnements. Un abonnement permet de gérer la facturation et la sécurité de l’ensemble de vos ressources et services Azure qui y sont utilisés, notamment les comptes de laboratoire.  Les abonnements Azure sont généralement gérés par votre service informatique.  Pour plus d’informations, consultez les rubriques suivantes :
 - [Guide de l’administrateur - Abonnement](./administrator-guide.md#subscription)

### <a name="estimate-the-number-of-vms-and-vm-sizes-that-you-need"></a>Estimer le nombre de machines virtuelles et les tailles de machines virtuelles dont vous avez besoin
Vous devez estimer le nombre de machines virtuelles et les [tailles de machines virtuelles](./administrator-guide.md#vm-sizing) dont votre établissement scolaire a besoin.  Lisez le billet de blog suivant pour obtenir des conseils sur la façon de structurer votre laboratoires\images.  Ce billet de blog vous aide également à déterminer le nombre de machines virtuelles et les tailles de machines virtuelles dont vous avez besoin :
- [Moving from a Physical Lab to Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931) (Passer d’un laboratoire physique à Azure Lab Services)

Consultez également cet article qui donne des conseils supplémentaires sur la façon de structurer les laboratoires :
- [Guide de l’administrateur - Laboratoire](./administrator-guide.md)

### <a name="understand-subscription-vm-limits-and-regional-vm-capacity"></a>Comprendre les limites de l’abonnement en termes de machines virtuelles et la capacité régionale en termes de machines virtuelles
Une fois que vous avez une estimation du nombre de machines virtuelles et des tailles de machines virtuelles pour vos laboratoires, vous devez procéder comme suit :

- Assurez-vous que la limite de capacité de votre abonnement Azure autorise le nombre de machines virtuelles et la taille de machine virtuelle que vous prévoyez d’utiliser dans vos laboratoires.

- Créez votre compte de laboratoire dans une région disposant de suffisamment de capacité de machine virtuelle.

Pour en savoir plus, lisez le billet de blog suivant : [VM Subscription Limits and Regional Capacity](https://techcommunity.microsoft.com/t5/azure-lab-services/vm-subscription-limits-and-regional-capacity/ba-p/1845553) (Limites de l’abonnement en termes de machines virtuelle et capacité régionale).

### <a name="decide-how-many-lab-accounts-to-create"></a>Déterminer le nombre de comptes de laboratoire à créer

Pour commencer rapidement, créez un seul compte de laboratoire dans son propre groupe de ressources.  Plus tard, vous pourrez créer des comptes de laboratoire supplémentaires (et des groupes de ressources) en fonction des besoins. Par exemple, vous pouvez avoir un compte de laboratoire et un groupe de ressources par service afin de clairement séparer les coûts.  Lisez les articles suivants pour en savoir plus sur les comptes de laboratoire, les groupes de ressources et la séparation des coûts :
- [Guide de l’administrateur - Groupe de ressources](./administrator-guide.md#resource-group)
- [Guide de l’administrateur - Compte de laboratoire](./administrator-guide.md#lab-account) 
- [Gestion des coûts pour Azure Lab Services](./cost-management-guide.md)

## <a name="planning-your-lab-accounts-settings"></a>Planification des paramètres de votre compte de laboratoire

Pour planifier les paramètres de votre compte de laboratoire, vous devez prendre en compte les questions ci-dessous.

### <a name="who-should-be-owners-and-contributors-of-the-lab-account"></a>Qui doivent être les propriétaires et les contributeurs du compte de laboratoire ?

   Les administrateurs informatiques de votre établissement scolaire sont généralement les Propriétaires et les Contributeurs d’un compte de laboratoire. Ils sont responsables de la gestion des stratégies qui s’appliquent à tous les laboratoires contenus dans le compte de laboratoire. La personne qui crée le compte de laboratoire en est automatiquement un Propriétaire. Vous pouvez ajouter d’autres Propriétaires et Contributeurs à partir du locataire Azure Active Directory (AD) associé à votre abonnement. Pour plus d’informations sur les rôles Propriétaire et Contributeur du compte de laboratoire, consultez :
   -  [Guide de l’administrateur - Gérer les identités](./administrator-guide.md#manage-identity).

   [!INCLUDE [Select a tenant](./includes/multi-tenant-support.md)]

   Les utilisateurs du labo ne voient qu’une seule liste des machines virtuelles auxquelles ils ont accès dans les locataires dans Azure Lab Services.

### <a name="who-will-be-allowed-to-create-labs"></a>Qui sera autorisé à créer les laboratoires ?

   Vous pouvez décider que votre service informatique et des enseignants peuvent créer des laboratoires. Lorsqu’un utilisateur crée un laboratoire, il est automatiquement désigné en tant que Propriétaire du laboratoire.  Pour créer des laboratoires, les utilisateurs (généralement issus du locataire Azure AD associé à votre abonnement) doivent être affectés au rôle Créateur de laboratoire dans le compte de laboratoire.  Pour plus d’informations sur le rôle Créateur de laboratoire, consultez :
   -  [Guide de l’administrateur - Gérer les identités](./administrator-guide.md#manage-identity)

### <a name="who-will-be-allowed-to-own-and-manage-labs"></a>Qui sera autorisé à posséder et à gérer les laboratoires ?

   Vous pouvez également décider que votre service informatique et les enseignants possèdent/gèrent les laboratoires *sans* leur donner la possibilité de créer des laboratoires.  Dans ce cas, les utilisateurs issus du locataire Azure AD de votre abonnement se voient attribuer le rôle Propriétaire ou Contributeur pour les laboratoires existants.  Pour plus d’informations sur les rôles Propriétaire et Contributeur d’un laboratoire, consultez :
   - [Guide de l’administrateur - Gérer les identités](./administrator-guide.md#manage-identity)

### <a name="do-you-want-to-save-images-that-can-be-shared-across-labs"></a>Souhaitez-vous enregistrer des images qui peuvent être partagées entre les laboratoires ?
Une galerie d’images partagées est un référentiel que vous pouvez utiliser pour enregistrer et partager des images. Pour les classes qui ont besoin de la même image, les créateurs de laboratoire peuvent créer l’image, puis l’exporter vers Shared Image Gallery.  Une fois qu’une image est exportée vers Shared Image Gallery, elle peut être utilisée pour créer de nouveaux laboratoires.

Vous pouvez également créer vos images dans votre environnement physique, puis les importer dans Shared Image Gallery.  Pour plus d’informations sur ce processus, consultez le billet de blog suivant : 
- [Import Custom Image to Shared Image Gallery](https://techcommunity.microsoft.com/t5/azure-lab-services/import-custom-image-to-shared-image-gallery/ba-p/1777353) (Importer une image personnalisée dans Shared Image Gallery)

Si vous décidez d’utiliser une galerie Shared Image Gallery, vous devrez en créer ou en attacher une à votre compte de laboratoire. Vous pouvez aussi différer cette décision, car la galerie peut être attachée à un compte de laboratoire à tout moment.  Pour plus d’informations sur Shared Image Gallery, consultez :
- [Guide de l’administrateur - Shared Image Gallery](./administrator-guide.md#shared-image-gallery)
- [Guide de l’administrateur - Tarification de Shared Image Gallery](./administrator-guide.md#shared-image-gallery-2)

### <a name="which-images-in-azure-marketplace-will-your-labs-use"></a>Quelles images de la Place de marché Azure vos laboratoires utiliseront-ils ?
La Place de marché Azure fournit des centaines d’images que vous pouvez activer afin que les créateurs de laboratoires puissent les utiliser pour créer leur laboratoire. Certaines images peuvent inclure tout ce dont un laboratoire a déjà besoin. Dans d’autres cas, vous pouvez utiliser une image comme point de départ, puis le créateur de laboratoire peut la personnaliser en installant des applications ou outils supplémentaires.

Si vous ne savez pas de quelles images vous avez besoin, vous pouvez revenir ultérieurement pour les activer. En outre, la meilleure façon de voir quelles images sont disponibles consiste à créer d’abord un compte de laboratoire. Cela vous donne l’accès, afin que vous puissiez consulter la liste des images disponibles et leur contenu.  Pour plus d’informations sur les images de la Place de marché, consultez :
- [Spécifier les images de la Place de marché accessibles aux créateurs lab](./specify-marketplace-images.md)
  
### <a name="do-the-labs-vms-need-to-have-access-to-other-azure-or-on-premises-resources"></a>Les machines virtuelles du laboratoire doivent-elles avoir accès à d’autres ressources Azure ou locales ?
Quand vous configurez un compte de laboratoire, vous pouvez également l’appairer avec un réseau virtuel (VNet).  N’oubliez pas que votre réseau virtuel et votre compte de laboratoire doivent se trouver dans la même région.  Pour déterminer si vous devez appairer avec un réseau virtuel, prenez en compte les scénarios suivants :

- **Accès à un serveur de licences**
  
   Lorsque vous utilisez des images de la Place de marché Azure, le coût de la licence du système d’exploitation est intégré à la tarification de Lab Services. Toutefois, vous n’avez pas besoin de fournir des licences pour le système d’exploitation lui-même. Toutefois, vous devrez fournir une licence appropriée pour les logiciels et applications supplémentaires installés.  Pour accéder à un serveur de licences :
   - Vous pouvez choisir de vous connecter à un serveur de licences local.  La connexion à un serveur de licences local nécessite une configuration supplémentaire.
   - Une autre option, plus rapide à configurer, consiste à créer un serveur de licences que vous hébergez sur une machine virtuelle Azure.  La machine virtuelle Azure se trouve dans un réseau virtuel que vous appairez à votre compte de laboratoire.

- **Accès à d’autres ressources locales, telles qu’un partage de fichiers ou une base de données**

   Vous créez un réseau virtuel pour fournir l’accès aux ressources locales, généralement à l’aide d’une passerelle de réseau virtuel de site à site. La configuration de ce type d’environnement prendra davantage de temps.

- **Accès à d’autres ressources Azure qui se trouvent en dehors d’un réseau virtuel**

   Si vous avez besoin d’accéder à des ressources Azure qui ne sont *pas* sécurisées au sein d’un réseau virtuel, vous pouvez accéder à ces ressources via l’Internet public sans effectuer de peering.

Pour plus d’informations sur les réseaux virtuels, consultez :
- [Notions de base de l’architecture - Réseau virtuel](./classroom-labs-fundamentals.md#virtual-network)
- [Comment se connecter à un réseau virtuel](./how-to-connect-peer-virtual-network.md)
- [Guide pratique pour créer un labo avec une ressource partagée dans Azure Lab Services](./how-to-create-a-lab-with-shared-resource.md)

## <a name="set-up-your-lab-account"></a>Configurer votre compte de laboratoire

Une fois que vous avez terminé la planification, vous êtes prêt à configurer votre compte de laboratoire.  Ces mêmes étapes s’appliquent à la configuration d’un laboratoire [Azure Lab Services avec Teams](./lab-services-within-teams-overview.md).

1. **Créez votre compte de laboratoire.** Pour obtenir des instructions, reportez-vous au tutoriel sur la [création d’un compte de laboratoire](./tutorial-setup-lab-account.md#create-a-lab-account).
   
    Pour plus d’informations sur la dénomination, consultez l’article suivant :

   - [Recommandations en matière d’affectation de noms pour ressources](./administrator-guide.md#naming)

2. **Ajoutez des utilisateurs au rôle Créateur de laboratoire.** Pour obtenir des instructions, consultez [Ajout d’utilisateurs au rôle Créateur de laboratoire](./tutorial-setup-lab-account.md#add-a-user-to-the-lab-creator-role).


3. **Connectez-vous à un réseau virtuel de pair.** Pour plus d’instructions, consultez [Connecter le réseau de votre labo avec un réseau virtuel pair](./how-to-connect-peer-virtual-network.md).

   Vous devrez peut-être également consulter les instructions sur [la configuration de la plage d’adresses des machines virtuelles de laboratoire](./how-to-configure-lab-accounts.md).

4. **Activez et vérifiez les images.** Pour plus d’instructions, consultez [Activation d’images de la Place de marché Azure pour les créateurs de laboratoires](./specify-marketplace-images.md).

   Pour examiner le contenu de chaque image de la Place de marché Azure, sélectionnez le nom de l’image. Par exemple, reportez-vous à la capture d’écran suivante qui montre les détails de l’image Ubuntu Data Science VM :

   ![Capture d’écran de révision des images de la Place de marché Azure](./media/setup-guide/review-marketplace-images.png)

   Si une galerie d’images partagées est attachée à votre compte de laboratoire et que vous souhaitez activer le partage d’images personnalisées par les créateurs de laboratoire, effectuez les mêmes étapes que celles indiquées dans la capture d’écran suivante :

   ![Capture d’écran de l’activation des images personnalisées dans la galerie d’images partagées](./media/setup-guide/enable-sig-custom-images.png)

## <a name="next-steps"></a>Étapes suivantes

Étapes suivantes qui sont communes à la configuration d’un environnement de laboratoire :

- [Gérer les comptes de laboratoire](how-to-manage-lab-accounts.md)
- [Guide de configuration d’un laboratoire](setup-guide.md)