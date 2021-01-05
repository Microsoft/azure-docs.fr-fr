---
title: Détecter un problème de Bureau à distance client dans Windows Virtual Desktop – Azure
description: Comment résoudre des problèmes lorsque vous configurez des connexions à un client dans un environnement de locataire Windows Virtual Desktop.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 08/11/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c6083dbcc270c0e9dde1da45ed01369d03146237
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108964"
---
# <a name="troubleshoot-the-remote-desktop-client"></a>Résoudre des problèmes du client Bureau à distance

Cet article décrit les problèmes courants liés au client Bureau à distance et leur résolution.

## <a name="remote-desktop-client-for-windows-7-or-windows-10-stops-responding-or-cannot-be-opened"></a>Le client Bureau à distance de Windows 7 ou Windows 10 ne répond plus ou ne peut pas être ouvert

Depuis la version 1.2.790, vous pouvez réinitialiser les données utilisateur à partir de la page À propos de ou à l’aide d’une commande.

Utilisez la commande suivante pour supprimer vos données utilisateur, restaurer les paramètres par défaut et annuler l’abonnement à tous les espaces de travail.

```cmd
msrdcw.exe /reset [/f]
```

Si vous utilisez une version antérieure du client Bureau à distance, nous vous recommandons de désinstaller et réinstaller le client.

## <a name="web-client-wont-open"></a>Le client Web ne s’ouvre pas

Tout d’abord, testez votre connexion Internet en ouvrant un autre site web dans votre navigateur. par exemple, [www.bing.com](https://www.bing.com).

Utilisez **nslookup** pour vérifier que DNS peut résoudre le nom de domaine complet :

```cmd
nslookup rdweb.wvd.microsoft.com
```

Essayez de vous connecter avec un autre client, comme le client Bureau à distance pour Windows 7 ou Windows 10, puis vérifiez si vous pouvez ouvrir le client web.

### <a name="cant-open-other-websites-while-connected-to-the-web-client"></a>Impossible d’ouvrir d’autres sites Web en étant connecté au client Web

Si vous ne pouvez pas ouvrir d’autres sites Web alors que vous êtes connecté au client Web, il peut y avoir des problèmes de connexion réseau ou une panne réseau. Nous vous recommandons de contacter le support réseau.

### <a name="nslookup-cant-resolve-the-name"></a>Nslookup ne peut pas résoudre le nom

Si nslookup ne peut pas résoudre le nom, il peut y avoir des problèmes de connexion réseau ou une panne réseau. Nous vous recommandons de contacter le support réseau.

### <a name="your-client-cant-connect-but-other-clients-on-your-network-can-connect"></a>Votre client ne peut pas se connecter, mais d’autres clients de votre réseau peuvent se connecter

Si votre navigateur commence à se comporter bizarrement ou cesse de fonctionner pendant que vous utilisez le client Web, suivez ces instructions pour résoudre le problème :

1. Redémarrez le navigateur.
2. Effacez les cookies du navigateur. Consultez l’article [How to delete cookie files in Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer) (Suppression des fichiers de cookie dans Internet Explorer).
3. Videz le cache du navigateur. Consultez l’article [Clear browser cache for your browser](https://binged.it/2RKyfdU) (Vider le cache du navigateur).
4. Ouvrez le navigateur en mode privé.

## <a name="client-doesnt-show-my-resources"></a>Le client n’affiche pas mes ressources

Tout d’abord, vérifiez le compte Azure Active Directory que vous utilisez. Si vous vous êtes déjà connecté avec un compte Azure Active Directory différent de celui que vous voulez utiliser pour Windows Virtual Desktop, vous devez vous déconnecter ou utiliser une fenêtre de navigateur privée.

Si vous utilisez Windows Virtual Desktop (classique), suivez le lien du client web de [cet article](./virtual-desktop-fall-2019/connect-web-2019.md) pour vous connecter à vos ressources.

Si cela ne fonctionne pas, assurez-vous que votre groupe d’applications est associé à un espace de travail.

## <a name="web-client-stops-responding-or-disconnects"></a>Le client web cesse de répondre ou se déconnecte

Essayez de vous connecter à l’aide d’un autre navigateur ou client.

### <a name="other-browsers-and-clients-also-malfunction-or-fail-to-open"></a>d’autres navigateurs et clients connaissent également un dysfonctionnement ou ne parviennent pas à s’ouvrir

Si les problèmes se poursuivent même après avoir changé de navigateurs, le problème peut ne pas être lié à votre navigateur, mais à votre réseau. Nous vous recommandons de contacter le support réseau.

## <a name="web-client-keeps-prompting-for-credentials"></a>Le client web ne cesse de demander les informations d’identification

Si le client web ne cesse de demander les informations d’identification, suivez ces instructions :

1. Vérifiez que l’URL du client web est correcte.
2. Vérifiez que les informations d’identification que vous utilisez correspondent à l’environnement Windows Virtual Desktop lié à l’URL.
3. Effacez les cookies du navigateur. Pour plus d’informations, voir [Comment faire pour supprimer des fichiers cookie dans Internet Explorer](https://support.microsoft.com/help/278835/how-to-delete-cookie-files-in-internet-explorer).
4. Videz le cache du navigateur. Pour plus d’informations, voir [Vider le cache du navigateur](https://binged.it/2RKyfdU).
5. Ouvrez votre navigateur en mode privé.

## <a name="windows-client-blocks-windows-virtual-desktop-classic-feed"></a>Le client Windows bloque le flux Windows Virtual Desktop (classique)

Si le flux du client Windows n’affiche pas les applications Windows Virtual Desktop (classique), suivez ces instructions :

1. Vérifiez si la stratégie d’accès conditionnel comprend les ID d’application associés à Windows Virtual Desktop (classique).
2. Vérifiez si la stratégie d’accès conditionnel bloque tous les accès à l’exception des ID d’application Windows Virtual Desktop (classique). Si c’est le cas, vous devez ajouter l’ID d’application **9cdead84-a844-4324-93f2-b2e6bb768d07** à la stratégie pour permettre au client de découvrir les flux.

Si vous ne trouvez pas l’ID d’application 9cdead84-a844-4324-93f2-b2e6bb768d07 dans la liste, vous devez inscrire le fournisseur de ressources Windows Virtual Desktop. Pour inscrire le fournisseur de ressources :

1. Connectez-vous au portail Azure.
2. Accédez à **Abonnements**, puis sélectionnez votre abonnement.
3. Dans le menu sur le côté gauche de la page, sélectionnez **Fournisseur de ressources**.
4. Recherchez et sélectionnez **Microsoft.DesktopVirtualization**, puis **Ré-inscrire**.

## <a name="next-steps"></a>Étapes suivantes

- Pour découvrir une vue d’ensemble de la résolution des problèmes Windows Virtual Desktop et des procédures d’escalade, consultez l’article [Vue d’ensemble du dépannage, commentaires et support](troubleshoot-set-up-overview.md).
- Pour résoudre des problèmes lors de la création d’un environnement Windows Virtual Desktop et d’un pool d’hôtes dans un environnement Windows Virtual Desktop, consultez [Création d’un environnement et d’un pool d’hôtes](troubleshoot-set-up-issues.md).
- Pour résoudre les problèmes de configuration d’une machine virtuelle dans Windows Virtual Desktop, consultez [Configuration d’une machine virtuelle hôte de session](troubleshoot-vm-configuration.md).
- Pour résoudre les problèmes d’utilisation de PowerShell avec Windows Virtual Desktop, consultez [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md).
- Suivez le [Didacticiel : Résoudre les problèmes liés aux déploiements de modèles Resource Manager](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
