## Architecture CI/CD AWS ECS/Fargate Multi-account

<img src="https://github.com/EnriqueTejeda/cicd-workflow-jenkins-ecs-fargate/blob/main/assets/diagram-multiaccount-pipeline.png" width=800>

This repository creates a simple workflow for deploy a go app to a ECS cluster with a jenkins pipeline, the tools used in this example is:

* Amazon ECS / Fargate
* AWS Elastic Load Balancing (ALB)
* Amazon Networking Base (VPC, Subnets, NAT Gateway, Security Groups, etc)
* Jenkins

## Infrastructure As A Code for ECS / Fargate Cluster

### Requirements 
#### IAM
You must have a valid credentials for create all the service which terraform template needs, for the configuration of the credentials for interact with the AWS API please follow the official [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-creds)
#### Tools
* terraform 
* aws-cli
### Deploying the infrastructure
The terraform template deploy the next components in your AWS console:

1. VPC named **dev-test-ecs** with the next CIDR 172.16.0.0/16 in two availibility zones.
2. Public/Private subnets (NAT & Internet Gateway, EIPs, Security Groups)
3. Application Load Balancer  
4. AWS ECS Cluster / Fargate
5. Task Definitions, Services
6. Cloudwatch Log Group 
7. Autoscaling policies 
9. All the resources is tagged and you can edit this in the terraform values file.

- Commands for deploy the infrastructure
  ```
  cd terraform/ecs-fargate
  terraform init
  terraform apply 
  ```
## Jenkins (CI Engine)
This example used Jenkins on a docker container behind a local machine or an server, you only needs to have the Docker runtime and the docker-compose binary installed in the machine for run the container, in few words this container is a privigiled container and mount the host docker api to the container itself, so with this technique jenkins has the power for run containers in the pipeline.

- Commands for initialize Jenkins
  ```
  cd jenkins/
  docker-compose up 
  ```
## Hello App Golang 
This app is a clone of the example app inside a [Google documentation](https://cloud.google.com/appengine/docs/flexible/go/quickstart) with a small change for print two environment variables named **LITTLE_SECRET** & **LITTLE_SECRET_CAT** with the objetive for the use of the AWS Secret Manager & AWS KMS.

Source code:

``` go
package main

import (
        "fmt"
        "log"
        "net/http"
        "os"
)

func main() {
        // register hello function to handle all requests
        mux := http.NewServeMux()
        mux.HandleFunc("/", hello)

        // use PORT environment variable, or default to 8080
        port := os.Getenv("PORT")
        if port == "" {
                port = "8080"
        }

        // start the web server on port and accept requests
        log.Printf("Server listening on port %s", port)
        log.Fatal(http.ListenAndServe(":"+port, mux))
}

// hello responds to the request with a plain-text "Hello, world" message.
func hello(w http.ResponseWriter, r *http.Request) {
        log.Printf("Serving request: %s", r.URL.Path)
        host, _ := os.Hostname()
        littleSecret := os.Getenv("LITTLE_SECRET")
        littleSecretCat := os.Getenv("LITTLE_SECRET_CAT")
        fmt.Fprintf(w, "Hello, world!\n")
        fmt.Fprintf(w, "Version: 1.0.0\n")
        fmt.Fprintf(w, "Hostname: %s\n", host)
	fmt.Fprintf(w, "My little secret: %s\n", littleSecret)
	fmt.Fprintf(w, "The little secret of my cat: %s\n", littleSecretCat)
}
```

Any comment please feel free for write me a email to [contact@enriquetejeda.com](mailto://contact@enriquetejeda.com) or open a issue in this repository.
