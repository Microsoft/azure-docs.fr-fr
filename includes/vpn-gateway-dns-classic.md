---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/08/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 0c5e3a095b312d36cb05a78e84a13bc2a73d95d5
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91875548"
---
Les paramètres DNS ne constituent pas une partie obligatoire de cette configuration, mais le DNS est nécessaire si vous souhaitez la résolution de noms entre vos machines virtuelles. La définition d’une valeur n’entraîne pas la création de serveur DNS. L’adresse IP du serveur DNS que vous spécifiez doit pouvoir résoudre les noms des ressources auxquelles vous vous connectez.

Après avoir créé votre réseau virtuel, vous pouvez ajouter l’adresse IP d’un serveur DNS pour gérer la résolution de noms. Ouvrez les paramètres de votre réseau virtuel, sélectionnez Serveurs DNS et ajoutez l’adresse IP du serveur DNS que vous souhaitez utiliser pour la résolution de noms.

1. Recherchez le réseau virtuel dans le portail.
2. Sur la page de votre réseau virtuel, dans la section **Paramètres**, sélectionnez **Serveurs DNS**.
3. Ajoutez un serveur DNS.
4. Pour enregistrer vos paramètres, sélectionnez **Enregistrer** en haut de la page.