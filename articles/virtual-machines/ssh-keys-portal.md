---
title: Créer des clés SSH dans le portail Azure
description: Découvrez comment générer et stocker des clés SSH dans le portail Azure pour la connexion des machines virtuelles Linux.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/25/2020
ms.author: cynthn
ms.openlocfilehash: abc9a2ae130d987c90ce87ffaecbf2bb44b06010
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88929434"
---
# <a name="generate-and-store-ssh-keys-in-the-azure-portal"></a>Générer et stocker des clés SSH dans le portail Azure

Si vous utilisez fréquemment le portail pour déployer des machines virtuelles Linux, vous pouvez simplifier l’utilisation des clés SSH en les créant directement dans le portail ou en les téléchargeant à partir de votre ordinateur.

Vous pouvez créer une clé SSH quand vous créez une machine virtuelle pour la première fois, puis la réutiliser pour d’autres machines virtuelles. Vous pouvez aussi créer des clés SSH séparément afin de disposer d’un ensemble de clés stockées dans Azure pour répondre aux besoins de votre organisation. 

Si vous disposez de clés existantes et souhaitez simplifier leur utilisation dans le portail, vous pouvez les télécharger et les stocker dans Azure en vue de les réutiliser.

Pour plus d’informations sur la création et l’utilisation de clés SSH avec des machines virtuelles Linux, consultez [Utiliser des clés SSH pour se connecter à des machines virtuelles Linux](./linux/ssh-from-windows.md).

## <a name="generate-new-keys"></a>Générer de nouvelles clés

1. Ouvrez le [Portail Azure](https://portal.azure.com).

1. En haut de la page, tapez *SSH* pour rechercher. Sous **Marketplace**, sélectionnez **Clés SSH**.

1. Dans la page **Clé SSH**, sélectionnez **Créer**.

   :::image type="content" source="./media/ssh-keys/portal-sshkey.png" alt-text="Créer un nouveau groupe de ressources et générer une paire de clés SSH":::

1. Dans **Groupe de ressources** sélectionnez **Créer nouveau** afin de créer un nouveau groupe de ressources pour stocker vos clés. Tapez un nom pour votre groupe de ressources, puis sélectionnez **OK**.

1. Dans **Région**, sélectionnez une région pour stocker vos clés. Vous pouvez utiliser les clés dans n’importe quelle région. Il s’agit simplement de la région dans laquelle elles seront stockées.

1. Tapez un nom pour votre clé dans **Nom de la paire de clés**.

1. Dans **Source de la clé publique SSH**, sélectionnez **Générer une source de clé publique**. 

1. Quand vous avez terminé, sélectionnez **Vérifier + créer**.

1. Après la validation, sélectionnez **Créer**.

1. Dans la fenêtre contextuelle qui s’affiche, sélectionnez **Télécharger la clé privée et créer une ressource**. Cela a pour effet de télécharger la clé SSH en tant que fichier .pem.

   :::image type="content" source="./media/ssh-keys/download-key.png" alt-text="Créer un nouveau groupe de ressources et générer une paire de clés SSH":::

1. Une fois le fichier. pem téléchargé, vous pouvez le déplacer quelque part sur votre ordinateur où il est facile de pointer dessus à partir de votre client SSH.


## <a name="connect-to-the-vm"></a>Connexion à la machine virtuelle

Sur votre ordinateur local, ouvrez une invite PowerShell et tapez :

```powershell
ssh -i <path to the .pem file> username@<ipaddress of the VM>
```

Par exemple, tapez `ssh -i /Downloads/mySSHKey.pem azureuser@123.45.67.890`.


## <a name="upload-an-ssh-key"></a>Charger une clé SSH

Vous pouvez également charger une clé SSH publique à stocker dans Azure. Pour plus d’informations sur la création d’une paire de clés SSH, consultez [Utiliser des clés SSH pour se connecter à des machines virtuelles Linux](./linux/ssh-from-windows.md).

1. Ouvrez le [Portail Azure](https://portal.azure.com).

1. En haut de la page, tapez *SSH* pour rechercher. Sous **Marketplace*, sélectionnez **Clés SSH**.

1. Dans la page **Clé SSH**, sélectionnez **Créer**.

   :::image type="content" source="./media/ssh-keys/upload.png" alt-text="Créer un nouveau groupe de ressources et générer une paire de clés SSH":::

1. Dans **Groupe de ressources** sélectionnez **Créer nouveau** afin de créer un nouveau groupe de ressources pour stocker vos clés. Tapez un nom pour votre groupe de ressources, puis sélectionnez **OK**.

1. Dans **Région**, sélectionnez une région pour stocker vos clés. Vous pouvez utiliser les clés dans n’importe quelle région. Il s’agit simplement de la région dans laquelle elles seront stockées.

1. Tapez un nom pour votre clé dans **Nom de la paire de clés**.

1. Dans **Source de clé publique SSH**, sélectionnez **Charger la clé publique existante**. 

1. Collez le contenu complet de la clé publique dans **Charger la clé** puis sélectionnez **Vérifier + créer**.

1. Une fois la validation terminée, sélectionnez **Créer**. 

Une fois la clé chargée, vous pouvez choisir de l’utiliser lors de la création d’une machine virtuelle.

## <a name="list-keys"></a>Afficher la liste des clés

Les clés SSH créées dans le portail sont stockées en tant que ressources, ce qui vous permet de filtrer l’affichage de vos ressources pour les voir toutes.

1. Dans le portail Azure, sélectionnez **Tous les ressources**.
1. Dans les filtres, sélectionnez **Type**, désélectionnez l’option **Tout sélectionner** pour effacer la liste.
1. Tapez **SSH** dans le filtre et sélectionnez **Clé SSH**.

   :::image type="content" source="./media/ssh-keys/filter.png" alt-text="Créer un nouveau groupe de ressources et générer une paire de clés SSH":::

## <a name="get-the-public-key"></a>Obtenir la clé publique

Si vous avez besoin de votre clé publique, vous pouvez facilement la copier à partir de la page du portail pour la clé. Listez simplement vos clés (à l’aide du processus indiqué dans la dernière section), puis sélectionnez une clé dans la liste. La page de votre clé s’ouvre et vous pouvez cliquer sur l’icône **Copier dans le Presse-papiers** en regard de la clé pour la copier.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation des clés SSH avec des machines virtuelles Azure, consultez [Utiliser des clés SSH pour se connecter à des machines virtuelles Linux](./linux/ssh-from-windows.md).
