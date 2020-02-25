# escape=`
###############
# Build       #
###############
FROM mcr.microsoft.com/windows/servercore:ltsc2019
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Retrieve .NET Core SDK
RUN Invoke-WebRequest -OutFile dotnet-install.ps1 'https://dot.net/v1/dotnet-install.ps1'; `
    $installDir = $env:ProgramFiles + '\dotnet\sdk\2.2'; `
    & .\dotnet-install.ps1 -Channel 2.2 -InstallDir $installDir; `
    Remove-Item -Force dotnet-install.ps1;

# Configure web servers to bind to port 80 when present
ENV ASPNETCORE_URLS=http://+:80 `
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip

RUN setx /M PATH $env:PATH+';C:\Program Files\dotnet\sdk\2.2';

# Trigger first run experience by running arbitrary cmd to populate local package cache
RUN dotnet help

# Install fonts (required on 2019 images)
ADD docker/build/selenium/windows-ltsc2019/FontsToAdd.tar /Fonts/
RUN C:\Fonts\Add-Font.ps1 C:\Fonts\Fonts

# Install Chrome dependency, Chocolatey, Chrome browser and selenium web driver
RUN Add-WindowsFeature Web-WebSockets; `
    Set-ExecutionPolicy Bypass -Scope Process -Force; `
    Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1')); `
    choco install googlechrome -y;

# Download the chrome driver based on the first 3 parts of the chrome version
RUN $chromeVersion3Part = [RegEx]::Match((Get-Item 'C:\Program Files (x86)\Google\Chrome\Application\chrome.exe').VersionInfo.FileVersion, '[0-9]{1,8}\.[0-9]{1,8}\.[0-9]{1,8}').Value; `
    $chromeDriverVersionUrl = 'https://chromedriver.storage.googleapis.com/LATEST_RELEASE_' + $chromeVersion3Part; `
    $chromeDriverVersion=(Invoke-WebRequest -UseBasicParsing $chromeDriverVersionUrl).Content; `
    New-Item -Type Directory 'C:\Selenium'; `
    $chromeDriverDownloadUrl='https://chromedriver.storage.googleapis.com/' + $chromeDriverVersion + '/chromedriver_win32.zip'; `
    Invoke-WebRequest $chromeDriverDownloadUrl -OutFile chromedriver.zip; `
    Expand-Archive -Path chromedriver.zip -DestinationPath 'C:\Selenium'; `
    Remove-Item chromedriver.zip;

LABEL OS=windows
LABEL MAINTAINER=tpittson
LABEL IMAGENAME=dotnetcore-selenium
LABEL VERSION=2.2-{ChromeVersion}-windows