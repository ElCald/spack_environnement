# spack_environnement
Installation de packages de profiling avec spack dans un environnement. <br>

Un environnement c'est un répertoire qui contiendra tous les packages, une fois installé il suffira d'activer celui-ci et les packages seront accessibles dans toute la session utilisateur. Généralement on crée ses répertoires dans un répertoire `env` que l'on met dans le `/.spack` ou on crée un autre répertoire pour les environnements, par exemple `/.spack-env`.

## Création d'un environnement

Créer un répertoire `env_profiler` où se trouvera l'installation de l'environnement avec les packages, par exemple dans `/$HOME/.spack/env/` ou `/$HOME/.spack-env/`, si le répertoire où se trouveront les environnements n'existe pas, le créer. <br>

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
spack add tau +papi +binutils +pdt ~mpi ~openmp
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

### Méthode avec ficher de config
Dans le répertoire `env_profiler`, si le fichier `spack.yaml` n'existe pas, le créer et ajouter ceci.

```
spack:
  specs:
  - gcc
  - papi
  - tau +papi +binutils +pdt ~mpi ~openmp
  - extrae +papi
  - valgrind 
  - likwid
  
  view: false

  packages:
    all:
      require: "%gcc"
    zstd:
      require: "@1.5.7"

  concretizer:
    unify: when_possible
```

- **specs** : packages à installer avec les [variants](####Les-variants).
- **view** : `true/false` unifie les liens symbolique (commodité), si `false` il faudra impérativement faire les `spack load <package>`. Il peut être source de conflit entre versions de package si `true`.
- **packages** : permet de forcer une version de package (ex: le cas où on installe 2 versions du même package) ou d'imposer un compilateur pour tous les packages, **ça peut créer des conflits**.
- **concretize** : `true/false/when_possible`, impose ou non l'usage d'une unique version de package. Par exemple si `gcc` a besoin d'une version de `zstd` et `tau` a besoin d'une autre, alors il y aura conflit, donc on peut soit mettre `false`, pour tout permettre ou `when_possible` pour maximiser l'usage d'une version d'un package.

#### Les variants
Les variants permettent à un package d'être compilé de manière à être compatible avec un autre package. Certains packages de compilations sont activé automatiquement, d'autres doivent l'être dans le fichier de configuration.
- **+** : ajouter le variant
- **~** : forcer la désactivation du variant

Pour trouver les variants disponibles.
```
spack info <package>
```

Par exemple avec `extrae` nous avons 5 variants disponibles.
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
Il est utile de reconfigurer les dépendances avant une installation, bien que la commande d'installation puisse le faire par défaut.
```
spack concretize -f
```
Le `-f` est là pour forcer dans le cas où le fichier `spack.lock` existe car crée au premier `concretize`. <br>

Pour faire l'installation de tous les packages.
```
spack install
```

**Selon le nombre et les packages à installer, ces 2 commandes peuvent prendre beaucoup de temps avant de faire un affichage en console.**

### Désinstallation
Pour supprimer un package, il faut avoir activé l'environnement, dans le cas contraire ça supprimera de manière globale.
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
