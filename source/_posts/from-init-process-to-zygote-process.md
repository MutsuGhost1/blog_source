title: From Init Process To Zygote Process
date: 2015-01-29 19:23:42
categories: Software
tags: [Android, Booting]
---

從 Android Booting Process 這篇文章中, 我們可以了解, 從開機 Power On 到 Boot Complete (Home Screen) 的流程.
<!--more-->

本篇文章將針對開機流程中的特定部分來做細部的說明:
就由 Init Process 如何產生出 Zygote Process 以及 System Server Process 說起好了.
一旦 Zygote Process 被產生後, 後續 Android Apk Process 都是透過 Zygote 產生的.

統一透過 Zygote 產生 Android Apk Process 是利用了 Unix-Like 系統的 Fork 特性, 
先將所有 Apk 需要的 Resources 都透過 Zygote 先產生, 到時候 Fork 出來的 Process 將直接複製原有的 Process Layout,
減少建立 Apk Process 時的 Loading.

## Sequence Diagram for Starting Zygote
![](/images/from-init-process-to-zygote-process/start-zygote.gif)

## Sequence Diagram for Start a Process from Zygote
![](/images/from-init-process-to-zygote-process/start-process.png)

## 相關檔案列表

- AppProcess 相關 
 - @ /system/core/rootdir/
     - init.rc
     - init.zygote*.rc
 - @ /frameworks/base/cmds/app_process/
     - app_main.cpp 

- Zygote 相關 
 - @ /frameworks/base/core/java/com/android/internal/os/
     - Zygote.java
     - ZygoteInit.java
     - ZygoteConnection.java
     - ZygoteSecurityException.java
     - RuntimeInit.java
     - WrapperInit.java
     - BaseCommand.java
 - @ /frameworks/base/core/jni/
     - com_android_internal_os_Zygote.cpp (用來重啟 system server)
 - @ /frameworks/base/cmds/* (讓 adb 可以與 framework 相關 service 做溝通)
     - 透過 app_main.cpp 中的 non-zygote mode 來達成  

## Misc

Android L 以前, 似乎只支援 zygote with start system server, 可以從下列代碼窺得一二. 
重點在於第一個參數不為 start-system-server 就會丟 exception, 也意味著一定要有此參數.


    // http://androidxref.com/4.4.4_r1/xref/
    //       frameworks/base/core/java/com/android/internal/os/ZygoteInit.java#562
    // from app_main.cpp
    public static void main(String argv[]) {
    	try {
    		// ... skip 
     
    		// If requested, start system server directly from Zygote
    		if (argv.length != 2) {
    			throw new RuntimeException(argv[0] + USAGE_STRING);
    		}
    
    		if (argv[1].equals("start-system-server")) {
    			startSystemServer();
    		} else if (!argv[1].equals("")) {
    			throw new RuntimeException(argv[0] + USAGE_STRING);
    		}
    
    		Log.i(TAG, "Accepting command socket connections");
    
    		runSelectLoop();
    
    		closeServerSocket();
    	} catch (MethodAndArgsCaller caller) {
    		caller.run();
    	} catch (RuntimeException ex) {
    		Log.e(TAG, "Zygote died with exception", ex);
    		closeServerSocket();
    		throw ex;
    	}
    }


相較於 L 版本的代碼, 可以使用 zygote without start-system-server, 我想應該與 Secondary Zygote 有關.
    
    // http://androidxref.com/5.0.0_r2/xref/
    //      frameworks/base/core/java/com/android/internal/os/ZygoteInit.java#644
    // from app_main.cpp
    public static void main(String argv[]) {
    	try {
    		// skip ...
    
		    if (startSystemServer) {
    			startSystemServer(abiList, socketName);
    		}
    
			Log.i(TAG, "Accepting command socket connections");
    		runSelectLoop(abiList);
    
		    closeServerSocket();
    	} catch (MethodAndArgsCaller caller) {
    		caller.run();
    	} catch (RuntimeException ex) {
    		Log.e(TAG, "Zygote died with exception", ex);
    		closeServerSocket();
    		throw ex;
    	}
    }

## Reference
1. http://blog.csdn.net/szzhaom/article/details/23621193#t2
2. http://blog.csdn.net/luoshengyang/article/details/6768304
3. http://blog.csdn.net/jeffbao/article/details/18242515