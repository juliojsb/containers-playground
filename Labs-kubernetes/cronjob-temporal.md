	$ cat job-hello.yaml
 
	apiVersion: batch/v1
	kind: Job
	metadata:
	  name: job-hello-1
	spec:
	  template:
	    metadata:
	      name: hello
	    spec:
	      containers:
	      - name: hello
	        image: busybox:1.28
	        command:
	         - "/bin/sh"
	         - "-c"
	         - "echo \"Hello world!! The date is $(date)\" ; sleep 5"
	      restartPolicy: Never

	$ k apply -f job-hello.yaml

	$ k get job
 
Este job crea un pod para realizar la tarea:

	$ k get pod
	NAME                    READY   STATUS    RESTARTS       AGE
	job-hello-rmjgt         1/1     Running   0              19s

Revisamos logs:

$ k logs job-hello-rmjgt
Hello world!! The date is Thu Aug  3 16:38:21 UTC 2023

Una vez completado, el pod queda en estado Completed 0/1

$ k get pod
NAME                    READY   STATUS      RESTARTS     AGE
job-hello-rmjgt         0/1     Completed   0            25s

Y el job ha finalizado correctamente:

$ k get job
NAME          COMPLETIONS   DURATION   AGE
job-hello     1/1           23s        4m37s

¿Si queremos que se ejecute de forma periódica? -> Cronjob

El schedule se planifica siguiendo el estándar cron:
	
	# ┌───────────── minute (0 - 59)
	# │ ┌───────────── hour (0 - 23)
	# │ │ ┌───────────── day of the month (1 - 31)
	# │ │ │ ┌───────────── month (1 - 12)
	# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
	# │ │ │ │ │                                   7 is also Sunday on some systems)
	# │ │ │ │ │
	# │ │ │ │ │
	# * * * * * 

$ cat cronjob-hello.yaml 
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - "/bin/sh"
            - "-c"
            - "echo \"Hello world!! The date is $(date)\" ; sleep 5"
          restartPolicy: Never

	$ k apply -f cronjob-hello.yaml

$ k get cronjob
NAME            SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cronjob-hello   * * * * *   False     1        31s             81s

Se irán creando los jobs:

$ k get job
NAME                     COMPLETIONS   DURATION   AGE
cronjob-hello-28184701   1/1           9s         2m11s
cronjob-hello-28184702   1/1           8s         71s
cronjob-hello-28184703   1/1           8s         11s

Y los pods correspondientes:

$ k get pod
NAME                           READY   STATUS               RESTARTS      AGE
cronjob-hello-28184701-gbnfq   0/1     Completed            0             2m27s
cronjob-hello-28184702-dqzcv   0/1     Completed            0             87s
cronjob-hello-28184703-prbsr   0/1     Completed            0             27s

Revisamos los logs:

$ k logs cronjob-hello-28184701-gbnfq
Hello world!! The date is Thu Aug  3 17:01:01 UTC 2023
$ k logs cronjob-hello-28184702-dqzcv
Hello world!! The date is Thu Aug  3 17:02:01 UTC 2023
$ k logs cronjob-hello-28184703-prbsr
Hello world!! The date is Thu Aug  3 17:03:01 UTC 2023
