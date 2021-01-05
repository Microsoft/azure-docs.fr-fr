---
title: Configurer un labo de création de scripts shell avec Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un labo pour enseigner la création de scripts shell sur Linux.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: ba14f2fb5263367014b57741c78d6e509df044b3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85444962"
---
# <a name="set-up-a-lab-to-teach-shell-scripting-on-linux"></a>Configurer un labo pour enseigner la création de scripts shell sur Linux
Cet article vous montre comment configurer un labo pour enseigner la création de scripts shell sur Linux. Dans le cadre de l’administration système, l’écriture de scripts permet aux administrateurs d’éviter les tâches répétitives. Dans cet exemple de scénario, les scripts bash traditionnels et les scripts améliorés sont abordés. Les scripts améliorés sont des scripts qui associent des commandes bash et Ruby. Cette approche permet à Ruby de passer des données et fournit des commandes bash pour interagir avec le shell. 

Les étudiants qui suivent ces cours d’écriture de scripts disposent d’une machine virtuelle Linux pour apprendre les bases de Linux et également se familiariser avec les scripts de shell bash. La machine virtuelle Linux est fournie avec un accès au Bureau à distance activé. En outre, les éditeurs de texte [gedit](https://help.gnome.org/users/gedit/stable/) et [Visual Studio Code](https://code.visualstudio.com/) y sont installés.

## <a name="lab-configuration"></a>Configuration du laboratoire
Pour configurer ce labo, vous avez besoin d’un abonnement Azure pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services ou utiliser un compte lab existant. Pour créer un compte lab, consultez le tutoriel suivant : [Tutoriel pour configurer un compte lab](tutorial-setup-lab-account.md).

Après avoir créé le compte lab, activez les paramètres suivants dans celui-ci : 

| Paramètres du compte lab | Instructions |
| ----------- | ------------ |  
| Images de la Place de marché | Activez l’image Ubuntu Server 18.04 LTS pour l’utiliser dans votre compte lab. Pour plus d’informations, consultez [Spécifier les images de la Place de Marché accessibles aux créateurs de labo](specify-marketplace-images.md). | 

Suivez [ce tutoriel](tutorial-setup-classroom-lab.md) pour créer un labo et appliquer les paramètres suivants :

| Paramètres du labo | Valeur/instructions | 
| ------------ | ------------------ |
| Taille de machine virtuelle | Petite  |
| Image de machine virtuelle | Ubuntu Server 18.04 LTS|
| Activer la connexion Bureau à distance | Activer. <p>L’activation de ce paramètre permet aux enseignants et aux étudiants de se connecter à leurs machines virtuelles à l’aide du Bureau à distance (RDP). Pour plus d’informations, consultez [Activer le Bureau à distance pour les machines virtuelles Linux dans un labo dans Azure Lab Services](how-to-enable-remote-desktop-linux.md). </p>|

## <a name="install-desktop-and-rdp"></a>Installer le Bureau et RDP
Aucun serveur Bureau à distance RDP n’est installé par défaut sur l’image Ubuntu Server 18.04 LTS. Suivez les instructions de l’article [Installer et configurer le Bureau à distance pour effectuer une connexion à une machine virtuelle Linux dans Azure](../virtual-machines/linux/use-remote-desktop.md) afin d’installer les packages nécessaires sur la machine modèle pour se connecter via le protocole RDP (Remote Desktop Protocol).

## <a name="install-ruby"></a>Installer Ruby
Ruby est un langage dynamique open source qui peut être combiné avec des scripts bash. Cette section montre comment utiliser `apt-get` pour installer la dernière version de [Ruby](https://www.ruby-lang.org/).

1. Installez les mises à jour en exécutant la commande suivante :

    ```bash
    sudo apt-get update 
    sudo apt-get upgrade 
    ```
2.  Installer [Ruby](https://www.ruby-lang.org/)  Ruby est un langage dynamique open source qui peut être combiné avec des scripts bash. 
    
    ```bash
    sudo apt-get install ruby-full
    ```

## <a name="install-development-tools"></a>Installer les outils de développement
Cette section vous montre comment installer deux éditeurs de texte. Gedit est l’éditeur de texte par défaut pour l’environnement de bureau Gnome. Il est conçu comme un éditeur de texte à usage général. Visual Studio Code est un éditeur de texte qui inclut une prise en charge du débogage et de l’intégration du contrôle de code source.

> [!NOTE]
> Plusieurs éditeurs de texte différents sont disponibles. Visual Studio Code et gedit sont simplement deux exemples.

1. Installez [gedit](https://help.gnome.org/users/gedit/stable/).

    ```bash
    sudo apt-get install gedit
    ```
1. Installez [Visual Studio Code](https://code.visualstudio.com/).  Visual Studio Code peut être installé à l’aide du Snap Store.  Pour obtenir d’autres options d’installation, consultez [Autres chargements de Visual Studio Code](https://code.visualstudio.com/#alt-downloads).

    ```bash
    sudo snap install vscode --classic 
    ```

    Le modèle est maintenant mis à jour et dispose à la fois du langage de programmation et des outils de développement nécessaires pour effectuer le labo. L’image du modèle peut à présent être publiée dans le labo. Sélectionnez le bouton **Publier** dans la page du modèle pour publier le modèle dans le labo.  

## <a name="cost"></a>Coût 
Si vous souhaitez estimer le coût de ce labo, vous pouvez utiliser l’exemple suivant :
 
Pour une classe de 25 étudiants avec 20 heures de cours planifiées et un quota de 10 heures pour le travail à la maison ou les devoirs, le prix du labo est le suivant : 

25 étudiants * (20 + 10) heures * 20 unités Lab * 0,01 USD par heure = 150 USD

Pour plus d’informations sur les tarifs, consultez le document suivant : [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusion
Cet article vous a présenté les étapes de création d’un labo pour les cours d’écriture de scripts. Cet article était dédié à la configuration des outils de script Ruby sur une machine Linux, mais la même configuration peut être utilisée pour d’autres cours de création de scripts comme Python sur Linux.

## <a name="next-steps"></a>Étapes suivantes
Les étapes suivantes sont communes à la configuration de n’importe quel labo :

- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users) 





