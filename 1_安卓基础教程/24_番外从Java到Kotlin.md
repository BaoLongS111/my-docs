![image-20260417154404191](..\imgs\image-20260417154404191.png)



### 1.创建项目，语言选择Kotlin，然后创建ViewModel

```kotlin
package com.example.kotlinapp

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel

class MyViewModel: ViewModel() {
    private val _number: MutableLiveData<Int> by lazy { MutableLiveData<Int>().also { it.value=0 } }
    val number:LiveData<Int>
        get() = _number
//    var number2 = MutableLiveData<Int>(0)
//    var number3: MutableLiveData<Int>
//    init{
//        number3 = MutableLiveData<Int>()
//        number3.value = 0
//    }

    fun modifyNumber(n:Int){
        _number.value = _number.value?.plus(n)
    }
}
```

### 2.在Controller里调用

```kotlin
package com.example.kotlinapp

import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import androidx.activity.enableEdgeToEdge
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider

class MainActivity : AppCompatActivity() {

//    val viewModel2: MyViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }
        val viewModel: MyViewModel = ViewModelProvider(this).get(MyViewModel::class.java)
        val tvNumber = findViewById<TextView>(R.id.tvNumber)
        val btnAdd = findViewById<Button>(R.id.btnAdd)
        val btnMinus = findViewById<Button>(R.id.btnMinus)

//        viewModel.number.observe(this, Observer{tvNumber.text = it.toString()})
        viewModel.number.observe(this) { value ->
            tvNumber.text = value.toString()
        }

        btnAdd.setOnClickListener { viewModel.modifyNumber(1) }
        btnMinus.setOnClickListener { viewModel.modifyNumber(-1) }

    }
}
```

