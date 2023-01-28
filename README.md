# WD-App

L’obiettivo principale del progetto è fornire un installazione vergine di wordpress hostata su AWS.

### Disegno infrastrutturale

![alt text](https://github.com/stead1995/wd-fargate/blob/master/wd.png)

### Prerequisiti
1.aws-cli (versione 2.9.13)
2.sam (versione 1.68.0)
3.docker (versione 20.10.22)

# Parametri

Tabella riassuntiva dei valori dei parametri da impostare nel deploy

| Nome parametro  | Descrizione |
| ------------- | ------------- | 
| ClusterName  | Nome del cluster ecs che si desidera creare  | 
| WDRelease  | Tag immagine ecr dell'applicativo. N.B. : Si consiglia di utilizzare l'hash della commit  |  
| Cpu  | Valore della cpu su cui si vuole fare eseguire il container. Il valore è espresso in MB  | 
| Memory  | Valore della memoria su cui si vuole fare eseguire il container. Il valore è espresso in MB  | 
| DesiredCapacity  | Numero di task in eseguizione del nostro servizio(nel primo deploy impostarlo a 0)  | 
| VpcId  | Id della rete su cui verrà isolato l'applicativo  | 
| Subnets  | Sottoreti della rete specificata in precedenza  | 

### Primo Deploy

Ottenere le credenziali dell'account su cui si desidera deployare l'applicativo ed impostare il ruolo nell'apposito file di credenziali.

Eseguire il primo deploy in modalità guidata dove dovranno essere inseriti i vari parametri(fare riferimento alla tabella riassuntiva dei parametri).

Eseguire il login sull'ecr dell'account AWS. Eseguire il segunte comando : 

```sh
$ sam build && sam deploy --guided --profile [PROFILO_AWS] --capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND CAPABILITY_NAMED_IAM --config-env dev
```

Ottenre il link dell'ECR spostarsi nella cartella app ed eseguire i seguenti comandi : 

```sh
$ aws ecr get-login-password --profile [PROFILO] --region [REGION] | docker login --username AWS --password-stdin [ACCOUNT_ID].dkr.ecr.[REGION].amazonaws.com
```

Spostarsi nella cartella app e buildare e pushare sul repository sull'ecr appena creato :

```sh
$ docker build -t [LINK_ECR]:[REALESE] -f Dockerfile_prod .
$ docker push [LINK_ECR]:[REALESE]
```

Eseguire il deploy eseguendo il seguente comando impostato il parametro di DesiredCapacity al valore che si preferisce(maggiore di 0) e la WDRelease con la realese pushata sul repository dell'ecr in precedenza.

```sh
$ sam build && sam deploy --guided --profile [PROFILO_AWS] --config-env dev
```

### Deploy

```sh
$ sam build && sam deploy --profile [PROFILO_AWS] --config-env dev
```

### Accedere all'applicazione

Nella console aws eseguire i seguenti passaggi : 

1. Nella console AWS accedere al servizio ecs.
2. Selezionare il cluster desiderato
3. Spostarsi nella tab dei task selezionare il task dell'app interessato
4. Nella sezione "Network" recuperare il valore del public ip e accedere via browser.