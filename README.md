# Pokémon Simulator

## Aperçu du projet

**Pokémon Simulator** est une application C++ qui simule un jeu de combat Pokémon. Les joueurs peuvent créer et gérer une équipe de Pokémon, affronter des leaders de gymnase pour gagner des badges, combattre des Maîtres Pokémon, interagir avec leurs Pokémon ou des entraîneurs vaincus, et suivre leurs statistiques. L'application utilise des concepts de programmation orientée objet (POO) pour modéliser les entités et leurs interactions, avec une gestion des données via des fichiers CSV. Les combats sont rendus réalistes grâce à des multiplicateurs de type (basés sur les forces et faiblesses des types Pokémon) et des tours multiples où tous les Pokémon de l'équipe participent.

### Fonctionnalités principales
- Création d'une équipe de 6 Pokémon avec au moins 3 types différents.
- Combats stratégiques contre des leaders et des Maîtres, avec multiplicateurs de type et tours multiples.
- Interactions avec les Pokémon et les entraîneurs vaincus via une interface commune.
- Chargement et sauvegarde des données (Pokémon, joueurs, leaders, Maîtres) depuis des fichiers CSV.
- Menu interactif pour gérer l'équipe, afficher les statistiques, lancer des combats, et restaurer les HP.

## Logique de l'application

L'application est centrée autour de la classe `Simulateur`, qui orchestre la logique du jeu. Voici une vue d'ensemble des composants et de leur fonctionnement :

### 1. Structure des données
- **Fichiers CSV** :
  - `pokemon.csv` : Contient les Pokémon disponibles (nom, type1, type2, HP, attaque, puissance).
  - `joueur.csv` : Stocke les joueurs et leurs équipes (nom, Pokémon1 à Pokémon6).
  - `leaders.csv` : Liste les leaders de gymnase (nom, gymnase, médaille, Pokémon1 à Pokémon6).
  - `maitres.csv` : Liste les Maîtres Pokémon (nom, Pokémon1 à Pokémon6).
- **Exemple de `pokemon.csv`** :
  ```csv
  Nom,Type1,Type2,HP,Attaque,Puissance
  Salamèche,Feu,Aucun,39,Flammèche,70
  Carapuce,Eau,Aucun,44,Pistolet à O,65
  Bulbizarre,Plante,Poison,45,Fouet Lianes,60
  ```

### 2. Classes principales
- **`Pokemon`** : Représente un Pokémon avec des attributs comme le nom, les types, les HP maximum (`hp`), les HP actuels (`hp_actuel`), l'attaque, et les dégâts. Inclut une méthode pour calculer les multiplicateurs de type.
- **`Entraineur`** : Représente un joueur, un leader ou un Maître, avec un nom, une équipe de Pokémon, et un état (vaincu ou non). Vérifie si l'équipe est K.O.
- **`Simulateur`** : Gère la logique du jeu, incluant le chargement des données, l'affichage du menu, la gestion de l'équipe, les combats, et les interactions.
- **`Interagir`** : Interface pour les interactions (utilisée par `Pokemon` et `Entraineur`).

### 3. Flux de l'application
1. **Initialisation** :
   - La fonction `main` crée une instance de `Simulateur` et appelle `chargerDonnees` pour charger les données depuis les CSV.
   - Extrait de code :
     ```cpp
     int main() {
         Simulateur sim;
         sim.chargerDonnees();
         std::cout << "Données chargées avec succès.\n";
         sim.afficherMenu();
         return 0;
     }
     ```
2. **Chargement des données** :
   - `chargerPokemons` lit `pokemon.csv` pour remplir `pokemons_disponibles`.
   - `chargerJoueur` lit `joueur.csv` ou crée un nouveau joueur.
   - `chargerLeaders` et `chargerMaitres` lisent respectivement `leaders.csv` et `maitres.csv`.
   - Extrait de `chargerPokemons` :
     ```cpp
     void chargerPokemons() {
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
     ```
3. **Menu interactif** :
   - `afficherMenu` propose 9 options : créer/modifier l'équipe, afficher les Pokémon, récupérer les HP, changer l'ordre, afficher les statistiques, affronter un gymnase, affronter un Maître, interagir, ou quitter.
   - Extrait :
     ```cpp
     void afficherMenu() {
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
     ```
4. **Gestion de l'équipe** :
   - `creerOuModifierEquipe` permet de créer une équipe de 6 Pokémon avec au moins 3 types différents et met à jour `joueur.csv`.
   - `changerOrdreEquipe` échange deux Pokémon dans l'équipe et met à jour `joueur.csv`.
   - Extrait de `creerOuModifierEquipe` :
     ```cpp
     std::set<Type> types;
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
     ```
5. **Combats** :
   - `affronterGymnase` permet de combattre un leader pour gagner un badge.
   - `affronterMaitre` est débloqué après avoir obtenu tous les badges (nombre de leaders dans `leaders.csv`).
   - `simulerCombat` gère des combats avec tours multiples et multiplicateurs de type. Chaque Pokémon affronte son adversaire, infligeant des dégâts ajustés par les types jusqu'à ce qu'une équipe soit K.O.
   - Extrait de `simulerCombat` :
     ```cpp
     while (index_joueur < joueur->equipe.size() && index_adversaire < adversaire->equipe.size()) {
         auto& p_joueur = joueur->equipe[index_joueur];
         auto& p_adversaire = adversaire->equipe[index_adversaire];
         if (p_joueur->estKo()) { index_joueur++; continue; }
         if (p_adversaire->estKo()) { index_adversaire++; continue; }
         std::cout << p_joueur->nom << " (HP: " << p_joueur->hp_actuel << ") vs " << p_adversaire->nom << " (HP: " << p_adversaire->hp_actuel << ")\n";
         double multiplicateur_joueur = p_adversaire->calculerMultiplicateur(p_joueur->type1);
         int degats_infliges_joueur = static_cast<int>(p_joueur->degats * multiplicateur_joueur);
         p_adversaire->hp_actuel -= degats_infliges_joueur;
         if (!p_adversaire->estKo()) {
             double multiplicateur_adversaire = p_joueur->calculerMultiplicateur(p_adversaire->type1);
             int degats_infliges_adversaire = static_cast<int>(p_adversaire->degats * multiplicateur_adversaire);
             p_joueur->hp_actuel -= degats_infliges_adversaire;
         }
         if (joueur->equipeKo()) {
             std::cout << adversaire->nom << " gagne !\n";
             combats_perdus++;
             return;
         }
         if (adversaire->equipeKo()) {
             std::cout << joueur->nom << " gagne !\n";
             combats_gagnes++;
             adversaire->vaincu = true;
             entraineurs_vaincus.push_back(adversaire);
             if (std::find(leaders.begin(), leaders.end(), adversaire) != leaders.end()) {
                 badges++;
             }
             return;
         }
     }
     ```
6. **Interactions** :
   - `interagirEntites` permet d'interagir avec les Pokémon ou les entraîneurs vaincus via l'interface `Interagir`.
   - Extrait :
     ```cpp
     std::cout << joueur->equipe[index - 1]->interagir() << "\n";
     ```

## Concepts POO utilisés

L'application utilise plusieurs concepts POO pour structurer le code de manière modulaire et maintenable. Voici les concepts clés et leur implémentation :

### 1. **Encapsulation**
- **Définition** : Regroupement des données et des méthodes dans une classe, avec contrôle d'accès via des modificateurs (`public`, `private`).
- **Implémentation** :
  - Les attributs de `Pokemon` (nom, type1, type2, hp, hp_actuel, attaque, degats) et de `Entraineur` (nom, equipe, vaincu) sont accessibles directement, mais les attributs de `Simulateur` (comme `pokemons_disponibles`, `joueur`, `badges`) sont `private`, accessibles via des méthodes publiques.
  - Exemple :
    ```cpp
    class Simulateur {
    private:
        std::vector<std::shared_ptr<Pokemon>> pokemons_disponibles;
        std::shared_ptr<Entraineur> joueur;
        int badges = 0;
    public:
        void chargerDonnees();
        void afficherMenu();
    };
    ```

### 2. **Héritage**
- **Définition** : Création de classes dérivées qui héritent des propriétés d'une classe de base.
- **Implémentation** :
  - `Pokemon` et `Entraineur` héritent de l'interface `Interagir` pour implémenter la méthode `interagir()`.
  - Exemple :
    ```cpp
    class Interagir {
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
    ```

### 3. **Polymorphisme**
- **Définition** : Capacité d'appeler des méthodes différentes via une interface commune, selon le type réel de l'objet.
- **Implémentation** :
  - L'interface `Interagir` définit une méthode virtuelle pure `interagir()`, implémentée différemment par `Pokemon` et `Entraineur`.
  - `interagirEntites` utilise le polymorphisme pour appeler `interagir()` sur des objets `Pokemon` ou `Entraineur`.
  - Exemple :
    ```cpp
    std::cout << joueur->equipe[index - 1]->interagir() << "\n"; // Appelle Pokemon::interagir()
    std::cout << entraineurs_vaincus[index - 1]->interagir() << "\n"; // Appelle Entraineur::interagir()
    ```

### 4. **Abstraction**
- **Définition** : Masquage des détails d'implémentation et exposition des fonctionnalités essentielles via des interfaces ou des méthodes abstraites.
- **Implémentation** :
  - L'interface `Interagir` définit un contrat pour les interactions sans spécifier leur implémentation.
  - `Simulateur` abstrait la logique complexe (chargement des CSV, gestion des combats avec multiplicateurs de type) derrière des méthodes comme `afficherMenu`.

### 5. **Composition**
- **Définition** : Une classe contient des instances d'autres classes comme membres pour représenter une relation "a-un".
- **Implémentation** :
  - `Entraineur` contient un `std::vector<std::shared_ptr<Pokemon>>` pour l'équipe.
  - `Simulateur` contient des collections (`pokemons_disponibles`, `leaders`, `maitres`, `entraineurs_vaincus`).
  - Exemple :
    ```cpp
    class Entraineur : public Interagir {
    public:
        std::string nom;
        std::vector<std::shared_ptr<Pokemon>> equipe;
        bool vaincu = false;
        void ajouterPokemon(std::shared_ptr<Pokemon> p) { equipe.push_back(p); }
    };
    ```

### 6. **Gestion de la mémoire**
- **Définition** : Utilisation de pointeurs intelligents pour gérer la durée de vie des objets.
- **Implémentation** :
  - `std::shared_ptr` est utilisé pour les objets `Pokemon` et `Entraineur` pour éviter les fuites mémoire.
  - Exemple :
    ```cpp
    pokemons_disponibles.push_back(std::make_shared<Pokemon>(nom, type1, type2, hp, attaque, degats));
    ```

## Comment les concepts POO sont utilisés

- **Encapsulation** : Protège les données internes de `Simulateur` (comme `badges`, `combats_gagnes`) et expose des méthodes publiques, réduisant les erreurs.
- **Héritage et polymorphisme** : Permettent à `Pokemon` et `Entraineur` de partager un comportement commun (`interagir()`) avec des implémentations spécifiques, facilitant l'extensibilité.
- **Abstraction** : Simplifie l'interaction avec le système via des interfaces claires, masquant la complexité des combats (multiplicateurs de type, tours multiples).
- **Composition** : Modélise les relations du jeu (un entraîneur a une équipe, le simulateur gère les leaders et Maîtres).
- **Gestion de la mémoire** : Assure une utilisation efficace des ressources avec `std::shared_ptr`, essentiel pour gérer de nombreuses instances de `Pokemon` et `Entraineur`.

## Utilisation

1. **Lancer le programme** : Exécutez le programme compilé.
2. **Charger ou créer un joueur** :
   - Si `joueur.csv` existe, choisissez un joueur existant ou créez-en un nouveau.
   - Créez une équipe de 6 Pokémon avec au moins 3 types différents.
3. **Menu principal** :
   - **Option 1** : Créer/modifier l'équipe (choisir des Pokémon stratégiques pour les multiplicateurs de type).
   - **Option 2** : Afficher les Pokémon (voir HP actuels/maximum).
   - **Option 3** : Récupérer les HP (restaurer `hp_actuel`).
   - **Option 4** : Changer l'ordre des Pokémon (important pour les combats).
   - **Option 5** : Afficher les statistiques (badges, combats gagnés/perdus).
   - **Option 6** : Affronter un leader pour gagner un badge.
   - **Option 7** : Affronter un Maître (nécessite tous les badges).
   - **Option 8** : Interagir avec les Pokémon ou entraîneurs vaincus.
   - **Option 9** : Quitter.
