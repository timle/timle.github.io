---
layout: default
---


<img src="blank.jpg" name="canvas" style="max-width:100%"/>


<script>
	var imagesArray = ["img_1.png","img_2.png","img_3.png","img_4.png","img_5.png","img_6.png","img_7.png","img_8.png","img_9.png","img_10.png"];


	function displayImage(){
	    var num = Math.floor(Math.random() * (imagesArray.length+1))-1;
	    document.canvas.src = '/images/banner/' + imagesArray[num];
	}
	
	console.log(num)
	
	displayImage();
	
</script>