---
title: Créer une infrastructure pour un cluster sur des machines virtuelles Azure
description: Dans ce tutoriel, vous découvrez comment configurer l’infrastructure de machines virtuelles Azure pour exécuter un cluster Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: e9f62f944fff331bcf2dad1b380161e563614219
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90561838"
---
# <a name="tutorial-create-azure-vm-infrastructure-to-host-a-service-fabric-cluster"></a>Tutoriel : Créer l’infrastructure de machines virtuelles Azure pour héberger un cluster Service Fabric

Les clusters autonomes Service Fabric vous permettent de choisir votre propre environnement et de créer un cluster dans le cadre de l’approche « n’importe quel SE, n’importe quel cloud » suivie par Service Fabric. Dans le cadre de cette série de tutoriels, vous créez un cluster autonome hébergé sur des machines virtuelles Azure et vous installez une application sur celui-ci.

Ce tutoriel est la première partie d’une série d’étapes. Dans cet article, vous générez les ressources de machines virtuelles Azure nécessaires pour héberger votre cluster autonome de Service Fabric. Pour les prochains articles, vous devez installer la suite autonome Service Fabric, installer un exemple d’application dans votre cluster et supprimer votre cluster.

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créer un ensemble d’instances de machines virtuelles Azure
> * Modifier le groupe de sécurité
> * Vous connecter à une des instances
> * Préparer l’instance pour Service Fabric

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer d’un abonnement Azure.  Si vous n’avez pas encore de compte, accédez au [portail Azure](https://portal.azure.com) pour en créer un.

## <a name="create-azure-virtual-machine-instances"></a>Créer des instances de machines virtuelles Azure

1. Connectez-vous au portail Azure et sélectionnez **Machines virtuelles** (et non pas « Machines virtuelles (classiques) »).

   ![Machine virtuelle du portail Azure][az-console]

2. Sélectionnez le bouton **Ajouter** pour ouvrir le formulaire **Créer une machine virtuelle**.

3. Sous l’onglet **Général**, veillez à choisir l’abonnement et le groupe de ressources souhaités (l’utilisation d’un nouveau groupe de ressources est recommandée).

4. Changez le type d’**Image** en **Windows Server 2016 Datacenter**. 
 
5. Changez la **Taille** de l’instance en **Standard DS2 v2**. Définissez un **Nom d’utilisateur** et un **Mot de passe** pour l’administrateur, et prenez-en note.

6. Laissez les **Règles des ports d’entrée** bloquées pour le moment ; nous les configurerons dans la section suivante.

7. Sous l’onglet **Réseau**, créez un **Réseau virtuel** et prenez note de son nom.

8. Ensuite, définissez le **Groupe de sécurité réseau de la carte réseau** sur **Avancé**. Créer un groupe de sécurité, notez son nom et créez les règles suivantes pour autoriser le trafic TCP depuis n’importe quelle source :

   ![Capture d’écran montrant la création de règles pour autoriser le trafic TCP entrant.][sf-inbound]

   * Port `3389` pour RDP et ICMP (connectivité de base).
   * Ports `19000-19003` pour Service Fabric.
   * Ports `19080-19081` pour Service Fabric.
   * Port `8080`, pour les demandes du navigateur web.

   > [!TIP]
   > Pour connecter vos machines virtuelles ensemble dans Service Fabric, celles qui hébergent votre infrastructure doivent avoir les mêmes informations d’identification.  Deux méthodes courantes vous permettent d’obtenir des informations d’identification homogènes : joindre toutes les machines virtuelles au même domaine ou définir le même mot de passe administrateur sur chacune des machines virtuelles. Heureusement, Azure permet à toutes les machines virtuelles sur le même **réseau virtuel** de se connecter facilement : nous sommes donc assurés d’avoir toutes nos instances sur le même réseau.

9. Ajoutez une autre règle. Définissez la source sur **Étiquette du Service** et définissez l’étiquette du service source sur **VirtualNetwork**. Service Fabric nécessite que les ports suivants soient ouverts pour la communication au sein du cluster : 135,137-139,445,20001-20031,20606-20861.

   ![Capture d’écran montrant la création de règles autorisant le trafic TCP pour un cluster.][vnet-inbound]

10. Le reste des options peuvent être acceptées dans leur état par défaut. Passez-les en revue si vous le souhaitez, puis lancez votre machine virtuelle.

## <a name="creating-more-instances-for-your-service-fabric-cluster"></a>Création de plus d’instances pour votre cluster Service Fabric

Lancez deux autres **machines virtuelles** en veillant à conserver les mêmes paramètres que ceux décrits dans la section précédente. En particulier, conservez le même nom d’utilisateur et le même mot de passe pour l’administrateur. Le **Réseau virtuel** et le **Groupe de sécurité réseau de la carte réseau** ne doivent pas être recréés ; sélectionnez ceux que vous avez déjà créés dans le menu déroulant. Le déploiement de chacune de vos instances peut prendre quelques minutes.

## <a name="connect-to-your-instances"></a>Se connecter à vos instances

1. Sélectionnez une de vos instances dans la section **Machine virtuelle**.

2. Sous l’onglet **Vue d’ensemble**, prenez note de l’adresse IP *privée*. Ensuite, cliquez sur **Se connecter**.

3. Sous l’onglet **RDP**, notez que nous utilisons l’adresse IP publique et le port 3389, que nous avons ouvert spécifiquement. Téléchargez le fichier RDP.
 
4. Ouvrez le fichier RDP et, quand vous y êtes invité, entrez le nom d’utilisateur et le mot de passe que vous avez spécifiés dans la configuration de la machine virtuelle.

5. Une fois que vous êtes connecté à une instance, vous devez vérifier que le Registre distant fonctionne, activer le protocole SMB, et ouvrir les ports nécessaires pour le protocole SMB et le Registre distant.

   Pour activer le protocole SMB, voici la commande PowerShell :

   ```powershell
   netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
   ```

6. Pour ouvrir les ports dans le pare-feu, voici la commande PowerShell :

   ```powershell
   New-NetFirewallRule -DisplayName "Service Fabric Ports" -Direction Inbound -Action Allow -RemoteAddress LocalSubnet -Protocol TCP -LocalPort 135, 137-139, 445
   ```

7. Répétez ce processus pour vos autres instances, en notant à nouveau les adresses IP privées.

## <a name="verify-your-settings"></a>Vérifier vos paramètres

1. Pour valider la connectivité de base, connectez-vous à une des machines virtuelles en utilisant RDP.

2. Ouvrez l’**invite de commandes** à partir de cette machine virtuelle, puis utilisez la commande ping pour vous connecter d’une machine virtuelle à une autre, en remplaçant l’adresse IP ci-dessous par une des adresses IP privées que vous avez notées précédemment (pas l’adresse IP de la machine virtuelle à laquelle vous êtes déjà connecté).

   ```
   ping 172.31.20.163
   ```

   Si votre sortie ressemble à `Reply from 172.31.20.163: bytes=32 time<1ms TTL=128` répété quatre fois, la connexion établie entre les instances fonctionne.

3. Vérifiez maintenant que votre partage SMB fonctionne avec la commande suivante :

   ```
   net use * \\172.31.20.163\c$
   ```

   Elle doit renvoyer `Drive Z: is now connected to \\172.31.20.163\c$.` comme sortie.


   Vos instances sont maintenant correctement préparées pour Service Fabric.

## <a name="next-steps"></a>Étapes suivantes

Dans la première partie de la série, vous avez découvert comment démarrer trois instances de machines virtuelles Azure et comment les configurer pour l’installation de Service Fabric :

> [!div class="checklist"]
> * Créer un ensemble d’instances de machines virtuelles Azure
> * Modifier le groupe de sécurité
> * Vous connecter à une des instances
> * Préparer l’instance pour Service Fabric

Passez à la deuxième partie de la série pour configurer Service Fabric sur votre cluster.

> [!div class="nextstepaction"]
> [Installer Service Fabric](service-fabric-tutorial-standalone-create-service-fabric-cluster.md)

<!-- IMAGES -->
[az-console]: ./media/service-fabric-tutorial-standalone-azure-create-infrastructure/az-console.png
[sf-inbound]: ./media/service-fabric-tutorial-standalone-azure-create-infrastructure/sf-inbound.png
[vnet-inbound]: ./media/service-fabric-tutorial-standalone-azure-create-infrastructure/vnet-inbound.png
