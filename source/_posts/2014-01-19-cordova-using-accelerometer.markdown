---
layout: post
title: "Cordova探索之旅系列（三）"
date: 2014-01-19 15:39
comments: true
categories: 
---
自从3.0之后，Cordova默认是关闭所有关于设备原生特性功能的，所以我们要通过添加插件来启动原生特性。

这里以Accelerometer（加速度感应器）为例，来学习如何使用设备原生特性。

**1.添加插件**

首先，需要在工程目录下，通过CLI命令添加插件。

{% codeblock lang:bash %}
cordova plugin add org.apache.cordova.device-motion
{% endcodeblock %}

通过ls命令，可以查看当前项目下，已经安装的插件。

{% codeblock lang:bash %}
cordova plugin ls
{% endcodeblock %}

**2.在config.xml文件中配置该特性**

路径：res/xml/config.xml

{% codeblock lang:xml %}
<feature name="Accelerometer">
	<param name="android-package" value="org.apache.cordova.devicemotion.AccelListener" />
</feature>
{% endcodeblock %}

完整配置如下：

{% codeblock lang:xml %}
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.example.hello.HelloWorld" version="0.0.1" 
xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
    <name>HelloCordova</name>
    <description>
        A sample Apache Cordova application that responds to the deviceready event.
    </description>
    <author email="dev@cordova.apache.org" href="http://cordova.io">
        Apache Cordova Team
    </author>
    <content src="index.html" />
    <access origin="*" />
    <feature name="Accelerometer">
        <param name="android-package" value="org.apache.cordova.devicemotion.AccelListener" />
    </feature>
</widget>
{% endcodeblock %}

某些插件还需要在Android的AndroidManifest.xml中添加uses-permission

例如：

{% codeblock lang:xml %}
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
{% endcodeblock %}

当然，这里不需要！

**3.API**

{% codeblock lang:javascript %}
navigator.accelerometer.getCurrentAcceleration(onSuccess, onError)
{% endcodeblock %}

onSuccess和onError是对应的回调函数

**4.完整例子**
{% codeblock lang:html %}

<!DOCTYPE html>
<html>
  <head>
    <title>Acceleration Example</title>
    <script type="text/javascript" charset="utf-8" src="cordova.js"></script>
    <script type="text/javascript" charset="utf-8">

    document.addEventListener("deviceready", onDeviceReady, false);

    function onDeviceReady() {
    
    }

    function getCurrrentAcceleration() {
        navigator.accelerometer.getCurrentAcceleration(onSuccess, onError);
    }

    function onSuccess(acceleration) {
        alert('Acceleration X: ' + acceleration.x + '\n' +
              'Acceleration Y: ' + acceleration.y + '\n' +
              'Acceleration Z: ' + acceleration.z + '\n' +
              'Timestamp: '      + acceleration.timestamp + '\n');
    }

    function onError() {
        alert('onError!');
    }
    </script>
  </head>
  <body>
    <button onClick="getCurrrentAcceleration()">click</button>
  </body>
</html>
{% endcodeblock %}

如果用Android的原生API，用Java代码来实现相同功能呢，如下：

Activity

{% codeblock lang:java %}
import android.app.Activity;
import android.hardware.Sensor;
import android.hardware.SensorEvent;
import android.hardware.SensorEventListener;
import android.hardware.SensorManager;
import android.os.Bundle;
import android.view.View;
import me.zeph.shakeshake.R;
import me.zeph.shakeshake.fragment.FireMissilesDialogFragment;

import static android.hardware.Sensor.TYPE_ACCELEROMETER;
import static android.hardware.SensorManager.SENSOR_DELAY_NORMAL;

public class MyActivity extends Activity {

    private SensorManager sensorManager;
    private Sensor sensor;
    private AccelerometerListener listener;
    private String text;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        initSensor();
    }

    private void initSensor() {
        sensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);
        sensor = sensorManager.getDefaultSensor(TYPE_ACCELEROMETER);
        listener = new AccelerometerListener();
        sensorManager.registerListener(listener, sensor, SENSOR_DELAY_NORMAL);
    }

    @Override
    protected void onStop() {
        super.onStop();
        sensorManager.unregisterListener(listener);
    }

    public void sendMessage(View view) {
        FireMissilesDialogFragment fireMissilesDialogFragment = new FireMissilesDialogFragment(text);
        fireMissilesDialogFragment.show(getFragmentManager(), "some tag");
    }

    class AccelerometerListener implements SensorEventListener {

        @Override
        public void onSensorChanged(SensorEvent event) {
            text = "Acceleration X: \n" + event.values[0] + "\n" +
                    "Acceleration Y: \n" + event.values[1] + "\n" +
                    "Acceleration Z: \n" + event.values[2] + "\n" +
                    "Timestamp: \n" + event.timestamp + "\n";
        }

        @Override
        public void onAccuracyChanged(Sensor sensor, int i) {

        }

    }

}
{% endcodeblock %}

Dialog

{% codeblock lang:java %}
import android.app.AlertDialog;
import android.app.Dialog;
import android.app.DialogFragment;
import android.content.DialogInterface;
import android.os.Bundle;
import me.zeph.shakeshake.R;

public class FireMissilesDialogFragment extends DialogFragment {
    private String text;

    public FireMissilesDialogFragment(String text) {
        this.text = text;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder
                .setTitle(R.string.dialog_accelerometer)
                .setMessage(text)
                .setNeutralButton(R.string.ok, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {

                    }
                });
        return builder.create();
    }
}
{% endcodeblock %}

main.xml

{% codeblock lang:xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="fill_parent"
              android:layout_height="fill_parent"
        >
    <Button android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="@string/click"
            android:onClick="sendMessage"
            />
</LinearLayout>
{% endcodeblock %}

