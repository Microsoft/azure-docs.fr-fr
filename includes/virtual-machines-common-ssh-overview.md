---
title: Fichier Include
description: Fichier Include
services: virtual-machines-linux
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/09/2020
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 612e7dc2f3d1ef6aa5d3598999aa214a2f377e1f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016361"
---
## <a name="overview-of-ssh-and-keys"></a>Vue d’ensemble de SSH et des clés

[SSH](https://www.ssh.com/ssh/) est un protocole de connexion chiffré qui permet d’ouvrir des sessions sécurisées via des connexions non sécurisées. SSH est le protocole de connexion par défaut pour les machines virtuelles Linux hébergées dans Azure. Bien que le protocole SSH offre une connexion chiffrée, l’utilisation de mots de passe avec des connexions SSH laisse néanmoins la machine virtuelle vulnérable aux attaques en force brute. Nous vous recommandons de vous connecter à une machine virtuelle via SSH à l’aide d’une paire de clés publique/privée, également appelées clés *SSH*. 

- La *clé publique* est placée sur votre machine virtuelle Linux.

- La *clé privée* reste sur votre système local. Protégez cette clé privée. Ne la partagez pas.

Quand vous utilisez un client SSH pour vous connecter à votre machine virtuelle Linux (qui a la clé publique), la machine virtuelle distante teste le client pour vérifier qu’il a la bonne clé privée. Si le client a la clé privée, il est autorisé à accéder à la machine virtuelle. 

Selon les stratégies de sécurité de votre organisation, vous pouvez réutiliser une paire de clés publique-privée unique pour accéder à plusieurs services et machines virtuelles Azure. Vous n’avez pas besoin d’une paire de clés distincte pour chaque machine virtuelle ou service auxquels vous souhaitez accéder. 

Votre clé publique peut être partagée avec n’importe qui, mais vous seul (ou votre infrastructure de sécurité locale) devez avoir accès à votre clé privée.