---
layout: post
title: Cache&FreshThread[2]
categories: [Development, Project]
---

{{ page.title }}
================
RefClass:</br>
com.guo.android_extend.cache.BitmapMonitor
com.guo.android_extend.cache.BitmapMonitorThread
  
接着介绍getView的性能提升，google之前就有介绍过，性能不够是因为findViewByID，因此建议大家使用Holder的形式，设置TAG，并且复用View。经过自己的试验和网上的公布的测试结果，的确是行之有效的手段。
典型的做法是这样的：

    @Override
		public View getView(int position, View convertView, ViewGroup parent) {
			// TODO Auto-generated method stub
			Holder holder = null;
			if (convertView != null) {
				holder = (Holder) convertView.getTag();
			} else {
				convertView = mLInflater.inflate(R.layout.item_sample, null);
				holder = new Holder();
				holder.siv = (ExtImageView) convertView.findViewById(R.id.imageView1);
				holder.tv = (TextView) convertView.findViewById(R.id.textView1);
				convertView.setTag(holder);
			}

			Data data = mData.get(position);

			holder.tv.setText(data.name);
			holder.siv.setImageResource(data.icon);
			holder.id = data.id;
			
			return convertView;
		}

这样已经足以应付一般情况下的要求。但是现实总是有很多变化和不定因素。比如要显示大量图片的时候，这些图还不一定全都是本地的图的时候，getView时不仅要解码Bitmap还要从网络下载图片。这样仍然会阻塞引起卡顿或者解码导致OOM。

为了应对这样的情况，就需要做延迟解码刷新，延迟下载。在这个项目中，我已经封装了延迟解码刷新和下载的组件。

线程的主要工作流程如下图所示：
<image src="http://gqjjqg.github.io/images/image_07032135.jpg" />

线程中的队列用LinkedHashMap实现，采用了View为Key值，能独立标记Bitmap的String或者ID为Value，这个Value被作为bitmap cache里的Key值。
简单的使用例子如下：
	  // mointor ImageView set as fresh View. Image path set as key for cache.
	  private class Mointor extends BitmapMonitor<ImageView, String> {

    		public Mointor(ImageView view, String id) {
				super(view, id);
				// TODO Auto-generated constructor stub
			}
			// you should implement the bitmap decode.
    		@Override
    		public Bitmap decodeImage() {
    			// TODO Auto-generated method stub
    			super.mBitmap = null;
    			try {
    				BitmapFactory.Options op = new BitmapFactory.Options();    
    		        op.inJustDecodeBounds = true;
    		        super.mBitmap = BitmapFactory.decodeFile(super.mBitmapID, op);
    		        op.inJustDecodeBounds = false;
    		        int h = op.outHeight;  
    		        int w = op.outWidth;  
    		        int beWidth = w / 320;  
    		        int beHeight = h / 240;  
    		        int be = 1;  
    		        if (beWidth < beHeight) {  
    		            be = beWidth;  
    		        } else {  
    		            be = beHeight;  
    		        }  
    		        if (be <= 0) {  
    		            be = 1;  
    		        }  
    		        op.inSampleSize = be;  
    		        super.mBitmap = BitmapFactory.decodeFile(super.mBitmapID, op);  
    		        super.mBitmap = ThumbnailUtils.extractThumbnail(super.mBitmap, 320, 240,  
    		                ThumbnailUtils.OPTIONS_RECYCLE_INPUT);  
    			} catch (Exception e) {
    		    	e.printStackTrace();
    		    }
    			return super.mBitmap;
    		}
			// after decode, you should implement how to fresh the ImageView.
			// isOld means View is update to set another bitmap.
			@Override
			protected void freshBitmap(boolean isOld) {
				// TODO Auto-generated method stub
				super.mView.setImageResource(0);
    			super.mView.setImageBitmap(null);
    			if (!isOld) {
					if (super.mBitmap != null) {
	    				super.mView.setImageBitmap(super.mBitmap);
	    			} else {
	    				super.mView.setImageResource(R.drawable.ic_launcher);
	    			}
    			} else {
    				super.mView.setImageResource(R.drawable.ic_launcher);
    			}
				super.mView.invalidate();
			}

        }
		...............
		// example for getView:
		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			// TODO Auto-generated method stub
			Holder holder = null;
			if (convertView != null) {
				holder = (Holder) convertView.getTag();
			} else {
				convertView = mLInflater.inflate(R.layout.item_sample, null);
				holder = new Holder();
				holder.siv = (ExtImageView) convertView.findViewById(R.id.imageView1);
				holder.tv = (TextView) convertView.findViewById(R.id.textView1);
				convertView.setTag(holder);
			}
			
			Data data = mData.get(position);
			
			holder.tv.setText(data.name);
			//set default image.
			holder.siv.setImageResource(R.drawable.ic_launcher);
			// here to post to the thread.
			mCacheThread.postLoadBitmap(new Mointor(holder.siv, data.path));
			
			return convertView;
		}
	
使用的关键代码都在上面，只要实现decodeImage和freshBitmap两个方法就可以让decode Bitmap完成异步刷新。
这么做的优点显而易见：

在有大量的图需要解码时，不会阻塞UI线程，不会导致因为解码导致的滑动卡住。

有些需要异步下载的情况也可以在freshBitmap时，很容易整合进去，直接post到download线程去做就可以。

这样UI 线程的工作负担就非常少，仅仅需要保持及时刷新和响应用户操作就可以。

更详细的使用，请参考项目中的sample工程。
