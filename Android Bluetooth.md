---
tags : Android,Bluetooth
title : Android Bluetooth
---
# Bluetooth

## 必需完成的主要工作：
1. 設定藍芽
2. 搜尋已配對的或接收範圍內的裝置
3. 連接裝置
4. 傳輸資料


#### 首先要讓程式有足夠的**權限**使用藍芽裝置: 
BLUETOOTH :讓程式有權限連接裝置、傳輸資料。
BLUETOOTH_ADMIN :讓程式有權限搜尋裝置、設定藍芽。
```shell= 
<manifest ... >
   <uses-permission android:name="android.permission.BLUETOOTH"/>
   <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
...
</manifest>
``` 

## 設定藍芽
程式要先確認執行環境是否支援藍芽，還要再確認藍芽是否有開啟，
如果沒有，可在程式中要求使用者開啟。

1. 透過取得 BluetoothAdapter 來判斷執行環境是否支援藍芽。
```shell= 
BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (mBluetoothAdapter == null) {
    // Device does not support Bluetooth
}
```
2. 使用BluetoothAdapter 的 isEnabled() 來判斷藍芽是否被開啟。
如果沒有，使用系統提供的BluetoothAdapter.ACTION_REQUEST_ENABLE 對話窗來要求使用者開啟藍芽。在onActivityResult()可以得知對話窗的結果–同意開啟或不同意。
```shell= 
if (!mBluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
}
```

## 詢問已配對裝置

BluetoothAdapter 的getBondedDevices()可以用來取得已配對裝置的列表。

當第一次要與裝置建立連線前，會對使用者發出一個配對的要求

當配對完成，裝置的基本資訊會被儲存下來，getBondedDevices()取得的就是這些資訊，有了這些資訊才能做接下來的連接。
```shell= 
Set<BluetoothDevice> pairedDevices = mBluetoothAdapter.getBondedDevices();
// If there are paired devices
if (pairedDevices.size() > 0) {
    // Loop through paired devices
    for (BluetoothDevice device : pairedDevices) {
        // Add the name and address to an array 
        // adapter to show in a ListView
        mArrayAdapter.add(device.getName() + "\n" + device.getAddress());
    }
}
```

## 搜尋裝置 

BluetoothAdapter 的startDiscovery()可以開始搜尋裝置
cancelDiscovery()則結束搜尋

為了要讓程式接收到搜尋的結果，必須要注冊一個BroadcastReceiver負責處理。
```shell=
// Create a BroadcastReceiver for ACTION_FOUND
private final BroadcastReceiver mReceiver = new BroadcastReceiver()
{
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        // When discovery finds a device
        if (BluetoothDevice.ACTION_FOUND.equals(action)) {
            // Get the BluetoothDevice object from the Intent
            BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
            // Add the name and address to an array adapter to 
            // show in a ListView
            mArrayAdapter.add(device.getName() + "\n" + device.getAddress());
        }
    }
};
// Register the BroadcastReceiver
IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
registerReceiver(mReceiver, filter); 
// Don't forget to unregister during onDestroy 
```
> 搜尋裝置的動作是非常佔用資源的，所以在找到想要的裝置後，應立刻結束搜尋。同時，如果有已經建立的藍芽連線時，也不適合做搜尋的動作，因為會嚴重影響到連線的動作。

## 連接裝置(當Server)
用BluetoothAdapter 的 listenUsingRfcommWithServiceRecord() 取得BluetoothServerSocket，然後呼叫BluetoothServerSocket的accept()來等待接受遠端的連接要求。

如果連接順利完成，accept()會傳回一個已連接的BluetoothSocket，然後可以用close()關閉BluetoothServerSocket，除非還需連接其他的裝置。 由於accept()會一直等待直到遠端有連接的要求，所以不建議直接在主要的Activity thread中使用。
```shell=
private class AcceptThread extends Thread {
    private final BluetoothServerSocket mmServerSocket;

    public AcceptThread() {
        // Use a temporary object that is later assigned to mmServerSocket,
        // because mmServerSocket is final
        BluetoothServerSocket tmp = null;
        try {
            // MY_UUID is the app's UUID string, also used by the client code
            tmp = mBluetoothAdapter.listenUsingRfcommWithServiceRecord(NAME, MY_UUID);
        } catch (IOException e) { }
        mmServerSocket = tmp;
    }

    public void run() {
        BluetoothSocket socket = null;
        // Keep listening until exception occurs or a socket is returned
        while (true) {
            try {
                socket = mmServerSocket.accept();
            } catch (IOException e) {
                break;
            }
            // If a connection was accepted
            if (socket != null) {
                // Do work to manage the connection (in a separate thread)
                manageConnectedSocket(socket);
                mmServerSocket.close();
                break;
            }
        }
    }

    /** Will cancel the listening socket, and cause the thread to finish */
    public void cancel() {
        try {
            mmServerSocket.close();
        } catch (IOException e) { }
    }
}
```
## 連接裝置(當Client)
必須先有遠端裝置BluetoothDevice，然後用它的createRfcommSocketToServiceRecord ()取得一個為連接到這個遠端裝置而初始化的BluetoothSocket，用connect()連接到這個遠端裝置。 
connect()同樣也是個blocking call，所以同樣建議要產生新的thread來用。

```shell=
private class ConnectThread extends Thread {
    private final BluetoothSocket mmSocket;
    private final BluetoothDevice mmDevice;

    public ConnectThread(BluetoothDevice device) {
        // Use a temporary object that is later assigned to mmSocket,
        // because mmSocket is final
        BluetoothSocket tmp = null;
        mmDevice = device;

        // Get a BluetoothSocket to connect with the given BluetoothDevice
        try {
            // MY_UUID is the app's UUID string, also used by the server code
            tmp = device.createRfcommSocketToServiceRecord(MY_UUID);
        } catch (IOException e) { }
        mmSocket = tmp;
    }

    public void run() {
        // Cancel discovery because it will slow down the connection
        mBluetoothAdapter.cancelDiscovery();

        try {
            // Connect the device through the socket. This will block
            // until it succeeds or throws an exception
            mmSocket.connect();
        } catch (IOException connectException) {
            // Unable to connect; close the socket and get out
            try {
                mmSocket.close();
            } catch (IOException closeException) { }
            return;
        }

        // Do work to manage the connection (in a separate thread)
        manageConnectedSocket(mmSocket);
    }

    /** Will cancel an in-progress connection, and close the socket */
    public void cancel() {
        try {
            mmSocket.close();
        } catch (IOException e) { }
    }
}
```
## 傳輸資料 

當兩個裝置成功的建立連線，雙方都會有一個已連線的BluetoothSocket
可以從BluetoothSocket 的getInputStream()或getOutputStream()來取得InputStream或OutputStream；用read(), write()來傳送資料。

> read()是個blocking call，會一直等到讀取足夠的資料，所以一定要產生新的thread來處理，而write()不一定會被block，除非遠端一直不做read()，那就有可能因為要做 flow control而block住write()。

```shell=
private class ConnectedThread extends Thread {
    private final BluetoothSocket mmSocket;
    private final InputStream mmInStream;
    private final OutputStream mmOutStream;

    public ConnectedThread(BluetoothSocket socket) {
        mmSocket = socket;
        InputStream tmpIn = null;
        OutputStream tmpOut = null;

        // Get the input and output streams, using temp objects because
        // member streams are final
        try {
            tmpIn = socket.getInputStream();
            tmpOut = socket.getOutputStream();
        } catch (IOException e) { }

        mmInStream = tmpIn;
        mmOutStream = tmpOut;
    }

    public void run() {
        byte[] buffer = new byte[1024];  // buffer store for the stream
        int bytes; // bytes returned from read()

        // Keep listening to the InputStream until an exception occurs
        while (true) {
            try {
                // Read from the InputStream
                bytes = mmInStream.read(buffer);
                // Send the obtained bytes to the UI Activity
                mHandler.obtainMessage(MESSAGE_READ, bytes, -1, buffer)
                        .sendToTarget();
            } catch (IOException e) {
                break;
            }
        }
    }

    /* Call this from the main Activity to send data to the remote device */
    public void write(byte[] bytes) {
        try {
            mmOutStream.write(bytes);
        } catch (IOException e) { }
    }

    /* Call this from the main Activity to shutdown the connection */
    public void cancel() {
        try {
            mmSocket.close();
        } catch (IOException e) { }
    }
}
```

:::info
Ref:
- [Android 上的 Bluetooth](https://ivanliang86.pixnet.net/blog/post/102107617)
- [Android Bluetooth（藍牙）實例](http://tw.gitbook.net/android/android_bluetooth.html)

:::





