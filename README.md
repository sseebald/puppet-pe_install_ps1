#### Table of Contents

1. [Overview](#overview)
1. [Setup - Important!](#setup)
1. [Usage](#usage)
  * [Windows Agent Install Process](#windows-agent-install-process)
  * [Customizing the install.ps1 script](#customizing-the-installps1-script)
1. [Reference](#reference)
1. [Limitations](#limitations)
1. [Development](#development)

## Overview

This module will create a PowerShell script, `install.ps1`, that Windows nodes can remotely execute to emulate the [frictionless puppet-agent installer](https://docs.puppetlabs.com/pe/latest/install_agents.html#about-the-platform-specific-install-script) that was made for Linux nodes.

## Setup

### Step 1: Add the Windows pe_repo platform to your Puppet Masters

This module only creates an `install.ps1` script. Use the `pe_repo` module that comes with Puppet Enterprise to stage the MSI puppet-agent installer before trying to use the `install.ps1` script.

In the Enterprise Console, add the `pe_repo::platform::windows_x86_64` class to the `PE Master` node group.

### Step 2: Download this module and classify your Puppet Masters with `ps_install_ps1`

Either download this module with `puppet module install natemccurdy-ps_install_ps1` or add it to your Puppetfile and deploy r10k/code_manager.

Then, add the `ps_install_ps1` class to your Puppet Masters and trigger a puppet agent run:

```puppet
include ::pe_install_ps1
```

## Usage

### Windows Agent Install Process

To execute the PowerShell-based installer on a Windows node, use one of the following methods.

#### From an administrative Command Prompt window

```cmd
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile('https://puppet.company.net:8140/packages/current/install.ps1', \"$env:temp\install-agent.ps1\"); & \"$env:temp\install-agent.ps1\""
```

#### From an administrative PowerShell window

```powershell
[Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}; $webClient = New-Object System.Net.WebClient; $webClient.DownloadFile("https://puppet.company.net:8140/packages/current/install.ps1", "$env:temp\install-agent.ps1"); & "$env:temp\install-agent.ps1"
```

**Note:** You must have your execution policy set to unrestricted (or at least in bypass) for this to work (`Set-ExecutionPolicy Unrestricted`).

#### Adjusting the Puppet agent's settings during installation

The values for `server` and `certname`, in the agent's puppet.conf can be tuned during installation by passing the `server` and `certname` parameters to the `install.ps1` script.

```powershell
$webClient = New-Object System.Net.WebClient; $webClient.DownloadFile("https://<fqdn_of_puppet_master>:8140/packages/current/install.ps1", "$env:temp\install-agent.ps1"); & "$env:temp\install-agent.ps1" -certname foo.custom.net -server puppet.custom.net
```

### Customizing the install.ps1 script

If using load-balanced compile masters, change the `server_setting` parameter to that of your load-balancer or VIP's name.

```puppet
class { 'pe_install_ps1':
  server_setting => '<FQDN_OF_YOUR_LOAD_BALANCER>',
}
```

Here's an example of changing other parameters:

```puppet
class { 'pe_install_ps1':
  msi_host       => 'puppet.company.net',
  server_setting => 'puppet.company.net',
}
```

## Reference

### Class: `pe_install_ps1`

#### `server_setting`
The value that will go in the `server` setting in the agent's puppet.conf.

Default value: `$::settings::server`

#### `msi_host`
The FQDN of the puppet server that is hosting the puppet-agent MSI installer.

Default value: `$::settings::server`

#### `public_dir`
The path to the public package share on the Puppet Master.

Default value: `/opt/puppetlabs/server/data/packages/public`

## Limitations

So far, this only works for **Puppet Enterprise 2015.2.1** or higher, and only for **x86_64** puppet-agent packages.

The `pe_repo` module only supports staging the Windows Puppet Agent MSI since Puppet Enterprise 2015.2.1, so at least that version is required to get any use out of this module.

## Development

Feel free to submit issues and PR's if there's something wrong or missing from this module.

