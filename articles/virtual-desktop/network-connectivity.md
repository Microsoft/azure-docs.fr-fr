---
title: Comprendre la connectivité réseau de Windows Virtual Desktop
titleSuffix: Azure
description: Découvrir la connectivité réseau de Windows Virtual Desktop
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: 4c0017a36d84973a4d99c49a5ea33faeb189b35f
ms.sourcegitcommit: 18046170f21fa1e569a3be75267e791ca9eb67d0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2020
ms.locfileid: "94639061"
---
# <a name="understanding-windows-virtual-desktop-network-connectivity"></a>Comprendre la connectivité réseau de Windows Virtual Desktop

Windows Virtual Desktop offre la possibilité d’héberger des sessions clientes sur les hôtes de session qui s’exécutent sur Azure. Microsoft gère des parties des services au nom du client et fournit des points de terminaison sécurisés pour la connexion des clients et des hôtes de session. Le diagramme ci-dessous donne une vue d’ensemble des connexions réseau utilisées par Windows Virtual Desktop.

:::image type="content" source="media/windows-virtual-desktop-network-connections.svg" alt-text="Diagramme des connexions réseau de Windows Virtual Desktop" lightbox="media/windows-virtual-desktop-network-connections.svg":::

## <a name="session-connectivity"></a>Connectivité des sessions

Windows Virtual Desktop utilise RDP (Remote Desktop Protocol) pour fournir des fonctionnalités de saisie et d’affichage à distance sur les connexions réseau. Le protocole RDP a initialement été publié avec Windows NT 4.0 Terminal Server Edition et a continué d’évoluer avec chaque version de Microsoft Windows et de Windows Server. Depuis le début, RDP a été développé pour être indépendant de sa pile de transport sous-jacente. Il prend désormais en charge plusieurs types de transport.

## <a name="reverse-connect-transport"></a>Transport de connexion inverse

Windows Virtual Desktop utilise le transport de connexion inverse pour établir la session à distance et pour transporter le trafic RDP. Contrairement aux déploiements de Services Bureau à distance locaux, le transport de connexion inverse n’utilise pas un écouteur TCP pour recevoir les connexions RDP entrantes. Au lieu de cela, il utilise la connectivité sortante vers l’infrastructure Windows Virtual Desktop via la connexion HTTPS.

## <a name="session-host-communication-channel"></a>Canal de communication de l’hôte de session

Lors du démarrage de l’hôte de session de Windows Virtual Desktop, le service Chargeur d’agent Bureau à distance établit le canal de communication permanent du répartiteur Windows Virtual Desktop. Ce canal de communication est superposé à une connexion TLS (Transport Layer Security) sécurisée et sert de bus pour l’échange de messages de service entre l’hôte de session et l’infrastructure de Windows Virtual Desktop.

## <a name="client-connection-sequence"></a>Séquence de connexion du client

Séquence de connexion du client décrite ci-dessous :

1. L’utilisateur utilise le client Windows Virtual Desktop pris en charge pour s’abonner à l’espace de travail Windows Virtual Desktop
2. Azure Active Directory authentifie l’utilisateur et retourne le jeton utilisé pour énumérer les ressources disponibles pour un utilisateur
3. Le client passe le jeton au service d’abonnement au flux Windows Virtual Desktop
4. Le service d’abonnement au flux Windows Virtual Desktop valide le jeton
5. Le service d’abonnement au flux Windows Virtual Desktop transmet la liste des bureaux et RemoteApps disponibles au client sous la forme d’une configuration de connexion signée numériquement
6. Le client stocke la configuration de la connexion pour chaque ressource disponible dans un ensemble de fichiers .rdp
7. Lorsqu’un utilisateur sélectionne la ressource à laquelle se connecter, le client utilise le fichier .rdp associé et établit la connexion TLS 1.2 sécurisée à l’instance de passerelle Windows Virtual Desktop la plus proche et transmet les informations de connexion
8. La passerelle Windows Virtual Desktop valide la requête et demande au répartiteur Windows Virtual Desktop d’orchestrer la connexion
9. Le répartiteur Windows Virtual Desktop identifie l’hôte de session et utilise le canal de communication permanent établi précédemment pour initialiser la connexion
10. La pile Bureau à distance initie la connexion TLS 1.2 à la même instance de passerelle Windows Virtual Desktop que celle utilisée par le client
11. Une fois que le client et l’hôte de session sont connectés à la passerelle, celle-ci commence à relayer les données brutes entre les deux points de terminaison, ce qui établit le transport de connexion inverse de base pour le protocole RDP
12. Une fois le transport de base défini, le client commence à établir la liaison RDP

## <a name="connection-security"></a>Sécurité de la connexion

TLS 1.2 est utilisé pour toutes les connexions initiées à partir des clients et des hôtes de session vers les composants de l’infrastructure de Windows Virtual Desktop.
Pour le transport de connexion inverse, le client et l’hôte de session se connectent à la passerelle Windows Virtual Desktop. Après avoir établi la connexion TCP, le client ou l’hôte de session valide le certificat de la passerelle Windows Virtual Desktop.
Une fois le transport de base établi, le protocole RDP établit une connexion TLS imbriquée entre le client et l’hôte de session à l’aide des certificats de l’hôte de session. Par défaut, le certificat utilisé pour le chiffrement RDP est généré automatiquement par le système d’exploitation pendant le déploiement. Si vous le souhaitez, les clients peuvent déployer des certificats gérés de manière centralisée émis par l’autorité de certification d’entreprise. Pour plus d’informations sur la configuration des certificats, consultez la [documentation de Windows Server](/troubleshoot/windows-server/remote/remote-desktop-listener-certificate-configurations).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les besoins en bande passante pour Windows Virtual Desktop, consultez [Comprendre les exigences en bande passante de RDP (Remote Desktop Protocol) pour Windows Virtual Desktop](rdp-bandwidth.md).
* Pour vous familiariser avec QoS (Qualité de service) pour Windows Virtual Desktop, consultez [Implémenter QoS (Qualité de service) pour Windows Virtual Desktop](rdp-quality-of-service-qos.md).
