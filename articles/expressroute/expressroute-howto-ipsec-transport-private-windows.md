---
title: 'Peering privé Azure ExpressRoute : Configurer le mode de transport IPsec – Hôtes Windows'
description: Découvrez comment activer le mode de transport IPsec entre des machines virtuelles Azure exécutant Windows et des hôtes Windows locaux via le peering privé ExpressRoute, à l’aide d’objets de stratégie de groupe ou d’unités d’organisation.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 10/17/2018
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 2dcb8489d94b9afc3ae4df829b37dd9785383d85
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92208241"
---
# <a name="configure-ipsec-transport-mode-for-expressroute-private-peering"></a>Configurer le mode de transport IPsec pour le peering privé ExpressRoute

Cet article explique comment créer des tunnels IPsec en mode Transport via un peering privé ExpressRoute entre des machines virtuelles Azure exécutant Windows et des hôtes Windows locaux. Les étapes décrites dans cet article utilisent des objets de stratégie de groupe pour créer cette configuration. Même s’il est possible de créer une telle configuration sans utiliser d’unités d’organisation ou d’objets de stratégie de groupe, le fait d’associer ces deux types d’objets permet de simplifier le contrôle de vos stratégies de sécurité et d’effectuer rapidement un scale-up. Les étapes de cet article supposent que vous disposez déjà d’une configuration Active Directory et que vous êtes familiarisé avec l’utilisation des unités d’organisation et des objets de stratégie de groupe.

## <a name="about-this-configuration"></a>À propos de cette configuration

Dans les étapes suivantes, la configuration utilise un réseau virtuel Azure avec un peering privé ExpressRoute. Toutefois, cette configuration peut s’étendre sur plusieurs réseaux virtuels Azure et réseaux locaux. Cet article vous montre comment définir une stratégie de chiffrement IPsec et comment l’appliquer à un groupe de machines virtuelles Azure et d’hôtes locaux faisant partie d’une même unité d’organisation. Vous ne devez configurer le chiffrement entre les machines virtuelles Azure (vm1 et vm2) et l’hôte local host1 que pour le trafic HTTP avec le port de destination 8080. Différents types de stratégies IPsec peuvent être créés selon vos besoins.

### <a name="working-with-ous"></a>Utilisation des unités d’organisation 

La stratégie de sécurité qui est associée à une unité d’organisation est envoyée (push) aux ordinateurs via un objet de stratégie de groupe. Voici certains des avantages que présente l’utilisation d’unités d’organisation, par rapport à l’application de stratégies à un seul hôte :

* Lorsque vous associez une stratégie à une unité d’organisation, vous avez la garantie que les ordinateurs qui appartiennent à une même unité d’organisation obtiendront les mêmes stratégies.
* Lorsque vous modifiez la stratégie de sécurité qui est associée à l’unité d’organisation, les modifications sont appliquées à tous les hôtes de l’unité d’organisation.

### <a name="diagrams"></a>Diagrammes

Le diagramme suivant montre l’interconnexion et l’espace d’adressage IP attribué. Les machines virtuelles Azure et l’hôte local exécutent Windows 2016. Les machines virtuelles Azure et l’hôte local host1 appartiennent au même domaine. Les machines virtuelles Azure et les hôtes locaux peuvent résoudre des noms correctement à l’aide du DNS.

[![1]][1]

Ce diagramme montre les tunnels IPsec en transit dans le peering privé ExpressRoute.

[![4]][4]

### <a name="working-with-ipsec-policy"></a>Utilisation de la stratégie IPsec

Dans Windows, le chiffrement est associé à la stratégie IPsec. Une stratégie IPsec détermine quel trafic IP est sécurisé, et quel mécanisme de sécurité est appliqué aux paquets IP.
**Les stratégies IPSec** sont composées des éléments suivants : **Listes de filtres**, **Actions de filtrage** et **Règles de sécurité**.

Lorsque vous configurez une stratégie IPsec, il est important de comprendre sa terminologie :

* **Stratégie IPsec :** collection de règles. Vous ne pouvez activer (« attribuer ») qu’une seule stratégie à la fois. Cependant, chaque stratégie peut avoir une ou plusieurs règles, qui peuvent être actives simultanément. Vous ne pouvez attribuer qu’une seule stratégie IPsec à la fois à un ordinateur. Toutefois, dans cette stratégie IPsec, vous pouvez définir plusieurs actions à exécuter selon la situation. Chaque ensemble de règles IPsec est associé à une liste de filtres qui affecte le type de trafic réseau auquel la règle s’applique.

* **Listes de filtres :** les listes de filtres comprennent un ou plusieurs filtres. Une même liste peut contenir plusieurs filtres. Un filtre définit si la communication est autorisée, sécurisée ou bloquée, selon les plages d’adresses IP, les protocoles ou même certains ports de protocole. Chaque filtre correspond à un ensemble de conditions, par exemple, des paquets envoyés à partir d’un sous-réseau donné vers un ordinateur donné via un port de destination donné. Lorsque les conditions réseau correspondent à un ou plusieurs de ces filtres, la liste de filtres est activée. Chaque filtre est défini au sein d’une liste de filtres. Les filtres ne peuvent pas être partagés par plusieurs listes. Cependant, une liste de filtres donnée peut être intégrée à plusieurs stratégies IPsec. 

* **Actions de filtrage :** une méthode de sécurité définit un ensemble regroupant des algorithmes de sécurité, des protocoles et une clé que l’ordinateur propose pendant les négociations IKE. Les actions de filtrage sont des listes de méthodes de sécurité, classées par ordre de préférence.  Lorsqu’un ordinateur négocie une session IPsec, il accepte ou envoie des propositions en fonction du paramètre de sécurité qui est stocké dans la liste des actions de filtrage.

* **Règles de sécurité :** les règles déterminent quand et comment une stratégie IPsec doit protéger les communications. Les **listes de filtres** et les **actions de filtrage** sont utilisées pour créer une règle IPsec dans le but d’établir la connexion IPsec. Cependant, chaque stratégie peut avoir une ou plusieurs règles, qui peuvent être actives simultanément. Chaque règle contient une liste de filtres IP et un groupe d’actions de sécurité qui sont exécutées lorsqu’une correspondance est établie avec la liste de filtres :
  * Actions de filtrage IP
  * Méthodes d’authentification
  * Paramètres de tunnel IP
  * Types de connexion

[![5]][5]

## <a name="before-you-begin"></a>Avant de commencer

Assurez-vous de satisfaire les prérequis suivants :

* Vous devez disposer d’une configuration Active Directory fonctionnelle que vous pouvez utiliser pour implémenter les paramètres de stratégie de groupe. Pour plus d’informations sur les objets de stratégie de groupe, consultez [Objets de stratégie de groupe](/previous-versions/windows/desktop/Policy/group-policy-objects).

* Vous devez disposer d’un circuit ExpressRoute actif.
  * Pour plus d’informations sur la création d’un circuit ExpressRoute, consultez [Créer un circuit ExpressRoute](expressroute-howto-circuit-arm.md). 
  * Vérifiez que le circuit a été activé par votre fournisseur de connectivité. 
  * Vérifiez que le peering privé Azure a été configuré pour votre circuit. Pour obtenir des instructions de routage, consultez l'article sur la [configuration du routage](expressroute-howto-routing-arm.md) . 
  * Vérifiez qu’un réseau virtuel et une passerelle de réseau virtuel ont été créés et entièrement provisionnés. Suivez les instructions pour [créer une passerelle de réseau virtuel pour ExpressRoute](expressroute-howto-add-gateway-resource-manager.md). Une passerelle de réseau virtuel pour ExpressRoute utilise le type de passerelle « ExpressRoute », pas de VPN.

* La passerelle de réseau virtuel ExpressRoute doit être connectée au circuit ExpressRoute. Pour plus d’informations, consultez [Connecter un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md).

* Vérifiez que les machines virtuelles Azure exécutant Windows ont été déployées sur le réseau virtuel.

* Vérifiez qu’une connexion a été établie entre les hôtes locaux et les machines virtuelles Azure.

* Vérifiez que les machines virtuelles Azure exécutant Windows et les hôtes locaux sont en mesure d’utiliser le DNS pour résoudre les noms correctement.

### <a name="workflow"></a>Workflow

1. Créez un objet de stratégie de groupe et associez-le à l’unité d’organisation.
2. Définissez une **action de filtrage** IPsec.
3. Définissez une **liste de filtres** IPsec.
4. Créez une stratégie IPsec avec des **règles de sécurité**.
5. Attribuez l’objet de stratégie de groupe IPsec à l’unité d’organisation.

### <a name="example-values"></a>Exemples de valeurs

* **Nom de domaine** : ipsectest.com

* **Unité d’organisation :** IPSecOU

* **Ordinateur Windows local** : host1

* **Machines virtuelles Azure exécutant Windows** : vm1, vm2

## <a name="1-create-a-gpo"></a><a name="creategpo"></a>1. Créer un objet de stratégie de groupe

1. Pour créer un objet de stratégie de groupe lié à une unité d’organisation, ouvrez le composant logiciel enfichable Gestion des stratégies de groupe, puis recherchez l’unité d’organisation à laquelle l’objet de stratégie de groupe est lié. Dans l’exemple, l’unité d’organisation se nomme **IPSecOU**. 

   [![9]][9]
2. Dans le composant logiciel enfichable Gestion des stratégies de groupe, sélectionnez l’unité d’organisation, puis cliquez dessus avec le bouton droit. Dans la liste déroulante, cliquez sur **Créer un objet GPO dans ce domaine, et le lier ici...** .

   [![10]][10]
3. Attribuez un nom explicite à l’objet de stratégie de groupe de manière à pouvoir le localiser facilement quand vous en aurez besoin. Cliquez sur **OK** pour créer et lier l’objet de stratégie de groupe.

   [![11]][11]

## <a name="2-enable-the-gpo-link"></a><a name="enablelink"></a>2. Activer la liaison avec l’objet de stratégie de groupe

Pour appliquer l’objet de stratégie de groupe à l’unité d’organisation, vous devez non seulement le lier à l’unité d’organisation, mais également activer cette liaison.

1. Recherchez l’objet de stratégie de groupe que vous venez de créer, cliquez dessus avec le bouton droit, puis sélectionnez **Modifier** dans la liste déroulante.
2. Pour appliquer l’objet de stratégie de groupe à l’unité d’organisation, sélectionnez **Lien activé**.

   [![12]][12]

## <a name="3-define-the-ip-filter-action"></a><a name="filteraction"></a>3. Définir l’action de filtrage IP

1. Dans la liste déroulante, cliquez avec le bouton droit sur **Stratégies de sécurité IP sur Active Directory**, puis cliquez sur **Gérer les listes de filtres IP et les actions de filtrage**.

   [![15]][15]
2. Sous l’onglet **Gérer les actions de filtrage**, cliquez sur **Ajouter**.

   [![16]][16]

3. Dans **l’Assistant Action de filtrage de sécurité IP**, cliquez sur **Suivant**.

   [![17]][17]
4. Attribuez un nom explicite à l’action de filtrage de manière à pouvoir la localiser facilement quand vous en aurez besoin. Dans cet exemple, l’action de filtrage se nomme **myEncryption**. Vous pouvez également ajouter une description. Cliquez ensuite sur **Suivant**.

   [![18]][18]
5. L’option **Négocier la sécurité** vous permet de définir le comportement si la connexion IPsec ne peut pas être établie avec un autre ordinateur. Sélectionnez **Négocier la sécurité**, puis cliquez sur **Suivant**.

   [![19]][19]
6. Dans la page **Communiquer avec des ordinateurs qui ne prennent pas en charge IPSec**, sélectionnez **Ne pas autoriser les communications non sécurisées**, puis cliquez sur **Suivant**.

   [![20]][20]
7. Dans la page **Trafic IP et sécurité**, sélectionnez **Personnalisé**, puis cliquez sur **Paramètres...** .

   [![21]][21]
8. Dans la page **Paramètres personnalisés de la méthode de sécurité**, sélectionnez **Chiffrement et intégrité des données (ESP) : SHA1, 3DES**. Cliquez ensuite sur **OK**.

   [![22]][22]
9. Dans la page **Gérer les actions de filtrage**, vous pouvez voir que le filtre **myEncryption** a bien été ajouté. Cliquez sur **Fermer**.

   [![23]][23]

## <a name="4-define-an-ip-filter-list"></a><a name="filterlist1"></a>4. Définir une liste de filtres IP

Créez une liste de filtres qui spécifie le trafic HTTP chiffré avec le port de destination 8080.

1. Pour qualifier les types de trafic qui doivent être chiffrés, utilisez une **liste de filtres IP**. Sous l’onglet **Gestion de listes de filtres IP**, cliquez sur **Ajouter** pour ajouter une liste de filtres IP.

   [![24]][24]
2. Dans le champ **Nom**, tapez un nom pour votre liste de filtres IP. Par exemple, **azure-onpremises-HTTP8080**. Cliquez ensuite sur **Ajouter**.

   [![25]][25]
3. Dans la page **Description du filtre IP et propriété mise en miroir**, sélectionnez **En miroir**. Le paramètre En miroir recherche les paquets entrants et sortants, ce qui permet une communication bidirectionnelle. Cliquez ensuite sur **Suivant**.

   [![26]][26]
4. Dans la page **Source du trafic IP**, dans la liste déroulante **Adresse source**, choisissez **Une adresse IP ou un sous-réseau spécifique**. 

   [![27]][27]
5. Spécifiez l’adresse source du trafic IP dans **Adresse IP ou sous-réseau**, puis cliquez sur **Suivant**.

   [![28]][28]
6. Spécifiez l’**adresse de destination :** adresse IP ou sous-réseau. Cliquez ensuite sur **Suivant**.

   [![29]][29]
7. Dans la page **Type de protocole IP**, sélectionnez **TCP**. Cliquez ensuite sur **Suivant**.

   [![30]][30]
8. Dans la page **Type de protocole IP**, sélectionnez **Depuis n’importe quel port** et **Vers ce port**. Tapez **8080** dans la zone de texte. Ces paramètres spécifient que seul le trafic HTTP passant par le port de destination 8080 est chiffré. Cliquez ensuite sur **Suivant**.

   [![31]][31]
9. Affichez la liste de filtres IP.  La configuration de la liste de filtres IP **azure-onpremises-HTTP8080** déclenche le chiffrement pour tout le trafic qui répond aux critères suivants :

   * N’importe quelle adresse source appartenant à la plage 10.0.1.0/24 (sous-réseau2 Azure)
   * N’importe quelle adresse de destination appartenant à la plage 10.2.27.0/25 (sous-réseau local)
   * Protocole TCP
   * Port de destination 8080

   [![32]][32]

## <a name="5-edit-the-ip-filter-list"></a><a name="filterlist2"></a>5. Modifier la liste de filtres IP

Pour chiffrer le même type de trafic dans la direction opposée (de l’hôte local vers la machine virtuelle Azure), vous avez besoin d’un deuxième filtre IP. Le processus de la configuration du nouveau filtre est le même que celui que vous avez utilisé pour définir le premier filtre IP. Les seules différences se situent au niveau des sous-réseaux source et de destination.

1. Pour ajouter un nouveau filtre IP à la liste de filtres IP, sélectionnez **Modifier**.

   [![33]][33]
2. Dans la page **Liste de filtres IP**, cliquez sur **Ajouter**.

   [![34]][34]
3. Créez un deuxième filtre IP en utilisant les paramètres de l’exemple suivant :

   [![35]][35]
4. Une fois le deuxième filtre IP créé, la liste de filtres IP ressemble à ceci :

   [![36]][36]

Si le chiffrement est exigé entre un emplacement local et un sous-réseau Azure pour protéger une application, au lieu de modifier la liste de filtres IP existante, vous pouvez ajouter une nouvelle liste de filtres IP. Le fait d’associer deux listes de filtres IP à la même stratégie IPsec fournit une plus grande flexibilité, car cela permet de modifier ou de supprimer une liste de filtres IP à tout moment sans impacter les autres listes de filtres IP.

## <a name="6-create-an-ipsec-security-policy"></a><a name="ipsecpolicy"></a>6. Créer une stratégie de sécurité IPsec 

Créez une stratégie IPsec avec des règles de sécurité.

1. Sélectionnez les **Stratégies de sécurité IP sur Active Directory** qui sont associées à l’unité d’organisation. Cliquez dessus avec le bouton droit, puis sélectionnez **Créer une stratégie de sécurité IP**.

   [![37]][37]
2. Nommez la stratégie de sécurité. Par exemple, **policy-azure-onpremises**. Cliquez ensuite sur **Suivant**.

   [![38]][38]
3. Cliquez sur **Suivant** sans cocher la case.

   [![39]][39]
4. Vérifiez que la case **Modifier les propriétés** est sélectionnée, puis cliquez sur **Terminer**.

   [![40]][40]

## <a name="7-edit-the-ipsec-security-policy"></a><a name="editipsec"></a>7. Modifier la stratégie de sécurité IPsec

Ajoutez à la stratégie IPsec la **liste de filtres IP** et **l’action de filtrage** que vous avez configurées précédemment.

1. Sous l’onglet **Règles** des propriétés de la stratégie HTTP, cliquez sur **Ajouter**.

   [![41]][41]
2. Sur la page d'accueil, cliquez sur **Suivant**.

   [![42]][42]
3. Une règle offre la possibilité de définir le mode IPsec : le mode Tunnel ou le mode Transport.

   * En mode Tunnel, le paquet d’origine est encapsulé par un ensemble d’en-têtes d’adresses IP. Le mode Tunnel protège les informations de routage internes en chiffrant l’en-tête IP du paquet d’origine. Le mode Tunnel est largement implémenté sur les passerelles, dans les scénarios VPN de site à site. Dans la plupart des cas, le mode Tunnel est utilisé pour le chiffrement des données de bout en bout entre les hôtes.

   * Le mode Transport chiffre uniquement la charge utile et le code de fin ESP. L’en-tête IP du paquet d’origine n’est pas chiffré. En mode Transport, la source et la destination des adresses IP des paquets restent inchangées.

   Sélectionnez **Cette règle ne spécifie aucun tunnel**, puis cliquez sur **Suivant**.

   [![43]][43]
4. L’option **Type de réseau** permet de choisir la connexion réseau à associer à la stratégie de sécurité. Sélectionnez **Toutes les connexions réseau**, puis cliquez sur **Suivant**.

   [![44]][44]
5. Sélectionnez la liste de filtres IP que vous avez créée précédemment (**azure-onpremises-HTTP8080**), puis cliquez sur **Suivant**.

   [![45]][45]
6. Sélectionnez l’action de filtrage que vous avez créée précédemment (**myEncryption**).

   [![46]][46]
7. Windows prend en charge quatre types d’authentification distincts : Kerberos, certificats, NTLMv2 et clé prépartagée. Étant donné que nous utilisons des ordinateurs hôtes joints à un domaine, sélectionnez **Authentification Active Directory par défaut (protocole Kerberos v.5)** , puis cliquez sur **Suivant**.

   [![47]][47]
8. La nouvelle stratégie crée la règle de sécurité **azure-onpremises-HTTP8080**. Cliquez sur **OK**.

   [![48]][48]

La stratégie IPsec nécessite que toutes les connexions HTTP passant par le port de destination 8080 utilisent le mode de transport IPsec. Étant donné que le protocole HTTP est non chiffré, le fait d’activer la stratégie de sécurité garantit le chiffrement des données lors de leur transfert via le peering privé ExpressRoute. La stratégie de sécurité IP pour Active Directory est plus complexe à configurer que le Pare-feu Windows avec fonctions avancées de sécurité. Toutefois, elle permet de personnaliser davantage la connexion IPsec.

## <a name="8-assign-the-ipsec-gpo-to-the-ou"></a><a name="assigngpo"></a>8. Attribuer l’objet de stratégie de groupe IPsec à l’unité d’organisation

1. Affichez la stratégie. La stratégie de groupe de sécurité est définie, mais pas encore attribuée.

   [![49]][49]
2. Pour attribuer la stratégie de groupe de sécurité à l’unité d’organisation **IPSecOU**, cliquez avec le bouton droit sur la stratégie de sécurité, puis choisissez **Attribuer**.
   Chaque ordinateur qui appartient à l’unité d’organisation se verra attribuer la stratégie de groupe de sécurité.

   [![50]][50]

## <a name="check-traffic-encryption"></a><a name="checktraffic"></a>Vérifier le chiffrement du trafic

Pour tester l’objet de stratégie de groupe qui est appliqué à l’unité d’organisation de chiffrement, installez IIS sur toutes les machines virtuelles Azure et sur l’hôte host1. Chaque IIS est personnalisé pour répondre aux requêtes HTTP sur le port 8080.
Pour vérifier le chiffrement, vous pouvez installer un renifleur de réseau (comme Wireshark) sur tous les ordinateurs de l’unité d’organisation.
Un script PowerShell agit comme un client HTTP afin de générer des requêtes HTTP sur le port 8080 :

```powershell
$url = "http://10.0.1.20:8080"
while ($true) {
try {
[net.httpWebRequest]
$req = [net.webRequest]::create($url)
$req.method = "GET"
$req.ContentType = "application/x-www-form-urlencoded"
$req.TimeOut = 60000

$start = get-date
[net.httpWebResponse] $res = $req.getResponse()
$timetaken = ((get-date) - $start).TotalMilliseconds

Write-Output $res.Content
Write-Output ("{0} {1} {2}" -f (get-date), $res.StatusCode.value__, $timetaken)
$req = $null
$res.Close()
$res = $null
} catch [Exception] {
Write-Output ("{0} {1}" -f (get-date), $_.ToString())
}
$req = $null

# uncomment the line below and change the wait time to add a pause between requests
#Start-Sleep -Seconds 1
}

```
La capture réseau suivante montre les résultats de l’hôte local host1, avec du code ESP de filtrage qui affiche uniquement le trafic chiffré :

[![51]][51]

Si vous exécutez le script PowerShell localement (sur un client HTTP), la capture réseau de la machine virtuelle Azure montrera une trace similaire.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur ExpressRoute, consultez la [FAQ sur ExpressRoute](expressroute-faqs.md).

<!--Image References-->

[1]: ./media/expressroute-howto-ipsec-transport-private-windows/network-diagram.png "Diagramme du réseau- Mode de transport IPsec via ExpressRoute"
[4]: ./media/expressroute-howto-ipsec-transport-private-windows/ipsec-interesting-traffic.png "IPsec intéressant du trafic"
[5]: ./media/expressroute-howto-ipsec-transport-private-windows/windows-ipsec.png "Stratégie IPsec Windows"
[9]: ./media/expressroute-howto-ipsec-transport-private-windows/ou.png "Unité d’organisation dans la stratégie de groupe"
[10]: ./media/expressroute-howto-ipsec-transport-private-windows/create-gpo-ou.png "Création d’un objet de stratégie de groupe associé à l’unité d’organisation"
[11]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-name.png "Attribution d’un nom à l’objet de stratégie de groupe associé à l’unité d’organisation"
[12]: ./media/expressroute-howto-ipsec-transport-private-windows/edit-gpo.png "Modification de l’objet de stratégie de groupe"
[15]: ./media/expressroute-howto-ipsec-transport-private-windows/manage-ip-filter-list-filter-actions.png "Gérer les listes de filtres IP et les actions de filtrage"
[16]: ./media/expressroute-howto-ipsec-transport-private-windows/add-filter-action.png "Ajout d’une action de filtrage"
[17]: ./media/expressroute-howto-ipsec-transport-private-windows/action-wizard.png "Assistant Action"
[18]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action-name.png "Nom de l’action de filtrage"
[19]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action.png "Action de filtrage"
[20]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-action-no-ipsec.png "Spécifier le comportement si une connexion non sécurisée est établie"
[21]: ./media/expressroute-howto-ipsec-transport-private-windows/security-method.png "Mécanisme de sécurité"
[22]: ./media/expressroute-howto-ipsec-transport-private-windows/custom-security-method.png "Méthode de sécurité personnalisée"
[23]: ./media/expressroute-howto-ipsec-transport-private-windows/filter-actions-list.png "Liste d’actions de filtrage"
[24]: ./media/expressroute-howto-ipsec-transport-private-windows/add-new-ip-filter.png "Ajout d’une nouvelle liste de filtres IP"
[25]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-http-traffic.png "Ajout du trafic HTTP au filtre IP"
[26]: ./media/expressroute-howto-ipsec-transport-private-windows/match-both-direction.png "Détecter des paquets dans les deux directions"
[27]: ./media/expressroute-howto-ipsec-transport-private-windows/source-address.png "Sélection du sous-réseau source"
[28]: ./media/expressroute-howto-ipsec-transport-private-windows/source-network.png "Réseau source"
[29]: ./media/expressroute-howto-ipsec-transport-private-windows/destination-network.png "Réseau de destination"
[30]: ./media/expressroute-howto-ipsec-transport-private-windows/protocol.png "Protocole"
[31]: ./media/expressroute-howto-ipsec-transport-private-windows/source-port-and-destination-port.png "Port source et port de destination"
[32]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-list.png "Liste de filtres"
[33]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-for-http.png "Liste de filtres IP avec trafic HTTP"
[34]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-add-second-entry.png "Ajout d’un deuxième filtre IP"
[35]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-second-entry.png "Entrée de la deuxième liste de filtres IP"
[36]: ./media/expressroute-howto-ipsec-transport-private-windows/ip-filter-list-2entries.png "Entrée de la deuxième liste de filtres IP"
[37]: ./media/expressroute-howto-ipsec-transport-private-windows/create-ip-security-policy.png "Création de la stratégie de sécurité IP"
[38]: ./media/expressroute-howto-ipsec-transport-private-windows/ipsec-policy-name.png "Nom de la stratégie IPsec"
[39]: ./media/expressroute-howto-ipsec-transport-private-windows/security-policy-wizard.png "Assistant Stratégie IPsec"
[40]: ./media/expressroute-howto-ipsec-transport-private-windows/edit-security-policy.png "Modification de la stratégie IPsec"
[41]: ./media/expressroute-howto-ipsec-transport-private-windows/add-new-rule.png "Ajout d’une nouvelle règle de sécurité à la stratégie IPsec"
[42]: ./media/expressroute-howto-ipsec-transport-private-windows/create-security-rule.png "Création d’une règle de sécurité"
[43]: ./media/expressroute-howto-ipsec-transport-private-windows/transport-mode.png "Mode de transport"
[44]: ./media/expressroute-howto-ipsec-transport-private-windows/network-type.png "Type de réseau"
[45]: ./media/expressroute-howto-ipsec-transport-private-windows/selection-filter-list.png "Sélection de la liste de filtres IP existante"
[46]: ./media/expressroute-howto-ipsec-transport-private-windows/selection-filter-action.png "Sélection de l’action de filtrage existante"
[47]: ./media/expressroute-howto-ipsec-transport-private-windows/authentication-method.png "Sélection de la méthode d’authentification"
[48]: ./media/expressroute-howto-ipsec-transport-private-windows/security-policy-completed.png "Fin du processus de création de la stratégie de sécurité"
[49]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-not-assigned.png "Stratégie IPsec liée à l’objet de stratégie de groupe, mais pas encore attribuée"
[50]: ./media/expressroute-howto-ipsec-transport-private-windows/gpo-assigned.png "Stratégie IPsec attribuée à l’objet de stratégie de groupe"
[51]: ./media/expressroute-howto-ipsec-transport-private-windows/encrypted-traffic.png "Capturer du trafic chiffré IPsec"