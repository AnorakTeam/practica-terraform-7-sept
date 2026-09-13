# practica-terraform-7-sept
> Ups, eran commits en español

## Parte 1

Outputs de la terminal de gcloud shell:

```bash
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.permitir_http will be created
  + resource "google_compute_firewall" "permitir_http" {
      + creation_timestamp = (known after apply)
      + deletion_policy    = "DELETE"
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "permitir-http"
      + network            = "default"
      + priority           = 1000
      + project            = "project-ded4209f-94f1-47b0-a63"
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]
      + target_tags        = [
          + "servidor-web",
        ]

      + allow {
          + ports    = [
              + "80",
            ]
          + protocol = "tcp"
        }
    }

  # google_compute_instance.web will be created
  + resource "google_compute_instance" "web" {
      + can_ip_forward          = false
      + cpu_platform            = (known after apply)
      + creation_timestamp      = (known after apply)
      + current_status          = (known after apply)
      + deletion_policy         = "DELETE"
      + deletion_protection     = false
      + effective_labels        = {
          + "goog-terraform-provisioned" = "true"
        }
      + id                      = (known after apply)
      + instance_id             = (known after apply)
      + label_fingerprint       = (known after apply)
      + machine_type            = "e2-micro"
      + metadata_fingerprint    = (known after apply)
      + metadata_startup_script = <<-EOT
            #!/bin/bash
            apt update && apt install -y nginx
            echo "<h1><identificacion></h1><p>Servida desde Terraform por $(hostname)</p>" > /var/www/html/index.html
        EOT
      + min_cpu_platform        = (known after apply)
      + name                    = "web-tf"
      + project                 = "project-ded4209f-94f1-47b0-a63"
      + self_link               = (known after apply)
      + tags                    = [
          + "servidor-web",
        ]
      + tags_fingerprint        = (known after apply)
      + terraform_labels        = {
          + "goog-terraform-provisioned" = "true"
        }
      + zone                    = "us-central1-a"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + guest_os_features          = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + architecture           = (known after apply)
              + image                  = "debian-cloud/debian-12"
              + labels                 = (known after apply)
              + provisioned_iops       = (known after apply)
              + provisioned_throughput = (known after apply)
              + resource_policies      = (known after apply)
              + size                   = (known after apply)
              + snapshot               = (known after apply)
              + type                   = (known after apply)
            }
        }

      + confidential_instance_config (known after apply)

      + guest_accelerator (known after apply)

      + network_interface {
          + igmp_query                  = (known after apply)
          + internal_ipv6_prefix_length = (known after apply)
          + ipv6_access_type            = (known after apply)
          + ipv6_address                = (known after apply)
          + name                        = (known after apply)
          + network                     = "default"
          + network_attachment          = (known after apply)
          + network_ip                  = (known after apply)
          + parent_nic_name             = (known after apply)
          + stack_type                  = (known after apply)
          + subnetwork                  = (known after apply)
          + subnetwork_project          = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity (known after apply)

      + scheduling (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

Output del apply:
```bash
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Y el estado del repo local:
```bash
$ git status
On branch main
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .terraform.lock.hcl

nothing added to commit but untracked files present (use "git add" to track)
```

## Parte 2

> Hecho el apply con el cambio del outputs.tf
```bash
$ terraform output ip_externa
"34.58.138.59"
```
> Y el test para probar que funciona como variable extraible
```bash
$ curl -m 8 http://$(terraform output -raw ip_externa)
<h1><identificacion></h1><p>Servida desde Terraform por web-tf</p>
```

## Parte 3

```bash
$ terraform apply
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

ip_externa = "34.58.138.59"
```

> Agregar un tag manualmente a la instancia (no se guardó la configuración del taller pasado donde se definía la zona, toncs sólo decirle que la zona recomendada no era y ya se solucionó)
```bash
$ gcloud compute instances add-tags web-tf --tags=prueba-manual
Did you mean zone [us-east1-b] for instance: [web-tf] (Y/n)?  n

No zone specified. Using zone [us-central1-a] for instance: [web-tf].
Updated [https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf].
```

> Ver lo que planea Terraform con respecto a lo anterior
```bash
$ terraform plan
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
        name                       = "web-tf"
      ~ tags                       = [
          - "prueba-manual",
            "servidor-web",
        ]
        # (25 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

> Y terraform le quita la tag a la instancia
```bash
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform apply
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
        name                       = "web-tf"
      ~ tags                       = [
          - "prueba-manual",
            "servidor-web",
        ]
        # (25 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.web: Modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m10s elapsed]
google_compute_instance.web: Modifications complete after 12s [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

ip_externa = "34.58.138.59"
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ gcloud compute instances describe web-tf --format="value(tags.items)"
Did you mean zone [us-east1-b] for instance: [web-tf] (Y/n)?  n

No zone specified. Using zone [us-central1-a] for instance: [web-tf].
servidor-web
```