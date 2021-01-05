---
title: Rapport d’utilisation et d’insights | Microsoft Docs
description: Introduction au rapport d’utilisation et d’insights dans le portail Azure Active Directory
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 3fba300d-18fc-4355-9924-d8662f563a1f
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/13/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 2c7cc68c84cc9f137ba5b51206526ff96111fe9a
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93122739"
---
# <a name="usage-and-insights-report-in-the-azure-active-directory-portal"></a>Rapport d’utilisation et d’insights dans le portail Azure Active Directory

Le rapport d’utilisation et d’insights vous fournit une vue centrée sur les applications de vos données de connexion. Vous y trouverez des réponses aux questions suivantes :

*   Quelles sont les principales applications utilisées dans mon organisation ?
*   Quelles sont les applications affichant le plus d’échecs de connexion ? 
*   Quelles sont les principales erreurs de connexion pour chaque application ?

## <a name="prerequisites"></a>Prérequis 

Pour accéder aux données du rapport d’utilisation et d’insights, vous avez besoin des éléments suivants :

* un locataire Azure AD ;
* une licence Azure AD premium (P1/P2) pour afficher les données de connexion
* Un utilisateur dans les rôles Administrateur général, Administrateur de la sécurité, Lecteur Sécurité ou Lecteur de rapport. De plus, tous les utilisateurs (non administrateurs) peuvent accéder à leurs propres connexions. 

## <a name="access-the-usage-and-insights-report"></a>Accès au rapport d’utilisation et d’insights

1. Accédez au [portail Azure](https://portal.azure.com).
2. Sélectionnez le rapport correspondant, **Azure Active Directory** , puis choisissez **Applications d’entreprise**.
3. Dans la section **Activité** , sélectionnez **Utilisation et insights** pour ouvrir le rapport. 

![Capture d’écran montrant l’option Utilisation et insights sélectionnée dans la section Activité.](./media/concept-usage-insights-report/main-menu.png)
                                     

## <a name="use-the-report"></a>Utiliser le rapport

Le rapport d’utilisation et d’insights affiche la liste des applications avec une ou plusieurs tentatives de connexion, et vous permet de trier selon le nombre de connexions réussies, de connexions échouées, et le taux de réussite.

L’option Charger plus en bas de la liste vous permet d’afficher d’autres applications sur la page. Vous pouvez sélectionner la plage de dates afin d’afficher toutes les applications utilisées dans cette plage.

Vous pouvez également définir le focus sur une application spécifique. Sélectionnez **Voir l'activité de connexion** afin d’afficher l’activité de connexion dans le temps pour l’application, ainsi que les principales erreurs.  

Lorsque vous sélectionnez un jour dans le graphique d’utilisation des applications, vous obtenez une liste détaillée des activités de connexion pour l’application.  

![Capture d’écran montrant l’option Utilisation & insights pour l’activité Application, dans laquelle vous pouvez sélectionner une plage et afficher l’activité de connexion pour différentes applications.](./media/concept-usage-insights-report/usage-and-insights-report.png)

## <a name="next-steps"></a>Étapes suivantes

* [Rapport de connexions](concept-sign-ins.md)