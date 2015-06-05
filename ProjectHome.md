# No more write code to check status and enable GPS, NETWORK(WiFi, 3G) and BLUETOOTH #

> Very reuseful approach! Just configure **`onResume`** for each activity according its needed resources. This approach is going to alert the user to enable resources and provide shortcuts to the resource settings. **The user can do nothing still everything is enabled.** He can finish current activity pressing **Exit** button as well.

**Of course, your `onCreate` can not have code accessing those resources. Put that code inside `pass`. See below.**

```
@Override protected void onResume() {
  super.onResume();
  new Checker(this).pass(new Checker.Pass() {
     @Override public void pass() {
        //do your stuff here, do nothing outside here
     }
  }).check(Resource.GPS, Resource.NETWORK, Resource.BLUETOOTH);
}
```

AndroidManifest.xml
```
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.BLUETOOTH" />
```

![http://i.minus.com/i26ABUheYskR8.jpg](http://i.minus.com/i26ABUheYskR8.jpg)
![http://i.minus.com/ik3GLUpbA89GM.jpg](http://i.minus.com/ik3GLUpbA89GM.jpg)
![http://i.minus.com/iuMOrC5ZBkpfE.jpg](http://i.minus.com/iuMOrC5ZBkpfE.jpg)

```
import java.util.Arrays;
import java.util.List;

import android.app.Activity;
import android.app.AlertDialog;
import android.bluetooth.BluetoothAdapter;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.location.LocationManager;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.provider.Settings;

public class Checker {

	/* FIELDS */

	private Activity activity;
	private Pass pass;
	private List<Resource> resourcesList;

	/* TYPES */

	public enum Resource {
		NETWORK, GPS, BLUETOOTH
	}

	public static abstract class Pass {
		public abstract void pass();
	}

	/* API */

	public Checker(Activity activity) {
		this.activity = activity;
	}

	public void check(Resource... resources) {
		resourcesList = Arrays.asList(resources);
		if (resourcesList.contains(Resource.GPS) && !isGPSActivated(activity)) {
			new AlertDialog.Builder(activity).setMessage("GPS required.").setCancelable(false).setPositiveButton("GPS", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					activity.startActivity(new Intent(Settings.ACTION_LOCATION_SOURCE_SETTINGS));
				}
			}).setNegativeButton("Exit", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					dialog.cancel();
					activity.finish();
				}
			}).create().show();
		} else if (resourcesList.contains(Resource.NETWORK) && !isNetworkActivated(activity)) {
			new AlertDialog.Builder(activity).setMessage("Network required.").setCancelable(false).setPositiveButton("3G", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					Intent intent = new Intent(Intent.ACTION_MAIN);
					intent.setClassName("com.android.phone", "com.android.phone.Settings");
					activity.startActivity(intent);
				}
			}).setNeutralButton("WiFi", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					activity.startActivity(new Intent(Settings.ACTION_WIFI_SETTINGS));
				}
			}).setNegativeButton("Exit", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					dialog.cancel();
					activity.finish();
				}
			}).create().show();
		} else if (resourcesList.contains(Resource.BLUETOOTH) && !isBluetoothActivated(activity)) {
			new AlertDialog.Builder(activity).setMessage("Bluetooth required.").setCancelable(false).setPositiveButton("Bluetooth",
					new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog, int id) {
							activity.startActivity(new Intent(Settings.ACTION_BLUETOOTH_SETTINGS));
						}
					}).setNegativeButton("Exit", new DialogInterface.OnClickListener() {
				public void onClick(DialogInterface dialog, int id) {
					dialog.cancel();
					activity.finish();
				}
			}).create().show();
		}else{
		   pass.pass();
		}
	}

	public Checker pass(Pass pass) {
		this.pass = pass;
		return this;
	}

	/* PRIVATE */

	private boolean isGPSActivated(Context context) {
		return ((LocationManager) context.getSystemService(Context.LOCATION_SERVICE)).isProviderEnabled(LocationManager.GPS_PROVIDER);
	}

	private boolean isBluetoothActivated(Context context) {
		return BluetoothAdapter.getDefaultAdapter().isEnabled();
	}

	private boolean isNetworkActivated(Context context) {
		ConnectivityManager connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
		NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
		return networkInfo != null && networkInfo.isConnected();
	}

}

```