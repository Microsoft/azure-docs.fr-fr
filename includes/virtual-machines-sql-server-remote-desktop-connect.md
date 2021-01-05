---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: fe5daa38c43723c85fb464e191ee4a3e85700e0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177019"
---
1. Une fois la machine virtuelle créée et en cours d’exécution, cliquez sur l’icône Machines virtuelles dans le portail Azure pour visualiser vos machines virtuelles.

1. Cliquez sur les points de suspension, **...** , en regard de votre nouvelle machine virtuelle.

1. Cliquez sur **Connecter**.

   ![Se connecter à une machine virtuelle dans le portail](./media/virtual-machines-sql-server-remote-desktop-connect/azure-virtual-machine-connect.png)

1. Ouvrez le fichier **RDP** que votre navigateur télécharge pour la machine virtuelle.

1. La connexion Bureau à distance vous informe que le serveur de publication de cette connexion à distance n’est pas identifiable. Cliquez sur **Connect** pour continuer.

1. Dans la boîte de dialogue **Sécurité de Windows**, cliquez sur **Utiliser un autre compte**. Vous pouvez avoir besoin de cliquer sur **Autres choix** pour voir cette option. Spécifiez le nom d’utilisateur et le mot de passe que vous avez configurés lorsque vous avez créé la machine virtuelle. Vous devez ajouter une barre oblique inverse avant le nom d’utilisateur.

   ![Authentification Bureau à distance](./media/virtual-machines-sql-server-remote-desktop-connect/remote-desktop-connect.png)

1. Cliquez sur **OK** pour vous connecter.
