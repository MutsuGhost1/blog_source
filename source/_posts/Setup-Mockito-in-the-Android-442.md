title: 'Setup Mockito in the Android 4.4.2'
date: 2014-06-07 21:43:23
categories: Software
tags: [Software Testing, Mock, Android]
---

Android 在 4.X 版本終於有內建 Mock 相關的 Framework 可以使用了, 而且它是使用 Mockito.
Mockito 的使用教學可以看我另一篇文章, Mockito Basic.
這邊要教大家的是在 Android 4.4.2 上面使用 Mockito, 需要注意到的一些眉角.

1. 設定 Dex-Maker 的 Cache 路徑
2. (Optional) 如果 Test Apk 有使用到 ShardUserId, 必須更改 ClassLoader

以下是相關的範例代碼:

{% codeblock %}
@Override
protected void setUp() throws Exception {
    // Workaround1: Space for Mock classes generation
    System.setProperty("dexmaker.dexcache", this.getInstrumentation()
            .getTargetContext().getCacheDir().getPath());
    // Workaround2: For sharedUserId
    Thread.currentThread().setContextClassLoader(getClass().getClassLoader());
}
{% endcodeblock %}