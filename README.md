# spack_environnement
Installation de packages de profiling avec spack dans un environnement.

### Création d'un environnement
Créer un répertoire où se trouvera l'installation de l'environnement, par exemple dans `/home/.spack-env/` ou `/home/.spack/env/`, si le répertoire où se trouveront les environnements n'existe pas, le créer. <br>
Le répertoire `env_profiler` sera celui qui contiendra les packages de profiling.
```
mkdir /home/.spack-env/env_profiler
cd /home/.spack-env/env_profiler
spack env create . 
```

### Chargement des packages
Il faut en amont activer l'environnment pour faire toute actions, on se trouve dans le répertoire `env_profiler`.
```
spack env activate .
```
Pour désactiver l'environnement.
```
spack env deactivate
```
#### Méthode manuelle
Ajout de chaque package, ce qui remplira le `specs`.
```
spack add gcc
spack add papi
spack add tau +papi +binutils +pdt ~mpi ~openmp
spack add extrae +papi
spack add valgrind
spack add likwid
```

Pour retirer un package de l'environnement. Ce sera nécessaire pour l'ajouter de nouveau avec des options. Il faudra par la suite refaire l'installation.
```
spack remove <package>
```
Pour tout retirer
```
spack remove --all
```
Cette commande modifie le fichier `spack.yaml`.

#### Méthode ficher de config
Dans le répertoire `env_profiler`, si le fichier `spack.yaml` n'existe pas le créer et ajouter ceci.

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

- **specs** : packages à installer avec les variants
- **view** :
- **packages** : permet de forcer une version de package (ex: le cas où on installe 2 versions du même package)
- **concretize** :

### Les variants
Les variants permettent à un package d'être compilé de manière à être compatible avec un autre package. Certains packages de compilations sont activé automatiquement, d'autres doivent l'être dans le fichier de configuration.
- **+** : ajouter le variant
- **~** : forcer la désactivation du variant

Pour trouver les variants disponibles.
```
spack info <package>
```

Par exemple avec `extrae`.
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

Nous avons cette ligne où `papi` est un variant compatible, ici le `false, true` signifie que c'est un paramètre booléen, dans ce cas-ci `[true]` indique que par défaut il est activé, donc faire un **+** serait redondant, mais on peut forcer sa désactivation avec **~**.
```
 papi [true]                     false, true
        Use PAPI to collect performance counters
```
