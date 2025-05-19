Here are the some basic terraform commands

- to lists terraform's available subcommands

terraform -help
usage= terraform [-version] [-help] <command> [args]

- Add any subcommand to terraform -help to learn more about what it does and available options.

terraform -help "apply" , "init" , "show" , "plan" , "destroy" , "validate"

- To enable autocomplete, run the following command and then restart your shell.

terraform -install-autocomplete

- Initialize the directory.

terraform init

- to plan the terraform work

terraform plan

- to create the infrastructure with terraform

terraform apply

- Inspect the current state using terraform show.

terraform show

terraform state list

- to skip interactive approval of plan before applying.

terraform apply -auto-approve

- to validates the Terraform files.

terraform validate

- to rewrite config files to canonical format.

terraform fmt

terraform console

terraform graph

terraform output
terraform output -json
terraform output tf-example-public-ip

terraform refresh
terraform apply -refresh=false

terraform destroy