> 仿支付宝chartview效果。

效果图,左边是支付宝的效果图，初始化动画、点击某一块旋转到中心线并且有弹出效果。

<img src="https://raw.githubusercontent.com/Jandzy/mypic/master/pic/GIF.gif" width="400" height="400" alt="zfb"/><img src="https://raw.githubusercontent.com/Jandzy/mypic/master/pic/GIF.gif" width="400" height="400" alt="chartview"/>

大概分为几个部

* chartview初始化旋转动画
* 点击不同区域旋转
* 点击区域弹出

思路：

* 自定义属性
* 绘制不同比例颜色的圆弧
* 添加加载动画
* 添加不同区域的点击事件
* 添加点击动画
* 添加点击弹出效果

思路大概就是这些。

## 自定义属性

自定义view的一般先看一下需要那些属性，自定义属性；重写onmesure方法；重写ondraw方法、、、

根据这个chartview自定义了三个属性，

````
<resources>
    <declare-styleable name="ChartView">
        <attr name="radius" format="dimension" />
        <attr name="pieModel">
            <enum name="chart" value="0"/>
            <enum name="arc" value="1"/>
        </attr>
        <attr name="showAnim" format="boolean" />
    </declare-styleable>
</resources>
````

radio是圆的半径，pieModel是实心圆还是空心圆，showAnim是是否显示加载动画。

自定义属性完后，在构造方法中获取属性值，代码如下

````
TypedArray typedArray = context.obtainStyledAttributes(attrs,R.styleable.ChartView);
mRadius = typedArray.getDimension(R.styleable.ChartView_radius, 40);
mPieModel = typedArray.getInt(R.styleable.ChartView_pieModel, 0);
mShowAnim = typedArray.getBoolean(R.styleable.ChartView_showAnim, false);
typedArray.recycle();
````

到此自定义属性基本结束。
## 绘制不同比例的圆弧

> 一般来说，自定义完属性后，需要重写onMeasuer方法测量view的宽和高，主要是widhth size和model，model又有三种。测量后重写ondraw绘制view。

绘制的时候需要先获得不同的数据占圆的比例，所以通过setValues方法来设置值，因为参数个数不确定所以用不定参数的方法，获得一个数组值。代码如下：

````
    public void setValues(float... values) {
        double sum = 0;
        mValues = new float[values.length];
        for (float value : values) {
            sum += value;
        }
        mAnimCount = values.length;
        float angleSum = 0;
        for (int i = 0; i < values.length; i++) {
            if (i == values.length - 1) {
                mValues[i] = 360 - angleSum;
            } else {
                mValues[i] = (float) (values[i] / sum) * 360;
                angleSum += mValues[i];
            }
        }
     ｝
````

这里，直接将值转换为所占圆圈（360）的角度值，然后按照0-360分段存放到数组mvalues中。因为每块对应的颜色也不一样，添加setColor方法设置画笔颜色值：

````
    public void setColors(int... colors) {
        mColors = colors;
    }
````


设置完画笔颜色和模块值后就可以带用画笔画对应的圆弧即可，如果是空心的话（其实是在弧形圆心上面画一个白色的内置圆即可）

canvas.drawArc(voal, startPosition, mValues[i], true, mPaint);


其中，voal参数是圆距左边、上边、右边（左边+直径）、下边（上边+直径）；startPostition是开始绘制的位置，其中三点中方向是0度，顺时针，所以十二点钟方向是270度，可以根据自身需求改变绘制的开始位置；mValues[i]数组是模块的占比度，true代表实现，mPaint画笔，设置画笔颜色什么的。这样基本的饼状图基本就有原型了。

> 注意，startPostion等一些变量初始化的时候不要在ondraw方法中，因为在activity生命周期变化的时候会调用ondraw方法（不如，acitivy在onstop，onrestart回到当前acitivy后，view的构造方法不会执行，当时view的ondraw方法会执行）。也算是一个意外收获。

## 添加加载动画

这里用的加载动画的原理是逐步绘制view，用一个新的数组大小为mValues来存放模块存放的占比，然后将mValues数组清空，逐步添加mVaulues的值，调用invalidate()方法强制重绘，直至完成。代码具体实现：

````
   if (mShowAnim) {
         count = 0;
         animValues = mValues;
         mValues = new float[mValues.length];
   }
````

修改setVaules()方法，判断是否添加加载动画，如果添加的话，用一个新的数组animVaules存放mVauels占比度，然后清空mVaues保持数组大小不变，count代表的是当前绘制的模块。在ondraw方法中判断当前绘制的第几个模块，如果count小于animValues数组长度时，判断当前模块已经绘制的大小，如果当前绘制的大小加上10（绘制增量，变大可以加快绘制速度）大于模块占比度，那么count++，当前绘制的模块等于模块占比度，强制重绘；如果当前绘制的大小加上10小于，绘制的大小加10，强制重绘；直至绘制完成。代码如下：

````
       /**
         * 为了动画效果，需要先确定mValues的length，然后逐个赋值，先给mValues[0]赋      值，如果mValues[0] 等于对应的百分比之后
         * 再给mValues[1]赋值，以此类推。
         */
        if(mShowAnim) {
            if (count <= mAnimCount - 1) {
                if (mValues[count] + 10 > animValues[count]) {
                    mValues[count] = animValues[count];
                    invalidate();
                    count++;
                } else {
                    if (mValues[count] < animValues[count]) {
                        mValues[count] += 10;
                        invalidate();
                    }
                }
            }
        }
````

这样加载动画就有啦。
## 添加不同模块的点击事件

点击事件原理主要是根据当前点击点的位置判断在那一个象限，算出角度值，判断在那个模块。重写onTouchEvent（）方法，以view圆心为中心，将view分成四个象限，然后，判断当前点击点的位置，判断点击点是否在圆弧的范围之内，这样就很容易的判断出点击了那个模块。代码如下：

````
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:

                select = -1;

                float x = event.getX();
                float y = event.getY();
                touchAngle = 0;
                //第一象限
                if (x > getWidth() / 2 && x <= getWidth() / 2 + mRadius && y < getHeight() / 2 && y >= getHeight() / 2 - mRadius) {
                    touchAngle = (float) (Math.atan((x - getWidth() / 2) / (getHeight() / 2 - y)) / Math.PI * 180);
                    quadrant = 1;
                }
                //第二象限
                if (x > getWidth() / 2 && x <= getWidth() / 2 + mRadius && y > getHeight() / 2 && y <= getHeight() / 2 + mRadius) {
                    touchAngle = 90 + (float) (Math.atan((y - getHeight() / 2) / (x - getWidth() / 2)) / Math.PI * 180);
                    quadrant = 2;
                }
                //第三象限
                if (x < getWidth() / 2 && x >= getWidth() / 2 - mRadius && y > getHeight() / 2 && y <= getHeight() / 2 + mRadius) {
                    touchAngle = 180 + (float) (Math.atan((getWidth() / 2 - x) / (y - getHeight() / 2)) / Math.PI * 180);
                    quadrant = 3;
                }
                //第四象限
                if (x < getWidth() / 2 && x >= getWidth() / 2 - mRadius && y < getHeight() / 2 && y >= getHeight() / 2 - mRadius) {
                    touchAngle = 270 + (float) (Math.atan((getHeight() / 2 - y) / (getWidth() / 2 - x)) / Math.PI * 180);
                    quadrant = 4;
                }


                for (int i = 0; i < angles.length; i++) {
                    if (i == 0) {
                        if (0 < touchAngle && touchAngle < angles[i]) {
                            select = i;
                        }
                    }
                    if (i > 0) {
                        if (touchAngle > angles[i - 1] && touchAngle < angles[i]) {
                            select = i;
                        }
                    }
                }
                if (select != -1) {
                    //动画
                    end = 180 - midlines[select];
                    startAnimot(start, end);
                }

                break;
        }
        return super.onTouchEvent(event);
    }
````

其中angles数组存放的是0-360的值，根据这个值判断点击的模块。这样就能够判断出点击的位置。

## 添加弹出效果

确定啦点击那个模块后就很容易实现弹出效果。弹出效果的原理是先算出当前模块的角平分线位置，然后绘制当前弧形的圆心时让当前圆心沿角平分线移动一小段距离。在setValues方法中，将每个模块的角平分线位置存放到lines数组中，这样根据点击选择的位置即可知道要移动的位置。

````
if (moingSelect == i) {
                float outX = 0;
                float outY = 0;
                switch (quadrant) {
                    case 1:
                        outX = (float) Math.sin(Math.PI * midlines[i] / 180) * outRadious;
                        outY = (float) Math.cos(Math.PI * midlines[i] / 180) * outRadious;

                        voal.left = getWidth() / 2 - mRadius + outX;
                        voal.top = getHeight() / 2 - mRadius - outY;
                        voal.right = getWidth() / 2 + mRadius + outX;
                        voal.bottom = getHeight() / 2 + mRadius - outY;

                        break;
                    case 2:
                        outX = (float) Math.cos(Math.PI * (midlines[i] - 90 )/ 180) * outRadious;
                        outY = (float) Math.sin(Math.PI * (midlines[i] - 90) / 180) * outRadious;

                        voal.left = getWidth() / 2 - mRadius + outX;
                        voal.top = getHeight() / 2 - mRadius + outY;
                        voal.right = getWidth() / 2 + mRadius + outX;
                        voal.bottom = getHeight() / 2 + mRadius + outY;

                        break;
                    case 3:
                        outX = (float) Math.sin(Math.PI * (midlines[i] - 180 ) / 180) * outRadious;
                        outY = (float) Math.cos(Math.PI * (midlines[i] - 180 )/ 180) * outRadious;

                        voal.left = getWidth() / 2 - mRadius - outX;
                        voal.top = getHeight() / 2 - mRadius + outY;
                        voal.right = getWidth() / 2 + mRadius - outX;
                        voal.bottom = getHeight() / 2 + mRadius + outY;

                        break;
                    case 4:
                        outX = (float) Math.cos(Math.PI * (midlines[i] - 270 )/ 180) * outRadious;
                        outY = (float) Math.sin(Math.PI * (midlines[i] - 270 ) / 180) * outRadious;

                        voal.left = getWidth() / 2 - mRadius - outX;
                        voal.top = getHeight() / 2 - mRadius - outY;
                        voal.right = getWidth() / 2 + mRadius - outX;
                        voal.bottom = getHeight() / 2 + mRadius - outY;
                        break;
                }
            } else {
                voal.left = getWidth() / 2 - mRadius;
                voal.top = getHeight() / 2 - mRadius;
                voal.right = getWidth() / 2 + mRadius;
                voal.bottom = getHeight() / 2 + mRadius;
            }

            if (i != 0) {
                startPosition += mValues[i - 1];
            }
            canvas.drawArc(voal, startPosition, mValues[i], true, mPaint);
````

## 添加旋转效果

旋转动换是以当前view为圆心，十二点方向为度，让被点击的模块的中线旋转到180度的位置，每次旋转后，view位置变化，但是0度和180度相对位置不变，所以只要改变绘制的其实位置startPostion的位置即可。实现Animator.AnimatorListener接口。

````
    /**
     * desc view旋转后，原来的坐标系不变
     *
     * @param start
     * @param end
     */
    private void startAnimot(float start, float end) {
        ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(this, "rotation", start, end);
        objectAnimator.setDuration(1000);
        objectAnimator.addListener(this);
        objectAnimator.start();
    }
````

为了实现在旋转过程中点击别的模块的动画和旋转时不弹出效果，需要在onAnimationStart，onAnimationCancle方法中做处理

````
    public void onAnimationStart(Animator animation) {
        moingSelect = -1;
        invalidate();
    }

    /**
     * 旋转是整个view旋转，旋转过后，点击时的象限位置不变，中线位置不变，起始位置变啦
     *
     * @param animation
     */
    @Override
    public void onAnimationEnd(Animator animation) {
        moingSelect = select;
        start = 180 - midlines[select];
        invalidate();
    }

    @Override
    public void onAnimationCancel(Animator animation) {
        moingSelect = -1;
        start = 180 - midlines[select];
        invalidate();
    }
````

## 总结

大致思路就是上面这些，有些算的说的比较模糊，可以看[源码](https://github.com/Jandzy/ChartView "chartview")一起学习。

# bug

* demo在4.4上面正常，在5.0上面加载动画很乱，不知道为什么，还需要在看看。