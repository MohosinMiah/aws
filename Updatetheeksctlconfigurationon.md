apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-east-1

vpc:
  subnets:
    private:
      us-east-1a:
        id: subnet-03065c49a57867c4e
      us-east-1b:
        id: subnet-0e88852bd52146a88
