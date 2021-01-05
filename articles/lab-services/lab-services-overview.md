---
title: À propos d’Azure Lab Services | Microsoft Docs
description: Découvrez comment Lab Services facilite la création, la gestion et la sécurisation de laboratoires grâce à des machines virtuelles pouvant être utilisées par les développeurs, les testeurs, les enseignants et les étudiants, entre autres.
ms.topic: overview
ms.date: 09/16/2020
ms.openlocfilehash: ad17ebb3a803a15d1ac9ef8cb71cf8ca7976243b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333934"
---
# <a name="an-introduction-to-azure-lab-services"></a>Présentation d’Azure Lab Services
**Azure Lab Services** vous permet de créer des labos dont l’infrastructure est managée par Azure. Actuellement, le labo de classe est le seul type de labo managé pris en charge par Azure Lab Services. Le service lui-même gère toute l’infrastructure d’un type de labo managé, du lancement des machines virtuelles à la gestion des erreurs, en passant par la mise à l’échelle de cette infrastructure. Une fois qu’un administrateur informatique a créé un compte de labo dans Azure Lab Services, un formateur peut rapidement configurer un labo pour sa classe, spécifier le nombre et le type des machines virtuelles nécessaires pour les exercices de la classe et ajouter des utilisateurs à la classe. Une fois qu’un utilisateur s’est inscrit à la classe, il peut accéder à la machine virtuelle pour effectuer les exercices de la classe.  

## <a name="key-capabilities"></a>Fonctionnalités clés
Azure Lab Services prend en charge les fonctionnalités clés suivantes :

- **Configuration rapide et flexible d’un laboratoire**. Grâce à Azure Lab Services, les propriétaires de laboratoire peuvent rapidement configurer un laboratoire selon leurs besoins. Le service permet de prendre en charge tous les travaux d’infrastructure Azure pour les types de labo managé. Le service intègre une mise à l’échelle et une résilience de l’infrastructure pour les laboratoires que le service gère pour vous.
- **Expérience simplifiée pour les utilisateurs du laboratoire**. Les utilisateurs de labo peuvent s’inscrire à un labo avec un code d’inscription et peuvent y accéder à tout moment pour utiliser les ressources qu’il contient. 
- **Optimisation et analyse des coûts**. Un propriétaire du laboratoire peut définir des planifications de laboratoire pour arrêter et démarrer automatiquement des machines virtuelles. Le propriétaire du labo peut définir une planification pour spécifier les créneaux pendant lesquelles les machines virtuelles du labo sont accessibles aux utilisateurs. Il peut également définir des stratégies d’utilisation par utilisateur ou par labo pour optimiser les coûts. 

Si vous voulez simplement ajouter ce dont vous avez besoin dans un labo et laisser le service configurer et gérer l’infrastructure requise, choisissez l’un des **types de labos managés**. Actuellement, le **labo de salle de classe** est le seul type de labo managé que vous pouvez créer avec Azure Lab Services.

Les sections suivantes décrivent plus en détail ces laboratoires. 

## <a name="managed-lab-types"></a>Types de laboratoires gérés
Azure Lab Services vous permet de créer des laboratoires dont l’infrastructure est gérée par Azure. Dans cet article, ces labos sont appelés types de labos managés. Les types de labos managés se déclinent en différents types adaptés à différents besoins. Le seul type de labo managé pris en charge est le **labo de classe**. 

Les types de labos managés permettent de commencer sans délai, avec une procédure d’installation minimale. Le service lui-même gère toute l’infrastructure du labo, du lancement des machines virtuelles à la gestion des erreurs, en passant par la mise à l’échelle de cette infrastructure. Pour créer un type de labo managé, par exemple un labo de salle de classe, commencez par créer un compte Lab pour votre organisation. Il sert de compte central, dans lequel sont gérés tous les laboratoires de l’entreprise. 

Lorsque vous créez et utilisez des ressources Azure dans ces types de labos managés, le service crée et gère les ressources dans des abonnements Microsoft internes, et non dans votre propre abonnement Azure. Le service effectue le suivi de l’utilisation de ces ressources dans des abonnements Microsoft internes, utilisation qui est refacturée à votre abonnement Azure associé au compte Lab.   

Voici quelques **cas d’usage des types de labos managés** : 

- Proposer aux étudiants un laboratoire de machines virtuelles configurées précisément pour répondre aux besoins de la classe ; Donner à chacun un temps limité d’utilisation des machines virtuelles pour le travail à la maison ou les projets personnels.
- Configurer un pool de machines virtuelles offrant des performances élevées en calcul pour effectuer des recherches nécessitant beaucoup de ressources système ou de graphismes. Exécutez les machines virtuelles suivant vos besoins, et nettoyez-les une fois que vous avez terminé. 
- Déplacer vers le cloud l’ordinateur physique qui servait de laboratoire à votre établissement. Redimensionner automatiquement et exclusivement le nombre de machines virtuelles à la limite maximale d’utilisation et au seuil de coûts définis sur le laboratoire.  
- Configurer rapidement un laboratoire de machines virtuelles pour héberger un hackathon. Supprimez le laboratoire d’un simple clic une fois que vous avez terminé. 

## <a name="next-steps"></a>Étapes suivantes
Consultez les tutoriels suivants pour obtenir des instructions pas à pas sur la création d’un compte labo et la création d’un labo de classe.

- [Tutoriel : Configurer un compte lab](tutorial-setup-lab-account.md)
- [Tutoriel : Créer un labo de classe](tutorial-setup-classroom-lab.md)
