<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8"/>
    <link rel="stylesheet" href="croppie.css" />
    <script src="jquery-3.6.0.min.js">
    </script> 
    <script type="text/javascript" src="fabric.min.js">
    </script>
    <link rel="shortcut icon" href="#" />
    <script src="croppie.js">
    </script>
    <style>
      .block{
        width:300px;
        height:350px;
      }
	  #container{
	  width:700px;
	  background-color: #D3D3D3;
	  }
	  #tokenDiv{
		display: inline-block;
		margin-top:0px;
		vertical-align:top;
		width:300px;
	  }
	  h3{
	  margin-left:50px;
	  text-align:center;
	  width:100%;
	  font-size:20px;
	  margin-top:0px;
	  }
	  a{
		margin-left:30px;
	  }
    </style>
  </head>
  <body style="overflow-y:hidden">
	<div id="container">
		<!-- blocco 1-->
		<div id="blocco1" class="block" style="display: inline-block;">
		  <canvas id="c1" width="280" height="280" style="border:5px solid black;">
		  </canvas>
		  <div style="width:100%;margin-top:20px;">
			<input type="button" value="Browse..." onclick="document.getElementById('uploadBtn').click();" />
			<input id="uploadBtn" type="file" style="display: none;">
			<input id="clearBtn" type="button" value="Clear"/>
			<input id="doneBtn" type="button" value="Done" style="margin-left:90px;width:60px;"/>
		  </div>
		</div>
		<!--My tokens-->
		<div id="tokenDiv">
			<h3>Tokens</h3>
			<ul id="linkList">
			
			</ul>
		</div>
	</div>
    <!-- blocco 2-->
    <div id="blocco2" class="block">
      <div class="resultImg" style="width: 560px;height: 560px;">
      </div>
    </div>
    <script>
      var canvas = new fabric.Canvas('c1');
      canvas.setHeight(280);
      canvas.setWidth(280);
      var pedina;
      var image;
      var basic=null;
      //ON LOAD CREA PEDINA
      $(document).ready(function() {
        createPedina();
        $("#blocco2").invisible();
      }
                       );
      function createPedina(){
        fabric.util.loadImage('pedina.png', function (img) {
          pedina = new fabric.Image(img, {
            selectable: false,
            scaleX: canvas.width / img.width,
            scaleY: canvas.height / img.height
          }
                                   );
          canvas.centerObject(pedina);
          canvas.add(pedina);
          canvas.renderAll();
        }
                             );
      }
      //ON UPLOAD
      document.getElementById('uploadBtn').onchange = function handleImage(e) {
		$('#clearBtn').trigger('click');
        var reader = new FileReader();
        reader.onload = function (event){
          var imgObj = new Image();
          imgObj.src = event.target.result;
          imgObj.onload = function () {
            image = new fabric.Image(imgObj);
            image.set({
              angle: 0,
              padding: 10,
              cornersize:10,
              opacity: 0.4,
              scaleY: canvas.height / image.width,
              scaleX: canvas.width / image.width
            }
                     );
            canvas.centerObject(image);
            canvas.add(image);
            canvas.renderAll();
          }
        }
        reader.readAsDataURL(e.target.files[0]);
      }
      //DONE
      $("#doneBtn").click(function(){
        image.opacity=1;
        window.canvas.remove(1);
        canvas.centerObject(pedina);
        canvas.add(pedina);
        canvas.renderAll();
        canvas.discardActiveObject();
        canvas.renderAll();
        let strDataURI = canvas.toDataURL({
          format: 'png',
          width: 280,
          height: 280,
          quality: 1.0,
          multiplier: 2.0,
        }
                                         );
        CreateCroppie(strDataURI);
      }
                         );
      //Create croppie canvas. after that call getImage
      function CreateCroppie(ImgUrl){
        //alert("favij");
        $('.resultImg').croppie('destroy');
        basic = $('.resultImg').croppie({
          viewport: {
            width: 560,
            height: 560,
            type: 'circle',
            viewport: 'original'
          }
          ,
          showZoomer:false,
          mouseWheelZoom: false,
          enableResize: false
        }
                                       );
        basic.croppie('bind', {
          url: ImgUrl,
          points: [5,5,549,549]
        }
                     ).then(function(results){
	
						getImage();
						});
      }
      //CLEAR
      $("#clearBtn").click(function(){
        canvas.clear();
        pedina=null;
        image=null;
        createPedina();
      }
                          );
	function getImage() {
		
		let name=$("#uploadBtn").val().split("fakepath")[1].slice(1);
		$("#linkList").append("<li>"+name+"</li>");
		$('.resultImg').croppie('result', {
          type: 'base64',
          format: 'png',
          size:  'viewport',
          circle: true,
          quality: 1
        }
        ).then(function(croppedbase64) {
	   
			string64toBlob(croppedbase64);
		});
	}
	  
      function string64toBlob(croppedbase64){
        const base64ImageData = croppedbase64;
        const contentType = 'image/png';
        const byteCharacters = atob(base64ImageData.substr(`data:${contentType};base64,`.length));
        const byteArrays = [];
        for (let offset = 0; offset < byteCharacters.length; offset += 1024) {
          const slice = byteCharacters.slice(offset, offset + 1024);
          const byteNumbers = new Array(slice.length);
          for (let i = 0; i < slice.length; i++) {
            byteNumbers[i] = slice.charCodeAt(i);
          }
          const byteArray = new Uint8Array(byteNumbers);
          byteArrays.push(byteArray);
        }
        const blob = new Blob(byteArrays, {
          type: contentType}
                             );
        const blobUrl = URL.createObjectURL(blob);
		$("#linkList").children('li').last().append("<a target='_blank' href='"+blobUrl+"'>"+"[download]"+"</a>");
        //window.open(blobUrl, '_blank');
      }
      (function($) {
        $.fn.invisible = function() {
          return this.each(function() {
            $(this).css("visibility", "hidden");
          }
                          );
        };
        $.fn.visible = function() {
          return this.each(function() {
            $(this).css("visibility", "visible");
          }
                          );
        };
      }
       (jQuery));
function resizedataURL(datas, wantedWidth, wantedHeight,text)
    {
        var img = document.createElement('img');

        img.onload = function()
            {        
                var canvas = document.createElement('canvas');
                var ctx = canvas.getContext('2d');

                canvas.width = wantedWidth;
                canvas.height = wantedHeight;

                ctx.drawImage(this, 0, 0, wantedWidth, wantedHeight);

                var dataURI = canvas.toDataURL();
				//window.open(dataURI);
				$("#linkList").children('li').last().append("<a target='_blank' href='"+dataURI+"'>"+text+"</a>");
            };

        img.src = datas;
    }
    </script>
  </body>
</html>
