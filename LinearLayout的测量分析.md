##LinearLayout的测量源码解析
看下面一部分代码
	
	如果屏幕宽1440，求两个textview的宽~~，很奇葩的（测试的时候发现第一个view比第二个view长,）
	 <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="20dp"
        android:orientation="horizontal"
        android:background="@color/black">
        <TextView
            android:layout_width="200dp"
            android:layout_weight="1"
            android:background="@color/white"
            android:layout_height="match_parent"/>
        <TextView
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="match_parent"/>
    </LinearLayout>
	

结果会很奇怪，在1440 的屏幕上 第一个view的宽度是 1120px，第二个是 320px;结果很奇怪,于是翻了下linearlayout的源码onmeasure方法：onmersure方法如下

	 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mOrientation == VERTICAL) {
            measureVertical(widthMeasureSpec, heightMeasureSpec);
        } else {
            measureHorizontal(widthMeasureSpec, heightMeasureSpec);
        }
    }
然后查看measureHorizontal方法里面有如下片段：

	 if (childWeight > 0) {
	        final int share = (int) (childWeight * remainingExcess / remainingWeightSum);
	        remainingExcess -= share;
	        remainingWeightSum -= childWeight;
	
	        final int childWidth;
	        if (mUseLargestChild && widthMode != MeasureSpec.EXACTLY) {
	            childWidth = largestChildWidth;
	        } else if (lp.width == 0 && (!mAllowInconsistentMeasurement
	                || widthMode == MeasureSpec.EXACTLY)) {
	            // This child needs to be laid out from scratch using
	            // only its share of excess space.
	            childWidth = share;
	        } else {
	            // This child had some intrinsic width to which we
	            // need to add its share of excess space.
	            childWidth = child.getMeasuredWidth() + share;
	        }

这里面是真正计算有weight的childview的宽的片段，childWeight 是当前child的比重，remainingExcess则是LinearLayout去除child的leftmargin，rightmargin(horizantal方向是去除这两个值)，以及LinearLayout的padding,以及width值为exactly的值，的总长度，在示例中是remainingExcess  = 1440 - 200 * 4(dpi) = 640，remainingWeightSum 是当前剩余的所有未测量的child的比重（默认值是mWeightSum,也就是LinearLayout设置的weightsum），否则就是所有child的比重值的和

	  float remainingWeightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;
	  
这时候 第一个view的lp.width == 200dp 所以这时 条件进入else语句，也就是
	
		childWidth = child.getMeasuredWidth() + share;
		
所以第一个view测量出来的值是 share(320px)+ child.getMeasuredWidth() (800 px) = 1120 px,剩下的就不做计算了


##总结：
	
	1.linearlayout中的测量以具体的width宽度值有限测量，优先顺序 width>weight
	2.如果weight和width混用，如果同时用在一个view上，先计算width的值，然后view的最终width则为 xml的widht+weight的分配
	

