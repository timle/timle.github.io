---
layout: default
---





<img src="blank.jpg" name="canvas" style="max-width:100%"/>


<script>
	var imagesArray = ["b3.jpg","b6.jpg","b8.jpg","b9.jpg","b10.jpg","b11.jpg","b13.jpg"];


	function displayImage(){


	    var num = Math.floor(Math.random() * (imagesArray.length+1));
	    document.canvas.src = '/images/banner/' + imagesArray[num];


	}
	
	displayImage();
	
</script>