title: Binder-RemoteException
date: 2015-01-22 22:13:19
tags: [Android, Binder]
---

當你的程式呼叫 binder call 的時候, 發生了 Exception.
該如判斷這個 Exception 是 Local Side 引起 or Remote Side 引起.

<!--more-->

e.g. 當你的程式發生下列的 Exception, 請問一下這個 NullPointerException 是怎麼產生的?

**java.lang.NullPointerException: Attempt to invoke virtual method 'void com.android.bluetooth.gatt.ServiceDeclaration.addCharacteristic(java.util.UUID, int, int)' on a null object reference**


	// Entire callstack when NullPointerException occurs
    FATAL EXCEPTION: main
	Process: com.test.bluetooth.le.server, PID: 2238
	java.lang.NullPointerException: Attempt to invoke virtual method 'void com.android.bluetooth.gatt.ServiceDeclaration.addCharacteristic(java.util.UUID, int, int)' on a null object reference
	   at android.os.Parcel.readException(Parcel.java:1552)
	   at android.os.Parcel.readException(Parcel.java:1499)
	   at android.bluetooth.IBluetoothGatt$Stub$Proxy.addCharacteristic(IBluetoothGatt.java:1351)
	   at android.bluetooth.BluetoothGattServer.addService(BluetoothGattServer.java:590)
	   at com.test.bluetooth.le.server.BluetoothLeService.initGattServerServices(BluetoothLeService.java:413)
	   at com.test.bluetooth.le.server.BluetoothLeService.openGattServer(BluetoothLeService.java:421)
	   at com.test.bluetooth.le.server.DeviceControlActivity.openAndDisplayGattServer(DeviceControlActivity.java:114)
	   at com.test.bluetooth.le.server.DeviceControlActivity.access$3(DeviceControlActivity.java:108)
	   at com.test.bluetooth.le.server.DeviceControlActivity$1.onServiceConnected(DeviceControlActivity.java:145)
	   at android.app.LoadedApk$ServiceDispatcher.doConnected(LoadedApk.java:1208)
	   at android.app.LoadedApk$ServiceDispatcher$RunConnection.run(LoadedApk.java:1225)
	   at android.os.Handler.handleCallback(Handler.java:739)
	   at android.os.Handler.dispatchMessage(Handler.java:95)
	   at android.os.Looper.loop(Looper.java:135)
	   at android.app.ActivityThread.main(ActivityThread.java:5245)
	   at java.lang.reflect.Method.invoke(Native Method)
	   at java.lang.reflect.Method.invoke(Method.java:372)
	   at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:903)
       at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:698)

事實上這個 NullPointerException 是發生在 Binder Call 的 Callee Process 端 (Proxy).
透過 Parcel 從 Callee 端將相關的 Exception 送回 Caller Process 端 (Callee).

下面是一個 Proxy 的代碼 (與上述的 callstack 並無關係):

	// Proxy
	// Method Prototype: 
	//   public void print(java.lang.String str) throws android.os.RemoteException;
	//   public int add(int a, int b) throws android.os.RemoteException;
	private static class Proxy implements com.cloud.test.ICloudManager {
	    private android.os.IBinder mRemote;
	    
	    Proxy(android.os.IBinder remote) {
	        mRemote = remote;
	    }
	    @Override
	    public android.os.IBinder asBinder() {
	        return mRemote;
	    }
	    public java.lang.String getInterfaceDescriptor() {
	        return DESCRIPTOR;
	    }
	
	    @Override
	    public void print(java.lang.String str) throws android.os.RemoteException {
	        android.os.Parcel _data = android.os.Parcel.obtain();
	        android.os.Parcel _reply = android.os.Parcel.obtain();
	        try {
	            _data.writeInterfaceToken(DESCRIPTOR);
	            _data.writeString(str);
	            mRemote.transact(Stub.TRANSACTION_print, _data, _reply, 0);
	            _reply.readException();
	        } finally {
	            _reply.recycle();
	            _data.recycle();
	        }
	    }
	
	    @Override
	    public int add(int a, int b) throws android.os.RemoteException {
	        android.os.Parcel _data = android.os.Parcel.obtain();
	        android.os.Parcel _reply = android.os.Parcel.obtain();
	        int _result;
	        try {
	            _data.writeInterfaceToken(DESCRIPTOR);
	            _data.writeInt(a);
	            _data.writeInt(b);
	            mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);
	            _reply.readException();
	            _result = _reply.readInt();
	        } finally {
	            _reply.recycle();
	            _data.recycle();
	        }
	        return _result;
	    }
	}

從代碼中可以發現, 每次執行完 Binder Call 之後, 一定會透過 **_reply.readException()**,
來判斷 Remote 端是否有發生 Exception.

那麼到底有哪些 Exception 會從 Remote 端傳回來呢 ? (應該是有限的吧 XD ...)
看一下 Parcel.java 裡面的代碼:

    // When exception occurs in the remote side,
    //   the method will be invoked
	public final void writeException(Exception e) {
	    int code = 0;
	    if (e instanceof SecurityException) {
	        code = EX_SECURITY;
	    } else if (e instanceof BadParcelableException) {
	        code = EX_BAD_PARCELABLE;
	    } else if (e instanceof IllegalArgumentException) {
	        code = EX_ILLEGAL_ARGUMENT;
	    } else if (e instanceof NullPointerException) {
	        code = EX_NULL_POINTER;
	    } else if (e instanceof IllegalStateException) {
	        code = EX_ILLEGAL_STATE;
	    } else if (e instanceof NetworkOnMainThreadException) {
	        code = EX_NETWORK_MAIN_THREAD;
	    } else if (e instanceof UnsupportedOperationException) {
	        code = EX_UNSUPPORTED_OPERATION;
	    }
	    writeInt(code);
	    StrictMode.clearGatheredViolations();
	    if (code == 0) {
	        if (e instanceof RuntimeException) {
	            throw (RuntimeException) e;
	        }
	        throw new RuntimeException(e);
	    }
	    writeString(e.getMessage());
	}

    // Each binder call is finished,
    //   the method will be invoked to check
    //   whether there's any remote exception occurs
	public final void readException() {
	    int code = readExceptionCode();
	    if (code != 0) {
	        String msg = readString();
	        readException(code, msg);
	    }
	}
	
	public final void readException(int code, String msg) {
	    switch (code) {
	        case EX_SECURITY:
	            throw new SecurityException(msg);
	        case EX_BAD_PARCELABLE:
	            throw new BadParcelableException(msg);
	        case EX_ILLEGAL_ARGUMENT:
	            throw new IllegalArgumentException(msg);
	        case EX_NULL_POINTER:
	            throw new NullPointerException(msg);
	        case EX_ILLEGAL_STATE:
	            throw new IllegalStateException(msg);
	        case EX_NETWORK_MAIN_THREAD:
	            throw new NetworkOnMainThreadException();
	        case EX_UNSUPPORTED_OPERATION:
	            throw new UnsupportedOperationException(msg);
	    }
	    throw new RuntimeException("Unknown exception code: " + code
	            + " msg " + msg);
	}

從上述代碼可以很清楚的知道, 除了下列 Exception 會被忠實的帶回到 Local Caller 端:
- SecurityException
- BadParcelableException
- IllegalArgumentException
- NullPointerException
- IllegalStateException
- NetworkOnMainThreadException
- UnsupportedOperationException

其它從 Remote Callee 端欲帶回的其他 Exception, 都會被視為 RuntimeException.
非 RuntimeException 的部分, 就不會帶回 Local Callee 端了.

PS: OutOfMemoryException 除外, 也會被轉為 RuntimeException 傳回.

最後叮嚀一點, 千萬不要認為這個 Exception 是在 Caller 端引起的 XD.

----------
[Reference]
1. [http://androidxref.com/5.0.0_r2/xref/frameworks/base/core/java/android/os/Parcel.java](http://androidxref.com/5.0.0_r2/xref/frameworks/base/core/java/android/os/Parcel.java "Parcel.java")
2. [http://androidxref.com/5.0.0_r2/xref/frameworks/base/core/java/android/os/Binder.java#435](http://androidxref.com/5.0.0_r2/xref/frameworks/base/core/java/android/os/Binder.java#435 "Remote Side onTransact")