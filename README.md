# gha-deploy-aws-sam

This GitHub Action deploys AWS CloudFormation stacks using the [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html). It supports setting custom parameters, tags, and other deployment configurations.

The action sets some default values based on [ServerlessOpsIO/aws-gha-integration](https://github.com/ServerlessOpsIO/aws-gha-integration) and [ServerlessOpsIO/gha-store-artifacts](https://github.com/ServerlessOpsIO/gha-store-artifacts).

_*NOTE: This workflow is opinionated and meets the needs of its author. It is provided publicly as a reference for others to use and modify as needed.*_

The `deploy-aws-cloudformation` action performs the following tasks:
1. Installs the AWS SAM CLI.
2. Sets the SAM S3 bucket prefix.
3. Sets the CloudFormation stack name.
4. Processes AWS tags and CloudFormation parameters.
5. Deploys using SAM CLI.

## Usage
See below for inputs, outputs, and examples.

### Inputs

- `aws_account_id` (required): Account ID of the account to deploy to.
- `aws_region` (optional): Region to deploy to. If not set, will use AWS_REGION set by [gha-assume-aws-credentials](https://github.com/ServerlessOpsIO/gha-assume-aws-credentials)
- `cfn_capabilities` (optional): Comma-separated list of capabilities to enable. (values: CAPABILITY_IAM, CAPABILITY_NAMED_IAM, CAPABILITY_AUTO_EXPAND, CAPABILITY_RESOURCE_POLICY)
- `cfn_exec_role_name` (optional): Name of the optional CloudFormation IAM execution role.
- `cfn_parameters_file` (optional): Name of the CloudFormation parameters file. Default is `cfn-parameters.json`.
- `cfn_tags_file` (optional): Name of the CloudFormation tags file. Default is `cfn-tags.json`.
- `stack_name` (optional): Name of the stack that will be deployed.
- `template_file` (optional): Name of the template file to deploy. Default is `template.yaml`.
- `sam_s3_bucket` (optional): S3 bucket for SAM deployment.
- `sam_s3_prefix` (optional): S3 prefix for SAM deployment.
- `env_json` (optional): JSON string of environment context object. For use resolving values in CFN parameters file.
- `vars_json` (optional): JSON string of variables context object. For use resolving values in CFN parameters file.
- `secret_json` (optional): JSON string of secret context object. For use resolving values in CFN parameters file.

### Outputs

This action does not produce any outputs.

### Examples

```yaml
name: CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Setup job workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1
        with:
          checkout_artifact: true

      - name: Assume AWS Credentials
        uses: ServerlessOpsIO/gha-assume-aws-credentials@v1
        with:
          build_aws_account_id: ${{ secrets.BUILD_AWS_ACCOUNT_ID }}
          aws_account_region: 'us-east-1'

      - name: Store artifacts
        uses: ServerlessOpsIO/gha-store-artifacts@v1
        with:
          use_aws_sam: true

  deploy:
    runs-on: ubuntu-latest
    needs:
      - build
    steps:
      - name: Setup job workspace
        uses: ServerlessOpsIO/gha-setup-workspace@v1
        with:
          checkout_artifact: true

      - name: Assume AWS Credentials
          uses: ServerlessOpsIO/gha-assume-aws-credentials@v1
          with:
          build_aws_account_id: ${{ secrets.BUILD_AWS_ACCOUNT_ID }}
          deploy_aws_account_id: ${{ secrets.DEPLOY_AWS_ACCOUNT_ID }}
          aws_account_region: 'us-east-1'

      - name: Deploy via AWS CloudFormation
        uses: ServerlessOpsIO/gha-deploy-aws-sam@v1
        with:
          aws_account_id: ${{ secrets.DEPLOY_AWS_ACCOUNT_ID }}
          sam_s3_bucket: ${{ secrets.AWS_CICD_SAM_BUCKET}}
```

### Configuration

Additional configuration notes.

#### S3 Bucket

The default value for `sam_s3_bucket` comes from [ServerlessOpsIO/aws-gha-integration](https://github.com/ServerlessOpsIO/aws-gha-integration). Both the `sam_s3_bucket` and `sam_s3_prefix` should not need to be configured but are avavailbel for unique circumstances.

#### CloudFormation Tags

By default the the action will look for a `cfn-tags.json` file in the root of the repository. The file used can be changed using the `cfn_tags_file` parameter. This file should be a JSON object with key-value pairs for tags. The action will automatically convert this file into a format that AWS SAM expects. The action will also set some additional default tags based on information gathered from the job. eg. GitHub repository, branch, and commit. These tags will be applied to the stack at deploy time and CloudFormation will apply them to resources that support tagging.

Example cfn-tags.json:
```json
{
  "MyTag": "MyValue",
  "MyOtherTag": "MyOtherValue"
}
```

#### CloudFormation Parameters

By default the the action will look for a `cfn-parameters.json` file in the root of the repository. The file used can be changed using the `cfn_parameters_file` parameter.  This file should be a JSON object with key-value pairs for parameters. The action will automatically convert this file into a format that AWS SAM expects.

Example cfn-parameters.json:
```json
{
  "MyParameter": "MyValue",
  "MyOtherParameter": "MyOtherValue"
}
```

##### Environment and Secrets Context values

You may wish to use environment or secrets context values as parameter values. This is a great way to handle different parameter values for different application environments. To do so, you must pass a JSON string of those context objects to the `env_json` or `secret_json` parameters of this action. (This is due to limitations in GitHub Actions for passing context objects to composite actions.)

Example GitHub Action workflow step:
```yaml
- name: Deploy via AWS SAM
  uses: ServerlessOpsIO/gha-deploy-aws-sam@v1
  with:
    aws_account_id: ${{ secrets.AWS_STACKSETS_ACCOUNT_ID }}
    env_json: ${{ toJson(env) }}
    secrets_json: ${{ toJson(secrets) }}
```

Example `cfn-parameters.json` file:
```json
{
  "MyParameterFromEnvironment": $env.MyValue,
  "MyParameterFromSecrets": $secrets.MySecretValue
}
```

This will unfortunately break any JSON linters you may have in your repository. You can ignore the error or disable the linter for the file.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request for any changes.

## Contact

For any questions or support, please open an issue in this repository.