FROM microsoft/windowsservercore:1709

ENV chocolateyUseWindowsCompression=false
RUN @powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

RUN choco config set cachelocation C:\chococache

RUN choco install \
    git  \
    netfx-4.5.2-devpack \
    netfx-4.6.1-devpack \
    netfx-4.6.2-devpack \
    dotnetcore-sdk \
    typescript \
    nuget.commandline \
    --confirm \
    --limit-output \
    --timeout 216000 \
    && rmdir /S /Q C:\chococache

SHELL ["powershell", "-ExecutionPolicy", "Bypass", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Download log collection utility
RUN $ErrorActionPreference = 'Stop'; \
    $ProgressPreference = 'SilentlyContinue'; \
    $VerbosePreference = 'Continue'; \
    Invoke-WebRequest -Uri https://aka.ms/vscollect.exe -OutFile C:\collect.exe

# Download build tools
RUN $ErrorActionPreference = 'Stop'; \
    $ProgressPreference = 'SilentlyContinue'; \
    $VerbosePreference = 'Continue'; \
    Invoke-WebRequest -Uri "https://aka.ms/vs/15/release/vs_buildtools.exe" -OutFile C:\vs_buildtools.exe;

# Install build tools
RUN $ErrorActionPreference = 'Stop'; \
    $VerbosePreference = 'Continue'; \
    $p = Start-Process -Wait -PassThru -FilePath C:\vs_buildtools.exe -ArgumentList '--quiet --nocache --wait'; \
    if ($ret = $p.ExitCode) { c:\collect.exe; throw ('Install failed with exit code 0x{0:x}' -f $ret) }

RUN Invoke-WebRequest "https://github.com/Microsoft/vsts-agent/releases/download/v2.124.0/vsts-agent-win7-x64-2.124.0.zip" \
    -OutFile vsts-agent.zip; \
    Expand-Archive vsts-agent.zip -DestinationPath "C:\BuildAgent" -Force; \
    Remove-Item -Force vsts-agent.zip

RUN Copy-Item '"C:\Program Files\dotnet\sdk\2.0.3\Sdks"' -Destination '"C:\Program Files (x86)\Microsoft Visual Studio\2017\BuildTools\MSBuild"' -Recurse -Container -Force

SHELL ["cmd", "/S", "/C"]

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip

RUN mkdir C:\warmup \
    && cd C:\warmup \
    && dotnet new mvc \
    && cd .. \
    && rmdir /S /Q C:\warmup 

WORKDIR C:/BuildAgent
COPY ./Start.* ./

CMD ["Start.cmd"]