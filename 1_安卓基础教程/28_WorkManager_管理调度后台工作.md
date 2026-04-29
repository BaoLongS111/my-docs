![image-20260421154524769](..\imgs\image-20260421154524769.png)

![image-20260421204902220](..\imgs\image-20260421204902220.png)

### 1.创建MyWork类，继承于Worker类，然后实现doWork方法

```kotlin
package com.example.workapplication

import android.content.Context
import android.content.SharedPreferences
import android.util.Log
import androidx.work.Worker
import androidx.work.WorkerParameters
import androidx.work.workDataOf
import androidx.core.content.edit

class MyWorker(context: Context, workerParams: WorkerParameters) : Worker(context, workerParams) {
    private val TAG: String? = "HelloWorld"

    override fun doWork(): Result {
        val name = inputData.getString(INPUT_WORK_KEY)
        Log.d(TAG, "doWork:Work ${name} started")
        Thread.sleep(3000)
        Log.d(TAG, "doWork:Work ${name} finished")
        val sp = applicationContext.getSharedPreferences(SHARED_PREFERENCES_NAME, Context.MODE_PRIVATE)
        var number:Int = sp.getInt(name,0)
        sp.edit { putInt(name, ++number) }
        return Result.success(workDataOf(OUTPUT_WORK_KEY to "$name output"))
    }
}
```

### 2.在MainActivity里调用

1. 先创建WorkManager的实例，它是singleton模式

2. 创建workRequest，可以设置约束，也就是Constraints

3. 把workRequest添加到队列里，就会按顺序执行

```kotlin
package com.example.workapplication

import android.content.Context
import android.content.SharedPreferences
import android.os.Bundle
import android.util.Log
import android.widget.Toast
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.work.Constraints
import androidx.work.NetworkType
import androidx.work.OneTimeWorkRequest
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkInfo
import androidx.work.WorkManager
import androidx.work.workDataOf
import com.example.workapplication.databinding.ActivityMainBinding

const val INPUT_WORK_KEY:String= "input_work_key"
const val OUTPUT_WORK_KEY:String= "output_work_key"
const val WORK_A_NAME:String = "work_a_name"
const val WORK_B_NAME:String = "work_b_name"
const val SHARED_PREFERENCES_NAME:String  = "sp_name"
const val NUMBER_KEY:String  = "number_key"
class MainActivity : AppCompatActivity(), SharedPreferences.OnSharedPreferenceChangeListener {
    private lateinit var binding: ActivityMainBinding
    private val TAG:String = "HelloWorld"
    private val workManager: WorkManager = WorkManager.getInstance(this)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        ViewCompat.setOnApplyWindowInsetsListener(binding.main) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        val sp = applicationContext.getSharedPreferences(SHARED_PREFERENCES_NAME,Context.MODE_PRIVATE)
        sp.registerOnSharedPreferenceChangeListener(this)
        updateView()
        binding.btnStart.setOnClickListener {
            val workARequest = createWork(WORK_A_NAME)
            val workBRequest = createWork(WORK_B_NAME)
            workManager.beginWith(workARequest)
                .then(workBRequest)
                .enqueue()

//            workManager.getWorkInfoByIdLiveData(workARequest.id).observe(this) {workInfo ->
//                if(workInfo!=null && workInfo.state == WorkInfo.State.SUCCEEDED) {
//                    Log.d(TAG, "onCreate: ${workInfo.outputData.getString(OUTPUT_WORK_KEY)}")
//                }
//            }

        }

    }

    private fun createWork(name:String): OneTimeWorkRequest {
        val constraint: Constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
        val workRequest = OneTimeWorkRequestBuilder<MyWorker>()
//                .setConstraints(constraint)
            .setInputData(workDataOf(INPUT_WORK_KEY to name))
            .build()
        return workRequest
    }

    private fun updateView(){
        val sp = applicationContext.getSharedPreferences(SHARED_PREFERENCES_NAME, Context.MODE_PRIVATE)
        binding.text1.text = sp.getInt(WORK_A_NAME,0).toString();
        binding.text2.text = sp.getInt(WORK_B_NAME,0).toString();
    }

    override fun onSharedPreferenceChanged(
        p0: SharedPreferences?,
        p1: String?
    ) {
        updateView()
    }

}
```




### 3.注意点

因为workManager都是后台异步执行，所以为了监听到sp里数据的变化，在MainActivity里实现了OnSharedPreferencesChangedListener

### 4.布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:id="@+id/text1"
        android:textSize="30sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        android:id="@+id/text2"
        android:textSize="30sp"
        android:layout_marginTop="20sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text1"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Start"
        android:id="@+id/btn_start"
        android:layout_marginTop="20sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toBottomOf="@id/text2"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

