---
title: Guide pratique pour résoudre les rapports d’erreurs de connexion | Microsoft Docs
description: Découvrez comment résoudre les erreurs de connexion avec les rapports Azure Active Directory dans le portail Azure
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: report-monitor
ms.date: 11/13/2018
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4de9f5a21f38f741dcf65d285446d9482716d4f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85608124"
---
# <a name="how-to-troubleshoot-sign-in-errors-using-azure-active-directory-reports"></a>Guide pratique pour résoudre les erreurs de connexion avec les rapports Azure Active Directory

Le [rapport des connexions](concept-sign-ins.md) dans Azure Active Directory (Azure AD) vous permet de trouver les réponses aux questions sur la gestion de l’accès aux applications dans votre organisation, notamment :

- Quel est le modèle de connexion d’un utilisateur ?
- Combien d’utilisateurs se sont connectés au cours d’une semaine ?
- Quel est l’état de ces connexions ?


En outre, le rapport des connexions peut également vous aider à résoudre les échecs de connexion pour les utilisateurs de votre organisation. Dans ce guide, vous découvrez comment isoler un échec de connexion dans le rapport des connexions et comment l’utiliser pour comprendre la cause racine de l’échec.

## <a name="prerequisites"></a>Conditions préalables requises

Ce dont vous avez besoin :

* Un locataire Azure AD avec une licence Premium (P1/P2). Consultez [Bien démarrer avec Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md) pour mettre à niveau votre édition d’Azure Active Directory.
* Un utilisateur dans le rôle **Administrateur général**, **Administrateur de la sécurité**, **Lecteur Sécurité** ou **Lecteur de rapport** pour le locataire. De plus, tout utilisateur peut accéder à ses propres connexions. 

## <a name="troubleshoot-sign-in-errors-using-the-sign-ins-report"></a>Résoudre les erreurs de connexion avec le rapport des connexions

1. Accédez au [portail Azure](https://portal.azure.com) et sélectionnez votre annuaire.
2. Sélectionnez **Azure Active Directory**, puis **Connexions** dans la section **Supervision**. 
3. Utilisez les filtres fournis pour trouver l’échec, en utilisant le nom d’utilisateur, l’identificateur d’objet, le nom de l’application ou la date. Sélectionnez aussi **Échec** dans la liste déroulante **État** pour afficher seulement les connexions en échec. 

    ![Filtrer les résultats](./media/howto-troubleshoot-sign-in-errors/filters.png)
        
4. Identifiez la connexion en échec que vous voulez examiner. Sélectionnez-la pour ouvrir la fenêtre contenant des informations détaillées sur la connexion en échec. Prenez note du **Code d’erreur de connexion** et de la **Raison de l’échec**. 

    ![Sélectionner un enregistrement](./media/howto-troubleshoot-sign-in-errors/sign-in-failures.png)
        
5. Vous pouvez également trouver ces informations sous l’onglet **Résolution des problèmes et support** dans la fenêtre des détails.

    ![Résolution des problèmes et support](./media/howto-troubleshoot-sign-in-errors/troubleshooting-and-support.png)

6. La raison de l’échec décrit l’erreur. Par exemple, dans le scénario ci-dessus, la raison de l’échec est **Nom d’utilisateur ou mot de passe non valide, ou Nom d’utilisateur ou mot de passe local non valide**. La solution consiste simplement à se reconnecter avec le nom d’utilisateur et le mot de passe corrects.

7. Vous pouvez obtenir des informations supplémentaires, notamment des idées pour remédier au problème, en recherchant le code d’erreur, **50126** dans cet exemple, dans les [informations de référence sur les codes d’erreur des connexions](reference-sign-ins-error-codes.md). 

8. Si tout le reste échoue ou si le problème persiste en dépit de l’application des actions recommandées, [ouvrez un ticket de support](../fundamentals/active-directory-troubleshooting-support-howto.md) en suivant les étapes décrites sous l’onglet **Résolution des problèmes et support**. 

## <a name="next-steps"></a>Étapes suivantes

* [Informations de référence sur les codes d’erreur des connexions](reference-sign-ins-error-codes.md)
* [Vue d’ensemble du rapport des connexions](concept-sign-ins.md)
