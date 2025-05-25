# Codes python

## Méthode de Monte-Carlo
```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.path import Path 
import random
import time


def generer_nuage(nb_points, rayon_base=10, variation=2, lissage=0.1):
    """
    Génère un ensemble de points décrivant la forme d'un nuage avec des variations douces.
    
    :param nb_points: Nombre de points décrivant le nuage.
    :param rayon_base: Rayon moyen du nuage.
    :param variation: Amplitude des variations autour du rayon.
    :param lissage: Facteur de lissage des variations (plus élevé = variations plus douces).
    :return: Liste de points ordonnés (x, y).
    """
    # Angles uniformément répartis pour garantir une couverture circulaire
    angles = np.linspace(0, 2 * np.pi, nb_points, endpoint=False)
    
    # Génération d'une variation douce sur les rayons
    # Lissage des variations : bruit gaussien cumulatif pour des variations plus douces
    bruit = np.random.normal(0, variation, nb_points)  # Bruit gaussien
    rayons = rayon_base + np.cumsum(bruit) * lissage  # Cumsum pour accumuler les variations
    
    # Limiter les rayons à un certain intervalle autour de rayon_base
    rayons = np.clip(rayons, rayon_base - variation, rayon_base + variation)
    
    # Génération des points (x, y)
    points = [(rayon * np.cos(angle), rayon * np.sin(angle)) 
              for rayon, angle in zip(rayons, angles)]
    
    return points

def tracer_nuage(points):
    """
    Trace le nuage donné par une liste de points.
    
    :param points: Liste de coordonnées (x, y) décrivant le nuage.
    """
    # Ajouter le premier point à la fin pour fermer le nuage
    points.append(points[0])
    
    # Séparer les coordonnées x et y
    x, y = zip(*points)
    
    # Tracer le nuage
    plt.figure(figsize=(10, 10))
    plt.plot(x, y, marker='o', label="Contour du nuage")
    plt.fill(x, y, alpha=0.3, color="blue", label="Nuage")
    plt.title("Forme générée du nuage")
    plt.xlabel("x")
    plt.ylabel("y")
    plt.axis("equal")
    plt.grid()
    plt.legend()
    plt.show()



# Travail sur le fichier 

def ecrire_points_dans_fichier(points, nom_fichier):
    """
    Écrit les points dans un fichier texte.
    
    :param points: Liste de coordonnées (x, y).
    :param nom_fichier: Nom du fichier dans lequel écrire les points.
    """
    with open(nom_fichier, 'w') as f:
        for x, y in points:
            f.write(f"{x} {y}\n")
    print(f"Les points ont été écrits dans le fichier '{nom_fichier}'.")

def lire_points_depuis_fichier(nom_fichier):
    """
    Lit les points depuis un fichier texte et retourne une liste de tuples (x, y).
    
    :param nom_fichier: Nom du fichier contenant les points.
    :return: Liste de coordonnées (x, y).
    """
    points = []
    with open(nom_fichier, 'r') as f:
        for ligne in f:
            # Découper la ligne en deux parties : x et y
            try:
                x, y = map(float, ligne.strip().split())
                points.append((x, y))
            except ValueError:
                print(f"Ligne invalide ignorée : {ligne.strip()}")
    return points


# Calcul de l'aire via la méthode de Monte-Carlo

def calculer_aire_monte_carlo(points, nb_essais):
    """
    Calcule l'aire du nuage à l'aide de la méthode de Monte Carlo.
    
    :param points: Liste de coordonnées (x, y) décrivant le contour du nuage.
    :param nb_essais: Nombre de tirages aléatoires à effectuer.
    :return: Aire estimée du nuage.
    """
    # Construire le rectangle englobant
    x_coords, y_coords = zip(*points)
    x_min, x_max = min(x_coords), max(x_coords)
    y_min, y_max = min(y_coords), max(y_coords)
    
    # Créer le chemin du nuage pour vérifier si un point est à l'intérieur
    nuage_path = Path(points)
    
    # Tirages aléatoires et comptage des points à l'intérieur
    points_dans_nuage = 0
    for _ in range(nb_essais):
        x_rand = random.uniform(x_min, x_max)
        y_rand = random.uniform(y_min, y_max)
        if nuage_path.contains_point((x_rand, y_rand)):
            points_dans_nuage += 1
    
    # Calcul de l'aire du rectangle englobant
    aire_rectangle = (x_max - x_min) * (y_max - y_min)
    
    # Estimation de l'aire du nuage
    aire_nuage = (points_dans_nuage / nb_essais) * aire_rectangle
    return aire_nuage

def calculer_plusieurs_aires_monte_carlo(points, nb_essais, repetitions, nom_fichier):
    """
    Effectue plusieurs calculs de Monte Carlo pour estimer l'aire du nuage
    et écrit les résultats dans un fichier texte, avec le temps requis pour chaque calcul.
    
    :param points: Liste de coordonnées (x, y) décrivant le contour du nuage.
    :param nb_essais: Nombre de tirages aléatoires par simulation.
    :param repetitions: Nombre de répétitions de la simulation.
    :param nom_fichier: Nom du fichier texte où enregistrer les résultats.
    """
    with open(nom_fichier, 'w') as f:
        for i in range(repetitions):
            start_time = time.time()  # Début de la mesure de temps
            aire = calculer_aire_monte_carlo(points, nb_essais)
            end_time = time.time()  # Fin de la mesure de temps
            
            # Temps écoulé en secondes
            temps_ecoule = end_time - start_time
            
            # Écrire les résultats dans le fichier
            f.write(f"Simulation {i + 1}: Aire estimée = {aire:.4f}, Temps requis = {temps_ecoule:.4f} secondes\n")
    
    print(f"Les résultats ont été enregistrés dans le fichier '{nom_fichier}'.")


# Contractions et dilatations 
def contracter_nuage(points, centre, facteur_contraction=0.1):
    """
    Contracte le nuage en déplaçant les points vers un point central donné.
    
    :param points: Liste des points du nuage (x, y).
    :param centre: Point central vers lequel déplacer tous les points (x, y).
    :param facteur_contraction: Facteur de contraction (0 = aucune contraction, 1 = contraction maximale).
    :return: Liste des points contractés.
    """
    points_contractes = []
    
    for x, y in points:
        # Calcul du vecteur de déplacement du point vers le centre
        vecteur_deplacement = np.array([centre[0] - x, centre[1] - y])
        
        # Calcul de la distance du point au centre
        distance = np.linalg.norm(vecteur_deplacement)
        
        # Normaliser le vecteur de déplacement pour avoir une direction sans changer la distance
        if distance != 0:
            vecteur_deplacement_normalise = vecteur_deplacement / distance
        else:
            vecteur_deplacement_normalise = np.array([0, 0])
        
        # Appliquer un déplacement léger selon le facteur de contraction
        deplacement = vecteur_deplacement_normalise * distance * facteur_contraction
        nouveau_point = (x + deplacement[0], y + deplacement[1])
        
        # Ajouter le point contracté à la liste
        points_contractes.append(nouveau_point)
    
    return points_contractes



def tracer_nuage_transition(points1, points2, couleur = "blue", nb_essais=30000):
    """
    Trace deux nuages différents avec une couleur ajustée en fonction du rapport des aires.
    
    :param points1: Liste de coordonnées (x, y) du premier nuage.
    :param points2: Liste de coordonnées (x, y) du second nuage.
    :param couleur: Couleur de base pour le premier nuage.
    :param nb_essais: Nombre de tirages pour calculer l'aire par la méthode de Monte Carlo.
    """
    # Calcul des aires des deux nuages
    aire_nuage1 = calculer_aire_monte_carlo(points1, nb_essais)
    aire_nuage2 = calculer_aire_monte_carlo(points2, nb_essais)
    
    # Calcul du rapport des aires
    if aire_nuage1 != 0:
        rapport_aire = aire_nuage2 / aire_nuage1
    else:
        rapport_aire = 0  # éviter la division par zéro, dans ce cas, tout le nuage est "réduit".
    
    # Modifier la couleur du nuage 2 en fonction du rapport des aires
    # Plus le rapport est petit, plus le nuage 2 sera foncé
    # On va modifier l'alpha (opacité) en fonction du rapport des aires.
    alpha_nuage2 = max(0.1, min(1, 1 - rapport_aire))  # l'alpha varie de 0.1 à 1
    
    ## On veut tracer un cycle
    points1.append(points1[0])
    points2.append(points2[0])
    # Séparer les coordonnées x et y pour chaque nuage
    x1, y1 = zip(*points1) if points1 else ([], [])
    x2, y2 = zip(*points2) if points2 else ([], [])
    
    # Créer une figure
    plt.figure(figsize=(6, 6))
    
    # Tracer le premier nuage avec la couleur de base (couleur)
    plt.plot(x1, y1, linestyle='-', color=couleur, label="Nuage 1")
    plt.fill(x1, y1, alpha=0.3, color=couleur)
    
    # Tracer le second nuage avec la couleur modifiée pour l'alpha
    plt.plot(x2, y2, linestyle='-', color=couleur, label="Nuage 2")
    plt.fill(x2, y2, alpha=alpha_nuage2, color=couleur)
    
    # Ajouter des labels et une légende
    plt.title("Transition entre deux nuages, f=0.2")
    plt.xlabel("x")
    plt.ylabel("y")
    plt.axis("equal")
    #plt.grid(True)
    plt.legend()
    
    # Afficher le graphique
    plt.show()



## Manoeuvres terminales

nom_fichier = "nuage_initial.txt"
#Generer le nuage 
#nb_points = 50  # Nombre de points pour décrire le nuage
#points_nuage = generer_nuage(nb_points)



# Lire un nuage
points_nuage = lire_points_depuis_fichier(nom_fichier)



# Contraction du nuage
# Choisir un centre (par exemple, le barycentre des points du nuage)
centre = np.mean(points_nuage, axis=0)  # Moyenne des coordonnées x et y (barycentre)
centre = points_nuage[0]
points_nuage_contracte = contracter_nuage(points_nuage, centre, facteur_contraction=0.2)

# Tracer le.s nuage.s
tracer_nuage(points_nuage_contracte)
tracer_nuage_transition(points_nuage,points_nuage_contracte)
tracer_nuage(points_nuage)

# Écrire les points dans un fichier texte
ecrire_points_dans_fichier(points_nuage, nom_fichier)
```
## Animation du nuage 

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from scipy.ndimage import map_coordinates



# Création de la carte des coûts (ex : 100x100 avec différentes zones)
# Ici, carte de la dernière slide
def generer_carte_cout(taille=100):
    carte = np.ones((taille, taille))  # Fond facile (valeur 1)

    # Rectangle central – difficulté moyenne
    carte[30:45, 30:60] = 4

    # Rectangle bleu – difficulté légère
    carte[0:10,30:95] = 2
    carte[10:90, 85:95] = 2


    # Rectangle en bas à gauche – difficulté élevée
    carte[10:30, 30:40] = 8
    carte[70:90,50:70]=8


    # Rectangle difficulté moyenne 
    carte[50:70,60:85]=3



    

    return carte





# Générer un nuage de points dans l'espace, ici on commencera simplement avec une forme circulaire
def generer_nuage(nb_points, centre=(50, 50), rayon=10):
    angles = np.linspace(0, 2 * np.pi, nb_points, endpoint=False)
    points = [(centre[0] + rayon * np.cos(a), centre[1] + rayon * np.sin(a)) for a in angles]
    return np.array(points)




# Fonction de coût en fonction de la carte des coûts
def fonction_cout(point, carte):
    x, y = point
    return map_coordinates(carte, [[x], [y]], order=1)[0]  # interpolation bilinéaire, on ne veut pas traiter que des entiers sur la cartes des coûts, mais aussi des positions flottantes 




# Calcul du gradient du coût
def calculer_gradient(point, carte, epsilon=1.0):
    # epsilon est une petite distance pour approximer la dérivée
    x, y = point
    grad_x = (fonction_cout((x + epsilon, y), carte) - fonction_cout((x - epsilon, y), carte)) / (2 * epsilon) # dérivée partielle selon x
    grad_y = (fonction_cout((x, y + epsilon), carte) - fonction_cout((x, y - epsilon), carte)) / (2 * epsilon) # dérivée partielle selon y
    return np.array([grad_x, grad_y])






# Moyenner les écarts : on ne veut pas un nuage qui se sépare !
def regulariser_positions(points, facteur_seuil=1.5):
    
    #Ajuste les positions des points pour éviter qu'ils ne se dispersent trop.
    #facteur_seuil est l'équivalent du epsilon_0 de la présentation 
    
    nouveaux_points = points.copy()
    nb_points = len(nouveaux_points)

    # Calcul des distances entre tous les points voisins (y compris premier <-> dernier)
    distances = [np.linalg.norm(nouveaux_points[i] - nouveaux_points[i-1]) for i in range(nb_points)]
    distance_moyenne = np.mean(distances)

    for i in range(nb_points):
        j = (i + 1) % nb_points  # Voisin suivant (avec boucle)
        d = np.linalg.norm(nouveaux_points[j] - nouveaux_points[i]) # norme = distance dans R^2
        
        if d > facteur_seuil * distance_moyenne:  # Trop éloigné
            correction = (nouveaux_points[i] + nouveaux_points[j]) / 2  # Moyenne entre les deux
            nouveaux_points[j] = correction  # Ajustement
    
    return np.array(nouveaux_points)



# Déplacement glouton en minimisant le coût total
def calculer_deplacement(A, B, carte, pas=1.0, force_repulsion=0.1, rayon_repulsion=10.0):
    nouveaux_A = []
    nb_points = len(A)

    for i, ((xA, yA), (xB, yB)) in enumerate(zip(A, B)):
        direction = np.array([xB - xA, yB - yA])
        direction /= np.linalg.norm(direction) + 1e-8  # Normalisation

        # Gradient du coût
        gradient = calculer_gradient((xA, yA), carte)
        direction -= 0.5 * gradient # le 1/2 de la fonction de coût et le signe - apparaissent bien

        # Répulsion en cas de points trop rapprochés
        repulsion = np.zeros(2)
        for j, (xj, yj) in enumerate(A):
            if i != j:
                vecteur = np.array([xA - xj, yA - yj])
                distance = np.linalg.norm(vecteur)
                if 0 < distance < rayon_repulsion:
                    # Force inversement proportionnelle à la distance
                    repulsion += vecteur / (distance**2 + 1e-8)

        direction += force_repulsion * repulsion

        nouveaux_A.append((xA + pas * direction[0], yA + pas * direction[1])) # on met à jour les coordonnées

    # On conserve la régularisation pour la cohésion
    return regulariser_positions(np.array(nouveaux_A))


# Simulation complète du transport : on mémorise 100 iterations dans une liste de coordonnées pour pouvoir les animer ensuite 
def simulation_transport(A, B, carte, iterations=100, pas=1.0):
    historique = [A]
    points = A
    for _ in range(iterations):
        points = calculer_deplacement(points, B, carte, pas)
        historique.append(points)
    return historique



# Paramètres et génération des données
nb_points = 20
carte_cout = generer_carte_cout()
A = generer_nuage(nb_points, centre=(20, 20))
B = generer_nuage(nb_points, centre=(80, 80))
historique = simulation_transport(A, B, carte_cout)




# Animation : gestion de matplotlib et des couleurs 
fig, ax = plt.subplots(figsize=(6, 6))
ax.imshow(carte_cout.T, cmap='coolwarm', origin='lower', extent=[0, 100, 0, 100])
points_plot, = ax.plot([], [], 'o-', color='black')
ax.set_xlim(0, 100)
ax.set_ylim(0, 100)

def update(frame):
    # met à jour l'image à chaque étape de l'animation
    x, y = zip(*historique[frame])
    points_plot.set_data(x, y)
    return points_plot,


# Animation gif + enregistrement dans les documents
ani = animation.FuncAnimation(fig, update, frames=len(historique), interval=100)
ani.save("transport_nuage_avec_cout.gif", writer="pillow", fps=10)
plt.show()

```
