---
title: RDP Shortpath de Windows Virtual Desktop (préversion)
titleSuffix: Azure
description: Comment configurer RDP Shortpath (préversion) pour Windows Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: 6ffe631dc237e7efaf1d6bfd9ac79ab7431c7371
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023137"
---
# <a name="windows-virtual-desktop-rdp-shortpath-preview"></a>RDP Shortpath de Windows Virtual Desktop (préversion)

> [!IMPORTANT]
> RDP Shortpath est actuellement disponible en préversion publique.
> Cette préversion étant fournie sans contrat de niveau de service, nous déconseillons son utilisation pour des charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

RDP Shortpath est une fonctionnalité de Windows Virtual Desktop qui établit un transport UDP direct entre un client et un hôte de session Bureau à distance. RDP utilise ce transport pour fournir Bureau à distance et RemoteApp tout en offrant une meilleure fiabilité et une latence cohérente.

## <a name="key-benefits"></a>Principaux avantages

* Le transport RDP Shortpath est basé sur un [protocole de contrôle de débit universel (Universal Rate Control Protocol, URCP)](https://www.microsoft.com/en-us/research/publication/urcp-universal-rate-control-protocol-for-real-time-communication-applications/) très efficace. Le protocole URCP améliore le protocole UDP avec la surveillance active des conditions du réseau, et assure une utilisation de liens équitable et complète. Le protocole URCP opère à des niveaux faibles de retard et de perte en fonction des besoins du Bureau à distance. Le protocole URCP permet d’obtenir des performances optimales en apprenant de manière dynamique des paramètres réseau et en fournissant un mécanisme de contrôle de débit.
* RDP Shortpath établit la connectivité directe entre le client et l’hôte de session Bureau à distance. La connectivité directe réduit la dépendance des passerelles de Windows Virtual Desktop, améliore la fiabilité de la connexion et augmente la bande passante disponible pour chaque session utilisateur.
* La suppression d’un relais supplémentaire réduit le temps d’aller-retour, ce qui améliore l’expérience utilisateur avec des applications et des méthodes d’entrée sensibles à la latence.
* RDP Shortpath apporte une prise en charge d’une priorité en matière de [configuration de la qualité de service (QoS)](./rdp-quality-of-service-qos.md) priorité pour les connexions RDP via des marques DSCP (Differentiated Services Code point)
* Le transport RDP Shortpath permet de [limiter le trafic réseau sortant](./rdp-bandwidth.md#limit-network-bandwidth-use-with-throttle-rate) en spécifiant un taux de limitation pour chaque session.

## <a name="connection-security"></a>Sécurité de la connexion

RDP Shortpath étend les fonctionnalités multitransport du protocole RDP. Il ne remplace pas le transport de connexion inverse mais le complète. L’ensemble de la répartition de session initiale est géré via l’infrastructure de Windows Virtual Desktop.

Le port UDP 3390 est utilisé uniquement pour le trafic Shortpath entrant qui est authentifié via un transport de connexion inverse. L’écouteur RDP Shortpath ignore toutes les tentatives de connexion à l’écouteur, sauf si elles correspondent à la session de connexion inverse.

RDP Shortpath utilise une connexion TLS entre le client et l’hôte de session à l’aide des certificats de l’hôte de la session. Par défaut, le certificat utilisé pour le chiffrement RDP est généré automatiquement par le système d’exploitation pendant le déploiement. Si vous le souhaitez, les clients peuvent déployer des certificats gérés de manière centralisée émis par l’autorité de certification d’entreprise. Pour plus d’informations sur les configurations de certificats, consultez la [documentation de Windows Server](/troubleshoot/windows-server/remote/remote-desktop-listener-certificate-configurations).

## <a name="rdp-shortpath-connection-sequence"></a>Séquence de connexion de RDP Shortpath

Après l’installation du [transport de connexion inverse](./network-connectivity.md), le client et l’hôte de session établissent la connexion RDP et négocient les fonctionnalités multitransport. Étapes supplémentaires décrites ci-dessous :

1. L’hôte de la session envoie la liste de ses adresses IPv4 et IPv6 privées et publiques au client.
2. Le client démarre le thread d’arrière-plan pour établir un transport UDP parallèle directement vers l’une des adresses IP de l’hôte.
3. Pendant que le client sonde les adresses IP fournies, il continue l’établissement de la connexion initiale sur le transport de connexion inverse pour éviter tout retard dans la connexion utilisateur.
4. Si le client dispose d’une ligne de vue directe et si la configuration du pare-feu est correcte, le client établit une connexion TLS sécurisée avec l’hôte de la session.
5. Après avoir établi le transport Shortpath, le protocole RDP déplace tous les canaux virtuels dynamiques (DVC), à savoir les graphiques distants, les entrées et la redirection des appareils vers le nouveau transport.
6. Si un pare-feu ou une topologie de réseau empêche le client d’établir une connexion UDP directe, le protocole RDP continue avec un transport de connexion inverse.

Le diagramme ci-dessous donne une vue d’ensemble des connexions réseau de RDP Shortpath.

:::image type="content" source="media/rdp-shortpath-connections.svg" alt-text="Diagramme des connexions réseau de RDP Shortpath" lightbox="media/rdp-shortpath-connections.svg":::

## <a name="requirements"></a>Spécifications

Pour prendre en charge RDP Shortpath, le client Windows Virtual Desktop a besoin d’une ligne de vue directe sur l’hôte de la session. Vous pouvez obtenir une ligne de vue directe à l’aide de l’une des technologies suivantes :

* [Peering privé ExpressRoute](../expressroute/expressroute-circuit-peerings.md)
* [VPN site à site (basé sur IPsec)](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [VPN point à site (basé sur IPsec)](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Affectation d’adresses IP publiques](../virtual-network/virtual-network-public-ip-address.md)

Si vous utilisez d’autres types de VPN pour vous connecter au réseau virtuel Azure, nous vous recommandons d’utiliser un VPN basé sur UDP pour obtenir des résultats optimaux. Bien que la majorité des solutions de VPN basées sur le protocole TCP, y compris le protocole UDP, encapsulent tous les paquets IP, elles ajoutent une surcharge héritée du contrôle de congestion TCP qui ralentit les performances du protocole RDP.

La ligne de vue directe signifie que les pare-feu ne bloquent pas le port UDP 3390 et que le client peut se connecter directement à l’hôte de la session.

## <a name="enabling-rdp-shortpath-preview"></a>Activation de la préversion de RDP Shortpath

Pour bénéficier de la préversion de RDP Shortpath, vous devez activer l’écouteur RDP Shortpath sur l’hôte de la session. Vous pouvez activer RDP Shortpath sur un nombre quelconque d’hôtes de session utilisés dans votre environnement. Il n’est pas nécessaire d’activer RDP Shortpath sur tous les hôtes au sein du pool.
Pour activer l’écouteur Shortpath, vous devez configurer les valeurs de registre suivantes :

> [!WARNING]
> De sérieux problèmes peuvent survenir si vous modifiez le registre de manière incorrecte à l’aide de l’Éditeur du Registre ou d’une autre méthode. Ces problèmes peuvent nécessiter la réinstallation du système d’exploitation. Microsoft ne peut pas garantir que ces problèmes puissent être résolus. Toute modification du registre relève de votre propre responsabilité.

1. Sur l’hôte de la session, démarrez regedit.exe, puis accédez à l’emplacement suivant :

    ```cmd
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations
    ```

2. Créez une valeur **DWORD** nommée **fUseUdpPortRedirector** et affectez-lui la valeur **1** (décimal).
3. Créez une valeur **DWORD** nommée **UdpPortNumber** et affectez-lui la valeur **3390** (décimal).
4. Quittez l'Éditeur du Registre.
5. Redémarrez l’hôte de la session.

Vous pouvez également exécuter les cmdlets ci-après dans une fenêtre PowerShell avec élévation de privilèges pour définir les valeurs de registre suivantes :

```powershell
$WinstationsKey = 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations'
New-ItemProperty -Path $WinstationsKey -Name 'fUseUdpPortRedirector' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 1 -Force
New-ItemProperty -Path $WinstationsKey -Name 'UdpPortNumber' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 3390 -Force
```

Vous pouvez également utiliser PowerShell pour configurer une stratégie de groupe.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'fUseUdpPortRedirector' -Value 1 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'UdpPortNumber' -Value 3390 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
```

## <a name="configure-windows-defender-firewall-with-advanced-security"></a>Configurez le Pare-feu Windows Defender avec une sécurité avancée.

Afin d’autoriser le trafic réseau entrant pour RDP Shortpath, utilisez le Pare-feu Windows Defender avec un nœud de sécurité avancée dans le composant logiciel enfichable MMC de Gestion des stratégies de groupe afin de créer des règles de pare-feu.

1. Ouvrez la console Gestion des stratégies de groupe pour le [Pare-feu Windows Defender avec Sécurité avancée](/windows/security/threat-protection/windows-firewall/open-the-group-policy-management-console-to-windows-firewall-with-advanced-security).
2. Dans le volet de navigation, sélectionnez **Règles de trafic entrant**.
3. Sélectionnez **Action**, puis **Nouvelle règle**.
4. Dans la page **Type de règle** de l’Assistant Nouvelle règle de trafic entrant, sélectionnez **Personnalisée**, puis **Suivant**.
5. Dans la page **Programme**, sélectionnez **Ce chemin d’accès de programme**, tapez « %SystemRoot%\system32\svchost.exe », puis sélectionnez **Suivant**.
6. Dans la page **Protocole et ports**, sélectionnez le type de protocole UDP. Dans **Port local**, sélectionnez « Ports spécifiques », puis tapez 3390.
7. Dans la page **Étendue**, vous pouvez spécifier que la règle s'applique seulement au trafic réseau vers ou depuis les adresses IP entrées sur cette page. Configurez conformément à votre conception, puis sélectionnez **Suivant**.
8. Dans la page **Action**, sélectionnez **Autoriser la connexion**, puis **Suivant**.
9. Dans la page **Profil**, sélectionnez les types d’emplacement réseau auxquels cette règle s’applique, puis sélectionnez **Suivant**.
10. Dans la page **Nom**, tapez un nom et une description pour votre règle, puis sélectionnez **Terminer**.

Vous pouvez vérifier que la nouvelle règle correspond aux captures d’écran ci-dessous : :::image type="content" source="media/rdp-shortpath-firewall-general-tab.png" alt-text="Capture d’écran de l’onglet Général pour la configuration du pare-feu pour les connexions réseau de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-general-tab.png":::

:::image type="content" source="media/rdp-shortpath-firewall-service-settings.png" alt-text="Capture d’écran de l’onglet Programmes et services pour la configuration du pare-feu pour les connexions réseau de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-service-settings.png":::

:::image type="content" source="media/rdp-shortpath-firewall-protocol-and-ports.png" alt-text="Capture d’écran de l’onglet Protocoles et ports pour la configuration du pare-feu pour les connexions réseau de RDP Shortpath" lightbox="media/rdp-shortpath-firewall-protocol-and-ports.png":::

Vous pouvez également utiliser PowerShell pour configurer le Pare-feu Windows :

```powershell
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP'  -PolicyStore PersistentStore -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True
```

### <a name="using-powershell-to-configure-windows-defender-firewall"></a>Utilisation de PowerShell pour configurer le Pare-feu Windows Defender

Vous pouvez également utiliser PowerShell pour configurer une stratégie de groupe.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
$gpoSession = Open-NetGPO -PolicyStore "$domainName\$policyName"
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP'  -PolicyStore PersistentStore -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True
```

## <a name="configuring-azure-network-security-group"></a>Configuration d’un groupe de sécurité réseau Azure

Pour autoriser l’accès à l’écouteur RDP Shortpath dans les limites de sécurité réseau, vous devez configurer le groupe de sécurité réseau Azure de manière à autoriser le port UDP entrant 3390.
Suivez la [documentation sur le groupe de sécurité réseau](../virtual-machines/windows/nsg-quickstart-portal.md) pour créer une règle de sécurité de trafic entrant autorisant le trafic avec les paramètres suivants, :

* **Source** - **Toute** ou la plage d’adresses IP dans laquelle résident les clients
* **Plages de ports sources** - * *\** _ _ **Destination** - **Toute**
* **Plages de ports de destination** - **3390**
* **Protocole** - **UDP**
* **Action** - **Autoriser**
* Si vous le souhaitez, modifiez la **Priorité**. La priorité affecte l’ordre dans lequel les règles sont appliquées : plus la valeur numérique est faible, plus la règle est appliquée précocement.
* **Nom** - - **RDP Shortpath**

### <a name="disabling-rdp-shortpath-for-a-specific-subnet"></a>Désactivation de RDP Shortpath pour un sous-réseau spécifique

Si vous devez bloquer des sous-réseaux spécifiques à l’aide du transport RDP Shortpath, vous pouvez configurer des groupes de sécurité réseau supplémentaires en spécifiant les plages d’adresses IP sources.

## <a name="verifying-the-connectivity"></a>Vérification de la connectivité

### <a name="using-connection-information-dialog"></a>Utilisation de la boîte de dialogue Informations de connexion

Pour vérifier que les connexions utilisent RDP Shortpath, ouvrez la boîte de dialogue « Informations de connexion » en cliquant sur l’icône d’antenne dans la barre d’outils de connexion.

:::image type="content" source="media/rdp-shortpath-connection-bar.png" alt-text="Image de la barre Connexion de Bureau à distance":::

:::image type="content" source="media/rdp-shortpath-connection-info.png" alt-text="Image de la boîte de dialogue Informations de connexion de Bureau à distance":::

### <a name="using-event-logs"></a>Utilisation des journaux des événements

Pour vérifier que la session utilise le transport RDP Shortpath :

1. Connectez-vous au bureau de la machine virtuelle à l’aide du client Windows Virtual Desktop.
2. Lancer l’Observateur d’événements et accédez au nœud suivant : **Journaux des applications et des services > Microsoft > Windows > RemoteDesktopServices-RdpCoreCDV > Microsoft-Windows-RemoteDesktopServices-RdpCoreCDV/Operational**
3. Pour déterminer si le transport RDP Shortpath est utilisé, recherchez l’ID d’événement 131.

### <a name="using-log-analytics-to-verify-shortpath-connectivity"></a>Utilisation de Log Analytics pour vérifier la connectivité de Shortpath

Si vous utilisez [Azure Log Analytics](./diagnostics-log-analytics.md), vous pouvez surveiller les connexions en interrogeant la [table WVDConnections](/azure/azure-monitor/reference/tables/wvdconnections). Une colonne nommée UdpUse indique si la pile RDP de Windows Virtual Desktop utilise le protocole UDP sur la connexion utilisateur actuelle.
Les valeurs possibles sont les suivantes :

* **0** - la connexion utilisateur n’utilise pas RDP Shortpath
* **1** - la connexion utilisateur utilise RDP Shortpath
  
La liste de requêtes suivante vous permet de consulter les informations de connexion. Vous pouvez exécuter cette requête dans l’[éditeur de requête Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md#write-a-query). Pour chaque requête, remplacez `userupn` par l’UPN de l’utilisateur que vous souhaitez rechercher.

```kusto
let Events = WVDConnections | where UserName == "userupn" ;
Events
| where State == "Connected"
| project CorrelationId , UserName, ResourceAlias , StartTime=TimeGenerated, UdpUse, SessionHostName, SessionHostSxSStackVersion
| join (Events
| where State == "Completed"
| project EndTime=TimeGenerated, CorrelationId, UdpUse)
on CorrelationId
| project StartTime, Duration = EndTime - StartTime, ResourceAlias, UdpUse,  SessionHostName, SessionHostSxSStackVersion
| sort by StartTime asc
```

## <a name="troubleshooting"></a>Dépannage

### <a name="verify-shortpath-listener"></a>Vérifier l’écouteur Shortpath

Pour vérifier que l’écouteur UDP est activé, utilisez la commande PowerShell suivante sur l’hôte de la session :

```powershell
Get-NetUDPEndpoint -OwningProcess ((Get-WmiObject win32_service -Filter "name = 'TermService'").ProcessId)  -LocalPort 3390
```

Si cette option est activée, la sortie suivant s’affiche

```dos
LocalAddress                             LocalPort
------------                             ---------
::                                       3390
0.0.0.0                                  3390
```

En cas de conflit, vous pouvez identifier le processus qui occupe le port à l’aide de la commande suivante :

```powershell
Get-Process -id (Get-NetUDPEndpoint  -LocalPort 3390 -LocalAddress 0.0.0.0).OwningProcess
```

## <a name="disabling-rdp-shortpath"></a>Désactivation de RDP Shortpath

Dans certains cas, vous devrez peut-être désactiver le transport RDP Shortpath. Vous pouvez désactiver RDP Shortpath à l’aide de la stratégie de groupe.

### <a name="disabling-rdp-shortpath-on-the-client"></a>Désactivation de RDP Shortpath sur le client

Pour désactiver les RDP Shortpath pour un client spécifique, vous pouvez utiliser la stratégie de groupe suivante afin de désactiver la prise en charge du protocole UDP :

1. Sur le client, exécutez **gpedit.msc**.
2. Accédez à **Configuration ordinateur > Modèles d’administration > Composants Windows > Services Bureau à distance > Client Connexion Bureau à distance**.
3. Définissez le paramètre de **Désactiver UDP sur le client** sur **Activé**.

### <a name="disabling-rdp-shortpath-on-the-session-host"></a>Désactivation de RDP Shortpath sur l’hôte de la session

Afin de désactiver RDP Shortpath pour un hôte de session spécifique, vous pouvez utiliser la stratégie de groupe suivante pour désactiver la prise en charge du protocole UDP :

1. Sur l’hôte de la session, exécutez **gpedit.msc**.
2. Accédez à **Configuration ordinateur > Modèles d’administration > Composants Windows > Services Bureau à distance > Hôte Connexion Bureau à distance > Connexions**.
3. Définissez le paramètre de **Sélectionner des protocoles de transfert RDP** sur **TCP uniquement**.

## <a name="public-preview-feedback"></a>Commentaires sur la préversion publique

Nous aimerions que vous nous fassiez part de votre expérience de la préversion publique.
* Pour toute question, demande, commentaire ou retour d’expérience, utilisez de [formulaire de commentaires](https://aka.ms/RDPShortpathFeedback).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la connectivité réseau de Windows Virtual Desktop, consultez [Comprendre la connectivité réseau de Windows Virtual Desktop](network-connectivity.md).
* Pour vous familiariser avec la qualité de service (QoS) pour Windows Virtual Desktop, consultez [Implémenter la qualité de service (QoS) pour Windows Virtual Desktop](rdp-quality-of-service-qos.md).