---
title: 'Créer une connexion entre réseaux virtuels : classique : Portail Azure'
description: Interconnectez des réseaux virtuels Azure à l’aide de PowerShell et du portail Azure.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 10/15/2020
ms.author: cherylmc
ms.openlocfilehash: 0d81e0474d898ffee7f128c0bcea61f077c3d758
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103218"
---
# <a name="configure-a-vnet-to-vnet-connection-classic"></a>Configurer une connexion de réseau virtuel à réseau virtuel (classique)

Cet article vous aide à créer une connexion de passerelle VPN entre des réseaux virtuels. Les réseaux virtuels peuvent être situés dans des régions identiques ou différentes et appartenir à des abonnements identiques ou différents.

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-portal-classic/v2vclassic.png" alt-text="Diagramme représentant une architecture de réseau virtuel à réseau virtuel classique":::

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

Les étapes mentionnées dans cet article s’appliquent au modèle de déploiement classique et au portail Azure. Vous pouvez également créer cette configuration à l’aide d’un autre outil ou modèle de déploiement en sélectionnant une option différente dans la liste suivante :

> [!div class="op_single_selector"]
> * [Classique](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Resource Manager](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [Connecter des réseaux virtuels selon différents modèles de déploiement](vpn-gateway-connect-different-deployment-models-portal.md)
>
>

## <a name="about-vnet-to-vnet-connections"></a>À propos des connexions de réseau virtuel à réseau virtuel

Dans le modèle de déploiement classique qui utilise une passerelle VPN, la connexion entre deux réseaux virtuels est similaire à la connexion d’un réseau virtuel à un emplacement de site local. Les deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé utilisant Ipsec/IKE.

Les réseaux virtuels que vous connectez peuvent être situés dans différents abonnements et différentes régions. Vous pouvez combiner une communication de réseau virtuel à réseau virtuel avec des configurations multisites. Vous établissez ainsi des topologies réseau qui combinent une connectivité entre différents locaux et une connectivité entre différents réseaux virtuels.

:::image type="content" source="./media/vpn-gateway-howto-vnet-vnet-portal-classic/aboutconnections.png" alt-text="Diagramme représentant une architecture de réseau virtuel à réseau virtuel classique":::

### <a name="why-connect-virtual-networks"></a><a name="why"></a>Pourquoi connecter des réseaux virtuels ?

Vous pouvez décider de connecter des réseaux virtuels pour les raisons suivantes :

* **Géo-redondance et présence géographique dans plusieurs régions**

  * Vous pouvez configurer la géo-réplication ou la synchronisation avec une connectivité sécurisée sans passer par les points de terminaison accessibles sur Internet.
  * Avec l’Azure Load Balancer et les technologies de clustering Microsoft ou tierces, vous pouvez configurer une charge de travail hautement disponible avec la géoredondance dans plusieurs régions Azure. Vous pouvez par exemple configurer SQL Always On avec des groupes de disponibilité répartis dans différentes régions Azure.
* **Applications multiniveaux régionales avec une forte limite d’isolement**

  * Dans la même région, vous pouvez configurer des applications multiniveaux avec plusieurs réseaux virtuels interconnectés avec une isolation renforcée et une communication sécurisée entre les niveaux.
* **Communication interorganisationnelle entre plusieurs abonnements dans Azure**

  * Si vous avez plusieurs abonnements Azure, vous pouvez désormais interconnecter des charges de travail de différents abonnements en toute sécurité entre des réseaux virtuels.
  * Pour les entreprises ou prestataires de services, il est désormais possible d’activer la communication interorganisationnelle avec une technologie VPN sécurisée au sein d’Azure.

Pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel, consultez la partie [Interconnexion de réseaux virtuels](#faq) à la fin de cet article.

## <a name="prerequisites"></a>Prérequis

Nous utilisons le portail pour la plupart des étapes, mais vous devez utiliser PowerShell pour créer les connexions entre les réseaux virtuels. Les connexions ne peuvent pas être créées à l'aide du portail Azure car rien ne permet d'y spécifier la clé partagée. [!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="planning"></a><a name="planning"></a>Planification

Il est important de choisir les plages que vous utiliserez pour configurer vos réseaux virtuels. Pour cette configuration, vous devez vous assurer que vos plages de réseau virtuel ne se chevauchent pas entre elles ou avec un réseau local auquel elles se connectent.

### <a name="vnets"></a><a name="vnet"></a>Réseaux virtuels

Dans le cadre de cet exercice, nous utilisons les exemples de valeurs suivants :

**Valeurs pour TestVNet1**

Nom : TestVNet1<br>
Espace d’adressage : 10.11.0.0/16, 10.12.0.0/16 (facultatif)<br>
Nom du sous-réseau : par défaut<br>
Plage d’adresses de sous-réseau : 10.11.0.0/24<br>
Groupe de ressources : ClassicRG<br>
Emplacement : USA Est<br>
Sous-réseau de passerelle : 10.11.1.0/27

**Valeurs pour TestVNet4**

Nom : TestVNet4<br>
Espace d’adressage : 10.41.0.0/16, 10.42.0.0/16 (facultatif)<br>
Nom du sous-réseau : par défaut<br>
Plage d’adresses de sous-réseau : 10.41.0.0/24<br>
Groupe de ressources : ClassicRG<br>
Emplacement : USA Ouest<br>
Sous-réseau de passerelle : 10.41.1.0/27

### <a name="connections"></a><a name="plan"></a>Connexions

Le tableau suivant contient un exemple de connexion aux réseaux virtuels. Utilisez les plages comme indication uniquement. Consignez les plages pour vos réseaux virtuels. Vous aurez besoin de ces informations pour les étapes ultérieures.

Dans cet exemple, TestVNet1 se connecte à un site réseau local que vous créez, « VNet4Local ». Les paramètres de VNet4Local contiennent les préfixes d’adresse pour TestVNet4.
Le site local de chaque réseau virtuel correspond à l’autre réseau virtuel. Notre configuration utilise les valeurs d’exemple suivantes :

**Exemple**

| Réseau virtuel | Espace d’adressage | Location | Se connecte au site de réseau local |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |USA Est |SiteVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |USA Ouest |SiteVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

## <a name="create-virtual-networks"></a><a name="vnetvalues"></a>Créer des réseaux virtuels

Au cours de cette étape, vous allez créer deux réseaux virtuels classiques, TestVNet1 et TestVNet4. Si vous utilisez cet article en guise d'exercice, utilisez les [valeurs d'exemple](#vnet).

**Lorsque vous créez vos réseaux virtuels, tenez compte des paramètres suivants :**

* **Espaces d’adresses du réseau virtuel**  : sur la page Espaces d’adresses du réseau virtuel, indiquez la plage d’adresses que vous voulez utiliser pour votre réseau virtuel. Il s’agit des adresses IP dynamiques qui seront affectées aux machines virtuelles et aux autres instances de rôle que vous déployez dans ce réseau virtuel.<br>Les espaces d’adressage que vous sélectionnez ne peuvent pas chevaucher les espaces d’adressage des autres réseaux virtuels ou les emplacements locaux auxquels ce réseau virtuel se connectera.

* **Emplacement**  : lors de la création d'un réseau virtuel, associez-le à une région (emplacement) Azure. Par exemple, si vous souhaitez que vos machines virtuelles déployées sur votre réseau virtuel soient physiquement situées dans la région USA Ouest, sélectionnez cet emplacement. Vous ne pouvez pas modifier l’emplacement associé à votre réseau virtuel après sa création.

**Une fois vos réseaux virtuels créés, vous pouvez ajouter les paramètres suivants :**

* **Espace d’adressage**  : aucun espace d’adressage supplémentaire n’est nécessaire pour cette configuration, mais vous pouvez ajouter un espace d’adressage supplémentaire après avoir créé votre réseau virtuel.

* **Sous-réseaux**  : les sous-réseaux supplémentaires ne sont pas requis pour cette configuration, mais vous souhaiterez peut-être placer vos machines virtuelles dans un sous-réseau distinct de vos autres instances de rôle.

* **Serveurs DNS**  : entrez le nom du serveur DNS et son adresse IP. Ce paramètre n’entraîne pas la création de serveur DNS. Il vous permet de spécifier le serveur DNS que vous souhaitez utiliser pour la résolution de noms pour ce réseau virtuel.

### <a name="to-create-a-classic-virtual-network"></a>Pour créer un réseau virtuel classique

[!INCLUDE [basic classic vnet](../../includes/vpn-gateway-vnet-classic.md)]

[!INCLUDE [basic classic DNS](../../includes/vpn-gateway-dns-classic.md)]

## <a name="configure-sites-and-gateways"></a><a name="localsite"></a>Configurer des sites et des passerelles

Azure utilise les paramètres spécifiés dans chaque site de réseau local pour déterminer comment router le trafic entre les réseaux virtuels. Chaque réseau virtuel doit pointer vers le réseau local vers lequel vous souhaitez acheminer le trafic. Vous déterminez le nom à utiliser pour faire référence à chaque site de réseau local. Il est préférable d’utiliser un nom descriptif.

Par exemple, TestVNet1 se connecte à un site de réseau local que vous créez, nommé « VNet4Local ». Les paramètres de VNet4Local contiennent les préfixes d’adresse pour TestVNet4.

Gardez à l'esprit que le site local de chaque réseau virtuel est l'autre réseau virtuel.

| Réseau virtuel | Espace d’adressage | Location | Se connecte au site de réseau local |
|:--- |:--- |:--- |:--- |
| TestVNet1 |TestVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |USA Est |SiteVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |
| TestVNet4 |TestVNet4<br>(10.41.0.0/16)<br>(10.42.0.0/16) |USA Ouest |SiteVNet1<br>(10.11.0.0/16)<br>(10.12.0.0/16) |

### <a name="to-configure-a-site"></a><a name="site"></a>Pour configurer un site

Le site local fait généralement référence à votre emplacement local. Il contient l’adresse IP du périphérique VPN avec lequel vous allez créer une connexion et les plages d’adresses IP qui seront acheminées via la passerelle VPN vers le périphérique VPN.

1. Sur la page de votre réseau virtuel, sous **Paramètres** , sélectionnez **Connexions de site à site** .
1. Dans la page Connexions de site à site, sélectionnez **+ Ajouter** .
1. Dans la page **Configurer une connexion VPN et une passerelle** , pour **Type de connexion** , gardez l’option **Site à site** sélectionnée.

   * **Adresse IP de la passerelle VPN :** Adresse IP publique du périphérique VPN pour votre réseau local. Dans le cadre de cet exercice, vous pouvez saisir une adresse fictive car vous ne disposez pas encore de l'adresse IP de la passerelle VPN de l'autre site. Par exemple, 5.4.3.2. Plus tard, lorsque vous aurez configuré la passerelle pour l'autre réseau virtuel, vous pourrez ajuster cette valeur.

   * **Espace d’adressage du client :** Répertoriez les plages d'adresses IP que vous souhaitez acheminer vers l'autre réseau virtuel par le biais de cette passerelle. Vous pouvez ajouter plusieurs plages d’espaces d’adressage. Assurez-vous que les plages que vous spécifiez ici ne se chevauchent pas avec des plages d’adresses d’autres réseaux auxquels votre réseau virtuel se connecte, ou avec les propres plages d’adresses du réseau virtuel.
1. En bas de la page, NE sélectionnez PAS passer Vérifier + créer. Au lieu de cela, sélectionnez **Suivant : Passerelle >** .

### <a name="to-configure-a-virtual-network-gateway"></a><a name="sku"></a>Pour configurer une passerelle de réseau virtuel

1. Sur la page **Passerelle** , sélectionnez les valeurs suivantes :

   * **Taille :** Il s’agit de la référence de passerelle que vous utilisez pour créer votre passerelle de réseau virtuel. Les passerelles VPN classiques utilisent les anciennes références SKU de passerelles. Pour en savoir plus sur les anciennes références SKU de passerelle, consultez la section [Utilisation des références SKU de passerelle de réseau virtuel (anciennes références SKU)](vpn-gateway-about-skus-legacy.md). Vous pouvez sélectionner **Standard** pour cet exercice.

   * **Type de routage :** Sélectionnez le type de routage pour votre passerelle. Cela correspond également au type de VPN. Il est important de sélectionner le type approprié, car vous ne pouvez pas convertir la passerelle d’un type à un autre. Votre périphérique VPN doit être compatible avec le type de routage que vous sélectionnez. Pour plus d’informations sur le type de routage, consultez [À propos des paramètres de la passerelle VPN](vpn-gateway-about-vpn-gateway-settings.md#vpntype). Certains articles font référence aux types de VPN « RouteBased » et « PolicyBased ». « RouteBased » correspond à un routage dynamique, et « PolicyBased » à un routage statique. Pour cette configuration, sélectionnez **Dynamique** .

   * **Sous-réseau de passerelle**  : La taille du sous-réseau de passerelle que vous spécifiez dépend de la configuration de la passerelle VPN que vous souhaitez créer. Bien qu’il soit possible de créer un sous-réseau de passerelle aussi petit que /29, nous vous recommandons d’utiliser un sous-réseau /27 ou /28. Cette opération crée un sous-réseau plus grand qui inclut plusieurs adresses. En choisissant un sous-réseau de passerelle plus vaste, vous disposez de suffisamment d’adresses IP pour prendre en charge d’éventuelles configurations futures.

1. Au bas de la page, sélectionnez **Vérifier + créer** pour confirmer vos paramètres. Sélectionnez **Créer** pour effectuer le déploiement. Selon la référence SKU que vous sélectionnez, cela peut prendre jusqu’à 45 minutes pour créer une passerelle de réseau virtuel.
1. Pendant la création de cette passerelle, vous pouvez commencer à passer à l'étape suivante.

### <a name="configure-testvnet4-settings"></a>Configurer les paramètres de TestVNet4

Répétez les étapes de [Création d'un site et d'une passerelle ](#localsite) pour configurer TestVNet4, en remplaçant les valeurs si cela s'avère nécessaire. Si vous effectuez cette opération en guise d'exercice, utilisez les [valeurs d'exemple](#planning).

## <a name="update-local-sites"></a><a name="updatelocal"></a>Mettre à jour les sites locaux

Une fois vos passerelles de réseau virtuel créées pour les deux réseaux virtuels, vous devez ajuster les propriétés d' **adresse IP de passerelle VPN** des sites locaux.

|Nom du réseau virtuel|Site connecté|Adresse IP de la passerelle|
|:--- |:--- |:--- |
|TestVNet1|VNet4Local|Adresse IP de passerelle VPN pour TestVNet4|
|TestVNet4|VNet1Local|Adresse IP de passerelle VPN pour TestVNet1|

### <a name="part-1---get-the-virtual-network-gateway-public-ip-address"></a>Partie 1 : obtenir l’adresse IP publique de passerelle de réseau virtuel

1. Accédez à votre réseau virtuel en accédant au **Groupe de ressources** et en sélectionnant le réseau virtuel.
1. Sur la page de votre réseau virtuel, dans le volet **Essentials** , à droite, recherchez l' **Adresse IP de la passerelle** et copiez-la dans le presse-papiers.

### <a name="part-2---modify-the-local-site-properties"></a>Partie 2 : modifier les propriétés des sites locaux

1. Sous Connexions de site à site, sélectionnez la connexion. Par exemple, SiteVNet4.
1. Sur la page **Propriétés** de la connexion de site à site, sélectionnez **Modifier le site local** .
1. Dans le champ **Adresse IP de la passerelle VPN** , collez l'adresse IP de la passerelle VPN que vous avez copiée à la section précédente.
1. Sélectionnez **OK** .
1. Le champ est mis à jour dans le système. Vous pouvez également utiliser cette méthode pour ajouter une adresse IP supplémentaire à acheminer vers ce site.

### <a name="part-3---repeat-steps-for-the-other-vnet"></a>Partie 3 : répéter les étapes pour l'autre réseau virtuel

Répétez les étapes pour TestVNet4.

## <a name="retrieve-configuration-values"></a><a name="getvalues"></a>Récupérer les valeurs de configuration

[!INCLUDE [retrieve values](../../includes/vpn-gateway-values-classic.md)]

## <a name="create-connections"></a><a name="createconnections"></a>Créer des connexions

Lorsque vous avez terminé les étapes précédentes, vous pouvez définir les clés prépartagées IPsec/IKE et créer la connexion. Cet ensemble d’étapes utilise PowerShell. Les connexions de réseau virtuel à réseau virtuel du modèle de déploiement classique ne peuvent pas être configurées sur le portail Azure car rien ne permet d'y spécifier la clé partagée.

Dans les exemples ci-dessous, notez que la clé partagée est exactement identique. La clé partagée doit toujours correspondre. Veillez à remplacer les valeurs de ces exemples par les noms exacts de vos réseaux virtuels et de vos sites de réseau local.

1. Créez la connexion de TestVNet1 à TestVNet4. Veillez à modifier les valeurs.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet1' `
   -LocalNetworkSiteName 'value for _VNet4Local' -SharedKey A1b2C3D4
   ```
2. Créez la connexion de TestVNet4 à TestVNet1.

   ```powershell
   Set-AzureVNetGatewayKey -VNetName 'Group ClassicRG TestVNet4' `
   -LocalNetworkSiteName 'value for _VNet1Local' -SharedKey A1b2C3D4
   ```
3. Attendez l’initialisation des connexions. Une fois la passerelle initialisée, le système affiche l’état « Réussite ».

   ```
   Error          :
   HttpStatusCode : OK
   Id             :
   Status         : Successful
   RequestId      :
   StatusCode     : OK
   ```

## <a name="faq-and-considerations"></a><a name="faq"></a>FAQ et considérations

Ces considérations s'appliquent aux réseaux virtuels classiques et aux passerelles de réseau virtuel classiques.

* Les réseaux virtuels peuvent être situés dans des abonnements identiques ou différents.
* Les réseaux virtuels peuvent être situés dans des régions (emplacements) identiques ou différentes.
* Un service cloud ou un point de terminaison d’équilibrage de charge ne peut pas s’étendre sur différents réseaux virtuels, même si ces derniers sont interconnectés.
* L’interconnexion de plusieurs réseaux virtuels ne nécessite pas d’appareils VPN.
* La connexion de réseau virtuel à réseau virtuel prend en charge la connexion de réseaux virtuels Azure. Elle ne prend pas en charge la connexion de machines virtuelles ou de services cloud qui ne sont pas déployés dans un réseau virtuel.
* La connectivité de réseau virtuel à réseau virtuel nécessite des passerelles de routage dynamiques. Les passerelles de routage statique Azure ne sont pas prises en charge.
* Une connectivité réseau virtuelle peut être utilisée en même temps que des VPN multisites. Il peut y avoir au maximum 10 tunnels VPN par passerelle VPN de réseau virtuel se connectant à d’autres réseaux virtuels ou à des sites locaux.
* Les espaces d’adressage des réseaux virtuels et sur les sites de réseau locaux ne doivent pas se chevaucher. Le chevauchement des espaces d’adressage entraîne l’échec de la création de réseaux virtuels ou du téléchargement de fichiers de configuration netcfg.
* Les tunnels redondants entre deux réseaux virtuels ne sont pas pris en charge.
* Tous les tunnels VPN du réseau virtuel, y compris les VPN de point à site, partagent la bande passante disponible pour la passerelle VPN, ainsi que le même contrat SLA concernant le temps d’activité des passerelles VPN dans Azure.
* Le trafic de réseau virtuel à réseau virtuel circule sur la dorsale principale d’Azure.

## <a name="next-steps"></a>Étapes suivantes

Vérifiez vos connexions. Voir [Vérifier une connexion de passerelle VPN](vpn-gateway-verify-connection-resource-manager.md).
