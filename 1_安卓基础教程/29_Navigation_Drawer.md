

![image-20260422193419099](..\imgs\image-20260422193419099.png)

![image-20260422193451470](..\imgs\image-20260422193451470.png)

![image-20260422193534034](..\imgs\image-20260422193534034.png)

### 1.创建三个Fragment，然后创建my_menu文件，再创建my_nav_graph文件，把三个fragment放进去

### 2.在主布局把根布局改成drawer_layout，id也是，然后记得在MainActivity里把R.id.main改成R.id.drawer_layout

### 3.在主布局里放入一个NavHostFragment，也就是FragmentContrainer。然后放入一个NavigationView

Navigation有两个属性必须设置，一个是menu，一个layout_gravity，设置成start。还有个head布局文件可设置

### 4.在MainActivity里设置navigationView和controller绑定

```kotlin
        navigationView = findViewById<NavigationView>(R.id.navigationView)
        val navHost: NavHostFragment = supportFragmentManager.findFragmentById(R.id.fragmentContainerView) as NavHostFragment
        navController = navHost.navController
        drawer_layout = findViewById<DrawerLayout>(R.id.drawer_layout)

        appBarConfiguration = AppBarConfiguration(
            setOf<Int>(R.id.textFragment,R.id.listFragment,R.id.pagerFragment),drawer_layout
        )

        setupActionBarWithNavController(navController,appBarConfiguration)
        navigationView.setupWithNavController(navController)

    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
    }
```

这时候就能实现最基本的点击切换功能了

### 5.把ConstraintLayout和NavHostFragment给抽出来，然后添加一个AppBarLayout，放到Constraint下面，就会生成appbar，collapsingToolBar，toolbar，ScrollView等等

### 6.后面就是在三个页面里填充内容了

* **MainActivity.kt**

```kotlin
package com.example.drawerapplication

import android.os.Bundle
import androidx.activity.enableEdgeToEdge
import androidx.appcompat.app.AppCompatActivity
import androidx.appcompat.widget.Toolbar
import androidx.core.view.ViewCompat
import androidx.core.view.WindowInsetsCompat
import androidx.drawerlayout.widget.DrawerLayout
import androidx.navigation.NavController
import androidx.navigation.NavHost
import androidx.navigation.findNavController
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.ui.AppBarConfiguration
import androidx.navigation.ui.navigateUp
import androidx.navigation.ui.setupActionBarWithNavController
import androidx.navigation.ui.setupWithNavController
import com.google.android.material.navigation.NavigationView

class MainActivity : AppCompatActivity() {
    private lateinit var navigationView: NavigationView
    private lateinit var navController: NavController
    private lateinit var drawer_layout: DrawerLayout
    private lateinit var appBarConfiguration: AppBarConfiguration

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val toolbar = findViewById<Toolbar>(R.id.toolbar)
        setSupportActionBar(toolbar)
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.drawer_layout)) { v, insets ->
            val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom)
            insets
        }

        navigationView = findViewById<NavigationView>(R.id.navigationView)
        val navHost: NavHostFragment = supportFragmentManager.findFragmentById(R.id.fragmentContainerView) as NavHostFragment
        navController = navHost.navController
        drawer_layout = findViewById<DrawerLayout>(R.id.drawer_layout)

        appBarConfiguration = AppBarConfiguration(
            setOf<Int>(R.id.textFragment,R.id.listFragment,R.id.pagerFragment),drawer_layout
        )

        setupActionBarWithNavController(navController,appBarConfiguration)
        navigationView.setupWithNavController(navController)
    }

    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
    }
}
```

activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:openDrawer="start"
    tools:context=".MainActivity">


    <include layout="@layout/content_layout" />

    <com.google.android.material.navigation.NavigationView
        android:id="@+id/navigationView"
        android:layout_gravity="start"
        app:itemIconTint="@color/navigation_item_color"
        app:itemTextColor="@color/navigation_item_color"
        app:headerLayout="@layout/drawer_head"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        app:menu="@menu/my_menu" />

</androidx.drawerlayout.widget.DrawerLayout>
```







**TextFragment.kt**

```kotlin
package com.example.drawerapplication

import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import com.google.android.material.appbar.CollapsingToolbarLayout

class TextFragment : Fragment() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_text, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        requireActivity().findViewById<CollapsingToolbarLayout>(R.id.collapsingToolBar).title = "Text"
        requireActivity().findViewById<ImageView>(R.id.collapsingImageView).setImageResource(R.drawable.menu_1)
    }
}
```

fragment_text.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".TextFragment">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="In the heart of the bustling city, where the hum of traffic blends with the distant laughter of passersby, there lies a quiet café tucked between two towering buildings. Its windows are framed with potted plants that sway gently in the breeze, and the aroma of freshly brewed coffee drifts out onto the sidewalk, inviting anyone passing by to step inside. The walls are lined with bookshelves filled with well-loved novels, their pages worn soft from years of being opened and closed. Sunlight streams through the large windows, casting warm golden streaks across the wooden tables where people sit, lost in their own worlds. Some type furiously on laptops, others flip through magazines, and a few simply stare out the window, watching the world go by. The barista behind the counter moves with practiced ease, pulling shots of espresso and steaming milk, while soft jazz plays in the background, creating a cozy, almost timeless atmosphere. Outside, the city continues to rush forward, but inside this small café, time seems to slow down, offering a peaceful escape from the chaos of everyday life. Whether you come here to work, read, or just sit and people-watch, there’s a sense of calm that wraps around you the moment you walk through the door. The little details make all the difference—the chipped mug that holds your latte, the faint sound of rain tapping against the glass, the friendly nod from the regular who sits at the same corner table every morning. These small, unspoken moments are what make places like this feel like home, even if you’re just visiting for the first time. As the day fades into evening, the lights dim and the café takes on a different kind of magic, with string lights twinkling above the tables and the smell of freshly baked pastries filling the air. It’s the kind of place where memories are made, where conversations flow easily, and where you can stay for hours without ever feeling rushed. So next time you find yourself in need of a break from the noise, step into a place like this. You might just find exactly what you didn’t know you were looking for: a quiet corner, a warm drink, and a moment to breathe." />

</FrameLayout>
```



**ListFragment.kt**

```kotlin
package com.example.drawerapplication

import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.google.android.material.appbar.CollapsingToolbarLayout

class ListFragment : Fragment() {

    // 1. 只初始化 1 次
    private val adapter by lazy { MyAdapter(false) }
    private val layoutManager by lazy { GridLayoutManager(context, 2) }
    private val iconList by lazy { getListIcon() } // 随机列表只生成一次

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        return inflater.inflate(R.layout.fragment_list, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // 设置标题
        requireActivity().findViewById<CollapsingToolbarLayout>(R.id.collapsingToolBar).title = "List"
        requireActivity().findViewById<ImageView>(R.id.collapsingImageView).setImageResource(R.drawable.menu_2)

        // 2. 只配置一次，不重复创建
        val recycler: RecyclerView = view.findViewById(R.id.recycler_view)
        recycler.layoutManager = layoutManager
        recycler.adapter = adapter

        // 3. 只提交一次数据！
        if (adapter.currentList.isEmpty()) {
            adapter.submitList(iconList)
        }
    }

}
// 生成随机图标
fun getListIcon(): List<Int> {
    val icons = intArrayOf(
        R.drawable.ic_1,
        R.drawable.ic_2,
        R.drawable.ic_3,
        R.drawable.ic_4,
        R.drawable.ic_5,
        R.drawable.ic_6,
        R.drawable.ic_7,
        R.drawable.ic_8,
        R.drawable.ic_9,
        R.drawable.ic_10
    )
    return Array(50) { icons.random() }.toList()
}
```

fragment_list.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ListFragment">

    <androidx.recyclerview.widget.RecyclerView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/recycler_view"
        />

</FrameLayout>
```



**PagerFragment.kt**

```kotlin
package com.example.drawerapplication

import android.os.Bundle
import androidx.fragment.app.Fragment
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import androidx.viewpager2.widget.ViewPager2
import com.google.android.material.appbar.CollapsingToolbarLayout

class PagerFragment : Fragment() {

    private lateinit var viewPager: ViewPager2
    private val adapter: MyAdapter by lazy { MyAdapter(true) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
    }
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_pager, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        requireActivity().findViewById<CollapsingToolbarLayout>(R.id.collapsingToolBar).title = "Pager"
        requireActivity().findViewById<ImageView>(R.id.collapsingImageView).setImageResource(R.drawable.menu_3)


        viewPager = view.findViewById<ViewPager2>(R.id.view_pager)
        viewPager.adapter = adapter
        adapter.submitList(getListIcon())
    }

}
```

fragment_pager.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    tools:context=".PagerFragment">

    <androidx.viewpager2.widget.ViewPager2
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/view_pager"/>

</FrameLayout>
```





**content_layout.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"


    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appbar"
        android:fitsSystemWindows="true"
        android:layout_height="160dp"
        android:layout_width="match_parent">

        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsingToolBar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            app:scrimAnimationDuration="200"
            app:toolbarId="@+id/toolbar">

            <ImageView
                android:id="@+id/collapsingImageView"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:src="@drawable/menu_1"
                app:layout_collapseMode="parallax"
                app:layout_collapseParallaxMultiplier="0.5"
                android:layout_gravity="end|center"
                android:layout_marginEnd="20dp"/>

<!--          app:layout_collapseMode="pin"
           这个设置可以让左边的三道杠保留，不用被toolbar挤没-->

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                app:layout_collapseMode="pin"
                android:layout_height="?attr/actionBarSize"></androidx.appcompat.widget.Toolbar>
        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>

    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fillViewport="true"
        app:layout_behavior="com.google.android.material.appbar.AppBarLayout$ScrollingViewBehavior">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:showIn="@layout/activity_main">

            <androidx.fragment.app.FragmentContainerView
                android:id="@+id/fragmentContainerView"
                android:name="androidx.navigation.fragment.NavHostFragment"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                app:defaultNavHost="true"
                app:navGraph="@navigation/my_nav_grap" />

        </androidx.constraintlayout.widget.ConstraintLayout>

    </androidx.core.widget.NestedScrollView>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```



**drawer_head.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <ImageView
        android:id="@+id/iv_icon"
        android:layout_width="80dp"
        android:layout_height="80dp"
        android:src="@drawable/menu_1"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```



**my_menu.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <group
        android:id="@+id/group1"
        android:checkableBehavior="single" >
        <item
            android:id="@+id/textFragment"
            android:icon="@drawable/menu_1"
            android:title="Text" />
    </group>
    <group
        android:id="@+id/group2"
        android:checkableBehavior="single">

        <item
            android:id="@+id/listFragment"
            android:icon="@drawable/menu_2"
            android:title="List" />
        <item
            android:id="@+id/pagerFragment"
            android:icon="@drawable/menu_3"
            android:title="Pager" />
    </group>
</menu>
```



**MyAdapter.kt**

```kotlin
package com.example.drawerapplication

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView

class MyAdapter(val isPager: Boolean) : ListAdapter<Int, RecyclerView.ViewHolder>(callback) {
    object callback : DiffUtil.ItemCallback<Int>() {
        override fun areItemsTheSame(p0: Int, p1: Int): Boolean {
            return p0 == p1
        }

        override fun areContentsTheSame(p0: Int, p1: Int): Boolean {
            return p0 == p1
        }
    }

    override fun onCreateViewHolder(
        p0: ViewGroup,
        p1: Int
    ): RecyclerView.ViewHolder {
        val view: View = LayoutInflater.from(p0.context).inflate(R.layout.cell_layout, p0, false)
        if(isPager){
            view.layoutParams.height = ViewGroup.LayoutParams.MATCH_PARENT
            view.findViewById<ImageView>(R.id.iv_icon).layoutParams.height = 0
            view.findViewById<ImageView>(R.id.iv_icon).layoutParams.width = 0
        }
        return object : RecyclerView.ViewHolder(view){}
    }

    override fun onBindViewHolder(
        p0: RecyclerView.ViewHolder,
        p1: Int
    ) {
        p0.itemView.findViewById<ImageView>(R.id.iv_icon).setImageResource(getItem(p1))
    }
}
```





**my_nav_graph.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/my_nav_grap"
    app:startDestination="@id/textFragment">

    <fragment
        android:id="@+id/textFragment"
        android:name="com.example.drawerapplication.TextFragment"
        android:label="Text"
        tools:layout="@layout/fragment_text" />
    <fragment
        android:id="@+id/listFragment"
        android:name="com.example.drawerapplication.ListFragment"
        android:label="List"
        tools:layout="@layout/fragment_list" />
    <fragment
        android:id="@+id/pagerFragment"
        android:name="com.example.drawerapplication.PagerFragment"
        android:label="Pager"
        tools:layout="@layout/fragment_pager" />
</navigation>
```

