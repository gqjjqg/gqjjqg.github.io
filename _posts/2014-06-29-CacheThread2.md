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

