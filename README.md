# HttpsWebApi

This repository contains basic scaffold project (Web API) generated using .NET Core CLI. Kestral server is configured to serve with our custom local domain which has self signed certificate.

## What is HTTPS?

HTTPS is an acronym for Hyper Text Transfer Protocol Secure. This is basically a HTTP with added security for data transfer between 2 end points.

This added security is Data Encryption.

Data encrypted using -

- SSL : Secure Socket Layer
- TLS : Transport Layer Security

## Certificates & Authorities

A certificate aims to prove the identity of the Server. A cerificate will be issued by an Authority (CA : Certifying Authority).

For development purpose, we are going to be issuing certificate on our own behalf. This type of certificate is called as Self Signed Certificate.

Clients need to trust certifying authority. For development purpose, we are trusting our own certificate. In production we can use any trusted Certifying Authority (CA) like -

- DigiCert
- GlobalSign
- VeriSign

### What does a Certificate contain?

- The domain name
- Issuing Authority
- What the certificate "Authorises"
- Issue & Expiration date
- Public Key (Private Key is secret)

### Public & Private key

A key is a (long) string used to -

1. Encrypt (scramble) data
2. Decrypt (unscramble) data

Public keys are shared to anyone. Client uses this key to encrypt the data which it sends to server.

Private key : Server uses private key to decrypt the data recieved from its client. Private keys are retained by the owner of the certificate and must be kept secure in server machine.

---

## Self Signed Certificate for "localhost"

Create & Run Web API project -

```
dotnet new webapi -n HttpsWebApi
```

```
dotnet run
```

By default, Kestral Server will serve APIs over https://localhost:5001 (Secure) & http://localhost:5000 (Unsecure). You can see this configured inside launchSettings.json file.

If the localhost is not provided with valid certificate, even though we used HTTPS, we will see unsecure connection in browser.

You can browse all available certificates in your Windows OS by simply opening "Manage user certificates" from Control Panel.

- This will open a window with title "certmgr"
- Open "Certificates" under "Trusted Root Certification Authorities"
- Check for "localhost" (Sometimes, even if there is a entry for localhost, certificate may not be valid)

If you dont have valid certificate, delete existing one and generate new one using the following command.

```
dotnet dev-certs https --trust
```

This above command will generate a Self Signed Certificate (means, issued by localhost and issued to localhost). After this step, try visiting https://localhost:5001. You will see secure connection.

For some reasons, if you cant see secure connection. Try visiting https://localhost:5001 in Incognito Window.

---

## Self Signed Certificate for custom local domain

(here we use, "localhost.com")

### Step 1 : Map your machine IP with a custom domain name

1. Open Notepad (Run as administrator)
2. Open "hosts" file in Notepad from the following path -

   ```
   C:\Windows\System32\drivers\etc
   ```

3. If 192.168.0.112 is your IP, and you want to map it with domain name localhost.com. Write down below line at the end of "hosts" file.
   ```
   192.168.0.112 localhost.com
   ```

### Step 2 : Generate Self Signed Certificate for localhost.com

1. Open Windows Powershell (Run as administrator)

2. Generate certificate & store in a variable called $cert

   ```
    $cert = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dns localhost.com
   ```

   You can check certificate generation using -

   ```
   $cert
   ```

   At this point, you can find this certificate in "certmgr" under "Certificates" of "Intermediate Certification Authorities".

3. Next, we should password protect this certificate ( here, password is pa55w0rd! )
   ```
   $pwd = ConvertTo-SecureString "pa55w0rd!" -Force -AsPlainText
   ```
4. Then, create a variable to hold path to certificate
   ```
   $certpath = "Cert:\localmachine\my\$($cert.Thumbprint)"
   ```
5. Finally, we can export this certificate (as .pfx file) using the command below -
   ```
   Export-PfxCertificate -Cert $certpath -FilePath d:\localhostcom.pfx -Password $pwd
   ```
   Now, we should import this certificate into "Certificates" under "Trusted Root Certification Authorities" in "certmgr".

- Right click on "Certificates" -> All Tasks -> Import...
- Next
- Browse & Select Certificate (localhostcom.pfx)
- Next
- Enter Password ( pa55w0rd! )
- Next
- Finish

### Step 3 : Configure your application to run on [localhost.com](https://localhost.com:5001)

You will need to use a GUID, Install an extenstion in VS Code called "Insert GUID" by Heath Stewart. This will help you in generation and insertion of GUID.

1. Open [HttpsWebApi.csproj](https://github.com/arjunreddy-001/HttpsWebApi/blob/main/HttpsWebApi.csproj) and add "UserSecretsId" inside "PropertyGroup"

   ```
   <PropertyGroup>
       <TargetFramework>netcoreapp3.1</TargetFramework>
       <UserSecretsId>2596a356-1715-4212-8c1f-86150ba87235</UserSecretsId>
   </PropertyGroup>
   ```

   To insert a GUID -

   - Place your cursor in between "UserSecretsId" tag
   - Press Ctrl + Shift + P
   - Type "Insert GUID" and press enter
   - Select GUID generated by "Insert GUID" tool

2. Next, use User Secrets management tool to generate user secret
   ```
   dotnet user-secrets set "CertPassword" "pa55w0rd!"
   ```
   This user secret will be stored in the following path -
   ```
   C:\Users\<userid>\AppData\Roaming\Microsoft\UserSecrets
   ```
3. Now make your application aware of certificate by specifying certificate path inside "[appsettings.Development.json](https://github.com/arjunreddy-001/HttpsWebApi/blob/main/appsettings.Development.json)"
   ```
   "CertPath": "D:\\localhostcom.pfx"
   ```
4. Open Program.cs file & add some configurations. Configure Kestrel server to use your custom certificate (Self Signed Certificate). We do this inside "ConfigureWebHostDefaults". Refer [Program.cs](https://github.com/arjunreddy-001/HttpsWebApi/blob/main/Program.cs) for implementation.

   After configuring you should be able to access -

   https://localhost.com:5001/weatherforecast
