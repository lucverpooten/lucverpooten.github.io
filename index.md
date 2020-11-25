<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <meta name="mobile-web-app-capable" content="yes" />
	<!-- 
	Note for Safari users: 
	To launch a web app in full screen mode it must first be added to home screen. This is as easy 
	as tapping the action icon in the status bar, as if to add as a bookmark, but instead tap the 
	button "Add to Home Screen". An icon will be added to your home screen that can be launched 
	just like a native iPhone app. 
	
	Chrome on Windows, Android:
	Create Shortcut on Desktop
	Windows 8 tile: Pin to Start
	-->
	<meta http-equiv="refresh" content="900">
	<link rel="shortcut icon" href="http://ditte.be/wp-content/uploads/2013/09/favoicondef2.jpg"/>	
	
    <title>Ditte.be</title>
    <style>
		body 
		{
		font-size:100%;
		margin:0px;
		border:0px;
		padding:0%;
		background-color:#000000;
		}
		div.container
		{
		margin:0px;
		border:0px;
		padding:0%;
		width:700px;
		height:700px;
		margin-left:auto;
		margin-right:auto;
		}
		img.ditte-image
		{
		height:700px;
		}
    </style>
	<script type='text/javascript' src='http://code.jquery.com/jquery-2.1.0.min.js'></script>
	<script>
	
		window.onload = function(){initialize()};
		
		var url = "http://ditte.be";
		var blogItemsWithPictures = new Array();
		var currentPicture;

		function initialize () {

			//Use jquery's JSONP and the whateverorigin.org server to get around the "same origin" restriction while loading the page from the URL.
			//If the URL includes the string "callback=?" (or similar, as defined by the server-side API), jquery treats the request as JSONP. 
			$.getJSON('http://whateverorigin.org/get?url=' + encodeURIComponent(url) + '&callback=?', function(data){
				var html = "" + data.contents;
				//console.log(html);
				
				//parse the html and collect all <a> with an <img> child
				//add to the <a> element the property target="_blank"
				//strip from the <img src="...IMG_9023-300x300.jpg" ...> the "-300x300"   
				
				parser=new DOMParser();
				htmlDoc=parser.parseFromString(html, "text/html");
				/*var anchors = htmlDoc.getElementsByTagName('a');
				for (var i = 0; i < anchors.length; i++) {
					//console.log(anchors[i].innerHTML);
				} */
				var images = htmlDoc.getElementsByTagName('img');
				//filter for images with a distinguished pattern
				for (var i = 0; i < images.length; i++) {
					//console.log(images[i].parentNode.getAttribute ("href"));
					//console.log(images[i].parentNode.childNodes[1].nodeValue);
					var parentN = images[i].parentNode;
					if (parentN.tagName == 'A') {
						for (var j = 0; j < parentN.childNodes.length; j++) {
							if (parentN.childNodes[j].tagName == "IMG") {
								var anchorHrefStr = parentN.getAttribute ("href");
								//console.log(anchorHrefStr);
								var imageSrcStr = parentN.childNodes[j].getAttribute ("src");
								//console.log(imageSrcStr);
								var pattern;
								//Note: escaping in search patterns needs double backslash
								//the first backslash is for escaping the backslash in javascript string literals
								pattern = new RegExp("^http://ditte\\.be/\\?p=","i");
								var resultStartsWithHttpDitteBeSlashQuestionPEqual = pattern.test(anchorHrefStr);
								pattern = new RegExp("^http://ditte\\.be/","i");
								var resultStartsWithHttpDitteBe = pattern.test(imageSrcStr);
								pattern = new RegExp("-300x300\\.jpg","i");
								var result300by300jpg = pattern.test(imageSrcStr);
								
								//console.log(resultStartsWithHttpDitteBeSlashQuestionPEqual);
								//console.log(resultStartsWithHttpDitteBe);
								//console.log(result300by300jpg);
								
								if (resultStartsWithHttpDitteBeSlashQuestionPEqual && resultStartsWithHttpDitteBe && result300by300jpg) {
									//adapt strings and store as valid hits
									imageSrcStr = imageSrcStr.replace(pattern,"\.jpg");
									//console.log(imageSrcStr);
									var blogItem = new Object();
									blogItem.ahref = anchorHrefStr;
									blogItem.imgsrc = imageSrcStr;
									blogItemsWithPictures.push(blogItem);
								}
							}
						}
					}
				}
				
				//console.log(blogItemsWithPictures);
				//console.log(blogItemsWithPictures[0].ahref);
				//console.log(blogItemsWithPictures[0].imgsrc);

				//start displaying the pictures
				currentPicture = 0;
				nextPicture ();

			});
		}
		
		function nextPicture () {
			if (blogItemsWithPictures.length != 0) {
		        //template: '<a href="http://ditte.be/?p=915" target="_blank"> <img src="http://ditte.be/wp-content/uploads/2014/04/IMG_7649.jpg" class="ditte-image"/> </a>';
				var html = '<a href="' + blogItemsWithPictures[currentPicture].ahref + '" target="_blank"> <img src="' + blogItemsWithPictures[currentPicture].imgsrc + '" class="ditte-image"/> </a>';
				var containerDiv = document.getElementById("contnr");
				containerDiv.innerHTML = html;
				currentPicture = currentPicture + 1;
				if (currentPicture >= blogItemsWithPictures.length) {
					currentPicture = 0;
				}
				setTimeout(nextPicture,15000);
			}
		}
		
		//This polyfill is for browsers that do not support DOMParser for HTML, like at least Safari/Webkit <= 5.1.7
		
		/*
		 * DOMParser HTML extension
		 * 2012-09-04
		 * 
		 * By Eli Grey, http://eligrey.com
		 * Public domain.
		 * NO WARRANTY EXPRESSED OR IMPLIED. USE AT YOUR OWN RISK.
		 */
		 
		/*! @source https://gist.github.com/1129031 */
		/*global document, DOMParser*/
		 
		(function(DOMParser) {
			"use strict";
		 
			var
			  DOMParser_proto = DOMParser.prototype
			, real_parseFromString = DOMParser_proto.parseFromString
			;
		 
			// Firefox/Opera/IE throw errors on unsupported types
			try {
				// WebKit returns null on unsupported types
				if ((new DOMParser).parseFromString("", "text/html")) {
					// text/html parsing is natively supported
					return;
				}
			} catch (ex) {}
		 
			DOMParser_proto.parseFromString = function(markup, type) {
				if (/^\s*text\/html\s*(?:;|$)/i.test(type)) {
					var
					  doc = document.implementation.createHTMLDocument("")
					;
						if (markup.toLowerCase().indexOf('<!doctype') > -1) {
							doc.documentElement.innerHTML = markup;
						}
						else {
							doc.body.innerHTML = markup;
						}
					return doc;
				} else {
					return real_parseFromString.apply(this, arguments);
				}
			};
		}(DOMParser));
				
	</script>
</head>

<body>
    <div id="contnr" class="container">
	</div>
</body>

</html>
