---
layout: post
title: 事件体系
date: 2017-04-16
tags: android    
---
> ACTION_CANCEL	事件被上层拦截时触发  

**Activity.dispatchTouchEvent**  
```
/**
     * Called when a touch screen event was not handled by any of the views
     * under it.  This is most useful to process touch events that happen
     * outside of your window bounds, where there is no view to receive it.
     *
     * @param event The touch screen event being processed.
     *
     * @return Return true if you have consumed the event, false if you haven't.
     * The default implementation always returns false.
     */
    public boolean onTouchEvent(MotionEvent event) {
        if (mWindow.shouldCloseOnTouch(this, event)) {
            finish();
            return true;
        }
        return false;
    }
```
**Activity.dispatchTouchEvent**  
```
public boolean dispatchTouchEvent(MotionEvent ev){
  if (ev.getAction==MotionEvent.ACTION_DOWN){
    onUserInteraction();
  }
  if (getWindow().superDispatchTouchEvent(ev)){
    return true;
  }
  return onTouchEvent(ev);
}
```
由上可以看出当getWindow().superDispatchTouchEvent(ev)返回false时，会调用activity的onTouchEvent
Activity.onUserInteraction 这个方法会在此Activity在栈顶时，触摸HOME、BACK、MENU键都会回调此方法。而下拉状态栏和旋转屏幕、锁屏也会回调此方法。

getWindow()获取的是一个Window对象，而我们知道PhoneWindow是Window类的唯一实现类。那我们看看PhoneWindow.superDispatchTouchEvent的源码
PhoneWindow.superDispatchTouchEvent
```
@Override
public boolean superDispatchTouchEvent(MotionEvent event){
  return mDecor.superDispatchTouchEvent(event);
}
```
DecorView实例对象mDecor是继承自FrameLayout，而FrameLayout是继承自ViewGroup，所以super.dispatchTouchEvent其实就是ViewGroup的dispatchTouchEvent方法。

**ViewGroup.dispatchTouchEvent**  
```
public boolean dispatchTouchEvent(MotionEvent ev){
 	 ......
       final int action = ev.getAction();
       	    final int actionMasked = action & MotionEvent.ACTION_MASK;
            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                intercepted = true;
            }
......
final View[] children = mChildren;
for (int i = childrenCount - 1; i >= 0; i--) {
    final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
    final View child = getAndVerifyPreorderedView(preorderedList, children, childIndex);
  ......
    if (!canViewReceivePointerEvents(child)
            || !isTransformedTouchPointInView(x, y, child, null)) {
        ev.setTargetAccessibilityFocus(false);
        continue;
    }
   ......
    resetCancelNextUpFlag(child);
    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
       ......
        mLastTouchDownX = ev.getX();
        mLastTouchDownY = ev.getY();
        newTouchTarget = addTouchTarget(child, idBitsToAssign);
        alreadyDispatchedToNewTouchTarget = true;
        break;
    }
}
......
if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
            。。。。。。。。。
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }
   ......
      return handled;
}

private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;
       ......
        final int oldAction = event.getAction();
        if (cancel || oldAction == MotionEvent.ACTION_CANCEL) {
            event.setAction(MotionEvent.ACTION_CANCEL);
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
            event.setAction(oldAction);
            return handled;
        }
......
return handled;
}

private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
        final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
 }
```
mFirstTouchTarget是dispatchTouchEvent返回true的子view；
当viewGroup不拦截down，down由后续事件时（down时，resetTouchState会重置状态，requestDisallow无用），requestDisallow才能起作用。但若一开始down就被viewGroup拦截了，requestDisallow就没用了。
当不是Action_Down的时候，如果之前viewgroup拦截了，那么直接交给他的onTouchEvent处理。如果没有拦截，那么接下来，需要先判断是否由子viewrequestDisallowInterceptTouchEvent了，如果是，就交给子view的dispatchTouchEvent。如果不是，再判断onInterceptTouchEvent， 如果onInterceptTouchEvent返回true，则判断是否有onTouchListener（有这个就不走onTouchEvent，onClickListener在onTouchEvent中）；如果返回false，则遍历子view的dispatchTouchEvent
遍历所有的子元素时，会先判断子元素是否能接收到点击事件（1.子元素是否在播放动画；2.Down的坐标是否落在子元素的区域内），如果满足这两个条件就会把事件传递给它处理。然后会调用dispatchTransformedTouchEvent方法，其实就是 child.dispatchTouchEvent，如果返回true，那么mFirstTouchTarget就会被赋值，并终止对子元素的遍历。
如果遍历所有的子元素后，事件都没有被处理，也就是if (mFirstTouchTarget == null)的情况，原因一般有2中情况，1：viewGroup没有子元素；2：子元素在dispatchTouchEvent中返回了false，这一般时因为在onTouchEvent中返回了false。当mFirstTouchTarget时，会调用viewGroup的 super.dispatchTouchEvent(event)，也就是当无view可用时，把viewGroup当view来使用。这也是为何当所有子view的onTouchEvent返回false时，会调用viewGroup的onTouchEvent。如果此时viewGroup.onTouchEvent返回true，那对于上一级viewGroup来说，这个viewGroup就是mFirstTouchTarget 

--------------------------------------------------------------------------------
**View的dispatchTouchEvent方法：**   
```
public boolean dispatchTouchEvent(MotionEvent event) {
        ......
        boolean result = false;
        ......
        if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
        ......
        return result;
    }
```

**View的onTouchEvent方法：**  
```
public boolean onTouchEvent(MotionEvent event) {
        final float x = event.getX();
        final float y = event.getY();
        final int viewFlags = mViewFlags;
        final int action = event.getAction();
        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE
                    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
        }
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
                (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
            switch (action) {
                case MotionEvent.ACTION_UP:
                    ......
      performClick();
      ......
                    break;
                case MotionEvent.ACTION_DOWN:
                    ......
                    break;
                case MotionEvent.ACTION_CANCEL:
                    ......
                    break;
                case MotionEvent.ACTION_MOVE:
                    ......
                    break;
            }
            return true;
        }
        return false;
    }
```
当View不可用时，OnTouchListener中不会被调用。onTouchEvent会被调用，但不会执行performClick。此时onTouchEvent的返回值由该View能不能点击（包括长按和短按等点击状态）来决定。也就是说view不可用时，onTouch,onClick无用，差别只是在于返回值，即事件是否被消费，返回值由view是否可点击决定。

当view可用时，若OnTouchListener.onTouch()不为空，并返回false，则执行onTouchEvent。在onTouchEvent中，若设置了TouchDelegate，其处理逻辑与OnTouchListener类似。此时，若view可点击，则返回true消费事件，并在up时，执行performClick。若不可点击，performClick不执行，返回false不消费事件。
View的LONG_CLICKABLE默认为false；可点击的view，CLICKABLE默认为true；不可点击的view默认为false。setOnClickListener / setOnLongClickListener会自动把CLICKABLE / LONG_CLICKABLE置为true。
---------------
MOVE - UP
当dispatchTouchEvent在进行事件分发的时候，只有ACTION_DOWN返回true，才会收到ACTION_MOVE和ACTION_UP的事件。
如果ACTION_DOWN事件是在dispatchTouchEvent消费，那么消费ACTION_DOWN 的函数dispatchTouchEvent 也能收到 ACTION_MOVE和ACTION_UP。
如果ACTION_DOWN事件是在onTouchEvent消费的，那么会把ACTION_MOVE或ACTION_UP事件传给该控件的onTouchEvent处理并结束传递。

--------------------------
例子：
ViewGroupA  --> ViewGroupB  --> ViewC
假设B是一个scrolling view，没有拦截DOWN事件，但它拦截了接下来的MOVE事件。

下面是事件被处理的顺序：  
1. DOWN事件被依次传到A和B的onInterceptTouchEvent方法中，它们都返回的false。
2. DOWN事件传递到C的onTouchEvent方法，返回了true。
3. 在后续到来MOVE事件时，A的onInterceptTouchEvent方法仍然返回false。
4. B的onInterceptTouchEvent方法收到了该MOVE事件，此时B注意到用户手指移动距离已经超过了一定的threshold（slop）。因此，B的onInterceptTouchEvent方法决定返回true，从而接管该手势（gesture）后续的处理。
然后，这个MOVE事件将会被系统变成一个CANCEL事件，这个CANCEL事件将会传递给C的onTouchEvent方法。
5. 现在，又来了一个MOVE事件，它被传递给A的onInterceptTouchEvent方法，A还是不关心该事件，因此onInterceptTouchEvent方法继续返回false。
6. 此时，该MOVE事件将不会再传递给B的onInterceptTouchEvent方法，该方法一旦返回一次true，就再也不会被调用了。事实上，该MOVE以及“手势剩余部分”都将传递给B的onTouchEvent方法（除非A决定拦截“手势剩余部分”）。
7. C再也不会收到该手势（gesture）产生的任何事件了。

如果一个ViewGroup拦截了最初的DOWN事件，该事件仍然会传递到该ViewGroup的onTouchEvent方法中。毕竟down是必须要有人处理的。   
另一方面，如果ViewGroup拦截了一个半路的事件（比如，MOVE），这个事件将会被系统变成一个CANCEL事件，并传递给之前处理该手势（gesture）的子View，而且不会再传递（无论是被拦截的MOVE还是系统生成的CANCEL）给ViewGroup的onTouchEvent方法。只有再到来的事件才会传递到ViewGroup的onTouchEvent方法中。


