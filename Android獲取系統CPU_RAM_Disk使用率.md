---
tags : Android
title : Android獲取系統CPU/RAM/Disk使用率

---
# 獲取系統CPU/RAM/Disk的使用率
## CPU
```shell=
/**獲取當前CPU佔比
 * 在實際測試中發現，有的手機會隱藏CPU狀態，不會完全顯示所有CPU資訊，例如MX5，所有建議只做參考
 * @return
 */
public static String getCPURateDesc(){
    String path = "/proc/stat";// 系統CPU資訊檔案
    long totalJiffies[]=new long[2];
    long totalIdle[]=new long[2];
    int firstCPUNum=0;//設定這個引數，這要是防止兩次讀取檔案獲知的CPU數量不同，導致不能計算。這裡統一以第一次的CPU數量為基準
    FileReader fileReader = null;
    BufferedReader bufferedReader = null;
    Pattern pattern=Pattern.compile(" [0-9]+");
    for(int i=0;i<2;i++) {
        totalJiffies[i]=0;
        totalIdle[i]=0;
        try {
            fileReader = new FileReader(path);
            bufferedReader = new BufferedReader(fileReader, 8192);
            int currentCPUNum=0;
            String str;
            while ((str = bufferedReader.readLine()) != null&&(i==0||currentCPUNum<firstCPUNum)) {
                if (str.toLowerCase().startsWith("cpu")) {
                    currentCPUNum++;
                    int index = 0;
                    Matcher matcher = pattern.matcher(str);
                    while (matcher.find()) {
                        try {
                            long tempJiffies = Long.parseLong(matcher.group(0).trim());
                            totalJiffies[i] += tempJiffies;
                            if (index == 3) {//空閒時間為該行第4條欄目
                                totalIdle[i] += tempJiffies;
                            }
                            index++;
                        } catch (NumberFormatException e) {
                            e.printStackTrace();
                        }
                    }
                }
                if(i==0){
                    firstCPUNum=currentCPUNum;
                    try {//暫停50毫秒，等待系統更新資訊。
                        Thread.sleep(50);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } finally {
            if (bufferedReader != null) {
                try {
                    bufferedReader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    double rate=-1;
    if (totalJiffies[0]>0&&totalJiffies[1]>0&&totalJiffies[0]!=totalJiffies[1]){
        rate=1.0*((totalJiffies[1]-totalIdle[1])-(totalJiffies[0]-totalIdle[0]))/(totalJiffies[1]-totalJiffies[0]);
    }

    return String.format("cpu:%.2f",rate);
}
```
:::info
Ref:
- [Android獲取應用cpu使用率](https://www.aiwalls.com/android%E8%BB%9F%E9%AB%94%E9%96%8B%E7%99%BC%E6%95%99%E5%AD%B8/24/17529.html)
- [Android系统获取系统 CPU 使用率](https://blog.csdn.net/su749520/article/details/79218952)
- [Android系統CPU使用率獲取（附java程式碼）](https://www.itread01.com/content/1549532008.html)
:::
## RAM & ROM
Layout的部分:
```shell=
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <TextView
        android:id="@+id/textView1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView" />

    <TextView
        android:id="@+id/textView2"
        android:layout_marginTop="10dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="TextView" />
    
    
    

</LinearLayout>
```
```shell=
public class MainActivity extends Activity{
	private TextView tv1;
	private TextView tv2;


	@TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR2)
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_memory_test);
		tv1 = (TextView) findViewById(R.id.textView1);
		tv2 = (TextView) findViewById(R.id.textView2);
		
		//獲取執行記憶體的資訊
		ActivityManager manager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);  
        MemoryInfo info = new MemoryInfo();  
        manager.getMemoryInfo(info);  
        StringBuilder sb = new StringBuilder();
        sb.append("可用RAM:");
        sb.append(info.availMem + "B");
        sb.append(",總RAM:");
        sb.append(info.totalMem + "B");
        sb.append("\r\n");
        sb.append(Formatter.formatFileSize(getBaseContext(), info.availMem));
        sb.append(",");
        LogUtil.print("totalMem:" + info.totalMem);
        sb.append(Formatter.formatFileSize(getBaseContext(), info.totalMem));
        tv1.setText(sb);
        
        
        
        sb.setLength(0);
        //獲取ROM記憶體資訊
        //呼叫該類來獲取磁碟資訊（而getDataDirectory就是內部儲存）
        final StatFs statFs = new StatFs(Environment.getDataDirectory().getPath());
        long totalCounts = statFs.getBlockCountLong();//總共的block數
        long availableCounts = statFs.getAvailableBlocksLong() ; //獲取可用的block數
        long size = statFs.getBlockSizeLong(); //每格所佔的大小，一般是4KB==
        long availROMSize = availableCounts * size;//可用內部儲存大小
        long totalROMSize = totalCounts *size; //內部儲存總大小
        sb.append("可用block數:" + availableCounts);
        sb.append("block總數:" + totalCounts);
        sb.append("\r\n");
        sb.append(" 每個block大小:" + size);
        sb.append("\r\n");
        sb.append(" 可用ROM:" + availROMSize + "B");
        sb.append(" 總ROM:" + totalROMSize + "B");
        tv2.setText(sb);
              
	}	
}
```
:::info
Ref:
- [android如何获取RAM和ROM使用情况](https://blog.csdn.net/qq_32072451/article/details/105662099)
- [Android 獲取記憶體資訊（RAM,ROM）](https://www.itread01.com/content/1550008804.html)
:::
## Disk
```shell=
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.lang.reflect.Field;
import java.util.List;
import android.app.ActivityManager;
import android.app.ActivityManager.RunningTaskInfo;
import android.content.Context;
public class DeviceInfoManager {
// private static final String TAG = "DeviceInfoManager";
private static ActivityManager mActivityManager;
public synchronized static ActivityManager getActivityManager(Context context) {
if (mActivityManager == null) {
mActivityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
}
return mActivityManager;
}
/**
* 用於獲取狀態列的高度。
*
* @return 返回狀態列高度的畫素值。
*/
public static int getStatusBarHeight(Context context) {
int statusBarHeight = 0;
try {
Class<?> c = Class.forName("com.android.internal.R$dimen");
Object o = c.newInstance();
Field field = c.getField("status_bar_height");
int x = (Integer) field.get(o);
statusBarHeight = context.getResources().getDimensionPixelSize(x);
} catch (Exception e) {
e.printStackTrace();
}
return statusBarHeight;
}
/**
* 計算已使用記憶體的百分比，並返回。
*
* @param context
*      可傳入應用程式上下文。
* @return 已使用記憶體的百分比，以字串形式返回。
*/
public static String getUsedPercentValue(Context context) {
long totalMemorySize = getTotalMemory();
long availableSize = getAvailableMemory(context) / 1024;
int percent = (int) ((totalMemorySize - availableSize) / (float) totalMemorySize * 100);
return percent   "%";
}
/**
* 獲取當前可用記憶體，返回資料以位元組為單位。
*
* @param context 可傳入應用程式上下文。
* @return 當前可用記憶體。
*/
public static long getAvailableMemory(Context context) {
ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();
getActivityManager(context).getMemoryInfo(mi);
return mi.availMem;
}
/**
* 獲取系統總記憶體,返回位元組單位為KB
* @return 系統總記憶體
*/
public static long getTotalMemory() {
long totalMemorySize = 0;
String dir = "/proc/meminfo";
try {
FileReader fr = new FileReader(dir);
BufferedReader br = new BufferedReader(fr, 2048);
String memoryLine = br.readLine();
String subMemoryLine = memoryLine.substring(memoryLine.indexOf("MemTotal:"));
br.close();
//將非數字的字元替換為空
totalMemorySize = Integer.parseInt(subMemoryLine.replaceAll("\\D ", ""));
} catch (IOException e) {
e.printStackTrace();
}
return totalMemorySize;
}
```

:::info
Ref:
- [Android程式設計實現獲取系統記憶體、CPU使用率及狀態列高度的方法示例](https://codertw.com/android-%E9%96%8B%E7%99%BC/336392/)
:::
