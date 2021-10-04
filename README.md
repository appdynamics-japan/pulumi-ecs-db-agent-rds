# AppDynamics DB Agent test on AWS ECS Fargate with RDS MySQL

This Pulumi project serves a WordPress site in AWS ECS Fargate using an RDS MySQL Backend.

following AWS products (and related Pulumi providers) are used:

- [Amazon VPC](https://aws.amazon.com/vpc): Used to set up a new virtual network in which the system is deployed.
- [Amazon RDS](https://aws.amazon.com/rds): A managed DB service used to provide the MySQL backend for WordPress.
- [Amazon ECS Fargate](https://aws.amazon.com/fargate): A container service used to run the WordPress frontend.

## Getting Started

There are no required configuration parameters for this project since the code will use defaults or generate values as needed - see the beginning of `__main__.py` to see the defaults.
However, you can override these defaults by using `pulumi config` to set the following values (e.g. `pulumi config set service_name my-wp-demo`).

- `service_name` - This is used as a prefix for resources created by the Pulumi program.
- `db_name` - The name of the MySQL DB created in RDS.
- `db_user` - The user created with access to the MySQL DB.
- `db_password` - The password for the DB user. Be sure to use `--secret` if creating this config value (e.g. `pulumi config set db_password --secret`).

### AppDynamics account settings

1. copy fronend_sample.py to frontend.py

```bash
cp fronend_sample.py fronend.py
```

2. In fronend.py, environment variables which start with APPDYNAMICS_ .

```python
'environment': [
    {'name': 'TZ',                                   'value': 'Asia/Tokyo'},
    {'name': 'APPDYNAMICS_CONTROLLER_HOST_NAME',     'value': 'your-account-name.saas.appdynamics.com'},
    {'name': 'APPDYNAMICS_CONTROLLER_PORT',          'value': '443'},
    {'name': 'APPDYNAMICS_CONTROLLER_SSL_ENABLED',   'value': 'true'},
    {'name': 'APPDYNAMICS_AGENT_ACCOUNT_NAME',       'value': 'your-account-name'},
    {'name': 'APPDYNAMICS_AGENT_ACCOUNT_ACCESS_KEY', 'value': 'your-access-key'},
    {'name': 'APPDYNAMICS_DB_AGENT_NAME',            'value': 'db-agent-ecs'},
],
```

## Deploying and running the program

Note: some values in this example will be different from run to run.

1. Create a new stack:

   ```bash
   $ pulumi stack init dev
   ```

1. Set the AWS region:

   ```bash
   $ pulumi config set aws:region ap-northeast-1
   ```

1. Deploy Pulumi stack.

   ```bash
   $ pulumi up
   ```
   After the preview is shown you will be prompted if you want to continue or not. 
   Note: If you set the `db_password` in the configuration as described above, you will not see the `RandomPassword` resource below.

1. Pulumi outputs the following values:

- `DB Endpoint`: This is the RDS DB endpoint. By default, the DB is deployed to disallow public access. This can be overriden in the resource declaration for the backend.
- `DB Password`: This is managed as a secret. To see the value, you can use `pulumi stack output --show-secrets`
- `DB User Name`: The user name for access the DB.
- `ECS Cluster Name`: The name of the ECS cluster created by the stack.
- `Web Service URL`: This is a link to the load balancer fronting the WordPress container. Note: It may take a few minutes for AWS to complete deploying the service and so you may see a 503 error initially.

1. Clean up resources

   ```bash
   pulumi destroy
   ```

## Troubleshooting

### 503 Error for the Web Service

AWS can take a few minutes to complete deploying the WordPress container and connect the load balancer to the service. So you may see a 503 error for a few minutes right after launching the stack. You can see the status of the service by looking at the cluster in AWS.

## Deployment Speed

Since the stack creates an RDS instance, ECS cluster, load balancer, ECS service, as well as other elements, the stack can take about 4-5 minutes to launch and become ready.


