---
title: Installer l’environnement de développement/test IBM zD&T sur Azure | Microsoft Docs
description: Déployez l’environnement de développement et de test IBM zD&T sur l’infrastructure Machine virtuelle Azure en tant que service IaaS (Infrastructure as a Service).
services: virtual-machines-linux
ms.service: virtual-machines-linux
ms.subservice: workloads
documentationcenter: ''
author: njray
ms.author: edprice
manager: edprice
editor: edprice
ms.topic: conceptual
ms.date: 04/02/2019
tags: ''
keywords: ''
ms.openlocfilehash: c12a812f66345a26701b2a1623487fd262ede4be
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968330"
---
# <a name="install-ibm-zdt-devtest-environment-on-azure"></a>Installer l’environnement de développement/test IBM zD&T sur Azure

Pour créer un environnement de développement/test pour les charges de travail mainframe sur des systèmes IBM Z, vous pouvez déployer IBM zD&T (Z Development and Test Environment ) sur l’infrastructure Machine virtuelle Azure en tant que service IaaS (Infrastructure as a Service).

Avec zD&T, vous pouvez tirer parti de la plateforme économique x86 pour vos environnements moins vitaux que sont les environnements de développement et de test et renvoyer ensuite (push) les mises à jour à un environnement de production Z System. Pour plus d’informations, consultez les [instructions d’installation d’IBM ZD&T](https://www-01.ibm.com/support/docview.wss?uid=swg24044565#INSTALL).

Azure et Azure Stack prennent en charge les versions suivantes :

- zD&T Personal Edition
- zD&T Parallel Sysplex
- zD&T Enterprise Edition

Toutes les éditions de zD&T s’exécutent uniquement sur des systèmes x86 Linux, et non sur Windows Server. La version Enterprise Edition est prise en charge sur Red Hat Enterprise Linux (RHEL) ou Ubuntu/Debian. Il existe des images de machine virtuelle RHEL et Debian pour Azure.

Cet article vous montre comment configurer zD&T Enterprise Edition sur Azure en vue d’utiliser le serveur web zD&T Enterprise Edition pour la création et la gestion d’environnements. L’installation de zD&T n’installe aucun environnement. Vous devez les créer séparément comme packages d’installation. Par exemple, les ADCD (Application Developers Controlled Distributions) sont des images de volume d’environnements de test. Elles se présentent sous forme d’images zip sur le média distribué par IBM. Découvrez comment [configurer un environnement ADCD sur Azure](demo.md).

Pour plus d’informations, consultez la [présentation de zD&T](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zdt.overview.gs.doc/topics/c_product_overview.html) sur le site IBM Knowledge Center.

Cet article vous montre comment configurer zD&T (Z Development and Test Environment) Enterprise Edition sur Azure. Vous pouvez ensuite utiliser le serveur web zD&T Enterprise Edition pour créer et gérer des environnements Z sur Azure.

## <a name="prerequisites"></a>Prérequis

> [!NOTE]
> IBM n’autorise l’installation de zD&T Enterprise Edition que dans des environnements de développement/test, et *non* dans des environnements de production.

- Un abonnement Azure. Si vous n’en avez pas, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Vous devez avoir accès au média, qui s’adresse uniquement aux clients et partenaires d’IBM. Pour plus d’informations, contactez votre représentant IBM ou procurez-vous les informations de contact sur le site web [zD&T](https://www.ibm.com/us-en/marketplace/z-systems-development-test-environment).

- Un [serveur de licences](https://www.ibm.com/support/knowledgecenter/en/SSTQBD_12.0.0/com.ibm.zsys.rdt.tools.user.guide.doc/topics/zdt_ee.html). Il s’agit d’une condition d’accès aux environnements. La façon dont vous le créez dépend de la façon dont vous avez acheté vos licences logicielles auprès d’IBM :

     - Le **serveur de licences matériel** nécessite une unité matérielle USB qui contient les jetons Rational nécessaires à l’accès à toutes les parties des logiciels. Vous devez vous la procurer auprès d’IBM.

     - Le **serveur de licences logiciel** vous impose de configurer un serveur centralisé pour la gestion des clés de licence. Il s’agit de la méthode conseillée. Elle vous contraint à configurer les clés que vous recevez d’IBM dans le serveur d’administration.

## <a name="create-the-base-image-and-connect"></a>Créer l’image de base et se connecter

1. Sur le portail Azure, [créez une machine virtuelle](../../../linux/quick-create-portal.md) avec la configuration de système d’exploitation de votre choix. Dans cet article, la configuration utilisée est une machine virtuelle B4ms (avec 4 processeurs virtuels et 16 Go de mémoire) exécutant Ubuntu 16.04.

2. Une fois la machine virtuelle créée, ouvrez les ports entrants 22 pour SSH, 21 pour FTP et 9443 pour le serveur web.

3. Obtenez les informations d’identification SSH figurant dans le panneau **Vue d’ensemble** de la machine virtuelle via le bouton **Connecter**. Sélectionnez l’onglet **SSH** et copiez la commande SSH logon dans le Presse-papiers.

4. Ouvrez une session sur un [shell Bash](../../../../cloud-shell/quickstart.md) à partir de votre PC local et collez la commande. Elle se présente sous la forme **ssh\<user id\>\@\<IP Address\>** . Quand vous y êtes invité, entrez vos informations d’identification pour établir une connexion à votre répertoire de base.

## <a name="copy-the-installation-file-to-the-server"></a>Copier le fichier d’installation sur le serveur

Le fichier d’installation du serveur web se nomme **ZDT\_Install\_EE\_V12.0.0.1.tgz**. Il se trouve sur le média fourni par IBM. Vous devez charger ce fichier sur votre machine virtuelle Ubuntu.

1. Sur la ligne de commande, entrez la commande suivante pour faire en sorte que tout ce qui se trouve dans l’image nouvellement créée soit à jour :

    ```
    sudo apt-get update
    ```

2. Créez le répertoire d’installation suivant :

    ```
    mkdir ZDT
    ```

3. Copiez le fichier de votre ordinateur local vers la machine virtuelle :

    ```
    scp ZDT_Install_EE_V12.0.0.1.tgz  your_userid@<IP Address /ZDT>   =>
    ```
    
> [!NOTE]
> Cette commande copie le fichier d’installation dans le répertoire ZDT de votre répertoire de base, qui varie selon que votre client exécute Windows ou Linux.

## <a name="install-the-enterprise-edition"></a>Installer Enterprise Edition

1. Accédez au répertoire ZDT et décompressez le fichier ZDT\_Install\_EE\_fichier V12.0.0.1.tgz en utilisant les commandes suivantes :

    ```
    cd ZDT
    chmod 755 ZDT\_Install\_EE\_V12.0.0.0.tgz
    ```

2. Exécutez le programme d’installation :

    ```
    ./ZDT_Install_EE_V12.0.0.0.x86_64
    ```

3. Sélectionnez **1** pour installer Enterprise Server.

4. Appuyez sur **Entrée** et lisez attentivement les contrats de licence. À la fin de la licence, entrez **Oui** pour continuer.

5. Quand vous êtes invité à modifier le mot de passe pour l’utilisateur nouvellement créé, **ibmsys1**, utilisez la commande **sudo passwd ibmsys1** et entrez le nouveau mot de passe.

6. Pour vérifier que l’installation a abouti, entrez

    ```
    dpkg -l | grep zdtapp
    ```

7. Vérifiez que la sortie contient la chaîne **zdtapp 12.0.0.0**, ce qui indique que le package a bien été installé.

### <a name="starting-enterprise-edition"></a>Démarrage d’Enterprise Edition

N’oubliez pas qu’au moment de démarrer, le serveur web s’exécute sous l’ID d’utilisateur zD&T qui a été créé au cours de l’installation.

1. Pour démarrer le serveur web, utilisez l’ID utilisateur racine pour exécuter la commande suivante :

    ```
    sudo /opt/ibm/zDT/bin/startServer
    ```

2. Copiez l’URL générée par le script, qui se présente comme ceci :

    ```
    https://<your IP address or domain name>:9443/ZDTMC/login.htm
    ```

3. Collez l’URL dans un navigateur web pour ouvrir le composant de gestion de votre installation zD&T.

## <a name="next-steps"></a>Étapes suivantes

[Configurer une instance ADCD (Application Developers Controlled Distribution) dans IBM zD&T v1](./demo.md)
