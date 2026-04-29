### 1.改善上一个实例

1.把getAllWords改为LiveData。

2.把Database改为singleton设计模式，也就是单例模式。

3.对数据库的操作不应该放在主线程上，应该用AsyncTask放在另外一个线程。

4.将Activity里的数据操作放到ViewModel里面。

5.对数据的操作不应该是ViewModel的职责，应该写一个仓库类放这些对数据的操作。

### 2.Word Entity(未改动)

```java Word.java
package com.example.helloworld.Entity;

import androidx.room.ColumnInfo;
import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity(table_name="WORD")
public class Word {
    @PrimaryKey(autoGenerate = true)
    private long id;

    @ColumnInfo(name = "english_word")
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

### 3.WordDAO(将getAllWords的类型改成了LiveData格式)

getAllWords不用异步请求，因为默认就是在异步。

```java WordDAO.java
package com.example.helloworld.DAO;

import androidx.lifecycle.LiveData;
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
    LiveData<List<Word>> getAllWords();
}

```

### 4.WordDatabase(改为singleton格式)

```java WordDatabase.java
package com.example.helloworld.Database;

import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Entity.Word;

@Database(entities = {Word.class}, version = 1, exportSchema = false)
public abstract class WordDatabase extends RoomDatabase {
    private static volatile WordDatabase INSTANCE;

    public static WordDatabase getDatabase(Context context) {
        if (INSTANCE == null) {
            synchronized (WordDatabase.class) {
                if (INSTANCE == null)
                {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(), WordDatabase.class, "word_database")
                            .build();
                }
            }
        }
        return INSTANCE;
    }

    public abstract WordDAO getWordDAO();
}
```

### 5.WrodRepository(创建仓库，里面写数据库的获取，和对应的数据操作)

```java WordRepository.java
package com.example.helloworld.Repository;
import android.content.Context;
import android.os.Handler;
import android.os.Looper;

import androidx.lifecycle.LiveData;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Database.WordDatabase;
import com.example.helloworld.Entity.Word;

import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class WordRepository {

    private LiveData<List<Word>> allWordsLive;
    private ExecutorService executorService;
    private Handler mainHandler;
    public WordDatabase wordDatabase;
    public WordDAO wordDAO;
    public WordRepository(Context context){
        wordDatabase = WordDatabase.getDatabase(context);
        wordDAO = wordDatabase.getWordDAO();
        executorService = Executors.newSingleThreadExecutor();
        mainHandler = new Handler(Looper.getMainLooper());
        allWordsLive = wordDAO.getAllWords(); //getAllWords()系统自动在子线程进行。
    }

    public void insertWords(OnDbOperationListener listener, Word...words){
        executorService.execute(()->{
            wordDAO.insertWords(words);
            mainHandler.post(()->listener.onSuccess());
        });
    }

    public void updateWords(OnDbOperationListener listener,Word...words){
        executorService.execute(()->{
            wordDAO.updateWords(words);
            mainHandler.post(()->listener.onSuccess());
        });

    }

    public void deleteWords(OnDbOperationListener listener,Word...words){
        executorService.execute(()->{
            wordDAO.deleteWords(words);
            mainHandler.post(()->listener.onSuccess());
        });

    }

    public void deleteAllWords(){
        executorService.execute(()->{
            wordDAO.deleteAllWords();
        });

    }

    public LiveData<List<Word>> getAllWordsLive(){
        return allWordsLive;
    }

    public interface OnDbOperationListener{
        void onSuccess();
    }

//    public interface OnQueryListener{
//        void onSuccess(List<Word> words);
//    }

}
```

### 6.WordViewModel(对WordRepository的封装，Activity只要操作WordViewModel就行了，更简洁)

注意，ViewModel里的对数据的方法尽量和Repository里一致。

```java WordViewModel.java
package com.example.helloworld;

import android.app.Application;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.LiveData;

import com.example.helloworld.Entity.Word;
import com.example.helloworld.Repository.WordRepository;

import java.util.List;

public class WordViewModel extends AndroidViewModel {
    WordRepository wordRepository;
    Application application;
    public WordViewModel(@NonNull Application application) {
        super(application);
        wordRepository = new WordRepository(application.getApplicationContext());
        this.application = application;
    }

    public LiveData<List<Word>> getAllWords(){
        return wordRepository.getAllWordsLive();
    }

    void insertWords(Word...words){
        wordRepository.insertWords(new WordRepository.OnDbOperationListener() {
            @Override
            public void onSuccess() {
                Toast.makeText(application.getApplicationContext(),"插入成功！",Toast.LENGTH_SHORT).show();
            }
        },words);
    }

    void updateWords(Word...words){
        wordRepository.updateWords(new WordRepository.OnDbOperationListener() {
            @Override
            public void onSuccess() {

            }
        },words);
    }

    void deleteWords(Word...words){
        wordRepository.deleteWords(new WordRepository.OnDbOperationListener() {
            @Override
            public void onSuccess() {

            }
        },words);
    }

    void deleteAllWords(){
        wordRepository.deleteAllWords();
    }
}
```

### 7.Activity里的调用。

首先不用写updateView()这个方法了，只需要对ViewModel里的getAllWords()进行监听就行了，发现改变就更新一下ui就行了。其他所有对数据的操作统一调用ViewModel

```java MainActivity.java
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
import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModelProvider;
import androidx.room.Room;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Database.WordDatabase;
import com.example.helloworld.Entity.Word;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    WordViewModel viewModel;
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

        viewModel = new ViewModelProvider(this).get(WordViewModel.class);
        viewModel.getAllWords().observe(this, new Observer<List<Word>>() {
            @Override
            public void onChanged(List<Word> words) {
                textView.setText("");
                for (Word word : words) {
                    textView.setText(textView.getText().toString()+word.getId()+" "+word.getWord()+":"+word.getChineseMeaning()+"\n");
                }
            }
        });

        btnInsert.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hello","你好");
                Word word2 = new Word("World","世界");
                viewModel.insertWords(word1,word2);
            }
        });

        btnUpdate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hi","你好啊");
                word1.setId(32);
                viewModel.updateWords(word1);
            }
        });

        btnDelete.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Word word1 = new Word("Hi","你好啊");
                word1.setId(32);
                viewModel.deleteWords(word1);
            }
        });

        btnClear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                viewModel.deleteAllWords();
            }
        });

    }
}
```

