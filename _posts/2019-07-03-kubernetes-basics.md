---
layout: post
title: Kubernetes basics notater (Norwegian)
enabledisqus: true
---

Denne basic guiden følger tutorial på [kubernetes.io](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

{% highlight shell %}
{% raw %}
# Prerequisite
# Installer og start minikube - setter opp lokalt Kubernetes cluster

#Module 2 - starte cluster 
################################
# Deploy en docker image med kubernetes på clusteret. Imaget er laget basert på tidligere docker tutorial
kubectl run friendlyhello --image=localhost:5000/friendlyhello:latest --port=80
# Sjekk deployment kjører på en node i en docker container
kubectl get deployments  
# Per default er applikasjonen tilgjengelig til andre pods og noder cluster men ikke utenfor clusternettverket. 

# kubectl bruker et API for å interagere med clusteret
# Kjøre proxy for å koble til API  (i ny terminalvindu)
kubectl proxy

# Må finne pod navnet for å kalle dens endepunkt via proxy:
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
# kalle podendepunktet via proxy
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/

#Module 3 - kubernetes pods og noder
################################
#POD = en eller flere containere hvor ressurser deles slik som volum, nettverk (unik cluster IP)
# Kan typisk bestå av containere for både databasen og applikasjon.
 
#NODE = kjører en en eller flere PODs. 
# Hver node kjører 
#      en Kubelet - en prosess som håndterer kommunikasjon mellom Kubernetes Master og noden
#      en container runtime f.eks. Docker

# finn kjørende PODs
kubectl get pods
# finn kjørende containere i POD. Describe kan brukes med nodes og deployments også
kubectl describe pods
# For å interagere med pod'en må vi gjennom API proxy beskrevet i module 3
# Når proxy kjører, finn nodenavnet og kjør følgende for logging - dvs det som skrives til STDOUT:
kubectl logs $POD_NAME
# Merk at siden det bare er en container i pod'en så trenger vi ikke spesifisere containernavnet

# kjøre kommandoer på pod'en - env
kubectl exec $POD_NAME env  
# Igjen så vil denne kommandoen kjøre i den ene containeren i pod'en (dvs ikke på selve pod'en)
# kjøre bash
kubectl exec -ti $POD_NAME bash

#Module 4 eksponere appen ut
################################
#Service=et sett med PODs, på tvers av noder, og en policy for å aksessere dem
# service muliggjør løs kobling mellom Pods
# service eksponerer pods ut av cluster på forskjellige måter
# service er en abstraksjon som muliggjør at pods kan dø og gjennoppstå (replicate) uten å påvirke applikasjonen
# en service defineres av en configfil vanligvis. 

# finn kjørende pods
kubectl get pods
# finn nåværende services. En service er laget default når minikube starter
kubectl get services
# lag ny service og eksponer den. NodePort er en av mange type service støttet av Kubernetes
# minikube støtter kun den
kubectl expose deployment/friendlyhello --type="NodePort" --port 80
# se nye service. friendlyhello fikk unik cluster ip + intern port + ekstern port
kubectl get services
# describe for se mer info
kubectl describe services/friendlyhello
# finne porten til noden: 
export NODE_PORT=$(kubectl get services/friendlyhello -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
# curl mot eksponert endepunkt - minikube ip er ip'en til clusteret som minikube satte opp:
curl $(minikube ip):$NODE_PORT
# slette service - men fortsatt la containeren kjøre i pod'en: 
kubectl delete service friendlyhello
{% endraw %}
{% endhighlight %}
