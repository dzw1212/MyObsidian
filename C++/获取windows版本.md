
# 各个版本的版本号

https://en.wikipedia.org/wiki/List_of_Microsoft_Windows_versions
https://learn.microsoft.com/zh-cn/windows/win32/sysinfo/operating-system-version

![windowsVersion|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230706123745.png)


# GetVersionEx

该函数在Windows8.1以上的版本中已弃用，获取的结果可能是错误的；
绕过弃用报错后，在win11的系统上使用，得到的结果是`6.2.9200`，是错误的；
[[绕过弃用报错]]

```cpp
#include <iostream>
#include <Windows.h>

int main() {
    OSVERSIONINFO osvi;
    ZeroMemory(&osvi, sizeof(OSVERSIONINFO));
    osvi.dwOSVersionInfoSize = sizeof(OSVERSIONINFO);

    if (GetVersionEx(&osvi)) {
        std::cout << "Windows Version: " << osvi.dwMajorVersion << "." << osvi.dwMinorVersion << std::endl;
    } else {
        std::cout << "Failed to get Windows version." << std::endl;
    }

    return 0;
}
```

# VerifyVersionInfo

该函数同样已被弃用；
在win11上，major为6和minor为2的组合可以验证通过；

```cpp
#include <iostream>
#include <Windows.h>

int main() {
    OSVERSIONINFOEX osvi;
    ZeroMemory(&osvi, sizeof(OSVERSIONINFOEX));
    osvi.dwOSVersionInfoSize = sizeof(OSVERSIONINFOEX);
    osvi.dwMajorVersion = 10; // Windows 10
    osvi.dwMinorVersion = 0; // Windows 10

    DWORDLONG conditionMask = 0;
    VER_SET_CONDITION(conditionMask, VER_MAJORVERSION, VER_EQUAL);
    VER_SET_CONDITION(conditionMask, VER_MINORVERSION, VER_EQUAL);

    if (VerifyVersionInfo(&osvi, VER_MAJORVERSION | VER_MINORVERSION, conditionMask)) {
        std::cout << "Windows version is Windows 10." << std::endl;
    } else {
        std::cout << "Windows version is not Windows 10." << std::endl;
    }

    return 0;
}
```

# IsWindow XXX

使用该类API目前没有办法能分辨win10和win11；

```cpp
#include <windows.h>
#include <stdio.h>
#include <VersionHelpers.h>

int 
__cdecl
wmain(
    __in int argc, 
    __in_ecount(argc) PCWSTR argv[]
    )
{
    UNREFERENCED_PARAMETER(argc);
    UNREFERENCED_PARAMETER(argv);

    if (IsWindowsXPOrGreater())
    {
        printf("XPOrGreater\n");
    }

    if (IsWindowsXPSP1OrGreater())
    {
        printf("XPSP1OrGreater\n");
    }

    if (IsWindowsXPSP2OrGreater())
    {
        printf("XPSP2OrGreater\n");
    }

    if (IsWindowsXPSP3OrGreater())
    {
        printf("XPSP3OrGreater\n");
    }

    if (IsWindowsVistaOrGreater())
    {
        printf("VistaOrGreater\n");
    }

    if (IsWindowsVistaSP1OrGreater())
    {
        printf("VistaSP1OrGreater\n");
    }

    if (IsWindowsVistaSP2OrGreater())
    {
        printf("VistaSP2OrGreater\n");
    }

    if (IsWindows7OrGreater())
    {
        printf("Windows7OrGreater\n");
    }

    if (IsWindows7SP1OrGreater())
    {
        printf("Windows7SP1OrGreater\n");
    }

    if (IsWindows8OrGreater())
    {
        printf("Windows8OrGreater\n");
    }

    if (IsWindows8Point1OrGreater())
    {
        printf("Windows8Point1OrGreater\n");
    }

    if (IsWindows10OrGreater())
    {
        printf("Windows10OrGreater\n");
    }

    if (IsWindowsServer())
    {
        printf("Server\n");
    }
    else
    {
        printf("Client\n");
    }
}
```


# RtlGetVersion

由于win10和win11的`MajorVersion`和`MinorVersion`均为10和0，因此需要通过`BuildNumber`区分；

https://www.codeproject.com/Articles/5336372/Windows-11-Version-Detection

```cpp
#include <windows.h>
#pragma comment(lib, "ntdll.lib")

extern "C"
NTSYSAPI
NTSTATUS
NTAPI
RtlGetVersion(
    _Out_ PRTL_OSVERSIONINFOW lpVersionInformation
);

bool WinVersion::GetVersion(VersionInfo& info)
{
	OSVERSIONINFOEXW osv;
	osv.dwOSVersionInfoSize = sizeof(OSVERSIONINFOEXW);
	if (RtlGetVersion(&osv) == 0)
	{
		info.Major = osv.dwMajorVersion;
		info.Minor = osv.dwMinorVersion;
		info.BuildNum = osv.dwBuildNumber;
		if (osv.dwBuildNumber >= 22000)
			info.Major = 11;
		return true;
	}
	return false;
}
```