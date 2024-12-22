# Drawable
- 一般用于View的背景，在 View.draw()中，会通过drawBackground()绘制；该方法中通过setBackgroundBounds()设置Drawable.mBounds，然后调用Drawable.draw()
- Drawable的宽高概念：1、Drawable无具体宽高概念，其大小一般等于他的View；2、其中 getIntrinsicWidth和 getIntrinsicHeight用于获取Drawable的内部宽高，但是不一定有值，比如ColorDrawable就是返回-1；3、getBounds()用于返回View给Drawable设置的矩形，该矩形等于(0，0， view.ge tWidth() ， view.ge tHeight() )
- draw()：参数为View传递的Canvas对象，使用该对象配合mBounds进行对应的绘制。