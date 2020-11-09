## ANR(android no response)

### 什么是ANR?

> 如果你的APP有一段时间响应不够灵敏，系统就会向用户显示一个对话框，这个对话框称之为应用程序无响应（ANR）对话框

### 什么情况导致ANR

1. Activity/UI线程**5秒**没有响应
2. 前台BroadcastReceiver**10秒**内灭有完成相关处理
3. 后台Broadcast是**60秒**
4. 前台Service**20秒**没有响应
5. 后台Service**200秒**没有响应
6. 用户的input事件**5秒**没有响应
7. 主线程中错误的操作，比如Thread.wait或者Thread.sleep等安卓系统会监视应用程序的响应情况，一旦出现下面两种情况，就会弹出ANR对话框。

### 如何防止ANR发生

1. 使用AsyncTask处理耗时IO操作
2. 设置主线程和子线程的优先级
3. 使用Handler处理子线程结果，而不是使用Thread.wait()和Thread.sleep()来阻塞主线程
4. Activity的onCreate()和onResume()回调尽量避免耗时的代码
5. BroadcastReceiver中onReceive代码也要尽量减少耗时,建议使用IntentService处理。

### 总结:

> **耗时操作都要在子线程里面完成，在通过Handler，AsyncTask等方式更新UI。无论如何都要保证用户界面的流畅度**