# Spack environnement
Installation de packages de profiling avec spack dans un environnement. <br>

Un environnement c'est un répertoire qui contiendra tous les packages, une fois installé il suffira d'activer celui-ci et les packages seront accessibles dans toute la session utilisateur. Généralement on crée ses répertoires dans un répertoire `env` que l'on met dans le `/.spack` ou on crée un autre répertoire pour les environnements, par exemple `/.spack-env`.

## Création d'un environnement

Créer un répertoire `env_profiler` (nom arbitraire) où se trouvera l'installation de l'environnement avec les packages, par exemple dans `/$HOME/.spack/env/` ou `/$HOME/.spack-env/`, si le répertoire où se trouveront les environnements n'existe pas, le créer. <br>

Dans ce cas, le répertoire `env_profiler` sera celui qui contiendra les packages de profiling et où on se trouvera pour l'exemple.
```
mkdir /$HOME/.spack-env/env_profiler
cd /$HOME/.spack-env/env_profiler
spack env create . 
```

## Chargement des packages
Il faut en amont activer l'environnement pour faire toute actions, on se trouve dans le répertoire `env_profiler`.
```
spack env activate .
```

Pour désactiver l'environnement.
```
spack env deactivate
```

### Méthode manuelle
Ajout de chaque package, ce qui remplira le `specs`.
```
spack add gcc
spack add papi
spack add tau +papi +binutils +pdt +mpi +openmp
spack add extrae +papi
spack add valgrind
spack add likwid
```

Pour retirer un package de l'environnement. Ce sera nécessaire pour l'ajouter de nouveau avec des options. Cette commande modifie le fichier `spack.yaml`, il faudra par la suite refaire l'installation.
```
spack remove <package>
```

Pour tout retirer
```
spack remove --all
```

### Méthode avec fichier de config
Dans le répertoire `env_profiler`, si le fichier `spack.yaml` n'existe pas, le créer et ajouter ceci.

```
spack:
  specs:
  - gcc
  - papi
  - tau +papi +binutils +pdt +mpi +openmp
  - extrae +papi
  - valgrind 
  - likwid

  packages:
    all:
      require: "%gcc"

  concretizer:
    unify: when_possible

  view: false
```

- **specs** : packages à installer avec les [variants](#Les-variants).
- **packages** : permet d'ajouter des préférences aux packages, de l'imposer pour tous les packages (`all`) ou un seul spécifique (`<package>`), **ça peut créer des conflits**.
  - require : force une version (ex: `@1.5.7`) ou impose un [compilateur](#Les-compilateurs).
  - variants : `+` impose un variant.
  - providers : force un fournisseur (ex : `mpi: [openmpi]`).
  - version : indique des versions de préférences **sans forcer** (ex : [1.23.1, 1.24.2]).
- **concretizer** : `true/false/when_possible`, impose ou non l'usage d'une unique version de package. Par exemple si `gcc` a besoin d'une version de `zstd` et `tau` a besoin d'une autre, alors il y aura conflit, donc on peut soit mettre `false`, pour tout permettre ou `when_possible` pour maximiser l'usage d'une version d'un package.
- **view** : `true/false` crée un fichier qui unifie les liens symbolique (commodité), si `false` il faudra impérativement faire les `spack load <package>`. Il peut être source de conflit entre versions de package si `true`.
  
#### Les variants
Les variants permettent à un package d'être compilé de manière à être compatible avec un autre package. Certains packages de compilations sont activé automatiquement, d'autres doivent l'être dans le fichier de configuration.
- **+** : activer le variant
- **~** : désactiver le variant

Pour trouver les variants disponibles.
```
spack info <package>
```

Par exemple avec `extrae` nous avons 6 variants disponibles.
```
Variants:
    build_system [autotools]        autotools
        Build systems supported by the package
    cuda [false]                    false, true
        Enable support for tracing CUDA
    cupti [false]                   false, true
        Enable CUPTI support
    dyninst [false]                 false, true
        Use dyninst for dynamic code installation
    papi [true]                     false, true
        Use PAPI to collect performance counters
    single-mpi-lib [false]          false, true
        Enable single MPI instrumentation library that supports both Fortran and C
```

Nous avons ci-dessous cette ligne où `papi` est un variant compatible, ici le `false, true` signifie que c'est un paramètre booléen, dans ce cas-ci `[true]` indique que par défaut il est activé, donc faire un **+** serait redondant, mais on peut forcer sa désactivation avec **~**.
```
 papi [true]                     false, true
        Use PAPI to collect performance counters
```

#### Les compilateurs
Pour connaitres les différents compilateurs disponibles.
```
spack compiler list
```

## Installation

### Résolution de dépendances
Il est utile de vérifier la résolution des dépendances avant une installation, bien que la commande d'installation puisse le faire par défaut. <br>
Le `-f` est là pour forcer dans le cas où le fichier `spack.lock` existe car crée au premier `concretize`.
```
spack concretize -f
```

### Vérifications
Voir tout ce qui va être installé après `concretize`.
- `-d` : voir les dépendances.
- `-v` : voir les variants.
- `-c` : voir les packages concretized à installer.
- `-l` : 
```
spack find
```
On peut voir les packages demandés :
- [+] = package installé
- [-] = package désinstallé
- [^] = package non installé ou juste référencé
- ~ = variante désactivée (exemple : ~mpi = sans MPI)
- \+ = variante activée (exemple : +papi = avec PAPI)
- \- = package non installé

L'affichage se fait ensuite sous forme de sections indiquant l'architecture et le(s) compilateur(s) utilisé(s), parfois aucun n'est nécessaire `no arch / no comilers`.
```
# -- architecture / compilateur(s) ----------------
-- linux-rhel9-zen4 / %c,cxx,fortran=gcc@14.2.0 -----------------
```

### Plan d'installation
Voir le plan d'installation (on peut ne rien mettre et ça va faire le spec du fichier `spack.yaml`).
- Montre exactement ce qui sera installé
- Toutes les dépendances et leurs versions
- Les variantes activées
```
spack spec <package> <variants>
```

### Simulation
Simulation d'une installation.
```
spack install --fake
```

### Installation
Pour faire l'installation de tous les packages.
```
spack install
```

**Selon le nombre et les packages à installer, ces commandes peuvent prendre beaucoup de temps avant de faire un affichage en console.**

### Désinstallation
Pour supprimer un package, il faut avoir activé l'environnement, dans le cas contraire ça supprimera de manière globale à l'utilisateur.
```
spack uninstall <package>
```

Pour tout supprimer.
```
spack uninstall --all
```

## Utilisation de l'environnement
Il suffit d'activer l'environnment comme vu au début et de `spack load <package>` pour les packages nécéssaires. De même ces commandes doivent être faite dans le fichier de soumission. <br>
Exemple de chemin absolu pour activer depuis n'importe quel répertoire.
```
spack env activate $HOME/.spack-env/env_profiler
```
[En cours...]
