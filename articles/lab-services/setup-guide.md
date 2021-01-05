---
title: Guide de configuration accélérée de laboratoire pour Azure Lab Services
description: Si vous êtes un créateur de laboratoire, ce guide peut vous aider à configurer rapidement un compte de laboratoire pour votre école.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 07f0d92ebd926616f1318b430bec2de32f753f7c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021726"
---
# <a name="lab-setup-guide"></a>Guide de configuration d’un laboratoire

Dans ce guide, vous apprendrez à créer un laboratoire pour les étudiants de votre école.

Le processus de publication d’un labo pour vos étudiants peut prendre plusieurs heures. Ce délai de configuration dépend du nombre de machines virtuelles que vous souhaitez créer dans votre labo. Accordez-vous au moins une journée pour vous assurer que le laboratoire fonctionne correctement et pour disposer de suffisamment de temps pour publier les machines virtuelles de vos étudiants.

## <a name="understand-the-lab-requirements-of-your-class"></a>Comprendre les conditions requises du labo de votre classe

Avant de configurer un nouveau labo, vous devez vous poser les questions suivantes.

### <a name="what-software-requirements-does-the-class-have"></a>Quelle est la configuration logicielle requise par la classe ?

En fonction des objectifs d’apprentissage de votre classe, vous déterminez le système d’exploitation, les applications et les outils qui doivent être installés sur les machines virtuelles du labo. Pour configurer des machines virtuelles de laboratoire, vous avez trois options :

- **Utiliser une image de la Place de marché Azure** : la Place de marché Azure fournit des centaines d’images que vous pouvez utiliser lors de la création d’un labo. Pour certaines classes, l’une de ces images peut déjà contenir tout ce dont vous avez besoin pour votre classe.

- **Créer une image personnalisée** : Vous pouvez créer votre propre image personnalisée en utilisant une image de la Place de marché Azure comme point de départ. Vous pouvez ensuite la personnaliser en installant des logiciels supplémentaires et en apportant des modifications à la configuration.

- **Utiliser une image personnalisée existante** : vous pouvez réutiliser des images personnalisées que vous avez créées ou qui ont été créées par d’autres administrateurs ou professeurs de votre établissement scolaire. Pour utiliser des images personnalisées, vos administrateurs doivent configurer une galerie d’images partagées.  Une galerie d’images partagées est un référentiel que vous pouvez utiliser pour enregistrer des images personnalisées.

> [!NOTE]
> Les administrateurs sont responsables de l’activation des images de la Place de marché Azure et des images personnalisées afin que vous puissiez les utiliser. Collaborez avec votre service informatique pour être sûr que les images dont vous avez besoin sont activées. Les images personnalisées que vous créez sont automatiquement activées pour une utilisation dans les laboratoires dont vous êtes propriétaire.

### <a name="what-hardware-requirements-does-the-class-have"></a>Quelle est la configuration matérielle requise pour la classe ?

Vous pouvez choisir parmi plusieurs tailles de calcul :

- **Tailles de virtualisations imbriquées** : Vous permet d’accorder aux étudiants l’accès à une machine virtuelle capable d’héberger plusieurs machines virtuelles imbriquées. Par exemple, vous pourriez utiliser cette taille de calcul pour les cours sur les réseaux ou le piratage éthique.

- **Tailles de GPU** : Permet à vos étudiants d’utiliser des types d’applications nécessitant beaucoup de ressources de calcul. Ce choix convient par exemple souvent à l’intelligence artificielle et au Machine Learning.

Pour obtenir des conseils sur la sélection de la taille de machine virtuelle appropriée, consultez les ressources suivantes :
- [Dimensionnement des machines virtuelles](./administrator-guide.md#vm-sizing)
- [Moving from a Physical Lab to Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931) (Passer d’un laboratoire physique à Azure Lab Services)

> [!NOTE]
> Étant donné que la disponibilité de la taille de calcul varie selon la région, la gamme de tailles disponibles pour votre laboratoire peut être inférieure. En règle générale, vous devez sélectionner la plus petite taille de calcul adaptée à vos besoins. Avec Azure Lab Services, vous pouvez configurer un nouveau labo avec une plus grande capacité de calcul ultérieurement, si nécessaire.

### <a name="what-dependencies-does-the-class-have-on-external-azure-or-network-resources"></a>Quelles sont les dépendances de la classe envers les ressources réseau ou Azure externes ?
Vos machines virtuelles de laboratoire peuvent avoir besoin d’accéder à des ressources externes, par exemple à une base de données, un partage de fichiers ou un serveur de licences.  Pour permettre aux machines virtuelles de votre laboratoire d’utiliser des ressources externes, contactez vos administrateurs informatiques.

> [!NOTE]
> Vous devez déterminer si vous pouvez réduire la dépendance de votre laboratoire aux ressources externes en fournissant les ressources réseau directement sur la machine virtuelle. Par exemple, pour éviter d’avoir à lire les données d’une base de données externe, vous pouvez installer la base de données directement sur la machine virtuelle.  

### <a name="how-will-you-control-costs"></a>Comment allez-vous contrôler les coûts ?
Les services Lab utilisent un modèle tarifaire avec paiement à l’utilisation, ce qui signifie que vous payez uniquement pour la durée d’exécution d’une machine virtuelle de labo. Pour contrôler les coûts, utilisez tout ou partie des options suivantes :

- **Planification** : utilisez des planifications pour contrôler automatiquement le moment où les machines virtuelles de votre labo sont démarrées et arrêtées.
- **Quota** : utilisez des quotas pour contrôler le nombre d’heures pendant lesquelles les étudiants ont accès à une machine virtuelle en dehors des heures planifiées.  Lorsqu’un étudiant utilise sa machine virtuelle et atteint son quota, la machine virtuelle s’arrête automatiquement.  L’étudiant ne peut pas redémarrer la machine virtuelle, sauf si vous augmentez le quota.
- **Arrêt automatique** : quand vous activez le paramètre d’arrêt automatique, les machines virtuelles Windows s’arrêtent automatiquement après qu’un étudiant s’est déconnecté d’une session RDP (Remote Desktop Protocol). Ce paramètre est désactivé par défaut.

Pour plus d’informations sur le contrôle des coûts, consultez les ressources suivantes :
- [Estimer les coûts](./cost-management-guide.md#estimate-the-lab-costs)
- [Gérer les coûts](./cost-management-guide.md#manage-costs)

### <a name="how-will-students-save-their-work"></a>Comment les étudiants vont-ils enregistrer leur travail ?
Chaque étudiant reçoit une machine virtuelle pour la durée de vie du laboratoire. Les étudiants peuvent enregistrer leur travail :

- Sur la machine virtuelle.
- Sur un emplacement externe, tel que OneDrive ou GitHub. Il est possible de configurer OneDrive automatiquement pour les étudiants sur leurs machines virtuelles de labo.

> [!NOTE]
> Pour être certain de disposer d’un accès continu à leur travail enregistré en dehors du labo, et après les cours, nous recommandons aux étudiants d’enregistrer leur travail dans un référentiel externe.

### <a name="how-will-students-connect-to-their-vms"></a>Comment les étudiants vont-ils se connecter à leurs machines virtuelles ?
Pour les connexions RDP aux machines virtuelles Windows, nous recommandons aux étudiants d’utiliser le [client Bureau à distance Microsoft](/windows-server/remote/remote-desktop-services/clients/remote-desktop-clients). Le client Bureau à distance prend en charge les appareils Mac, Chromebook et Windows.

Pour les machines virtuelles Linux, les étudiants peuvent utiliser le protocole Secure Shell (SSH) ou RDP. Pour que les étudiants puissent se connecter à l’aide du protocole RDP, vous devez installer et configurer les packages RDP et Graphical User Interface (GUI) nécessaires.

### <a name="will-students-also-use-microsoft-teams"></a>Les étudiants utiliseront-ils aussi Microsoft Teams ?
Azure Lab Services s’intègre à Microsoft Teams afin que les enseignants puissent créer et gérer leurs laboratoires dans Teams.  De même, les étudiants peuvent accéder à leurs laboratoires dans Teams.

Pour plus d’informations, consultez [Azure Lab Services dans Microsoft Teams](./lab-services-within-teams-overview.md).

## <a name="set-up-your-lab"></a>Configurer votre lab

Une fois que vous avez compris les conditions requises pour le labo de votre classe, vous êtes prêt à le configurer. Pour en savoir plus, suivez les liens de cette section. Des instructions sur la configuration des laboratoires sont également fournies dans Teams.

1. **Créez un labo**. Consultez les didacticiels suivants :
    - [Créer un laboratoire de classe](./tutorial-setup-classroom-lab.md#create-a-classroom-lab)
    - [Créer un laboratoire dans Teams](./how-to-get-started-create-lab-within-teams.md)

    > [!NOTE]
    > Si votre classe nécessite une virtualisation imbriquée, consultez l’article [Activer la virtualisation imbriquée](./how-to-enable-nested-virtualization-template-vm.md).

1. **Personnalisez des images et publiez des machines virtuelles de labo**. Pour vous connecter à une machine virtuelle spéciale appelée « modèle de machine virtuelle », consultez les ressources suivantes :
    - [Créer et gérer un modèle de machine virtuelle](./tutorial-setup-classroom-lab.md#publish-the-template-vm)
    - [Utiliser une galerie d’images partagées](./how-to-use-shared-image-gallery.md)

    > [!NOTE]
    > Si vous utilisez Windows, consultez également [Configurer un modèle de machine virtuelle Windows](./how-to-prepare-windows-template.md). Ces instructions incluent les étapes de configuration pour OneDrive et Office afin que vos étudiants puissent les utiliser.

1. **Gérez le pool et la capacité des machines virtuelles**. Vous pouvez facilement effectuer un scale-up ou un scale-down des machines virtuelles en fonction des besoins de votre classe. N’oubliez pas que l’augmentation de la capacité des machines virtuelles peut prendre plusieurs heures, car de nouvelles machines virtuelles sont alors configurées. Voir les articles suivants :
    - [Configurer et gérer un pool de machines virtuelles](./how-to-set-virtual-machine-passwords.md)
    - [Gérer un pool de machines virtuelles dans Lab Services dans Teams](./how-to-manage-vm-pool-within-teams.md)

1. **Ajouter et gérer des utilisateurs de labo**. Pour ajouter des utilisateurs à votre laboratoire, consultez les ressources suivantes :
   - [Ajouter des utilisateurs au labo](./tutorial-setup-classroom-lab.md#add-users-to-the-lab)
   - [Envoyer des invitations aux utilisateurs](./tutorial-setup-classroom-lab.md#send-invitation-emails-to-users)
   - [Gérer les listes d’utilisateurs de Lab Services dans Teams](./how-to-manage-user-lists-within-teams.md)

    Pour plus d’informations sur les types de comptes que les étudiants peuvent utiliser, consultez [Comptes étudiants](./how-to-configure-student-usage.md#student-accounts).
  
1. **Maîtriser les coûts**. Pour définir une planification, établir des quotas et activer l’arrêt automatique, consultez les tutoriels suivants :

   - [Définir un calendrier](./tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)

        > [!NOTE]
        > En fonction du système d’exploitation que vous avez installé, le démarrage d’une machine virtuelle peut prendre plusieurs minutes. Pour vous assurer qu’une machine virtuelle de labo est prête à l’emploi pendant vos heures planifiées, nous vous recommandons de la démarrer 30 minutes à l’avance.

   - [Définir des quotas pour les utilisateurs](./how-to-configure-student-usage.md#set-quotas-for-users) et [définir un quota supplémentaire pour des utilisateurs spécifiques](./how-to-configure-student-usage.md#set-additional-quotas-for-specific-users)
  
   - [Activer l’arrêt automatique lors de la déconnexion](./how-to-enable-shutdown-disconnect.md)

        > [!NOTE]
        > Les planifications et les quotas ne s’appliquent pas à la machine virtuelle du modèle, mais les paramètres d’arrêt automatique s’appliquent. 
        > 
        > Quand vous créez un labo, le modèle de machine virtuelle est créé, mais il ne démarre pas. Vous pouvez démarrer le modèle de machine virtuelle, vous y connecter et installer les logiciels prérequis pour le labo, puis le publier. Quand vous publiez le modèle de machine virtuelle, il s’arrête automatiquement si vous ne l’avez pas arrêté vous-même manuellement. 
        > 
        > Étant donné que les modèles de machines virtuelles engendrent des *coûts* pendant leur exécution, prenez soin de vérifier que le modèle de machine virtuelle est arrêté quand vous n’avez pas besoin de l’exécuter.

    - [Créer et gérer des planifications Azure Lab Services dans Teams](./how-to-create-schedules-within-teams.md) 

1. **Utiliser le tableau de bord**. Pour obtenir des instructions, consultez [Utiliser le tableau de bord de laboratoire de classe](./use-dashboard.md).

    > [!NOTE]
    > Le coût estimé montré dans le tableau de bord est le coût maximal auquel vous pouvez vous attendre pour l’utilisation du labo par les étudiants. Par exemple, les heures de quota inutilisées par vos élèves ne sont *pas* facturées. Les coûts estimés *ne reflètent pas* les frais d’utilisation de la machine virtuelle modèle ou de la galerie d’images partagées ni ceux encourus quand le créateur du labo démarre la machine d’un utilisateur.

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de la gestion de vos laboratoires, consultez les articles suivants :
- [Suivre l’utilisation du laboratoire de classe](tutorial-track-usage.md)  
- [Accéder à un laboratoire de salle de classe](tutorial-connect-virtual-machine-classroom-lab.md)
