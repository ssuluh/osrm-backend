# escape=`

# Copyright (C) Microsoft Corporation. All rights reserved.
# Licensed under the MIT license. See LICENSE.txt in the project root for license information.

ARG FROM_IMAGE=mcr.microsoft.com/windows/servercore:ltsc2022
FROM mcr.microsoft.com/windows/servercore:ltsc2022 as base

# Reset the shell.
SHELL ["cmd", "/S", "/C"]

# Set up environment to collect install errors.
COPY windows-container-build\install.cmd C:\TEMP\
ADD https://aka.ms/vscollect.exe C:\TEMP\collect.exe

# # Download channel for fixed install.
ARG CHANNEL_URL=https://aka.ms/vs/17/release/channel
ADD ${CHANNEL_URL} C:\TEMP\VisualStudio.chman

RUN `
    # Download the Build Tools bootstrapper.
    curl -SL --output vs_buildtools.exe https://aka.ms/vs/17/release/vs_buildtools.exe `
    `
    # Install Build Tools with the Microsoft.VisualStudio.Workload.AzureBuildTools workload, excluding workloads and components with known issues.
    && (call C:\TEMP\Install.cmd vs_buildtools.exe --quiet --wait --norestart --nocache install `
        --installPath "%ProgramFiles(x86)%\Microsoft Visual Studio\2022\BuildTools" `
        --channelUri C:\TEMP\VisualStudio.chman `
        --installChannelUri C:\TEMP\VisualStudio.chman `
        --add Microsoft.VisualStudio.Workload.ManagedDesktopBuildTools `
        --add Microsoft.VisualStudio.Workload.VCTools `
        --add Microsoft.VisualStudio.Workload.MSBuildTools `
        --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 `
        --add Microsoft.VisualStudio.Component.Windows10SDK.18362) `
    `
    # Cleanup
    && del /q vs_buildtools.exe
# ========== setup chocolatey package manager and tools ==========
RUN powershell.exe -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SETX PATH "%PATH%;%ALLUSERSPROFILE%\chocolatey\bin" 


# Moving forward this is OSRM specific
FROM base AS osrm_builder


RUN choco install -y git.install python3 && choco install -y nodejs --version=16.20.0

# Install conan tooling
RUN py -m pip install cmake
RUN py -m pip install conan==1.53.0

# Prep conan profile and cache
ENV CONAN_USER_HOME="c:/cache"
ENV CONAN_USER_HOME_SHORT="c:/cache/conan_shortpaths"
VOLUME ["c:/cache"]
RUN conan profile new default --detect

WORKDIR C:\OSRM
COPY package.json .
COPY package-lock.json .
RUN npm install --ignore-scripts && npm link --ignore-scripts
COPY . .
# RUN .\scripts\ci\windows-build.bat


# # we'll mount a volume from the host where the conan packages will be installed
# ENV CONAN_USER_HOME="c:/cache"
# ENV CONAN_USER_HOME_SHORT="c:/cache/conan_shortpaths"
# RUN conan profile new default --detect
# # adapt msvc runtime if you need dynamic linking or debug symbols
# RUN conan profile update settings.compiler.runtime=MT default

# # create the docker volume
# VOLUME ["c:/cache"] 

# ENTRYPOINT ["C:\\BuildTools\\Common7\\Tools\\VsDevCmd.bat", "&&", "powershell.exe", "-NoLogo", "-ExecutionPolicy", "Bypass"]
# Define the entry point for the Docker container.
# This entry point starts the developer command prompt and launches the PowerShell shell.
ENTRYPOINT ["C:\\Program Files (x86)\\Microsoft Visual Studio\\2022\\BuildTools\\Common7\\Tools\\VsDevCmd.bat", "&&", "powershell.exe", "-NoLogo", "-ExecutionPolicy", "Bypass"]
