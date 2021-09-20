<br /><br />
<p align="center">
  <img width="230" src="assets/icon.png" />
</p>
<br /><br />

# ansible-ssm
> An ansible role to provision physical and virtual hosts with the AWS SSM agent.

[![Build Status](https://app.travis-ci.com/HQarroum/ansible-ssm.svg?token=yUo2UsFcyh65AXErgq4K&branch=master)](https://app.travis-ci.com/HQarroum/ansible-ssm)

Current version: **1.0.0**

Lead Maintainer: [Halim Qarroum](mailto:hqm.post@gmail.com)

## 📋 Table of content

- [Features](#-features)
- [Installation](#-install)
- [Pre-requisites](#-pre-requisites)
- [Description](#-description)
- [Usage](#-usage)
- [Playbook Variables](#-playbook-variables)
- [Playbook Tags](#-playbook-tags)
- [Compatibility Matrix](#-compatibility-matrix)
- [See also](#-see-also)

## 🚀 Install

In order to use this role, you can download it on your development machine using Ansible Galaxy.

```bash
ansible-galaxy install hqarroum.aws_ssm_agent
```

## 🔖 Features

- Provisions the AWS SSM Agent on your physical and virtual hosts.
- Works on different flavors of Linux and Windows.
- Supports Linux distributions without package management.
- Support for [auto-activation](#enable-auto-activation) of provisioned instances using the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and auto-creation of an IAM role for the instances.

## 🎒 Pre-requisites

- You must have `Ansible 2.8.0` or a superior version installed on your deployment machine. ([Instructions](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
- The `aws-cli` version 2 must be installed on your deployment machine when using auto-activation only. ([Instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html))

## 🔰 Description

This Ansible playbook provides developers and system engineers with the ability to deploy at scale the AWS SSM Agent on remote Linux, Mac and Windows hosts across different architectures. The AWS SSM agent will be registered as a service on the target operating system.

When dealing with managed instances, AWS SSM requires you to [create an activation](https://docs.aws.amazon.com/systems-manager/latest/userguide/activations.html) for one or multiple hosts that you can pass to this playbook using Ansible variables.

## 🛠 Usage

The below sections gives you an overview of the different actions at your disposal by using this playbook. We highly recommend that you be familiar with Ansible before using it, here is a link to the [Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html).

### Create a first deployment

It is possible to specify an existing [AWS SSM activation](https://docs.aws.amazon.com/systems-manager/latest/userguide/activations.html) to this playbook while deploying it on your instances. This allows you to restrict the roles of your deployment machine or deployment team(s) and have them rely on the activation identifier and code that you previously created.

Below is an example of how to pass an activation identifier and activation code to your instances.

```bash
ansible-playbook playbook.yml \
  -i <path-to-your-inventory> \
  --extra-vars "aws_ssm_activation_code=<code> aws_ssm_activation_id=<id> aws_ssm_ec2_region=<region>"
```

If you want to specify a different activation code and identifier to each of your instance, use your inventory to pass both variables for each one of them.

<details><summary>See the example</summary>
<p>

```yaml
[all]
<host-1> aws_ssm_activation_code=<code> aws_ssm_activation_id=<id> aws_ssm_ec2_region=<region>
<host-2> aws_ssm_activation_code=<code> aws_ssm_activation_id=<id> aws_ssm_ec2_region=<region>
```

</p>
</details>

### Enable auto-activation

You can delegate the creation of the activation to the playbook by enabling auto-activation (disabled by default) by setting the `auto_activation` variable to `enabled`. See the [Playbook Variables](#-playbook-variables) section for more information.

> This example will require the AWS CLI to be installed on your deployment machine provisioned with the appropriate IAM roles to create an activation code and an IAM Instance Role for your host(s) automatically. The AWS region will be inferred automatically from the AWS CLI.

```bash
ansible-playbook playbook.yml \
  -i <path-to-your-inventory> \
  --extra-vars "auto_activation=enabled"
```

### Associate tags to your instances

If you do **not** specify an activation identifier and code for your instances, you must set the `auto_activation` variable to `enabled` which will cause this role to automatically create them using the AWS CLI on your deployment machine for each provisioned instance. While doing so, it is possible to bind tags to the created activations such that these tags gets automatically associated with your instance in AWS SSM.

Below is an example of how to bind tags to your activation identifier and activation code.

```bash
ansible-playbook playbook.yml \
  -i <path-to-your-inventory> \
  --extra-vars "auto_activation=enabled instance_tags\"Key=foo,Value=bar\""
```

### Installing the AWS SSM Agent in a specific path

By default, the AWS SSM Agent will be installed using the package manager available on your system (for example, Ubuntu uses `apt` and RedHat uses `yum`). However, embedded operating systems such as Yocto often don't have a package manager available, in which case it can be challenging to install software in a portable and automated way. To address this issue, the `ansible-ssm` role will install AWS SSM by un-archiving its binaries (according to the architecture of your hosts) on the system.

The default root path in which the un-archiving process target will be the root directory (`/`). This can lead to some issues on some systems for which the root file-system is mounted as read-only. In that case, you can choose in which directory to install the AWS SSM Agent by specifying the `aws_ssm_agent_install_directory` such as in the following example.

```bash
ansible-playbook playbook.yml \
  -i <path-to-your-inventory> \
  --extra-vars "auto_activation=enabled aws_ssm_agent_install_directory=/var/ssm"
```

> ⚠️ Only specify this variable if you are deploying on hosts that do not have any supported package manager available.

## 📗 Playbook Variables

As seen in previous sections, this playbook exposes different variables which you can pass to the `ansible-playbook` command when creating a deployment to customize it, or specify them in your Ansible inventory file on a host-by-host basis. Below are descriptions of all the supported variables by this playbook.

Variable                  |                       Description                        |  Example values
------------------------- | -------------------------------------------------------- | ---------------
**aws_ssm_activation_code** | Defines an AWS SSM activation code to provide to the playbook when provisioning your instances. **Required if you don't enable auto-activation.** | `a1b2c3d4e5f6g7h8i1j2` |
**aws_ssm_activation_id** | Defines an AWS SSM activation identifier to provide to the playbook when provisioning your instances. **Required if you don't enable auto-activation.** | `62275ac1-276f-47f0-9309-ef382c710b25` |
**aws_ssm_ec2_region** | Sets the AWS region to use. | `us-east-1` |
**auto_activation** | Enables or disables automatic creation of an activation code for your instances using the AWS CLI. | `enabled` or `disabled` |
**aws_ssm_iam_role** | The Amazon Identity and Access Management (IAM) role name that  you  want to  assign  to  the  managed  instance.  This  IAM role must provide AssumeRole permissions for the  Systems  Manager  service  principal ssm.amazonaws.com  . For more information, see Create an IAM service role for a hybrid environment in the AWS Systems Manager User Guide (if not provided, will be automatically created). **Only valid when auto-activation** is enabled. | `AWSSSMInstanceRole`
**instance_tags** | Defines tags to associate with AWS SSM activations created by this playbook. **Only valid when auto-activation** is enabled. | `Key=foo,Value=bar Key=bar,Value=baz` |
**aws_ssm_agent_install_directory** | Defines the target installation directory for AWS SSM when no package managers are available on your system. | `/var/ssm`

## 🏷 Playbook Tags

In addition to the Ansible variables it makes available, this playbook has the different parts of the deployment process tagged so that you can either cherry-pick the processes that interests you or rather exclude some of the tagged processes part of the deployment. Below is a table of the available tags associated with each deployment processes exposed by this playbook.

Tag name                          | Process                      | Description
--------------------------------- | ---------------------------- | -----------
**install-agent** | AWS SSM Agent Installation | The process associated with this tag will install the AWS SSM agent in the target host given its current architecture and operating system.
**register-service** | AWS SSM Agent Service Registration | The process associated with this tag will register the provisioned AWS SSM Agent as a service on the host operating system, such that the agent will automatically start and run in the background when the host system starts.
**register-agent** | AWS SSM Agent Registration | The process associated with this tag will automatically register the provisioned AWS SSM Agent with the AWS SSM service using either the provided activation code and identifier, or if not provided, by auto-generating a new activation code and identifier locally using the AWS CLI. If you skip this tag, only the AWS SSM Agent will be provisioned on the host, but no activations will be performed.

## 🏁 Compatibility Matrix

Below is a matrix of the operating systems, architectures that have been tested with this Ansible playbook.

OS                                   | Architecture | Compatible         |
------------------------------------ | ------------ | ------------------ |
 **Ubuntu 20.04**                    |    x86_64    | :white_check_mark: |
 **Ubuntu 18.04**                    |    x86_64 and arm64    | :white_check_mark: |
**Ubuntu Server 14.04 LTS**          |    x86_64    | :white_check_mark: |
 **Ubuntu Server 16.04 LTS**         |    x86_64    | :white_check_mark: |
 **MacOS Big Sur**                   |    x86_64    | :white_check_mark: |
 **Amazon Linux**                    |    x86_64    | :white_check_mark: |
 **Amazon Linux 2**                  |    x86_64    | :white_check_mark: |
 **SUSE Linux Enterprise Server 15** |    x86_64    | :white_check_mark: |
 **OpenSUSE Leap 42.1**              |    x86_64    | :white_check_mark: |
 **Red Hat Enterprise Linux**        |    x86_64 and arm64    | :white_check_mark: |
 **Debian Stretch**                  |    x86_64    | :white_check_mark: |
 **CentOS**                          |    x86_64    | :white_check_mark: |
 **Fedora**                          |    x86_64    | :white_check_mark: |
 **Raspbian Stretch**                |    armv7l    | :white_check_mark: |
 **Microsoft Windows Server 2019**   |    x86_64    | :white_check_mark: |
 **Yocto Linux Distribution**        |    x86_32    | :white_check_mark: |

 > Contributions are welcome if you have reported a successful or unsuccessful deployment of the AWS SSM Agent using this Ansible playbook on an operating system that is not listed above.

## 👀 See also

- The [Ansible Official Documentation](https://docs.ansible.com/ansible/latest/index.html) and the [Ansible User Guide](https://docs.ansible.com/ansible/latest/user_guide/index.html).
- The [AWS Systems Manager Official Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/what-is-systems-manager.html) page.
- The [AWS Systems Manager Activations Documentation](https://docs.aws.amazon.com/systems-manager/latest/userguide/activations.html) page.
