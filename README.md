# Helm Repo creation for multiple apps & publish to single repo 

### Step-1: Helm package for each application
  
  ````bash
  git clone https://github.com/hmsvigle/app-01.git
  git clone https://github.com/hmsvigle/app-02.git
  ````

### Step-2: Create Helm chart for each application
   #### 2.1. Test helm charts by installing in lower environment  
   ````bash
   helm upgrade --install  app-01 tibco/chart/ -n stg \
   -f tibco/chart/app-01/values.yaml \
   -f tibco/environments/stg-values.yaml \
   -f tibco/chart/app-01/environments/stg-values.yaml \
   --set appNameGeneric=app-01 \
   --set appName=app-01
   ````
   #### 2.2. Create Package
    
   ````bash
    $ cd app-01
    $ helm package tibco/chart   
      Successfully packaged chart and saved it to: /root/app-01/app-01-0.1.0.tgz
   ````

   #### 2.3. Repeat the Steps `2.1` & `2.2`  for app-02 & all other applications
   
   #### 2.4. Move Packages to `helm_repo` repository. 
   ```bash
    cp app-01/app-01-0.1.0.tgz helm_repo/
    cp app-02/app-02-0.1.0.tgz helm_repo/ 
   ```
   
   #### 2.5. Create/Update index with all packages 
   
    * If it is the first time, create Index i.e `helm repo index . --url https://hmsvigle.github.io/helm_repo/`
    * If not first time, merge/update the index i.e `helm repo index . --merge index.yaml --url https://hmsvigle.github.io/helm_repo/`
   
   #### 2.6. Commit index.yaml to `helm_repo` git
   ```bash
   git add .
   git commit -m "Added app-01 and app-02 Helm charts"
   git push origin main
   ```

### Step-3: Install Helm package to any environment
  #### 3.1. Add Helm Repo 
  ```bash
  $ helm repo add my-helm-repo https://hmsvigle.github.io/helm_repo/
  $ helm repo list
    NAME                    URL                                  
    hmsvigle_helm_repo      https://hmsvigle.github.io/helm_repo/
  ```

  #### 3.2. Update the helm repo
  
  ```bash
  $ helm repo update
  ```
  
  #### 3.3. Install specific helm application from that helm repo
   * Search a specific application repo from the helm repo
   ```bash
    $ helm search repo hmsvigle
    NAME                            CHART VERSION   APP VERSION     DESCRIPTION                
    hmsvigle_helm_repo/app-01       0.1.0           1.0.0           A Helm chart for Kubernetes
    hmsvigle_helm_repo/app-02       0.1.0           1.0.2           A Helm chart for Kubernetes
   ```
   * All application values yamls should be updated at single repo in git. That should be referred everytime we deploy to any environment
   ```bash
    $ helm install app-01 hmsvigle_helm_repo/app-01 -n reg \
      -f tibco/apps/app-01/values.yaml \
      -f tibco/environments/reg-values.yaml \
      -f tibco/apps/app-01/environments/reg-values.yaml \
      --set appNameGeneric=app-01 \
      --set appName=app-01
   ````
  #### 3.4. verify application deployment 
   ````bash
    controlplane $ k get po -n reg
      NAME                                 READY   STATUS    RESTARTS   AGE
      app-01-deployment-85ffd68d86-h7bnp   1/1     Running   0          82s
   ````
  
