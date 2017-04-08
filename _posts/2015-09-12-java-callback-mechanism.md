---
layout: post
title: 一个经典例子让你彻彻底底理解java回调机制
categories: java
tags: callback
---

以前不理解什么叫回调，天天听人家说加一个回调方法啥的，心里想我草，什么叫回调方法啊？然后自己就在网上找啊找啊找，找了很多也不是很明白，现在知道了，所谓回调：就是A类中调用B类中的某个方法C，然后B类中反过来调用A类中的方法D，D这个方法就叫回调方法，这样子说你是不是有点晕晕的，其实我刚开始也是这样不理解，看了人家说比较经典的回调方式：

- Class A实现接口CallBack callback——背景1
- class A中包含一个class B的引用b ——背景2
- class B有一个参数为callback的方法f(CallBack callback) ——背景3
- A的对象a调用B的方法 f(CallBack callback) ——A类调用B类的某个方法 C
- 然后b就可以在f(CallBack callback)方法中调用A的方法 ——B类调用A类的某个方法D

<!-- more -->

大家都喜欢用打电话的例子，好吧，为了跟上时代，我也用这个例子好了，我这个例子采用异步加回调
有一天小王遇到一个很难的问题，问题是“1 + 1 = ?”，就打电话问小李，小李一下子也不知道，就跟小王说，等我办完手上的事情，就去想想答案，小王也不会傻傻的拿着电话去等小李的答案吧，于是小王就对小李说，我还要去逛街，你知道了答案就打我电话告诉我，于是挂了电话，自己办自己的事情，过了一个小时，小李打了小王的电话，告诉他答案是2

```java
/**
 * 这是一个回调接口
 * @author xiaanming
 *
 */
public interface CallBack {
	/**
	 * 这个是小李知道答案时要调用的函数告诉小王，也就是回调函数
	 * @param result 是答案
	 */
	public void solve(String result);
}
```

```java
/**
 * 这个是小王
 * @author xiaanming
 * 实现了一个回调接口CallBack，相当于----->背景一
 */
public class Wang implements CallBack {
	/**
	 * 小李对象的引用
	 * 相当于----->背景二
	 */
	private Li li; 

	/**
	 * 小王的构造方法，持有小李的引用
	 * @param li
	 */
	public Wang(Li li){
		this.li = li;
	}
	
	/**
	 * 小王通过这个方法去问小李的问题
	 * @param question  就是小王要问的问题,1 + 1 = ?
	 */
	public void askQuestion(final String question){
		//这里用一个线程就是异步，
		new Thread(new Runnable() {
			@Override
			public void run() {
				/**
				 * 小王调用小李中的方法，在这里注册回调接口
				 * 这就相当于A类调用B的方法C
				 */
				li.executeMessage(Wang.this, question); 
			}
		}).start();
		
		//小网问完问题挂掉电话就去干其他的事情了，诳街去了
		play();
	}

	public void play(){
		System.out.println("我要逛街去了");
	}

	/**
	 * 小李知道答案后调用此方法告诉小王，就是所谓的小王的回调方法
	 */
	@Override
	public void solve(String result) {
		System.out.println("小李告诉小王的答案是--->" + result);
	}
	
}
```

```java
/**
 * 这个就是小李啦
 * @author xiaanming
 *
 */
public class Li {
	/**
	 * 相当于B类有参数为CallBack callBack的f()---->背景三
	 * @param callBack  
	 * @param question  小王问的问题
	 */
	public void executeMessage(CallBack callBack, String question){
		System.out.println("小王问的问题--->" + question);
		
		//模拟小李办自己的事情需要很长时间
		for(int i=0; i<10000;i++){
			
		}
		
		/**
		 * 小李办完自己的事情之后想到了答案是2
		 */
		String result = "答案是2";
		
		/**
		 * 于是就打电话告诉小王，调用小王中的方法
		 * 这就相当于B类反过来调用A的方法D
		 */
		callBack.solve(result); 

	}
	
}
```

```java
/**
 * 测试类
 * @author xiaanming
 *
 */
public class Test {
	public static void main(String[]args){
		/**
		 * new 一个小李
		 */
		Li li = new Li();

		/**
		 * new 一个小王
		 */
		Wang wang = new Wang(li);
		
		/**
		 * 小王问小李问题
		 */
		wang.askQuestion("1 + 1 = ?");
	}
}
```

通过上面的那个例子你是不是差不多明白了回调机制呢，上面是一个异步回调，我们看看同步回调吧，onClick（）方法
现在来分析分析下Android View的点击方法onclick（）;我们知道onclick()是一个回调方法，当用户点击View就执行这个方法，我们用Button来举例好了

```java
//这个是View的一个回调接口
/**
 * Interface definition for a callback to be invoked when a view is clicked.
 */
public interface OnClickListener {
    /**
     * Called when a view has been clicked.
     *
     * @param v The view that was clicked.
     */
    void onClick(View v);
}
```
```java
package com.example.demoactivity;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;

/**
 * 这个就相当于Class A
 * @author xiaanming
 * 实现了 OnClickListener接口---->背景一
 */
public class MainActivity extends Activity implements OnClickListener{
	/**
	 * Class A 包含Class B的引用----->背景二
	 */
	private Button button;

	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		button = (Button)findViewById(R.id.button1);
		
		/**
		 * Class A 调用View的方法,而Button extends View----->A类调用B类的某个方法 C
		 */
		button.setOnClickListener(this);
	}

	/**
	 * 用户点击Button时调用的回调函数，你可以做你要做的事
	 * 这里我做的是用Toast提示OnClick
	 */
	@Override
	public void onClick(View v) {
		Toast.makeText(getApplication(), "OnClick", Toast.LENGTH_LONG).show();
	}

}
```

下面是View类的setOnClickListener方法，就相当于B类咯，只把关键代码贴出来

```java
/**
 * 这个View就相当于B类
 * @author xiaanming
 *
 */
public class View implements Drawable.Callback, KeyEvent.Callback, AccessibilityEventSource {
	/**
     * Listener used to dispatch click events.
     * This field should be made private, so it is hidden from the SDK.
     * {@hide}
     */
    protected OnClickListener mOnClickListener;
    
    /**
     * setOnClickListener()的参数是OnClickListener接口------>背景三
     * Register a callback to be invoked when this view is clicked. If this view is not
     * clickable, it becomes clickable.
     *
     * @param l The callback that will run
     *
     * @see #setClickable(boolean)
     */
    
    public void setOnClickListener(OnClickListener l) {
        if (!isClickable()) {
            setClickable(true);
        }
        mOnClickListener = l;
    }
    
    
    /**
     * Call this view's OnClickListener, if it is defined.
     *
     * @return True there was an assigned OnClickListener that was called, false
     *         otherwise is returned.
     */
    public boolean performClick() {
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        if (mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            
            //这个不就是相当于B类调用A类的某个方法D，这个D就是所谓的回调方法咯
            mOnClickListener.onClick(this);
            return true;
        }

        return false;
    }
}
```

这个例子就是Android典型的回调机制，看完这个你是不是更进一步的理解了回调机制呢？ 线程run()也是一个回调方法，当执行Thread的start（）方法就会回调这个run()方法，还有处理消息都比较经典等等

这也是小弟对回调机制的一点拙见，不懂的请留言，如果有错误希望指出！多谢！

原文链接：[http://blog.csdn.net/xiaanming/article/details/8703708](http://blog.csdn.net/xiaanming/article/details/8703708)
