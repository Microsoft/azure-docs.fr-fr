---
title: Utiliser Azure Lab Services pour un hackathon
description: Cet article explique comment utiliser Azure Lab Services pour créer des laboratoires dans le cadre d’un hackathon.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a0917ede4502dcbb59d1a30b7985b06c06975599
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602578"
---
# <a name="use-azure-lab-services-for-your-next-hackathon"></a>Utiliser Azure Lab Services pour votre prochain hackathon
Conçu pour être léger et facile à utiliser, Azure Lab Services vous permet de configurer rapidement un nouveau laboratoire de machines virtuelles pour votre hackathon.  Utilisez la liste de vérification suivante pour vous assurer que votre hackathon se déroule le mieux possible. Cette liste de vérification doit être remplie par votre service informatique ou un professeur en charge de la création et de la gestion du laboratoire de votre hackathon. 

Pour utiliser Lab Services dans le cadre d’un hackathon, veillez à créer le compte lab et votre laboratoire au moins quelques jours avant le début du hackathon. Suivez également les instructions ci-dessous :

## <a name="guidance"></a>Assistance

- **Créez le laboratoire dans une région ou un emplacement le plus proche des participants**. 

    Pour réduire la latence, créez votre laboratoire dans une région la plus proche des participants à votre hackathon.  Si vos participants sont situés dans le monde entier, vous devez faire appel à votre meilleur jugement pour créer un laboratoire central.  Vous pouvez également diviser le hackathon en plusieurs laboratoires en fonction des emplacements de vos participants.
- **Choisissez une taille de calcul adaptée aux besoins d’utilisation**.

    En général, plus la taille de calcul est grande, plus la machine virtuelle est rapide. Toutefois, pour limiter les coûts, vous devez sélectionner la taille de calcul la mieux adaptée aux besoins de vos participants. Pour plus d’informations sur les tailles de calcul disponibles, consultez [les informations sur le dimensionnement des machines virtuelles dans le guide de l’administrateur](administrator-guide.md#vm-sizing).
- **Configurez RDP\SSH pour la connexion Bureau à distance aux machines virtuelles Linux**.

    Si votre hackathon utilise des machines virtuelles Linux, vérifiez que le Bureau à distance est activé pour que vos participants puissent utiliser le protocole RDP (Remote Desktop Protocol) ou SSH (Secure Shell) pour se connecter à leur machine virtuelle. Cette étape est uniquement requise pour les machines virtuelles Linux et doit être activée lors de la création du laboratoire. Par ailleurs, pour le protocole RDP, vous devrez peut-être installer et configurer le serveur RDP et les packages GUI sur la machine virtuelle modèle avant de procéder à la publication.  Pour plus d’informations, consultez le [guide pratique pour activer le Bureau à distance pour Linux](how-to-enable-remote-desktop-linux.md).

- **Installez et arrêtez les mises à jour Windows**. 

    Si vous utilisez une image Windows, nous vous recommandons d’installer les dernières mises à jour Windows sur la [machine virtuelle modèle](how-to-create-manage-template.md) du laboratoire avant de la publier pour créer les machines virtuelles du laboratoire. Le but est de veiller à la sécurité et d’éviter que les participants ne soient contraints d’installer des mises à jour durant le hackathon, ce qui pourrait également provoquer le redémarrage de leur machine virtuelle. Vous pouvez également désactiver les mises à jour Windows pour empêcher toute interruption future. Consultez le [guide pratique pour installer et configurer des mises à jour Windows](how-to-prepare-windows-template.md#install-and-configure-updates).
- **Décidez de la façon dont les étudiants sauvegarderont leur travail**. 

    Chaque étudiant se voit affecter une machine virtuelle pendant toute la durée de vie du hackathon. Les étudiants peuvent enregistrer leur travail directement sur la machine, mais il est recommandé qu’ils le sauvegardent pour pouvoir y accéder une fois le hackathon terminé. Ils peuvent par exemple enregistrer leur travail dans un emplacement externe comme OneDrive, GitHub, etc. Pour utiliser OneDrive, vous pouvez le configurer automatiquement pour les étudiants sur leur machine virtuelle de laboratoire. Consultez le [guide pratique pour installer et configurer OneDrive](how-to-prepare-windows-template.md#install-and-configure-onedrive).
- **Définissez la capacité des machines virtuelles en fonction du nombre de participants**. 

    Veillez à définir la capacité des machines virtuelles de votre laboratoire en fonction du nombre prévu de participants à votre hackathon. Quand vous publiez la machine virtuelle modèle, la création de toutes les machines du laboratoire peut prendre plusieurs heures. C’est pourquoi nous vous recommandons de le faire bien avant le début du hackathon. Pour plus d’informations, consultez le [guide pratique pour mettre à jour la capacité du laboratoire](how-to-set-virtual-machine-passwords.md#update-the-lab-capacity).

- **Décidez s’il faut restreindre l’accès au laboratoire**. 

    Quand vous ajoutez des utilisateurs au laboratoire, l’option de restriction de l’accès est activée par défaut. Vous devez donc ajouter tous les e-mails des participants à votre hackathon à la liste pour qu’ils puissent s’inscrire et accéder au laboratoire à l’aide du lien d’inscription. Si vous ne connaissez pas à l’avance les participants à votre hackathon, vous pouvez désactiver l’option de restriction de l’accès. Toute personne peut alors s’inscrire au laboratoire en cliquant sur le lien d’inscription. Pour plus d’informations, consultez le [guide pratique pour ajouter des utilisateurs](how-to-configure-student-usage.md).

- **Vérifiez les paramètres de planification, de quota et d’arrêt automatique**. 

    Lab Services fournit plusieurs contrôles de coût pour limiter l’utilisation des machines virtuelles. Toutefois, si ces paramètres sont mal configurés, ils peuvent entraîner l’arrêt inattendu des machines virtuelles de votre laboratoire. Pour vérifier que ces paramètres sont configurés correctement pour votre hackathon, vérifiez ce qui suit :

    **Planification** : une [planification](how-to-create-schedules.md) vous permet de contrôler automatiquement à quel moment les machines de vos laboratoires sont démarrées et arrêtées. Par défaut, aucune planification n’est configurée quand vous créez un laboratoire. Toutefois, vous devez vous assurer que la planification de votre laboratoire est adaptée à votre hackathon.  Par exemple, si votre hackathon commence le samedi à 8h00 et se termine le dimanche à 17h00, vous pouvez créer une planification qui démarre automatiquement la machine à 7h30 le samedi (environ 30 minutes avant le début du hackathon) et l’arrête à 17h00 le dimanche. Mais vous pouvez très bien ne pas utiliser de planification.

    **Quota** : le [quota](how-to-configure-student-usage.md#set-quotas-for-users) contrôle le nombre d’heures pendant lesquelles les participants ont accès à une machine virtuelle en dehors des heures planifiées. Si le quota est atteint pendant qu’un participant utilise une machine, celle-ci est automatiquement arrêtée. Dans ce cas, le participant ne peut la redémarrer que si le quota est augmenté. Par défaut, quand vous créez un laboratoire, le quota est de 10 heures. Là encore, vous devez définir un quota qui prévoit suffisamment de temps pour le hackathon, en particulier si vous n’avez pas créé de planification.

    **Arrêt automatique** : si cette option est activée, le paramètre d’[arrêt automatique](how-to-enable-shutdown-disconnect.md) entraîne l’arrêt automatique des machines virtuelles Windows après un certain laps de temps, une fois qu’un étudiant s’est déconnecté de sa session RDP. Par défaut, ce paramètre est désactivé.

- **Configurez les paramètres de pare-feu pour autoriser les connexions aux machines virtuelles du laboratoire**. 

    Vérifiez que les paramètres de pare-feu de votre établissement scolaire ou de votre organisation autorisent la connexion aux machines virtuelles du laboratoire à l’aide du protocole RDP\SSH. Pour plus d’informations, consultez le [guide pratique pour configurer les paramètres de pare-feu de votre réseau](how-to-configure-firewall-settings.md).

- **Installez le client RDP\SSH sur les tablettes, ordinateurs Mac, PC, etc. des participants**.

    Les participants au hackathon doivent disposer d’une tablette ou d’un ordinateur portable sur lequel le client RDP et/ou SSH est installé pour se connecter aux machines virtuelles du laboratoire. Vous avez le choix entre plusieurs clients RDP ou SSH. Par exemple :

    - L’application **Connexion Bureau à distance** de Microsoft pour les connexions RDP. L’application Connexion Bureau à distance est prise en charge sur différents types de plateformes, notamment les Chromebooks et les [Mac](https://techcommunity.microsoft.com/t5/azure-lab-services/connecting-to-azure-lab-services-environments-on-your-macos/ba-p/1290162).
    - [Putty](https://techcommunity.microsoft.com/t5/azure-lab-services/connecting-to-azure-lab-services-environments-on-your-macos/ba-p/1290162), qui permet d’utiliser SSH pour se connecter à une machine virtuelle Linux.
- **Vérifiez les machines virtuelles du laboratoire**. 

    Après avoir publié les machines virtuelles du laboratoire, vous devez vérifier qu’elles sont bien configurées. Vous devez uniquement vérifier l’une des machines virtuelles d’un participant au laboratoire :

    1. Connectez-vous à l’aide du protocole RDP et/ou SSH.
    2. Ouvrez les applications et outils supplémentaires que vous avez installés pour personnaliser l’image de machine virtuelle de base.
    3. Explorez quelques scénarios de base représentatifs des activités que les participants effectueront afin de vérifier que les performances des machines virtuelles sont adaptées à la taille de calcul sélectionnée.

## <a name="on-the-day-of-hackathon"></a>Le jour du hackathon
Cette section décrit les étapes à effectuer le jour de votre hackathon.

1. **Démarrez les machines virtuelles du laboratoire**.

    En fonction de votre système d’exploitation, le démarrage d’une machine de laboratoire peut prendre jusqu’à 30 minutes. Il est donc important de démarrer les machines avant le début du hackathon pour ne pas faire attendre vos participants. Si vous utilisez une planification, veillez à démarrer automatiquement les machines virtuelles au moins 30 minutes à l’avance.
2. **Invitez les étudiants à s’inscrire et à accéder à leur machine virtuelle de laboratoire**. 

    Donnez aux participants les informations suivantes pour qu’ils puissent accéder à leur machine virtuelle de laboratoire. 

    - Lien d’inscription au laboratoire. 
    - Informations d’identification à utiliser pour se connecter à la machine. Cette étape s’applique uniquement si votre laboratoire utilise une image Windows et que vous avez configuré toutes les machines virtuelles pour utiliser le même mot de passe.
    - Instructions sur la façon dont les participants peuvent se connecter à l’aide du protocole SSH et/ou RDP à leur machine.

        Pour plus d’informations, consultez le [guide pratique pour envoyer des invitations à des utilisateurs](how-to-configure-student-usage.md?branch=master#send-invitations-to-users) et le [guide de connexion à des machines virtuelles Linux](how-to-use-remote-desktop-linux-student.md?branch=master). 

## <a name="next-steps"></a>Étapes suivantes
Commencez par créer un compte de labo dans les labos en suivant les instructions de l’article [Tutoriel : Configurer un compte de laboratoire avec Azure Lab Services](tutorial-setup-lab-account.md).
