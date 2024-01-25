<br />
<p align="center">
  <h1 align="center"> Polygonisation de batiments </h1>  
  </p>
</p>

<p align="justify">
plusieurs méthodes utilisent le post-processing sur le masque de segmentation pour prédire les polygones qui délimitent les objets. Cependant, du fait du déséquilibre des classes, la segmentation sémantique présente souvent des défauts au niveau de la  prédiction des contours des objets (imprécisions ...). Ainsi, des méthodes qui se basent sur la segmentation sémantique pour extraire les noeuds ou les contours des objets souffrent d'imprécisions. Pour contrer les problèmes dus aux déséquilibres des classes, l'apprentissage d'une fonction distance TSD , permettant une meilleur  modélisation des contours des objets  est utilisée pour extraire d'une part le masque de segmentation et d'autre part la carte des vertex. Ceci permet à la fois une meilleur prédiction du masque de segmentation et des noeuds.
l'approche implémenté permet une  construction directe des  polygones.  

</p>

### 1. Les données: 
#### a) Les données crowdAI Mapping Challenge:

<p align="justify">
Les données utilisées sont  issues du crowdAI Mapping Challenge, il s’agit d’images satellitaires de dimension 300*300 pixels et  annotées au format MS-COCO . Toute l’étude a été conduite en sélectionnant 1000 images  du jeux de données mise à disposition. Les images  utilisées  ont été découpées comme suit: 70 % pour l’entrainement du modèle et 15 % pour la validation et le test. Les images utilisées ainsi que les masques de segmentation et les masques des polygones ont été redimensionnés à 224*224 en utilisant la transformation linéaire. 

</p>

#### b) La carte des distances TSD et la carte de densité:

<p align="justify">
L’implémentation de la méthode des auteurs (Xu et al ) a nécessité le calcul de la carte biaisé TSD et la matrice de densité des vertex pour chaque image.

</p>

#### Carte de distance biaisé TSD :
<p align="justify">
Similaire à la carte de segmentation, elle contient une  information supplémentaire relative à la  limite des objets, ici les  bâtiments  en spécifiant les pixels les plus proches des contours. La distance biaisé TSD associe un score positif de 2 à un pixel à l’intérieur du bâtiment et un score négatif de -2  à chaque pixel à l’extérieur du bâtiments (figure 1 ).  La valeur de  l’hyperparamettre  quant à elle va conditionner le voisinage des limites des bâtiments (figure 1 ).

La fonction de distance TSD est définie comme suit (figure 2): Db (p) est la distance euclidienne entre le pixel p et le pixel le plus proche le long du contour de l’objet  b, et τ est un hyperparamètre qui contrôle la troncature de la distance signée.

L’apprentissage de cette distance encourage le modèle à apprendre les contours des bâtiments. 

La carte de distance TSD a été normalisée en 0 pour obtenir des valeurs comprises entre -1 et 1 et centrées sur 0.
</p>

image  |masque   | τ = 1    | τ = 3  
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
 <img width="368" alt="Capture d’écran 2024-01-21 à 19 32 56" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/428eb1c3-4631-4366-aed3-013e9938f1d0">|  <img width="368" alt="Capture d’écran 2024-01-21 à 19 33 04" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/1db1d0fe-c539-4d12-8100-835188c7860d"> |   <img width="368" alt="Capture d’écran 2024-01-21 à 19 29 20" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/c31e473e-01df-4a0a-9e3b-d29a943da27f"> | <img width="368" alt="Capture d’écran 2024-01-21 à 19 29 25" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/d49221af-85ad-4e6f-9ad2-e0aab1eb790d">

 τ =   6|τ =  10 | τ =  20  | τ =   30 
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
<img width="368" alt="Capture d’écran 2024-01-21 à 19 29 25 2" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/a90f4891-e195-42de-a7fe-c1477c63ba1d">  |  <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 07" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/5f8d443a-033d-4da4-b26d-b4b7b73686e2">|  <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 22" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/20cad6f8-63af-497c-87c0-c7d1e83e82cc"> | <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 37" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/f441b744-2faa-4d19-84f9-232423a8a274">


#### Carte de densité des vertex: 

<p align="justify">
Elle représente la réponse maximale de chaque pixel après application d’un filtre gaussaien 2D (figure 3)  à la carte des vertex binaires. La carte de densité permet une meilleure représentation des vertex que celle obtenue par la carte binaire des vertex, en associant un signal aux pixels et à ses voisins (figure 4). Cette carte associe un signal maximal aux sommets des bâtiments et définit un voisinage auquel elle associe un signal non nul. La définition du voisinage des vertex dépend de la valeur de sigma, plus celle-ci est élevée et plus le voisinage augmente.
  
</p>

<p align="justify">
La carte de densité a été normalisée pour obtenir des valeurs entre 0 et 1.
La définition de ces deux métriques permet de répondre aux limites que soulève  l’apprentissage d’un masque de segmentation, notamment du fait du déséquilibre des classes. Cette limite est d’autant plus exacerbée lors de l’apprentissage d’un masque de contours des bâtiments et des vertex. Ainsi, la matrice de TSD est utilisée pour forcer le modèle à apprendre les contours des bâtiments. Ce qui permet une meilleure précision des segmentations obtenues ainsi que de la position des vertex.
  
</p>

σ = 1   | σ = 4| σ = 8   | σ = 16  | σ =   32
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
<img width="302" alt="Capture d’écran 2024-01-21 à 19 27 50" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/36927803-3699-4c36-a283-409273916693"> |  <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 03" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/90fa8f29-7f01-475b-90b2-18d8651a8b0d">| <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 19" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/971ed6b6-5c02-4338-93ab-fbd574a52a44"> | <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 27" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/71372b1d-b23d-4dd7-aab1-b2b1966e0cbd"> | <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 45" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/7d06d5db-c504-464b-98ba-b78739e8ccd1">


#### 2.  La méthode :


#### a) méthode  de polygonisation directe:


<img width="788" alt="Capture d’écran 2024-01-24 à 20 37 21" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/87e679b8-ecd0-47d5-b3e4-a3e3c798600d">

<p align="justify">
La méthode de polygonisation directe proposée par  de Xu et.al a été choisi en raison de la simplicité du modèle et de son efficacité ( elle permet l’analyse d’une scène entière en 1 fois).  Les auteurs traitent le problème  comme un problème de régression  via un double-branched network pour contrer le déséquilibre de classes rencontré lors de l’apprentissage de la segmentation et de la matrice binaire des vertex ou des contours. Le modèle développé par les auteurs prédit la segmentation et la carte de densité des vertex à partir de la carte de distance TSD.
Dans un premier temps les auteurs utilisent un modèle composé de 3 régresseurs, le premier permettant de prédire la carte des TSD à partir des caractéristiques extraites ( via un backebones).  A partir de cela,  deux régresseurs sont utilisés pour extraire les informations relatives aux vertex (carte de densité, branche 1  ) et à la segmentation (branche 2). La carte de segmentation prédite constitue le polygone initial.
  
</p>

<p align="justify">
La deuxième phase consiste à affiner le polygone initial en utilisant la carte de densité prédite et le polygone initial définit par la carte de segmentation. Chaque vertex du polygone initial est remplacé par le vertex voisin qui possède la réponse la plus  élevée dans la carte de densité. Pour cela, un voisinage autour du vertex est défini. Puis les vertex sont filtrés permettant de garder uniquement les vertex qui ont une réponse supérieure au seuil. 
Ainsi, la construction des polygones repose à la fois sur la segmentation prédite, qui permet de définir le polygone initial et sur la capacité du modèle à prédire la carte de densité, puisqu’un seuil trop stringent et un voisinage faible applique un filtre trop strict sur les vertex.  Dans le cadre de cette étude, nous avons choisi d’utiliser un seuil de binarisation du masque de segmentation à 0.7. Les différents paramètres tels que le nombre de voisins et les différents seuils de réponse seront présentés lors de l’analyse des résultats.
  
</p>

#### b) Structure du modèle:


<img width="788" alt="Capture d’écran 2024-01-24 à 20 36 30" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/081c2f17-aa3d-4ef1-ad2d-9fd0752fab6a">

<p align="justify">
- L’extracteur des caractéristiques est un backbone avec une architecture  U-net et encodeur de type ResNet-101 pré-entraîné sur ImageNet et renvoie une carte des caractéristiques de profondeur 128.
  
<p>

<p align="justify"> 
- Le  régresseur  de la carte de TSD utilise un réseau de régression séquentiel qui commence par une couche de convolution suivie d'une normalisation par lot et d'une activation ReLU. Il intègre ensuite un bloc résiduel pour approfondir le réseau sans augmenter la complexité, suivi par une autre couche de convolution avec normalisation, activation ReLU, et un dropout élevé ( 0.4) pour la régularisation. Finalement, le réseau se termine par une couche de convolution qui réduit les canaux à une seule sortie avec une activation Tanh pour prédire une carte de distance normalisée entre -1 et 1.
  
<p>

<p align="justify">  
- La branche 1 du modèle utilisée pour prédire la carte de densité, débute avec une couche de convolution réduisant les canaux d'entrée de 129 à 64, suivie d'une normalisation par lots et d'une activation ReLU. Après un bloc résiduel et une couche de dropout (0.4) pour la régularisation, une autre couche de convolution réduit encore les canaux de 64 à 1, et la fonction d'activation Sigmoid est appliquée à la sortie pour produire une probabilité entre 0 et 1 pour chaque pixel.
  
<p>

<p align="justify">   
- La branche 2 utilise une architecture  similaire à la branche 1.
  
<p>
  
#### c)L’apprentissage du modèle :

<p align="justify">
Pour l’apprentissage du modèle, l’erreur quadratique moyenne (MSE) est utilisée pour l‘apprentissage et de la TSD et la carte de densité.  Pour l’apprentissage de la carte de densité, des facteurs de normalisations ont été introduits pour réduire le problème lié au déséquilibre de classes. Dans le cas de la fonction loss associé à la densité, le premier terme représente l’erreur quadratique moyenne pour les pixels présentant un signal positive et le dixième terme correspond à l’erreurs quadratique moyenne pour les pixels présentant un signal nul.
L’entropie  croisée binaire a  est utilisé pour l’apprentissage de la segmentation.
Des facteur alpha , beta et gamma sont respectivement associés aux fonctions pertes de la carte de la TSD , de la carte de densité des vertex et de segmentation.

Nous avons utilisé 2 combinaisons de poids associées aux fonctions pertes:
Alph = 1 , beta = 1 et gamma = 1
Alph = 0.6 , beta = 1 et gamma = 0.4 : ici le poids maximum est associé à la fonction perte de la carte des densité. Cette combinaison de paramètre est utilisée pour forcer notre modèle à apprendre la carte des densités.

Dans ce qui suit, l’apprentissage du modèle s’est fait sur 30 époques et en définissant un Batch size de 3.

<p>
  
#### 3.Résultats:

<p align="justify">
Pour évaluer la capacité du modèle à construire les polygones , nous avons utilisé le score IoU qui représente le rapport entre l'intersection et l'union des polygones prédits par le modèle et les polygones annotés dans la base de données (vérité terrain) par image analysé . Puis une moyenne de ses scores a été faite pour obtenir le score IoU moyen par image analysé .
Les scores IoU sont calculés pour des voisinages différents  ( 5,10,15 voisins ) et des seuils de réponses différents ( 0.5,0.7).
La qualité de l’apprentissage du modèle  et la généralisation  est discutée  respectivement par l’étude des courbes de fonctions pertes sur les données d’apprentissages et de validations au cours des époques. 
<p>


<img align="right" width="300" alt="Capture d’écran 2024-01-24 à 20 44 40" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/35b2ef64-4c62-4d4d-aceb-e36950bfe5a0">

<p align="justify">
L’analyse des courbes loss pour un poids de 1 associé aux fonctions pertes de la carte de TSD, la densité et à la     segmentation, montre une décroissance de la fonction perte sur les données train, synonyme de l’apprentissage du modèle (figure 5). Cependant on voit que notre modèle est sujet au sur-apprentissage (figure 5), puisque la fonction perte obtenu sur les données validations présente un écart avec celle obtenue sur les données d’apprentissages.
<p>



 F1 (segmentation) |   IoU  (seuil: 0.7, voisinage: 5) |  IoU (seuil: 0.7, voisinage: 10)  | IoU (seuil: 0.5, voisinage: 10)
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
0.75|0.0031|0.0113|0.48|


<p align="justify">
L’analyse de la capacité du modèle à prédire les polygones montre que le modèle est incapable de prédire les polygones  en cas de stringence à la fois sur le  voisinage et la valeur de la densité associée aux vertex du polygone initial ( défini à partir de la segmentation prédite). Cependant , même si une augmentation du voisinage et une diminution du seuil de réponse permettent une meilleure prédiction des polygones, ses résultats restent insuffisant.
  
<p>

<p align="justify">
L’étude de la qualité de la segmentation via le f1 score ( qui est une moyenne harmonique de la précision et du rappel ) montre un score élevé de 0.75, témoignant ainsi de la capacité du modèle à prédire correctement le masque de segmentation à partir de la carte des distances TSD prédites. Donc  les capacités limités du modèle à prédire les polygones résident dans le défaut de prédiction de la carte de densité des vertex.

<p>
  
<img align="right" width="300" alt="Capture d’écran 2024-01-24 à 20 50 08" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/54875262-29e7-48cb-94c7-b1866c3f28ff">
<p align="justify">
Pour forcer le modèle à apprendre la carte de densité des vertex, nous allons diminuer les poids associés à la fonction perte de la carte des TSD  (alpha) et du masque de segmentation (gamma). 

<p>
  
Hyperparamètres  | F1 (segmentation) |   IoU  (seuil: 0.7, voisinage: 5) |  IoU (seuil: 0.7, voisinage: 10)  | IoU (seuil: 0.5, voisinage: 10)
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
Alpha =  1, beta = 1, gamma = 1|0.74|0.0031|0.0113|0.48|
Alpha =  0.6, beta = 1, gamma = 0.4|0.70|0.01|0.06|0.57|

<p align="justify">
Le modèle prédit correctement les masques de segmentations ( f1 score de 0.70) et l’analyse des scores IoU montre une augmentation de la qualité des polygones prédits ( 0.003 contre 0.0129 pour un voisinage de 5 pixels et un seuil de réponse de 0.7). Cependant, ses résultats ne sont pas suffisamment élevés pour conclure quant à la capacité du modèle à prédire la carte de densité.
L’analyse de la courbe de perte montre que le modèle fait du sur-apprentissage, pour limiter ce phénomène, nous avons ajouté une couche de droupout. 

<p>

 Without    Dropout|  Dropout sur le régresseur de la densité (p =  0.05) | Dropout sur le régresseur de la TSD et de la  densité (p =  0.4) | Dropout sur le régresseur de la TSD (p = 0.6) et de la  densité (p =  0.4)
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
<img width="199" alt="Capture d’écran 2024-01-24 à 21 23 40" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/4d786eb6-61e9-42ca-810f-5b0c34757be2">|<img width="194" alt="Capture d’écran 2024-01-24 à 21 23 55" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/f7ff42c7-2813-41a6-9ee7-89e9ccd17f71">|<img width="199" alt="Capture d’écran 2024-01-24 à 21 24 12" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/982add9b-1107-4af0-91d8-6954e4c2babc">|<img width="199" alt="Capture d’écran 2024-01-24 à 21 24 20" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/b6f41013-f468-4cf7-8495-3ea561985d99">|



<p align="justify">
Nous avons effectué les tests suivants: 
- L’ajout d’une couche de droupout avec une probabilité de 0.05 à la branche du modèle qui prédit la carte des densités: on a une légère diminution du sur-apprentissage visible.
- L’ajout d’une couche de droupout avec une probabilité de 0.4 à la branche du modèle qui prédit la carte des densités et à la branche qui prédit la TSD : on remarque cette fois-ci une diminution globale du sur-apprentissage.
- L’ajout d’une couche de droupout avec une probabilité de 0.6 et 0.4 respectivement aux  branches du régresseur de la carte des densités et de la TSD : les capacités d’apprentissages du modèle sont légèrement détériorées ( augmentation de la loss sur les données d’apprentissages  à partir de la 10 ème époque ).

<p>
  
Hyperparamètres  | F1 (segmentation) |   IoU  (seuil: 0.7, voisinage: 5) |  IoU (seuil: 0.7, voisinage: 10)  | IoU (seuil: 0.5, voisinage: 10)
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
Alpha =  0.6, beta = 1, gamma = 0.4|0.70|0.01|0.06|0.57|
Alpha =  0.6, beta = 1, gamma = 0.4|0.72|0,001|0.01|0.47|

<p align="justify">
L’ajout d’une couche de droupout avec une probabilité de 0.4 à la branche du modèle qui prédit la carte des densité et à la branche qui prédit la TSD permet donc une diminution du sur-apprentissage sans altérer l’apprentissage du modèle. L’analyse du f1 score montre une bonne capacité du modèle à prédire les masques de segmentation ( 0.721) cependant, les scores IoU des polygones prédits ne suggèrent pas une amélioration de la capacité du modèle à prédire les polygones.

<p>


### Citation
La méthode  implementé se base sur l’article suivant:
```bibtex
@INPROCEEDINGS{10144146,
  author={Xu, Yajin and Schuegraf, Philipp and Bittner, Ksenia},
  booktitle={2023 Joint Urban Remote Sensing Event (JURSE)}, 
  title={Vertex Aided Building Polygonization from Satellite Imagery Applying Deep Learning}, 
  year={2023},
  volume={},
  number={},
  pages={1-4},
  keywords={Deep learning;Satellites;Image recognition;Buildings;Complexity theory;Convolutional neural networks;Task analysis;building vectorization;building extraction;pattern recognition;deep learning},
  doi={10.1109/JURSE57346.2023.10144146}}
```
