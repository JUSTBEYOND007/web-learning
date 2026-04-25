标准盒模型 和 IE 盒模型（怪异盒模型）最大的区别是 **width / height 是否包含 padding 和 border**，即 width 的计算方式。

标准盒模型
			元素总宽度 = width + padding + border + margin
				width 只表示 content 的宽度
IE 早期使用的盒模型
				元素总宽度 = width + margin
				width = content + padding + border