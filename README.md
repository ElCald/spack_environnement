# spack_environnement
Installation de packages de profiling avec spack dans un environnement.

#### Création d'un environnement
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
Ajout de chaque package.
```
spack add gcc
spack add papi
spack add tau +papi +binutils +pdt ~mpi ~openmp
spack add extrae +papi
spack add valgrind
spack add likwid
```
Forcer la compilation avec `gcc`.
```
spack package
```
