# Deploy Microsoft HPC Pack 2016 preview cluster in Azure

### **Note:** see [Pre-Requisites](#prerequisites) section on this page before starting your deployment.

You can now deploy a Microsoft HPC Pack 2016 cluster in Azure. Choose one from the following templates and click "Deploy to Azure" button to deploy.

### Template 1: High-availability cluster for Windows workloads with Active Directory Domain
This template deploys an HPC Pack cluster with high availability for Windows HPC workloads in Active Directory Domain forest. The cluster includes one domain controller, **three** head nodes, one Database Server with SQL Server 2016 Standard version, and a configurable number of **Windows** compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-templates%2Fthree-hns-wincn-ad.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Template 2: High-availability cluster for Windows workloads
This template deploys an HPC Pack cluster with high availability for Windows HPC workloads. The cluster includes **three** head nodes, one Database Server with SQL Server 2016 Standard version, and a configurable number of **Windows** compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-templates%2Fthree-hns-wincn-noad.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Template 3: High-availability cluster for Linux workloads
This template deploys an HPC Pack cluster with high availability for Windows HPC workloads. The cluster includes **three** head nodes, one Database Server with SQL Server 2016 Standard version, and a configurable number of **Linux** compute nodes.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-templates%2Fthree-hns-lnxcn.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Template 4: Single head node cluster for Windows workloads

This template deploys an HPC Pack cluster with one **single** head node and a configurable number of **Windows** compute nodes. The head node is with local databases (SQL server 2016 Express version).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-templates%2Fsingle-hn-wincn-noad.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

### Template 5: Single head node cluster for Linux workloads

This template deploys an HPC Pack cluster with one **single** head node and a configurable number of **Linux** compute nodes. The head node is with local databases (SQL server 2016 Express version).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fsunbinzhu%2FHPCPack2016%2Fmaster%2Fnewcluster-templates%2Fsingle-hn-lnxcn.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## <a name="prerequisites"></a>Pre-Requisites:

Microsoft HPC Pack 2016 cluster requires a Personal Information Exchange (PFX) certificate to secure the communication between the HPC nodes. The certificate must meet the following requirements: 3.Have a private key capable of **key exchange**; 2.Key usage includes **Digital Signature** and **Key Encipherment**; 3.Enhanced key usage includes **Client Authentication** and **Server Authentication**.

Before deploying the HPC cluster, you shall upload the certificate to an Azure Key Vault as a secret, and remember the following information which will be used in deployment: key vault name, resource group name, secret Id, and certificate thumbprint. More details about uploading certificate to Azure Key Vault please see [description of vaultCertificates.certificateUrl]( https://msdn.microsoft.com/en-us/library/mt163591.aspx#bk_vaultcert), or you can refer to the PowerShell script as below.

    #Give the following values
    $VaultName = "mytestvault"
    $SecretName = "hpcpfxcert"
    $VaultRG = "myresourcegroup"
    $location = "westus"
    $PfxFile = "c:\Temp\mytest.pfx"
    $Password = "yourpfxkeyprotectionpassword"
    #Validate the pfx file
    try {
        $pfxCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList $PfxFile, $Password
    }
    catch [System.Management.Automation.MethodInvocationException]
    {
        throw $_.Exception.InnerException
    }
    $thumbprint = $pfxCert.Thumbprint
    $pfxCert.Dispose()
    # Create and encode the JSON object
    $pfxContentBytes = Get-Content $PfxFile -Encoding Byte
    $pfxContentEncoded = [System.Convert]::ToBase64String($pfxContentBytes)
    $jsonObject = @"
    {
    "data": "$pfxContentEncoded",
    "dataType": "pfx",
    "password": "$Password"
    }
    "@
    $jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
    $jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
    #Create an Azure key vault and upload the certificate as a secret
    $secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
    $rg = Get-AzureRmResourceGroup -Name $VaultRG -Location $location -ErrorAction SilentlyContinue
    if($null -eq $rg)
    {
        $rg = New-AzureRmResourceGroup -Name $VaultRG -Location $location
    }
    $hpcKeyVault = New-AzureRmKeyVault -VaultName $VaultName -ResourceGroupName $VaultRG -Location $location -EnabledForDeployment -EnabledForTemplateDeployment
    $hpcSecret = Set-AzureKeyVaultSecret -VaultName $VaultName -Name $SecretName -SecretValue $secret
    "The following Information will be used in the deployment template"
    "Vault Name             :   $VaultName"
    "Vault Resource Group   :   $VaultRG"
    "Certificate URL        :   $($hpcSecret.Id)"
    "Certificate Thumbprint :   $thumbprint"
