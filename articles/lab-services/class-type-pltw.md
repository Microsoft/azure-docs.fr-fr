---
title: Configuration d’instances Lab Projet Lead The Way avec Azure Lab Services
description: Découvrez comment configurer des instances Lab pour enseigner dans le cadre de cours Project Lead The Way.
ms.topic: article
ms.date: 10/28/2020
ms.openlocfilehash: ca4fdae2372895c17c4a98dd3959935108846744
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024617"
---
# <a name="set-up-labs-for-project-lead-the-way-classes"></a>Configuration d’instances Lab pour des cours Project Lead The Way

[Project Lead The Way (PLTW)](https://www.pltw.org/) est un organisme à but non lucratif qui propose des programmes éducatifs à l’intention des élèves du primaire et du secondaire à travers les États-Unis dans les domaines de l’informatique, de l’ingénierie et des sciences biomédicales.  Dans chaque cours PLTW, les élèves utilisent diverses applications logicielles dans le cadre de leurs travaux pratiques.  La plupart des applications logicielles exigent un processeur rapide voire, dans certains cas, un GPU.  Cet article explique comment configurer les instances Lab pour les cours PLTW suivants, qui sont généralement dispensés à des lycéens :

- **Introduction à la conception technique**

    Les élèves sont initiés au processus de conception technique, avec notamment l’utilisation du logiciel de [conception assistée par ordinateur (CAO) Autodesk Inventor](https://www.autodesk.com/products/inventor/new-features) pour la modélisation 3D.

- **Principes de l’ingénierie**
    
    Les élèvent découvrent les mécanismes de l’ingénierie, de la résistance structurelle et des matériaux, et de l’automatisation.  Dans ce cours sont utilisés des logiciels tels que [MDSolids](https://s3.amazonaws.com/support-downloads.pltw.org/2020-21/MD+Solids/MD+Solids+Software+Installation+Guide.pdf), [West Point Bridge Designer](https://s3.amazonaws.com/support-downloads.pltw.org/2020-21/West+Point+Bridge+Builder/Installation+Guide+for+West+Point+Bridge+Designer.pdf) et la [simulation America’s Army](https://s3.amazonaws.com/support-downloads.pltw.org/2020-21/America's+Army/Installation+Guide+for+Americas+Army+Simulation+17-18.pdf).

- **Génie civil et architecture**

    Les élèves reçoivent un enseignement en construction et en conception et développement de sites avec le logiciel de conception architecturale [Autodesk Revit](https://www.autodesk.com/products/revit/overview) pour la modélisation et conception automatisée de bâtiments (BIM) en 3D.

- **Productique**

    Les élèvent explorent les processus de fabrication modernes qui font appel à la robotique et l’automation.   Dans ce cours, les élèves utilisent le logiciel de [CAO Autodesk Inventor](https://www.autodesk.com/products/inventor/new-features) et le logiciel de [fabrication assistée par ordinateur (FAO) Autodesk Inventor](https://www.autodesk.com/products/inventor-cam/overview). 

- **Électronique numérique**

    Les élèves étudient les circuits et les appareils à logique électronique à l’aide du logiciel de conception de circuits et de simulation [National Instruments Multisim](https://www.ni.com/en-us/shop/electronic-test-instrumentation/application-software-for-electronic-test-and-instrumentation-category/what-is-multisim.html).

- **Conception technique et développement**

    Les élèves participent à une solution de bout en bout en combinant recherche, conception et tests, qu’ils présentent à un groupe d’ingénieurs.  Dans ce cours, les élèves utilisent le logiciel de [CAO Autodesk Inventor](https://www.autodesk.com/products/inventor/new-features).

- **Notions de base de l’informatique**

    Les élèves sont initiés aux concepts et outils informatiques.  Après une phase initiale de programmation par blocs, ils passent au codage textuel en utilisant des environnements de codage tels que les [blocs VEXcode v5](https://s3.amazonaws.com/support-downloads.pltw.org/2020-21/VEXcode+V5+Blocks/VexCode+V5+Blocks+Installation+Guide.pdf).

- **Principes de l’informatique**
    
    Les élèves développent leurs compétences en programmation avec [Python](https://www.python.org/) en utilisant [l’environnement de développement Microsoft Visual Studio Code](https://code.visualstudio.com/). 

- **Informatique A**

    Dans ce cours, les élèves étendent leurs compétences en programmation en apprenant à développer des applications mobiles.  Ils apprennent [Java](https://www.java.com/) à l’aide de [l’environnement de développement Microsoft Visual Studio Code](https://code.visualstudio.com/).  Les élèves utilisent aussi un émulateur qui leur permet d’exécuter et tester le code de leur application mobile.  Pour savoir comment configurer un émulateur dans Azure Lab Services, [contactez Azure Lab Services](mailto:AzLabsCOVIDSupport@microsoft.com).

Pour obtenir la liste complète des logiciels du cours, accédez au site [PLTW](https://www.pltw.org/pltw-software) du cours.

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour pouvoir configurer des instances Lab pour PLTW, il vous faut un abonnement Azure et un compte Lab. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. 

Dès lors que vous disposez d’un abonnement Azure, vous pouvez créer un compte Lab dans Azure Lab Services. Pour plus d’informations sur la création d’un compte Lab, consultez [Configuration d’un compte Lab](./tutorial-setup-lab-account.md). Vous pouvez également utiliser un compte Lab existant.

Une fois que vous avez configuré un compte Lab, vous avez tout intérêt à créer une instance Lab distincte pour chaque session de cours PLTW dispensée dans votre établissement.  Nous vous recommandons aussi de créer des images distinctes pour chaque type de cours PLTW.  Pour savoir comment structurer vos instances Lab et vos images, consultez le billet de blog [Passage d’une instance Lab physique à Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931).

### <a name="lab-account-settings"></a>Paramètres du compte Lab

Activez les paramètres de votre compte Lab (cf. tableau suivant). Pour plus d’informations sur l’activation des images de la Place de marché Azure, consultez [Spécification des images de la Place de marché Azure accessibles aux créateurs d’instances Lab](./specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| -------------------- | ----- |
| Image de la Place de marché | Activez l’image Windows 10 Pro pour l’utiliser dans votre compte lab. |

<br>

### <a name="lab-settings"></a>Paramètres du labo
La taille de la machine virtuelle que nous conseillons d’utiliser pour les cours PLTW dépend des types de charges de travail effectuées par les élèves en cours.  Pour les cours listés plus faut, nous vous recommandons les tailles GPU de petite taille (visualisation) et Machine virtuelle de grande taille. Lorsque vous configurez des instances Lab pour vos cours PLTW, consultez l’aide figurant dans le tableau ci-dessous :

| Paramètre de labo | Valeur et description | Recommandation de cours |
| ------------ | ------------------ | --- |
| Taille de la machine virtuelle | **GPU de petite taille (visualisation)**<br>Idéal pour la visualisation à distance, le streaming, les jeux et l’encodage avec des infrastructures de type OpenGL et DirectX. | Nous vous recommandons d’utiliser cette taille pour les cours PLTW suivants : Génie civil et architecture civile, Électronique numérique, Productique et Conception technique et développement.
| Taille de la machine virtuelle | **Grande**<br>Idéal pour les applications exigeant des processeurs plus rapides, un niveau de performance du disque local plus élevé, des bases de données volumineuses et des caches mémoire volumineux. | Nous vous recommandons d’utiliser cette taille pour les cours PLTW suivants : Introduction à la conception technique, Principes de l’ingénierie, Notions de base de l’informatique, Principes de l’informatique et Informatique A. |

<br>

### <a name="license-server"></a>Serveur de licences
La plupart des logiciels utilisés dans les cours PLTW mentionnés plus haut n’exigent *pas* d’accès à un serveur de licences.  Cependant, vous devrez accéder à un serveur de licences si vous envisagez d’utiliser le modèle de licence réseau Autodesk pour les logiciels suivants :
-   Revit
-   Inventor
-   Inventor CAM

Pour utiliser des licences réseau avec les logiciels Autodesk, [PLTW explique de manière détaillée](https://www.pltw.org/pltw-software) comment installer le gestionnaire de licences réseau Autodesk sur un serveur de licences.  En général, ce serveur de licences se trouve sur le réseau local ou est hébergé sur une machine virtuelle Azure au sein du réseau virtuel Azure.

Une fois votre serveur de licences configuré, vous devez procéder au [peering entre le réseau virtuel](./how-to-connect-peer-virtual-network.md) et votre [compte Lab](./tutorial-setup-lab-account.md). Cet appairage doit être effectué *avant* de créer l’instance Lab afin que les machines virtuelles Lab puissent accéder au serveur de licences et inversement.

Les fichiers de licence générés par Autodesk incorporent l’adresse MAC du serveur de licences.  Si vous décidez d’héberger votre serveur de licences à l’aide d’une machine virtuelle Azure, il est important de vérifier que l’adresse MAC de votre serveur de licences ne change pas. Si elle évolue, vous devrez regénérer vos fichiers de licence. Pour éviter qu’elle ne change, procédez comme suit :

- [Définissez une adresse IP privée et une adresse MAC statiques](./how-to-create-a-lab-with-shared-resource.md#static-private-ip-and-mac-address) pour la machine virtuelle Azure qui héberge votre serveur de licences.
- Veillez à configurer votre compte Lab et le réseau virtuel du serveur de licences dans une région ou à un emplacement qui dispose d’une capacité de machine virtuelle suffisante pour ne pas avoir à déplacer par la suite ces ressources dans une nouvelle région ou un nouvel emplacement.

Pour plus d’informations, consultez [Configuration d’un serveur de licences comme ressource partagée](./how-to-create-a-lab-with-shared-resource.md).

### <a name="template-machine"></a>Machine modèle
Certains des fichiers d’installation nécessaires pour PLTW sont volumineux. Lors du téléchargement sur une machine virtuelle de modèle Lab, la copie est susceptible de prendre beaucoup de temps.

Au lieu de les télécharger les fichiers d’installation sur la machine modèle et de tout y installer, nous vous recommandons de créer vos images PLTW dans votre environnement physique.  Vous pourrez alors importer les images personnalisées dans Shared Image Gallery, et ainsi les utiliser pour créer vos instances Lab.  Pour plus d’informations, consultez [Chargement d’une image personnalisée dans Shared Image Gallery](./upload-custom-image-shared-image-gallery.md).

Si vous suivez cette recommandation, tenez compte des principales tâches à effectuer pour configurer une instance Lab :

1. Dans votre environnement physique, créez l’image pour le cours.

    a.  Suivez la procédure détaillée de PLTW pour télécharger les fichiers d’installation et installer les logiciels nécessaires.

    > [!NOTE]    
    > Lorsque vous installez les applications Autodesk, l’ordinateur sur lequel vous les installez doit être en mesure de communiquer avec votre serveur de licences. L’Assistant Installation Autodesk vous invitera à spécifier le nom de l’ordinateur sur lequel le serveur de licences est hébergé.  Si vous hébergez votre serveur de licences sur une machine virtuelle Azure, vous devrez peut-être attendre que l’Assistant Installation accède à votre serveur de licences pour installer Autodesk sur la machine virtuelle de modèle Lab.

    b.  [Installez et configurez OneDrive](./how-to-prepare-windows-template.md#install-and-configure-onedrive) ou les autres options de sauvegarde utilisées par votre établissement.
    
    c.  [Installez et configurez les mises à jour Windows](./how-to-prepare-windows-template.md#install-and-configure-updates).

1.  Chargez l’image personnalisée sur [l’instance Shared Image Gallery rattachée à votre compte Lab](./how-to-attach-detach-shared-image-gallery.md).

1.  Créez une instance Lab, puis sélectionnez l’image personnalisée chargée à l’étape précédente.

1.  Une fois l’instance Lab créée, démarrez la machine virtuelle de modèle et connectez-vous-y pour vérifier que l’image fonctionne comme prévu.

1.  Enfin, publiez la machine virtuelle de modèle pour créer les machines virtuelles des élèves.

## <a name="student-devices"></a>Appareils des élèves
Les élèves peuvent se connecter à leurs machines virtuelles Lab à partir d’ordinateurs Windows, de Mac et de Chromebooks. Pour obtenir des instructions, consultez les liens suivants :

- [Se connecter à partir de Windows](./how-to-use-classroom-lab.md#connect-to-the-vm)
- [Se connecter à partir d’un Mac](./connect-virtual-machine-mac-remote-desktop.md)
- [Se connecter à partir d’un Chromebook](./connect-virtual-machine-chromebook-remote-desktop.md)

## <a name="cost"></a>Coût
Étudions un exemple de coût estimatif des cours PLTW.  Cette estimation n’inclut pas les coûts liés à l’exécution d’un serveur de licences ni à l’utilisation d’une instance Shared Image Gallery. Prenons une classe de 25 élèves, chacun ayant 20 heures de cours planifiées.  Chaque élève dispose également d’un quota de 10 heures pour faire ses devoirs et activités à la maison en dehors des heures de cours prévues.  Voici les coûts estimés :

- **Machine virtuelle de grande taille**

    25 élèves &times; (20 heures planifiées + 10 heures de quota) &times; 70 unités Lab &times; 0,01 USD par heure = 525,00 USD

- **GPU de petite taille (visualisation)**

    25 élèves &times; (20 heures planifiées + 10 heures de quota) &times; 160 unités Lab &times; 0,01 USD par heure = 1 200,00 USD

> [!IMPORTANT] 
> L’estimation du coût est fournie à titre d’exemple uniquement.  Pour connaître les tarifs actuels, consultez [Tarifs Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

> [!NOTE] 
> La plupart des cours PLTW utilisent des applications accessibles via un navigateur, comme MIT App Inventor.  Ces applications sur navigateur n’ont pas besoin d’un processeur ni d’un GPU rapide. Elles sont accessibles à partir de n’importe quel appareil disposant d’une connexion Internet.  Quand les élèves utilisent des applications de ce type, nous vous recommandons de leur demander de passer par le navigateur de leur appareil physique plutôt que par celui de leur machine virtuelle Lab. En limitant l’usage de leur machine virtuelle Lab aux applications qui demandent un processeur ou un GPU rapide, les élèves contribuent à réduire les coûts.

## <a name="next-steps"></a>Étapes suivantes

Au fil de la configuration de votre instance Lab, consultez les articles suivants :

- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir les quotas](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users) 
