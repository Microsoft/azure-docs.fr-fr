---
title: À propos des labos dans Azure Lab Services | Microsoft Docs
description: 'Découvrez comment configurer rapidement un environnement de lab de salle de classe dans le cloud : configurez un lab avec un modèle de machine virtuelle avec les logiciels nécessaires pour la classe et rendez une copie de la machine virtuelle disponible pour chaque étudiant de la classe.'
ms.topic: overview
ms.date: 06/26/2020
ms.openlocfilehash: 79d6f1cc9f2a6d8d8be872359a4677b36602eea5
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434802"
---
# <a name="introduction-to-labs"></a>Introduction aux labos
Azure Lab Services vous permet de configurer rapidement un environnement pour votre laboratoire de salle de classe dans le cloud. Un formateur crée un laboratoire de salle de classe, configure Windows ou des machines virtuelles Linux, installe les logiciels et laboratoires d’outils nécessaires dans la classe, puis les rend disponibles aux étudiants. Les étudiants de la classe se connectent à des machines virtuelles (VM) dans le laboratoire puis s’en servent pour leurs projets, devoirs et exercices en classe. 

Les labos sont des types de labos managés par Azure. Le service lui-même gère toute l’infrastructure d’un type de labo managé, de l’augmentation de la capacité des machines virtuelles à la gestion des erreurs, en passant par la mise à l’échelle de cette infrastructure. Vous spécifiez le type d’infrastructure dont vous avez besoin et installez les outils ou les logiciels requis pour la classe. 

## <a name="automatic-management-of-azure-infrastructure-and-scale"></a>Gestion automatique de l’infrastructure et de la mise à l’échelle Azure 
Azure Lab Services est un service managé, ce qui signifie que le service gère automatiquement le provisionnement et la gestion de l’infrastructure sous-jacente du labo. Ainsi, vous pouvez vous concentrer simplement sur la préparation de l’expérience de labo pour vos utilisateurs. Laissez le service gérer tout le reste et déployer les machines virtuelles de votre labo pour votre public. Mettez à l’échelle votre labo pour des centaines de machines virtuelles en un simple clic.

## <a name="simple-experience-for-your-lab-users"></a>Expérience aisée pour les utilisateurs de votre labo 
Les utilisateurs invités à votre labo bénéficient d’un accès immédiat aux ressources que vous mettez à leur disposition. Il leur suffit de se connecter pour voir la liste complète des machines virtuelles auxquelles ils ont accès dans les différents labos. Ils cliquent sur un simple bouton pour se connecter aux machines virtuelles et commencer à travailler. Ils n’ont pas besoin d’un abonnement Azure pour utiliser le service. 

## <a name="cost-optimization-and-tracking"></a>Optimisation et analyse des coûts  
Maîtrisez votre budget en contrôlant le nombre exact d’heures pendant lesquelles les utilisateurs de votre labo peuvent utiliser les machines virtuelles. Établissez des plannings dans le labo pour permettre aux utilisateurs d’utiliser les machines virtuelles uniquement pendant les créneaux horaires désignés ou configurez des heures récurrentes d’arrêt et de démarrage automatiques. Gardez une trace de l’utilisation pour chaque utilisateur et définissez des limites.

## <a name="example-class-types"></a>Exemples de types de classes
Vous pouvez configurer des laboratoires pour plusieurs types de classes avec Azure Lab Services. Pour obtenir des exemples de types de classes qu’il est possible d’utiliser avec des laboratoires dans Azure Lab Services, consultez l’article [Exemples de types de classes dans Azure Lab Services](class-types.md). 

## <a name="next-steps"></a>Étapes suivantes
Prendre en main un compte de laboratoire qui est requis pour créer un laboratoire de salle de classe à l’aide de Azure Lab Services :

- [Configurer un compte de laboratoire](tutorial-setup-lab-account.md)
