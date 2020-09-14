#Activity
---
###Activity生命周期
* 正常情况
    ```flow
    begain=>start: Activity启动
    running=>start: Activity运行
    stop=>start: Activity销毁
    onCreate=>operation: onCreate
    onStart=>operation: onStart
    onResume=>operation: onResume
    onPause=>operation: onPause
    onStop=>operation: onStop
    onDestroy=>operation: onDestroy
    onRestart=>operation: onRestart
    cond=>condition: 判断框(是或否?)
    sub1=>subroutine: 子流程 
    io=>inputoutput: 输入输出框
    e=>end: 结束框
    begain->onCreate->onStart->onResume->running->onPause->onStop->onDestroy->stop  

    onPause(right)->onRestart
    onRestart(right)->onStart
    cond(yes)->io->e
    cond(no)->sub1(right)->onCreate
    ```