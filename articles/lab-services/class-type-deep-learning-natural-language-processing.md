---
title: Configurer un labo axé sur le deep learning avec Azure Lab Services | Microsoft Docs
description: Découvrez comment configurer un laboratoire axé sur l’apprentissage profond (Deep Learning) dans le cadre du traitement en langage naturel (NLP) à l’aide d’Azure Lab Services.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 01199e76b7b9cb1a6360b5aba010d0cfd8936c23
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91251454"
---
# <a name="set-up-a-lab-focused-on-deep-learning-in-natural-language-processing-using-azure-lab-services"></a>Configurer un labo axé sur le deep learning dans le cadre du traitement en langage naturel avec Azure Lab Services
Cet article vous montre comment configurer un labo axé sur le deep learning dans le cadre du traitement en langage naturel (NLP) avec Azure Lab Services. Le traitement en langage naturel (NLP) est une forme d’intelligence artificielle (IA) qui fournit aux ordinateurs des fonctionnalités de traduction, de reconnaissance vocale et d’autres fonctionnalités de compréhension de la langue.  

Les étudiants qui suivent un cours de traitement en langage naturel disposent d’une machine virtuelle Linux pour apprendre à appliquer des algorithmes de réseau neuronal afin de développer des modèles de deep learning utilisés pour l’analyse de la langue humaine écrite. 

## <a name="lab-configuration"></a>Configuration du laboratoire
Pour configurer ce labo, vous avez besoin d’un abonnement Azure pour commencer. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer. Une fois que vous disposez d’un abonnement Azure, vous pouvez créer un compte lab dans Azure Lab Services ou utiliser un compte lab existant. Pour créer un compte lab, consultez le tutoriel suivant : [Tutoriel pour configurer un compte lab](tutorial-setup-lab-account.md).
 
Après avoir créé le compte lab, activez les paramètres suivants dans celui-ci : 

| Paramètres du compte lab | Instructions |
| ----------- | ------------ |  
| Images de la Place de marché | Activez l’image Data Science Virtual Machine pour Linux (Ubuntu) à utiliser au sein de votre compte lab.  Pour obtenir des instructions, consultez l’article suivant : [Spécifier les images de la Place de marché accessibles aux créateurs de labo](specify-marketplace-images.md). | 

Suivez [ce tutoriel](tutorial-setup-classroom-lab.md) pour créer un labo et appliquer les paramètres suivants :

| Paramètres du labo | Valeur/instructions | 
| ------------ | ------------------ |
| Taille de machine virtuelle | **Petite avec GPU (calcul)** . Cette taille est optimisée pour les applications nécessitant beaucoup de ressources système et de ressources réseau, notamment l’intelligence artificielle et le deep learning. |
| Image de machine virtuelle | [Data Science Virtual Machine pour Linux (Ubuntu)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804). Cette image fournit des outils et des frameworks de deep learning pour le machine learning et la science des données. Pour voir la liste complète des outils installés sur cette image, consultez l’article suivant : [Ce qui est inclut dans la machine DSVM](../machine-learning/data-science-virtual-machine/overview.md#whats-included-on-the-dsvm). |
| Activer la connexion Bureau à distance | <p>L’image de science des données est déjà configurée pour utiliser X2Go afin que les enseignants et les étudiants puissent se connecter à l’aide d’un Bureau à distance de l’interface graphique utilisateur.  X2Go n’exige *pas* l’activation du paramètre **Activer la connexion Bureau à distance**.  Ce paramètre doit être activé uniquement si vous choisissez plutôt d’utiliser RDP.

>**Important !** Bien que nous vous recommandons d’utiliser X2Go avec l’image de science des données, si vous choisissez d’utiliser à la place RDP, vous devrez vous connecter à la machine virtuelle Linux à l’aide de SSH la première fois et installer les packages RDP et GUI.  Ensuite, vous/vos étudiants pouvez vous connecter à la machine virtuelle Linux à l’aide de RDP ultérieurement.  Pour plus d’informations, consultez [Utiliser le Bureau à distance graphique pour des machines virtuelles Linux](how-to-enable-remote-desktop-linux.md).

L’image Data Science Virtual Machine pour Linux fournit les outils et infrastructures de deep learning nécessaires pour ce type de cours. Par conséquent, une fois la machine modèle créée, vous n’avez pas besoin de la personnaliser davantage. Elle peut être publiée afin d’être utilisée par les étudiants. Sélectionnez le bouton **Publier** dans la page du modèle pour publier le modèle dans le labo.  

## <a name="cost"></a>Coût
Si vous souhaitez estimer le coût de ce labo, vous pouvez utiliser l’exemple suivant : 

Pour une classe de 25 étudiants avec 20 heures de cours planifiées et un quota de 10 heures pour le travail à la maison ou les devoirs, le prix du labo est le suivant : 25 étudiants * (20 + 10) heures * 139 unités Lab * 0,01 USD par heure = 1 042,5 USD

Pour plus d’informations sur les tarifs, consultez [Tarification Azure Lab Services](https://azure.microsoft.com/pricing/details/lab-services/).

## <a name="conclusion"></a>Conclusion
Cet article vous a présenté les étapes de création d’un labo pour le cours sur le traitement en langage naturel. Vous pouvez utiliser une configuration similaire pour d’autres cours de deep learning.

## <a name="next-steps"></a>Étapes suivantes
Les étapes suivantes sont communes à la configuration de n’importe quel labo :

- [Ajout d'utilisateurs](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Définir un quota](how-to-configure-student-usage.md#set-quotas-for-users)
- [Définir un calendrier](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [Envoyer par mail des liens d’inscription aux étudiants](how-to-configure-student-usage.md#send-invitations-to-users) 

