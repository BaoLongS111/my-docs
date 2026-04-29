## 一、示例图

1.未使用ViewModelSavedState的生命周期示例图。

![image-20260401094021761](..\imgs\image-20260401094021761.png)

2.使用ViewModelSavedState的生命周期示例图。

当短暂切出其他应用的时候，ViewModel的数据会恢复。但当长时间切出去，系统会无声的杀死你的应用，这时候数据就会清零，但如果使用了ViewModelSavedState，就会恢复数据。

![image-20260401120820402](..\imgs\image-20260401120820402.png)

## 二、onSaveInstanceState方法

```java
package com.example.helloworld;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

public class MyViewModel extends ViewModel {
    private MutableLiveData<Integer> nums;

    public MutableLiveData<Integer> getNums(){
        if(nums==null){
            nums = new MutableLiveData<>();
            nums.setValue(0);
        }
        return nums;
    }

    public void Add(){
        nums.setValue(nums.getValue()+1);
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
import androidx.lifecycle.ViewModelProvider;

import com.example.helloworld.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    ActivityMainBinding binding;
    public MyViewModel myViewModel;

    public final static String KEY_NUMBER = "my_number";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        binding = DataBindingUtil.setContentView(this,R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });
        ViewModelProvider.AndroidViewModelFactory instance = ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());
        myViewModel = new ViewModelProvider(this,instance).get(MyViewModel.class);

        if(savedInstanceState!=null){
            myViewModel.getNums().setValue(savedInstanceState.getInt(KEY_NUMBER));
        }
        binding.setData(myViewModel);
        binding.setLifecycleOwner(this);
    }

    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        outState.putInt(KEY_NUMBER,myViewModel.getNums().getValue());
    }
}

```

## 三、ViewModelState方法

1.改写MyViewModel类，新增SavedStateHandle变量，重写MyViewModel构造方法，控制器使用的时候要传handle属性。

```java
package com.example.helloworld;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.SavedStateHandle;
import androidx.lifecycle.ViewModel;

public class MyViewModel extends ViewModel {
    private SavedStateHandle handle;

    public MyViewModel(SavedStateHandle handle){
        this.handle = handle;
    }
    public MutableLiveData<Integer> getNums(){
        if(!handle.contains(MainActivity.KEY_NUMBER)){
            handle.set(MainActivity.KEY_NUMBER,0);
        }
        return handle.getLiveData(MainActivity.KEY_NUMBER);
    }

    public void Add(){
        getNums().setValue(getNums().getValue()+1);
    }
}

```

2.MainActivity类

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

    ActivityMainBinding binding;
    public MyViewModel myViewModel;

    public final static String KEY_NUMBER = "my_number";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        binding = DataBindingUtil.setContentView(this,R.layout.activity_main);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });
        myViewModel = new ViewModelProvider(this).get(MyViewModel.class);

        binding.setData(myViewModel);
        binding.setLifecycleOwner(this);
    }

}

```

