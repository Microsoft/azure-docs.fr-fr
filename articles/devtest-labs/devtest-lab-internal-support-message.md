---
title: Ajouter une déclaration de support interne à un lab dans Azure DevTest Labs
description: Découvrez comment publier une déclaration de support interne sur un lab dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1c9920e6fe7fbfe2a8d0aeacb896150b342981b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480471"
---
# <a name="add-an-internal-support-statement-to-a-lab-in-azure-devtest-labs"></a>Ajouter une déclaration de support interne à un lab dans Azure DevTest Labs

Azure DevTest Labs vous permet de personnaliser votre lab avec une déclaration de support interne qui fournit aux utilisateurs des informations de support sur le lab. Par exemple, vous pouvez fournir des informations de contact afin que l’utilisateur sache comment atteindre le support interne quand il a besoin d’aide pour résoudre des problèmes ou accéder à des ressources du lab. Vous pouvez également fournir des liens vers des sites web internes ou des Forums aux questions auxquels votre équipe peut accéder avant de contacter le support.

Vous pouvez utiliser une déclaration de support interne pour publier des informations sur le lab qui sont peu susceptibles d’évoluer. Pour notifier les utilisateurs d’informations sur le lab plus provisoires par essence, telles que les mises à jour récentes des stratégies du lab, consultez [Publier une annonce dans un lab](devtest-lab-announcements.md).

Vous pouvez facilement désactiver ou modifier une déclaration de support quand elle n’est plus applicable.

## <a name="steps-to-add-a-support-statement-to-an-existing-lab"></a>Procédure à suivre pour ajouter une déclaration de support à un lab existant

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Si nécessaire, sélectionnez **Autres services**, puis **DevTest Labs** dans la liste. (Votre lab peut déjà être affiché dans le Tableau de bord sous **Toutes les ressources**).
1. Dans la liste des labs, sélectionnez le lab auquel vous souhaitez ajouter une déclaration de support.  
1. Dans la zone **Vue d’ensemble** du laboratoire, sélectionnez **Configuration et stratégies**.  

    ![Bouton Configuration et stratégies](./media/devtest-lab-internal-support-message/devtestlab-config-and-policies.png)

1. Sur la gauche, sous **PARAMÈTRES**, sélectionnez **Support interne**.

    ![Bouton Support interne](./media/devtest-lab-internal-support-message/devtestlab-internal-support.png)

1. Pour créer un message de support interne à l’intention des utilisateurs de ce lab, définissez Activé sur **Oui**.

1. Dans le champ **Message de support**, entrez la déclaration de support interne que vous souhaitez présenter aux utilisateurs de votre lab. Le message de support accepte Markdown. Au fur et à mesure que vous entrez le texte du message, vous pouvez observer la zone **Aperçu** en bas de l’écran pour voir le message tel qu’il apparaît aux utilisateurs.

    ![Écran Support interne pour créer le message](./media/devtest-lab-internal-support-message/devtestlab-add-support-statement.png)


1. Sélectionnez **Enregistrer** quand la déclaration est prête à être publiée.

Quand vous ne souhaitez plus afficher ce message de support pour les utilisateurs du lab, revenez à la page **Support interne** et définissez **Activé** sur **Non**.

## <a name="steps-for-users-to-view-the-support-message"></a>Procédure permettant aux utilisateurs d’afficher le message de support

1. À partir du [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040), sélectionnez un lab.

1. Dans la zone **Vue d’ensemble** du lab, sélectionnez **Support interne**.  

    ![Bouton Support interne](./media/devtest-lab-internal-support-message/devtestlab-internal-support.png)


1. Si un message de support est publié, l’utilisateur peut le voir dans le volet Support interne.

    ![Volet Support interne montrant le message de support publié](./media/devtest-lab-internal-support-message/devtestlab-view-suport-statement.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
* Les déclarations de support interne sont généralement utilisées pour fournir des informations de support qui ne changent pas fréquemment. Vous pouvez également découvrir comment [publier une annonce sur un lab](devtest-lab-announcements.md) pour informer les utilisateurs de mises à jour ou de modifications temporaires du lab.
* La page [Définir des stratégies et des planifications](devtest-lab-set-lab-policy.md) vous indique comme vous pouvez appliquer d’autres conventions et restrictions sur votre abonnement à l’aide de stratégies personnalisées.