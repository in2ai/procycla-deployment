# procycla-deployment
Repository with code to deploy Procycla project


## Steps to follow for deployment

* Install `docker`, `kubectl` and `doctl`
* Follow DIgital Ocean instructions to log in with doctl, using 
* Once installed go to Digital Ocean and in the tab COntainer Registry create one, and follow the instructions to connect to the registry just created (select integrate with all kubernetes clusters so you don't have to manually add it to your kubernetes cluster)
    * one registry has already been created and can be used to do the next steps
* Run `docker-compose` build to automatically build all images 
    * If you want to provide custom images names you will have to individually build each container manually with `docker build`
* Login into de registry with `docker login <registry_name>`
* Once images have been built, tag them to the repository you created 
    * `docker tag <image_name> <registry_name>/<image_name:tag>`
* Push images to the registry with `docker push <registry_name>/<image_name:tag>`
* Go to the kubernetes tab in Digital Ocean and create a kubernetes cluster, then follow the instructions shown to connect to your just created registry
* Once done, after connecting with kubectl to the cluster, deploy each image
    * If you have created your own registry, you must go to each `*-deployment.yaml` file and change the parameter image for `<registry_name>/<image_name:tag>`
* Then use `kubectl apply -f <files>`
    * IMPORTANT: Make sure you are deploying first all db componets. Either way, the api component will crash as the db would not yet be created
        * Deploy db-deployment.yaml, db-clusterip.yaml and db-persistent-volume-claim before each other container and wait over a minute to make sure it has completed its deployment
* In networking tab in Digital Ocean, at domains, enter the domain you want to use, and then, use the container name as host name (web, api, db) to create a new A rule, and then use the LoadBalancer that kubernetes cluster has associated. Only web and api containers have been set to accept internet outside connections with this method, if you want to open any other container connections, you should add it to the k8s/host.yaml file, and then create the A rule.
    * If you don't want to misuse the IP because of having many LoadBalancers, destroy the kubernetes cluster we used to make the deployment before setting up yours
* While you don't have a domain active set, you can access those two containers with `<container_name>.<load_balancer_ip>.nip.io`
* Load Balancer IP can be checked in the Networking tab

## My secret 
* To generate the values in my secret file, we must use a base64 encoded value.
* If for example, we want one of the values to be procycla, instead of using procycla as input, we will use procycla encoded in base64.
* To encode any value we can use the CLI tool base64 or use a page like this for example: https://www.base64encode.org/