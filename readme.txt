andrew id- dubale
Part 1: Environment Setup:
I am using wsl2 (ubuntu) on windows docker desktop with kubernestes enabled. ()

Step 1: To enable Kubernetes in Docker Dashboard, go to Settings, enable Kubernetes, apply changes, and confirm
 installation. (see 1enableKubernestes)

Step 2: Open Docker Desktop and go to Settings > Resources > WSL Integration. Enable the integration for the 
WSL distribution you want to use (e.g., Ubuntu). (I am assuming wsl2 is already installed) 
(see 2enableWsl)

Step 3: Download isito
	curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.17.2 TARGET_ARCH=x86_64 sh 

Step 4:	add to path
	cd istio-1.17.2
	export PATH=$PWD/bin:$PATH

Step 5: Install istio
	istioctl install --set profile=demo -y
	(see 3installIstio)

Step 6: Add a namespace label to instruct Istio to automatically inject Envoy sidecar proxies when you deploy your
application later
	kubectl label namespace default istio-injection=enabled

Step 7: Install Argo Rollouts
	kubectl create namespace argo-rollouts
	kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

Step 8: On GKE, you will need grant your account the ability to create new cluster roles
	kubectl create clusterrolebinding YOURNAME-cluster-admin-binding --clusterrole=cluster-admin 
	--user=YOUREMAIL@gmail.com
	kubectl apply -k https://github.com/argoproj/argo-rollouts/manifests/crds\?ref\=stable

Step 9: Install plugin
	curl -LO https://github.com/argoproj/argo-rollouts/releases/latest/download/kubectl-argo-rollouts-linux-amd64
	chmod +x ./kubectl-argo-rollouts-darwin-amd64
	sudo mv ./kubectl-argo-rollouts-darwin-amd64 /usr/local/bin/kubectl-argo-rollouts

Step 10: Confirm argo installtion
	kubectl argo rollouts version
	(see 4argoInstallSuccess)

Step 11: Enable Rollouts UI
	kubectl argo rollouts dashboard
	(see 6argoRollOutSample)

Step 12: Deploying a Rollout (sample from the example)
	kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/rollout.yaml
	kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-rollouts/master/docs/getting-started/basic/service.yam

Step 13: Watch Demo
	kubectl argo rollouts get rollout rollouts-demo --watch
	(see 8rolloutWatch )

Step 14: Update roll out
	kubectl argo rollouts set image rollouts-demo \
  	rollouts-demo=argoproj/rollouts-demo:yellow

Step 15: kubectl argo rollouts get rollout rollouts-demo --watch
	
Part 2: The next step is to deploy a VirtualService, Gateway, and the Rollout. VirtualService and 
Gateway are components of Istio.

I am using a weather app (for the project)

Step 16: Create a folder and download the project  
	git clone https://github.com/infracloudio/ArgoRollout-WeatherExample-Istio.git

Step 17: Configure Environment
	open ubuntu in another windows and start the service
	minikube start --cpus=4 --memory=8g

Step 18: Install
	 istioctl install 
	  
Step 19: Once Istio is installed, check the status of Istio Gateway:
	 kubectl get services -n istio-system

Step 20: Setup Argo Rollouts Controller
	 kubectl create namespace argo-rollouts

Step 21: Ensure that all the components and pods for Argo Rollouts are in the running state. You can check it by 
running the following command:
	 kubectl get all -n argo-rollout
	
	(see 10statusRollout)

Step 22: Configure VirtualService, Gateway and Rollout 
	Change virtualservice file (istio-vsvc.yaml) change 90 to 75 and 10 to 25. (as per the assignment)

Step 23: Deploy (make sure you are in the folder)
	kubectl apply -f istio-vsvc.yaml
	kubectl apply -f istio-gateway.yaml
	kubectl apply -f rollout.yaml

Step 24: Check the status of the rollout: 
	kubectl argo rollouts get rollout rollout-istio

	(see 10statusRollout)

Step 25: Open new terminal and start the services
	minikube tunnel

Step 26: On our current windows
	 minikube service istio-ingressgateway -n istio-system

	(see 11miniKubeStatus)

Step 27: To promote a canary version of the app
	kubectl argo rollouts set image rollout-istio weather-app=docker.io/atulinfracloud/weathersample:v2
	(see 12v2Canaryv1Stable)
	v2 is normalversion and v1 is canary version


references:

https://www.infracloud.io/blogs/progressive-delivery-service-mesh-argo-rollouts-istio/
https://argoproj.github.io/argo-rollouts/getting-started/
https://argo-cd.readthedocs.io/en/stable/
https://istio.io/latest/docs/concepts/traffic-management/#virtual-services