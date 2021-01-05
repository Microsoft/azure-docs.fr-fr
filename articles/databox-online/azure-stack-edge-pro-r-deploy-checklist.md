---
title: Liste de contrôle préalable au déploiement d’un appareil Azure Stack Edge Pro R | Microsoft Docs
description: Cet article décrit les informations qui peuvent être collectées avant de déployer un appareil Azure Stack Edge Pro R.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 10/19/2020
ms.author: alkohli
ms.openlocfilehash: 3a4428b2e6f42facd8230fa4d9f6f637f6fe4659
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531736"
---
# <a name="deployment-checklist-for-your-azure-stack-edge-pro-r-device"></a>Liste de contrôle du déploiement d’un appareil Azure Stack Edge Pro R  

Cet article décrit les informations qui peuvent être collectées avant le déploiement proprement dit d’un appareil Azure Stack Edge Pro R. 

Suivez la liste de contrôle ci-dessous pour vérifier que vous disposez de ces informations après avoir passé commande d’un appareil Azure Stack Edge Pro R et avant de le recevoir. 

## <a name="deployment-checklist"></a>Liste de vérification de déploiement 

| Étape                             | Paramètre                                                                                                                                                                                                                           | Détails                                                                                                           |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Gestion des appareils               | <li>Abonnement Azure</li><li>Fournisseurs de ressources inscrits</li><li>Compte de Stockage Azure</li>|<li>Activé pour Azure Stack Edge Pro/Data Box Gateway, accès Propriétaire ou Contributeur.</li><li>Sur le portail Azure, accédez à **Accueil > Abonnements > Votre abonnement > Fournisseurs de ressources**. Recherchez `Microsoft.DataBoxEdge` et inscrivez-vous. Répétez cette opération pour `Microsoft.Devices` si vous déployez des charges de travail IoT.</li><li>Informations d’identification d’accès requises</li> |
| Installation des périphériques               | Câbles d’alimentation dans le package. <br>Pour les États-Unis, un câble SVE 18/3 homologué 125 V et 15 A avec un connecteur NEMA 5-15P vers C13 (entrée vers sortie) est livré. | Pour plus d’informations, consultez la liste des [Cordons d’alimentation pris en charge selon les pays](azure-stack-edge-technical-specifications-power-cords-regional.md).  |
|                                   | <li>Au moins un câble réseau 1-GbE RJ-45 pour le port 1  </li><li> Au moins un câble cuivre 25-GbE SFP+ pour le port 3 et 4</li>| Il revient au client de se procurer ces câbles.<br>Pour obtenir la liste complète des câbles réseau, des commutateurs et des transmetteurs pris en charge pour les cartes réseau de l’appareil, consultez [Matrice d’interopérabilité de la série Cavium FastlinQ 41000](https://www.marvell.com/documents/xalflardzafh32cfvi0z/) et [Produits compatibles avec la carte réseau Mellanox double port 25G ConnectX-4 canaux](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).| 
| Première connexion de l’appareil      | <li>Ordinateur portable dont les paramètres IPv4 peuvent être modifiés. Cet ordinateur portable se connecte au Port 1 via un commutateur ou un adaptateur USB vers Ethernet.  </li><!--<li> A minimum of 1 GbE switch must be used for the device once the initial setup is complete. The local web UI will not be accessible if the connected switch is not at least 1 Gbe.</li>-->|   |
| Connexion de l’appareil                       | Mot de passe de l'administrateur de l'appareil, compris entre 8 et 16 caractères et contenant trois des éléments suivants : majuscules, minuscules, chiffres et caractères spéciaux.                                            | Le mot de passe par défaut, *Password1*, expire à la première connexion.                                                     |
| Paramètres réseau                  | L’appareil est fourni avec des ports réseau 2 × 1 GbE, 4 × 25 GbE. <li>Le port 1 n’est utilisé que pour configurer les paramètres de gestion. Il est possible de connecter et de configurer un ou plusieurs ports de données. </li><li> Il faut qu’au moins une interface réseau de données entre le port 2 et le port 6 soit connectée à Internet (avec connectivité à Azure).</li><li> Prise en charge du protocole DHCP et de la configuration IPv4 statique. | L’adresse IP, le serveur DNS et la passerelle par défaut sont nécessaires pour la configuration IPv4 statique.   |
| Paramètres réseau de calcul     | <li>Deux adresses IP libres, statiques et contiguës sont requises pour les nœuds Kubernetes, ainsi qu'une  adresse IP statique pour le service IoT Edge.</li><li>Une adresse IP supplémentaire est requise pour chaque service ou module supplémentaire déployé.</li>| Seule la configuration IPv4 statique est prise en charge.|
| (Facultatif) Paramètres de proxy web     | <li>Adresse IP/nom de domaine complet du serveur proxy web, port </li><li>Nom d'utilisateur du proxy web, mot de passe</li> | Proxy web non pris en charge avec l'installation du calcul. |
| Paramètre de pare-feu et de port        | Si vous utilisez un pare-feu, vérifiez que les [modèles et ports d'URL listés](azure-stack-edge-system-requirements.md#networking-port-requirements) sont autorisés pour les adresses IP des appareils. |  |
| (Recommandé) Paramètres d’heure       | Configurer le fuseau horaire, le serveur NTP principal et le serveur NTP secondaire. | Configurez le serveur NTP principal et secondaire sur le réseau local.<br>Si le serveur local n’est pas disponible, des serveurs NTP publics peuvent être configurés.                                                    |
| (Facultatif) Paramètres du serveur de mise à jour | <li>Exiger l’adresse IP du serveur de mise à jour sur le réseau local, chemin du serveur WSUS. </li> | Par défaut, c’est le serveur Windows Update public qui est utilisé.|
| Paramètres de l’appareil | <li>Nom de domaine complet (FQDN) de l'appareil </li><li>Domaine DNS</li> | |
| (Facultatif) Certificats  | Si vous apportez vos propres certificats comprenant la ou les chaînes de signature, [ajoutez les certificats](azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption.md#bring-your-own-certificates) au format approprié.| Ne configurez des certificats que si vous changez le nom de l'appareil et/ou le domaine DNS. |
| VPN  | <!--Need VPN certificate, VPN gateway, firewall setup in Azure,  passphrase and region info VPN scripts. -->   | |
| Chiffrement des données au repos  | Recommandez l’utilisation de la clé de chiffrement générée automatiquement.   |Si vous utilisez votre propre clé, vous avez besoin d’une clé encodée base-64 de 32 caractères.  |
| Activation  | Exiger une clé d’activation de la ressource Azure Stack Edge Pro/Data Box Gateway.    | Une fois générée, la clé expire au bout de trois jours. |

<!--
| (Optional) MAC Address            | If MAC address needs to be approved, get the address of the connected port from local UI of the device. |                                                                                                                   |
| (Optional) Network switch port    | Device hosts Hyper-V VMs for compute. Some network switch port configurations don’t accommodate these setups by default.                                                                                                        |                                                                                                                   |-->


## <a name="next-steps"></a>Étapes suivantes

Préparez le déploiement de votre [appareil Azure Stack Edge Pro](azure-stack-edge-pro-r-deploy-prep.md).
