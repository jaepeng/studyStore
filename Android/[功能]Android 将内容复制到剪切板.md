# [功能]Android 将内容复制到剪切板

```java
//获取数据
String code = mTicketCode.getText().toString().trim();
//对数据打印
LogUtils.d(TicketActivity.class,"code:"+code);
//Manager设置
ClipboardManager cm = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
//复制到粘贴表
ClipData clipdata = ClipData.newPlainText("sob_tao_bao_ticket_code", code);
cm.setPrimaryClip(clipdata);
```

