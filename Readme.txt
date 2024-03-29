1. Run following script in PowerShell
param (
    [Parameter(Mandatory=$true)][string]$certificatename,
    [Parameter(Mandatory=$true)][SecureString]$certificatepassword
 )
# setup certificate properties including the commonName (DNSName) property for Chrome 58+
$certificate = New-SelfSignedCertificate `
    -Subject localhost `
    -DnsName localhost `
    -KeyAlgorithm RSA `
    -KeyLength 2048 `
    -NotBefore (Get-Date) `
    -NotAfter (Get-Date).AddYears(2) `
    -CertStoreLocation "cert:CurrentUser\My" `
    -FriendlyName "Localhost Certificate for .NET Core" `
    -HashAlgorithm SHA256 `
    -KeyUsage DigitalSignature, KeyEncipherment, DataEncipherment `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1") 
$certificatePath = 'Cert:\CurrentUser\My\' + ($certificate.ThumbPrint)
# create temporary certificate path

$tmpPath = "C:\tmp"
If(!(test-path $tmpPath))
{
New-Item -ItemType Directory -Force -Path $tmpPath
}

# set certificate password here
$pfxPassword = $certificatepassword
$pfxFilePath = $tmpPath + "\" + $certificatename + ".pfx"
$cerFilePath = $tmpPath + "\" + $certificatename + ".cer"

# create pfx certificate
Export-PfxCertificate -Cert $certificatePath -FilePath $pfxFilePath -Password $pfxPassword

Export-Certificate -Cert $certificatePath -FilePath $cerFilePath
# import the pfx certificate
Import-PfxCertificate -FilePath $pfxFilePath Cert:\LocalMachine\My -Password $pfxPassword -Exportable

# trust the certificate by importing the pfx certificate into your trusted root
Import-Certificate -FilePath $cerFilePath -CertStoreLocation Cert:\CurrentUser\Root

# optionally delete the physical certificates (don’t delete the pfx file as you need to copy this to your app directory)
# Remove-Item $pfxFilePath
Remove-Item $cerFilePath
2. To convert PFX file to seperate PEM and KEY files
#CREATE PEM CERT FROM PFX
openssl pkcs12 -in C:/tmp/localhost.pfx -clcerts -nokeys -out C:/tmp/pem/certificate.pem
#CREATE PEM KEY FROM PFX
openssl pkcs12 -in C:/tmp/localhost.pfx -nocerts -out C:/tmp/pem/key.pem -nodes
#CREATE PRIVATE.KEY FROM KEY.PEM
openssl rsa -in C:/tmp/pem/key.pem -out C:/tmp/pem/private.key


3. Copy files under /etc/ folder and use in nginx.conf as following
http {
   server {
      listen 443 ssl;
		
      ssl_certificate /etc/nginx/ssl/certificate.pem;
      ssl_certificate_key /etc/nginx/ssl/private.key;
   }
}
