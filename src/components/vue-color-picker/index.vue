<template lang="html">
<div>
	<div class="color-picker-container" :id="containerId" :class="{'active':isShowPicker}">
		<div class="color"></div>
		<div class="wheel"></div>
		<div class="overlay"></div>
		<div class="h-marker marker"></div>
		<div class="sl-marker marker"></div>
	</div>
</div>
</template>

<script>
/**
 * author : alex 
 * email : 961163829@qq.com
 */
export default {
	props:{
		/*由父组件传递的默认颜色*/
		defaultColor:{
			type:String,
			default:"#000000"
		},
		/*目标元素，可以是input框或者按钮等*/
		targetElem:null
	},
	data(){
		return {
			isShowPicker:false,
			wheel:document.querySelector('.wheel'),
			color:this.defaultColor,
			containerId:"color-picker-container",
			dom:{
				hMarker:null,
				slMarker:null,
				color:null,
				targetElem:null,
				container:null
			},
			radius:84,
			square:100,
			width:194
		}
	},
	mounted(){

		this.dom.container=document.querySelector('#'+this.containerId);
		this.dom.hMarker=document.querySelector('.h-marker');
		this.dom.slMarker=document.querySelector('.sl-marker');
		this.dom.color=document.querySelector('.color');
		this.dom.targetElem=document.querySelector(this.targetElem);


		this.init();
	},
	methods:{
		init:function(){
			var self = this;
			self.posInit();
			self.eventBind();

		      // Init color
		    self.setColor(self.color);
		},
		openPicker:function(){
			var self = this;
			this.isShowPicker = true;
			this.wheel=document.querySelector('.wheel');
			/*色盘打开的时候绑定点击事件*/
			document.addEventListener("click",self.documentClick);
		},
		closePicker:function(){
			this.isShowPicker = false;
			/*色盘关闭的时候解绑事件*/
			document.removeEventListener("click",this.documentClick);
		},
		eventBind:function(){
			var self = this;
			self.dom.container.addEventListener("mousedown",self.mousedown);
		},
		documentClick:function(e){
			/*你可能不需要关闭色盘，那这个方法也是可以不需要的*/
			var parents1 = this.getParents(e,this.dom.container,true);;
			var parents2 = this.getParents(e,this.dom.targetElem,true);
			if(parents1.length===0&&parents2.length===0){
				this.closePicker();
			}
		},
		posInit:function(){
			/*色盘的位置计算*/
			var target = this.dom.targetElem;

			var top = this.getElementViewTop(target);
			var left = this.getElementViewLeft(target);

			this.dom.container.style.position = "fixed";
			this.dom.container.style.top = (top)+'px';
			
			this.dom.container.style.left = (left + this.dom.targetElem.offsetWidth)+'px';
			
		},
		getElementViewLeft:function(element){
			/*获取元素距离视窗左部距离*/
	　　　　var actualLeft = element.offsetLeft;
	　　　　var current = element.offsetParent;

	　　　　while (current !== null){
	　　　　　　actualLeft += current.offsetLeft;
	　　　　　　current = current.offsetParent;
	　　　　}

	　　　　if (document.compatMode == "BackCompat"){
	　　　　　　var elementScrollLeft=document.body.scrollLeft;
	　　　　} else {
	　　　　　　var elementScrollLeft=document.documentElement.scrollLeft; 
	　　　　}

	　　　　return actualLeft-elementScrollLeft;
	　　},
		getElementViewTop:function(element){
			/*获取元素距离视窗顶部距离*/
	　　　　var actualTop = element.offsetTop;
	　　　　var current = element.offsetParent;

	　　　　while (current !== null){
	　　　　　　actualTop += current. offsetTop;
	　　　　　　current = current.offsetParent;
	　　　　}

	　　　　if (document.compatMode == "BackCompat"){
	　　　　　　var elementScrollTop=document.body.scrollTop;
	　　　　} else {
	　　　　　　var elementScrollTop=document.documentElement.scrollTop; 
	　　　　}

	　　　　return actualTop-elementScrollTop;
	　　},
		getParents:function(e,parent,andSelf){
			/*获取祖先节点，返回一个数组*/
		    var target = e.target;
		    var parent = typeof parent==='string'?document.querySelector(parent):parent;

		    var curTarget = target;

		    var arr = typeof andSelf === "undefined"?[curTarget]:[];
		    var result = [];
		    
		    while(true){
		        if((typeof parent !== 'undefined'&&curTarget == parent)||
		            typeof parent === 'undefined'&&curTarget.nodeType===9){
		            arr.push(curTarget);
		            break;
		        }
		        if(!!curTarget){arr.push(curTarget)}

		        if(!!curTarget.parentNode){
		            curTarget = curTarget.parentNode
		        }else{
		            break;
		        }
		    }
		    if(!!parent){
		        return arr.indexOf(parent)>-1?arr:[];
		    }else{
		        return arr;
		    }
		},
		getParent:function(e){
			/*获取父节点*/
		    return e.target.nodeType!==9&&e.target.parentNode;
		},
		/**
		 * 以下为组件内部函数
		 * 算法来自网络
		 */
	    updateValue:function (event) {
			var self = this;

			var that = event.target;
			if (that.value && that.value != self.color) {
				self.setColor(that.value);
			}
	  	},
		/**
		* Change color with HTML syntax #123456
		*/
		setColor:function (color) {
			var self = this;
			var unpack = self.unpack(color);
			if (self.color != color && unpack) {

				self.color = color;
				self.rgb = unpack;
				self.hsl = self.RGBToHSL(self.rgb);
				self.updateDisplay();
			}
		},
		/**
		* Change color with HSL triplet [0..1, 0..1, 0..1]
		*/
		setHSL:function (hsl) {
			var self = this;
			self.hsl = hsl;
			self.rgb = self.HSLToRGB(hsl);
			self.color = self.pack(self.rgb);
			self.updateDisplay();
		},
		/**
		* Retrieve the coordinates of the given event relative to the center
		* of the widget.
		*/
	  	widgetCoords:function (event) {
			var self = this;
		    var x, y;
		    var el = event.target || event.srcElement;
		    var reference = self.wheel;
		    
		    if (typeof event.offsetX != 'undefined') {
		      // Use offset coordinates and find common offsetParent
		      var pos = { x: event.offsetX, y: event.offsetY };

		      // Send the coordinates upwards through the offsetParent chain.
		      var e = el;
		      while (e) {
		        e.mouseX = pos.x;
		        e.mouseY = pos.y;
		        pos.x += e.offsetLeft;
		        pos.y += e.offsetTop;
		        e = e.offsetParent;
		      }

		      // Look for the coordinates starting from the wheel widget.
		      var e = reference;
		      var offset = { x: 0, y: 0 }
		      while (e) {
		        if (typeof e.mouseX != 'undefined') {
		          x = e.mouseX - offset.x;
		          y = e.mouseY - offset.y;
		          break;
		        }
		        offset.x += e.offsetLeft;
		        offset.y += e.offsetTop;
		        e = e.offsetParent;
		      }

		      // Reset stored coordinates
		      e = el;
		      while (e) {
		        e.mouseX = undefined;
		        e.mouseY = undefined;
		        e = e.offsetParent;
		      }
		    }
		    else {
		      // Use absolute coordinates
		      var pos = self.absolutePosition(reference);
		      x = (event.pageX || 0*(event.clientX + document.documentElement.scrollLeft)) - pos.x;
		      y = (event.pageY || 0*(event.clientY + document.documentElement.scrollTop)) - pos.y;
		    }
		    // Subtract distance to middle
		    return { x: x - self.width / 2, y: y - self.width / 2 };
		},
		/**
		* Mousedown handler
		*/
		mousedown:function (event) {
			var self = this;
			if (!document.dragging) {
				document.documentElement.addEventListener('mousemove', self.mousemove);
				document.documentElement.addEventListener('mouseup', self.mouseup);
				document.dragging = true;
			}

			// Check which area is being dragged
			var pos = self.widgetCoords(event);
			self.circleDrag = Math.max(Math.abs(pos.x), Math.abs(pos.y)) * 2 > self.square;

			// Process
			self.mousemove(event);
			return false;
		},
		/**
		* Mousemove handler
		*/
		mousemove:function (event) {
			var self = this;
		// Get coordinates relative to color picker center
			var pos = self.widgetCoords(event);

			// Set new HSL parameters
			if (self.circleDrag) {
			var hue = Math.atan2(pos.x, -pos.y) / 6.28;
			if (hue < 0) hue += 1;
				self.setHSL([hue, self.hsl[1], self.hsl[2]]);
			}
			else {
				var sat = Math.max(0, Math.min(1, -(pos.x / self.square) + .5));
				var lum = Math.max(0, Math.min(1, -(pos.y / self.square) + .5));
				self.setHSL([self.hsl[0], sat, lum]);
			}
			return false;
		},
		/**
		* Mouseup handler
		*/
		mouseup:function () {
			var self = this;
			// Uncapture mouse
			document.documentElement.removeEventListener('mousemove', self.mousemove);
			document.documentElement.removeEventListener('mouseup', self.mouseup);
			document.dragging = false;
		},
		/**
		* Update the markers and styles
		*/
		updateDisplay:function () {
			var self = this;
			// Markers
			var angle = self.hsl[0] * 6.28;
			
			self.dom.hMarker.style.left = Math.round(Math.sin(angle) * self.radius + self.width / 2) + 'px';
			self.dom.hMarker.style.top = Math.round(-Math.cos(angle) * self.radius + self.width / 2) + 'px';

			self.dom.slMarker.style.left = Math.round(self.square * (.5 - self.hsl[1]) + self.width / 2) + 'px';
			self.dom.slMarker.style.top = Math.round(self.square * (.5 - self.hsl[2]) + self.width / 2) + 'px';

			// Saturation/Luminance gradient
			self.dom.color.style.backgroundColor = self.pack(self.HSLToRGB([self.hsl[0], 1, 0.5]));


			// important
			// 将获取到的最新的color值emit出去
			self.$emit("onChange",self.color);
			//self.dom.targetElem.style.backgroundColor = self.color;
			//self.dom.targetElem.style.color = self.hsl[2] > 0.5 ? '#000' : '#fff';

			// Change linked value
			// if (self.dom.targetElem.value && self.dom.targetElem.value != self.color) {
			// 	self.dom.targetElem.value = self.color;
			// }
		},
		/**
		* Get absolute position of element
		*/
		absolutePosition:function (el) {
			var self = this;
			var r = { x: el.offsetLeft, y: el.offsetTop };
			// Resolve relative to offsetParent
			if (el.offsetParent) {
				var tmp = self.absolutePosition(el.offsetParent);
				r.x += tmp.x;
				r.y += tmp.y;
			}
			return r;
		},
		/* Various color utility functions */
		pack:function (rgb) {
			var r = Math.round(rgb[0] * 255);
			var g = Math.round(rgb[1] * 255);
			var b = Math.round(rgb[2] * 255);
			return '#' + (r < 16 ? '0' : '') + r.toString(16) +
						 (g < 16 ? '0' : '') + g.toString(16) +
						 (b < 16 ? '0' : '') + b.toString(16);
		},
		unpack:function (color) {
			if (color.length == 7) {
				return [parseInt('0x' + color.substring(1, 3)) / 255,
						parseInt('0x' + color.substring(3, 5)) / 255,
						parseInt('0x' + color.substring(5, 7)) / 255];
			}
			else if (color.length == 4) {
				return [parseInt('0x' + color.substring(1, 2)) / 15,
						parseInt('0x' + color.substring(2, 3)) / 15,
						parseInt('0x' + color.substring(3, 4)) / 15];
			}
		},
		HSLToRGB:function (hsl) {
			var m1, m2, r, g, b;
			var h = hsl[0], s = hsl[1], l = hsl[2];
			m2 = (l <= 0.5) ? l * (s + 1) : l + s - l*s;
			m1 = l * 2 - m2;
			return [this.hueToRGB(m1, m2, h+0.33333),
				    this.hueToRGB(m1, m2, h),
				    this.hueToRGB(m1, m2, h-0.33333)];
		},
		hueToRGB:function (m1, m2, h) {
			h = (h < 0) ? h + 1 : ((h > 1) ? h - 1 : h);
			if (h * 6 < 1) return m1 + (m2 - m1) * h * 6;
			if (h * 2 < 1) return m2;
			if (h * 3 < 2) return m1 + (m2 - m1) * (0.66666 - h) * 6;
			return m1;
		},
		RGBToHSL:function (rgb) {
			var min, max, delta, h, s, l;
			var r = rgb[0], g = rgb[1], b = rgb[2];
			min = Math.min(r, Math.min(g, b));
			max = Math.max(r, Math.max(g, b));
			delta = max - min;
			l = (min + max) / 2;
			s = 0;
			if (l > 0 && l < 1) {
				s = delta / (l < 0.5 ? (2 * l) : (2 - 2 * l));
			}
			h = 0;
			if (delta > 0) {
				if (max == r && max != g) h += (g - b) / delta;
				if (max == g && max != b) h += (2 + (b - r) / delta);
				if (max == b && max != r) h += (4 + (r - g) / delta);
				h /= 6;
			}
			return [h, s, l];
		}
	}
}
</script>

<style lang="css" scoped>
.color-picker-container {
	position: fixed;
	display: none;
  
}
.color-picker-container.active{
	display: block;
}
.color-picker-container * {
	position: absolute;
	cursor: crosshair;
}
.color-picker-container, .color-picker-container .wheel {
	width: 195px;
	height: 195px;
}
.color-picker-container .color, .color-picker-container .overlay {
	top: 47px;
	left: 47px;
	width: 101px;
	height: 101px;
}
.color-picker-container .wheel {
	width: 195px;
	height: 195px;
	background: url(./images/wheel.png) no-repeat;
	-webkit-background-size: 100%;
	background-size: 100%;
}
.color-picker-container .overlay {
	background: url(./images/mask.png) no-repeat;
	-webkit-background-size: 100%;
	background-size: 100%;
}
.color-picker-container .marker {
	width: 17px;
	height: 17px;
	margin: -8px 0 0 -8px;
	overflow: hidden;
	background: url(./images/marker.png) no-repeat;
	-webkit-background-size: 100%;
	background-size: 100%;
}
</style>
