# Getting Started with Terraform

[Terraform](https://www.terraform.io/) is an Infrastructure as Code (IaC) tool for creating and managing infrastrucure in a repeatable and consistent manner. Using the HashiCorp Configuration Language (HCL), you define your infrastrucuture resources in configuration files and Terraform will create, update, and destroy the resources as needed to meet the desired state.

In this tutorial, you will initialize a Terraform project and learn some of the basic commands to deploy a Docker container using Terraform.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Terraform CLI installed locally](https://learn.hashicorp.com/tutorials/terraform/install-cli)

## Create Project Directory and Files

Create a new directory for your Terraform project and change into that directory.

```bash
  mkdir terraform-demo
  cd terraform-demo
```

Create the two configuration files you will need for this tutorial. The `providers.tf` file outlines the infrastruture provider that Terraform will use to create and manage resources.  The `main.tf` file outlines the configuration for the resources that Terraform will create using the declared Docker provider.

```bash
  touch providers.tf
  touch main.tf
```

## Configure Terraform Provider and Resources

Open `providers.tf` in your text editor, paste in the configuration below, and save the file. The file tells Terraform to use the [Docker provider](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs) to create and manage Docker resources as well as which Docker host to use for the deployment. See the [Terraform Registry](https://registry.terraform.io/browse/providers) for the complete list of supported Terraform Providers.

  ```hcl
  terraform {
    required_providers {
      docker = {
        source = "kreuzwerker/docker"
      }
    }
  }

  provider "docker" {
      host = "unix:///var/run/docker.sock"
  }
  ```

Open `main.tf` in your text editor, paste in the configuration below, and save the file. The file tells Terraform to create a Docker container using the image `nginx:latest` and to expose the internal container port `80` to port `80` on the Docker host.

```hcl
  resource "docker_image" "nginx" {
    name = "nginx:latest"
  }

  resource "docker_container" "nginx" {
    image = docker_image.nginx.image_id
    name  = "training"
    ports {
      internal = 80
      external = 80
    }
  }
```

## Initialize Terraform

With the configuration files defined, initialize Terraform by executing the `init` command. The `init` command is used to download and install the provider plugins needed to manage the resources defined in the configuration files.

```bash
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of kreuzwerker/docker from the dependency lock file
- Installing kreuzwerker/docker v2.22.0...
- Installed kreuzwerker/docker v2.22.0 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## Create Terraform Plan

Now that the directory is initialized and the provider plugins are installed, use the `plan` command to create an execution plan that outlines the changes Terraform needs to make to meet the desired state. At a high level a `terraform plan` will:

- Look at the current state of deployed resources 
- Compare the current state of the deployed resources to the desired state defined in the configuration files
- List the changes that will be made to meet the desired state

By default the `plan` command will output the plan to the terminal. You can also output the plan to a file by adding the `-out` flag and specifying a file name. This is useful if you want to run the `apply` command at a later time.

```bash
$ terraform plan -out default.tfplan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + gateway                                     = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ip_address                                  = (known after apply)
      + ip_prefix_length                            = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "training"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Saved the plan to: default.tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "default.tfplan"
```

## Apply Terraform Plan

To deploy the resources outlined in the previous `plan` step, run the `apply` command to provision the Docker container. Running the `apply` command without a saved plan file will generate a new plan and prompt you to accept the proposed changes. When you are run `terraform apply` on a saved plan you will not be prompted to accept the changes.

```bash
$ terraform apply default.tfplan

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 0s [id=sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862fnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=1f8e167fcebb094b367888f06030c21954335366cb2afb8f075d0c4d15b6e5b8]
```

To verify that the container was deployed successfully, issue the command `curl -I localhost:80`:

```bash
$ curl -I localhost:80
HTTP/1.1 200 OK
Server: nginx/1.23.1
Date: Wed, 05 Oct 2022 20:00:56 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 19 Jul 2022 14:05:27 GMT
Connection: keep-alive
ETag: "62d6ba27-267"
Accept-Ranges: bytes
```

## Make a Change and Generate a New Plan

As Terraform is all about current state vs desired state, you can update the current configuration and run `plan` to see the proposed changes

Open `main.tf` in your text editor, paste in this updated configuration, and save the file.

```hcl
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "development"
  ports {
    internal = 80
    external = 8080
  }
}
```

With the changes made run `terraform plan` and inspect the output. You should see that Terraform will destroy the existing container and create a new one based on the changes to the configuration.

```bash
terraform plan 
docker_image.nginx: Refreshing state... [id=sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862fnginx:latest]
docker_container.nginx: Refreshing state... [id=bbb0c67ff1690758aa9c99a24cdb30461fcdf0062f71b25c3773ba2d0b0b6a8d]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # docker_container.nginx must be replaced
-/+ resource "docker_container" "nginx" {
      + bridge                                      = (known after apply)
      ~ command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> (known after apply)
      + container_logs                              = (known after apply)
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      ~ entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> (known after apply)
      ~ env                                         = [] -> (known after apply)
      + exit_code                                   = (known after apply)
      ~ gateway                                     = "172.17.0.1" -> (known after apply)
      - group_add                                   = [] -> null
      ~ hostname                                    = "bbb0c67ff169" -> (known after apply)
      ~ id                                          = "bbb0c67ff1690758aa9c99a24cdb30461fcdf0062f71b25c3773ba2d0b0b6a8d" -> (known after apply)
      ~ init                                        = false -> (known after apply)
      ~ ip_address                                  = "172.17.0.2" -> (known after apply)
      ~ ip_prefix_length                            = 16 -> (known after apply)
      ~ ipc_mode                                    = "private" -> (known after apply)
      - links                                       = [] -> null
      ~ log_driver                                  = "json-file" -> (known after apply)
      - log_opts                                    = {} -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      ~ name                                        = "training" -> "development" # forces replacement
      ~ network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> (known after apply)
      - network_mode                                = "default" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      ~ runtime                                     = "runc" -> (known after apply)
      ~ security_opts                               = [] -> (known after apply)
      ~ shm_size                                    = 64 -> (known after apply)
      ~ stop_signal                                 = "SIGQUIT" -> (known after apply)
      ~ stop_timeout                                = 0 -> (known after apply)
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
        # (12 unchanged attributes hidden)

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      ~ ports {
          ~ external = 80 -> 8080 # forces replacement
            # (3 unchanged attributes hidden)
        }
    }
```

## Destroy Terraform Resources

To clean up the resources you can issue the `destroy` command. This will generate a plan for the resources to be deleted and then prompt you to accept the changes and destroy the resources. You can also use `plan -destroy` or `apply -destroy` to initiate a destruction plan for the resources.

```bash
$ terraform destroy               
docker_image.nginx: Refreshing state... [id=sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862fnginx:latest]
docker_container.nginx: Refreshing state... [id=1f8e167fcebb094b367888f06030c21954335366cb2afb8f075d0c4d15b6e5b8]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach                                      = false -> null
      - command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - container_read_refresh_timeout_milliseconds = 15000 -> null
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      - entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env                                         = [] -> null
      - gateway                                     = "172.17.0.1" -> null
      - group_add                                   = [] -> null
      - hostname                                    = "1f8e167fcebb" -> null
      - id                                          = "1f8e167fcebb094b367888f06030c21954335366cb2afb8f075d0c4d15b6e5b8" -> null
      - image                                       = "sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862f" -> null
      - init                                        = false -> null
      - ip_address                                  = "172.17.0.2" -> null
      - ip_prefix_length                            = 16 -> null
      - ipc_mode                                    = "private" -> null
      - links                                       = [] -> null
      - log_driver                                  = "json-file" -> null
      - log_opts                                    = {} -> null
      - logs                                        = false -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      - must_run                                    = true -> null
      - name                                        = "training" -> null
      - network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode                                = "default" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      - read_only                                   = false -> null
      - remove_volumes                              = true -> null
      - restart                                     = "no" -> null
      - rm                                          = false -> null
      - runtime                                     = "runc" -> null
      - security_opts                               = [] -> null
      - shm_size                                    = 64 -> null
      - start                                       = true -> null
      - stdin_open                                  = false -> null
      - stop_signal                                 = "SIGQUIT" -> null
      - stop_timeout                                = 0 -> null
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
      - tty                                         = false -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862fnginx:latest" -> null
      - image_id    = "sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862f" -> null
      - latest      = "sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862f" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:2f770d2fe27bc85f68fd7fe6a63900ef7076bc703022fe81b980377fe3d27b70" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=1f8e167fcebb094b367888f06030c21954335366cb2afb8f075d0c4d15b6e5b8]
docker_container.nginx: Destruction complete after 2s
docker_image.nginx: Destroying... [id=sha256:51086ed63d8cba3a6a3d94ecd103e9638b4cb8533bb896caf2cda04fb79b862fnginx:latest]
docker_image.nginx: Destruction complete after 0s

Apply complete! Resources: 0 added, 0 changed, 2 destroyed.
```

## Next steps

In this tutorial, you have learned how to:

- Create and initialize a Terraform configuration using HCL
- Create and apply a Terraform Plan to deploy a Docker container
- Clean up deployed resources with the `terraform destroy` command

Now that you have a basic understanding of Terraform, extend this example with the use of [Variables](https://www.terraform.io/language/values/variables#input-variables) or explore the [Terraform Registry](https://registry.terraform.io/) to find more Terraform providers and modules. 