---
title: Créer, modifier ou supprimer un groupe de sécurité réseau Azure
titlesuffix: Azure Virtual Network
description: Découvrez où trouver des informations sur les règles de sécurité et sur la création, la modification ou la suppression d’un groupe de sécurité réseau.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/13/2020
ms.author: kumud
ms.openlocfilehash: 42efc2bee88f073f2a628b1d2041afcc310cb871
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91822977"
---
# <a name="create-change-or-delete-a-network-security-group"></a>Créer, changer ou supprimer un groupe de sécurité réseau

Les règles de sécurité dans les groupes de sécurité réseau permettent de filtrer le type de trafic réseau qui peut circuler vers et depuis les interfaces réseau et les sous-réseaux de réseau virtuel. Pour en savoir plus sur les groupes de sécurité réseau, consultez [Vue d’ensemble du groupe de sécurité réseau](security-overview.md). Ensuite, suivez le tutoriel [Filtrer le trafic réseau](tutorial-filter-network-traffic.md) pour gagner en expérience avec les groupes de sécurité réseau.

## <a name="before-you-begin"></a>Avant de commencer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si vous n’en avez pas, configurez un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio). Effectuez une des tâches suivantes avant de commencer les étapes décrites dans la suite de cet article :

- **Utilisateurs du portail** : Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

- **Utilisateurs de PowerShell** : exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/powershell) ou exécutez PowerShell à partir de votre ordinateur. Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. Sous l’onglet de navigateur Azure Cloud Shell, ouvrez la liste déroulante **Sélectionner un environnement**, puis choisissez **PowerShell** si ce n’est pas déjà fait.

    Si vous exécutez PowerShell localement, utilisez le module Azure PowerShell version 1.0.0 ou ultérieure. Exécutez `Get-Module -ListAvailable Az.Network` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

- **Utilisateurs de l’interface de ligne de commande Azure (CLI)**  : Exécutez les commandes dans [Azure Cloud Shell](https://shell.azure.com/bash) ou exécutez l’interface CLI à partir de votre ordinateur. Si vous exécutez l’interface Azure CLI localement, utilisez Azure CLI version 2.0.28 ou ultérieure. Exécutez `az --version` pour rechercher la version installée. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli). Exécutez `az login` pour créer une connexion avec Azure.

Le compte auquel vous vous connectez ou avec lequel vous vous connectez à Azure doit disposer du [rôle Contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou d’un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des actions appropriées listées dans [Autorisations](#permissions).

## <a name="work-with-network-security-groups"></a>Utiliser des groupes de sécurité réseau

Vous pouvez effectuer différentes opérations concernant les groupes de sécurité réseau : en créer, [les voir tous](#view-all-network-security-groups), [afficher les détails d’un des groupes](#view-details-of-a-network-security-group), [les changer](#change-a-network-security-group) et [en supprimer](#delete-a-network-security-group). Vous pouvez également [associer ou dissocier](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface) un groupe de sécurité réseau à ou d’une interface réseau ou sous-réseau.

### <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Le nombre de groupes de sécurité réseau que vous pouvez créer par abonnement et par emplacement Azure est limité. Pour en savoir plus, consultez [Abonnement Azure et les limites, quotas et contraintes des services](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Dans le menu du [Portail Azure](https://portal.azure.com) ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

2. Sélectionnez **Mise en réseau**, puis **Groupe de sécurité réseau**.

3. Dans la page **Créer un groupe de sécurité réseau**, sous l’onglet **De base**, définissez les valeurs des paramètres suivants :

    | Paramètre | Action |
    | --- | --- |
    | **Abonnement** | Choisissez votre abonnement. |
    | **Groupe de ressources** | Choisissez un groupe de ressources existant ou sélectionnez **Créer** pour en créer un. |
    | **Nom** | Entrez une chaîne de texte unique au sein d’un groupe de ressources. |
    | **Région** | Choisissez l’emplacement de votre choix. |

4. Sélectionnez **Revoir + créer**.

5. Quand le message **Validation réussie** s’affiche, sélectionnez **Créer**.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg create](/cli/azure/network/nsg#az-network-nsg-create) |
| PowerShell | [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) |

### <a name="view-all-network-security-groups"></a>Voir tous les groupes de sécurité réseau

Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**. La liste des groupes de sécurité réseau de votre abonnement s’affiche.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg list](/cli/azure/network/nsg#az-network-nsg-list) |
| PowerShell | [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) |

### <a name="view-details-of-a-network-security-group"></a>Voir les détails d’un groupe de sécurité réseau

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Sélectionnez le nom de votre groupe de sécurité réseau.

Dans la barre de menus du groupe de sécurité réseau, sous **Paramètres** vous pouvez voir les **Règles de sécurité de trafic entrant**, les **Règles de sécurité de trafic sortant**, les **Interfaces réseau** et les **Sous-réseaux** auxquels le groupe de sécurité réseau est associé.

Sous **Surveillance**, vous pouvez activer ou désactiver les **Paramètres de diagnostic**. Sous **Support + résolution des problèmes**, sélectionnez **Règles de sécurité effectives**. Pour plus d’informations, consultez [Journalisation des diagnostics pour un groupe de sécurité réseau](virtual-network-nsg-manage-log.md) et [Diagnostiquer un problème de filtre de trafic réseau de machines virtuelles](diagnose-network-traffic-filter-problem.md).

Pour en savoir plus sur les paramètres Azure courants répertoriés, consultez les articles suivants :

- [Journal d’activité](../azure-monitor/platform/platform-logs-overview.md)
- [Contrôle d’accès (IAM)](../role-based-access-control/overview.md)
- [Balises](../azure-resource-manager/management/tag-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Verrous](../azure-resource-manager/management/lock-resources.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
- [Script Automation](../azure-resource-manager/templates/export-template-portal.md)

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg show](/cli/azure/network/nsg#az-network-nsg-show) |
| PowerShell | [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) |

### <a name="change-a-network-security-group"></a>Changer un groupe de sécurité réseau

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Sélectionnez le nom du groupe de sécurité réseau que vous souhaitez changer.

Les modifications les plus courantes consistent à [ajouter une règle de sécurité](#create-a-security-rule), [supprimer une règle de sécurité](#delete-a-security-rule) et [associer un groupe de sécurité réseau à un sous-réseau ou à une interface réseau ou l’en dissocier](#associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface).

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg update](/cli/azure/network/nsg#az-network-nsg-update) |
| PowerShell | [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) |

### <a name="associate-or-dissociate-a-network-security-group-to-or-from-a-subnet-or-network-interface"></a>Associer un groupe de sécurité réseau à un sous-réseau ou à une interface réseau ou l’en dissocier

Pour associer un groupe de sécurité réseau à une interface réseau ou l’en dissocier, consultez [Associer un groupe de sécurité réseau à une interface réseau ou l’en dissocier](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group). Pour associer un groupe de sécurité réseau à un sous-réseau ou l’en dissocier, consultez [Modifier les paramètres de sous-réseau](virtual-network-manage-subnet.md#change-subnet-settings).

### <a name="delete-a-network-security-group"></a>Supprimer un groupe de sécurité réseau

Si un groupe de sécurité réseau est associé à des sous-réseaux ou à des interfaces réseau, il ne peut pas être supprimé. Dissociez un groupe de sécurité réseau de tous les sous-réseaux et interfaces réseau avant de tenter de le supprimer.

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Sélectionnez le nom du groupe de sécurité réseau que vous souhaitez supprimer.

3. Dans la barre d’outils du groupe de sécurité réseau, sélectionnez **Supprimer**. Ensuite, sélectionnez **Oui** dans la boîte de dialogue de confirmation.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg delete](/cli/azure/network/nsg#az-network-nsg-delete) |
| PowerShell | [Remove-AzNetworkSecurityGroup](/powershell/module/az.network/remove-aznetworksecuritygroup) |

## <a name="work-with-security-rules"></a>Utiliser des règles de sécurité

Un groupe de sécurité réseau contient zéro règle de sécurité, ou plus. Vous pouvez effectuer différentes opérations concernant les règles de sécurité : en créer, [les voir toutes](#view-all-security-rules), [afficher les détails d’une des règles](#view-details-of-a-security-rule), [les changer](#change-a-security-rule) et [en supprimer](#delete-a-security-rule).

### <a name="create-a-security-rule"></a>Créer une règle de sécurité

Le nombre de règles par groupe de sécurité réseau que vous pouvez créer par abonnement et par emplacement Azure est limité. Pour en savoir plus, consultez [Abonnement Azure et les limites, quotas et contraintes des services](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Sélectionnez le groupe de sécurité réseau auquel vous souhaitez ajouter une règle de sécurité.

3. Dans la barre de menus du groupe de sécurité réseau, choisissez **Règles de sécurité de trafic entrant** ou **Règles de sécurité de trafic sortant**.

    Plusieurs règles existantes sont répertoriées, y compris quelques-unes que vous n’avez peut-être pas ajoutées. Lorsque vous créez un groupe de sécurité réseau, plusieurs règles de sécurité par défaut y sont créées. Pour plus d’informations, consultez [Règles de sécurité par défaut](security-overview.md#default-security-rules).  Vous ne pouvez pas supprimer les règles de sécurité par défaut, mais vous pouvez les remplacer par des règles qui ont une priorité plus élevée.

4. <a name="security-rule-settings"></a>Sélectionnez **Ajouter**. Sélectionnez ou ajoutez des valeurs pour les paramètres suivants, puis sélectionnez **OK** :

    | Paramètre | Valeur | Détails |
    | ------- | ----- | ------- |
    | **Source** | Valeurs possibles :<ul><li>**Any**</li><li>**Adresses IP**</li><li>**Balise de service** (règle de sécurité de trafic entrant) ou **VirtualNetwork** (règle de sécurité de trafic sortant)</li><li>**Groupe de&nbsp;sécurité&nbsp;d’application**</li></ul> | <p>Si vous choisissez **Adresses IP**, vous devez également spécifier les **Plages d’adresses IP/CIDR sources**.</p><p>Si vous choisissez **Balise de service**, vous pouvez également choisir une **balise de service source**.</p><p>Si vous choisissez **Groupe de sécurité d’application**, vous devez également sélectionner un groupe de sécurité d’application existant. Si vous choisissez **Groupe de sécurité d’application** en tant que **source** et **destination**, les interfaces réseau dans les deux groupes de sécurité d’application doivent se trouver dans le même réseau virtuel.</p> |
    | **Plages d’adresses IP/CIDR sources** | Liste délimitée par des virgules d’adresses IP et de plages CIDR (Classless Interdomain Routing) | <p>Ce paramètre apparaît si vous changez **Source** en **Adresses IP**. Vous devez spécifier une valeur unique ou une liste de valeurs séparées par des virgules. Un exemple de plusieurs valeurs est `10.0.0.0/16, 192.188.1.1`. Le nombre de valeurs que vous pouvez spécifier est limité. Pour plus d’informations, consultez [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).</p><p>Si l’adresse IP que vous spécifiez est attribuée à une machine virtuelle Azure, spécifiez son adresse IP privée, et non son adresse IP publique. Azure traite les règles de sécurité sont traitées une fois l’adresse IP publique convertie en adresse IP privée pour les règles de sécurité liées au trafic entrant, mais avant de convertir l’adresse IP privée en adresse IP publique pour les règles de trafic sortant. Pour plus d’informations sur les adresses IP privées et publiques dans Azure, consultez [Types d’adresses IP](virtual-network-ip-addresses-overview-arm.md).</p> |
    | **Balise du service source** | Une balise de service dans la liste déroulante | Ce paramètre facultatif apparaît si vous définissez **Source** sur **Balise de service** pour une règle de sécurité de trafic entrant. Une balise de service est un identificateur prédéfini pour une catégorie d’adresses IP. Pour en savoir plus sur les balises de service disponibles et ce que représente chaque balise, consultez [Balises de service](security-overview.md#service-tags). |
    | **Groupe de sécurité d’application source** | Groupe de sécurité d’application source existant | Ce paramètre apparaît si vous définissez **Source** sur **Groupe de sécurité d’application**. Sélectionnez un groupe de sécurité d’application qui existe dans la même région que l’interface réseau. Découvrez comment [créer un groupe de sécurité d’application](#create-an-application-security-group). |
    | **Plages de ports source** | Valeurs possibles :<ul><li>Un port unique, par exemple `80`</li><li>Un plage de ports, par exemple `1024-65535`</li><li>Une liste séparée par des virgules de ports uniques et/ou de plages de ports, par exemple `80, 1024-65535`</li><li>Un astérisque (`*`) pour autoriser le trafic sur n’importe quel port</li></ul> | Ce paramètre spécifie les ports sur lesquels la règle autorise ou refuse le trafic. Le nombre de ports que vous pouvez spécifier est limité. Pour plus d’informations, consultez [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). |
    | **Destination** | Valeurs possibles :<ul><li>**Any**</li><li>**Adresses IP**</li><li>**Balise de service** (règle de sécurité de trafic sortant) ou **VirtualNetwork** (règle de sécurité de trafic entrant)</li><li>**Groupe de&nbsp;sécurité&nbsp;d’application**</li></ul> | <p>Si vous sélectionnez **Adresses IP**, spécifiez les **Plages d’adresses IP/CIDR de destination**.</p><p>Si vous choisissez **VirtualNetwork**, le trafic est autorisé sur toutes les adresses IP dans l’espace d’adressage du réseau virtuel. **VirtualNetwork** est une balise de service.</p><p>Si vous sélectionnez **Groupe de sécurité d’application**, vous devez également sélectionner un groupe de sécurité d’application existant. Découvrez comment [créer un groupe de sécurité d’application](#create-an-application-security-group).</p> |
    | **Plages d’adresses IP/CIDR de destination** | Liste délimitée par des virgules d’adresses IP et de plages CIDR | <p>Ce paramètre apparaît si vous changez **Destination** en **Adresses IP**. À l’image de **Source** et de **Plages d’adresses IP/CIDR sources**, vous pouvez spécifier une seule adresse ou plage, ou bien plusieurs. Le nombre de ports que vous pouvez spécifier est limité. Pour plus d’informations, consultez [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).</p><p>Si l’adresse IP que vous spécifiez est attribuée à une machine virtuelle Azure, vérifiez que vous spécifiez bien son adresse IP privée, et non son adresse IP publique. Azure traite les règles de sécurité sont traitées une fois l’adresse IP publique convertie en adresse IP privée pour les règles de sécurité liées au trafic entrant, mais avant de convertir l’adresse IP privée en adresse IP publique pour les règles de trafic sortant. Pour plus d’informations sur les adresses IP privées et publiques dans Azure, consultez [Types d’adresses IP](virtual-network-ip-addresses-overview-arm.md).</p> |
    | **Balise d’identification de destination** | Une balise de service dans la liste déroulante | Ce paramètre facultatif apparaît si vous définissez **Destination** sur **Balise de service** pour une règle de sécurité de trafic entrant. Une balise de service est un identificateur prédéfini pour une catégorie d’adresses IP. Pour en savoir plus sur les balises de service disponibles et ce que représente chaque balise, consultez [Balises de service](security-overview.md#service-tags). |
    | **Groupe de sécurité d’application de destination** | Groupe de sécurité d’application source existant | Ce paramètre apparaît si vous définissez **Destination** sur **Groupe de sécurité d’application**. Sélectionnez un groupe de sécurité d’application qui existe dans la même région que l’interface réseau. Découvrez comment [créer un groupe de sécurité d’application](#create-an-application-security-group). |
    | **Plages de ports de destination** | Valeurs possibles :<ul><li>Un port unique, par exemple `80`</li><li>Un plage de ports, par exemple `1024-65535`</li><li>Une liste séparée par des virgules de ports uniques et/ou de plages de ports, par exemple `80, 1024-65535`</li><li>Un astérisque (`*`) pour autoriser le trafic sur n’importe quel port</li></ul> | Comme pour **Plages de ports source**, vous pouvez spécifier un ou plusieurs ports et plages. Le nombre de ports que vous pouvez spécifier est limité. Pour plus d’informations, consultez [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). |
    | **Protocole** | **Any**, **TCP**, **UDP** ou **ICMP** | Vous pouvez restreindre la règle au protocole TCP (Transmission Control Protocol), UDP (User Datagram Protocol) ou ICMP (Internet Control Message Protocol). La valeur par défaut correspond à la règle qui s’applique à tous les protocoles. |
    | **Action** | **Autoriser** ou **Refuser** | Ce paramètre spécifie si cette règle autorise ou refuse l’accès pour la configuration source et de destination fournie. |
    | **Priorité** | Entrez une valeur comprise entre 100 et 4096 qui est unique pour toutes les règles de sécurité au sein du groupe de sécurité réseau. | Azure traite les règles de sécurité par ordre de priorité. Plus le numéro est faible, plus la priorité est élevée. Nous vous recommandons de laisser un écart entre les numéros de priorité quand vous créez des règles, par exemple, 100, 200, 300. Cela permet par la suite d’intercaler de nouvelles règles et de leur donner une priorité plus ou moins élevée que les règles existantes. |
    | **Nom** | Nom unique de la règle au sein du groupe de sécurité réseau | Le nom peut comprendre jusqu’à 80 caractères. Il doit commencer par une lettre ou un chiffre et se terminer par une lettre, un chiffre ou un trait de soulignement. Le nom peut contenir uniquement des lettres, des chiffres, des traits d’union, des traits de soulignement et des points. |
    | **Description** | Description texte | Vous pouvez éventuellement spécifier une description texte de la règle de sécurité. La description ne doit pas avoir plus de 140 caractères. |

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) |
| PowerShell | [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) |

### <a name="view-all-security-rules"></a>Voir toutes les règles de sécurité

Un groupe de sécurité réseau contient zéro ou plusieurs règles. Pour en savoir plus sur les informations affichées avec les règles, consultez [Vue d’ensemble des groupes de sécurité réseau](security-overview.md).

1. Accédez au [portail Azure](https://portal.azure.com) pour voir les règles des groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Dans la liste, sélectionnez le nom du groupe de sécurité réseau dont vous souhaitez voir les règles.

3. Dans la barre de menus du groupe de sécurité réseau, choisissez **Règles de sécurité de trafic entrant** ou **Règles de sécurité de trafic sortant**.

La liste contient toutes les règles que vous avez créées et les [règles de sécurité par défaut](security-overview.md#default-security-rules) du groupe de sécurité réseau.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg rule list](/cli/azure/network/nsg/rule#az-network-nsg-rule-list) |
| PowerShell | [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig) |

### <a name="view-details-of-a-security-rule"></a>Voir les détails d’une règle de sécurité

1. Accédez au [portail Azure](https://portal.azure.com) pour voir les règles des groupes de sécurité réseau. Recherchez et sélectionnez **Groupes de sécurité réseau**.

2. Dans la liste, sélectionnez le nom du groupe de sécurité réseau dont vous souhaitez voir une règle en détail.

3. Dans la barre de menus du groupe de sécurité réseau, choisissez **Règles de sécurité de trafic entrant** ou **Règles de sécurité de trafic sortant**.

4. Sélectionnez la règle dont vous souhaitez voir les détails. Pour obtenir une explication détaillée de tous les paramètres, consultez les [paramètres de règle de sécurité](#security-rule-settings).

    > [!NOTE]
    > Cette procédure s’applique uniquement aux règles de sécurité personnalisées. Cela ne fonctionne pas si vous choisissez une règle de sécurité par défaut.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg rule show](/cli/azure/network/nsg/rule#az-network-nsg-rule-show) |
| PowerShell | [Get-AzNetworkSecurityRuleConfig](/powershell/module/az.network/get-aznetworksecurityruleconfig) |

### <a name="change-a-security-rule"></a>Changer une règle de sécurité

1. Effectuez les étapes indiquées dans [Voir les détails d’une règle de sécurité](#view-details-of-a-security-rule).

2. Changez les paramètres comme vous le souhaitez, puis sélectionnez **Enregistrer**. Pour obtenir une explication détaillée de tous les paramètres, consultez les [paramètres de règle de sécurité](#security-rule-settings).

    > [!NOTE]
    > Cette procédure s’applique uniquement aux règles de sécurité personnalisées. Vous n’êtes pas autorisé à modifier les règles de sécurité par défaut.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg rule update](/cli/azure/network/nsg/rule#az-network-nsg-rule-update) |
| PowerShell | [Set-AzNetworkSecurityRuleConfig](/powershell/module/az.network/set-aznetworksecurityruleconfig) |

### <a name="delete-a-security-rule"></a>Supprimer une règle de sécurité

1. Effectuez les étapes indiquées dans [Voir les détails d’une règle de sécurité](#view-details-of-a-security-rule).

2. Sélectionnez **Supprimer**, puis cliquez sur **Oui**.

    > [!NOTE]
    > Cette procédure s’applique uniquement aux règles de sécurité personnalisées. Vous n’êtes pas autorisé à supprimer les règles de sécurité par défaut.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network nsg rule delete](/cli/azure/network/nsg/rule#az-network-nsg-rule-delete) |
| PowerShell | [Remove-AzNetworkSecurityRuleConfig](/powershell/module/az.network/remove-aznetworksecurityruleconfig) |

## <a name="work-with-application-security-groups"></a>Utiliser des groupes de sécurité d’application

Un groupe de sécurité d’application contient zéro interface réseau, ou plus. Pour en savoir plus, consultez [Groupes de sécurité d’application](security-overview.md#application-security-groups). Toutes les interfaces réseau dans un groupe de sécurité d’application doivent exister dans le même réseau virtuel. Pour savoir comment ajouter une interface réseau à un groupe de sécurité d’application, consultez [Ajouter une interface réseau à un groupe de sécurité d’application](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups).

### <a name="create-an-application-security-group"></a>Créer un groupe de sécurité d’application

1. Dans le menu du [Portail Azure](https://portal.azure.com) ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

2. Dans la zone de recherche, entrez *Groupe de sécurité d’application*.

3. Dans la page **Groupe de sécurité d’application**, sélectionnez **Créer**.

4. Dans la page **Créer un groupe de sécurité d’application**, sous l’onglet **De base**, définissez des valeurs des paramètres suivants :

    | Paramètre | Action |
    | --- | --- |
    | **Abonnement** | Choisissez votre abonnement. |
    | **Groupe de ressources** | Choisissez un groupe de ressources existant ou sélectionnez **Créer** pour en créer un. |
    | **Nom** | Entrez une chaîne de texte unique au sein d’un groupe de ressources. |
    | **Région** | Choisissez l’emplacement de votre choix. |

5. Sélectionnez **Revoir + créer**.

6. Dans l’onglet **Revoir + créer**, une fois que vous avez vu **Validation réussie**, sélectionnez **Créer**.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network asg create](/cli/azure/network/asg#az-network-asg-create) |
| PowerShell | [New-AzApplicationSecurityGroup](/powershell/module/az.network/new-azapplicationsecuritygroup) |

### <a name="view-all-application-security-groups"></a>Voir tous les groupes de sécurité d’application

Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité d’application. Recherchez et sélectionnez **Groupes de sécurité d’application**. Le portail Azure affiche la liste de vos groupes de sécurité d’application.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network asg list](/cli/azure/network/asg#az-network-asg-list) |
| PowerShell | [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup) |

### <a name="view-details-of-a-specific-application-security-group"></a>Voir les détails d’un groupe de sécurité d’application spécifique

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité application. Recherchez et sélectionnez **Groupes de sécurité d’application**.

2. Sélectionnez le nom du groupe de sécurité d’application dont vous souhaitez afficher les détails.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network asg show](/cli/azure/network/asg#az-network-asg-show) |
| PowerShell | [Get-AzApplicationSecurityGroup](/powershell/module/az.network/get-azapplicationsecuritygroup) |

### <a name="change-an-application-security-group"></a>Changer un groupe de sécurité d’application

1. Accédez au [portail Azure](https://portal.azure.com) pour voir vos groupes de sécurité application. Recherchez et sélectionnez **Groupes de sécurité d’application**.

2. Sélectionnez le nom du groupe de sécurité d’application que vous souhaitez modifier.

3. Sélectionnez **Modifier** en regard du paramètre que vous souhaitez modifier. Par exemple, vous pouvez ajouter ou supprimer des **balises** ou vous pouvez modifier le **groupe de ressources** ou l’**abonnement**.

    > [!NOTE]
    > Vous ne pouvez pas modifier l’emplacement.

    Dans la barre de menus, vous pouvez également sélectionner **Contrôle d’accès (IAM)** . Dasn la page **Contrôle d’accès (IAM)** , vous pouvez ajouter ou supprimer des autorisations pour le groupe de sécurité d’application.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network asg update](/cli/azure/network/asg#az-network-asg-update) |
| PowerShell | Pas de cmdlet PowerShell |

### <a name="delete-an-application-security-group"></a>Supprimer un groupe de sécurité d’application

Vous ne pouvez pas supprimer un groupe de sécurité d’application s’il contient des interfaces réseau. Pour supprimer toutes les interfaces réseau du groupe de sécurité d’application, vous pouvez modifier les paramètres d’interface réseau ou supprimer les interfaces réseau. Pour plus d’informations, consultez [Ajout ou suppression dans les groupes de sécurité d’application](virtual-network-network-interface.md#add-to-or-remove-from-application-security-groups) ou [Supprimer une interface réseau](virtual-network-network-interface.md#delete-a-network-interface).

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer vos groupes de sécurité d’application. Recherchez et sélectionnez **Groupes de sécurité d’application**.

2. Sélectionnez le nom du groupe de sécurité d’application que vous souhaitez supprimer.

3. Sélectionnez **Supprimer**, puis sélectionnez **Oui** pour supprimer le groupe de sécurité d’application.

#### <a name="commands"></a>Commandes

| Outil | Commande |
| ---- | ------- |
| Azure CLI | [az network asg delete](/cli/azure/network/asg#az-network-asg-delete) |
| PowerShell | [Remove-AzApplicationSecurityGroup](/powershell/module/az.network/remove-azapplicationsecuritygroup) |

## <a name="permissions"></a>Autorisations

Pour effectuer des tâches sur des groupes de sécurité réseau, des règles de sécurité et des groupes de sécurité d’application, votre compte doit disposer du rôle de [contributeur réseau](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou d’un rôle [personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) disposant des autorisations appropriées, comme présentées dans les tableaux suivants :

### <a name="network-security-group"></a>Groupe de sécurité réseau

| Action                                                        |   Nom                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/read                  |   Obtenir un groupe de sécurité réseau                                          |
| Microsoft.Network/networkSecurityGroups/write                 |   Créer ou mettre à jour un groupe de sécurité réseau                             |
| Microsoft.Network/networkSecurityGroups/delete                |   Supprimer un groupe de sécurité réseau                                       |
| Microsoft.Network/networkSecurityGroups/join/action           |   Associez un groupe de sécurité réseau à un sous-réseau ou à une interface réseau 

### <a name="network-security-group-rule"></a>Règle de groupe de sécurité réseau

| Action                                                        |   Nom                                                                |
|-------------------------------------------------------------- |   -------------------------------------------                         |
| Microsoft.Network/networkSecurityGroups/securityRules/read            |   Obtenir une règle                                                            |
| Microsoft.Network/networkSecurityGroups/securityRules/write           |   Créer ou mettre à jour une règle                                               |
| Microsoft.Network/networkSecurityGroups/securityRules/delete          |   Supprimer une règle                                                         |

### <a name="application-security-group"></a>Groupe de sécurité d’application

| Action                                                                     | Nom                                                     |
| --------------------------------------------------------------             | -------------------------------------------              |
| Microsoft.Network/applicationSecurityGroups/joinIpConfiguration/action     | Joindre une configuration IP aux groupes de sécurité d’application|
| Microsoft.Network/applicationSecurityGroups/joinNetworkSecurityRule/action | Joindre une règle de sécurité aux groupes de sécurité d’application    |
| Microsoft.Network/applicationSecurityGroups/read                           | Obtenir un groupe de sécurité d’application                        |
| Microsoft.Network/applicationSecurityGroups/write                          | Créer ou mettre à jour un groupe de sécurité d’application           |
| Microsoft.Network/applicationSecurityGroups/delete                         | Supprimer un groupe de sécurité d’application                     |

## <a name="next-steps"></a>Étapes suivantes

- Créer un réseau ou un groupe de sécurité d’application avec les exemples de scripts [PowerShell](powershell-samples.md) ou [Azure CLI](cli-samples.md), ou des [modèles Azure Resource Manager](template-samples.md)
- Créer et attribuer des [définitions Azure Policy](policy-samples.md) pour les réseaux virtuels
