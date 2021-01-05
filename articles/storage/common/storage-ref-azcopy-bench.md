---
title: azcopy bench | Microsoft Docs
description: Cet article fournit des informations de référence sur la commande azcopy bench.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: c1028d0a4a458746c08fd6fa4f16aa952d9962a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87282005"
---
# <a name="azcopy-benchmark"></a>référence azcopy

Exécute un test d’évaluation des performances en chargeant ou téléchargeant les données de test vers ou depuis une destination spécifiée. Pour les chargements, les données de test sont générées automatiquement.

La commande benchmark exécute le même processus que « copy », à ceci près que : 

  - Au lieu de demander des paramètres source et de destination, benchmark n’en prend qu’un. Il s’agit du conteneur d’objets blob, du partage Azure Files ou du système de fichiers Azure Data Lake Storage Gen2 à partir duquel vous souhaitez charger ou télécharger.

  - Le paramètre « mode » indique si AzCopy doit tester les chargements vers ou téléchargements à partir de la cible donnée. Les valeurs valides sont « Upload » et « Download ». La valeur par défaut est « Upload ».

  - Pour les tests d’évaluation de chargement, la charge utile est décrite par les paramètres de ligne de commande qui contrôlent le nombre de fichiers générés automatiquement et l’importance de leur taille. Le processus de génération s’effectue entièrement en mémoire. Le disque n’est pas utilisé.

  - Pour les téléchargements, la charge utile est constituée par des fichiers qui existent déjà à la source. (Voir l’exemple ci-dessous illustrant la façon de générer des fichiers de test, si nécessaire).
  
  - Seuls quelques-uns des paramètres facultatifs qui sont disponibles pour la commande copy sont pris en charge.
  
  - Des diagnostics supplémentaires sont mesurés et signalés.
  
  - Pour les chargements, le comportement par défaut consiste à supprimer les données transférées à la fin de la série de tests.  Pour les téléchargements, les données ne sont jamais enregistrées localement.

Le mode « test d’évaluation » se règle automatiquement en fonction du nombre de connexions TCP parallèles qui fournit le débit maximal. Il affiche ce nombre à la fin. Pour empêcher le réglage automatique, affectez un nombre spécifique de connexions à la variable d’environnement AZCOPY_CONCURRENCY_VALUE. 

Tous les types d’authentification habituels sont pris en charge. Toutefois, l’approche la plus pratique pour le test d’évaluation de chargement vise généralement à créer un conteneur vide avec un jeton SAS, et à utiliser l’authentification SAS. (Le mode de téléchargement demande la présence d’un ensemble de données de test dans le conteneur cible.)

## <a name="examples"></a>Exemples

```azcopy
azcopy benchmark [destination] [flags]
```

Exécuter un test d’évaluation avec les paramètres par défaut (adapté aux réseaux d’évaluation jusqu’à 1 Gbit/s) :’

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>"
```
Exécuter un test d’évaluation qui charge 100 fichiers, chacun d’une taille de 2 Gio : (adapté à l’évaluation sur un réseau rapide, par exemple 10 Gbits/s) :’

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>"--file-count 100 --size-per-file 2G
```
Exécutez un test de référence, mais utiliser 50 000 fichiers, chacun d’une taille de 8 Mio, et calculer leurs hachages MD5 (de la même façon qu’effectué par l’indicateur `--put-md5` dans la commande copy). L’objectif de `--put-md5` lors des tests d’évaluation est de tester si le calcul MD5 affecte le débit pour le nombre de fichiers et la taille sélectionnés :

```azcopy
azcopy bench --mode='Upload' "https://[account].blob.core.windows.net/[container]?<SAS>" --file-count 50000 --size-per-file 8M --put-md5
```

Exécuter un test d’évaluation qui télécharge des fichiers existants à partir d’une cible

```azcopy
azcopy bench --mode='Download' "https://[account].blob.core.windows.net/[container]?<SAS?"
```

Exécuter un chargement qui ne supprime pas les fichiers transférés. (Ces fichiers peuvent ensuite servir de charge utile pour un test de téléchargement)

```azcopy
azcopy bench "https://[account].blob.core.windows.net/[container]?<SAS>" --file-count 100 --delete-test-data=false
```

## <a name="options"></a>Options

**--blob-type** string  Définit le type d’objet blob au niveau de la destination. Permet d’effectuer un test d’évaluation de différents types d’objets blob. Identique au paramètre du même nom dans la commande copy (par défaut : « Detect »).

**--block-size-mb** float  Utiliser cette taille de bloc (spécifiée en Mio). La valeur par défaut est calculée automatiquement en fonction de la taille du fichier. Les fractions décimales sont autorisées, par exemple 0,25. Identique au paramètre du même nom dans la commande copy.

**--check-length**  Vérifiez la longueur d’un fichier sur la destination après le transfert. En cas d’incompatibilité entre la source et la destination, le transfert est marqué comme ayant échoué. (par défaut : true)

**--delete-test-data**  Si la valeur est true, les données du test d’évaluation sont supprimées à la fin de l’exécution du test.  Affectez-lui la valeur false si vous souhaitez conserver les données dans la destination, par exemple afin de les utiliser pour des tests manuels en dehors du mode « test d’évaluation » (par défaut : true).

**--file-count** uint.  Nombre de fichiers de données générés automatiquement à utiliser (par défaut : 100).

**--help**  Aide pour la commande bench

**--log-level** string Définit le niveau de détail pour le fichier journal. Niveaux disponibles : INFO (toutes les requêtes/réponses), WARNING (réponses lentes), ERROR (uniquement les échecs de requêtes) et NONE (aucun journal de sortie) (par défaut : « INFO »)

**--mode** string  Définit si Azcopy doit tester les chargements ou les téléchargements à partir de cette cible. Les valeurs valides sont « upload » et « download ». L’option par défaut est « upload ». (par défaut : « upload »)

**--number-of-folders** uint  Si la valeur est supérieure à 0, créez des dossiers pour diviser les données.

**--put-md5**  Crée un hachage MD5 de chaque fichier, puis enregistre le hachage en tant que propriété Content-MD5 de l’objet blob ou du fichier de destination. (par défaut, le hachage n’est pas créé.) Identique au paramètre du même nom dans la commande copy.

**--size-per-file** string  Taille de chaque fichier de données généré automatiquement. Doit être un nombre immédiatement suivi de K, M ou G, par exemple 12K ou 200G (par défaut : « 250M »).

## <a name="options-inherited-from-parent-commands"></a>Options héritées des commandes parentes

**--cap-mbps float**  Limite la vitesse de transfert, en mégabits par seconde. Par moment, le débit peut dépasser légèrement cette limite. Si cette option est définie sur zéro ou si elle est omise, le débit n’est pas limité.

**--output-type** string   Met en forme la sortie de la commande. Les formats possibles sont « text » et « JSON ». La valeur par défaut est « text ». (par défaut : « text »).

La chaîne **--trusted-microsoft-suffixes** spécifie des suffixes de domaine supplémentaires où des jetons de connexion Azure Active Directory peuvent être envoyés.  La valeur par défaut est «  *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net ». Tous les éléments répertoriés ici sont ajoutés à la valeur par défaut. Pour la sécurité, vous devez placer uniquement des domaines Microsoft Azure ici. Séparez plusieurs entrées par des points-virgules.


## <a name="see-also"></a>Voir aussi

- [azcopy](storage-ref-azcopy.md)
