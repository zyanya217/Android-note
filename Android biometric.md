---
tags : Android,Biometric
title : Android biometric
---
# Biometric
## FingerprintManager

#### 1. 加入 FingerPrint 權限
(在 AndroidManifest.xml 中加入)
```shell=
<?xml version="1.0" encoding="utf-8"?> //XML的UTF-8編碼方式
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.xylon.fingerprintsample" >
 
    <uses-permission android:name="android.permission.USE_FINGERPRINT" />
    //AndroidManifest許可權宣告
 
    <application>
     ...
    </application>
</manifest>
```

#### 2. 確認硬體裝置是否支援以及環境設定是否完成。
isHardwareDetected() 判斷是否有硬體支援 : 
```shell=
 if (!mManager.isHardwareDetected()) {
       Toast.makeText(mContext, "沒有指紋識別模組", Toast.LENGTH_SHORT).show();
       return false;
     }
```

hasEnrolledFingerprints() 判斷是否錄入有指紋 : 
```shell=
if (!mManager.hasEnrolledFingerprints()) {
    Toast.makeText(mContext, "沒有指紋錄入", Toast.LENGTH_SHORT).show();
    return false;
}
```
建立指紋開啟的回調方法->建立AuthenticationCallback物件 : 
```shell=
 FingerprintManager.AuthenticationCallback mSelfCancelled = new FingerprintManager.AuthenticationCallback() {
    @Override
    public void onAuthenticationError(int errorCode, CharSequence errString) {
        //多次指紋密碼驗證錯誤後，進入此方法；並且，不可再驗（短時間）
        //errorCode是失敗的次數
        ToastUtils.show(mContext, "嘗試次數過多，請稍後重試", 3000);
    }

    @Override
    public void onAuthenticationHelp(int helpCode, CharSequence helpString) {
        //指紋驗證失敗，可再驗，可能手指過髒，或者移動過快等原因。
    }

    @Override
    public void onAuthenticationSucceeded(FingerprintManager.AuthenticationResult result) {
        //指紋密碼驗證成功
    }

    @Override
    public void onAuthenticationFailed() {
        //指紋驗證失敗，指紋識別失敗，可再驗，錯誤原因為：該指紋不是系統錄入的指紋。
    }
};
```

#### 3. 執行 Authenticate。
authenticate(...) 啟動指紋識別
```shell=
mManager.authenticate(null, mCancellationSignal, 0, mSelfCancelled, null);
```
:::info
Ref:
- [顯示生物識別身份驗證對話框](https://developer.android.com/training/sign-in/biometric-auth?hl=zh-cn)
- [指紋辨識功能的基本用法](https://xnfood.com.tw/android-fingerprintmanager-api/#step01)
- [利用Android-FingerprintManager類實現指紋識別](https://www.itread01.com/content/1544417828.html)
- [How to implement fingerprint biometric authentication in your Android App? (有影片) ](https://programmerworld.co/android/how-to-implement-fingerprint-biometric-authentication-in-your-android-app-complete-source-code/) 
:::

## Face id 

#### 1. 將第三方jar包加入Android studio專案
#### 2. 呼叫jar包裡的AipOcr函式進行人臉識別
```shell=
 public class AipOcr extends BaseClient { 
    public AipOcr(String appId, String aipKey, String aipToken)
    //呼叫引數有APPID、apiKey、secretKey
    { 
    super(appId, aipKey, aipToken); 
    } 
```
#### 3. 獲取圖片
* 從相簿中選擇
```shell=
getImage=(Button)findViewById(R.id.getImage);
        getImage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent in=new Intent(Intent.ACTION_PICK);
                in.setType("image/*");
                startActivityForResult(in,PHOTO_ALBUM);
            }
        });
```
* 呼叫相機拍攝
先獲取呼叫許可權:
```shell=
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
```shell=
void readRequest(){
        if(checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},1);
        }
        if(checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},2);
        }
    }
```
```shell=
Camera.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                File outputImage=new File(Environment.getExternalStorageDirectory()+File.separator+"face.jpg");
                try{
                    if(outputImage.exists()){
                        outputImage.delete();
                    }
                    outputImage.createNewFile();
                }catch (IOException e){
                    e.printStackTrace();
                }
                imageUri=Uri.fromFile(outputImage);
                ImagePath=outputImage.getAbsolutePath();//拍攝後圖片的儲存路徑
                Intent intent=new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri);
                startActivityForResult(intent,CAMERA);
            }
        });
```
```shell=
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //從相簿中選擇的圖片
        if(requestCode==PHOTO_ALBUM){
            if(data!=null){
                Uri uri=data.getData();
                Cursor cursor=getContentResolver().query(uri,null,null,null,null);
                cursor.moveToNext();
                ImagePath=cursor.getString(cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA));
                cursor.close();
                resizePhoto();
                myPhoto.setImageBitmap(myBitmapImage);
                Log.i("圖片路徑",ImagePath);
            }
        }
        //相機拍攝的圖片
        else if(requestCode==CAMERA){
            try{
                resizePhoto();
                myPhoto.setImageBitmap(myBitmapImage);
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
    //調整圖片的比例，使其大小小於1M,能夠顯示在手機螢幕上
    public void resizePhoto(){
        BitmapFactory.Options options=new BitmapFactory.Options();
        options.inJustDecodeBounds=true;//返回圖片寬高資訊
        BitmapFactory.decodeFile(ImagePath,options);
        //讓圖片小於1024
        double radio=Math.max(options.outWidth*1.0d/1024f,options.outHeight*1.0d/1024f);
        options.inSampleSize=(int)Math.ceil(radio);//向上取整倍數
        options.inJustDecodeBounds=false;//顯示圖片
        myBitmapImage=BitmapFactory.decodeFile(ImagePath,options);
    }
```

#### 4. 呼叫函式進行人臉識別
```shell=
detect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                res=null;
                detect_tip.setVisibility(View.VISIBLE);
                detect_tip.setText("識別中...");
               if(myBitmapImage==null){
                   myBitmapImage=BitmapFactory.decodeResource(getResources(),R.mipmap.face2);
                   bitmapSmall=Bitmap.createBitmap(myBitmapImage,0,0,myBitmapImage.getWidth(),myBitmapImage.getHeight());
               }
               else{//由於有些圖片在一些型號手機角度會傾斜，需要進行擺正
                   int degree=getPicRotate(ImagePath);
                   Matrix m=new Matrix();
                   m.setRotate(degree);
                   bitmapSmall=Bitmap.createBitmap(myBitmapImage,0,0,myBitmapImage.getWidth(),myBitmapImage.getHeight(),m,true);
               }
               //將圖片由路徑轉為二進位制資料流
                ByteArrayOutputStream stream=new ByteArrayOutputStream();
                //圖片轉資料流
                bitmapSmall.compress(Bitmap.CompressFormat.JPEG,100,stream);
                final byte[] arrays=stream.toByteArray();
                //網路申請呼叫函式進行人臉識別
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        HashMap<String,String> options=new HashMap<>();
                        options.put("face_fields","age,gender,race,beauty,expression");//人臉屬性：年齡，性別，膚色，顏值，笑容
                        AipFace client=new AipFace("10734368","6cvleSFbyRIRHzhijfYrHZFj","SDnCUfrtH0lgrK01HgTe2ZRLNsmCx5xy");
                        client.setConnectionTimeoutInMillis(2000);
                        client.setSocketTimeoutInMillis(6000);
                        res=client.detect(arrays,options);
                        try{
                            Message message = Message.obtain();
                            message.what = 1;
                            message.obj = res;
                            handler.sendMessage(message);
                        }catch (Exception e){
                            e.printStackTrace();
                            Message message = Message.obtain();
                            message.what = 2;
                            handler.sendMessage(message);
                        }
                    }
                }).start();
            }
        });
    }
```

#### 5. 將返回來的網路資料進行處理
```shell=private Handler handler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if(msg.what==1){
                JSONObject res=(JSONObject) msg.obj;
                face_resultNum=res.optString("result_num");
                if(Integer.parseInt(face_resultNum)>=1) {
                    try {
                        JSONArray js = new JSONArray(res.optString("result"));
                        face_age = js.optJSONObject(0).optString("age");
                        face_gender = js.optJSONObject(0).optString("gender");
                        if (face_gender.equals("female")) {
                            face_gender = "女";
                        } else {
                            face_gender = "男";
                        }
                        face_race = js.optJSONObject(0).optString("race");
                        if (face_race.equals("yellow")) {
                            face_race = "黃種人";
                        } else if (face_race.equals("white")) {
                            face_race = "白種人";
                        } else if (face_race.equals("black")) {
                            face_race = "黑種人";
                        }else if(face_race.equals("arabs")){
                            face_race = "阿拉伯人";
                        }
                        int express = Integer.parseInt(js.optJSONObject(0).optString("expression"));
                        if (express == 0) {
                            face_expression = "無";
                        } else if (express == 1) {
                            face_expression = "微笑";
                        } else {
                            face_expression = "大笑";
                        }
                        face_beauty = js.optJSONObject(0).optString("beauty");
                        double  beauty=Math.ceil(Double.parseDouble(face_beauty)+25);
                        if(beauty>=100){//對獲得的顏值資料進行一定處理，使得結果更為合理
                            beauty=99.0;
                        }
                        else if(beauty<70){
                            beauty+=10;
                        }
                        else if(beauty>80 && beauty<90){
                            beauty+=5;
                        }
                        else if(beauty>=90 && beauty<95){
                            beauty+=2;
                        }
                        face_beauty=String.valueOf(beauty);
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                    detect_tip.setVisibility(View.GONE);
                    AlertDialog.Builder alertDialog = new AlertDialog.Builder(faceRecognition.this);
                    String[] mItems = {"性別：" + face_gender, "年齡：" + face_age, "膚色：" + face_race, "顏值：" + face_beauty, "笑容：" + face_expression};
                    alertDialog.setTitle("人臉識別報告").setItems(mItems, null).create().show();
                }else{
                    detect_tip.setVisibility(View.VISIBLE);
                    detect_tip.setText("圖片不夠清晰，請重新選擇");
                }
            }else{
                detect_tip.setVisibility(View.VISIBLE);
                detect_tip.setText("圖片不夠清晰，請重新選擇");
            }
        }
    };
private Handler handler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if(msg.what==1){
                JSONObject res=(JSONObject) msg.obj;
                face_resultNum=res.optString("result_num");
                if(Integer.parseInt(face_resultNum)>=1) {
                    try {
                        JSONArray js = new JSONArray(res.optString("result"));
                        face_age = js.optJSONObject(0).optString("age");
                        face_gender = js.optJSONObject(0).optString("gender");
                        if (face_gender.equals("female")) {
                            face_gender = "女";
                        } else {
                            face_gender = "男";
                        }
                        face_race = js.optJSONObject(0).optString("race");
                        if (face_race.equals("yellow")) {
                            face_race = "黃種人";
                        } else if (face_race.equals("white")) {
                            face_race = "白種人";
                        } else if (face_race.equals("black")) {
                            face_race = "黑種人";
                        }else if(face_race.equals("arabs")){
                            face_race = "阿拉伯人";
                        }
                        int express = Integer.parseInt(js.optJSONObject(0).optString("expression"));
                        if (express == 0) {
                            face_expression = "無";
                        } else if (express == 1) {
                            face_expression = "微笑";
                        } else {
                            face_expression = "大笑";
                        }
                        face_beauty = js.optJSONObject(0).optString("beauty");
                        double  beauty=Math.ceil(Double.parseDouble(face_beauty)+25);
                        if(beauty>=100){//對獲得的顏值資料進行一定處理，使得結果更為合理
                            beauty=99.0;
                        }
                        else if(beauty<70){
                            beauty+=10;
                        }
                        else if(beauty>80 && beauty<90){
                            beauty+=5;
                        }
                        else if(beauty>=90 && beauty<95){
                            beauty+=2;
                        }
                        face_beauty=String.valueOf(beauty);
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                    detect_tip.setVisibility(View.GONE);
                    AlertDialog.Builder alertDialog = new AlertDialog.Builder(faceRecognition.this);
                    String[] mItems = {"性別：" + face_gender, "年齡：" + face_age, "膚色：" + face_race, "顏值：" + face_beauty, "笑容：" + face_expression};
                    alertDialog.setTitle("人臉識別報告").setItems(mItems, null).create().show();
                }else{
                    detect_tip.setVisibility(View.VISIBLE);
                    detect_tip.setText("圖片不夠清晰，請重新選擇");
                }
            }else{
                detect_tip.setVisibility(View.VISIBLE);
                detect_tip.setText("圖片不夠清晰，請重新選擇");
            }
        }
    };
```
:::info
Ref:
- [Android開發實現人臉識別](https://www.itread01.com/content/1549195586.html)
- [Android 人臉偵測](https://nickcarter9.github.io/2018/07/09/2018/2018_07_09-facedetect/)
:::


