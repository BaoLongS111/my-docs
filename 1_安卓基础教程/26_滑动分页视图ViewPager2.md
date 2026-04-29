![image-20260418160741203](..\imgs\image-20260418160741203.png)

### 1.创建三个Fragment

### 2.MainActivity.xml里放入一个TabLayout，一个ViewPager2

MainActivity.kt

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
import androidx.fragment.app.Fragment
import androidx.lifecycle.Observer
import androidx.lifecycle.ViewModelProvider
import androidx.viewpager2.adapter.FragmentStateAdapter
import androidx.viewpager2.widget.ViewPager2
import com.google.android.material.tabs.TabLayout
import com.google.android.material.tabs.TabLayoutMediator

class MainActivity : AppCompatActivity() {

    private lateinit var tabLayout: TabLayout
    private lateinit var viewpager: ViewPager2

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContentView(R.layout.activity_main)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        tabLayout = findViewById<TabLayout>(R.id.tab_layout)
        viewpager = findViewById<ViewPager2>(R.id.view_pager)

        viewpager.adapter = object : FragmentStateAdapter(this){
            override fun createFragment(position: Int): Fragment {
                return when (position){
                    0 -> RotateFragment()
                    1 -> ScaleFragment()
                    else -> TransplateFragment()
                }
            }
            override fun getItemCount() = 3
        }

        TabLayoutMediator(tabLayout,viewpager){tab,position->
            when (position){
                0 -> tab.text = "旋转"
                1 -> tab.text = "缩放"
                else -> tab.text = "平移"
            }

        }.attach()

    }
}
```

ScaleFragment.kt

```kotlin
package com.example.kotlinapp

import android.animation.ObjectAnimator
import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import kotlin.random.Random

class ScaleFragment : Fragment() {
    private lateinit var imageView: ImageView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_scale, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        imageView = view.findViewById<ImageView>(R.id.iv_logo)

        imageView.setOnClickListener {
            val factor = if(Random.nextBoolean()) 0.1f else -0.1f
            ObjectAnimator.ofFloat(imageView,"scaleX",imageView.scaleX+factor).start()
            ObjectAnimator.ofFloat(imageView,"scaleY",imageView.scaleY+factor).start()
        }
    }
}
```

