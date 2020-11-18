# 逆向工程与软件安全实验报告  
## dll注入相关  
### 实验要求  
 - [x] 参照示例代码，成功编写dll文件(并未实现远程注入)  
 - [x] 调用已有函数，实现dll注入(注入notepad.exe)  

### 实验过程  
##### 编写dll  

1. 查看[示例代码](https://github.com/fdiskyou/injectAllTheThings)  
2. 在dllpoc项目下修改`dllpoc.cpp`  
    ```C
    #include <windows.h>
    #include <stdio.h>

    BOOL APIENTRY DllMain(HMODULE hModule, DWORD  ul_reason_for_call, LPVOID lpReserved)
    {
        switch (ul_reason_for_call)
        {
        case DLL_PROCESS_ATTACH:
            break;
        case DLL_PROCESS_DETACH:
            break;
        case DLL_THREAD_ATTACH:
            break;
        case DLL_THREAD_DETACH:
            break;
        }
        return TRUE;
    }

    //将函数声明为导出函数
    extern "C" __declspec(dllexport) BOOL cuc() 
    {
        MessageBox(NULL, L"CUC is called!", L"Inject All The Things!", 0);

        TCHAR szExePath[MAX_PATH], szInfo[MAX_PATH + 100];

        GetModuleFileName(NULL, szExePath, MAX_PATH);

        printf("I'm Process %d", GetCurrentProcessId());
    }
    ```  
    生成结果  
    ![生成结果](./img/生成结果.png)  
3. 在项目中新建`load.c`文件，实现成功调用`dllpoc.dll`  
![newdllexe](./img/newdllexe.png)  
    ```C
    //load.c
    #include <Windows.h>
    #include <stdio.h>

    typedef BOOL(*CUC_PROC_ADDR)();

    int main() 
    {
        //加载dll
        HMODULE hMoudle = LoadLibraryA("dllpoc.dll");
        CUC_PROC_ADDR cuc_ptr = (CUC_PROC_ADDR)GetProcAddress(hMoudle, "cuc");
        void* cuc = GetProcAddress(hMoudle, "cuc");
        //输出位置信息
        printf("CUC's pos: %p\n", cuc);
        cuc_ptr();
    }
    ```  
4. 成功调用  
![成功调用](./img/成功调用.png)  
5. 输出进程号  
![进程号](./img/进程号.png)  
6. 任务管理器证明  
![任务管理器证明](./img/证明.png)  

##### 实现dll注入  
通过调用[示例代码](https://github.com/fdiskyou/injectAllTheThings)中的`injectAllTheThings.exe`实现将新生成dll注入`notepad.exe`  
注：**要根据注入的目标程序更改dll生成**  
例如：`notepad.exe`程序为64位，在加载dll时要注意更改为64位，不然会出现注入失败的报错  
![64位](./img/64位.png)  

1. 在主机中打开`notepad.exe`程序，为注入创造必要条件  
![打开记事本](./img/打开记事本.png)  
2. 根据命令提示运行`injectAllTheThings.exe`实现dll注入  
`injectAllTheThings.exe -t 1 notepad.exe D:\injectAllTheThings-master\injectAllTheThings-master\x64\Release\dllmain.dll`  
![注入成功](./img/注入成功.png)  
3. 弹窗证明  
![注入成功](./img/成功.png)  



