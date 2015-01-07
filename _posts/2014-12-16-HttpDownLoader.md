---
layout: post
title: Http download
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:</br>
~~com.guo.android_extend.network.HttpDownloader~~
~~com.guo.android_extend.network.HttpDownloadThread~~

Http下载部分和Bitmap异步延迟刷新机制是一样的。主要的函数如下：

    /**
    	 * download the object.
    	 * 
    	 * @param url
    	 * @param localdir
    	 * @return true if success.
    	 */
    	public boolean syncDownload() {
    		String cache = getLocalDownloadFile();
    
    		try {
    			URL url = new URL(mUrl);
    			HttpURLConnection conn = (HttpURLConnection) url.openConnection();
    			conn.setConnectTimeout(30000);
    			conn.setReadTimeout(30000);
    			conn.setInstanceFollowRedirects(true);
    
    			InputStream is = conn.getInputStream();
    			File file = new File(cache);
    			OutputStream os = new FileOutputStream(file);
    
    			byte[] bytes = new byte[1024];
    			int length = 0;
    			while ((length = is.read(bytes, 0, 1024)) != -1) {
    				os.write(bytes, 0, length);
    			}
    			os.close();
    			is.close();
    			conn.disconnect();
    
    			file.renameTo(new File(getLocalFile()));
    			return true;
    		} catch (Exception ex) {
    			ex.printStackTrace();
    		}
    		return false;
    	}
~~处理下载的主要函数，可以重载，和decode的调用时机一样，调用结束之后，接着调用finish的接口。使用时，需要实现finush的接口，这个接口会告诉你什么时候下载结束了，是不是下载成功的消息，这里仅仅是进行了简化的处理，无论是网络连接失败还是写文件失败，还是其他IO问题都归结为下载失败。所以需要对网络连接部分和IO进行处理的话，要重新写过。
工程中提供的是简易的下载流程，单线程排队下载，所以如果是网络图片的显示，需要在下载失败时，重新发起下载请求。~~

--------------------------------------
考虑到简易的排队下载流程使用场合不多，对这部分重新进行了调整。

RefClass:</br>
com.guo.android_extend.network.Downloader
com.guo.android_extend.network.DownloaderManager
com.guo.android_extend.network.DownloaderStructure

调整之后 DownloaderManager 内放了一个线程池，默认开启5个下载线程处理5个下载任务。 

**每个Downloader代表了一个下载任务，分配有独立的唯一的ID，这部分需要使用者控制，确保唯一。构造时，需要传入一个Url以及一个本地目录存放下载文件，下载的临时文件名为同目录下cache结尾的文件。 **

**Downloader包含onDownloadUpdate，onTaskOver，onDownloadFinish，三个回调，update时，android上不能直接更新UI，需要用handler等post到主线程更新。**

**onFinsh时代表下载已经结束，但数据仍然还在cache的文件。onTaskOver时，真正结束任务，这里需要注意的是继承时 仍然需要调用父类的接口，这是因为任务结束时需要把任务从DownloaderManager的列表中删除，否则有相同ID的下载任务无法再次提交。**
