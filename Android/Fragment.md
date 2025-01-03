# Fragment
- 生命周期
    - onAttach -> onCreate -> onCreateView -> onActivityCreated -> onStart -> onResume
    - onPause -> onStop -> onDestroyView -> onDestroy -> onDetach
    - FragmentActivity会在onStart()回调中：1、首先执行（第一次调用的时候）mFragments.dispatchActivityCreated()（Fragment.onAttach()/onCreate()/onCreateView()/onActivityCreated()）；2、然后执行mFragments.dispatchStart()（即回调Fragment.onStart()）。这两个方法最终都是通过FragmentManagerImpl.moveToState()来完成回调的，该方法会根据期望State和Fragment.state做一个比较，然后依次执行回调，使Fragment.state最终到达期望State(一步一步回调到期望State)。
- 原理
    - Fragment的添加/移除等
        - Activity对Fragment的逻辑主要在FragmentActivity，其中操作都交由FragmentController(在FragmentActivity成员域中被初始化)完成并传入了FragmentActivity的内部类HostCallbacks来帮助回调。
        - 1、FragmentActivity.getSupportFragmentManager()返回的是FragmentManagerImpl；2、 FragmentManager. beginTrasaction()返回的是BackStackRecord()。
        - BackStackRecord.add()/remove()/attach()/detach()等对Fragment的操作：会通过BackStackRecord存储在一个Op的集合中。
        - BackStackRecord. commit()：最终会调用FragmentManager.enqueueAction()，把自己存储在FM中
        - Fragment.enqueueAction()：将BackStackRecord实现的接口存储在PendingActions（OpGenerator被BSR实现）成员中，之后通过Handler在主线程运行execPendingActions()方法
        - FM.execPendingActions()：该方法循环PendingActions，最终会生成一个集合存储BSR，然后循环BSR集合，去调用BSR.executeOps()，最后会清空PendingActions集合
        - BSR.executeOps()： 该方法会循环BSR内部的Op集合，根据Op的操作码执行对应的FM方法（如addFragment），然后调用FM. moveToState()来是Fragment根据当前FM状态（与Activity当前生命周期有关）来回调到目标生命周期
        - FM.xxxFragment()：会把Fragment加入/移除到集合mAdded中，然后改变其状态。Fragment通过mAdded集合来持有。addFragment()方法还会额外把Fragment加入到mActive容器中，只有处于mActive中的Fragment才会被回调生命周期。
    - Fragment的声明周期触发
        - 1、FragmentManager的生命周期通过FragmentActivity/父级Fragment调用dispatchXXX()(在Activity/Fragment对应生命周期调用)来调整。2、dispatchXXX()最终通过调用moveToState()完成，该方法遍历了mAdded和mActive去循环获取Fragment调用moveFragmentToExpectedState()。3、moveFragmentToExpectedState()会做判断对不处于mActive()中的Fragment会直接返回，然后对处于mActive的Fragment最终调用另一个重载的moveToState()。
        - 最终调用的moveToState()：1、当Fragment要remove时，根据是否要进入后退栈来改变期望state（如果要入回退栈，期望周期为CREATED，否则为INITIALIZING）2、根据Fragment.mMaxState再次调整期望state（前两点会影响生命周期调用）。3、比较期望state和Fragment.mState，并根据swtich-case一步一步回调到目标state。
        - 当前状态
            - INITIALIZING：期望大于该状态则1、通过saveFragmentState恢复一些参数。2、f.HostCallback设置为自己的HostCallback，调用onAttach()和onCreate()
            - CREATED：期望状态大于该状态，则会1、获取Fragment的外在容器Container，调用onCreateView()初始化自身的mView。2、如果创建了mView则调用onViewCreated()并传入saveState去通知View创建完成和完成重建，并将该View添加到Container中。3、调用onActivityCreated()并传入saveState。4、mView不为空则调用restoreViewState()恢复View状态；如果期望状态小于该状态则1、回调onDestroy()和onDetach()。2、判断并把Fragment从mActive中移除
            - ACTIVITY_CREATED：期望状态大于该状态，则回调onStart()；期望状态小于该状态则1、mView不为空，保存mView状态。2、回调onDestroyView()。3、Container移除mView，之后Container和mView置空
            - STARTED：期望状态大于该状态，则回调onResume()；期望状态小于该状态则回调onStop()
            - RESUMED：期望状态小于该状态，则回调onPause()。
    - 其他
        - 1、onCreateView回调前，会根据f.containerId获取容器（通过调用FragmentActivity.findViewById()）。2、然后通过回调onCreateView创建f.view。3、使得f.innerView = f.view； containerView.add(f.view)，然后回调onViewCreated()。结论：当回调onViewCreated()时，f.innerView会有值且等于f.view，并且此时该View已经被添加到ContainerView中了。
        - Fragment.mHidden仅仅会影响mView是否要显示，具体使用setVisibility()设置其显示
        - Fragment的懒加载方案：FragmentStatePagerAdapter/FragmentPagerAdaper都弃用了 f.se tUserVisbleHint()的方法，采用 BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT行为，该行为采用 BSR.se tMaxLifecycle()来限制Fragment最大的生命周期，会影响到moveToState()的第2步，通过影响该方法中的期望state，来使得Fragment的声明周期受到控制。采取该方案后，可以在f.onResume()进行可见操作，f.onPause()中进行不可见操作，而生命周期受到限制的Fragment，最多只会回调到onStart()和从onStop()开始回调。
        - FragmentStatePagerAdapter会使得滑动超出缓存容量的Fragment回调到onDetach()。而FragmentPagerAdapter只会回调到onDestroyView()
        - 1、mAdded用于记录挂载的Fragment，add/remove/attach/detach都会影响这个容器；2、mActive用于记录活跃的Fragment，处于mActive中的Fragment才会被回调生命周期。add()方法会影响这个容器，而remove()方法会修改mRemoving变量为true，使得moveToState()中Fragment的期望生命周期被调整为INITIALIZING(不进入回退栈时)，使得在最后一步将Fragment从这个容器中移除。