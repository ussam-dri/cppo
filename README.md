Pokémon Simulator
Aperçu du projet
Pokémon Simulator est une application C++ qui simule un jeu de combat Pokémon. Les joueurs peuvent créer une équipe de Pokémon, affronter des leaders de gymnase pour gagner des badges, combattre des Maîtres Pokémon, gérer leur équipe, et interagir avec leurs Pokémon ou des entraîneurs vaincus. L'application utilise des concepts de programmation orientée objet (POO) pour modéliser les entités et leurs interactions, avec une gestion des données via des fichiers CSV.
Fonctionnalités principales

Création et gestion d'une équipe de 6 Pokémon avec au moins 3 types différents.
Combats contre des leaders de gymnase et des Maîtres Pokémon basés sur la comparaison des dégâts.
Interactions avec les Pokémon et les entraîneurs vaincus via une interface commune.
Chargement des données (Pokémon, joueurs, leaders, Maîtres) depuis des fichiers CSV.
Menu interactif pour gérer l'équipe, afficher les statistiques, et lancer des combats.

Logique de l'application
L'application est centrée autour de la classe Simulateur, qui orchestre la logique du jeu. Voici une vue d'ensemble des composants et de leur fonctionnement :
1. Structure des données

Fichiers CSV :
pokemon.csv : Contient les Pokémon disponibles (nom, type1, type2, HP, attaque, puissance).
joueur.csv : Stocke les joueurs et leurs équipes (nom, Pokémon1 à Pokémon6).
leaders.csv : Liste les leaders de gymnase (nom, gymnase, médaille, Pokémon1 à Pokémon6).
maitres.csv : Liste les Maîtres Pokémon (nom, Pokémon1 à Pokémon6).


Exemple de pokemon.csv :Nom,Type1,Type2,HP,Attaque,Puissance
Salamèche,Feu,Aucun,39,Flammèche,70
Carapuce,Eau,Aucun,44,Pistolet à O,65
Bulbizarre,Plante,Poison,45,Fouet Lianes,60



2. Classes principales

Pokemon : Représente un Pokémon avec des attributs comme le nom, les types, les HP, l'attaque et les dégâts.
Entraineur : Représente un joueur, un leader ou un Maître, avec un nom, une équipe de Pokémon, et un état (vaincu ou non).
Simulateur : Gère la logique du jeu, incluant le chargement des données, l'affichage du menu, la gestion de l'équipe, les combats, et les interactions.
Interagir : Interface pour les interactions (utilisée par Pokemon et Entraineur).

3. Flux de l'application

Initialisation :
La fonction main crée une instance de Simulateur et appelle chargerDonnees pour charger les données depuis les CSV.
Extrait de code :int main() {
    Simulateur sim;
    sim.chargerDonnees();
    std::cout << "Données chargées avec succès.\n";
    sim.afficherMenu();
    return 0;
}




Chargement des données :
chargerPokemons lit pokemon.csv pour remplir pokemons_disponibles.
chargerJoueur lit joueur.csv ou crée un nouveau joueur.
chargerLeaders et chargerMaitres lisent respectivement leaders.csv et maitres.csv.
Extrait de chargerPokemons :void chargerPokemons() {
    auto donnees = lireCSV("pokemon.csv");
    for (size_t i = 1; i < donnees.size(); ++i) {
        auto& ligne = donnees[i];
        if (ligne.size() < 6) continue;
        try {
            std::string nom = ligne[0];
            Type type1 = stringToType(ligne[1]);
            Type type2 = ligne[2].empty() ? Type::Aucun : stringToType(ligne[2]);
            int hp = std::stoi(ligne[3]);
            std::string attaque = ligne[4];
            int degats = std::stoi(ligne[5]);
            pokemons_disponibles.push_back(std::make_shared<Pokemon>(nom, type1, type2, hp, attaque, degats));
        } catch (const std::exception& e) {
            std::cerr << "Erreur lors du chargement du Pokémon à la ligne " << i + 1 << ": " << e.what() << "\n";
        }
    }
}




Menu interactif :
afficherMenu propose 9 options : créer/modifier l'équipe, afficher les Pokémon, récupérer les HP, changer l'ordre, afficher les statistiques, affronter un gymnase, affronter un Maître, interagir, ou quitter.
Extrait :void afficherMenu() {
    while (true) {
        std::cout << "\n=== Menu Pokémon ===\n";
        std::cout << "1. Créer/Modifier l'équipe\n";
        std::cout << "2. Afficher les Pokémon\n";
        // ... autres options
        std::cout << "Choix : ";
        int choix;
        std::cin >> choix;
        switch (choix) {
            case 1: creerOuModifierEquipe(); break;
            case 2: afficherPokemons(); break;
            // ... autres cas
            case 9: return;
            default: std::cout << "Choix invalide.\n";
        }
    }
}




Gestion de l'équipe :
creerOuModifierEquipe permet de créer une équipe de 6 Pokémon avec au moins 3 types différents et met à jour joueur.csv.
changerOrdreEquipe échange deux Pokémon dans l'équipe et met à jour joueur.csv.
Extrait de creerOuModifierEquipe :std::set<Type> types;
for (auto& p : equipe) {
    types.insert(p->type1);
    if (p->type2 != Type::Aucun) types.insert(p->type2);
}
if (types.size() >= 3) {
    joueur = std::make_shared<Entraineur>(nom);
    for (auto& p : equipe) joueur->ajouterPokemon(p);
    enregistrerJoueur(nom, equipe);
    break;
} else {
    std::cout << "Erreur : l'équipe doit inclure au moins 3 types différents. Recommencez.\n";
}




Combats :
affronterGymnase permet de combattre un leader pour gagner un badge.
affronterMaitre est débloqué après avoir obtenu tous les badges (nombre de leaders dans leaders.csv).
simulerCombat compare les dégâts du premier Pokémon de chaque équipe.
Extrait de simulerCombat :int degats_joueur = joueur->equipe[0]->degats;
int degats_adversaire = adversaire->equipe[0]->degats;
if (degats_joueur > degats_adversaire) {
    std::cout << joueur->nom << " gagne !\n";
    combats_gagnes++;
    adversaire->vaincu = true;
    entraineurs_vaincus.push_back(adversaire);
    if (std::find(leaders.begin(), leaders.end(), adversaire) != leaders.end()) {
        badges++;
    }
}




Interactions :
interagirEntites permet d'interagir avec les Pokémon ou les entraîneurs vaincus via l'interface Interagir.
Extrait :std::cout << joueur->equipe[index - 1]->interagir() << "\n";





Concepts POO utilisés
L'application utilise plusieurs concepts POO pour structurer le code de manière modulaire et maintenable. Voici les concepts clés et leur implémentation :
1. Encapsulation

Définition : Regroupement des données et des méthodes qui agissent sur ces données dans une classe, avec contrôle d'accès via des modificateurs (public, private).
Implémentation :
Les attributs de Pokemon (nom, type1, type2, hp, attaque, degats) et de Entraineur (nom, equipe, vaincu) sont déclarés dans la section public pour un accès direct, mais pourraient être rendus private avec des getters/setters pour un contrôle plus strict.
Les attributs de Simulateur (comme pokemons_disponibles, joueur, leaders) sont private, accessibles uniquement via des méthodes publiques.
Exemple :class Simulateur {
private:
    std::vector<std::shared_ptr<Pokemon>> pokemons_disponibles;
    std::shared_ptr<Entraineur> joueur;
    int badges = 0;
public:
    void chargerDonnees();
    void afficherMenu();
};





2. Héritage

Définition : Création de classes dérivées qui héritent des propriétés et comportements d'une classe de base.
Implémentation :
Pokemon et Entraineur héritent de l'interface Interagir pour implémenter la méthode interagir().
Exemple :class Interagir {
public:
    virtual std::string interagir() const = 0;
    virtual ~Interagir() = default;
};

class Pokemon : public Interagir {
public:
    std::string interagir() const override {
        return nom + " est heureux de vous voir !";
    }
};





3. Polymorphisme

Définition : Capacité d'appeler des méthodes différentes via une interface commune, en fonction du type réel de l'objet.
Implémentation :
L'interface Interagir définit une méthode virtuelle pure interagir(), implémentée différemment par Pokemon et Entraineur.
interagirEntites utilise le polymorphisme pour appeler interagir() sur des objets Pokemon ou Entraineur sans connaître leur type exact.
Exemple :std::cout << joueur->equipe[index - 1]->interagir() << "\n"; // Appelle Pokemon::interagir()
std::cout << entraineurs_vaincus[index - 1]->interagir() << "\n"; // Appelle Entraineur::interagir()





4. Abstraction

Définition : Masquage des détails d'implémentation et exposition des fonctionnalités essentielles via des interfaces ou des méthodes abstraites.
Implémentation :
L'interface Interagir est une abstraction qui définit un contrat pour les interactions sans spécifier comment elles sont implémentées.
Simulateur abstrait la logique complexe (chargement des CSV, gestion des combats) derrière des méthodes simples comme afficherMenu et chargerDonnees.



5. Composition

Définition : Une classe contient des instances d'autres classes comme membres pour représenter une relation "a-un".
Implémentation :
Entraineur contient un std::vector<std::shared_ptr<Pokemon>> pour représenter l'équipe.
Simulateur contient des collections (pokemons_disponibles, leaders, maitres, entraineurs_vaincus) pour gérer les entités du jeu.
Exemple :class Entraineur : public Interagir {
public:
    std::string nom;
    std::vector<std::shared_ptr<Pokemon>> equipe;
    bool vaincu = false;
    void ajouterPokemon(std::shared_ptr<Pokemon> p) { equipe.push_back(p); }
};





6. Gestion de la mémoire

Définition : Utilisation de pointeurs intelligents pour gérer la durée de vie des objets.
Implémentation :
std::shared_ptr est utilisé pour les objets Pokemon et Entraineur afin d'éviter les fuites mémoire et de gérer automatiquement la destruction.
Exemple :pokemons_disponibles.push_back(std::make_shared<Pokemon>(nom, type1, type2, hp, attaque, degats));





Comment les concepts POO sont utilisés

Encapsulation : Protège les données internes de Simulateur (comme badges, combats_gagnes) et expose des méthodes publiques pour interagir avec elles, réduisant les risques d'erreurs.
Héritage et polymorphisme : Permettent à Pokemon et Entraineur de partager un comportement commun (interagir()) tout en fournissant des implémentations spécifiques, facilitant l'extensibilité (par exemple, ajouter de nouveaux types d'entraîneurs).
Abstraction : Simplifie l'interaction avec le système via des interfaces claires, masquant la complexité du chargement des CSV ou des combats.
Composition : Modélise les relations naturelles du jeu (un entraîneur a une équipe, le simulateur gère les leaders et les Maîtres).
Gestion de la mémoire : Assure une utilisation efficace des ressources en évitant les fuites mémoire, particulièrement important avec les nombreuses instances de Pokemon et Entraineur.

Installation et exécution
Prérequis

Compilateur C++ (g++, clang++) avec support C++11 ou supérieur.
Bibliothèques standard : <iostream>, <fstream>, <sstream>, <vector>, <string>, <memory>, <set>, <algorithm>, <random>.

Compilation
g++ -std=c++11 pokemon_simulator.cpp -o pokemon_simulator

Exécution
./pokemon_simulator

Fichiers requis

pokemon.csv : Liste des Pokémon disponibles.
joueur.csv : Liste des joueurs et leurs équipes (créé automatiquement si absent).
leaders.csv : Liste des leaders de gymnase.
maitres.csv : Liste des Maîtres Pokémon.

Utilisation

Lancer le programme : Exécutez ./pokemon_simulator.
Charger ou créer un joueur :
Si joueur.csv existe, choisissez un joueur existant ou créez-en un nouveau.
Créez une équipe de 6 Pokémon avec au moins 3 types différents.


Menu principal :
Option 1 : Créer/modifier l'équipe.
Option 2 : Afficher les Pokémon.
Option 3 : Récupérer les HP.
Option 4 : Changer l'ordre des Pokémon.
Option 5 : Afficher les statistiques (badges, combats gagnés/perdus).
Option 6 : Affronter un leader pour gagner un badge.
Option 7 : Affronter un Maître (nécessite tous les badges).
Option 8 : Interagir avec les Pokémon ou entraîneurs vaincus.
Option 9 : Quitter.



Structure du projet
pokemon_simulator/
├── pokemon_simulator.cpp  # Code source principal
├── pokemon.csv            # Données des Pokémon
├── joueur.csv             # Données des joueurs (généré)
├── leaders.csv           # Données des leaders
├── maitres.csv           # Données des Maîtres
├── README.md             # Documentation

Limitations et améliorations possibles

Combats simplistes : Les combats comparent uniquement les dégâts du premier Pokémon. Ajouter des multiplicateurs de type ou des tours multiples rendrait les combats plus réalistes.
Types limités : Seuls certains types (Feu, Eau, Plante, etc.) sont supportés. Étendre stringToType pour tous les types de l'enum Type.
Interface utilisateur : Ajouter une interface graphique ou améliorer la robustesse des entrées utilisateur.
Persistance des données : Stocker les badges et les statistiques dans joueur.csv pour une sauvegarde complète.

