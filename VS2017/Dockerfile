FROM microsoft/windowsservercore:10.0.14393.1358
ENV WINDOWS_IMAGE_VERSION=10.0.14393

ENV chocolateyUseWindowsCompression=false
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

RUN choco config set cachelocation C:\chococache

RUN choco install \
    git  \
    nodejs \
    curl \
    docker \
    netfx-4.5.2-devpack \
    netfx-4.6.1-devpack \
    netfx-4.6.2-devpack \
    dotnetcore-sdk \
    typescript \
    --confirm \
    --limit-output \
    --timeout 216000 \
    && rmdir /S /Q C:\chococache

# common node tools
RUN npm install -g less && npm install phantomjs-prebuilt -g

ENV TEST_CONTAINER=1 \
    VS_CHANNEL_URI=https://aka.ms/vs/15/release/channel \
    VS_BUILDTOOLS_URI=https://aka.ms/vs/15/release/vs_buildtools.exe \
    NUGET_URI=https://dist.nuget.org/win-x86-commandline/latest/nuget.exe \
    NUGET_SHA256=4C1DE9B026E0C4AB087302FF75240885742C0FAA62BD2554F913BBE1F6CB63A0

SHELL ["powershell", "-ExecutionPolicy", "Bypass", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Download nuget.exe
RUN $ErrorActionPreference = 'Stop'; \
    $ProgressPreference = 'SilentlyContinue'; \
    $VerbosePreference = 'Continue'; \
    New-Item -Path C:\bin -Type Directory | Out-Null; \
    [System.Environment]::SetEnvironmentVariable('PATH', "\"${env:PATH};C:\bin\"", 'Machine'); \
    Invoke-WebRequest -Uri $env:NUGET_URI -OutFile C:\bin\nuget.exe;

# Download log collection utility
RUN $ErrorActionPreference = 'Stop'; \
    $ProgressPreference = 'SilentlyContinue'; \
    $VerbosePreference = 'Continue'; \
    Invoke-WebRequest -Uri https://aka.ms/vscollect.exe -OutFile C:\collect.exe

# Download vs_community.exe
RUN $ErrorActionPreference = 'Stop'; \
    $ProgressPreference = 'SilentlyContinue'; \
    $VerbosePreference = 'Continue'; \
    Invoke-WebRequest -Uri "https://aka.ms/vs/15/release/vs_community.exe" -OutFile C:\vs_community.exe;

# Install Visual Studio Community
RUN $ErrorActionPreference = 'Stop'; \
    $VerbosePreference = 'Continue'; \
    $p = Start-Process -Wait -PassThru -FilePath C:\vs_community.exe -ArgumentList '--quiet --nocache --wait --addProductLang en-US --includeRecommended --add Microsoft.VisualStudio.Workload.ManagedDesktop --add Microsoft.VisualStudio.Workload.Data --add Microsoft.VisualStudio.Workload.NetWeb --add Microsoft.VisualStudio.Workload.NetCoreTools --add Microsoft.VisualStudio.Workload.Azure --add Component.GitHub.VisualStudio'; \
    if ($ret = $p.ExitCode) { c:\collect.exe; throw ('Install failed with exit code 0x{0:x}' -f $ret) }

RUN Invoke-WebRequest "https://github.com/Microsoft/vsts-agent/releases/download/v2.124.0/vsts-agent-win7-x64-2.124.0.zip" \
    -OutFile vsts-agent.zip; \
    Expand-Archive vsts-agent.zip -DestinationPath "C:\BuildAgent" -Force; \
    Remove-Item -Force vsts-agent.zip

SHELL ["cmd", "/S", "/C"]

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip

RUN mkdir C:\warmup \
    && cd C:\warmup \
    && dotnet new mvc \
    && dotnet restore \
    && cd .. \
    && rmdir /S /Q C:\warmup 

WORKDIR C:/BuildAgent
COPY ./Start.* ./

CMD ["Start.cmd"]