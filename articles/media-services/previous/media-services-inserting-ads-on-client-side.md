---
title: Insertion de publicités du côté client | Microsoft Docs
description: Cet article montre comment insérer des publicités dans votre support côté client.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 65c9c747-128e-497e-afe0-3f92d2bf7972
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 70db1ff63e3c2a7e7806e5ef3c55b4e4af4a5259
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89263567"
---
# <a name="inserting-ads-on-the-client-side"></a>Insertion de publicités du côté client

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Cet article contient des informations sur l’insertion de différents types de publicité du côté client.

Pour en savoir plus sur la prise en charge du sous-titrage codé et des publicités dans les vidéos en flux live, consultez la page [Normes de sous-titrage codé et d’insertion de publicités prises en charge](media-services-live-streaming-with-onprem-encoders.md#cc_and_ads).

> [!NOTE]
> Actuellement, Azure Media Player ne prend pas en charge les publicités.
> 
> 

## <a name="inserting-ads-into-your-media"></a><a id="insert_ads_into_media"></a>Insertion de publicités dans vos supports
Azure Media Services assure la prise en charge des insertions publicitaires via la plateforme Windows Media : Infrastructures de lecteur. Des infrastructures de lecteur avec prise en charge des publicités sont disponibles pour les périphériques Windows 8, Silverlight, Windows Phone 8 et iOS. Chaque infrastructure de lecteur contient un exemple de code qui montre comment implémenter une application de type lecteur. Il existe trois sortes de publicités différentes que vous pouvez insérer dans votre liste multimédia :

* **Linéaire** : publicité en plein cadre qui interrompt la vidéo principale.
* **Non linéaire** : publicité superposée qui s’affiche pendant la lecture de la vidéo principale ; il s’agit généralement d’un logo ou d’une autre image statique apparaissant à l’intérieur du cadre du lecteur.
* **Compagnon** : publicité d’accompagnement qui s’affiche hors du cadre du lecteur.

Les publicités peuvent être placées à n’importe quel point dans la chronologie de la vidéo principale. Vous devez indiquer au lecteur les publicités à diffuser, ainsi que le moment auquel le faire. Cela s’effectue via un ensemble de fichiers XML standard : Video Ad Service Template (VAST), Digital Video Multiple Ad Playlist (VMAP), Media Abstract Sequencing Template (MAST) et Digital Video Player Ad Interface Definition (VPAID). Les fichiers VAST spécifient les publicités à afficher. Les fichiers VMAP indiquent quand diffuser les différentes publicités ; ils contiennent le code XML VAST. Les fichiers MAST permettent également de séquencer des publicités contenant aussi du code XML VAST. Les fichiers VPAID définissent une interface entre le lecteur vidéo et la publicité ou le serveur publicitaire.

Chaque infrastructure de lecteur fonctionne différemment et fait l’objet d’un article distinct. Cet article décrit les mécanismes de base utilisés pour insérer des publicités. Les applications de lecteur vidéo demandent des publicités auprès d’un serveur d’annonces. Ce dernier peut répondre de diverses manières :

* renvoyer un fichier VAST ;
* renvoyer un fichier VMAP (avec VAST incorporé) ;
* renvoyer un fichier MAST (avec VAST incorporé) ;
* renvoyer un fichier VAST avec des publicités VPAID.

### <a name="using-a-video-ad-service-template-vast-file"></a>Utilisation d’un fichier VAST (Video Ad Service Template)
Un fichier VAST spécifie la ou les publicités à afficher. Le code XML suivant constitue un exemple de fichier VAST pour une publicité linéaire :

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="115571748">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://www.myserver.com/tracking-resource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <TrackingEvents>
                  <Tracking event="start"><![CDATA[http://www.myserver.com/start-tracking-resource]]></Tracking>
                  <Tracking event="midpoint"><![CDATA[http://www.myserver.com/midpoint-tracking-resource]]></Tracking>
                  <Tracking event="complete"><![CDATA http://www.myserver.com/complete-tracking-resource]]></Tracking>
                  <Tracking event="expand"><![CDATA[http://www.myserver.com/expand-tracking-resource]]></Tracking>
                </TrackingEvents>
                <VideoClicks>
                  <ClickThrough id="Atlas Redirect"><![CDATA[http://www.myserver.com/click-resource]]></ClickThrough>
                  <ClickTracking id="Spare"></ClickTracking>
                </VideoClicks>
                <MediaFiles>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_200_4x3.wmv]]>
                  </MediaFile>
                  <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                    <![CDATA[http://www.myserver.com/media/myad_300_4x3.wmv]]>
                  </MediaFile>
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
          <Extensions>
            <Extension type="Atlas">
            </Extension>
          </Extensions>
        </InLine>
      </Ad>
    </VAST>
```

La publicité linéaire est décrite par l’élément <**Linear**>. Ce dernier spécifie la durée de la publicité, les événements de suivi, les clics publicitaires, le suivi des clics et un certain nombre d’éléments **MediaFile**. Les événements de suivi sont spécifiés au sein de l’élément **<TrackingEvents>** et permettent à un serveur publicitaire de suivre différents événements qui se produisent lors du visionnage de la publicité. Dans ce cas, le suivi porte sur les événements de début, de milieu et de fin de visionnage, ainsi que sur les événements de développement. L’événement de début se produit lorsque la publicité est affichée. L’événement de milieu survient quand au moins 50 % de la chronologie de la publicité ont été visionnés. L’événement de fin a lieu lorsque la publicité a été visionnée dans son intégralité. L’événement de développement se produit lorsque l’utilisateur développe le lecteur vidéo afin de l’afficher en mode plein écran. Les clics publicitaires sont spécifiés dans un élément <**ClickThrough**> au sein d'un élément <**VideoClicks**> et indiquent un URI vers une ressource à afficher lorsque l'utilisateur clique sur la publicité. Le suivi des clics est spécifié dans un élément <**ClickTracking**>, figurant également au sein de l’élément <**VideoClicks**> et indique une ressource de suivi que le lecteur doit demander lorsque l’utilisateur clique sur la publicité. Les éléments <**MediaFile**> spécifient des informations sur le codage spécifique d’une publicité. En présence de plusieurs éléments <**MediaFile**>, le lecteur vidéo peut choisir l’encodage le plus approprié pour la plateforme.

Les publicités linéaires peuvent être affichées dans un ordre bien précis. Pour ce faire, ajoutez des éléments `<Ad>` supplémentaires au fichier VAST et spécifiez l’ordre à l’aide de l’attribut de séquence. L'exemple suivant illustre ce mécanisme :

```xml
    <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
      <Ad id="1" sequence="0">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad1trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:32</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
      <Ad id="2" sequence="1">
        <InLine>
          <AdSystem version="2.0 alpha">Atlas</AdSystem>
          <AdTitle>Unknown</AdTitle>
          <Description>Unknown</Description>
          <Survey></Survey>
          <Error></Error>
          <Impression id="Atlas"><![CDATA[http://myserver.com/Impression/Ad2trackingResource]]></Impression>
          <Creatives>
            <Creative id="video" sequence="0" AdID="">
              <Linear>
                <Duration>00:00:30</Duration>
                <MediaFiles>
                  <!-- ... -->
                </MediaFiles>
              </Linear>
            </Creative>
          </Creatives>
        </InLine>
      </Ad>
    </VAST>
```

Les publicités non linéaires sont également spécifiées dans un élément `<Creative>`. L’exemple suivant montre un élément `<Creative>` qui décrit une publicité non linéaire.

```xml
    <Creative id="video" sequence="1" AdID="">
      <NonLinearAds>
        <NonLinear width="216" height="121" minSuggestedDuration="00:00:15">
          <StaticResource creativeType="image/png"><![CDATA[http://myserver/images/image.png]]></StaticResource>
          <StaticResource creativeType="image/jpg"><![CDATA[http://myserver/images/image.jpg]]></StaticResource>
        </NonLinear>
        <TrackingEvents>
             <Tracking event="acceptInvitation"><![CDATA[http://myserver/tracking/trackingID]></Tracking>
             <Tracking event="collapse"><![CDATA[http://myserver/tracking/trackingID2]]></Tracking>
         </TrackingEvents>
       </NonLinearAds>
    </Creative>
```

L’élément **<NonLinearAds>** peut contenir un ou plusieurs éléments **<NonLinear>** , chacun d’eux pouvant décrire une publicité non linéaire. L’élément <**NonLinear**> spécifie la ressource de la publicité non linéaire. La ressource peut être une <**StaticResource**>, une <**IFrameResource**> ou une <**HTMLResource**>. \<**StaticResource**> décrit une ressource non HTML et définit un attribut creativeType qui indique le mode d’affichage de la ressource :

Image/gif, image/jpeg, image/png : la ressource est affichée dans une balise HTML <**img**>.

Application/x-javascript : la ressource est affichée dans une balise HTML <**script**>.

Application/x-shockwave-flash : la ressource est affichée dans un lecteur Flash.

**IFrameResource** décrit une ressource HTML qui peut être affichée dans un IFrame. **HTMLREsource** décrit un fragment de code HTML qui peut être inséré dans une page web. **TrackingEvents** spécifie les événements de suivi et l’URI à demander lorsque l’événement se produit. Dans cet exemple, les événements acceptInvitation et de réduction (collapse) font l’objet d’un suivi. Pour plus d'informations sur l’élément **NonLinearAds** et ses enfants, consultez IAB.NET/VAST. Notez que l’élément **TrackingEvents** se trouve dans l’élément **NonLinearAds** plutôt que dans l’élément **NonLinear**.

Les publicités d'accompagnement sont définies dans un élément `<CompanionAds>`. L’élément `<CompanionAds>` peut contenir un ou plusieurs éléments `<Companion>`. Chaque élément `<Companion>` décrit une publicité d'accompagnement et peut contenir un `<StaticResource>`, `<IFrameResource>` ou `<HTMLResource>`, lesquels sont spécifiés de la même manière que dans une publicité non linéaire. Un fichier VAST peut contenir plusieurs publicités d’accompagnement ; il revient alors à l’application de type lecteur de choisir la publicité la plus appropriée à afficher. Pour plus d'informations sur VAST, consultez [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf).

### <a name="using-a-digital-video-multiple-ad-playlist-vmap-file"></a>Utilisation d’un fichier VMAP (Video Multiple Ad Playlist)
Un fichier VMAP vous permet de spécifier le moment où des coupures publicitaires doivent avoir lieu, la durée de chaque coupure, le nombre et les types de publicités pouvant être affichées pendant une coupure. L’exemple suivant de fichier VMAP définit une seule coupure publicitaire :

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <VAST version="2.0" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="oxml.xsd">
              <Ad id="115571748">
                <InLine>
                  <AdSystem version="2.0 alpha">Atlas</AdSystem>
                  <AdTitle>Unknown</AdTitle>
                  <Description>Unknown</Description>
                  <Survey></Survey>
                  <Error></Error>
                  <Impression id="Atlas"><![CDATA[https://view.atdmt.com/000/sview/115571748/direct;ai.201582527;vt.2/01/634364885739970673]]></Impression>
                  <Creatives>
                    <Creative id="video" sequence="0" AdID="">
                      <Linear>
                        <Duration>00:00:32</Duration>
                        <MediaFiles>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_200" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="200" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_200_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_300" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="300" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_300_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_500" maintainAspectRatio="true" scalable="true"  delivery="progressive" bitrate="500" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_1_000_500_4x3.wmv]]>
                          </MediaFile>
                          <MediaFile apiFramework="Windows Media" id="windows_progressive_700" maintainAspectRatio="true" scalable="true" delivery="progressive" bitrate="700" width="400" height="300" type="video/x-ms-wmv">
                            <![CDATA[http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv]]>
                          </MediaFile>
                        </MediaFiles>
                      </Linear>
                    </Creative>
                  </Creatives>
                </InLine>
              </Ad>
            </VAST>
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

Un fichier VMAP commence par un élément `<VMAP>` qui contient un ou plusieurs éléments `<AdBreak>`, chacun définissant une coupure publicitaire. Chaque coupure publicitaire spécifie un type de coupure, un identificateur et une durée de décalage. L’attribut breakType spécifie le type de publicité pouvant être diffusée pendant la coupure : linéaire, non linéaire ou d’affichage. Les publicités d’affichage correspondent à des publicités VAST d’accompagnement. Plusieurs types de publicités peuvent être spécifiés dans une liste séparée par des virgules (sans espaces). L’attribut breakID est un identificateur facultatif pour la publicité. L’attribut timeOffset indique quand la publicité doit être affichée. Il peut être spécifié de l’une des manières suivantes :

1. Durée : au format hh:mm:ss ou hh:mm:ss.mmm, où .mmm représente des millisecondes. La valeur de cet attribut indique la durée depuis le début de la chronologie vidéo jusqu’au début de la coupure publicitaire.
2. Pourcentage : au format n%, où n correspond au pourcentage de la chronologie vidéo à lire avant de diffuser la publicité.
3. Début/Fin : spécifie qu’une publicité doit être diffusée avant ou après l’affichage de la vidéo.
4. Position : spécifie l’ordre des coupures publicitaires, lorsque le délai des coupures publicitaires est inconnu, comme lors d’une diffusion en direct. L’ordre de chaque coupure publicitaire est spécifié au format #n, où n représente un entier supérieur ou égal à 1. 1 signifie que la publicité doit être diffusée à la première occasion, 2 qu’elle doit l’être à la deuxième occasion, et ainsi de suite.

L’élément `<AdBreak>` peut renfermer un élément <**AdSource**>. L’élément <**AdSource**> contient les attributs suivants :

1. Id : spécifie l’identificateur de la source publicitaire.
2. allowMultipleAds : valeur booléenne qui spécifie si plusieurs publicités peuvent être affichées au cours de la coupure publicitaire.
3. followRedirects : valeur booléenne facultative qui spécifie si le lecteur vidéo doit procéder aux redirections dans une réponse publicitaire.

L’élément <**AdSource**> fournit au lecteur une réponse publicitaire insérée ou une référence à une réponse publicitaire. Il peut contenir l’un des éléments suivants :

* `<VASTAdData>` : indique qu’une réponse publicitaire VAST est incorporée au sein du fichier VMAP.
* `<AdTagURI>` : URI qui fait référence à une réponse publicitaire émanant d’un autre système.
* `<CustomAdData>` : chaîne arbitraire qui représente une réponse non VAST.

Dans cet exemple, une réponse publicitaire insérée est spécifiée avec un élément `<VASTAdData>` qui contient une réponse publicitaire VAST. Pour plus d’informations sur les autres éléments, consultez [VMAP](https://www.iab.net/guidelines/508676/digitalvideo/vsuite/vmap).

L’élément <**AdBreak**> peut également contenir un élément <**TrackingEvents**>. L’élément <**TrackingEvents**> vous permet de suivre le début ou la fin d’une coupure publicitaire ou de déterminer si une erreur s’est produite pendant la coupure. L’élément <**TrackingEvents**> contient un ou plusieurs éléments <**Tracking**>, chacun d’eux spécifiant un événement et un URI de suivi. Les événements de suivi possibles sont les suivants :

1. breakStart : effectue le suivi du début d’une coupure publicitaire.
2. breakEnd : effectue le suivi de la fin d’une coupure publicitaire.
3. error : effectue le suivi d’une erreur qui s’est produite pendant la coupure publicitaire.

L’exemple suivant illustre un fichier VMAP qui spécifie des événements de suivi

```xml
    <vmap:VMAP xmlns:vmap="http://www.iab.net/vmap-1.0" version="1.0">
      <vmap:AdBreak breakType="linear" breakId="mypre" timeOffset="start">
        <vmap:AdSource allowMultipleAds="true" followRedirects="true" id="1">
          <vmap:VASTData>
            <!--Inline VAST -->
          </vmap:VASTData>
        </vmap:AdSource>
        <vmap:TrackingEvents>
          <vmap:Tracking event="breakStart">
            http://MyServer.com/breakstart.gif
          </vmap:Tracking>
          <vmap:Tracking event="breakend">
            http://MyServer.com/breakend.gif
          </vmap:Tracking>
          <vmap:Tracking event="error">
            http://MyServer.com/error.gif
          </vmap:Tracking>
        </vmap:TrackingEvents>
      </vmap:AdBreak>
    </vmap:VMAP>
```

Pour plus d’informations sur l’élément <**TrackingEvents**> et ses enfants, consultez http://iab.net/VMAP.pdf

### <a name="using-a-media-abstract-sequencing-template-mast-file"></a>Utilisation d’un fichier MAST (Media Abstract Sequencing Template)
Un fichier MAST vous permet de spécifier les déclencheurs qui définissent le moment où une publicité doit être affichée. L’exemple suivant illustre un fichier MAST qui contient des déclencheurs pour une publicité de début de bande, mi-bande et de fin de bande.

```xml
    <MAST xsi:schemaLocation="http://openvideoplayer.sf.net/mast http://openvideoplayer.sf.net/mast/mast.xsd" xmlns="http://openvideoplayer.sf.net/mast" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
      <triggers>
        <trigger id="preroll" description="preroll every item"  >
          <startConditions>
            <condition type="event" name="OnItemStart" />
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="midroll" description="midroll at 15 sec."  >
          <startConditions>
            <condition type="property" name="Position" value="00:00:15.0" operator="GEQ" />
          </startConditions>
          <endConditions>
            <condition type="event" name="OnItemEnd"/>
            <!--This 'resets' the trigger for the next clip-->
          </endConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>

        <trigger id="postroll" description="postroll"  >
          <startConditions>
            <condition type="event" name="OnItemEnd"/>
          </startConditions>
          <sources>
            <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
              <sources />
            </source>
          </sources>
        </trigger>
      </triggers>
    </MAST>
```


Un fichier MAST commence par un élément **MAST** qui contient un élément **triggers**. L’élément `<triggers>` contient un ou plusieurs éléments **trigger** qui définissent le moment où une publicité doit être affichée.

L’élément **trigger** contient un élément **startConditions** qui spécifie quand la diffusion d’une publicité doit commencer. L’élément **startConditions** contient un ou plusieurs éléments `<condition>`. Quand chaque `<condition>` prend la valeur True, un déclencheur est lancé ou révoqué, selon que la `<condition>` est contenue respectivement dans un élément **startConditions** ou **endConditions**. En présence de plusieurs éléments `<condition>`, ces derniers sont traités comme une condition OR implicite ; autrement dit, toute condition prenant la valeur True entraîne le lancement du déclencheur. Les éléments `<condition>` peuvent être imbriqués. Lorsque des éléments `<condition>` enfants sont prédéfinis, ils sont traités comme une condition AND implicite ; autrement dit, toutes les conditions doivent prendre la valeur True pour que le déclencheur soit lancé. L’élément `<condition>` contient les attributs suivants qui définissent la condition :

1. **type** : spécifie le type de condition, d’événement ou de propriété.
2. **name** : nom de la propriété ou de l’événement à utiliser au cours de l’évaluation.
3. **value** : valeur à laquelle une propriété sera comparée.
4. **operator** : opération à utiliser pendant l’évaluation : EQ (égal à), NEQ (différent de), GTR (supérieur à), GEQ (supérieur ou égal à), LT (inférieur à), LEQ (inférieur ou égal à), MOD (modulo).

**&lt;endConditions&gt;** contient également des éléments `<condition>`. Quand une condition prend la valeur true, le déclencheur est réinitialisé. L’élément `<trigger>` contient également un élément `<sources>` qui comprend un ou plusieurs éléments `<source>`. Les éléments `<source>` définissent l’URI vers la réponse publicitaire et le type de réponse publicitaire. Dans cet exemple, un URI est donné à une réponse VAST.

```xml
    <trigger id="postroll" description="postroll"  >
      <startConditions>
        <condition/>
      </startConditions>
      <sources>
        <source uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" format="vast">
          <sources />
        </source>
      </sources>
    </trigger>
```

### <a name="using-video-player-ad-interface-definition-vpaid"></a>Utilisation de VPAID (Video Player-Ad Interface Definition)
VPAID est une API permettant la communication d’unités publicitaires exécutables avec un lecteur vidéo. Cela permet d’offrir une expérience publicitaire hautement interactive. L’utilisateur peut interagir avec la publicité, cette dernière pouvant répondre aux actions effectuées par la personne qui la visionne. Par exemple, une publicité peut afficher des boutons permettant à l’utilisateur d’afficher des informations supplémentaires ou une version plus longue de la publicité. Le lecteur vidéo doit prendre en charge l’API VPAID ; quant à la publicité exécutable, elle doit implémenter cette API. Lorsqu’un lecteur demande une publicité auprès d’un serveur publicitaire, ce dernier peut renvoyer une réponse VAST qui contient une publicité VPAID.

Une publicité exécutable est créée dans un code qui doit être exécuté dans un environnement d’exécution, tel qu’Adobe Flash™ ou JavaScript, pouvant être exécuté dans un navigateur web. Lorsqu’un serveur publicitaire renvoie une réponse VAST contenant une publicité VPAID, la valeur de l’attribut apiFramework dans l’élément `<MediaFile>` doit être « VPAID ». Cet attribut spécifie que la publicité contenue est une publicité VPAID exécutable. L’attribut de type doit être défini sur le type MIME de l’exécutable, comme « application/x-shockwave-flash » ou « application/x-javascript ». L’extrait de code XML suivant montre l’élément `<MediaFile>` d’une réponse VAST contenant une publicité VPAID exécutable.

```xml
    <MediaFiles>
       <MediaFile id="1" delivery="progressive" type="application/x-shockwaveflash"
                  width="640" height="480" apiFramework="VPAID">
           <!-- CDATA wrapped URI to executable ad -->
       </MediaFile>
    </MediaFiles>
```

Une publicité exécutable peut être initialisée à l’aide de l’élément `<AdParameters>` au sein des éléments `<Linear>` ou `<NonLinear>` dans une réponse VAST. Pour plus d’informations sur l’élément `<AdParameters>`, consultez [VAST 3.0](https://www.iab.net/media/file/VASTv3.0.pdf). Pour plus d’informations sur l’API VPAID, consultez [VPAID 2.0](https://www.iab.net/media/file/VPAID_2.0_Final_04-10-2012.pdf).

## <a name="implementing-a-windows-or-windows-phone-8-player-with-ad-support"></a>Implémentation d’un lecteur Windows ou Windows Phone 8 avec prise en charge de publicités
Plateforme multimédia Microsoft : infrastructure de lecteur pour Windows 8 et Windows Phone 8 contient une collection d’exemples d’applications qui vous montrent comment implémenter une application de lecteur vidéo à l’aide de l’infrastructure. Vous pouvez télécharger l'infrastructure depuis la page [Player Framework pour Windows 8 et Windows Phone 8](https://playerframework.codeplex.com).

Lorsque vous ouvrez la solution Microsoft.PlayerFramework.Xaml.Samples, vous découvrez un certain nombre de dossiers au sein du projet. Le dossier Advertising contient l’exemple de code permettant la création d’un lecteur vidéo avec prise en charge de publicités. Le dossier Advertising renferme plusieurs fichiers XAML/cs, chacun d’eux montrant comment insérer des publicités d’une manière particulière. Le tableau ci-dessous décrit chaque fichier :

* AdPodPage.xaml Montre comment afficher un bloc publicitaire.
* AdSchedulingPage.xaml Montre comment planifier des publicités.
* FreeWheelPage.xaml Montre comment utiliser le plug-in FreeWheel pour planifier des publicités.
* MastPage.xaml Montre comment planifier des publicités avec un fichier MAST.
* ProgrammaticAdPage.xaml Montre comment planifier par programmation des publicités dans une vidéo.
* ScheduleClipPage.xaml Montre comment planifier une publicité sans fichier VAST.
* VastLinearCompanionPage.xaml Montre comment insérer une publicité linéaire et d’accompagnement.
* VastNonLinearPage.xaml Montre comment insérer une publicité non linéaire.
* VmapPage.xaml Montre comment spécifier des publicités avec un fichier VMAP.

Chacun de ces exemples utilise la classe MediaPlayer définie par l’infrastructure de lecteur. La plupart des exemples utilisent des plug-ins qui apportent une prise en charge de différents formats de réponses publicitaires. L’exemple ProgrammaticAdPage interagit par programmation avec une instance MediaPlayer.

### <a name="adpodpage-sample"></a>Exemple AdPodPage
Cet exemple utilise AdSchedulerPlugin pour définir quand afficher une publicité. Dans cet exemple, une publicité mi-bande est planifiée pour être diffusée après un délai de cinq secondes. Le bloc publicitaire (groupe de publicités à afficher dans un ordre précis) est spécifié dans un fichier VAST renvoyé depuis un serveur publicitaire. L'URI vers le fichier VAST est spécifiée dans l'élément `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">

        <mmppf:MediaPlayer.Plugins>
            <ads:AdSchedulerPlugin>
                <ads:AdSchedulerPlugin.Advertisements>

                    <ads:MidrollAdvertisement Time="00:00:05">
                        <ads:MidrollAdvertisement.Source>
                            <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_adpod.xml" Type="vast"/>
                        </ads:MidrollAdvertisement.Source>
                    </ads:MidrollAdvertisement>

                </ads:AdSchedulerPlugin.Advertisements>
            </ads:AdSchedulerPlugin>
            <ads:AdHandlerPlugin/>
        </mmppf:MediaPlayer.Plugins>
    </mmppf:MediaPlayer>
```

Pour plus d'informations sur AdSchedulerPlugin, consultez [Publicités dans l'infrastructure de lecteur sous Windows 8 et Windows Phone 8](https://playerframework.codeplex.com/wikipage?title=Advertising&referringTitle=Windows%208%20Player%20Documentation)

### <a name="adschedulingpage"></a>AdSchedulingPage
Cet exemple utilise également AdSchedulerPlugin. Il planifie trois publicités, une publicité de début de bande, une publicité mi-bande et une publicité de fin de bande. L'URI vers le fichier VAST de chaque publicité est spécifié dans un élément `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:PrerollAdvertisement>
                                <ads:PrerollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PrerollAdvertisement.Source>
                            </ads:PrerollAdvertisement>

                            <ads:MidrollAdvertisement Time="00:00:15">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                            <ads:PostrollAdvertisement>
                                <ads:PostrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml" Type="vast"/>
                                </ads:PostrollAdvertisement.Source>
                            </ads:PostrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="freewheelpage"></a>FreeWheelPage
Cet exemple utilise FreeWheelPlugin qui spécifie un attribut Source, lequel indique un URI qui désigne un fichier SmartXML précisant le contenu publicitaire, ainsi que les informations de planification de la publicité.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:FreeWheelPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/freewheel.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="mastpage"></a>MastPage
Cet exemple utilise MastSchedulerPlugin qui vous permet d’employer un fichier MAST. L’attribut Source spécifie l’emplacement du fichier MAST.
```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:MastSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/mast.xml" />
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="programmaticadpage"></a>ProgrammaticAdPage
Cet exemple interagit par programmation avec MediaPlayer. Le fichier ProgrammaticAdPage.xaml instancie MediaPlayer :

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4"/>
```

Le fichier ProgrammaticAdPage.xaml.cs crée un AdHandlerPlugin, ajoute un TimelineMarker pour indiquer le moment où une publicité doit être affichée, puis ajoute un gestionnaire pour l’événement MarkerReached qui charge un RemoteAdSource spécifiant un URI vers un fichier VAST. Enfin, il diffuse la publicité.

```csharp
    public sealed partial class ProgrammaticAdPage : Microsoft.PlayerFramework.Samples.Common.LayoutAwarePage
        {
            AdHandlerPlugin adHandler;

            public ProgrammaticAdPage()
            {
                this.InitializeComponent();
                adHandler = new AdHandlerPlugin();
                player.Plugins.Add(new AdHandlerPlugin());
                player.Markers.Add(new TimelineMarker() { Time = TimeSpan.FromSeconds(5), Type = "myAd" });
                player.MarkerReached += pf_MarkerReached;
            }

            async void pf_MarkerReached(object sender, TimelineMarkerRoutedEventArgs e)
            {
                if (e.Marker.Type == "myAd")
                {
                    var adSource = new RemoteAdSource() { Type = VastAdPayloadHandler.AdType, Uri = new Uri("http://smf.blob.core.windows.net/samples/win8/ads/vast_linear.xml") };
                    //var adSource = new AdSource() { Type = DocumentAdPayloadHandler.AdType, Payload = SampleAdDocument };
                    var progress = new Progress<AdStatus>();
                    try
                    {
                        await player.PlayAd(adSource, progress, CancellationToken.None);
                    }
                    catch { /* ignore */ }
                }
            }
```

### <a name="scheduleclippage"></a>ScheduleClipPage
Cet exemple utilise AdSchedulerPlugin pour planifier une publicité mi-bande en spécifiant un fichier .wmv qui contient la publicité en question.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.cloudapp.net/html5/media/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:AdSource Type="clip">
                                        <ads:AdSource.Payload>
                                            <ads:ClipAdPayload MediaSource="http://smf.blob.core.windows.net/samples/ads/media/XBOX_HD_DEMO_700_2_000_700_4x3.wmv" MimeType="video/x-ms-wmv" />
                                        </ads:AdSource.Payload>
                                    </ads:AdSource>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearcompanionpage"></a>VastLinearCompanionPage
Cet exemple montre comment utiliser AdSchedulerPlugin pour planifier une publicité mi-bande linéaire avec une publicité d’accompagnement. L'élément `<RemoteAdSource>` spécifie l'emplacement du fichier VAST.

```xml
    <mmppf:MediaPlayer Grid.Row="1"  x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_companions.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vastlinearnonlinearpage"></a>VastLinearNonLinearPage
Cet exemple utilise AdSchedulerPlugin pour planifier une publicité linéaire et non linéaire. L’emplacement du fichier VAST est spécifié avec l’élément `<RemoteAdSource>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:AdSchedulerPlugin>
                        <ads:AdSchedulerPlugin.Advertisements>

                            <ads:MidrollAdvertisement Time="00:00:05">
                                <ads:MidrollAdvertisement.Source>
                                    <ads:RemoteAdSource Uri="http://smf.blob.core.windows.net/samples/win8/ads/vast_linear_nonlinear.xml" Type="vast"/>
                                </ads:MidrollAdvertisement.Source>
                            </ads:MidrollAdvertisement>

                        </ads:AdSchedulerPlugin.Advertisements>
                    </ads:AdSchedulerPlugin>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

### <a name="vmappage"></a>VMAPPage
Cet exemple utilise VmapSchedulerPlugin pour planifier des publicités à l’aide d’un fichier VMAP. L'URI vers le fichier VMAP est spécifié dans l'attribut Source de l'élément `<VmapSchedulerPlugin>`.

```xml
    <mmppf:MediaPlayer x:Name="player" Source="http://smf.blob.core.windows.net/samples/videos/bigbuck.mp4">
                <mmppf:MediaPlayer.Plugins>
                    <ads:VmapSchedulerPlugin Source="http://smf.blob.core.windows.net/samples/win8/ads/vmap.xml"/>
                    <ads:AdHandlerPlugin/>
                </mmppf:MediaPlayer.Plugins>
            </mmppf:MediaPlayer>
```

## <a name="implementing-an-ios-video-player-with-ad-support"></a>Implémentation d'un lecteur vidéo iOS avec prise en charge de publicités
Plateforme multimédia Microsoft : infrastructure de lecteur pour iOS contient une collection d’exemples d’applications qui vous montrent comment implémenter une application de lecteur vidéo à l’aide de l’infrastructure. Vous pouvez télécharger l'infrastructure de lecteur et les exemples depuis la page [Azure Media Player Framework](https://github.com/CloudMetal/azure-media-player-framework). La page GitHub comporte un lien vers un site Wiki qui contient des informations supplémentaires sur l'infrastructure de lecteur, ainsi qu'une présentation de l'exemple de lecteur : [Lecteur multimédia Azure Wiki](https://github.com/CloudMetal/azure-media-player-framework/wiki/How-to-use-Azure-media-player-framework).

### <a name="scheduling-ads-with-vmap"></a>Planification de publicités avec VMAP
L’exemple ci-dessous montre comment planifier des publicités à l’aide d’un fichier VMAP.

```csharp
    // How to schedule an Ad using VMAP.
    //First download the VMAP manifest

    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVMAP.xml"]])
            {
                [self logFrameworkError];
            }
            else
            {
                // Schedule a list of ads using the downloaded VMAP manifest
                if (![framework scheduleVMAPWithManifest:manifest])
                {
                    [self logFrameworkError];
                }
            }
```

### <a name="scheduling-ads-with-vast"></a>Planification de publicités avec VAST
L’exemple ci-dessous montre comment planifier une publicité VAST à liaison tardive.


```csharp
    //Example:3 How to schedule a late binding VAST ad.
    // set the start time for the ad
    adLinearTime.startTime = 13;
    adLinearTime.duration = 0;
    // Specify the URI of the VAST file
    NSString *vastAd1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml";
    // Create an AdInfo object
     AdInfo *vastAdInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URL to VAST file
    vastAdInfo1.clipURL = [NSURL URLWithString:vastAd1];
    // set running time of ad
    vastAdInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    vastAdInfo1.mediaTime.clipBeginMediaTime = 0;
    vastAdInfo1.mediaTime.clipEndMediaTime = 10;
    vastAdInfo1.policy = @"policy for late binding VAST";
    // specify ad type
    vastAdInfo1.type = AdType_Midroll;
    vastAdInfo1.appendTo=-1;
    adIndex = 0;
    // schedule ad
    if (![framework scheduleClip:vastAdInfo1 atTime:adLinearTime forType:PlaylistEntryType_VAST andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

   L’exemple ci-dessous montre comment planifier une publicité VAST à liaison anticipée.

```csharp
    //Example:4 Schedule an early binding VAST ad
    //Download the VAST file
    if (![framework.adResolver downloadManifest:&manifest withURL:[NSURL URLWithString:@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/vast/PlayerTestVAST.xml"]])
    {
        [self logFrameworkError];
    }
    else
    {
        adLinearTime.startTime = 7;
        adLinearTime.duration = 0;

        // Create AdInfo instance
        AdInfo *vastAdInfo2 = [[[AdInfo alloc] init] autorelease];
        vastAdInfo2.mediaTime = [[[MediaTime alloc] init] autorelease];
        vastAdInfo2.policy = @"policy for early binding VAST";
        // specify ad type
        vastAdInfo2.type = AdType_Midroll;
        vastAdInfo2.appendTo=-1;
        // schedule ad
        if (![framework scheduleVASTClip:vastAdInfo2 withManifest:manifest atTime:adLinearTime andGetClipId:&adIndex])
        {
            [self logFrameworkError];
        }
    }
```

L’exemple ci-dessous indique comment insérer une publicité à l’aide de RCE (Rough Cut Editing).

```csharp
    //Example:1 How to use RCE.
    // specify manifest for ad content
    NSString *secondContent=@"http://wamsblureg001orig-hs.cloudapp.net/6651424c-a9d1-419b-895c-6993f0f48a26/The%20making%20of%20Microsoft%20Surface-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // specify ad length
    mediaTime.currentPlaybackPosition = 0;
    mediaTime.clipBeginMediaTime = 0;
    mediaTime.clipEndMediaTime = 80;
    // append ad content
    if (![framework appendContentClip:[NSURL URLWithString:secondContent] withMediaTime:mediaTime andGetClipId:&clipId])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier un bloc publicitaire.

```csharp
    //Example:5 Schedule an ad Pod.
    // Set start time for ad
    adLinearTime.startTime = 23;
    adLinearTime.duration = 0;

    // Specify URL to content
    NSString *adpodSt1=@"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    // Create an AdInfo instance
    AdInfo *adpodInfo1 = [[[AdInfo alloc] init] autorelease];
    // set URI to ad content
    adpodInfo1.clipURL = [NSURL URLWithString:adpodSt1];
    // Set ad running time
    adpodInfo1.mediaTime = [[[MediaTime alloc] init] autorelease];
    adpodInfo1.mediaTime.clipBeginMediaTime = 0;
    adpodInfo1.mediaTime.clipEndMediaTime = 17;
    adpodInfo1.policy = @"policy for ad pod 1";
    // Set ad type
    adpodInfo1.type = AdType_Midroll;
    adpodInfo1.appendTo=-1;
    adIndex = 0;
    // Schedule ad
    if (![framework scheduleClip:adpodInfo1 atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier une publicité mi-bande non répétitive. Une publicité non répétitive est diffusée une seule fois, quelle que soit la recherche effectuée par la personne qui la visionne.

```csharp
    //Example:6 Schedule a single non sticky mid roll Ad
    // specify URL to content
    NSString *oneTimeAd=@"http://wamsblureg001orig-hs.cloudapp.net/5389c0c5-340f-48d7-90bc-0aab664e5f02/Windows%208_%20You%20and%20Me%20Together-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";

    // create an AdInfo instance
    AdInfo *oneTimeInfo = [[[AdInfo alloc] init] autorelease];
    // set URL of ad
    oneTimeInfo.clipURL = [NSURL URLWithString:oneTimeAd];
    oneTimeInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    oneTimeInfo.mediaTime.clipBeginMediaTime = 0;
    oneTimeInfo.mediaTime.clipEndMediaTime = -1;
    oneTimeInfo.policy = @"policy for one-time ad";
    // set ad start time
    adLinearTime.startTime = 43;
    adLinearTime.duration = 0;
    // set ad type
    oneTimeInfo.type = AdType_Midroll;
    // non sticky ad
    oneTimeInfo.deleteAfterPlayed = YES;
    // schedule ad
    if (![framework scheduleClip:oneTimeInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier une publicité mi-bande répétitive. Une publicité répétitive est diffusée chaque fois que le point spécifié sur la chronologie de la vidéo est atteint.

```csharp
    //Example:7 Schedule a single sticky mid roll Ad
    NSString *stickyAd=@"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *stickyAdInfo = [[[AdInfo alloc] init] autorelease];
    // set URI to ad
    stickyAdInfo.clipURL = [NSURL URLWithString:stickyAd];
    stickyAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    stickyAdInfo.mediaTime.clipBeginMediaTime = 0;
    stickyAdInfo.mediaTime.clipEndMediaTime = 15;
    stickyAdInfo.policy = @"policy for sticky mid-roll ad";
    // set ad start time
    adLinearTime.startTime = 64;
    adLinearTime.duration = 0;
    // set ad type
    stickyAdInfo.type = AdType_Midroll;
    stickyAdInfo.deleteAfterPlayed = NO;
    // schedule ad
    if (![framework scheduleClip:stickyAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier une publicité de fin de bande.

```csharp
    //Example:8 Schedule Post Roll Ad
    NSString *postAdURLString=@"http://wamsblureg001orig-hs.cloudapp.net/aa152d7f-3c54-487b-ba07-a58e0e33280b/wp-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    // create AdInfo instance
    AdInfo *postAdInfo = [[[AdInfo alloc] init] autorelease];
    postAdInfo.clipURL = [NSURL URLWithString:postAdURLString];
    postAdInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    postAdInfo.mediaTime.clipBeginMediaTime = 0;
    // set ad length
    postAdInfo.mediaTime.clipEndMediaTime = 45;
    postAdInfo.policy = @"policy for post-roll ad";
    // set ad type
    postAdInfo.type = AdType_Postroll;
    adLinearTime.duration = 0;
    if (![framework scheduleClip:postAdInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier une publicité de début de bande.

```csharp
    //Example:9 Schedule Pre Roll Ad
    NSString *adURLString = @"http://wamsblureg001orig-hs.cloudapp.net/2e4e7d1f-b72a-4994-a406-810c796fc4fc/The%20Surface%20Movement-m3u8-aapl.ism/Manifest(format=m3u8-aapl)";
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 40; //You could play a portion of an Ad. Yeh!
    adInfo.mediaTime.clipEndMediaTime = 59;
    adInfo.policy = @"policy for pre-roll ad";
    adInfo.appendTo = -1;
    adInfo.type = AdType_Preroll;
    adLinearTime.duration = 0;
    // schedule ad
    if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```

L’exemple ci-dessous montre comment planifier une publicité mi-bande de recouvrement.

```csharp
    // Example10: Schedule a Mid Roll overlay Ad
    NSString *adURLString = @"https://portalvhdsq3m25bf47d15c.blob.core.windows.net/asset-e47b43fd-05dc-4587-ac87-5916439ad07f/Windows%208_%20Cliffjumpers.mp4?st=2012-11-28T16%3A31%3A57Z&se=2014-11-28T16%3A31%3A57Z&sr=c&si=2a6dbb1e-f906-4187-a3d3-7e517192cbd0&sig=qrXYZBekqlbbYKqwovxzaVZNLv9cgyINgMazSCbdrfU%3D";
    //Create AdInfo instance
    AdInfo *adInfo = [[[AdInfo alloc] init] autorelease];
    adInfo.clipURL = [NSURL URLWithString:adURLString];
    adInfo.mediaTime = [[[MediaTime alloc] init] autorelease];
    adInfo.mediaTime.currentPlaybackPosition = 0;
    adInfo.mediaTime.clipBeginMediaTime = 0;
    // specify ad length
    adInfo.mediaTime.clipEndMediaTime = 20;
    adInfo.policy = @"policy for mid-roll overlay ad";
    adInfo.appendTo = -1;
    // specify ad type
    adInfo.type = AdType_Midroll;
    // specify ad start time & duration
    adLinearTime.startTime = 300;
    adLinearTime.duration = 20;
    // schedule ad            if (![framework scheduleClip:adInfo atTime:adLinearTime forType:PlaylistEntryType_Media andGetClipId:&adIndex])
    {
        [self logFrameworkError];
    }
```


## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
