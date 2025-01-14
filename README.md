
 ### Crash解决
 
 环境：
 
 Mac OS: 1.5.2 
 
 XCode: 13.1

编译和运行遇到问题：

1）macOS not find

解决方法：target --> build setting --> base SDK 设置为支持的版本

2）string can't find

解决方法：target --> build Phases --> Link Binary 添加libc++.tbd

3）```- (void)writeString:(NSString *)str toFile:(FILE *)pFile```运行崩溃

解决方法：

```
- (void)writeString:(NSString *)str toFile:(FILE *)pFile
{
    if (str) { // 对str做判断
        fwrite(CSTRING(str), [str length] + 1, 1, pFile);
    }
}
```

4）```setHidden:```运行崩溃

解决方法：

```- (void)updateStatus: (NSString *)status
{
    dispatch_async(dispatch_get_main_queue(), ^{ // 放在主线程执行
        NSNotificationCenter * nc = [NSNotificationCenter defaultCenter];
        [nc postNotificationName:MVThreadStateChangedNotification
                          object:self
                        userInfo:[NSDictionary dictionaryWithObject:status forKey:MVStatusUserInfoKey]];
    });
  
}
```


这些修改已经在develop分支上提交，develop分支现在可运行成功。

### 官方介绍
   _____                .__     ____________   ____.__               
  /     \ _____    ____ |  |__  \_____  \   \ /   /|__| ______  _  __
 /  \ /  \\__  \ _/ ___\|  |  \  /   |   \   Y   / |  |/ __ \ \/ \/ /
/    Y    \/ __ \\  \___|   Y  \/    |    \     /  |  \  ___/\     / 
\____|__  (____  /\___  >___|  /\_______  /\___/   |__|\___  >\/\_/  
        \/     \/     \/     \/         \/                 \/        

A fork from MachOView to update and fix some bugs, mostly Mountain Lion & iOS 6 related.
Also some small changes to the original behaviour.

Original MachOView by psaghelyi at http://sourceforge.net/projects/machoview/.
Thanks to psaghelyi for his great work :-)

Latest versions are Lion+ only.
The LLVM disassembler was replaced with Capstone. This eliminates Clang/LLVM packages requirements.
The downside is that Capstone stops disassembling on bad instructions which means that for now data in code and jump tables data will create problems and __text section disassembly might be incomplete in binaries that contain such data.
Capstone improved disassembly on error but data in code locations are available in header so this can and should be improved.

A static Capstone library extracted from the official DMG is included in the repo.
If you want to be safe you should download Capstone and compile it yourself.

Now features the attach option to analyse headers of a running process.
To use this feature you will need to codesign the binary.
Follow this LLDB guide to create the certificate and then codesign MachOView binary.
https://llvm.org/svn/llvm-project/lldb/trunk/docs/code-signing.txt
The necessary entitlements are already added to Info.plist.

Be warned that this allows MachOView to have task_for_pid() privs under current under and control
every process from user running it.
The whole Mach-O parsing code needs to be reviewed and made more robust.

Enjoy,
fG!

Note:
This repo is frozen in time and there are kinda active forks out there.
The main problem of this codebase is that the Mach-O parser has quite some problems
and needs a significant overhaul to make it more robust and secure.
I do have much better code but it's under NDAs etc and I don't have energy to reinvent
the wheel once again. Secure executable binary parsing is a ton of work in C/C++.
It's possible but it's is exhausting.
