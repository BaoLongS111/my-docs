![image-20260413135120849](..\imgs\image-20260413135120849.png)

```java
dependencies {
  def paging_version = "3.4.2"

  implementation "androidx.paging:paging-runtime:$paging_version"

  // alternatively - without Android dependencies for tests
  testImplementation "androidx.paging:paging-common:$paging_version"

  // optional - RxJava2 support
  implementation "androidx.paging:paging-rxjava2:$paging_version"

  // optional - RxJava3 support
  implementation "androidx.paging:paging-rxjava3:$paging_version"

  // optional - Guava ListenableFuture support
  implementation "androidx.paging:paging-guava:$paging_version"

  // optional - Jetpack Compose integration
  implementation "androidx.paging:paging-compose:3.5.0-beta01"
}
```

- **PagingSource** —— 负责加载每一页数据
- **PagingData** —— 分页数据载体
- **PagingDataAdapter** —— 列表适配器
- **Pager + Flow / LiveData** —— 提供分页数据流



### 1.依赖

```java
    val room_version = "2.6.1"

    implementation("androidx.room:room-runtime:$room_version")

    // If this project only uses Java source, use the Java annotationProcessor
    // No additional plugins are necessary
    annotationProcessor("androidx.room:room-compiler:$room_version")

    // optional - Test helpers
    testImplementation("androidx.room:room-testing:$room_version")
    implementation("androidx.room:room-paging:${room_version}")

    var paging_version = "3.2.1"

    implementation("androidx.paging:paging-runtime:$paging_version")
```

### 2.依次编写Student, StudentDAO,StudentDatabase,StudentRepository,StudentViewModel,MyPagingAdapter

Student.java

```java
package com.example.helloworld;

import androidx.room.ColumnInfo;
import androidx.room.Entity;
import androidx.room.PrimaryKey;

@Entity(tableName = "student_table")
public class Student {
    @PrimaryKey(autoGenerate = true)
    private int id;

    @ColumnInfo(name = "student_number")
    private int studentNnumber;

    public int getStudentNnumber() {
        return studentNnumber;
    }

    public void setStudentNnumber(int studentNnumber) {
        this.studentNnumber = studentNnumber;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
}
```

StudentDAO.java

```java
package com.example.helloworld;

import androidx.paging.PagingSource;
import androidx.room.Dao;
import androidx.room.Insert;
import androidx.room.Query;

@Dao
public interface StudentDao {

    @Insert
    void insertStudents(Student ...students);

    @Query("DELETE FROM student_table")
    void deleteAllStudents();

    @Query("SELECT * FROM student_table")
    PagingSource<Integer,Student> getAllStudents();
}

```

StudentDatabase.java

```java
package com.example.helloworld;

import android.content.Context;

import androidx.room.Database;
import androidx.room.Room;
import androidx.room.RoomDatabase;

@Database(entities = {Student.class},version = 1,exportSchema = false)
public abstract class StudentDataBase extends RoomDatabase {
    private static volatile StudentDataBase INSTANCE;

    public static StudentDataBase getDatabase(Context context){
        if(INSTANCE==null){
            synchronized (StudentDataBase.class){
                if(INSTANCE==null){
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),StudentDataBase.class,"student_database")
                            .build();
                }
            }
        }
        return INSTANCE;
    }

    public abstract StudentDao getStudentDao();

}
```

StudentRepository.java

```java
package com.example.helloworld;

import android.content.Context;
import android.os.Handler;
import android.os.Looper;

import androidx.paging.PagingSource;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class StudentRepository {
    public StudentDataBase studentDataBase;
    public StudentDao studentDao;
    public ExecutorService executorService;
    public Handler mainHandler;

    public StudentRepository(Context context){
        studentDataBase  = StudentDataBase.getDatabase(context);
        studentDao = studentDataBase.getStudentDao();
        executorService = Executors.newSingleThreadExecutor();
        mainHandler = new Handler(Looper.getMainLooper());
    }

    public void insertStudents(OnDBOperationListener listener,Student ...students){
        executorService.execute(()->{
            studentDao.insertStudents(students);
            mainHandler.post(()->listener.success());
        });
    }

    public void deleteAllStudents(OnDBOperationListener listener){
        executorService.execute(()->{
            studentDao.deleteAllStudents();
            mainHandler.post(()->listener.success());
        });
    }

    public PagingSource<Integer,Student> getAllStudents(){
        return studentDao.getAllStudents();
    }

    public interface OnDBOperationListener{
        void success();
    }

}
```

StudentViewModel.java

```java
package com.example.helloworld;

import android.app.Application;

import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.LiveData;
import androidx.paging.Pager;
import androidx.paging.PagingConfig;
import androidx.paging.PagingData;
import androidx.paging.PagingLiveData;

public class StudentViewModel extends AndroidViewModel {
    private StudentRepository repository;

    private final Pager<Integer,Student> pager = new Pager<>(
            new PagingConfig(10,3,false),
            ()->repository.getAllStudents()
    );

    public final LiveData<PagingData<Student>> getAllStudents(){
        return PagingLiveData.getLiveData(pager);
    };

    public void insertStudents(Student ...students){
        repository.insertStudents(new StudentRepository.OnDBOperationListener() {
            @Override
            public void success() {

            }
        },students);
    }

    public void deleteAllStudents(){
        repository.deleteAllStudents(new StudentRepository.OnDBOperationListener() {
            @Override
            public void success() {

            }
        });
    }

    public StudentViewModel(@NonNull Application application) {
        super(application);
        repository = new StudentRepository(application.getApplicationContext());
    }
}
```

MyPagingAdapter.java

```java
package com.example.helloworld;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.paging.PagingDataAdapter;
import androidx.recyclerview.widget.DiffUtil;
import androidx.recyclerview.widget.RecyclerView;

public class MyPagingAdapter extends PagingDataAdapter<Student, MyPagingAdapter.MyViewHolder> {

    public MyPagingAdapter() {
        super(itemCallback);
    }

    private static final DiffUtil.ItemCallback<Student> itemCallback = new DiffUtil.ItemCallback<Student>() {
        @Override
        public boolean areItemsTheSame(@NonNull Student oldItem, @NonNull Student newItem) {
            return oldItem.getId()==newItem.getId();
        }

        @Override
        public boolean areContentsTheSame(@NonNull Student oldItem, @NonNull Student newItem) {
            return oldItem.getStudentNnumber()==newItem.getStudentNnumber();
        }
    };

    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater inflater = LayoutInflater.from(parent.getContext());
        View v = inflater.inflate(R.layout.cell_text,parent,false);
        return new MyViewHolder(v);
    }

    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, int position) {
            Student student = getItem(position);
            if(student!=null){
                holder.textView.setText(String.valueOf(student.getStudentNnumber()));
            }
    }

    static class MyViewHolder extends RecyclerView.ViewHolder{
        TextView textView;
        public MyViewHolder(@NonNull View itemView) {
            super(itemView);
            textView = itemView.findViewById(R.id.tv);
        }
    }
}
```

### 3.MainActivity里调用

```java
package com.example.helloworld;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import androidx.lifecycle.Observer;
import androidx.lifecycle.ViewModelProvider;
import androidx.navigation.NavController;
import androidx.navigation.NavHost;
import androidx.navigation.Navigation;
import androidx.navigation.fragment.NavHostFragment;
import androidx.navigation.ui.AppBarConfiguration;
import androidx.navigation.ui.NavigationUI;
import androidx.paging.PagingData;
import androidx.recyclerview.widget.DividerItemDecoration;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.google.android.material.bottomnavigation.BottomNavigationView;

public class MainActivity extends AppCompatActivity {

    StudentViewModel viewModel;
    MyPagingAdapter adapter;
    RecyclerView recyclerView;
    Button btnCreate,btnClear;

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

        btnCreate = findViewById(R.id.btn_create);
        btnClear = findViewById(R.id.btn_clear);
        recyclerView = findViewById(R.id.recyclerView);

        viewModel = new ViewModelProvider(this).get(StudentViewModel.class);
        adapter = new MyPagingAdapter();

        recyclerView.setLayoutManager(new LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false));
        recyclerView.addItemDecoration(new DividerItemDecoration(this,DividerItemDecoration.VERTICAL));
        recyclerView.setAdapter(adapter);

        viewModel.getAllStudents().observe(this, new Observer<PagingData<Student>>() {
            @Override
            public void onChanged(PagingData<Student> studentPagingData) {
                adapter.submitData(getLifecycle(),studentPagingData);
            }
        });

        btnCreate.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Student[] students = new Student[500];
                for(int i = 0;i<students.length;i++){
                    Student student = new Student();
                    student.setStudentNnumber(i);
                    students[i] = student;
                }
                viewModel.insertStudents(students);
            }
        });

        btnClear.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                viewModel.deleteAllStudents();
            }
        });


    }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.constraintlayout.widget.Guideline
        android:id="@+id/gl90"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.9" />

    <Button
        android:id="@+id/btn_create"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:text="生成"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@id/btn_clear"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="@id/gl90" />

    <Button
        android:id="@+id/btn_clear"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="20dp"
        android:layout_marginEnd="20dp"
        android:text="清空"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@id/btn_create"
        app:layout_constraintTop_toTopOf="@id/gl90" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="@id/gl90"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:id="@+id/cell_text"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"
        android:text="hello"
        android:textSize="22sp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

![image-20260413183336556](..\imgs\image-20260413183336556.png)
