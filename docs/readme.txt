Link al corso https://app.pluralsight.com/library/courses/kubernetes-developers-core-concepts/table-of-contents
Repo github dell'autore https://github.com/DanWahlin/DockerAndKubernetesCourseCode

Introduzione
	Parliamo di concetti core di K8S come 
		- Pod (eseguono i container)
		- Deployment (possibile eseguire il deploy senza un downtime)
		- Services (per es. il networking)
		- Opzioni di storage (volumi persistenti etc)
		- Mappe di configurazione (per avere config. differente per vari ambienti) e secrets (per salvare una configurazione sensibile)
	Uniamo tutto insieme per vedere la costruzione di un ambiente completo
	NOTA: il corso non riguarda l'amministrazione di K8S, ci sono corsi ad hoc che spiegano come amministrare K8S (es. cluster, la parte di cloud etc)
	K8S e' un sistema open source per automatizzare il deployment, scalabilita', e gestione delle app contenerizzati, vedi https://kubernetes.io/
	NOTA: docker-compose e' un tool da usare in locale, e funziona molto bene, in teoria lo possiamo usare anche per un ambiente di produzione, ma con dei limiti
		non e' un gestore di container, non gestisce il failover e scaling
	K8S e' un orchestratore di container 
	Cosa puo' fare orchestratore K8S (funzionalita' chiavi per un dev):
		- discovering di servizi e load balancing
		- gestire lo storage all'interno di un cluster K8S 
		- automatizzare rollbak e rollout (deploy delle nuove versioni)
		- ripristino di un container crashato
		- gestione della configurazione e secrets (dati sensibili salvati in una secret area)
		- scaling orizzontale di container 
	The big picture di K8S
		K8S sviluppato da google per 15 anni e dopo donato (open sourced) alla cloud community
		K8S si basa sullo stato del cluster 
			garantisce che il cluster mantiene uno stato desiderato/configurato 
			abbiamo 1 o piu' nodi master - e' il servizio responsabile di monitoraggio del cluster e gestione di worker nodes (server fisici, VM)
				N worker nodes formano il cluster 
			master istanzia all'interno di worker node i Pods
				Pod - e' il modo con cui facciamo il hosting di container 
				Pod praticamente e' un Box/Scatola contenente i container 
				Pod puo' essere visto come trasportatore di Container, che alla fine girano fisicamente sui worker nodes 
			Per deployare un Pod abbiamo oggetto Deployment in K8S 
				Pod e' minima unita' deployabile
				Oggetto Deployment incapsula ReplicaSet (vedi sotto)
			Esiste esigenza di far comunicare i Pod tra di loro all'interno del cluster K8S
				per fare questa impostazione abbiamo oggetto Service in K8S 
		Come e' stato gia' detto prima un Pod gira su una VM (macchina virtuale), chiamiamolo Node (nodo)
		Il nodo Master comunica con gli altri Nodi all'interno del cluster 
			il nodo Master ha uno store - etcd 
			praticamente e' un DB con tutte le info relative al nodo master 
			lo store contiene tutto quello che deve fare Master 
			nodo Master ha un Controller Manager che gestisce tutte le richieste in ingresso (es. nuova configurazione del cluster, scaling etc.)
			nodo Master usa Scheduler per schedulare varie operazioni
				es. quando un Pod all'interno di un Nodo deve essere avviato 
			nodo Master espone un API che puo' essere acceduto usando il tool kubectl
				kubectl prende in input un file di configurazione (YAML/JSON) e lo invio al nodo Master 
				-> nodo Master ci pensa a portare lo stato di cluster alla situazione desiderata
			API Server del nodo Master e' un API REST, kubectl esegue delle chiamate REST 
		Ogni Nodo che ospita i Pod ha un agent Kubelet responsabile della comunicazione tra il Master e i Nodi 
		Ogni nodo contiene anche il container runtime per eseguire i container 
		Ogni nodo contiene anche Kube-Proxy per la parte di networking
			garantisce che ogni Pod ha un indirizzo IP univoco
	Benifici e Use Cases
		recap dei benifici di utilizzo di container
			- onboarding piu' veloce di nuovi dev
			- possiamo avere tutto l'ambiente in locale usando docker-compose
			- eliminazione conflitti tra app che girano sulla stessa macchina 
			- consistenza degli ambienti (test, staging, prod)
			- condivisione di software piu' veloce 
		benifici di utilizzo di K8S
			- orchestrazione di container 
			- zero downtime deployment 
			- self healing (ripristino di container nel caso di errori)
			- scaling automatizzato 
		benifici per dev
			- emulare ambiente di produzione in locale 
			- se azienda usa K8S in produzione e' utile sapere i concetti di K8S
			- creare un ambiente per i test E2E
			- assicurarsi che l'app scala opportunamente
			- assicurarsi che secrets/config funziona in modo corretto tra i vari ambienti 
			- performance test 
			- workload per CI/CD
			- studiare come bilanciare opportunamente il deploy delle app
			- essere di aiuto a Devops per il trouble shooting, se loro usano K8S 
	Runnure K8S in locale 
		usare minikube https://github.com/kubernetes/minikube
			ci sono delle limitazioni, vedi sotto
		usare docker desktop 
			usiamo questo tool nel corso 
		usare https://kind.sigs.k8s.io/ 
			aka kubernetes in docker 
			abbiamo una certa flessibilita' sulle opzioni di scaling 
			vedi docs ufficile 
		usare kubeadm https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
			se vogliamo avere il kubernetes in locale
			e' fuori dallo scope di questo corso 
			riguarda piu' amministratori di un cluster K8S 
			ci sono dei corsi pluralsight dedicati 
	Installiamo Docker desktop e abilitiamo Kubernetes
	tool kubectl
		kubectl versioni
		kubectl cluster-info
		kubectl get all (per leggere le info di tutti Pods, Deployments, Services etc.)
			kubectl get pods
			kubectl get services
		kubectl run [container-name] --image=[image-name] (creazione di un Deployment per il Pod, modo facile)
		kubectl port-forward [pod] [ports] 
			di default ad un Pod viene associato un IP del cluster e solo altri Pod e nodi del cluster lo posso raggiungere
			per esporre quel IP all'esterno di cluster usiamo il comando port-forward
		kubectl expose ...
			esporre una porta per Deployment/Pod 
		kubectl create [resource]
			creazione di Deployment/Service etc.
		kubectl apply [resource] 
			creazione/modifica di risorse 
			e' il comando per passare da un stato all'altro di K8S 
	Dashboard Web UI
		per installare la dashboard gli step da seguire sono:
			- kubectl apply [dashboard-yaml-url] (NOTA: url puo' cambiare, verificare la documentazione ufficiale)
				https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/ vedi qui ultimo URL 
			- kubectl describe secret -n kube-system
			- recuperare il token da kubernetes.io/service-account-token 
			- kubectl proxy
			- aprire l'URL della dashboard ed eseguire login usando il token 
				URL da aprire e' recuperabile dalla doc, si apre la pagina dove dobbiamo inserire il token appena copiato 
Creazione di Pods
	concetti core
		un pod e' una unita minima che possiamo manipolare con k8s (creare, deployare)
		pod rappresenta un ambiente per container 
		possiamo pensare in che modo organizzare la nostra app in un set di Pods (per app server, cache, db, etc..)
		Pod puo' ospitare piu' container che condividono Pod IP, memoria, volume di dati
		i Pods possanno essere scalati usando la replica di Pod 
			cioe', un nodo puo' ospitare N versioni di un Pod, aggiornati continuamente e fare in modo che il carico viene bilanciato 
			se un Pod muore, il nodo Master lo sostituisce con un nuovo Pod 
			NOTA: un Pod non viene ripristinato o riavviato -> nodo Master rimpiazza il Pod con una versione nuova 
		Pod viene creato o distrutto/rimosso dal nodo Master, vedi sotto la parte di health check di Pod 
		i container in un Pod condividono la stessa rete e IP del Pod
			hanno la stessa interfaccia di loopback - localhost
			questo facilita la comunicazione tra i container all'interno del Pod 
			container condividono lo spazio delle porte di rete e devono usare porte differenti 
		invece se container sono nei Pod differenti, possano usare le stesse porte 
		NOTA: un Pod gira solo sun un worker node!
	creazione di Pod 
		ci sono due strade
			1. kubectl run [podname] --image=nginx:alpine
			2. usare il file YAML (un approccio dichiarativo)
		kubectl get pods - per stampare la lista di pods 
		di default un Pod e' accessibile solo all'interno del cluster 
		per esporre le porte all'esterno usiamo il comando kubectl port-forward [name-of-pod] 8080:80
			8080 - porta esterna 
			80 - porta interna 
		per eliminare un pod usiamo kubectl delete pod [name-of-pod]
			NOTA: un Pod eliminato un questo modo viene rimpiazzato da un nuovo Pod da nodo Master 
			per eliminare un Pod definitivamente dobbiamo eliminare il Deployment
			per eliminare il Deployment il comando da lanciare e' kubectl delete deployment [name-of-deployment]
	fondamenti YAML 
		e' un file txt composto da mappe e liste 
		indentazione e' importante, usiamo gli spazi 
			serve per separare le sezioni 
		mappa
			insieme di chiave:valore
			puo' contenere altre mappe 
		lista
			sequenza di item 
			puo' contenere un insieme di mappe 
		esempio di un file YAML:
		
			key: value
			complexMap:
			  key1: value
			  key2:
			    subKey: value
			items:
			  - item1
			  - item2 
			itemsMap
			  - map1: value
			    map1Prop: value 
			  - map2: value
			    map2Prop: value
			
	definizione di un Pod usando YAML
		vedi esempio nel file di repo
		sezione metadata contieme la mappa con la chiave name usata per definire il nome del Pod
		sezione spec conteiene la specifica del Pod, es. container 
		per creare il Pod partendo dal file YAML lanciamo il comando 
			kubectl create -f nginx.pod.yml --dry-run --validate=true 
				--dry-run serve per vedere l'impatto di istruzioni presenti nel file di config., la modifica non impatta il cluster 
				--validate=true, e' true di default, impostare a false se non vogliamo eseguire la validazione del file YAML 
			kubectl create -f nginx.pod.yml
				creazione del Pod, le modifiche si propagano sul cluster K8S 
			NOTA: il comando create se lanciamo con il nome del Pod che esiste gia', ritorna un errore 
		per aggiornare il Pod eseguiamo il comando 
			kubectl apply -f nginx.pod.yml
			in questo caso se il Pod non esiste, viene creato, se esiste, viene aggiornato 
		per salvare la configurazione della risorsa nelle annotations eseguire il comando di create con il parametro --save-config 
			ci permette di aggiornare il Pod successivamente usando il comando apply 
		kubectl edit - consente editare il file di una risorsa (Pod, Deployment, Service) nella console 
		kubectl patch - consente ad aggiornare un proprieta' specifica della risorsa 
		per eliminare il Pod creato usando YAML eseguiamo il comando 
			kubectl delete -f nginx.pod.yml
		nei metadati di YAML possiamo definire sezione lables usata per poter linkare piu' risorse tra di loro (es. Deployment con Services, Pods etc..)
		sezione ports all'interno di containers permette di definire le porte in uso dal container 
		
		kubectl get pod/my-nginx -o yaml (stampa i dati del pod in formato YAML, possiamo passare json)
		kubectl describe pod/my-nginx (stampiamo i dettagli del Pod creato, storico eventi etc.)
			NOTA: vediamo lo storico di aggiornamento di Pod 
		
		kubectl exec my-nginx -it sh (per aprire la shell/console all'interno di container che gira nel Pod)
		kubectl edit -f nginx.pod.yml (per editare le proprieta' del Pod)
	Pod health
		K8S usa il termine Probes nell'ottica di determinazione della salute di container associati ad un Pod 
		Probe e' la diagnostica eseguita periodicamente da parte di kubelet verso i container 
		ci sono due tipi di Probes
			1. Liveness Probe - se e' live, sta girando, se e' tutto ok
			2. Readiness Probe - determina se il Pod puo' ricevere le richieste 
		se un Pod crasha il nodo Master ci pensa a rimpiazzarlo 
		se crasha un container nel Pod, di default abbiamo il parametro restartPolicy=Always, quindi il container si riavvia 
		il fatto che determina se un container e' OK dipende molto dall'app che sta girando 
			possiamo definire una ExecAction, viene eseguita una azione nel container, e se ritorna 0, tutto OK
			TCPSocketAction, e' un check TCP su un indirizzo e porta specifica del container 
			HTTPGetAction, una richiesta HTTP GET verso il container 
		Probes possano avere i seguenti risultati
			- Success
			- Failure
			- Unknown 
		usiamo YAML per definire i Probes
			NOTA: cmq e' il carico di dev di prevedere health endpoints che eseguono tutte le verifiche necessarie per determinare se l'app e' up and running
			es. nella sezione containers, per il container di interesse, definiamo:
			
				livenessProbe:
				  httpGet: 
				    path: /index.html
					port: 80
				  initialDelaySeconds: 15  /* un ritardo iniziale utile ad assicurarsi che il container e' su */
				  timeoutSeconds: 2
				  periodSeconds: 5 		/* ogni 5 sec viene fatto il check */
				  failureTreshold: 1	/* se fallisce al meno un check il container non e' OK */
				  
			es. di ExecAction (vedi la doc ufficiale, dovrebbe esserci un esempio del genere, in questo caso il check si aspetta il file health sotto /tmp)
			
				args:
				  - /bin/sh
				  - -c
				  - touch /tmp/healthy; sleep 30;
				    rm -rf /tmp/healthy; sleep 600
					
				livenessProbe:
				  exec:
				    command:
					  - cat 
					  - /tmp/healthy
				  initialDelaySeconds: 5
				  periodSeconds: 5
				  
			Readiness Probe - quando il container e' pronto a ricevere il traffico in ingresso
			Liveness Probe - quando il container deve essere riavviato 
			
			per fare una prova prova ad eliminare index.html dalla cartella di nginx 
			per la demo di ExecAction vedi YAML della doc ufficiale 
Creazione di Deployments 
	NOTA: nella descrizione di Pod abbimo creato dei Pod senza un Deployment, se il Pod muore, non abbiamo niente che lo ripristina 
	concetti base di un Deployment
		ReplicaSet - il modo dichiarativo di gestire i Pods, qualcosa che tiene d'occhio i Pods e garantisce che stanno girando in modo efficiente
		Deployment - il modo dichiarativo di gestire i Pods usando ReplicaSet
		NOTA: nel corso di evoluzione di K8S ReplicaSet e' nata prima, dopo e' stato introdotto il Deployment che ha wrappato ReplicaSet al suo interno
	ReplicaSet agisce come un Pod controller
		- meccanismo self-healing (es. se eliminiamo un Pod, ReplicaSet insieme a Deployment ne crea una nuova istanza di Pod)
		- garantisce che il numero desiderato di Pod e' sempre un and running 
		- garantice fault-tolerance
		- scaling di Pods 
		- usa i Pod template (visti nel modulo precedente)
		- non dobbiamo creare direttamente i Pods, usiamo Deployment e ReplicaSet
		- ReplicaSet viene usata da Deployment
	Deployment gestisce i Pod 
		- i Pod sono gestiti usando ReplicaSet
		- esegue lo scaling di ReplicaSets, che a sua volta fa scalare i Pods
		- supporta gli aggiornamento senza downtime, creando e distrugendo ReplicaSets
		- fornisce funzionalita' di rollback 
		- crea delle label univoci assegnati alla ReplicaSet e Pod generati
		- YAML e' molto simile a quello di ReplicaSet
	creazione del Deployment usando YAML 
		NOTA: a noi basta creare il Deployment, ReplicaSet viene usata da Deployment
		esempio base di YAML per creare un Deployment
		
			apiVersion: apps/v1		# fare rif. sempre alla documentazione ufficiale per impostare la versione corretta 
			kind: Deployment
			metadata:				# metadati come label per esempio 
			spec:
			  selector:				# selettore del template da usare per creare il Pod (nel nostro caso il template e' subito sotto)
			  template:
			    spec:				# questa sezione e' identica alla definizione del Pod 
				  containers:
				    - name: my-nginx
					  image: nginx:alpine 
					  
		esempio di YAML con metadati e selector 
		
			apiVersion: apps/v1
			kind: Deployment
			metadata:
			  name: frontend		# nome del Deployment
			  labels:
			    app: my-nginx
				tier: frontend 
			spec:
			  selector:				# qui abbiamo definito il selettore per usare il template definito subito sotto (ma questo template puo' risiedere su un file separato)==
			    matchLabels:		# passiamo N file al tool kubectl, e usando lables linkiamo le risorse 
				  tier: frontend
			  template:
			    metadata:
				  tier: frontend
			    spec:
				  containers:				# questa sezione puo' contenere i probes, come nel modulo Pods!
				    - name: my-nginx
					  image: nginx:alpine 
					  
		per manipolare un Deployment usiamo i seguenti comandi di kubectl
			kubectl create -f file.deployment.yml (NOTA: ricordiamo di usare il parametro --save-config se nel futuro intendiamo usare comando apply per apportare le modifiche al Deployment)
			kubectl apply -f file.deployment.yml
			kubectl get deployments
			kubectl get deployment --show-labels 
		NOTA: label possano essere usate per organizzare le cose (possiamo avere N deploy di FrontEnd, N di BackEnd, o altri layer)
			kubectl get deployment -l app=nginx (stampiamo i dettagli del Deployment con una label specifica)
			kubectl delete deployment [deployment-name]
			
			kubectl scale deployment [deployment-name] --replicas=5
				per scalare i Pods di Deployment a 5 unita'
				abbiamo 5 Pods che girano 
				stessa cosa possiamo ottenere usando il nome del file YAML 'kubectl scale -f file.deployment.yml --replicas=5'
			
			possiamo inserire il numero di replicas all'interno di YAML, es:
				spec:
				  replicas: 3
				  selector:
				    tier: frontend
					
	kubectl Deployment in action
		vedi esempio nel repo git
		sezione resources - serve per impostare un limite all'utilizzo di risorse del Nodo da parte di Pod 
		NOTA: e' importante impostare questi limiti in modo che un Deployment non mette in crisi intero Nodo (aka VM)
		
	opzioni di Deployment
		possiamo ottenere un deployment senza il fermo macchina per utente finale 
		NOTA: c'e' un corso dedicato al deployment 
		praticamente, nel momento di aggiornamento, viene creato un nuovo Pod (es. abbiamo aggiornato immagine docker)
			e successivamente tutto il traffic viene reindirizzato verso questo nuovo Pod facendo morire il Pod vecchio
		le opzioni disponibili per il Deployment sono:
			- Rolling updates (spiegato piu' avanti)
			- Blue-green deployments (quando abbiamo diversi ambienti e prima di switchare vogliamo essere sicuri che ambiente nuovo e' OK)
			- Canary deployments (quando abbiamo una parte piccola di traffico che va verso il nuovo Deployment,
				testiamo, e se tutto ok, switchiamo utenti verso il nuovo Deployment)
			- Rollbacks (il test di nuovo deployment e' fallito, eseguiamo il rollback)
		Rolling deployments
			vedi slides per una visualizzazione piu' chiara
			praticamente viene creato nuovo Pod (es. con la nuova versione del software) e quando e' up e running (ricordiamo che abbiamo probes per questo scopo),
				viene eliminato un Pod vecchio -> si procede con creazione di un altra istanza del Pod nuovo (se abbiamo un cluster con N Pod)
				-> e quando e' su si elimina una vecchia e cosi via...
			questo aggiornamento avviene in automatico usando kubectl apply fornendo un nuovo file YAML 
			vedi i file di esempio nel repo git dell'autore
			NOTA: il parametro minReadySeconds serve per rimanere in attesa del Pod per N secs per assicurarsi che il container non e' crashato 
				(container puo' crashare se non riesce a partire per mancanza delle dipendenze o risorse esterne etc..)
				se container crasha, viene rischedulato
				solo dopo N secs il Pod puo' iniziare a ricevere il traffico rete 
			Nell'esempio e' stato creato anche un servizio che fa da Load Balancer
				vedi cmq sotto la descrizione di servizi K8S 
				serve per bilanciare il traffico tra N Pod creati durante un deployment 
				NOTA: il bilanciatore si basa sulla connessione, una volta creata, invia tutte le richieste verso lo stesso Pod!
Creazione di Servizi
	concetti di base
		il servizio e' unico punto di accesso a 1 o piu' Pod
		per esempio non possiamo fidarsi di indirizzi IP dei Pod - cambiano frequentemente, ogni modifica al Pod puo' portare al cambiamento dell'IP
		un Pod ottiene indirizzo IP quando e' stato schedulato 
		lo scopo del servizio e' creare una astrazzione sopra IP 
		qui ritorniamo al concetto di label - possiamo associare delle label ai Pod e servizi - 
		kube-proxy crea un IP virtuale per servizi 
			viene usato livello 4 - TCP/UDP over IP
		un servizio deve essere sempre su, accessibile, e' il punto di ingresso per i client esterni 
		viene creato un endpoint tra il servizio e Pod 
		servizi fanno un load balancing tra i Pods 
			se abbiamo i Pod replicati, il servizio eseguo il load balancing in base alla connessione in ingresso, che arriva da fuori 
			la stessa connessione, finche' e' viva, va verso lo stesso Pod
			se il Pod muore, il servizio trova un Pod nuovo per la stessa connessione 
		NOTA: in K8s ci molto networking all'interno, in questo corso vediamo solo le basi, vedi corsi su pluralsight specifici sul networking di K8s 
	tipi di servizi
		- ClusterIP, viene esposto il servizio sull'IP interno al cluster (default)
			il servizio e' visibile solo all'interno del cluster 
			in questo modo facciamo comunicare N Pods all'interno del cluster
		- NodePort, viene esposto il servizio su ogni IP del Nodo, porta statica
			viene allocata una porta nel range 30000 - 32767
			ogni Nodo proxa la porta allocata verso il Pod finale 
			vedi slide della presentazione - praticamente lato il Nodo (VM, server fisico) viene fatto un port forwarding verso il servizio che gira in locale
				e il servizio fa di nuovo un redirect verso il Pod finale all'inerno del quale puo' girare 1 o piu' container che usano la porta esposta 
		- LoadBalancer, il servizio viene esposto sull'IP esterno che funzionera' come load balancing tra i Pods finali 
			espone il servizio esternamente
			K8s implementa vari bilanciatori di carico (es. nginx, uno di default, etc, vedi sotto)
			un loadbalancer viene visto all'esterno come la coppia di IP:porta - proxa ogni richiesta verso IP e la porta interna di un Nodo del cluster 
			NOTA: anche in questa configurazione ogni Nodo che ospita i Pods ha anche dei servizi locali che espongono i Pods all'interno del cluster 
		- ExternalName, mappa il servizio su un nome del DNS 
			e' un alias 
			e' utile quando vogliamo proxare le chiamate verso un servizio esterno 
			per esempio quando chiamiamo un servizo web terzo - creiamo un servizio ExternalName, e quando cambia IP del servizio chiamato, 
				e' sufficiente aggiornare il nostro servizio ExternalName
	utilizzo di kubectl per creare i servizi
		di default nessun Pod del cluster e' accessibile dall'esterno - abbiamo solo l'IP del cluster 
		pero usando il port forwarding, possiamo esporre un Pod all'esterno 
			kubectl port-forward pod/{pod-name} 8080:80 (comando per mappare la porta locale 8080 sulla porta 80 del Pod)
		stesso comando lo possiamo eseguire anche a livello di Deployment
			kubectl port-forward deployment/{deployment-name} 8080
			in questo caso tutte le richieste alla porta locale 8080 sono mappate alla porta 8080 di deployment
		oppure a livello di servizio
			kubectl port-forward service/{service-name} 8080
		questo comando e' utile per debugging, quando vogliamo accedere ad un Pod specifico da locale 
	utilizzo di YAML per creare i servizi
		struttura base e' la seguente:
		
			apiVersion: v1
			kind: Service
			metadata:
				# il nome del servizio (es. frontend, backend etc.)
			spec:
				type:
					# tipo, default e' ClusterIP
				selector:	
					# qui specifichiamo per esempio i Pods ai quali il servizio deve essere agganciato
				ports:
					# porta esterna 
					# porta interna al container in running sul Pod 
				
		
		esempio di NodePort
		
			apiVersion: v1
			kind: Service
			metadata:
				...
			spec:
				type: NodePort
				selector: 
					app: my-nginx
				ports:
					- port: 80
					  targetPort: 80
					  nodePort: 31000 # e' facoltativo, se non specificato, viene assegnato una porta nel range visto prima (30000 - ...)
					
		esempio di LoadBalancer
		
			apiVersion: v1
			kind: Service
			metadata:
				...
			spec:
				type: LoadBalancer
				selector: 
					app: my-nginx
				ports:
					- port: 80			# porta esposta all'esterno 
					  targetPort: 80	# porta del Pod/container, nel nostro caso e' quella usata da Nginx
					
		esempio di ExternalName
		
			apiVersion: v1
			kind: Service
			metadata:
				name: external-name		# noi usiamo questo nome per fare le chiamate al servizio esterno
			spec:
				type: ExternalName
				externalName: api.myexample.com		# FQN del servizio esterno
				ports:
					- port: 9000
					
		usiamo kubectl
			kubectl create -f file.service.yml	(NOTA: ricordiamo di usare --save-config per salvare la configurazione iniziale utile ad usare il comando apply per aggiornare il servizio in un secondo momento)
			kubectl apply -f file.service.yml
			NOTA: manipolando le label e selector di servizi possiamo cambiare facilmente la configurazione del nostro cluster
			kubectl delete -f file.service.yml
			
			per testare la configurazione, per esempio se un Pod sta raggiungendo un'altro Pod, possiamo eseguire il comando 
				kubectl exec {pod-name} -- curl -s http://podIP
			se curl non e' installato sul container, usiamo 
				kubectl exec {pod-name} -it sh
				viene aperto il terminale dove possiamo aggiungere il curl digitando 
					> apk add curl (NOTA: dipende dalla versione linux utilizzata)
					> curl -s http://podIP
		
			usiamo il comando describe per recuperare i dettagli di un Pod in esecuzione (per es. IP)
			
Opzioni di storage 
	e' un servizio che permette di scambiare i dati tra i Pod all'interno del cluster K8s
	viene introdotto il concetto di Volumes
		abbiamo una parte dello stato dell'applicazione che puo' essere persistita
		i dati rimangono anche se un Pod viene riavviato 
	un Pod puo' avere piu' Volumes agganciati 
	un Container fa riferimento a mountPath per accedere al volume di dati 
	k8s supporta 
		- Volumes
			referenzia una storage location
			ha un nome univoco
			agganciata al Pod e' puo' essere legata o NO al ciclo di vita del Pod (dipende dal tipo di volume, vedi sotto)
			il nome di un Volume referenzia un mountPath 
		- PersistentVolumes
		- PersistentVolumeClaims
		- StorageClasses
	
	Volume types 
		emptyDir - una directory vuota utile a salvare i dati temporanei condivisi tra i container del Pod, condivide il ciclo di vita del Pod!
		hostPath - Pod esegue il mounting sul filesystem del Nodo, qui siamo legati al ciclo di vita del Nodo!
		nfs - Pod esegue il mounting di una cartella di Network File System, una cartella condivisa nella rete di Pods 
		configMap/secret - e' un tipo speciale di volume che rende le risorse di K8s accessibili da parte di Pods 
		persistentVolumeClaim - e' un volume persistito, il Pod ha piu' controllo sulla persistenza usando delle astrazioni specifiche
		cloud - cluster-wide storage 
		NOTA: ci sono altri N tipi di volumi, vedi la slide, per es. azureDisk, azureFile, aws*, e altri...
			dipende dalle scelte dell'ambiente di produzione
		
		esempio di emptyDir:
		
			apiVersione: v1
			kind: Pod
			spec:
			  volumes:
				- name: html	# importante, viene usato sotto 
				  emptyDir: {}
			  containers:
				- name: nginx
				  image: nginx:alpine
				  volumeMounts:
				    - name: html						# referenziamo il volume configurato prima
					  mountPath: /usr/share/nginx/html	# il volume html e' stato montato su questo path all'interno di container 
					  readOnly: true
				- name: container2
				  image: ...
				  volumeMounts:
				    - name: html						# abbiamo un secondo container che utilizza lo stesso volume 
					  mountPath: /html					
		
		esempio di hostPath (esempio riguarda uno possibile scenario di integrazione con il docker daemon che gira sul Nodo)
		
			apiVersione: v1
			kind: Pod
			spec:
			  volumes:
				- name: docker-socket
				  hostPath:
				    path: /var/run/docker.sock
					type: Socker				# connessione socker con docker daemon 
												# altri tipi supportati sono: DirectoryOrCreate, Directory, FileOrCreate, File, Socker, CharDevice, BlockDevice 
			  containers:
			    - name: docker
				  image: docker
				  command: ["sleep"]
				  args: ["100000"]
				  volumeMounts:
				    - name: docker-socket
					  mountPath: /var/run/docker.sock 
					  
			NOTA: questo tipo di volume e' sul worker node (VM), se muore il nodo perdiamo i dati
			
		i provider cloud che forniscono uno servizio di storage affidabile:
			Azure - Azure Disk / Azure File
			AWS - Elastic block store 
			GCP - GCE Persistent Disk 
		
			esempio x Azure:
			
				apiVersione: v1
				kind: Pod
				metadata:
				  name: my-pod
				spec:
				  volumes:
					- name: data
					  azureFile:
					    secretName: <azure-secret>
						shareName: <share-name>
						readOnly: false
				  container:
				    - name: <container-name>
					  image: <image-name>
					  volumeMounts:
						- name: data
						  mountPath: /data/storage 
			
			NOTA: fare riferimento alla documentazione del cloud provider per impostazioni di dettaglio 
		utilizzo di kubectl
			per vedere i volumi montati usiamo sempre il comando kubectl describe pod {pod-name}
			oppure
				kubectl get pod {pod-name} -o yaml 
				
		vedi esempi di file yaml nel repo 
	PersistentVolumes e PersistentVolumeClaims
		PersistentVolume e' il riferimento ad uno storage aziendale per esempio, gestito dall'amministratore dedicato (se previsto)
		PersistentVolumeClaim e' la richiesta ad un PersistentVolume, e' quello che usiamo noi nel momento di definizione di un Pod o Deployment 
		PersistentVolume e' uno storage cluster-wide (accessibile dal cluster)
			molto spesso si usa un network-attached storage (NAS) 
			es. in una rete locale, cloud 
			non e' specifico per un Pod o Worker Node
			viene usato un storage provider, per es. NFS, cloud storage. 
			viene associato al Pod usando PVC (PersistentVolumeClaim)
			PersistentVolume viene registrato nel cluster usando le API di K8s
			vedi slide con lo schema di iterazioni...
	PersistentVolumes e PersistentVolumeClaims YAML
		NOTA: e' sempre un po' tricky avere un YAML corretto per fare tutte le impostazioni necessarie,
			fare riferimento sempre agli esempio disponibili su github del profilo di Kubernetes
			vedi https://github.com/kubernetes/examples
		
		Step da seguire:
			1. configurare PersistentVolume
			2. configurare PersistentVolumeClaim
			3. referenziare il claim all'interno di Pod 
		
		1. esempio della porzione di YAML per PersistentVolume:
		
			apiVersion: v1
			kind: PersistentVolume
			metadata:
				name: my-pv
			spec:
				capacity: 10Gi
				accessModes: 
				  - ReadWriteOnce	# solo un Pod puo' eseguire il mounting con permessi di scrittura
				  - ReadOnlyMany	# piu' Pod possa richiedere il mounting in sola lettura
				persistentVolumeRelaimPolicy: Retain	# non fare niente con il volume anche se il claim viene eliminato 
				azureFile:								# impostazioni per il servizio Azure File 
				  secretName: <azure-secret>
				  shareName: <name-from-azure>
				  readOnly: false 
		
		2. esempio della pozione YAML per PersistentVolumeClaim (e' una richiesta al PV)
		
			kind: PersistentVolumeClaim
			apiVersion: v1
			metadata:
			  name: pv-hdd-5g
			spec:
			  accessModes:
			    - ReadWriteOnce
			  resources:
			    requests:
				  storage: 5Gi		# richiesta di 5GB di 10GB disponibili
				  
		3. agganciamo il claim al Pod
		
		   kind: Pod
		   apiVersion: v1
		   metadata:
		     name: pod-uses-hdd-5g
			 labels:
			   name: storage
		   spec:
		     containers:
			   - name: my-nginx 
			     image: nginx
				 volumeMounts: 
				   - name: blobdisk01
				     mountPath: /mnt/blobdisk 
			 volumes:
			   - name: blobdisk01
			     persistentVolumeClaim: 
				   claimName: pv-hdd-5g
			
	Classi dello storage (StorageClasses)
		e' un tipo di template che puo' essere usato per fare il provisioning dinamico dello storage 
		per esempio un amministratore puo' fornirci il template che possiamo usare 
		possiamo definire diversi classi dello storage 
		gli step da seguire per configurare uno StorageClasse
			1. Creiamo Storage Class
			2. PVC referenzia la classe creata prima 
			NOTA: non esiste ancora PV, il suo provisioning viene fatto in modo dinamico 
			una volta che viene creata la PV in modo dinamico, viene fatto l'agancio con PVC di prima in modo automatico 
				e in questo modo abbiamo lo storage configurato
		esempio di YAML 
		
			apiVersion: storage.k8s.io/v1
			kind: StorageClass
			metadata
				name: local-storage
			reclaimPolicy: Retain 			# lo storage rimane se il claim viene eliminato 
			provisioner: kubernetes.io/no-provisioner	# metendo no-provisioner dobbiamo creare il PV, 
														# possiamo specificare altri valori usando il meccanismo di auto provisioning
			volumeBindingMode: WaitForFirstConsumer		# in questo caso il volume NON viene creato finche' non abbiamo il primo POD che lo sta usando avviato 
			
		esempio di YAML che riguarda il PersistentVolume
		
			apiVersion: v1
			kind: PersistentVolume
			metadata
			  name: my-pv
			spec:
			  capacity: 
			    storage: 10Gi
			  volumeMode: Block
			  accessModes:
			    - ReadWriteOnce
			  storageClassName: local-storage	# referenziamo lo storage class creato prima 
			  local:
			    path: /data/storage 	# e' il path sul worker node (VM)
			  nodeAffinity:								# con questa sezione selezioniamo il Nodo dove il PV viene creato 
			    required:
				  nodeSelectorTerms:
				    - matchExpressions:
					  - key: kubernetes.io/hostname		# usiamo il nome del worker node
					    operator: In
						values: 
						  - <node-name> 				# qui specifichiamo il nome del worker node dove vogliamo che il PV viene creato 
						  
		esempio di YAML che riguarda il PersistentVolumeClaim
		
			apiVersion: v1
			kind: PersistentVolumeClaim
			metadata:
				name: my-pvc
			spec:
				accessModes:
				  - ReadWriteOnce
				storageClassName: local-storage			# referenziamo il class name, in questo modo abbiamo PVC configurato per utilizzo anche di un provisioning dinamico
														# NOTA: il nostro esempio cmq crea PV in modo statico, per seplicita
				resources:
				  requests:
				    storage: 1Gi
		
		come ultima cosa, usiamo il volume a livello di Pod/Deployment
		
			apiVersion: v1
			kind: [Pod | Deployment]
			...
			spec:
			  volumes:
			    - name: my-volume
				  persistentVolumeClaim:
				    claimName: my-pvc 
					
	Demo utilizzo di StorageClasses
		vedi i file nel repo
		viene usato StatefulSet nella creazione di container
			usando StatefulSet possiamo prevedere i nodi di Pod creati (in base a replicas, viene impostato un indice 0 based nel nome, suffisso del nome
		ricordiamo che StorageClasses sono usati per creare in modo dinamico PV
Creazione di ConfigMaps e Secrets
	fanno parte dell'area Storage (vedi slide del corso)
	ConfigMaps
		e' il modo di salvare la configurazione e renderla disponibile ai container
		i nostri container, indipendentemente dai Pod e worker node sono in grado di accedere a questi informazioni di configurazione 
		praticamente iniettiamo i dati di configurazione nei nostri container
		possiamo usare i file di configurazione o una mappa chiave-valore
			- se usiamo il file, il nome del file fa dalla chiave, il contenuto e' il valore
				file puo' essere in formato JSON, XML, o un insieme di righe chiave=valore
			- possiamo passare la configurazione al comando kubectl
			- usare ConfigMap manifest
		ConfigMaps sono accessibili dai Pod usando
			- variabili di ambiente (chiave/valore)
			- ConfigMap volume (accesso ai file)
	Creazione di ConfigMaps
		possiamo creare un manifest dedicato, di tipo ConfigMap
		
			apiVersion: v1
			kind: ConfigMap
			metadata:
			  name: app-settings
			  labels:
			    app: app-settings
			data:						# sezione contenente i dati di configurazione recuperati in un secondo momento usando il name app-settings
			  enemies: aliens			# nota la proprieta' enemies che contiene i dati di configurazione complessi (gerarchia style)
			  lives: "3"
			  enemies.cheat: "true"
			  enemies.cheat.level: "custom level"
				
		per aggiungere questi dati di configurazione a K8s eseguiamo il comando
			kubectl create -f file.configmap.yml
		
		si puo' caricare i dati di configurazione in K8s partendo da un file di properties, per es.
			abbiamo il file custom.config che contiene le righe
				enemies=aliens
				lives=3
				enemies.cheat=true
				enemies.cheat.level=customLevel
			per creare una ConfigMap partendo da questo file eseguiamo il comando
				kubectl create configmap [cm-name] --from-file=[path-to-file]
			il risultato di questo comando e' un po' diverso da quello visto prima (cioe' creazione ConfigMap partendo dal file di manifesto completo)
				praticamente, la parte data diventa cosi:
				
					data:
						custom.config: |-
							enemies=aliens
							lives=3
							enemies.cheat=true
							enemies.cheat.level=customLevel
							
				abbiamo la chiave che corrisponde al nome del file
				il pipe |- viene usato per marcare inizio dei dati di configurazione - NOTA: questi dati possano esserci in formato JSON, XML e key-value pairs 
				come abbiamo visto prima
			un'altra possibilita' che abbiamo e usare il comando
				kubectl create configmap [cm-name] --from-env-file=[path-to-file]
				il risultato di questo comando e' la creazione di una ConfigMap con sezione data senza la chiave = nome del file
				praticamento questo e' il modo per iniettare la configurazione in K8s in base all'ambiente (staging, test, prod)
			ultimo approccio (usato meno frequentemente) e' usare il comando sotto per caricare la config dalla linea di comando
				kubectl create configmap [cm-name] 
				  --from-literal=apiUrl=https://my-api
				  --from-literal=otherKey=otherValue
				questo approccio viene usato raramente, di solito abbiamo la nostra configurazione versionata su un repo per vari ambienti 
			
	Utilizzo di ConfigMap
		per recuperare una ConfigMap usiamo il comando
			kubectl get cm [cm-name] -o yaml
		per usare la configurazione da un container, ecco la porzione della sintassi da usare nel file di manifesto
			apiVersion: apps/v1
			...
			spec:
			  template:
			    ...
			  spec:
			    containers: ...
				env:
				  - name: ENEMIS		# variabile di ambiente usata dai container per recuperare il valore
				    valueFrom:
					  configMapKeyRef:
					    key: app-settings	# nome della ConfigMap 
						name: enemis		# nome della proprieta' all'interno di ConfigMap, il suo valore viene assegnato alla variabile ENEMIS definita prima
						
		invece se vogliamo caricare tutta la configurazione usiamo seguente template
		
			apiVersion: apps/v1
			...
			spec:
			  template:
			    ...
			  spec:
			    containers: ...
				  envFrom: 
				    - configMapRef: 
					  name: app-settings
					  
			in questo caso sono creati N variabili di ambiente, il nome corrisponde alla chiave di configurazione usata nella definizione di ConfigMap (es. enemis)
			il primo approccio serve a caricare una chiave individuale
			
		
		la terza possibilita' che abbiamo e' creare la configurazione usando i volumes
		in questo caso ogni chiave di configurazione diventa un file a parte contenente il valore associato alla chiave 
		se ConfigMap viene aggiornata K8s (con qualche delay, es. anche 60sec) aggiorna automaticamente i file di configurazione 
		se volgiamo usare questo approccio, il file di manifesto diventa:
		
			apiVersion: apps/v1
			...
			spec:
			  template:
			    ...
			  spec:
			    volumes:
				  - name: app-config-vol
				    configMap:
					  name: app-settings		# se teniamo esempio di prima di ConfigMap, alla fine abbiamo 4 file creati 
				containers:
				  volumeMounts:
				    - name: app-config-vol
					  mountPath: /etc/config	# il container usa questo path per caricare i file di configurazione (sta al sw di eseguire il caricamento della configurazione necessaria)
			
		variabili di ambiente sono utili quando abbiamo la configurazione che non cambia frequentemente 
		invece se abbiamo l'esigenza di caricare la configurazione dai file, ultimo approccio visto puo' essere utile 
	ConfigMaps in action
		vedi esempi e readme.md nel repo
	Secrets concetti core
		e' un oggetto che contiene i dati sensibili (es. password, token, key, certificato etc)
		in questo modo non dobbiamo salvare le nostre pwd in file, nelle immagini o file di manifesto 
		possiamo fare il mounting di questi dati nei Pod usando i volumes o variabili di ambiente visti prima con ConfigMap
		Kubernetes mette i dati sensibili a disposizione solo di Pods che ne hanno bisogno 
		i secrets sono salvati nel tmpfs su un Nodo 
		per abilitare la encryption sui dati del cluster fare riferimento alla doc. link https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
		e' opportuno limitare accesso a etcd (che e' sul nodo master), dove i secrets sono salvati, a solo amministratori di sistema 
		e' opportuno usare SSL/TLS per le comunicazioni con etcd 
		e' possibile mettere i dati sensibili nel file di manifesto ma in questo modo i dati sono solo codificati in base64 e dobbiamo essere sicuri
			che i file di manifesto sono in un posto protetto
		i Pods possano accedere ai dati di Secrets, quindi e' meglio mettere un livello di sicurezza anche su chi puo' creare i Pods 
			possiamo usare RBAC (Role Based Access Control)
	Creazione di secrets
		possiamo usare i comandi di kubectl
			se vogliamo passare i dati dalla linea di comando
				kubectl create secret generic [secret-name] --from-literal=pwd=my-pwd 
			se vogliamo passare i dati da un file 
				kubectl create secret generic [secret-name] --from-file=ssh-privatekey=~/.ssh/id_rsa
					--from-file=ssh-publickey=~/.ssh/id_rsa.pub
			se vogliamo passare i dati del certificato
				kubectl create secret tls [secret-name] --cert=path/to/tls.cert --key=path/to/tls.key
		possiamo usare direttamente il file di manifesto YAML (attenzione che in questo caso i dati sono codificati in Base64, e se versionati su un repo, 
			chiunque accede al repo puo' facilmente decodificarli)
			esempio di file YAML 
			
				apiVersion: v1
				kind: Secret
				metadata:
				  name: db-passwords
				type: Opaque
				data:
				  db-password: ZmFzbGRmamFzbA==
				  admin-password: ZnNsZmRqZg==
				  
			usiamo il solito comando kubectl apply -f secret.file.yml
	Utilizzo di secrets 
		per leggere secrets lanciamo 
			kubectl get secrets 
			kubectl get secrets db-passwords -o yaml
		per usare secrets dai Pods il file di manifesto fa riferimento al secret usando il nome, es:
		
			apiVersion: apps/v1
			...
			spec:
			  containers: ...
			  env:
			    - name: DATABASE_PASSWORD	# viene creata la variabile di ambiente con questo nome 
				  valueFrom:
				    secretKeyRef: 
					  name: db-passwords	# il nome di Secret  
					  key: db-password
				    
		se vogliamo usare i volumi (approccio simile visto gia' con ConfigMap, per ogni chiave viene creato il file con contenuto = al valore della chiave)
		
			apiVersion: apps/v1
			...
			spec:
			  template:
			    ...
			  spec:
			    volumes:
				  - name: secrets
				    secret:
					  secretName: db-passwords
				containers:
				volumeMounts:
				  - name: secrets
				    mountPath: /etc/db-passwords
					readOnly: true
	Secrets in action
		vedi i file di esempio nel repo 
Unione di tutto (demo di una app)
	app ha un server nginx, server di node, usa mongodb e redis 
		ci sono i file di configurazione divisi per ambiente (test, production)
		per lanciare app e' richiesto di impostare 2 variabili di ambiente nella linea di comando 
			variabile1 - contiene ambiente che stiamo lanciando (es. test / production)
			variabile2 - contiene accounte da usare per immagini docker locali 
		per ogni parte abbiamo un immagine custom (cartella .docker)
		per ogni parte che vogliamo deployare abbiamo il file *.yml (cartella .k8s)
		cartella config contiene i file di configurazione per ogni ambiente 
		per buildare immagini usiamo docker-compose
			due variabili di ambiente menzionati prima devono essere settati prima di lanciare docker-compose
			buildiamo le nostre immagini lanciando docker-compose build 
		come passo successivo creiamo Secrets che servono per il deploy in Kubernetes cluster 
		lanciamo il deploy usando il comando
			kubectl apply -f .k8s (NOTA: passiamo il riferimento alla cartella contenente N file di deploy)
			kubectl delete -f .k8s (per eliminare le risorse create prima con il comando apply)
		giusto come promemoria, StatefulSet e' un tipo di deploy, usato in questa demo per il DB di Mongo
		vedi il repo di autore per il codice sorgente della demo 
	tecniche di troubleshooting
		kubectl logs [pod-name]
		kubectl logs [pod-name] -c [container-name]
		kubectl logs -p [pod-name]	# per stampare i log di un Pod che era in esecuzione nel passato 
		kubectl logs -f [pod-name]  # per fare lo streaming di log 
		
		kubectl describe pod [pod-name] # utile anche per vedere gli eventi su un Pod, oltre ai dettagli del Pod 
		kubectl get pod [pod-name] -o yaml 	# configurazione del Pod in dettaglio 
		kubectl get deployment [deployment-name] -o yaml 
		
		kubectl exec [pod-name] -it sh 		# aprire il terminale in un container in un pod 
		
		kubectl get pods --watch 	# rimaniamo in ascolto sullo stato di pod, per esempio se sono in fase di stop e' utile vedere quando tutti i pod si sono fermati
		
		demo riguardava due problematiche:
			1. errore nella configurazione dell'app -> nome del server di mongodb era sbagliato -> configurazione veniva inserita nel container -> una volta corretta 
				la configurazione abbiamo rifatto la build di immagini e rieseguito il deploy nel cluster k8s
			2. il deploy in k8s e' stato lanciato senza creare prima i Secrets necessari -> eliminate tutte le risorse di deploy -> create Secrets 
					-> rifatto il deploy 

		