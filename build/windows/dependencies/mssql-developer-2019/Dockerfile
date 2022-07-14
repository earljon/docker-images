# escape=`
ARG BASE_IMAGE

FROM ${BASE_IMAGE}

LABEL maintainer "Earljon Hidalgo<earljon.hidalgo@mynrma.com.au>"

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV sa_password='_' `
    sa_password_path='C:\ProgramData\Docker\secrets\sa-password' `
    attach_dbs='[]' `
    ACCEPT_EULA='_' `
    DATA_PATH=C:\\data

# make install files accessible
COPY Start.ps1 .

RUN New-Item -Path 'C:\\install' -ItemType 'Directory' | Out-Null; `
    curl.exe -sS -L -o C:\\install\\SQL.exe 'https://download.microsoft.com/download/7/c/1/7c14e92e-bdcb-4f89-b7cf-93543e7112d1/SQLServer2019-DEV-x64-ENU.exe'; `
    curl.exe -sS -L -o C:\\install\\SQL.box 'https://download.microsoft.com/download/7/c/1/7c14e92e-bdcb-4f89-b7cf-93543e7112d1/SQLServer2019-DEV-x64-ENU.box'; `
    Start-Process -FilePath 'C:\\install\\SQL.exe' -ArgumentList '/qs', '/x:C:\install\setup' -NoNewWindow -Wait; `
    Start-Process -FilePath 'C:\\install\\setup\\setup.exe' -ArgumentList '/q', '/ACTION=Install', '/INSTANCENAME=MSSQLSERVER', '/FEATURES=SQLEngine', '/UPDATEENABLED=0', '/SQLSVCACCOUNT=\"NT AUTHORITY\System\"', '/SQLSYSADMINACCOUNTS=\"BUILTIN\ADMINISTRATORS\"', '/TCPENABLED=1', '/NPENABLED=0', '/IACCEPTSQLSERVERLICENSETERMS' -NoNewWindow -Wait; `
    Stop-Service MSSQLSERVER; `
    Set-ItemProperty -Path 'HKLM:\software\microsoft\microsoft sql server\mssql15.MSSQLSERVER\mssqlserver\supersocketnetlib\tcp\ipall' -Name 'tcpdynamicports' -Value ''; `
    Set-ItemProperty -Path 'HKLM:\software\microsoft\microsoft sql server\mssql15.MSSQLSERVER\mssqlserver\supersocketnetlib\tcp\ipall' -Name 'tcpport' -Value 1433; `
    Set-ItemProperty -Path 'HKLM:\software\microsoft\microsoft sql server\mssql15.MSSQLSERVER\mssqlserver' -Name 'LoginMode' -Value 2; `
    curl.exe -sS -L -o C:\\install\\DacFramework.msi 'https://download.microsoft.com/download/9/2/2/9228AAC2-90D1-4F48-B423-AF345296C7DD/EN/x64/DacFramework.msi'; `
    Start-Process -FilePath 'msiexec.exe' -ArgumentList '/i', 'C:\\install\\DacFramework.msi', '/quiet', '/norestart' -NoNewWindow -Wait; `
    Remove-Item -Path 'C:\\install' -Recurse -Force;

HEALTHCHECK CMD [ "sqlcmd", "-Q", "select 1" ]

CMD .\Start.ps1 -sa_password $env:sa_password -ACCEPT_EULA $env:ACCEPT_EULA -attach_dbs \"$env:attach_dbs\" -DataDirectory $env:DATA_PATH -Verbose