---
title: Schéma de métadonnées de sortie Azure Media Services | Microsoft Docs
description: Cet article fournit une vue d’ensemble du schéma de métadonnées de sortie d’Azure Media Services.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1ed84c88-eea5-4a24-9c4f-f2428157d08a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: cd81ef78ecc5ef9cea71adb387597681460d50c8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89261320"
---
# <a name="output-metadata"></a>Métadonnées de sortie

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble
Un travail d’encodage est associé à un élément multimédia d’entrée (ou plusieurs) sur lequel vous souhaitez effectuer des tâches d’encodage. Par exemple, encoder un fichier MP4 en ensembles de fichiers MP4 à vitesse de transmission adaptative H.264, créer une miniature, créer des superpositions. À l’achèvement d’une tâche, une ressource de sortie est générée.  L’élément multimédia de sortie contient la vidéo, l’audio, les miniatures, et ainsi de suite. L’élément multimédia de sortie contient également un fichier avec des métadonnées relatives à l’élément multimédia de sortie. Le nom du fichier XML de métadonnées a le format suivant : &lt;nom_fichier_source&gt;_manifest.xml (par exemple, BigBuckBunny_manifest.xml).  

Media Services n’analyse pas préemptivement les éléments multimédia d’entrée pour générer des métadonnées. Les métadonnées d’entrée sont générées uniquement sous forme d’artefact lorsqu’un élément multimédia d’entrée est traité au sein d’un travail. Par conséquent, cet artefact est écrit dans l’élément multimédia de sortie. Différents outils sont utilisés pour générer les métadonnées pour les éléments multimédias d’entrée et de sortie. Ainsi, les métadonnées d’entrée présentent un schéma légèrement différent de celui des métadonnées de sortie.

Si vous souhaitez examiner le fichier de métadonnées, vous pouvez créer un localisateur **SAS** et télécharger le fichier sur votre ordinateur local.  

Cet article décrit les éléments et types du schéma XML sur lesquels les métadonnées de sortie (&lt;source_file_name&gt;_manifest.xml) sont basées. Pour plus d’informations sur le fichier qui contient des métadonnées sur la ressource d’entrée, consultez Métadonnées d’entrée.  

Vous pouvez trouver le Code du schéma complet et un exemple de XML à la fin de cet article.  

## <a name="assetfiles-root-element"></a><a name="AssetFiles"></a> Élément racine AssetFiles
Collection d’entrées AssetFile pour le travail d’encodage.  

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **AssetFile**<br/><br/> minOccurs="0" maxOccurs="1" |Un élément AssetFile qui fait partie de la collection AssetFiles. |

## <a name="assetfile-element"></a><a name="AssetFile"></a> Élément AssetFile
Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Nom**<br/><br/> Obligatoire |**xs:string** |Le nom du fichier multimédia. |
| **Taille**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:long** |Taille du fichier de ressource en octets. |
| **Durée**<br/><br/> Obligatoire |**xs:duration** |Durée de lecture du contenu. |

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **Sources** |Collection de fichiers multimédias d’entrée/source qui a été traitée afin de produire cet AssetFile. Pour plus d’informations, consultez la page Élément source. |
| **VideoTracks**<br/><br/> minOccurs="0" maxOccurs="1" |Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. Pour plus d’informations, consultez l’élément VideoTracks. |
| **AudioTracks**<br/><br/> minOccurs="0" maxOccurs="1" |Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. Il s’agit de la collection de toutes les pistes audio. Pour plus d’informations, consultez l’élément AudioTracks. |

## <a name="sources-element"></a><a name="Sources"></a> Sources (élément)
Collection de fichiers multimédias d’entrée/source qui a été traitée afin de produire cet AssetFile.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **Source**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Un fichier d’entrée/source utilisé lors de la génération de cette ressource. Pour plus d’informations, consultez la page Élément source. |

## <a name="source-element"></a><a name="Source"></a> Élément source
Un fichier d’entrée/source utilisé lors de la génération de cette ressource.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Nom**<br/><br/> Obligatoire |**xs:string** |Nom du fichier source d’entrée. |

## <a name="videotracks-element"></a><a name="VideoTracks"></a> Élément VideoTracks
Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes vidéo entrelacées dans un format de conteneur approprié. L’élément **VideoTracks** représente une collection de toutes les pistes vidéo.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **VideoTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Une piste vidéo spécifique dans l’élément AssetFile parent. Pour plus d’informations, consultez l’élément VideoTrack. |

## <a name="videotrack-element"></a><a name="VideoTrack"></a> Élément VideoTrack
Une piste vidéo spécifique dans l’élément AssetFile parent.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Index de base zéro de cette piste vidéo. **Remarque :**  Cet **ID** ne correspond pas nécessairement au TrackID tel qu’utilisé dans un fichier MP4. |
| **FourCC**<br/><br/> Obligatoire |**xs:string** |Code FourCC du codec vidéo. |
| **Profil** |**xs:string** |Profil H264 (uniquement applicable au codec H264). |
| **Niveau** |**xs:string** |Niveau H264 (uniquement applicable au codec H264). |
| **Width**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Largeur vidéo encodée en pixels. |
| **Height**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Hauteur vidéo encodée en pixels. |
| **DisplayAspectRatioNumerator**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:double** |Numérateur des proportions d’affichage vidéo. |
| **DisplayAspectRatioDenominator**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:double** |Dénominateur des proportions d’affichage vidéo. |
| **Framerate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:decimal** |Fréquence d’images vidéo mesurée au format .3f. |
| **TargetFramerate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:decimal** |Fréquence d’images vidéo cible prédéfinie au format .3f. |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Vitesse de transmission vidéo moyenne en kilobits par seconde, telle que calculée à partir du fichier de ressource. Compte uniquement la charge utile de flux élémentaire et n’inclut pas la surcharge de packaging. |
| **TargetBitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Vitesse de transmission cible moyenne pour cette piste vidéo, comme demandé via paramètres prédéfinis d’encodage, en kilobits par seconde. |
| **MaxGOPBitrate**<br/><br/> minInclusive ="0" |**xs:int** |Vitesse de transmission moyenne GOP maximum pour cette piste vidéo, en kilobits par seconde. |

## <a name="audiotracks-element"></a><a name="AudioTracks"></a> Élément AudioTracks
Chaque élément AssetFile physique peut contenir zéro ou plusieurs pistes audio entrelacées dans un format de conteneur approprié. L’élément **AudioTracks** représente une collection de toutes les pistes audio.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **AudioTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |Une piste audio spécifique dans l’élément AssetFile parent. Pour plus d’informations, consultez l’élément AudioTrack. |

## <a name="audiotrack-element"></a><a name="AudioTrack"></a> Élément AudioTrack
Une piste audio spécifique dans l’élément AssetFile parent.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Index de base zéro de cette piste audio. **Remarque :**  Il ne s’agit pas nécessairement du TrackID tel qu’utilisé dans un fichier MP4. |
| **Codec** |**xs:string** |Chaîne du codec de piste audio. |
| **EncoderVersion** |**xs:string** |Chaîne de version d’encodeur facultative, requise pour EAC3. |
| **Canaux**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Nombre de canaux audio. |
| **SamplingRate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Taux d’échantillonnage audio en échantillons/sec ou Hz. |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Débit binaire audio moyen en bits par seconde, tel que calculé à partir du fichier de ressource. Compte uniquement la charge utile de flux élémentaire et n’inclut pas la surcharge de packaging. |
| **BitsPerSample**<br/><br/> minInclusive ="0"<br/><br/> Obligatoire |**xs:int** |Bits par échantillon pour le type de format wFormatTag. |

### <a name="child-elements"></a>Éléments enfants
| Nom | Description |
| --- | --- |
| **LoudnessMeteringResultParameters**<br/><br/> minOccurs="0" maxOccurs="1" |Paramètres de résultat de mesure du niveau sonore. Pour plus d’informations, consultez l’élément LoudnessMeteringResultParameters. |

## <a name="loudnessmeteringresultparameters-element"></a><a name="LoudnessMeteringResultParameters"></a> Élément LoudnessMeteringResultParameters
Paramètres de résultat de mesure du niveau sonore.  

Vous trouverez un exemple de XML [Exemple de XML](#xml).  

### <a name="attributes"></a>Attributs
| Nom | Type | Description |
| --- | --- | --- |
| **DPLMVersionInformation** |**xs:string** |Version du kit de développement **Dolby** Professional Loudness Metering. |
| **DialogNormalization**<br/><br/> minInclusive="-31" maxInclusive="-1"<br/><br/> Obligatoire |**xs:int** |DialogNormalization généré par DPLM, requis lorsque LoudnessMetering est défini |
| **IntegratedLoudness**<br/><br/> minInclusive="-70" maxInclusive="10"<br/><br/> Obligatoire |**xs:float** |Niveau sonore intégré |
| **IntegratedLoudnessUnit**<br/><br/> Obligatoire |**xs:string** |Unité de niveau sonore intégrée. |
| **IntegratedLoudnessGatingMethod**<br/><br/> Obligatoire |**xs:string** |Identificateur de régulation |
| **IntegratedLoudnessSpeechPercentage**<br/><br/> minInclusive ="0" maxInclusive="100" |**xs:float** |Contenu vocal sur le programme, sous forme de pourcentage. |
| **SamplePeak**<br/><br/> Obligatoire |**xs:float** |Valeur d’échantillonnage absolue de crête, depuis la réinitialisation ou depuis la dernière suppression, par canal.  Les unités sont dBFS. |
| **SamplePeakUnit**<br/><br/> fixed="dBFS"<br/><br/> Obligatoire |**xs:anySimpleType** |Exemple d’unité de crête. |
| **TruePeak**<br/><br/> Obligatoire |**xs:float** |Valeur true peak maximale, selon ITU-R BS.1770-2, depuis la réinitialisation ou depuis la dernière suppression, par canal. Les unités sont dBTP. |
| **TruePeakUnit**<br/><br/> fixed="dBTP"<br/><br/> Obligatoire |**xs:anySimpleType** |Unité de true peak. |

## <a name="schema-code"></a>Code du schéma
```xml
<?xml version="1.0" encoding="utf-8"?>  
<xs:schema xmlns:xs="https://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.2"  
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
            targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
            elementFormDefault="qualified">  
  <xs:element name="AssetFiles">  
    <xs:annotation>  
      <xs:documentation>Collection of AssetFile entries for the encoding job</xs:documentation>  
    </xs:annotation>  
    <xs:complexType>  
      <xs:sequence>  
        <xs:element name="AssetFile" minOccurs="1" maxOccurs="unbounded">  
          <xs:annotation>  
            <xs:documentation>asset file</xs:documentation>  
          </xs:annotation>  
          <xs:complexType>  
            <xs:sequence>  
              <xs:element name="Sources">  
                <xs:annotation>  
                  <xs:documentation>Collection of input/source media files, that was processed in order to produce this AssetFile</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="Source" minOccurs="1" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>An input/source file used when generating this asset</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
                        <xs:attribute name="Name" type="xs:string" use="required">  
                          <xs:annotation>  
                            <xs:documentation>input source file name</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                      </xs:complexType>  
                    </xs:element>  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="VideoTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                </xs:annotation>  
                <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="VideoTrack" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
                        <xs:attribute name="Id" use="required">  
                          <xs:annotation>  
                            <xs:documentation>zero-based index of this video track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="FourCC" type="xs:string" use="required">  
                          <xs:annotation>  
                            <xs:documentation>video codec FourCC code</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Profile" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>H264 profile (only appliable for H264 codec)</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Level" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>H264 level (only appliable for H264 codec)</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Width" use="required">  
                          <xs:annotation>  
                            <xs:documentation>encoded video width in pixels</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Height" use="required">  
                          <xs:annotation>  
                            <xs:documentation>encoded video height in pixels</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="DisplayAspectRatioNumerator" use="required">  
                          <xs:annotation>  
                            <xs:documentation>video display aspect ratio numerator</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:double">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="DisplayAspectRatioDenominator" use="required">  
                          <xs:annotation>  
                            <xs:documentation>video display aspect ratio denominator</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:double">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Framerate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>measured video frame rate in .3f format</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:decimal">  
                              <xs:minInclusive value="0"/>  
                              <xs:fractionDigits value="3"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="TargetFramerate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>preset target video frame rate in .3f format</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:decimal">  
                              <xs:minInclusive value="0"/>  
                              <xs:fractionDigits value="3"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Bitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="TargetBitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>target average bitrate for this video track, as requested via the encoding preset, in kilobits per second</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="MaxGOPBitrate">  
                          <xs:annotation>  
                            <xs:documentation>Max GOP average bitrate for this video track, in kilobits per second</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                      </xs:complexType>  
                    </xs:element>  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
              <xs:element name="AudioTracks" minOccurs="0">  
                <xs:annotation>  
                  <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                </xs:annotation>  
                 <xs:complexType>  
                  <xs:sequence>  
                    <xs:element name="AudioTrack" maxOccurs="unbounded">  
                      <xs:annotation>  
                        <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
                      </xs:annotation>  
                      <xs:complexType>  
                        <xs:sequence>  
                          <xs:element name="LoudnessMeteringResultParameters" minOccurs="0" maxOccurs="1">  
                            <xs:annotation>  
                              <xs:documentation>Loudness Metering Result Parameters</xs:documentation>  
                            </xs:annotation>  
                            <xs:complexType>  
                              <xs:attribute name="DPLMVersionInformation" type="xs:string">  
                                <xs:annotation>  
                                  <xs:documentation>Dolby Professional Loudness Metering Development Kit Version</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="DialogNormalization" use="required">  
                                <xs:annotation>  
                                  <xs:documentation> DialogNormalization generated through DPLM, required when LoudnessMetering is set</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:int">  
                                    <xs:minInclusive value="-31"/>  
                                    <xs:maxInclusive value="-1"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudness" use="required">  
                                <xs:annotation>  
                                  <xs:documentation>Integrated loudness</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:float">  
                                    <xs:minInclusive value="-70"/>  
                                    <xs:maxInclusive value="10"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessUnit" use="required" type="xs:string">  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessGatingMethod" use="required" type="xs:string">  
                                <xs:annotation>  
                                  <xs:documentation>Gating identifier</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="IntegratedLoudnessSpeechPercentage">  
                                <xs:annotation>  
                                  <xs:documentation>Speech content over the program, as a percentage.</xs:documentation>  
                                </xs:annotation>  
                                <xs:simpleType>  
                                  <xs:restriction base="xs:float">  
                                    <xs:minInclusive value="0"/>  
                                    <xs:maxInclusive value="100"/>  
                                  </xs:restriction>  
                                </xs:simpleType>  
                              </xs:attribute>  
                              <xs:attribute name="SamplePeak" use="required" type="xs:float">  
                                <xs:annotation>  
                                  <xs:documentation>Peak absolute sample value, since reset or since it was last cleared, per channel.  Units are dBFS.</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="SamplePeakUnit" use="required" fixed="dBFS">  
                              </xs:attribute>  
                              <xs:attribute name="TruePeak" use="required" type="xs:float">  
                                <xs:annotation>  
                                  <xs:documentation>Maximum True Peak value, as per ITU-R BS.1770-2, since reset or since it was last cleared, per channel.  Units are dBTP.</xs:documentation>  
                                </xs:annotation>  
                              </xs:attribute>  
                              <xs:attribute name="TruePeakUnit" use="required" fixed="dBTP">  
                              </xs:attribute>  
                            </xs:complexType>  
                          </xs:element>  
                        </xs:sequence>  
                        <xs:attribute name="Id" use="required">  
                          <xs:annotation>  
                            <xs:documentation>zero-based index of this audio track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Codec" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>audio track codec string</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="EncoderVersion" type="xs:string">  
                          <xs:annotation>  
                            <xs:documentation>optional encoder version string, required for EAC3</xs:documentation>  
                          </xs:annotation>  
                        </xs:attribute>  
                        <xs:attribute name="Channels" use="required">  
                          <xs:annotation>  
                            <xs:documentation>number of audio channels</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="SamplingRate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>audio sampling rate in samples/sec or Hz</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="Bitrate" use="required">  
                          <xs:annotation>  
                            <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                        <xs:attribute name="BitsPerSample" use="required">  
                          <xs:annotation>  
                            <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
                          </xs:annotation>  
                          <xs:simpleType>  
                            <xs:restriction base="xs:int">  
                              <xs:minInclusive value="0"/>  
                            </xs:restriction>  
                          </xs:simpleType>  
                        </xs:attribute>  
                      </xs:complexType>  
                    </xs:element>  
                  </xs:sequence>  
                </xs:complexType>  
              </xs:element>  
            </xs:sequence>  
            <xs:attribute name="Name" type="xs:string" use="required">  
              <xs:annotation>  
                <xs:documentation>the media asset file name</xs:documentation>  
              </xs:annotation>  
            </xs:attribute>  
            <xs:attribute name="Size" use="required">  
              <xs:annotation>  
                <xs:documentation>size of file in bytes</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:long">  
                  <xs:minInclusive value="0"/>  
                </xs:restriction>  
              </xs:simpleType>  
            </xs:attribute>  
            <xs:attribute name="Duration" use="required">  
              <xs:annotation>  
                <xs:documentation>content play back duration</xs:documentation>  
              </xs:annotation>  
              <xs:simpleType>  
                <xs:restriction base="xs:duration"/>  
              </xs:simpleType>  
            </xs:attribute>  
          </xs:complexType>  
        </xs:element>  
      </xs:sequence>  
    </xs:complexType>  
  </xs:element>  
</xs:schema>  
```



## <a name="xml-example"></a><a name="xml"></a> Exemple XML

Le fichier XML suivant est un exemple de fichier de métadonnées de sortie.  

```xml
<AssetFiles xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="https://www.w3.org/2001/XMLSchema"   
            xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata">  
  <AssetFile Name="BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4" Size="4646283" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.2" Width="1280" Height="720" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="4250" TargetBitrate="3400" MaxGOPBitrate="5514"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4" Size="3166728" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
      <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="2846" TargetBitrate="2250" MaxGOPBitrate="3630"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4" Size="2205095" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1932" TargetBitrate="1500" MaxGOPBitrate="2513"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4" Size="1508567" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1271" TargetBitrate="1000" MaxGOPBitrate="1527"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4" Size="1057155" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="843" TargetBitrate="650" MaxGOPBitrate="1086"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4" Size="699262" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <VideoTracks>  
      <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="1.3" Width="320" Height="180" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="503" TargetBitrate="400" MaxGOPBitrate="661"/>  
    </VideoTracks>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_AAC_und_ch2_96kbps.mp4" Size="166780" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
  <AssetFile Name="BigBuckBunny_AAC_und_ch2_56kbps.mp4" Size="124576" Duration="PT8.4288444S">  
    <Sources>  
      <Source Name="BigBuckBunny.mp4"/>  
    </Sources>  
    <AudioTracks>  
      <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="53" BitsPerSample="16"/>  
    </AudioTracks>  
  </AssetFile>  
</AssetFiles>  
```

## <a name="next-steps"></a>Étapes suivantes
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
