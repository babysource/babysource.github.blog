---
layout: false
---
<html>
	<head>
		<link href="http://fonts.googleapis.com/css?family=Open+Sans:regular,semibold,italic,italicsemibold|PT+Sans:400,700,400italic,700italic|PT+Serif:400,700,400italic,700italic" rel="stylesheet" />
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
		<meta name="viewport" content="width=1024"/>
		<meta name="apple-mobile-web-app-capable" content="yes"/>
		<link href="./about.css" rel="stylesheet"/>
		<script language="javascript" type="text/javascript" src="../js/impress.min.js"></script>
		<title>关于我</title>
	</head>
	<body class="impress-not-supported">
		<div class="fallback-message">
		    <p>您的浏览器 <b>不满足</b> impress.js的展示要求, 将为您呈现简化版的展示</p>
		    <p>为了获取最佳效果，请使用最新版的<b>Chrome</b>, <b>Safari</b> 或 <b>Firefox</b> 浏览器</p>
		    <p>珍爱生命，保护地球，远离IE。</p>
		</div>
		<div id="impress">
			<div id="welcome" class="step" data-x="-1000" data-y="-2500">
		        <p>欢迎光临</p>
		        <p><span class="blogname"><b>弦 外 之 影</b></span></p>
		        <p><span class="blognote">—&nbsp;&nbsp;<a href="https://babysource.github.io">Wythe's Blog</a>&nbsp;&nbsp;—</span></p>
		    </div>
		    <div id="overview" class="step" data-x="3000" data-y="1500" data-scale="10"></div>
		</div>
		<div class="hint">
		    <p><b>使用空格或方向键继续</b></p>
		</div>
	</body>
</html>
<script>
	impress().init();
	if ("ontouchstart" in document.documentElement) { 
	    document.querySelector(".hint").innerHTML = "<p><b>在左侧或右侧点击继续</b></p>";
	}	
</script>