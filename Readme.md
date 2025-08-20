1.Create a cluster as follows 

`k3d cluster create voip-test2 \
  --agents 1 \
  --api-port 6443 \
  --port "5060:30060/udp" \
  --port "16384-16484:16384-16484/udp"`


2.create a docker image for rtpengine using command 

    docker build -t rtpengine-custom:latest .
    k3d image import rtpengine-custom:latest -c voip-test2

3.import the image for freeswitch to cluster 
    docker build -t custom-fsrtp:latest .
    k3d image import custom-fsrtp:latest -c voip-test2 
4.create namespaces as follows 
    kubectl create namespace voip-test2
5.create the developments 
    kubectl apply -f rtpengine-deployment.yaml 
    kubectl apply -f freeswitch-config.yaml 

6.check the entrypoint as follows 
    docker run -it --rm --entrypoint sh rtpengine-custom:latest
7.check if module rtpengine is enabled 
    kubectl exec deployment/freeswitch -n voip-test2 -- cat /etc/freeswitch/autoload_configs/modules.conf.xml

8.check config of rtpengine modules
    kubectl exec deployment/freeswitch -n voip-test2 -- cat /etc/freeswitch/autoload_configs/rtpengine.conf.xml

    