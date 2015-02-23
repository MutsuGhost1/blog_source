title: Check equals for Objects
date: 2015-02-05 20:22:49
tags: [Programming, Pitfall]
---

在網路上常會看到一種 check object 是否 equal 的做法,
<!--more-->
一般的做法如下:

	// Before
	String name;
	
	/// 等於 sss 就不會等於 null
	if (name != null && name.equals("sss")) {
	   //do something
    }

為了簡化判斷式, 通常會簡化為:

	// After
	String name;
	
	/// 等於 sss 就不會等於 null
	if ("sss".equals(name)) {
	   //do something
    }

但是如果要 check object 是否 !equal 的做法, 則不能這樣套用.

	// Before
	String name;
	
	/// 不等於 sss 且不等於 null
	if (name != null && !name.equals("sss")) {
	   //do something
    }

並不等價於下列:

	// After
	String name;
	
	// 不等於 sss 還是有可能等於 null
	if (!"sss".equals(name)) {
	   //do something
    }

發現有些人會亂套用, 稍微動一下腦就知道不一樣了.
