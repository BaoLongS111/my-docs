### 1.创建一个Class然后继承EditText，选择三个参数的构造方法，然后用@JvmOverloads constructor()包裹。记得默认给attr为null，然后defaultStyle为EditTextStyle（androidx.appcompat.R.attr.editTextStyle）

```kotlin
package com.example.autoview

import android.content.Context
import android.graphics.Rect
import android.graphics.drawable.Drawable
import android.util.AttributeSet
import android.view.MotionEvent
import androidx.core.content.ContextCompat

class EditTextWitchClear @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = androidx.appcompat.R.attr.editTextStyle
) :
    androidx.appcompat.widget.AppCompatEditText(context, attrs, defStyleAttr) {

        private var iconDrawable: Drawable?=null

    init {
        context.theme.obtainStyledAttributes(attrs,R.styleable.EditTextWitchClear,0,0)
            .apply {
                try {
                    val iconID = getResourceId(R.styleable.EditTextWitchClear_clearIcon,0)
                    if(iconID!=0){
                        iconDrawable = ContextCompat.getDrawable(context,iconID)
                    }
                }finally {
                    recycle()
                }
            }
    }
    override fun onTextChanged(
        text: CharSequence?,
        start: Int,
        lengthBefore: Int,
        lengthAfter: Int
    ) {
        super.onTextChanged(text, start, lengthBefore, lengthAfter)
        toggleClearIcon()
    }

    override fun onTouchEvent(event: MotionEvent?): Boolean {
        event?.let { e->
            iconDrawable?.let {
                if( e.action == MotionEvent.ACTION_UP
                    && e.x > width - it.intrinsicWidth - 20
                    && e.x < width + 20
                    && e.y > height / 2 - it.intrinsicHeight / 2 - 20
                    && e.y < height / 2 + it.intrinsicHeight / 2 + 20
                ){
                    text?.clear()
                }
            }
        }
        performClick()
        return super.onTouchEvent(event)
    }

    override fun onFocusChanged(focused: Boolean, direction: Int, previouslyFocusedRect: Rect?) {
        super.onFocusChanged(focused, direction, previouslyFocusedRect)
        toggleClearIcon()
    }

    override fun performClick(): Boolean {
        return super.performClick()
    }

    private fun toggleClearIcon(){
        val icon = if(isFocused && text?.isNotEmpty() == true) iconDrawable else null
        setCompoundDrawablesRelativeWithIntrinsicBounds(null,null,icon,null)
    }

}
```

### 2.因为要开放一个icon出去，所以要创建attrs文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="EditTextWitchClear">
        <attr name="clearIcon" format="reference" />
    </declare-styleable>
</resources>
```

