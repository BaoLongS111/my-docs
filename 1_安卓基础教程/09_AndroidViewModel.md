### 1.AndroidViewModel

AndroidViewModel和普通的ViewModel的区别就是他是Andorid系统内置的ViewModel，它天生自带了context上下文，也就是getApplication()，getApplication也是继承自context的，就是说不用我们手动在构造方法里传context上下文了。

### 2.下面依次放MyViewModel,Layout和MainActivity的代码。

```java
package com.example.helloworld;

import android.app.Application;
import android.content.Context;
import android.content.SharedPreferences;

import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.LiveData;
import androidx.lifecycle.SavedStateHandle;

public class MyViewModel extends AndroidViewModel {

    public String key_number = getApplication().getResources().getString(R.string.my_key);
    public String sh_name = getApplication().getResources().getString(R.string.my_data);

    private SavedStateHandle handle;

    public MyViewModel(@NonNull Application application, SavedStateHandle handle) {
        super(application);
        this.handle = handle;
        if(!handle.contains(key_number)){
            load();
        }
    }

    public LiveData<Integer> getNumber(){
        return handle.getLiveData(key_number);
    }

    public void save(){
        SharedPreferences sp = getApplication().getSharedPreferences(sh_name, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = sp.edit();
        editor.putInt(key_number,getNumber().getValue()==null?0:getNumber().getValue());
        editor.apply();
    }

    public void load(){
        SharedPreferences sp = getApplication().getSharedPreferences(sh_name,Context.MODE_PRIVATE);
        int x = sp.getInt(key_number,0);
        handle.set(key_number,x);
    }

    public void Add(int x){
        handle.set(key_number,getNumber().getValue()==null?0:getNumber().getValue()+x);
    }

}
```

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
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_percent="0.25"
            android:id="@+id/gl25"/>

        <androidx.constraintlayout.widget.Guideline
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            app:layout_constraintGuide_percent="0.35"
            android:id="@+id/gl35"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="35sp"
            android:id="@+id/tv_dispaly"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="@id/gl25"
            android:text="@{String.valueOf(data.getNumber())}"
            tools:text="200"/>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="+"
            android:id="@+id/btnAdd"
            android:textSize="20sp"
            android:onClick="@{()->data.Add(1)}"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="@id/glv50"
            app:layout_constraintTop_toTopOf="@id/gl25"
            app:layout_constraintBottom_toBottomOf="@id/gl35"
            android:layout_marginLeft="60dp"/>

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/btnMinus"
            android:textSize="20sp"
            android:onClick="@{()->data.Add(-1)}"
            android:text="-"
            app:layout_constraintStart_toStartOf="@id/glv50"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="@id/gl25"
            app:layout_constraintBottom_toBottomOf="@id/gl35"
            android:layout_marginRight="60dp"/>

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
import androidx.lifecycle.SavedStateViewModelFactory;
import androidx.lifecycle.ViewModelProvider;

import com.example.helloworld.databinding.ActivityMainBinding;

public class MainActivity extends AppCompatActivity {

    MyViewModel myViewModel;
    ActivityMainBinding binding;
    public final static String TAG = "MY_TAG";
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

    @Override
    protected void onPause() {
        super.onPause();
        myViewModel.save();
    }
}
```

