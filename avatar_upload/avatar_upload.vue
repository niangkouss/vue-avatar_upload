<template>
	<div id="avatarUpload">
		<div class="canvas-area" @mousedown="clickMask($event)" @mousemove="moveMask($event)"
		     @mouseup="leaveBox($event)" ref="canvasArea">
			<div class="canvas-combination">
				<canvas id="bottomCanvas" :width="this.canvasShowLength" :height="this.canvasShowLength"
				        style="border:1px solid #c3c3c3;box-sizing: border-box;" ></canvas>
				<canvas id="maskCanvas" :width="this.canvasShowLength" :height="this.canvasShowLength"
				        style="border:1px solid #c3c3c3;box-sizing: border-box;" :class="{bg:!hasPic}"></canvas>
			</div>
			<div class="selected-box" ref="selectedBox" v-show="hasPic">
				<div class="cursor-move" ref="cursorMove"></div>
				<span class="scale-mask" id="topBorder"></span>
				<span class="scale-mask" id="leftBorder"></span>
				<span class="scale-mask" id="rightBorder"></span>
				<span class="scale-mask" id="bottomBorder"></span>
				<i class="scale-mask" id="num1"></i>
				<i class="scale-mask" id="num2"></i>
				<i class="scale-mask" id="num3"></i>
				<i class="scale-mask" id="num4"></i>
			</div>
			<canvas id="saveCanvas" :width="this.saveCanvasLength" :height="this.saveCanvasLength"
			        style="border:1px solid #c3c3c3;box-sizing: border-box;"></canvas>
		</div>
		<div class="button-box">
			<input type="file" class="choose-pic" accept="image/gif, image/jpeg, image/jpg, image/png" @change="uploadImg($event)" style="cursor: pointer"
			       ref="choosePic">
			<a href="javascript:;" class="choose-btn">选择图片</a>
			<a href="javascript:;" class="clip-btn" @click="saveAvatar">裁剪</a>
		</div>
	</div>
</template>
<script type="text/ecmascript-6">
	export default {
		data() {
			return {
				hasPic: false,
				maskCxt: null,
				bottomCtx: null,
				imgWidth: 0,
				imgHeight: 0,
				limitLength: 0,
				dashedDefaultLength: 200,
				isClick: false,
				isScale: false,
				isFirst: false,
				targetId: '',
				mouseX: 0,
				mouseY: 0,
				offsetBoxX: 0,
				offsetBoxY: 0,
				scaleX: 0,
				scaleY: 0,
				leftMaskX: 0,
				topMaskY: 0,
				moveOffsetX: 0,
				moveOffsetY: 0,
				target: 0,
				targetWidth: 0,
				targetHeight: 0,
				targetLeft: 0,
				targetTop: 0,
				moveTop: 0,
				resultCode: null

			}
		},
		props: {
			value: {
				type: String
			},
			canvasShowLength: {
				default: 200
			},
			saveCanvasLength: {
				default: 100
			},
			limitFileSize: {
				default: 1
			},
		},
		methods: {
			uploadImg(evt) {
				if( !evt || !evt.target || !evt.target.files[0] ){
					return;
				}
				let file = evt.target.files[0];
				if (file.size / 1024 > this.limitFileSize * 1000) {
					this.$MessageBox({content: `请选择小于${this.limitFileSize}MB的图片`, status: 'fail'});
					return;
				}
				if (file.length == 0) {
					this.$MessageBox({content: `请选择图片`, status: 'fail'});
					return;
				} else {
					let reader = new FileReader();
					reader.readAsDataURL(file);
					reader.onload = (e) => {
						let imgUrl = e.target.result;
						this.canvasShow(imgUrl);
					}
				}
			},
			canvasShow(src) {
				let img = new Image();
				this.bottomCtx = document.getElementById('bottomCanvas').getContext('2d');
				this.bottomCtx.clearRect(0, 0, this.canvasShowLength, this.canvasShowLength);
				img.src = src;
				img.onload = () => {
					this.hasPic = true;
					this.imgWidth = img.width;
					this.imgHeight = img.height;
					let bigger = this.imgWidth - this.imgHeight,
						ratio = null,
						showArea = null;
					if (bigger > 0) {
						ratio = this.imgWidth / this.canvasShowLength;
						showArea = this.imgHeight / ratio;
						this.limitLength = showArea;
						let yAxis = (this.canvasShowLength - showArea) / 2;
						showArea < this.dashedDefaultLength ? this.dashedDefaultLength = showArea : void 0;
						let xAxis = this.canvasShowLength / 2 - this.dashedDefaultLength / 2;
						$(this.$refs.selectedBox).width(this.dashedDefaultLength);
						$(this.$refs.selectedBox).height(this.dashedDefaultLength);
						$(this.$refs.selectedBox).css("left", xAxis);
						$(this.$refs.selectedBox).css("top", yAxis);
						this.bottomCtx.drawImage(img, 0, yAxis, this.canvasShowLength, showArea);
						this.drawMask(xAxis, yAxis);
					} else {
						ratio = this.imgHeight / this.canvasShowLength;
						showArea = this.imgWidth / ratio;
						this.limitLength = showArea;
						let xAxis = (this.canvasShowLength - showArea) / 2;
						showArea < this.dashedDefaultLength ? this.dashedDefaultLength = showArea : void 0;
						let yAxis = this.canvasShowLength / 2 - this.dashedDefaultLength / 2;
						$(this.$refs.selectedBox).width(this.dashedDefaultLength);
						$(this.$refs.selectedBox).height(this.dashedDefaultLength);
						$(this.$refs.selectedBox).css("left", xAxis);
						$(this.$refs.selectedBox).css("top", yAxis);
						this.bottomCtx.drawImage(img, xAxis, 0, showArea, this.canvasShowLength);
						this.drawMask(xAxis, yAxis);
					}

					$(this.$refs.cursorMove).css({
						"height": this.dashedDefaultLength - 8,
						"width": this.dashedDefaultLength - 8,
					});

				}
			},
			drawMask(leftX, topY) {
				this.maskCxt.beginPath();
				this.maskCxt.clearRect(0, 0, this.canvasShowLength, this.canvasShowLength);
				this.maskCxt.fillStyle = 'rgba(0,0,0,0.4)';
				this.maskCxt.fillRect(0, 0, this.canvasShowLength, this.canvasShowLength);
				if (this.isScale) {
					let $ex = $(this.$refs.selectedBox).width(),
						$ey = $(this.$refs.selectedBox).height(),
						Y = Math.abs(this.scaleY),
						X = Math.abs(this.scaleX);
					Y < X ? this.scaleY = this.scaleX : void 0;
					switch (this.targetId) {
						case 'num1':
							this.maskCxt.clearRect(this.targetLeft + this.scaleY, this.targetTop + this.scaleY, $ex, $ey);
							break;
						case 'num2':
							this.maskCxt.clearRect(this.targetLeft, this.targetTop - this.scaleX, $ex, $ey);
							break;
						case 'num3':
							this.maskCxt.clearRect(this.targetLeft + this.scaleX, this.targetTop, $ex, $ey);
							break;
						case 'num4':
							this.maskCxt.clearRect(this.targetLeft + this.scaleX - this.scaleX, this.targetTop, $ex, $ey);
							break;
					}
					this.maskCxt.closePath();
					return;
				}
				if (this.isClick) {
					let $boxWidth = $(this.$refs.selectedBox).width(),
						$boxHeight = $(this.$refs.selectedBox).height();
					this.maskCxt.clearRect(this.leftMaskX, this.topMaskY, $boxWidth, $boxHeight);
					this.maskCxt.closePath();
					return
				}
				let $boxWidth = $(this.$refs.selectedBox).width(),
					$boxHeight = $(this.$refs.selectedBox).height();
				this.maskCxt.clearRect(leftX, topY, $boxWidth, $boxHeight);
				this.maskCxt.closePath();
			},
			clickMask(evt) {
				if ($(evt.target).hasClass('cursor-move')) {
					this.isClick = true;
					this.offsetBoxX = evt.offsetX;
					this.offsetBoxY = evt.offsetY;
					return;
				}
				if ($(evt.target).hasClass('scale-mask')) {
					this.isScale = true;
					this.targetId = $(evt.target).attr('id');
					this.mouseX = evt.x;
					this.mouseY = evt.y;
				}
			},
			moveMask(evt) {
				if (this.isClick) {
					let leftBox = parseFloat(this.$refs.selectedBox.style.left),
						topBox = parseFloat(this.$refs.selectedBox.style.top),
						$boxWidth = $(this.$refs.selectedBox).width(),
						$boxHeight = $(this.$refs.selectedBox).height();
					this.moveOffsetX = evt.offsetX - this.offsetBoxX;
					this.moveOffsetY = evt.offsetY - this.offsetBoxY;
					this.leftMaskX = leftBox + this.moveOffsetX;
					this.topMaskY = topBox + this.moveOffsetY;
					this.leftMaskX <= 0 ? this.leftMaskX = 0 : this.leftMaskX >= this.canvasShowLength - $boxWidth ? this.leftMaskX = this.canvasShowLength - $boxWidth : void 0;
					this.topMaskY <= 0 ? this.topMaskY = 0 : this.topMaskY >= this.canvasShowLength - $boxHeight ? this.topMaskY = this.canvasShowLength - $boxHeight : void 0;
					this.$refs.selectedBox.style.left = this.leftMaskX + 'px';
					this.$refs.selectedBox.style.top = this.topMaskY + 'px';
					this.drawMask();
				}
				if (this.isScale) {
					this.scaleX = evt.x - this.mouseX;
					this.scaleY = evt.y - this.mouseY;
					this.drawMask();
					this.scaleMaskBox();
				}
			},
			scaleMaskBox() {
				if (!this.isFirst) {
					this.target = $(this.$refs.selectedBox);
					this.targetLeft = parseFloat(this.$refs.selectedBox.style.left);
					this.targetTop = parseFloat(this.$refs.selectedBox.style.top);
					this.targetWidth = this.target.width();
					this.targetHeight = this.target.height();
					this.isFirst = true;
					this.moveTop = $(this.$refs.cursorMove).width();
				}
				let Y = Math.abs(this.scaleY),
					X = Math.abs(this.scaleX);
				Y < X ? this.scaleY = this.scaleX : void 0;
				switch (this.targetId) {
					case 'num1':
						this.target.css({
							"height": this.targetHeight - this.scaleY,
							"width": this.targetWidth - this.scaleY,
							"left": this.targetLeft + this.scaleY,
							"top": this.targetTop + this.scaleY
						});
						$(this.$refs.cursorMove).css({
							"height": this.moveTop - this.scaleY,
							"width": this.moveTop - this.scaleY,
						});
						break;
					case 'num2':
						this.target.css({
							"height": this.targetHeight + this.scaleX,
							"width": this.targetWidth + this.scaleX,
							"top": this.targetTop - this.scaleX
						});
						$(this.$refs.cursorMove).css({
							"height": this.moveTop + this.scaleX,
							"width": this.moveTop + this.scaleX,
						});
						break;
					case 'num3':
						this.target.css({
							"height": this.targetHeight - this.scaleX,
							"width": this.targetWidth - this.scaleX,
							"left": this.targetLeft + this.scaleX,
						});
						$(this.$refs.cursorMove).css({
							"height": this.moveTop - this.scaleX,
							"width": this.moveTop - this.scaleX,
						});
						break;
					case 'num4':
						this.target.css({
							"height": this.targetHeight + this.scaleY,
							"width": this.targetWidth + this.scaleY
						});
						$(this.$refs.cursorMove).css({
							"height": this.moveTop + this.scaleY,
							"width": this.moveTop + this.scaleY,
						});
						break;
				}

			},
			leaveBox(evt) {
				this.isClick = false;
				this.isScale = false;
				this.scaleX = 0;
				this.scaleY = 0;
				this.isFirst = false;
			},
			saveAvatar() {
				if( !this.hasPic ){
					this.$MessageBox({content: `请选择图片`, status: 'fail'});
					return;
				}
				let srcX = this.$refs.selectedBox.offsetLeft,
					srcY = this.$refs.selectedBox.offsetTop,
					sWidth = this.$refs.selectedBox.offsetWidth,
					sHeight = this.$refs.selectedBox.offsetHeight,
					imgData = this.bottomCtx.getImageData(srcX, srcY, sWidth, sHeight),
					tempCanvas = document.createElement("canvas"),
					tempcxt = tempCanvas.getContext("2d"),
					img3 = new Image();
				tempCanvas.width = sWidth;
				tempCanvas.height = sHeight;
				tempcxt.putImageData(imgData, 0, 0, 0, 0, tempCanvas.width, tempCanvas.height);
				//let img2 = tempCanvas.toDataURL("image/png");
				let img2 = tempCanvas.toDataURL("image/jpeg", 0.8);
				let savecxt = saveCanvas.getContext("2d");
				savecxt.clearRect(0, 0, saveCanvas.width, saveCanvas.height);
				img3.src = img2;
				img3.onload = () => {
					savecxt.drawImage(img3, 0, 0, saveCanvas.width, saveCanvas.height);
					//let baseCode = saveCanvas.toDataURL("image/png");
					let baseCode = saveCanvas.toDataURL("image/jpeg", 0.8);
					let resultSlice = baseCode.indexOf("base64,") + 7;
					this.resultCode = baseCode.slice(resultSlice);//截取后
					this.$emit('input', this.resultCode);
				}
			},
		},
		mounted() {
			$(this.$refs.choosePic).hover(() => {
				$(this.$refs.choosePic).addClass("hover");
			}, () => {
				$(this.$refs.choosePic).removeClass("hover");
			}).click(() => {
				$(this.$refs.choosePic).addClass("active");
				setTimeout(() => {
					$(this.$refs.choosePic).removeClass("active");
				}, 100);
			});
			$('.canvas-combination').width(this.canvasShowLength + 30);
			$('.canvas-combination').height(this.canvasShowLength + 2);

			this.dashedDefaultLength = this.canvasShowLength / 2;
			this.maskCxt = document.getElementById('maskCanvas').getContext('2d');
			this.leftMaskX = parseFloat(this.$refs.selectedBox.style.left);
			this.topMaskY = parseFloat(this.$refs.selectedBox.style.top);
		}
	}
</script>
<style lang="scss" scoped>
	#avatarUpload {
		-webkit-user-select: none;
		-moz-user-select: none;
		-o-user-select: none;
		user-select: none;
		.canvas-area {
			position: relative;
			.canvas-combination {
				position: relative;
				display: inline-block;
				vertical-align: bottom;
				#bottomCanvas {
					position: absolute;
					top: 0;
					left: 0;
				}
				#maskCanvas {
					position: absolute;
					top: 0;
					left: 0;

				}
				.bg{
					background: url('./img/avatarbg.gif') center no-repeat;
				}
			}
			.selected-box {
				position: absolute;
				cursor: all-scroll;
				.cursor-move {
					width: 92px;
					height: 92px;
					position: absolute;
					top: 4px;
					left: 4px;
					z-index: 999;
				}
				#topBorder {
					width: 100%;
					height: 5px;
					border-top: 1px dashed #186AE5;
					position: absolute;
					top: 0;
					left: 0;
					cursor: n-resize;
					z-index: 999;
				}
				#leftBorder {
					width: 5px;
					height: 100%;
					border-left: 1px dashed #186AE5;
					position: absolute;
					top: 0;
					left: 0;
					cursor: w-resize;
					z-index: 999;
				}
				#rightBorder {
					width: 5px;
					height: 100%;
					border-right: 1px dashed #186AE5;
					position: absolute;
					top: 0;
					right: -1px;
					cursor: e-resize;
					z-index: 999;
				}
				#bottomBorder {
					width: 100%;
					height: 5px;
					border-bottom: 1px dashed #186AE5;
					position: absolute;
					bottom: -1px;
					left: 0;
					cursor: s-resize;
					z-index: 999;
				}
				i {
					position: absolute;
					display: block;
					width: 6px;
					height: 6px;
					border-radius: 1px;
					background-color: #186AE5;
					z-index: 999;
				}
				#num1 {
					top: -2px;
					left: -2px;
					cursor: nw-resize;
				}
				#num2 {
					top: -2px;
					right: -2px;
					cursor: ne-resize;
				}
				#num3 {
					bottom: -2px;
					left: -2px;
					cursor: sw-resize;
				}
				#num4 {
					bottom: -2px;
					right: -2px;
					cursor: se-resize;
				}
			}
			#saveCanvas {
				display: inline-block;
				vertical-align: bottom;
			}
		}
		.button-box {
			position: relative;
			margin-top: 20px;
			.choose-pic {
				position: absolute;
				top: 0;
				left: 0;
				width: 97px;
				height: 30px;
				font-size: 0;
				opacity: 0;
				cursor: pointer;
				z-index: 9999;
				&.hover + .choose-btn {
					border-radius: 3px;
					background: #186AE5;
				}
				&.active + .choose-btn {
					border-radius: 3px;
					box-shadow: inset 0 2px 6px 0 rgba(7, 70, 165, 0.80);
					background: #186AE5;
				}
			}
			.choose-btn {
				display: inline-block;
				height: 30px;
				width: 97px;
				text-align: center;
				line-height: 30px;
				margin-right: 10px;
				color: #fff;
				background: #3380F4;
				border-radius: 3px;
				transition: all 0.2s;
				-webkit-transition: all 0.2s;

			}
			.clip-btn {
				display: inline-block;
				height: 30px;
				width: 76px;
				text-align: center;
				line-height: 30px;
				margin-right: 2px;
				color: #fff;
				background: #3380F4;
				border-radius: 3px;
				transition: all 0.2s;
				-webkit-transition: all 0.2s;
				&:hover {
					border-radius: 3px;
					background: #186AE5;
				}
				&:active {
					border-radius: 3px;
					box-shadow: inset 0 2px 6px 0 rgba(7, 70, 165, 0.80);
					background: #186AE5;
				}
			}
		}

	}

</style>
