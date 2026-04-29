### 1.Student实现Parcelable，并且实现方法

### 2.通过intent传递对象

intent可以跨进程传递对象，第二种传递对象的方式是实现一个MyAppliaction的子类，然后写一个全局变量，这样就可以共享变量了。但是这种方法有一个缺陷，就是不能够跨进程，因为不同进程的MyApplication不是同一个文件了。

MainActivity.java

```java
package com.example.helloworld;

import android.content.Intent;
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
    Button btnSave;

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
                Bundle bundle =  new Bundle();
                bundle.putParcelable("data",student);
                Intent intent = new Intent(MainActivity.this, MainActivity2.class);
                intent.putExtra("my_bundle",bundle);
                startActivity(intent);
                tvName.getText().clear();
                tvAge.getText().clear();
                tvMath.getText().clear();
                tvEnglish.getText().clear();
                tvChinese.getText().clear();
                tvGrade.setText("-");
            }
        });

    }
}
```

MainActivity2.java

```java
package com.example.helloworld;

import android.content.Intent;
import android.os.Bundle;
import android.widget.EditText;
import android.widget.TextView;

import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;

public class MainActivity2 extends AppCompatActivity {
    EditText tvName,tvAge,tvMath,tvEnglish,tvChinese;
    TextView tvGrade;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        EdgeToEdge.enable(this);
        setContentView(R.layout.activity_main2);
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);
            return insets;
        });
        Intent intent = getIntent();
        Bundle bundle = intent.getBundleExtra("my_bundle");
        Student student = bundle.getParcelable("data");

        tvName = findViewById(R.id.et_name);
        tvAge = findViewById(R.id.et_age);
        tvMath = findViewById(R.id.et_math);
        tvEnglish = findViewById(R.id.et_english);
        tvChinese = findViewById(R.id.et_chinese);
        tvGrade = findViewById(R.id.tv_grade);

        tvName.setText(student.getName());
        tvAge.setText(String.valueOf(student.getAge()));
        tvMath.setText(String.valueOf(student.getScore().getMath()));
        tvEnglish.setText(String.valueOf(student.getScore().getEnglish()));
        tvChinese.setText(String.valueOf(student.getScore().getChinese()));
        tvGrade.setText(student.getGrade());

    }
}
```

