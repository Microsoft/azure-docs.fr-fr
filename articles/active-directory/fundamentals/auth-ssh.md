---
title: Authentification SSH avec Azure Active Directory
description: Guide architectural sur l’implémentation de SSH avec Azure Active Directory
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/19/2020
ms.author: baselden
ms.reviewer: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3b77ab0832fa19149c270d6ba5a6641069548cbe
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172718"
---
# <a name="ssh"></a>SSH  

Le protocole SSH (Secure Shell) est un protocole réseau qui fournit un chiffrement permettant d’exploiter en toute sécurité des services réseau sur un réseau non sécurisé. SSH fournit également une connexion en ligne de commande, exécute des commandes à distance et transfère des fichiers de façon sécurisée. Il est couramment utilisé sur les systèmes UNIX tels que Linux®. SSH remplace le protocole Telnet, qui ne fournit pas de chiffrement sur un réseau non sécurisé. 

Azure Active Directory (Azure AD) fournit une extension de machine virtuelle (VM) pour les systèmes Linux® exécutés sur Azure. 

## <a name="use-when"></a>Cas d'utilisation 

* Utiliser des machines virtuelles Linux® nécessitant une connexion à distance

* Exécution de commandes à distance dans les systèmes Linux®

* Transférer des fichiers de façon sécurisée sur un réseau non sécurisé

![Diagramme d’Azure AD avec le protocole SSH](./media/authentication-patterns/ssh-auth.png)

SSH avec Azure AD

## <a name="components-of-system"></a>Composants du système 

* **Utilisateur** : Démarre le client SSH pour établir une connexion avec les machines virtuelles Linux® et fournit des informations d’identification pour l’authentification.

* **Navigateur Web** : Composant avec lequel l’utilisateur interagit. Il communique avec le fournisseur d’identité (Azure AD) pour authentifier et autoriser l’utilisateur de façon sécurisée.

* **Client SSH** : Pilote le processus de configuration de la connexion.

* **Azure AD** : Authentifie l’identité de l’utilisateur à l’aide du flux d’appareil et émet un jeton sur les machines virtuelles Linux.

* **Machine virtuelle Linux** : Accepte le jeton et fournit une connexion réussie.

## <a name="implement-ssh-with-azure-ad"></a>Implémenter SSH avec Azure AD 

* [Se connecter à une machine virtuelle Linux® avec les informations d’identification d’Azure Active Directory - Machines virtuelles Azure](../../virtual-machines/linux/login-using-aad.md) 

* [Flux de code d’appareil OAuth 2.0 - Plateforme d’identités Microsoft](../develop/v2-oauth2-device-code.md)

* [Intégrer à Azure Active Directory (akamai.com)](https://learn.akamai.com/webhelp/enterprise-application-access/enterprise-application-access/GUID-6B16172C-86CC-48E8-B30D-8E678BF3325F.html)

