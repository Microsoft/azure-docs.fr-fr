---
title: Utiliser des labos pour des formations - Azure Lab Services
description: Cet article explique comment utiliser Azure DevTest Labs afin de créer des laboratoires sur Azure pour les scénarios d'apprentissage.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: f9eb3d44d470a978e676f6268cd693b283dd2703
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433570"
---
# <a name="use-labs-for-trainings"></a>Utiliser des labos pour des formations
Les services Azure Labs permettent aux formateurs (enseignants, professeurs, animateurs ou assistants, etc.) de créer rapidement et facilement un laboratoire en ligne afin d’approvisionner des environnements d’apprentissage préconfigurés pour les stagiaires. Chaque candidat doit être capable d’utiliser des environnements identiques et isolés pour la formation. Vous pouvez appliquer des stratégies pour vous assurer que les environnements de formation sont à la disposition des participants uniquement lorsque ces derniers en ont besoin et qu’ils contiennent suffisamment de ressources (telles que des machines virtuelles) pour la formation. 

![Laboratoire de classe](./media/classroom-labs-scenarios/classroom.png)

Les labos répondent aux conditions suivantes, indispensables pour diriger une formation dans un environnement virtuel : 

- Les participants peuvent configurer rapidement leur environnement de formation.
- Toutes les machines de formation sont identiques.
- Les participants ne voient pas les machines virtuelles créées par les autres participants.
- Contrôlez les coûts en vous assurant que les participants ne peuvent pas obtenir plus de machines virtuelles que nécessaire et qu’ils arrêtent les machines virtuelles lorsqu’ils ne les utilisent pas.
- Partagez facilement le laboratoire de formation avec chaque participant.
- Réutilisez le laboratoire de formation à l’infini.

Cet article présente les différentes fonctionnalités d’Azure Lab Services qui peuvent être utilisées pour satisfaire les exigences de formation décrites précédemment. Il détaille la procédure à suivre pour mettre en place un laboratoire à des fins de formation.  

## <a name="create-the-lab-account-as-a-lab-account-administrator"></a>Créer le compte de laboratoire en tant qu’administrateur de compte de laboratoire
La première étape à effectuer dans Azure Lab Services consiste à créer un compte de laboratoire dans le portail Azure. L’administrateur qui crée le compte de laboratoire ajoute les utilisateurs qui souhaitent créer des laboratoires au rôle **Créateur de laboratoire**. Les formateurs créent des laboratoires avec des machines virtuelles pour que les étudiants y effectuent les exercices du cours qu’ils enseignent. Pour plus d’informations, consultez [Créer et gérer des comptes de laboratoire](how-to-manage-lab-accounts.md)

## <a name="create-and-manage-labs"></a>Créer et gérer des labos
Un formateur, qui est membre du rôle Créateur de laboratoire dans un compte lab, peut créer un ou plusieurs laboratoires dans le compte lab. Créez et configurez un modèle de machine virtuelle avec tous les logiciels requis pour effectuer les exercices de votre cours. Choisissez une image prête à l’emploi parmi les images disponibles pour la création d’un laboratoire de classe et personnalisez-la en installant le logiciel requis pour le laboratoire. Pour plus d’informations, consultez [Créer et gérer des labos](how-to-manage-classroom-labs.md).

## <a name="configure-usage-settings-and-policies"></a>Configurer des paramètres et des stratégies d’utilisation
Le créateur de laboratoire peut ajouter ou supprimer des utilisateurs au laboratoire, obtenir le lien d’inscription à envoyer aux utilisateurs du laboratoire, définir des stratégies telles que la définition de quotas individuels par utilisateur, mettre à jour le nombre de machines virtuelles disponibles dans le laboratoire et bien plus encore. Pour plus d’informations, consultez [Configurer des paramètres et des stratégies d’utilisation](how-to-configure-student-usage.md).

## <a name="create-and-manage-schedules"></a>Créer et gérer des planifications
Les planifications vous permettent de configurer un laboratoire de classe de sorte que les machines virtuelles utilisées dans le laboratoire démarrent et s’arrêtent automatiquement à une heure précise. Vous pouvez définir une planification ponctuelle ou périodique. Pour plus d’informations, consultez [Créer et gérer des planifications de labo](how-to-create-schedules.md).

## <a name="set-up-and-publish-a-template-vm"></a>Configurer et publier un modèle de machine virtuelle
Le modèle de laboratoire est une image de machine virtuelle de base, à partir de laquelle toutes les machines virtuelles des utilisateurs sont créées. Configurez le modèle de machine virtuelle de façon à ce qu’il propose exactement ce que vous souhaitez fournir aux participants à la formation. Vous pouvez indiquer le nom et la description du modèle que voient les utilisateurs du laboratoire. Ensuite, publiez le modèle pour rendre les instances du modèle de machine virtuelle accessibles aux utilisateurs de votre laboratoire. Quand vous publiez un modèle, Azure Lab Services crée les machines virtuelles dans le laboratoire à l’aide du modèle. Le nombre de machines virtuelles créées dans ce processus est identique au nombre maximal d’utilisateurs autorisés dans le laboratoire, que vous pouvez définir dans la politique d’utilisation du laboratoire. Toutes les machines virtuelles ont la même configuration que le modèle. Pour plus d’informations, consultez [Configurer et publier des modèles de machine virtuelle](how-to-create-manage-template.md). 

## <a name="use-vms-in-the-classroom-lab"></a>Utiliser des machines virtuelles dans un laboratoire de classe
Un étudiant ou un participant à une formation s’inscrit au laboratoire et se connecte à la machine virtuelle pour effectuer les exercices du cours. Pour plus d’informations, consultez [Accéder à un laboratoire de classe](how-to-use-classroom-lab.md).

## <a name="next-steps"></a>Étapes suivantes
Commencez par créer un compte de labo dans les labos en suivant les instructions de l’article [Tutoriel : Configurer un compte de laboratoire avec Azure Lab Services](tutorial-setup-lab-account.md).