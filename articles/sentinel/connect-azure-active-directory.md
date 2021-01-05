---
title: Connecter des données Azure Active Directory à Azure Sentinel | Microsoft Docs
description: Découvrez comment collecter des données à partir d’Azure Active Directory et diffuser en continu des journaux de connexion Azure AD et des journaux d’audit dans Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 0a8f4a58-e96a-4883-adf3-6b8b49208e6a
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/11/2020
ms.author: yelevin
ms.openlocfilehash: 279f54c3de964580cc37d1288a6e1b7726348e10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88208623"
---
# <a name="connect-data-from-azure-active-directory-azure-ad"></a>Connecter des données à partir d’Azure Active Directory (Azure AD)



Vous pouvez utiliser le connecteur intégré d’Azure Sentinel pour collecter des données à partir d’[Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) et de les diffuser en continu vers Azure Sentinel. Le connecteur vous permet de diffuser des [journaux de connexion](../active-directory/reports-monitoring/concept-sign-ins.md) et [journaux d’audit](../active-directory/reports-monitoring/concept-audit-logs.md).

## <a name="prerequisites"></a>Prérequis


- Toute licence Azure AD (Free/O365/P1/P2) est suffisante pour recevoir les journaux de connexion dans Azure Sentinel. Des frais supplémentaires par gigaoctet peuvent s’appliquer pour Azure Monitor (Log Analytics) et Azure Sentinel.

- Le rôle Contributeur Azure Sentinel doit être attribué à votre utilisateur sur l’espace de travail.

- Les rôles Administrateur général ou Administrateur de sécurité doivent être attribués à votre utilisateur sur le locataire à partir duquel vous souhaitez diffuser les journaux.

- Votre utilisateur doit disposer d’autorisations en lecture et en écriture sur les paramètres de diagnostic Azure AD pour pouvoir voir l’état de la connexion. 


## <a name="connect-to-azure-active-directory"></a>Connexion à Azure Active Directory

1. Dans Azure Sentinel, sélectionnez **Connecteurs de données** dans le menu de navigation.

1. Dans la galerie des connecteurs de données, sélectionnez **Azure Active Directory**, puis sélectionnez **Ouvrir la page du connecteur**.

1. Cochez les cases associées aux journaux que vous souhaitez diffuser dans Azure Sentinel, puis cliquez sur **Se connecter**.

1. Vous pouvez indiquer si vous voulez que les alertes d’Azure AD génèrent automatiquement des incidents dans Azure Sentinel. Sous **Créer des incidents**, sélectionnez **Activer** pour activer la règle analytique par défaut qui crée automatiquement des incidents à partir des alertes générées dans le service de sécurité connecté. Vous pouvez ensuite modifier cette règle sous **Analytique**, puis **Règles actives**.

1. Pour utiliser le schéma approprié dans Log Analytics afin d’exécuter une requête pour les alertes Azure AD, saisissez `SigninLogs` ou `AuditLogs` dans la fenêtre de requête.

## <a name="next-steps"></a>Étapes suivantes
Dans ce document, vous avez appris à connecter Azure Active Directory à Azure Sentinel. Pour en savoir plus sur Azure Sentinel, voir les articles suivants :
- Découvrez comment [avoir une visibilité sur vos données et les menaces potentielles](quickstart-get-visibility.md).
- Prise en main de la [détection des menaces avec Azure Sentinel](tutorial-detect-threats-built-in.md).
