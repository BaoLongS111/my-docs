### 在不同Fragment中，请求回来的ViewModel其实是同一个对象，这就是SingleTon的含义，故能共享数据。



![image-20260403190517550](..\imgs\image-20260403190517550.png)

MyViewModel.class
```java
package com.example.helloworld;

import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;

public class MyViewModel extends ViewModel {
    private MutableLiveData<Integer> number;
    public MutableLiveData<Integer> getNumber(){
        if(number==null){
            number = new MutableLiveData<>();
            number.setValue(0);
        }
        return number;
    }

    public void Add(int x){
        number.setValue(getNumber().getValue()+x);
        if(number.getValue()<=0){
            number.setValue(0);
        }
    }
}

```

MasterFragment.class
```java
package com.example.helloworld;

import android.os.Bundle;

import androidx.databinding.DataBindingUtil;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProvider;
import androidx.navigation.NavController;
import androidx.navigation.NavHost;
import androidx.navigation.Navigation;
import androidx.navigation.fragment.NavHostFragment;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.SeekBar;

import com.example.helloworld.databinding.FragmentMasterBinding;

public class MasterFragment extends Fragment {

    private MyViewModel myViewModel;
    private FragmentMasterBinding binding;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        myViewModel = new ViewModelProvider(getActivity()).get(MyViewModel.class);
        binding = DataBindingUtil.inflate(inflater,R.layout.fragment_master,container,false);

        binding.sb.setProgress(myViewModel.getNumber().getValue());

        binding.btnConfirm.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                NavController controller = Navigation.findNavController(view);
                controller.navigate(R.id.action_masterFragment_to_detailFragment);
            }
        });

        binding.sb.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int i, boolean b) {
                myViewModel.getNumber().setValue(i);
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {

            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {

            }
        });

        binding.setData(myViewModel);
        binding.setLifecycleOwner(getActivity());

        return binding.getRoot();

        // Inflate the layout for this fragment
//        return inflater.inflate(R.layout.fragment_master, container, false);
    }
}
```

DetailFragment.class
```java
package com.example.helloworld;

import android.os.Bundle;

import androidx.databinding.DataBindingUtil;
import androidx.fragment.app.Fragment;
import androidx.lifecycle.ViewModelProvider;
import androidx.navigation.Navigation;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.example.helloworld.databinding.FragmentDetailBinding;

public class DetailFragment extends Fragment {

    private MyViewModel myViewModel;

    private FragmentDetailBinding binding;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        myViewModel = new ViewModelProvider(getActivity()).get(MyViewModel.class);
        binding = DataBindingUtil.inflate(inflater,R.layout.fragment_detail,container,false);

        binding.btnConfirm.setOnClickListener(Navigation.createNavigateOnClickListener(R.id.action_detailFragment_to_masterFragment));
        // Inflate the layout for this fragment
//        return inflater.inflate(R.layout.fragment_detail, container, false);

        binding.setData(myViewModel);
        binding.setLifecycleOwner(getActivity());
        return binding.getRoot();
    }
}
```

