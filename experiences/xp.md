# Les expériences de mon TIPE

## I. Modéliser un nuage en mouvement et calcul de densité

L'un des premiers enjeux du TIPE est de modéliser ce que j'appelle la « densité » d'un nuage, comme on peut le voir sur certains visuels météorologiques avec plusieurs zones : 

<div align="center"> <img src="https://gray-wvue-prod.gtv-cdn.com/resizer/v2/M6CBCU4GUJBEVOJBB5LVTUSOQU.png?auth=bae44e8a628dea8ec3f69868e51c920a2c8d764d79c3907549d808874ffd2a6c&width=1600&height=900&smart=true" alt="Katrina Hurricane" width="600"/> </div>
<div align="center">  Image de l'ouragan Katrina, Fox 8 </div>

### Expérience I.1 - Calcul intégral
Pour cela, une conception du problème (très simplifiée) est de voir cela comme un calcul intégral : **on calcule la surface du nuage** puis on procède au raisonnement suivant, pour une quantité donnée « d'accumulation » (on peut parler de particules ici) plus la surface couverte par le nuage est grande, moins la densité est élevée (cela s'apparenterait à du bleu sur la carte). À l'inverse, si le nuage couvre une très petite zone avec la même accumulation, sa densité sera très élevée (zone très rouge sans ce cas).

Une fois cela dit, on peut ensuite dire que un phénomène météorologique (comme l'ouragan) est une superposition de nuages à densités différentes (on voit bien sur l'image de l'ouragan Katrina que le phénomène n'est pas uniquement bleu ou rouge).
