---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/14/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 8fba8aa577bcb3b5ef44d57c388a1f1de7494782
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995611"
---
Si vous souhaitez que les utilisateurs soient invités à fournir un deuxième facteur d’authentification pour obtenir l’accès, vous pouvez configurer Azure AD Multi-Factor Authentication (MFA). Vous pouvez configurer MFA par utilisateur ou tirer parti de l’authentification MFA via l’[accès conditionnel](../articles/active-directory/conditional-access/overview.md).

* Vous pouvez activer MFA par utilisateur sans frais supplémentaires. Dans ce cas, l’utilisateur est invité à fournir un deuxième facteur d’authentification pour toutes les applications liées au locataire Azure AD. Les étapes à suivre sont présentées dans [Option 1](#peruser).
* L’accès conditionnel autorise un contrôle plus précis sur la façon dont le deuxième facteur doit être promu. Il peut autoriser l’affectation de l’authentification MFA au VPN uniquement et exclure les autres applications liées au client Azure AD. Les étapes à suivre sont présentées dans [Option 2](#conditional).