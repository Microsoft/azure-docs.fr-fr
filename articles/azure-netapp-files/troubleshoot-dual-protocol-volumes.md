---
title: Résoudre les problèmes liés aux volumes à deux protocoles pour Azure NetApp Files | Microsoft Docs
description: Décrit les messages d’erreur et les résolutions qui peuvent vous aider à résoudre les problèmes de double protocole pour Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 10/01/2020
ms.author: b-juche
ms.openlocfilehash: cc046a27fec1b9e361ff840c7ae0f077e2987b67
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91654055"
---
# <a name="troubleshoot-dual-protocol-volumes"></a>Résoudre les problèmes des volumes à deux protocoles

Cet article décrit les résolutions des conditions d’erreur que vous pouvez rencontrer lors de la création ou de la gestion de volumes à deux protocoles.

## <a name="error-conditions-and-resolutions"></a>Conditions d’erreur et résolutions

|     Conditions d’erreur    |     Résolution    |
|-|-|
| La création d’un volume à deux protocoles échoue avec l’erreur `This Active Directory has no Server root CA Certificate`.    |     Si cette erreur se produit lorsque vous créez un volume à deux protocoles, assurez-vous que le certificat racine de l’autorité de certification est chargé dans votre compte NetApp.    |
| La création d’un volume à deux protocoles échoue avec l’erreur `Failed to validate LDAP configuration, try again after correcting LDAP configuration`.    |  Tenez compte des résolutions suivantes :   <ul><li>Assurez-vous que le certificat racine requis est ajouté lorsque vous joignez Active Directory (AD) au compte NetApp. Voir [Charger le certificat racine public de l’autorité de certification Active Directory](create-volumes-dual-protocol.md#upload-active-directory-certificate-authority-public-root-certificate).   </li><li>L’enregistrement du pointeur (PTR) de l’ordinateur hôte AD est peut-être manquant sur le serveur DNS. Vous devez créer une zone de recherche inversée sur le serveur DNS, puis ajouter un enregistrement PTR de l’ordinateur hôte AD dans cette zone de recherche inversée. <br> Par exemple, supposons que l’adresse IP de l’ordinateur AD est `1.1.1.1`, le nom d’hôte de l’ordinateur AD (trouvé à l’aide de la commande `hostname`) est `AD1` et le nom de domaine est `myDomain.com`.  L’enregistrement PTR ajouté à la zone de recherche inversée doit être `1.1.1.1` -> `AD1.myDomain.com`. </li></ul>  |
| La création d’un volume à deux protocoles échoue avec l’erreur `Failed to create the Active Directory machine account \\\"TESTAD-C8DD\\\". Reason: Kerberos Error: Pre-authentication information was invalid Details: Error: Machine account creation procedure failed\\n [ 434] Loaded the preliminary configuration.\\n [ 537] Successfully connected to ip 1.1.1.1, port 88 using TCP\\n**[ 950] FAILURE`. |  Cette erreur indique que le mot de passe AD est incorrect lorsqu’Active Directory est joint au compte NetApp. Mettez à jour la connexion AD avec le mot de passe correct, puis réessayez. |
| La création d’un volume à deux protocoles échoue avec l’erreur `Could not query DNS server. Verify that the network configuration is correct and that DNS servers are available`. |   Cette erreur indique que DNS n’est pas accessible. Cela peut être dû au fait que l’adresse IP DNS est incorrecte ou qu’il y a un problème réseau. Vérifiez l’adresse IP DNS entrée dans la connexion AD et assurez-vous que l’adresse IP est correcte. <br> En outre, assurez-vous que l’instance AD et le volume se trouvent dans la même région et dans le même réseau virtuel. S’ils se trouvent dans des réseaux virtuels différents, assurez-vous que l’appairage de réseaux virtuels est établi entre les deux réseaux virtuels.|
| Erreur « Autorisation refusée » lors du montage d’un volume à deux protocoles. | Un volume à double protocoles prend en charge les protocoles NFS et SMB.  Lorsque vous tentez d’accéder au volume monté sur le système UNIX, celui-ci tente de mapper l’utilisateur UNIX que vous utilisez à un utilisateur Windows. Si aucun mappage n’est trouvé, l’erreur « Autorisation refusée » se produit. <br> Cette situation s’applique également lorsque vous utilisez l’utilisateur « racine » pour l’accès. <br> Pour éviter l’erreur « Autorisation refusée », assurez-vous que Windows Active Directory inclut `pcuser` avant d’accéder au point de montage. Si vous ajoutez `pcuser` après avoir rencontré l’erreur « Autorisation refusée », patientez 24 heures jusqu’à ce que l’entrée du cache s’efface avant de réessayer d’accéder. |

## <a name="next-steps"></a>Étapes suivantes  

* [Créer un volume à deux protocoles](create-volumes-dual-protocol.md)
* [Configurer un client NFS pour Azure NetApp Files](configure-nfs-clients.md)
