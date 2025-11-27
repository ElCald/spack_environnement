# Profiling

## Papi

### Profilers compatibles
- Tau
- Extrae

## Tau
Génère un fichier `profile.x.x.x` qui contient les métriques.
### Sans GUI
```
pprof <profile>
```
- \a : 
- \s : 
- \c :

### Avec GUI
```
paraprof
```

### Variables d'environnement
```
export LD_LIBRARY_PATH=$(spack location -i papi)/lib:$LD_LIBRARY_PATH
```

### Combiner avec Papi
Compiler et exécuter pour chaque groupe de métriques (ne peut dépasser 3 ou 4 métriques).
```
#export TAU_METRICS=PAPI_TOT_INS,PAPI_TOT_CYC
```


## Extrae
*extrae.xml*
```
<trace>
    <enabled>true</enabled>
    <mode>seq</mode>

    <buffersize>20000000</buffersize>

    <counters enabled="yes">
        <counter enabled="yes">PAPI_TOT_INS</counter>
        <counter enabled="yes">PAPI_TOT_CYC</counter>
        <counter enabled="yes">PAPI_L1_DCM</counter>
        <counter enabled="yes">PAPI_L2_DCM</counter>
    </counters>
</trace>
```
