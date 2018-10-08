# nexus-conf

Custom nexus documentation. Docker version: 18.06.1-ce. Docker  API version: 1.38.

### Run Nexus as a docker container

	docker-compose.exe -f nexus.yml up -d
	
Check. login as admin/admin123:
	
	http://localhost:8091/
	

#####Create Nexus blob stores : 
* docker-public
* docker-private
* docker-group

#####Create Docker repositories
Following https://help.sonatype.com/repomanager3/private-registry-for-docker/proxy-repository-for-docker create Proxy docker-hub repository with values:
* name: docker-proxy
* URL: http://localhost:8091/repository/docker-proxy/
* blob-store: docker-public
* Enable Docker V1 API 

Following https://help.sonatype.com/repomanager3/private-registry-for-docker/hosted-repository-for-docker-%28private-registry-for-docker%29 create hosted docker repository
* name: docker-private
* URL: http://localhost:8091/repository/docker-private/
* HTTP: 8093
* Force basic auth
* Enable Docker V1 API 
* blob-store: docker-private

Following https://help.sonatype.com/repomanager3/private-registry-for-docker/repository-groups-for-docker create a docker group repository pointing on both private and proxy repos
* name: docker-group
* HTTP: 8094
* Enable docker V1 API
* groups: docker-proxy, docker-private

#####Setting up docker client to use both proxy and private registries

> Running nexus as a container on the same docker engine you want to create the private registry for is **wrong**. To try it out anyway: set the unsecure registries to the inner docker network IP.

1. Get the nexus container inner docker network IP: 

    docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' NEXUS_CONTAINER_ID

2. Add NEXUS_IP:8094 and NEXUS_IP:8093 to unsecure registries following docker-engine documentation.

#####Pull a dockerhub image

1. Login to the public group registry

    docker login -u admin -p admin123 NEXUS_CONTAINER_IP:8094
    
2. Pull (and cache to proxy) a public image

    docker pull NEXUS_CONTAINER_IP:8094/apache/nifi:latest
    
#####Push an image to private registry

1. Login to the private registry

    docker tag IMAGE_ID:VERSION NEXUS_CONTAINER_IP:8093/IMAGE_ID:VERSION
    
2. Tag image
    
    docker login -u admin -p admin123 NEXUS_CONTAINER_IP:8093
    
2. Push image to private registry

    docker push NEXUS_CONTAINER_IP:8093/IMAGE_ID:VERSION
    

    
 

