---
title: Créer une infrastructure pour un cluster sur AWS
description: Dans ce didacticiel, vous apprenez à configurer l’infrastructure AWS pour exécuter un cluster Service Fabric.
ms.topic: tutorial
ms.date: 05/11/2018
ms.custom: mvc
ms.openlocfilehash: c7a18b0dcdc04bdf66ac4b36ce7376ee018eb238
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842901"
---
# <a name="tutorial-create-aws-infrastructure-to-host-a-service-fabric-cluster"></a>Didacticiel : Créer l’infrastructure AWS pour héberger un cluster Service Fabric

Les clusters autonomes Service Fabric vous permettent de choisir votre propre environnement et de créer un cluster dans le cadre de l’approche « n’importe quel SE, n’importe quel cloud » suivie par Service Fabric. Dans le cadre de cette série de tutoriels, vous créez un cluster autonome hébergé sur AWS et vous installez une application dans celui-ci.

Ce tutoriel est la première partie d’une série d’étapes. Dans cet article, vous générez les ressources AWS nécessaires pour héberger votre cluster autonome de Service Fabric. Pour les prochains articles, vous devez installer la suite autonome Service Fabric, installer un exemple d’application dans votre cluster et supprimer votre cluster.

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créer un ensemble d’instances EC2
> * Modifier le groupe de sécurité
> * Se connecter à l’une des instances
> * Préparer l’instance pour Service Fabric

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre ce didacticiel, vous avez besoin d’un compte AWS.  Si vous n’en avez pas, accédez à la [console AWS](https://aws.amazon.com/) pour en créer un.

## <a name="create-ec2-instances"></a>Créer des instances EC2

Connectez-vous à la console AWS > entrez **EC2** dans la zone de recherche > **EC2 Virtual Servers in the Cloud** (Serveurs virtuels EC2 dans le cloud).

![Recherche dans la console AWS][aws-console]

Sélectionnez **Launch Instance** (Démarrer une instance), puis dans l’écran suivant choisissez **Sélectionner** en regard de Microsoft Windows Server 2016 Base.

![Sélection de l’instance EC2][aws-ec2instance]

Sélectionnez **t2.medium**, puis **Next: Configure Instance Details** (Suivant : configurer les détails de l’instance). Sur l’écran suivant, modifiez le nombre d’instances sur `3`, puis sélectionnez **Détails avancés** pour développer cette section.

Pour connecter vos machines virtuelles ensemble dans Service Fabric, celles qui hébergent votre infrastructure doivent avoir les mêmes informations d’identification.  Deux méthodes courantes vous permettent d’obtenir des informations d’identification homogènes : joindre toutes les machines virtuelles au même domaine ou définir le même mot de passe administrateur sur chacune des machines virtuelles.  Pour ce didacticiel, vous utilisez un script de données utilisateur pour définir les instances EC2 afin qu’elles aient toutes le même mot de passe.  Dans un environnement de production, il est plus sûr de joindre les hôtes à un domaine Windows.

Entrez le script suivant dans le champ de données utilisateur sur la console :

```powershell
<powershell>
$user = [adsi]"WinNT://localhost/Administrator,user"
$user.SetPassword("serv1ceF@bricP@ssword")
$user.SetInfo()
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
New-NetFirewallRule -DisplayName "Service Fabric Ports" -Direction Inbound -Action Allow -RemoteAddress LocalSubnet -Protocol TCP -LocalPort 135, 137-139, 445
</powershell>
```

Une fois que vous avez entré le script PowerShell, sélectionnez **Review and Launch** (Examiner et démarrer).

![Examiner et démarrer EC2][aws-ec2configure2]

Sur l’écran vous permettant d’examiner l’instance, sélectionnez **Démarrer**.  Choisissez ensuite l’option **Proceed without a key pair** (Continuer sans paire de clés) dans la liste déroulante, et cochez la case indiquant que vous connaissez le mot de passe.

![Sélection de la paire de clés AWS][aws-keypair]

Enfin, sélectionnez **Launch Instances** (Démarrer les instances), puis **Afficher les instances**.  La base de votre cluster Service Fabric étant créée, vous devez désormais ajouter quelques configurations finales aux instances pour les préparer à la configuration Service Fabric.

## <a name="modify-the-security-group"></a>Modifier le groupe de sécurité

Service Fabric a besoin d’un certain nombre de ports ouverts entre les hôtes de votre cluster. Pour ouvrir ces ports dans l’infrastructure AWS, sélectionnez une des instances que vous avez créées. Sélectionnez ensuite le nom du groupe de sécurité, par exemple **launch-wizard-1**. À présent, sélectionnez l’onglet **Entrant**.

Pour éviter d’ouvrir ces ports à tout le monde, vous devez les ouvrir uniquement pour les hôtes appartenant au même groupe de sécurité. Notez l’ID de groupe de sécurité ; dans l’exemple il s’agit de **sg-c4fb1eba**.  Sélectionnez ensuite **Modifier**.

Puis, ajoutez quatre règles au groupe de sécurité pour les dépendances du service et trois autres pour Service Fabric. La première règle doit autoriser le trafic ICMP pour les vérifications de la connectivité de base. Les autres règles ouvrent les ports requis pour activer le protocole SMB et le registre à distance.

Pour la première règle, sélectionnez **Ajouter une règle**, puis choisissez **All ICMP - IPv4** (Tout le trafic ICMP - IPv4) dans le menu de liste déroulante. Sélectionnez la zone de saisie en regard de Personnalisée et entrez l’ID de groupe de sécurité indiqué plus haut.

Pour les trois dernières dépendances, vous devez suivre un processus similaire.  Sélectionnez **Ajouter une règle**, puis choisissez **Custom TCP Rule** (Règle TCP personnalisée) dans la liste déroulante. Ensuite, dans la plage de ports, entrez une plage de `135`, `137-139` et `445` pour chaque règle. Enfin, dans la zone Source, entrez votre ID de groupe de sécurité.

![Ports du groupe de sécurité][aws-ec2securityports]

Maintenant que les ports pour les dépendances sont ouverts, vous devez faire la même chose pour les ports utilisés par Service Fabric pour communiquer. Sélectionnez **Ajouter une règle**, puis choisissez **Custom TCP Rule** (Règle TCP personnalisée) dans la liste déroulante. Ensuite, dans la plage de ports, entrez une plage de `20001-20031`. Enfin, entrez le groupe de sécurité dans la zone Source.

Ensuite, ajoutez une règle pour la plage de ports éphémère.  Sélectionnez **Ajouter une règle**, puis choisissez **Custom TCP Rule** (Règle TCP personnalisée) dans la liste déroulante. Ensuite, dans la plage de ports, entrez une plage de `20606-20861`. Enfin, dans la zone Source, entrez votre ID de groupe de sécurité.

Pour les deux dernières règles de Service Fabric, ouvrez-les à tout le monde pour que vous puissiez gérer votre cluster Service Fabric depuis votre ordinateur personnel. Sélectionnez **Ajouter une règle**, puis choisissez **Custom TCP Rule** (Règle TCP personnalisée) dans la liste déroulante. Ensuite, dans la plage de ports, entrez une plage de `19000-19003` et de `19080-19081`. Puis, sélectionnez la valeur N’importe où dans la liste déroulante Source.

Enfin, nous devons simplement ouvrir le port 8080 pour que vous puissiez voir l’application quand elle est déployée. Sélectionnez **Ajouter une règle**, puis choisissez **Custom TCP Rule** (Règle TCP personnalisée) dans la liste déroulante. Ensuite, dans la plage de ports, entrez une plage de `8080`. Puis, sélectionnez la valeur N’importe où dans la liste déroulante Source.

Toutes les règles sont désormais entrées. Sélectionnez **Enregistrer**.

## <a name="connect-to-an-instance-and-validate-connectivity"></a>Se connecter à une instance et valider la connectivité

Dans l’onglet Sécurité de groupe, sélectionnez **Instances** dans le menu de gauche.  Sélectionnez chacune des instances que vous avez créées et notez leurs adresses IP privées. Pour les exemples ci-dessous, nous allons utiliser `172.31.21.141` et `172.31.20.163`.

Une fois que vous avez toutes les adresses IP, sélectionnez l’une des instances à laquelle vous connecter, cliquez avec le bouton droit sur l’instance et sélectionnez **Connecter**.  Ici, vous pouvez télécharger le fichier RDP pour cette instance particulière.  Sélectionnez **Download Remote Desktop File** (Télécharger le fichier Bureau à distance), puis ouvrez le fichier téléchargé pour établir votre connexion Bureau à distance (RDP) à cette instance.  Entrez votre mot de passe `serv1ceF@bricP@ssword` lorsque vous y êtes invité.

![Télécharger le fichier Bureau à distance][aws-rdp]

Une fois que vous êtes connecté à votre instance, vérifiez que vous pouvez les connecter entre elles et que vous pouvez partager des fichiers.  Vous avez collecté les adresses IP de toutes les instances, sélectionnez celle à laquelle vous n’êtes pas connecté actuellement. Accédez à **Démarrer**, entrez `cmd` et sélectionnez **Invite de commandes**.

Dans ces exemples, la connexion RDP a été établie vers l’adresse IP suivante : 172.31.21.141. L’ensemble des tests de connectivité se produisent ensuite sur l’autre adresse IP : 172.31.20.163.

Pour valider que cette connectivité de base fonctionne, utilisez la commande ping.

```
ping 172.31.20.163
```

Si votre sortie ressemble à `Reply from 172.31.20.163: bytes=32 time<1ms TTL=128` répété quatre fois, la connexion établie entre les instances fonctionne.  Vérifiez maintenant que votre partage SMB fonctionne avec la commande suivante :

```
net use * \\172.31.20.163\c$
```

Elle doit renvoyer `Drive Z: is now connected to \\172.31.20.163\c$.` comme sortie.

## <a name="prep-instances-for-service-fabric"></a>Préparer les instances pour Service Fabric

Si vous avez commencé de zéro, vous allez devoir suivre quelques étapes supplémentaires.  Vous allez devoir vérifier que le registre à distance fonctionnait, activer le protocole SMB et ouvrir les ports requis pour le protocole SMB et le registre à distance.

Pour simplifier cela, vous avez intégré tout ce travail lorsque vous avez démarré les instances avec votre script de données utilisateur.

Pour activer le protocole SMB, voici la commande PowerShell que vous avez utilisée :

```powershell
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

Pour ouvrir les ports dans le pare-feu, voici la commande PowerShell :

```powershell
New-NetFirewallRule -DisplayName "Service Fabric Ports" -Direction Inbound -Action Allow -RemoteAddress LocalSubnet -Protocol TCP -LocalPort 135, 137-139, 445
```

## <a name="next-steps"></a>Étapes suivantes

Dans la première partie de la série, vous avez appris comment démarrer trois instances EC2 et comment les configurer pour l’installation de Service Fabric :

> [!div class="checklist"]
> * Créer un ensemble d’instances EC2
> * Modifier le groupe de sécurité
> * Se connecter à l’une des instances
> * Préparer l’instance pour Service Fabric

Passez à la deuxième partie de la série pour configurer Service Fabric sur votre cluster.

> [!div class="nextstepaction"]
> [Installer Service Fabric](service-fabric-tutorial-standalone-create-service-fabric-cluster.md)

<!-- IMAGES -->
[aws-console]: ./media/service-fabric-tutorial-standalone-cluster/aws-console.png
[aws-ec2instance]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2instance.png
[aws-ec2configure2]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2configure2.png
[aws-rdp]: ./media/service-fabric-tutorial-standalone-cluster/aws-rdp.png
[aws-ec2securityports]: ./media/service-fabric-tutorial-standalone-cluster/aws-ec2securityports.png
[aws-keypair]: ./media/service-fabric-tutorial-standalone-cluster/aws-keypair.png