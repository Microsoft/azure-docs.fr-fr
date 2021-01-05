---
title: Azure VMware Solution by CloudSimple - Configurer les tables et les règles de pare-feu
description: Explique comment configurer des tables et des règles de pare-feu de cloud privé pour limiter le trafic sur les sous-réseaux et les VLAN.
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 986f4b0da7254ebac3725a704f32af785c72fbcc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85846874"
---
# <a name="set-up-firewall-tables-and-rules-for-private-clouds"></a>Configurer des tables et des règles de pare-feu pour des clouds privés

Les tables de pare-feu et les règles associées vous permettent de spécifier des restrictions à appliquer au trafic sur des sous-réseaux et des VLAN particuliers.

* Vous pouvez associer un sous-réseau à une table de pare-feu.
* Vous pouvez associer une table de pare-feu à plusieurs sous-réseaux.

## <a name="add-a-new-firewall-table"></a>Ajouter une nouvelle table de pare-feu

1. [Accédez au portail CloudSimple](access-cloudsimple-portal.md), puis sélectionnez **Network** dans le menu latéral.
2. Sélectionnez **Firewall Tables** (Tables de pare-feu).
3. Sélectionnez **Create firewall table** (Créer une table de pare-feu).

    ![Page de VLAN/sous-réseau](media/firewall-tables-page.png)

4. Entrez un nom pour la table.
5. Une règle par défaut est listée pour la table. Cliquez sur **Create New Rule** (Créer une règle) pour créer une règle supplémentaire. Pour plus d’informations, consultez la procédure suivante.
6. Cliquez sur **Terminé** pour enregistrer la table de pare-feu.

> [!IMPORTANT]
> Il est possible de créer jusqu’à deux tables de pare-feu par cloud privé.

## <a name="firewall-rules"></a>Règles de pare-feu

Les règles de pare-feu déterminent la manière dont le pare-feu traite certains types de trafic. L’onglet **Rules** (Règles) de la table de pare-feu sélectionnée liste toutes les règles associées.

![Table de règles de pare-feu](media/firewall-rules-tab.png)

## <a name="create-a-firewall-rule"></a>Créer une règle de pare-feu

1. Affichez les paramètres permettant de créer une règle de pare-feu en procédant de l’une des manières suivantes :
    * Cliquez sur **Add Rule** (Ajouter une règle) au moment de créer une table de pare-feu.
    * Sélectionnez une table de pare-feu particulière dans la page **Network > Firewall Tables** (Réseau > Tables de pare-feu), puis cliquez sur **Create new firewall rule** (Créer une règle de pare-feu).
2. Configurez la règle comme suit :
    * **Nom**. Nommez la règle.
    * **Priorité**. Affectez une priorité à la règle. Les règles qui comportent des numéros inférieurs sont exécutées en premier.
    * **Traffic type** (Type de trafic). Indiquez si la règle s’applique au trafic du cloud privé, au trafic Internet ou au trafic du VPN (sans état), ou si elle s’applique à une adresse IP publique (avec état).
    * **Protocole**. Sélectionnez le protocole couvert par la règle (TCP, UDP ou tout autre protocole).
    * **Direction**. Indiquez si la règle s’applique au trafic entrant ou sortant. Vous devez définir des règles distinctes pour le trafic entrant et sortant.
    * **Action**. Sélectionnez l’action à entreprendre si la règle a une correspondance (autoriser ou refuser).
    * **Source**. Spécifiez les sources couvertes par la règle (bloc CIDR, interne ou toute autre source).
    * **Source port range** (Plage de ports sources). Spécifiez la plage de ports soumise à la règle.
    * **Direction**. Sélectionnez le type entrant ou sortant.
    * **Destination**. Spécifiez les destinations couvertes par la règle (bloc CIDR, interne ou toute autre source).
    * **Source port range** (Plage de ports sources). Spécifiez la plage de ports soumise à la règle.

    ![Ajout d’une règle dans la table de pare-feu](media/firewall-rule-create.png)

3. Cliquez sur **Terminé** pour enregistrer la règle et l’ajouter à la liste des règles de la table de pare-feu.

> [!IMPORTANT]
> Chaque table de pare-feu peut comporter jusqu’à 10 règles de trafic entrant et 20 règles de trafic sortant. Il est possible d’augmenter ces limites en [contactant le support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="attach-vlanssubnets"></a><a name="attach-vlans-subnet"></a>Attacher des VLAN/sous-réseaux

Une fois que vous avez défini une table de pare-feu, vous pouvez spécifier les sous-réseaux soumis aux règles de la table.

1. Dans la page **Network**(Réseau) > **Firewall Tables**(Tables de pare-feu), sélectionnez une table de pare-feu.
2. Ouvrez l’onglet **Attached VLANs/Subnet** (VLAN/Sous-réseaux attachés).
3. Cliquez sur **Attach to a VLAN/Subnet** (Attacher à un VLAN/sous-réseau).
4. Sélectionnez le cloud privé et le VLAN. Le nom de sous-réseau associé et le bloc CIDR sont affichés.
5. Cliquez sur **Envoyer**.
