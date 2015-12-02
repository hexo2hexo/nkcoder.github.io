title: 记App Store审核被拒的经历
categories: iOS
date: 2015-11-24 23:13:37
tags: app-review
---

我们的app最近一次上架的过程颇费周折，被拒了两次。我试图还原整个过程，包括被拒的原因以及我与`App Review Team`的沟通内容，将本文作为经验教训给看到的朋友一个参考。

<!--more-->

### 1. 第一次被拒

我们的app是`Oct 31, 2015 at 6:34 PM`提交的，第一次审核结果出来是`Nov 10, 2015 at 7:56 AM`，差不多9天。被拒的原因如下：

![app-reject-v2.0.0-1.0](/images/post/app-reject-v2.0.0-1.0.jpg)

被拒的原因说得很详细，就是我们的app描述文字里出现了`逼格`关键字，app的启动页中有一张图片上出现了`逗逼`的关键字。

相关的文字是：

	3. 晒图大厅，一个涵盖了世界的逼格大秀场！
	4. 社区，整合了大部分的海外租房、二手信息，以及海归招聘信息！

涉及的图片是：

![app-reject-v2.0.0-1.1](/images/post/app-reject-v2.0.0-1.1.png)

将文字和图片修改后，需要重新打包提交（因为启动页图片修改了），所以重新排队`Waiting For Review`。这是我们自己的错误，所以没什么可抱怨的。

> 反思：我们这次上线的是V2.0.0版本，是一个大版本，有很多的功能改进和优化，所以介绍性文字和图片有一些调整，但是在提交审核的时候又没有引起足够的重视，一方面可能是之前的提交太顺利了（V2.0.0之前共提交了7个版本，每次都是一次过，每次审核都没有超过7天），导致这次有点大意，另一方面确实是我自己的疏忽，提交之前没有仔细检查。所以，这样的教训也提醒大家，在提交有较多更新的大版本时，最好把更新的部分好好检查一下。

### 2. 第二次被拒

`Nov 10, 2015 at 7:56 AM`第二次提交后，`Nov 18, 2015 at 6:39 AM`有了结果，不过还是被拒：`Metadata Rejected`：

![app-reject-v2.0.0-2.0](/images/post/app-reject-v2.0.0-2.0.jpg)
![app-reject-v2.0.0-2.1](/images/post/app-reject-v2.0.0-2.1.jpg)

被拒的原因是我们提供的演示帐号失效了。我测试了一下，果然如此！！！因为提供的演示帐号带有`test`关键字，肯定是哪个后台管理员（我们得好好聊聊了）误操作把它清除了，真是自作孽，不可活！

含着悲愤的心情，准备好演示帐号后，我在想接下来该怎么办？因为按照被拒的提示，我只需要提供需要的信息，保存，重新提交，review过程会继续，但是网上也有很多朋友说，重新提交就意味着重新排队！最后我还是选择了相信官方的提示，保存，然后重新提交， App的状态变成了`Waiting For Review`。

> 反思：演示帐号失效，应该是最不应该犯的一个低级错误了。我自己吸取的教训就是：1）演示帐号不要带有`test`等可被忽略的关键字；2）以后提交审核时，所有被审核的内容都应该重新测试，检查其有效性，包括演示帐号、app描述、app支持URL等；3）确保线上数据可控，以免被随意修改。

### 3. 与`App Review Team`的沟通

重新提交之后，我忐忑不安地等待了一天半，但是App的状态一直没变。网上有一些朋友的经验是：`Metadata Rejected`的情况下一般很快就会进入`In Review`的状态，有的是立即，有的是两三个小时。那我这个不太正常啊，不会是重新进入排队了吧？！于是我试着发信询问，选择的是[get the status of my app](https://developer.apple.com/contact/app-store/)，询问的内容为：

	Hi App Review Team:

	Yesterday morning, I got a message of "Metadata Rejected" for my app. According to the message, I just need to provide some information(new binary is not required), and save it and then "Submit for Review", you can continue the review. I did that exactly.

	The status of my app is still "Waiting for review" after one and a half days. I just want to know that if you will continue the review or I have to wait for about one week, so that I can prepare for it. It's very kind of you to give some feedback!

第二天就收到了邮件回复，内容如下：

![app-review-reply-1](/images/post/app-review-reply-1.jpg)

他说我reject了app，意思就是修改了上传的程序包并重新提交了！天啊，冤枉啊！于是我觉得有必要再发信澄清一下，这次选择了[get clarification  on an app rejection](https://developer.apple.com/contact/app-store/)，询问的内容为：

	Dear App Review Team:

	Thanks for your replay, but sorry I have to appeal for my app review.

	You reply to me that: "We noticed that you have rejected your app on at least one occasion.". but I didn't. "Metadata Rejected" messages ask me to do the following:

		- Login to iTunes Connect
		- Click on “My Apps”
		- Select your app
		- Click on the app version on the left side of the screen
		- Scroll down to “App Review Information”
		- Provide information in “Demo Account” and/or “Notes” as appropriate
		- Click “Save”
		- Once you’ve completed all changes, click the “Submit for Review” button at the top of the App version information page.

		While your iTunes Connect Application State shows as Metadata Rejected, we don't require a new binary to correct this issue. Once this information is available, we can continue your review.

	And I did exactly following the above steps, I promise. I didn't reject or update the binary, it doesn't make sense because it's not necessary. I don't think it's fair to add our app to the end of the review queue. I'm really confused, please help to clarify, thank you!

我的意思就是我只是更新了一些信息，并没有重新提交程序包（后来想到，程序包的上传是有个时间戳的，那个完全可以证明我没有重新上传呀，只是当时没想到这一点）。还是第二天，不是邮件回复，而是`Itunes Connect`中的`Resolution Center`中有回复，内容如下：

![app-review-reply-2](/images/post/app-review-reply-2.jpg)

意思是说，由于我重新提交了，为了尽快审核，他们需要关闭当前的申诉会话，继续审核；如果我执意申诉，他们就会取消审核过程！好吧，惹不起，那我保持沉默，麻烦您尽快审核吧！

过了差不多4个小时，也是在当天，状态变成了`In Review`，第二天早上，状态成为了`Pending Developer Release`：

![app-v2.0.0-status](/images/post/app-v2.0.0-status.jpg)

其中，从18号到21号那几天就是与`App Review Team`的沟通耗时。

至此，我终于松了一口气！这个版本，从第一次提交，到最后上线，花了整整21天，这是我们预先没有想到的，因此对整体的规划有一定的影响。

> 反思：app被拒时，最好按照官方的提示进行修改，因为每个人被拒的情形都不太一样，所以别人的经验不一定适合，即使后面遇到问题需要沟通，这个也是可以作为证据的嘛。另外，如果对审核的过程不理解、不明白，可以直接与`App Review Team`沟通，他们的回复还是挺及时有效的。我想，如果我没有发信询问，也许在第二次被拒后，还需要再等一周也说不定呢！

