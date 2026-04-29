### 1.导入依赖库

```java
dependencies {
  implementation 'com.google.code.gson:gson:2.13.2'
}
```

### 2.创建Student对象

### 3.正向序列化为json字符串

json就是javascript object notation

```java
package com.example.helloworld;

import android.os.Bundle;
import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import com.google.gson.Gson;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

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

        Gson gson = new Gson();
        List<Student> studentList = new ArrayList<>();
        Student student1 = new Student("Tom",18,new Score(99,98,70));
        Student student2 = new Student("Jack",19,new Score(90,80,80));
        studentList.add(student1);
        studentList.add(student2);
        String jsonStudents = gson.toJson(studentList);


    }
}
```

```json
[
  {
    "age": 18,
    "grade": "C",
    "name": "Tom",
    "score": {
      "chinese": 70,
      "english": 98,
      "math": 99
    }
  },
  {
    "age": 19,
    "grade": "B",
    "name": "Jack",
    "score": {
      "chinese": 80,
      "english": 80,
      "math": 90
    }
  }
]
```

### 4.反向序列化为对象

```java
package com.example.helloworld;

import android.os.Bundle;
import androidx.activity.EdgeToEdge;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.graphics.Insets;
import androidx.core.view.ViewCompat;
import androidx.core.view.WindowInsetsCompat;
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;

import java.lang.reflect.Type;
import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

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

        Gson gson = new Gson();
        List<Student> studentList = new ArrayList<>();
        String jsonStudents = "[\n" +
                "  {\n" +
                "    \"age\": 18,\n" +
                "    \"grade\": \"C\",\n" +
                "    \"name\": \"Tom\",\n" +
                "    \"score\": {\n" +
                "      \"chinese\": 70,\n" +
                "      \"english\": 98,\n" +
                "      \"math\": 99\n" +
                "    }\n" +
                "  },\n" +
                "  {\n" +
                "    \"age\": 19,\n" +
                "    \"grade\": \"B\",\n" +
                "    \"name\": \"Jack\",\n" +
                "    \"score\": {\n" +
                "      \"chinese\": 80,\n" +
                "      \"english\": 80,\n" +
                "      \"math\": 90\n" +
                "    }\n" +
                "  }\n" +
                "]";
        Type studentType = new TypeToken<List<Student>>(){}.getType();
        studentList = gson.fromJson(jsonStudents,studentType);
    }
}
```

如果把对象转换为json的时候想起个别名，就在Student实体的变量名上面做一个注解

```java
    @SerializedName("student_name")
    private String name;
```

