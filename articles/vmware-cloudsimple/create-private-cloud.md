---
title: Azure VMware Solution by CloudSimple - Créer un cloud privé CloudSimple
description: Décrit comment créer un cloud privé CloudSimple pour étendre les charges de travail VMware sur le cloud à l’aide avec la continuité des activités et la flexibilité opérationnelle
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4f700ac34b6c6e2a651366bee7dd1785c608064f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77024787"
---
# <a name="create-a-cloudsimple-private-cloud"></a>Créer un cloud privé CloudSimple

Un cloud privé est une pile VMware isolée qui prend en charge les hôtes ESXi, vCenter, vSAN et NSX. Les clouds privés sont managés via le portail CloudSimple. Ils ont leur serveur vCenter situé dans son propre domaine de gestion. La pile s’exécute sur des nœuds dédiés et des nœuds matériels nus (bare metal) isolés.

La création d’un cloud privé vous permet de répondre à divers besoins courants pour l’infrastructure réseau :

* **Croissance**. Si vous avez atteint un point d’actualisation du matériel pour votre infrastructure existante, un cloud privé vous permet de développer sans aucun investissement matériel nouveau requis.

* **Expansion rapide**. En cas de besoins en capacité temporaire ou non planifiée, un cloud privé vous permet de créer de la capacité supplémentaire sans délai.

* **Protection accrue**. Avec un cloud privé de trois nœuds ou plus, vous bénéficiez de la redondance automatique et de la protection haute disponibilité.

* **Besoins d’infrastructure à long terme**. Si vos centres de données sont au maximum de leur capacité ou que vous souhaitez restructurer pour réduire vos coûts, un cloud privé vous permet de mettre hors service des centres de données et de migrer vers une solution cloud tout en maintenant la compatibilité avec les opérations de votre entreprise.

Lorsque vous créez un cloud privé, vous obtenez un cluster vSphere unique et toutes les machines virtuelles qui sont créées dans ce cluster de gestion.

## <a name="before-you-begin"></a>Avant de commencer

Les nœuds doivent être configurés avant de pouvoir créer votre cloud privé. Pour plus d’informations sur l’approvisionnement des nœuds, consultez l’article [Approvisionner des nœuds pour Azure VMware Solution by CloudSimple](create-nodes.md).

Allouez une plage CIDR pour les sous-réseaux vSphere/vSAN pour le cloud privé. Un cloud privé est créé sous la forme d’un environnement isolé de la pile VMware (avec hôtes ESXi, vCenter, vSAN et NSX) géré par un serveur vCenter. Les composants de gestion sont déployés sur le réseau sélectionné pour la plage CIDR de sous-réseaux vSphere/vSAN. La plage CIDR du réseau est divisée en plusieurs sous-réseaux pendant le déploiement. L’espace d’adressage du sous-réseau vSphere/vSAN doit être unique. Il ne doit chevaucher aucun réseau qui communique avec l’environnement CloudSimple. Les réseaux qui communiquent avec CloudSimple incluent les réseaux locaux et les réseaux virtuels Azure. Pour plus d’informations sur les sous-réseaux vSphere/vSAN, consultez Vue d’ensemble des réseaux locaux virtuels et des sous-réseaux.

* Préfixe de plage CIDR de sous-réseaux vSphere/vSAN minimum : /24
* Préfixe de plage CIDR de sous-réseaux vSphere/vSAN maximum : /21


## <a name="access-the-cloudsimple-portal"></a>Accéder au portail CloudSimple

Accédez au [portail CloudSimple](access-cloudsimple-portal.md).

## <a name="create-a-new-private-cloud"></a>Créer un nouveau cloud privé

1. Sélectionnez **Tous les services**.
2. Recherchez **Services CloudSimple**.
3. Sélectionnez le service CloudSimple où vous voulez créer votre cloud privé.
4. Dans la **vue d’ensemble**, cliquez sur **Créer un cloud privé** afin d’ouvrir un nouvel onglet de navigateur pour le portail CloudSimple. Si vous y êtes invité, connectez-vous avec vos informations d’identification Azure.

    ![Créer un cloud privé depuis Azure](media/create-private-cloud-from-azure.png)

5. Dans le portail CloudSimple, attribuez un nom à votre cloud privé.
6. Sélectionnez l’**Emplacement** de votre cloud privé.
7. Sélectionnez le **Type de nœud**, conformément à ce que vous avez approvisionné sur Azure.
8. Spécifiez le **Nombre de nœuds**.  Au moins trois nœuds sont nécessaires pour créer un cloud privé.

    ![Créer un cloud privé - Informations de base](media/create-private-cloud-basic-info.png)

9. Cliquez sur **Suivant : Options avancées**.
10. Entrez la plage CIDR pour les sous-réseaux vSphere/vSAN. Assurez-vous que la plage CIDR ne chevauche pas votre réseau local, d’autres sous-réseaux Azure (réseaux virtuels) ou le sous-réseau de la passerelle.

    **Options de la plage CIDR :** /24, /23, /22 ou /21. Une plage CIDR /24 prend en charge jusqu'à neuf nœuds, une plage /23 jusqu'à 41 nœuds et une plage /22 ou /21 jusqu'à 64 nœuds (le nombre maximal de nœuds dans un cloud privé).

    > [!IMPORTANT]
    > L’utilisation des adresses IP de la plage CIDR vSphere/vSAN est réservée à l’infrastructure du cloud privé.  N’utilisez pas d’adresses IP de cette plage pour les machines virtuelles.

11. Cliquez sur **Suivant : Examiner et créer**.
12. Passez en revue les paramètres. Si vous avez besoin modifier des paramètres, cliquez sur **Précédent**.
13. Cliquez sur **Créer**.

Le processus de provisionnement du cloud privé démarre. Le provisionnement du cloud privé peut prendre jusqu’à deux heures.

Pour plus d’instructions sur le développement d’un Cloud privé existant, consultez [Développer un cloud privé](expand-private-cloud.md).
