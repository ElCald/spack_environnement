# Profiling

## Papi

### Profilers compatibles
- Tau
- Extrae

### Métrique 
Pour trouver les différentes métriques existantes avec leur description.
```
papi_avail
```

### Programme
Fonction qui permet de parser les métriques à utiliser.
```
void parse_papi_events(int *events, int *num_events) {
    char *papi_env = getenv("PAPI_EVENTS");
    
    if (papi_env == NULL) {
        // Valeurs par défaut si pas d'export
        printf("Aucune variable PAPI_EVENTS, utilisation par défaut\n");
        events[0] = PAPI_TOT_INS;
        events[1] = PAPI_TOT_CYC;
        *num_events = 2;
        return;
    }
    
    // Copier pour pouvoir modifier
    char *env_copy = strdup(papi_env);
    char *token = strtok(env_copy, ",");
    *num_events = 0;
    
    while (token != NULL && *num_events < MAX_EVENTS) {
        // Convertir le nom en code PAPI
        int event_code;
        if (PAPI_event_name_to_code(token, &event_code) == PAPI_OK) {
            events[*num_events] = event_code;
            (*num_events)++;
            printf("Événement ajouté : %s\n", token);
        } else {
            fprintf(stderr, "Attention : événement inconnu '%s'\n", token);
        }
        token = strtok(NULL, ",");
    }
    
    free(env_copy);
}
```

Dans le main.
```
int main(){
    // ---------- CODE ----------

    int events[MAX_EVENTS];
    int num_events = 0;
    long long values[MAX_EVENTS];
    int EventSet = PAPI_NULL;
    
    // Initialiser PAPI
    PAPI_library_init(PAPI_VER_CURRENT);
    
    // Lire les événements depuis l'environnement
    parse_papi_events(events, &num_events);
    
    // Créer l'EventSet
    PAPI_create_eventset(&EventSet);
    
    // Ajouter les événements
    for (int i = 0; i < num_events; i++) {
        PAPI_add_event(EventSet, events[i]);
    }
    
    PAPI_start(EventSet);
    
    // ------------ PROGRAMME ------------ //
    //             À PROFILER              //
    // ----------------------------------- //
    
    PAPI_stop(EventSet, values);
    
    printf("With papi\n");
    for(int i=0; i<NB_VALUES; i++)
        printf("%d : %lld\n", i+1, values[i]);
    
    printf("%d,%d,%d,%lf\n", nx, ny, niters, toc-tic);

    // ---------- RESTE DU CODE ----------
}
```

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
