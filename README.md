# Microsoft HPC Pack 2016 Daily Builds

## Deploy an HPC cluster in Azure
**Note:** see Pre-Requisites below before starting deployment
### Deploy an HPC cluster with three head nodes in Azure
Deploy an HPC cluster in Azure with one domain controller, one Database Server with SQL Server 2014 Standard version, three head nodes and a number of compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-three-hns-daily%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### [No AD Domain] Deploy an HPC cluster with three head nodes in Azure
Deploy an HPC cluster in Azure with one Database Server with SQL Server 2014 Standard version, three head nodes and a number of compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-three-hns-noad-daily%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### [No AD Domain] Deploy an HPC cluster with single head node in Azure
Deploy an HPC cluster in Azure with a single node and a number of compute nodes. The head node is with local databases (SQL server 2014 Express version).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-single-hn-noad-daily%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### [Linux CN] Deploy an HPC cluster with three head nodes in Azure
Deploy an HPC cluster in Azure with one Database Server with SQL Server 2014 Standard version, three head nodes and a number of Linux compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-three-hns-lnxcn-daily%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### [Linux CN] Deploy an HPC cluster with single head node in Azure
Deploy an HPC cluster in Azure with a single node and a number of Linux compute nodes. The head node is with local databases (SQL server 2014 Express version).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-single-hn-lnxcn-daily%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Pre-Requisites:

A Personal Information Exchange(PFX) certificate is required to secure the communication between the HPC nodes. Before you deploy the cluster, you shall upload a PFX certificate to Azure Key Valut as a secret, see the [description of vaultCertificates.certificateUrl]( https://msdn.microsoft.com/en-us/library/mt163591.aspx#bk_vaultcert). The following PowerShell script is an example for your reference.

    #Give the following values
    $VaultName = "mytestvault"
    $SecretName = "hpcpfxcert"
    $VaultRG = "myresourcegroup"
    $location = "westus"
    $PfxFile = "c:\Temp\mytest.pfx"
    $Password = "yourpfxkeyprotectionpassword"

    #Encode the PFX file as a base 64 string according to 
    # https://msdn.microsoft.com/en-us/library/mt163591.aspx#bk_vaultcert
    $pfxContentBytes = Get-Content $PfxFile -Encoding Byte
    $pfxContentEncoded = [System.Convert]::ToBase64String($pfxContentBytes)
    # Create and encode the JSON object as a base 64 string.
    $jsonObject = @"
    {
    "data": "$pfxContentEncoded",
    "dataType": "pfx",
    "password": "$Password"
    }
    "@
    $jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
    $jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
    # Convert the JSON to a secure string
    $secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force

    #Create an Azure key vault and upload the certificate as a secret
    New-AzureRmKeyVault -VaultName $VaultName -ResourceGroupName $VaultRG -Location $location -EnabledForDeployment -EnabledForTemplateDeployment
    Set-AzureKeyVaultSecret -VaultName $VaultName -Name $SecretName -SecretValue $secret
