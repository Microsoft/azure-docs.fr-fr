---
title: Utilisation d’applications dans Azure Blockchain Workbench
description: Préversion du tutoriel sur l’utilisation des contrats d’application dans Azure Blockchain Workbench.
ms.date: 10/14/2019
ms.topic: tutorial
ms.reviewer: brendal
ms.openlocfilehash: 5761bf1294691c2d50e0e389fe69ec286df4a06c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74324038"
---
# <a name="tutorial-using-applications-in-azure-blockchain-workbench"></a>Tutoriel : Utilisation d’applications dans Azure Blockchain Workbench

Vous pouvez utiliser Blockchain Workbench pour créer et exécuter des opérations sur des contrats. Vous pouvez également afficher les détails du contrat, tels que l’état et l’historique des transactions.

Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un contrat
> * Prendre une mesure sur un contrat

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Un déploiement Blockchain Workbench. Pour plus de détails sur le déploiement, consultez [Azure Blockchain Workbench deployment](deploy.md) (Déploiement d’Azure Blockchain Workbench).
* Une application blockchain déployée dans Blockchain Workbench. Voir [Créer une application blockchain dans Azure Blockchain Workbench](create-app.md)

[Ouvrez Blockchain Workbench](deploy.md#blockchain-workbench-web-url) dans votre navigateur.

![Blockchain Workbench](./media/use/workbench.png)

Vous devez vous connecter en tant que membre de Blockchain Workbench. Si aucune application n’est répertoriée, cela signifie que vous êtes bien membre de Blockchain Workbench, mais que vous n’êtes membre d’aucune application. L’administrateur Blockchain Workbench peut attribuer des membres à des applications.

## <a name="create-new-contract"></a>Créer un contrat

Pour créer un contrat, vous devez être un membre désigné comme **initiateur** de contrat. Pour plus d’informations sur les rôles d’application et les initiateurs de contrat, consultez la section [Flux de travail dans la vue d’ensemble de la configuration](configuration.md#workflows). Pour plus d’informations sur l’affectation de membres à des rôles d’application, consultez [Ajouter un membre à l’application](manage-users.md#add-member-to-application).

1. Dans la section des applications Blockchain Workbench, sélectionnez la vignette d’application qui contient le contrat que vous souhaitez créer. La liste des contrats actifs s’affiche.

2. Pour créer un contrat, sélectionnez **Nouveau contrat**.

    ![Bouton Nouveau contrat](./media/use/contract-list.png)

3. Le volet **Nouveau contrat** s’affiche. Spécifiez les valeurs de paramètres initiales. Sélectionnez **Create** (Créer).

    ![Volet Nouveau contrat](./media/use/new-contract.png)

    Le contrat que vous venez de créer s’affiche dans la liste avec les autres contrats actifs.

    ![Liste des contrats actifs](./media/use/active-contracts.png)

## <a name="take-action-on-contract"></a>Effectuer une opération sur un contrat

Selon l’état du contrat, les membres peuvent entreprendre des actions pour passer à la phase suivante du contrat. Les actions sont définies comme les [transitions](configuration.md#transitions) d’un [état](configuration.md#states). Les membres affectés à un rôle d’application ou d’instance qui est autorisé à effectuer la transition peuvent entreprendre cette action. 

1. Dans la section des applications Blockchain Workbench, sélectionnez la vignette d’application qui contient le contrat sur lequel vous souhaitez exécuter une action.
2. Sélectionnez le contrat dans la liste. Les détails du contrat sont affichés dans différentes sections. 

    ![Détails du contrat](./media/use/contract-details.png)

    | Section  | Description  |
    |---------|---------|
    | Statut | Répertorie l’état d’avancement des étapes du contrat |
    | Détails | Valeurs actuelles du contrat |
    | Action | Détails de la dernière action |
    | Activité | Historique des transactions du contrat |
    
3. Dans la section **Action**, sélectionnez **Effectuer une action**.

4. Les détails de l’état actuel du contrat s’affichent affichés dans un volet. Choisissez l’action à exécuter dans la liste déroulante. 

    ![Choisir une action](./media/use/choose-action.png)

5. Pour démarrer une action, sélectionnez **Entreprendre une action**.
6. Si des paramètres doivent être configurés pour cette action, spécifiez les valeurs nécessaires.

    ![Effectuer une action](./media/use/take-action.png)

7. Pour exécuter une action, sélectionnez **Entreprendre une action**.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Versions de l’application Azure Blockchain Workbench](version-app.md)
