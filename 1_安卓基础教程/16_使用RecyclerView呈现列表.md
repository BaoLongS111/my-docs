![image-20260405180345131](..\imgs\image-20260405180345131.png)

![image-20260405180413146](..\imgs\image-20260405180413146.png)

### 1.把要显示的地方替换成RecyclerView

### 2.创建cell_normal或者cell_card布局文件，作为每一个item的布局

![image-20260405181542907](..\imgs\image-20260405181542907.png)

```xml cel_normal.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?selectableItemBackground"
    android:clickable="true">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/gl10"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.1" />

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/gl85"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="0.85" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="1"
        android:id="@+id/tv_id"
        android:textSize="16sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="@id/gl10"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"/>

    <TextView
        android:layout_marginTop="8dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="22sp"
        android:text="Name"
        android:id="@+id/tv_english"
        app:layout_constraintStart_toStartOf="@id/gl10"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        android:text="Chinese"
        android:id="@+id/tv_chinese"
        app:layout_constraintStart_toStartOf="@id/gl10"
        app:layout_constraintTop_toBottomOf="@id/tv_english"/>

</androidx.constraintlayout.widget.ConstraintLayout>
```

![image-20260405181621731](..\imgs\image-20260405181621731.png)

```xml cell_card.xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginStart="8dp"
    android:layout_marginEnd="8dp"
    android:layout_marginTop="8dp"
    android:foreground="?selectableItemBackground"
    android:clickable="true">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/gl10"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.1" />

        <androidx.constraintlayout.widget.Guideline
            android:id="@+id/gl85"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            app:layout_constraintGuide_percent="0.85" />

        <TextView
            android:id="@+id/tv_id"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="1"
            android:textSize="16sp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="@id/gl10"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_english"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="8dp"
            android:text="Name"
            android:textSize="22sp"
            app:layout_constraintStart_toStartOf="@id/gl10"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/tv_chinese"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Chinese"
            android:textSize="16sp"
            app:layout_constraintStart_toStartOf="@id/gl10"
            app:layout_constraintTop_toBottomOf="@id/tv_english" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</androidx.cardview.widget.CardView>
```

### 3.创建Adapter类，继承自RecyclerView.Adapter类。然后创建内部静态Holder类，继承自Recycler.Holder类。Holder主要是持有cell里的组件，Adapter类完成数据到组件的绑定。

```java MyViewAdapter.java
package com.example.helloworld;

import android.content.Intent;
import android.net.Uri;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import com.example.helloworld.Entity.Word;

import java.util.ArrayList;
import java.util.List;

public class MyViewAdapter extends RecyclerView.Adapter<MyViewAdapter.MyHolder>{

    private List<Word> allWords = new ArrayList<>();

    private boolean flag;

    public MyViewAdapter(boolean flag){
        this.flag = flag;
    }

    @NonNull
    @Override
    public MyHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater layoutInflater = LayoutInflater.from(parent.getContext());
        View itemView ;
        if(flag){
            itemView = layoutInflater.inflate(R.layout.cell_card,parent,false);
        }else{
            itemView = layoutInflater.inflate(R.layout.cell_normal,parent,false);
        }
        return new MyHolder(itemView);
        // return null;
    }

    @Override
    public void onBindViewHolder(@NonNull MyHolder holder, int position) {
        Word word = allWords.get(position);
        holder.tvId.setText(String.valueOf(position+1));
        holder.tvEnglish.setText(word.getWord());
        holder.tvChinese.setText(word.getChineseMeaning());

        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Uri uri = Uri.parse("https://dict.youdao.com/m/result?word="+word.getWord()+"&lang=en");
                Intent intent = new Intent(Intent.ACTION_VIEW);
                intent.setData(uri);
                holder.itemView.getContext().startActivity(intent);
            }
        });
    }

    @Override
    public int getItemCount() {
        return allWords.size();
    }

    public void setAllWords(List<Word> allWords) {
        this.allWords = allWords;
    }

    static class MyHolder extends RecyclerView.ViewHolder{
        TextView tvId,tvEnglish,tvChinese;

        public MyHolder(@NonNull View itemView) {
            super(itemView);
            tvId =  itemView.findViewById(R.id.tv_id);
            tvEnglish = itemView.findViewById(R.id.tv_english);
            tvChinese = itemView.findViewById(R.id.tv_chinese);
        }
    }
}

```

注意OnCreateViewHolder里代码的编写: LayoutInflater layoutInflater = LayoutInflater.from(parent.getContext());

itemView = layoutInflater.inflate(R.layout.cell_normal,parent,false);

return new MyHolder(itemView);



### 4.在MainActivity里调用

```java MainActivity.java
package com.example.helloworld;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.CompoundButton;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.SwitchCompat;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModelProvider;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import androidx.room.Room;

import com.example.helloworld.DAO.WordDAO;
import com.example.helloworld.Database.WordDatabase;
import com.example.helloworld.Entity.Word;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    MyViewAdapter myViewAdapter1;
    MyViewAdapter myViewAdapter2;
    SwitchCompat switchCompat;
    RecyclerView recyclerView;
    WordViewModel viewModel;
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
        viewModel = new ViewModelProvider(this).get(WordViewModel.class);
        btnInsert = findViewById(R.id.btnInsert);
        btnUpdate = findViewById(R.id.btnUpdate);
        btnDelete = findViewById(R.id.btnDelete);
        btnClear = findViewById(R.id.btnClear);
        recyclerView = findViewById(R.id.recycler);
        switchCompat = findViewById(R.id.switchCompact);

        myViewAdapter1 = new MyViewAdapter(false);
        myViewAdapter2 = new MyViewAdapter(true);


        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.setAdapter(myViewAdapter1);


        viewModel.getAllWords().observe(this, new Observer<List<Word>>() {
            @Override
            public void onChanged(List<Word> words) {
                myViewAdapter1.setAllWords(viewModel.getAllWords().getValue());
                myViewAdapter2.setAllWords(viewModel.getAllWords().getValue());
                myViewAdapter1.notifyDataSetChanged();
                myViewAdapter2.notifyDataSetChanged();
            }
        });

        switchCompat.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(@NonNull CompoundButton compoundButton, boolean b) {
                if(b){
                    recyclerView.setAdapter(myViewAdapter2);
                }else{
                    recyclerView.setAdapter(myViewAdapter1);
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

