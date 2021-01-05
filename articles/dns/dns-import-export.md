---
title: Importer et exporter un fichier de zone de domaine - Azure CLI
titleSuffix: Azure DNS
description: Apprenez à importer et exporter un fichier de zone DNS vers Azure DNS à l’aide de l’interface de ligne de commande Azure
services: dns
author: rohinkoul
ms.service: dns
ms.date: 7/30/2020
ms.author: rohink
ms.topic: how-to
ms.openlocfilehash: e2b998432f6c4417da0242d86347ed43acb5071a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968228"
---
# <a name="import-and-export-a-dns-zone-file-using-the-azure-cli"></a>Importer et exporter un fichier de zone DNS en utilisant l’interface CLI Azure

Cet article vous guide dans l’importation et l’exportation de fichiers de zone DNS pour Azure DNS à l’aide d’Azure CLI.

## <a name="introduction-to-dns-zone-migration"></a>Introduction à la migration de zone DNS

Un fichier de zone DNS est un fichier texte qui contient les détails de tous les enregistrements DNS contenus dans la zone. Il suit un format standard, ce qui permet le transfert d’enregistrements DNS entre différents systèmes DNS. L’utilisation d’un fichier de zone offre un moyen rapide, fiable et pratique de transférer une zone DNS vers ou depuis Azure DNS.

Azure DNS prend en charge l’importation et l’exportation de fichiers de zone à l’aide de l’interface de ligne de commande (CLI). L’importation de fichier de zone n’est actuellement **pas** prise en charge via Azure PowerShell ou le portail Azure.

La CLI Azure est un outil de ligne de commande multiplateforme permettant de gérer des services Azure. Elles est disponible pour les plateformes Windows, Mac et Linux sur la [page des téléchargements d’Azure](https://azure.microsoft.com/downloads/). La prise en charge multiplateforme est importante pour l’importation et l’exportation de fichiers de zone car le logiciel serveur de noms le plus courant, [BIND](https://www.isc.org/downloads/bind/), s’exécute en général sur Linux.

## <a name="obtain-your-existing-dns-zone-file"></a>Comment obtenir votre fichier de zone DNS existant

Avant d’importer un fichier de zone DNS dans Azure DNS, vous devez obtenir une copie du fichier de zone. La source de ce fichier dépend de l’emplacement où la zone DNS est actuellement hébergée.

* Si votre zone DNS est hébergée par un service tiers (comme un bureau d’enregistrement de domaines, un hébergeur DNS dédié ou un autre fournisseur de cloud), ce service doit fournir la possibilité de télécharger le fichier de zone DNS.
* Si votre zone DNS est hébergée sur le DNS Windows, le dossier par défaut pour les fichiers de zone est **%systemroot%\system32\dns**. Le chemin complet vers chaque fichier de zone s’affiche également sous l’onglet **Général** de la console DNS.
* Si votre zone DNS est hébergée via BIND, l’emplacement du fichier de zone pour chaque zone est spécifié dans le fichier de configuration BIND, **named.conf**.

## <a name="import-a-dns-zone-file-into-azure-dns"></a>Importation d’un fichier de zone DNS dans Azure DNS

L’importation d’un fichier de zone crée une nouvelle zone dans Azure DNS si celle-ci n’existe pas. Si la zone existe déjà, les jeux d’enregistrements du fichier de zone doivent être fusionnés avec les jeux d’enregistrements existants.

### <a name="merge-behavior"></a>Comportement de la fusion

* Par défaut, les nouveaux jeux d’enregistrements sont fusionnés avec les jeux d’enregistrements existants. Les enregistrements identiques dans un jeu d’enregistrements fusionné sont dédupliqués.
* Lorsque les jeux d’enregistrements sont fusionnés, la TTL des jeux d’enregistrements existants est utilisée.
* Les paramètres SOA (Start of Authority) (excepté `host`) sont toujours pris dans le fichier de zone importé. De même, pour le jeu d’enregistrements du serveur de noms à l’extrémité de la zone, la TTL est toujours dérivée du fichier de zone importé.
* Un enregistrement CNAME importé ne remplace pas un enregistrement CNAME existant portant le même nom.  
* En cas de conflit entre un enregistrement CNAME et un autre enregistrement du même nom ayant un type différent (qu’il soit nouveau ou existant), l’enregistrement existant est conservé. 

### <a name="additional-information-about-importing"></a>Informations supplémentaires sur l’importation

Les remarques suivantes fournissent des informations techniques supplémentaires concernant le processus d’importation de zones.

* La directive `$TTL` , facultative, est prise en charge. Quand aucune directive `$TTL` n’est spécifiée, les enregistrements sans durée de vie explicite sont importés avec une durée de vie de 3 600 secondes par défaut. Lorsque deux enregistrements du même jeu d’enregistrements spécifient des TTL différentes, la moindre valeur est utilisée.
* La directive `$ORIGIN` , facultative, est prise en charge. Quand aucune directive `$ORIGIN` n’est définie, la valeur utilisée par défaut est le nom de zone tel qu’il est spécifié sur la ligne de commande (avec le point final).
* Les directives `$INCLUDE` et `$GENERATE` ne sont pas prises en charge.
* Les types d’enregistrements pris en charge sont les suivants : A, AAAA, CAA, CNAME, MX, NS, SOA, SRV et TXT.
* L’enregistrement SOA est créé automatiquement par Azure DNS lors de la création d’une zone. Quand vous importez un fichier de zone, tous les paramètres SOA sont pris dans le fichier de zone, *sauf* le paramètre `host`. Ce paramètre utilise la valeur fournie par Azure DNS. Ce paramètre doit en effet faire référence au serveur de noms principal fourni par Azure DNS.
* Le jeu d’enregistrements du serveur de noms, à l’extrémité de la zone, est créé automatiquement par Azure DNS au moment de la création de la zone. Seule la durée de vie de ce jeu d’enregistrements est importée. Ces enregistrements contiennent les noms de serveurs de nom fournis par Azure DNS. Les données d’enregistrement ne sont pas remplacées par les valeurs contenues dans le fichier de zone importé.
* Pour la version préliminaire publique, Azure DNS prend uniquement en charge les enregistrements à chaîne unique. Les enregistrements TXT MultiString sont concaténés et tronqués à 255 caractères.

### <a name="cli-format-and-values"></a>Format et valeurs de l’interface CLI

Le format de la commande de l’interface CLI Azure pour importer une zone DNS est :

```azurecli
az network dns zone import -g <resource group> -n <zone name> -f <zone file name>
```

Valeurs :

* `<resource group>` est le nom du groupe de ressources correspondant à la zone dans Azure DNS.
* `<zone name>` est le nom de la zone.
* `<zone file name>` est le chemin/nom du fichier de zone à importer.

S’il n’existe aucune zone de ce nom dans le groupe de ressources, cette zone est automatiquement créée. Si la zone existe déjà, les jeux d’enregistrements importés sont fusionnés avec les jeux d’enregistrements existants. 

### <a name="step-1-import-a-zone-file"></a>Étape 1. Importer un fichier de zone

Pour importer un fichier de zone pour la zone **contoso.com**.

1. Si ce n’est déjà fait, vous devez créer un groupe de ressources Azure Resource Manager.

    ```azurecli
    az group create --resource-group myresourcegroup -l westeurope
    ```

2. Pour importer la zone **contoso.com** à partir du fichier **contoso.com.txt** dans une nouvelle zone DNS du groupe de ressources **myresourcegroup**, vous exécutez la commande `az network dns zone import`.<BR>Cette commande charge le fichier de zone et l’analyse. Cette commande exécute une série de commandes sur le service Azure DNS, afin de créer la zone et tous les jeux d’enregistrements associés. Elle rend compte de la progression dans la fenêtre de console et signale les éventuels avertissements ou erreurs. Étant donné que les jeux d’enregistrements sont créés en série, l’importation d’un fichier de zone volumineux peut prendre quelques minutes.

    ```azurecli
    az network dns zone import -g myresourcegroup -n contoso.com -f contoso.com.txt
    ```

### <a name="step-2-verify-the-zone"></a>Étape 2. Vérifier la zone

Pour vérifier la zone DNS après avoir importé le fichier, vous pouvez utiliser une des méthodes suivantes :

* Vous pouvez répertorier les enregistrements à l’aide de la commande Azure CLI suivante :

    ```azurecli
    az network dns record-set list -g myresourcegroup -z contoso.com
    ```

* Vous pouvez répertorier les enregistrements à l'aide de la commande Azure CLI `az network dns record-set ns list`.
* Vous pouvez également utiliser `nslookup` pour vérifier la résolution des noms pour les enregistrements. Puisque la zone n’est pas encore déléguée, vous devez spécifier explicitement les serveurs de noms DNS Azure appropriés. L’exemple suivant montre comment récupérer les noms du serveur de noms assignés à la zone. Il montre également comment interroger l’enregistrement « www » à l’aide de `nslookup`.

    ```azurecli
    az network dns record-set ns list -g myresourcegroup -z contoso.com  --output json 
    ```

    ```json
    [
      {
       .......
       "name": "@",
        "nsRecords": [
          {
            "additionalProperties": {},
            "nsdname": "ns1-03.azure-dns.com."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns2-03.azure-dns.net."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns3-03.azure-dns.org."
          },
          {
            "additionalProperties": {},
            "nsdname": "ns4-03.azure-dns.info."
          }
        ],
        "resourceGroup": "myresourcegroup",
        "ttl": 86400,
        "type": "Microsoft.Network/dnszones/NS"
      }
    ]
    ```

    ```cmd
    nslookup www.contoso.com ns1-03.azure-dns.com

        Server: ns1-01.azure-dns.com
        Address:  40.90.4.1

        Name:www.contoso.com
        Addresses:  134.170.185.46
        134.170.188.221
    ```

### <a name="step-3-update-dns-delegation"></a>Étape 3. Mettre à jour la délégation DNS

Après avoir vérifié que la zone a été importée correctement, vous devez mettre à jour la délégation DNS afin qu’elle pointe vers les serveurs de noms Azure DNS. Pour plus d’informations, consultez l’article [Mettre à jour la délégation DNS](dns-domain-delegation.md).

## <a name="export-a-dns-zone-file-from-azure-dns"></a>Exportation d’un fichier de zone DNS à partir d’Azure DNS

Le format de la commande de l’interface de ligne de commande Azure pour exporter une zone DNS est :

```azurecli
az network dns zone export -g <resource group> -n <zone name> -f <zone file name>
```

Valeurs :

* `<resource group>` est le nom du groupe de ressources correspondant à la zone dans Azure DNS.
* `<zone name>` est le nom de la zone.
* `<zone file name>` est le chemin d’accès/nom du fichier de zone à exporter.

Pour importer une zone, vous devez d’abord vous connecter, sélectionner votre abonnement, puis configurer l’interface de ligne de commande Azure pour utiliser le mode Azure Resource Manager.

### <a name="to-export-a-zone-file"></a>Pour exporter un fichier de zone

Pour exporter la zone Azure DNS existante **contoso.com** du groupe de ressources **myresourcegroup** vers le fichier **contoso.com.txt** (dans le dossier actuel), exécutez `azure network dns zone export`. Cette commande appelle le service Azure DNS pour énumérer les jeux d’enregistrements dans la zone et exporter les résultats dans un fichier de zone compatible BIND.

```azurecli
az network dns zone export -g myresourcegroup -n contoso.com -f contoso.com.txt
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [gérer des jeux d’enregistrements et des enregistrements](./dns-getstarted-cli.md) dans votre zone DNS.

* Découvrez comment [déléguer votre domaine à Azure DNS](dns-domain-delegation.md).