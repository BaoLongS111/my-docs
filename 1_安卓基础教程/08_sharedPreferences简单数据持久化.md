### 1.getApplicationContext()：这个指向顶级App的引用，不会引起内存泄露的问题。

### 什么是内存泄漏呢，就是比如你写了一个MyData数据操作类，然后这个类要传入context，你在某一个Activity里使用了这个类，传了个this进去，这样就会引起内存泄漏的问题。因为Activity在程序的运行过程中会频繁的创建和摧毁，你把this传进去，MyData就会持有这个引用，那即使用不到这个Activity里，系统也不会摧毁它，就相当于内存一直在耗费资源保存一个没有用的东西。

sp现在已经被DataStore给替代了

### 2.创建MyData类，里面写save和load函数

```java
package com.example.helloworld;

import android.content.Context;
import android.content.SharedPreferences;

public class MyData {
    private Context context;
    private int number;
    public MyData(Context context){
        this.context = context;
    }

    public void save(){
        String dataName = context.getResources().getString(R.string.my_data);
        String keyName = context.getResources().getString(R.string.my_key);
        SharedPreferences sp = context.getSharedPreferences(dataName,Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sp.edit();
        editor.putInt(keyName,number);
        editor.apply();
    }

    public int load(){
        String dataName = context.getResources().getString(R.string.my_data);
        String keyName = context.getResources().getString(R.string.my_key);
        SharedPreferences sp = context.getSharedPreferences(dataName,Context.MODE_PRIVATE);
        number = sp.getInt(keyName,context.getResources().getInteger(R.integer.defValue));
        return number;
    }

    public void setNumber(int number){
        this.number = number;
    }

    public int getNumber(){
        return this.number;
    }


}
```

```java
package com.example.helloworld;

import android.os.Bundle;
import android.os.PersistableBundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.databinding.DataBindingUtil;
import androidx.lifecycle.Observer;
import androidx.lifecycle.SavedStateViewModelFactory;
import androidx.lifecycle.ViewModelProvider;

import com.example.helloworld.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    public final static String TAG = "MY_TAG";
    MyData myData;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });

        myData = new MyData(getApplicationContext());
        myData.setNumber(100);
        myData.save();
        int n = myData.load();
        Log.d(TAG, "onCreate: "+n);


    }

}
```

