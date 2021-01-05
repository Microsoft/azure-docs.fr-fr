---
title: Renforcement du réseau adaptatif dans Azure Security Center | Microsoft Docs
description: Découvrez comment utiliser des modèles de trafic réels pour renforcer vos règles de groupes de sécurité réseau (NSG) et améliorer votre position de sécurité.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 09d62d23-ab32-41f0-a5cf-8d80578181dd
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/11/2020
ms.author: memildin
ms.openlocfilehash: e2b17e15c5548b4c9b93a62a7d4dfe62ff44404c
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341751"
---
# <a name="adaptive-network-hardening-in-azure-security-center"></a>Renforcement du réseau adaptatif dans Azure Security Center
Découvrez comment configurer le renforcement du réseau adaptatif dans Security Center.

## <a name="availability"></a>Disponibilité
|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|Nécessite [Azure Defender pour les serveurs](defender-for-servers-introduction.md)|
|Rôles et autorisations obligatoires :|Autorisations en écriture sur les groupes de sécurité de la machine|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||

## <a name="what-is-adaptive-network-hardening"></a>Qu’est-ce que le renforcement du réseau adaptatif ?
L’application de [groupes de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md) pour filtrer le trafic vers et depuis des ressources améliore votre posture de sécurité réseau. Il peut toutefois rester des cas dans lesquels le trafic réel qui transite via le groupe de sécurité réseau est un sous-ensemble des règles NSG définies. Dans ces cas, une amélioration supplémentaire de la posture de sécurité est possible en renforçant les règles NSG en fonction des modèles de trafic réel.

Le renforcement du réseau adaptatif fournit des recommandations visant à renforcer davantage les règles NSG. Il utilise un algorithme de Machine Learning factorisé dans le trafic réel, appelé une configuration approuvée, l’intelligence des menaces et d’autres indicateurs de compromis, puis fournit des suggestions pour autoriser uniquement le trafic provenant de tuples IP/port spécifiques.

Par exemple, supposons que la règle NSG existante consiste à autoriser le trafic provenant de 140.20.30.10/24 sur le port 22. En se basant sur l’analyse du trafic, le renforcement du réseau adaptatif peut recommander de limiter la plage pour autoriser le trafic provenant de 140.23.30.10/29 et de refuser tout autre trafic sur ce port.

>[!Note]
> Les recommandations de renforcement du réseau adaptatif sont uniquement prises en charge sur les ports spécifiques suivants (pour UDP et TCP) : 13, 17, 19, 22, 23, 53, 69, 81, 111, 119, 123, 135, 137, 138, 139, 161, 162, 389, 445, 512, 514, 593, 636, 873, 1433, 1434, 1900, 2049, 2301, 2323, 2381, 3268, 3306, 3389, 4333, 5353, 5432, 5555, 5800, 5900, 5900, 5985, 5986, 6379, 6379, 7000, 7001, 7199, 8081, 8089, 8545, 9042, 9160, 9300, 11211, 16379, 26379, 27017, 37215


## <a name="view-and-manage-hardening-alerts-and-rules"></a>Consulter et gérer les alertes et les règles de renforcement

1. Dans le menu de Security Center, ouvrez le tableau de bord **Azure Defender**, puis sélectionnez la vignette de renforcement du réseau adaptatif (1) ou l’élément du panneau des insights relatif au renforcement du réseau adaptatif (2). 

    :::image type="content" source="./media/security-center-adaptive-network-hardening/traffic-hardening.png" alt-text="Accès aux outils de renforcement du réseau adaptatif" lightbox="./media/security-center-adaptive-network-hardening/traffic-hardening.png":::

    > [!TIP]
    > Le panneau des insights affiche le pourcentage de vos machines virtuelles actuellement protégées par le renforcement du réseau adaptatif. 

1. La page de détails de la recommandation **Les recommandations de renforcement de réseau adaptatif doivent être appliquées sur les machines virtuelles accessibles à partir d’Internet** s’ouvre avec vos machines virtuelles réseau regroupées sous trois onglets :
   * **Ressources non saines** : machines virtuelles ayant actuellement des recommandations et des alertes qui ont été déclenchées en exécutant l’algorithme de renforcement du réseau adaptatif. 
   * **Ressources saines** : Machines virtuelles sans alertes ni suggestions.
   * **Ressources non analysées** : machines virtuelles dont l’algorithme de renforcement du réseau adaptatif ne peut pas être exécuté pour l’une des raisons suivantes :
      * **Les machines virtuelles sont des machines virtuelles Classic** : Seules les ressources Azure Resource Manager sont prises en charge.
      * **Les données disponibles sont insuffisantes** : Pour générer des suggestions de renforcement du trafic précises, Security Center requiert au moins 30 jours de données de trafic.
      * **La machine virtuelle n’est pas protégée par Azure Defender** : Seules les machines virtuelles protégées avec [Azure Defender pour les serveurs](defender-for-servers-introduction.md) peuvent bénéficier de cette fonctionnalité.

    :::image type="content" source="./media/security-center-adaptive-network-hardening/recommendation-details-page.png" alt-text="Accès aux outils de renforcement du réseau adaptatif":::

1. Dans l’onglet **Ressources non saines**, sélectionnez une machine virtuelle pour afficher ses alertes et les règles de renforcement recommandées à appliquer.

    - L’onglet **Règles** liste les règles que le renforcement du réseau adaptatif vous recommande d’ajouter.
    - L’onglet **Alertes** répertorie les alertes qui ont été générées en raison de trafic, qui traverse la ressource, et qui ne se trouve pas dans la plages d’adresses IP autorisée dans les règles recommandées.

1. Vous pouvez éventuellement modifier les règles :

    - [Modifier une règle](#modify-rule)
    - [Supprimer une règle](#delete-rule) 
    - [Ajouter une règle](#add-rule)

3. Sélectionnez les règles que vous souhaitez appliquer sur le groupe de sécurité réseau, puis cliquez sur **Appliquer**.

    > [!TIP]
    > Si la valeur « Aucune » est indiquée pour les plages d’adresses IP sources autorisées, cela signifie que la règle recommandée est une règle de *refus* ; sinon, il s’agit d’une règle d’*autorisation*.

    :::image type="content" source="./media/security-center-adaptive-network-hardening/hardening-alerts.png" alt-text="Accès aux outils de renforcement du réseau adaptatif":::

      > [!NOTE]
      > Les règles appliquées sont ajoutées aux groupes de sécurité réseau qui protègent la machine virtuelle. (Une machine virtuelle peut être protégée par un groupe de sécurité réseau qui est associé à sa carte réseau, ou le sous-réseau dans lequel réside la machine virtuelle, ou les deux)

### <a name="modify-a-rule"></a>Modifier une règle <a name ="modify-rule"></a>

Vous souhaiterez peut-être modifier les paramètres d’une règle qui a été recommandée. Par exemple, vous souhaiterez modifier les plages d’IP recommandées.

Voici des instructions importantes relatives à la modification d’une règle de renforcement du réseau adaptatif :

* Vous pouvez modifier les paramètres de règles « allow » uniquement. 
* Vous ne pouvez pas modifier des règles « allow » en règles « deny ». 

  > [!NOTE]
  > La création et la modification de règles « deny » sont effectuées directement sur le groupe de sécurité réseau. Pour plus d’informations, consultez [Créer, changer ou supprimer un groupe de sécurité réseau](../virtual-network/manage-network-security-group.md).

* Une règle **Deny all traffic** (Refuser tout le trafic) est la seule règle de type « deny » qui serait répertoriée ici, et elle ne peut pas être modifiée. Vous pouvez toutefois la supprimer (consultez [Supprimer une règle](#delete-rule)).
  > [!NOTE]
  > Une règle **Deny all traffic** (Refuser tout le trafic) est recommandée lorsque, suite à l’exécution de l’algorithme, Security Center n’identifie pas de trafic qui doit être autorisé en fonction de la configuration de groupe de sécurité réseau existante. Par conséquent, la règle recommandée consiste à refuser tout le trafic vers le port spécifié. Le nom de ce type de règle apparaît comme « *System Generated* » (Généré par le système). Après l’application de cette règle, son nom réel dans le groupe de sécurité réseau sera une chaîne constituée du protocole, de la direction du trafic, de « DENY » et d’un nombre aléatoire.

*Pour modifier une règle de renforcement du réseau adaptatif :*

1. Pour modifier certains paramètres d’une règle, dans l’onglet **Règles**, cliquez sur les trois points (…) à la fin de la ligne de la règle, puis cliquez sur **Modifier**.

   ![Modification d’une règle](./media/security-center-adaptive-network-hardening/edit-hard-rule.png)

1. Dans la fenêtre **Modifier une règle**, mettez à jour les détails que vous souhaitez modifier, puis cliquez sur **Enregistrer**.

   > [!NOTE]
   > Après avoir cliqué sur **Enregistrer**, vous avez modifié avec succès de la règle. *Toutefois, vous ne l’avez pas appliquée au groupe de sécurité réseau.* Pour l’appliquer, vous devez sélectionner la règle dans la liste, puis sélectionnez **Appliquer** (comme expliqué dans l’étape suivante).

   ![Sélection d’un enregistrement](./media/security-center-adaptive-network-hardening/edit-hard-rule3.png)

3. Pour appliquer la règle mise à jour, sélectionnez la règle mise à jour dans la liste, puis cliquez sur **Appliquer**.

    ![appliquer une règle](./media/security-center-adaptive-network-hardening/enforce-hard-rule.png)

### <a name="add-a-new-rule"></a>Ajouter une nouvelle règle <a name ="add-rule"></a>

Vous pouvez ajouter une règle « allow » qui n’a pas été recommandée par Security Center.

> [!NOTE]
> Seules des règles « allow » peuvent être ajoutées ici. Si vous souhaitez ajouter des règles « deny », vous pouvez le faire directement sur le groupe de sécurité réseau. Pour plus d’informations, consultez [Créer, changer ou supprimer un groupe de sécurité réseau](../virtual-network/manage-network-security-group.md).

*Pour ajouter une règle de renforcement du réseau adaptatif :*

1. Cliquez sur **Ajouter une règle** (situé dans le coin supérieur gauche).

   ![ajouter une règle](./media/security-center-adaptive-network-hardening/add-hard-rule.png)

1. Dans la fenêtre **Nouvelle règle**, entrez les détails et cliquez sur **Ajouter**.

   > [!NOTE]
   > Après avoir cliqué sur **Ajouter**, vous avez ajouté la règle avec succès, et elle est répertoriée avec les autres règles recommandées. Toutefois, vous ne l’avez pas appliquée au groupe de sécurité réseau. Pour l’activer, vous devez sélectionner la règle dans la liste, puis cliquez sur **Appliquer** (comme expliqué dans l’étape suivante).

3. Pour appliquer la nouvelle règle, sélectionnez la nouvelle règle dans la liste, puis cliquez sur **Appliquer**.

    ![appliquer une règle](./media/security-center-adaptive-network-hardening/enforce-hard-rule.png)


### <a name="delete-a-rule"></a>Supprimer une règle <a name ="delete-rule"></a>

Si nécessaire, vous pouvez supprimer une règle recommandée pour la session active. Par exemple, vous pouvez déterminer que l’application d’une règle suggérée est susceptible de bloquer du trafic légitime.

*Pour supprimer une règle de renforcement du réseau adaptatif pour votre session actiuelle :*

1. Dans l’onglet **Règles**, cliquez sur les trois points (…) à la fin de la ligne de la règle, puis cliquez sur **Supprimer**.  

    ![Suppression d'une règle](./media/security-center-adaptive-network-hardening/delete-hard-rule.png)