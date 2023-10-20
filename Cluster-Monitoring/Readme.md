## You might need to change some config into the Deployment Yaml file depends on your requests Otherwise Deploy the metric server using the command:

  ```sh
    kubectl apply -f metric-server-deployment.yaml
  ```

## Verification

  ```sh
  alias k=kubectl
  k top pods
  k top nodes
  ```

## For more Details Check the Official Github repository
  
  https://github.com/kubernetes-sigs/metrics-server
