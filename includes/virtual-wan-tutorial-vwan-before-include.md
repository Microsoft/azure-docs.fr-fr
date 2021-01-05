---
title: Fichier Include
description: Fichier Include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 09/12/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 945d701a2a7dffc259c601b4dab9fa1333ccc066
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023519"
---
Avant de commencer la configuration, veillez à répondre aux critères suivants :

* Si vous disposez déjà d’un réseau virtuel auquel vous souhaitez vous connecter, vérifiez qu’aucun des sous-réseaux de votre réseau local ne le chevauche. Votre réseau virtuel n’a pas besoin d’un sous-réseau de passerelle et ne peut pas comprendre de passerelles de réseau virtuel. Si vous ne disposez pas d’un réseau virtuel, vous pouvez en créer un en suivant les étapes fournies dans cet article.
* Obtenez une plage d’adresses IP pour la région de votre hub. Le hub est un réseau virtuel, et la plage d’adresses que vous spécifiez pour la région du hub ne peut pas chevaucher le réseau virtuel existant auquel vous vous connectez. Elle ne peut pas non plus chevaucher les plages d’adresses auxquelles vous vous connectez localement. Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, contactez une personne en mesure de vous aider.
* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.