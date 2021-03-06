# CircleShape渐变颜色圆环
## 设计思路
通过自定义控件实现。将整个圆环拆分成一个个的小圆弧，每个小圆弧画笔的色值不一样，每个圆弧画笔的色值都是起始色值和终止色值的中间过渡色，由起始色值逐渐向终止色值靠拢，最后形成渐变颜色的圆环。
## 知识点
#### 1、自定义控件
自定义控件分为三种：继承控件，即在现有控件直接简单修改；组合控件，即GroupView；自定义View。具体自定义控件的细节，这里不再细说，可以参阅网上资料。

#### 2、画笔
画笔类型（setStyle）分为三种：STROKE只绘制图形轮廓，FILL只绘制图形内容，FILL_AND_STROKE既绘制轮廓也绘制内容；我们采用的是STROKE，也就是描边。

笔刷样式（setStrokeCap）分为三种：Round圆形冒；SQUARE方形冒；BUTT无冒。这里需要注意，冒的意思就是多出一部分，所以我们采用的是BUTT，下面会详细说明。

画笔宽度（setStrokeWidth）

#### 3、画图
RectF：RectF有四个参数(float left, float top, float right, float bottom)

<img src="app/screenshots/Rect.jpg" width="50%" height="50%" />

drawArc：画弧，主要关注startAngle、sweepAngle两个参数

<img src="app/screenshots/drawArc.jpg" width="50%" height="50%" />

## 实现
（1）onMeasure设定画大小

```   
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
     super.onMeasure(widthMeasureSpec, heightMeasureSpec);
     mMeasureWidth = getMeasuredWidth()
     mMeasureHeight = getMeasuredHeight();
     if (pRectF == null) {
         float halfProgressWidth = progressWidth / 2;
         pRectF = new RectF(halfProgressWidth + getPaddingLeft(),
                    halfProgressWidth + getPaddingTop(),
                    mMeasureWidth - halfProgressWidth - getPaddingRight(),
                    mMeasureHeight - halfProgressWidth - getPaddingBottom());
        }
    }
 ```

（2）onDraw确定怎么画，这里主要看一下drawProgress，整个圆弧由多个小圆弧组成，通过获取过渡色设置笔刷颜色，笔刷颜色从起始色值开始，每增加一度都向终止色值渐变

```    
private void drawProgress(Canvas canvas) {
     for (int i = 0, end = (int) (curProgress * unitAngle); i < end; i++) {
         progressPaint.setColor(getGradient(i / (float) end, progressStartColor, progressEndColor));
         canvas.drawArc(pRectF,
                 startAngle + i,
                 2,
                 false,
                 progressPaint);
        }
    }
```
    
（3）获取过渡色的方法

```    
public int getGradient(float fraction, int startColor, int endColor) {
     if (fraction > 1) fraction = 1;
     int alphaStart = Color.alpha(startColor);
     int redStart = Color.red(startColor);      
     int blueStart = Color.blue(startColor);
     int greenStart = Color.green(startColor);
     int alphaEnd = Color.alpha(endColor);
     int redEnd = Color.red(endColor);      
     int blueEnd = Color.blue(endColor);
     int greenEnd = Color.green(endColor);
     int alphaDifference = alphaEnd - alphaStart;
     int redDifference = redEnd - redStart;
     int blueDifference = blueEnd - blueStart;
     int greenDifference = greenEnd - greenStart;
     int alphaCurrent = (int) (alphaStart + fraction * alphaDifference);
     int redCurrent = (int) (redStart + fraction * redDifference);
     int blueCurrent = (int) (blueStart + fraction * blueDifference);
     int greenCurrent = (int) (greenStart + fraction * greenDifference);
     return Color.argb(alphaCurrent, redCurrent, greenCurrent, blueCurrent);
    }
 ```

（4）问题解决  

画圆环的过程中会遇到一个问题，上面提到的笔刷样式的选择，下面是三种笔刷的示意图：

Paint.Cap.ROUND 头尾多出了一块圆形笔帽

<img src="app/screenshots/StrokeCap_Round.png" width="50%" height="50%" />

Paint.Cap.SQUARE 头尾多出了一块方形笔帽

<img src="app/screenshots/StrokeCap_SQUARE.png" width="50%" height="50%" />

Paint.Cap.BUTT 头尾不多出笔帽，但是每个小圆环中间有空隙

<img src="app/screenshots/StrokeCap_BUTT.png" width="50%" height="50%" />

解决方案：
sweepAngle比下一次的startAngle多一点。

<img src="app/screenshots/StrokeCap_NORMAL.png" width="50%" height="50%" />

（5）后续拓展
通过showAnim来控制圆环动效

## 使用
1、在layout中设置width、height、画笔宽度
  
```
<com.component.circleshape.CircleShape
    android:id="@+id/circle_shape"
    android:layout_width="320dp"
    android:layout_height="320dp"
    android:layout_gravity="center"
    a:pr_progress_width="30dp" />
```

2、在类中设置progress、背景色、进度起始色、进度终止色

```        
CircleShape mPR = findViewById(R.id.circle_shape);
mPR.setProgress(60);
mPR.setColor(R.color.bgColor,R.color.progressStartColor, R.color.progressEndColor);
```
