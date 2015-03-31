# Senza

Senza is a command line tool for generating and executing AWS Cloud Formation templates in a sane way. It supports
Cloud Formation templates as YAML input and adds own 'components' on top. Components are predefined Cloud Formation
snippets that are easy to configure and generate all the boilerplate JSON that is required by Cloud Formation.

## Installation


## Usage

    $ ./senza/cli.py ./my-definition.yaml create eu-west-1 1.0

## Senza Definition

```yaml
# basic information for generating and executing this definition
SenzaInfo:
  StackName: kio
  OperatorEMail: kio-ops@example.com
  Parameters:
    - ImageVersion:
        Description: "Docker image version of Kio."

# a list of senza components to apply to the definition
SenzaComponents:

  # this basic configuration is required for the other components
  - Configuration:
      Type: Senza::Configuration
      ServerSubnets:
        eu-west-1:
          - subnet-123456
          - subnet-123456
          - subnet-123456
        eu-central-1:
          - subnet-123456
          - subnet-123456
      LoadBalancerSubnets:
        eu-west-1:
          - subnet-123456
          - subnet-123456
          - subnet-123456
        eu-central-1:
          - subnet-123456
          - subnet-123456
      AppImage:
        eu-west-1: ami-123456
        eu-central-1: ami-123456

  # will create a launch configuration and auto scaling group with scaling triggers
  - AppServer:
      Type: Senza::TaupageAutoScalingGroup
      InstanceType: t2.micro
      Image: AppImage
      SecurityGroups:
        - sg-123456
      ElasticLoadBalancer: AppLoadBalancer
      TaupageConfig:
        runtime: Docker
        source: stups/kio:{{args.imageversion}}
        ports:
          8080: 8080
        notify_cfn:
          stack: "{{SenzaInfo.StackName}}-{{args.version}}"
          resource: "AppServer"
        environment:
          HTTP_CORS_ORIGIN: "*.example.com"
          PGSSLMODE: verify-full
          DB_SUBNAME: "//kio.example.eu-west-1.rds.amazonaws.com:5432/kio?ssl=true"
          DB_USER: kio
          DB_PASSWORD: aws:kms:abcdef1234567890abcdef=
      AutoScaling:
        Minimum: 2
        Maximum: 10
        MetricType: CPU
        ScaleUpThreshold: 70
        ScaleDownThreshold: 40

  # creates an ELB entry and Route53 domains to this ELB
  - AppLoadBalancer:
      Type: Senza::ElasticLoadBalancer
      HTTPPort: 8080
      SSLCertificateId: arn:aws:iam::1234567890:server-certificate/kio-example-com
      HealthCheckPath: /ui/
      SecurityGroups:
        - sg-123456
      Domains:
        - Domain: kio.example.com
          Type: weighted
        - Domain: kio-{{args.version}}.example.com
          Type: standalone


# just plain Cloud Formation definitions are fully supported:

Outputs:
  URL:
    Description: "The ELB URL of the new Kio deployment."
    Value:
      "Fn::Join":
        - ""
        -
          - "http://"
          - "Fn::GetAtt":
            - AppLoadBalancer
            - DNSName

```

## Components

* Senza::Configuration
* Senza::TaupageAutoScalingGroup
* Senza::ElasticLoadBalancer