###  1.LiveData也是JetPack 2018年推出的，和ViewModel以及DataBinding差不多。

1.1创建一个MyViewModel类，继承ViewModel，在里面写一个LiveData的数据。

```java
public class MyViewModel extends ViewModel{
    private MutableLiveData<Integer> likedNumber;
    public MutableLiveData<Integer> getLikdedNumber(){
        if(likedNumber==null){
            likedNumber = new MutableLiveData<>();
            likedNumber.setValue(0);
        }
        return likedNumber;
    }
    
    public void Add1(){
        this.likedNumber.setValue(this.likedNumber.getLikedNumber()+1);
    }
    
    public void Minus1(){
        this.likedNumber.setValue(this.likedNumber.getLikedNumber()-1);
    }
}
```

1.2在Controller类，比如Activity里使用这个ViewModel，实现它的观察者模式，数据一但变化就会自动刷新界面。

```java
MyViewModel myViewModel;
ViewModelProvider.AndroidViewModelFactory instance = ViewModelPorovider.AndroidViewModelFactory.getInstance(getApplication());
myViewModel = new ViewModelProvider(this,instance).get(MyViewModel.class);

myView.getLikedNumber().observe(this,new Observer<Integer>(){
    @Override
    public void OnChanged(Integer integer){
        textView.setText(String.valueOf(integer));
    } 
})
```



![image-20260330145749089](..\imgs\image-20260330145749089.png)

![image-20260330152710495](..\imgs\image-20260330152710495.png)

### 2.DataBinding

1.在build.gradle里android的标签下，创建一个buildFetures，然后写上dataBinding{enable=true}，再Sync一下。

2.在布局的左上角点一下小灯泡，转换成data binding layout。

3.在布局文件的data标签里写上数据名和类型，然后在要显示的textView上使用@{data.getNumber()}这种。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="data"
            type="com.example.helloworld.MyViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/main"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/glv50"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.5" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/glh50"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_percent="0.50" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/gl25"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_percent="0.25" />

        <TextView
            android:id="@+id/tv_dispaly"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(data.getLikedNumber())}"
            android:textSize="30sp"
            app:layout_constraintBottom_toBottomOf="@id/gl25"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <Button
            android:id="@+id/btnAdd1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="+1"
            android:onClick="@{()->data.Add1()}"
            app:layout_constraintBottom_toBottomOf="@id/glh50"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="@id/gl25" />

        <Button
            android:id="@+id/btnMinus1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="-1"
            android:onClick="@{()->data.Minus1()}"
            android:layout_marginTop="40dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="@id/btnAdd1"
            app:layout_constraintBottom_toBottomOf="@id/glh50" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
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

    MyViewModel myViewModel;
    ActivityMainBinding binding;

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
        ViewModelProvider.AndroidViewModelFactory instance = 		ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());
        myViewModel = new ViewModelProvider(this,instance).get(MyViewModel.class);
        binding.setData(myViewModel);
        binding.setLifecycleOwner(this);

    }
    
}

```

