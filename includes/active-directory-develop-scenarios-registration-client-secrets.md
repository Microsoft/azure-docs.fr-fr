---
title: Fichier include
description: Fichier include pour les pages d’accueil du scénario de client confidentiel (démon, application web, API web)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/14/2020
ms.author: jmprieur
ms.openlocfilehash: 42102f38959911388cefcc141d949e59f24a2c31
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996002"
---
## <a name="add-a-client-secret-or-certificate"></a>Ajouter un certificat ou une clé secrète client

Comme pour toute application cliente confidentielle, vous devez ajouter un secret ou un certificat pour servir d’*informations d’identification* à l’application afin qu’elle puisse s’authentifier elle-même, sans intervention de l’utilisateur.

Vous pouvez ajouter des informations d’identification à l’inscription de votre application cliente à l’aide du [portail Azure](#add-client-credentials-by-using-the-azure-portal) ou en utilisant un outil en ligne de commande comme [PowerShell](#add-client-credentials-by-using-powershell).

### <a name="add-client-credentials-by-using-the-azure-portal"></a>Ajouter les informations d’identification du client à l’aide du portail Azure

Pour ajouter des informations d’identification à l’inscription de votre application cliente confidentielle, suivez les étapes décrites dans [Démarrage rapide : Inscrire une application avec la plateforme d’identités Microsoft](../articles/active-directory/develop/quickstart-register-app.md) pour le type d’informations d’identification que vous souhaitez ajouter :

* [Ajouter un secret client](../articles/active-directory/develop/quickstart-register-app.md#add-a-client-secret)
* [Ajouter un certificat](../articles/active-directory/develop/quickstart-register-app.md#add-a-certificate)

### <a name="add-client-credentials-by-using-powershell"></a>Ajouter les informations d’identification du client à l’aide de PowerShell

Vous pouvez également ajouter des informations d’identification lorsque vous inscrivez votre application auprès de la plateforme d’identités Microsoft à l’aide de PowerShell.

L’exemple de code [active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) sur le site de GitHub montre comment ajouter un certificat ou un secret d’application lors de l’inscription d’une application :

- Pour plus d’informations sur l’ajout d’une **clé secrète client**, consultez [AppCreationScripts/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts/Configure.ps1#L190).
- Pour plus d’informations sur l’ajout d’un **certificat** avec PowerShell, consultez [AppCreationScripts-withCert/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts-withCert/Configure.ps1#L162-L178).
