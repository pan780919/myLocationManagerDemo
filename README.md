# myLocationManagerDemo
1、首先需要引入Google Service gcm location SDK，引入這個SDK的原因是：省電。並且防止狀態欄上的GPS標識一直閃爍，谷歌的官方文檔說的很直白,不推薦使用Android原生的接口，原文如下

theGoogle Play services location APIsare preferred over the Android framework location APIs (android.location) as a way of adding location awareness to your app. If you are currently using the Android framework location APIs, you are strongly encouraged to switch to the Google Play services location APIs as soon as possible.
該SDK的官方文檔地址：https://developer.android.com/training/location/index.html

引入該SDK需要在gradle文件裡添加
compile 'com.google.android.gms:play-services-location:8.4.0'

void onCreateGPS(Application context)
用於開啟一次gps位置獲取，程序會先獲取最近一次的GPS位置，如果失敗的話就開啟GPS追蹤和基站，wifi定位等
void restartGPS(Application context)
在進入一些對GPS要求比較高的頁面時，重新獲取當前的准確位置的方法，本方法會先暫停掉當前的GPS追蹤，然後重新獲取一遍位置
void stopGPS()
在退出app時，為了節省電力而徹底關閉GPS追蹤
void pauseGPS()
當程序放到後台時為了防止後台耗電而停止GPS跟蹤

import android.app.Activity;
import android.app.ActivityManager;
import android.content.Context;
import android.os.Bundle;
import android.support.v4.app.Activity;
import android.util.Log;

import com.imaginato.qravedconsumer.service.AlxLocationManager;
import com.imaginato.qravedconsumer.utils.JLogUtils;

import java.util.List;

public class BaseActivity extends Activity {
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);

	}
	@Override
	protected void onStop() {
		super.onStop();

		if (!isAppOnForeground(this)) {
			//app 進入後台
			JLogUtils.i("AlexLocation","程序進入後台運行");
			//全局變量isActive = false 記錄當前已經進入後台
			isRunningBackGround = true;
			AlxLocationManager.pauseGPS();
		}
	}

	@Override
	protected void onRestart() {
		super.onRestart();
		//從後台恢復，如果還沒有很好的獲得經緯度就繼續獲得
		if(isRunningBackGround && AlxLocationManager.manager!=null && AlxLocationManager.manager.currentStatus!= AlxLocationManager.STATUS.TRYING_FIRST)AlxLocationManager.onCreateGPS(getApplication());
		isRunningBackGround = false;
	}

	/**
	 * 程序是否在前台運行
	 *
	 * @return
	 */
	public static boolean isAppOnForeground(Activity activity) {
		// Returns a list of application processes that are running on the
		// device

		ActivityManager activityManager = (ActivityManager) activity.getApplicationContext().getSystemService(Context.ACTIVITY_SERVICE);
		String packageName = activity.getApplicationContext().getPackageName();

		List appProcesses = activityManager
				.getRunningAppProcesses();
		if (appProcesses == null)
			return false;

		for (ActivityManager.RunningAppProcessInfo appProcess : appProcesses) {
			// The name of the process that this object is associated with.
			if (appProcess.processName.equals(packageName)
					&& appProcess.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
				return true;
			}
		}

		return false;
	}

}



