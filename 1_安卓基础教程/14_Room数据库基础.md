### 1.Room是什么

Room是一套工具，是对SQLite数据库的抽象，Android的原生态是支持SQLite的，后来在18年的时候，Google才加入了Room这个库，是为了达到能够流畅的访问SQLite数据库。![image-20260404141121486](..\imgs\image-20260404141121486.png)

Repository用来对应用程序和数据的解耦和隔离，使得我们对数据的读取都放在Repository的单独的类中实现。

RecyclerView是用来对数据库数据的呈现。

### 2.怎么使用Room

1.在官网搜索Room

![image-20260404143906181](..\imgs\image-20260404143906181.png)

2.复制依赖到build.gradle.kts(Module:app)文件里，然后sync now

```kotlin
dependencies {
    val room_version = "2.8.4"

    implementation("androidx.room:room-runtime:$room_version")

    // If this project uses any Kotlin source, use Kotlin Symbol Processing (KSP)
    // See Add the KSP plugin to your project
    ksp("androidx.room:room-compiler:$room_version")

    // If this project only uses Java source, use the Java annotationProcessor
    // No additional plugins are necessary
    annotationProcessor("androidx.room:room-compiler:$room_version")

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation("androidx.room:room-ktx:$room_version")

    // optional - RxJava2 support for Room
    implementation("androidx.room:room-rxjava2:$room_version")

    // optional - RxJava3 support for Room
    implementation("androidx.room:room-rxjava3:$room_version")

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation("androidx.room:room-guava:$room_version")

    // optional - Test helpers
    testImplementation("androidx.room:room-testing:$room_version")

    // optional - Paging 3 Integration
    implementation("androidx.room:room-paging:$room_version")
}
```

### 3.创建Entity

Entity就是实例，这里创建的Word实体对象。

类名上面要写上@Entity的注解，然后id是自增长主键，也要写上注解。每个字段名字可以写上字段的名字方便我们记忆，这个是可写可不写。

```Word.java
package com.example.helloworld.Entity;

import androidx.room.ColumnInfo;
import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity
public class Word {
    @PrimaryKey(autoGenerate = true)
    private long id;

    @ColumnInfo(name = "word_name")
    private String word;

    @ColumnInfo(name = "chinese_meaning")
    private String chineseMeaning;

    public Word(String word, String chineseMeaning) {
        this.word = word;
        this.chineseMeaning = chineseMeaning;
    }

    public long getId(){
        return this.id;
    }

    public void setId(long id){
        this.id = id;
    }

    public String getWord() {
        return word;
    }

    public void setWord(String word) {
        this.word = word;
    }

    public String getChineseMeaning() {
        return chineseMeaning;
    }

    public void setChineseMeaning(String chineseMeaning) {
        this.chineseMeaning = chineseMeaning;
    }
}

```

### 4.DAO 

DAO也就是Database access object，数据库访问对象，一般是一个接口或者抽象类，里面定义增删改查的各种操作。但是不用实现，因为写上注解后，Room工具会自动给我们实现WordDAO_Impl.java

```WordDAO.java
package com.example.helloworld.DAO;

import androidx.room.Dao;
import androidx.room.Delete;
import androidx.room.Insert;
import androidx.room.Query;
import androidx.room.Update;

import com.example.helloworld.Entity.Word;

import java.util.List;

@Dao  //database access object
public interface WordDAO {
    @Insert
    void insertWords(Word...words);

    @Update
    void updateWords(Word...words);

    @Delete
    void deleteWords(Word...words);

    @Query("DELETE FROM WORD")
    void deleteAllWords();

    @Query("SELECT * FROM WORD ORDER BY id DESC")
    List<Word> getAllWords();
}
```

### 5.Database

database是一个抽象类，要继承自RoomDataBase。Database主要做的任务就是返回Dao对象，如果有多个Entity，就要返回多个DAO，后面我们在控制器里就要通过database来得到对应实体的DAO。database要写上注解，版本号就是如果数据库更改了，新增字段或者删除某一字段，就要依靠版本号来进行marigation迁移。

```WordDatabase.java
package com.example.helloworld.Database;

import androidx.room.Database;
import androidx.room.RoomDatabase;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Entity.Word;

@Database(entities = {Word.class},version = 1,exportSchema = false)
public abstract class WordDatabase extends RoomDatabase {
    public abstract WordDAO getWordDAO();
}

```

### 6.控制器访问数据库并显示

![image-20260404164102454](..\imgs\image-20260404164102454.png)

```java MainActivity
package com.example.helloworld;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.room.Room;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Database.WordDatabase;
import com.example.helloworld.Entity.Word;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    WordDatabase wordDatabase;
    WordDAO wordDAO;
    TextView textView;
    Button btnInsert,btnUpdate,btnDelete,btnClear;

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
        textView = findViewById(R.id.tv_display);
        btnInsert = findViewById(R.id.btnInsert);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        btnClear = findViewById(R.id.btnClear);
        wordDatabase = Room.databaseBuilder(this, WordDatabase.class, "word_database")
                .allowMainThreadQueries()
                .build();
        wordDAO = wordDatabase.getWordDAO();
        updateView();

        btnInsert.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hello","你好");
                Word word2 = new Word("World","世界");
                wordDAO.insertWords(word1,word2);
                updateView();
            }
        });

        btnUpdate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hi","你好啊");
                word1.setId(1);
                wordDAO.updateWords(word1);
                updateView();
            }
        });

        btnDelete.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hi","你好啊");
                word1.setId(2);
                wordDAO.deleteWords(word1);
                updateView();
            }
        });

        btnClear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                wordDAO.deleteAllWords();
                updateView();
            }
        });

    }

    void updateView() {
        textView.setText("");
        List<Word> lists = wordDAO.getAllWords();
        for (Word word : lists) {
            textView.setText(textView.getText().toString()+word.getId()+" "+word.getWord()+":"+word.getChineseMeaning()+"\n");
        }
    }
}
```

布局文件

```layout_main.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
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
        android:id="@+id/gl50"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5" />

    <androidx.constraintlayout.widget.Guideline
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.65"
        android:id="@+id/gl65"/>

    <androidx.constraintlayout.widget.Guideline
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.80"
        android:id="@+id/gl80"/>

    <ScrollView
        android:id="@+id/scroll"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="@id/gl50"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text=""
            android:textSize="25sp"
            android:id="@+id/tv_display"/>

    </ScrollView>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/btnInsert"
        android:text="insert"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="@id/glv50"
        app:layout_constraintTop_toTopOf="@id/gl50"
        app:layout_constraintBottom_toBottomOf="@id/gl65"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/btnUpdate"
        android:text="update"
        app:layout_constraintStart_toStartOf="@id/glv50"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="@id/gl50"
        app:layout_constraintBottom_toBottomOf="@id/gl65"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/btnDelete"
        android:text="delete"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="@id/glv50"
        app:layout_constraintTop_toTopOf="@id/gl65"
        app:layout_constraintBottom_toBottomOf="@id/gl80"/>

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/btnClear"
        android:text="clear"
        app:layout_constraintStart_toStartOf="@id/glv50"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="@id/gl65"
        app:layout_constraintBottom_toBottomOf="@id/gl80"/>


</androidx.constraintlayout.widget.ConstraintLayout>
```

