---
title: Surveillance et gestion de session Azure Bastion | Microsoft Docs
description: Cet article explique comment sélectionner une session en cours et forcer sa déconnexion ou la supprimer.
services: bastion
author: charwen
ms.service: bastion
ms.topic: how-to
ms.date: 05/21/2020
ms.author: charwen
ms.openlocfilehash: beb4f8fc5b9e22bc3e2bdd4732c8321dfcd70780
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90980720"
---
# <a name="session-monitoring-and-management-for-azure-bastion"></a>Surveillance et gestion de session Azure Bastion

Une fois que le service Bastion approvisionné et déployé dans votre réseau virtuel, vous pouvez l’utiliser pour vous connecter sans problème à toute machine virtuelle dans ce réseau virtuel. Quand des utilisateurs se connectent à des charges de travail, Azure bastion permet de surveiller les sessions à distance et de prendre des mesures de gestion rapides. La surveillance de session Azure Bastion vous permet de vois quels utilisateurs sont connectés à quelles machines virtuelles. Elle indique l’adresse IP à partir de laquelle l’utilisateur est connecté, la durée pendant laquelle il a été connecté et le moment auquel il s’est connecté. L’expérience de gestion de session vous permet de sélectionner une session en cours ainsi que forcer sa déconnexion ou la supprimer afin d’en déconnecter l’utilisateur.

## <a name="monitor-remote-sessions"></a><a name="monitor"></a>Surveiller des sessions à distance

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre ressource Azure Bastion, puis sélectionnez **Sessions** dans la page Azure Bastion.

   ![Capture d’écran montrant le menu Paramètres du Portail Azure avec les sessions sélectionnées.](./media/session-monitoring/sessions.png)
2. Dans la page **Sessions**, vous pouvez voir les sessions à distance en cours sur le côté droit.

   ![afficher une session](./media/session-monitoring/view-session.png)
3. Sélectionnez **Actualiser** pour afficher la liste à jour des sessions à distance. Lorsque vous sélectionnez Actualiser, Azure bastion récupère les informations de surveillance les plus récentes et les actualise dans le portail.

   ![Capture d’écran montrant une session qui utilise Azure Bastion avec l’option d’actualisation mise en surbrillance.](./media/session-monitoring/refresh.png)


## <a name="delete-or-force-disconnect-an-ongoing-remote-session"></a><a name="view"></a>Forcer la déconnexion d’une session à distance en cours ou la supprimer

Vous pouvez sélectionner un ensemble de sessions et forcer leur déconnexion. Les étapes suivantes montrent comment supprimer des sessions à distance :

1. Accédez à votre ressource Azure Bastion et sélectionnez **Sessions** dans la page Azure Bastion.

   ![Capture d’écran montrant le Portail Azure avec les Sessions sélectionnées sous Paramètres.](./media/session-monitoring/navigate.png)
2. Une fois l’option Sessions sélectionnée, la liste des sessions à distance s’affiche.

   ![afficher la liste des sessions](./media/session-monitoring/list.png)
3. Sélectionnez une session à distance et les trois ellipses situées à droite de la ligne de la session, puis choisissez **Supprimer**.

   ![Capture d’écran montrant le Portail Azure présentant une session avec l’icône Supprimer sélectionnée.](./media/session-monitoring/delete.png)
4. Lorsque vous sélectionnez Supprimer, la session à distance est déconnectée et l’utilisateur voit s’afficher un message « Vous avez été déconnecté » dans la session à distance.

   ![Capture d’écran montrant un message indiquant que vous avez été déconnecté.](./media/session-monitoring/disconnect.png)

## <a name="next-steps"></a>Étapes suivantes

Lisez les [questions fréquentes sur Bastion](bastion-faq.md).
