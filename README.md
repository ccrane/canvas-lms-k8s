# Canvas LMS - Local Deployment setup

### Local deployment of Canvas LMS using Helm Charts
1. Install [kubectl](https://kubernetes.io/docs/tasks/tools/).
- Install [minikube](https://minikube.sigs.k8s.io/docs/start/).
- Start the minikube cluster
    - `minikube start`
- Verify minikube cluster is running.
    - `minikube status`
- Install [Helm](https://helm.sh/docs/intro/install/)
- In the terminal window, change directory to the `canvas-lms/helm` directory.
- Run the following command to install the Canvas LMS Helm Chart
    - `helm install --generate-name ./canvas-lms-app`
- Verify that all resources are deployed and running
    - `kubectl get all`
    - This should show 6 total services: 5x services above (eg, postgres-service.yaml) + kubernetes service
      - The 5x services listed above should be listed as 'pods' and next to them the status should show "Running"
      - On first run this could take 20 minutes or so for status to update from "ContainerCreating" to "Running" as large images are downloaded. 
      - You can type `kubectl describe pod` to see the status 
        - look for 'events' under each service and there should be a status such as: 'scheduled', 'pulling', 'pulled', 'created', 'started'
        - if the last event is 'pulling' then you know the image is downloading and you must wait.
- Perform initial Canvas setup
    - `kubectl exec -it canvas-lms-0 -- sh` 
      - After executing this your terminal will be in the shell for Canvas
    - `bundle exec rake db:create`
    - `bundle exec rake db:migrate` 
      - Wait for a while, after a few mins you should see several tables being created and migrations completed in stdout
      - This will take ~20min in total
      - **NOTE:** This sometimes fails on a migration, if it does just re-run `bundle exec rake db:migrate` to complete the migrations
    - `bundle exec rake db:initial_setup`
      - This will require you to answer some questions in the terminal (eg, admin username/password)
    - `bundle exec rake canvas:compile_assets`
      - This can take some time (~20min)
    - `bundle exec rake brand_configs:generate_and_upload_all`
      - This should be far quicker than previous steps
    - `exit`
- Get the URL associated with the `canvas-lms-0` service and open in a browser.
    - `minikube service list`
- Verify login functionality
    - Attempt to login using the email address and password you supplied during the initial setup of canvas
    - **NOTE:** If you encounter an error during login run the following command to resolve the issue
        - `kubectl rollout restart sts canvas-lms`


### Local deployment of Canvas LMS using Kubernetes and Minikube

1. Install [kubectl](https://kubernetes.io/docs/tasks/tools/).
- Install [minikube](https://minikube.sigs.k8s.io/docs/start/).
- Start the minikube cluster
    - `minikube start`
- Verify minikube cluster is running.
    - `minikube status`
- In the terminal window, change directory to the `canvas-lms` directory.
- Run the following commands to create Canvas services.
    - `kubectl apply -f postgres-configmap.yaml`
    - `kubectl apply -f postgres-secret.yaml`	
    - `kubectl apply -f postgres-service.yaml`
    - `kubectl apply -f mail-configmap.yaml`
    - `kubectl apply -f mail-secret.yaml`
    - `kubectl apply -f mail-service.yaml`
    - `kubectl apply -f redis-service.yaml`
    - `kubectl apply -f canvas-configmap.yaml`
    - `kubectl apply -f canvas-service.yaml`
    - `kubectl apply -f canvas-worker-service.yaml`
- Verify all Canvas services are running
    - `kubectl get all`
      - This should show 6 total services: 5x services above (eg, postgres-service.yaml) + kubernetes service
      - The 5x services listed above should be listed as 'pods' and next to them the status should show "Running"
      - On first run this could take 20 minutes or so for status to update from "ContainerCreating" to "Running" as large images are downloaded. 
      - You can type `kubectl describe pod` to see the status 
        - look for 'events' under each service and there should be a status such as: 'scheduled', 'pulling', 'pulled', 'created', 'started'
        - if the last event is 'pulling' then you know the image is downloading and you must wait.
- Perform initial Canvas setup
    - `kubectl exec -it canvas-lms-0 -- sh` 
      - After executing this your terminal will be in the shell for Canvas
    - `bundle exec rake db:create`
    - `bundle exec rake db:migrate` 
      - Wait for a while, after a few mins you should see several tables being created and migrations completed in stdout
      - This will take ~20min in total
      - This sometimes fails on a migration, if it does just re-run `bundle exec rake db:migrate` to complete the migrations
    - `bundle exec rake db:initial_setup`
      - This will require you to answer some questions in the terminal (eg, admin username/password)
    - `bundle exec rake canvas:compile_assets`
      - This can take some time (~20min)
      - you may need to kill this process if it gets stuck at the end after outputting "Done [###]s"
    - `bundle exec rake brand_configs:generate_and_upload_all`
      - This should be far quicker than previous steps
    - `exit`
- Get the URL associated with the `canvas-lms-0` service and open in a browser.
    - `minikube service list`
    - NB: if you are using OSX, this final step will NOT work. Various links that may help in de-bugging/workarounds:
      - https://github.com/kubernetes/minikube/issues/11193
      - https://stackoverflow.com/questions/66873437/access-minikube-cluster-on-the-same-network/66945182#66945182
      - https://minikube.sigs.k8s.io/docs/handbook/accessing/ (see DNS resolution heading re: docker driver not working on Macs)
      - https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/
- Verify login functionality
    - Attempt to login using the email address and password you supplied during the initial setup of canvas
    - **NOTE:** If you encounter an error during login run the following command to resolve the issue
        - `kubectl rollout restart sts canvas-lms`

### Local deployment of Canvas LMS using docker and docker-compose.

1. Setup [docker](https://docs.docker.com/get-docker/) for your environment.

2. Setup [docker-compose](https://docs.docker.com/compose/install/) for your environment.

3. In GitLab, create or use an existing, personal access token.

4. In a terminal window, docker login to the GitLab registry using your GitLab username and personal access token created in step 3.
  `docker login registry.gitlab.com`

5. Pull the Canvas LMS image from the GitLab education-module container registry.
  `docker pull registry.gitlab.com/onestepprojects/education-module/canvas-lms:stable`

6. Verify the Canvas LMS image is in your docker images repository.
  `docker images`  
   You should see `registry.gitlab.com/onestepprojects/education-module/canvas-lms` listed as a repository, and `stable` listed as the tag.

7. In a terminal window, change directory to the location of the `education-module`, and verify the existence of the `canvas-lms` directory.

8. In the terminal window, change directory to the `canvas-lms` directory.

9. In the terminal window, spin up the images and performance initial setup commands:
   1. `mkdir .data && sudo chmod -R 777 .data` *-- grant full access to the data store for the container*
   2. `docker-compose up -d db` *-- starts up all the database container identified in the `docker-compose.yml` file.*
   3. `docker-compose run --rm app bundle exec rake db:create db:initial_setup` *-- initialized the canvas lms database, this only needs to run once*
   4. `docker-compose down` *-- shutdown all containers*
   5. `sudo nano /etc/hosts` and add the line `127.0.0.1 docker_canvas_app` to the file, `ctrl-x` to exit
   6. `docker-compose up -d`  *-- starts up all the containers identified in the `docker-compose.yml` file.*
   7. `docker ps -a` *-- shows a list of running docker containers, confirm one of the running containers is named canvas-lm_app_1*
   8. `docker exec -u root -it canvas-lms_app_1 /bin/bash` *-- opens a shell into the canvas-lms_app_1 container*
   9. `bundle exec rake canvas:compile_assets` *-- creates all canvas assets*
   10. `bundle exec rake brand_configs:generate_and_upload_all` *-- generate css assets*
   11. `exit` *-- exist the contain shell*
10. If all steps are completed successfully, open a browser and vist http://docker_canvas_app:8900/
 
### Start Server
`docker-compose up -d` 
 
### Check Logs
#### app
`docker logs -f docker-canvas_app_1`

#### more detailed app logs
`docker exec -it docker-canvas_app_1 tail -f log/production.log`

#### worker
`docker logs -f docker-canvas_worker_1`

#### db
`docker logs -f docker-canvas_db_1`

#### redis
`docker logs -f docker-canvas_redis_1`

#### mail
`docker logs -f docker-canvas_mail_1`

### Shutdown Server
`docker-compose down`
  
  