## Activity：
* 启动 Activiy：onCreate -> onStart() -> onResume(), Activity 进入运行状态
* Activity 退居后台 ( Home 或启动新 Activity ): onPause() -> onStop()
* Activity 返回前台: onRestart() -> onStart() -> onResume()
* Activity 后台期间内存不足情况下当再次启动会重新执行启动流程
* 锁屏: onPause() -> onStop()
* 解锁: onStart() -> onResume()

## Fragment:
* 创建 Fragment: onAttach() -> onCreate() -> onCreateView() -> onActivityCreate() -> onStart() -> onResume()
* 销毁 Fragment: onPause() -> onStop() -> onDestroyView() -> onDestroy() -> onDetach()
* 从返回栈中回到上一个 Fragment: onDestroyView() -> onCreateView() -> onActivityCreated() -> onStart() -> onResume()

![](res/fragment与activity的关系.png)
