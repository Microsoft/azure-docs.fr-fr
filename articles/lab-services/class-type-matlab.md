---
title: Configurer un laboratoire pour enseigner MATLAB avec Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un laboratoire pour enseigner MATLAB avec Azure Lab Services.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 137959f51b08dceee150962f77110ee2ac1dc193
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85444996"
---
# <a name="setup-a-lab-to-teach-matlab"></a>Configurer un laboratoire pour enseigner MATLAB

[MATLAB](https://www.mathworks.com/products/matlab.html), qui signifie Matrix Laboratory, est la plate-forme de programmation de [MathWorks](https://www.mathworks.com/).  Elle combine la puissance de calcul et la visualisation, ce qui en fait un outil populaire dans les domaines des mathématiques, de l’ingénierie, de la physique et de la chimie.

Si vous utilisez une [licence à l’échelle du campus](https://www.mathworks.com/academia/tah-support-program/administrators.html), consultez les instructions de la section [Télécharger les fichiers d’installation de MATLAB](https://www.mathworks.com/matlabcentral/answers/259632-how-can-i-get-matlab-installation-files-for-use-on-an-offline-machine) pour télécharger les fichiers du programme d’installation MATLAB sur le modèle d’ordinateur.  

Dans cet article, nous montrerons comment configurer une classe qui utilise le logiciel client MATLAB avec un serveur de licences.

## <a name="license-server"></a>Serveur de licences

Avant de modifier le modèle d’ordinateur pour votre laboratoire, vous devrez configurer le serveur pour qu’il exécute le logiciel [Network License Manager](https://www.mathworks.com/help/install/administer-network-licenses.html).  Ces instructions s’appliquent uniquement aux établissements qui choisissent l’option de gestion de licences en réseau pour MATLAB permettant aux utilisateurs de partager un pool de clés de licence.  Vous devrez également enregistrer le fichier de licence et la clé d’installation de fichier pour une utilisation ultérieure.  Pour obtenir des instructions détaillées sur le téléchargement d’un fichier de licence, reportez-vous à la première étape de l’article [Installer Network License Manager avec une connexion Internet](https://www.mathworks.com/help/install/ug/install-network-license-manager-with-internet-connection.html).

Pour obtenir des instructions détaillées sur l’installation d’un serveur de licences, consultez [Installer Network License Manager avec une connexion Internet](https://www.mathworks.com/help/install/ug/install-network-license-manager-with-internet-connection.html).  Pour l’emprunt de licences, consultez l’article [Emprunter une licence](https://www.mathworks.com/help/install/license/borrow-licenses.html).

En supposant que le serveur de licences se trouve sur un réseau local ou un réseau privé dans Azure, n’oubliez pas [d’interconnecter le réseau virtuel](how-to-connect-peer-virtual-network.md) à votre [compte de laboratoire](tutorial-setup-lab-account.md).  Le peering de réseaux doit être effectué avant de créer le laboratoire afin que les machines virtuelles du laboratoire puissent accéder au serveur de licences.

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour configurer ce labo, vous avez besoin d’un abonnement Azure pour commencer.  Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services ou utiliser un compte existant.  Pour créer un compte de laboratoire, consultez le [tutoriel sur la configuration d’un compte de laboratoire](tutorial-setup-lab-account.md).

Pour créer un laboratoire, suivez le [tutoriel sur la configuration d’un laboratoire de salle de classe](tutorial-setup-classroom-lab.md).  Appliquez les paramètres suivants :

| Taille de la machine virtuelle | Image |
| -------------------- | ----- |
| Moyenne | Windows 10 |

MATLAB est pris en charge sur un plus grand nombre de systèmes d’exploitation.  Consultez [Configuration système requise pour MATLAB](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) pour plus d’informations.

> [!WARNING]
> N’oubliez pas [d’interconnecter le réseau virtuel](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) du compte de laboratoire au réseau virtuel du serveur de licences avant de créer le laboratoire.

## <a name="template-machine"></a>Machine modèle

Une fois la machine modèle créée, démarrez-la et connectez-vous à celle-ci pour effectuer les tâches principales suivantes.

1. Téléchargez les fichiers d’installation du logiciel client MATLAB.
2. Installez MATLAB à l’aide de la clé d’installation du fichier.

L’installation de MATLAB s’effectuera en plusieurs étapes.  La première étape téléchargera les fichiers pour MATLAB et les autres produits que vous souhaitez installer.  L’utilisation d’une clé d’installation de fichier nécessite que tous les fichiers d’installation des produits à installer soient prétéléchargés.  La deuxième étape installe le logiciel MATLAB sur la machine virtuelle du modèle et active le logiciel.  Si le modèle de machine virtuelle est configuré pour être activé à l’aide du serveur de licences, les machines virtuelles des élèves appliqueront également cette procédure.

### <a name="download-installation-files"></a>Télécharger les fichiers d’installation

Vous devez être administrateur de licence pour télécharger les fichiers d’installation et obtenir le fichier de licence et la clé d’installation du fichier.  Vous trouverez ci-dessous les étapes pour télécharger les fichiers d’installation.

1. Connectez-vous à votre compte sur le site [https://www.mathworks.com](https://www.mathworks.com).
2. Choisissez **My Account** (Mon compte).
3. Sous la section **My Software** (Mes logiciels) de la page du compte, cliquez sur la licence associée au programme d’installation de Network License Manager pour le laboratoire.
4. Sur la page des détails de la licence, cliquez sur **Download Products** (Télécharger des produits).
5. Attendez l’extraction automatique du programme d’installation.
6. Démarrez le programme d’installation.  
7. Sur la page **Sign in to your MathWorks Account** (Connexion à votre compte MathWorks), indiquez votre comte MathWorks.
8. Sur la page **MathWorks License Agreement** (Accord de licence MathWorks), acceptez l’accord, puis cliquez sur **Next** (Suivant).
9. Cliquez sur menu déroulant **Advanced Options** (Options avancées), puis choisissez **I want to download without installing** (Télécharger sans installer).
10. Sur la page **Select destination folder** (Sélectionner le dossier de destination), cliquez sur **Next** (Suivant).
11. Sélectionnez **Windows** comme plate-forme de l’ordinateur sur lequel vous allez installer MATLAB.
12. Sur la page **Select product** (Sélectionner un produit), vérifiez que MATLAB est sélectionné ainsi que tous les autres produits MathWorks que vous souhaitez installer.
13. Sur la page **Confirm Selections and Download** (Confirmer les sélections et télécharger), cliquez sur **Begin Download** (Démarrer le téléchargement).  
14. Attendez la fin du téléchargement des produits sélectionnés.  Cliquez sur **Terminer**.

Vous pouvez également télécharger une image ISO à partir du site Web MathWorks.

1. Connectez-vous à votre compte sur le site [https://www.mathworks.com](https://www.mathworks.com).
2. Accédez à [https://www.mathworks.com/downloads](https://www.mathworks.com/downloads).
3. Sélectionnez la version de MATLAB que vous souhaitez installer.
4. Cliquez sur le lien « Get {version}.iso image » sous les liens connexes où {version} ressemble à R2020a.
5. Cliquez sur le lien bleu **download release** (télécharger la version) pour Windows.

### <a name="run-installer"></a>Exécuter le programme d'installation

Une fois les fichiers téléchargés, la deuxième étape consiste à exécuter le programme d’installation. Ici aussi, vous devez être administrateur de licence pour effectuer cette étape.  Seuls les administrateurs de licence peuvent installer MATLAB avec une clé d’installation de fichier.

1. Vérifiez le fichier de licence téléchargé et assurez-vous que la ligne SERVER indique correctement le serveur de licences.  Pour plus d’informations sur la façon dont le fichier de licence doit être formaté, consultez les articles [Mettre à jour une licence réseau](https://www.mathworks.com/help/install/ug/network-license-files.html), [Emprunter une licence](https://www.mathworks.com/help/install/license/borrow-licenses.html) et [Rechercher l’ID d’hôte](https://www.mathworks.com/matlabcentral/answers/101892-what-is-a-host-id-how-do-i-find-my-host-id-in-order-to-activate-my-license).
2. Lancez le programme d’installation MATLAB.
3. Sur la page **Sign in to your MathWorks Account** (Connexion à votre compte MathWorks), indiquez votre comte MathWorks.
4. Sur la page **MathWorks License Agreement** (Accord de licence MathWorks), acceptez l’accord, puis cliquez sur **Next** (Suivant).
5. Cliquez sur le menu déroulant **Advanced Options** (Options avancées) et choisissez **I have a File Installation Key** (J’ai une clé d’installation de fichier).
6. Sur la page **Install using File Installation Key** (Installer à l’aide d’une clé d’installation de fichier), entrez la clé d’installation de fichier pour le serveur de licences.   Cliquez sur **Suivant**.
7. Sur la page **Select License File** (Sélectionner un fichier de licence), accédez au fichier de licence enregistré précédemment lors du téléchargement des fichiers d’installation.
8. Sur la page **Select Destination Folder** (Sélectionner le dossier de destination), cliquez sur **Next** (Suivant).
9. Sur la page **Select Products** (Sélectionner les produits), cliquez sur **Next** (Suivant).
10. Sur la page **Select Options** (Sélectionner les options), cliquez sur **Next** (Suivant).
11. Sur la page **Confirm Selections and Install** (Confirmer les sélections et installer), cliquez sur **Begin Install** (Démarrer l’installation).
12. Sur la page **Installation Complete** (Installation terminée), vérifiez que l’option **Activate MATLAB** (Activer MATLAB) est sélectionnée.  Cliquez sur **Terminer**.

## <a name="cost-estimate"></a>Estimation du coût

Nous allons aborder une estimation de coût possible pour cette classe.  Cette estimation n’inclut pas le coût d’utilisation du serveur de licences.  Nous allons utiliser une classe de 25 élèves.  Nous prévoyons 20 heures de temps de classe.  En outre, chaque étudiant obtient un quota de 10 heures pour les devoirs ou travaux en dehors des heures de cours planifiées.  La taille de machine virtuelle que nous avons choisie était moyenne, soit 55 unités Lab.

Voici un exemple d’estimation de coût possible pour cette classe :

25 élèves \* (20 heures planifiées + 10 heures de quota) \* 55 unités Lab \* 0,01 USD par heure = 412,50 USD

>[!IMPORTANT]
> L’estimation du coût est fournie à titre d’exemple uniquement.  Pour en savoir plus sur les tarifs actuels, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).  

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer, gérer et publier un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)
