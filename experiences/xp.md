# Les expériences de mon TIPE

## I. Modéliser un nuage en mouvement et calcul de densité

L'un des premiers enjeux du TIPE est de modéliser ce que j'appelle la « densité » d'un nuage, comme on peut le voir sur certains visuels météorologiques avec plusieurs zones : 

<div align="center"> <img src="https://gray-wvue-prod.gtv-cdn.com/resizer/v2/M6CBCU4GUJBEVOJBB5LVTUSOQU.png?auth=bae44e8a628dea8ec3f69868e51c920a2c8d764d79c3907549d808874ffd2a6c&width=1600&height=900&smart=true" alt="Katrina Hurricane" width="600"/> </div>
<div align="center">  Image de l'ouragan Katrina, Fox 8 </div>

### Expérience I.1 - Calcul intégral
Pour cela, une conception du problème (très simplifiée) est de voir cela comme un calcul intégral : **on calcule la surface du nuage** puis on procède au raisonnement suivant, pour une quantité donnée « d'accumulation » (on peut parler de particules ici) plus la surface couverte par le nuage est grande, moins la densité est élevée (cela s'apparenterait à du bleu sur la carte). À l'inverse, si le nuage couvre une très petite zone avec la même accumulation, sa densité sera très élevée (zone très rouge sans ce cas).

Une fois cela dit, on peut ensuite dire que un phénomène météorologique (comme l'ouragan) est une superposition de nuages à densités différentes (on voit bien sur l'image de l'ouragan Katrina que le phénomène n'est pas uniquement bleu ou rouge).

★ L'un des gros problèmes de ce calcul intégral est qu'on repère un nuage par des coordonnées de points (cf. images ci-dessous), et que ce n'est pas une forme connue (si on avait eu un cercle ça aurait été beaucoup plus simple). On va alors procéder par une _méthode de Monte Carlo_ pour en déduire une densité approximative. En guise d'exemple simple, prenons un cercle pour commencer : 

<div align="center"> <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/20/MonteCarloIntegrationCircle.svg/440px-MonteCarloIntegrationCircle.svg.png" alt="Méthode de Monte Carlo pour calculer l'aire d'un cercle" width="400"/> </div>
<div align="center">  Méthode de Monte Carlo pour calculer l'aire d'un cercle </div>
<br />

Bon, et maintenant pour calculer l'aire d'une forme quelconque repérée par un ensemble de points, c'est exactement la même chose ! 
J'ai donc procédé à des calculs d'intégrales à l'aide de cette méthode en munissant le nuage de la même accumulation à chaque fois : 
<br />
<div align="center"> <img src="/experiences/img/contract_bary_f_0_4.png" alt="Contraction barycentre" width="400"/> </div>
<div align="center">  XP I.1.A : Deux nuages possédant la même accumulation mais pas la même densité, contraction vers le barycentre (f=0.4) </div>
<br />
💡 Le paramètre f représente la _contraction_ vers le barycentre : plus on contracte, plus la densité augmente.

On peut d'ailleurs faire autre chose qu'une contraction vers le centre, pour vérifier que notre modèle de calcul fonctionne : 
<div align="center"> <img src="/experiences/img/contract_f_0_2.png" alt="Contraction bord droit" width="400"/> </div>
<div align="center">  XP I.1.B : Deux nuages possédant la même accumulation mais pas la même densité, contraction vers le bord droit (f=0.2)</div>
<br />
<div align="center"> <img src="/experiences/img/contract_f_0_4.png" alt="Contraction bord droit" width="400"/> </div>
<div align="center">  XP I.1.C : Deux nuages possédant la même accumulation mais pas la même densité, contraction vers le bord droit  (f=0.4)</div>
<br />
<div align="center"> <img src="/experiences/img/contract_f_0_6.png" alt="Contraction bord droit" width="400"/> </div>
<div align="center">  XP I.1.D : Deux nuages possédant la même accumulation mais pas la même densité, contraction vers le bord droit   (f=0.6)</div>
<br />


## II. Nuage en mouvement : Principe de la _gradient descent_
L'algorithme de descente du gradient est le suivant : à une fonction de coût $`c`$ non nulle, on associe à $`f`$ son inverse ($`f=1/c`$). Dès lors, comme le gradient de $`f`$ nous dirige vers là où $`f`$ serait maximale (du moins localement), il nous indique (par inverse) là où $`c`$ serait minimale, d'où le coût minimisé.

Le principe est donc de suivre localement là où le gradient de $`f`$ nous oriente afin de réaliser un coût total minimal.
## II.A - Déplacement optimal du nuage à l'aide du gradient : phase 1 (point par point)

Techniquement, on pourrait dire qu'il s'agit juste de déplacer chaque point d'une coordonnée $`X_{\text{init}} = (x_1,y_1)`$ vers une coordonnée $`X_{\text{final}} = (x_2,y_2)`$. On considèrera donc que les transitions se feront entre deux états du même nuage avec le même nombre de points. 

Dans un premier temps, on réalisera des trajectoires rectilignes car on fonctionne avec une distance euclidienne (on visualise bien le chemin le plus court entre deux points de $`\mathbb{R}^2`$ comme le segment les reliant.

Ainsi, on prendra comme _fonction de coût_ (cf. brouillons et présentation pour + d'explications) la distance euclidienne mélée à la direction vers l'arrivée (le gradient indiquera ainsi bien la direction où la distance sera minimale et dirigera vers l'arrivée) : 

<div align="center"> <img src="/experiences/img/transport_nuage_simple.gif" alt="Transport nuage simple" width="400"/> </div>
<div align="center">  XP II.1.A : Simple déplacement entre deux nuages circulaires de même rayon </div>


<br />

📝 Une chose se remarque assez rapidement, on ne retrouve pas un déplacement en bloc de manière rectiligne (i.e tous les points subissent la même translation), **chaque point poursuit son trajet en fonction de la valeur de son gradient**, et ça pose certains problèmes : chaque point agit pour l'optimalité de son déplacement à lui, pas pour celui du corps entier. En effet, quand ce dernier fait face à un obstacle (cf. XP II.1.B), les points vont suivre leur propre gradient et **cela amène à une division du nuage** (ce que l'on voulait absolument éviter !!)

<div align="center"> <img src="/experiences/img/transport_nuage_divisible.gif" alt="Nuage qui se divise" width="400"/> </div>
<div align="center">  XP II.1.B : le nuage se divise en deux en faisant face à un obstacle </div>


<br />
📝 Pour pallier ce problème, **on pourrait imaginer un barycentre**, qui donnerait une direction principale pour le corps, et ensuite que les points suivent + ou - cette direction en fonction de leur gradient. Mais dans les cas extrêmes, cela ne changerait rien (le nuage finirait quand même par se diviser), il faut donc imaginer une sorte de limite dans les écarts entre les points pour éviter la situation visible en _XP II.1.C_ : 

<div align="center"> <img src="/experiences/img/ecart_points.png" alt="Points très écartés" width="400"/> </div>
<div align="center">  XP II.1.C : Points très écartés -> situation à éviter </div>


<br />

