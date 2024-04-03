# Ansible con AWS
https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/guide_aws.html

En esta guía, utilizando Ansible, vamos a crear una pequeña infraestructura en AWS.


## Instalación de los requisitos previos

Instala la librería: boto3

```
$ pip install boto3

```

## Define las credenciales para AWS

- Mediante un profile

- Mediante variables de entorno

En esta guía, vamos a suponer que ye tenemos definido el profile default.


## Configura Ansible para usar el pluging de AWS y  crea un inventario dinámico

```

# ansible.cfg

[defaults]
enable_plugins=aws_ec2

inventory = ./hosts_aws_ec2.yaml
remote_user = ec2-user
private_key_file = ../vockey.pem

action_warnings = False

```
Inventario dinámico:

```yaml

# hosts_aws_ec2.yaml

plugin: amazon.aws.aws_ec2
profile: default
regions:
  - us-east-1
filters:
  instance-state-name : running
compose:
  ansible_host: public_ip_address

keyed_groups:
  - key: tags.Environment
    separator: ''
```

## Crea la infraestrutura

En esta guía, vamos a crear una máquina virtual, mediante un playbook que requiere tres variables:
nombre (de la máquina), accion (present / absent) y entorno (para ponerle un tag para agruparlas.)

Crear el playbook (provision_ec2.yaml):

```yaml
# provision_ec2.yml

- hosts: localhost
  gather_facts: False

  tasks:

    - name: Provision an EC2 instance with a public IP address
      amazon.aws.ec2_instance:
        name: "{{nombre}}"
        state: "{{accion}}"
        key_name: "vockey"
        instance_type: t2.nano
        security_group: default
        network:
          assign_public_ip: true
        image_id: ami-0c101f26f147fa7fd
        tags:
          Environment: "{{entorno}}"
      register: result

    - name: Show Results
      debug:
        var: result

```

Crea una máquina virtual con:

```
$ ansible-playbook provision_ec2.yaml --extra-vars "nombre=webserver accion=present entorno=web"
```

y crea otra máquina con:

```
$ ansible-playbook provision_ec2.yaml --extra-vars "nombre=database accion=present entorno=db"
```

Lista el inventario con:

```
$ ansible-inventory --list
```