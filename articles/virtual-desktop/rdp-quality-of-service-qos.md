---
title: Implémenter QoS (Qualité de service) pour Windows Virtual Desktop (préversion)
titleSuffix: Azure
description: Comment configurer QoS (préversion) pour Windows Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: b61faf74d96e2571e91f7bf9d10eac88cdbf8345
ms.sourcegitcommit: 18046170f21fa1e569a3be75267e791ca9eb67d0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/16/2020
ms.locfileid: "94639067"
---
# <a name="implement-quality-of-service-qos-for-windows-virtual-desktop-preview"></a>Implémenter QoS (Qualité de service) pour Windows Virtual Desktop (préversion)

> [!IMPORTANT]
> La prise en charge de la stratégie QoS (Qualité de service) pour Windows Virtual Desktop est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service, c’est pourquoi nous déconseillons son utilisation pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[RDP Shortpath](./shortpath.md) fournit un transport UDP direct entre le Bureau à distance client et l’Hôte de session. RDP Shortpath permet de configurer des stratégies QoS (Qualité de service) pour les données RDP.
QoS dans Windows Virtual Desktop permet au trafic RDP en temps réel, qui est sensible aux retards réseau, de passer avant le trafic qui y est moins sensible. Un exemple de trafic moins sensible serait le téléchargement d’une nouvelle application, où une seconde supplémentaire nécessaire pour le téléchargement ne constitue pas un gros problème. QoS utilise des objets Stratégie de groupe Windows pour identifier et marquer tous les paquets dans des flux en temps réel et aider votre réseau à fournir au trafic RDP une partie dédiée de la bande passante.

Si vous prenez en charge un grand groupe d’utilisateurs rencontrant l’un des problèmes décrits dans cet article, vous devrez probablement implémenter QoS. Une petite entreprise avec peu d’utilisateurs peut ne pas avoir besoin de QoS, mais cela pourrait quand même lui être utile.

Sans une certaine forme de QoS, les problèmes suivants peuvent survenir :

* Instabilité : les paquets RDP arrivent à des taux différents, ce qui peut entraîner des problèmes visuels et audio
* Perte de paquets : des paquets sont supprimés, ce qui entraîne une retransmission qui nécessite du temps supplémentaire
* Temps d’aller-retour (RTT) retardé : les paquets RDP prennent beaucoup de temps pour atteindre leurs destinations, ce qui entraîne des délais perceptibles entre l’entrée et la réaction de l’application à distance.

La méthode la moins compliquée pour résoudre ces problèmes consiste à augmenter la taille des connexions de données, à la fois en interne et vers Internet. Étant donné que cela a souvent un coût prohibitif, QoS offre un moyen de gérer les ressources que vous avez de manière plus efficace au lieu d’ajouter de la bande passante. Pour résoudre les problèmes de qualité, nous vous recommandons d’utiliser d’abord QoS, puis d’ajouter de la bande passante uniquement lorsque cela est nécessaire.

Pour que QoS soit efficace, vous devez appliquer des paramètres QoS cohérents à l’échelle de votre organisation. Toute partie du chemin qui échoue à supporter vos priorités QoS peut dégrader la qualité de la session RDP.

## <a name="introduction-to-qos-queues"></a>Présentation des files d’attente QoS

Pour fournir QoS, les appareils réseau doivent disposer d’un moyen de classer le trafic et être en mesure de distinguer le trafic RDP de tout autre trafic réseau.

Lorsque le trafic réseau entre dans un routeur, le trafic est placé dans une file d’attente. Si aucune stratégie QoS n’est configurée, il n’y a qu’une seule file d’attente et toutes les données sont traitées selon le principe « premier arrivé, premier sorti » avec la même priorité. Cela signifie que le trafic RDP peut rester bloqué derrière du trafic où un délai de quelques millisecondes supplémentaires ne serait pas un problème.

Lorsque vous implémentez QoS, vous définissez plusieurs files d’attente à l’aide de l’une des nombreuses fonctionnalités de gestion de la congestion, telles que la mise en file d’attente prioritaire de Cisco et [CBWFQ (Class-Based Weighted Fair Queueing, mise en file d’attente juste pondérée basée sur la classe)](https://www.cisco.com/en/US/docs/ios/12_0t/12_0t5/feature/guide/cbwfq.html#wp17641) et des fonctionnalités d’évitement de congestion, par exemple [WRED (weighted random early detection, détection anticipée aléatoire pondérée)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_conavd/configuration/15-mt/qos-conavd-15-mt-book/qos-conavd-cfg-wred.html).

Une analogie simple est que QoS crée des « files de covoiturage » virtuelles dans votre réseau de données. Donc, certains types de données ne subissent jamais, voire rarement, de délai. Une fois que vous avez créé ces files, vous pouvez ajuster leur taille relative et gérer plus efficacement la bande passante de connexion dont vous disposez tout en offrant des expériences de qualité professionnelle aux utilisateurs de votre organisation.

## <a name="qos-implementation-checklist"></a>Liste de contrôle de l’implémentation de QoS

À un niveau élevé, procédez comme suit pour implémenter QoS :

1. [Vérifiez que votre réseau est prêt](#make-sure-your-network-is-ready)
2. [Vérifiez que RDP Shortpath est activé](./shortpath.md) : les stratégies QoS ne sont pas prises en charge pour le transport de connexion inverse
3. [Implémentez l’insertion de marqueurs DSCP](#insert-dscp-markers) sur les hôtes de session

Lorsque vous vous préparez à implémenter QoS, gardez les instructions suivantes à l’esprit :

* Le chemin le plus court vers l’hôte de session est le mieux adapté
* Tous les obstacles, tels que les proxys ou les appareils d’inspection des paquets, ne sont pas recommandés

## <a name="make-sure-your-network-is-ready"></a>Vérifier que votre réseau est prêt

Si vous envisagez d’implémenter QoS, vous devez déjà avoir déterminé vos besoins en bande passante et d’autres [exigences relatives au réseau](/windows-server/remote/remote-desktop-services/network-guidance?context=/azure/virtual-desktop/context/context).
  
La congestion du trafic sur un réseau aura un impact significatif sur la qualité des supports. Un manque de bande passante entraîne une dégradation des performances et une expérience utilisateur médiocre. À mesure que l’adoption et l’utilisation de Windows Virtual Desktop augmentent, utilisez [Log Analytics](./diagnostics-log-analytics.md) pour identifier les problèmes et effectuer les ajustements nécessaires à l’aide de QoS et d’ajouts de bande passante sélectifs.

### <a name="vpn-considerations"></a>Considérations relatives au réseau privé virtuel (VPN)

QoS fonctionne uniquement comme prévu lorsqu’il est implémenté sur toutes les liaisons entre les clients et les hôtes de session. Si vous utilisez QoS sur un réseau interne et qu’un utilisateur se connecte à partir d’un emplacement à distance, vous ne pouvez prioriser le trafic que sur votre réseau interne managé. Bien que les emplacements à distance puissent recevoir une connexion managée par l’implémentation d’un réseau privé virtuel (VPN), un VPN ajoute par nature une surcharge de paquets et crée des retards dans le trafic en temps réel.

Dans une organisation mondiale avec des liaisons managées entre plusieurs continents, nous recommandons vivement d’utiliser QoS, car la bande passante pour ces liaisons est limitée par rapport au réseau local.

## <a name="insert-dscp-markers"></a>Insérer des marqueurs DSCP

Vous pouvez implémenter QoS à l’aide d’un objet Stratégie de groupe (GPO) pour demander aux hôtes de session d’insérer un marqueur DSCP dans les en-têtes de paquets IP afin d’identifier ces paquets comme un type de trafic particulier. Les routeurs et autres appareils réseau peuvent être configurés pour reconnaître ces marqueurs et placer le trafic dans une file d’attente distincte avec une priorité plus élevée.

Vous pouvez comparer les marqueurs DSCP à des timbres qui indiquent aux employés postaux le niveau d’urgence de la livraison et la meilleure façon de trier ces éléments pour une livraison rapide. Une fois que vous avez configuré votre réseau pour prioriser les flux RDP, le nombre de paquets perdus et de paquets en retard devrait diminuer considérablement.

Lorsque tous les appareils réseau utilisent les mêmes classifications, marqueurs et priorités, il est possible de réduire ou d’éliminer les retards, les paquets ignorés et l’instabilité. Du point de vue du trafic RDP, l’étape de configuration essentielle est la classification et le marquage des paquets. Toutefois, pour que QoS réussisse de bout en bout, vous devez également aligner avec soin la configuration RDP et la configuration réseau sous-jacente.
La valeur DSCP indique à un réseau configuré correspondant la priorité à attribuer à un paquet ou à un flux.

Nous vous recommandons d’utiliser la valeur DSCP 46 qui est mappée à la classe DSCP **EF (Expedited Forwarding, transfert express)** .

### <a name="implement-qos-on-session-host-using-group-policy"></a>Implémenter QoS sur l’hôte de session à l’aide de Stratégie de groupe

Vous pouvez utiliser QoS (Qualité de service) basée sur la stratégie dans Stratégie de groupe pour définir la valeur DSCP prédéfinie.

Pour créer une stratégie QoS pour les hôtes de session joints à un domaine, connectez-vous d’abord à un ordinateur sur lequel la Gestion des stratégies de groupe a été installée. Ouvrez Gestion des stratégie de groupe (sélectionnez Démarrer, pointez sur Outils d’administration, puis sélectionnez Gestion des stratégie de groupe), puis procédez comme suit :

1. Dans Gestion des stratégies de groupe, localisez le conteneur dans lequel la nouvelle stratégie doit être créée. Par exemple, si tous vos ordinateurs hôtes de session sont situés dans une unité d’organisation nommée **« hôtes de session »** , la nouvelle stratégie doit être créée dans l’unité d’organisation Hôtes de session.

2. Cliquez avec le bouton droit sur le conteneur approprié, puis sélectionnez **Créer un objet GPO dans ce domaine, et le lier ici**.

3. Dans la boîte de dialogue **Nouvel objet Stratégie de groupe**, tapez le nom du nouvel objet Stratégie de groupe dans la zone **Nom** et sélectionnez **OK**.

4. Cliquez avec le bouton droit sur la nouvelle stratégie, puis sélectionnez **Modifier**.

5. Dans l’Éditeur Gestion des stratégies de groupe, développez **Configuration de l’ordinateur**, développez **Paramètres Windows**, cliquez avec le bouton droit sur **QoS basée sur la stratégie**, puis sélectionnez **Créer une stratégie**.

6. Dans la boîte de dialogue **QoS basée sur la stratégie**, dans la page d’ouverture, tapez un nom pour la nouvelle stratégie dans la zone **Nom**. Sélectionnez **Spécifier la valeur DSCP** et définissez la valeur sur **46**. Laissez l’option **Spécifier le taux d’accélération du trafic sortant** désélectionnée, puis sélectionnez **Suivant**.

7. Sur la page suivante, sélectionnez **Uniquement les applications avec ce nom d’exécutable** et entrez le nom **svchost.exe**, puis sélectionnez **Suivant**. Ce paramètre indique à la stratégie de prioriser uniquement le trafic correspondant à partir du service Bureau à distance.

8. Sur la troisième page, assurez-vous que les options **Toute adresse IP source** et **Toute adresse IP de destination** sont sélectionnées, puis sélectionnez **Suivant**. Ces deux paramètres garantissent que les paquets seront gérés quels que soient l’ordinateur (adresse IP) qui a envoyé les paquets et l’ordinateur (adresse IP) qui recevra les paquets.

9. Sur la quatrième page, sélectionnez **UDP** dans la liste déroulante **Sélectionner le protocole auquel cette stratégie QoS s’applique**.

10. Sous le titre **Spécifier le numéro de port source**, sélectionnez **À partir de ce port ou de cette plage source**. Dans la zone de texte correspondante, tapez **3390**. Sélectionnez **Terminer**.

Les nouvelles stratégies que vous avez créées ne prennent pas effet tant que la Stratégie de groupe n’a pas été actualisée sur vos ordinateurs hôtes de session. Bien que la Stratégie de groupe s’actualise régulièrement, vous pouvez forcer une actualisation immédiate en procédant comme suit :

1. Sur chaque hôte de session pour lequel vous souhaitez actualiser la Stratégie de groupe, ouvrez une invite de commandes en tant qu’administrateur (*Exécuter en tant qu’administrateur*).

1. À l’invite de commandes, tapez

   ```console
   gpupdate /force
   ```

### <a name="implement-qos-on-session-host-using-powershell"></a>Implémenter QoS sur l’hôte de session à l’aide de PowerShell

Vous pouvez définir QoS pour RDP Shortpath à l’aide de la cmdlet PowerShell ci-dessous :

```powershell
New-NetQosPolicy -Name "RDP Shortpath" -AppPathNameMatchCondition "svchost.exe" -IPProtocolMatchCondition UDP -IPSrcPortStartMatchCondition 3390 -IPSrcPortEndMatchCondition 3390 -DSCPAction 46 -NetworkProfile All
```

## <a name="related-articles"></a>Articles connexes

* [Stratégie QoS (Qualité de service)](/windows-server/networking/technologies/qos/qos-policy-top)

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les besoins en bande passante pour Windows Virtual Desktop, consultez [Comprendre les exigences en bande passante de RDP (Remote Desktop Protocol) pour Windows Virtual Desktop](rdp-bandwidth.md).
* Pour en savoir plus sur la connectivité réseau de Windows Virtual Desktop, consultez [Comprendre la connectivité réseau de Windows Virtual Desktop](network-connectivity.md).
