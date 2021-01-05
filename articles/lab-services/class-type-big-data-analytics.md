---
title: Configurer un labo pour enseigner l’analytique du Big Data à l’aide d’Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un labo pour enseigner l’analytique du Big Data à l’aide du déploiement Docker de Hortonworks Data Platform (HDP).
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: 5eb9cd00350c41645d4427e30a6f25a6c163358c
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659894"
---
# <a name="set-up-a-lab-for-big-data-analytics-using-docker-deployment-of-hortonworks-data-platform"></a>Configurer un labo pour l’analytique du Big Data à l’aide du déploiement Docker de la plateforme de données Hortonworks

Cet article explique comment configurer un labo pour enseigner une classe d’analytique du Big Data.  Grâce à ce type de classe, les étudiants apprennent à gérer de gros volumes de données et à appliquer des algorithmes d’apprentissage automatique et statistique pour obtenir des informations sur les données.  L’un des principaux objectifs pour les étudiants est d’apprendre à utiliser les outils d’Analytique données, tels que le [package logiciel open source d’Apache Hadoop](https://hadoop.apache.org/) qui fournit des outils pour le stockage, la gestion et le traitement du Big Data.

Dans ce labo, les étudiants utilisent une version commerciale bien connue de Hadoop fournie par [Cloudera](https://www.cloudera.com/), appelée [Hortonworks de données (HDP)](https://www.cloudera.com/products/hdp.html).  Plus précisément, les étudiants utilisent [HDP Sandbox 3.0.1](https://www.cloudera.com/tutorials/getting-started-with-hdp-sandbox/1.html) qui est une version simplifiée et facile à utiliser de la plateforme, gratuite et destinée à l’apprentissage et à l’expérimentation.  Bien que cette classe puisse utiliser des machines virtuelles Windows ou Linux avec HDP Sandbox déployé, cet article montre comment utiliser Windows.

Autre aspect intéressant de ce labo, nous allons déployer HDP Sandbox sur les machines virtuelles de laboratoire à l’aide de conteneurs [Docker](https://www.docker.com/).  Chaque conteneur Docker fournit son propre environnement isolé pour que les applications logicielles s’y exécutent.  Sur le plan conceptuel, les conteneurs Docker sont similaires aux machines virtuelles imbriquées et peuvent être utilisés pour déployer et exécuter facilement un large éventail d’applications logicielles basées sur des images conteneur fournies sur [Docker Hub](https://www.docker.com/products/docker-hub).  Le script de déploiement de Cloudera pour HDP Sandbox extrait automatiquement l’[image Docker de HDP Sandbox 3.0.1](https://hub.docker.com/r/hortonworks/sandbox-hdp) à partir de Docker Hub et exécute deux conteneurs Docker :
  - sandbox-hdp
  - sandbox-proxy

## <a name="lab-configuration"></a>Configuration du laboratoire

Pour configurer ce labo, vous avez besoin d’un abonnement Azure et d’un compte Lab pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services. Pour plus d’informations sur la création d’un compte de laboratoire, consultez notre [Tutoriel pour configurer un compte lab](tutorial-setup-lab-account.md).  Vous pouvez également utiliser un compte Lab existant.

### <a name="lab-account-settings"></a>Paramètres du compte Lab

Activez les paramètres décrits dans le tableau ci-dessous pour le compte lab. Pour plus d’informations sur l’activation des images de la Place de marché, consultez [Spécifier les images de la Place de marché accessibles aux créateurs de labo](./specify-marketplace-images.md).

| Paramètres du compte lab | Instructions |
| ------------------- | ------------ |
|Image de la Place de marché| Activez l’image Windows 10 Pro pour l’utiliser dans votre compte lab.|

### <a name="lab-settings"></a>Paramètres du labo

Utilisez les paramètres du tableau ci-dessous lors de la configuration d’un laboratoire de classe.  Pour plus d’informations sur la création d’un laboratoire de classe, consultez le [didacticiel Configurer un laboratoire de salle de classe](tutorial-setup-classroom-lab.md).

| Paramètres du labo | Valeur/instructions |
| ------------ | ------------------ |
|Taille de la machine virtuelle| Moyenne (virtualisation imbriquée). Cette taille de machine virtuelle est idéale pour les bases de données relationnelles, la mise en cache en mémoire et l’analytique.  Cette taille prend également en charge la virtualisation imbriquée.|  
|Image de machine virtuelle| Windows 10 Pro|

> [!NOTE] 
> Nous devons utiliser la virtualisation moyenne (virtualisation imbriquée), car le déploiement de HDP Sandbox à l’aide de Docker requiert :
>   - Windows Hyper-V avec virtualisation imbriquée
>   - Au moins 10 Go de RAM

## <a name="template-machine-configuration"></a>Configuration du modèle de machine

Pour configurer le modèle d’ordinateur virtuel, nous allons effectuer les opérations suivantes :
- Installation de Docker
- Déployer HDP Sandbox
- Utilisation de PowerShell et du Planificateur de tâches Windows pour démarrer automatiquement les conteneurs Docker

### <a name="install-docker"></a>Installation de Docker

Les étapes de cette section sont basées sur les [instructions de Cloudera pour un déploiement à l’aide de conteneurs Docker](https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html). 

Pour utiliser des conteneurs Docker, vous devez d’abord installer Docker Desktop sur le modèle de machine virtuelle :

1. Suivez les étapes de la section [Prerequisites](https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html#prerequisites) (Configuration requise) pour installer [Docker pour Windows](https://docs.docker.com/docker-for-windows/install/). 

    > [!IMPORTANT] 
    > Assurez-vous que l’option de configuration **Utiliser des conteneurs Windows au lieu de conteneurs Linux** n’est pas cochée.

1. Assurez-vous que **les conteneurs Windows et les fonctionnalités Hyper-V** sont activés.
   ![Turn Windows features on or off](./media/class-type-big-data-analytics/windows-hyperv-features.png)

1. Suivez les étapes de la section [Memory for Windows](https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html#memory-for-windows) (Mémoire pour Windows) pour paramétrer la configuration de la mémoire de Docker.

    > [!WARNING]
    > Si vous cochez par inadvertance l’option **Utiliser des conteneurs Windows au lieu de conteneurs Linux** lors de l’installation de Docker, vous ne verrez pas les paramètres de configuration de la mémoire.  Pour corriger ce problème, vous pouvez basculer vers l’utilisation de conteneurs Linux en [cliquant sur l’icône Docker dans la barre d’état système de Windows](https://docs.docker.com/docker-for-windows/#docker-settings-dialog) ; lorsque le menu Docker Desktop s’ouvre, sélectionnez **Basculer vers les conteneurs Linux**.
 
### <a name="deploy-hdp-sandbox"></a>Déployer HDP Sandbox

Dans cette section, vous allez déployer HDP Sandbox, puis y accéder à l’aide du navigateur.

1. Assurez-vous d’avoir installé [Git Bash](https://gitforwindows.org/) comme indiqué dans la section [Prerequisites](https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html#prerequisites) (Configuration requise) du guide, car cela est recommandé pour effectuer les étapes suivantes.

1. À l’aide du [guide de Cloudera pour le déploiement et l’installation de Docker](https://www.cloudera.com/tutorials/sandbox-deployment-and-install-guide/3.html), effectuez les étapes décrites dans les sections suivantes :
   
   -    Déployer HDP Sandbox
   -    Vérifier HDP Sandbox

    > [!WARNING] 
    > Lorsque vous téléchargez le fichier .zip le plus récent pour HDP, assurez-vous de ne *pas* enregistrer le fichier .zip dans un chemin d’accès de répertoire qui comprend des espaces blancs.

    > [!NOTE] 
    > Si vous recevez une exception au cours du déploiement indiquant que le **lecteur n’a pas été partagé**, vous devez partager votre lecteur C avec Docker afin que les conteneurs Linux de HDP puissent accéder aux fichiers Windows locaux.  Pour corriger ce problème, [cliquez sur l’icône Docker dans la barre d’état système de Windows](https://docs.docker.com/docker-for-windows/#docker-settings-dialog) pour ouvrir le menu Docker Desktop et sélectionnez **Paramètres**.  Lorsque la boîte de dialogue **Paramètres de Docker** s’ouvre, sélectionnez **Ressources > Partage de fichiers** et cochez le lecteur **C**.  Vous pouvez ensuite répéter les étapes pour déployer HDP Sandbox.

1. Une fois que les conteneurs Docker pour HDP Sandbox sont déployés et fonctionnent, vous pouvez accéder à l’environnement en lançant votre navigateur et en suivant les instructions de Cloudera pour ouvrir la [page d’accueil de Sandbox](https://www.cloudera.com/tutorials/learning-the-ropes-of-the-hdp-sandbox.html#welcome-page) et lancer le tableau de bord HDP.

    > [!NOTE] 
    > Ces instructions supposent que vous avez d’abord mappé l’adresse IP locale de l’environnement du bac à sable à l’adresse sandbox-hdp.hortonworks.com dans le fichier hôte sur votre modèle de machine virtuelle.  Si vous **n’avez pas** effectué ce mappage, vous pouvez accéder à la page d’accueil de Sandbox en accédant à l’adresse `http://localhost:8080`.

### <a name="automatically-start-docker-containers-when-students-log-in"></a>Démarrer automatiquement les conteneurs Docker lorsque les étudiants se connectent

Pour fournir une expérience facile à utiliser pour les étudiants, nous allons utiliser un script PowerShell qui effectue automatiquement les opérations suivantes :
  - Démarre les conteneurs Docker HDP Sandbox lorsqu’un étudiant démarre sa machine virtuelle de labo et s’y connecte.
  - Lance le navigateur et accède à la page d’accueil de Sandbox.
Nous utiliserons également le Planificateur de tâches Windows pour exécuter automatiquement ce script lorsqu’un étudiant se connecte à sa machine virtuelle.
Pour ce faire, procédez comme suit : [Création de scripts d’analytique du Big Data](https://github.com/Azure/azure-devtestlab/tree/master/samples/ClassroomLabs/Scripts/BigDataAnalytics/).

## <a name="cost-estimate"></a>Estimation du coût

Si vous souhaitez estimer le coût de ce labo, vous pouvez utiliser l’exemple suivant.

Pour une classe de 25 étudiants avec 20 heures de cours planifiées et un quota de 10 heures pour le travail à la maison ou les devoirs, le prix du labo est le suivant :
  - 25 étudiants * (20 + 10) heures * 55 unités Lab * 0,01 USD par heure = 412,50 USD

Pour plus d’informations sur les tarifs, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusion

Cet article vous a présenté les étapes nécessaires à la création d’un labo pour une classe d’analytique du Big Data qui utilise Hortonworks Data Platform déployé avec Docker.  La configuration de ce type de classe peut être utilisée pour des classes similaires d’Analytique données.  Cette configuration peut également s’appliquer à d’autres types de classes qui utilisent Docker pour le déploiement.

## <a name="next-steps"></a>Étapes suivantes

Les étapes suivantes sont communes à la configuration de n’importe quel labo.

- [Créer et gérer un modèle](how-to-create-manage-template.md)
- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users)