![image-20260404103056469](..\imgs\image-20260404103056469.png)

### 1.LifeCycles就是让我们自己创建的对象，可以时刻观察Activity或者Fragment的生命周期，然后在对应的生命周期做对应的事。这样做的好处就是解耦，不用在控制器的生命周期函数做过多的逻辑处理，代码独立性更高。

### 2.这里的例子是一个计数器，创建一个MyChronometer，继承自Chronometer，然后实现DefaultLifeCycleObserver,以前的注解写法被安卓官方在2021年弃用了，因为以前的注解写法太耗费资源。Chronometer是继承自TextView的。注意最后一定要在控制器里添加Observer

```java MyChronomter.class
package com.example.helloworld;

import android.content.Context;
import android.os.SystemClock;
import android.system.SystemCleaner;
import android.util.AttributeSet;
import android.widget.Chronometer;

import androidx.annotation.NonNull;
import androidx.lifecycle.DefaultLifecycleObserver;
import androidx.lifecycle.LifecycleOwner;

public class MyChronometer extends Chronometer implements DefaultLifecycleObserver {

    private static long elaspedTime;
    public MyChronometer(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
        DefaultLifecycleObserver.super.onPause(owner);
        elaspedTime = SystemClock.elapsedRealtime() - getBase();
        stop();
    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
        DefaultLifecycleObserver.super.onResume(owner);
        setBase(SystemClock.elapsedRealtime()-elaspedTime);
        start();
    }
}

```

```java MainActivity.class
package com.example.helloworld;

import android.os.Bundle;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

public class MainActivity extends AppCompatActivity {

    MyChronometer myChronometer;

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

        myChronometer = findViewById(R.id.tv_display);
        getLifecycle().addObserver(myChronometer);
    }
}
```

``` xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.5"
        android:id="@+id/glv50"
        />

    <androidx.constraintlayout.widget.Guideline
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5"
        android:id="@+id/gl50"/>

    <com.example.helloworld.MyChronometer
        android:id="@+id/tv_display"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="50"
        android:textSize="50sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="@id/gl50"
        app:layout_constraintBottom_toBottomOf="@id/gl50"/>


</androidx.constraintlayout.widget.ConstraintLayout>
```

