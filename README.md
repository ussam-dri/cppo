Simulateur de Jeu Pokémon en C++

Description
-----------
Ce projet est une simulation d’un jeu Pokémon en ligne de commande, écrit en C++, basé sur des mécaniques simples inspirées des jeux originaux : gestion d’équipe, combats contre des leaders de gymnase, interaction avec des entités, et affrontement des maîtres Pokémon. Il met en œuvre les principes de programmation orientée objet (POO) : encapsulation, héritage, classes abstraites, et polymorphisme.

Structure du Projet
-------------------
Le projet est structuré autour de plusieurs classes principales :
- Interagir (interface abstraite)
- Pokemon
- Entraineur
- Simulateur

Données d’entrée :
- pokemon.csv
- joueur.csv
- leaders.csv
- maitres.csv

Détails des Classes
-------------------

enum class Type
---------------
Représente les types Pokémon :
enum class Type { Aucun, Feu, Eau, Plante, Electrik, Poison, Sol, Vol, Combat, Psy };

Interface Interagir
-------------------
Classe abstraite avec une méthode interagir() purement virtuelle.
Elle impose un comportement d’interaction à toutes les entités dérivées (Pokemon, Entraineur).

Classe Pokemon
--------------
Contient :
- nom
- type1, type2
- hp, attaque, degats
Méthode interagir() retourne un message d’interaction.

Classe Entraineur
-----------------
Contient :
- nom
- équipe de Pokémon
- état vaincu ou non
Peut être un joueur, un leader ou un maître Pokémon.

Classe Simulateur
-----------------
Gère toute la logique du jeu :
- joueur
- pokemons_disponibles
- leaders
- maitres
- statistiques : badges, victoires, défaites

Comportement du Jeu
-------------------

1. Lancement du jeu
   - Simulateur sim;
   - sim.chargerDonnees();
   - sim.afficherMenu();

2. Chargement des Données :
   - lireCSV(fichier)
   - chargerPokemons()
   - chargerJoueur() (crée une équipe si aucune sauvegarde)
   - chargerLeaders()
   - chargerMaitres()

3. Création et Modification d’Équipe :
   - Nom du joueur
   - Choix de 6 Pokémon uniques
   - Vérifie la diversité de types (au moins 3)

4. Interagir avec une entité :
   - Avec un Pokémon ou leader déjà vaincu

5. Gestion de l’équipe :
   - afficherPokemons()
   - recupererHP() : soin à 100 HP
   - changerOrdreEquipe() : influe sur les combats

6. Statistiques :
   - afficherStatistiques() : badges, victoires, défaites

7. Combats :
   - affronterGymnase() : contre un leader
   - affronterMaitre() : après tous les badges
   - simulerCombat(adversaire) : compare les dégâts

Menu Principal
--------------
1. Créer ou modifier l'équipe
2. Afficher mes Pokémon
3. Récupérer les HP
4. Changer l'ordre des Pokémon
5. Afficher mes statistiques
6. Affronter un leader de gymnase
7. Affronter un maître Pokémon
8. Interagir avec une entité
9. Quitter

Exemple de Fichier pokemon.csv :
--------------------------------
Dracaufeu,Feu,Vol,100,84,60
Tortank,Eau,Aucun,100,83,60
Florizarre,Plante,Poison,100,82,60

Fonctionnalités prises en charge :
----------------------------------
- Création d’équipe avec contrainte de diversité de types
- Sauvegarde/rechargement des données
- Combats simulés
- Interactions dynamiques
- Statistiques du joueur
- Interface en ligne de commande intuitive
- Encapsulation, héritage, polymorphisme

À venir :
---------
- Avantages de types
- Système d’évolution
- Interface graphique (Qt ou SFML)

Auteur :
--------
Projet de simulation Pokémon en C++ dans le cadre d’un exercice orienté objet.
