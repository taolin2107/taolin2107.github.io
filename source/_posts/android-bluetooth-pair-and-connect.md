---
title: Android蓝牙的配对与连接
date: 2015-12-23 15:49:16
tags:  [Android, bluetooth, 蓝牙配对, 蓝牙连接]
categories: android
---

这段时间在项目中做蓝牙的设置模块，功能跟系统设置中的蓝牙设置部分差不多，由于此项目只是一个普通app，蓝牙设置中的很多api只有系统app才能调用，如何在普通app中做蓝牙的设置项，记录如下

## 1. AndroidManifest中添加蓝牙管理权限
```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

## 2. 打开蓝牙

打开蓝牙有两种方式，发送蓝牙开启请求或代码后台自动打开蓝牙。

### 2.1. 发送蓝牙开启请求

先判断BluetoothAdapter是不是为空，为空有可能是系统没有蓝牙模块，再判断蓝牙的状态是不是开启的，不是开启的就发送请求。
```java
BluetoothAdapter btAdapter = BluetoothAdapter.getDefaultAdapter();
if (btAdapter != null && !btAdapter.isEnabled()) {
    Intent intent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivity(intent)
}
```
{% asset_img 01.png %}

### 2.2. 后台自动打开蓝牙
```java
if (btAdapter != null && !btAdapter.isEnabled()) {
    btAdapter.enable();
}
```
<!-- more -->

## 3. 获取已经绑定(配对)的设备
```java
//返回已绑定的蓝牙设备集
Set<BluetoothDevice> bondedDevices = btAdapter.getBondedDevices();
```

**BluetoothDevice有如下方法：**
- getBondState()  //获取设备绑定状态，BOND_NONE(未绑定), BOND_BONDING(正在绑定), BOND_BONDED(已绑定)
- getType()       //获取蓝牙类型，DEVICE_TYPE_CLASSIC(普通类型BR/EDR), DEVICE_TYPE_LE(低功耗类型), DEVICE_TYPE_DUAL(双类型BR/EDR/LE), DEVICE_TYPE_UNKNOWN(未知)
- getAddress()    //蓝牙地址
- getBluetoothClass()   //获取蓝牙类别(BluetoothClass)，如手机、电脑、耳机，注意与蓝牙类型的区别
- getName()       //蓝牙设备名字

**BluetoothClass蓝牙类别说明：**
- 目前有12个大类，定义在BluetoothClass.Device.Major下
BITMASK、MISC、COMPUTER、PHONE、NETWORKING、AUDIO_VIDEO、PERIPHERAL、IMAGING、WEARABLE、TOY、HEALTH、UNCATEGORIZED

- 每个大类有若干个小类别，定义在BluetoothClass.Device下
  如: AUDIO_VIDEO_WEARABLE_HEADSET、AUDIO_VIDEO_HANDSFREE、AUDIO_VIDEO_RESERVED、AUDIO_VIDEO_MICROPHONE、AUDIO_VIDEO_LOUDSPEAKE

## 4. 监听蓝牙设备的变化
```java
// 监听蓝牙设备的变化
IntentFilter filter = new IntentFilter();
filter.addAction(BluetoothDevice.ACTION_FOUND);      //发现新设备
filter.addAction(BluetoothDevice.ACTION_BOND_STATE_CHANGED);   //绑定状态改变
filter.addAction(BluetoothAdapter.ACTION_DISCOVERY_STARTED);   //开始扫描
filter.addAction(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);  //结束扫描
filter.addAction(BluetoothAdapter.ACTION_CONNECTION_STATE_CHANGED);   //连接状态改变
filter.addAction(BluetoothAdapter.ACTION_STATE_CHANGED);       //蓝牙开关状态改变
BluetoothReceiver receiver = new BluetoothReceiver();
mActivity.registerReceiver(receiver, filter);

...

private class BluetoothReceiver extends BroadcastReceiver {
  @Override
  public void onReceive(Context context, Intent intent) {
    switch(intent.getAction()) {
      case BluetoothAdapter.ACTION_FOUND:
        //获取扫描到的设备
        BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
        //判断扫描到的设备是不是在已绑定设备列表中，在就跳过
        for (BluetoothDevice d: bondedDevices) {
          if (d.getAddress().equals(device.getAddress())) {
            return;
          }
        }
        mScannedDevices.add(device);
        break;

      case BluetoothAdapter.ACTION_DISCOVERY_STARTED:
        //开始扫描
        break;

      case BluetoothAdapter.ACTION_DISCOVERY_FINISHED:
        //结束扫描
        break;

      case BluetoothAdapter.ACTION_CONNECTION_STATE_CHANGED:
        //绑定状态改变
        break;

      case BluetoothAdapter.ACTION_STATE_CHANGED:
        //蓝牙开关状态改变
        int state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, BluetoothAdapter.ERROR);
        switch (state) {
          case BluetoothAdapter.STATE_OFF:
          break;
          case BluetoothAdapter.STATE_ON:
          break;
        }
        break;
    }
  }
}
```

## 5. 绑定蓝牙设备

绑定比较简单，调用 BluetoothDevice的createBond就一句代码，调用后就开始尝试绑定。
```java
private boolean createBond(BluetoothDevice device) {
  return device.createBond();
}
```
绑定成功后会触发刚注册的蓝牙监听器，action为 ACTION_BOND_STATE_CHANGED

## 6. 取消绑定

BluetoothDevice绑定的方法是开放的，取消绑定的方法却是隐藏的，只对系统app开放，只能用反射来解决了。
```java
private boolean removeBond(BluetoothDevice device) {
  Class btDeviceCls = BluetoothDevice.class;
  Method removeBond = btDeviceCls.getMethod("removeBond");
  removeBond.setAccessible(true);
  return (boolean) removeBond.invoke(device);
}
```

## 7. 连接

绑定(配对)和连接是两个不同的过程，绑定是指两个设备发现了对方的存在，可以获取到对方的名称、地址等信息，有能力建立起连接；连接是指两个设备共享了一个RFCOMM通道，有能力进行数据互传。确认绑定上了之后，才能开始连接。可以试试蓝牙音箱的连接过程，就是先点击一次，开始配对，配对成功后出现在已绑定的列表中，再点击一次，就开始连接，连接成功后蓝牙音箱就有声音了。 
网上搜到的都是下面的这种连接方式
```java
UUID MY_UUID = UUID.fromString("00001101-0000-1000-8000-00805F9B34FB");
private void connect(BluetoothDevice device) {
  BluetoothSocket socket = device.createInsecureRfcommSocketToServiceRecord(MY_UUID);
  socket.connect();
  ...

  socket.close();
}
```
这种连接是用蓝牙来进行Socket通信的，而我要做的是如何让手机连上蓝牙耳机或蓝牙音箱，这种方式是不行的。 
最后找到了下面的连接方法，专门针对AUDIO、VIDEO类型的蓝牙设备的连接。
```java
 if (device.getBluetoothClass().getMajorDeviceClass() != BluetoothClass.Device.Major.AUDIO_VIDEO) {
     return;
 }
 btdapter.getProfileProxy(context, new BluetoothProfile.ServiceListener() {
     @Override
     public void onServiceConnected(int profile, BluetoothProfile proxy) {
         BluetoothHeadset bluetoothHeadset = (BluetoothHeadset) proxy;
         Class btHeadsetCls = BluetoothHeadset.class;
         try {
             Method connect = btHeadsetCls.getMethod("connect", BluetoothDevice.class);
             connect.setAccessible(true);
             connect.invoke(bluetoothHeadset, device);
         } catch (Exception e) {
             Log.e(TAG, e + "");
         }
     }

     @Override
     public void onServiceDisconnected(int profile) {

     }
 }, BluetoothProfile.HEADSET);
```
