![image-20260413182758951](..\imgs\image-20260413182758951.png)

![image-20260413182830534](..\imgs\image-20260413182830534.png)

![image-20260413182901982](..\imgs\image-20260413182901982.png)

![image-20260413183120120](..\imgs\image-20260413183120120.png)

### 1.创建Student，一定要实现Serializable

```java
package com.example.helloworld;

import java.io.Serializable;

public class Student implements Serializable {
    private String name;
    private int age;
    private Score score;
    private String grade;

    public Student(String name, int age, Score score) {
        this.name = name;
        this.age = age;
        this.score = score;
        if(score.getMath()>=90&&score.getEnglish()>=90&&score.getChinese()>=90){
            this.grade = "A";
        }else if(score.getMath()>=80&&score.getEnglish()>=80&&score.getChinese()>=80){
            this.grade="B";
        }else{
            this.grade = "C";
        }
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getGrade() {
        return grade;
    }

    public void setGrade(String grade) {
        this.grade = grade;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

class Score implements Serializable{
    private int math;
    private int english;
    private int chinese;

    public Score(int math, int english, int chinese) {
        this.math = math;
        this.english = english;
        this.chinese = chinese;
    }

    public int getMath() {
        return math;
    }

    public void setMath(int math) {
        this.math = math;
    }

    public int getEnglish() {
        return english;
    }

    public void setEnglish(int english) {
        this.english = english;
    }

    public int getChinese() {
        return chinese;
    }

    public void setChinese(int chinese) {
        this.chinese = chinese;
    }
}
```

### 2.在MainActivity里实现传递对象

```java
package com.example.helloworld;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

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

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.OutputStream;

public class MainActivity extends AppCompatActivity {

    EditText tvName,tvAge,tvMath,tvEnglish,tvChinese;
    TextView tvGrade;
    Button btnSave,btnLoad;

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

        tvName = findViewById(R.id.et_name);
        tvAge = findViewById(R.id.et_age);
        tvMath = findViewById(R.id.et_math);
        tvEnglish = findViewById(R.id.et_english);
        tvChinese = findViewById(R.id.et_chinese);
        tvGrade = findViewById(R.id.tv_grade);
        btnSave = findViewById(R.id.btn_save);
        btnLoad = findViewById(R.id.btn_load);

        btnSave.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                int math = Integer.parseInt(tvMath.getText().toString());
                int english = Integer.parseInt(tvEnglish.getText().toString());
                int chinese = Integer.parseInt(tvChinese.getText().toString());
                String name = tvName.getText().toString();
                int age = Integer.parseInt(tvAge.getText().toString());
                Score score = new Score(math,english,chinese);
                Student student = new Student(name,age,score);
                try {
                    ObjectOutputStream outputStream = new ObjectOutputStream(openFileOutput("myfile.data",MODE_PRIVATE));
                    outputStream.writeObject(student);
                    outputStream.flush();
                    outputStream.close();
                    Toast.makeText(MainActivity.this, "data saved!", Toast.LENGTH_SHORT).show();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }

                tvName.setText("");
                tvAge.setText("");
                tvMath.setText("");
                tvEnglish.setText("");
                tvChinese.setText("");
                tvGrade.setText("");
            }
        });

        btnLoad.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                try {
                    ObjectInputStream objectInputStream = new ObjectInputStream(openFileInput("myfile.data"));
                    Student student = (Student) objectInputStream.readObject();
                    tvName.setText(student.getName());
                    tvAge.setText(String.valueOf(student.getAge()));
                    tvMath.setText(String.valueOf(student.getScore().getMath()));
                    tvEnglish.setText(String.valueOf(student.getScore().getEnglish()));
                    tvChinese.setText(String.valueOf(student.getScore().getChinese()));
                    tvGrade.setText(student.getGrade());
                    objectInputStream.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                } catch (ClassNotFoundException e) {
                    throw new RuntimeException(e);
                }
            }
        });

    }
}
```



![image-20260414153828231](..\imgs\image-20260414153828231.png)