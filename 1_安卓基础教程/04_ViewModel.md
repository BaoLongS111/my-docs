### ViewModel是JetPack 2018年推出的，是一个使UI和数据分离的控制器，具体使用方法如下：

1.先创建一个ViewModel的子类，然后里面写上我们要控制的数据

```java
public class MyViewModel extends ViewModel{
    private int nubmer = 0;
    public String getNumber(){
        return String.valueOf(this.number);
    }
    public void setNumber(int number){
        this.number = number;
    }
}
```

2. 在MainActivity里通过ViewModelProvider创建MyViewModel的实例

```java
MyViewModel myViewModel;

ViewModelProvider.AndroidViewModelFactory instance = ViewModelProvider.AndroidViewModelFactory.getInstance(getApplication());
myViewModel = new ViewModelProvider(this,instance).get(MyViewModel.class);
```

