---
layout: post
title: 'Tip #7: Use winsat as systeminfo'
date: 2023-02-17 09:00 +0100
description: 
image: 
category:
- tips
- windows
tags: tips minitip powershell
mermaid: false
---
You may be familiar with the ``systeminfo`` command or ``msinfo32``. These commands are useful to get a quick overview of your hardware.

But did you know that Windows, even the latest Win11, has another tool called **Windows System Assessment Tool**[^1] with its command ``winsat``? It is, or rather was, designed to evaluate the hardware performance of your system. However, we can (ab)use it off-label: to extract some more detailed system infos by running.

>``winsat features -eef -xml c:\yourpath\winsatinfo.xml``
{: .prompt-info }

The resulting xml would look something like this:
Notice it get the even memory manufacturer, graphics driver version, CPU features, etc...

```xml
<?xml version="1.0" encoding="UTF-16"?>
<WinSAT>
    <ProgramInfo>
        <Name>WinSAT</Name>
        <Version>V10.0 ENG Build-22621.1</Version>
        <WinEIVersion>WinEI-1.60</WinEIVersion>
        <Title>Windows System Assessment Tool</Title>
        <ModulePath>C:\Windows\System32\WinSAT.exe</ModulePath>
        <CmdLine><![CDATA["C:\Windows\System32\WinSAT.exe" features -eef -xml d:\test.xml]]></CmdLine>
        <Note><![CDATA[]]></Note>
    </ProgramInfo>
    <SystemEnvironment>
        <ExecDateTOD Friendly="Thursday February 16, 2023  7:29:13pm">738567:70153144</ExecDateTOD>
        <IsOfficial>1</IsOfficial>
        <RanOverTs>0</RanOverTs>
        <RanOnBatteries>0</RanOnBatteries>
    </SystemEnvironment>
    <WinSPR>
        <VideoEncodeScore>9.9</VideoEncodeScore>
        <Dx9SubScore>9.9</Dx9SubScore>
        <Dx10SubScore>9.9</Dx10SubScore>
        <GamingScore>9.9</GamingScore>
        <LimitsApplied>
            <GamingScore>
                <LimitApplied Friendly="We no longer run the D3D test. Returned scores and metrics are hardcoded sentinel values.">NoD3DTestRun</LimitApplied>
            </GamingScore>
        </LimitsApplied>
    </WinSPR>
    <Metrics>
        <CPUMetrics>
            <DshowEncodeTime units="s">0.00000</DshowEncodeTime>
        </CPUMetrics>
        <GraphicsMetrics>
            <DshowVideoDecodeDur units="s">0.00000</DshowVideoDecodeDur>
            <MFVideoDecodeDur units="s">0.00000</MFVideoDecodeDur>
        </GraphicsMetrics>
    </Metrics>
    <SystemConfig>
        <OperationVersion Major="1" Minor="0" Build="0" Revision="0"/>
        <CmdLine/>
        <OSVersion>
            <Major>10</Major>
            <Minor>0</Minor>
            <Build>22621</Build>
            <ProductType>48</ProductType>
            <ProductName>Windows 10 Pro</ProductName>
            <OSName><![CDATA[Windows 11 Pro]]></OSName>
            <BuildLab>22621.ni_release.220506-1250</BuildLab>
        </OSVersion>
        <HistoryVersion>0</HistoryVersion>
        <Platform>
            <IsMobile>0</IsMobile>
            <PlatformRole desc="Desktop">1</PlatformRole>
        </Platform>
        <System>
            <MotherBoard>
                <Manufacturer><![CDATA[ASUSTeK COMPUTER INC.]]></Manufacturer>
                <Product><![CDATA[ROG STRIX Z390-E GAMING]]></Product>
                <Type>10</Type>
            </MotherBoard>
            <BIOS>
                <Vendor><![CDATA[American Megatrends Inc.]]></Vendor>
                <Version><![CDATA[2004]]></Version>
                <ReleaseDate><![CDATA[11/02/2021]]></ReleaseDate>
            </BIOS>
            <Machine>
                <Manufacturer><![CDATA[ASUS]]></Manufacturer>
                <ProductName><![CDATA[System Product Name]]></ProductName>
                <Version><![CDATA[System Version]]></Version>
            </Machine>
        </System>
        <Processor>
            <Instance id="0">
                <ProcessorName>Intel(R) Core(TM) i9-9900K CPU @ 3.60GHz</ProcessorName>
                <TSCFrequency>0</TSCFrequency>
                <NumProcs>1</NumProcs>
                <NumCores>8</NumCores>
                <NumCPUs>16</NumCPUs>
                <NumCPUsPerCore>2</NumCPUsPerCore>
                <NumCoresPerProcessor>8</NumCoresPerProcessor>
                <CoresAreThreaded>1</CoresAreThreaded>
                <X64Capable>1</X64Capable>
                <X64Running>1</X64Running>
                <Signature>
                    <Manufacturer friendly="Intel">2</Manufacturer>
                    <Stepping>12</Stepping>
                    <Model>14</Model>
                    <Family>6</Family>
                    <ExtendedModel>9</ExtendedModel>
                    <ExtendedFamily>0</ExtendedFamily>
                    <CompactSignature>REDACTED</CompactSignature>
                </Signature>
                <L1Cache>
                    <Size>32768</Size>
                    <Ways>8</Ways>
                    <LineSize>64</LineSize>
                    <SectorSize>32768</SectorSize>
                </L1Cache>
                <L2Cache>
                    <Size>262144</Size>
                    <Ways>4</Ways>
                    <LineSize>64</LineSize>
                    <SectorSize>262144</SectorSize>
                </L2Cache>
                <L3Cache>
                    <Size>16777216</Size>
                    <Ways>16</Ways>
                    <LineSize>64</LineSize>
                    <SectorSize>16777216</SectorSize>
                </L3Cache>
                <MMX>Yes</MMX>
                <SSE>Yes</SSE>
                <SSE2>Yes</SSE2>
                <SSE3>Yes</SSE3>
                <LogicalProcessorInfo>REDACTED</LogicalProcessorInfo>
            </Instance>
        </Processor>
        <Memory>
            <TotalPhysical>
                <Size>32GB</Size>
                <Bytes>34270449664</Bytes>
            </TotalPhysical>
            <AvailablePhysical>
                <Size>21GB</Size>
                <Bytes>23048445952</Bytes>
            </AvailablePhysical>
            <DIMM>
                <SizeInKB>16777216</SizeInKB>
                <Manufacturer><![CDATA[Kingston]]></Manufacturer>
                <PartNumber><![CDATA[KHX3000C15/16GX]]></PartNumber>
                <FormFactor>9</FormFactor>
                <MemoryType>26</MemoryType>
                <TypeDetail>128</TypeDetail>
            </DIMM>
            <DIMM>
                <SizeInKB>16777216</SizeInKB>
                <Manufacturer><![CDATA[Kingston]]></Manufacturer>
                <PartNumber><![CDATA[KHX3000C15/16GX]]></PartNumber>
                <FormFactor>9</FormFactor>
                <MemoryType>26</MemoryType>
                <TypeDetail>128</TypeDetail>
            </DIMM>
        </Memory>
        <Monitors>
            <Count>1</Count>
            <TotalMonitorPixels>4953600</TotalMonitorPixels>
            <Monitor id="0" primary="1">
                <DeviceName><![CDATA[\\.\DISPLAY1]]></DeviceName>
                <Width>3440</Width>
                <Height>1440</Height>
                <TotalMonitorPixels>4953600</TotalMonitorPixels>
            </Monitor>
        </Monitors>
        <Graphics>
            <AdapterDescription><![CDATA[NVIDIA GeForce RTX 2080 Ti]]></AdapterDescription>
            <AdapterManufacturer><![CDATA[NVIDIA]]></AdapterManufacturer>
            <DriverProvider><![CDATA[NVIDIA]]></DriverProvider>
            <DriverVersion Friendly="31.0.15.2824">8725724279016200</DriverVersion>
            <DriverDate Friendly="2023\1\15">738535:0</DriverDate>
            <DedicatedVideoMemory>11577327616</DedicatedVideoMemory>
            <DedicatedSystemMemory>0</DedicatedSystemMemory>
            <SharedSystemMemory>17135224832</SharedSystemMemory>
            <Suports32BitsPerPixel>1</Suports32BitsPerPixel>
            <D3D9OrBetter>1</D3D9OrBetter>
            <VertexShaderProfile>vs_3_0</VertexShaderProfile>
            <PixelShaderProfile>ps_3_0</PixelShaderProfile>
            <PixelShader2OrBetter>1</PixelShader2OrBetter>
            <PixelShader3OrBetter>1</PixelShader3OrBetter>
            <LDDM>1</LDDM>
            <WHQL>0</WHQL>
            <PNPID><![CDATA[PCI\VEN_10DE&DEV_1E07&SUBSYS_866A1043&REV_A1]]></PNPID>
            <DWMRunningOnStart>1</DWMRunningOnStart>
            <DWMRunning>1</DWMRunning>
        </Graphics>
        <Disk>
            <Disk id="\\.\PhysicalDrive0">
                <DiskNum>0</DiskNum>
                <Vendor><![CDATA[]]></Vendor>
                <Model><![CDATA[Samsung SSD 850 PRO 1TB]]></Model>
                <Size units="bytes">1024209543168</Size>
                <WriteCacheEnabled>TRUE</WriteCacheEnabled>
            </Disk>
            <Disk id="\\.\PhysicalDrive1">
                <DiskNum>1</DiskNum>
                <Vendor><![CDATA[]]></Vendor>
                <Model><![CDATA[Samsung SSD 840 PRO Series]]></Model>
                <Size units="bytes">512110190592</Size>
                <WriteCacheEnabled>TRUE</WriteCacheEnabled>
            </Disk>
            <SystemDisk id="\\.\PhysicalDrive2">
                <DiskNum>2</DiskNum>
                <Vendor><![CDATA[]]></Vendor>
                <Model><![CDATA[Samsung SSD 970 EVO 250GB]]></Model>
                <Size units="bytes">250059350016</Size>
                <WriteCacheEnabled>TRUE</WriteCacheEnabled>
            </SystemDisk>
            <Disk id="\\.\PhysicalDrive3">
                <DiskNum>3</DiskNum>
                <Vendor><![CDATA[]]></Vendor>
                <Model><![CDATA[INTEL SSDPEKNW010T8]]></Model>
                <Size units="bytes">1024209543168</Size>
                <WriteCacheEnabled>TRUE</WriteCacheEnabled>
            </Disk>
        </Disk>
        <CompletionStatus description="Success">0</CompletionStatus>
        <AssessmentRunTime>
            <Seconds>0</Seconds>
            <Description>00:00:00.00</Description>
        </AssessmentRunTime>
    </SystemConfig>
    <CompletionStatus description="Success">0</CompletionStatus>
    <TotalRunTime>
        <Seconds>0.16</Seconds>
        <Description>00:00:00.16</Description>
    </TotalRunTime>
</WinSAT>
```

### Footnotes

[^1]: see more here <https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc742091(v=ws.11)>
