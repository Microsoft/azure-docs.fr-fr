---
title: Diagnostiquer les problèmes de configuration des liaisons privées sur Azure Key Vault
description: Résoudre les problèmes courants liés aux liaisons privées avec Key Vault et approfondir la configuration
author: msfcolombo
ms.author: fcolombo
ms.date: 09/30/2020
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: 03abe4e4e098d46060e33ba114872905e54a443f
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317059"
---
# <a name="diagnose-private-links-configuration-issues-on-azure-key-vault"></a>Diagnostiquer les problèmes de configuration des liaisons privées sur Azure Key Vault

## <a name="introduction"></a>Introduction

Cet article aide les utilisateurs à diagnostiquer et à résoudre les problèmes liés à Key Vault et à la fonctionnalité de liaisons privées. Ce guide vous aide sur les aspects de la configuration, tels que la première obtention de liaisons privées ou la résolution d’une situation dans laquelle des liaisons privées cessent de fonctionner en raison d’une modification.

Si vous ne connaissez pas cette fonctionnalité, consultez [Intégrer Key Vault avec Azure Private Link](private-link-service.md).

### <a name="problems-covered-by-this-article"></a>Problèmes traités dans cet article

- Vos requêtes DNS renvoient toujours une adresse IP publique pour le coffre de clés au lieu d’une adresse IP privée que vous attendez de l’utilisation de la fonctionnalité de liaisons privées.
- Toutes les requêtes effectuées par un client donné qui utilise une liaison privée échouent avec des délais d’attente ou des erreurs réseau, et le problème n’est pas intermittent.
- Le coffre de clés a une adresse IP privée, mais les requêtes reçoivent toujours une réponse `403` avec le code d’erreur interne `ForbiddenByFirewall`.
- Vous utilisez des liaisons privées, mais votre coffre de clés accepte toujours des requêtes provenant de l’Internet public.
- Votre coffre de clés a deux points de terminaison privés. Les requêtes qui en utilisent un fonctionnent correctement, mais les requêtes qui utilisent l’autre échouent.
- Vous disposez d’un autre abonnement, d’un coffre de clés ou d’un réseau virtuel qui utilise des liaisons privées. Vous souhaitez effectuer un nouveau déploiement similaire, mais vous ne parvenez pas à obtenir de liaisons privées pour l’utiliser.

### <a name="problems-not-covered-by-this-article"></a>Problèmes NON traités dans cet article

- Il y a un problème de connectivité intermittente. Dans un client donné, vous constatez que certaines requêtes fonctionnent et que d’autres non. *Les problèmes intermittents ne sont généralement pas dus à un problème dans la configuration des liaisons privées. Ils sont le signe d’une surcharge du réseau ou du client.*
- Vous utilisez un produit Azure qui prend en charge BYOK (Bring Your Own Key), CMK (clés gérées par le client) ou l’accès aux secrets stockés dans le coffre de clés. Lorsque vous activez le pare-feu dans les paramètres du coffre de clés, ce produit ne peut pas accéder à votre coffre de clés. *Consultez la documentation spécifique au produit. Assurez-vous qu’il prend explicitement en charge les coffres de clés avec le pare-feu activé. Contactez le support pour ce produit spécifique si nécessaire.*

### <a name="how-to-read-this-article"></a>Lecture de cet article

Si vous débutez avec les liaisons privées ou si vous évaluez un déploiement complexe, nous vous recommandons de lire l’intégralité de l’article. Sinon, n’hésitez pas à choisir la section la plus appropriée au problème que vous rencontrez.

Commençons.

## <a name="1-confirm-that-you-own-the-client-connection"></a>1. Confirmer que vous êtes propriétaire de la connexion cliente

### <a name="confirm-that-your-client-runs-at-the-virtual-network"></a>Confirmer que votre client s’exécute sur le réseau virtuel

Ce guide est conçu pour vous aider à résoudre les connexions au coffre de clés qui proviennent du code d’application. Les exemples sont des applications et des scripts qui s’exécutent sur des machines virtuelles Azure, des clusters Azure Service Fabric, Azure App Service, Azure Kubernetes Service (AKS) et d’autres similaires. Ce guide s’applique également aux accès effectués dans l’interface utilisateur web du portail Azure, où le navigateur accède directement à votre coffre de clés.

Par définition des liaisons privées, l’application, le script ou le portail doivent s’exécuter sur un ordinateur, un cluster ou un environnement connecté au réseau virtuel sur lequel la [ressource de point de terminaison privé](../../private-link/private-endpoint-overview.md) a été déployée.

Si l’application, le script ou le portail s’exécute sur un réseau arbitraire connecté à Internet, ce guide n’est PAS applicable et il est probable que des liaisons privées ne puissent pas être utilisées. Cette limitation s’applique également aux commandes exécutées dans Azure Cloud Shell, car celles-ci s’exécutent sur un ordinateur Azure distant fourni à la demande au lieu du navigateur de l’utilisateur.

### <a name="if-you-use-a-managed-solution-refer-to-specific-documentation"></a>Si vous utilisez une solution managée, reportez-vous à la documentation correspondante

Ce guide n’est PAS applicable aux solutions qui sont managées par Microsoft, où le coffre de clés est accessible par un produit Azure qui existe indépendamment du réseau virtuel client. Des exemples de scénarios de ce type sont Stockage Azure ou Azure SQL configuré pour le chiffrement au repos, Azure Event Hub chiffrant des données avec des clés fournies par le client, Azure Data Factory accédant aux informations d’identification du service stockées dans le coffre de clés, Azure Pipelines récupérant des secrets du coffre de clés et d’autres scénarios similaires. Dans ce cas, *vous devez vérifier si le produit prend en charge les coffres de clés avec le pare-feu activé*. Cette prise en charge s’effectue généralement à l’aide de la fonctionnalité de [services approuvés](overview-vnet-service-endpoints.md#trusted-services) du pare-feu Key Vault. De nombreux produits ne sont toutefois pas inclus dans la liste des services approuvés pour diverses raisons. Dans ce cas, obtenez un support spécifique au produit.

Un petit nombre de produits Azure prend en charge le concept d’*injection de réseau virtuel*. En termes simples, le produit ajoute un périphérique réseau au réseau virtuel du client, ce qui lui permet d’envoyer des requêtes comme s’il était déployé sur le réseau virtuel. [Azure Databricks](/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject) en est un exemple probant. Les produits comme celui-ci peuvent effectuer des requêtes auprès du coffre de clés à l’aide des liaisons privées, et ce guide de résolution des problèmes peut être utile.

## <a name="2-confirm-that-the-connection-is-approved-and-succeeded"></a>2. Confirmer que la connexion est approuvée et réussie

Les étapes suivantes permettent de confirmer que la connexion au point de terminaison privé est approuvée et réussie :

1. Ouvrez le portail Azure et ouvrez votre ressource de coffre de clés.
2. Dans le menu de gauche, sélectionnez **Mise en réseau**.
3. Cliquez sur l’onglet **Connexions de point de terminaison privé**. Cette opération affiche toutes les connexions de point de terminaison privé et leurs états respectifs. S’il n’existe aucune connexion, ou si la connexion de votre réseau virtuel est manquante, vous devez créer un nouveau point de terminaison privé. Ce sujet sera abordé ultérieurement.
4. Toujours dans **Connexions de point de terminaison privé**, recherchez celle que vous diagnostiquez et confirmez que « État de la connexion » est **Approuvé** et que « État de l’approvisionnement » est **Réussi**.
    - Si la connexion est à l’état « En attente », vous pouvez simplement l’approuver.
    - Si la connexion est à l’état « Rejeté », « Échec », « Erreur », « Déconnecté » ou un autre état, cela signifie qu’elle n’est pas effective et vous devez créer une nouvelle ressource de point de terminaison privé.

Il convient de supprimer les connexions inactives pour que tout reste clair.

## <a name="3-confirm-that-the-key-vault-firewall-is-properly-configured"></a>3. Confirmer que le pare-feu du coffre de clés est correctement configuré

>[!IMPORTANT]
> La modification des paramètres du pare-feu peut supprimer l’accès de clients légitimes qui n’utilisent pas de liaisons privées. Veillez à avoir pris connaissance des implications de chaque modification de la configuration du pare-feu.

Une notion importante est que la fonctionnalité de liaisons privées *donne* uniquement accès à votre coffre de clés dans un réseau virtuel fermé pour empêcher l’exfiltration de données. Elle ne *supprime* aucun accès existant. Pour bloquer efficacement les accès depuis l’Internet public, vous devez activer explicitement le pare-feu du coffre de clés :

1. Ouvrez le portail Azure et ouvrez votre ressource de coffre de clés.
2. Dans le menu de gauche, sélectionnez **Mise en réseau**.
3. Assurez-vous que l’onglet **Pare-feux et réseaux virtuels** est sélectionné en haut.
4. Assurez-vous que l’option **Point de terminaison privé et réseaux sélectionnés** est sélectionnée. Si **Tous les réseaux** est sélectionné, cela explique pourquoi des clients externes sont toujours en mesure d’accéder au coffre de clés.

Les considérations suivantes s’appliquent également aux paramètres du pare-feu :

- La fonctionnalité de liaisons privées ne requiert pas nécessairement de spécifier un « réseau virtuel » dans les paramètres du pare-feu du coffre de clés. Toutes les requêtes qui utilisent l’adresse IP privée du coffre de clés (voir la section suivante) doivent fonctionner, même si aucun réseau virtuel n’est spécifié dans les paramètres du pare-feu du coffre de clés.
- La fonctionnalité de liaisons privées ne requiert pas nécessairement de spécifier une adresse IP dans les paramètres du pare-feu du coffre de clés. Là encore, toutes les requêtes qui utilisent l’adresse IP privée du coffre de clés doivent fonctionner, même si aucune adresse IP n’a été spécifiée dans les paramètres du pare-feu.

Si vous utilisez des liaisons privées, les seules motivations pour spécifier un réseau virtuel ou une adresse IP dans le pare-feu de coffre de clés sont les suivantes :

- Vous disposez d’un système hybride dans lequel certains clients utilisent des liaisons privées, certains utilisent des points de terminaison de service et d’autres utilisent une adresse IP publique.
- Vous effectuez la transition vers des liaisons privées. Dans ce cas, une fois que vous avez confirmé que tous les clients utilisent des liaisons privées, vous devez supprimer les réseaux virtuels et les adresses IP des paramètres du pare-feu du coffre de clés.

## <a name="4-make-sure-the-key-vault-has-a-private-ip-address"></a>4. Vérifier que le coffre de clés a une adresse IP privée

### <a name="difference-between-private-and-public-ip-addresses"></a>Différence entre adresses IP privées et publiques

Une adresse IP privée se présente toujours dans l’un des formats suivants :

- 10.x.x.x : Exemples : `10.1.2.3`, `10.56.34.12`.
- 172.16.x.x à 172.32.x.x : Exemples : `172.20.1.1`, `172.31.67.89`.
- 192.168.x.x : Exemples : `192.168.0.1`, `192.168.100.7`

Certaines adresses IP et plages sont réservées :

- 224.x.x.x : Multidiffusion
- 255.255.255.255 : Diffusion
- 127.x.x.x : Bouclage
- 169.254.x.x : Lien-local
- 168.63.129.16 : DNS interne

Toutes les autres adresses IP sont publiques.

Lorsque vous parcourez le portail ou que vous exécutez une commande qui affiche l’adresse IP, assurez-vous de pouvoir déterminer si cette adresse IP est privée, publique ou réservée. Pour que les liaisons privées fonctionnent, le nom d’hôte du coffre de clés doit être résolu en une adresse IP privée appartenant à l’espace d’adressage du réseau virtuel.

### <a name="find-the-key-vault-private-ip-address-in-the-virtual-network"></a>Rechercher l’adresse IP privée du coffre de clés dans le réseau virtuel

Vous devez diagnostiquer la résolution du nom d’hôte. Pour cela, vous devez connaître l’adresse IP privée exacte de votre coffre de clés avec les liaisons privées activées. Pour trouver cette adresse, procédez comme suit :

1. Ouvrez le portail Azure et ouvrez votre ressource de coffre de clés.
2. Dans le menu de gauche, sélectionnez **Mise en réseau**.
3. Cliquez sur l’onglet **Connexions de point de terminaison privé**. Cette opération affiche toutes les connexions de point de terminaison privé et leurs états respectifs.
4. Recherchez celle que vous diagnostiquez et confirmez que « État de la connexion » est **Approuvé** et que « État de l’approvisionnement » est **Réussi**. Si vous ne le voyez pas, revenez aux sections précédentes de ce document.
5. Lorsque vous trouvez l’élément approprié, cliquez sur le lien dans la colonne **Point de terminaison privé** . Cette opération ouvre la ressource du point de terminaison privé.
6. La page Vue d’ensemble peut afficher une section appelée **Paramètres DNS personnalisés**. Vérifiez qu’une seule entrée correspond au nom d’hôte du coffre de clés. Cette entrée indique l’adresse IP privée du coffre de clés.
7. Vous pouvez également cliquer sur le lien situé au niveau de **Interface réseau** et vérifier que l’adresse IP privée est la même que celle affichée à l’étape précédente. L’interface réseau est un appareil virtuel qui représente le coffre de clés.

L’adresse IP est celle que les machines virtuelles et d’autres périphériques appareils *en cours d’exécution dans le même réseau virtuel* utiliseront pour se connecter au coffre de clés. Notez l’adresse IP ou conservez l’onglet du navigateur ouvert et ne le touchez pas pendant les recherches ultérieures.

>[!NOTE]
> Si votre coffre de clés comporte plusieurs points de terminaison privés, il a plusieurs adresses IP privées. Cela est utile uniquement si vous avez plusieurs réseaux virtuels qui accèdent au même coffre de clés, chacun par le biais de son propre point de terminaison privé (le point de terminaison privé appartient à un seul réseau virtuel). Veillez à diagnostiquer le problème du réseau virtuel approprié, puis sélectionnez la connexion de point de terminaison privé appropriée dans la procédure ci-dessus. En outre, ne créez **pas** plusieurs points de terminaison privés pour le même Key Vault dans le même réseau virtuel. Cela n’est pas nécessaire et est source de confusion.

## <a name="5-validate-the-dns-resolution"></a>5. Valider la résolution DNS

La résolution DNS est le processus de traduction du nom d’hôte du coffre de clés (exemple : `fabrikam.vault.azure.net`) en adresse IP (exemple : `10.1.2.3`). Les sous-sections suivantes montrent les résultats de résolution DNS attendus dans chaque scénario.

### <a name="key-vaults-without-private-link"></a>Coffres de clés sans liaison privée

Cette section est conçue à des fins d’apprentissage. Lorsque le coffre de clés n’a pas de connexion de point de terminaison privé à l’état Approuvé, la résolution du nom d’hôte donne un résultat similaire à celui-ci :

Windows :

```console
C:\> nslookup fabrikam.vault.azure.net
```

```output
Non-authoritative answer:
Address:  52.168.109.101
Aliases:  fabrikam.vault.azure.net
          data-prod-eus.vaultcore.azure.net
          data-prod-eus-region.vaultcore.azure.net
```

Linux :

```console
joe@MyUbuntu:~$ host fabrikam.vault.azure.net
```

```output
fabrikam.vault.azure.net is an alias for data-prod-eus.vaultcore.azure.net.
data-prod-eus.vaultcore.azure.net is an alias for data-prod-eus-region.vaultcore.azure.net.
data-prod-eus-region.vaultcore.azure.net has address 52.168.109.101
```

Vous pouvez voir que le nom est résolu en une adresse IP publique et qu’il n’y a pas d’alias `privatelink`. L’alias sera expliqué ultérieurement, ne vous en souciez pas pour le moment.

Le résultat ci-dessus est attendu quelle que soit la machine connectée au réseau virtuel ou qu’il s’agisse d’une machine arbitraire disposant d’une connexion Internet. Cela est dû au fait que le coffre de clés n’a pas de connexion de point de terminaison privé à l’état Approuvé. Le coffre de clés ne doit donc pas nécessairement prendre en charge les liaisons privées.

### <a name="key-vault-with-private-link-resolving-from-arbitrary-internet-machine"></a>Coffre de clés avec résolution de liaison privée à partir d’un ordinateur Internet arbitraire

Lorsque le coffre de clés a une ou plusieurs connexions de point de terminaison privé à l’état Approuvé et que vous résolvez le nom d’hôte à partir d’un ordinateur arbitraire connecté à Internet (un ordinateur qui *n’est pas* connecté au réseau virtuel sur lequel se trouve le point de terminaison privé), vous devez trouver ce qui suit :

Windows :

```console
C:\> nslookup fabrikam.vault.azure.net
```

```output
Non-authoritative answer:
Address:  52.168.109.101
Aliases:  fabrikam.vault.azure.net
          fabrikam.privatelink.vaultcore.azure.net
          data-prod-eus.vaultcore.azure.net
          data-prod-eus-region.vaultcore.azure.net
```

Linux :

```console
joe@MyUbuntu:~$ host fabrikam.vault.azure.net
```

```output
fabrikam.vault.azure.net is an alias for fabrikam.privatelink.vaultcore.azure.net.
fabrikam.privatelink.vaultcore.azure.net is an alias for data-prod-eus.vaultcore.azure.net.
data-prod-eus.vaultcore.azure.net is an alias for data-prod-eus-region.vaultcore.azure.net.
data-prod-eus-region.vaultcore.azure.net has address 52.168.109.101
```

La différence notable par rapport au scénario précédent est qu’il existe un nouvel alias avec la valeur `{vaultname}.privatelink.vaultcore.azure.net`. Cela signifie que le plan de données du coffre de clés est prêt à accepter les requêtes de liaisons privées.

Cela ne signifie pas que les requêtes exécutées à partir d’ordinateurs *en dehors* le réseau virtuel (comme celui que vous venez d’utiliser) utilisent des liaisons privées. Vous pouvez le constater car le nom d’hôte est toujours résolu en une adresse IP publique. Seules les ordinateurs *connectés au réseau virtuel* peuvent utiliser des liaisons privées. Plus d’informations seront fournies ultérieurement.

Si vous ne voyez pas l’alias `privatelink`, cela signifie que le coffre de clés n’a aucune connexion de point de terminaison privé à l’état `Approved`. Revenez à [cette section](#2-confirm-that-the-connection-is-approved-and-succeeded) avant de réessayer.

### <a name="key-vault-with-private-link-resolving-from-virtual-network"></a>Coffre de clés avec résolution de liaison privée à partir d’un réseau virtuel

Lorsque le coffre de clés a une ou plusieurs connexions de point de terminaison privé à l’état Approuvé et que vous résolvez le nom d’hôte à partir d’un ordinateur connecté au réseau virtuel sur lequel le point de terminaison privé a été créé, la réponse attendue est la suivante :

Windows :

```console
C:\> nslookup fabrikam.vault.azure.net
```

```output
Non-authoritative answer:
Address:  10.1.2.3
Aliases:  fabrikam.vault.azure.net
          fabrikam.privatelink.vaultcore.azure.net
```

Linux :

```console
joe@MyUbuntu:~$ host fabrikam.vault.azure.net
```

```output
fabrikam.vault.azure.net is an alias for fabrikam.privatelink.vaultcore.azure.net.
fabrikam.privatelink.vaultcore.azure.net has address 10.1.2.3
```

Deux différences notables sont à noter. Premièrement, le nom est résolu en une adresse IP privée. Il doit s’agir de l’adresse IP que nous avons trouvée dans la [section correspondante](#find-the-key-vault-private-ip-address-in-the-virtual-network) de cet article. Deuxièmement, il n’y a pas d’autres alias après `privatelink`. Cela se produit parce que les serveurs DNS du réseau virtuel *interceptent* la chaîne d’alias et retournent l’adresse IP privée directement à partir du nom `fabrikam.privatelink.vaultcore.azure.net`. Cette entrée est en réalité un enregistrement `A` dans une zone DNS privée. Plus d’informations seront fournies ultérieurement.

>[!NOTE]
> Le résultat ci-dessus se produit uniquement sur une machine virtuelle connectée au réseau virtuel sur lequel le point de terminaison privé a été créé. Si aucune machine virtuelle n’est déployée sur le réseau virtuel qui contient le point de terminaison privé, déployez-en une et connectez-vous à celle-ci à distance, puis exécutez la commande `nslookup` (Windows) ou la commande `host` (Linux) ci-dessus.

Si vous exécutez ces commandes sur une machine virtuelle connectée au réseau virtuel sur lequel le point de terminaison privé a été créé et qu’elles ne retournent **pas** l’adresse IP privée du coffre de clés, la section suivante peut aider à résoudre le problème.

## <a name="6-validate-the-private-dns-zone"></a>6. Valider la zone DNS privée

Si la résolution DNS ne fonctionne pas comme décrit dans la section précédente, il peut y avoir un problème lié à votre zone DNS privée et cette section peut vous aider. Si la résolution DNS affiche l’adresse IP privée du coffre de clés correcte, votre zone DNS privée est probablement correcte. Vous pouvez ignorer toute cette section.

### <a name="confirm-that-the-required-private-dns-zone-resource-exists"></a>Confirmer l’existence de la ressource de zone DNS privée requise

Votre abonnement Azure doit avoir une ressource de [zone DNS privée](../../dns/private-dns-privatednszone.md) avec ce nom exact :

`privatelink.vaultcore.azure.net`

Vous pouvez vérifier la présence de cette ressource en accédant à la page de l’abonnement dans le portail et en sélectionnant « Ressources » dans le menu de gauche. Le nom de la ressource doit être `privatelink.vaultcore.azure.net`, et le type de ressource doit être **Zone DNS privée**.

Cette ressource est normalement créée automatiquement lorsque vous créez un point de terminaison privé à l’aide d’une procédure courante. Toutefois, dans certains cas, cette ressource n’est pas créée automatiquement et vous devez le faire manuellement. Cette ressource peut également avoir été supprimée par accident.

Si vous ne disposez pas de cette ressource, créez une nouvelle ressource de zone DNS privée dans votre abonnement. N’oubliez pas que le nom doit être exactement `privatelink.vaultcore.azure.net`, sans espaces ni points supplémentaires. Si vous spécifiez un nom incorrect, la résolution de noms expliquée dans cet article ne fonctionnera pas. Pour plus d’informations sur la création de cette ressource, consultez [Créer une zone Azure DNS privée avec le portail Azure](../../dns/private-dns-getstarted-portal.md). Si vous suivez cette page, vous pouvez ignorer la création de réseau virtuel car, à ce stade, vous en avez déjà un. Vous pouvez également ignorer les procédures de validation avec les machines virtuelles.

### <a name="confirm-that-the-private-dns-zone-is-linked-to-the-virtual-network"></a>Confirmer que la zone DNS privée est liée au réseau virtuel

Il ne suffit pas d’avoir une zone DNS privée. Elle doit également être liée au réseau virtuel qui contient le point de terminaison privé. Si la zone DNS privée n’est pas liée au réseau virtuel approprié, toute résolution DNS à partir de ce réseau virtuel ignorera la zone DNS privée.

Ouvrez la ressource de zone DNS privée et cliquez sur l’option des **liens du réseau virtuel** dans le menu de gauche. Cette opération affiche une liste de liens, chacun avec le nom d’un réseau virtuel dans votre abonnement. Le réseau virtuel qui contient la ressource de point de terminaison privé doit être répertorié ici. Si ce n’est pas le cas, ajoutez-le. Pour connaître la procédure détaillée, consultez [Lier le réseau virtuel](../../dns/private-dns-getstarted-portal.md#link-the-virtual-network). Vous pouvez laisser l’option « Activer l’inscription automatique » décochée, car cette fonctionnalité n’est pas liée aux liaisons privées.

Une fois que la zone DNS privée est liée au réseau virtuel, les requêtes DNS provenant du réseau virtuel rechercheront des noms dans la zone DNS privée. Cela est requis pour une résolution d’adresse correcte effectuée sur des machines virtuelles connectées au réseau virtuel sur lequel le point de terminaison privé a été créé.

>[!NOTE]
> Si vous venez d’enregistrer le lien, son application peut prendre un certain temps, même si le portail indique que l’opération est terminée. Vous devrez peut-être également redémarrer la machine virtuelle que vous utilisez pour tester la résolution DNS.

### <a name="confirm-that-the-private-dns-zone-contains-the-right-a-record"></a>Confirmer que la zone DNS privée contient l’enregistrement A approprié

À l’aide du portail, ouvrez la zone DNS privée nommée `privatelink.vaultcore.azure.net`. La page Vue d’ensemble affiche tous les enregistrements. Par défaut, il y aura un enregistrement avec le nom `@` et le type `SOA`, c’est-à-dire le début de l’autorité. N’y touchez pas.

Pour que la résolution de noms du coffre de clés fonctionne, il doit exister un enregistrement `A` avec le nom du coffre simple sans suffixe ni points. Par exemple, si le nom d’hôte est `fabrikam.vault.azure.net`, il doit exister un enregistrement `A` portant le nom `fabrikam`, sans suffixe ni points.

De même, la valeur de l’enregistrement `A` (adresse IP) doit être l’[adresse IP privée du coffre de clés](#find-the-key-vault-private-ip-address-in-the-virtual-network). Si vous trouvez l’enregistrement `A` mais qu’il contient une adresse IP incorrecte, vous devez supprimer l’adresse IP incorrecte et en ajouter une nouvelle. Il est recommandé de supprimer la totalité de l’enregistrement `A` et d’en ajouter un nouveau.

>[!NOTE]
> Lorsque vous supprimez ou modifiez un enregistrement `A`, l’ordinateur peut toujours résoudre l’ancienne adresse IP car la valeur TTL (Durée de vie) n’a peut-être pas encore expiré. Il est recommandé de toujours spécifier une valeur de durée de vie inférieure ou égale à 60 secondes (une minute) et pas plus de 600 secondes (10 minutes). Si vous spécifiez une valeur trop importante, la récupération de vos clients peut prendre trop de temps.

### <a name="dns-resolution-for-more-than-one-virtual-network"></a>Résolution DNS pour plusieurs réseaux virtuels

S’il existe plusieurs réseaux virtuels et que chacun possède sa propre ressource de point de terminaison privé référençant le même coffre de clés, le nom d’hôte du coffre de clés doit être résolu en une adresse IP privée différente en fonction du réseau. Cela signifie que plusieurs zones DNS privées sont également nécessaires, chacune liée à un réseau virtuel différent et utilisant une adresse IP différente dans l’enregistrement `A`.

Dans des scénarios plus avancés, le Peering est peut-être activé dans les réseaux virtuels. Dans ce cas, un seul réseau virtuel a besoin de la ressource de point de terminaison privé, même si deux peuvent devoir être liés à la ressource de zone DNS privée. Ce scénario n’est pas directement traité dans ce document.

### <a name="understand-that-you-have-control-over-dns-resolution"></a>Comprendre que vous contrôlez la résolution DNS

Comme expliqué dans la [section précédente](#key-vault-with-private-link-resolving-from-arbitrary-internet-machine), un coffre de clés avec des liaisons privées a l’alias `{vaultname}.privatelink.vaultcore.azure.net` dans son *inscription publique*. Le serveur DNS utilisé par le réseau virtuel utilise l’inscription publique, mais vérifie chaque alias pour une inscription *privée* et, s’il en existe un, il cesse de suivre les alias définis lors de l’inscription publique.

Cette logique signifie que si le réseau virtuel est lié à une zone DNS privée nommée `privatelink.vaultcore.azure.net` et que l’inscription DNS publique du coffre de clés a l’alias `fabrikam.privatelink.vaultcore.azure.net` (notez que le suffixe du nom d’hôte du coffre de clés correspond exactement au nom de la zone DNS privée), la requête DNS recherche un enregistrement `A` nommé `fabrikam` *dans la zone DNS privée*. Si l’enregistrement `A` est trouvé, son adresse IP est retournée dans la requête DNS et aucune recherche supplémentaire n’est effectuée lors de l’inscription DNS publique.

Comme vous pouvez le constater, vous contrôlez la résolution de noms. Les raisonnements de cette conception sont les suivants :

- Vous pouvez avoir un scénario complexe qui implique des serveurs DNS personnalisés et une intégration aux réseaux locaux. Dans ce cas, vous devez contrôler comment les noms sont traduits en adresses IP.
- Vous devrez peut-être accéder à un coffre de clés sans liaisons privées. Dans ce cas, la résolution du nom d’hôte à partir du réseau virtuel doit retourner l’adresse IP publique, ce qui se produit parce que les coffres de clés sans liaisons privées n’ont pas l’alias `privatelink` dans l’inscription du nom.

## <a name="7-validate-that-requests-to-key-vault-use-private-link"></a>7. Vérifier que les requêtes envoyées au coffre de clés utilisent une liaison privée

### <a name="query-the-healthstatus-endpoint-of-the-key-vault"></a>Interroger le point de terminaison `/healthstatus` du coffre de clés

Votre coffre de clés fournit le point de terminaison `/healthstatus`, qui peut être utilisé pour les diagnostics. Les en-têtes de réponse incluent l’adresse IP d’origine, telle qu’elle est affichée par le service de coffre de clés. Vous pouvez appeler ce point de terminaison à l’aide de la commande suivante (**n’oubliez pas d’utiliser le nom d’hôte de votre coffre de clés**) :

Windows (PowerShell) :

```powershell
PS C:\> $(Invoke-WebRequest -UseBasicParsing -Uri https://fabrikam.vault.azure.net/healthstatus).Headers
```

```output
Key                           Value
---                           -----
Pragma                        no-cache
x-ms-request-id               3729ddde-eb6d-4060-af2b-aac08661d2ec
x-ms-keyvault-service-version 1.2.27.0
x-ms-keyvault-network-info    addr=10.4.5.6;act_addr_fam=InterNetworkV6;
Strict-Transport-Security     max-age=31536000;includeSubDomains
Content-Length                4
Cache-Control                 no-cache
Content-Type                  application/json; charset=utf-8
```

Linux ou une version récente de Windows 10 qui comprend `curl` :

```console
joe@MyUbuntu:~$ curl -i https://fabrikam.vault.azure.net/healthstatus
```

```output
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: application/json; charset=utf-8
x-ms-request-id: 6c090c46-0a1c-48ab-b740-3442ce17e75e
x-ms-keyvault-service-version: 1.2.27.0
x-ms-keyvault-network-info: addr=10.4.5.6;act_addr_fam=InterNetworkV6;
Strict-Transport-Security: max-age=31536000;includeSubDomains
Content-Length: 4
```

Si vous n’obtenez pas une sortie similaire à celle-ci, ou si vous obtenez une erreur réseau, cela signifie que votre coffre de clés n’est pas accessible via le nom d’hôte que vous avez spécifié (`fabrikam.vault.azure.net` dans l’exemple). Soit le nom d’hôte n’est pas résolu dans l’adresse IP correcte, soit vous rencontrez un problème de connectivité au niveau de la couche de transport. Cela peut être dû à des problèmes de routage, à des suppressions de packages et à d’autres raisons. Vous devez approfondir vos investigations.

La réponse doit inclure un en-tête `x-ms-keyvault-network-info` :

```console
x-ms-keyvault-network-info: addr=10.4.5.6;act_addr_fam=InterNetworkV6;
```

Le champ `addr` dans l’en-tête `x-ms-keyvault-network-info` indique l’adresse IP de l’origine de la requête. Cette adresse IP peut être l’une des suivantes :

- L’adresse IP privée de l’ordinateur formulant la requête. Cela indique que la requête utilise des liaisons privées ou des points de terminaison de service. Il s’agit du résultat attendus pour les liaisons privées.
- Une autre adresse IP privée ne provenant *pas* de l’ordinateur formulant la requête. Cela indique qu’un routage personnalisé s’applique. Cela indique toujours que la requête utilise des liaisons privées ou des points de terminaison de service.
- Une adresse IP publique. Cela indique que la requête est acheminée vers Internet via une passerelle (NAT). Cela indique que la requête n’utilise PAS de liaisons privées et qu’un problème doit être résolu. Les raisons les plus courantes sont que 1) le point de terminaison privé n’est pas à l’état approuvé et réussi ; et que 2) le nom d’hôte n’est pas résolu dans l’adresse IP privée du coffre de clés. Cet article comprend des actions de résolution des problèmes pour les deux cas.

>[!NOTE]
> Si la requête vers `/healthstatus` fonctionne mais que l’en-tête `x-ms-keyvault-network-info` est manquant, cela signifie que le point de terminaison n’est probablement pas desservi par le coffre de clés. Étant donné que les commandes ci-dessus valident également un certificat HTTPS, l’en-tête manquant peut être le signe d’une falsification.

### <a name="query-the-key-vault-ip-address-directly"></a>Interroger directement l’adresse IP du coffre de clés

>[!IMPORTANT]
> L’accès au coffre de clés sans validation du certificat HTTPS peut être dangereux et ne peut être utilisé qu’à des fins d’apprentissage. Le code de production ne doit JAMAIS accéder au coffre de clés sans validation côté client. Même si diagnostiquez simplement des problèmes, vous pouvez être victime de tentatives de falsification qui n’indiqueront pas si vous désactivez fréquemment la validation du certificat HTTPS dans vos requêtes auprès du coffre de clés.

Si vous avez installé une version récente de PowerShell, vous pouvez utiliser `-SkipCertificateCheck` pour ignorer les vérifications de certificat HTTPS, puis cibler directement l’[adresse IP du coffre de clés](#find-the-key-vault-private-ip-address-in-the-virtual-network) :

```powershell
PS C:\> $(Invoke-WebRequest -SkipCertificateCheck -Uri https://10.1.2.3/healthstatus).Headers
```

Si vous utilisez `curl`, vous pouvez faire de même avec l’argument `-k` :

```console
joe@MyUbuntu:~$ curl -i -k https://10.1.2.3/healthstatus
```

Les réponses doivent être identiques à celles de la section précédente, ce qui signifie qu’elles doivent inclure l’en-tête `x-ms-keyvault-network-info` avec la même valeur. Le point de terminaison `/healthstatus` ne se soucie pas de savoir si vous utilisez le nom d’hôte ou l’adresse IP du coffre de clés.

Si vous constatez que `x-ms-keyvault-network-info` retourne une valeur pour la requête utilisant le nom d’hôte du coffre de clés et une autre valeur pour la requête utilisant l’adresse IP, cela signifie que chaque requête cible un point de terminaison différent. Reportez-vous à l’explication du champ `addr` de `x-ms-keyvault-network-info` dans la section précédente pour déterminer quel cas est incorrect et doit être corrigé.

## <a name="8-other-changes-and-customizations-that-cause-impact"></a>8. Autres modifications et personnalisations ayant un impact

Les éléments suivants sont des actions d’investigation non exhaustives. Ils vous indiquent où rechercher d’autres problèmes, mais vous devez avoir une connaissance approfondie du réseau pour résoudre les problèmes dans ces scénarios.

### <a name="diagnose-custom-dns-servers-at-virtual-network"></a>Diagnostiquer des DNS personnalisés sur un réseau virtuel

Dans le portail, ouvrez la ressource de réseau virtuel. Dans le menu de gauche, ouvrez **Serveurs DNS**. Si vous utilisez « Personnalisé », la résolution DNS peut ne pas être la même que celle décrite dans ce document. Vous devez diagnostiquer la manière dont vos serveurs DNS résolvent le nom d’hôte du coffre de clés.

Si vous utilisez les serveurs DNS fournis par Azure par défaut, ce document entier s’applique.

### <a name="diagnose-hosts-overriding-or-custom-dns-servers-at-virtual-machine"></a>Diagnostiquer le remplacement d’hôtes ou les serveurs DNS personnalisés sur la machine virtuelle

De nombreux systèmes d’exploitation autorisent la définition d’une adresse IP fixe explicite par nom d’hôte, généralement en modifiant le fichier `hosts`. Ils peuvent également autoriser le remplacement des serveurs DNS. Si vous utilisez l’un de ces scénarios, passez aux options de diagnostic spécifiques au système.

### <a name="promiscuous-proxies-fiddler-etc"></a>Proxies de proximité (Fiddler, etc.)

Sauf mention contraire, les options de diagnostic dans cet article ne fonctionnent que si aucun proxy de proximité n’est présent dans l’environnement. Bien que ces proxies soient souvent installés exclusivement dans l’ordinateur faisant l’objet d’un diagnostic (Fiddler en est l’exemple le plus courant), les administrateurs avancés peuvent remplacer les autorités de certification (CA) racine et installer un proxy de proximité dans les périphériques de passerelle qui servent plusieurs ordinateurs du réseau. Ces proxies peuvent avoir un impact considérable sur la sécurité et la fiabilité. Microsoft ne prend pas en charge les configurations qui utilisent ces produits.

### <a name="other-things-that-may-affect-connectivity"></a>Autres éléments susceptibles d’affecter la connectivité

Il s’agit d’une liste non exhaustive d’éléments qui peuvent être présents dans des scénarios avancés ou complexes :

- Les paramètres du pare-feu, qu’il s’agisse du Pare-feu Azure connecté au réseau virtuel ou une solution de pare-feu personnalisée déployée dans le réseau virtuel ou sur l’ordinateur.
- Le Peering de réseau, qui peut avoir un impact sur les serveurs DNS utilisés et sur la façon dont le trafic est acheminé.
- Des solutions de passerelle (NAT) personnalisées, qui peuvent avoir un impact sur la façon dont le trafic est acheminé, notamment le trafic provenant de requêtes DNS.