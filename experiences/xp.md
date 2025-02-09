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


### Expérience I.2 - Modéliser un nuage en mouvement

Techniquement, on pourrait dire qu'il s'agit juste de déplacer chaque point d'une coordonnée $`X_{\text{init}} = (x_1,y_1)`$ vers une coordonnée $`X_{\text{final}} = (x_2,y_2)`$. On considèrera donc que les transitions se feront entre deux états du même nuage avec le même nombre de points. 

Dans un premier temps, on réalisera des trajectoires rectilignes car on fonctionne avec une distance euclidienne (on visualise bien le chemin le plus court entre deux points de $`\mathbb{R}^2`$ comme le segment les reliant. Ainsi, on prendra comme _fonction de coût_ (cf. brouillons et présentation pour + d'explications) la distance euclidienne mélée à la direction vers l'arrivée (le gradient indiquera ainsi bien la direction où la distance sera minimale et dirigera vers l'arrivée) : 

