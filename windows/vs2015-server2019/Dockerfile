FROM microsoft/dotnet-framework:4.7.2-sdk-windowsservercore-ltsc2019
# See https://github.com/Microsoft/dotnet-framework-docker/blob/33531c3bae718f970b704730703ba1fb0f3dff80/4.7.2/sdk/windowsservercore-ltsc2019/Dockerfile

# Container Compatibility
# See https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility

# We use a similar configuration as Microsoft's VSTS agents (this is deprecated)
# https://github.com/Microsoft/vsts-agent-docker/blob/master/windows/servercore/10.0.14393/standard/VS2017/Dockerfile

ENV chocolateyUseWindowsCompression=false

RUN iex (wget 'https://chocolatey.org/install.ps1' -UseBasicParsing)

RUN choco feature disable -n=showDownloadProgress
RUN choco config set cachelocation C:\chococache

RUN choco install \
    git \
    nodejs \
    jdk8 \
    curl \
    maven \
    gradle \
    docker \
    --confirm \
    --limit=output \
    --timeout 3600 \
    ; Remove-Item C:\chococache  -Recurse -Force

# NOTE: This step takes FOREVER. Be prepared to wait 4+ hours.
RUN choco install \
    visualstudio2015enterprise \
    --package-parameters "--passive --locale en-US --includeOptional" \
    --confirm \
    --limit-output \
    --timeout 81600 \
    ; Remove-Item C:\chococache  -Recurse -Force

# Install NuGet, .NET Framework 4.5, 4.6, 4.7
RUN Install-PackageProvider -Name chocolatey -RequiredVersion 2.8.5.130 -Force

RUN Install-Package nuget.commandline -Force -Confirm
RUN Install-Package netfx-4.0.3-devpack -Force -Confirm
RUN Install-Package netfx-4.5.2-devpack -Force -Confirm
RUN Install-Package netfx-4.6.2-devpack -Force -Confirm
RUN Install-Package netfx-4.7.2-devpack -Force -Confirm

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# Install .NET Core 1.0.5, 1.1.2
ENV DOTNET_DOWNLOAD_URL     https://download.microsoft.com/download/D/7/A/D7A9E4E9-5D25-4F0C-B071-210CB8267943/dotnet-win-x64.1.1.2.zip
ENV DOTNET_SDK_DOWNLOAD_URL https://download.microsoft.com/download/E/7/8/E782433E-7737-4E6C-BFBF-290A0A81C3D7/dotnet-dev-win-x64.1.0.4.zip

RUN Invoke-WebRequest $Env:DOTNET_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet.zip

RUN Invoke-WebRequest $Env:DOTNET_SDK_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet.zip

# Install .NET Core 2.0
ENV DOTNET_DOWNLOAD_URL     https://download.microsoft.com/download/5/F/0/5F0362BD-7D0A-4A9D-9BF9-022C6B15B04D/dotnet-runtime-2.0.0-win-x64.zip
ENV DOTNET_SDK_DOWNLOAD_URL https://download.microsoft.com/download/1/B/4/1B4DE605-8378-47A5-B01B-2C79D6C55519/dotnet-sdk-2.0.0-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet.zip

RUN Invoke-WebRequest $Env:DOTNET_SDK_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet.zip

# Install .NET Core 2.2
ENV DOTNET_DOWNLOAD_URL https://download.visualstudio.microsoft.com/download/pr/c332d70f-6582-4471-96af-4b0c17a616ad/5f3043d4bc506bf91cb89fa90462bb58/dotnet-sdk-2.2.103-win-x64.zip

RUN Invoke-WebRequest $Env:DOTNET_DOWNLOAD_URL -OutFile dotnet.zip; \
    Expand-Archive dotnet.zip -DestinationPath $Env:ProgramFiles\dotnet -Force; \
    Remove-Item -Force dotnet.zip

RUN [Environment]::SetEnvironmentVariable('PATH', "${Env:path}" + ';C:\Program Files (x86)\MSBuild\14.0\Bin', [EnvironmentVariableTarget]::Machine);
RUN [Environment]::SetEnvironmentVariable('PATH', "${Env:path}" + ';C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE', [EnvironmentVariableTarget]::Machine);
RUN [Environment]::SetEnvironmentVariable('PATH', "${Env:path}" + ';C:\Program Files (x86)\Microsoft Visual Studio 14.0\Common7\IDE\CommonExtensions\Microsoft\TestWindow', [EnvironmentVariableTarget]::Machine);
RUN [Environment]::SetEnvironmentVariable('PATH', "${Env:path}" + ';C:\Program Files\dotnet', [EnvironmentVariableTarget]::Machine);

# Trigger the population of the local package cache
ENV NUGET_XMLDOC_MODE skip

RUN dotnet --list-sdks ; \
    mkdir C:\warmup ; \
    cd C:\warmup ; \
    dotnet new ; \
    cd .. ; \
    Remove-Item C:\warmup -Force -Recurse
