---
title: Exécuter Micro Focus Enterprise Server 5.0 dans un conteneur Docker sur Azure | Microsoft Docs
description: Dans cet article, découvrez comment exécuter Micro Focus Enterprise Server 5.0 dans un conteneur Docker sur Microsoft Azure.
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
ms.openlocfilehash: f34767c160c8229eb5b63806924926a46ea00cc2
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93127193"
---
# <a name="run-micro-focus-enterprise-server-50-in-a-docker-container-on-azure"></a>Exécuter Micro Focus Enterprise Server 5.0 dans un conteneur Docker sur Azure

Vous pouvez exécuter Micro Focus Enterprise Server 5.0 dans un conteneur Docker sur Azure. Ce didacticiel vous explique les procédures. Il utilise la démonstration acctdemo CICS (Customer Information Control System) Windows pour Enterprise Server.

Docker ajoute la portabilité et l’isolation aux applications. Par exemple, vous pouvez exporter une image Docker d’une machine virtuelle Windows pour l’exécuter sur une autre ou d’un référentiel vers un serveur Windows avec Docker. L’image Docker s’exécute au nouvel emplacement avec la même configuration, sans devoir installer Enterprise Server. Il est inclus dans l’image. Les considérations relatives aux licences s’appliquent toujours.

Ce didacticiel installe la machine virtuelle **Windows 2016 Datacenter with Containers** à partir de la Place de marché Azure. Cette machine virtuelle inclut **Docker 18.09.0**. Les étapes ci-dessous vous montrent comment déployer le conteneur, l’exécuter, puis s’y connecter avec un émulateur 3270.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, consultez les prérequis suivants :

-   Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

-   Le logiciel Micro Focus et une licence valide (ou licence d’essai). Si vous êtes déjà client de Micro Focus, contactez votre représentant Micro Focus. Sinon, [demandez une version d’évaluation](https://www.microfocus.com/products/enterprise-suite/enterprise-server/trial/).

    > [!Note] 
    > Les fichiers de démonstration Docker sont inclus avec Enterprise Server 5.0. Ce didacticiel utilise ent\_server\_dockerfiles\_5.0\_windows.zip. Accédez-y du même endroit où vous avez accédé au fichier d’installation d’Enterprise Server ou accédez à *Micro Focus* pour commencer.

-   La documentation de [Enterprise Server and Enterprise Developer](https://www.microfocus.com/documentation/enterprise-developer/#%22).

## <a name="create-a-vm"></a>Créer une machine virtuelle

1.  Sécurisez le média du fichier ent\_server\_dockerfiles\_5.0\_windows.zip. Sécurisez le fichier de licence ES-Docker-Prod-XXXXXXXX.mflic (nécessaire pour créer les images Docker).

2.  Créez la machine virtuelle. Pour cela, ouvrez le Portail Azure, sélectionnez **Créer une ressource** dans le menu en haut à gauche et filtrez par *système d’exploitation serveur windows*. Dans les résultats, sélectionnez **Windows Server.** Dans l’écran suivant, sélectionnez **Windows Server 2016 Datacenter – with Containers**.

    ![Capture d’écran des résultats de la recherche du portail Azure](./media/run-image-1.png)

3.  Pour configurer les propriétés de la machine virtuelle, choisissez des détails d’instance :

    1.  Choisissez une taille de machine virtuelle. Pour ce didacticiel, envisagez d’utiliser une machine virtuelle **Standard DS2\_v3** avec 2 processeurs virtuels et 16 Go de mémoire.

    2.  Sélectionnez la **région** et le **groupe de ressources** sur lesquels vous souhaitez effectuer le déploiement.

    3.  Pour les **options de disponibilité** , utilisez le paramètre par défaut.

    4.  Pour le **nom d’utilisateur** , saisissez le compte administrateur que vous souhaitez utiliser et le mot de passe.

    5.  Assurez-vous que le **port 3389 RDP** est ouvert. Seul ce port doit être exposé publiquement pour que vous puissiez vous connecter à la machine virtuelle. Acceptez ensuite toutes les valeurs par défaut, puis cliquez sur **Vérifier + créer**.

    ![Capture d’écran du volet Créer une machine virtuelle](./media/run-image-2.png)

4.  Attendez que le déploiement se termine (quelques minutes). Un message indique que votre machine virtuelle a été créée.

5.  Sélectionnez **Accéder à la ressource** pour accéder au panneau **Vue d’ensemble** de votre machine virtuelle.

6.  À droite, sélectionnez **Connecter**. Les options **Se connecter à la machine virtuelle** apparaissent à droite.

7.  Sélectionnez le bouton **Téléchargement du fichier RDP** pour télécharger le fichier RDP qui vous permet d’être lié à la machine virtuelle.

8.  Lorsque le téléchargement du fichier est terminé, ouvrez-le et saisissez le nom d’utilisateur et le mot de passe que vous avez créés pour la machine virtuelle.

    > [!Note]    
    > N’utilisez pas vos informations d’identification d’entreprise pour vous connecter. (Le client RDP suppose que vous souhaitez les utiliser. Mais vous ne le voulez pas.)

9.  Sélectionnez **Autres choix** , puis sélectionnez les informations d’identification de votre machine virtuelle.

À ce stade, la machine virtuelle est en cours d’exécution et est jointe via RDP. Vous êtes connecté et prêt pour l’étape suivante.

## <a name="create-a-sandbox-directory-and-upload-the-zip-file"></a>Créer un répertoire de bac à sable et charger le fichier zip

1.  Créez un répertoire sur la machine virtuelle dans lequel vous pouvez télécharger les fichiers de démonstration et de licence. Par exemple, **C:\\Sandbox**.

2.  Chargez **ent\_server\_dockerfiles\_5.0\_windows.zip** et le fichier **ES-Docker-Prod-XXXXXXXX.mflic** dans le répertoire que vous avez créé.

3.  Extrayez le contenu du fichier zip dans le répertoire **ent\_server\_dockerfiles\_5.0\_windows** créé par le processus d’extraction. Ce répertoire inclut un fichier readme (aux formats .html et .txt) et deux sous-répertoires, **EnterpriseServer** et **Examples**.

4.  Copiez le fichier **ES-Docker-Prod-XXXXXXXX.mflic** dans les répertoires CICS C:\\Sandbox\\ent\_server\_dockerfiles\_5.0\_windows\\EnterpriseServer et C:\\Sandbox\\ent\_server\_dockerfiles\_5.0\_windows\\Examples\\.  
      
    > [!Note]
    > Assurez-vous de copier le fichier de licence dans les deux répertoires. Ils sont nécessaires dans l’étape de création Docker pour s’assurer que les images disposent des licences adéquates.

## <a name="check-docker-version-and-create-base-image"></a>Vérifier la version de Docker et créer l’image de base

> [!Important]  
> La création de l’image Docker appropriée est un processus en deux étapes. Créez tout d’abord l’image de base Enterprise Server 5.0. Puis créez une autre image pour la plateforme x64. Bien que vous puissiez créer une image x86 (32 bits), utilisez l’image 64 bits.

1.  Ouvrez une invite de commandes.

2.  Vérifiez que Docker est installé. À l'invite de commandes, tapez **version de docker**  
    Par exemple, la version était 18.09.0 au moment de l’écriture.

3.  Pour modifier le répertoire, tapez :  
    **cd \\Sandbox\\ent\_server\_dockerfiles\_5.0\_windows\\EnterpriseServer**.

4.  Saisissez **bld.bat IacceptEULA** pour commencer le processus de création de l’image de base initiale. Attendez quelques minutes pour que ce processus s’exécute. Dans les résultats, notez les deux images qui ont été créées : une pour x64 et l’autre pour x86 :

    ![Fenêtre Commande montrant les images](./media/run-image-3.png)

5.  Pour créer l’image finale pour la démonstration CICS, basculez vers le répertoire CICS en tapant **cd\\Sandbox\\ent\_server\_dockerfiles\_5.0\_windows\\Examples\\CICS**.

6.  Pour créer l’image, saisissez **bld.bat x64**. Attendez quelques minutes pour que le processus s’exécute et que le message indiquant que l’image a été créée s’affiche.

7.  Saisissez **images docker** pour afficher une liste de toutes les images Docker installées sur la machine virtuelle. Assurez-vous que **microfocus/es-acctdemo** est l’une d’entre elles.

    ![Fenêtre Commande montrant l’image es-acctdemo](./media/run-image-4.png)

## <a name="run-the-image"></a>Exécuter l’image

1.  Pour lancer Enterprise Server 5.0 et l’application acctdemo, à l’invite de commandes, tapez :

    ```
    **docker run -p 16002:86/tcp -p 16002:86/udp -p 9040-9050:9040-9050 -p 9000-9010:9000-9010 -ti --network="nat" --rm microfocus/es-acctdemo:win\_5.0\_x64
    ```

1.  Installez un émulateur de terminal 3270 tel que [x3270](http://x3270.bgp.nu/) et utilisez-le pour vous lier, via le port 9040, à l’image est en cours d’exécution.

2.  Obtenez l’adresse IP du conteneur acctdemo pour que Docker puisse agir comme un serveur DHCP pour les conteneurs qu’il gère :

    1.  Obtenez l’ID du conteneur en cours d’exécution. Saisissez **Docker ps** à l’invite de commandes et notez l’ID ( **22a0fe3159d0** dans cet exemple). Notez-le pour l’étape suivante.

    2.  Pour obtenir l’adresse IP du conteneur acctdemo, utilisez l’ID de conteneur de l’étape précédente comme suit :

    ```
    docker inspect \<containerID\> --format="{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}"
    ```

    Par exemple :

    ```
    docker inspect 22a0fe3159d0 --format="{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}"
    ```

4. Notez l’adresse IP de l’image acctdemo. Par exemple, l’adresse dans la sortie suivante est 172.19.202.52.

    ![Capture d’écran de la fenêtre Commande montrant l’adresse IP](./media/run-image-5.png)

5. Montez l’image à l’aide de l’émulateur. Configurez l’émulateur pour utiliser l’adresse de l’image acctdemo et le port 9040. Il s’agit ici de **172.19.202.52:9040**. La vôtre sera similaire. L’écran **Signon to CICS** (Se connecter à CICS) s’ouvre.

    ![Capture d’écran de Signon to CICS (Se connecter à CICS)](./media/run-image-6.png)

6. Connectez-vous à la région CICS en entrant **SYSAD** pour **USERID** et **SYSAD** pour **Mot de passe**.

7. Effacez l’écran à l’aide du mappage de touches de l’émulateur. Pour x3270, sélectionnez l’option de menu **Keymap** (Mappage de touches).

8. Pour lancer l’application acctdemo, saisissez **ACCT**. L’écran initial de l’application s’affiche.

     ![Capture d’écran représentant une fenêtre de console qui affiche l’application.](./media/run-image-7.png)

9. Faites des essais avec les types de compte d’affichage. Par exemple, saisissez **D** pour la requête et **11111** pour le **compte**. D’autres numéros de comptes à essayer sont 22222, 33333 et ainsi de suite.

    ![Capture d’écran montrant comment modifier différentes valeurs dans l’application.](./media/run-image-8.png)

10. Pour afficher la console d’administration d’Enterprise Server, accédez à l’invite de commandes et tapez **start http:172.19.202.52:86**.

    ![Console d’administration d’Enterprise Server](media/run-image-9.png)

Et voilà ! Vous exécutez et gérez maintenant une application CICS dans un conteneur Docker.

## <a name="next-steps"></a>Étapes suivantes

-   [Installer Micro Focus Enterprise Server 5.0 et Enterprise Developer 5.0 sur Azure](./set-up-micro-focus-azure.md)

-   [Migration d’applications mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/application-strategies)
