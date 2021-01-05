---
title: Installer Micro Focus Enterprise Server 5.0 et Enterprise Developer 5.0 sur Azure | Microsoft Docs
description: Cet article explique comment à installer Micro Focus Enterprise Server 5.0 et Enterprise Developer 5.0 sur Microsoft Azure.
services: virtual-machines-linux
documentationcenter: ''
author: maggsl
ms.author: edprice
manager: edprice
editor: edprice
ms.topic: conceptual
ms.date: 06/29/2020
tags: ''
keywords: ''
ms.service: multiple
ms.openlocfilehash: d9e5f9b531fc28caf8f3162a70318927d40bb923
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483060"
---
# <a name="install-micro-focus-enterprise-server-50-and-enterprise-developer-50-on-azure"></a>Installer Micro Focus Enterprise Server 5.0 et Enterprise Developer 5.0 sur Azure

Cet article explique la procédure de configuration de [Micro Focus Enterprise Server 5.0](https://www.microfocus.com/documentation/enterprise-developer/ed50pu5/ES-WIN/GUID-F7D8FD6E-BDE0-4169-8D8C-96DDFFF6B495.html) et de [Micro Focus Enterprise Developer 5.0](https://www.microfocus.com/documentation/enterprise-developer/ed50/) sur Microsoft Azure.

Dans Azure, l’environnement de développement et de test constitue une charge de travail courante. Ce scénario est courant, car il est très économique, ainsi que facile à déployer et à supprimer. Avec Enterprise Server, Micro Focus a créé l’une des plus grandes plateformes de réhébergement de mainframe actuellement disponibles. Dans Azure, vous pouvez exécuter des charges de travail z/OS sur une plateforme x86 moins coûteuse à l’aide de machines virtuelles Windows ou Linux.

Cette configuration utilise des machines virtuelles Azure exécutant l’image Windows Server 2016 provenant de la place de marché Azure, avec Microsoft SQL Server 2017 déjà installé. Cette configuration s’applique également à Azure Stack.

L’environnement de développement correspondant à Enterprise Server est Enterprise Developer, qui peut être exécuté sur Microsoft Visual Studio 2017 ou version ultérieure, Visual Studio Community (téléchargement gratuit) ou Eclipse. Cet article explique comment le déployer à l’aide d’une machine virtuelle Windows Server 2016 fournie avec Visual Studio 2017 ou version ultérieure.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, consultez les prérequis suivants :

-   Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

-   Le logiciel Micro Focus et une licence valide (ou licence d’essai). Si vous êtes déjà client de Micro Focus, contactez votre représentant Micro Focus. Sinon, [demandez une version d’évaluation](https://www.microfocus.com/products/enterprise-suite/enterprise-server/trial/).

-   Obtenez la documentation sur [Enterprise Server et Enterprise Developer](https://www.microfocus.com/documentation/enterprise-developer/ed50/).

    > [!Note]
    > Il existe plusieurs options de contrôle de l’accès à vos machines virtuelles :
    > -   Une meilleure pratique consiste à configurer [Azure Bastion](https://azure.microsoft.com/services/azure-bastion/).
    > -   Un tunnel de [réseau privé virtuel (VPN) site à site](../../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md).
    > -   Une machine virtuelle Jumpbox.

## <a name="install-enterprise-server"></a>Installer Enterprise Server

1.  Pour une sécurité améliorée et une plus grande facilité de gestion, vous pouvez créer un groupe de ressources uniquement pour ce projet (par exemple, **RGMicroFocusEntServer**). Pour le type de ressource, utilisez la première partie du nom tel qu’il apparaît dans Azure afin de le repérer plus facilement dans une liste.

2.  Création d’une machine virtuelle Dans la place de marché Azure, sélectionnez la machine virtuelle et le système d’exploitation de votre choix. Voici la configuration recommandée :

    -   **Enterprise Server :** sélectionnez une **machine virtuelle ES2 v3** (avec 2 processeurs virtuels et 16 Go de mémoire) où sont installés Windows Server 2016 et SQL Server 2017. Cette image est disponible dans la Place de marché Azure. Enterprise Server peut également utiliser Azure SQL Database.

    -   **Enterprise Developer :** sélectionnez une **machine virtuelle B2ms** (avec 2 processeurs virtuels et 8 Go de mémoire) où sont installés Windows 10 et Visual Studio. Cette image est disponible dans la Place de marché Azure.

3.  Dans la section **Basics** (De base), entrez votre nom d’utilisateur et votre mot de passe. Sélectionnez l’abonnement (**Subscription**), ainsi que la région ou l’emplacement (**Location/Region**) à utiliser pour les machines virtuelles. Pour le groupe de ressources, sélectionnez **RGMicroFocusEntServer**.

4.  Placez les deux machines virtuelles sur le même réseau virtuel, afin qu’elles puissent communiquer entre elles.

5.  Acceptez les valeurs par défaut pour les autres paramètres. Notez le nom d’utilisateur et le mot de passe que vous avez créés pour l’administrateur de ces machines virtuelles.

6.  Une fois les machines virtuelles créées, ouvrez les ports entrants **9003, 86** et **80** pour HTTP, le port **3389** pour RDP (Remote Desktop Protocol) sur la machine Enterprise Server, et le port **3389** sur la machine Developer.

7.  Pour vous connecter à la machine virtuelle Enterprise Server, dans le portail Azure, sélectionnez la machine virtuelle ES2 v3. Accédez à la section **Overview** (Vue d’ensemble), puis sélectionnez **Connect** (Se connecter) pour lancer une session RDP. Connectez-vous en utilisant les informations d’identification utilisateur que vous avez créées pour la machine virtuelle.

8.  Dans la session RDP, chargez les deux fichiers suivants. Étant donné que vous utilisez Windows, vous pouvez glisser-déplacer les fichiers dans la session RDP :

    -   `es\_50.exe`, le fichier d’installation d’Enterprise Server.

    -   `mflic`, le fichier de licence correspondant (Enterprise Server ne peut pas se charger sans lui).

9.  Pour démarrer l’installation, double-cliquez sur le fichier. Dans la première fenêtre, sélectionnez l’emplacement d’installation et acceptez le contrat de licence d’utilisateur final.

    ![Capture d’écran montrant la boîte de dialogue Micro Focus Enterprise Server dans laquelle vous pouvez démarrer l’installation.](media/install-image-1.png)

    Lorsque l’installation est terminée, le message suivant s’affiche :

    ![Capture d’écran montrant un message de réussite dans la boîte de dialogue Micro Focus Enterprise Server.](media/install-image-2.png)

 ### <a name="check-for-updates"></a>Rechercher les mises à jour

Après l’installation, vérifiez si des mises à jour supplémentaires sont disponibles, car certains éléments prérequis, comme Microsoft C++ Redistributable et .NET Framework, sont installés en même temps qu’Enterprise Server.

### <a name="upload-the-license"></a>Charger la licence

1.  Démarrez Micro Focus License Administration.

2.  Sélectionnez **Start** (Démarrer) \> **Micro Focus License Manager** (Gestionnaire des licences Micro Focus) \> **License Administration** (Gestion des Licences), puis cliquez sur l’onglet **Install** (Installer). Choisissez le type de format de licence à charger : un fichier de licence ou un code de licence de 16 caractères. Par exemple, pour un fichier, dans **Fichier de licence**, accédez au fichier *`mflic` précédemment chargé dans la machine virtuelle, puis sélectionnez **Installer les licences**.

    ![Capture d’écran montrant la boîte de dialogue Micro Focus License Administration dans laquelle vous pouvez sélectionner Installer les licences.](media/install-image-3.png)

3.  Vérifiez qu’Enterprise Server se charge. Essayez de lancer le site Enterprise Server Administration à partir d’un navigateur à l’aide de cette URL : `http://localhost:86/`. La page Enterprise Server Administration s’affiche ainsi.

    ![Page Enterprise Server Administration](media/install-image-4.png)

## <a name="install-enterprise-developer-on-the-developer-machine"></a>Installer Enterprise Developer sur la machine de développement

1.  Sélectionnez le groupe de ressources créé précédemment (par exemple, **RGMicroFocusEntServer**), puis sélectionnez l’image Developer.

2.  Pour vous connecter à la machine virtuelle, accédez à la section **Overview** (Vue d’ensemble), puis sélectionnez **Connect** (Se connecter). Cette connexion lance une session RDP. Connectez-vous en utilisant les informations d’identification utilisateur que vous avez créées pour la machine virtuelle.

3.  Dans la session RDP, chargez les deux fichiers suivants (vous pouvez utiliser le glisser-déplacer) :

    -   `edvs2017.exe`, le fichier d’installation d’Enterprise Server.

    -   `mflic`, le fichier de licence correspondant (Enterprise Developer ne peut pas se charger sans lui).

4.  Pour démarrer l’installation, double-cliquez sur le fichier **edvs2017.exe**. Dans la première fenêtre, sélectionnez l’emplacement d’installation et acceptez le contrat de licence d’utilisateur final. Si vous le souhaitez, choisissez **Install Rumba 9.5** (Installer Rumba 9.5) pour installer cet émulateur de terminal dont vous aurez probablement besoin.

    ![Boîte de dialogue d’installation Micro Focus Enterprise Developer pour Visual Studio 2017](media/install-image-5.png)

5.  Lorsque l’installation est terminée, le message suivant s’affiche :

    ![Message de confirmation de l’installation](media/install-image-6.png)

6.  Démarrez Micro Focus License Manager comme vous l’avez fait pour Enterprise Server. Choisissez **Start** (Démarrer) \> **Micro Focus License Manager** (Gestionnaire des licences Micro Focus) \> **License Administration** (Gestion des Licences), puis cliquez sur l’onglet **Install** (Installer).

7.  Choisissez le type de format de licence à charger : un fichier de licence ou un code de licence de 16 caractères. Par exemple, pour un fichier, dans **License file** (Fichier de licence), accédez au fichier `mflic` précédemment chargé dans la machine virtuelle, puis sélectionnez **Install Licenses** (Installer les licences).

    ![Boîte de dialogue Micro Focus License Administration](media/install-image-7.png)

Si Enterprise Developer se charge, cela signifie que le déploiement de votre environnement de développement et de test Micro Focus sur Azure est terminé.

**Étapes suivantes**

-   [Configurer l’application Bank Demo](./demo.md)

-   [Exécuter Enterprise Server dans des conteneurs Docker](./run-enterprise-server-container.md)

-   [Migration d’applications mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/application-strategies)