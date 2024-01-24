<br />
<p align="center">
  <h1 align="center">🔭 Polygonisation de batiments </h1>
  #<h3 align="center">A probing benchmark for spatial undersranding of vision-language models.</h3>
  
  </p>
</p>



### 1. Les données: 
#### a) Les données crowdAI Mapping Challenge:

Les données utilisées sont  issues du crowdAI Mapping Challenge, il s’agit d’images satellitaires de dimension 300*300 pixels et  annotées au format MS-COCO . Toute l’étude a été conduite en sélectionnant 1000 images  du jeux de données mise à disposition. Les images  utilisées  ont été découpées comme suit: 70 % pour l’entrainement du modèle et 15 % pour la validation et le test. Les images utilisées ainsi que les masques de segmentation et les masques des polygones ont été redimensionnés à 224*224 en utilisant la transformation linéaire.


#### b) La carte des distances TSD et la carte de densité:

L’implémentation de la méthode des auteurs (Xu et al ) a nécessité le calcul de la carte biaisé TSD et la matrice de densité des vertex pour chaque image.
#### Carte de distance biaisé TSD :
Similaire à la carte de segmentation, elle contient une  information supplémentaire relative à la  limite des objets, ici les  bâtiments  en spécifiant les pixels les plus proches des contours. La distance biaisé TSD associe un score positif de 2 à un pixel à l’intérieur du bâtiment et un score négatif de -2  à chaque pixel à l’extérieur du bâtiments (figure 1 ).  La valeur de  l’hyperparamettre  quant à elle va conditionner le voisinage des limites des bâtiments (figure 1 ).

La fonction de distance TSD est définie comme suit (figure 2): Db (p) est la distance euclidienne entre le pixel p et le pixel le plus proche le long du contour de l’objet  b, et τ est un hyperparamètre qui contrôle la troncature de la distance signée.

L’apprentissage de cette distance encourage le modèle à apprendre les contours des bâtiments. 

La carte de distance TSD a été normalisée en 0 pour obtenir des valeurs comprises entre -1 et 1 et centrées sur 0.


image  |masque   | τ = 1    | τ = 3  
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
 <img width="368" alt="Capture d’écran 2024-01-21 à 19 32 56" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/428eb1c3-4631-4366-aed3-013e9938f1d0">|  <img width="368" alt="Capture d’écran 2024-01-21 à 19 33 04" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/1db1d0fe-c539-4d12-8100-835188c7860d"> |   <img width="368" alt="Capture d’écran 2024-01-21 à 19 29 20" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/c31e473e-01df-4a0a-9e3b-d29a943da27f"> | <img width="368" alt="Capture d’écran 2024-01-21 à 19 29 25" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/d49221af-85ad-4e6f-9ad2-e0aab1eb790d">

 τ =   6|τ =  10 | τ =  20  | τ =   30 
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
<img width="368" alt="Capture d’écran 2024-01-21 à 19 29 25 2" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/a90f4891-e195-42de-a7fe-c1477c63ba1d">  |  <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 07" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/5f8d443a-033d-4da4-b26d-b4b7b73686e2">|  <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 22" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/20cad6f8-63af-497c-87c0-c7d1e83e82cc"> | <img width="368" alt="Capture d’écran 2024-01-21 à 19 30 37" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/f441b744-2faa-4d19-84f9-232423a8a274">


#### Carte de densité des vertex: 
Elle représente la réponse maximale de chaque pixel après application d’un filtre gaussaien 2D (figure 3)  à la carte des vertex binaires. La carte de densité permet une meilleure représentation des vertex que celle obtenue par la carte binaire des vertex, en associant un signal aux pixels et à ses voisins (figure 4). Cette carte associe un signal maximal aux sommets des bâtiments et définit un voisinage auquel elle associe un signal non nul. La définition du voisinage des vertex dépend de la valeur de sigma, plus celle-ci est élevée et plus le voisinage augmente. 
La carte de densité a été normalisée pour obtenir des valeurs entre 0 et 1.
La définition de ces deux métriques permet de répondre aux limites que soulève  l’apprentissage d’un masque de segmentation, notamment du fait du déséquilibre des classes. Cette limite est d’autant plus exacerbée lors de l’apprentissage d’un masque de contours des bâtiments et des vertex. Ainsi, la matrice de TSD est utilisée pour forcer le modèle à apprendre les contours des bâtiments. Ce qui permet une meilleure précision des segmentations obtenues ainsi que de la position des vertex.

σ = 1   | σ = 4| σ = 8   | σ = 16  | σ =   32
:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:|:-------------------------:
<img width="302" alt="Capture d’écran 2024-01-21 à 19 27 50" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/36927803-3699-4c36-a283-409273916693"> |  <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 03" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/90fa8f29-7f01-475b-90b2-18d8651a8b0d">| <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 19" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/971ed6b6-5c02-4338-93ab-fbd574a52a44"> | <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 27" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/71372b1d-b23d-4dd7-aab1-b2b1966e0cbd"> | <img width="302" alt="Capture d’écran 2024-01-21 à 19 28 45" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/7d06d5db-c504-464b-98ba-b78739e8ccd1">


#### 2.  La méthode :


#### a) méthode  de polygonisation directe:

<img width="788" alt="Capture d’écran 2024-01-24 à 20 37 21" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/87e679b8-ecd0-47d5-b3e4-a3e3c798600d">

La méthode de polygonisation directe proposée par  de Xu et.al a été choisi en raison de la simplicité du modèle et de son efficacité ( elle permet l’analyse d’une scène entière en 1 fois).  Les auteurs traitent le problème  comme un problème de régression  via un double-branched network pour contrer le déséquilibre de classes rencontré lors de l’apprentissage de la segmentation et de la matrice binaire des vertex ou des contours. Le modèle développé par les auteurs prédit la segmentation et la carte de densité des vertex à partir de la carte de distance TSD.
Dans un premier temps les auteurs utilisent un modèle composé de 3 régresseurs, le premier permettant de prédire la carte des TSD à partir des caractéristiques extraites ( via un backebones).  A partir de cela,  deux régresseurs sont utilisés pour extraire les informations relatives aux vertex (carte de densité, branche 1  ) et à la segmentation (branche 2). La carte de segmentation prédite constitue le polygone initial.

La deuxième phase consiste à affiner le polygone initial en utilisant la carte de densité prédite et le polygone initial définit par la carte de segmentation. Chaque vertex du polygone initial est remplacé par le vertex voisin qui possède la réponse la plus  élevée dans la carte de densité. Pour cela, un voisinage autour du vertex est défini. Puis les vertex sont filtrés permettant de garder uniquement les vertex qui ont une réponse supérieure au seuil. 
Ainsi, la construction des polygones repose à la fois sur la segmentation prédite, qui permet de définir le polygone initial et sur la capacité du modèle à prédire la carte de densité, puisqu’un seuil trop stringent et un voisinage faible applique un filtre trop strict sur les vertex.  Dans le cadre de cette étude, nous avons choisi d’utiliser un seuil de binarisation du masque de segmentation à 0.7. Les différents paramètres tels que le nombre de voisins et les différents seuils de réponse seront présentés lors de l’analyse des résultats.

#### b) Concernant la structure du modèle:

<img width="788" alt="Capture d’écran 2024-01-24 à 20 36 30" src="https://github.com/hajarVMI/Projet-Vision/assets/157238036/081c2f17-aa3d-4ef1-ad2d-9fd0752fab6a">

- L’extracteur des caractéristiques est un backbone avec une architecture  U-net et encodeur de type ResNet-101 pré-entraîné sur ImageNet et renvoie une carte des caractéristiques de profondeur 128.

- Le  régresseur  de la carte de TSD utilise un réseau de régression séquentiel qui commence par une couche de convolution suivie d'une normalisation par lot et d'une activation ReLU. Il intègre ensuite un bloc résiduel pour approfondir le réseau sans augmenter la complexité, suivi par une autre couche de convolution avec normalisation, activation ReLU, et un dropout élevé ( 0.4) pour la régularisation. Finalement, le réseau se termine par une couche de convolution qui réduit les canaux à une seule sortie avec une activation Tanh pour prédire une carte de distance normalisée entre -1 et 1.
  
-La branche 1 du modèle utilisée pour prédire la carte de densité, débute avec une couche de convolution réduisant les canaux d'entrée de 129 à 64, suivie d'une normalisation par lots et d'une activation ReLU. Après un bloc résiduel et une couche de dropout (0.4) pour la régularisation, une autre couche de convolution réduit encore les canaux de 64 à 1, et la fonction d'activation Sigmoid est appliquée à la sortie pour produire une probabilité entre 0 et 1 pour chaque pixel.

- La branche 2 utilise une architecture  similaire à la branche 1.

  #### c)L’apprentissage du modèle :

  Pour l’apprentissage du modèle, l’erreur quadratique moyenne (MSE) est utilisée pour l‘apprentissage et de la TSD et la carte de densité.  Pour l’apprentissage de la carte de densité, des facteurs de normalisations ont été introduits pour réduire le problème lié au déséquilibre de classes. Dans le cas de la fonction loss associé à la densité, le premier terme représente l’erreur quadratique moyenne pour les pixels présentant un signal positive et le dixième terme correspond à l’erreurs quadratique moyenne pour les pixels présentant un signal nul.
L’entropie  croisée binaire a  est utilisé pour l’apprentissage de la segmentation.
Des facteur alpha , beta et gamma sont respectivement associés aux fonctions pertes de la carte de la TSD , de la carte de densité des vertex et de segmentation.

Nous avons utilisé 2 combinaisons de poids associées aux fonctions pertes:
Alph = 1 , beta = 1 et gamma = 1
Alph = 0.6 , beta = 1 et gamma = 0.4 : ici le poids maximum est associé à la fonction perte de la carte des densité. Cette combinaison de paramètre est utilisée pour forcer notre modèle à apprendre la carte des densités.

Dans ce qui suit, l’apprentissage du modèle s’est fait sur 30 époques et en définissant un Batch size de 3.

#### 3.Résultats:

