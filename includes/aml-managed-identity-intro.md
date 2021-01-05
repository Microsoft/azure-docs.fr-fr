---
title: Fichier include
description: inclure fichier
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 08/24/2020
ms.openlocfilehash: 70b636b7bb508b71475a7464983b091d5d10e0e1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96026736"
---
 Les clusters de calcul Azure Machine Learning prennent également en charge les [identités managées](../articles/active-directory/managed-identities-azure-resources/overview.md) pour authentifier l'accès aux ressources Azure sans inclure d'informations d'identification dans votre code. Il existe deux types d’identités administrées :

* Une **identité managée attribuée par le système** est activée directement sur le cluster de calcul Azure Machine Learning. Le cycle de vie d'une identité attribuée par le système est directement lié au cluster de calcul. Si le cluster de calcul est supprimé, Azure efface automatiquement les informations d'identification et l'identité dans Azure AD.
* Une **identité managée attribuée par l'utilisateur** est une ressource Azure autonome fournie via le service Azure Managed Identity. Vous pouvez attribuer une identité managée attribuée par l'utilisateur à plusieurs ressources et la conserver aussi longtemps que vous le souhaitez.