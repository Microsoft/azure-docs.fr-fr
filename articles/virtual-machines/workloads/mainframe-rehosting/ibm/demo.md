---
title: Configurer une instance ADCD (Application Developers Controlled Distribution) dans IBM zD&T v1 | Microsoft Docs
description: Exécutez un environnement de développement et de test IBM Z (zD&T) sur des machines virtuelles Azure.
services: virtual-machines-linux
ms.service: virtual-machines-linux
ms.subservice: workloads
documentationcenter: ''
author: njray
manager: edprice
ms.author: edprice
editor: edprice
ms.topic: conceptual
ms.date: 02/22/2019
tags: ''
keywords: ''
ms.openlocfilehash: dcd354b906b4d6c92d8b3186fc8e09c94a31ca55
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968398"
---
# <a name="set-up-an-application-developers-controlled-distribution-adcd-in-ibm-zdt-v1"></a>Configurer une instance ADCD (Application Developers Controlled Distribution) dans IBM zD&T v1

Vous pouvez exécuter un environnement de développement et de test IBM Z (zD&T) sur des machines virtuelles Azure. Cet environnement émule l’architecture IBM série Z. Il peut héberger une variété de systèmes d’exploitation ou d’installations série Z (également appelés instances ou packages Z), qui sont accessibles via des offres groupées personnalisées appelées IBM Application Developers Controlled Distributions (ADCD).

Cet article vous montre comment configurer une instance ADCD dans un environnement zD&T sur Azure. Les ADCD créent des implémentations de système d’exploitation série Z complètes pour les environnements de développement et de test exécutés dans zD&T.

Comme zD&T, les ADCD sont accessibles uniquement aux clients et partenaires d’IBM et sont exclusivement conçus à des fins de développement et de test. Ils ne doivent ne pas être utilisés pour les environnements de production. De nombreux packages d’installation IBM sont disponibles en téléchargement via [Passport Advantage](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zsys.rdt.guide.adcd.doc/topics/installation_ps.html) ou [IBM PartnerWorld](https://www.ibm.com/partnerworld/public).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- [Environnement zD&T][ibm-install-z] configuré précédemment sur Azure. Cet article suppose que vous utilisez la même image de machine virtuelle Ubuntu 16.04 créée précédemment.

- Accédez au média ADCD via IBM PartnerWorld ou Passport Advantage.

- Un [serveur de licences](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zsys.rdt.tools.user.guide.doc/topics/zdt_ee.html). Il est nécessaire pour exécuter IBM zD&T. La façon dont vous le créez dépend de la façon dont vous avez acheté vos licences logicielles auprès d’IBM :

  - Le **serveur de licences matériel** nécessite une unité matérielle USB qui contient les jetons Rational nécessaires à l’accès à toutes les parties des logiciels. Vous devez vous la procurer auprès d’IBM.

  - Le **serveur de licences logiciel** vous impose de configurer un serveur centralisé pour la gestion des clés de licence. Il s’agit de la méthode conseillée. Elle vous contraint à configurer les clés que vous recevez d’IBM dans le serveur d’administration.

## <a name="download-the-installation-packages-from-passport-advantage"></a>Télécharger les packages d’installation sur Passport Advantage

Accédez au média ADCD requis. Les étapes ci-dessous supposent que vous êtes un client d’IBM et que vous pouvez utiliser Passport Advantage. Les partenaires d’IBM peuvent utiliser [IBM PartnerWorld](https://www.ibm.com/partnerworld/public).

> [!NOTE]
> Cet article part du principe qu’un PC Windows est utilisé pour accéder au Portail Azure et pour télécharger le média IBM. Si vous utilisez un ordinateur de bureau Mac ou Ubuntu, les commandes et le processus d’obtention du média IBM peuvent être légèrement différents.

1. Connectez-vous à [Passport Advantage](https://www.ibm.com/software/howtobuy/passportadvantage/paocustomer).

2. Sélectionnez **Software Downloads** (Téléchargements de logiciels) et **Media Access** (Accès aux médias).

3. Sélectionnez **Program offering and agreement number** (Offre de programme et numéro de contrat), puis cliquez sur **Continuer**.

4. Entrez la description ou le numéro de pièce, puis cliquez sur **Recherche**.

5. Vous pouvez éventuellement cliquer sur la liste par ordre alphabétique pour afficher le produit par nom.

6. Sélectionnez **Tous les systèmes d’exploitation** dans le champ **Système d’exploitation** et **Toutes les langues** dans le champ **Langues**. Cliquez ensuite sur **Atteindre**.

7. Cliquez sur **Select individual files** (Sélectionner des fichiers individuels) pour développer la liste et afficher le média individuel à télécharger.

8. Vérifiez le ou les packages que vous souhaitez télécharger, sélectionnez **Télécharger**, puis téléchargez les fichiers dans le répertoire souhaité.

## <a name="upload-the-adcd-packages"></a>Charger le ou les packages ADCD

Maintenant que vous disposez du ou des packages, vous devez les charger sur votre machine virtuelle sur Azure.

1. Dans le Portail Azure, ouvrez une session **SSH** avec la machine virtuelle Ubuntu que vous avez créée. Accédez à votre machine virtuelle, sélectionnez le panneau **Vue d’ensemble**, puis sélectionnez **Connecter**.

2. Sélectionnez l’onglet **SSH**, puis copiez la commande SSH dans le Presse-papiers.

3. Connectez-vous à votre machine virtuelle à l’aide de vos informations d’identification et le [client SSH](../../../linux/use-remote-desktop.md) choisi. Cette démonstration utilise les extensions Linux pour Windows 10, qui ajoutent un interpréteur de commandes bash à l’invite de commandes Windows. PuTTY fonctionne également correctement.

4. Une fois connecté, créez un répertoire pour charger les packages d’IBM. N’oubliez pas Linux respecte la casse. Par exemple, cette démonstration suppose que les packages sont chargés sur :

    `/home/MyUserID/ZDT/adcd/nov2017/volumes`

5. Chargez les fichiers à l’aide d’un client SSH tel que [WinSCP](https://winscp.net/eng/index.php). SCP faisant partie du protocole SSH, il utilise le port 22, qui est utilisé par SSH. Si votre ordinateur local n’est pas sous Windows, vous pouvez saisir la [command scp](http://man7.org/linux/man-pages/man1/scp.1.html) dans votre session SSH.

6. Lancez le chargement dans le répertoire de machine virtuelle Azure que vous avez créé, qui devient le stockage des images pour zD&T.

    > [!NOTE]
    > Assurez-vous que **ADCDTOOLS.XML** est inclus dans le chargement dans le répertoire **home/MyUserID/ZDT/adcd/nov2017**. Vous en aurez besoin ultérieurement.

7. Attendez la fin du chargement des fichiers, qui peut prendre du temps en fonction de votre connexion à Azure.

8. Lorsque les chargements sont terminés, accédez au répertoire de volumes et décompressez tous les volumes **gz** :

    ```console
    gunzip \*.gz
    ```
    
![Explorateur de fichiers montrant des volumes gz décompressés](media/01-gunzip.png)

## <a name="configure-the-image-storage"></a>Configurer le stockage des images

L’étape suivante consiste à configurer zD&T pour utiliser le ou les packages chargés. Le processus de stockage des images dans zD&T vous permet de monter et d’utiliser les images. Il peut utiliser le protocole SSH ou FTP.

1. Démarrez **zDTServer**. Pour cela, vous devez être au niveau racine. Entrez les deux commandes suivantes dans cet ordre :
    ```console
    sudo su -
    /opt/ibm/zDT/bin/startServer
    ```
2. Notez la sortie de l’URL par la commande et utilisez cette URL pour accéder au serveur web. Il ressemble à :
     > https://(nom ou adresse IP de votre machine virtuelle):9443/ZDTMC/index.html
     >
     > N’oubliez pas votre accès web utilise le port 9443. Utilisez-le pour vous connecter au serveur web. L’identifiant d’utilisateur pour ZD&T est **zdtadmin** et le mot de passe est **password**.

    ![Écran d’accueil d’IBM zD&T Enterprise Edition](media/02-welcome.png)

3. Dans la page **Démarrage rapide**, sous **Configurer**, sélectionnez **Stockage des images**.

     ![Écran de démarrage rapide d’IBM zD&T Enterprise Edition](media/03-quickstart.png)

4. Dans la page **Configure image storage** (Configurer le stockage des images), sélectionnez **SSH File Transfer Protocol**.

5. Pour **Nom d’hôte**, saisissez **Localhost** et entrez le chemin du répertoire où vous avez chargé les images. Par exemple, /home/MyUserID/ZDT/adcd/nov2017/volumes.

6. Entrez l’**identifiant d’utilisateur** et le **mot de passe** de la machine virtuelle. N’utilisez pas l’identifiant d’utilisateur et le mot de passe de ZD&T.

7. Testez la connexion pour vérifier que vous avez accès, puis sélectionnez **Enregistrer** pour enregistrer la configuration.

## <a name="configure-the-target-environments"></a>Configurer les environnements cibles

L’étape suivante consiste à configurer l’environnement cible zD&T. Cet environnement hébergé émulé est l’emplacement où vos images sont exécutées.

1. Dans la page **Démarrage rapide**, sous **Configurer**, sélectionnez **Target environments** (Environnements cibles).

2. Dans la page **Configure target environments** (Configurer des environnements cibles), sélectionnez **Ajouter une cible**.

3. Sélectionnez **Linux**. IBM prend en charge deux types d’environnements, Linux et Cloud(OpenStack), mais cette démonstration s’exécute sur Linux.

4. Dans la page **Add target environment** (Ajouter un environnement cible), pour **Nom d’hôte**, entrez **localhost**. Conservez le **port SSH** défini sur **22**.

5. Dans la zone **Target Environment label** (Étiquette d’environnement cible), entrez une étiquette comme **MyCICS**.

     ![Écran d’ajout d’environnement cible](media/04-add-target.png)

## <a name="configure-adcd-and-deploy"></a>Configurer ADCD et déployer

Lorsque les étapes de configuration précédentes sont terminées, vous devez configurer zD&T pour utiliser le ou les packages et l’environnement cible. Vous pouvez réutiliser le processus de stockage des images dans zD&T qui vous permet de monter et d’utiliser les images. Il peut utiliser le protocole SSH ou FTP.

1. Dans la page **Démarrage rapide**, sous **Configurer**, sélectionnez **ADCD**. Un ensemble d’instructions vous présentant les étapes à effectuer avant de pouvoir monter un package ADCD s’affiche. C’est la raison pour laquelle nous avons nommé le répertoire cible précédemment.

2. En supposant que toutes les images ont été chargées dans les répertoires appropriés, cliquez sur le lien **IMAGE from ADCD** affiché dans le coin inférieur droit (indiqué à l’étape 7 dans la capture d’écran suivante).

     ![IBM zD&T Enterprise Edition - Écran de configuration d’ADCD](media/05-adcd.png)

## <a name="create-the-image"></a>Création de l’image

Lorsque l’étape précédente de la configuration est terminée, la page **Create an image using ADCD Components** (Créer une image à l’aide de composants d’ADCD) s’affiche.

1. Sélectionnez le volume (Nov 2017 dans ce cas) pour afficher les différents packages qui se trouvent dans ce volume.

2. Pour cette démonstration, sélectionnez **Customer Information Control System (CICS) - 5.3**.

3. Dans la zone **Nom de l’image**, saisissez un nom pour l’image comme **MyCICS Image**.

4. Sélectionnez le bouton **Créer une image** bouton en bas à droite.

     ![IBM zD&T Enterprise Edition - Écran de création d’une image à l’aide de composants d’ADCD](media/06-adcd.png)

5. Dans la fenêtre qui s’affiche, qui vous indique que l’image a bien été déployée, choisissez **Deploy images** (Déployer des images).

6. Dans la page **Deploy an image to a target environment** (Déployer une image sur un environnement cible), sélectionnez l’image que vous avez créée dans la page précédente (**MyCICS Image**) et l’environnement cible créé précédemment (**MyCICS**).

7. Dans l’écran suivant, indiquez vos informations d’identification pour la machine virtuelle (autrement dit, pas les informations d’identification ztadmin).

8. Dans le volet Propriétés, entrez le nombre de **processeurs centraux (PC)** , la quantité de **mémoire système (en Go)** et le **répertoire de déploiement** pour l’image en cours d’exécution. Dans la mesure où il s’agit d’une démonstration, utilisez de faibles valeurs.

9. Assurez-vous que la case **Automatically issue IPL command to z/OS after deploy** (Exécuter automatiquement la commande IPL sur z/OS après le déploiement) est cochée.

     ![Écran des propriétés](media/07-properties.png)

10. Sélectionnez **Terminer**.

11. Sélectionnez **Déployer une image** dans la page **Deploy an image to a target environment** (Déployer une image sur un environnement cible).

Votre image peut maintenant être déployée et est prête à être montée par un émulateur de terminal 3270.

> [!NOTE]
> Si vous recevez une erreur indiquant que vous n’avez pas suffisamment d’espace disque, notez que la région nécessite 151 Go.

Félicitations ! Vous exécutez à présent un environnement de mainframe IBM sur Azure.

## <a name="learn-more"></a>En savoir plus

- [Migration de mainframe : mythes et réalités](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/myths-and-facts)
- [IBM DB2 pureScale sur Azure](../../../linux/ibm-db2-purescale-azure.md)
- [Dépannage](../../../troubleshooting/index.yml)
- [Démystifier la migration d’un mainframe vers Azure](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration/)

<!-- INTERNAL LINKS -->
[microfocus-get-started]: /microfocus/get-started.md
[microfocus-setup]: /microfocus/set-up-micro-focus-on-azure.md
[microfocus-demo]: /microfocus/demo.md
[ibm-get-started]: /ibm/get-started.md
[ibm-install-z]: install-ibm-z-environment.md
[ibm-demo]: /ibm/demo.md
