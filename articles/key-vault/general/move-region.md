---
title: Déplacer un coffre de clés dans une autre région – Azure Key Vault | Microsoft Docs
description: Cet article fournit des conseils pour déplacer un coffre de clés dans une autre région.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/24/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to move my vault to another region.
ms.openlocfilehash: 54a1894c88d12cfac1a2d8062425de9549c4292e
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285201"
---
# <a name="move-an-azure-key-vault-across-regions"></a>Déplacer un coffre de clés Azure d’une région à une autre

Azure Key Vault ne prend pas en charge les opérations de déplacement de ressources permettant de déplacer un coffre de clés d’une région à une autre. Cet article propose des solutions de contournement pour les organisations qui ont le besoin métier de déplacer un coffre de clés dans une autre région. Chaque option de contournement présente des limitations. Il est essentiel de comprendre les implications de ces solutions de contournement avant d’essayer de les appliquer dans un environnement de production.

Pour déplacer un coffre de clés dans une autre région, créez un coffre de clés dans cette région, puis copiez manuellement chaque secret du coffre de clés existant vers le nouveau. Pour ce faire, vous pouvez utiliser l’une des deux options suivantes.

## <a name="design-considerations"></a>Remarques relatives à la conception

Avant de commencer, tenez compte des concepts suivants :

* Les noms de coffres de clés sont globalement uniques. Vous ne pouvez pas réutiliser un nom de coffre de clés.
* Vous devez reconfigurer vos stratégies d’accès et paramètres de configuration réseau dans le nouveau coffre de clés.
* Vous devez reconfigurer la suppression réversible et la protection contre le vidage dans le nouveau coffre de clés.
* L’opération de sauvegarde et restauration ne préserve pas vos paramètres de rotation automatique. Vous serez peut-être amené à reconfigurer les paramètres.

## <a name="option-1-use-the-key-vault-backup-and-restore-commands"></a>Option 1 : Utiliser les commandes de sauvegarde et restauration de coffre de clés

Vous pouvez sauvegarder chacun des secrets, clés et certificats de votre coffre à l’aide de la commande de sauvegarde. Vos secrets sont téléchargés sous forme d’objet blob chiffré. Vous pouvez ensuite restaurer le blob dans votre nouveau coffre de clés. Pour obtenir la liste des commandes, consultez [Commandes Azure Key Vault](/powershell/module/azurerm.keyvault/?view=azurermps-6.13.0#key_vault).

L’utilisation des commandes de sauvegarde et restauration présentent deux limitations :

* Vous ne pouvez pas sauvegarder un coffre de clés dans une zone géographique et le restaurer dans une autre. Pour plus d’informations, consultez [Azure geographies](https://azure.microsoft.com/global-infrastructure/geographies/) (Zones géographiques Azure).

* La commande de sauvegarde effectue une sauvegarde de toutes les versions de chaque secret. Si vous avez un secret avec un grand nombre de versions antérieures (plus de 10), il y a un risque que la requête dépasse la taille de requête maximale autorisée et que l’opération échoue.

## <a name="option-2-manually-download-and-upload-the-key-vault-secrets"></a>Option°2 : Télécharger et charger manuellement les secrets du coffre de clés

Vous pouvez télécharger certains types de secrets manuellement. Par exemple, vous pouvez télécharger les certificats sous forme de fichier PFX. Cette option élimine les restrictions géographiques pour certains types de secrets, tels que les certificats. Vous pouvez charger les fichiers PFX dans un coffre de clés de n’importe quelle région. Votre secret est téléchargé dans un format non protégé par mot de passe. C’est à vous qu’il revient de sécuriser vos secrets pendant le déplacement.