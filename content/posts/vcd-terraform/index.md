---
title: vCloud Director and Terraform
date: 2023-03-19
tags:
- VMWare
showtableofcontents: true
showtaxonomies: true
showsummary: true
summary: "We go through deploying VMs to vCloud Director using Terraform" 


---


## Deploying a VM to VMware vCloud Director using Terraform
If you're using VMware vCloud Director to manage your virtual infrastructure, you might be interested in using Terraform to automate the deployment of VMs. Terraform is an open-source tool that allows you to define infrastructure as code, making it easy to manage and version control your infrastructure changes.

In this blog post, we'll walk through the steps to deploy a VM to VMware vCloud Director using Terraform. We'll cover the following topics:

* Setting up the Terraform environment
* Creating a vCloud Director provider configuration
* Creating a vApp and VM definition
* Deploying the VM to vCloud Director


# Setting up the Terraform environment
Before we can get started with Terraform, we need to set up our environment. We'll assume that you already have Terraform installed on your local machine. If not, you can download it from the official website [here][1].

Next, we need to create a new Terraform project directory. Create a new directory and navigate to it in your terminal:


```powershell 
$ mkdir terraform-vcloud
$ cd terraform-vcloud
```

We also need to install the `terraform-provider-vcd` plugin, which provides support for vCloud Director resources in Terraform. To do this, create a .`terraformrc` file in your home directory and add the following lines:

```powershell 
providers {
  vcd = "/path/to/terraform-provider-vcd"
}
```

Replace `/path/to` with the path to your `terraform-provider-vcd` binary. If you don't have it installed, you can download it from the official repository [here][2].


# Creating a vCloud Director provider configuration
Now that we have our environment set up, we need to create a vCloud Director provider configuration. This configuration tells Terraform how to connect to your vCloud Director instance.

Create a new file called `provider.tf` in your project directory, and add the following code:
hcl

```powershell
provider "vcd" {
  user         = "your-vcloud-username"
  password     = "your-vcloud-password"
  org          = "your-vcloud-organization"
  vdc          = "your-vcloud-vdc"
  url          = "https://your-vcloud-url"
  allow_unverified_ssl = true
}
```

Replace the values in quotes with your own vCloud Director details. The `allow_unverified_ssl` option is set to true to allow connection to vCloud Director instances with self-signed SSL certificates. If your vCloud Director instance has a trusted SSL certificate, you can remove this option.

# Creating a vApp and VM definition
Now that we have our provider configuration set up, we can start defining our vCloud Director resources. In this example, we'll create a new vApp and VM.

Create a new file called `main.tf` in your project directory, and add the following code:
hcl

```powershell
resource "vcd_vapp" "my-vapp" {
  name              = "my-vapp"
  description       = "My vApp description"
  catalog_name      = "my-catalog"
  template_name     = "my-template"
}

resource "vcd_vm" "my-vm" {
  name              = "my-vm"
  memory            = "2048"
  cpus              = "2"
  vapp_name         = "${vcd_vapp.my-vapp.name}"
  network           = "my-network"
  guest_customization = {
    computer_name = "my-vm"
    admin_password = "P@ssw0rd"
  }

  storage_profile = "my-storage-profile"
}

```

Let's break down what's happening here.

First, we're defining a new vApp resource called `my-vapp`. We're setting the name and description of the vApp, as well as specifying the catalog name and template name from which to create the vApp.

Next, we're defining a new VM resource called `my-vm`. We're setting the name of the VM, as well as the amount of memory and number of CPUs. We're also specifying the vApp name and network name to which the VM will be connected.

Finally, we're defining the guest customization and storage profile for the VM. The guest customization allows us to set properties of the guest operating system, such as the computer name and admin password. The storage profile specifies the storage policy to use for the VM's disks.

# Deploying the VM to vCloud Director
Now that we have our vApp and VM definition, we can deploy it to vCloud Director. Run the following commands in your project directory:


```powershell
$ terraform init
$ terraform apply
```

The `terraform init` command initializes the Terraform environment, downloading any necessary plugins and modules. The `terraform apply` command applies the changes defined in your Terraform configuration.

You should see output similar to the following:


```yaml
vcd_vm.my-vm: Creating...
vcd_vm.my-vm: Creation complete after 1m0s [id=my-vm]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Congratulations, you've successfully deployed a VM to vCloud Director using Terraform! You can view the VM in the vCloud Director web interface.

# Conclusion
In this blog post, we walked through the steps to deploy a VM to VMware vCloud Director using Terraform. We covered setting up the Terraform environment, creating a vCloud Director provider configuration, creating a vApp and VM definition, and deploying the VM to vCloud Director.

Terraform provides a powerful way to manage your infrastructure as code, making it easy to version control and automate your infrastructure changes. With the `terraform-provider-vcd` plugin, you can easily manage your VMware vCloud Director resources using Terraform.



[1]: https://www.terraform.io/downloads.html
[2]: https://github.com/vmware/terraform-provider-vcd/releases