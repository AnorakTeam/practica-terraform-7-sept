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

## Parte 4

> Un git pull + git status más tarde...

```bash
$ terraform plan
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```

> Terraform evitando duplicar instancia aún corriendo (porque tienen mismo nombre pero se cambió tipo de máquina)

```bash
$ terraform apply -var tipo_maquina=e2-small
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
      ~ machine_type               = "e2-micro" -> "e2-small"
        name                       = "web-tf"
        tags                       = [
            "servidor-web",
        ]
        # (24 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.web: Modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
╷
│ Error: Changing the machine_type, min_cpu_platform, service_account, enable_display, shielded_instance_config, scheduling.node_affinities, scheduling.max_run_duration or network_interface.[#d].(network/subnetwork/subnetwork_project) or advanced_machine_features on a started instance requires stopping it. To acknowledge this, please set allow_stopping_for_update = true in your config. You can also stop it by setting desired_status = "TERMINATED", but the instance will not be restarted after the update.
│ 
│   with google_compute_instance.web,
│   on main.tf line 29, in resource "google_compute_instance" "web":
│   29: resource "google_compute_instance" "web" {
│ 
╵
```

> Y ahora tiene autorización para apagar y recrear por su cuenta

```bash
$ terraform apply -var tipo_maquina=e2-small
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
      + allow_stopping_for_update  = true
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
      ~ machine_type               = "e2-micro" -> "e2-small"
        name                       = "web-tf"
        tags                       = [
            "servidor-web",
        ]
        # (24 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_compute_instance.web: Modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m10s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m20s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m30s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m40s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m50s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 01m00s elapsed]
google_compute_instance.web: Modifications complete after 1m4s [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

ip_externa = "34.56.64.178"
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform output ip_externa
"34.56.64.178"
```

IPs obtenidas antes y después:
- "34.58.138.59"
- "34.56.64.178"

## Parte 5

### Tabla de comparación


| Cómo                                  | Tiempo                                                                                                                                                | Qué queda después                                                                                                     |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Interfaz gráfica (Práctica 1, fase 1) | más de 2 minutos haciéndolo rápido                                                                                                                    | Nada. Ni siquiera la lista de clics.                                                                                  |
| `gcloud` (Práctica 1, fase 5)         | 13.959s (probablemente porque ya existía el tag de firewall, terraform lo tuvo que crear y en esta práctica no en este momento de ejecutar el código) | Un comando en el historial, si no se borra.                                                                           |
| Terraform (hoy)                       | real    0m19.071s                                                                                                                                     | Un repositorio que cualquiera puede clonar, leer y volver a ejecutar, con el historial de cómo llegó a ser lo que es. |


### Output de los comandos

```bash
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ time terraform destroy -auto-approve
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_compute_firewall.permitir_http will be destroyed
  - resource "google_compute_firewall" "permitir_http" {
      - creation_timestamp      = "2026-09-12T17:52:14.446-07:00" -> null
      - deletion_policy         = "DELETE" -> null
      - destination_ranges      = [] -> null
      - direction               = "INGRESS" -> null
      - disabled                = false -> null
      - id                      = "projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http" -> null
      - name                    = "permitir-http" -> null
      - network                 = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/global/networks/default" -> null
      - priority                = 1000 -> null
      - project                 = "project-ded4209f-94f1-47b0-a63" -> null
      - self_link               = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http" -> null
      - source_ranges           = [
          - "0.0.0.0/0",
        ] -> null
      - source_service_accounts = [] -> null
      - source_tags             = [] -> null
      - target_service_accounts = [] -> null
      - target_tags             = [
          - "servidor-web",
        ] -> null
        # (1 unchanged attribute hidden)

      - allow {
          - ports    = [
              - "80",
            ] -> null
          - protocol = "tcp" -> null
        }
    }

  # google_compute_instance.web will be destroyed
  - resource "google_compute_instance" "web" {
      - allow_stopping_for_update  = true -> null
      - can_ip_forward             = false -> null
      - cpu_platform               = "Intel Broadwell" -> null
      - creation_timestamp         = "2026-09-12T17:52:15.211-07:00" -> null
      - current_status             = "RUNNING" -> null
      - deletion_policy            = "DELETE" -> null
      - deletion_protection        = false -> null
      - effective_labels           = {
          - "goog-terraform-provisioned" = "true"
        } -> null
      - enable_display             = false -> null
      - id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf" -> null
      - instance_id                = "1283135381214612816" -> null
      - label_fingerprint          = "vezUS-42LLM=" -> null
      - labels                     = {} -> null
      - machine_type               = "e2-small" -> null
      - metadata                   = {} -> null
      - metadata_fingerprint       = "-AhNgzbBrM0=" -> null
      - metadata_startup_script    = <<-EOT
            #!/bin/bash
            apt update && apt install -y nginx
            echo "<h1><identificacion></h1><p>Servida desde Terraform por $(hostname)</p>" > /var/www/html/index.html
        EOT -> null
      - name                       = "web-tf" -> null
      - project                    = "project-ded4209f-94f1-47b0-a63" -> null
      - resource_policies          = [] -> null
      - self_link                  = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf" -> null
      - tags                       = [
          - "servidor-web",
        ] -> null
      - tags_fingerprint           = "5CR-LlH8X8c=" -> null
      - terraform_labels           = {
          - "goog-terraform-provisioned" = "true"
        } -> null
      - zone                       = "us-central1-a" -> null
        # (4 unchanged attributes hidden)

      - boot_disk {
          - auto_delete                     = true -> null
          - device_name                     = "persistent-disk-0" -> null
          - force_attach                    = false -> null
          - guest_os_features               = [
              - "UEFI_COMPATIBLE",
              - "VIRTIO_SCSI_MULTIQUEUE",
              - "GVNIC",
              - "SEV_CAPABLE",
              - "SEV_LIVE_MIGRATABLE_V2",
            ] -> null
          - mode                            = "READ_WRITE" -> null
          - source                          = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/disks/web-tf" -> null
            # (6 unchanged attributes hidden)

          - initialize_params {
              - architecture                = "X86_64" -> null
              - enable_confidential_compute = false -> null
              - image                       = "https://www.googleapis.com/compute/v1/projects/debian-cloud/global/images/debian-12-bookworm-v20260908" -> null
              - labels                      = {} -> null
              - provisioned_iops            = 0 -> null
              - provisioned_throughput      = 0 -> null
              - replica_zones               = [] -> null
              - resource_manager_tags       = {} -> null
              - resource_policies           = [] -> null
              - size                        = 10 -> null
              - type                        = "pd-standard" -> null
                # (2 unchanged attributes hidden)
            }
        }

      - network_interface {
          - internal_ipv6_prefix_length = 0 -> null
          - name                        = "nic0" -> null
          - network                     = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/global/networks/default" -> null
          - network_ip                  = "10.128.0.8" -> null
          - queue_count                 = 0 -> null
          - stack_type                  = "IPV4_ONLY" -> null
          - subnetwork                  = "https://www.googleapis.com/compute/v1/projects/project-ded4209f-94f1-47b0-a63/regions/us-central1/subnetworks/default" -> null
          - subnetwork_project          = "project-ded4209f-94f1-47b0-a63" -> null
          - vlan                        = 0 -> null
            # (6 unchanged attributes hidden)

          - access_config {
              - nat_ip                 = "34.56.64.178" -> null
              - network_tier           = "PREMIUM" -> null
                # (1 unchanged attribute hidden)
            }
        }

      - scheduling {
          - automatic_restart           = true -> null
          - availability_domain         = 0 -> null
          - host_error_timeout_seconds  = 0 -> null
          - min_node_cpus               = 0 -> null
          - on_host_maintenance         = "MIGRATE" -> null
          - preemptible                 = false -> null
          - provisioning_model          = "STANDARD" -> null
            # (2 unchanged attributes hidden)
        }

      - shielded_instance_config {
          - enable_integrity_monitoring = true -> null
          - enable_secure_boot          = false -> null
          - enable_vtpm                 = true -> null
        }
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Changes to Outputs:
  - ip_externa = "34.56.64.178" -> null
google_compute_firewall.permitir_http: Destroying... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Destroying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
google_compute_firewall.permitir_http: Still destroying... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http, 00m10s elapsed]
google_compute_instance.web: Still destroying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m10s elapsed]
google_compute_firewall.permitir_http: Destruction complete after 12s
google_compute_instance.web: Still destroying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m20s elapsed]
google_compute_instance.web: Destruction complete after 22s

Destroy complete! Resources: 2 destroyed.

real    0m24.269s
user    0m2.900s
sys     0m0.551s
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ time terraform apply -auto-approve

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
      + allow_stopping_for_update = true
      + can_ip_forward            = false
      + cpu_platform              = (known after apply)
      + creation_timestamp        = (known after apply)
      + current_status            = (known after apply)
      + deletion_policy           = "DELETE"
      + deletion_protection       = false
      + effective_labels          = {
          + "goog-terraform-provisioned" = "true"
        }
      + id                        = (known after apply)
      + instance_id               = (known after apply)
      + label_fingerprint         = (known after apply)
      + machine_type              = "e2-micro"
      + metadata_fingerprint      = (known after apply)
      + metadata_startup_script   = <<-EOT
            #!/bin/bash
            apt update && apt install -y nginx
            echo "<h1><identificacion></h1><p>Servida desde Terraform por $(hostname)</p>" > /var/www/html/index.html
        EOT
      + min_cpu_platform          = (known after apply)
      + name                      = "web-tf"
      + project                   = "project-ded4209f-94f1-47b0-a63"
      + self_link                 = (known after apply)
      + tags                      = [
          + "servidor-web",
        ]
      + tags_fingerprint          = (known after apply)
      + terraform_labels          = {
          + "goog-terraform-provisioned" = "true"
        }
      + zone                      = "us-central1-a"

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

Changes to Outputs:
  + ip_externa = (known after apply)
google_compute_firewall.permitir_http: Creating...
google_compute_instance.web: Creating...
google_compute_firewall.permitir_http: Still creating... [00m10s elapsed]
google_compute_instance.web: Still creating... [00m10s elapsed]
google_compute_firewall.permitir_http: Creation complete after 12s [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Creation complete after 17s [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

ip_externa = "34.56.64.178"

real    0m19.071s
user    0m2.602s
sys     0m0.466s
```

## Parte 6

Resultados de crear el bucket, modificar main.tf, y hacer las pruebas de que funciona el estado:

```bash
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$  gcloud storage buckets create gs://tfstate-project-ded4209f-94f1-47b0-a63 --lo
cation=us-central1 --uniform-bucket-level-access
Creating gs://tfstate-project-ded4209f-94f1-47b0-a63/...
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ gcloud storage buckets update gs://tfstate-project-ded4209f-94f1-47b0-a63 --ver
sioning
Updating gs://tfstate-project-ded4209f-94f1-47b0-a63/...                                                                                                                  
  Completed 1                                                                                                                                                             
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 2), reused 3 (delta 2), pack-reused 0 (from 0)
Unpacking objects: 100% (3/3), 428 bytes | 428.00 KiB/s, done.
From https://github.com/AnorakTeam/practica-terraform-7-sept
   8f15262..2710c77  main       -> origin/main
Updating 8f15262..2710c77
Fast-forward
 main.tf | 5 +++++
 1 file changed, 5 insertions(+)
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform init -migrate-state
Initializing the backend...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "gcs" backend. No existing state was found in the newly
  configured "gcs" backend. Do you want to copy this state to the new "gcs"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes


Successfully configured the backend "gcs"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v8.2.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform state list
google_compute_firewall.permitir_http
google_compute_instance.web
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ gcloud storage ls gs://tfstate-project-ded4209f-94f1-47b0-a63/practica-2/
gs://tfstate-project-ded4209f-94f1-47b0-a63/practica-2/default.tfstate
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ rm terraform.tfstate terraform.tfstate.backup
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform plan
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
```

## RETO

El reto es buscar cómo conservar esa IP incluso si se destruye la máquina. Claramente, 
hay un servicio para eso. Se declara un compute address, le ponemos un nombre para identificarlo
y Terraform se encarga de todo el linkeo y manejo de esa ip para poder usarla dentro de la config
de nuestra instancia "web" del compute instance.

Plan: 1 to add, 1 to change, 0 to destroy.

Outputs de las dos instancias (se creo una default y la otra tipo small):
- ip_externa = "35.255.29.82"
- ip_externa = "35.255.29.82"

Acá está el output del shell de todo lo hecho para el reto como evidencia:

```bash
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ git pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 2), reused 3 (delta 2), pack-reused 0 (from 0)
Unpacking objects: 100% (3/3), 412 bytes | 412.00 KiB/s, done.
From https://github.com/AnorakTeam/practica-terraform-7-sept
   2710c77..3549159  main       -> origin/main
Updating 2710c77..3549159
Fast-forward
 main.tf | 8 +++++++-
 1 file changed, 7 insertions(+), 1 deletion(-)
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform plan
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_address.ip_estatica will be created
  + resource "google_compute_address" "ip_estatica" {
      + address            = (known after apply)
      + address_id         = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + deletion_policy    = "DELETE"
      + effective_labels   = {
          + "goog-terraform-provisioned" = "true"
        }
      + id                 = (known after apply)
      + label_fingerprint  = (known after apply)
      + name               = "ip-estatica-web"
      + network_tier       = (known after apply)
      + prefix_length      = (known after apply)
      + project            = "project-ded4209f-94f1-47b0-a63"
      + purpose            = (known after apply)
      + region             = (known after apply)
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + terraform_labels   = {
          + "goog-terraform-provisioned" = "true"
        }
      + users              = (known after apply)
    }

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
        name                       = "web-tf"
        tags                       = [
            "servidor-web",
        ]
        # (26 unchanged attributes hidden)

      ~ network_interface {
            name                        = "nic0"
            # (14 unchanged attributes hidden)

          ~ access_config {
              ~ nat_ip                 = "34.56.64.178" -> (known after apply)
                # (2 unchanged attributes hidden)
            }
        }

        # (3 unchanged blocks hidden)
    }

Plan: 1 to add, 1 to change, 0 to destroy.

Changes to Outputs:
  ~ ip_externa = "34.56.64.178" -> (known after apply)

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform apply -auto-approve && terraform output ip_externa
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_address.ip_estatica will be created
  + resource "google_compute_address" "ip_estatica" {
      + address            = (known after apply)
      + address_id         = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + deletion_policy    = "DELETE"
      + effective_labels   = {
          + "goog-terraform-provisioned" = "true"
        }
      + id                 = (known after apply)
      + label_fingerprint  = (known after apply)
      + name               = "ip-estatica-web"
      + network_tier       = (known after apply)
      + prefix_length      = (known after apply)
      + project            = "project-ded4209f-94f1-47b0-a63"
      + purpose            = (known after apply)
      + region             = (known after apply)
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + terraform_labels   = {
          + "goog-terraform-provisioned" = "true"
        }
      + users              = (known after apply)
    }

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
        name                       = "web-tf"
        tags                       = [
            "servidor-web",
        ]
        # (26 unchanged attributes hidden)

      ~ network_interface {
            name                        = "nic0"
            # (14 unchanged attributes hidden)

          ~ access_config {
              ~ nat_ip                 = "34.56.64.178" -> (known after apply)
                # (2 unchanged attributes hidden)
            }
        }

        # (3 unchanged blocks hidden)
    }

Plan: 1 to add, 1 to change, 0 to destroy.

Changes to Outputs:
  ~ ip_externa = "34.56.64.178" -> (known after apply)
google_compute_address.ip_estatica: Creating...
google_compute_address.ip_estatica: Still creating... [00m10s elapsed]
google_compute_address.ip_estatica: Creation complete after 12s [id=projects/project-ded4209f-94f1-47b0-a63/regions/us-central1/addresses/ip-estatica-web]
google_compute_instance.web: Modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m10s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m20s elapsed]
google_compute_instance.web: Modifications complete after 23s [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Apply complete! Resources: 1 added, 1 changed, 0 destroyed.

Outputs:

ip_externa = "35.255.29.82"
"35.255.29.82"
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform apply -var tipo_maquina=e2-small -auto-approve
google_compute_firewall.permitir_http: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/global/firewalls/permitir-http]
google_compute_address.ip_estatica: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/regions/us-central1/addresses/ip-estatica-web]
google_compute_instance.web: Refreshing state... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # google_compute_instance.web will be updated in-place
  ~ resource "google_compute_instance" "web" {
        id                         = "projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf"
      ~ machine_type               = "e2-micro" -> "e2-small"
        name                       = "web-tf"
        tags                       = [
            "servidor-web",
        ]
        # (25 unchanged attributes hidden)

        # (4 unchanged blocks hidden)
    }

Plan: 0 to add, 1 to change, 0 to destroy.
google_compute_instance.web: Modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m10s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m20s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m30s elapsed]
google_compute_instance.web: Still modifying... [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf, 00m40s elapsed]
google_compute_instance.web: Modifications complete after 43s [id=projects/project-ded4209f-94f1-47b0-a63/zones/us-central1-a/instances/web-tf]

Apply complete! Resources: 0 added, 1 changed, 0 destroyed.

Outputs:

ip_externa = "35.255.29.82"
anorakteam@cloudshell:~/sept-07/practica-terraform-7-sept (project-ded4209f-94f1-47b0-a63)$ terraform output ip_externa
"35.255.29.82"
```