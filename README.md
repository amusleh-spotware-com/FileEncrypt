# FileEncrypt
A simple command line tool for encryption and decrypting files with AES

I created this tool for using with Github actions when publishing signed Nuget packages.

Here as an example of Github action that you can use:

```yaml
name: Publish Nuget Package When Pre-Released

on:
  release:
    types: [prereleased]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup NuGet
      uses: NuGet/setup-nuget@v1.0.5
      with:
        nuget-api-key: ${{secrets.NUGET_API_KEY}}
        nuget-version: 'latest'

    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.0.x'

    - name: Restore dependencies
      run: dotnet restore ./src/OpenAPI.Net/OpenAPI.Net.csproj

    - name: Build
      run: dotnet build ./src/OpenAPI.Net/OpenAPI.Net.csproj --configuration Release --no-restore

    - name: Decrypt Certificate
      run: .\FileEncrypt decrypt .\certificate.pfx.encrypted ${{secrets.CERTIFICATE_DECRYPTION_KEY}} ${{secrets.CERTIFICATE_DECRYPTION_IV}}

    - name: Sign Package
      run: nuget sign **\*.nupkg -CertificatePath certificate.pfx -Timestamper http://timestamp.digicert.com/ -CertificatePassword ${{secrets.CERTIFICATE_PASSWORD}} -NonInteractive

    - name: Publish Package
      run: nuget push **\*.nupkg -Source 'https://api.nuget.org/v3/index.json'

    - name: Publish Symbols
      run: nuget push **\*.snupkg -Source 'https://api.nuget.org/v3/index.json'
```

You must publish the FileEncrypt app as self contained and put it on your Github repository root.

Use FileEncrypt to encrypt your PFX certificate file on your local system, then put the certificate encrypted file on your repository root.

You have to add the decryption key and IV on your Github repository secrets as CERTIFICATE_DECRYPTION_KEY and CERTIFICATE_DECRYPTION_IV, FileEncrypt will create a .key file that contains both after encryption. 
