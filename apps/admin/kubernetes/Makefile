# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
BASEDIR = $(shell pwd)

include ../../../Makefile.properties


reset: clean.deployment app
	say "App Reset"

reset.safe: 
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"
	gcloud container builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin
	-kubectl delete deployment admin-deployment
	-kubectl run admin-deployment --image=$(DOCKERREPO)/admin --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO)/api"
	say "app refresh complete"		

app: build deploy

main:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o "$(BASEDIR)/../containers/main" "$(BASEDIR)/../containers/main.go" "$(BASEDIR)/../containers/kubernetes.go"	

build: env main
	gcloud container builds submit "$(BASEDIR)/../containers/." --tag=$(DOCKERREPO)/admin

deploy: env creds deployment service

deploy.minikube: deployment service
	
test: 
	cd 	"$(BASEDIR)/../containers" && go test

deployment:
	kubectl run admin-deployment --image=$(DOCKERREPO)/admin --replicas=1 --port=8080 --labels=app=admin --env="APIIMAGE=$(DOCKERREPO)/api"

service:
	kubectl expose deployment admin-deployment --name=admin --target-port=8080  --type=NodePort --labels="app=admin"

clean: env creds clean.deployment clean.service 

clean.minikube: clean.deployment clean.service 

clean.deployment: 
	-kubectl delete deployment admin-deployment
	
clean.service: 
	-kubectl delete service admin

retry: clean build deploy

config: env 
	echo "No custom config needed for admin"
