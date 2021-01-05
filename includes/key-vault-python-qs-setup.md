---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 09/03/2020
ms.author: msmbaldwin
ms.openlocfilehash: 9440a33858cb70389e86621ae76be9c08ec56d61
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89482122"
---
1. Veillez à disposer d’un [compte Azure avec un abonnement actif](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

1. Installez [Python 2.7+ ou 3.5.3+](https://www.python.org/downloads).

1. Installez [Azure CLI](/cli/azure/install-azure-cli).

1. Suivez les instructions de la section [Configurer l’authentification pour un développement local](/azure/developer/python/configure-local-development-environment?tabs=cmd#configure-authentication) pour créer un principal de service local et le mettre à la disposition du client Azure Key Vault pour Python via des variables d’environnement. 

    Pour exécuter du code directement dans Azure, il n’est pas nécessaire de disposer d’un principal de service distinct si l’application utilise une identité managée.

1. Dans un terminal ou une invite de commandes, créez un dossier de projet approprié, puis créez et activez un environnement virtuel Python comme décrit dans [Utiliser des environnements virtuels Python](/azure/developer/python/configure-local-development-environment?tabs=cmd#use-python-virtual-environments)

1. Installez la bibliothèque d’identités Azure Active Directory :

    ```terminal
    pip install azure.identity
    ```
