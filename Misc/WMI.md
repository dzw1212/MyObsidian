`Windows Management Instrumentation (WMI)` 是 Windows 操作系统提供的一种基础管理技术。它允许脚本和应用程序通过统一的接口来管理和监控系统组件。`WMI` 提供了对操作系统、硬件、网络、服务、进程等各种系统信息的访问。

文档：
*https://learn.microsoft.com/en-us/windows/win32/CIMWin32Prov/cimwin32-wmi-providers*

# 主要功能

1. **系统信息查询**：获取操作系统、硬件、网络等信息。
2. **系统管理**：管理系统服务、进程、事件日志等。
3. **事件监控**：监控系统事件，如硬件变化、服务状态变化等。
4. **脚本自动化**：通过脚本语言（如 VBScript、PowerShell）自动化管理任务。

# 使用步骤

1. **初始化 COM 库**：WMI 基于 COM（Component Object Model）技术，需要先初始化 COM 库。
2. **设置安全级别**：确保 WMI 查询的安全性。
3. **获取 WMI 服务指针**：通过 `IWbemLocator` 接口连接到 WMI 服务。
4. **执行 WMI 查询**：使用 WQL（WMI Query Language）查询所需的信息。
5. **处理查询结果**：遍历查询结果并提取所需信息。
6. **清理资源**：释放 COM 对象和资源。

# 示例代码

以下是一个使用 C++ 通过 WMI 查询操作系统信息的示例代码：

```cpp
#include <iostream>
#include <comdef.h>
#include <Wbemidl.h>

#pragma comment(lib, "wbemuuid.lib")

void QueryWMI(const BSTR query, const std::vector<BSTR>& properties) {
    HRESULT hres;
    IWbemLocator *pLoc = NULL;
    IWbemServices *pSvc = NULL;
    IEnumWbemClassObject* pEnumerator = NULL;

    // 初始化COM库
    hres = CoInitializeEx(0, COINIT_MULTITHREADED);
    if (FAILED(hres)) {
        std::cerr << "Failed to initialize COM library. Error code = 0x" 
                  << std::hex << hres << std::endl;
        return;
    }

    // 设置COM安全级别
    hres = CoInitializeSecurity(
        NULL, 
        -1, 
        NULL, 
        NULL, 
        RPC_C_AUTHN_LEVEL_DEFAULT, 
        RPC_C_IMP_LEVEL_IMPERSONATE, 
        NULL, 
        EOAC_NONE, 
        NULL
    );

    if (FAILED(hres)) {
        std::cerr << "Failed to initialize security. Error code = 0x" 
                  << std::hex << hres << std::endl;
        CoUninitialize();
        return;
    }

    // 获取WMI服务的指针
    hres = CoCreateInstance(
        CLSID_WbemLocator,             
        0, 
        CLSCTX_INPROC_SERVER, 
        IID_IWbemLocator, (LPVOID *)&pLoc);

    if (FAILED(hres)) {
        std::cerr << "Failed to create IWbemLocator object. "
                  << "Error code = 0x" << std::hex << hres << std::endl;
        CoUninitialize();
        return;
    }

    hres = pLoc->ConnectServer(
        _bstr_t(L"ROOT\\CIMV2"), 
        NULL, 
        NULL, 
        0, 
        NULL, 
        0, 
        0, 
        &pSvc
    );

    if (FAILED(hres)) {
        std::cerr << "Could not connect. Error code = 0x" 
                  << std::hex << hres << std::endl;
        pLoc->Release();     
        CoUninitialize();
        return;
    }

    // 设置WMI连接的安全级别
    hres = CoSetProxyBlanket(
       pSvc,                        
       RPC_C_AUTHN_WINNT,           
       RPC_C_AUTHZ_NONE,            
       NULL,                        
       RPC_C_AUTHN_LEVEL_CALL,      
       RPC_C_IMP_LEVEL_IMPERSONATE, 
       NULL,                        
       EOAC_NONE                    
    );

    if (FAILED(hres)) {
        std::cerr << "Could not set proxy blanket. Error code = 0x" 
                  << std::hex << hres << std::endl;
        pSvc->Release();
        pLoc->Release();     
        CoUninitialize();
        return;
    }

    // 执行WMI查询
    hres = pSvc->ExecQuery(
        bstr_t("WQL"), 
        query,
        WBEM_FLAG_FORWARD_ONLY | WBEM_FLAG_RETURN_IMMEDIATELY, 
        NULL,
        &pEnumerator);

    if (FAILED(hres)) {
        std::cerr << "Query failed. Error code = 0x" 
                  << std::hex << hres << std::endl;
        pSvc->Release();
        pLoc->Release();
        CoUninitialize();
        return;
    }

    // 处理查询结果
    IWbemClassObject *pclsObj = NULL;
    ULONG uReturn = 0;

    while (pEnumerator) {
        HRESULT hr = pEnumerator->Next(WBEM_INFINITE, 1, &pclsObj, &uReturn);

        if (0 == uReturn) {
            break;
        }

        for (const auto& property : properties) {
            VARIANT vtProp;
            hr = pclsObj->Get(property, 0, &vtProp, 0, 0);
            std::wcout << property << L" : " << vtProp.bstrVal << std::endl;
            VariantClear(&vtProp);
        }

        pclsObj->Release();
    }

    // 清理和释放资源
    pSvc->Release();
    pLoc->Release();
    pEnumerator->Release();
    CoUninitialize();
}

int main() {
    std::vector<BSTR> properties = {
        L"Caption",
        L"Version",
        L"Manufacturer",
        L"BuildNumber",
        L"LastBootUpTime",
        L"OSArchitecture",
        L"SerialNumber",
        L"InstallDate",
        L"Name",
        L"ServicePackMajorVersion",
        L"ServicePackMinorVersion",
        L"SystemDirectory",
        L"WindowsDirectory",
        L"FreePhysicalMemory",
        L"TotalVirtualMemorySize",
        L"FreeVirtualMemory",
        L"NumberOfProcesses"
    };

    QueryWMI(L"SELECT * FROM Win32_OperatingSystem", properties);

    return 0;
}
```

# 命令行查询

Powershell下：

```powershell
PS C:\Users\dzw> Get-WmiObject -Class Win32_VideoController | Select-Object *
PS C:\Users\dzw> Get-WmiObject -Class Win32_OperatingSystem | Select-Object *
PS C:\Users\dzw> Get-WmiObject -Class Win32_Processor | Select-Object *
```
# 参考资料

- [WMI Classes](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-classes)
- [WMI Query Language (WQL)](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wql-sql-for-wmi)
- [WMI Tools](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-tools)

# 第三方工具

*https://github.com/vinaypamnani/wmie2*